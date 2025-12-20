---
layout: post
title: pytest로 시스템 테스트를 하는 방법 - 3 
subtitle: 행위 기반 테스트(Behavior driven development) 
parent: Tests
grand_parent: Python
comments: true
categories: ["Programming", "Python3", "Testing"]
tag: ["Test", "Python", "pytest", "BDD", "behave"]
---

## 개요

behave는 행위 기반 테스트(Behavior driven test)의 행위를 순차적으로 정의하기 위해 만들어진 Cucumber라고 하는 언어와 테스트 도구를 담고있는 프레임워크이다. Cucumber는 C/C++, Java, Python, Golang, Swift, Javascript 등등의 다양한 언어를 지원하고, 행위에 대한 정의를 위해 별도의 문법을 갖고있다. 그 중에서 Python에는 behave라는 패키지가 Semi-official 이라는 레이블로 관리되고 있는데, 이 도구는 유닛테스트 도구인 pytest에 pytest-bdd라는 패키지로 모듈화한 것으로 볼 수 있다.

이 도구의 가장 큰 장점으로 생각할 수 있는 것으로는 기존에 pytest에서 독립적으로 수행하던 유닛테스트를 단계적으로 실행할 수 있게 해주고, 유닛테스트로 사용하던 테스트 함수를 재사용하는 것으로 불필요한 코드 작성을 막을 수 있다.

이 문서는 behave와 gherkin을 실제로 사용 할 때 즉시 참고하기 위한 방향으로 작성해보려 한다. 

## 사용

behave는 최소 2개의 파일로 구성된다. 하나는 test_*.py 파일이고, 다른 하나는 test_*.feature이다.

### feature 파일

아주 단순하게, feature 파일의 구조는 대략 이렇게 작성하게 되어있다.

``` gherkin
Feature: 이 테스트의 요약 설명
  Background
    Given "테스트를 위한 바탕 설명"
    And "테스트를 위한 바탕 설명"
    ...
    
  Scenario: 테스트하려는 항목의 시나리오 요약 설명
    Given "선행 조건 설명"
    And "선행 조건 설명"
    ...
    When "테스트 대상 설명"
    And "태스트 대상 설명"
    ...
    Then "테스트 결과 설명"
    Then "테스트 결과 설명"
    But "테스트 결과 설명"
    ...
    
  Scenario: 테스트하려는 항목의 시나리오 요약 설명
    Given "선행 조건 설명"
    * "선행 조건 설명  # Given 하위에서 And와 같은 동작을 한다."
    ...
    When "테스트 대상 설명"
    * "태스트 대상 설명  # When 하위에서 And와 같은 동작을 한다."
    ...
    Then "테스트 결과 설명"
    * "테스트 결과 설명  # Then 하위에서 And와 같은 동작을 한다."
    ...

  ...
```

feature파일은 계층화된 파일로, 하나의 Feature를 뿌리로 하는 형태이다. Feature 아래에는 하나 이상의 Scenario를 정의할 수 있으며, 개별 Scenario에서는 Given, When, Then, And/But 항목으로 테스트 흐름을 정의할 수 있다. 추가로 Scenario 아래에 Example 등의 섹션을 통해서 더 구조화된 테스트를 작성할 수 있게 된다.


### test 파일

테스트 파일은 pytest를 기본으로 하고있기 때문에 특별한 조건이 없다면 파일 이름을 test_를 전치사로 갖도록 만들어야 한다. 여기에는 위의 feature 파일에 대한 실질적인 동작들을 정의해야 하는데, 기존의 pytest에서 사용하던 것들도 함께 쓸 수 있기 때문에 재사용성을 고려하면서 코드 구조를 잘 만들어야 할 것으로 생각된다.

``` python
@Given("{Given/And/*에서의 선행 조건 설명}", target_fixture="{파라미터로 전달할 때 fixture 이름}")
def prerequisite_alpha():
    pass

@Given("{Given/And/*에서의 선행 조건 설명}", target_fixture="{파라미터로 전달할 때 fixture 이름}")
def prerequisite_beta():
    pass

@When("{When/And/*에서의 테스트 대상 설명}")
def runner_alpha(prerequisite_alpha, prerequiesite_beta, ...):
    pass

@When("{When/And/*에서의 테스트 대상 설명}")
def runner_beta(prerequiesite_beta, ...):
    pass

@Then("{Then에서의 테스트 결과 설명")
def assertion_true_checker():
    pass

@Then("{But에서의 테스트 결과 설명")
def assertion_false_checker():
    pass
```

대략 이런식으로 작성하게되고, 다른 시나리오에서 같은 Given, Then 등을 재사용할 때는 feature에서 시나리오를 작성할 때 동일한 설명문구를 넣는 것으로 실질적인 함수 재사용을 할 수 있다.

이제 아래에 상세 설명으로 테스트 코드와 feature 파일에서 사용되고있는 각 키워드가 어떤 의미를 갖는지 behave와 Cucumber에서 제시하고있는 목적을 바탕으로 아래에 간단하게 정리해본다.

## 상세 설명

### Feature

여러 시나리오를 묶는 가장 큰 범주로, 개별 시나리오에 대한 대략적인 설명을 개발자에게 안내하기 위한 항목이다. 단일 라인만 아니라 줄바꿈을 통해서 복수 라인으로도 설명을 제공할 수 있다. 이 항목에 쓰는 설명 등은 테스트 파일에 들어갈 필요는 없다.

#### Feature 파일에서…

* 한줄 요약

``` gherkin
Feature <summary>
```

* 자세한 설명

``` gherkin
Feature: <title>
   <body>
   <body>
   ...
```

### Scenario

아래의 Given, When, Then등으로 구성되는 테스트 시나리오를 의미한다. 시나리오 자체도 레이블로 식별하기 때문에 유의미하면서 유니크한 문구를 넣는 것이 좋다. 대체로 지금 하려는 테스트 시나리오에대한 설명을 요약하면 좋다. 특히 여기서부터는 수행하려는 테스트의 특성을 개발자도 파악할 수 있어야 하기 때문에 어떤 테스트 시나리오인지 한눈에 파악할 수 있도록 잘 적어두는 것이 좋다.

#### Feature 파일에서…

``` gherkin
Feature: Some feature
    Scenario: <step-summary>
```

#### Python3 코드에서…

이 섹션에서 주의해야 할 점은 `behave` 프레임워크와 `pytest-bdd` 도구에서의 표현이 서로 다르다는 것이다. `behave` 프레임워크에서는 적어줄 필요가 없지만 `pytest-bdd` 에서는 이렇게 표현된다. 

``` python
@scenario("filename.feature", "<step-summary>")
def test_master_bdd():
    pass
```

이 테스트 함수에서는 따로 내용이 없기 때문에 `pass` 구문을 이용해서 함수의 body를 채워둔다. 만약 Pycharm 같은 도구를 사용하고 있다면 테스트 실행 시점을 이 함수로 표기해준다.

### Given

테스트를 수행하기 전에 선행으로 제공해야 할 것을 정의하는 항목이다. pytest에서는 fixture와 유사한 기능을 하지만 의미 상으로는 테스트 단계를 진행하기 전에 선행해야 하는 동작으로 볼 수 있다. decorator에서는 target_fixture 파라미터에 테스트 함수에 fixture로 전달 할 값의 이름을 별칭으로 지정할 수 있다.

#### Feature 파일에서…

``` gherkin
Feature
    Scenario
        Given <item-for-test>
```

#### Python3 코드에서…

``` python
@given()
def test_master_bdd():
    pass
```

### When

미리 테스트를 위해 필요한 리소스를 `Given`을 통해서 준비하고 나면 이제 실제 테스트를 순차적으로 진행하기 위한 단계를 정의해야 하는데, 이때 사용하는게 `When` 구문이다. `When` 구문을 순차적으로 실행하면서 전체 시나리오 단계를 진행하는 것이다. 실제 사용은 아래에 작성해둔다.

#### Feature 파일에서…

``` gherkin
Feature
    Scenario
        Given
        When <item-for-test>
```

#### Python3 코드에서…

``` python
@when("<item-for-test>")
def test_master_bdd():
    pass
```

### Then

테스트 시나리오를 작성할 때는 내가 테스트하기위해 필요한 자원을 미리 정의하고 순차적인 테스트 단계를 정의하게 될 것이다. 그러고 나면 무엇을 확인해서 테스트가 성공적으로 완료되었는지 아니면 실패했는지 검증이 필요하다. behave 도구의 용법을 생각하면 `Given` -> `When` 순서로 테스트를 진행하고 난 뒤에 테스트 결과 어떤 상태에 도달해야 하는지 확인하는 Assertion 구문이 필요하다는 것이다. Feature 파일에 이 내용을 시나리오로 정의해서 뭘 할 것인지 기록해둘 수 있다. 그 구문을 behave(gherkin) 에서 `Then` 이라고 한다. 

#### Feature 파일에서…

``` gherkin
Feature
    Scenario
        Given
        When 
        Then <item-for-test>
```

#### Python3 코드에서…

``` python
@then("<item-for-test>")
def test_master_bdd():
    pass
```

### And/But

`gherkin` 문법에는 Given/When/Then 섹션을 좀 더 사람이 읽을 수 있는 문서처럼 표현하게 하기위해 `And`와 `But`이 정의 되어있다. 이 구문은 python 코드 안에서는 `given`, `when`, `then` 으로 decorator를 작성한다. 



#### Feature 파일에서…

``` gherkin
Feature
    Scenario
        Given <given-line>
        And <given-and-line>
        And <given-and-line>
        
        When <when-line>
        And <when-and-line>
        
        Then <then-line>
        And <then-and-line>
        But <then-but-line>
```

여기에서는 `But`구문을 `Then` 아래에만 적었지만, `When` 에서도 똑같이 활용이 가능하다. 어디까지나 사람이 더 자연스럽게 테스트 시나리오를 표현하기 위해 사용하는 연결자로 다루면 된다. 

#### Python3 코드에서…

``` python
@given("<given-line>")
def test_given():
    pass
@given("<given-and-line>")
def test_given_and():
    pass
@given("<given-and-line>")
def test_given_and_more():
    pass

@when("<when-line>")
def test_when():
    pass 
@when("<when-and-line>")
def test_when_and():
    pass

@then("<then-line>")
def test_then():
    pass
@then("<then-and-line>")
def test_then_and():
    pass
@then("<then-but-line>")
def test_then_but():
    pass
```

## 결론

시스템이 고도화되고 기능이 다른 기능과 유기적으로 연결될 수록 기존의 유닛테스트로는 어려운 테스트들이 너무 많아진다. 물론 setup, teardown 단계를 두어 이 모든 것들을 미리 준비해놓고 테스트할 수 있고 fixture 등을 활용해서 테스트 목적에 맞춰 다양한 환경을 미리 구성해놓을 수 있지만, 어떤 기능이 다른 기능에 의존해서 선, 후가 나뉘고 이 단계가 조건에따라 조정되어야 한다면 이것만으로는 상당히 까다로울 수 있다. 

이 cucumber, gherkin, behave, pytest-bdd 를 활용해보면 이런 단계들을 재사용하면서도 상황을 좀 더 사람이 읽기 쉽게 구성할 수 있고 이 상황에 맞춰 환경을 빠르게 구성해볼 수 있을 것이다. 잘 작성되어있다면 어떤 `given` 함수는 다른 시나리오에서도 재사용될 수 있고 모든 시나리오가 완료된 뒤에 assertion 을 시나리오의 목적에 맞춰 검증해볼 수 있을 것이다. 

이런 테스트는 물론 잘 정의 된 기획과 사용자 시나리오, 다양한 조건과 환경, 그에 따른 사용자 풀의 변화를 어느정도 예측할 수 있어야 하기 때문에 연습이 잘 되어있어야 할 것으로 생각된다. 하지만 잘 활용할 수 있다면 한 소프트웨어의 생명주기를 좀 더 길게 바라볼 수도 있을 것이다. 기능이 추가되거나 변경되고 기존 기능이 삭제되더라도 다른 기능과의 연관관계를 파악하면서 여러 기능이 서로에게 어떤 영향을 주고받는지 예측할 수 있게 되기 때문이다.

## 사용 예

이 섹션은 위의 gherkin 문법과 python3과의 연결을 활용하는 하나의 예로 완성된 코드를 첨부 해두기 위해 적어두었다.

``` gherkin
Feature: Blog
    A site where you can publish your articles.

    Scenario: Publishing the article
        Given I'm an author user
        And I have an article

        When I go to the article page
        And I press the publish button

        Then I should not see the error message
        # Note: will query the database
        And the article should be published  
```

``` python
from behave import given, when, then


@given("I'm an author user")
def author_user(auth, author):
    auth['user'] = author.user


@given("I have an article", target_fixture="article")
def article(author):
    return create_test_article(author=author)


@when("I go to the article page")
def go_to_article(article, browser):
    browser.visit(urljoin(browser.url, '/manage/articles/{0}/'.format(article.id)))


@when("I press the publish button")
def publish_article(browser):
    browser.find_by_css('button[name=publish]').first.click()


@then("I should not see the error message")
def no_error_message(browser):
    with pytest.raises(ElementDoesNotExist):
        browser.find_by_css('.message.error').first


@then("the article should be published")
def article_is_published(article):
    article.refresh()  # Refresh the object in the SQLAlchemy session
    assert article.is_published
```

## 참고 문서

* https://cucumber.io/docs/cucumber/

* https://pytest-bdd.readthedocs.io/en/stable/#example 
