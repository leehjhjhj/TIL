# SAM 맛보기
- AWS SAM(AWS Serverless Application Model)은 서버리스 애플리케이션을 손쉽게 구축하고 배포할 수 있도록 돕는 프레임워크
- 예전에는 github action와 s3 버킷을 직접 만들어 람다 파일을 관리했었는데, 테스트 및 리소스 관리가 어려워 SAM을 공부하고 사용해보았다.

## 설치 및 명령어

```bash
sam init
```
- AWS에서 지원하는 템플릿을 사용해서 빠르게 빌드 가능

```
├── README.md
├── __init__.py
├── events
│   └── event.json
├── samconfig.toml
├── scan_account
│   ├── __init__.py
│   ├── app.py
│   └── requirements.txt
├── template.yaml
└── tests
    ├── __init__.py
    ├── integration
    │   ├── __init__.py
    │   └── test_api_gateway.py
    ├── requirements.txt
    └── unit
        ├── __init__.py
        └── test_handler.py
```

- 이러한 디렉토리로 생성된다.
- **template.yaml**는 AWS CloudFormation 템플릿의 일종으로, 서버리스 애플리케이션의 리소스를 정의
    - **Resources**에는 the SAM specification라는 서버리스용 리소스 뿐만 아니라 기존 CloudFormation의 리소스 타입까지 정의 가능하다. SAM은 CloudFormation 위에서 작동하기 때문이다.
    - **SAM specification**에는 다음과 같은 것들이 있다.
        - AWS::Serverless::Function
        - AWS::Serverless::Api
        - AWS::Serverless::HttpApi
        - AWS::Serverless::Application
        - AWS::Serverless::SimpleTable
        - AWS::Serverless::LayerVersion
    - Event Source도 여기서 정의할 수 있는데, 아직 써보지 않았기 때문에 후에 다시 살펴보자. [공식 레포 문서](https://github.com/aws/serverless-application-model/blob/master/versions/2016-10-31.md#event-source-types)
- **samconfig.toml**은 SAM CLI 명령어의 기본 구성을 저장하는 파일로, 반복적인 설정 입력을 피할 수 있다.
- **requirements.txt**에 패키지 명을 적으면 람다 도커 이미지에 맞는 의존성 버전을 알아서 설치한다.
    - `build`시 패키지들을 로컬 .aws-sam/build에 패키징 된다.
    - 이후 `deploy`시 패키징된 파일을 S3에 업로드하고 이를 기반으로 Lambda 함수가 생성 or 업데이트 된다.

```bash
sam build
sam build --use-container
```

- 두 개의 차이점은 `--use-container`를 붙이면 AWS 람다 환경과 동일한 Docker 컨테이너에서 코드를 빌드함
- 서버리스 애플리케이션을 빌드할 때, 로컬 환경과 AWS Lambda 실행 환경 사이의 차이를 줄이기 위해 사용

```bash
sam deploy
sam deploy --guided
```

- `--guided`를 붙이면 배포시 상세 설정을 물어본다.

## local

```bash
sam local invoke HelloWorldFunction --event 
```

- docker가 켜져 있음에도 `Error: Running AWS SAM projects locally requires Docker. Have you got it installed and running?` 에러가 날 시 DOCKER HOST를 직접 명시해준다.

```bash
DOCKER_HOST=unix://$HOME/.docker/run/docker.sock sam local invoke "HelloWorldFunction" -e events/event.json
```

- 이렇게 해주면 도커 컨테이너가 실행되고 event request로 테스트가 실행된다.

```bash
python -m pytest tests/unit -v
```

- pytest를 통해서 테스트 또한 가능하다.

## 생성하는 람다 역할에 권한 정책 주기

```yaml
Resources:
  ScanFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: /
      Handler: app.lambda_handler
      Runtime: python3.9
      Architectures:
      - x86_64
      Policies:
      - AWSLambda_FullAccess
      - AmazonRDSDataFullAccess
```

- template.yaml에서 간단하게 람다 실행 역할에서 권한을 줄 수 있다.
- Policies에서 AWS 리소스에 대한 권한을 부여한다.