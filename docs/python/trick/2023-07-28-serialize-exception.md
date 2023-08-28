---
layout: post
title: Exception 객체를 직렬화 하는 방법
subtitle: tblib에 대한 설명과 사용법
parent: Trick of code
grand_parent: Python
comments: true
categories: ["Programming", "Python3", "Exception"]
tag: ["Python3", "Exception", "Pickling"]
---

## 상황

### 개요

예를 들어 Master-Slave 구조의 시스템이 있다고 생각하고, Master에서 Slave 시스템의 오류 로그 등을 수집하고 감시를 할 수 있을 것이다.

이때 Python3에서 발생하는 Exception을 Master 시스템에 직접 넘겨주고 로그를 분석할 수 있다면 편할 것이라는 아이디어가 있었다.

### 구조

전체적인 디자인은 다음과 같다.

```
+----------------+                     +----------------+
| Slave          |                     | Master         |
| +-----------+  |                     |   +----------+ |
| | reporter  |--+---Send stacktrace---+-->| logger   | |
| +-----------+  |                     |   +----------+ |
|       ^        |                     |        |       |  
| throw Exeption |                     +--------+--------+
|       |        |    +----------+              |
| +-----------+  |    | Log DB   |<--Recording--+
| | procedure |  |    +----------+
| +-----------+  |
+----------------+
```

Master 시스템에 다양한 모듈이 있겠지만 대충 이런 느낌이 될 것이다. 여기에서 `Send stacktrace`가 핵심인데, 다양한 방법이 있을 수 있었다. 예를 들면 Slave::reporter가 모든 stacktrace를 미리 추출해서 평문으로 정리한 뒤에 보낼 수 있다.

### 문제

실제로 작업을 하면서 봤던 문제를 간략화 하려니 내용이 작위적인 느낌이 있는데, 위의 구조에서 Slave::reporter -> Master::logger 로 전달하는 것이 Exception의 객체 자체를 보내면 어떨까 라는 발상을 떠올리게되었다.

더 간략하게 함수로 호출 실험을 해보면 이렇게 된다.

``` python
"""
object serialize using pickle
"""

import pickle
import binascii
import traceback

def func():
    """
    something function
    """
    raise Exception("Hello!")

def send_exception() -> str:
    """
    exception serialize
    """
    try:
        func()
    except Exception as e:
        obj = pickle.dumps(e)
        hex_obj = binascii.hexlify(obj).decode()
        
        return hex_obj


def receive_exception(exc_obj: str):
    """
    exception deserialize
    """
    bytes_obj = binascii.unhexlify(exc_obj.encode()ise Exception("Hello!")

def send_exception() -> str:
    """
    exception serialize
    """
    try:
        func()
    except Exception as e:
        obj = pickle.dumps(e)
        hex_obj = binascii.hexlify(obj).decode()
        
        return hex_obj


def receive_exception(exc_obj: str):
    """
    exception deserialize
    """
    bytes_obj = binascii.unhexlify(exc_obj.encode())
    exc = pickle.loads(bytes_obj)
    msg = traceback.format_exception(exc)
    print(msg)

if __name__=="__main__":
    exc_obj = send_exception()
    receive_exception(exc_obj)

```

코드에서 보면 이런 순서로 진행된다.

1. `send_exception` 함수에서 호출한 `func` 함수로부터 Exception객체를 가져온다.
2. `pickle`로 직렬화 한다.
3. 직렬화된 bytes를 binascii를 이용해서 16진수 문자열로 변환한다.
4. `receive_exception` 함수에서 가져와서 다시 bytes로 변환한다.
5. unpickling 후에 표준적인 파이썬 오류메시지 포맷에 맞춰 출력한다.

이렇게 했을 때 기대하는 결과는 이렇게 될것이다.

```python
Traceback (most recent call last):

  File ".../practice_exception.py", line 17, in send_exception_by_str
    func()

  File ".../practice_exception.py", line 10, in func
    raise Exception("Hello!")

Exception: Hello!
```

하지만 실제로는 이렇게 나온다.

```python
Exception: Hello!
```

이 결과의 차이가 바로 문제인 것이다. `func` 함수의 stacktrace가 전혀 보존되지 않는다.

## 해결

### 원인

이 문제의 원인은 Exception 객체에 있는데, Python3은 함수 실행시 호출 스택을 Frame이라고 하는 객체구조로 관리를 한다. 이 안에 함수의 파일위치, 파라미터 및 지역변수 정보, Exception이 발생한 코드 라인 등의 정보가 저장되어있다. 그리고 Exception 객체는 이런 정보를 메모리에서 계속 옮기는데는 낭비가 심하기 때문에 Sequence가 아니라 Iterator 형태로 관리를 한다.

여기서 pickling의 문제가 바로 Iterator로 탐색가능한 객체를 전부 훑어서 직렬화 하지 않는다는 점이다. 그래서 unpickling 후에 Frame iterator가 잘못된 객체를 가리키기 때문에 탐색하지 않고 Exception 메시지만 출력되는 것이다. 이 문제를 해결하기 위해 다양한 방법이 있겠지만, 가장 간단한 방법으로 외부 라이브러리인 `tblib`를 사용해서 해결하는 방법을 적용했다.

### tblib

이 패키지는 Traceback 객체를 조작할 수 있게 도와주는 패키지이다. 주어진 코드를 통해 동작을 추정해보면 `pickle` 모듈은 `copyreg`라는 모듈을 이용해서 dump/load 함수의 동작을 `dispatch_table` 이라는 맵을 통해 유저 커스텀이 가능하도록 하는 기능을 제공하는데, 이 `tblib`는 이것을 이용해서 builtin Traceback 클래스와 거의 동일한 구조이면서 Frame Generator를 계속 호출하면서 모든 Frame 객체를 추출해내고 이 추출한 객체를 Sequence 데이터 타입으로 변환하는 것으로 생각된다. 이러한 코드가 `tblib` 패키지의 코드들을 추적하다보면 대략적으로 파악이 가능하다.

`tblib` 자체로도 Traceback 객체의 Frame sequence 를 조작하는 방법을 예시로 보여주고있는데, 위의 코드를 수정하면 이렇게 할 수 있다.

```python
...
from tblib import pickling_support
...

pickling_support.install()

def func():
    ...
```

중복되는 코드는 최대한 생략하고 필요한 코드만 보면 이 두개의 라인이 추가되어야 한다. 이렇게 하면 raise 명령어의 동작을 hook 하는 방식으로 Exception 객체를 만들게 되고, 사용자는 Iterator가 아닌 Sequence로 이루어진 Frame 객체들을 볼 수 있게 된다.

### pickling

위의 `tblib` 패키지를 이용해서 Exception 객체를 피클링 가능하도록 하면 어떻게 차이가 나는지 그냥 봐도 알 수 있다. 처음 예제 코드에서 발생한 Exception을 pickling 후 `binascii` 모듈의 `hexlify` 함수를 이용해 16진수 문자열로 변환한 결과를 출력해봤다.

* `pickling_support.install()` 함수를 호출하지 않는 경우

  `Pickling exception with hex string:  80049527000000000000008c086275696c74696e73948c09457863657074696f6e9493948c0648656c6c6f2194859452942e`

  아래는 Exception 발생 직후 객체의 StackSummary 리스트이다.

  ```python
  > print(traceback.StackSummary.extract(traceback.walk_tb(e.__traceback__)))
  [<FrameSummary file ../practice_serialize/practice_pickle.py, line 21 in send_exception_by_str>, <FrameSummary file ../practice_serialize/practice_pickle.py, line 14 in func>]
  ```

  그리고 아래는 Exception 객체를 `unhexlify` 후 출력 결과이다.

  ```python
  > print(traceback.StackSummary.extract(traceback.walk_tb(exc.__traceback__)))
  []
  ```

* `pickling_support.install()` 함수를 호출해서 `tblib` 모듈을 활성화 한 경우

  `Pickling exception with hex string:  80049582020000000000008c1674626c69622e7069636b6c696e675f737570706f7274948c12756e7069636b6c655f657863657074696f6e949394288c086275696c74696e73948c09457863657074696f6e9493948c0648656c6c6f219485944e68008c12756e7069636b6c655f74726163656261636b9493948c0574626c6962948c054672616d659493942981947d94288c08665f6c6f63616c73947d948c09665f676c6f62616c73947d94288c085f5f6e616d655f5f948c085f5f6d61696e5f5f948c085f5f66696c655f5f948c582f686f6d652f616872616d2f576f726b2f636f64656c61622d707974686f6e2f7372632f70726163746963655f66696c65732f70726163746963655f73657269616c697a652f70726163746963655f7069636b6c652e707994758c06665f636f646594680a8c04436f64659493942981947d94288c0b636f5f66696c656e616d659468168c07636f5f6e616d65948c1573656e645f657863657074696f6e5f62795f737472948c0b636f5f617267636f756e74944b008c11636f5f6b776f6e6c79617267636f756e74944b008c0b636f5f7661726e616d657394298c0a636f5f6e6c6f63616c73944b008c0c636f5f737461636b73697a65944b008c08636f5f666c616773944b408c0e636f5f66697273746c696e656e6f944b0075628c08665f6c696e656e6f944b1775624b15680a8c0954726163656261636b9493942981947d94288c0874625f6672616d6594680c2981947d9428680f7d9468117d9428681368146815681675681768192981947d9428681c6816681d8c0466756e6394681f4b0068204b0068212968224b0068234b0068244b4068254b00756268264b0e75628c0974625f6c696e656e6f944b0e756287945294749452942e`

  아래는 Exception 발생 직후 객체의 StackSummary 리스트이다.

  ```python
  > print(traceback.StackSummary.extract(traceback.walk_tb(e.__traceback__)))
  [<FrameSummary file ../practice_serialize/practice_pickle.py, line 21 in send_exception_by_str>, <FrameSummary file ../practice_serialize/practice_pickle.py, line 14 in func>]
  ```

  그리고 아래는 Exception 객체를 `unhexlify` 후 출력 결과이다.

  ```python
  > print(traceback.StackSummary.extract(traceback.walk_tb(exc.__traceback__)))
  [<FrameSummary file ../practice_serialize/practice_pickle.py, line 21 in send_exception_by_str>, <FrameSummary file ../practice_serialize/practice_pickle.py, line 14 in func>]
  ```

차이는 바로 Frame 객체 목록이 보존된다는 것이다.

### 새로운 코드

```python
"""
object serialize using pickle
"""

import pickle
import binascii
import traceback

### 여기서부터
from tblib import pickling_support

pickling_support.install()
### 여기까지의 코드가 추가되어야 한다.

def func():
    """
    something function
    """
    raise Exception("Hello!")

def send_exception() -> str:
    """
    exception serialize
    """
    try:
        func()
    except Exception as e:
        obj = pickle.dumps(e)
        hex_obj = binascii.hexlify(obj).decode()
        
        return hex_obj


def receive_exception(exc_obj: str):
    """
    exception deserialize
    """
    bytes_obj = binascii.unhexlify(exc_obj.encode())
    exc = pickle.loads(bytes_obj)
    msg = traceback.format_exception(exc)
    print(msg)

if __name__=="__main__":
    exc_obj = send_exception()
    receive_exception(exc_obj)

```

## 결론

여러 방법이 있을 수 있을 것이다. 하지만 이렇게 하면 객체 자체를 주고 받으면서 try-except 문법을 그대로 사용할 수 있어 가장 문법적으로 어색하지 않은 방법이 될 수 있을 것이라고 생각했다. 이 객체를 받아와서 다시 Exception raise를 할 수도 있다.

```python
raise Exception("Got exception from external system) from exc
```

이렇게 `raise-from` 구문을 활용하면 Exception을 체이닝 할 수 있기 때문에 더 자세한 스택 목록을 출력해볼 수 있고 디버깅에 도움이 될 수도 있을 것 같다. 단점으로는 출력되는 메시지 길이가 너무 길어지고 불필요한 정보가 섞여서 분석을 방해할 수 있기 때문에 용도와 코드 구조, 환경을 잘 파악해서 적용 할 필요가 있다. 참고로 만약 그렇게 하고싶지 않다면 그냥 이렇게 하면 된다.

```python
raise exc
```
