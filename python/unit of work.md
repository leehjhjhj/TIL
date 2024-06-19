# Unit Of Work
- `Architecture patterns with python` 에서의 UoW는 이렇게 정의된다.
    - 작업 단위의 패턴, 즉, 원자적 연산이라는 개념에 대한 추상화이다.
    - 작업의 트랜잭션 단위를 정하고, 서비스 계층의 직접적인 데이터 베이스 요청으로 부터 분리시킬 수 있다.

```python
from sqlalchemy.orm import Session

class UnitOfWork:
    def __init__(self, db: Session, repository_cls):
        self._db = db
        self._repository_cls = repository_cls

    def __enter__(self):
        self.repo = self._repository_cls(self._db)
        return self

    def __exit__(self, *args):
        self._db.rollback()
        self._db.close()

    def commit(self):
        try:
            self._db.commit()
        except SQLAlchemyError:
            self.rollback()
            raise

    def rollback(self):
        self.db._rollback()
```
- 책에서 나온 UoW를 조금 변형에서, 레포지토리 클래스를 초기화 때 받아 여러 곳에서 재사용 가능하게 만들었다.