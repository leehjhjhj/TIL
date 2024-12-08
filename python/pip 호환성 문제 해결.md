> ImportError: dlopen(/Users/dev/test/.venv/lib/python3.9/site-packages/gevent/_gevent_c_hub_local.cpython-39-darwin.so, 0x0002): tried: '/Users/dev/test/.venv/lib/python3.9/site-packages/gevent/_gevent_c_hub_local.cpython-39-darwin.so' (mach-o file, but is an incompatible architecture (have 'x86_64', need 'arm64')), '/System/Volumes/Preboot/Cryptexes/OS/Users/dicemono/dev/test/.venv/lib/python3.9/site-packages/gevent/_gevent_c_hub_local.cpython-39-darwin.so' (no such file), '/Users/dev/test/.venv/lib/python3.9/site-packages/gevent/_gevent_c_hub_local.cpython-39-darwin.so' (mach-o file, but is an incompatible architecture (have 'x86_64', need 'arm64'))
> 

대충 Apple Silicon(ARM64) Mac에서 x86_64 아키텍처용으로 컴파일된 gevent 바이너리를 실행하려고 할 때 발생하는 호환성 문제라고 합니다.

Locust는 gevent 기반으로 돌아가는데요, 대충 뭐냐면 이벤트 기반으로 돌아가는 Python의 비동기 프레임워크로, greenlet을 사용하여 동시성을 구현합니다.

그래서 해결법은 다음과 같습니다.

```bash
python -m pip uninstall locust
```

우선 다시 locust를 지워주고

```bash
arch -arm64 python -m pip install locust --no-cache
```

이렇게 아티텍처를 명시하고 install 해줍니다. 저 `--no-cache` 도 핵심 키워드입니당

locust 뿐만 아니라 가끔 파이썬 패키지 설치할 때 나는 오류니 기억해두면 좋을 것 같아요~