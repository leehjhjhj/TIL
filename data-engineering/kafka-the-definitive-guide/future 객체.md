# Future 객체

## java의 future 인터페이스

- Future 인터페이스는 Java 5에서 java.util.concurrent 패키지의 일부로 도입
- 비동기 결과를 나타냄
- `boolean isDone()`: 작업이 완료되었는지 여부
- `boolean cancel(boolean mayInterruptIfRunning)`: 작업 취소 시도, mayInterruptIfRunning이 true면 이미 실행 중인 작업일 중단하려고 시도
- `V get() throws InterruptedException, ExecutionException`: 작업의 결과 반환
    - 작업이 완료될 때 까지 블로킹된다.
    - 예외가 발생하면 `ExecutionException` 발생
- `V get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException`: 지정된 시간 동안만 작업의 결과를 기다린다.
    - 시간이 지나면 `TimeoutException` 예외 터뜨림

```java
NewTopic newTopic = new NewTopic(topicName, partitions, replicationFactor);

CreateTopicsResult result = adminClient.createTopics(Collections.singleton(newTopic));

// 특정 토픽의 생성 결과에 대한 Future 객체 얻기
KafkaFuture<Void> future = result.values().get(topicName);

future.get(10, TimeUnit.SECONDS); 
```

- 예시는 AdminClient를 사용해 kafka 토픽을 생성한다.
- `createTopics`는 `CreateTopicsResult` 객체를 반환하고, 토픽 생성 결과에 대한 `KafkaFuture`를 얻는다.
- `future.get()`을 호출하여 작업 완료를 기다리고 결과를 얻는다.

## CompletableFuture

- java 8에서 생긴 클래스로 작업완료시 콜백을 등록할 수 있고, 여러 비동기 작업을 연결, 조합할 수도 있다.

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    // 시간이 걸리는 작업
    return "작업 결과";
});

future.thenAccept(result -> System.out.println("결과: " + result));
```

## Python의 Future 객체

- 파이썬의 asyncio 라이브러리에도 future 객체가 존재한다.
- 세가지 상태가 존재
    - Pending: 아직 완료되지 않은 상태
    - Done: 정상적으로 완료된 상태
    - Cancelled: 취소된 상태
- asnycio의 코루틴은 내부적으로 Future 객체로 래핑된다. 따라서 `await`을 사용해서 코루틴이나 Future 객체를 기다릴 수 있다.

```python
import asyncio

async def main():
    # Future 객체 생성
    future = asyncio.Future()

    # 결과 설정
    future.set_result("Hello, Future!")

    # 결과 확인
    result = await future
    print(result)  # 출력: Hello, Future!

    # 상태 확인
    print(future.done())  # 출력: True

    # 콜백 추가
    future.add_done_callback(lambda f: print("Future is done!"))

    # 새로운 Future 생성
    another_future = asyncio.Future()

    # 취소
    another_future.cancel()
    print(another_future.cancelled())  # 출력: True

    try:
        await another_future
    except asyncio.CancelledError:
        print("Future was cancelled")
asyncio.run(main())
```
- `set_result(result)`: Future의 결과를 설정
- `done()`: Future가 완료되었는지 확인
- `add_done_callback(callback)`: Future가 완료되었을 때 실행될 콜백을 추가
- `cancel()`: Future를 취소
- `cancelled()`: Future 취소 확인
- 일반적으로 직접 future 객체를 생성하기보다는 `asyncio.ensure_future()` 또는 `asyncio.create_task()`를 사용하여 코루틴을 Future로 변환
    - `asyncio.ensure_future()`로 코루틴을 Future 객체로 변환
    - `asyncio.gather()`를 사용하여 모든 Future 객체들이 완료될 때까지 기다린다.

```python
async def fetch_and_process(url):
    data = await fetch_data(url)
    result = await process_data(data)
    return result

async def main():
    urls = ['http://example.com', 'http://example.org', 'http://example.net']
    
    # Future 객체들의 리스트 생성
    futures = [asyncio.ensure_future(fetch_and_process(url)) for url in urls]
    
    # 모든 Future 객체들이 완료될 때까지 대기
    results = await asyncio.gather(*futures)
    
    for result in results:
        print(result)

asyncio.run(main())
```

- 하지만 future 객체의 서브 클래스인 `Task`가 더 많이 쓰인다.
    - future 객체는 수동으로 결과를 설정해야하지만 task는 코루틴 실행 결과를 자동으로 처리
    - Task는 생성되는 즉시 이벤트 루프에 의해 자동으로 스케줄링된다.
    - `asyncio.create_task()`이라는 명확한 고수준 API가 있음
    - Task는 코루틴 내부에서 발생한 예외를 알아서 핸들링한다.   

```python
import asyncio
import aiohttp

async def fetch_url(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.text()

async def process_url(url):
    print(f"Fetching {url}")
    content = await fetch_url(url)
    print(f"Processed {url}: {len(content)} characters")
    return len(content)

async def main():
    urls = [
        "http://example.com",
        "http://example.org",
        "http://example.net"
    ]
    
    # 방법 1: Task 리스트 생성
    tasks = [asyncio.create_task(process_url(url)) for url in urls]
    
    # 모든 Task 완료 대기
    results = await asyncio.gather(*tasks)
    print(f"All URLs processed. Total characters: {sum(results)}")

    # 방법 2: as_completed 사용 - 다 된 순서대로 보여줌
    tasks = [asyncio.create_task(process_url(url)) for url in urls]
    for completed_task in asyncio.as_completed(tasks):
        result = await completed_task
        print(f"A task completed with result: {result}")

asyncio.run(main())
```