# 나만의 Pytest

- sam의 test를 위한 pytest에 대해서 알아본 것을 정리한다.

## pytest의 경로 찾기
- 만약 src 디렉토리와 test 파일들의 위치가 다르다면 pytest.ini을 활용할 수 있다.

```
my_project/
├── src/
│   ├── my_module.py
│   └── ...
├── tests/
│   ├── test_my_module.py
│   └── ...
└── pytest.ini

```

- 경로가 다음과 같을 때 이런 식으로 지정해준다. =

```ini
[pytest]
pythonpath = src
```

- pythonpath를 직접 환경변수에 넣지 않고 pytest.ini을 사용하면 편하게 pypath를 지정할 수 있다.
- 이런 식으로 명시하면 `module not found error` 가 발생하지 않고, 무사히 pytest가 imoprt 할 수 있다.

