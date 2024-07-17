# cvector-generator

이 예제는 gguf 모델을 사용하여 제어 벡터를 생성하는 방법을 보여줍니다.

관련 PR:
- [제어 벡터 지원 추가](https://github.com/ggerganov/llama.cpp/pull/5970)
- (문제) [llama.cpp를 사용하여 제어 벡터 생성](https://github.com/ggerganov/llama.cpp/issues/6880)
- [cvector-generator 예제 추가](https://github.com/ggerganov/llama.cpp/pull/7514)

## 예시

```sh
# CPU only
./cvector-generator -m ./llama-3.Q4_K_M.gguf

# With GPU
./cvector-generator -m ./llama-3.Q4_K_M.gguf -ngl 99

# With advanced options
./cvector-generator -m ./llama-3.Q4_K_M.gguf -ngl 99 --pca-iter 2000 --pca-batch 100

# Using mean value instead of PCA
./cvector-generator -m ./llama-3.Q4_K_M.gguf --method mean

# To see help message
./cvector-generator -h
# Then, have a look at "cvector" section
```

## 팁 및 꿀팁

여러 줄이 있는 프롬프트의 경우, 줄 바꿈 문자를 이스케이프하여 `\n`으로 변경할 수 있습니다. 예를 들어:

```
<|im_start|>system\nAct like a person who is extremely happy.<|im_end|>
<|im_start|>system\nYou are in a very good mood today<|im_end|>
```

`llama-cli`를 사용하여 출력 파일을 사용하는 예시:

(팁: 제어 벡터는 10보다 높은 레이어에 적용할 때 더 효과적입니다)

```sh
./llama-cli -m ./llama-3.Q4_K_M.gguf -p "<|start_header_id|>system<|end_header_id|>\n\nYou are a helpful assistant<|eot_id|><|start_header_id|>user<|end_header_id|>\n\nSing a song<|im_end|><|eot_id|><|start_header_id|>assistant<|end_header_id|>\n\n" --special --control-vector-scaled ./control_vector.gguf 0.8 --control-vector-layer-range 10 31
```
