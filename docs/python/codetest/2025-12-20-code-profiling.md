---
layout: post
title: 코드의 성능을 측정하는 방법
subtitle: Profiler 사용방법에 대한 소개
parent: Tests
grand_parent: Python
comments: true
categories: ["Programming", "Python3", "Testing"]
tag: ["Test", "Python", "Profiler"]
---

## 개요

현대에는 워낙 컴퓨팅 자원이 풍부하고 프로세서의 성능이 좋아서 코드가 아주 빠르게 작동하기 때문에 과거처럼 효율 중심의 프로그래밍 언어를 선택하지 않게 되었다. 그보다는 얼마나 더 빨리 작성할 수 있는지, 그리고 얼마나 쉽게 유지보수를 할 수 있는지를 더 중요하게 생각하게 되었는데, 그 결과 우리는 여전히 특정 상황에서 아주 느려지는 프로그램을 계속해서 만들게 되었다.

Python3은 3.9 버전까지는 bytecode의 실행 속도가 느리고 최적화가 잘 안되어서 코드 실행이 상당히 느린 편이었다. 3.10부터 Interpreter를 대대적으로 개선하면서 실행속도가 40%이상 빨라졌다고 자랑할 정도였다. 그에 비해서 계산량이 엄청나게 많은 AI를 통합하기 위해 자연어 처리가 용이하면서 여러 계층을 포괄할 수 있고 문법이 쉬운 언어로 Python3이 각광을 받으면서 성능 최적화는 당연히 해야 하는 작업인데도 생산성의 문제로 느린 코드가 방치되기 십상이었다.

어느날에 회사에서 진행중인 웹서비스 프로젝트 중에서 특정 API의 응답이 용납할 수 없을 정도로 느려지는 모습을 본 뒤로 성능 측정이 왜 필요한지 느끼게 되었다. 그래서 거기에 대한 정보를 좀 정리해보려고 한다.

## 패키지에 대한 소개

Python3 공식 홈페이지의 문서 중에 Profile에 대한 문서가 있다(참고: [The Python Profilers](https://docs.python.org/3.13/library/profile.html)). 여기에는 두개의 패키지가 소개되어있는데, 하나는 `cProfile`, 그리고 `pstats`이다. 여기에 추가로 코드 실행 흐름을 추적하기 위한 호출 스택을 스냅샷으로 추출하기위해 `tracemalloc` 패키지를 추가로 나열해봤다.

### cProfile

사실 `cProfile`과 `profile` 두가지 패키지가 함께 소개되고있는데, 하나의 독립된 모듈을 외부에서 실행하지 않고 특정 영역만 선별해서 실행하려고 하면 `cProfile`을 사용해야 한다.

공식 문서에서는 여러가지 소개하고 있지만 몇가지 코드 패턴만 알면 된다.

``` python
import cProfile


# without context manager
profiler = cProfile.Profile()
profiler.enable()
...  # Call to function for measuring performance
profiler.disable()

# with context manager
with cProfile.Profile() as profiler:
    ...  # Call to function for measuring performance
```

이후에 측정 결과에 대한 통계는 `pstats` 패키지를 통해 출력하게 하면 된다. (3.13 버전부터 `cProfile.Profile` 객체에 `.print_stats()` method가 추가되었다.)

### pstats

이 패키지는 프로파일링 결과를 통계로 추출하기 위한 각종 기능이 들어있는 패키지이다. 이걸 이용해서 다양한 펙터로 통계를 볼 수 있기 때문에 잘 활용하는게 중요하다.

아래 코드는 가장 단순하게 기본 설정으로 성능 측정 결과를 출력하는 코드 패턴이다. 이 코드는 각 모듈의 실행시간을 보여주는데, 결과는 호출 스택의 순서대로 출력된다. 또한 결과는 표준 출력 버퍼를 통해 성능을 출력하기 때문에 원격 서버를 테스트할 때는 출력 스트림을 file buffer 등으로 재정의 하는 것이 좋다.

``` python
import pstats

... # already defined instance of cProfile.Profile() as profiler

st = pstats.Stats(profiler)

st.print_stats()
```



### tracemalloc

## 프로파일링 예

## 결론

## 참고