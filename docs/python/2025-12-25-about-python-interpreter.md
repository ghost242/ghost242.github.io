---
layout: post
title: 소스 코드 수준에서 바라본 Python3
subtitle: Python3 Interpreter Project 분석
parent: Python
comments: true
categories: ["Programming", "Python3"]
tag: ["Interpreter", "Python", "Project"]
---

## Python에 대해서

Python은 C언어로 작성된 인터프리터를 기반으로 하는 스크립트 언어이다. 기본적으로 절차적 실행과 OOP의 특징, 그리고 함수형 프로그래밍 언어의 장점을 잘 모아서 만든것 같은 언어로, 문법이 비교적 단순하고 코드 구성이 읽기 쉽게 되어있어서 접근성이 높은 편이다. 특히 중괄호나 대괄호같은 기호도 기반 언어인 C언어에 비하면 적은 편이고 코드블럭은 들여쓰기로 구분하기 때문에 Highlighting이 없는 평문으로 코드를 보면 자연어와 유사하게 읽을 수 있다는 것을 알 수 있다.

Python 1버전은 `Guido van Rossum` 이라는 연구자가 1991년에 처음 배포했다고 알려져있고, Wikipedia 문서에서는 1980년대 후반부터 작업을 해왔다고 기록이 되어있다. 언어적 특성은 1970년대에 개발되었던 `ABC` 라는 프로그래밍 언어에서 출발했다고 되어있는데, 들여쓰기로 코드 블럭을 구분하거나, for-in 등의 표현방식이 비슷하다는 것을 알 수 있다. 심지어 함수 호출도 현재 Python3에서는 반드시 괄호로 파라미터를 감싸도록 되어있는데, Python2에서는 `print "Hello world!"`와 같이 파라미터를 띄어쓰기로 구분하기도 했다.

현재는 Python2.7.4 버전에서 Python2는 완전히 지원이 중단되었고, Python3.14 버전이 가장 최신 버전이다. 여기까지 오면서 현재는 문법도 아주 복잡해졌고 많은 기능이 추가되어서 간단한 코드를 빠르게 만들 수 있다는 설명이 지금도 유효한지 모르겠지만, 인터프리팅 언어의 특징인 코드에 오류가 있더라도 오류 발생지점까지는 코드가 실행되기 때문에 간단히 실행시켜가면서 계속 조각조각 개발해나갈 수 있는 점이 아직 큰 장점이라고 생각한다.

최근에는 문자열을 다루기 간편하면서도 `numpy`, `scipy`, `pandas` 처럼 데이터를 다루는 라이브러리들이 잘 지원되고 있어서 많은 AI 엔지니어들이 Python3을 활용하고 있다.

단점은 인터프리팅 언어이기 때문에 3.10 버전까지는 소스 코드를 미리 컴파일 되어있는 바이트 코드로 치환해서 실행하는 방식으로 동작해서 속도가 느린 편이었다. 그러나 3.11 이후로는 그런 부분들이 개선되었다.

## Python3 Interpreter Project

아래는 Python3의 인터프리터가 어떤 과정으로 코드를 해석하고 실행하는지에 대한 과정을 소스코드 관점에서 간략하게 따라가 본 기록이다. 이 작업은 순전히 흥미 중심으로 진행한 과정이라서 많은 중간과정들이 생략되어있다. 다만 Python3 프로젝트를 볼 때 어디서부터 따라가면 좋을지 가이드라인으로 참고하기 위한 용도이다.

### Entrypoint

현재는 리눅스 환경에서 python3.14 기준으로,

`python.c`에 entrypoint가 정의되어있기 때문에 python 실행기에서부터 시작하면

`Programs/python.c:main()`
-> `Modules/main.c:Py_BytesMain()`
-> `Modules/main.c:pymain_main()`
-> `Modules/main.c:pymain_init()`

순서로 코드가 실행되어 매개변수를 분석하고 실행환경을 준비한다.

### Initialize Interpreter

이때 `pymain_init` 함수에서는 python3의 인터프리터를 위한 환경설정을 불러와서 실행 환경을 초기화하는 bootstraping 과정을 수행한다.

`Modules/main.c:pymain_init()`
-> `Python/pylifecycle.c:Py_InitializeFromConfig()`
-> `Python/pylifecycle.c:_PyRuntime_Initialize`

를 실행하면서 초기화 상태를 확인하고,

`Python/pylifecycle.c:Py_InitializeFromConfig()`
-> `Python/pylifecycle.c:pyinit_core()`
-> `Python/pylifecycle.c:pyinit_config()`
-> `Python/pylifecycle.c:pycore_init_runtime()`
-> `Python/import.c:_PyImport_INIT()`
-> `Python/import.c:init_builtin_modules_table()`

단계로 함수를 호출하면서 `pycore_init_runtime` 함수 내부에서 Runtime 환경이 현재 인터프리터가 실행중인지, 아니면 실행 가능한 상태인지 확인한 후 전역적으로 `import` 명령어를 수행할 수 있도록 메모리 상태를 초기화한다. 이 단계가 정상실행 되어야 그 뒤에 `pycore_init_runtime` 함수에서 인터프리터의 상태를 `Enable`로 변경한다.

이제 다시 `pyinit_config` 함수로 돌아가야 한다.

`Python/pylifecycle.c:pyinit_config()`
-> `Python/pylifecycle.c:pycore_create_interpreter()`

위의 `pycore_int_runtime` 바로 뒤에 `pycore_create_interpreter` 함수가 실행되는데, 이 안에서 비로소 인터프리터를 생성하고 Thread를 함께 만든다. GIL(Global Interpreter Lock) 또한 이 함수 안에서 초기화된다.

### __main__ Module

위에서 인터프리터를 생성한 뒤 Python 실행기는 진입 파일을 모듈로 로드한 뒤 __main__ 모듈로 참조하도록 환경을 구축한다. 인터프리터 초기화를 마친 뒤에 실행되기 때문에 `pymain_main` 함수로 돌아와야 한다.

`Modules/main.c:pymain_main()`
-> `Modules/main.c:Py_RunMain()`
-> `Modules/main.c:pymain_run_python()`

이 프로세스에서 대기상태인 인터프리터 객체를 불러온 뒤 config 객체에서 실행 대상을 확인한다. `-c` 스위치로 인라인 코드를 입력 받은 경우, `-m` 스위치로 모듈, 또는 모듈 경로를 입력 받은 경우, 파일 이름을 직접 입력받은 경우, 그리고 어떤 입력도 없는 경우, REPL로 진입하는 경우가 있다. 지금은 ".py" 파일을 입력받은 경우로 추적 중이기 때문에 파일 입력 분기로 진행한다.

`Modules/main.c:pymain_run_python()`
-> `Modules/main.c:pymain_run_file()`
-> `Modules/main.c:pymain_run_file_obj()`
-> `Python/pythonrun.c:_PyRun_AnyFileObject()`
-> `Python/pythonrun.c:_PyRun_SimpleFileObject()`
-> `Python/pythonrun.c:set_main_loader()`

이러한 순서로 함수가 호출되면서 파일 객체를 분석해서 모듈을 불러온다. `_PyRun_SimpleFileObject`의 내부에는 `pyc` 파일이 존재하는지 체크하는 함수가 있어서 직후에 모듈 로더를 생성하는 단계에서 `SourceFileLoader`, 또는 `SourcelessFileLoader` 두가지 로더중 하나로 분기하게 된다. 어느쪽으로 가던지 코드는 `set_main_loader` 함수에 도달하게 되어있다.

가장 마지막 줄에 있는 `set_main_loader` 함수에서 유저가 의도한 스크립트 파일을 메모리 위에 모듈로 가져오는 동작이 정의되어있다.

``` c
// Python/pythonrun.c
435 |static int
436 |set_main_loader(PyObject *d, PyObject *filename, const char *loader_name)
437 |{
...
445 |    PyObject *loader = PyObject_CallFunction(loader_type,
446 |                                             "sO", "__main__", filename);
...
458 |}
```

코드에 있는 `PyObject_CallFunciton`은 C-API에 정의되어있는 함수인데, Python 인터프리터를 통해 Python3 코드로 정의되어있는 함수를 호출하고 실행하도록 해준다. 파라미터 순서대로,

* `loader_type`; 호출하려는 함수 객체의 이름
* `"sO"`; 뒤에 추가될 가변 매개변수 배열에 대한 정보(sO는 각각 str, object 타입으로 변수가 할당된다는 의미이다.)
* `"__main__"`; 위의 "sO"에서 "s" 위치에 대응되는 값
* `filename`; 위의 "sO"에서 "O" 위치에 대응되는 값

이러한 구조로 되어있고, `loader_type` 변수에 할당되는 문자열인 `SourceFileLoader`와 `SourcelessFileLoader` 모두 `FileLoader` 객체를 상속하는 객체의 이름이다. 또한 `FileLoader`는 `def __init__(self, fullname, path)` 라는 헤더로 정의되어있는데, 이렇게 호출되는 것과 동일한 효과를 갖는다.

``` python
loader = SourceFileLoader("__main__", filename)
# or
loader = SourcelessFileLoader("__main__", filename)
```

이렇게 되면 우선 최초 진입 모듈을 가리키는 `__main__`이 있는 file이 메모리에 로드된다.

이제 분기에 따라 다르지만 `set_main_loader` 함수 직후에 나오는 `run_pyc_file` 또는 `pyrun_file` 함수가 모듈의 실행 절차를 수행한다.

`Python/pythonrun.c:_PyRun_SimpleFileObject()`
-> `Python/pythonrun.c:run_pyc_file()`

여기에서는 기존에 컴파일되어있던 캐시파일인 `.pyc`를 읽고 내부에서 `run_eval_code_obj` 함수를 통해 사용자가 실행을 요청한 파일을 모듈로써 인터프리터 위에서 동작을 실행한다.

`Python/pythonrun.c:_PyRun_SimpleFileObject()`
-> `Python/pythonrun.c:pyrun_file()`

이 안에서는 파일을 모듈로 메모리에 로드하기 위한 절차로 메모리 영역을 확보하고, 파일을 AST(Abstract Syntax Tree) Parser를 이용해서 코드를 컴파일 한 후 코드 객체를 인터프리터 쓰레드에서 실행하는 것으로 유저의 코드로 완전히 진입하게 된다.

## Conclusion

전부터 한번은 Python3 실행기가 최초로 시작될 때, 코드의 진입 지점부터 모듈 실행지점까지 과정을 한번은 분석해보고 싶었다. 이번 프로젝트는 3.14버전을 기준으로 했지만, 나중에는 3.10 이전 버전의 프로젝트도 분석해서 코드 해석과정이나 바이트코드로 컴파일하는 과정에 어떤 변경이 있었는지 한번 살펴보면 좋을 것 같다.
