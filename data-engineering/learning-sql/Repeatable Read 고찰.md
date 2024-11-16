## Repeatable Read 고찰

- 데이터베이스의 격리수준에서 가장 널리 사용되는 Repeatable Read에 대해서 더욱 고찰해보았다.
- Repeatable Read 격리 수준에서는 첫 번째 트랜잭션에서 읽은 데이터의 스냅샷을 유지한다.

```sql
CREATE TABLE deposits (
    id INT PRIMARY KEY AUTO_INCREMENT,
    hash VARCHAR(50),
    amount INT,
    status VARCHAR(20)
);

-- 테스트 데이터 삽입
INSERT INTO deposits (hash, amount, status) VALUES 
    ('user1', 1000, 'PENDING'),
    ('user2', 1000, 'PENDING')
```

- 하지만 자신의 트랜잭션 내 변경사항은 볼 수 있다.
- 이러한 상태에서 해당 SQL을 실행해보자.

```sql
START TRANSACTION;
    SELECT * FROM deposits WHERE status = 'PENDING';
    UPDATE deposits SET status = 'SUCCESS' WHERE hash = 'user1';
    SELECT * FROM deposits WHERE hash = 'user1';
COMMIT;
```

> '1', 'user1', '1000', 'SUCCESS'
'2', 'user2', '1000', 'PENDING'
> 
- 마지막 SELECT 절에서 보인 결과는 다음과 같다.
- 즉, 같은 트랜잭션 내에서의 변경사항은 바로바로 적용이 되어 SELECT 절에 보이게 된다.
- Save Point도 마찬가지이다.

```sql
START TRANSACTION;
    SAVEPOINT sp1;
        SELECT * FROM deposits WHERE status = 'PENDING';
        UPDATE deposits SET status = 'SUCCESS' WHERE hash = 'user1';
        COMMIT;
    
    SAVEPOINT sp2;
		SELECT * FROM deposits WHERE hash = 'user1';
        COMMIT;
COMMIT;
```

- 이것도 위와 같은 결과를 보이게 된다.
- 하지만 다른 트랜잭션에서 UPDATE한 것은 어떻게 보일까?

```sql
-- 트랜잭션 1
START TRANSACTION;  -- 이 시점의 스냅샷을 사용
    SELECT * FROM deposits WHERE status = 'PENDING';
    -- 결과: user1, user2 두 건이 PENDING 상태로 조회

    -- 이 시점에 트랜잭션 2가 실행되어 변경을 수행
    
    SELECT * FROM deposits WHERE hash = 'user1' AND status = 'PENDING';
    -- 여전히 user1, user2 두 건이 PENDING 상태로 조회됨
    -- 트랜잭션 2의 변경사항은 보이지 않음
COMMIT;

-- 트랜잭션 2
START TRANSACTION;
    UPDATE deposits SET status = 'SUCCESS' hash = 'user1';
COMMIT;
```

- 하지만 이렇게 SQL을 작성하게 되면 메인 트랜잭션 속의 트랜잭션에서 실행된 것이라 ‘**자신의 트랜잭션 내 변경사항은 볼 수 있다**‘ 때문에 user1의 상태가 `SUCCESS`로 보인다. 제대로 된 테스트를 위해 Python code를 짜보자

```python
import threading
import time
from sqlalchemy import create_engine, text

# DB 연결 설정
engine = create_engine('mysql+pymysql://root:0000@localhost/test')

def transaction1():
    with engine.connect() as conn:
        with conn.begin():
            # 첫 번째 SELECT
            result1 = conn.execute(text("SELECT * FROM deposits WHERE status = 'PENDING'"))
            print("Transaction 1 - First SELECT:", result1.mappings().all())
            
            # 트랜잭션2를 위한 잠시 대기
            time.sleep(2)
            
            # 두 번째 SELECT
            result2 = conn.execute(text("SELECT * FROM deposits WHERE status = 'PENDING'"))
            print("Transaction 1 - Second SELECT:", result2.mappings().all())

def transaction2():
		time.sleep(1)
    with engine.connect() as conn:
        with conn.begin():
            conn.execute(text("UPDATE deposits SET status = 'SUCCESS' WHERE hash = 'user1'"))
            print("Transaction 2 - UPDATE executed")

def run_test():
    # 테스트 데이터 초기화
    with engine.connect() as conn:
        with conn.begin():
            conn.execute(text("DELETE FROM deposits"))
            conn.execute(text("""
                INSERT INTO deposits (hash, amount, status) 
                VALUES ('user1', 1000, 'PENDING'), ('user2', 1000, 'PENDING')
            """))

    thread1 = threading.Thread(target=transaction1)
    thread2 = threading.Thread(target=transaction2)
    
    thread1.start()
    thread2.start()
    
    thread1.join()
    thread2.join()

if __name__ == "__main__":
    run_test()
```

- 코드 내용은 다음과 같다.
    - 두 개의 별도 스레드에서 각각 트랜잭션을 실행
    - 트랜잭션 1에서 SELECT 실행 후 잠시 대기
    - 그 동안 트랜잭션 2에서 UPDATE 실행
    - 트랜잭션 1에서 다시 SELECT 실행
- 이렇게 별도의 스레드에서 트랜잭션을 진행하게 되면 다음과 같은 결과를 얻을 수 있다.

```python
Transaction 1 - First SELECT: [{'id': 24, 'hash': 'user1', 'amount': 1000, 'status': 'PENDING'},
{'id': 25, 'hash': 'user2', 'amount': 1000, 'status': 'PENDING'}]
Transaction 2 - UPDATE executed
Transaction 1 - Second SELECT: [{'id': 24, 'hash': 'user1', 'amount': 1000, 'status': 'PENDING'},
{'id': 25, 'hash': 'user2', 'amount': 1000, 'status': 'PENDING'}]
```

- Repeatable Read 격리 수준에서는 트랜잭션 1의 두 번째 SELECT도 첫 번째와 동일한 결과를 얻게 된다.
- 트랜잭션 2의 변경사항은 트랜잭션 1이 커밋되고 **새로운 트랜잭션**이 시작될 때까지 반영되어 보이지 않는다.
- 하지만 두번째 SELECT에서 `FOR UDPATE` 를 붙인다면 다음과 같은 결과를 얻을 수 있다.

```python
Transaction 1 - First SELECT: [{'id': 26, 'hash': 'user1', 'amount': 1000, 'status': 'PENDING'},
{'id': 27, 'hash': 'user2', 'amount': 1000, 'status': 'PENDING'}]
Transaction 2 - UPDATE executed
Transaction 1 - Second SELECT: [{'id': 27, 'hash': 'user2', 'amount': 1000, 'status': 'PENDING'}]
```

- 이렇게 두번째 SELECT에도 반영된 결과과 보이는 이유는 `SELECT FOR UPDATE`는 Repeatable Read 격리 수준에서도 최신 데이터를 읽어오는 특별한 케이스이기 때문이다.
- 해당 이유는 다음과 같다.
    - SELECT FOR UPDATE는 해당 row에 대해 쓰기 잠금(write lock)을 획득하려고 시도
    - 쓰기 잠금을 획득하기 위해서는 해당 row의 최신 상태를 알아야 함
    - 따라서 트랜잭션 시작 시의 스냅샷이 아닌, 현재 커밋된 데이터를 읽음
    - 이러한 동작은 데이터 일관성과 동시성 제어를 위한 것
        - 데이터를 업데이트하기 전에 **최신 상태**를 확인하고 싶을 때
        - 여러 트랜잭션에서 **동시에 같은 데이터를 수정하는 것을 방지**하고 싶을 때