## Callable

- Callale은 **호출할 수 있는 객체**를 말한다.
- 즉, 객체 뒤에서 괄호 `()` 를 붙여서 실행할 수 있는 모든 것이 Callable이다.
    - 일반 함수(def), `__call__` 이 있는 class, 람다 함수 등

## Callable 타입 힌팅

- (`func: Callable[[int], int], value: int` ) 와 같은 타입힌팅이 가능하다.
- 첫번째 `[int]`는 함수의 입력 매개변수 타입들을 말한다.
- 두번째 `int` 는 함수의 반환 타입이다. 예시를 살펴보자.

```python
from typing import Callable

def execute_function(func: Callable[[int], int], value: int) -> int:
    return func(value)

def double(x: int) -> int:
    return x * 2

square = lambda x: x * x

result1 = execute_function(double, 5)  # OK: 10
result2 = execute_function(square, 5)  # OK: 25
```

- `execute_function`는 callable은 객체를 받는데, 매개변수는 int 한 개이고, 반환은 int 타입 하나를 반환한다.
- `double`과 `square` 은 모두 int 하나를 받아 int 하나를 반환하기 때문에 가능하다.
- 만약 여러 개의 매개 변수를 받는다면 다음과 같이 할 수 있다.

```python
def process_two_numbers(func: Callable[[int, int], int], x: int, y: int) -> int:
    return func(x, y)
 
def add(a: int, b: int) -> int:
    return a + b

result = process_two_numbers(add, 5, 3)  # OK: 8
```

- 다양한 타입 또한 표기할 수 있다.

```python
from typing import Callable

def validate(func: Callable[[str], bool], text: str) -> bool:
    return func(text)

# 여러 타입의 매개변수를 받는 함수
def complex_operation(
    func: Callable[[str, int, bool], float],
    text: str,
    number: int,
    flag: bool
) -> float:
    return func(text, number, flag)

# 매개변수가 없는 함수
def execute_simple(func: Callable[[], str]) -> str:
    return func()

```

- `validate` 은 문자열을 받아서 불리언을 반환하는 함수이다.
- `complex_operation` 은 string, int, bool 등 다양한 타입의 매개변수를 받는 함수이다.
- `execute_simple` 처럼 매개변슈가 없는 함수도 표현이 가능하다.
- 실제로 활용해보면 다음과 같이 활용이 가능하다.

```python
from typing import Callable, List

# 리스트의 각 요소에 함수를 적용
def apply_to_list(func: Callable[[int], int], numbers: List[int]) -> List[int]:
    return [func(num) for num in numbers]

# 데이터 변환
numbers = [1, 2, 3, 4, 5]
doubled = apply_to_list(lambda x: x * 2, numbers)  # [2, 4, 6, 8, 10]
squared = apply_to_list(lambda x: x ** 2, numbers)  # [1, 4, 9, 16, 25]

# 필터링 함수
def create_filter(threshold: int) -> Callable[[int], bool]:
    def filter_function(x: int) -> bool:
        return x > threshold
    return filter_function

filter_above_3 = create_filter(3)
filtered_numbers = list(filter(filter_above_3, numbers))  # [4, 5]
```

## 그래서 보통 어디에 쓰이는가?

- 데코레이터나 콜백함수를 정의할 때 쓰인다.
    
    ```python
    from typing import Callable, List
    
    def process_items(items: List[int], 
                     success_callback: Callable[[int], None],
                     error_callback: Callable[[str], None]) -> None:
        for item in items:
            try:
                result = item * 2
                success_callback(result)
            except Exception as e:
                error_callback(str(e))
    
    # 사용
    def on_success(result: int) -> None:
        print(f"성공: {result}")
    
    def on_error(error: str) -> None:
        print(f"에러: {error}")
    
    process_items([1, 2, 3], on_success, on_error)
    ```
    
    - `process_items` 아이템들과 성공시 호출할 함수, 실패시 호출할 함수를 매개변수로 받는다.
    - 이 때, `on_success` 와 `on_error` 를 넣을 때 타입힌팅을 사용할 수 있다.
- 그 이외 매개변수에 함수를 받는 모든 곳에서 callable 타입힌팅을 사용할 수 있다.
    - 의존성 주입
    - 이벤트 핸들러 등등
- Callable를 사용함으로써 매개변수로 받는 함수의 매개변수의 타입, 반환 값의 타입을 알 수 있기 때문에, 코드를 보고도 결과가 예측이 가능해진다.
- 린트를 사용하여 더욱 안전하게 파이썬를 사용할 수 있다.
- 그런데 매개변수 부분이 너무 길어져서 가독성 면은 잘 모르겠다. 트레이드 오프는 당연히 있는 것 같다.