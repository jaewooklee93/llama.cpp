# finetune

기본 사용 방법 지침:

```bash
# get training data
wget https://raw.githubusercontent.com/brunoklein99/deep-learning-notes/master/shakespeare.txt

# finetune LORA adapter
./bin/llama-finetune \
        --model-base open-llama-3b-v2-q8_0.gguf \
        --checkpoint-in  chk-lora-open-llama-3b-v2-q8_0-shakespeare-LATEST.gguf \
        --checkpoint-out chk-lora-open-llama-3b-v2-q8_0-shakespeare-ITERATION.gguf \
        --lora-out lora-open-llama-3b-v2-q8_0-shakespeare-ITERATION.bin \
        --train-data "shakespeare.txt" \
        --save-every 10 \
        --threads 6 --adam-iter 30 --batch 4 --ctx 64 \
        --use-checkpointing

# predict
./bin/llama-cli -m open-llama-3b-v2-q8_0.gguf --lora lora-open-llama-3b-v2-q8_0-shakespeare-LATEST.bin
```

**단지 llama 기반 모델만 지원됩니다!** 출력 파일은 N번 간격으로 저장됩니다( `--save-every N` 으로 구성). 
출력 파일 이름의 'ITERATION' 패턴은 반복 횟수로 대체되며, 최신 출력에는 'LATEST'가 사용됩니다.
위 예시에서 10번 반복 후에는 다음과 같은 파일이 작성됩니다:
- chk-lora-open-llama-3b-v2-q8_0-shakespeare-10.gguf
- chk-lora-open-llama-3b-v2-q8_0-shakespeare-LATEST.gguf
- lora-open-llama-3b-v2-q8_0-shakespeare-10.bin
- lora-open-llama-3b-v2-q8_0-shakespeare-LATEST.bin

10번 더 반복한 후:
- chk-lora-open-llama-3b-v2-q8_0-shakespeare-20.gguf
- chk-lora-open-llama-3b-v2-q8_0-shakespeare-LATEST.gguf
- lora-open-llama-3b-v2-q8_0-shakespeare-20.bin
- lora-open-llama-3b-v2-q8_0-shakespeare-LATEST.bin

체크포인트 파일 (`--checkpoint-in FN`, `--checkpoint-out FN`)은 훈련 과정을 저장합니다. 입력 체크포인트 파일이 존재하지 않으면 새롭게 초기화된 어댑터로부터 훈련을 시작합니다.

llama.cpp 호환 LORA 어댑터는 `--lora-out FN`으로 지정된 파일 이름으로 저장됩니다.
이러한 LORA 어댑터는 `llama-cli`와 기본 모델과 함께 사용할 수 있습니다.

`llama-cli`에서 여러 LORA 어댑터를 로드할 수도 있으며, 이는 혼합되어 사용됩니다.

예를 들어 `lora-open-llama-3b-v2-q8_0-shakespeare-LATEST.bin`과 `lora-open-llama-3b-v2-q8_0-bible-LATEST.bin` 두 개의 LORA 어댑터가 있다면 다음과 같이 혼합할 수 있습니다:

```bash
./bin/llama-cli -m open-llama-3b-v2-q8_0.gguf \
  --lora lora-open-llama-3b-v2-q8_0-shakespeare-LATEST.bin \
  --lora lora-open-llama-3b-v2-q8_0-bible-LATEST.bin
```

`--lora-scaled FN SCALE`을 사용하여 기본 모델에 각 LORA 어댑터가 얼마나 강하게 적용되는지 변경할 수 있습니다.

예를 들어 'shakespeare' LORA 어댑터를 40%, 'bible' LORA 어댑터를 80%, 그리고 다른 어댑터를 100% 적용하려면 다음과 같습니다.

```bash
./bin/llama-cli -m open-llama-3b-v2-q8_0.gguf \
  --lora-scaled lora-open-llama-3b-v2-q8_0-shakespeare-LATEST.bin 0.4 \
  --lora-scaled lora-open-llama-3b-v2-q8_0-bible-LATEST.bin 0.8 \
  --lora lora-open-llama-3b-v2-q8_0-yet-another-one-LATEST.bin
```

규모 수는 1에 더해져야 하지 않으며, 어댑터의 영향력을 더욱 높이기 위해 1보다 큰 숫자를 사용할 수도 있습니다. 그러나 값을 너무 크게 설정하면 출력이 나빠질 수 있습니다. 좋은 값을 찾기 위해 시도해 보세요.

Gradient checkpointing은 메모리 요구량을 약 50% 줄이지만 실행 시간을 늘립니다.
충분한 RAM이 있다면 `--no-checkpointing`으로 checkpointing을 비활성화하여 fine-tuning을 조금 더 빠르게 할 수 있습니다.

기본 LORA 랭크는 `--lora-r N`으로 지정할 수 있습니다.
각 모델 텐서 유형별로 LORA 랭크를 별도로 구성할 수 있습니다.

```bash
  --lora-r N                 LORA r: default rank. Also specifies resulting scaling together with lora-alpha. (default 4)
  --rank-att-norm N          LORA rank for attention norm tensor (default 1)
  --rank-ffn-norm N          LORA rank for feed-forward norm tensor (default 1)
  --rank-out-norm N          LORA rank for output norm tensor (default 1)
  --rank-tok-embd N          LORA rank for token embeddings tensor (default 4)
  --rank-out N               LORA rank for output tensor (default 4)
  --rank-wq N                LORA rank for wq tensor (default 4)
  --rank-wk N                LORA rank for wk tensor (default 4)
  --rank-wv N                LORA rank for wv tensor (default 4)
  --rank-wo N                LORA rank for wo tensor (default 4)
  --rank-ffn_gate N          LORA rank for ffn_gate tensor (default 4)
  --rank-ffn_down N          LORA rank for ffn_down tensor (default 4)
  --rank-ffn_up N            LORA rank for ffn_up tensor (default 4)
```

'norm' 텐서의 LORA 랭크는 항상 1이어야 합니다.

모든 사용 가능한 옵션을 확인하려면 `llama-finetune --help`를 사용하십시오.
