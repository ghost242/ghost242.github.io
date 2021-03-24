---
layout: post
title: 파이썬 함수 객체에 대해서
subtitle: 파이썬에서 구현된 일급 함수 객체에 대한 연구
comments: true
categories: ["Python"]
tag: ["python", "function", "functional", "object"]
---

## 함수형 언어에 대해서

위키피디아에서 함수형 언어에 대해 찾아보면 다음과 같은 문구가 가장 처음에 적혀있다.

In computer science, functional programming is a programming paradigm—a style of building the structure and elements of computer programs—that treats computation as the evaluation of mathematical functions and avoids changing-state and mutable data.

이 말은, 함수형 패러다임이란 현대의 프로그래밍 언어에서 흔히 다루고있는 function을 좀 더 수학적으로 다루겠다는 의미이다. 수학에서의 함수는 입력 파라미터가 같은 값이면 출력 결과도 항상 같기 때문이다. 이 안에는 어떤 내부/외부 상태의 변화나 입력 값이 변경되거나 하는 일이 없다.

우리는 여러 언어에서 다양한 방식으로 이러한 명칭을 사용하는데, 대개 다음과 같은 용어를 혼용해서 부르거나 구분해서 부르기도 하는데, [Oxford learner's dictionary](https://www.oxfordlearnersdictionaries.com/)를 찾아보면 다음과 같은 의미를 갖는 것으로 나온다.

* Procedure:

    > 1. [countable, uncountable] procedure (for something) a way of doing something, especially the usual or correct way
    > 2. [uncountable] the official or formal order or way of doing something, especially in business, law or politics
    > 3. [countable] (medical) a medical operation

* Subroutine:

    > 1. a set of instructions which perform a task within a program

* Method:

    > 1. [countable] a particular way of doing something
    > 2. [uncountable] the quality of being well planned and organized

* Function:

    > 1. [countable, uncountable] a special activity or purpose of a person or thing to fulfil/perform a function
    > 2. [countable] a social event or official ceremony
    > 3. **[countable] (mathematics) a quantity whose value depends on the varying values of others. In the statement 2x=y, y is a function of x.**
    > 4. [countable] (computing) a part of a program, etc. that performs a basic operation

잘 보면 대강 어떤 의미를 가져오고 있는지 알 수 있다. 모두 뭔가 하는 방법을 의미하고있는데, Function에는 수학적 용어로서의 함수가 따로 있다. 함수형 패러다임은 바로 이 부분을 살린 패러다임이라고 볼 수 있다.

이 패러다임의 특징이 바로 이 함수의 수학적 정의에 있다. 중학교 수학에서 우리가 배울 수 있는 수학에서의 함수의 특징 중에 하나는 입력에 대해 출력이 항상 일정하다는 것이다. 이를 위해서 프로그램에서는 함수 내부에는 파라미터와 이미 정적으로 코드에 정의된 상수 이외에 외부요인에 의해 변화하는 변수가 사용될 수 없다. 함수는 반드시 같은 입력에는 같은 출력을 보장해야 하기 때문이다. 이는 수학에서 사용되는 사상(Mapping, 또는 Function)과 의미가 직접 관계 되기 때문이다.

다시 말하자면, 임의의 함수 f의 파라미터가 t개 있다고 하면, 이 파라미터는 t 차원의 vector로 표현할 수 있고, 정의역 M을 정의할 수 있다. 그리고 함수에서 반환되는 값이 s개의 집합이라면, 마찬가지로 s 차원의 vector로 표현하고 치역 N으로 정의할 수 있다. 이제 우리는 함수 f:M->N을 정의할 수 있다.

그러면 여기서 의문을 가질 수 있다. 과연 우리가 작성하는 함수 f는 일대일 대응 조건을 만족해야 하는가. 결론부터 말하자면 그렇다고 할 수 있다. 정의역과 치역이 1:1 대응이 되지 않는 함수가 작성될 경우 함수를 개발한 개발자는 물론이고 사용자 또한 함수를 호출하는데 부담을 가질 수 밖에 없다. 따라서 함수를 작성하는 개발자는 입력에 대해 출력을 1:1 대응이 되도록 할 필요가 있다. 특히 정상적인 출력에 경우에는 더 그렇다. 그래야 사용자가 입력값에 대해 출력값을 예측할 수 있다.

프로그래밍 영역에서 함수라는 요건을 충족하기 위한 특징들이 몇가지 있다.

### 일급 객체, 또는 고차 함수

위키피디아에서 일급객체는 다음과 같이 설명하고 있다.

> *In programming language design, a first-class citizen (also type, object, entity, or value) in a given programming language is an entity which supports all the operations generally available to other entities. These operations typically include being passed as an argument, returned from a function, modified, and assigned to a variable.*

말하자면 일급객체란 다른 객체와의 모든 연산을 지원하는 객체를 의미한다.

### 순수 함수

### 

## 일급 객체에 대해서

## 파이썬에서의 함수의 의미
