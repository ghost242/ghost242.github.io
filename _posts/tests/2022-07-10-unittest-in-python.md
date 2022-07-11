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
@pytest.fixture(scope="session")
def client():
    # TestClient 객체를 만들고 해당 객체를 반환한다. 이때 이 객체는 backend로 anyio 패키지를 사용한다. 이 비동기 패키지를 정의하는 파라미터는 asyncio, trio 등으로 선언할수있다.
    return TestClient(service_app)

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

여기에서 Dictionary 객체는 key, value라는 컬럼을 갖는 DB 엔티티 의미한다. 이제 이 DB에 값을 넣는 API를 정의한다.

``` python
@service_app.post("/{key}", status_code=status.HTTP_201_CREATED)
def insert_value(value, sess=Depends(session)):
    record = Dictionary(key, value)
    sess.add(record)
    sess.commit()

    return {"key": key, "value": value}
```

이 코드에 대한 테스트코드는 이렇게 만들어볼 수 있다.

``` python
@pytest.fixture(scope="fuction")
def sample_value():
    return "Hello", "World!"

def test_app_insert(client, sample_value):
    key, value = sample_value
    res = client.post(f"/{key}", body=value)
    
    return res == dict(key=key, value=value)
```

샘플 코드는 FastAPI를 이용한 ASGI 서버로 한다. 이후 모든 테스트에 걸쳐 계속해서 사용된다.

### 서비스 앱

간단하게 네개의 API를 구현한 service app을 정의하는 코드영역.

``` python
service_app = FastAPI()

def session_generator():
    sess = SessionInst()

    try:
        yield sess

    finally:
        sess.close()

@service_app.get("/")
async def index():
    return {"message": "Hello world!!"}

@service_app.post("/insert", response_model=InsertDevModel)
async def insert(model: InsertDevModel, sess: SessionInst = Depends(session_generator)):
    record = DevModel(key=model.key, value=model.value)

    try:
        db.add(record)
    except:
        db.rollback()
    else:
        db.commit()

    return record

@service_app.post("/upload", response_model=InsertDevModel)
async def upload(file: UploadFile):
    content = await file.read()
    return UploadDevModel(name=file.filename, size=len(content))


@service_app.get('/{key}', response_model=GetDevModel)
async def find(key: str, sess: SessionInst = Depends(session_generator)):
    record = db.query(DevModel).filter(DevModel.key == key).first()

    return GetDevModel(key=record.key, value=record.value)
```

### 데이터 모델

위의 앱 구현체에서 사용하는 Request/Response 모델

``` python
class BaseDevModel(BaseModel):
    idx: int

class InsertDevModel(BaseModel):
    key: str
    value: str

    class Config:
        orm_mode = True

class GetDevModel(BaseModel):
    key: str
    value: str

    class Config:
        orm_mode = False

class UploadDevModel(BaseModel):
    name: str
    size: int
```

### 스키마 객체

물리DB의 구조를 객체화한 데이터 모델

``` python
class DevModel(Base):
    __tablename__ = "tb_dev_model"
    __table_args__ = { "schema": "dev_db" }

    idx = Column(Integer, primary_key=True, autoincrement=True)
    key = Column(Text)
    value = Column(Text)
```

## 테스트 코드

테스트 코드는 크게 두가지로 나뉜다.

### conftest.py

``` python
def pytest_sessionstart():
    logging.getLogger().setLevel(logging.DEBUG)

    logging.debug("test session start")
    service_initializer()  # FastAPI 서비스 동작을 위해 시스템 구성을 초기화하는 함수
```

### test_function.py

``` python
client = TestClient(service_app)

@fixture()
def model_data():
    return {"key": str(uuid4()), "value": "World!"}

@fixture()
def session():
    return SessionInst()

def test_app_index():
    response = client.get("/")

    assert response.json() == {"message": "Hello world!!"}


def test_insert(session, model_data):
    response_post = client.post("/insert", json=model_data)

    assert response_post.json() == model_data

    records = session.query(DevModel).filter(DevModel.key==model_data['key']).all()

    assert len(records) > 0

def test_get(session, model_data):
    record = DevModel(key=str(uuid4()), value="This is value column.")
    session.add(record)
    session.commit()

    response_get = client.get(f"/{record.key}").json()

    assert response_get["key"] == record.key
    assert response_get["value"] == record.value
```
