# llama.cpp/examples/imatrix

모델과 주어진 텍스트 데이터셋에 대한 중요도 행렬을 계산합니다. 양자화 중에 양자 모델의 품질을 향상시키는 데 사용할 수 있습니다.
더 자세한 내용은 여기를 참조하십시오: https://github.com/ggerganov/llama.cpp/pull/4861

## 사용법

```
./llama-imatrix \
    -m model.gguf -f some-text.txt [-o imatrix.dat] [--process-output] [--verbosity 1] \
    [--no-ppl] [--chunk 123] [--output-frequency 10] [--save-frequency 0] \
    [--in-file imatrix-prev-0.dat --in-file imatrix-prev-1.dat ...]
```

여기서 `-m`은 모델 이름과 `-f`는 훈련 데이터를 포함하는 파일(예: `wiki.train.raw`)을 지정해야 합니다.
괄호 안의 매개변수는 선택 사항이며 다음과 같은 의미를 갖습니다.
* `-o` (또는 `--output-file`)는 계산된 데이터가 저장될 파일 이름을 지정합니다. 누락되면 `imatrix.dat`가 사용됩니다.
* `--verbosity`는 출력의 세부 정보 수준을 지정합니다. `0`으로 설정하면 처리된 조각의 perplexity 외에는 다른 출력이 생성되지 않습니다. `1`로 설정하면 결과가 저장될 때마다 `stderr`에 메시지가 출력됩니다. `>=2`로 설정하면 데이터가 어떤 텐서에 대해 수집될 때마다 메시지가 출력됩니다. 기본 세부 정보 수준은 `1`입니다.
* `--output-frequency`는 현재 계산된 결과가 디스크에 저장되는 빈도를 지정합니다. 기본값은 10입니다(즉, 10개의 조각마다).
* `--save-frequency`는 imatrix를 별도의 파일로 저장하는 빈도를 지정합니다. 기본값은 0입니다(즉, 절대 저장하지 않습니다).
* `--process-output`는 `output.weight` 텐서에 대한 데이터가 수집될지 여부를 지정합니다. 제 경험에 따르면, `output.weight`를 양자화할 때 중요도 행렬을 사용하는 것은 좋지 않으므로 기본값은 `false`입니다.

더 빠른 계산을 위해 `-ngl` 인수를 사용하여 GPU 오프로드를 사용하는 것이 좋습니다.

## 예시

```bash
GGML_CUDA=1 make -j

# generate importance matrix (imatrix.dat)
./llama-imatrix -m ggml-model-f16.gguf -f train-data.txt -ngl 99

# use the imatrix to perform a Q4_K_M quantization
./llama-quantize --imatrix imatrix.dat ggml-model-f16.gguf ./ggml-model-q4_k_m.gguf q4_k_m
```
