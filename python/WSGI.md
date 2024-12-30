# WSGI

![출처: 나](https://velog.velcdn.com/images/leehjhjhj/post/0afe8e63-859f-4c2a-8fb6-3d068b221b7a/image.png)

자바진영에서는 톰캣, 아파치와 같은 WAS가 존재하고, 뒤에 실제 로직이 돌아가는 Web Application이 존재합니다. 파이썬 서버는 이와 유사하지만 조금 다른 점이 있습니다.

우선 자바 진영과 비슷하게 nginx, 아파치와 같은 Web Server에서가 존재하여 클라이언트로부터 HTTP 요청을 받거나 정적 파일 서빙, 로드 밸런싱 등등 여러 역할을 수행합니다.

그리고 실제 비지니스 로직이 구현되는 계층인 Web Application까지 동일하나 그 앞에 WSGI라는 것이 존재합니다. **WSGI는 ‘Web Server Gateway Interface’** 로 Web Server와 Python Web Application 사이의 중간 계층에 있으며, WSGI 프로토콜을 구현하여 Web Server와 Python Application 간의 통신을 표준화해줍니다. 하는 일을 요약하자면 웹 서버와 웹 애플리케이션 사이에서 프로세스와 스레드 관리를 해주고 요청 큐잉 등을 담당합니다.

그렇다면 이 WSGI는 왜 필요한 걸까요?

이유는 파이썬의 **GIL** 때문입니다. 파이썬에 관심이 조금 있으시면 이 GIL의 존재를 아실텐데요, 쉽게 말해서 하나의 프로세스에서 하나의 스레드만 사용할 수 있도록 스레드 실행를 동기화해주는 겁니다.

따라서 이 GIL 존재 때문에 Java의 Servlet 컨테이너처럼 단일 프로세스에서 효율적인 멀티스레딩을 하기가 어렵습니다. 특시 CPU bound 작업에서의 멀티스레딩은 불가능하죠.

그래서 이 WSGI는 웹 애플리케이션의 앞에서 웹서버의 요청을 받은 후 HTTP 요청을 파이썬 애플리케이션이 이해할 수 있는 형태로 변환을 해주고, 여러 worker 프로세스와 스레드를 생성하여 요청을 여러 worker에 분배해 파이썬 애플리케이션에서도 동시에 많은 요청을 수행할 수 있도록 해줍니다. 대표적으로 gunicorn과 uWSGI가 존재합니다. 

이와 같은 WSGI의 존재로 django, fastapi와 같이 많은 종류의 파이썬 웹 프레임워크들이 웹 서버와 표준화된 방식으로 통신할 수 있게되고, 애플리케이션 서버와 웹 서버의 역할을 명확히 해줄 수 있습니다.

최근에는 비동기 처리를 위한 ASGI(Asynchronous Server Gateway Interface)도 등장했는데, WebSocket이나 HTTP/2와 같은 비동기 프로토콜을 WSGI보다 더 잘 지원하고, 특히 uvicorn같은 ASGI는 파이썬의 asyncIO 기반으로 설계되어 네이티브하게 비동기 로직을 구현할 수 있어 인기가 많습니다.