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

## 날짜 Fix

- `freezegun`을 사용하면 `datetime.time()`와 같은 동적 시간을 고정시킬 수 있다.

```bash
pip install freezegun
```

```python
from freezegun import freeze_time
import datetime

@freeze_time("2024-07-18")
def test_date():
    assert datetime.datetime.now() == datetime.datetime(2024, 7, 18)
```

- 이렇게 freeze_time 데코레이션으로 시간을 고정시킬 수 있다.

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
```

### rollback

- 본격적으로 롤백에 대해서 알아보자. 지속적으로 테스트를 진행하다보면 미리 준비된 fixture sql문을 실행한다.

```python
class TestMatching:
    sql_file_path = 'tests/fixture.sql'
```

- 그리고 해당 sql문을 읽고 `excute`한 DB 세션을 yield한다. 이것을 fixture로 생성한다.

```python
@pytest.fixture(scope="function")
    def db(self):
        session = SessionLocal_Write()
        with open(self.sql_file_path, 'r') as file:
            sql_statements = file.read()
        for statement in sql_statements.strip().split(';'):
            if statement.strip():
                session.execute(text(statement.strip()))
        yield session
        session.rollback()
        session.close()
```

- `rollback()`을 하지 않으면 테스트 하나 당 SQL 문이 반복적으로 돌기 때문에 pk 중복 등 테스트가 정상적으로 작동하지 않는다.
- 하지만 더 중요한 것은 테스트 내에서 `commit()` 이다. 저장이나 수정 로직이 정상적으로 실행됐는지 알기 위해서는 commit을 한 뒤의 결과를 알아야 한다.
- 하지만 앞서 탐구했듯, 한 `trasaction`에서는 하나의 commit과 rollback 만 허용되고, 이 이후에는 트랜잭션을 종료시킨다.
- 따라서 commit이 반복적으로 이뤄지는 로직에 `with session.begin_nested():` 을 추가하여 nested trasaction을 생성한다.
- 이렇게 되면 transaction 하나에 하나의 commit이 보장되고, 테스트를 안정적으로 할 수 있다.

## DB 동기화

- 개발 DB라도 데이터를 삽입 및 롤백을 하다보면 데이터가 겹칠 수 있어 테스트용 DB를 사용해야한다.
- 띠라서 도커에 mysql을 띄우고, 개발 DB의 스키마를 사용해서 테스트DB를 만들어준다.
- 이 때, alembic을 사용하지 않더라도 동기화를 시킬 수 있어야 했다. 따라서, 자동화 코드를 만들어 사용한다.

```python
SCHEMA_DUMP_FILE = 'schema.sql'

def dump_schema():
    dump_command = f"mysqldump -h {DEV_DB_HOST} -u {DEV_DB_USER} -p{DEV_DB_PASSWORD} --no-data {DEV_DB_NAME} > {SCHEMA_DUMP_FILE}"
    subprocess.run(dump_command, shell=True, check=True)
    print("스키마 덤프 완료")

def apply_schema():
    with open(SCHEMA_DUMP_FILE, 'r') as file:
        schema_sql = file.read()

    connection = pymysql.connect(
        host=TEST_DB_HOST,
        user=TEST_DB_USER,
        password=TEST_DB_PASSWORD,
        database=TEST_DB_NAME
    )

    try:
        with connection.cursor() as cursor:
            for statement in schema_sql.split(';'):
                if statement.strip():
                    cursor.execute(statement)
        connection.commit()
        print("스키마 적용 완료")
    finally:
        connection.close()

if __name__ == "__main__":
    dump_schema()
    apply_schema()
```

- 코드는 간단하다. 개발 디비의 스키마를 `subprocess` 를 사용해 mysqldump 명령어로 DDL을 만든다.
- 그리고 스키마 파일을 읽어 도커에 띄어져있는 디비에 스키마를 적용시킨다.
- 테이블 변경이 발생했을 때 해당 스크립트만 실행하면 되기 때문에 간편하다.
- 하지만 덤프 과정이 오래 걸려서 효율이 떨어진다면 alembic을 통해서 버전관리를 하는 것을 검토해보아야겠다.