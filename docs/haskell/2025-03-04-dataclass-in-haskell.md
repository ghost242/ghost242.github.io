---
layout: post
title: 데이터 클래스에 대해서
subtitle: Basic of dataclass in Programming
parent: Haskell
has_toc: true
comments: true
categories: ["Programming", "Haskell"]
tag: ["Haskell", "Functional programming", "Dataclass"]
---

## Abstract

Functional Programming 을 공부하기 위해서 Haskell을 보던 중에 이해하기 어려웠던 개념은 여기서부터 시작이었다. 그래도 이 시점까지는 그렇게까지 어렵지 않았는데, 어떻게 보면 그나마도 쉬웠던 마지막 지점이 바로 여기였다. 이 문서는 현재 아주 널리 퍼져있는 절차적 언어와 객체지향 언어와 차이가 생기는 이 개념에 대해 내가 어떻게 이해하고 있는지를 정리한 문서이다.

## Basic

데이터 클래스는 흥미롭게도 두가지 단어의 조합으로 되어있다. 하나는 데이터, 그리고 또 하나는 클래스이다. 데이터는 말 그대로 데이터이고, 클래스는 집합론에서의 대상을 의미하는 것으로 보인다. 그 이유는 앞의 문서에서도 이야기했지만, 이 언어가 다루는 가장 중요한 객체가 바로 함수이기 때문이다.

### Class in the Set theory

잠깐 정리하면 집합론에서의 함수는 어떤 집합의 정의역과 다른 집합의 치역의 관계를 정의한다. 관계는 1:1, 1:N, N:1, M:N 관계가 있을 수 있지만, 그 중에서 함수는 1:1과 N:1 관계를 함수라고 한다.

그럼 클래스는 무엇인가 하면, 위키피디아에서 정의를 참조해보면 이렇게 되어있다.

`집합론에서 모임 또는 클래스(영어: class)는 특정한 성질을 만족하는 집합(혹은 그 외의 수학적 대상)을 모은 것이다.` ([클래스 in Wikipedia](https://en.wikipedia.org/wiki/Class_(set_theory)))

또는

`In set theory and its applications throughout mathematics, a class is a collection of sets (or sometimes other mathematical objects) that can be unambiguously defined by a property that all its members share.` ([Class in Wikipedia](https://en.wikipedia.org/wiki/Class_(set_theory)))

라고 되어있다. 솔직히 영문 위키의 정의가 좀 더 이해하기 쉬웠지만, 아무튼 이런 의미이다.

왜 이 내용을 알아야 하는지, 그것은 이 개념이 OOP에서의 클래스와는 비슷하면서도 다르다는 것을 확실히 하고 싶어서이다. OOP에서의 클래스는 이 추상화된 개념들을 통해 구체화한 객체(Instance)의 공통속성을 묶음으로 정의하기 위하여 사용되는데, 목적이 객체에 있음이 중요하다. 객체가 시스템 안에서 할 수 있는 기능이나 행위를 Method로 정의해서 객체에 종속시키기 때문이다.

하지만 데이터 클래스는 이렇게 종속된 Method가 없다. 하위의 멤버인 집합을 모음으로 갖고있을 뿐이다. 상속과 다형성의 개념도 갖고있고, 클래스를 공역/치역으로 두는 함수도 있다. 

## Dataclass in Haskell

## Conclusion
