---
layout: post
title: AWS SAM로 서비스를 만들어 보자 (3)
subtitle: SAM 명령어로 서비스를 배포하는 방법
comments: true
categories: ["Technology"]
tag: ["AWS", "SAM", "Beginner", "Serverless", "Application"]
---

## CLI 도구

AWS에서는 코드자동화를 위해 cli 도구를 지원한다. AWSCLI가 그 프로그램인데, 사용자의 접근키를 이용해 AWS의 각종 서비스를 콘솔에서 제어할 수 있도록 되어있다. 당연히 제어가능한 서비스에는 Cloudformation도 있는데, SAM은 Cloudformation을 간소화하면서 CLI 도구도 별도로 만들어졌다.

그게 바로 AWS-SAM-CLI이다. github 저장소는 여기([https://github.com/awslabs/aws-sam-cli](https://github.com/awslabs/aws-sam-cli))이고, python으로 개발되어서 pypi에 공개되어있기 때문에 pip 도구를 이용해 바로 설치가 가능하다. 이 SAM CLI에서는 다양한 명령어가 있지만, AWS에 배포할 때 사용되는 명령어는 주로 아래 4가지이다.

참고로, 이 명령어들에 대한 설명과 파라미터 값에 대한 내용을 확인하고 싶으면 함께 첨부하는 링크에있는 문서들을 확인하는 것이 좋다. (SAM CLI는 aws cloudformation의 )

1. validate

    앞서 작성한 template.yaml의 정합성을 체크해주는 명령어이다. 아쉬운점은 sam cli의 버전이 낮은 탓인지 정합성이 제대로 테스트되지 않는다. 실제로 해보면 yaml 문법정도만 체크해주는 것으로 보이는데, schema가 개선되서 각 컴포넌트별로 검증이 잘 되어야 더 잘 쓸 수 있을 것 같다.

    [참고]
    * SAM 문서: [https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-validate.html](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-validate.html)

2. build

    lambda를 구성하는 언어가 python인 경우에는 C언어 패키지들을 AMI 플랫폼에 맞춰서 컴파일해야 할 필요가 있는데, build 명령이 바로 그 역할을 해준다. 해당 명령어를 사용하면 기본 값으로 .build 폴더가 생성되면서 그 안에 코드가 복사되고, requirements.txt에 의존성이 있는 패키지는 새로 빌드되서 추가된다.

    [참고]
    * SAM 문서: [https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-build.html](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-build.html)

3. package

    현재 template.yaml 파일이 cloudformation에서 사용되는 template에 맞춰 변경된다. 이 파일이 실제로 aws 서버에 업로드되면서 스택을 구성하는 구조가 된다. 그리고 위에서 빌드한 코드와 패키지들을 압축해서 지정한 s3 버킷에 업로드를 한다.

    [참고]
    * SAM 문서: [https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-package.html](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-package.html)
    * AWSCLI 문서: [https://docs.aws.amazon.com/cli/latest/reference/cloudformation/package.html](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/package.html)

4. deploy

    미리 s3 버킷에 업로드된 코드를 바탕으로 cloudformation 스택을 생성한다. 이때 생성되는 스택의 이름은 파라미터로 입력하는 값이다.

    [참고]
    * SAM 문서: [https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-deploy.html](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-deploy.html)
    * AWSCLI 문서: [https://docs.aws.amazon.com/cli/latest/reference/cloudformation/deploy/index.html](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/deploy/index.html)

명령어 실행 순서를 단계적으로 정리하면 아래와 같은 순서로 실행된다.

실행 예)

```bash
# 공통적으로 들어가는 --profile 스위치는 awscli에서 관리하는 사용자 접근키에 대한 이름을 의미한다. 이 값은 ini 형식으로 작성되며, 리눅스/osx 기준으로 ~/.aws/config 파일에서 확인 가능하다.

sam validate \
--profile "${PROFILE_NAME}" \
--template "${TEMPLATE_FILE}"

sam build \
--profile "${PROFILE_NAME}" \
--template "${TEMPLATE_FILE}" \
--build-dir ${BUILD_PATH}

sam package \
--profile "${PROFILE_NAME}" \
--template-file ${BUILD_PATH}/template.yaml \
--output-template-file ${BUILD_PATH}/${PACKAGED_TEMPLATE_FILE} \
--s3-bucket "${EXTERNAL_DEST}"

sam deploy \
--profile "${PROFILE_NAME}" \
--template-file ${BUILD_PATH}/${PACKAGED_TEMPLATE_FILE} \
--capabilities CAPABILITY_IAM CAPABILITY_AUTO_EXPAND \
--stack-name "${STACK_NAME}"
```

## 작업 환경

pypi에서 해당 패키지를 찾아보면 python 3.6, 3.7 버전을 지원한다고 되어있다. 그 이외의 버전에서 호환되는지는 확인할 수 없었지만, 아마 3.5 이하 버전에서는 표준 패키지등의 호환성 문제로 일부 기능이 정상동작하지 않을 가능성이 있다.



## 배포 과정

## 결과

