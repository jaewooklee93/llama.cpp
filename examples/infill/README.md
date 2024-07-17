# llama.cpp/example/infill

이 예제는 Code Llama 모델에서 지원하는 infill 모드를 사용하는 방법을 보여줍니다.
현재 7B 및 13B 모델이 infill 모드를 지원합니다.

Infill은 메인 예제에서 사용 가능한 대부분의 옵션을 지원합니다.

더 자세한 내용은 llama.cpp/example/main/README.md의 메인 README.md를 참조하십시오.

## 일반 옵션

이 섹션에서는 `infill` 프로그램을 LLaMA 모델과 함께 실행할 때 가장 일반적으로 사용되는 옵션을 설명합니다.

-   `-m FNAME, --model FNAME`: LLaMA 모델 파일의 경로를 지정합니다 (예: `models/7B/ggml-model.bin`).
-   `-i, --interactive`: 프로그램을 상호 작용 모드로 실행하여 직접 입력을 제공하고 실시간 응답을 받을 수 있도록 합니다.
-   `-n N, --n-predict N`: 텍스트를 생성할 때 예측할 토큰 수를 설정합니다. 이 값을 조정하면 생성된 텍스트의 길이에 영향을 미칠 수 있습니다.
-   `-c N, --ctx-size N`: 프롬프트 맥락의 크기를 설정합니다. 기본값은 512이지만, LLaMA 모델은 2048의 맥락으로 구축되었으며, 더 긴 입력/인프런스에 대해 더 나은 결과를 제공합니다.
-   `--spm-infill`: 일부 모델은 이를 선호하기 때문에 infill에 대해 접미사/전두사/중간 패턴을 사용합니다 (전두사/접미사/중간 패턴 대신).

## 입력 프롬프트

`infill` 프로그램은 입력 프롬프트를 사용하여 LLaMA 모델과 상호 작용하는 여러 가지 방법을 제공합니다.

-   `--in-prefix PROMPT_BEFORE_CURSOR`: 프롬프트를 명령줄 옵션으로 직접 제공합니다.
-   `--in-suffix PROMPT_AFTER_CURSOR`: 프롬프트를 명령줄 옵션으로 직접 제공합니다.
-   `--interactive-first`: 프로그램을 상호 작용 모드로 실행하고 즉시 입력을 기다립니다. (아래에서 자세히 설명)

## 상호 작용

`infill` 프로그램은 사용자가 실시간으로 채우기 제안을 받을 수 있도록 LLaMA 모델과 상호 작용하는 간편한 방법을 제공합니다. 상호 작용 모드는 `--interactive` 및 `--interactive-first`를 사용하여 트리거할 수 있습니다.

### 상호 작용 옵션

-   `-i, --interactive`: 사용자가 모델로부터 실시간 코드 제안을 받을 수 있도록 상호 작용 모드로 프로그램을 실행합니다.
-   `--interactive-first`: 프로그램을 상호 작용 모드로 실행하고 텍스트 생성을 시작하기 전에 사용자 입력을 즉시 기다립니다.
-   `--color`: 프롬프트, 사용자 입력 및 생성된 텍스트를 시각적으로 구분하기 위해 색상 출력을 활성화합니다.

### 예시

인필 기능을 지원하는 모델을 다운로드합니다. 예를 들어 CodeLlama를 사용하세요:
```console
scripts/hf.sh --repo TheBloke/CodeLlama-13B-GGUF --file codellama-13b.Q5_K_S.gguf --outdir models
```

```bash
./llama-infill -t 10 -ngl 0 -m models/codellama-13b.Q5_K_S.gguf -c 4096 --temp 0.7 --repeat_penalty 1.1 -n 20 --in-prefix "def helloworld():\n    print(\"hell" --in-suffix "\n   print(\"goodbye world\")\n    "
```
