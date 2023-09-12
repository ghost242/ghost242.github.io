---
layout: post
title: AWS SAM로 서비스를 만들어 보자 (2)
subtitle: template 파일 구성하는 방법
parent: AWS
comments: true
categories: ["Technology", "CloudService", "AWS"]
tag: ["AWS", "SAM", "Beginner", "Serverless", "Application"]
---

## SAM Template 작성

이제 이전에 이야기했던 서버리스 구성을 어떻게 SAM의 Template 파일로 정의할지 이야기해보자.

> (Input) -> [API Gateway] - [Lambda] - [SQS] - [Lambda] - [S3]

이 구성을 정의하기위한 파일은 다음과 같은 영역들로 구성된다.

### Template file format version (Optional)

Template 파일의 포맷 버전을 정의하는 위치이다. CloudFormation을 구성하는 Template 파일과 같은 내용을 갖는다.

```YAML
AWSTemplateFormatVersion: '2010-09-09'
```

### Transform (Optional)

이 영역은 CloudFormation에서 SAM으로 변환된 Template파일의 버전을 정의하는 영역이다. CloudFormation 문서에서는 옵션으로 표시되어있다.

```YAML
Transform: AWS::Serverless-2016-10-31
```

### Description (Optional)

이 Template 파일에 대한 사람이 읽을 수 있는 적당한 설명을 작성하는 영역이다.

```YAML
Description: >
  AWS 
  
  Sample SAM Template for AWS
```

### Resource -- global section (Optional)

여기서부터 본격적인 Resource 구성의 설정을 작성하는 영역이다. Globals로 시작하는 이 지점은 각 Resource들의 공통 설정을 작성하는 부분인데, 현재는 Lambda(Function) 외에는 각각 1개의 리소스만으로 정의되므로 여기는 공통으로 사용될 설정이 필요한, Function에 대한 global properties를 정의한다.

```YAML
Globals:
  Function:
    CodeUri: . # 소스코드의 위치. Template file로부터 상대위치를 의미한다. 이 위치에 requirements.txt파일이 있어야 한다.
    Runtime: python3.7 # 인터프리터(컴파일러)의 종류와 버전을 표기하는 부분이다.
    Timeout: 30 # 이 Lambda의 실행 제한시간을 정의한다. 단위는 초이며, 현재는 30초이상 실행되면 타임앋웃이 걸린다.
```

### Resource -- resouces section (Required)

여기는 실제 각 Resource에 대한 정의를 만드는 영역이다. 만들 수 있는 resource들은 CloudFormation 내에서는 서로 분리된 resource인 경우가 많아서 CloudFormation의 개별 resource에 비해 Property 영역이 복잡한 경향이 있다. 따라서 내가 만들려는 resource에 필요한 설정이 무엇인지 잘 파악하고 만들어야 한다.

```YAML
Resources:
  GetMessageFunc: # API Gateway를 통해 메시지를 받는 Lambda를 정의하는 영역이다.
    Type: AWS::Serverless::Function 
    Properties:
      # CodeUri 로부터 Python 패키지를 표현하듯이 작성한다. 최종적으로 event와 context를 파라미터로 받는 함수가 Lambda의 실행함수가 되는데, 여기서는 아래와 같다.
      Handler: app.get_message.app.lambda_handler 
      # 이 Lambda가 갖는 다른 resource에 대한 권한을 기술하는 부분이다. 이 Lambda는 API로부터 값을 받아 SQS로 전달하는 역할만 하므로, SQSSendMessage에 대한 권한만 갖는다. 
      Policies:
        - SQSSendMessagePolicy:
            QueueName: !GetAtt DataPipeQueue.QueueName
      # 환경변수에 대해 정의하는 영역이다. 이 영역은 Lambda 함수 정의 대쉬보드의 환경변수 영역과 같다.
      Environment:
        Variables:
          # SQS의 URL을 환경변수로 전달하는 지점
          DATAPIPE_QUEUEURL: !Ref DataPipeQueue
      # Lambda를 호출하는 event source 들을 정의하는 영역이다. 
      Events:
        # API Gateway
        GetMessageAPI:
          Type: Api 
          Properties:
            Path: /write
            Method: get
        # CloudWatchLog
        GetMessageLogs:
          Type: CloudWatchLogs
          Properties:
            LogGroupName: !Ref GetMessageLogsGroup
            FilterPattern: Error
  StoreMessageFunc:
    Type: AWS::Serverless::Function
    Properties:
      Handler: app.store_message.app.lambda_handler
      # 여기는 SQS로부터 받은 메시지를 S3에 저장하기 위한 권한을 정의하는 영역이다.
      Policies:
        - S3CrudPolicy:
            BucketName: !Ref DataStoreS3
      Environment:
        Variables:
          # S3의 URL을 환경변수로 전달하는 지점
          DATASTORE_BUCKETNAME: !Ref DataStoreS3
      Events:
        # 이 Lambda가 SQS를 통해 메시지를 받았을 때 호출됨을 알 수 있다.
        DataPipeEvent:
          Type: SQS
          Properties:
            Queue: !GetAtt DataPipeQueue.Arn
            BatchSize: 10
            Enabled: true
        StoreMessageLogs:
          Type: CloudWatchLogs
          Properties:
            LogGroupName: !Ref StoreMessageLogsGroup
            FilterPattern: Error
  # 수신된 메시지를 담아두기 위한 S3 버킷을 정의하는 영역이다. 여기에 대한 설정은 CloudFormation을 따른다.
  DataStoreS3:
    Type: AWS::S3::Bucket
    Properties:
      AccessControl: "Private"
      BucketName: "sam-practice-data-store-s3"
  # 위의 S3에 대한 권한을 정의하는 영역이다.
  DataStoreS3Policy:
    Type: AWS::S3::BucketPolicy
    Properties:
      Bucket: !Ref DataStoreS3
      PolicyDocument:
        Statement:
          Effect: Allow
          Principal: "*"
          Action: "s3:*"
          Resource: !Join ["", ["arn:aws:s3:::", !Ref "DataStoreS3", "/*"]]
  # 마찬가지로 SQS에 대해 정의하는 영역이다.
  DataPipeQueue:
    Type: AWS::SQS::Queue
    Properties:
      QueueName: "DataPipeQueue"
  # SQS 권한에 대산 설정
  DataPipeQueuePolicy:
    Type: AWS::SQS::QueuePolicy
    Properties:
      Queues:
        - !Ref DataPipeQueue
      PolicyDocument:
        Statement:
          Effect: Allow
          Principal: "*"
          Action: "sqs:*"
          Resource: "*"
  # lambda의 log를 기록하기위한 CloudWatchLogsGroup에 대해 정의하는 영역이다. GetMessageFunc와 StoreMessageFunc를 각각 따로 정의했다.
  GetMessageLogsGroup:
    Type: AWS::Logs::LogGroup
    Properties:
      RetentionInDays: 7
  StoreMessageLogsGroup:
    Type: AWS::Logs::LogGroup
    Properties:
      RetentionInDays: 7
```

### Outputs (Optional)

여기에 작성되는 내용은 CloudFormation에서 스택 그룹을 선택하면 나오는 [출력(Output)] 탭에 출력될 내용을 작성하는 부분이다. 필수 작성 영역은 아니지만, 있으면 각 스택에 접근하는 URL, ARN 등의 설정을 보기도 좋고, 어떤 스택으로 이루어져있는지, IAM Role이 어떤 Role을 따르는지 등의 정보를 파악하기 좋다.

```YAML
Outputs:
  MessageStorage:
    Description: "GetMessageFunc Lambda API entrypoint"
    Value: !Sub "https://${ServerlessRestApi}.execute-api.${AWS::Region}.amazonaws.com/Prod/write/"
  GetMessageFunc:
    Description: "Get message Lambda Function ARN"
    Value: !GetAtt GetMessageFunc.Arn
  StoreMessageFunc:
    Description: "Data send & receive function ARN"
    Value: !GetAtt StoreMessageFunc.Arn
  GetMessageFuncIamRole:
    Description: "Implicit IAM Role created for Hello World function"
    Value: !GetAtt GetMessageFuncRole.Arn
```

## 결론

위에 조각조각 나뉘어진 YAML 코드들을 template.yaml 이라는 파일 하나에 묶어서 만들면 SAM에 대한 template.yaml 파일 구성이 끝난다. 이렇게 Template 파일을 정의하고 나면 다음은 실제 코드를 작성해야 한다. 앞선 포스트에서 설명했듯이 sam init 명령어를 통해 만든 파일들을 적당히 수정하면서 만들어보는 방향으로 시도해보면 비교적 익히기 쉬울 것이다.

## 참고

* [Template 파일 영역구성 설명](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-anatomy.html)
* [SAM 구성에 대한 설명](https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md)
* [CloudFormation 스택 구성에 대한 설명](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)
* [SAM에 대한 전반적인 설명에 대한 AWS 문서](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html)
