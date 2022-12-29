---
layout: post
title: pytest로 시스템 테스트를 하는 방법 - 2
subtitle: SQLAlchemy를 사용하는 시스템에서의 테스트
comments: true
categories: ["Programming"]
tag: ["Test", "Python", "pytest", "SQLAlchemy", "factoryboy"]
---

## pytest-factoryboy에 대해서

이 모듈은 pytest라고 하는 python의 유닛테스트 툴에 factoryboy를 플러그인화 한 패키지이다. 이 모듈을 사용하지 않는다고 해도 fixture로 각 객체를 생성하도록 하는 테스트 코드를 작성할 수는 있지만, 팩토리 클래스와 데이터 모델의 결합으로 코드에서 사용되고있는 객체 모델과 동일한 구조로 fixture 를 만들 수 있고 여러 설정 값을 이용해서 코드를 더욱 추상화하는 것도 가능하기 때문에 여러모로 유용하다고 할수있다.

### 설치

이 패키지를 설치하는 방법으로 여러가지가 있지만, 아래에 두가지정도만 적어본다.

* poetry를 이용하는 경우

    ``` python
    $ poetry add pytest-factoryboy --dev
    ```

* pip를 이용하는 경우

    ``` python
    pip를 이용하는 경우
    $ pip install pytest-factoryboy
    ```

### 간단한 사용 기능

여기에는 `Person`과 `House`, `MobilePhone` 세개의 객체를 예로 들어 Factory 클래스가 어떻게 정의되는지 기록해두려한다.
전체적인 객체 구조는 다음과 같다.

``` mermaid
erDiagram
    Person {
        string id
        string name
        string email
    }
    House {
        string address
        integer size
    }
    MobilePhone {
        string number
        string model
    }
    Person }|--|| House : livein
    MobilePhone ||--|| Person : has 
    Person }o--o{ House : owned
```

그리고 이 모델 구조 전체를 구현한 코드는 이렇다.

``` python
class Person(Base):
    id = Column(String(32), primary_key=True, )
    name = Column(String(length=255), nullable=True, )
    email = Column(String(length=255), nullable=True, unique=True, )
    residence = Column(ForeignKey("House.address"))

    livedin = relationship("House", back_populates="residents", uselist=False)
    phone = relationship("MobilePhone", back_populates="user", uselist=False)
    drivible = relationship("Car", back_populates="driver", secondary="CarDriver")
    
class House(Base):
    address = Column(String(128), primary_key=True, )
    size = Column(Integer)

    residents = relationship("Person", back_populates="livedin")
    

class MobilePhone(Base):
    number = Column(String(16), primary_key=True, )
    user_id = Column(ForeignKey("Person.id"))
    model = Column(String(16))

    user = relationship("Person", back_populates="phone", uselist=False)

class Car(Base):
    car_number = Column(String(16), primary_key=True)
    name = Column(String(64))
    maker = Column(String(32))

    driver = relationship("Person", back_populates="drivible", secondary="CarDriver")    


class CarDriver(Base):
    car_number = Column(ForeignKey("Car.car_number"), primary_key=True, )
    driver_id = Column(ForeignKey("Person.id"), primary_key=True, )
```

#### 하나의 모델에 대해서

SQLAlchemy를 이용해 정의한 이런 형태의 객체 모델이 있다고 할때, `Person` 모델은 factoryboy로 이렇게 표현한다.

``` python
class PersonFactory(factory.alchemy.SQLAlchemyModelFactory):
    class Meta:
        model=Person
        sqlalchemy_session=session
        sqlalchemy_session_persistence="commit"
        sqlalchemy_get_or_create=("id",)

    id = factory.Faker("ssn", locale="ko_KR")
    name = factory.Faker("name", locale="ko_KR")
    email = factory.Faker("email", locale="ko_KR")

person = PersonFactory()

record = session.query(Person).filter(Person.id==person.id).one()

assert record.id == person.id
```

하나씩 설명하자면 이렇다.

* Meta: nested class로 정의되어있는 이 클래스가 Factory 클래스의 설정 값을 갖고있다. 자세하게는 이렇게 되어있다.
    * model: 이 Factory 클래스가 어떤 객체모델을 표현하는 것인지를 나타낸다. Django나 MongoDB 모델에도 동일하게 사용된다.
    * sqlalchemy_session: 이 Factory의 `build`/`create stretagy`에 따라 실행하는 쿼리가 사용될 세션을 의미한다. `pytest-factoryboy`에서는 각 테스트 함수, 그리고 테스트 쓰레드간의 독립성을 유지하기위해 `scoped_session`을 사용할 것을 권장하고 있다.
    * sqlalchemy_session_persistence: session strategy에 따라 단위 작업간에 어떤 동작을 수행할 것인지에 대한 값인데, `None`, `flush`, `commit` 세가지 값 중 하나로 정의할 수 있다. `None`인 경우에는 DB에 아무 값도 쓰지 않고 모델 객체만 생성한다.
    * sqlalchemy_get_or_create: unique key가 중복되는 레코드가 이미 있는 경우에 새로운 객체를 생성하지 않고 기존 객체를 불러올 수 있는데, 그 기준이 되는 컬럼을 의미한다. tuple 타입으로 정의해야 한다.

아래의 `id`, `name`, `email`은 `Person` 객체에 있는 컬럼을 의미하고, 여기에 `Faker` 를 이용해서 임의 값을 생성하도록 한다는 의미이다.

#### 둘 이상의 모델에 대해서

이제 위의 `Person`, `House`, `MobilePhone` 세가지 객체 모델의 Factory 예를 적어두려 한다. `SQLAlchemy`에서의 관계를 정의하는 방법이라거나 Factory를 어떻게 구현하는가에는 단일 모델보다는 좀 더 복잡하기 때문에 시행착오를 함께 기록해둔다.

##### 1:1 관계

1:1 관계에 있는 `MobilePhone`과 `Person`의 Factory는 이렇게 만들어진다.

``` python
class PersonFactory(factory.alchemy.SQLAlchemyModelFactory):
    class Meta:
        model=Person
        sqlalchemy_session=session
        sqlalchemy_session_persistence="commit"
        sqlalchemy_get_or_create=("id",)

    id = factory.Faker("ssn", locale="ko_KR")
    name = factory.Faker("name", locale="ko_KR")
    email = factory.Faker("email", locale="ko_KR")

    phone = factory.SubFactory(".MobilePhoneFactory")  # 객체를 파라미터로 전달하지 않을 때는 모듈의 패키지 경로를 넣어야 한다.

class MobilePhoneFactory(factory.alchemy.SQLAlchemyModelFactory):
    class MEta:
        model = MobilePhone
        sqlalchemy_session = scoped_session(sessionmaker())
    number = factory.Faker("phone_number", locale="ko_KR")
    model = "iPhone 14 Pro max"
    
    user = factory.SubFactory(PersonFactory)  # Factory 객체 자체를 파라미터로 전달할 수 있다.

person = PersonFactory(phone=None)
phone = MobilePhoneFactory(user=person)

person.phone = phone
session.commit()


record = session.query(Person).filter(Person.id==person.id).one()

assert record.id == person.id
assert record.phone.number == phone.number
```

##### 1:m 관계

`House`와 `Person`의 관계를 거주지와 거주민으로 구분해서 관계를 정의해보겠다. 이런 경우에는 하나의 집에 여러 사람이 살 수 있기 때문에 1:m 관계로 정의했다.

``` python
class PersonFactory(factory.alchemy.SQLAlchemyModelFactory):
    class Meta:
        model=Person
        sqlalchemy_session=session
        sqlalchemy_session_persistence="commit"
        sqlalchemy_get_or_create=("id",)

    id = factory.Faker("ssn", locale="ko_KR")
    name = factory.Faker("name", locale="ko_KR")
    email = factory.Faker("email", locale="ko_KR")

    livedin = factory.SubFactory(".HouseFactory")

class HouseFactory(factory.alchemy.SQLAlchemyModelFactory):
    class Meta:
        model=House
        sqlalchemy_session=session
        sqlalchemy_session_persistence="commit"
        sqlalchemy_get_or_create=("address",)

    address = factory.Faker("address", locale="ko_KR")
    size = 100

    residents = factory.SubFactory(PersonFactory)

person = PersonFactory(livedin=None)
house = HouseFactory(residents=[person])

person.livedin = house
session.commit()

record = session.query(Person).filter(Person.id==person.id).one()

assert record.id == person.id
assert record.house.address == house.address
```

##### m:n 관계

이제 `Car`와 `Person`의 관계를 살펴보겠다. 이 둘은 차와 운전자의 관계에 있는데, 한 사람이 여러 차의 운전자가 될 수 있고 하나의 차는 여러 운전자가 운전할 수 있기 때문에 m:n의 관계를 갖는다고 볼 수 있다.

``` python

class PersonFactory(factory.alchemy.SQLAlchemyModelFactory):
    class Meta:
        model=Person
        sqlalchemy_session=session
        sqlalchemy_session_persistence="commit"
        sqlalchemy_get_or_create=("id",)

    id = factory.Faker("ssn", locale="ko_KR")
    name = factory.Faker("name", locale="ko_KR")
    email = factory.Faker("email", locale="ko_KR")

    drivible = factory.SubFactory(
        ".CarFactory"
    )

class CarFactory(factory.alchemy.SQLAlchemyModelFactory):
    class Meta:
        model=Car
        sqlalchemy_session=session
        sqlalchemy_session_persistence="commit"
        sqlalchemy_get_or_create=("car_number",)
    
    car_number = factory.Faker("license_plate", locale="en_US")
    name = factory.Faker("name", locale="en_US")
    maker = factory.Faker("name", locale="en_US")

    driver = factory.SubFactory(PersonFactory)

person = PersonFactory(drivible=[])
car = CarFactory(driver=[person])

person.drivible.append(car)
session.commit()

record = session.query(Person).filter(Person.id==person.id).one()

assert record.id == person.id
assert car in record.drivible
```

이미 객체 모델에서 relationship으로 이 부분에 대한 정의가 되어있기 때문에 `SubFactory`에서는 그냥 서로의 Factory 클래스를 서로 전달하는 것으로 충분하다.

#### 둘 이상의 모델 객체를 생성하는 방법

이렇게 만든 Factory 클래스는 단일 객체만 생성할 수 있는 것이 아니다. 위에서 만들었던 `PersonFactory`를 예로 들어보겠다.

``` python
# 객체를 생성하기만 함
person = PersonFactory.build()  # 하나의 객체만 생성
people = PersonFactory.build_batch(size=10)  # 10개의 객체를 생성

# 객체 생성 후에 동작을 함께 수행함
person = PersonFactory.create()  # 하나의 객체만 생성 후 sqlalchemy_session_persistence에서 정의한 동작을 함께 수행함
people = PersonFactory.create_batch(size=10)  # 10개의 객체를 생성 후 sqlalchemy_session_persistence에서 정의한 동작을 함께 수행함
```

`build()`, `create()` 함수에는 맴버 변수에 넣을 값을 미리 파라미터로 전달할 수 있는데, 이는 `build_batch()`, `create_batch()`에도 똑같이 적용된다.

### 테스트 코드와의 결합

이렇게 만든 Factory 클래스를 pytest와 결합해서 사용할 수 있는데, 이 부분은 간단한 테스트 함수 몇개만 기록해 둔다.

``` python
# 위의 Factory를 pytest와 결합하기 위한 decorator를 위한 코드로, 생략되어있는 코드는 위에 만들었던 Factory 클래스의 내용과 같다.

@register
class PersonFactory(factory.alchemy.SQLAlchemyModelFactory): ... 
@register
class HouseFactory(factory.alchemy.SQLAlchemyModelFactory): ... 
@register
class MobilePhoneFactory(factory.alchemy.SQLAlchemyModelFactory): ... 
@register
class CarFactory(factory.alchemy.SQLAlchemyModelFactory): ... 

def test_person_factory(person_factory):
    person = person_factory()
    record = session.query(Person).filter(Person.id==person.id).one()
    assert record.id == person.id

def test_person(person):
    # person_factory의 register decorator가 동작하면서 기본 파라미터를 적용한 person fixture를 함께 생성한다.

    record = session.query(Person).filter(Person.id==person.id).one()
    assert record.id == person.id
```

위의 코드는 `@register` decorator를 이용해서 Factory 클래스를 pytest의 fixture로 추가시키는 것을 의미한다. 이외에도 `register(FactoryClass)` 와 같은 형태로도 사용할 수 있기 때문에 Factory 클래스를 먼저 정의하고 fixture를 따로 정의해서 Factory 클래스의 값을 재정의할 수도 있다.

아예 `@register`를 사용하지 않고 fixture를 직접 정의해서 factory 클래스 자체를 반환하도록 할수도 있다.

## 결론

pytest-factoryboy는 이렇게 패턴화된 데이터를 만들도록 하는 Factory 패턴을 통해 테스트 데이터를 만들기 쉽도록 도와준다. 상당히 편리하긴 하지만, 사용할 때 몇가지 알아야 할 것이 있다. 하나는 `sqlalchemy_session_persistence` 에 따라 이후 동작을 따로 해주어야 하는것인데, 이 값이 `None`인 경우에는 `create()` 함수를 호출하더라도 실제 DB에 레코드를 추가하지 않는다. `flush`와 `commit`은 DB 세션의 동작을 함께 정의하는데, 이 특성을 잘 알고 사용해야 한다. 
그리고 또 하나는 pytest-factoryboy에 있는데, factoryboy는 build -> insert -> commit 까지 수행해주지만 테스트 후에 teardown은 직접 해줘야 한다. DB에서 `DELETE` 쿼리는 따로 해줘야 한다는 것이다. 이에 대해 일반적인 사용 패턴은 python3의 unittest 라이브러리에서는 테스트 클래스 범위안에서 teardown 맴버함수를 정의하고 이 안에서 DB환경을 리셋하는 식으로 사용한다.

``` python
class TestPersonFactory(unittest.TestCase):
    def setUp(self):
        self.Session = sessionmaker()
        self.person = PersonFactory()

    def test_somthine(self,):
        ...

    def tearDown(self):
        sess = self.Session()
        sess.delete(self.person)
        sess.commit()

        self.Session.close_all_sessions()
```

unittest에서는 이렇고, pytest에서는 이런 코드를 찾을 수 있었다.

``` python
@fixture
def person_factory(request):
    person_ = PersonFactory()
    def _teardown():
        session.delete(person_)
        session.commit()

    request.addfinalizer(_teardown)

    return person_

@fixture
def person_factory(request):
    person_ = PersonFactory()

    yield person_

    session.delete(person_)
    session.commit()
```

이런 구성으로 `addfinalizer`에 reusable teardown 함수를 추가하는 fixture를 정의하거나 `yield`를 이용해서 yieldinig 후에 delete를 하도록 할 수도 있다. 이 부분을 알고 사용한다면 편하게 사용할 수 있는 좋은 헬퍼 모듈이 될것 같다.
