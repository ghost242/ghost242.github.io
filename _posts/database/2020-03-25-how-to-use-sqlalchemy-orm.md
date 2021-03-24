---
layout: post
title: SQLAlchemy에 대한 사용 안내 - ORM 기본편
subtitle: 내가 보려고 만드는 메뉴얼
comments: true
categories: "Database"
tag: ["python", "Database", "ORM", "SQLAlchemy"]
---

## DDL

DDL은 Data Definition Language의 약자로, 데이터의 형태와 구조(또는 모델)를 정의하는 언어를 말한다. 이 쿼리문을 보면 데이터의 알기 쉽다. ORM은 프로그래밍 언어에서의 개념적 Object와 데이터의 논리적 구조인 Model과의 맵핑하는것을 의미한다. 이전에는 쿼리를 통해서만 데이터를 구조화 할 수 있었는데, ORM은 객체화된 논리적 모델을 물리적인 데이터모델과 연결시키기 때문에 아주 쉽게 데이터를 다룰 수 있게 되었다.

### Create Table

Object를 통해 테이블을 만드는 방법은 다음과 같다.

우선은 테이블의 구조를 갖는 SQLAlchemy에서 읽을 수 있도록 정의해야 한다.

``` python
Base = declarative_base()

class Object1(Base):
    __tablename__ = "table_1_name"

    prim_column_name = Column(Integer, primary_key=True, autoincrement=True)
    value_column_name = Column(String(100))

class Object2(Base):
    __tablename__ = "table_2_name"

    prim_column_name = Column(Integer, primary_key=True, autoincrement=True)
    value_1_column_name = Column(String(100))
    value_2_column_name = Column(DateTime, default=datetime.now, nullable=False)
```

그리고 Database connector 패키지와의 연결 객체를 만든다.

``` python
engine = create_engine('sqlite://', echo=True)
```

이제 위에서 만든 Base가 가지고있는 하위 객체들에 대한 정보를 이용해 테이블을 만든다.

``` python
Base.metadata.create_all(engine)
```

이렇게 하면 `table_1_name`과 `table_2_name`가 메모리영역에 만들어진 가상의 Database에 생성된다.

### Drop Table

테이블을 제거하는 방법은 어렵지 않다. 위에서 만들었던 테이블을 예로 들어본다면 테이블 삭제 코드는 다음과 같다.

```python
class Object1(Base):
    __tablename__ = "table_1_name"

    prim_column_name = Column(Integer, primary_key=True, autoincrement=True)
    value_column_name = Column(String(100))

# drop 함수에서 engine(DB 연결객체)을 바인딩해서 쿼리를 전달하도록 한다.
Object1.__table__.drop(engine)
```

이렇게 하면 sqlalchemy 내부에서 drop table 쿼리를 생성해서 DB에 요청하게 된다.

## DML

DML은 Data Manipulation Language의 약자로, 데이터베이스에 있는 데이터를 조작하기 위한 언어를 의미한다. 데이터베이스로부터 데이터를 검색하고, 추가하고, 수정하고, 삭제하는 것이다.

### Select from

가장 중요하고, 가장 많이 사용하고, 성능에 가장 민감한 쿼리이다. DBMS로부터 가장 트래픽을 많이 차지하는 쿼리이기 때문에 반드시 쿼리를 최적화 해야 할 필요가 있다. 하지만 기본적으로 데이터를 가져오는 방법은 그렇게 어렵지 않다.

특정 테이블에서 데이터를 가져오는 방법으로 아래와 같은 형태의 코드를 사용한다.

```python
# 이런 테이블 모델이 있다고 가정
class Object1(Base):
    __tablename__ = "table_1_name"

    prim_column_name = Column(Integer, primary_key=True, autoincrement=True)
    value_column_name = Column(String(100))


# DBMS와의 커넥션 엔진을 만들어야 함.
engine = create_engine('sqlite://', echo=True)

# 1. 세션에서 query를 실행하는 방법



# 2. Query객체에 세션을 바인딩 하는 방법
```

```python
```

### Insert table

### Update table

### Delete table

## DCL
