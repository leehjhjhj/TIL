## asyncio와 ThreadPoolExecutor 차이

- **Asyncio**
    - 단일 스레드 내에서 코루틴들을 이벤트 루프로 관리
    - `await` 키워드로 실행 중단점을 명시적으로 표시
    - 실제 병렬 처리는 아니지만, I/O 작업에서 효율적인 동시성 제공
- ThreadPoolExecutor
    - 실제 여러 개의 OS 스레드를 생성하여 관리
    - 각 작업이 별도의 스레드에서 실행
    - 스레드 풀을 통해 리소스를 효율적으로 관리
- 많은 I/O 작업을 처리할 때는 asyncio 사용
- 블로킹 I/O나 오래 걸리는 작업은 ThreadPoolExecutor 사용
- CPU 집약적 작업은 ProcessPoolExecutor 사용
- 정리하자면 asyncio는 하나의 스레드에서 코루틴들을 이벤트 루프로 관리하여, 효율적인 동시성으로 여러 작업을 수행하는 것이고, ThreadPoolExecutor은 직접 실제 여러 개의 OS 스레드를 생성, 스레드 풀을 생성해서 여러 작업을 수행하는 것