---
layout: post
title: Short circuiting을 이용해 if-else 구문을 축약하기
subtitle: 삼항연산자를 쓰기싫을때
comments: true
categories: ["Programming"]
tag: ["Python", "Short circuiting", "trick"]
---

## {variable} = {expr-1} if {condition} else {expr-2}

코딩하다 가끔 이런걸 쓰기 싫을때가 있다.

``` python
... 
x = 123

if x == 123:
    y = "123"
else:
    y = "NaN"
```

간단헤보이는 코드를 4줄이나 할애해야 하는데다 단순히 하나의 변수에 값을 할당하기만 하는데도 왠지 번잡한 느낌이 나서 좀 그럴싸하게 쓰고싶을때 보통 이렇게 바꿔쓴다.

``` python
x = 123
y = "123" if x == 123 else "NaN"
```

그런데, `and`/`or`를 이용하면 이렇게 할수있다.

``` python
x = 123
y = x == 123 and "123" or "NaN"
```

문법적으로 어색해보일 수 있지만, `and`와 `or`의 처리 방법을 이용한 것이다. 장점으로는 conditional 블럭을 앞으로 빼면서 조건부 값을 뒤에 모을 수 있기 때문에 True/False 조건에서 어떤 코드가 적용되는지 한눈에 구분이 된다는 것이다.

문제가 있다면 어색함이다. `{expr-1} if {condition} else {expr-2} == {condition} and {expr-1} or {expr-2}` 로 동치관계로 쓸 수 있지만, `if-else`가 좀 더 자연어에 가깝기 때문에 어색하게 느껴진다.

또 하나는 `and`와 `or`의 활용인데, 일반적으로 boolean 연산자로 기대하고 코드를 작성하기 때문에 이 구문에대한 이해가 없다면 결과를 예측할 수 없다.

그래도 재밌어서 한번 써봤다. 나중에 또 재밌는 `and`, `or` 구문 활용법이 생각나면 새로운 시도를 해보는것도 재밌을 것 같다.
