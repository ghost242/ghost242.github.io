---
layout: post
title: Inference Engine 1. Inference Engine란 무엇인가
subtitle: Inference Engine에 대해서
parent: Symbolic AI
grand_parent: AI
comments: true
categories: ["Programming", "Symbolic", "Inference Engine"]
tag: ["Inference Engine", "Symbolic System"]
---

## From Knowledge to Inference

이전 포스트인 [Prolog으로 살펴보는 지식 베이스의 구조](docs/ai/symbolic/2025-10-05-introduce-knowledge_base.md)에서는 사실(Facts)과 규칙(Rules)을 어떤 구조로 정의하고 관리해야 하는지에 대해 정리해봤다. 거기에 더해서 Prolog 언어를 이용해서 질의하고 결과를 얻는 문법구조까지 함께 정리했었는데, 이번 문서에서는 좀 더 Inference라는 영역에 초점을 맞춰보려고 한다.

이 포스트의 목표는 이렇게 사실과 규칙을 나열하고있는 Knowledge Base가 새로운 사실을 추론하는데 어떻게 활용될 수 있는지 조사하고 나름의 이해를 거친 내용들을 정리해보려고 한다. 다만 이번 문서에서는 좀 더 원론적인 내용으로 정리하고, Inference Engine의 구현체는 다음에 정리할 예정이다.

이 문서의 타이틀 구성은 ChatGPT를 통해 작성하였음.

### Separation of Knowledge and Inference

개인적으로는 Prolog 혹은 다른 Knowledge Base를 다루는 논리형 언어를 공부했을 때, 문법에서 기본적으로 제공하는 질의까지 한번에 다루기 때문에 추론이 무엇인지 정리되지 않고 자연스럽게 넘어가버리는 문제가 있었다. 그래서 추론이 대체 무엇인지, 지식과 다른점이 무엇인지 확실히 할 필요가 있다.

추론이란 한마디로 주어진 사실과 규칙을 통해서 질의로 입력된 명제의 참/거짓 여부, 그리고 Knowledge Base에 암묵적으로 포함되어있는 사실을 도출해내는 과정을 말한다. 논리학에서 흔히 제시하는 3단논법이 바로 추론의 전형적인 사례이다.

| 1. 사람은 모두 죽는다.(Human is mortal)
| 2. 소크라테스는 사람이다.(Socrates is human)
| 3. 소크라테스는 죽는다.(Socrates is mortal)

이 문장을 Knowledge Base로 정리한다면 이렇게 될 것이다.

| 1. ∀x (Human(x) → Mortal(x))
| 2. Human(Socrates)
| 3. Mortal(Socrates)

이러한 식으로 포함관계를 통해 새로운 사실을 추론해낼 수 있다. 위의 논리 전개의 각각은 Knowledge Base의 개별 사실과 규칙으로 이해할 수 있고, 1, 2를 통해 3을 도출하는 것이 추론이기 때문에 분리해서 이해할 필요가 있다.

## Inference in Symbolic AI

Knowledge Base와 Inference는 Symbolic AI 시스템을 구성하는 주요한 요소이기 때문에, 이 Inference가 Symbolic AI에서 어떤 위치에 있는지 알 필요가 있다.

``` mermaid
graph TB
    User --> QE[Query Engine]
    IE --> User 
    subgraph "Symbolic AI"
        QE --> IE[Inference Engine]
        IE --> KB[Knowledge Base]
        KB --> IE
    end 
```

단순화한 Symbolic AI의 도식에서 Inference는 이러한 위치에 있다.

위의 도식에서 Query Engine은 사용자의 질의를 해석하고 Inference Engine이 처리할 수 있도록 구조화해서 추론 목표를 구체적으로 제시하는 역할을 맡는다. 이런 역할은 사용자가 단순한 형태의 질의만 입력하지 않고 여러가지 의미로 해석될 수 있는 복합 질의를 입력할 수 있기 때문에 이 질의를 반드시 주어진 목표에 맞춰 처리해줄 필요가 있다. Query Engine에 대해서는 Inference Engine을 다룬 이후에 좀 더 자세히 다뤄보면 좋을 것 같다.

하지만 만약 사용자가 자연어로 질의를 하기 시작하면 아래와 같은 역할이 추가된다.

``` mermaid
graph TB
    User -- Query as Natural Language --> TRS[Translator]
    TRS -- Query to NL --> User
    subgraph "Symbolic AI"
        TRS -- NL to Query --> QE[Query Engine]
        IE --> TRS
        QE --> IE[Inference Engine]
        IE --> KB[Knowledge Base]
        KB --> IE
    end 
```

첫번째 그림과의 차이는 Translator의 추가인데, 이 Component는 사용자의 자연어 질의를 기호화된 Query로 변환해주고, Inference Engine으로부터 반환된 결과를 다시 자연어로 변환해서 사용자에게 전달하는 역할을 갖는다. 여기서 Translator는 자연어를 처리하는 기능이 필요한데, LLM으로 구성했을 때 Inference Engine과 Knowledge Base를 초월하는 새로운 사실을 임의로 생성하지 않도록 시스템 수준에서 제어할 필요가 있다.

### The Role of the Inference Engine

위의 구조를 살펴보면 Inference Engine이 어떤 역할을 하고있는지 쉽게 알 수 있다. 그리고 이 부분에서 Inference Engine에 필요한 기능이 무엇인지도 알아낼 수 있다.

사용자는 Query Engine을 통해 필요한 사실을 질의하고, Query Engine은 Inference Engine에 사용자의 질의를 전달한다. 그러면 Inference Engine은 Knowledge Base에서 관리되고있는 사실과 규칙들을 기반으로, 논리 규칙을 따르는 추론과정을 거쳐 질의로 입력된 명제에 대한 결론을 도출한다.

즉, Inference Engine은 간단한 3단 논법부터 복잡한 논리구조의 해석까지도 가능해야 함을 알 수 있다. 인간은 이미 있는 사실들을 연결지어서 새로운 사실을 얻어내기도 하지만, 때로는 어떤 사실과 규칙에는 없는 패턴을 다른 사실과 규칙들을 통해 찾아낼 수도 있다. 따라서 Inference Engine의 기능과 역할을 이해한다는 것은 인간이 논리적인 사고를 통해 어떻게 새로운 사실, 규칙들을 추론해내는지 찾아가는 과정으로 이해할 수도 있는 것이다.

### Inputs and Outputs of the Inference Engine

Inference Engine은 구조화된 `Query`가 가리키는 목표를 입력으로 받아 논리적 추론을 수행한 결과를 반환한다. 이 반환은 질의에 대한 추론 결과이기 때문에 `Result`라고 표현한다.

그러면 Inference Engine이 작동하기 위해서 필요한 입력과, 그 결과에 대한 출력 결과를 몇가지 사례로 이해해볼 수 있다. 만약 위의 3단 논법에서 1, 2가 이미 Knowledge Base에 주어져있다고 가정하고, 3이 사실인지 확인하려 한다고 가정하면 이렇게 볼 수 있다.

``` text
Query: Mortal(Socrates)
Result: True
```

Inference Engine은 위와 같은 논리적 추론을 거쳐 Mortal(Socrates)라는 결론을 얻어낼 수 있다. 그에 대한 반환은 사용자의 질의와 일치하는지 판단한 결과, 참/거짓을 출력한다.

또는 `Mortal`에 해당하는 객체를 얻고싶다면 이렇게 할 수도 있다.

``` text
Query: Mortal(x)
Result: {Socrates} 
# Mortal(x)를 만족하는 변수 바인딩의 집합
```

이 외에도 논리적 추론에서 필요한 연산은 Inference Engine 자체에 대해 학습할 때 좀 더 알아보려고 한다.

## Conclusion

최근 AI에대해 공부하던 중에 알게되었던 AI 구조 중에서 Symbolic AI를 좀 더 깊게 파면서 알게된 Inference Engine을 좀 정리해봤다. 그래서 Inference가 무엇인지, Knowledge Base와 어떻게 구분되어야 하는지, 그리고 Symbolic AI에서 Inference Engine의 역할에 대해 우선 정리해봤다.

이후에는 Inference Engine 자체에 대해 더 깊게 파고들어서 Symbolic AI 자체를 이해해보려고 한다.
