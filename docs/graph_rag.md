Graph와 Knowledge base에 대해서

개요

나는 지식을 머릿속에 막연히 나열하는것 보다는 대표 키워드를 중심으로 구조화 하고 키워드간의 관계를 구축하는 것이 기억에도 유리하고, 관련 지식을 다양하게 꺼내는데도 도움이 된다는 것을 감각적으로 느끼고 있었다. 하지만 LLM은 어떻게 하고있고, Graph로 구조화된 지식을 어떻게 다루게 하는지에 대해서는 최근에 알게 되었다. 이 문서는 그렇게 최근에 알게된, Knowledge base를 Graph로 구조화 하는것과 LLM과의 통합에 대해 간략하게 정리하는 것을 목표로 한다.

Knowledge base

대표 키워드로 정의되는 지식과, 지식들 간의 관계를 정의해야 한다는 것이 어떤 의미인지는 최근까지 이해하지 못하고 있었다. 알았다면 <프로그래밍 언어> 강의를 들을 때  `Prolog` 언어를 이해하는데 도움이 되었을 것이다. 당장 떠오르는 사례가 Prolog 이기 때문에, 간단하게 Prolog 문법으로 표현된 database를 이용해서 정리해보려고 한다.

Prolog

Logical Programming 이라는 언어 패러다임을 갖고있는 Prolog는 Procedure Language를 패러다임으로 갖고있는 언어들과는 구조가 너무 달라서 이해하기 어려웠으나, 코드가 순차실행되지 않는다는 사실만 받아들이면 된다. 이 특징은 현대에 연구되고있는 Functional language 패러다임 언어에서도 나타나는 특징이기 때문에 지금은 비교적 쉽게 샘플 코드를 얻을 수 있다.



## Graph에 대해서

Knowledge Graph에 대해 이해하기 위해서는 Graph에 대해 이해해야 할 필요가 있다. 수학적인 관점에서 Graph는 정점과 간선 두개의 집합 쌍으로 정의된다.

$$
\begin{array}{ll}
    &G = (V, E) \\
    &V = \{v_1, v_2, v_3, ...\} \\
    &E = \{(x, y) | x, y \in V\}
\end{array}
$$

컴퓨터 공학에서는 Node라고 부르는 대상이 수학의 Graph에서는 Vertex라고 불린다. 이 두가지는 Graph를 바라보는 관점에 따라 미묘한 차이가 있지만, 거의 동일하게 다루어도 크게 문제가 되지 않는다.

더 깊게 파고들기 전에 용어들을 간단하게 정리하는 것이 좋을 것 같아 아래와같은 의미로 사용하려고 한다.

* **Node:** 데이터 구조 관점에 Graph의 정점을 가리키는 용어이다. 특히 Knowledge Graph에서는 Knowledge를 다루는 단위로 볼 수 있을것 같다.
* **Edge:** 두 Node 사이의 연결을 가리킨다. Edge의 성격에 따라 Graph가 두종류로 나뉘는데, V_a, V_b에 대해 E = (V_a, V_b), E' = (V_b, V_a)가 있을 때, E ≡ E'인 Graph를 Indirect Graph라고 하고, E ≢ E' 인 Graph를 Direct Graph라고 한다.
* **Neighbor:** Node와 하나의 Edge로 연결되어있는 Node들을 말한다. `a -- b -- c`에서 a, c는 b의 Neighbor이다.
* **Degree:** 어떤 Node를 시점/종점으로 갖고있는 Edge의 수를 가리킨다. `a -- b -- c` 이런 구조에서 Degree(a) = 1, Degree(b) = 2, Degree(c) = 1이다. Graph의 종류에 따라 Indirect Graph인 경우에는 Node에 대한 Degree는 하나의 값이지만, Direct Graph는 Degree_in, Degree_out 두가지로 구분해서 정의한다.
* **Distance:** 어떤 Node에서 다른 Node까지 그 사이에 있는 Edge의 집합을 중에서, 크기가 최소인 집합의 크기를 가리킨다.
* **Path:** 어떤 Node에서 다른 Node를 연결하는 Edge의 집합이다. 단일 Edge가 아닌 이유는 대상이 되는 두 Node 사이에 다른 Node가 포함되어있을 수 있기 때문이다. 예를들어 `a -- b -- c`와 같은 Graph가 있을 때 a에서 c까지 Path를 P={(a,b), (b,c)} 와 같이 표현할 수 있다.

이런 기본적인 요소들을 활용해서 수학적으로는 다뤄야 하는 대상 자체로, 그리고 컴퓨터공학에서는 데이터의 구조로써 다루게 된다.