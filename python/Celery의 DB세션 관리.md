[Celery Tasks: A Guide to SQLAlchemy Session Handling](https://celery.school/sqlalchemy-session-celery-tasks)

오늘도 Celery입니다. celery 코드를 짜다보면 세션 관리에 대해서 고민이 들 때가 있는데 매우 좋은 글이 있어서 내용을 요약해보았습니다.

Celery task 내에서 DB 세션을 열고 닫는 방법은 여러 가지인데, 이 방법을 알기 전까지는 이렇게 사용했습니다.

```python
@contextmanager
def get_db_reader_for_celery():
    try:
        db = SessionLocal()
        yield db
    finally:
        db.close()
 
# celery_task.py

@app.task(bind=True)
def my_task(self):
    with get_db_reader_for_celery() as session:
            ...
```

이렇게 람다에서 사용하듯이 db session을 yield해서 받은 뒤 context manager에서 나가면 알아서 `db.close()` 로 세션을 닫아주고, 에러 발생시 `db.rollback()` 까지 시켜주는 간편한 코드입니다.

그런데 서브 태스크를 비롯한 모든 태스크에 매번 저렇게 세션을 초기화하니깐, 반복되는 코드를 작성해야 하는 상황이 발생했습니다.

그래서 아티클에서는 아예 `@app.task` 로 상속받는 Celery Task Class를 커스터마이징해서 태스크가 시작할 때, 끝날 때 DB 세션을 자동으로 만들어주는 방법을 소개하고 있습니다.

```python
class MyTask(celery.Task):
    def __init__(self):
        self.sessions = {}

    def before_start(self, task_id, args, kwargs):
        self.sessions[task_id] = Session(...)
        super().before_start(task_id, args, kwargs)

    def after_return(self, status, retval, task_id, args, kwargs, einfo):
        session = self.sessions.pop(task_id)
        session.close()
        super().after_return(status, retval, task_id, args, kwargs, einfo)

    @property
    def session(self):
        return self.sessions[self.request.id] 
```

이렇게 `celery.Task` 를 상속 받고, `before_start` 과 `after_return` 에서 task_id로 세션을 관리해줍니다. 

> 여기에서 `self.request.id`와 콜백함수 `before_start` 과 `after_return`에서 받는 task_id는 항상 동일한 값 입니다.
> 

해당 클래스를 `base` 를 통해서 태스크에 연결시켜줍니다.

```python
@app.task(bind=True, base=MyTask)
def my_task(self):
	...
	self.session.add()
	...
```

그리고 태스크 내에서는 self를 통해서 session에 접근이 가능합니다. 이렇게 해서 Celery에서 DB 세션을 간편하게 관리를 할 수 있습니다.