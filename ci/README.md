# CI

[Github Actions](https://github.com/ggerganov/llama.cpp/actions) 외에도 `llama.cpp`는 다음과 같은 맞춤형 CI 프레임워크를 사용합니다:

https://github.com/ggml-org/ci

`master` 브랜치에서 새로운 커밋을 감시하고, [ci/run.sh](https://github.com/ggerganov/llama.cpp/blob/master/ci/run.sh) 스크립트를 전용 클라우드 인스턴스에서 실행합니다. 이를 통해 Github Actions만 사용하는 것보다 더 무거운 작업을 수행할 수 있습니다. 또한 시간이 지남에 따라 클라우드 인스턴스는 GPU 및 Apple Silicon 인스턴스를 포함한 다양한 하드웨어 아키텍처를 커버하도록 확장될 것입니다.

협력자는 커밋 메시지에 `ggml-ci` 키워드를 추가하여 CI 실행을 수동으로 트리거할 수 있습니다.
단, 이 레포지토리의 브랜치만이 이 키워드를 감시합니다.

변경 사항을 게시하기 전에, 로컬 머신에서 전체 CI를 실행하는 것이 좋습니다:

```bash
mkdir tmp

# CPU-only build
bash ./ci/run.sh ./tmp/results ./tmp/mnt

# with CUDA support
GG_BUILD_CUDA=1 bash ./ci/run.sh ./tmp/results ./tmp/mnt

# with SYCL support
source /opt/intel/oneapi/setvars.sh
GG_BUILD_SYCL=1 bash ./ci/run.sh ./tmp/results ./tmp/mnt
```
