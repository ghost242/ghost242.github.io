---
layout: post
title: Knowledge Base 3. Knowledge Base의 Graph 표현
subtitle: 그래프 이론이 적용된 Knowledge Base에 대해서
parent: Symbolic AI
grand_parent: AI
comments: true
categories: ["Programming", "Symbolic", "Knowledge Base"]
tag: ["Knowledge Base", "Set Theory", "Logic"]
---

## 도입

이 전에 Knoweledge Base을 수학적 모델을 통해 다루기위해 Set을 도입하여 추상화된 형태로만 다뤄봤는데, 현실에서의 Knowledge는 적어도 이보다는 더 복잡하다. Knowledge Base를 Graph로 표현하고 다루기 위해서 Knowledge에 대해 좀 더 파고들 필요가 있다는 생각을 하게 되었다.

Graph는 정보를 다루는 구조가 크게 두가지로 구분될 수 있다고 생각한다. 하나는 Node이고, 또 하나는 Edge이다. Node는 그 자체로 정보의 단위가 될 수 있고, Edge는 시점과 종점이 되는 Node 사이의 연결 정보를 표현하는 것이다. 따라서 Knowledge를 Graph로 표현할 때 Node와 Edge를 잘 구분하지 않으면 시스템 내에서 정보의 모순이 발생하거나 구조적으로 불안정해질 수 있을 것이라고 생각된다.

사실은 Knowledge Graph에 대해 post를 작성하던 도중에 아무리 고쳐써봐도 내용이 어딘가 붕 뜨는 것 같아서 Knowledge 자체를 먼저 찾아보고, 그에 대해 고민했던 내용을 기록하기 위해 post를 작성한다.

## Knowledge Graph에 대해서

왜 Knowledge Graph가 필요하게 되는지, 어떻게 Knowledge를 Graph로 표현할 수 있을지에 대해서 생각해볼 필요가 있다. Fact와 Rule로 구성된 Knowledge Base도 Knowledge를 표현하는데는 큰 문제가 없었다. 다만 소규모에 특정 도메인을 위한 Knowledge로 구성할 때는 의미가 있었지만 규모가 커지고 다양한 도메인을 수용하게 되면서 Fact와 Rule 외에 더 다양한 관계와 다양한 형태를 다룰 필요가 생기면서 Knowledge Base 만으로는 부족하게 되었다.

### Knowledge에 대해서

Knowledge에 대한 보다 근본적인 이해가 필요하다. 그리스 철학에서 외부에 본질이 있고 우리는 그저 촛불에 비친 그림자로 대상을 이해할 뿐이라는 이데아론이 있었는데, 다른건 몰라도 언어에는 확실히 이런 속성이 있다. 같은 대상이라도 언어마다 다르게 부르는것은 물론이고 같은 의미로 해석하지만 미묘한 차이를 갖고있는 경우도 있고 대상이 다른데 단어가 같은 경우도 있다. 같은 언어 내에도 이름이 완전히 같은데 시대에 따라 가리키는 대상이 다른 경우도 있다.

위키피디아에 의하면, 인식론에서 Knowledge는 "Knowledge is an awareness, familiarity, understanding, or skill." 라고 하고 있다. 즉, 명백한 것과 익숙한 것, 이해하고 있는 것, 경험에서 체득한 것이라고 할 수 있을 것 같다. 이 모든 것들은 대상에 접촉하는 경험에 따라 같은 대상인데도 다르게 받아들여지는 경우가 있기 때문에 본질을 중심으로 통합되어야 한다.

이런 관점에서 다양한 도메인에 걸쳐서 대규모로 Knowledge를 그것을 대표하는 심볼을 중심으로 구조화 하기 위해 기존의 Knowledge Base만으로는 한계가 있었다.

### 구조화된 Knowledge

사람은 자신이 받아들인 Knowledge를 문자 그대로 기억하지 않는다. 항상 자신의 경험과 직접적이든 간접적으로 연결되어있는 다른 Knowledge와 관계를 맺는다. 일반적으로 이것을 "연상기억법"이라고 부르기도 하지만, 사람은 보통 시간 흐름이나 맥락, 과거의 기억들과 끊임없이 관계를 맺으면서 기억한다.

"사과"라는 사물을 중심으로 적당히 떠오르는 것들을 연결해보면 이런 식이다.

``` mermaid
---
config:
    flowchart:
        defaultRenderer: "elk"
---
graph
    apple --looks--> red
    apple --looks--> green
    apple --shaped--> sphere
    apple --taste--> sour
    apple --taste--> sweet
    apple --in--> fruit
    apple --grows on--> at[apple tree]

    sour --feels on--> tongue
    sour --comes--> saliva
    sweet --feels on--> tongue
    
    red --feels on--> eye
    strawberry --looks--> red
    strawberry --taste--> sour
    strawberry --taste--> sweet
    strawberry --in--> fruit

    pear --in--> fruit
    peach --in--> fruit

    sour --in--> flavor
    sweet --in--> flavor

    at --has--> steem
    at --has--> leaf
    at --has--> branch
    at --has--> flower
    at --has--> fruit

    fruit --grows on--> at
    fruit --has--> seed

    at --in--> fp[flowering plant]
    fp --in--> plant
```

위의 Graph에는 신맛, 또는 단맛에서 사과를 떠올릴 수 있다는것이 구조화 되어있다. 그리고 신맛은 혀에서 느껴지고, 신맛을 느끼면 침이 나오는걸 볼 수 있다. 그리고 신맛과 단맛에서 딸기를 떠올리기도 한다. 사과라는 과일을 보면 배나 복숭아도 함께 떠올린다.

이런 Graph를 아주 성실하게 만들다보면 굉장히 복합적이고 추상적인 개념도 표현할 수 있게 된다. 또한 동사, 조사를 각 Entity의 Relationship으로 정의하고 있다. 이것은 여러가지 표현방법중 하나라서 어떤 기준으로 표현하냐에 따라 다를 수 있다. 어떤 경우는 Relationship의 이름을 따로 지정하지 않고 모두 Node로써 표현하기도 하기 때문이다.

만약 "사과가 식물인가요?" 라는 질의를 제시한다고 가정하면, 위의 Graph에서 사과와 식물을 찾을 것이다. 그리고 사과를 시점으로 Graph를 탐색한다. 그리고 탐색 중에 식물을 찾으면 서로 관계가 있음을 알 수 있다. 그러면 이 Graph에서 이런 SubGraph를 얻을 수 있다.

``` mermaid
graph
    apple --grows on--> at[apple tree]
    at --in--> fp[flowering plant]
    fp --in--> plant
```

이것으로 사과와 식물 사이에 관계가 있음을 알 수 있고, Relationship을 통해 사과가 사과나무에서 자라나고, 사과나무는 속씨식물에 포함되고, 속씨식물이 식물에 포함되는 것을 통해서 사과가 열리는 사과나무는 식물이라는 것을 알 수 있다.

"사과의 다리는 몇개인가요?" 라는 질의를 가정해볼 수도 있다. 이 경우에 위의 질의와 마찬가지로 사과를 시작으로 Graph를 탐색한다. 하지만 Graph 전체를 탐색한 결과 사과와 다리의 관계는 위의 Graph에서 찾을 수 없다. 따라서 이 질의에는 적합한 결과가 없기 때문에 탐색 실패를 답으로 내놓을 수 있다. 이 결과는 경우에 따라 다르게 해석될 수 있으며, 지금의 경우는 "사과에는 다리가 없다"고 해석할 수 있다.

이런 구조 자체만 두고 보면 Knowledge Graph와 Knowledge Base의 차이를 알기 어렵다. 위의 Graph에 대응하는 Set을 구축하는 것이 가능하기 때문이다.

### Ontology

인식론 관점에서 Knowledge는 하나의 대상에 대한 다양한 해석이라고 할 수 있다. 해석을 하는 주체는 관찰자이기 때문에 사람마다 같은 대상을 보고도 다르게 이해하는 것이다. Knowledge Graph는 이것을 총체적으로 다루는 방법을 포함하고있다.

유명한 관용구 중에서 "만약 어떤 것이 오리처럼 생겼고, 오리처럼 헤엄치고, 오리처럼 꽥꽥거린다면 그건 아마도 오리일 것입니다” 라는 말이 있다. 이 구문을 Knowledge Graph로 만들어보면 이렇게 될 것이다.

``` mermaid
graph
    duck --swim on--> water
    duck --fly in--> sky
    duck --sounds--> quack
```

어떤 대상이 Knowledge Graph로 표현되어있을 때 이 Graph를 포함하고 있다면, 이 대상은 오리라고 부른다고 할 수 있다는 것이다.

단순히 몇가지 조건을 만족하는 경우와는 다른 점이 있다. 바로 "duck"을 중심으로 "water", "sky", "quack" 세개의 Knowledge가 각각 "swim on", "fly in", "sounds" 라는 Relationship으로 연결되어있는 구조 자체가 중요하다는 것이다. 만약 수많은 생물의 특성을 모두 저장하고있는 Knowledge Graph가 있다면, 위의 Graph와 같은 Graph 구조를 포함하고있는 생물을 분류하면, 이 생물은 "오리"가 된다.

관점에 대해 유명한 또 다른 사례로, 눈을 가린 사람 세명이 오직 손으로 더듬으면서 코끼리를 파악하는 테스트를 하는 상황을 상상해볼 수 있다. 사전 정보로는 "코끼리"라는 이름만 주어지고 서로 다른 부위를 더듬은 결과 이런 묘사를 내놓을 수 있다.

---

1. 코를 만진 사람

* 묘사

  > "코끼리는 거친 표면에비해 말랑말랑한 내 키만큼 기다란 기관을 가진 동물이다. 이 긴 기관의 한쪽 끝에는 두개의 구멍이 나란히 뚫려있다. 안에 뼈가 있는것 같진 않고 주름이 엄청 많은데 꿈틀거리거나 돌돌 말릴 수 있을 정도로 유연하다. 반대쪽 끝은 다른 기관과 연결되어있는데 이 부위는 엄청 크다."

* Knowledge Graph

---

2. 다리를 만진 사람

* 묘사

  > "코끼리는 피부가 거칠고 아주 두꺼운 기둥같은 동물이다. 바닥에 닿아있는 부위까지 뭉툭하고 끝에 발톱같은 것이 느껴지긴 하는데 발가락이 만져지지 않는것을 보면 다리인지는 확실하지 않지만 몸의 어떤 부위인것은 분명하다."

* Knowledge Graph

---

3. 머리를 만진 사람

* 묘사

  > "코끼리의 표면은 딱딱하고 꼭대기의 둥그스름한 부위에 거친 털같은 것이 만져진다. 그에 비해 양 옆에는 얇은 피막이 만져지는데 상단에 연골이 있어서 아무렇게나 축 쳐져있지 않고, 코끼리가 스스로 펄럭거릴 수 있는것으로 보아 날개같은 기관을 갖고있다. 앞부분에는 말랑말랑한 부위가 만져지는데, 끝이 어딘지 알 수 없을 정도로 긴 것같다."

* Knowledge Graph

---

최대한 부분적인 관점에서 미묘한 오해를 포함한 묘사를 가정해봤다. 실제 코끼리는 커다란 동물이고 이렇게 더듬어서 전체를 파악하기 어려운 점을 통해 다양한 관점을 수용하고 자신의 생각만 옳다는 생각을 버리자는 교훈을 제시하는 짧은 글을 봤던 기억이 떠올라서 사례로 추가했다.

각각의 사례에서 Knowledge Graph는 모두 코끼리를 중심으로 서로 다른 부분들을 각각 독립된 Node로 만들 수 있다. 중요한 것은, 이 세개의 Knowledge Graph의 중심이 모두 같은 것을 가리키는 것을 알고 있기 때문에 하나의 Graph로 병합할 수 있다.

* Knowledge Graph

코끼리 자체를 Knowledge Graph로 만든다면, 이 Graph는 적어도 위의 Knowledge Graph를 부분으로 포함하고 있을 것이다. 만약 어떤 사람이 포유동물의 각 부위에 대한 설명을 정리한 내용이 있다고 생각해볼 수 있다.

1. 머리: 동물의 앞부분. 단단한 두개골이 감싸고있는 뇌가 있고, 빛을 매개로 외부 정보를 받아들이는 눈이 두개, 호흡과 공기에 포함된 냄새를 느낄 수 있는 코, 외부 물질을 섭취하기 위한 입이 하나 있다.

  1. 눈: 눈은 얇은 피막으로 보호되고있고, 동물들은 앞을 볼 때는 이 피막을 자유롭게 걷었다가 눈을 보호해야 할 때는 피막으로 눈을 덮을 수 있다. 

  2. 코: 코는 안쪽에 단단한 뼈가 없고, 외부 공기를 받아들이기 위한 구멍이 두개 뚫려있다.

  3. 입: 은 코 바로 아래에 있는 큰 구멍인데, 구멍의 가장자리를 유연한 근육으로 이루어진 입술이 감싸고 있다.

  4. 귀, 