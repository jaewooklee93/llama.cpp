# export-lora

기본 모델에 LORA 어댑터를 적용하고 결과 모델을 내보냅니다.

```
usage: llama-export-lora [options]

options:
  -h, --help                         show this help message and exit
  -m FNAME, --model-base FNAME       model path from which to load base model (default '')
  -o FNAME, --model-out FNAME        path to save exported model (default '')
  -l FNAME, --lora FNAME             apply LoRA adapter
  -s FNAME S, --lora-scaled FNAME S  apply LoRA adapter with user defined scaling S
  -t N, --threads N                  number of threads to use during computation (default: 4)
```

예를 들어:

```bash
./bin/llama-export-lora \
    -m open-llama-3b-v2-q8_0.gguf \
    -o open-llama-3b-v2-q8_0-english2tokipona-chat.gguf \
    -l lora-open-llama-3b-v2-q8_0-english2tokipona-chat-LATEST.bin
```

여러 LORA 어댑터를 적용하려면 여러 개의 `-l FN` 또는 `-s FN S` 명령줄 매개변수를 전달합니다.
