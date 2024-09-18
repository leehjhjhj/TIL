# SSM parameter store와 sam을 통합하여 람다 개발 환경 분리

## samconfig.toml & templaye.yaml
- 우선 SAM의 samconfig.toml과 template.yaml을 통해서 개발 환경을 분리해보자
- samconfig.toml은 SAM CLI의 배포 설정을 저장하는 구성 파일이다.

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