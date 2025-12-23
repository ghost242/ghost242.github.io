---
layout: post
title: 코드의 성능을 측정하는 방법 - 시간에 대해서
subtitle: Profiler 사용방법에 대한 소개
parent: Tests
grand_parent: Python
comments: true
categories: ["Programming", "Python3", "Profiling"]
tag: ["Test", "Python", "cProfile"]
---

## Abstract

현대에는 워낙 컴퓨팅 자원이 풍부하고 프로세서의 성능이 좋아서 코드가 아주 빠르게 작동하기 때문에 과거처럼 효율 중심의 프로그래밍 언어를 선택하지 않게 되었다. 그보다는 얼마나 더 빨리 작성할 수 있는지, 그리고 얼마나 쉽게 유지보수를 할 수 있는지를 더 중요하게 생각하게 되었는데, 그 결과 우리는 여전히 특정 상황에서 아주 느려지는 프로그램을 계속해서 만들게 되었다.

Python3은 3.10 버전까지 꾸준히 최적화를 진행해왔지만 전통적인 bytecode 구조로 설계된 인터프리터의 한계로인해 코드 실행이 상당히 느린 편이었다. 3.11부터 Interpreter를 대대적으로 개선하면서 실행속도가 평균 40% 내외만큼 빨라졌다고 자랑할 정도였다. 그에 비해서 계산량이 엄청나게 많은 AI를 통합하기 위해 자연어 처리가 용이하면서 여러 계층을 포괄할 수 있고 문법이 쉬운 언어로 Python3이 각광을 받으면서 성능 최적화는 당연히 해야 하는 작업인데도 생산성의 문제로 느린 코드가 방치되기 십상이었다.

어느날에 회사에서 진행중인 웹서비스 프로젝트 중에서 특정 API의 응답이 용납할 수 없을 정도로 느려지는 모습을 본 뒤로 성능 측정이 왜 필요한지 느끼게 되었다. 그래서 거기에 대한 정보를 좀 정리해보려고 한다.

## Profiling tool - cProfile

Python3 공식 홈페이지의 문서 중에 Profile에 대한 문서가 있다([The Python Profilers][1]). 여기에는 세개의 패키지가 소개되어있는데, 프로파일링을 위한 `cProfile`와 `profile`, 그리고 프로파일링 통계를 사용자의 의도에 맞게 정리해서 출력하는 헬퍼 모듈인 `pstats`이다.

공식 문서에 따르면 `cProfile`과 `profile`은 둘 다 프로파일링 도구이지만, 모듈 내에서 특정 구간의 프로파일링을 수행해야 한다면 `cProfile`을 사용해야 한다고 설명하고 있다. `profile` 패키지는 외부 파일, 혹은 인라인 코드를 문자열로 입력받아 프로파일링을 수행하는 `.run` 등의 메소드만을 제한적으로 제공하고있기 때문이다.

공식 문서에서는 여러 기능을 자세하게 설명하고있지만, 이 포스트에서는 직접 사용해보고 쏠쏠하게 써먹었던 코드 패턴만 소개해보려고 한다.

``` python
import cProfile

# without context manager
profiler = cProfile.Profile()
profiler.enable()
...  # Call to function for measuring performance
profiler.disable()
```

여기 적지 않았지만 `contextmanager` 를 통해 프로파일링 구간을 관리하는 기능이 Python 3.8 버전부터 지원되기 시작했다고 한다.

이후에 측정 결과에 대한 통계는 `pstats` 패키지를 통해 출력하게 하면 된다. (3.13 버전부터 `cProfile.Profile` 객체에 있는 `.print_stats()` method에 둘 이상의 정렬키를 입력할 수 있도록 업데이트 되었다.)

## Statitics tool - pstats

이 패키지는 프로파일링 결과를 통계로 추출하기 위한 각종 기능이 들어있는 패키지이다. 이걸 이용해서 다양한 펙터로 통계를 볼 수 있기 때문에 잘 활용하는게 중요하다.

아래 코드는 가장 단순하게 기본 설정으로 성능 측정 결과를 출력하는 코드 패턴이다. 이 코드는 각 모듈의 실행시간을 보여주는데, 정렬 옵션을 따로 설정하지 않으면 랜덤한 순서로 출력된다고 명시되어있다. 필요한 조건에 맞춰 정렬하는 방법은 별도로 정리한다([Sorting & Filtering](#sorting--filtering)). 또한 결과는 표준 출력 버퍼를 통해 출력하기 때문에 원격 서버를 테스트할 때는 출력 스트림을 file buffer 등으로 재정의 하는 것이 좋다.

``` python
import pstats

... # already defined instance of cProfile.Profile() as profiler

st = pstats.Stats(profiler)

st.print_stats()
```

### For Recursive Function

`cProfile`을 이용해서 재귀함수를 분석하면 어떻게 나오는지 보기위해 콜라츠 추측([Collatz Conjecture][2])의 구현체를 재귀함수 형태로 정의했다. 콜라츠 추측은 아래 함수로 이루어져있다.

$$
f(x) = \begin{cases}
        \frac x 2 &\text{if } x \in even \\
        3x + 1 &\text{if } x \in odd
       \end{cases}\\
$$

이 함수는 초기 값, n이 1이 될 때까지 반복해서 적용하도록 되어있다.

단일 함수에서 연산량이 많지않아 함수 로직 내에서 추가로 시간이 오래걸리는 지점을 시뮬레이션하기 위해 `time.sleep`을 추가해봤다.

``` python
def collatz_conjecture(n, it=0):
    if n == 1:
        return it
    
    if n % 2 == 0:
        n = n // 2
    else:
        n = 3 * n + 1
    
    time.sleep(0.1)
    return collatz_conjecture(n, it + 1)
```

이 함수를 총 세번 호출한 후 결과를 출력하면 이렇다.

``` python
# 실행 코드
profiler = cProfile.Profile()
profiler.enable()
collatz_conjecture(123456789)
collatz_conjecture(123456789)
collatz_conjecture(123456789)
profiler.disable()

st = pstats.Stats(profiler)
st.print_stats()
```

``` shell
# 프로파일링 결과
         1066 function calls (535 primitive calls) in 53.181 seconds

   Random listing order was used

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
      531   53.175    0.100   53.175    0.100 {built-in method time.sleep}
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
    534/3    0.006    0.000   53.181   17.727 /.../profiling.py:97(collatz_conjecture)
```

이걸 통해 collatz_conjecture 함수를 외부에서(재귀 호출은 제외) 3번 호출되고, 총 534회 재귀호출되었다는 의미이다. 그리고 `collatz_conjecture` 함수 내에서 `time.sleep` 함수가 531번 호출되었다. `collatz_conjecture` 함수와 횟수에서 차이가 나는 이유는 종료조건인 `n == 1`에서는 `time.sleep` 함수가 호출되지 않기 때문에 1회 차이가 발생했다.

또 하나, `collatz_conjecture`에서 `tottime`과 `cumtime`의 값이 다른 것은 `cumtime`은 호출한 외부 시점을 중심으로 측정한 시간이고, `tottime`은 대상 함수 자체의 실행시간(함수 내부에서 다른 함수를 호출한 시간을 제외함)을 표기하기 때문이다. 이 함수는 외부의 다른 함수(`time.sleep`)을 호출하고 있기 때문에 외부에서 측정한 시간인 53.181s 에서 `time.sleep` 함수의 `cumtime`인 53.175s 를 뺀 0.006이 `tottime`으로 표기된다.

만약 이 결과를 통해 최적화를 한다면 두가지 최적화 관점이 있을 수 있다. 하나는 0.1s를 소모하는 로직을 최적화해서 실행 시간 관점에서 이득을 보는 방법이 있고, 다른 하나는 531번이나 반복 호출하는 영역을 캐싱해서 반복 횟수를 줄이면 `cumtime`에서 이득을 볼 수 있다.

### For Heavy Function

이번엔 단일 함수의 실행시간 자체가 오래걸리는 경우에 어떻게 나오는지 보기위해 아래와 같은 함수를 작성해봤다.

``` python
def leibniz_formula_pi(it):
    pi_quarter = 0
    for k in range(it):
        pi_quarter += ((-1) ** k) / (2 * k + 1)

    return 4 * pi_quarter
```

이 함수의 원본은 라이프니츠가 고안했던 방법([Leibniz formula for π][3])으로, arctan의 정의에서 테일러 급수를 유도한 것이다. 하지만 수학적으로 pi의 값에 도달하는 시간이 오래걸린다는 이유로 지금은 다른 더 훌륭한 수식으로 최적화되었다고 한다.

이 함수는 아래 함수식을 코드화한 것이다.

$$
{\frac \pi 4} = \sum_{k=0}^{\infty} {\frac {(-1)^{k}} {2k + 1}}
$$

다만 실제 수식과 다른점은 실제 수식이 무한 급수로 되어있는데 반해 컴퓨터는 유한 시간동안만 계산할 수 있기 때문에 크기에 제약을 두기위해 `it`를 임의로식 지정해서 작성한다.

``` python
# 실행 코드
profiler = cProfile.Profile()
profiler.enable()
leibniz_formula_pi(1_000_000)
leibniz_formula_pi(1_000_000)
profiler.disable()

st = pstats.Stats(profiler)
st.print_stats()
```

``` shell
# 프로파일링 결과
         3 function calls (3 primitive calls) in 0.337 seconds

   Random listing order was used

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
        2    0.337    0.169    0.337    0.169 /.../profiling.py:105(leibniz_formula_pi)
```

위 결과에서는 `leibniz_formula_pi` 라는 함수가 2번 호출되었고, 한번 호출되었을 때 0.169s, 그리고 전체 실행시간이 0.337s 임을 알 수 있다. 의도했던대로 이 함수의 실행시간이 오래걸린다는게 통계에서 확인되었고, 이 통계를 통해 어떻게 최적화해야 할지 계획을 만들 수 있다.

만약에 이 함수를 최적화하려면 어떻게 할 수 있을까, 이 함수는 수학적 정의를 토대로 만들어져있기 때문에 더 빠른 수식으로 교체하는 방법이 있을 수 있다. 또는, 특정 입력에 대한 결과 값이 빈번하게 참조된다면 전체 실행시간동안 한번만 호출하고 결과를 캐싱하도록 해서 같은 계산을 불필요하게 많이 하지 않도록 개선할 수 있을 것이다.

### Sorting & Filtering

현실적으로 통계가 이렇게 단순할 리가 없다. 그래서 `pstats`에는 정렬과 필터를 같이 제공한다. `pstats.Stats` 객체에는 `.sort_stats(<param>)` 메소드가 정의되어있다. 여기 `<param>`에 원하는 정렬 키를 입력하는 것이다.

이 기준은 profile 문서에서 Stats 객체에 대한 설명 중, sort_stats 메소드에 표료 정리되어있다.([The Python3 Profilers](https://docs.python.org/3.13/library/profile.html#the-stats-class)) 주로 쓸만한 정렬 키로는 호출 횟수(`CALLS`), 실행 측정 시간(`CUMULATIVE`), 이름(`NAME`) 등이 있을 수 있다.

이번엔 쓸데없이 복잡한 함수를 몇개 만들어서 정렬을 해보려고 한다.

``` python
import time

def d(n):
    time.sleep(0.4)
    return [c(n // 2) for _ in range(4)]

def c(n):
    time.sleep(0.3)
    return [b(n - 1) for _ in range(3)]

def b(n):
    time.sleep(0.2)
    return [a(n * 3) for _ in range(2)]

def a(n):
    time.sleep(0.1)
    return n + 100
```

`d` 함수를 호출하면 `d -> c -> b -> a` 순서로 함수가 호출되면서 적당한 연산과 함께 `time.sleep`을 호출해서 기다린다. 이 함수를 실행하고 결과를 보면 이렇게 나온다.

``` shell
         83 function calls in 6.411 seconds

   Random listing order was used

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
       41    6.410    0.156    6.410    0.156 {built-in method time.sleep}
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
       24    0.000    0.000    2.406    0.100 /../profiling.py:139(a)
        1    0.000    0.000    6.411    6.411 /../profiling.py:127(d)
        4    0.000    0.000    6.011    1.503 /../profiling.py:131(c)
       12    0.000    0.000    4.809    0.401 /../profiling.py:135(b)
```

이 결과를 통해 이 코드는 총 83번의 함수 호출이 있고, 그 중에 `time.sleep`이 41번 호출된다. 중요한건 `tottime`, `cumtime`은 실행시간의 합이라서 6.410s 이지만 `percall`은 호출횟수에 대한 평균이기 때문에 0.156s 로 표기된다는 점이다. 같은 함수라도 호출 상태에따라 실행시간이 다른데, `percall`은 전체 실행에 대한 평균이기 때문에 특정 상황이 반영되지 않는다.

또 하나는 이 결과는 정렬되어있지 않다는 것이다. 임의 순서로 나열되어있는데, 만약 이 통계가 100줄 이상이라면 보기 정말 힘들것이다. 여기서는 함수 `a`, `b`를 같은 폴더 내의 외부 모듈로 옮겨서 더 확실히 구분할 수 있다.

최종적으로 `profiling.py`, `ext_module.py` 두개 모듈만 출력되면서 파일 이름, 측정 시간을 정렬키로 사용하면 이렇게 할 수있다.

``` python
stats = (
    pstats
    .Stats(profiler, stream=s)
    .strip_dirs()  # filename에서 directory path 제거
    .sort_stats(SortKey.FILENAME, SortKey.CUMULATIVE)  # filename 으로 우선 정렬 후 cumulative time으로 정렬
)
stats.print_stats(".py")  # filename에 .py가 있는 측정 결과만 출력
```

이 코드에 대한 결과는 아래와 같다.

``` shell
         83 function calls in 6.404 seconds

   Ordered by: file name, cumulative time
   List reduced from 6 to 4 due to restriction <'.py'>

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
       12    0.000    0.000    4.804    0.400 ext_module.py:4(b)
       24    0.000    0.000    2.402    0.100 ext_module.py:9(a)
        1    0.000    0.000    6.404    6.404 profiling.py:129(d)
        4    0.000    0.000    6.004    1.501 profiling.py:133(c)
```

정렬 키에는 호출 횟수 기준으로 정렬하는 `SortKey.PCALLS`도 있기 때문에 만약 최적화 방향이 함수의 호출 횟수를 줄이는 방향이라면 이걸 이용해볼 수 있다(재귀 호출을 제외한 외부 호출을 기준으로 정렬). 자세한 정렬키는 "[The Python Profilers][1]" 문서에서 확인할 수 있다.

## 결론

이 모듈은 Python3의 표준 라이브러리에 포함되어있어서 내용도 나름 잘 정리되어있고 Python3 프로젝트를 통해 관리되고있어서 프로파일링 도구 자체로는 편리한 편이라고 생각한다. 이 라이브러리의 약점중에 하나라면 아마도 C언어로 작성된 모듈 자체의 호출 시간은 측정할 수 있지만, 그 내부 구현 단계까지는 추적할 수 없는 점이라고 생각된다. 만약 프로젝트가 `numpy`처럼 `C-API`를 통해 C언어 모듈들을 매개하고 있고, 이 내부의 기능까지 프로파일링을 해야 한다면 다른 도구를 사용해야 한다. 나중에 그런 도구를 사용해보게 되면 한번 더 정리해보면 좋을 것 같다.

[1]: https://docs.python.org/3/library/profile.html "Python Official Doc - The Python Profilers"
[2]: https://en.wikipedia.org/wiki/Collatz_conjecture "Wikipedia - Collatz Conjecture"
[3]: https://en.wikipedia.org/wiki/Leibniz_formula_for_%CF%80 "Wikipedia - Leibniz formula for π"
