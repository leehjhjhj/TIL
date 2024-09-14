# DB 동시 수정 피하기

# 문제상황

- 송금과 관련된 민감한 데이터의 동시성을 제어하고 일관성을 유지해야하는 요구사항이 생겼다.

```
transfer_id | transfer_status
-------------|----------------
1            | PENDING
```

- `transfer` 이라는 테이블의 `transfer_status`의 상태에 따라서 송금 여부가 결정된다고 가정하자.
- 만약 동일한 DB row에 대해서 여러 트랜잭션이 수정을 시도하는 일이 빈번하다면 이런 문제가 발생한다.
    - `트랜잭션 A`: 송금을 처리하는 트랜잭션으로 transfer_id = 1의 transfer_status가 PENDING인 경우에 송금을 수행
    - `트랜잭션 B`: 동시에 실행되는 또 다른 송금 처리 트랜잭션으로 동일한 transfer_id = 1의 송금 상태를 확인하고 송금을 수행
    - 트랜잭션 A가 transfer_id 1의 `transfer_status`을 조회하여 pending임을 확인하고 송금을 진행하는데, 그 사이에 트랜잭션 B가 transfer_id 1의 데이터를 조회한다.
    - 하지만 송금이 진행 중이라 여전히 transfer_id 1의 상태는 `pending`이고, 때문에 트랜잭션 A가 완료되기 전에 B도 송금을 수행한다.
    - 결국 중복으로 처리되어 받는 이는 돈을 두번 받게된다!
- 이런 상황에서 동시성 제어를 위해 낙관적 락과 비관적 락이 존재하는데, 나의 경우는 실제 송금이 이루어지는 로직에 사용되는 테이블에 대한 락을 걸어야 했기에 조금더 보수적인 비관적 락을 선택하였다.
- 그중에서도 행 수준 잠금을 택했는데 실제 테이블에는 `transfer_status` 말고도 이와함께 결정되는 다른 테이블들이 많았기 때문이다.
    - 가정을 하자면 만약 환불과 관련된 송금 건이라면 `transfer_status`가 변경되고 동시에 같은 row의 `refund_status` 도 한 트랜잭션에 변경되어야 한다.
    - `refund_status`만을 조회하는 다른 트랜잭션이 존재할 수 있기에, row 단위로 한 트랜잭션에서 데이터가 일관성있게 변경되어야 했다.

# 내가 문제를 해결한 방법

- 행 수준 잠금은 DB 수준에서 쉽게 구현이 가능한데 바로 `SELECT FOR`이다.
- 레디스를 활용한 방법도 있지만, 딱히 분산환경도 아니고 조금 성능이 떨어지더라도 확실한 선점을 하기 위해서다.

## select for update

- 행 단위로 동시 접근을 제어한다.
- 먼저 락을 얻은 트랜잭션은 트랜잭션이 끝날 때까지 해당 행을 선점한다.
- 락을 얻지 못한 트랜잭션은 락을 얻은 트랜잭션이 끝날 때(commit) 까지 대기한다.

## 내가 구현한 방법

- 우선 송금을 위해 `pending` 중인 transfer 데이터를 불러올 때 select for을 사용한다.

```python
.where(
    Transfer.transfer_status == TransferStatus.PENDING
    ).with_for_update()
```

- 기존 송금 로직에서는 `transfer_status`가 `pending`인 것을 `success`로 변경해주고 실제 송금을 진행한다. 앞선 코드에서 `with_for_update()`를 통해 `pending`인 transfer 데이터를 선점하고 로직을 수행한다.
- 만약 한 로직에서 `transfer_status`가 `pending`인 transfer 데이터를 불러오고, 같은 row에 환불을 위한 특정 컬럼을 업데이트 한 뒤 SQS나 SNS로 메시지를 보내야하는 요구사항이 있다고 가정하자.
- 만약 송금 로직에서 row를 선점하고 있을 때, 위의 환불 로직에 의해 새로운 트랜잭션이 해당 row에 접근을 한다.
- 환불 로직은 WHERE 조건에서 `transfer_status`가 `pending`인 데이터만 불러오고, 다른 컬럼의 업데이트를 성공한 데이터만 메시지를 보내야하기 때문에 update시 rowcount를 검사한다.

```python
def update_refund(self, dto):
        query = text("""
        UPDATE
            transfer       
        SET
            refund_status = 'pending',
            refund_amount = :refund_amount
        WHERE
            transfer_id = :transfer_id
            AND status = "pending"
        """).bindparams(**dto.model_dumps())
        result = self._db.execute(query)
        return result

result = uow.repo.update_refund(dto)
if result.rowcount > 0:
    event_data = SqsEvent(
        #...
    )
    try:
        send_to_sqs(event_data)
```

- 이렇게 다른 트랜잭션에서의 선점을 고려해서 로직 실행 조건을 검사하거나 update가 됐는지 안됐는지 확인하는 로직이 필요하다.

# SELECT FOR UPDATE의 주의점

- select for update를 걸면 UPDATE는 트랜잭션은 대기해야 하지만, 일반적인 SELECT는 데이터를 읽을 수 있다.
- 이는 읽기 작업이 쓰기 작업을 방해하지 않기 때문
- 때문에 정합성이 중요한 로직에서 SELECT 시에는 `SELECT FOR UPDATE`를 사용하여 트랜잭션이 완료(COMMIT 또는 ROLLBACK)될 때까지 기다려야 한다.
- 격리 수준 설정에 따라 일반 SELECT도 막을 수 있다. 예를 들어, READ COMMITTED 격리 수준에서는 SELECT FOR UPDATE가 선점하고 있을 때 일반 SELECT 문으로 읽기가 가능하지만 SERIALIZABLE 격리 수준에서는 읽지 못한다.