---
title: AST로 알아보는 Python3 언어 구조
subtitle: Module indexing을 위한 객체간의 참조관계
parent: Basic syntax 
grand_parent: Python
comments: true
categories: ["Programming", "Python3", "Syntax"]
tag: ["Python", "Syntax", "AST"]
---

## Abstract

LLM을 이용해서 프로그램을 작성하다 보면 생성되는 코드 안에 높은 확률로 할루시네이션이 포함된다. LLM으로 코드 작업을 할 때 가장 자주 하는 작업이 코드 생성, 리팩토링 두가지 인데, 이 두 작업 모두 할루시네이션이 발생한다. 특히 리팩토링 작업에서는 기존에 없던 로직을 추가하거나, 코드의 구조를 변경하는 등의 문제까지 있지만 이 문제는 다음에 다시 다뤄보도록 한다. 

이 문서 에서는 단일 프로젝트 내에 구현되어있는 모듈과 코드 컴포넌트간의 참조관계를 구조적으로 캐싱하는 방법과 빠르게 탐색할 수 있도록 DB에 저장하는 방법 들을 정리해보려고 한다. 

## Overview

어떤 대상간의 관계를 표현하는데 가장 좋은 것은 아마도 Graph일 것이다. Graph는 Node와 Edge로 구성되어있는데, 특히 각 데이터 객체간의 관계를 표현하기 적합한 자료구조라고도 할 수 있다. Network가 주로 Graph로 표현되고 최근에는 LLM에서 토큰간의 관계를 표현하거나 흔히 지식으로 부르는 정보간의 관계를 입체적으로 구현하기 위해 GraphDB가 많이 활용되기도 한다. 

이 포스트에서의 주요하게 다룰 언어인 Python3는 몇가지 언어 패러다임을 포함하는데, 그 중에서 Node로 Class와 Function, 을 중심으로 하고 값을 보존하기 위한 Variable, 그리고 이 요소들을 포함하는 Module을 대상으로 해보려고 한다.

우선은 아래의 하위 항목에서 각 요소들의 다른 요소와의 관계를 정리해본다.

### Function

* Function: Call, Has, Argument

* Lambda: Has, Argument

* Variable: Has, Argument, Referenced

* Class: Call, Has, Argument

* Module: Import, ImportFrom, Alias

### Class

### Variable

### Module

### Python Sample

여기에서는 Python3의 문법에 따라 최소한의 단위로 예제를 정의 해보려고 한다. Function, Class, Variable 세가지를 조합해서 몇가지 가능한 샘플을 작성하고, 이 코드를 대상으로 아래에서 Abstract Syntax Tree(AST)로 표현할 것이다. 

``` python
VALUE = 0

def callee(): pass

def caller():
    global VALUE
    local_value: float = 0.0

    callee()

def fn(a) -> int:
    """This is fn"""
    def ffn(): pass

    ffn()
    return 0

class cls:
    """This is sample class"""
    def super_method(self):
        pass

class ccls(cls):

    def sub_method(self):
        """This is the method in subclass"""

        self.super_method()

```


## AST

## Conclusion
