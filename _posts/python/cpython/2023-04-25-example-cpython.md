---
layout: post
title: CPython을 사용하는 C언어 코드와 Python3 코드 비교
subtitle: 기본적인 코드 구조를 파악하기 위한 간단한 예
comments: true
categories: ["Programming", "Python3", "CPython"]
tag: ["Python", "CPython", "Example"]
---

## Python3과 C언어 간의 관계

이런걸 하려고 하는 개발자라면 이미 C언어에서의 데이터 타입과 Python3에서 다루는 데이터 타입을 이미 이해하고 있을 것이라고 생각한다. 그래서 간단한 Python3 모듈과 C언어에서 표현되는 코드 구조를 간단하게 예로 기록해두려고 한다.


### Python3

```python
import operator

def ssum(items: list[int]):
    s = 0
    for item in items:
        s = operator.add(s, item)

    print(s)

    return s

def main():
    print(ssum([1,2,3,4,5]) == 15)

main()
```

### C

```c
#define PY_SSIZE_T_CLEAN
#include <Python.h>


PyObject* py_operator_add = NULL;

PyObject* py_ssum(std::vector<int>& values) {
    // BEGIN: s = 0
    PyObject* py_s = PyLong_FromLong(0);
    // END: s = 0

    // for (int idx = 0; idx < size; idx ++) {
    for (std::vector<int>::iterator iter = values.begin();iter != values.end(); iter++){
        // BEGIN: s = operator.add(s, item)
        PyObject* py_args = PyTuple_New(2);
        PyTuple_SetItem(0, py_s);
        PyTuple_SetItem(1, PyLong_FromLong(*iter));

        py_s = PyObject_Call(py_operator_add, py_args, NULL);

        Py_DECREF(py_args);
        // END: s = operator.add(s, item)
    }

    // BEGIN: print(s)
    PyObject_Print(py_s)
    // END: print(s)

    // BEGIN: return s
    return py_s;
    // END: return s
}

void py_main() {
    // BEGIN: ssum([1,2,3,4,5]) == 15
    std::vector<int> values = {1,2,3,4,5};

    PyObject* py_res = py_ssum(values);

    PyObject_Print(PyBool_FromLong(PyLong_AsLong(py_res) == 15));

    Py_DECREF(py_res);
    // END: ssum([1,2,3,4,5]) == 15
}

int main(char** argv, int argc) {
    Py_Initialize();

    // BEGIN: import operator
    PyObject* py_operator = PyImport_ImportModule_Operator("operator");
    py_operator_add = PyObject_GetAttrString(py_operator, "add");
    Py_DECREF(py_operator);
    // END: import operator

    py_main();

    Py_DECREF(py_operator_add);

    Py_Finalize();
}
```

Python3의 각종 함수들을 사용하기 위한 C-API 코드는 Python3에 비하면 몇배 길다. C언어에서 포인터 변수를 생성하고 값을 할당하고 Callable 객체를 호출하고 더 이상 사용하지 않는 변수는 GC가 제거할 수 있도록 참조 카운터를 일련의 작업들이 Python3에서는 몇 줄만으로 표현 되는 것이다.

위의 코드들을 서로 대응하도록 비교하면 이렇게 볼 수 있다.

Python3 | C
---|---
1: import operator | 50: PyObject* py_operator = PyImport_ImportModule_Operator("operator");<br/>52: Py_DECREF(py_operator);<br/>...<br/>57: Py_DECREF(py_operator_add);
3: def ssum(items: list[int]): | 7: PyObject* py_ssum(std::vector<int>& values) { ... }
4: s = 0 | 9: PyObject* py_s = PyLong_FromLong(0);
5: for item in items: | 13: for (std::vector<int>::iterator iter = values.begin();iter != values.end(); iter++) { ... }
6: s = operator.add(s, item) | 5: PyObject* py_operator_add = NULL;<br/>51: py_operator_add = PyObject_GetAttrString(py_operator, "add");<br/>15: PyObject* py_args = PyTuple_New(2);<br/> 16: PyTuple_SetItem(0, py_s);<br/>17: PyTuple_SetItem(1, PyLong_FromLong(*iter));<br/>19: py_s = PyObject_Call(py_operator_add, py_args, NULL);<br/>21: Py_DECREF(py_args);
8: print(s) | 26: PyObject_Print(py_s)
10: return s | 30: return py_s;
12: def main(): | 34: void py_main() { ... }
13: print(ssum(\[1,2,3,4,5\]) == 15) | 36: std::vector<int> values = {1,2,3,4,5};<br/>38: PyObject* py_res = py_ssum(values);<br/>40: PyObject_Print(PyBool_FromLong(PyLong_AsLong(py_res) == 15));42: <br/>Py_DECREF(py_res);
15: main() | 55: py_main();

코드가 서로 완전히 순차적으로 대응할수가 없기 때문에 C++코드 라인이 다소 뒤죽박죽이 될 수 밖에 없어서 보기 어렵긴 하지만, 각 라인의 동작이 어떻게 이해될 수 있는지를 중심으로 이해할 수 있을 것 같다. 

## 결론

Python3은 확실히 편한 언어이고 굉장히 추상화된 언어라는 것을 알 수 있다. 이 코드는 어떻게 Python3에 있는 모듈, 함수 등을 불러와서 C언어 코드 위에서 호출할 수 있는지 확인하기위한 예제였고, 다음에는 C언어로 만든 함수를 Python3에서 호출할 수 있고, 어떤 코드를 작성해야 하는지 적어보려고 한다.
