---
layout: post
title: 국가, 언어, 시간 표기에 대해서
subtitle: 파이썬과 현실세계 데이터의 연결
comments: true
categories: ["Python"]
tag: ["Review", "Python", "Beginner"]
---

## 개요

프로그램 개발을 하면서 항상 제대로 알고있는건지 의심하게 되는, 사소할 수 있는 정보를 한번 정리해봤다. 우선 결론부터 이야기하면 한국의 국가-언어 코드는 ko-kr(ko_KR)이고 유니코드 또는 EUC-KR 인코딩 방식을 사용하고, Asia/Seoul(UTC+0900) 시간대에 속해있다. 이 문서는 우리가 사용중인 이런 표기법에 대한 설명이다.

## 국가 및 언어 표기

국가와 언어는 묶어서 표현된다. en-us(en_US) 또는 en-uk(en_UK), de-de(de_DE), ko-kr(ko_KR), ja-jp(ja_JP) 등으로 표기가 된다. 각각 언어 코드, 국가 코드 순서로 표기되고 linux에서도 이런 표기를 따르도록 되어있다. 예를들어 linux terminal에서 한국에서 유니코드로 한글표기를 기본 출력으로 설정하고 싶다면 LC_LANG=ko_KR.UTF-8 환경변수를 설정한다.

### 국가 코드 표기

국가 코드는 ISO 3166 으로 표준화 되어있다. 모든 국가 코드 목록은 Country code 에서 검색을 통해 찾을 수 있다.

이 코드는 처음에 2개의 알파벳으로 표기되었는데 이후에 국가가 추가되면서 중복되는 코드가 생겨서 3개의 알파뱃으로 새로운 표준이 생겼다. 각각 Alpha 2, Alpha 3으로 표현한다. 그러나 아직 ISO 3166 Alpha 2가 기본이므로 대부분의 시스템이 이 표준을 따르고 있다.

한국은 kr로 표기한다.

### 언어 코드 표기

언어 코드는 ISO 639-1, ISO 639-2, ISO 639-5 로 표준화되어있다. 모든 언어 코드 목록은 미국 정부 페이지인 Library of Congress의 ISO 639.2 code list 페이지를 참고하는 것이 좋다.

이 코드는 처음에 2개의 알파뱃으로 표기되었는데 이후에 새로운 언어가 추가되면서 3개의 알파뱃을 사용하는 ISO 639-2가 새로운 표준으로 정의되었고 이후에 지역 언어들을 그룹화 하면서 ISO 639-3, ISO 639-5(ISO 639 part.5) 로 표준이 다시 정의되었다. ISO 639-5는 표기 자체도 복잡하고 언어 지역, 어족 등으로 Hierarchy로 표현하고있기 때문에 표기법이 너무 복잡해져서 대부분의 시스템이 ISO 639-1 표준을 따르고있다.

한국은 ko로 표기한다.

### 파이썬에서의 로케일

locale 모듈을 이용하면 현재 시스템의 코드를 알 수 있다.

``` python 
import locale

print(locale.getlocale())  # 파라미터를 넣지 않으면 LC_CTYPE 변수를 반환하도록 되어있다.

locale.setlocale(locale.LC_ALL, 'ko_KR')  # locale code를 한국어로 설정하고싶을 경우에.
```

이 부분은 linux/unix 시스템에 대한 환경변수에 대해 알아보는 것이 더 낫다.

## 인코딩 방식

컴퓨터에는 인간의 언어를 기계어로 변환하는 코드표가 존재한다. 컴퓨터 프로그래밍을 배울때 가장 처음 배우는 코드표는 아스키코드로, 128비트 안에서 영어표기를 표현하기 위해 만들어졌다. 이후에 여러언어로 확장되면서 256비트로 늘어났지만 세계의 모든 언어를 표현할 수는 없었기 때문에 이후에 다양한 인코딩 방법이 만들어진다. 그 중에서도 한국에서 널리 사용되는 인코딩 방식으로는 EUC-KR, CP949, 그리고 유니코드가 있다.

### 유니코드

현재 가장 최근버전 유니코드로 14버전이 있고, 15버전이 alpha버전으로 만들어지는 중으로 보인다. 전체 코드표는 pdf파일로 되어있다.(https://www.unicode.org/Public/14.0.0/charts/CodeCharts.pdf)

유니코드는 ISO 8859 표준 논의에서 파생된 표준 코드이다. 유니코드는 몇가지 바리에이션이 있는데, UTF-7, UTF-8, UTF-16, UTF-32 등으로 분화된다. 여기에서 현재 가장 많이 쓰이는것은 UTF-8과 UTF-16인데, UTF-16은 UTF-8에서 확장되는 형태로 표준이 제정되었다. 유니코드에서 참고할만한 것은 정규식으로, 알파뱃을 다 포함시키려면 [a-zA-Z] 라고 표현하면 된다는 것은 다 알고 있을 것이다. 한글을 이렇게 포함시키려면 [가-힣] 이렇게 쓰면 된다. 유니코드 테이블에서 한글의 가장 마지막 글자가 힣 이기 때문이다.

### EUC-KR

이 포맷은 한국 산업규격인 KS X 1001을 바탕으로 만든 완성형 코드표로 한글을 컴퓨터로 다루기위해 만들어진 국내 표준 규격이다. 이때에 한국에는 조합형 코드도 있었지만 코드 자체가 이해하기 까다롭고 cpu 아키텍처가 확장되면서 코드를 파싱하는데 어려움이 따랐다고 전해진다. 완성형 코드표이기 때문에 8,836개의 글자를 코드화 해서 담고있다(위키피디아 KS X 1001 페이지 참고). 그래서 옛날에는 한국에서 잘 쓰이는데도 컴퓨터로 쓸 수 없는 글자들이 많이 있었다.

### CP949

이 인코딩 포맷은 Code page 949의 약자로, 마이크로소프트가 한글 윈도우즈를 만들면서 EUC-KR 포맷을 확장해서 만들었다. 그래서 전국의 90%이상의 컴퓨터가 윈도우즈 운영체제를 사용중이었던 한국에서는 완성형 코드인 CP949라는 인코딩 포맷이 da facto로 자리잡았고 최근까지도 간혹 한글 윈도우즈에서 유니코드와 CP949가 충돌하면서 글자가 깨지는 문제가 있었다. 만약 구버전 윈도우즈에서 만들어진 텍스트파일의 글자가 정상적으로 출력되지 않는다면 이 인코딩 방식을 의심해볼 필요가 있다.

### 파이썬에서의 인코딩

파이썬3에서는 공식적으로 모든 스크립트의 인코딩 방식이 유니코드인 것을 기본으로 하고있다. 그러나 유니코드를 실제로 다뤄볼 때는 또 다른 문제가 생기기도 한다. 파이썬 공식문서의 예제를 보면 이런 코드가 있다.

``` python
import unicodedata

def compare_strs(s1, s2):
    def NFD(s):
        return unicodedata.normalize('NFD', s)

    return NFD(s1) == NFD(s2)

single_char = 'ê' # 0x00EA
multiple_chars = '\N{LATIN SMALL LETTER E}\N{COMBINING CIRCUMFLEX ACCENT}'  # 0x0065 0x0302
print('length of first string=', len(single_char))  # 1
print('length of second string=', len(multiple_chars))  # 2
print(compare_strs(single_char, multiple_chars))  # True; 둘 다 같은 글자.
```

저 문자는 로마자로 위에 강세를 표현하는 기호가 혼합된 글자인데, 이 둘을 비교한 것이다. 위 코드에서 말하는 것은 같은 글자를 표기하더라도 두 기호를 합성한 글자인 경우에는 single_char이 multiple_chars로 서로 다를 수 있다는 것이다. 저 둘을 단순히 비교하면 False가 나온다. 이것을 위해서 서로다른 표기를 normalize하는 과정이 필요한데, 그게 저 함수인 unicodedata.normalize이다.

그럼 한글은 어떨까… 싶어서 이런 코드를 실행시켜봤다.

``` python
>>> import unicodedata
>>> s = '맥북프로'
>>> nfc_s = unicodedata.normalize('NFC', s)
>>> nfkc_s = unicodedata.normalize('NFKC', s)
>>> nfd_s = unicodedata.normalize('NFD', s)
>>> nfkd_s = unicodedata.normalize('NFKD', s)
>>> nfc_s
'맥북프로'
>>> nfkc_s
'맥북프로'
>>> nfd_s
'ㅁㅐㄱㅂㅜㄱㅍㅡㄹㅗ'
>>> nfkd_s
'ㅁㅐㄱㅂㅜㄱㅍㅡㄹㅗ'
>>> [ord(c) for c in nfc_s]  # 각 글자들의 코드값
[47589, 48513, 54532, 47196]
>>> [ord(c) for c in nfkc_s]
[47589, 48513, 54532, 47196]
>>> [ord(c) for c in nfd_s]
[4358, 4450, 4520, 4359, 4462, 4520, 4369, 4467, 4357, 4457]
>>> [ord(c) for c in nfkd_s]
[4358, 4450, 4520, 4359, 4462, 4520, 4369, 4467, 4357, 4457]
```

NFC, NFKC는 조합된 완성형 한글로 글자를 다루고있고, NFD, NFKD는 한글 초성, 중성, 종성을 모두 분리해서 따로 다루고 있음을 알 수 있다.

NFC(D)와 NFKC(D)의 차이는 무엇인가 하면 stack overflow에서 해답을 찾을 수 있다.

``` python
>>> unicodedata.normalize('NFC', '\u2167')
'Ⅷ'
>>> unicodedata.normalize('NFKC', '\u2167')
'VIII'
```

`\u`는 유니코드 코드임을 표현하는 prefix이고, 2167은 로마자 8을 의미한다. 이걸 NFKC로 normalize하면 저 조합을 알파뱃 V,  알파뱃 I, 알파뱃 I, 알파뱃 I 이렇게 따로 분해해버린다. 파이썬 문서에서는 compatibility equivalence 이라고 되어있는데, 아무래도 VIII가 로마자 숫자 8과 같은 모양으로 표기되는 것을 서로 호환된다고 보는것 같다.
시간대

시간대는 영국 그리니치 천문대의 위치를 기준으로 삼고있다. 즉, GMT = UTC+0000인 것이다. 여기에서 동쪽으로 갈 수록 시간대가 +로 표기되고 서쪽으로 갈 수록 시간대가 -로 표시된다. 이 시간대는 태평양 한가운데에 아주 복잡하게 -10, -11, +13, -12, +12 등이 뒤섞여있고, 다시 경도에 따라 시간대가 얼추 나눠진다. 세계지도에서 시간대를 표기한 표는 이렇게 생겼다.

{{국제 시간대 지도}}

지도를 잘 보면 아주 복잡하게 선이 나뉘는 것을 알 수 있는데, 정치적인 부분도 하나의 원인일 것으로 보인다. ISO 8601 표기법을 자세히보면 timezone을 표기할 때는 시, 분을 같이 표기하고 있다. 이것은 비슷한 경도라도 더 정밀하게 시간대를 구분해야 할 필요가 있기 때문으로 보인다. 그리고 IANA에서 관리하고있는 timezone database를 뒤져보면 한국의 시간대와 표기에 몇번의 변화가 있었음을 알 수 있다.  

| Offset | Rules | Until |
| :-- | :-- | :-- |
| 08:27:52 | LMT | 1908-04-01|
| 08:30|KST|1912-01-01|
| 09:00|JST|1945-09-08|
| 09:00|KST|1954-03-21|
| 08:30|KST|1961-08-10|
| 09:00|KST||

한국의 시간대는 일본 도쿄를 기준으로 하는 시간대에 편입되어 같이 지정되었던 이후로 거의 변화가 없다. 따라서 현재 한국의 시간대는 +0900라고 표기한다. 그리고 중요한건 아니지만, 북한의 시간대의 이름은 Asia/Pyongyang이고 KST와 동일하다.

### 파이썬에서의 시간대

파이썬3의 날짜, 시간모듈은 datetime인데, 여기에는 ISO 8601 표준에서 정의하고있는 시간표기방법을 모두 지원하고 있다. 그러나 문제는 python3의 표준라이브러리에서는 timezone database 정보가 빠져있다. 따라서 개발자가 서비스를 운영하려는 지역의 timezone offset을 모두 알고 적용하 지원하지 못하는 문제가 있다. 따라서 3.8 버전까지는 <ins>Stuart Bishop</ins>(<stuart@stuartbishop.net>) 라는 사람이 개발한 pytz라는 모듈이 거의 필수모듈처럼 사용되고있었다. 3.9로 버전업되면서 IANA의 timezone database를 참고로 하는 zoneinfo라는 모듈이 추가되었다. 그래서 시간대를 다루는 코드는 python 3.8까지의 코드와 python 3.9까지의 코드가 약간 달라지게 된다.

* 예) python 3.8에서 한국 시간대(+09:00) 표기

    ``` python
    # without pytz 
    from datetime import datetime, timezone, timedelta  

    kst = timezone(timedelta(hours=9))

    # datetime.datetime(2022, 3, 4, 13, 0, 0, 0, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400)) 
    print(datetime.now(tz=kst))
    # with pytz 
    from datetime import datetime 
    import pytz  

    kst = pytz.timezone('Asia/Seoul')    

    # datetime(2022, 3, 4, 13, 0, 0, 0, tzinfo=<DstTzInfo 'Asia/Seoul' KST+9:00:00 STD>) 
    print(datetime.now(tz=pytz.timezone("Asia/Seoul"))) 
    ```

* 예) python 3.9에서 한국 시간대 표기

    ``` python
    import datetime 
    import zoneinfo  

    kst = zoneinfo.ZoneInfo("Asia/Seoul")

    # datetime(2022, 3, 4, 13, 0, 0, 0, tzinfo=zoneinfo.ZoneInfo(key='Asia/Seoul')) 
    print(datetime.now(tz=kst))
    ```

pytz 패키지는 timezone database를 바탕으로 하고있는 프로젝트이기 때문에 database를 관리하고있는 IANA의 정보가 업데이트 되는데 맞춰서 패키지 업데이트가 이뤄지고있다. 그리고 3.9에서부터 추가된 zoneinfo 객체는 /usr/share/zoneinfo 경로에서 갖고있는 데이터베이스 값을 가져오도록 되어있다. 또한 osx의 경우에는 /usr/share/zoneinfo -> /var/db/timezone/tz/{version}/zoneinfo 경로에 가면 데이터를 확인할 수 있다.
