# WSGI

![출처: 나](https://velog.velcdn.com/images/leehjhjhj/post/0afe8e63-859f-4c2a-8fb6-3d068b221b7a/image.png)

자바진영에서는 톰캣, 아파치와 같은 WAS가 존재하고, 뒤에 실제 로직이 돌아가는 Web Application이 존재합니다. 파이썬 서버는 이와 유사하지만 조금 다른 점이 있습니다.

우선 자바 진영과 비슷하게 nginx, 아파치와 같은 Web Server에서가 존재하여 클라이언트로부터 HTTP 요청을 받거나 정적 파일 서빙, 로드 밸런싱 등등 여러 역할을 수행합니다.

그리고 실제 비지니스 로직이 구현되는 계층인 Web Application까지 동일하나 그 앞에 WSGI라는 것이 존재합니다. **WSGI는 ‘Web Server Gateway Interface’** 로 Web Server와 Python Web Application 사이의 중간 계층에 있으며, WSGI 프로토콜을 구현하여 Web Server와 Python Application 간의 통신을 표준화해줍니다. 하는 일을 요약하자면 웹 서버와 웹 애플리케이션 사이에서 프로세스와 스레드 관리를 해주고 요청 큐잉 등을 담당합니다.

그렇다면 이 WSGI는 왜 필요한 걸까요?

