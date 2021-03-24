---
layout: post
title: SQLAlchemy에 대한 사용 안내 - DB 서버에 CRUD에 대한 Event hook
subtitle: 내가 보려고 만드는 메뉴얼
comments: true
categories: "Database"
tag: ["python", "Database", "ORM", "SQLAlchemy"]
---

## Event hook이란

보통 Event hook이라고 하면, event 기반의 시스템에서 특정 사건이 발생하거나 작업을 수행하는 시점을 캐치해서 그 전후에 어떤 작업을 수행하는 것을 말한다. 당연히 DBMS는 쿼리를 받아서 수행하고 결과를 반환하는 이벤트를 수행하는 독립적인 시스템 이므로 작게는 CRUD 쿼리들부터 DB커넥션, 트랜잭션, 일반적인 쿼리 전반에 대한 수행 단계를 캐치해서 원하는 작업을 하도록 만들 수 있다.

## CRUD란

DB의 역할은 사용자가 데이터를 쓰고, 읽고, 수정하고, 지우는 작업을 잘 해주는데 있다. 그걸 사람들이 Create, Read, Update, Delete의 앞 글자를 따서 CRUD라고 부른다.

## Query별 이벤트

Select를 제외한 Insert, Update, Delete는 맵퍼 이벤트로 분류된다. ([SQLAlchemy 공식 문서 참고](https://docs.sqlalchemy.org/en/13/orm/events.html#mapper-events))

따라서 맵핑되어있는 객체를 통해 쿼리를 수행할 때 맵퍼에서 생기는 이벤트에 훅을 하는 방식이다.

```python

```
