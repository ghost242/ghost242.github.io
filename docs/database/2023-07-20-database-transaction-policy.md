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
: : sess.commit() | : |
: : sess.query.update({RecordA.val: 54321}) | | 
: : sess.commit() | |

## 사례에 대한 해결 전략

## 결론
