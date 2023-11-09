---
layout: post
title: Python3에서 함수 합성
subtitle: Python3에서 함수 합성
parent: Trick of code
grand_parent: Python
comments: true
categories: ["Programming", "Python3", "Code"]
tag: ["Python3", "FP", "Function"]
---

## 함수 합성이란

간단히 말하면 f(x)와 g(x)가 있을 때, h(x) = g(f(x))를 h = g &compfn; f 으로 표현하는 것을 말한다. 쓰는 방식만 변한 것으로 보이는데, 사실 여러 함수를 중첩해서 수식으로 정리하려고 하면 너무 괄호가 많아지는 등의 이유로 수식의 가독성이 떨어져서 쓰는게 아닐까 하고 생각하고있다.

좀 더 파고들어보면 g(f(x))는 f: X -> Y, g: Y -> Z인 경우에, X -> Y -> Z 의 관계를 갖는 함수를 간략화해서 X -> Z로 표현하려고 하는 것을 의미한다. 추가로 호기심에 친구에게 물어보니 다변수함수도 함수 합성이 가능하다고 한다. 근데 너무 복잡해지기도 하고 표현도 이상해지니까 그냥 일변수함수에 한정해서 합성 함수 코드를 만들어봤다. 

## composition(g, f)(x) == g(f(x)) 

파이썬에서 하면 이렇게 표현할 수 있다.

``` python
def composition(*funcs):
    return funcs[0] if len(funcs) == 1 else lambda x: funcs[0](composition(*funcs[1:])(x))
```

코드를 한줄로 표현하려고 최대한 노력을 했던 코드인데, `if-else`를 풀어쓰면 이렇게 된다.

``` python
def composition(*funcs):
    if len(funcs) == 1:
        return funcs[0]
    else:
        return lambda x: funcs[0](composition(*funcs[1:])(x))
```

`len(funcs) != 1`인 조건을 주목해야 하는데, 반환하는 변수는 어디까지나 변수를 하나 갖는 callable 변수여야 한다. 파라미터가 1개인 익명 함수를 반환하는 이유가 바로 이 때문이다. 그 위의 `return funcs[0]`도 파라미터가 1개인 함수를 의미한다.

호출하는 함수들은 재귀적으로 호출한다. 메모리 사용에서는 낭비가 있겠지만 코드 자체의 구조가 간결해지고 조금이라도 직관적으로 이해할 수 있을것이다. 

따라서 이 결과는 이렇게 나온다. 

```python 
a = lambda x: x + 1
b = lambda x: x ** 2
b(a(2)) == composition(b, a)(2) // True
```

### 위키피디아

위키피디아에는 다양한 언어로 composition을 구현한 코드들이 있는데, 이 중에서 파이썬 항목에 이런 코드가 있다. 

``` python
from functools import reduce

def composite_funcs(*funcs: Callable) -> Callable:
    return reduce(lambda f, g: lambda x: f(g(x)), funcs)
```

이 코드는 `reduce` 함수를 이용해서 위에서 만든 `if-else` 구조를 제거했다. 좀 더 간결한 코드가 만들어졌고 재귀도 하지 않기 때문에 코드 구조가 훨씬 단순해졌지만, 개인적으로는 `reduce` 함수를 사용하지 않도록 만들고싶어 해본 시도였던 것이었고, 실제로 사용하기엔 좀 미묘한 면이 있을 수도 있을것 같다. 

## 참고

* [Function composition in Wikipedia](https://en.wikipedia.org/wiki/Function_composition)
* [Function composition(Computer science) in Wikipedia](https://en.wikipedia.org/wiki/Function_composition_(computer_science))
