---
layout: post
title: Knowledge Base 2. Knowledge Base의 수학적 모델링
subtitle: 집합 이론과 Knowledge Base의 관계에 대해서
parent: Symbolic AI
grand_parent: AI
comments: true
categories: ["Programming", "Symbolic", "Knowledge Base"]
tag: ["Knowledge Base", "Set Theory", "Logic"]
---

## 문제 인식

이전 포스트에서 `Knowledge Base`를 `Fact`, `Rule`로 이루어진 데이터 구조로써 다뤄봤다. 이 내용을 엄밀하게 다루기 위해서 자료를 조사하고 GPT를 이용해서 내용을 정리해보던 중에 한가지 느꼈던 문제가 있었는데, 이 데이터 구조를 수학적으로는 어떻게 다룰 수 있는지 더 원론적인 내용으로 접근해볼 필요가 있다는 것이었다. 이런 직관은 앞서 작성했던 `Knowledge Base`와 `Inference Engine`을 통합해서 시스템으로 확장해 나가는 중에 느꼈던 문제가 포함되어있다.

## 집합과 지식 기반의 관계

`Prolog`에서 `Fact`, `Rule`, `Query` 세가지 구성요소가 있다고 했지만, `Fact`를 구성하는 `Element`가 반드시 필요하다. 3단논법을 다시 한번 예로 들어보면 이러하다.

> 1. 모든 사람은 죽는다.
> 2. 소크라테스는 사람이다.
> 3. 소크라테스는 죽는다.

이걸 집합으로 표현하면 이러하다.

> 1. $Socrates \in Human$
> 2. $Human \subseteq Mortal$
> 3. Then, $Socrates \in Mortal$

위 수식들은 "`Socrates`는 `Human`의 속해있고, `Human`은 `Mortal`의 부분집합이기 때문에, `Socrates`는 Mortal에 속한다", 라는 논리를 표현한다. 이렇게 표현했을 때 집합의 요소들과 `Knowledge Base`의 요소가 어떻게 연관되는지 알 수 있다.

> * Element ↔ Socrates
> * Fact ↔ $Socrates \in Human$
> * Rule ↔ $Human \subseteq Mortal$
> * Query ↔ $Socrates \in Mortal$

여기서 Query는 3단논법에서는 결론에 해당하지만, `Inference Engine`이 입력받는 질의로도 이해할 수 있다. 만약 $Socrates \in Mortal$ 가 Query가 아니고 이미 `Knowledge Base`에 있는 정보라면, `Fact`로 분류할 수도 있다.

### 지식 기반의 시각화

앞에서 정의한 집합 관계를 간단한 다이어그램으로 표현하면 이렇게 볼 수 있다. 그 뒤에 이걸 직관적으로 이해할 수 있다는 사실도 알게되었다.

``` mermaid
graph
    subgraph Mortal
        subgraph Human
            e((Socrates))
        end
    end   
```

그림으로 그렸을 때 `Mortal`은 `Socrates`를 갖고있는 `Human`을 부분집합으로 갖고있다. 따라서 3단 논리에서 1, 2를 `Knowledge Base`에 갖고 있다면, `Socrates`가 `Mortal`에 속한다는 것을 추론해낼 수 있는 것이다.

지금은 단순한 논리 구조 만으로도 `Knowledge Base` 안에서 새로운 `Fact`를 추론하는 것을 집합 이론을 기반으로 정리한 후 이 과정을 직관적으로 이해기위해 시각화 해본 내용을 정리해봤다.

## 지식 기반의 추론과 검증

추론, 검증을 보기 전에 `Knowledge Base`의 특징에 따라 결과의 해석이 어떻게 다르게 발생하는지 먼저 정리가 필요하다. 이 특징은 "외부에서 새로운 정보가 입력될 수 있는지"인데, 어떻게 다른지 먼저 정리할 필요가 있다.

### 열린계와 닫힌계

`Knowledge Base`가 외부의 정보에대해 열려있는 구조를 `Open-world`라고 하고, 외부의 정보가 차단되어있는 시스템을 `Closed-world`라고 한다. 이렇게 `Fact`, `Rule`이 계속 추가될 수 있는지 없는지에 따라 질의에 대한 `Closed-world Knowledge Base`에서는 `True`, `False`로 정의되고, 여기에 `Open-world Knowledge Base`에서는 `Unknown`이라는 결론이 추가된다.

이 `Unknown`이라는 미묘한 결과가 어떤 방식으로 나올 수 있는지 처음에 제시한 3단논법으로 구축한 `Knowledge Base`에 새로운 `Fact`와 `Rule`을 추가해서 사례를 만들어보았다.

> 1. Athena는 God이다.
> 2. God은 Immortal이다.
> 3. 그러므로 Athena는 Immortal이다.

이렇게 의미상 반대인 Mortal/Immortal로 `Knowledge Base`를 구축하고 나면,

> * Socrates는 Immortal이다.

이렇게 서로 반대인 의미로 해석될 수 있는 질의가 입력되면 결과가 당연히 `False`라고 생각할 수 있다. 하지만 `Open-world`로 정의된 `Knowledge Base`에서는 그 결과가 `Unknown`이 될 수 있다.

더 자세하게 파고들어 케이스를 정리해보면, 이 `Knowledge Base`안에서, _"Socrates는 Human이다"_ 라는 명제는 이미 `Knowledge Base` 안에서 `Fact`이기 때문에 이 질의에 대한 결론은 `True`로 명확하다. 하지만 _"Socrates는 Immortal이다"_ 는 `Fact`로 정의되어있지 않고, 추론을 통해서 얻을 수 없는 명제이다. 이러한 명제가 질의로 입력되면 `Closed-World`에서는 `False`로 결정할 수 있지만, `Open_World`에서는 `Unknown`으로 추론 결과를 해석하는 것이 가능하다.

### 지식 기반의 확장

지식 기반의 복잡도를 높여서 추론 문제를 보면 좋을것 같아 지식 기반을 확장해봤다.

``` mermaid
graph
    subgraph Universal Set
    direction TB
        subgraph Mortal
        direction TB
            subgraph Human
            direction TB
                e((Socrates))
            end
            subgraph Insect
            direction TB
                a((Fly))
                b((Ladybug))
            end
        end

        subgraph Immortal
        direction TB
            subgraph God
                f((Athena))
            end
            subgraph Atom
            direction TB
                t((Hydrogen))
                r((Neon))
            end
        end
    end
```

> 1. $Atom = \{Hydrogen, Neon\}$
> 2. $God = \{Athena\}$
> 3. $Insect = \{Fly, Ladybug\}$
> 4. $Human = \{Socrates\}$
> 5. $Immortal \supseteq Atom \cup God \text{ and } \emptyset \equiv Atom \cap God$
> 6. $Mortal \supseteq Insect \cup Human \text{ and } \emptyset \equiv Insect \cap Human$
> 7. $U \supseteq Immortal \cup Mortal \text{ and } \emptyset \equiv Immortal \cap Mortal$

이 수식에서 1에서 4까지 집합을 정의하고, 5에서 7까지는 집합간의 포함관계를 정의한 내용이다. 위에 정의되어있는 집합들에 대한 포함관계가 중요한데, Immortal과 Mortal, Atom과 God, Insect와 Human은 각각이 서로소인 관계라는 것이다. 물론 현실세계의 모델을 `Knowledge Base`에 `Fact`와 `Rule`로 구축한다면 이보다는 훨씬 복잡한 집합 구조가 필요할 것이다.

### 수학적 모델링이 중요한 이유

이렇게 `Knowledge Base`이 수학적으로 모델링될 수 있다는 것을 간단한 사례를 이용해서 전개해봤는데, 여기서는 수학적 모델링이 중요하다는 것을 정리해보려고 한다.

크게는 두가지 이유를 들 수 있는데, 첫번째로는 명확한 논리 구조와 수학적 모델링이 안되어있는 `Knowledge Base`의 크기가 커졌을 때, 부분적으로 개념이 충돌하거나 논리적으로 모순이 발생하는지 알기가 어렵다. 따라서 검증가능한 수학적 모델이 반드시 필요하다.

또한 새로운 `Fact`가 입력될 때 기존의 논리 구조에 대해서도 적합한지 검증할 때도 유용하게 활용할 수 있다. 위의 `Knowledge Base`에 새로운 `Fact`인 "Socrates는 Immortal이다"를 입력한다고 가정해보면, 수학적 모델을 배제한 상태로 논리적으로 모순인지 알기위해서는 일일이 `Fact`와 연결된 `Rule`을 대조하면서 찾아내야 한다. 하지만 이걸 집합으로 표현해보면 간단히 모순이 발생하고 있음을 알 수 있다.

$\text{If }Socrates \in Immortal,$

위의 명제를 참으로 가정하고 정의되어있는 `Knowledge Base`를 전개해보면 아래와 같이 표현된다.

> 1. $U \supseteq Mortal \cup Immortal \text{ and } \emptyset \equiv Mortal \cap Immortal$</br>
> 2. $Socrates \in Mortal$</br>
> 3. $\therefore Socrates \in Mortal \text{ and } Socrates \in Immortal$</br>

1번을 통해 `Mortal`과 `Immortal`의 교집합이 공집합임을 알 수 있고, 2를 통해서 Socrates는 Mortal에 속하는 요소임을 알 수 있다. 그런데 초기 가정인 $Socrates \in Immortal$이 참이기 때문에 결론에서 Socrates는 Mortal에 속하면서 동시에 Immortal에 속하게된다. 이 내용을 수식으로 표현한 것이 3번인데, 이 자체가 모순이기 때문에 이 가정 자체가 거짓이라는 것을 알 수 있다.

다시 말해, `Knowledge Base`는 논리적 모순이 없음을 증명하고, 새로운 `Fact`를 검증할 때 집합을 통해 정의한 모델을 활용하면 빠르고 명확하게 결과를 얻어낼 수 있다.

## 결론

이 글을 작성하기 전에, `Knowledge Base`를 집합으로 이해할 수 있지 않을까 하는 직관적인 접근이 먼저 있었다. 그리고 그걸 위해 간단한 집합을 공부하고 적용해보면서 수학적 모델이 얼마나 강력한지 확실히 느낄 수 있었다. 그래서 이후에 `Knowledge Base`를 구축하고 활용해야 할 때 참고하기 위한 목적으로 단순화해서 정리한 글이다.

다음에는 지금 이 글을 바탕으로 지식을 표현하는 또 다른 방법으로 `Graph` 이론을 도입했을 때 `Knowledge Base`가 얼마나 강력해지는지, 왜 `Knowledge Graph`가 만들어지게 되는지 알아보려고 한다.
