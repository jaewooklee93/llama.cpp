# 서버 테스트

Python 기반 서버 테스트 시나리오는 [BDD](https://en.wikipedia.org/wiki/Behavior-driven_development) 와 [behave](https://behave.readthedocs.io/en/latest/)를 사용합니다:

* [issues.feature](./features/issues.feature) 미처리된 문제 시나리오
* [parallel.feature](./features/parallel.feature) 여러 슬롯과 동시 요청을 포함하는 시나리오
* [security.feature](./features/security.feature) 보안, CORS 및 API 키
* [server.feature](./features/server.feature) 서버 기본 시나리오: 완성, 삽입, 토큰화 등...

테스트는 4 vCPU를 가진 GitHub 워크플로우 작업 실행자를 대상으로 합니다.

요청은 [aiohttp](https://docs.aiohttp.org/en/stable/client_reference.html), [asyncio](https://docs.python.org/fr/3/library/asyncio.html) 기반 HTTP 클라이언트를 사용합니다.

참고: 호스트 아키텍처의 추론 속도가 GitHub runner보다 빠르면 병렬 시나리오가 무작위로 실패할 수 있습니다. 이를 완화하려면 `n_predict`, `kv_size` 값을 늘릴 수 있습니다.

### 의존성 설치

`pip install -r requirements.txt`

### 테스트 실행

1. 서버 빌드

```shell
cd ../../..
cmake -B build -DLLAMA_CURL=ON
cmake --build build --target llama-server
```

2. 테스트 시작: `./tests.sh`

일부 시나리오 단계 값은 환경 변수로 재정의할 수 있습니다:

| variable                 | description                                                                                    |
|--------------------------|------------------------------------------------------------------------------------------------|
| `PORT`                   | `context.server_port` to set the listening port of the server during scenario, default: `8080` |
| `LLAMA_SERVER_BIN_PATH`  | to change the server binary path, default: `../../../build/bin/llama-server`                         |
| `DEBUG`                  | "ON" to enable steps and server verbose mode `--verbose`                                       |
| `SERVER_LOG_FORMAT_JSON` | if set switch server logs to json format                                                       |
| `N_GPU_LAYERS`           | number of model layers to offload to VRAM `-ngl --n-gpu-layers`                                |

### @bug, @wip 또는 @wrong_usage로 표시된 시나리오 실행

특징 또는 시나리오는 기본 범위에 포함되려면 `@llama.cpp`로 표시되어야 합니다.

- `@bug` 어노테이션은 시나리오를 GitHub 문제와 연결하는 데 사용됩니다.
- `@wrong_usage`은 실제로 예상되는 동작인 사용자 문제를 보여주는 데 사용됩니다.
- `@wip`는 진행 중인 시나리오에 초점을 맞추는 데 사용됩니다.
- `@slow`는 기본적으로 비활성화된 무거운 테스트입니다.

`@bug`로 표시된 시나리오를 실행하려면 다음을 시작하십시오:

```shell
DEBUG=ON ./tests.sh --no-skipped --tags bug --stop
```

 `steps.py` 에서 논리 변경 후, `@bug` 와 `@wrong_usage` 시나리오를 업데이트해야 합니다.

```shell
./tests.sh --no-skipped --tags bug,wrong_usage || echo "should failed but compile"
```
