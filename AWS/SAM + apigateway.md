# SAM + APIGATEWAY

## Resource와 API 정의

- Lambda를 배포할 때 바로 API로 배포해야하는 경우가 있는 경우 SAM을 활용하면 쉽게 API 배포가 가능하다.
- 우선 Resource에 API를 정의한다.

```yaml
Resources:
  DirectRegisterApi:
    Type: 'AWS::Serverless::Api'
    Properties:
      Name: 'Api'
      StageName: 'prod'
      Cors:
        AllowMethods: "'POST,OPTIONS'"
        AllowHeaders: "'content-type'"
        AllowOrigin: "'https://.com'"
```

- `AWS::Serverless::Api` 리소스를 사용하여 API Gateway를 생성
- 여기에서 Cors도 설정해줄 수 있다. 그리고 Function 부분에 Events를 작성해준다

```yaml
Events:
    GetResource:
        Type: Api
        Properties:
        RestApiId:
            Ref: Api
        Path: /post
        Method: post
```

- 여기에서 Path는 실제 API의 경로이고, stage의 이름에 합쳐져서 api gateway endpoint가 나온다.
- 또한 RestApiID는 상위의 `Ref`를 통해서 리소스의 ID를 참조한다.
- Method도 여기서 정의할 수 있다. 만약에 Post 로직이라면 Request Body를 받아야 하는데 이는 `AWS::ApiGateway::Model`를 활용해서 정의한다.

## Model 정의

```yaml
UserModel:
    Type: 'AWS::ApiGateway::Model'
    Properties:
      RestApiId: !Ref Api
      ContentType: application/json
      Name: UserModel
      Schema:
        type: object
        properties:
          user_id:
            type: integer
        required:
          - user_id
```

- ContentType은 데이터의 MIME 타입을 지정
- 해당 예시시는 `user_id` 라는 정수형 필드를 필요로 한다. 실제로 request body는 다음과 같다.

```
{
    "user_id": 0
}
```

- 만약의 nested request 처럼 복잡한 요청 본문을 정의 하라면 다음과 같이 중첩된 객체나 여러 필드를 포함한ㅌ다.

```yaml
UserModel:
  Type: 'AWS::ApiGateway::Model'
  Properties:
    RestApiId: !Ref Api
    ContentType: application/json
    Name: UserModel
    Schema:
      type: object
      properties:
        user_id:
          type: integer
        name:
          type: string
        contact_info:
          type: object
          properties:
            email:
              type: string
              format: email
            phone:
              type: string
        addresses:
          type: array
          items:
            type: object
            properties:
              street:
                type: string
              city:
                type: string
              state:
                type: string
              postal_code:
                type: string
      required:
        - user_id
        - name
```

- 이런 경우의 실제 request_body는 다음과 같다

```
{
  "user_id": 123,
  "name": "John Doe",
  "contact_info": {
    "email": "john.doe@example.com",
    "phone": "123-456-7890"
  },
  "addresses": [
    {
      "street": "123 Main St",
      "city": "Anytown",
      "state": "Anystate",
      "postal_code": "12345"
    },
    {
      "street": "456 Elm St",
      "city": "Othertown",
      "state": "Otherstate",
      "postal_code": "67890"
    }
  ]
}

```

