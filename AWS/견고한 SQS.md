# 견고한 SQS

- 중요한 메시지를 처리하거나 중복되어 실행되면 안되는 로직이 있을 때, SQS에서 딱 한번만 메시지를 멱등성 있게 처리하기 위해서 SQS를 튜닝해보자
- 아키텍처는 특정 메시지 발행처로부터(예시는 람다가 publish) SQS는 메시지를 받고, 수신하는 람다가 이를 처리한다.

## SQS 생성시
- 중복 제거를 `MessageDeduplicationId`를 통해서 자동으로 해주는 **FIFO SQS**를 선택
- 하지만 중복 제거의 방법은 컨텐츠 기반 중복 제거도 존재한다.

### 컨텐츠 기반 중복 제거와 MessageDeduplicationId 설정 차이
- 

### MessageGroupId
-

## Lambda Trigger 생성시

### 배치크기

### 메시지 삭제