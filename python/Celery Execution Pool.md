## Celery Execution Pool

### Prefork Worker Pool

- Celery의 기본 실행 워커 모델, 여러 개의 worker 프로세스를 미리 생성해서 작업을 처리
- Master Process가 여러 개의 Worker Process를 미리 생성
- 각 Worker는 독립적인 프로세스로 실행되어 메모리 격리를 보장
- 다중 프로세스 기반으로 CPU 바운드 작업에 적합하다.
- `--concurrency` 옵션으로 worker 프로세스의 수를 지정 가능
    - 보통 CPU 코어 수를 고려하여 prefork worker수 설정

```bash
celery -A your_app worker --concurrency=4 --prefetch-multiplier=1
```

### Eventlet & Gevent Worker Pool

- 둘다 경량 스레드인 greenlet 기반 비동기 처리
- 동기 함수를 비동기 이벤트 루프 위에 실행하기 위해 monkey patching 실행

```bash
celery worker --pool=eventlet or gevent
```

### 그 외..

- 단일 프로세스에서 실행하는 Solo Pool과 멀티 스레드 기반의 Thread Pool 이있는데 별로 쓸 일이 없을 것 같아서 스킵

