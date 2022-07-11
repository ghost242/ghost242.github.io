---
layout: post
title: 유닛테스트 코드 작성 - 1
subtitle: pytest를 활용하는 방법
comments: true
categories: ["Programming"]
tag: ["Test", "Python", "pytest"]
---

## 개요

유닛테스트에 대한것들은 여기저기에서 다 찾아볼 수 있으므로 불필요한 설명 없이 pytest가 어떻게 동작하는지에 대해 기록해본다.

## 서비스 코드와 테스트 코드가 서로 대응하는 방식

FastAPI는 TestClient라고 하는 requests 패키지를 기본으로 내부에 FakeServer와 연결하는 객체를 갖는다. 이걸 이용해서 테스트를 한다.

### 아주 기본적인 테스트코드

``` python
# FastAPI 서비스 객체를 생성하는 코드라인
service_app = FastAPI()

# 기본적인 "GET /" 요청을 처리하는 함수
@service_app.get("")
async def index():
    return {"message": "Hello world!!"}
```

이제 이 코드에 대한 테스트코드를 정의해본다.

``` python
# fixture 예제
@pytest.fixture(scope="session")  # 이 fixture는 테스트 세션 전체에서 재활용되어야 하므로, scope="session"으로 fixture 파라미터를 지정한다.
def client():
    # TestClient 객체를 만들고 해당 객체를 반환한다. 이때 이 객체는 backend로 anyio 패키지를 사용한다. 이 비동기 패키지를 정의하는 파라미터는 asyncio, trio 등으로 선언할수있다.
    return TestClient(service_app)

@pytest.mark.anyio
def test_app_index(client):
    # 위에서 정의한 client fixture를 이용해서 get 요청을 하는 코드
    resp = client.get("/")

    assert resp.status == 200
    assert resp.json() == {"message": "Hello world!!"}
```

단순하게 만들면 이게 전부이다. 하지만 백엔드 서비스는 항상 데이터 모델이 필요하기 때문에 여기에 DB를 얹어본다.

### DB가 추가된 테스트코드

우선 스키마 모델을 정의해본다. ORM 라이브러리는 SQLAlchemy를 사용한다.

``` python
class Dictionary(Base):
    __tablename__ = "tb_dictionary"
    __table_args__ = { "schema": "sample" }

    key = Column(String(128), primary_key=True)
    value = Column(Text)

engine = create_engine("sqlite:///")
SessionInst = sessionmaker(bind=engine)

def session():
    return scoped_session(SessionInst)
```

여기에서 Dictionary 객체는 key, value라는 컬럼을 갖는 DB 엔티티 의미한다. 이제 이 DB에 값을 넣고 조회하는 API를 정의한다.

``` python
@service_app.get(
    f"/{key}", 
    responses={
        status.HTTP_200_OK: {
            "content": {
                "Application/json": {
                    "content": {},
                }
            },
            "description": "Return success key-value.",
        },
        status.HTTP_404_NOT_FOUND: {
            "content": {
                "text/plain": {
                    "schema": {
                        "type": "string"
                    }
                }
            },
            "description": "Return not found message",
        }
    },
)
def get_value(sess=Depends=(session)):
    try:
        record = session.query(Dictionary).filter(Dictionary.key==key).first()

        return {"key": record.key, "value": record.value}
    except SQLAlchemyError:
        return f"{key} does not exists." 

@service_app.post(
    f"/{key}", 
    responses={
        status.HTTP_200_OK: {
            "content": {
                "Application/json": {
                    "content": {},
                }
            },
            "description": "Return success key-value.",
        },
        status.HTTP_404_NOT_FOUND: {
            "content": {
                "text/plain": {
                    "schema": {
                        "type": "string"
                    }
                }
            },
            "description": "Return not found message",
        }
    },
)
def insert_value(value, sess=Depends(session)):
    try:
        record = Dictionary(key, value)
        sess.add(record)
    except SQLAlchemyError:
        sess.rollback()
    else:
        sess.commit()
        return {"key": key, "value": value}
    finally:
        sess.close()
```

이 코드에 대한 테스트코드는 이렇게 만들어볼 수 있다.

``` python
@pytest.fixture(scope="function")  # 이 fixture는 각 테스트 함수에서 사용되고 종료되어야 하므로 scope="function"으로 지정한다.
def test_session():
    sess = scoped_session(SessionInst)
    yield sess
    sess.close()

# Selection api에 대한 테스트
@pytest.fixture(scope="function")  # 이 fixture는 test_app_selection 함수에서만 사용되기 때문에 scope="function"으로 지정한다.
def searching_value(test_session):
    sample = Dictionary("Key1", "Value1")

    try:
        session.add(sample)
        session.commit()
        yield sample
    except SQLAlchemyError:
        session.rollback()
    else:
        session.delete(sample)
        session.commit()

@pytest.mark.anyio
def test_app_selection(client, searching_value):
    resp = client.get(f"/{searching_value.key}")

    assert resp.status == 200
    
    content = resp.json()
    assert content["key"] == searching_value.key

# Insertion api에 대한 테스트
@pytest.fixture(scope="fuction")
def sample_value():
    return "Hello", "World!"

@pytest.mark.anyio
def test_app_insert(client, sample_value):
    key, value = sample_value
    resp = client.post(f"/{key}", body=value)
    
    assert resp == dict(key=key, value=value)  # {"key": "Hello", "value": "World!} 가 반환되므로 assertion True
```

## 요약

이 문서에서 한 테스트는 FastAPI의 TestClient를 이용한 FakerServer에 request에 대한 repsonse의 assertion 테스트를 하고, SQLAlchemy를 이용해서 간단한 ORM 모델을 만들어서 Create, Read에 대한 테스트를 하는 방법에 대해 적어보았다.
여기에 아직 정리하지 않은 내용들은 다음 포스트에 정리해보려고 한다. 전에 Flask나 Django를 아주 가볍게 봤을 때는 이러한 웹프레임워크를 pytest와 연동해서 어떻게 테스트할 수 있는지 궁금했는데 이번에 이 내용을 조사하면서 어느정도 이해할 수 있게 되었다.

## 참고 자료

* [Pytest 공식 문서 -- fixture]()
* [FastAPI 공식 문서 -- Test](https://fastapi.tiangolo.com/tutorial/testing/)
