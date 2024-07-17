## llama2.c 모델을 ggml로 변환하기

이 예제는 [llama2.c](https://github.com/karpathy/llama2.c) 프로젝트에서 가중치를 읽어 ggml 호환 형식으로 저장합니다. `models/ggml-vocab.bin`에 있는 단어 목록이 기본적으로 사용됩니다.

모델을 변환하려면 먼저 [llama2.c](https://github.com/karpathy/llama2.c) 저장소에서 모델을 다운로드합니다.

`$ make -j`

성공적으로 컴파일된 후 다음과 같은 사용 옵션이 사용 가능합니다:
```
usage: ./llama-convert-llama2c-to-ggml [options]

options:
  -h, --help                       show this help message and exit
  --copy-vocab-from-model FNAME    path of gguf llama model or llama2.c vocabulary from which to copy vocab (default 'models/7B/ggml-model-f16.gguf')
  --llama2c-model FNAME            [REQUIRED] model path from which to load Karpathy's llama2.c model
  --llama2c-output-model FNAME     model path to save the converted llama2.c model (default ak_llama_model.bin')
```

[karpathy/tinyllamas](https://huggingface.co/karpathy/tinyllamas)에서 가져온 모델을 사용하는 예시 명령어는 다음과 같습니다.

`$ ./llama-convert-llama2c-to-ggml --copy-vocab-from-model llama-2-7b-chat.gguf.q2_K.bin --llama2c-model stories42M.bin --llama2c-output-model stories42M.gguf.bin`

참고: `stories260K.bin`의 어휘는 [karpathy/tinyllamas/stories260K](https://huggingface.co/karpathy/tinyllamas/tree/main/stories260K)에서 찾을 수 있는 `tok512.bin`과 같은 독립적인 토크나이저를 사용해야 합니다.

이제 다음과 같은 명령어로 모델을 사용할 수 있습니다.

`$ ./llama-cli -m stories42M.gguf.bin -p "One day, Lily met a Shoggoth" -n 500 -c 256`
