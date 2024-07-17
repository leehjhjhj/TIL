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

## sqlalchemy와 곁들인 DB롤백

- rollback 어노테이션만 붙이면 끝나는 Spring의 Junit과 다르게 트랜잭션을 직접 관리해주어야 한다.
- 이 때, DB session을 fixture로 관리해주면 아주 편리한데, 이전에 fixture scope이라는 것을 알아야한다.

### Fixture Scope

- fixture의 scope는 fixture가 언제 생성되고 언제 수명을 다하는지 결정한다. 이는 네 가지를 제공한다.
    - function: 기본값이며, 각 테스트 함수마다 fixture를 새로 생성한다.
    - class: 한 클래스의 모든 테스트 메서드가 끝난 뒤 없어진다. 클래스 내의 여러 메서드가 동일한 설정을 공유 할 때 유용하다.
    - module: 모듈 내 모든 테스트가 끝난 뒤 fixture가 없어진다.
    - session: 테스트 세션 전체에서 공유해야 하는 자원(예: 데이터베이스 연결, 서버 등)이 있을 때
- 각 test method마다 데이터가 달라질 수 있으니 본인은 db session fixture에 fuction scope를 주었다.

```python
@pytest.fixture(scope="function")
    def db(self):
        session = SessionMaker()
        with open(self.sql_file_path, 'r') as file:
            sql_statements = file.read()
        for statement in sql_statements.strip().split(';'):
            if statement.strip():
                session.execute(text(statement.strip()))
        yield session
        session.rollback()
        session.close()
```