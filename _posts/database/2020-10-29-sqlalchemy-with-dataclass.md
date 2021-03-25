---
layout: post
title: SQLAlchemy ORM을 조금 더 객체 처럼 사용하기
subtitle: 내가 보려고 만드는 메뉴얼
comments: true
categories: ["Python"]
tag: ["python", "database", "sqlalchemy"]
---

## SQLAlchemy ORM을 조금 더 객체 처럼 사용하기

SQLAlchemy은 DB를 Object처럼 만들어서 다루기 위해 만들어진 패키지입니다. 우리는 Table과 Object를 1:1로 맵핑해서 사용하고 있지만, 여기서 작성하는 글은 좀 더 파이썬에 잘 녹여서 잘 사용하기 위한 내용입니다.

1. Dataclass

    Dataclass는 파이썬 3.7에서 새로 추가된 클래스 타입입니다. 원형은 함수형 언어에서 사용하는 데이터 객체인데, OOP 언어에서 사용되는 Object와는 다르게 활용될 수 있습니다. 같은 속성을 가진 데이터끼리 한번에 묶어서 처리해야 할 때 쓰기 좋은 데이터 객체입니다. 부가적으로 내부에 dict, tuple 등의 데이터 구조로 변환하는 함수를 함께 지원합니다.

    만약에 사람을 Dataclass로 표현한다면 다음과 같은 구조가 됩니다.

    ``` python
    @dataclass
    class Person:
        name: str
        age: int
        address: str
        job: str
    ```

예를 들어 Table object를 하나 보겠습니다.

``` python
# 사람 정보를 입력하는 간단한 Table
class Person:
    __tablename__ = "Person"

    index = Column(Integer, primary_key=True, autoincrement=True, )
    name = Column(Text, nullable=False)
    age = Column(Integer, nullable=False)
    address = Column(Text, nullable=False)
    job = Column(Text, default="NEET")

# 사람 정보를 조회하는 간단한 코드
session.query(Person).all()
```

이 Object는 간단하게 SQLAlchemy의 ORM API를 이용해 정의한 테이블입니다. 이 테이블에서 데이터를 조회하는 방법은 이미 활용되고 있는 것처럼 아주 간단합니다. 그러나 다음과 같이 서로 Releation이 만들어진 모습은 예로 들어보겠습니다.

``` python
class Class:
    __tablename__ = "Class"

    index = Column(Integer, primary_key=True, autoincrement=True, )
    name = Column(Text, unique=True)


class Teacher:
    __tablename__ = "Teacher"

    index = Column(Integer, primary_key=True, autoincrement=True, )
    class_index = Column(Integer, ForeignKey("Class.index"))
    position = Column(Text)
    person_info = Column(Integer, ForeignKey("Person.index"))

class Student:
    __tablename__ = "Student"

    index = Column(Integer, primary_key=True, autoincrement=True, )
    class_index = Column(Integer, ForeignKey("Class.index"))
    person_info = Column(Integer, ForeignKey("Person.index"))
```

이렇게 아주 간단하게 학교 구성원을 테이블로 구성한다고 하면 