### SQS의 MessageAttributes를 사용한 메타데이터 전송

- SQS 메시지를 보낼 때, 환경 분리를 하고 싶을 수가 있다. 이럴 때 큐 자체를 나누어 Lambda도 별도로 생성해도 되지만, 매우 간단한 로직이라면 `MessageAttributes`를 사용하면 간편하다.
- `MessageAttributes` 는 SQS 메시지를 보낼 때 같이 보낼 수 있는 메타데이터이다. 다음과 같이 전송한다.

```python
response = sqs_client.send_message(
    QueueUrl='https://sqs.ap-northeast-2.amazonaws.com//',
    MessageBody=json.dumps(message),
    DelaySeconds=remaining_seconds,
    MessageAttributes={
        'Environment': {
            'DataType': 'String',
            'StringValue': 'dev'  # or 'prod'
        }
    }
)
```

- 이렇게 하면 SQS 트리거를 사용한 Lambda에서는 다음과 같이 해당 데이터에 접근 가능하다.

```python
environment = record['messageAttributes']['Environment']['stringValue']
```

- MessageAttributes는 꼭 `DataType` 을 명시하고, 이에 맞는 Value로 같이 보내야 한다.
- 숫자는 이렇게 string으로 보내고, 받을 때 int로 변환해준다.

```python
response = sqs_client.send_message(
    QueueUrl=queue_url,
    MessageBody=json.dumps(message),
    MessageAttributes={
        'Version': {
            'DataType': 'Number',
            'StringValue': '123'
        },
        # 받고나서 float로도 변환 가능하다
        'Price': {
            'DataType': 'Number',
            'StringValue': '99.99'
        }
    }
)
```