---
layout: post
title: 타입에 대해서
subtitle: Basic of type in Programming
parent: Haskell
has_toc: true
comments: true
categories: ["Programming", "Haskell"]
tag: ["Haskell", "Functional programming", "Type"]
---

## Abstract

Functional Programming 을 공부하기 위해서 Haskell을 보던 중에 이해하기 어려웠던 개념은 여기서부터 시작이었다. 이 언어는 다른 언어에서는 약간의 인간의 직관에 의존하는 듯한 개념들을 전부 수학적인 증명을 토대로 하고있는것 처럼 보이는데, 그 이유가 타입에 있다는 생각을 하게 되었다.

Haskell은 강한 타입 언어에 속한다고 알려져 있다. 이 언어는 얼마나 엄격한지 C언어에서 제한적으로 허용하는 암시적 형변환(Implicit type casting)을 절대 허용하지 않는다. 예를 들어 우리는 2 ^ 10이 암묵적으로 Integer를 반환하는 함수라고 인지한다. 하지만 Haskell 안에서 거듭제곱을 의미하는 이 ^ 연산자는 Num 타입을 상속하는 타입 a를 입력받아 같은 타입 a를 출력한다고 정의된다. 말하자면 Int로 입력하면 반드시 Int가 출력되고, Floating이 입력되면 Floating 타입이 출력된다는 의미이다.

이 문서는 바로 Haskell에서 관리하는 타입에 대해 정리하는 문서이다.

## Basic

간단히 내용을 나열했지만 사실 타입이란 컴퓨터가 자료를 적절하게 다루기 위해 다루는 것으로 이해하면 충분하다. 하지만 Haskell에서, 적어도 지금 문서를 작성중인 내 시점에서, 비슷하게 사용되는 키워드가 아주 다양하다. 

### Class in the Set theory

잠깐 정리하면 집합론에서의 함수는 어떤 집합의 정의역과 다른 집합의 치역의 관계를 정의한다. 관계는 1:1, 1:N, N:1, M:N 관계가 있을 수 있지만, 그 중에서 함수는 1:1과 N:1 관계를 함수라고 한다.

그럼 클래스는 무엇인가 하면, 위키피디아에서 정의를 참조해보면 이렇게 되어있다.

`집합론에서 모임 또는 클래스(영어: class)는 특정한 성질을 만족하는 집합(혹은 그 외의 수학적 대상)을 모은 것이다.` ([클래스 in Wikipedia](https://en.wikipedia.org/wiki/Class_(set_theory)))

또는

`In set theory and its applications throughout mathematics, a class is a collection of sets (or sometimes other mathematical objects) that can be unambiguously defined by a property that all its members share.` ([Class in Wikipedia](https://en.wikipedia.org/wiki/Class_(set_theory)))

라고 되어있다. 영문 위키의 정의가 좀 더 이해하기 쉽게 느껴진다.

### Class in the Knowledge reprsentation

위의 설명과 지금 하려는 Class에 대한 설명은 약간의 차이가 있다. 아마도 그 차이는 실제 세계를 수학적 모델로 다루는 Class의 의미와 객체로서의 분류를 의미하는 Class와의 의미 차이때문인 것으로 보인다. 위키피디아에서는 지식의 표현에 있어서 Class를 다음과 같은 문구로 정의하고 있다.

`A class is a collection of individuals or individuals objects.` ([클래스 in Wikipedia](https://en.wikipedia.org/wiki/Class_(knowledge_representation)))



본격적인 문법 수준에서의 표현으로 넘어가기 전에 조금만 더 이론적인 시점에서 살펴본 후 넘어가려고 한다. 

## Dataclass in Haskell

## Conclusion
