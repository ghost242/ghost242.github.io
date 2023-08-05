---
layout: post
title: 데이터베이스의 트랜잭션과 데이터 레코드에 대해서
subtitle: 트랜잭션 정책이 데이터 레코드에 미치는 영향
parent: Database
category: ["Database", "Transaction"]
tags: ["Database", "MySQL", "PostgreSQL", "Python", "SQLAlchemy"]
---

## 개요

그 전까지는 미리 정의한 구조 안에 데이터를 던지고 데이터베이스가 알아서 관리해주길 기대하면서 사용했었는데, 언젠가 개발중인 시스템에 문제가 발생해서 그 원인을 연구하고 분석한 경험이 있었다.
이 문서는 그 경험 사례와 그외에 생각할 수 있는 시나리오를 몇 개 들어서 트랜잭션이 뭔지, 어떻게 동작하는건지, 개발자로서 어떻게 이해해야할지를 정리해보려고 한다.

## 바탕이 되는 사례

Python3의 SQLAlchemy를 이용해서 DB를 제어할 때 웹서비스에서 비동기 요청을 처리하는 경우에 종종 서로 다른 스레드에서 같은 레코드에 접근하는 경우에 서로의 동작이 충돌하고, 그에 대한 방어코드가 없는 문제로 시스템이 비주기적으로 프리즈 되는 현상이 발견되었다. 

## 트랜잭션

위의 사례에서 다양한 원인을 가설로 상황을 재현했을때, 결과적으로 트랜잭션의 문제로 이해할 수 있었다. 여기서 트랜잭션이 뭔지 생각해봐야하는데, 트랜잭션이란 DBMS와 클라이언트간에 CRUD 동작이 이뤄지는 동안 다른 클라이언트에의한 쿼리로 데이터의 무결성이 깨지지 않게 하는 원자성을 보장하는 단위로 볼수있다. 비슷하게는 쓰레드의 Lock이 있을 수 있다. 

이런 기능이 필요한 이유는 개발자와 시스템이 알고있는 데이터의 상태가 서로 다른 문제가 있는데, 예를들어 이런 경우가 있다.

ProcA | ProcB | Note
---|---|---
TranA BEGIN | | 
Insert RecordA | TranB BEGIN |
Update RecordA | Select RecordA | !ERROR
TranA COMMIT | TranB COMMIT |

간단하게 순서를 맞춰봤는데, 개발자는 RecordA가 이미 시스템에 있을 것을 예측하고 이 RecordA의 데이터를 가져오려 시도하지만, 아직 트랜잭션 A가 완전히 커밋이 되지 않았기 때문에 트랜잭션 B의 쿼리는 실패하게 된다. 여기서 주의해야 할 점은 Update RecordA 쿼리가 실패하는 것이 아니라는 점이다.

이런 일이 설마 있을까 싶지만 실제로 웹서비스에서는 여러 Request 를 처리하다 보면 서로 다른 요청간에 충돌이 생길 수 있다.

SQLAlchemy 라이브러리를 이용해서 DB를 제어하려고 하면 이 지점에서 오해가 생기기 쉽다. 위의 테이블을 SQLAlchemy 코드형식에 맞춰서 다시 작성해보면 이렇게 볼수있다.

ProcA | ProcB | Note
---|---|---
with Session(engine) as sess: | | 
: with sess.begin(): | with Session(engine) as sess: |
: : sess.add(RecordA) | : with sess.begin(): |
: : sess.commit() | |
: : sess.query(RecordA).update({RecordA.val: 54321}) | : : sess.query(RecordA).all() | 
: : sess.commit() | |

거칠긴 하지만 이 상황에서 Transaction isolation level에 따라 시스템이 멈춰버리거나, 잘못된 데이터가 나오거나 하는 문제가 생기게 된다. 직접 겪었던 문제는 시스템이 멈춰버리는 현상이었다. 

## 사례에 대한 해결 전략

Transaction isolation level이 어떤것인지 찾아보기위해 SQLAlchemy 문서를 뒤져봤는데, wikipedia 문서의 내용을 이렇게 인용하고 있다.

| "The isolation property of the ACID model ensures that the concurrent execution of transactions results in a system state that would be obtained if transactions were executed serially, i.e. one after the other. Each transaction must execute in total isolation i.e. if T1 and T2 execute concurrently then each should remain independent of the other. (via [isolation(Wikipedia)](https://en.wikipedia.org/wiki/Isolation_(database_systems)))" 

이 Isolation level이 문제가 되었던 것으로 파악했는데, `create_engine` 함수를 이용해서 DB와 통신을 시작할 때 별도의 Isolation level을 지정해주지 않으면 기본 값으로 DB dialect에따라 다른 값이 들어갈 수 있다. 이게 무슨 문제를 일으킬 지 예측하기 어렵다는 것이 문제의 핵심이었다.

따라서 이 문제는 Isolation level을 명시적으로 지정해주는 방식으로 해결할 수 있었다. 구체적으로는 위에서 제시했던대로 `create_engine` 함수의 파라미터에 넣었지만, 활용방법에 따라 여러 코드 형태가 있을 수 있다.

### 코드 예제

* create_engine

```python
url = URL.create(
    drivername="mysql+mysqlconnector",
    username="client",
    password="password",
    host="127.0.0.1",
    port=3306,
    database="database",
)
### isolation_level
engine = create_engine(
    url,
    isolation_level="SERIALIZABLE",  # Transaction isolation level을 지정하는 위치
)
### execution_options
engine = create_engine(
    url,
    execution_options={
        "isolation_level": "SERIALIZABLE",  # keyword argument가 아니라 dict 타입으로 옵션을 정의하는 형식도 지원함을 알 수 있다.
    }
)
```

* Connect.execution_options

```python
engine = create_engine(url)

with engine.connect().execution_options(
    isolation_level="SERIALIZABLE",
) as conn:
    with conn.begin():
        ...
```

* Engine.execution_options

```python
engine = create_engine(url)
opt_eng = engine.execution_engine(isolation_level="SERIALIZABLE")
...
```

이렇게 다양한 방식을 지원하는 이유는 core 레벨의 API와 orm 레벨의 API에따라 Transaction을 다루거나 코드를 구성하는 방식이 다르기 때문이다. Transaction을 어떻게 만들고 제어하는지는 다음에 정리해봐야 할 것 같다.

### Isolation level

Isolation Level은 Transaction 동시성에서 여러 쿼리를 어떻게 처리할지에 대한 전략이라고 할 수 있다. MySQL과 PostgreSQL이 약간 달랐늗데, MySQL은 기본적으로 `transaction begin;`이 발생하면 직전에 snapshot을 만들고 이 snapshot을 locking하는 전략으로 이해해야 한다. PostgreSQL에서는 "READ UNCOMMITTED" 전략을 배제하고 있어서 실질적으로 세가지 Level만 지정할 수 있다.

또한 MySQL의 InnoDB 엔진에서는 "REPEATABLE READ"이 기본값이고, PostgreSQL의 엔진에서는 "READ COMMITTED"가 기본 값이라고 각 문서에 언급되어있다. PostgreSQL 문서에서는 Transaction의 동시성에서 아래와 같은 문제가 발생할 수 있다고 한다. 

* dirty read

    > A transaction reads data written by a concurrent uncommitted transaction.

* nonrepeatable read

    > A transaction re-reads data it has previously read and finds that data has been modified by another transaction (that committed since the initial read).

* phantom read

    > A transaction re-executes a query returning a set of rows that satisfy a search condition and finds that the set of rows satisfying the condition has changed due to another recently-committed transaction.

* serialization anomaly

    > The result of successfully committing a group of transactions is inconsistent with all possible orderings of running those transactions one at a time.

Isolation Level | Dirty Read | Nonrepeatable Read | Phantom Read | Serialization Anomaly
---|---|---|---|---
Read uncommitted | Allowed, but not in PG | Possible | Possible | Possible
Read committed | Not possible | Possible | Possible | Possible
Repeatable read | Not possible | Not possible | Allowed, but not in PG | Possible
Serializable | Not possible | Not possible | Not possible | Not possible

*출처: [13.Concurrency control > 13.2 Transaction isolation - PosgtreSQL](https://www.postgresql.org/docs/current/transaction-iso.html)*

한국어로 번역하다가 너무 어려워서 안했는데, 이름이 특징을 잘 드러내고있다고 본다. 

Isolation level은 시스템 성격에따라 섬세하게 다룰 필요가 있다. 그래서 특성에 따라서 적절하게 제어하기 위해 Isolation level에 대해서도 조사했다. Standard isolation level로 4가지가 있고, SQLAlchemy 라이브러리에서 제공하는 level이 하나 더 있다. 

Isolation Level | Description
"AUTOCOMMIT" | 세션 단위에서 commit을 하면 Transaction도 함께 매번 commit을 수행한다.(SQLAlchemy 라이브러리 수준에서의 제어)
"READ COMMITTED" | 어떤 Transaction이 쿼리할 수 있는 데이터는 다른 Transaction이 이미 Commit을 끝낸 데이터이다. MySQL에서는 다른 Transaction이 commit을 완료할 때 snapshot을 갱신한다. 
"READ UNCOMMITTED" | 어떤 Transaction이 다른 Transaction에서 아직 commit을 끝내지 않은 데이터를 쿼리할 수 있다. PostgreSQL에서는 이 값을 지정하더라도 "READ COMMITTED"로 처리한다. 
"REPEATABLE READ" | `transaction begin;` 전에 commit을 마친 데이터만 볼 수 있다. 실행중인 Transaction 안에서는 다른 Transaction 데이터 변경을 commit 하더라도 볼 수 없다. MySQL에서는 Transaction이 시작될 때 snapshot을 만들고 이 snapshot을 갱신하지 않는다.
"SERIALIZABLE" | 동시성에서 생기는 side-effect를 차단하기 위해 모든 Transaction에서 요청되는 쿼리를 순차적으로 처리하는 방법이다. 어떤 record의 row에 쿼리를 할 때 Transaction이 close된 뒤에 다른 Transaction이 접근할 수 있다. multi process, multi thread를 다루는 프로그램에서는 전체적인 성능 하락이 발생할 수도 있다.

한가지 더 붙여두자면, OracleDB에서 정리해둔 문서도 있는데, Reference로만 남겨둔다.

## 결론

이번 이슈를 해결하는 과정에서 제일 큰 난관은 Isolation level이라는게 있을 수 있다는걸 생각하지 못했던 문제였다. 여러 Transaction이 만들어졌을 때 코드의 실행 흐름을 flow chart로 그려보고 충돌이 발생할만한 지점들을 각각 실험하기위한 코드들을 만들어서야 원인을 찾을 수 있었고 해결 방안도 동시성에서 race condition이 생길 수 있다는 것을 잊어버리고 있다가 떠올리고 나서야 조사할 수 있었다. 

또한 이 문제를 조사하면서 Transaction에 대해서도 어렴풋이 알고있던 것들을 명확한 모델로 머릿속에 그려볼 수 있었기 때문에 가장 큰 성과는 Isolation level이 있구나 하는 것을 찾아낸게 아니라 DB가 동시성에서 생기는 race condition을 어떻게 다루는지 생각해볼 수 있었던 고찰 그 자체였다고 생각한다. 

## 참고

* [SQLAlchemy - Setting Transaction Isolation Levels including DBAPI Autocommit](https://docs.sqlalchemy.org/en/20/core/connections.html#setting-transaction-isolation-levels-including-dbapi-autocommit)
* [MySQL - 15.7.2.1 Transaction Isolation Levels](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html)
* [PostgreSQL - Chapter 13. Concurrency Control > 13.2. Transaction Isolation](https://www.postgresql.org/docs/current/transaction-iso.html)
* [OracleDB - 9 Data Concurrency and Consistency](https://docs.oracle.com/cd/E25054_01/server.1111/e25789/consist.htm)
