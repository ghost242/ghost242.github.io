---
layout: post
title: AWS SAM로 서비스를 만들어 보자 (1)
subtitle: 시작하는 방법
comments: true
categories: ["Technology"]
tag: ["AWS", "SAM", "Beginner", "Serverless"]
---

## 전제

SAM은 AWS의 모든 서비스를 아우르던 CloudFormationd에서 파생된만큼, 내장된 기능을 몽땅 설명하는 것은 너무 범위가 많기 때문에 그렇게 했다간 얘기가 끝나지 않는다. 사실 그런건 AWS의 공식 문서를 보면 모든 설명이 너무 잘 나와있어서 굳이 그런걸 할 필요가 없기도 하다. 따라서 여기서는 적당히 기능들을 사용한 예제 어플리케이션 제작을 기준으로 진행하고 필요한 설명을 첨부하도록 한다.

## 시나리오

예제로 제작할 샘플 어플리케이션 구성은 다음과 같다.

(Input) -> [API Gateway] - [Lambda] - [SQS] - [Lambda] - [S3]

우선 어떤 HTTP(S) Path로 요청이 들어오면, 그 요청을 Trigger로 Lambda가 동작하고, SQS에 메시지를 넣고 프로세스가 종료된다. 그 다음에 Lambda가 SQS를 Trigger로 동작해서 전달된 메시지를 S3에 저장하고 프로세스가 종료된다.

## IAM 설정

AWS는 root권한을 갖는 마스터 계정(사용자의 AWS계정)이 모든 것을 통제하는 상황을 권장하지 않는다. 가급적이면 AWS 시스템 내에서 관리하는 서브권한을 이용하라고 추천해준다. 따라서 IAM 설정 화면으로 들어가보면 사용자가 제어할 수 있는 수많은 서비스에 대한 관리권한들을 볼 수 있다. 사실 너무 많아서 몽땅 다 보는건 말이 안되고, 그 중에서 몇가지 필수적인 권한만 보려고 한다. 

* CloudFormation 구성
    * "cloudformation:CreateChangeSet",
    * "cloudformation:ExecuteChangeSet",
    * "cloudformation:GetTemplateSummary"
 
* IAM 권한 부여
    * "iam:ListPolicies",
    * "iam:DetachRolePolicy",
    * "iam:DeleteRolePolicy",
    * "iam:CreateRole",
    * "iam:DeleteRole",
    * "iam:AttachRolePolicy",
    * "iam:PutRolePolicy"

* 리소스 생성/수정/삭제
    * 


## SAM CLI 설치

SAM CLI를 설치하는 방법은 정말 다양하지만, 그 중에서 나는 Python 3.7 버전에서 작업을 진행했다. 따라서 선행작업으로 Python 3.7의 설치가 필수이다.

Python 3.7이 설치되어있다면 pip를 이용해 간단히 설치가 가능하다.
```bash
$ pip install aws-sam-cli --user
```

**⚠️참고**: 만약 다른 Runtime 인터프리터를 사용한다면 그에 맞게 설치하면 된다. 예를들어 Node 10.4 에서 작업을 한다면 아래와 같은 명령어를 통해 설치하면 된다.
```bash
$ npm install -g aws-sam-local
```

## SAM 초기화 및 yaml 구성

SAM에 이미 익숙한 사람이라면 template 파일을 직접 제작해도 상관없지만, 아직 익숙하지 않은 사람들을 위해 초기 구성을 제공하는 init 명령어가 있다. 
```bash
$ sam init --runtime python3.7
```
이 명령어는 아래와 같은 파일들을 만들어준다.
```bash
.
+-- template.yaml # 어플리케이션 구성을 정의하는 문서
+-- event.json # 로컬환경에서 람다를 테스트할 때 event 파라미터로 입력되는 값
+-- hello_world # 프로그램 코드
|   +-- __init__.py
|   +-- app.py # 람다 핸들러 함수가 들어있는 파일
+-- tests
|   +-- unit
|   |   +-- __init__.py
|   |   +-- test_handler.py # 람다 핸들러 함수를 테스트하기위한 유닛테스트 코드
```

이 중에서 template.yaml이 바로 AWS CloudFormation에서 각 리소스의 배치를 정의하는 문서를 의미한다. CloudFormation의 GUI Designer를 이용하면 비슷한 파일을 만들 수 있지만, 몇가지 결정적으로 다른부분이 있다. SAM은 Lambda, API Gateway, SimpleTable(DynamoDB), Layer로 이루어진 리소스를 정의하고, Lambda의 트리거로 S3, SQS, Kinesis, DynamoDB, SNS, Api, Schedule, CloudWatchEvent, CloudWatchLogs, IoTRule, AlexaSkill를 이용할 수 있다고 되어있다.

뭔가 이상하지 않은가.. API Gateway와 DynamoDB는 리소스이면서 이벤트 트리거의 역할도 동시에 한다. 그래서 그런지 API는 굳이 리소스로 정의하지 않아도 Lambda의 이벤트에 넣기만 해도 나중에 Deploy했을 때 자동으로 트리거항목에 추가되고 나중에 CloudFormation의 GUI Designer로 살펴봐도 항목이 존재한다. 아무래도 SAM이 아직 베타 버전이기 때문에 생기는 문제점이 아닌가 생각된다. 

SAM을 이용한 어플리케이션 구성은 바로 여기서부터 시작하게 된다. 이 이후의 작업은 다음 포스트에서 다뤄보도록 하겠다.