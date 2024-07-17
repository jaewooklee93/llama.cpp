# 이진 파일 이름 마이그레이션 안내

> [!중요]
[2024년 6월 12일] 이진 파일 이름에 `llama-` 접두사가 추가되었습니다. `main`은 `llama-cli`로, `server`는 `llama-server`로 변경되었습니다. (https://github.com/ggerganov/llama.cpp/pull/7809)

이 마이그레이션은 중요하지만, 사용자에게 항상 즉시 명확하지 않을 수 있는 버전 변경 사항입니다.

모든 스크립트 및 워크플로우에서 새 이진 파일 이름을 사용하도록 업데이트하십시오.

| Old Filename | New Filename |
| ---- | ---- |
| main | llama-cli |
| server | llama-server |
| llama-bench | llama-bench |
| embedding | llama-embedding |
| finetune | llama-finetune |
| quantize | llama-quantize |
| tokenize | llama-tokenize |
| export-lora | llama-export-lora |
| libllava.a | libllava.a |
| baby-llama | llama-baby-llama |
| batched | llama-batched |
| batched-bench | llama-batched-bench |
| benchmark-matmult | llama-benchmark-matmult |
| convert-llama2c-to-ggml | llama-convert-llama2c-to-ggml |
| eval-callback | llama-eval-callback |
| gbnf-validator | llama-gbnf-validator |
| gguf | llama-gguf |
| gguf-split | llama-gguf-split |
| gritlm | llama-gritlm |
| imatrix | llama-imatrix |
| infill | llama-infill |
| llava-cli | llama-llava-cli |
| lookahead | llama-lookahead |
| lookup | llama-lookup |
| lookup-create | llama-lookup-create |
| lookup-merge | llama-lookup-merge |
| lookup-stats | llama-lookup-stats |
| parallel | llama-parallel |
| passkey | llama-passkey |
| perplexity | llama-perplexity |
| q8dot | llama-q8dot |
| quantize-stats | llama-quantize-stats |
| retrieval | llama-retrieval |
| save-load-state | llama-save-load-state |
| simple | llama-simple |
| speculative | llama-speculative |
| train-text-from-scratch | llama-train-text-from-scratch |
| vdot | llama-vdot |
| tests/test-c.o | tests/test-c.o |

