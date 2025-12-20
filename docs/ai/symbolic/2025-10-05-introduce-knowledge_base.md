---
layout: post
title: Knowledge Base 1. Prolog으로 살펴보는 지식 베이스의 구조
subtitle: 관계를 중심으로 지식을 표현하는 방법
parent: Symbolic AI
grand_parent: AI
comments: true
categories: ["Programming", "Symbolic", "Knowledge Base"]
tag: ["Knowledge Base", "Logic programming", "Prolog"]
---

## 지식 베이스란 무엇인가

데이터베이스는 보통 “값(value)”을 저장하고 관리하기 위한 구조다.  하지만 현실의 많은 문제는 단순히 데이터를 개별적으로 다루는 것 보다, 데이터의 대상 간의 관계와 의미를 표현하는 것이 더 중요하다.

예를 들어 “CPU는 메인보드의 일부이다”, “메인보드는 컴퓨터의 일부이다”라는 정보는 데이터라기보다 지식(knowledge) — 즉, 관계로 이루어진 사실이다.

이렇게 사실(facts)과 관계(rules)를 체계적으로 표현해두고, 시스템이 그 관계를 기반으로 새로운 결론을 추론(inference) 할 수 있도록 만든 구조가 바로 `지식 베이스(Knowledge Base)`이다.

## Prolog의 기본 문법 요약

`Prolog`는 이런 지식 구조를 기술하기에 적합한 Logic programming 패러다임을 기반으로 하는 언어이다. 절차를 명령하는 대신, 무엇이 참인가를 선언하는 방식으로 동작한다. 이 글에서는 지식 구조를 표현하기위해 `Prolog`를 활용하고, 최대한 간단하게 문법 구조를 설명한다.

`Prolog`의 핵심 구성 요소는 세 가지다:

1. `사실(Fact)`: “A는 B의 일부이다.” 같은 참인 문장  

2. `규칙(Rule)`: “A가 B의 일부이거나, B가 C의 일부이면 A는 C의 일부이다.” 같은 논리적 정의  

3. `질의(Query)`: “CPU는 컴퓨터의 일부인가?” 같은 질문  

예를 들어 문법은 다음처럼 간단하다.

```prolog
fact.                      % 사실
rule :- condition.         % 규칙
?- query.                  % 질의
```

이 구조를 통해 `Prolog`는 데이터가 아닌 논리적 관계를 저장하고 추론한다.

## 예제: 컴퓨터 부품을 이용한 지식 베이스

이재 지식 베이스를 이해하기 위해 예제로 컴퓨터의 구성 관계를 Prolog 문법을 활용해서 표현해보려고 한다.

### 사실 정의

먼저 각 부품이 어떤 구성 요소에 속하는지를 사실로 정의한다.

```prolog
% 상위 구조
part_of(motherboard, computer).
part_of(power_supply, computer).
part_of(storage, computer).
part_of(case, computer).
part_of(peripheral, computer).

% 메인보드 구성
part_of(cpu, motherboard).
part_of(memory, motherboard).
part_of(gpu, motherboard).
part_of(chipset, motherboard).

% CPU 내부 구성
part_of(core, cpu).
part_of(cache, cpu).
part_of(alu, cpu).
part_of(control_unit, cpu).

% 저장 장치 구성
part_of(hard_disk, storage).
part_of(ssd, storage).

% 주변 장치 구성
part_of(keyboard, peripheral).
part_of(mouse, peripheral).
part_of(monitor, peripheral).
part_of(speaker, peripheral).
```

이것은 “CPU는 메인보드의 일부이다”, “메인보드는 컴퓨터의 일부이다”와 같은 기초 지식*이다.

### 규칙 정의

이제 이 관계를 바탕으로 더 넓은 의미의 포함 관계를 추론하는 규칙을 정의할 수 있다.

```prolog
is_part_of(X, Y) :- part_of(X, Y).
is_part_of(X, Y) :- part_of(X, Z), is_part_of(Z, Y).
```

이 규칙은

* X가 Y의 직접적인 부품이면 참이다.
* X가 Z의 부품이고, Z가 Y의 일부라면, X도 Y의 일부로 본다.

즉, 계층적인 관계를 재귀적으로 추론하도록 만든 것이다.

### 질의 수행

이제 시스템에 질문을 던져보자.

```prolog
?- is_part_of(cpu, computer).
```

Prolog는 다음과 같은 과정을 통해 결과를 찾아낸다.

```plaintext
alu → cpu → motherboard → computer
```

결과: `true.`

또는 컴퓨터에 포함된 모든 부품을 물을 수도 있다.

```prolog
?- is_part_of(X, computer).
```

출력:

```plaintext
X = motherboard ;
X = cpu ;
X = memory ;
X = gpu ;
X = chipset ;
X = core ;
X = cache ;
X = alu ;
X = control_unit ;
X = power_supply ;
X = storage ;
X = hard_disk ;
X = ssd ;
X = case ;
X = peripheral ;
X = keyboard ;
X = mouse ;
X = monitor ;
X = speaker.
```

이 결과는 지식 베이스가 단순한 데이터 저장소가 아니라, 관계 정의만으로 논리적 전체 구조를 스스로 탐색하는 시스템임을 보여준다.

## 결론

이 간단한 예제는 지식 베이스의 핵심 구조를 이해하기 위해 작성했따. 지식 베이스는 데이터를 나열하는 것이 아니라, 데이터로 표현되는 사물 간의 관계를 정의하고 그 관계를 통해 새로운 사실(fact)을 추론하는 시스템이다.

Prolog는 이러한 구조를 가장 단순하고 명확한 형태로 표현할 수 있게 해준다. 즉, “무엇을 할 것인가”가 아니라 “무엇이 참인가”를 선언하고, 그로부터 시스템이 스스로 답을 찾아내는 방식을 실험할 수 있는 언어다. 이 언어의 문법을 이용해 사실을 나열하고 규칙을 정의한 뒤 질의를 통해 지식을 인출하는 방식을 표현했다.

이것이 지식 베이스의 본질이며, 오늘날의 인공지능이 다루는 지식 표현(knowledge representation)의 출발점이다.
