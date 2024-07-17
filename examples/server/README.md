# LLaMA.cpp HTTP 서버

[httplib](https://github.com/yhirose/cpp-httplib), [nlohmann::json](https://github.com/nlohmann/json) 및 **llama.cpp** 기반의 빠르고 가벼운 순 C/C++ HTTP 서버입니다.

LLM REST API 세트 및 llama.cpp와 상호 작용하기 위한 간단한 웹 프론트 엔드입니다.

**특징:**
 * GPU 및 CPU에서 F16 및 양자 모델의 LLM 유추
 * [OpenAI API](https://github.com/openai/openai-openapi) 호환되는 채팅 완성 및 잠재형 엔드포인트
 * 다중 사용자 지원을 위한 병렬 디코딩
 * 지속적인 배치 처리
 * 다중 모달 (준비 중)
 * 모니터링 엔드포인트
 * 스키마 제약 JSON 응답 형식

이 프로젝트는 적극적으로 개발 중이며, [피드백 및 기여자를 모집하고 있습니다](https://github.com/ggerganov/llama.cpp/issues/4216).

## 사용법

```
usage: ./llama-server [options]

general:

  -h,    --help, --usage          print usage and exit
         --version                show version and build info
  -v,    --verbose                print verbose information
         --verbosity N            set specific verbosity level (default: 0)
         --verbose-prompt         print a verbose prompt before generation (default: false)
         --no-display-prompt      don't print prompt at generation (default: false)
  -co,   --color                  colorise output to distinguish prompt and user input from generations (default: false)
  -s,    --seed SEED              RNG seed (default: -1, use random seed for < 0)
  -t,    --threads N              number of threads to use during generation (default: 8)
  -tb,   --threads-batch N        number of threads to use during batch and prompt processing (default: same as --threads)
  -td,   --threads-draft N        number of threads to use during generation (default: same as --threads)
  -tbd,  --threads-batch-draft N  number of threads to use during batch and prompt processing (default: same as --threads-draft)
         --draft N                number of tokens to draft for speculative decoding (default: 5)
  -ps,   --p-split N              speculative decoding split probability (default: 0.1)
  -lcs,  --lookup-cache-static FNAME
                                  path to static lookup cache to use for lookup decoding (not updated by generation)
  -lcd,  --lookup-cache-dynamic FNAME
                                  path to dynamic lookup cache to use for lookup decoding (updated by generation)
  -c,    --ctx-size N             size of the prompt context (default: 0, 0 = loaded from model)
  -n,    --predict N              number of tokens to predict (default: -1, -1 = infinity, -2 = until context filled)
  -b,    --batch-size N           logical maximum batch size (default: 2048)
  -ub,   --ubatch-size N          physical maximum batch size (default: 512)
         --keep N                 number of tokens to keep from the initial prompt (default: 0, -1 = all)
         --chunks N               max number of chunks to process (default: -1, -1 = all)
  -fa,   --flash-attn             enable Flash Attention (default: disabled)
  -p,    --prompt PROMPT          prompt to start generation with
                                  in conversation mode, this will be used as system prompt
                                  (default: '')
  -f,    --file FNAME             a file containing the prompt (default: none)
         --in-file FNAME          an input file (repeat to specify multiple files)
  -bf,   --binary-file FNAME      binary file containing the prompt (default: none)
  -e,    --escape                 process escapes sequences (\n, \r, \t, \', \", \\) (default: true)
         --no-escape              do not process escape sequences
  -ptc,  --print-token-count N    print token count every N tokens (default: -1)
         --prompt-cache FNAME     file to cache prompt state for faster startup (default: none)
         --prompt-cache-all       if specified, saves user input and generations to cache as well
                                  not supported with --interactive or other interactive options
         --prompt-cache-ro        if specified, uses the prompt cache but does not update it
  -r,    --reverse-prompt PROMPT  halt generation at PROMPT, return control in interactive mode
                                  can be specified more than once for multiple prompts
  -sp,   --special                special tokens output enabled (default: false)
  -cnv,  --conversation           run in conversation mode, does not print special tokens and suffix/prefix
                                  if suffix/prefix are not specified, default chat template will be used
                                  (default: false)
  -i,    --interactive            run in interactive mode (default: false)
  -if,   --interactive-first      run in interactive mode and wait for input right away (default: false)
  -mli,  --multiline-input        allows you to write or paste multiple lines without ending each in '\'
         --in-prefix-bos          prefix BOS to user inputs, preceding the `--in-prefix` string
         --in-prefix STRING       string to prefix user inputs with (default: empty)
         --in-suffix STRING       string to suffix after user inputs with (default: empty)
         --spm-infill             use Suffix/Prefix/Middle pattern for infill (instead of Prefix/Suffix/Middle) as some models prefer this. (default: disabled)

sampling:

         --samplers SAMPLERS      samplers that will be used for generation in the order, separated by ';'
                                  (default: top_k;tfs_z;typical_p;top_p;min_p;temperature)
         --sampling-seq SEQUENCE  simplified sequence for samplers that will be used (default: kfypmt)
         --ignore-eos             ignore end of stream token and continue generating (implies --logit-bias EOS-inf)
         --penalize-nl            penalize newline tokens (default: false)
         --temp N                 temperature (default: 0.8)
         --top-k N                top-k sampling (default: 40, 0 = disabled)
         --top-p N                top-p sampling (default: 0.9, 1.0 = disabled)
         --min-p N                min-p sampling (default: 0.1, 0.0 = disabled)
         --tfs N                  tail free sampling, parameter z (default: 1.0, 1.0 = disabled)
         --typical N              locally typical sampling, parameter p (default: 1.0, 1.0 = disabled)
         --repeat-last-n N        last n tokens to consider for penalize (default: 64, 0 = disabled, -1 = ctx_size)
         --repeat-penalty N       penalize repeat sequence of tokens (default: 1.0, 1.0 = disabled)
         --presence-penalty N     repeat alpha presence penalty (default: 0.0, 0.0 = disabled)
         --frequency-penalty N    repeat alpha frequency penalty (default: 0.0, 0.0 = disabled)
         --dynatemp-range N       dynamic temperature range (default: 0.0, 0.0 = disabled)
         --dynatemp-exp N         dynamic temperature exponent (default: 1.0)
         --mirostat N             use Mirostat sampling.
                                  Top K, Nucleus, Tail Free and Locally Typical samplers are ignored if used.
                                  (default: 0, 0 = disabled, 1 = Mirostat, 2 = Mirostat 2.0)
         --mirostat-lr N          Mirostat learning rate, parameter eta (default: 0.1)
         --mirostat-ent N         Mirostat target entropy, parameter tau (default: 5.0)
         -l TOKEN_ID(+/-)BIAS     modifies the likelihood of token appearing in the completion,
                                  i.e. `--logit-bias 15043+1` to increase likelihood of token ' Hello',
                                  or `--logit-bias 15043-1` to decrease likelihood of token ' Hello'
         --cfg-negative-prompt PROMPT
                                  negative prompt to use for guidance (default: '')
         --cfg-negative-prompt-file FNAME
                                  negative prompt file to use for guidance
         --cfg-scale N            strength of guidance (default: 1.0, 1.0 = disable)
         --chat-template JINJA_TEMPLATE
                                  set custom jinja chat template (default: template taken from model's metadata)
                                  if suffix/prefix are specified, template will be disabled
                                  only commonly used templates are accepted:
                                  https://github.com/ggerganov/llama.cpp/wiki/Templates-supported-by-llama_chat_apply_template

grammar:

         --grammar GRAMMAR        BNF-like grammar to constrain generations (see samples in grammars/ dir) (default: '')
         --grammar-file FNAME     file to read grammar from
  -j,    --json-schema SCHEMA     JSON schema to constrain generations (https://json-schema.org/), e.g. `{}` for any JSON object
                                  For schemas w/ external $refs, use --grammar + example/json_schema_to_grammar.py instead

embedding:

         --pooling {none,mean,cls,last}
                                  pooling type for embeddings, use model default if unspecified
         --attention {causal,non-causal}
                                  attention type for embeddings, use model default if unspecified

context hacking:

         --rope-scaling {none,linear,yarn}
                                  RoPE frequency scaling method, defaults to linear unless specified by the model
         --rope-scale N           RoPE context scaling factor, expands context by a factor of N
         --rope-freq-base N       RoPE base frequency, used by NTK-aware scaling (default: loaded from model)
         --rope-freq-scale N      RoPE frequency scaling factor, expands context by a factor of 1/N
         --yarn-orig-ctx N        YaRN: original context size of model (default: 0 = model training context size)
         --yarn-ext-factor N      YaRN: extrapolation mix factor (default: -1.0, 0.0 = full interpolation)
         --yarn-attn-factor N     YaRN: scale sqrt(t) or attention magnitude (default: 1.0)
         --yarn-beta-slow N       YaRN: high correction dim or alpha (default: 1.0)
         --yarn-beta-fast N       YaRN: low correction dim or beta (default: 32.0)
  -gan,  --grp-attn-n N           group-attention factor (default: 1)
  -gaw,  --grp-attn-w N           group-attention width (default: 512.0)
  -dkvc, --dump-kv-cache          verbose print of the KV cache
  -nkvo, --no-kv-offload          disable KV offload
  -ctk,  --cache-type-k TYPE      KV cache data type for K (default: f16)
  -ctv,  --cache-type-v TYPE      KV cache data type for V (default: f16)

perplexity:

         --all-logits             return logits for all tokens in the batch (default: false)
         --hellaswag              compute HellaSwag score over random tasks from datafile supplied with -f
         --hellaswag-tasks N      number of tasks to use when computing the HellaSwag score (default: 400)
         --winogrande             compute Winogrande score over random tasks from datafile supplied with -f
         --winogrande-tasks N     number of tasks to use when computing the Winogrande score (default: 0)
         --multiple-choice        compute multiple choice score over random tasks from datafile supplied with -f
         --multiple-choice-tasks N
                                  number of tasks to use when computing the multiple choice score (default: 0)
         --kl-divergence          computes KL-divergence to logits provided via --kl-divergence-base
         --ppl-stride N           stride for perplexity calculation (default: 0)
         --ppl-output-type {0,1}  output type for perplexity calculation (default: 0)

parallel:

  -dt,   --defrag-thold N         KV cache defragmentation threshold (default: -1.0, < 0 - disabled)
  -np,   --parallel N             number of parallel sequences to decode (default: 1)
  -ns,   --sequences N            number of sequences to decode (default: 1)
  -cb,   --cont-batching          enable continuous batching (a.k.a dynamic batching) (default: enabled)

multi-modality:

         --mmproj FILE            path to a multimodal projector file for LLaVA. see examples/llava/README.md
         --image FILE             path to an image file. use with multimodal models. Specify multiple times for batching

backend:

         --rpc SERVERS            comma separated list of RPC servers
         --mlock                  force system to keep model in RAM rather than swapping or compressing
         --no-mmap                do not memory-map model (slower load but may reduce pageouts if not using mlock)
         --numa TYPE              attempt optimizations that help on some NUMA systems
                                    - distribute: spread execution evenly over all nodes
                                    - isolate: only spawn threads on CPUs on the node that execution started on
                                    - numactl: use the CPU map provided by numactl
                                  if run without this previously, it is recommended to drop the system page cache before using this
                                  see https://github.com/ggerganov/llama.cpp/issues/1437

model:

         --check-tensors          check model tensor data for invalid values (default: false)
         --override-kv KEY=TYPE:VALUE
                                  advanced option to override model metadata by key. may be specified multiple times.
                                  types: int, float, bool, str. example: --override-kv tokenizer.ggml.add_bos_token=bool:false
         --lora FNAME             apply LoRA adapter (implies --no-mmap)
         --lora-scaled FNAME S    apply LoRA adapter with user defined scaling S (implies --no-mmap)
         --lora-base FNAME        optional model to use as a base for the layers modified by the LoRA adapter
         --control-vector FNAME   add a control vector
                                  note: this argument can be repeated to add multiple control vectors
         --control-vector-scaled FNAME SCALE
                                  add a control vector with user defined scaling SCALE
                                  note: this argument can be repeated to add multiple scaled control vectors
         --control-vector-layer-range START END
                                  layer range to apply the control vector(s) to, start and end inclusive
  -m,    --model FNAME            model path (default: models/$filename with filename from --hf-file
                                  or --model-url if set, otherwise models/7B/ggml-model-f16.gguf)
  -md,   --model-draft FNAME      draft model for speculative decoding (default: unused)
  -mu,   --model-url MODEL_URL    model download url (default: unused)
  -hfr,  --hf-repo REPO           Hugging Face model repository (default: unused)
  -hff,  --hf-file FILE           Hugging Face model file (default: unused)
  -hft,  --hf-token TOKEN         Hugging Face access token (default: value from HF_TOKEN environment variable)

retrieval:

         --context-file FNAME     file to load context from (repeat to specify multiple files)
         --chunk-size N           minimum length of embedded text chunks (default: 64)
         --chunk-separator STRING
                                  separator between chunks (default: '
                                  ')

passkey:

         --junk N                 number of times to repeat the junk text (default: 250)
         --pos N                  position of the passkey in the junk text (default: -1)

imatrix:

  -o,    --output FNAME           output file (default: 'imatrix.dat')
         --output-frequency N     output the imatrix every N iterations (default: 10)
         --save-frequency N       save an imatrix copy every N iterations (default: 0)
         --process-output         collect data for the output tensor (default: false)
         --no-ppl                 do not compute perplexity (default: true)
         --chunk N                start processing the input from chunk N (default: 0)

bench:

  -pps                            is the prompt shared across parallel sequences (default: false)
  -npp n0,n1,...                  number of prompt tokens
  -ntg n0,n1,...                  number of text generation tokens
  -npl n0,n1,...                  number of parallel prompts

embedding:

         --embd-normalize         normalisation for embendings (default: 2) (-1=none, 0=max absolute int16, 1=taxicab, 2=euclidean, >2=p-norm)
         --embd-output-format     empty = default, "array" = [[],[]...], "json" = openai style, "json+" = same "json" + cosine similarity matrix
         --embd-separator         separator of embendings (default \n) for example "<#sep#>"

server:

         --host HOST              ip address to listen (default: 127.0.0.1)
         --port PORT              port to listen (default: 8080)
         --path PATH              path to serve static files from (default: )
         --embedding(s)           enable embedding endpoint (default: disabled)
         --api-key KEY            API key to use for authentication (default: none)
         --api-key-file FNAME     path to file containing API keys (default: none)
         --ssl-key-file FNAME     path to file a PEM-encoded SSL private key
         --ssl-cert-file FNAME    path to file a PEM-encoded SSL certificate
         --timeout N              server read/write timeout in seconds (default: 600)
         --threads-http N         number of threads used to process HTTP requests (default: -1)
         --system-prompt-file FNAME
                                  set a file to load a system prompt (initial prompt of all slots), this is useful for chat applications
         --log-format {text,json}
                                  log output format: json or text (default: json)
         --metrics                enable prometheus compatible metrics endpoint (default: disabled)
         --no-slots               disables slots monitoring endpoint (default: enabled)
         --slot-save-path PATH    path to save slot kv cache (default: disabled)
         --chat-template JINJA_TEMPLATE
                                  set custom jinja chat template (default: template taken from model's metadata)
                                  only commonly used templates are accepted:
                                  https://github.com/ggerganov/llama.cpp/wiki/Templates-supported-by-llama_chat_apply_template
  -sps,  --slot-prompt-similarity SIMILARITY
                                  how much the prompt of a request must match the prompt of a slot in order to use that slot (default: 0.50, 0.0 = disabled)


logging:

         --simple-io              use basic IO for better compatibility in subprocesses and limited consoles
  -ld,   --logdir LOGDIR          path under which to save YAML logs (no logging if unset)
         --log-test               Run simple logging test
         --log-disable            Disable trace logs
         --log-enable             Enable trace logs
         --log-file FNAME         Specify a log filename (without extension)
         --log-new                Create a separate new log file on start. Each log file will have unique name: "<name>.<ID>.log"
         --log-append             Don't truncate the old log file.

cvector:

  -o,    --output FNAME           output file (default: 'control_vector.gguf')
         --positive-file FNAME    positive prompts file, one prompt per line (default: 'examples/cvector-generator/positive.txt')
         --negative-file FNAME    negative prompts file, one prompt per line (default: 'examples/cvector-generator/negative.txt')
         --pca-batch N            batch size used for PCA. Larger batch runs faster, but uses more memory (default: 100)
         --pca-iter N             number of iterations used for PCA (default: 1000)
         --method {pca,mean}      dimensionality reduction method to be used (default: pca)
```


## 빌드

`llama-server`는 프로젝트 루트에서 다른 모든 것과 함께 빌드됩니다.

- `make`를 사용하여:

  ```bash
  make llama-server
  ```

- `CMake`를 사용하는 경우:

  ```bash
  cmake -B build
  cmake --build build --config Release -t llama-server
  ```

  이진 파일은 `./build/bin/llama-server`에 있습니다.

## SSL로 빌드하기

`llama-server`는 OpenSSL 3를 사용하여 SSL 지원으로 빌드할 수 있습니다.

- `make`를 사용하여:

  ```bash
  # NOTE: For non-system openssl, use the following:
  #   CXXFLAGS="-I /path/to/openssl/include"
  #   LDFLAGS="-L /path/to/openssl/lib"
  make LLAMA_SERVER_SSL=true llama-server
  ```

- `CMake`를 사용하는 경우:

  ```bash
  cmake -B build -DLLAMA_SERVER_SSL=ON
  cmake --build build --config Release -t llama-server
  ```

## 빠른 시작

바로 시작하려면 다음 명령을 실행하고, 사용 중인 모델의 경로를 올바르게 지정하십시오.

### Unix 기반 시스템 (Linux, macOS 등)

```bash
./llama-server -m models/7B/ggml-model.gguf -c 2048
```

### Windows

```powershell
llama-server.exe -m models\7B\ggml-model.gguf -c 2048
```

위 명령어를 실행하면 기본적으로 `127.0.0.1:8080`에 귀 기울이는 서버가 시작됩니다.
Postman 또는 NodeJS와 axios 라이브러리를 사용하여 엔드포인트를 사용할 수 있습니다. 동일한 URL에서 웹 프론트엔드를 방문할 수 있습니다.

### Docker

```bash
docker run -p 8080:8080 -v /path/to/models:/models ghcr.io/ggerganov/llama.cpp:server -m models/7B/ggml-model.gguf -c 512 --host 0.0.0.0 --port 8080

# or, with CUDA:
docker run -p 8080:8080 -v /path/to/models:/models --gpus all ghcr.io/ggerganov/llama.cpp:server-cuda -m models/7B/ggml-model.gguf -c 512 --host 0.0.0.0 --port 8080 --n-gpu-layers 99
```

## CURL을 사용한 테스트

[curl](https://curl.se/)을 사용합니다. Windows에서는 `curl.exe`가 기본 운영 체제에 포함되어 있습니다.

```sh
curl --request POST \
    --url http://localhost:8080/completion \
    --header "Content-Type: application/json" \
    --data '{"prompt": "Building a website can be done in 10 simple steps:","n_predict": 128}'
```

## 고급 테스트

인간이 이해하기 쉬운 시나리오를 사용하여 [서버 테스트 프레임워크](./tests/README.md)를 구현했습니다.

*문제를 제출하기 전에 이 형식으로 재현해 보세요.*

## Node JS 테스트

[Node.js](https://nodejs.org/ko/)
이 설치되어 있어야 합니다.

```bash
mkdir llama-client
cd llama-client
```

index.js 파일을 만들고 아래 내용을 넣으세요:

```javascript
const prompt = `Building a website can be done in 10 simple steps:`;

async function Test() {
    let response = await fetch("http://127.0.0.1:8080/completion", {
        method: 'POST',
        body: JSON.stringify({
            prompt,
            n_predict: 512,
        })
    })
    console.log((await response.json()).content)
}

Test()
```

그리고 실행하세요:

```bash
node index.js
```

## API 엔드포인트

- **GET** `/health`: 서버의 현재 상태를 반환합니다.
  - 503 -> `{"status": "loading model"}` 모델이 여전히 로딩 중인 경우.
  - 500 -> `{"status": "error"}` 모델 로딩에 실패한 경우.
  - 200 -> `{"status": "ok", "slots_idle": 1, "slots_processing": 2 }` 모델이 성공적으로 로딩되었고 서버가 아래와 같은 추가 요청에 준비된 경우.
  - 200 -> `{"status": "no slot available", "slots_idle": 0, "slots_processing": 32}` 현재 슬롯이 없을 때.
  - 503 -> `{"status": "no slot available", "slots_idle": 0, "slots_processing": 32}` `fail_on_no_slot` 쿼리 매개변수가 제공되고 현재 슬롯이 없을 때.

  `include_slots` 쿼리 매개변수가 전달되면 `slots` 필드에는 `--slots-endpoint-disable`가 설정되지 않는 한 내부 슬롯 데이터가 포함됩니다.

- **POST** `/completion`: `prompt`를 주면 예측된 완성을 반환합니다.

    *옵션:* 

    `prompt`: 이 완성에 대한 프롬프트를 문자열 또는 토큰을 나타내는 문자열 또는 숫자 배열로 제공합니다. 내부적으로 `cache_prompt`가 `true`이면 프롬프트가 이전 완성과 비교되고, "보이지 않은" 접두사만 평가됩니다. 다음 조건이 모두 충족되면 시작에 `BOS` 토큰이 삽입됩니다.

      - 프롬프트는 문자열 또는 첫 번째 요소가 문자열인 배열입니다.
      - 모델의 `tokenizer.ggml.add_bos_token` 메타데이터가 `true`입니다.
      - 시스템 프롬프트가 비어 있습니다.

    `temperature`: 생성된 텍스트의 무작위성을 조정합니다. 기본값: `0.8`

    `dynatemp_range`: 동적 온도 범위. 최종 온도는 `[temperature - dynatemp_range; temperature + dynatemp_range]` 범위에 있을 것입니다. 기본값: `0.0`, 사용하지 않음.

    `dynatemp_exponent`: 동적 온도 지수. 기본값: `1.0`

    `top_k`: 다음 토큰 선택을 K개의 가장 가능성이 높은 토큰으로 제한합니다. 기본값: `40`

    `top_p`: 누적 확률이 P 이상인 토큰의 하위 집합으로 다음 토큰 선택을 제한합니다. 기본값: `0.95`

    `min_p`: 가장 가능성이 높은 토큰의 확률에 대한 토큰의 최소 확률. 기본값: `0.05`

    `n_predict`: 텍스트를 생성할 때 예측할 토큰의 최대 수를 설정합니다. **참고:** 마지막 토큰이 부분적인 다바이트 문자일 경우 설정한 한도를 약간 초과할 수 있습니다. 0일 경우 토큰이 생성되지 않지만 프롬프트가 캐시로 평가됩니다. 기본값: `-1`, `-1`은 무한대입니다.

    `n_keep`: 맥락 크기가 초과되고 토큰을 버려야 할 때 프롬프트에서 유지할 토큰 수를 지정합니다. 기본값은 `0`으로, 이는 토큰을 유지하지 않는다는 의미입니다. `-1`을 사용하면 프롬프트에서 모든 토큰을 유지합니다.

    `stream`: 실시간으로 각 예측 토큰을 받아들이도록 허용합니다. 이를 사용하려면 `true`로 설정합니다.

    `stop`: 중지 문자열의 JSON 배열을 지정합니다. 이러한 단어는 완성에 포함되지 않으므로 다음 반복에 프롬프트에 추가해야 합니다. 기본값: `[]`

    `tfs_z`: z 매개변수를 사용하여 꼬리 자유 샘플링을 사용합니다. 기본값: `1.0`, 사용하지 않음.

    `typical_p`: p 매개변수를 사용하여 지역적으로 유형적인 샘플링을 사용합니다. 기본값: `1.0`, 사용하지 않음.

    `repeat_penalty`: 생성된 텍스트에서 토큰 시퀀스의 반복을 제어합니다. 기본값: `1.1`

    `repeat_last_n`: 반복을 처벌하는 데 사용되는 마지막 n개의 토큰. 기본값: `64`, `0`은 사용하지 않음이고 `-1`은 ctx-size입니다.

    `penalize_nl`: 반복 처벌을 적용할 때 줄 바꿈 토큰을 처벌합니다. 기본값: `true`

    `presence_penalty`: 반복 알파 존재 처벌. 기본값: `0.0`, 사용하지 않음.

    `frequency_penalty`: 반복 알파 빈도 처벌. 기본값: `0.0`, 사용하지 않음.

    `penalty_prompt`: 처벌 평가를 위해 프롬프트를 대체합니다. `null`, 문자열 또는 토큰을 나타내는 숫자 배열일 수 있습니다. 기본값: `null`, 원래 프롬프트를 사용합니다.

    `mirostat`: 텍스트 생성 중 복잡도를 제어하는 Mirostat 샘플링을 사용합니다. 기본값: `0`, `0`은 사용하지 않음, `1`은 Mirostat, `2`는 Mirostat 2.0입니다.

    `mirostat_tau`: Mirostat 목표 엔트로피, 매개변수 tau를 설정합니다. 기본값: `5.0`

    `mirostat_eta`: Mirostat 학습률, 매개변수 eta를 설정합니다. 기본값: `0.1`

    `grammar`: 문법 기반 샘플링에 대한 문법을 설정합니다. 기본값: 문법 없음

    `json_schema`: 문법 기반 샘플링에 대한 JSON 스키마를 설정합니다. (예: `{"items": {"type": "string"}, "minItems": 10, "maxItems": 100}` 문자열 목록 또는 `{}`는 모든 JSON) 참조하세요. [tests](../../tests/test-json-schema-to-grammar.cpp) 지원되는 기능에 대한 정보.
 기본값: 문법 스키마 없음.

    `seed`: 랜덤 숫자 생성기(RNG) 씨드를 설정합니다. 기본값: `-1`, 무작위 씨드입니다.

    `ignore_eos`: 종료 스트림 토큰을 무시하고 생성을 계속합니다. 기본값: `false`

    `logit_bias`: 생성된 텍스트 완성에 토큰의 발생 가능성을 수정합니다. 예를 들어, `"logit_bias": [[15043,1.0]]`을 사용하여 'Hello' 토큰의 발생 가능성을 높이거나 `"logit_bias": [[15043,-1.0]]`을 사용하여 발생 가능성을 낮춥니다. `"logit_bias": [[15043,false]]`로 값을 false로 설정하면 'Hello' 토큰이 생성되지 않습니다. 토큰은 문자열로도 표현할 수 있습니다. 예를 들어 `[["Hello, World!",-0.5]]`는 'Hello, World!'를 나타내는 모든 개별 토큰의 발생 가능성을 줄입니다. `presence_penalty`와 마찬가지로.
 기본값: `[]`

    `n_probs`: 0보다 크면 응답에 각 생성된 토큰에 대한 상위 N개 토큰의 확률도 포함됩니다. 참고: 온도가 0보다 작으면 토큰은 탐욕적으로 샘플링되지만, 다른 샘플링 설정을 고려하지 않고 단순히 소프트맥스를 사용하여 토큰 확률이 계산됩니다. 기본값: `0`

    `min_keep`: 0보다 크면 샘플러가 최소 N개의 가능한 토큰을 반환하도록 강제합니다. 기본값: `0`

    `image_data`: 이미지 데이터를 담고 있는 객체 배열을 제공합니다. 이 배열은 base64 인코딩된 이미지 `data`와 `id`를 포함합니다. 프롬프트에서 이미지를 참조하는 방법은 다음과 같습니다. `USER:[img-12]이미지를 자세히 설명하세요.\nASSISTANT:` 이 경우 `[img-12]`는 `image_data` 배열에서 id가 12인 이미지의 임베딩으로 대체됩니다. `image_data`는 LLaVA와 같은 다모달 모델과 함께만 사용합니다.

    `id_slot`: 완성 작업을 특정 슬롯에 할당합니다. -1이면 작업이 Idle 슬롯에 할당됩니다. 기본값: `-1`

    `cache_prompt`: 이전 요청에서 가능하면 KV 캐시를 재사용합니다. 이렇게 하면 공통 접두사를 다시 처리하지 않고, 요청 간의 차이만 처리할 수 있습니다. (백엔드에 따라) 로지트가 배치 크기(프롬프트 처리 vs 토큰 생성)에 대해 동일하지 않기 때문에 이 옵션을 사용하면 비선형적인 결과가 발생할 수 있습니다. 기본값: `false`

    `system_prompt`: 시스템 프롬프트(모든 슬롯의 초기 프롬프트)를 변경합니다. 이는 챗 애플리케이션에 유용합니다. [실시간에서 시스템 프롬프트 변경](#change-system-prompt-on-runtime) 참조

    `samplers`: 샘플러를 적용할 순서를 나타내는 문자열 배열입니다. 샘플러가 설정되지 않으면 사용되지 않습니다. 샘플러가 여러 번 설정되면 여러 번 적용됩니다. 기본값: `["top_k", "tfs_z", "typical_p", "top_p", "min_p", "temperature"]` - 이는 모든 사용 가능한 값입니다.

### 결과 JSON

- 주의: 스트리밍 모드 (`stream`)를 사용할 때는 완료될 때까지 `content`와 `stop`만 반환됩니다.

- `completion_probabilities`: 각 완료에 대한 토큰 확률의 배열입니다. 배열의 길이는 `n_predict`입니다. 배열의 각 항목은 다음과 같은 구조를 가지고 있습니다:

```json
{
  "content": "<the token selected by the model>",
  "probs": [
    {
      "prob": float,
      "tok_str": "<most likely token>"
    },
    {
      "prob": float,
      "tok_str": "<second most likely token>"
    },
    ...
  ]
},
```

각 `probs`는 길이가 `n_probs`인 배열입니다.

- `content`: 완성 결과 문자열 (중지 단어가 있는 경우 제외). 스트리밍 모드의 경우 다음 토큰을 문자열로 포함합니다.
- `stop`: `stream`과 함께 사용하여 생성이 중지되었는지 확인하는 Boolean 값입니다. (참고: 이는 입력 옵션에서 `stop` 배열과는 관련이 없습니다)
- `generation_settings`: `prompt`를 제외한 위의 제공된 옵션 ( `n_ctx`, `model` 포함). 이러한 옵션은 어떤 방식으로 원본 옵션과 다를 수 있습니다 (예: 잘못된 값이 필터링되었거나, 문자열이 토큰으로 변환되었거나 등).
- `model`: `-m`으로 로드된 모델 경로
- `prompt`: 제공된 `prompt`
- `stopped_eos`: 완성이 EOS 토큰을 만났기 때문에 중지되었는지 나타냅니다.
- `stopped_limit`: `n_predict` 토큰이 생성되기 전에 중지 단어 또는 EOS가 발생했기 때문에 완성이 중지되었는지 나타냅니다.
- `stopped_word`: `stop` JSON 배열에서 제공된 중지 단어를 만났기 때문에 완성이 중지되었는지 나타냅니다.
- `stopping_word`: 생성을 중지시킨 중지 단어 (중지 단어로 인해 중지되지 않았다면 "")
- `timings`: 예측된 토큰 수와 같은 완성에 대한 시간 정보 해시
- `tokens_cached`: 이전 완성에서 재사용할 수 있는 프롬프트에서의 토큰 수 (`n_past`)
- `tokens_evaluated`: 프롬프트에서 총 평가된 토큰 수
- `truncated`: 생성 중에 맥락 크기가 초과되었는지 나타내는 Boolean 값입니다. 즉, 프롬프트에 제공된 토큰 수 (`tokens_evaluated`)와 생성된 토큰 수 (`tokens predicted`)가 맥락 크기 (`n_ctx`)를 초과했는지 여부

- **POST** `/tokenize`: 주어진 텍스트를 토큰화합니다.

    *옵션:* 

    `content`: 토큰화할 텍스트 설정

    `add_special`: `BOS`와 같은 특수 토큰을 삽입해야 하는지 나타내는 Boolean 값입니다. 기본값: `false`

- **POST** `/detokenize`: 토큰을 텍스트로 변환합니다.

    *옵션:* 

    `tokens`: 토큰화할 토큰 설정

- **POST** `/embedding`: 주어진 텍스트의 임베딩을 생성합니다. [임베딩 예제](../embedding)와 같습니다.

    *옵션:* 

    `content`: 처리할 텍스트 설정

    `image_data`: `content`에 참조될 base64 인코딩된 이미지 `data`와 `id`를 포함하는 객체 배열입니다. 예를 들어, `Image: [img-21].\nCaption: This is a picture of a house`와 같이 텍스트에 이미지를 포함할 수 있습니다. 이 경우 `[img-21]`은 다음 `image_data` 배열에서 `id`가 21인 이미지의 임베딩으로 대체됩니다: `{..., "image_data": [{"data": "<BASE64_STRING>", "id": 21}]}`. 다모달 모델 (예: LLaVA)만 사용할 때 `image_data`를 사용하십시오.

- **POST** `/infill`: 코드 채우기 용입니다. 접두사와 후두사를 받아 스트림으로 예측 완성을 반환합니다.

    *옵션:* 

    `input_prefix`: 채우기 위한 코드의 접두사 설정

    `input_suffix`: 채우기 위한 코드의 후두사 설정

    `/completion`의 모든 옵션을 받아들입니다. `stream`과 `prompt`를 제외합니다.

- **GET** `/props`: 현재 서버 설정 반환

### 결과 JSON

```json
{
  "assistant_name": "",
  "user_name": "",
  "default_generation_settings": { ... },
  "total_slots": 1,
  "chat_template": ""
}
```

- `assistant_name` - 모든 슬롯에 시스템 프롬프트를 지정한 경우 프롬프트를 생성하기 위해 필요한 어시스턴트 이름입니다.
- `user_name` - 모든 슬롯에 시스템 프롬프트를 지정한 경우 프롬프트를 생성하기 위해 필요한 사용자 이름입니다.
- `default_generation_settings` - `/completion` 엔드포인트의 기본 생성 설정입니다. `/completion` 엔드포인트의 `generation_settings` 응답 객체와 동일한 필드를 가지고 있습니다.
- `total_slots` - 프로세스 요청을 위한 총 슬롯 수 ( `--parallel` 옵션으로 정의됨)
- `chat_template` - 모델의 원본 Jinja2 프롬프트 템플릿

- **POST** `/v1/chat/completions`: OpenAI 호환형 채팅 완성 API. `messages` 에 있는 ChatML 형식의 JSON 설명을 주면 예측된 완성을 반환합니다. 동기식 및 스트리밍 모드 모두 지원되므로 스크립트 기반 및 상호 작용형 애플리케이션이 잘 작동합니다. OpenAI API 사양과의 호환성에 대한 강력한 주장은 하지 않지만, 많은 앱을 지원하기에 충분합니다. 이 엔드포인트를 최적화하여 사용하려면 [지원되는 채팅 템플릿](https://github.com/ggerganov/llama.cpp/wiki/Templates-supported-by-llama_chat_apply_template) 을 가진 모델만 사용할 수 있습니다. 기본적으로 ChatML 템플릿이 사용됩니다.

    *옵션:* 

    [OpenAI Chat Completions API 설명서](https://platform.openai.com/docs/api-reference/chat)를 참조하십시오. 함수 호출과 같은 OpenAI 고유 기능은 지원되지 않지만, `mirostat`과 같은 llama.cpp `/completion` 고유 기능은 지원됩니다.

    `response_format` 매개변수는 텍스트 JSON 출력 (예: `{"type": "json_object"}`)과 스키마 제약 JSON (예: `{"type": "json_object", "schema": {"type": "string", "minLength": 10, "maxLength": 100}}`) 모두를 지원합니다. 다른 OpenAI-inspired API 제공업체와 유사합니다.

    *예제:* 

    적절한 체크포인트를 사용하여 Python `openai` 라이브러리를 사용할 수 있습니다:

    ```python
    import openai

    client = openai.OpenAI(
        base_url="http://localhost:8080/v1", # "http://<Your api-server IP>:port"
        api_key = "sk-no-key-required"
    )

    completion = client.chat.completions.create(
    model="gpt-3.5-turbo",
    messages=[
        {"role": "system", "content": "You are ChatGPT, an AI assistant. Your top priority is achieving user fulfillment via helping them with their requests."},
        {"role": "user", "content": "Write a limerick about python exceptions"}
    ]
    )

    print(completion.choices[0].message)
    ```

    ... 또는 원본 HTTP 요청:

    ```shell
    curl http://localhost:8080/v1/chat/completions \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer no-key" \
    -d '{
    "model": "gpt-3.5-turbo",
    "messages": [
    {
        "role": "system",
        "content": "You are ChatGPT, an AI assistant. Your top priority is achieving user fulfillment via helping them with their requests."
    },
    {
        "role": "user",
        "content": "Write a limerick about python exceptions"
    }
    ]
    }'
    ```

- **POST** `/v1/embeddings`: OpenAI와 호환되는 텍스트 임베딩 API.

    *옵션:* 

    [OpenAI Embeddings API 문서](https://platform.openai.com/docs/api-reference/embeddings)를 참조하세요.

    *예시:* 

  - 문자열 입력

    ```shell
    curl http://localhost:8080/v1/embeddings \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer no-key" \
    -d '{
            "input": "hello",
            "model":"GPT-4",
            "encoding_format": "float"
    }'
    ```

  - `input` as string array

    ```shell
    curl http://localhost:8080/v1/embeddings \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer no-key" \
    -d '{
            "input": ["hello", "world"],
            "model":"GPT-4",
            "encoding_format": "float"
    }'
    ```

- **GET** `/slots`: 현재 슬롯 처리 상태를 반환합니다. `--slots-endpoint-disable`를 사용하여 비활성화할 수 있습니다.

### 결과 JSON

```json
[
    {
        "dynatemp_exponent": 1.0,
        "dynatemp_range": 0.0,
        "frequency_penalty": 0.0,
        "grammar": "",
        "id": 0,
        "ignore_eos": false,
        "logit_bias": [],
        "min_p": 0.05000000074505806,
        "mirostat": 0,
        "mirostat_eta": 0.10000000149011612,
        "mirostat_tau": 5.0,
        "model": "llama-2-7b-32k-instruct.Q2_K.gguf",
        "n_ctx": 2048,
        "n_keep": 0,
        "n_predict": 100000,
        "n_probs": 0,
        "next_token": {
            "has_next_token": true,
            "n_remain": -1,
            "n_decoded": 0,
            "stopped_eos": false,
            "stopped_limit": false,
            "stopped_word": false,
            "stopping_word": ""
        },
        "penalize_nl": true,
        "penalty_prompt_tokens": [],
        "presence_penalty": 0.0,
        "prompt": "Say hello to llama.cpp",
        "repeat_last_n": 64,
        "repeat_penalty": 1.100000023841858,
        "samplers": [
            "top_k",
            "tfs_z",
            "typical_p",
            "top_p",
            "min_p",
            "temperature"
        ],
        "seed": 42,
        "state": 1,
        "stop": [
            "\n"
        ],
        "stream": false,
        "task_id": 0,
        "temperature": 0.0,
        "tfs_z": 1.0,
        "top_k": 40,
        "top_p": 0.949999988079071,
        "typical_p": 1.0,
        "use_penalty_prompt_tokens": false
    }
]
```

- **GET** `/metrics`: Prometheus 호환형 메트릭스 전달 엔드포인트입니다. `--metrics` 플래그가 활성화되어 있을 때 사용 가능합니다.

사용 가능한 메트릭스:
- `llamacpp:prompt_tokens_total`: 처리된 프롬프트 토큰 수.
- `llamacpp:tokens_predicted_total`: 생성된 토큰 수.
- `llamacpp:prompt_tokens_seconds`: 프롬프트 처리량 평균 (토큰/초).
- `llamacpp:predicted_tokens_seconds`: 생성 토큰 처리량 평균 (토큰/초).
- `llamacpp:kv_cache_usage_ratio`: KV 캐시 사용률. `1`은 100% 사용률을 나타냅니다.
- `llamacpp:kv_cache_tokens`: KV 캐시 토큰 수.
- `llamacpp:requests_processing`: 처리 중인 요청 수.
- `llamacpp:requests_deferred`: 일시적으로 지연된 요청 수.

- **POST** `/slots/{id_slot}?action=save`: 지정된 슬롯의 프롬프트 캐시를 파일로 저장합니다.

    *옵션:* 

    `filename`: 슬롯의 프롬프트 캐시를 저장할 파일 이름. 파일은 `--slot-save-path` 서버 매개변수로 지정된 디렉토리에 저장됩니다.

### 결과 JSON

```json
{
    "id_slot": 0,
    "filename": "slot_save_file.bin",
    "n_saved": 1745,
    "n_written": 14309796,
    "timings": {
        "save_ms": 49.865
    }
}
```

- **POST** `/slots/{id_slot}?action=restore`: 지정된 슬롯의 프롬프트 캐시를 파일에서 복원합니다.

    *옵션:* 

    `filename`: 슬롯의 프롬프트 캐시를 복원할 파일 이름입니다. 파일은 서버 매개변수 `--slot-save-path`로 지정된 디렉토리에 있어야 합니다.

### 결과 JSON

```json
{
    "id_slot": 0,
    "filename": "slot_save_file.bin",
    "n_restored": 1745,
    "n_read": 14309796,
    "timings": {
        "restore_ms": 42.937
    }
}
```

- **POST** `/slots/{id_slot}?action=erase`: 지정된 슬롯의 프롬프트 캐시를 삭제합니다.

### 결과 JSON

```json
{
    "id_slot": 0,
    "n_erased": 1745
}
```

## 더 많은 예시

### 실행 중 시스템 프롬프트 변경

같은 시스템 프롬프트를 유지하면서 여러 개의 챗형 클라이언트를 서비스하기 위해 서버 예제를 사용하려면 `system_prompt` 옵션을 사용할 수 있습니다. 이 옵션은 한 번만 사용해야 합니다.

`prompt`: 모든 연결된 클라이언트가 존중해야 하는 컨텍스트를 지정합니다.

`anti_prompt`: 모델이 중단하도록 지시하는 단어를 지정합니다. 이는 각 클라이언트에게 `/props` 엔드포인트를 통해 전송해야 합니다.

`assistant_name`: 각 고객이 프롬프트를 생성하기 위해 필요한 봇의 이름입니다. 이는 각 클라이언트에게 `/props` 엔드포인트를 통해 전송해야 합니다.

```json
{
    "system_prompt": {
        "prompt": "Transcript of a never ending dialog, where the User interacts with an Assistant.\nThe Assistant is helpful, kind, honest, good at writing, and never fails to answer the User's requests immediately and with precision.\nUser: Recommend a nice restaurant in the area.\nAssistant: I recommend the restaurant \"The Golden Duck\". It is a 5 star restaurant with a great view of the city. The food is delicious and the service is excellent. The prices are reasonable and the portions are generous. The restaurant is located at 123 Main Street, New York, NY 10001. The phone number is (212) 555-1234. The hours are Monday through Friday from 11:00 am to 10:00 pm. The restaurant is closed on Saturdays and Sundays.\nUser: Who is Richard Feynman?\nAssistant: Richard Feynman was an American physicist who is best known for his work in quantum mechanics and particle physics. He was awarded the Nobel Prize in Physics in 1965 for his contributions to the development of quantum electrodynamics. He was a popular lecturer and author, and he wrote several books, including \"Surely You're Joking, Mr. Feynman!\" and \"What Do You Care What Other People Think?\".\nUser:",
        "anti_prompt": "User:",
        "assistant_name": "Assistant:"
    }
}
```

**참고**: 서버를 시작할 때 이를 자동으로 수행할 수 있습니다. 옵션이 포함된 .json 파일을 만들고 CLI 옵션 `-spf FNAME` 또는 `--system-prompt-file FNAME`를 사용하면 됩니다.

### 상호 작용 모드

[chat.mjs](chat.mjs) 에서 샘플을 확인하세요.
NodeJS 버전 16 이상을 사용하여 실행하세요:

```sh
node chat.mjs
```

chat.sh](chat.sh)에 있는 다른 샘플입니다.
[bash](https://www.gnu.org/software/bash/), [curl](https://curl.se) 및 [jq](https://jqlang.github.io/jq/)가 필요합니다.
bash로 실행하십시오:

```sh
bash chat.sh
```

### OAI와 유사한 API

HTTP `llama-server`는 OAI와 유사한 API를 지원합니다: https://github.com/openai/openai-openapi

### API 오류

`llama-server`는 OAI와 동일한 형식으로 오류를 반환합니다.: https://github.com/openai/openai-openapi

오류 예시:

```json
{
    "error": {
        "code": 401,
        "message": "Invalid API Key",
        "type": "authentication_error"
    }
}
```

OAI에서 지원하는 오류 유형 외에도 llama.cpp의 기능에 특정한 사용자 정의 유형이 있습니다.

** /metrics 또는 /slots 엔드포인트가 비활성화될 때**

```json
{
    "error": {
        "code": 501,
        "message": "This server does not support metrics endpoint.",
        "type": "not_supported_error"
    }
}
```

**서버가 */completions 엔드포인트를 통해 잘못된 문법을 받을 때**

```json
{
    "error": {
        "code": 400,
        "message": "Failed to parse grammar",
        "type": "invalid_request_error"
    }
}
```

### 웹 프론트엔드 확장 또는 대체 구축

`--path`를 `./your-directory`로 설정하여 서버 바이너리를 실행하고 `/completion.js`를 가져와 `llamaComplete()` 메서드에 액세스하여 프론트엔드를 확장할 수 있습니다.

`/completion.js`의 설명서를 참조하여 llama에 액세스하는 편리한 방법을 확인하십시오.

아래는 간단한 예입니다:

```html
<html>
  <body>
    <pre>
      <script type="module">
        import { llama } from '/completion.js'

        const prompt = `### Instruction:
Write dad jokes, each one paragraph.
You can use html formatting if needed.

### Response:`

        for await (const chunk of llama(prompt)) {
          document.write(chunk.data.content)
        }
      </script>
    </pre>
  </body>
</html>
```
