# 디버깅 테스트 팁

## 특정 테스트를 실행하거나 디버그하는 방법은 무엇입니까?

scripts 폴더에 debug-test.sh라는 스크립트가 있으며, 매개변수로 REGEX와 선택적인 테스트 번호를 받습니다.

예를 들어, 다음 명령을 실행하면 상호 작용하는 목록이 출력되어 테스트를 선택할 수 있습니다. 형식은 다음과 같습니다.

`debug-test.sh [OPTION]... <test_regex> <test_number>`

그러면 디버거에서 빌드 및 실행됩니다.

테스트를 실행하고 PASS 또는 FAIL 메시지를 받으려면 다음을 실행합니다.

```bash
./scripts/debug-test.sh test-tokenizer
```

GDB에서 테스트하려면 `-g` 플래그를 사용하여 GDB 테스트 모드를 활성화합니다.

```bash
./scripts/debug-test.sh -g test-tokenizer

# Once in the debugger, i.e. at the chevrons prompt, setting a breakpoint could be as follows:
>>> b main
```

테스트 루프를 빠르게 진행하려면, 테스트 번호를 알고 있다면 아래와 같이 테스트를 실행할 수 있습니다.

```bash
./scripts/debug-test.sh test 23
```

참고로 `debug-test.sh -h`를 사용하여 도움말을 출력할 수 있습니다.

&nbsp;

### 스크립트 작동 방식
스크립트에 포함된 개념을 별도로 사용하고 싶다면, 아래에 간략하게 설명된 중요한 개념들을 참고하세요.

#### 1단계: 재설정 및 폴더 컨텍스트 설정

이 저장소의 기본 위치에서 `build-ci-debug`를 빌드 컨텍스트로 만들겠습니다.

```bash
rm -rf build-ci-debug && mkdir build-ci-debug && cd build-ci-debug
```

#### 2단계: 디버깅 환경 설정 및 테스트 바이너리 컴파일

디버그 모드에서 빌드를 설정하고 트리거합니다. 필요에 따라 인수를 조정할 수 있지만, 이 경우에는 합리적인 기본값입니다.

```bash
cmake -DCMAKE_BUILD_TYPE=Debug -DLLAMA_CUDA=1 -DLLAMA_FATAL_WARNINGS=ON ..
make -j
```

#### 3단계: REGEX에 맞는 모든 테스트 찾기

이 명령어의 출력은 GDB를 실행하기 위한 명령어 및 인수를 제공합니다.

* `-R test-tokenizer` : `test-tokenizer*`로 이름이 지정된 모든 테스트 파일을 찾습니다 (R=Regex)
* `-N` : "show-only"는 테스트 실행을 비활성화하고 GDB에 전달할 수 있는 테스트 명령을 표시합니다.
* `-V` : 자세한 정보 모드

```bash
ctest -R "test-tokenizer" -V -N
```

아래와 유사한 출력이 나타날 수 있습니다 (주의해야 할 핵심 줄에 초점을 맞춥니다):

```bash
...
1: Test command: ~/llama.cpp/build-ci-debug/bin/test-tokenizer-0 "~/llama.cpp/tests/../models/ggml-vocab-llama-spm.gguf"
1: Working Directory: .
Labels: main
  Test  #1: test-tokenizer-0-llama-spm
...
4: Test command: ~/llama.cpp/build-ci-debug/bin/test-tokenizer-0 "~/llama.cpp/tests/../models/ggml-vocab-falcon.gguf"
4: Working Directory: .
Labels: main
  Test  #4: test-tokenizer-0-falcon
...
```

#### 4단계: 디버깅을 위한 테스트 명령어 확인

위의 테스트 #1을 보면 다음과 같은 두 가지 관련 정보를 알 수 있습니다.
* 테스트 바이너리: `~/llama.cpp/build-ci-debug/bin/test-tokenizer-0`
* 테스트 GGUF 모델: `~/llama.cpp/tests/../models/ggml-vocab-llama-spm.gguf`

#### 5단계: 테스트 명령어에 GDB 실행

위 ctest '테스트 명령어' 보고서를 기반으로 아래 명령어를 통해 gdb 세션을 실행할 수 있습니다.

```bash
gdb --args ${Test Binary} ${Test GGUF Model}
```

예시:

```bash
gdb --args ~/llama.cpp/build-ci-debug/bin/test-tokenizer-0 "~/llama.cpp/tests/../models/ggml-vocab-llama-spm.gguf"
```
## 디버깅 및 테스트

Llama.cpp를 개발하고 유지보수하는 데 도움이 되는 디버깅 및 테스트 도구와 전략에 대한 자세한 내용은 다음을 참조하십시오.

* **디버깅:**
    * **GDB:** Llama.cpp 프로젝트를 디버깅하기 위해 GDB를 사용할 수 있습니다. GDB는 강력한 디버깅 도구로, 코드 실행을 단계별로 진행하고 변수 값을 확인하는 데 유용합니다.
    * **LLM Debugger:** LLM Debugger는 대규모 언어 모델의 디버깅에 특화된 도구입니다. Llama.cpp와 호환되며, 모델의 내부 상태를 시각화하고 분석하는 데 도움이 됩니다.
* **테스트:**
    * **단위 테스트:** Llama.cpp의 각 기능을 테스트하기 위해 단위 테스트를 작성하는 것이 좋습니다. 단위 테스트는 코드의 작동 방식을 확인하고 버그를 조기에 발견하는 데 도움이 됩니다.
    * **통합 테스트:** Llama.cpp의 여러 기능이 함께 작동하는지 테스트하기 위해 통합 테스트를 작성하는 것이 좋습니다. 통합 테스트는 시스템 전체의 작동 방식을 확인하고 문제를 조기에 발견하는 데 도움이 됩니다.
    * **테스트 프레임워크:** Llama.cpp를 테스트하는 데 사용할 수 있는 다양한 테스트 프레임워크가 있습니다. 예를 들어, Google Test, Catch2, Boost.Test 등이 있습니다.

**추가 정보:**

* Llama.cpp 개발자 문서: https://github.com/ggerganov/llama.cpp
* LLM Debugger: https://github.com/huggingface/llama-debugger

