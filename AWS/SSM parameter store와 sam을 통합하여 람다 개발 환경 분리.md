# SSM parameter store와 sam을 통합하여 람다 개발 환경 분리

## samconfig.toml & templaye.yaml
- 우선 SAM의 samconfig.toml과 template.yaml을 통해서 개발 환경을 분리해보자
- samconfig.toml은 SAM CLI의 배포 설정을 저장하는 구성 파일이다.

### samconfig.toml

```toml
[dev]
[dev.global]
[dev.global.parameters]
stack_name = "dev-test"

[dev.deploy]
[dev.deploy.parameters]
s3_prefix = "dev-test"
region = "ap-northeast-2"
resolve_s3 = true
disable_rollback = true
image_repositories = []
capabilities = "CAPABILITY_IAM"
confirm_changeset = true
parameter_overrides = [
    "Environment=dev"
]

[prod]
[prod.global]
[prod.global.parameters]
stack_name = "prod-test"

[prod.deploy]
[prod.deploy.parameters]
s3_prefix = "prod-test"
region = "ap-northeast-2"
resolve_s3 = true
disable_rollback = true
image_repositories = []
capabilities = "CAPABILITY_IAM"
confirm_changeset = true
parameter_overrides = [
    "Environment=prod"
]
```

- `[dev]` 이나 `[prod]`와 같이 배포 설정을 분리할 수 있다.
- stack name
- `parameter_overrides`로 sam의 template.yaml 의 `Parameters`을 오버라이딩 할 수 있다.
- 예시에서는 Environment 파라미터를 오버라이딩해서 template.yaml에서 배포 환경에 따라 해당 값을 다르게 전달한다.

### template.yaml

- template.yaml에서는 cloudformation 템플릿을 사용해 서버리스 리소스(Fucntion, Scheduler) 을 정의한다.
- 기존 cloudformation 처럼 Globals, Parameters와 같은 설정이 있다.
- 여기에서 ssm parameter store에 운영환경 (dev, prod)별로 경로를 다르게 설정 한 뒤 변수들을 저장시켜둔다.
    - 예를 들면 `/example/prod/db-url` 과 `/example/dev/db-url`로 경로를 설정할 수 있다.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31

Parameters:
  NowEnvironment:
    Type: String

Globals:
  Function:
    Timeout: 30
    MemorySize: 256

    Tracing: Active
    LoggingConfig:
      LogFormat: JSON
  Api:
    TracingEnabled: true
Resources:
  TransferFunction:
    Type: AWS::Serverless::Function
    Properties:
      FunctionName: !Sub "${NowEnvironment}-smart-transfer"
      CodeUri: test/
      Handler: app.lambda_handler
      Runtime: python3.9
      Architectures:
      - x86_64
      Policies:
      - AmazonSQSFullAccess
      - SSMParameterReadPolicy:
            ParameterName: '/example/*'
      Events:
        SQSEvent:
          Type: SQS
          Properties:
            Queue: !Sub '{{resolve:ssm:/smart/${NowEnvironment}/transfer-sqs-url}}'
            BatchSize: 5
            Enabled: true
      Environment:
        Variables:
          DB_URL: !Sub '{{resolve:ssm:/example/${NowEnvironment}/db-url}}'
          ENV: !Ref NowEnvironment
      ...
Outputs:
    ...
```

- samconfig.toml에서 설정한 parameter_overrides을 통해서 template.yaml의 `Parameters`을 환경별로 오버라이딩한다.
- sam의 환경변수나 기타 다른 설정에 ssm parameter store의 값을 가져오려면 정책에 `SSMParameterReadPolicy`를 추가해야 한다.
- 그리고 이것을 cloudformation의 `!Sub` 문법이나 `!Ref` 문법을 통해서 여러 곳에서 사용한다.
    - 예를 들면 `'{{resolve:ssm:/example/${NowEnvironment}/db-url}}'`은 cloudformation에서 ssm parameter store의 변수를 가져오는 문법인데, 여기에 `!Sub`를 통해서 Parameters의 `NowEnvironment` 값을 넣을 수 있다.
- 참고로 아직 sam template 에서는 `SecretString` 유형의 파라미터를 불러올 수 없다.