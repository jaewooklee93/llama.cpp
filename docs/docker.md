# Docker

## 필요 조건
* Docker가 시스템에 설치되어 실행 중이어야 합니다.
* 큰 모델 및 중간 파일을 저장할 폴더를 생성하십시오 (예: /llama/models)

## 이미지
이 프로젝트에는 다음 세 가지 Docker 이미지가 있습니다.

1. `ghcr.io/ggerganov/llama.cpp:full`: 이 이미지에는 메인 실행 파일과 LLaMA 모델을 ggml로 변환하고 4비트 양자화로 변환하는 도구가 모두 포함되어 있습니다. (플랫폼: `linux/amd64`, `linux/arm64`)
2. `ghcr.io/ggerganov/llama.cpp:light`: 이 이미지에는 메인 실행 파일만 포함되어 있습니다. (플랫폼: `linux/amd64`, `linux/arm64`)
3. `ghcr.io/ggerganov/llama.cpp:server`: 이 이미지에는 서버 실행 파일만 포함되어 있습니다. (플랫폼: `linux/amd64`, `linux/arm64`)

또한, 위와 유사한 다음 이미지가 있습니다.

- `ghcr.io/ggerganov/llama.cpp:full-cuda`: CUDA 지원으로 컴파일된 `full`과 동일합니다. (플랫폼: `linux/amd64`)
- `ghcr.io/ggerganov/llama.cpp:light-cuda`: CUDA 지원으로 컴파일된 `light`와 동일합니다. (플랫폼: `linux/amd64`)
- `ghcr.io/ggerganov/llama.cpp:server-cuda`: CUDA 지원으로 컴파일된 `server`와 동일합니다. (플랫폼: `linux/amd64`)
- `ghcr.io/ggerganov/llama.cpp:full-rocm`: ROCm 지원으로 컴파일된 `full`과 동일합니다. (플랫폼: `linux/amd64`, `linux/arm64`)
- `ghcr.io/ggerganov/llama.cpp:light-rocm`: ROCm 지원으로 컴파일된 `light`와 동일합니다. (플랫폼: `linux/amd64`, `linux/arm64`)
- `ghcr.io/ggerganov/llama.cpp:server-rocm`: ROCm 지원으로 컴파일된 `server`와 동일합니다. (플랫폼: `linux/amd64`, `linux/arm64`)

GPU 지원 이미지는 현재 CI에서 테스트되지 않았습니다.  `.devops/` 디렉토리에 정의된 Dockerfile 및 `.github/workflows/docker.yml` 파일에 정의된 GitHub Action에서 정의된 Dockerfile와 동일한 설정으로 빌드됩니다. CUDA 또는 ROCm 라이브러리 등 다른 설정이 필요한 경우 현재는 로컬로 이미지를 빌드해야 합니다.

## 사용법

모델을 다운로드하고 ggml로 변환하고 최적화하는 가장 쉬운 방법은 전체 Docker 이미지를 포함하는 `--all-in-one` 명령을 사용하는 것입니다.

아래 `/path/to/models`를 실제로 모델을 다운로드한 경로로 바꿔주세요.

```bash
docker run -v /path/to/models:/models ghcr.io/ggerganov/llama.cpp:full --all-in-one "/models/" 7B
```

완료되면 놀기 시작할 준비가 되었습니다!

```bash
docker run -v /path/to/models:/models ghcr.io/ggerganov/llama.cpp:full --run -m /models/7B/ggml-model-q4_0.gguf -p "Building a website can be done in 10 simple steps:" -n 512
```

혹은 가벼운 이미지를 사용하여:

```bash
docker run -v /path/to/models:/models ghcr.io/ggerganov/llama.cpp:light -m /models/7B/ggml-model-q4_0.gguf -p "Building a website can be done in 10 simple steps:" -n 512
```

혹은 서버 이미지를 사용하여:

```bash
docker run -v /path/to/models:/models -p 8000:8000 ghcr.io/ggerganov/llama.cpp:server -m /models/7B/ggml-model-q4_0.gguf --port 8000 --host 0.0.0.0 -n 512
```

## CUDA와 함께 사용하는 Docker

Linux에서 [nvidia-container-toolkit](https://github.com/NVIDIA/nvidia-container-toolkit)을 제대로 설치하거나 GPU 지원 클라우드를 사용하는 경우, 컨테이너 내부에서 `cuBLAS`에 액세스할 수 있습니다.

## 로컬 Docker 이미지 빌드

```bash
docker build -t local/llama.cpp:full-cuda -f .devops/full-cuda.Dockerfile .
docker build -t local/llama.cpp:light-cuda -f .devops/llama-cli-cuda.Dockerfile .
docker build -t local/llama.cpp:server-cuda -f .devops/llama-server-cuda.Dockerfile .
```

컨테이너 호스트에서 지원하는 CUDA 환경 및 GPU 아키텍처에 따라 `ARGS`를 다르게 전달해야 할 수 있습니다.

기본값은 다음과 같습니다.

- `CUDA_VERSION`이 `11.7.1`로 설정됨
- `CUDA_DOCKER_ARCH`가 `all`로 설정됨

결과적으로 생성되는 이미지는 CUDA 이미지와 동일합니다.

1. `local/llama.cpp:full-cuda`: 이 이미지에는 LLaMA 모델을 ggml로 변환하고 4비트 양자화로 변환하는 도구와 함께 메인 실행 파일이 포함되어 있습니다.
2. `local/llama.cpp:light-cuda`: 이 이미지에는 메인 실행 파일만 포함되어 있습니다.
3. `local/llama.cpp:server-cuda`: 이 이미지에는 서버 실행 파일만 포함되어 있습니다.

## 사용법

현재 로컬로 빌드한 후 사용법은 CUDA 예제와 유사하지만, `--gpus` 플래그를 추가해야 합니다. 또한 `--n-gpu-layers` 플래그를 사용하는 것이 좋습니다.

```bash
docker run --gpus all -v /path/to/models:/models local/llama.cpp:full-cuda --run -m /models/7B/ggml-model-q4_0.gguf -p "Building a website can be done in 10 simple steps:" -n 512 --n-gpu-layers 1
docker run --gpus all -v /path/to/models:/models local/llama.cpp:light-cuda -m /models/7B/ggml-model-q4_0.gguf -p "Building a website can be done in 10 simple steps:" -n 512 --n-gpu-layers 1
docker run --gpus all -v /path/to/models:/models local/llama.cpp:server-cuda -m /models/7B/ggml-model-q4_0.gguf --port 8000 --host 0.0.0.0 -n 512 --n-gpu-layers 1
```
