# llama.cpp/example/passkey

패스키 검색 작업은 언어 모델이 긴 맥락에서 정보를 검색하는 능력을 측정하는 평가 방법입니다.

더 자세한 내용은 다음 PR를 참조하십시오.

- https://github.com/ggerganov/llama.cpp/pull/3856
- https://github.com/ggerganov/llama.cpp/pull/4810

### 사용법

```bash
make -j && ./llama-passkey -m ./models/llama-7b-v2/ggml-model-f16.gguf --junk 250
```
