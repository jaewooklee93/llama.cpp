# llama.cpp/example/simple

이 예제는 llama.cpp를 사용하여 주어진 프롬프트로 텍스트를 생성하는 최소한의 방법을 보여줍니다.

```bash
./simple -m ./models/llama-7b-v2/ggml-model-f16.gguf -p "Hello my name is"

...

main: n_len = 32, n_ctx = 2048, n_parallel = 1, n_kv_req = 32

 Hello my name is Shawn and I'm a 20 year old male from the United States. I'm a 20 year old

main: decoded 27 tokens in 2.31 s, speed: 11.68 t/s

llama_print_timings:        load time =   579.15 ms
llama_print_timings:      sample time =     0.72 ms /    28 runs   (    0.03 ms per token, 38888.89 tokens per second)
llama_print_timings: prompt eval time =   655.63 ms /    10 tokens (   65.56 ms per token,    15.25 tokens per second)
llama_print_timings:        eval time =  2180.97 ms /    27 runs   (   80.78 ms per token,    12.38 tokens per second)
llama_print_timings:       total time =  2891.13 ms
```
