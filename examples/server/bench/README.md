### 서버 성능 측정 도구

성능 측정은 [k6](https://k6.io/)을 사용합니다.

##### k6 및 sse 확장 설치

k6에서 SSE는 기본적으로 지원되지 않습니다. [xk6-sse](https://github.com/phymbert/xk6-sse) 확장을 사용하여 k6를 빌드해야 합니다.

예시:
```shell
go install go.k6.io/xk6/cmd/xk6@latest
xk6 build master \
--with github.com/phymbert/xk6-sse
```

#### 데이터셋 다운로드

이 데이터셋은 처음에 [vLLM 벤치마크](https://github.com/vllm-project/vllm/blob/main/benchmarks/README.md)에서 제안되었습니다.

```shell
wget https://huggingface.co/datasets/anon8231489123/ShareGPT_Vicuna_unfiltered/resolve/main/ShareGPT_V3_unfiltered_cleaned_split.json
```

#### 모델 다운로드
PHI-2 예시

```shell
../../../scripts/hf.sh --repo ggml-org/models --file phi-2/ggml-model-q4_0.gguf
```

#### 서버 시작
서버는 `http://localhost:8080/v1` 또는 환경 변수 `SERVER_BENCH_URL`에 따라 OAI Chat 완성 요청에 응답해야 합니다.

예시:
```shell
server --host localhost --port 8080 \
  --model ggml-model-q4_0.gguf \
  --cont-batching \
  --metrics \
  --parallel 8 \
  --batch-size 512 \
  --ctx-size 4096 \
  --log-format text \
  -ngl 33
```

#### 벤치마크 실행

최대 10분 동안 8개의 동시 사용자로 500개의 채팅 완료 요청을 실행하려면 다음을 실행합니다.
```shell
./k6 run script.js --duration 10m --iterations 500 --vus 8
```

벤치마크 값은 다음과 같이 재정의할 수 있습니다.
- `SERVER_BENCH_URL` 채팅 완성을 위한 서버 URL 접두사, 기본값 `http://localhost:8080/v1`
- `SERVER_BENCH_N_PROMPTS` 벤치마크에서 무작위로 선택할 총 프롬프트 수, 기본값 `480`
- `SERVER_BENCH_MODEL_ALIAS` 완성 요청에 전달할 모델 별칭, 기본값 `my-model`
- `SERVER_BENCH_MAX_TOKENS` 예측할 최대 토큰 수, 기본값: `512`
- `SERVER_BENCH_DATASET` 벤치마크 데이터셋 파일 경로
- `SERVER_BENCH_MAX_PROMPT_TOKENS` 데이터셋에서 필터링할 최대 프롬프트 토큰 수: 기본값 `1024`
- `SERVER_BENCH_MAX_CONTEXT` 완성 요청의 최대 컨텍스트 크기(프롬프트 + 예측 토큰)를 필터링하는 데 사용, 기본값 `2048`

참고: 로컬 토크나이저는 문자 공백 분할일 뿐이며, 실제 토큰 수는 다를 수 있습니다.

또는 [k6 옵션](https://k6.io/docs/using-k6/k6-options/reference/)을 사용하여:

```shell
SERVER_BENCH_N_PROMPTS=500 k6 run script.js --duration 10m --iterations 500 --vus 8
```

HTTP 요청 디버깅을 위해 `--http-debug="full"` 를 사용하세요. ([https://k6.io/docs/using-k6/http-debugging/](https://k6.io/docs/using-k6/http-debugging/))

#### 지표

OAI 채팅 완료 응답 `usage`에서 계산되는 다음 지표가 사용됩니다.
- `llamacpp_tokens_second` `usage.total_tokens / 요청 지속 시간`의 추세
- `llamacpp_prompt_tokens` `usage.prompt_tokens`의 추세
- `llamacpp_prompt_tokens_total_counter` `usage.prompt_tokens`의 카운터
- `llamacpp_completion_tokens` `usage.completion_tokens`의 추세
- `llamacpp_completion_tokens_total_counter` `usage.completion_tokens`의 카운터
- `llamacpp_completions_truncated_rate` 완료된 것의 비율이 잘려졌는지, 즉 `finish_reason === 'length'`인 경우
- `llamacpp_completions_stop_rate` 모델에 의해 중단된 완료된 것의 비율, 즉 `finish_reason === 'stop'`인 경우

완료된 것의 수가 너무 많이 잘려지면 스크립트가 실패합니다. `llamacpp_completions_truncated_rate`를 참조하세요.

K6 지표는 [서버 지표](../README.md)와 비교할 수 있습니다.

```shell
curl http://localhost:8080/metrics
```

### CI Python 스크립트 사용
`bench.py` 스크립트는 다음과 같은 작업을 수행합니다.
- 서버 시작
- k6에 대한 좋은 변수 정의
- k6 스크립트 실행
- Prometheus에서 메트릭 추출

CI에서 사용하도록 설계되었지만 수동으로 실행할 수도 있습니다.

```shell
LLAMA_SERVER_BIN_PATH=../../../cmake-build-release/bin/llama-server python bench.py \
              --runner-label local \
              --name local \
              --branch `git rev-parse --abbrev-ref HEAD` \
              --commit `git rev-parse HEAD` \
              --scenario script.js \
              --duration 5m \
              --hf-repo ggml-org/models	 \
              --hf-file phi-2/ggml-model-q4_0.gguf \
              --model-path-prefix models \
              --parallel 4 \
              -ngl 33 \
              --batch-size 2048 \
              --ubatch-size	256 \
              --ctx-size 4096 \
              --n-prompts 200 \
              --max-prompt-tokens 256 \
              --max-tokens 256
```
