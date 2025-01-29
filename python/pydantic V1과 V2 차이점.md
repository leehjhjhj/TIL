### alias, serialization_alias의 차이

본디 V2에서 `alias`와 `serialization_alias` 는 이렇게 사용되었습니다.

```python
class UrlRequest(BaseModel):
		origin_url: str = Field(..., serialization_alias='originUrl', alias='ou')
```

`alias` 는 JSON 데이터를 파싱할 때 다른 이름으로 된 키를 매핑할 때 주로 사용됩니다. 즉, 역직렬화를 할 때 `ou` 라는 키를 받아서 파이썬 객체로 `origin_url` 로 데이터를 사용할 수 있게 해줍니다.

반대로 `serialization_alias` 는 JSON으로 직렬화 할 때 사용할 이름을 지정해줍니다. 데이터 형식을 보면 이해가 빠를 겁니다.

위의 코드에서는 Request의 키에 맞춰서 역직렬화 필드 이름을 정해주는 `alias` 를 ‘ou’로 지정해줬습니다.

```javascript
{
		"ou": "https://naver.com"
}
```

그리고 Response를 해줄 때는 직렬화 필드 이름을 정해주는 `serialization_alias` 에서 지정한 `originUrl` 로 반환합니다.

```javascript
{
		"originUrl": "https://naver.com"
}
```

자, 그럼 V1에서는 어떻게 동작할까요? V1과 V2의 가장 큰 차이점은 `serialization_alias` 가 V1에 없다는 점입니다. 대신 V1의 `alias`가 V2의 `serialization_alias` 의 역할도 수행합니다. 쉽게 말해서 다음과 같은 상황에서는 직렬화도, 역직렬화할 때 모두 originUrl로 변환됩니다.

```python
class UrlRequest(BaseModel):
		origin_url: str = Field(..., alias='originUrl')
```

여기서 조금 유연함을 주는 설정이 `allow_population_by_field_name` 설정인데, 이를 True로 설정하면 alias와 원래 필드명 모두를 사용하여 모델을 생성할 수 있게 해줍니다. 즉 다음과 같은 경우에서는 역직렬화(JSON → 파이썬 객체)시 원래 필드명인 `origin_url`은 물론이고 alias 설정인 `originUrl` 로도 데이터를 받을 수 있습니다.

```javascript
data1 = {
		"originUrl": "https://naver.com"
}

data2 = {
		"originUrl": "https://naver.com"
}
```

그리고 Response는 여전히 `alias` 를 사용해서 변환시킵니다.

```json
{
		"originUrl": "https://naver.com"
}
```

### Config 선언 차이

`allow_population_by_field_name` 와 같은 설정을 해주는 방식 또한 다른데, V1에서는 내부 클래스를 통해서 설정해주었습니다.

```python
class MyModel(BaseModel):
    class Config:
        # 기본적인 설정들
        allow_population_by_field_name = True  # alias와 원래 필드명 모두 허용
        allow_mutation = True  # 모델 인스턴스의 필드값 수정 허용 
        extra = 'forbid'  # 추가 필드 처리 ('allow', 'forbid', 'ignore')
        
        # 검증 관련 설정
        validate_assignment = True  # 필드 할당 시에도 검증 수행
        validate_all = False  # 기본값도 검증할지 여부
        
        # 직렬화 관련 설정
        orm_mode = False  # ORM 객체를 직접 파싱할지 여부
        arbitrary_types_allowed = False  # 임의의 타입 허용 여부
        json_encoders = {}  # 커스텀 JSON 인코더
        
        # 기타 설정
        title = None  # 스키마 제목
        schema_extra = {}  # 추가적인 JSON 스키마 속성
```

반면에 V2에서는 저러한 방식도 가능하지만 `ConfigDict` 을 사용해서 간편하게 설정합니다.

```python
from pydantic import BaseModel, ConfigDict

class UserModel(BaseModel):
    name: str
    age: int
    
    # Config 클래스 대신 model_config 사용
    model_config = ConfigDict(
        str_to_lower=True,
        str_strip_whitespace=True,
        validate_assignment=True,
        populate_by_name=True,  # V1의 allow_population_by_field_name와 동일
        extra='forbid',
        frozen=False,  # V1의 allow_mutation=True와 동일
        from_attributes=True,  # V1의 orm_mode=True와 동일
    )
```