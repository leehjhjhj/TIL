# Lambda와 SQS의 재시도 메커니즘.md

## 람다의 재시도 메커니즘
- 람다 함수는 기본 생성시 비동기식 호출에서 두 번의 재시도를 한다.
- 즉, 만약에 람다에서 에러가 발생하면 1번 + 2번의 재시도를 하여 3번 실행하게 된다.

## SQS의 재시도 메커니즘
- SQS는 메시지를 대기열에서 Trigger 설정이 되어있는 Lambda로 전달할 때, 람다가 메시지를 성공적으로 처리하지 못하면 메시지를 삭제하지 않는다.
- visibility timeout 동안은 다른 consumer가 폴링을 해도 해당 message는 반환되지 않는다.
- 그러다가 visibility timeout 타임 내에 메시지를 처리하지 않으면 타임아웃 이후에 메시지는 다시 큐에 들어가게 된다.
- 다시 큐에 돌아올 때 마다 `ReceiveCount`가 추가된다.
- DLQ(전달 받지 못한 대기열)을 설정하면 **최대 수신 수** 라는 것을 설정하게 되는데, 최대 수신 수는 DLQ로 보내는 `ReceiveCount`의 척도이다.

## 예시
- 만약에 SQS의 DLQ 최대 수신 수를 **2**로 설정하고, Lambda의 함수 재시도 횟수를 **1**로 설정한다.
- 그리고 람다 함수에서 고의로 Exception을 발생시킨다.

```python
sqs = boto3.client('sqs')
dlq_url = ""

def send_to_dlq(message_body):
    response = sqs.send_message(
                QueueUrl=dlq_url,
                MessageBody=json.dumps(message_body),
                MessageGroupId=str(message_body.get('user_id', 'default')),
                MessageDeduplicationId=str(uuid.uuid4())
            )
    return response

def lambda_handler(event, context):
    records = event['Records']
    for record in records:
        body = json.loads(record['body'])
        user_id = body['user_id']
        try:
            if user_id == "1":
                raise Exception("DLQ 실험")
        except:
            print('send to dlq')
            print(send_to_dlq(body))
            
        if user_id == "2":
            raise Exception("Lambda 재시도 실행")
```

- 해당 Lambda는 SQS로부터 메시지를 받아서 user_id가 `1`이면 예외 처리를 통해 DLQ로 전송시킨다.
- 그리고, 재시도 실험을 위해서 user_id가 `2`일 때, 예외를 터뜨린다.