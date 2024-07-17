# `llama.cpp`에 새로운 모델 아키텍처 추가하기

새로운 모델을 추가하려면 다음과 같은 몇 가지 단계를 거쳐야 합니다.

1. 모델을 GGUF 형식으로 변환합니다.
2. `llama.cpp`에서 모델 아키텍처를 정의합니다.
3. GGML 그래프 구현을 빌드합니다.

이러한 단계를 따르면 새로운 모델을 위한 Pull Request를 제출할 수 있습니다.

또한, 예제 및 주요 GGML 백엔드 (CUDA, METAL, CPU)가 새로운 아키텍처와 함께 작동하는지 확인하는 것이 중요합니다. 특히 다음을 확인하십시오.
- [main](/examples/main/)
- [imatrix](/examples/imatrix/)
- [quantize](/examples/quantize/)
- [server](/examples/server/)

### 1. 모델을 GGUF로 변환하기

이 단계는 [gguf](https://pypi.org/project/gguf/) 라이브러리를 사용하여 Python의 `convert` 스크립트로 수행됩니다.
모델 아키텍처에 따라 [convert_hf_to_gguf.py](/convert_hf_to_gguf.py) 또는 [examples/convert_legacy_llama.py](/examples/convert_legacy_llama.py) (`.pth` 형식의 `llama/llama2` 모델을 위해)를 사용할 수 있습니다.

변환 스크립트는 모델 구성, 토크나이저, 텐서 이름+데이터를 읽어 GGUF 메타데이터 및 텐서로 변환합니다.

HF 모델에 대한 필요한 단계는 다음과 같습니다.

1. 새로운 `Model` 서브클래스에서 모델 `Model.register` 어노테이션을 정의합니다. 예를 들어,

```python
@Model.register("MyModelForCausalLM")
class MyModel(Model):
    model_arch = gguf.MODEL_ARCH.GROK
```

2. [constants.py](/gguf-py/gguf/constants.py)에서 GGUF 텐서 레이아웃 정의

`MODEL_ARCH`에 열거형 항목을 추가하고, 모델의 사용자 친화적인 이름을 `MODEL_ARCH_NAMES`에 추가하고, `MODEL_TENSORS`에 GGUF 텐서 이름을 추가합니다.

`falcon` 모델의 예시:
```python
    MODEL_ARCH.FALCON: [
        MODEL_TENSOR.TOKEN_EMBD,
        MODEL_TENSOR.OUTPUT_NORM,
        MODEL_TENSOR.OUTPUT,
        MODEL_TENSOR.ATTN_NORM,
        MODEL_TENSOR.ATTN_NORM_2,
        MODEL_TENSOR.ATTN_QKV,
        MODEL_TENSOR.ATTN_OUT,
        MODEL_TENSOR.FFN_DOWN,
        MODEL_TENSOR.FFN_UP,
    ]
```

3. 원본 텐서 이름을 GGUF의 표준화된 동등물에 매핑합니다.

일반적으로, 새로운 텐서 이름을 GGUF에 추가하기 전에 동등한 이름이 이미 존재하지 않는지 확인하십시오.

GGUF 텐서 이름에 해당하는 것을 찾은 후, [tensor_mapping.py](/gguf-py/gguf/tensor_mapping.py) 파일로 추가하십시오.

텐서 이름이 반복적인 레이어/블록의 일부라면, 키워드 `bid`가 대체됩니다.

주의 레이어의 정규화 텐서에 대한 예시:

```python
block_mappings_cfg: dict[MODEL_TENSOR, tuple[str, ...]] = {
        # Attention norm
        MODEL_TENSOR.ATTN_NORM: (
            "gpt_neox.layers.{bid}.input_layernorm",                # gptneox
            "transformer.h.{bid}.ln_1",                             # gpt2 gpt-j refact qwen
            "transformer.blocks.{bid}.norm_1",                      # mpt
            ...
        )
}
```

`transformer.blocks.{bid}.norm_1`은 GGUF에서 `blk.{bid}.attn_norm`으로 매핑됩니다.

모델 구성, 토크나이저, 코드 및 텐서 레이아웃에 따라 다음을 재정의해야 할 수 있습니다.
- `Model#set_gguf_parameters`
- `Model#set_vocab`
- `Model#write_tensors`

참고: 텐서 이름은 `.weight` 접미사로 끝나야 합니다. 이는 관례이며 `quantize`와 같은 여러 도구가 가중치를 앞서 이를 기대하기 때문입니다.

### 2. `llama.cpp`에서 모델 아키텍처 정의

모델 파라미터 및 텐서 레이아웃은 `llama.cpp`에서 정의해야 합니다.
1. 새로운 `llm_arch` 정의
2. `LLM_TENSOR_NAMES`에서 텐서 레이아웃 정의
3. `llm_load_hparams`에서 비표준 메타데이터 추가
4. `llm_load_tensors`에서 인퍼런스를 위한 텐서 생성
5. 모델에 RoPE 연산이 있는 경우 `llama_rope_type`에 로프 유형 추가

참고: `ggml`의 차원은 일반적으로 `pytorch` 차원의 역순입니다.

### 3. GGML 그래프 구현 빌드

가장 재미있는 부분입니다. 새로운 모델 아키텍처의 인퍼런스 그래프 구현을 `llama_build_graph` 에 제공해야 합니다.

`build_llama`, `build_dbrx` 또는 `build_bert` 와 같은 기존 구현을 참조하세요.

새로운 그래프를 구현할 때, 기본 `ggml` 백엔드가 모든 것을 지원하지 않을 수 있습니다. 누락된 백엔드 작업에 대한 지원은 다른 PR 에서 추가될 수 있습니다.

참고: 인퍼런스 그래프를 디버그하려면 [llama-eval-callback](/examples/eval-callback/) 을 사용할 수 있습니다.

## GGUF 사양

https://github.com/ggerganov/ggml/blob/master/docs/gguf.md

## 자원

- YaRN RoPE scaling https://github.com/ggerganov/llama.cpp/pull/2268
- Baichuan serial models 지원 https://github.com/ggerganov/llama.cpp/pull/3009
- attention bias 지원 https://github.com/ggerganov/llama.cpp/pull/4283
- Mixtral 지원 https://github.com/ggerganov/llama.cpp/pull/4406
- BERT embeddings 지원 https://github.com/ggerganov/llama.cpp/pull/5423
- Grok-1 지원 https://github.com/ggerganov/llama.cpp/pull/6204
- Command R Plus 지원 https://github.com/ggerganov/llama.cpp/pull/6491
- arch DBRX 지원 https://github.com/ggerganov/llama.cpp/pull/6515
- HuggingFace 모델을 GGUF 형식으로 변환하는 방법 https://github.com/ggerganov/llama.cpp/discussions/2948
