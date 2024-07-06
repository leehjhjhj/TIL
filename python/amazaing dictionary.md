# Amazing Dictionary

```python
dict1 = {"name": "현제", "old": 27, "job": "Developer"}
dict2 = {"address": "Seoul"}
dict3 = {**dict1, **dict2}
print(dict3)# {'name': '현제', 'old': 27, 'job': 'Developer', 'address': 'Seoul'}
```

- 파이썬 딕셔너리는 `**`로 딕셔너리를 언패킹하고, `,`로 풀어진 딕셔너리를 합칠 수 있다.

```python
dict4 = {"address": "homeless"}
dict5 = {**dict1, **dict2, **dict4}
print(dict5) # {'name': '현제', 'old': 27, 'job': 'Developer', 'address': 'homeless'}

dict6 = {**dict4, "status": "hungry"}
print(dict6) # {'address': 'homeless', 'status': 'hungry'}
```
- 단 같은 키가 들어오면 후에 들어온 딕셔너리의 키값이 우선된다.

## 문자열의 각 문자 개수 세기

```python
from collections import Counter

count = Counter("hello world")
print(count)  # Counter({'l': 3, 'o': 2, 'h': 1, 'e': 1, ' ': 1, 'w': 1, 'r': 1, 'd': 1})
```

- Counter를 사용하면 매우 쉽게 알파벳 개수를 뽑아 올 수 있다.

## 잊을 법한 문법

```python
dict6.update({"car": None, "computer": "Mac"}) 
print(dict6) # {'address': 'homeless', 'status': 'hungry', 'car': None, 'computer': 'Mac'}
```

- fastapi 공식문서에서 update를 통해서 데이터를 갱신하는 예시가 있다. 잊지 말도록하자.

