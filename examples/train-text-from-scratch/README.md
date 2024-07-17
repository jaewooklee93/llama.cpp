# train-text-from-scratch

기본 사용 방법 지침:

```bash
# get training data
wget https://raw.githubusercontent.com/brunoklein99/deep-learning-notes/master/shakespeare.txt

# train
./bin/llama-train-text-from-scratch \
        --vocab-model ../models/ggml-vocab-llama.gguf \
        --ctx 64 --embd 256 --head 8 --layer 16 \
        --checkpoint-in  chk-shakespeare-256x16-LATEST.gguf \
        --checkpoint-out chk-shakespeare-256x16-ITERATION.gguf \
        --model-out ggml-shakespeare-256x16-f32-ITERATION.gguf \
        --train-data "shakespeare.txt" \
        -t 6 -b 16 --seed 1 --adam-iter 256 \
        --no-checkpointing

# predict
./bin/llama-cli -m ggml-shakespeare-256x16-f32.gguf
```

출력 파일은 N번째 반복마다 저장됩니다 (config에서 `--save-every N`을 사용하여 설정). 
출력 파일 이름에서 "ITERATION" 패턴은 반복 번호로, "LATEST"는 최신 출력으로 대체됩니다. 

GGUF 모델을 훈련하려면 `--checkpoint-in FN`에 그 모델을 전달합니다. 
