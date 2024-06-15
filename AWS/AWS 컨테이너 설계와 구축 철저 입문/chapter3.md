# 컨테이너를 이용한 AWS 아키텍처
- AWS well architected
    - 운영 우수성
    - 보안
    - 안정성
    - 성능 효율
    - 비용 최적화
    - 지속 가능성
- 설계 아키텍처
    - Ingress용 public ALB
    - 애플리케이션용 private subnet
    - Aurora용 private subnet
- 옵저버빌리티(가관측성): 시스템 전체를 살펴보며 내부 상태까지 깊이 파고들 수 있는 상태
    - 탐지 -> 파악 -> 특정
## 로깅 설계
- Cloudwatch logs 구독필터를 사용해서 로그 내에서 특정 문자열이 포함된 경우의 로그만을 추출 가능
    - 이 로그를 lambda -> SNS로 장애 통보 가능
- Firelens
    - SaaS에도 로그를 전송 가능, firehose로 S3나 Redshift, OpenSearch Service에도 로그 전송 가능
    - 로그 라우팅 담당하는 `Fluentd` or `Fluent Bit` 선택 가능
    - `Fluent bit`는 `Fluentd`에 비해서 플러그온이 적지만 자원 효율이 좋아서 추천
        - 게다가 `Fluent bit`는 ECS 태스크 정의에서 AWS의 공식 컨테이너를 사용 가능
        - S3에는 Firehose를 경유하지 않고 직접 S3에 출력 가능하다
        - 심지어 cloudwatch에도 동시에 로그를 전송할 수 있다.
- 로그 장기 보관을 위해서 애플리케이션 접속 로그는 S3, 에러 로그만 CloudWatch Logs에 전송하려면?
    - `Fluent bit`를 사용해라.
    - 기본 제공 `Fluent bit` 컨테이너로는 이것을 하지 못하기 때문에 설정파일을 S3 버킷에 넣어두거나, 직접 이미지를 만들어라
## 지표 설계
- 지표는 정기적으로 계측되고 수집되는 시스템 내부의 정량적인 동작 데이터
    - CloudWatch 지표, CloudWatch Container Insights
    - CloudWatch Container Insights를 사용하면 ECS의 테스크 수준의 정보를 파악 가능
    - task수가 많아지면 그만큼 비용이 늘어난다.
    - ECS 클러스터 단위로 명시적인 옵트인이 필요
- X ray를 사용하면 서비스 맵 대시보드도 제공해서 시스템 전체를 시각적으로 표시
    - 사이드카 형태로 x-ray 배포
    - VPC 밖에 서비스 엔드포인트가 있어서 애플리케이션이 VPC 내부에 있다면 따로 VPC 엔드포인트를 만들어줘야함
## CICD
- ECR 팁
    - 개발서버나 스테이징, 프로덕션 이미지에 태그를 달아서 수명 주기 정책을 유연하게 사용
    - 그리고 이미지 만들 때 `운영환경_소스코드저장소 커밋 ID`로 만들면 더 좋다

## 배스천
- 퍼블릭 서브넷에 EC2를 배포하고, 프라이빗 서버와 연결하기 위한 점프 서버
- 내부 작업에 대한 작업 기록도 취득 가능
- 그런데 퍼블릭 서브넷임으로 좀 위험하다
    - 대안으로 배스천서버도 프라이빗으로 넣고, Session manager나 요즘 나온 EC2 Instance connect endpoint 사용
