---
layout: post
title: 알고리즘 최적화 구현에 대해서
subtitle: 파이콘 코리아 2019 리뷰 - Optimization algorithm implementation in Python
comments: true
categories: ["Technology"]
tag: ["python", "pycon", "pycon 2019", "algorithm"]
---

문제는 목적 함수와 제약 함수로 정의할 수 있다.

목적 함수란 어떤 작업, 알고리즘을 최적화 하기 위해 그 작업, 알고리즘에 사용되는 변수들의 관계를 표현한 함수를 의미한다. 이 변수들의 상관관계를 분석해서 최적값을 찾아내는 것이 목표이다.

제약 함수란 각 변수들이 충족해야 할 조건, 제약들을 함수로 표현한 것을 의미한다. 제약이란 변수가 취할 수 있는 값의 범위, 혹은 값의 한계를 의미하므로 등호제약조건, 부등호제약조건으로 수식화된다.

변수, 계수, 상수란 목적 함수, 제약 함수를 구성하는 모든 수를 의미한다. 변수는 말 그대로 변할 수 있는 수, 게수는 변수간의 상관관계를 정의하는 수, 상수는 외부 요인으로 우리가 컨트롤할 수 없는 수이다.

최적화 문제들은 다음과 같은 종류가 있다.

LP([Linear programming](https://en.wikipedia.org/wiki/Linear_programming)) 
IP([Integer programming](https://en.wikipedia.org/wiki/Integer_programming)) 
> NP-complete
QP([Quadratic Programming](https://en.wikipedia.org/wiki/Quadratic_programming))
MILP((Mixed-Integer Linear Programming)
> NP-hard
MIQP(Mixed-Integer Quadratic Programming) 
MIQCP(Mixed-Integer Quadratic Constraints Programming)
NLP(Non-linear Programming)
