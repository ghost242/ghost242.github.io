---
layout: post
title: SQLAlchemy에 대한 사용 안내 - Core 기본편
subtitle: 내가 보려고 만드는 메뉴얼
comments: true
categories: "Database"
tag: ["python", "Database", "ORM", "SQLAlchemy"]
---

### Core 관점에서의 DDL

이 방법이 위의 방법보다는 단순해보이지만 좀 더 효율적으로 테이블을 구성할 수 있다. 

우선 위와 같이 연결 객체를 만들어준다.

``` python
engine = create_engine('sqlite://', echo=True)
```

그리고 각 테이블 객체를 만들어준다.

``` python
metadata = Metadata()
# Table 객체는 Mapper 타입이다.
table_1_name = Table('table_1_name', metadata,
                    Column('prim_column_name', Integer, primary_key=True, autoincrement=True),
                    Column('value_column_name', String(100))
)
table_2_name = Table('table_2_name', metadata,
                    Column('prim_column_name', Integer, primary_key=True, autoincrement=True),
                    Column('value_column_name_1', String(100)),
                    Column('value_column_name_2', DateTime)
)
```

그리고 위에서 만든 mapper들을 이용해 테이블을 생성한다.

``` python
metadata.create_all(engine)
```
