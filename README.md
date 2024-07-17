# llama.cpp

![llama](https://user-images.githubusercontent.com/1991296/230134379-7181e485-c521-4d23-a0d6-f7b3b61ba524.png)

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Server](https://github.com/ggerganov/llama.cpp/actions/workflows/server.yml/badge.svg?branch=master&event=schedule)](https://github.com/ggerganov/llama.cpp/actions/workflows/server.yml)
[![Conan Center](https://shields.io/conan/v/llama-cpp)](https://conan.io/center/llama-cpp)

[Roadmap](https://github.com/users/ggerganov/projects/7) / [Project status](https://github.com/ggerganov/llama.cpp/discussions/3471) / [Manifesto](https://github.com/ggerganov/llama.cpp/discussions/205) / [ggml](https://github.com/ggerganov/ggml)

Meta의 [LLaMA](https://arxiv.org/abs/2302.13971) 모델(및 기타 모델)을 순수 C/C++에서 추론하는 소프트웨어

> [!중요]
[2024년 6월 12일] 바이너리 파일 이름이 `llama-` 접두사를 사용하여 변경되었습니다. `main`은 `llama-cli`로, `server`는 `llama-server`로 변경되었습니다. (https://github.com/ggerganov/llama.cpp/pull/7809)

## 최근 API 변경 사항

- [2024년 6월 26일] 소스 코드 및 CMake 빌드 스크립트가 재구성되었습니다. https://github.com/ggerganov/llama.cpp/pull/8006
- [2024년 4월 21일] `llama_token_to_piece`는 이제 특수 토큰을 렌더링할 수 있습니다. https://github.com/ggerganov/llama.cpp/pull/6807
- [2024년 4월 4일] 상태 및 세션 파일 함수가 `llama_state_*` 아래에서 재정렬되었습니다. https://github.com/ggerganov/llama.cpp/pull/6341
- [2024년 3월 26일] 로그와 임베딩 API가 간결해졌습니다. https://github.com/ggerganov/llama.cpp/pull/6122
- [2024년 3월 13일] `llama_synchronize()` 및 `llama_context_params.n_ubatch` 추가 https://github.com/ggerganov/llama.cpp/pull/6017
- [2024년 3월 8일] `llama_kv_cache_seq_rm()`은 `void` 대신 `bool`을 반환하고, 새로운 `llama_n_seq_max()`는 배치에서 허용 가능한 `seq_id`의 상한선을 반환합니다. (여러 문장을 처리할 때 관련됨) https://github.com/ggerganov/llama.cpp/pull/5328
- [2024년 3월 4일] 임베딩 API 업데이트 https://github.com/ggerganov/llama.cpp/pull/5796
- [2024년 3월 3일] `struct llama_context_params` https://github.com/ggerganov/llama.cpp/pull/5849

## 뜨거운 주제

- **`convert.py`는 `examples/convert_legacy_llama.py`로 이동되어 불사용되었습니다. `convert_hf_to_gguf.py`를 사용하세요** https://github.com/ggerganov/llama.cpp/pull/7430
- 초기 Flash-Attention 지원: https://github.com/ggerganov/llama.cpp/pull/5021
- BPE 사전 토큰화 지원이 추가되었습니다: https://github.com/ggerganov/llama.cpp/pull/6920
- MoE 메모리 레이아웃이 업데이트되었습니다 - `mmap` 지원을 위해 모델을 다시 변환하고 `imatrix`를 재생성하세요 https://github.com/ggerganov/llama.cpp/pull/6387
- `gguf-split`을 사용한 모델 조각화 지침 https://github.com/ggerganov/llama.cpp/discussions/6404
- Metal 배치 추론에서 주요 버그 수정 https://github.com/ggerganov/llama.cpp/pull/6225
- 다중 GPU 파이프라인 병렬 지원 https://github.com/ggerganov/llama.cpp/pull/6017
- Deepseek 지원 추가를 위한 기여를 요청합니다: https://github.com/ggerganov/llama.cpp/issues/5981
- 양자화 실명 테스트: https://github.com/ggerganov/llama.cpp/discussions/5962
- 초기 Mamba 지원이 추가되었습니다: https://github.com/ggerganov/llama.cpp/pull/5328

----

## 설명

`llama.cpp`의 주요 목표는 다양한 하드웨어에서 최소한의 설정과 최첨단 성능으로 LLM 추론을 가능하게 하는 것입니다.

- C/C++로 작성된 순수한 구현으로 의존성이 없습니다.
- Apple silicon은 ARM NEON, Accelerate 및 Metal 프레임워크를 통해 최적화된 최우선 지원 대상입니다.
- x86 아키텍처에 대한 AVX, AVX2 및 AVX512 지원
- 더 빠른 추론과 기억 사용량 감소를 위한 1.5비트, 2비트, 3비트, 4비트, 5비트, 6비트 및 8비트 정수 양자화
- NVIDIA GPU에서 LLM을 실행하기 위한 사용자 정의 CUDA 커널 (HIP을 통한 AMD GPU 지원)
- Vulkan 및 SYCL 백엔드 지원
- 전체 VRAM 용량보다 큰 모델을 부분적으로 가속화하기 위한 CPU+GPU 하이브리드 추론

[시작](https://github.com/ggerganov/llama.cpp/issues/33#issuecomment-1465108022) 이후로 프로젝트는 많은 기여 덕분에 크게 발전했습니다. `ggml` 라이브러리에 대한 새로운 기능 개발의 주요 플랫폼입니다.

**지원되는 모델:**

일반적으로 아래 기본 모델의 미세 조정된 버전도 지원됩니다.

- [X] LLaMA 🦙
- [x] LLaMA 2 🦙🦙
- [x] LLaMA 3 🦙🦙🦙
- [X] [Mistral 7B](https://huggingface.co/mistralai/Mistral-7B-v0.1)
- [x] [Mixtral MoE](https://huggingface.co/models?search=mistral-ai/Mixtral)
- [x] [DBRX](https://huggingface.co/databricks/dbrx-instruct)
- [X] [Falcon](https://huggingface.co/models?search=tiiuae/falcon)
- [X] [Chinese LLaMA / Alpaca](https://github.com/ymcui/Chinese-LLaMA-Alpaca) 및 [Chinese LLaMA-2 / Alpaca-2](https://github.com/ymcui/Chinese-LLaMA-Alpaca-2)
- [X] [Vigogne (프랑스어)](https://github.com/bofenghuang/vigogne)
- [X] [BERT](https://github.com/ggerganov/llama.cpp/pull/5423)
- [X] [Koala](https://bair.berkeley.edu/blog/2023/04/03/koala/)
- [X] [Baichuan 1 & 2](https://huggingface.co/models?search=baichuan-inc/Baichuan) + [파생](https://huggingface.co/hiyouga/baichuan-7b-sft)
- [X] [Aquila 1 & 2](https://huggingface.co/models?search=BAAI/Aquila)
- [X] [Starcoder 모델](https://github.com/ggerganov/llama.cpp/pull/3187)
- [X] [Refact](https://huggingface.co/smallcloudai/Refact-1_6B-fim)
- [X] [MPT](https://github.com/ggerganov/llama.cpp/pull/3417)
- [X] [Bloom](https://github.com/ggerganov/llama.cpp/pull/3553)
- [x] [Yi 모델](https://huggingface.co/models?search=01-ai/Yi)
- [X] [StableLM 모델](https://huggingface.co/stabilityai)
- [x] [Deepseek 모델](https://huggingface.co/models?search=deepseek-ai/deepseek)
- [x] [Qwen 모델](https://huggingface.co/models?search=Qwen/Qwen)
- [x] [PLaMo-13B](https://github.com/ggerganov/llama.cpp/pull/3557)
- [x] [Phi 모델](https://huggingface.co/models?search=microsoft/phi)
- [x] [GPT-2](https://huggingface.co/gpt2)
- [x] [Orion 14B](https://github.com/ggerganov/llama.cpp/pull/5118)
- [x] [InternLM2](https://huggingface.co/models?search=internlm2)
- [x] [CodeShell](https://github.com/WisdomShell/codeshell)
- [x] [Gemma](https://ai.google.dev/gemma)
- [x] [Mamba](https://github.com/state-spaces/mamba)
- [x] [Grok-1](https://huggingface.co/keyfan/grok-1-hf)
- [x] [Xverse](https://huggingface.co/models?search=xverse)
- [x] [Command-R 모델](https://huggingface.co/models?search=CohereForAI/c4ai-command-r)
- [x] [SEA-LION](https://huggingface.co/models?search=sea-lion)
- [x] [GritLM-7B](https://huggingface.co/GritLM/GritLM-7B) + [GritLM-8x7B](https://huggingface.co/GritLM/GritLM-8x7B)
- [x] [OLMo](https://allenai.org/olmo)
- [x] [GPT-NeoX](https://github.com/EleutherAI/gpt-neox) + [Pythia](https://github.com/EleutherAI/pythia)
- [x] [ChatGLM3-6b](https://huggingface.co/THUDM/chatglm3-6b) + [ChatGLM4-9b](https://huggingface.co/THUDM/glm-4-9b)

(더 많은 모델을 지원하는 방법에 대한 지침: [HOWTO-add-model.md](./docs/development/HOWTO-add-model.md))

**다중 모달 모델:**

- [x] [LLaVA 1.5 모델](https://huggingface.co/collections/liuhaotian/llava-15-653aac15d994e992e2677a7e), [LLaVA 1.6 모델](https://huggingface.co/collections/liuhaotian/llava-16-65b9e40155f60fd046a5ccf2)
- [x] [BakLLaVA](https://huggingface.co/models?search=SkunkworksAI/Bakllava)
- [x] [Obsidian](https://huggingface.co/NousResearch/Obsidian-3B-V0.5)
- [x] [ShareGPT4V](https://huggingface.co/models?search=Lin-Chen/ShareGPT4V)
- [x] [MobileVLM 1.7B/3B 모델](https://huggingface.co/models?search=mobileVLM)
- [x] [Yi-VL](https://huggingface.co/models?search=Yi-VL)
- [x] [Mini CPM](https://huggingface.co/models?search=MiniCPM)
- [x] [Moondream](https://huggingface.co/vikhyatk/moondream2)
- [x] [Bunny](https://github.com/BAAI-DCAI/Bunny)

**바인딩:**

- Python: [abetlen/llama-cpp-python](https://github.com/abetlen/llama-cpp-python)
- Go: [go-skynet/go-llama.cpp](https://github.com/go-skynet/go-llama.cpp)
- Node.js: [withcatai/node-llama-cpp](https://github.com/withcatai/node-llama-cpp)
- JS/TS (llama.cpp 서버 클라이언트): [lgrammel/modelfusion](https://modelfusion.dev/integration/model-provider/llamacpp)
- JavaScript/Wasm (브라우저에서 작동): [tangledgroup/llama-cpp-wasm](https://github.com/tangledgroup/llama-cpp-wasm)
- TypeScript/Wasm (더 좋은 API, npm에서 사용 가능): [ngxson/wllama](https://github.com/ngxson/wllama)
- Ruby: [yoshoku/llama_cpp.rb](https://github.com/yoshoku/llama_cpp.rb)
- Rust (더 많은 기능): [edgenai/llama_cpp-rs](https://github.com/edgenai/llama_cpp-rs)
- Rust (더 좋은 API): [mdrokz/rust-llama.cpp](https://github.com/mdrokz/rust-llama.cpp)
- Rust (더 직접적인 바인딩): [utilityai/llama-cpp-rs](https://github.com/utilityai/llama-cpp-rs)
- C#: [SciSharp/LLamaSharp](https://github.com/SciSharp/LLamaSharp)
- Scala 3: [donderom/llm4s](https://github.com/donderom/llm4s)
- Clojure: [phronmophobic/llama.clj](https://github.com/phronmophobic/llama.clj)
- React Native: [mybigday/llama.rn](https://github.com/mybigday/llama.rn)
- Java: [kherud/java-llama.cpp](https://github.com/kherud/java-llama.cpp)
- Zig: [deins/llama.cpp.zig](https://github.com/Deins/llama.cpp.zig)
- Flutter/Dart: [netdur/llama_cpp_dart](https://github.com/netdur/llama_cpp_dart)
- PHP (llama.cpp에 대한 API 바인딩 및 위에서 구축된 기능): [distantmagic/resonance](https://github.com/distantmagic/resonance) [(더 자세한 내용)](https://github.com/ggerganov/llama.cpp/pull/6326)
- Guile Scheme: [guile_llama_cpp](https://savannah.nongnu.org/projects/guile-llama-cpp)

**UI:**

다음 프로젝트는 일반적으로 오픈 소스이며 허용하는 라이센스를 사용합니다.

- [iohub/coLLaMA](https://github.com/iohub/coLLaMA)
- [janhq/jan](https://github.com/janhq/jan) (AGPL)
- [nat/openplayground](https://github.com/nat/openplayground)
- [Faraday](https://faraday.dev/) (사용자 지정)
- [LMStudio](https://lmstudio.ai/) (사용자 지정)
- [Layla](https://play.google.com/store/apps/details?id=com.laylalite) (사용자 지정)
- [LocalAI](https://github.com/mudler/LocalAI) (MIT)
- [LostRuins/koboldcpp](https://github.com/LostRuins/koboldcpp) (AGPL)
- [Mozilla-Ocho/llamafile](https://github.com/Mozilla-Ocho/llamafile)
- [nomic-ai/gpt4all](https://github.com/nomic-ai/gpt4all)
- [ollama/ollama](https://github.com/ollama/ollama)
- [oobabooga/text-generation-webui](https://github.com/oobabooga/text-generation-webui) (AGPL)
- [psugihara/freechat](https://github.com/psugihara/freechat)
- [cztomsik/ava](https://github.com/cztomsik/ava) (MIT)
- [ptsochantaris/emeltal](https://github.com/ptsochantaris/emeltal)
- [pythops/tenere](https://github.com/pythops/tenere) (AGPL)
- [RAGNA Desktop](https://ragna.app/) (사용자 지정)
- [RecurseChat](https://recurse.chat/) (사용자 지정)
- [semperai/amica](https://github.com/semperai/amica)
- [withcatai/catai](https://github.com/withcatai/catai)
- [Mobile-Artificial-Intelligence/maid](https://github.com/Mobile-Artificial-Intelligence/maid) (MIT)
- [Msty](https://msty.app) (사용자 지정)
- [LLMFarm](https://github.com/guinmoon/LLMFarm?tab=readme-ov-file)(MIT)
- [KanTV](https://github.com/zhouwg/kantv?tab=readme-ov-file)(Apachev2.0 or later)
- [Dot](https://github.com/alexpinel/Dot) (GPL)
- [MindMac](https://mindmac.app) (사용자 지정)
- [KodiBot](https://github.com/firatkiral/kodibot) (GPL)
- [eva](https://github.com/ylsdamxssjxxdd/eva) (MIT)
- [AI Sublime Text 플러그인](https://github.com/yaroslavyaroslav/OpenAI-sublime-text) (MIT)
- [AIKit](https://github.com/sozercan/aikit) (MIT)
- [LARS - The LLM & Advanced Referencing Solution](https://github.com/abgulati/LARS) (AGPL)

*(프로젝트가 `llama.cpp`에 의존한다고 명시되어야만 여기에 포함됩니다.)

**도구:**

- [akx/ggify](https://github.com/akx/ggify) – HuggingFace Hub에서 PyTorch 모델을 다운로드하여 GGML로 변환합니다.
- [crashr/gppm](https://github.com/crashr/gppm) – NVIDIA Tesla P40 또는 P100 GPU를 사용하여 llama.cpp 인스턴스를 실행하고, 비활성 상태 시 전력 소비량을 줄입니다.

**인프라:**

- [Paddler](https://github.com/distantmagic/paddler) - llama.cpp에 맞춤형 로드 밸런서입니다.

## 데모

<details>
<summary>M2 Ultra에서 LLaMA v2 13B를 사용한 일반적인 실행</summary>

```
$ make -j && ./llama-cli -m models/llama-13b-v2/ggml-model-q4_0.gguf -p "Building a website can be done in 10 simple steps:\nStep 1:" -n 400 -e
I llama.cpp build info:
I UNAME_S:  Darwin
I UNAME_P:  arm
I UNAME_M:  arm64
I CFLAGS:   -I.            -O3 -std=c11   -fPIC -DNDEBUG -Wall -Wextra -Wpedantic -Wcast-qual -Wdouble-promotion -Wshadow -Wstrict-prototypes -Wpointer-arith -Wmissing-prototypes -pthread -DGGML_USE_K_QUANTS -DGGML_USE_ACCELERATE
I CXXFLAGS: -I. -I./common -O3 -std=c++11 -fPIC -DNDEBUG -Wall -Wextra -Wpedantic -Wcast-qual -Wno-unused-function -Wno-multichar -pthread -DGGML_USE_K_QUANTS
I LDFLAGS:   -framework Accelerate
I CC:       Apple clang version 14.0.3 (clang-1403.0.22.14.1)
I CXX:      Apple clang version 14.0.3 (clang-1403.0.22.14.1)

make: Nothing to be done for `default'.
main: build = 1041 (cf658ad)
main: seed  = 1692823051
llama_model_loader: loaded meta data with 16 key-value pairs and 363 tensors from models/llama-13b-v2/ggml-model-q4_0.gguf (version GGUF V1 (latest))
llama_model_loader: - type  f32:   81 tensors
llama_model_loader: - type q4_0:  281 tensors
llama_model_loader: - type q6_K:    1 tensors
llm_load_print_meta: format         = GGUF V1 (latest)
llm_load_print_meta: arch           = llama
llm_load_print_meta: vocab type     = SPM
llm_load_print_meta: n_vocab        = 32000
llm_load_print_meta: n_merges       = 0
llm_load_print_meta: n_ctx_train    = 4096
llm_load_print_meta: n_ctx          = 512
llm_load_print_meta: n_embd         = 5120
llm_load_print_meta: n_head         = 40
llm_load_print_meta: n_head_kv      = 40
llm_load_print_meta: n_layer        = 40
llm_load_print_meta: n_rot          = 128
llm_load_print_meta: n_gqa          = 1
llm_load_print_meta: f_norm_eps     = 1.0e-05
llm_load_print_meta: f_norm_rms_eps = 1.0e-05
llm_load_print_meta: n_ff           = 13824
llm_load_print_meta: freq_base      = 10000.0
llm_load_print_meta: freq_scale     = 1
llm_load_print_meta: model type     = 13B
llm_load_print_meta: model ftype    = mostly Q4_0
llm_load_print_meta: model size     = 13.02 B
llm_load_print_meta: general.name   = LLaMA v2
llm_load_print_meta: BOS token = 1 '<s>'
llm_load_print_meta: EOS token = 2 '</s>'
llm_load_print_meta: UNK token = 0 '<unk>'
llm_load_print_meta: LF token  = 13 '<0x0A>'
llm_load_tensors: ggml ctx size =    0.11 MB
llm_load_tensors: mem required  = 7024.01 MB (+  400.00 MB per state)
...................................................................................................
llama_new_context_with_model: kv self size  =  400.00 MB
llama_new_context_with_model: compute buffer total size =   75.41 MB

system_info: n_threads = 16 / 24 | AVX = 0 | AVX2 = 0 | AVX512 = 0 | AVX512_VBMI = 0 | AVX512_VNNI = 0 | FMA = 0 | NEON = 1 | ARM_FMA = 1 | F16C = 0 | FP16_VA = 1 | WASM_SIMD = 0 | BLAS = 1 | SSE3 = 0 | VSX = 0 |
sampling: repeat_last_n = 64, repeat_penalty = 1.100000, presence_penalty = 0.000000, frequency_penalty = 0.000000, top_k = 40, tfs_z = 1.000000, top_p = 0.950000, typical_p = 1.000000, temp = 0.800000, mirostat = 0, mirostat_lr = 0.100000, mirostat_ent = 5.000000
generate: n_ctx = 512, n_batch = 512, n_predict = 400, n_keep = 0


 Building a website can be done in 10 simple steps:
Step 1: Find the right website platform.
Step 2: Choose your domain name and hosting plan.
Step 3: Design your website layout.
Step 4: Write your website content and add images.
Step 5: Install security features to protect your site from hackers or spammers
Step 6: Test your website on multiple browsers, mobile devices, operating systems etc…
Step 7: Test it again with people who are not related to you personally – friends or family members will work just fine!
Step 8: Start marketing and promoting the website via social media channels or paid ads
Step 9: Analyze how many visitors have come to your site so far, what type of people visit more often than others (e.g., men vs women) etc…
Step 10: Continue to improve upon all aspects mentioned above by following trends in web design and staying up-to-date on new technologies that can enhance user experience even further!
How does a Website Work?
A website works by having pages, which are made of HTML code. This code tells your computer how to display the content on each page you visit – whether it’s an image or text file (like PDFs). In order for someone else’s browser not only be able but also want those same results when accessing any given URL; some additional steps need taken by way of programming scripts that will add functionality such as making links clickable!
The most common type is called static HTML pages because they remain unchanged over time unless modified manually (either through editing files directly or using an interface such as WordPress). They are usually served up via HTTP protocols – this means anyone can access them without having any special privileges like being part of a group who is allowed into restricted areas online; however, there may still exist some limitations depending upon where one lives geographically speaking.
How to
llama_print_timings:        load time =   576.45 ms
llama_print_timings:      sample time =   283.10 ms /   400 runs   (    0.71 ms per token,  1412.91 tokens per second)
llama_print_timings: prompt eval time =   599.83 ms /    19 tokens (   31.57 ms per token,    31.68 tokens per second)
llama_print_timings:        eval time = 24513.59 ms /   399 runs   (   61.44 ms per token,    16.28 tokens per second)
llama_print_timings:       total time = 25431.49 ms
```

</details>

<details>
<summary>M1 Pro MacBook에서 LLaMA-7B와 whisper.cpp를 동시에 실행하는 데모</summary>

다음은 M1 Pro MacBook에서 LLaMA-7B와 [whisper.cpp](https://github.com/ggerganov/whisper.cpp)를 동시에 실행하는 데모입니다:

https://user-images.githubusercontent.com/1991296/224442907-7693d4be-acaa-4e01-8b4f-add84093ffff.mp4

</details>

## 사용법

다음은 대부분의 지원 모델에 대한 종합적인 이진 파일 빌드 및 모델 변환 단계입니다.

### 기본 사용법

먼저, 바이너리를 가져야 합니다. 따라할 수 있는 다른 방법이 있습니다.
- 방법 1: 이 저장소를 복제하고 로컬로 빌드합니다. [빌드 방법](./docs/build.md)을 참조하세요.
- 방법 2: MacOS 또는 Linux를 사용하는 경우, [brew, flox 또는 nix](./docs/install.md)를 통해 llama.cpp를 설치할 수 있습니다.
- 방법 3: Docker 이미지를 사용합니다. [Docker 설명서](./docs/docker.md)를 참조하세요.
- 방법 4: [릴리즈](https://github.com/ggerganov/llama.cpp/releases)에서 미리 빌드된 바이너리를 다운로드합니다.

다음 명령어를 사용하여 기본 완성을 실행할 수 있습니다:

```bash
llama-cli -m your_model.gguf -p "I believe the meaning of life is" -n 128

# Output:
# I believe the meaning of life is to find your own truth and to live in accordance with it. For me, this means being true to myself and following my passions, even if they don't align with societal expectations. I think that's what I love about yoga – it's not just a physical practice, but a spiritual one too. It's about connecting with yourself, listening to your inner voice, and honoring your own unique journey.
```

자세한 매개변수 목록은 [이 페이지](./examples/main/README.md)를 참조하세요.

### 대화 모드

ChatGPT와 유사한 경험을 원하시면 `-cnv`를 매개변수로 전달하여 대화 모드로 실행할 수 있습니다:

```bash
llama-cli -m your_model.gguf -p "You are a helpful assistant" -cnv

# Output:
# > hi, who are you?
# Hi there! I'm your helpful assistant! I'm an AI-powered chatbot designed to assist and provide information to users like you. I'm here to help answer your questions, provide guidance, and offer support on a wide range of topics. I'm a friendly and knowledgeable AI, and I'm always happy to help with anything you need. What's on your mind, and how can I assist you today?
#
# > what is 1+1?
# Easy peasy! The answer to 1+1 is... 2!
```

기본적으로 챗 템플릿은 입력 모델에서 가져옵니다. 다른 챗 템플릿을 사용하려면 `--chat-template NAME`을 매개변수로 전달하십시오. [지원되는 템플릿 목록](https://github.com/ggerganov/llama.cpp/wiki/Templates-supported-by-llama_chat_apply_template)을 참조하십시오.

```bash
./llama-cli -m your_model.gguf -p "You are a helpful assistant" -cnv --chat-template chatml
```

또한, in-prefix, in-suffix 및 reverse-prompt 매개변수를 통해 자신의 템플릿을 사용할 수 있습니다:

```bash
./llama-cli -m your_model.gguf -p "You are a helpful assistant" -cnv --in-prefix 'User: ' --reverse-prompt 'User:'
```

### 웹 서버

[llama.cpp 웹 서버](./examples/server/README.md)는 가벼운 [OpenAI API](https://github.com/openai/openai-openapi) 호환 HTTP 서버로, 로컬 모델을 제공하고 기존 클라이언트에 쉽게 연결할 수 있습니다.

사용 예시:

```bash
./llama-server -m your_model.gguf --port 8080

# Basic web UI can be accessed via browser: http://localhost:8080
# Chat completion endpoint: http://localhost:8080/v1/chat/completions
```

### 상호 작용 모드

> [!참고]
> 기본 사용법을 선호하는 경우, 대화 모드를 상호 작용 모드 대신 사용하는 것을 고려해 보세요

이 모드에서는 Ctrl+C를 눌러 생성을 항상 중단하고 하나 이상의 줄의 텍스트를 입력하여 현재 맥락에 추가할 수 있습니다. 또한 `-r "반전 프롬프트 문자열"` 매개변수로 *반전 프롬프트*를 지정할 수 있습니다. 이렇게 하면 생성에서 정확한 토큰 문자열이 반전 프롬프트 문자열과 일치하는 경우 사용자 입력이 요청됩니다. 일반적인 사용 방법은 LLaMA가 Alice와 Bob과 같은 여러 사용자 간의 대화를 에뮬레이트하도록 프롬프트를 사용하고 `-r "Alice:"`를 전달하는 것입니다.

다음은 명령어로 호출된 몇 가지 샷 상호 작용의 예입니다.

```bash
# default arguments using a 7B model
./examples/chat.sh

# advanced chat with a 13B model
./examples/chat-13B.sh

# custom arguments using a 13B model
./llama-cli -m ./models/13B/ggml-model-q4_0.gguf -n 256 --repeat_penalty 1.0 --color -i -r "User:" -f prompts/chat-with-bob.txt
```

사용자 입력과 생성된 텍스트를 구분하기 위해 `--color` 를 사용하는 것을 주의하십시오. 다른 매개변수는 `llama-cli` 예제 프로그램의 [README](examples/main/README.md)에서 자세히 설명되어 있습니다.

![image](https://user-images.githubusercontent.com/1991296/224575029-2af3c7dc-5a65-4f64-a6bb-517a532aea38.png)

### 지속적인 상호 작용

프롬프트, 사용자 입력 및 모델 생성은 `./llama-cli` 호출 간에 `--prompt-cache` 및 `--prompt-cache-all` 을 사용하여 저장 및 재개할 수 있습니다. `./examples/chat-persistent.sh` 스크립트는 장시간 지속되고 재개 가능한 채팅 세션을 지원하여 이를 보여줍니다. 이 예제를 사용하려면 초기 채팅 프롬프트를 캐시할 파일과 채팅 세션을 저장할 디렉토리를 제공해야 하며, `chat-13B.sh`와 동일한 변수를 선택적으로 제공할 수 있습니다. 동일한 프롬프트 캐시는 새 채팅 세션에 다시 사용할 수 있습니다. 프롬프트 캐시와 채팅 디렉토리는 초기 프롬프트(`PROMPT_TEMPLATE`) 및 모델 파일과 연결되어 있습니다.

```bash
# Start a new chat
PROMPT_CACHE_FILE=chat.prompt.bin CHAT_SAVE_DIR=./chat/default ./examples/chat-persistent.sh

# Resume that chat
PROMPT_CACHE_FILE=chat.prompt.bin CHAT_SAVE_DIR=./chat/default ./examples/chat-persistent.sh

# Start a different chat with the same prompt/model
PROMPT_CACHE_FILE=chat.prompt.bin CHAT_SAVE_DIR=./chat/another ./examples/chat-persistent.sh

# Different prompt cache for different prompt/model
PROMPT_TEMPLATE=./prompts/chat-with-bob.txt PROMPT_CACHE_FILE=bob.prompt.bin \
    CHAT_SAVE_DIR=./chat/bob ./examples/chat-persistent.sh
```

### 문법을 사용한 제한된 출력

`llama.cpp`는 모델 출력을 제한하기 위한 문법을 지원합니다. 예를 들어, 모델이 JSON만 출력하도록 강제할 수 있습니다.

```bash
./llama-cli -m ./models/13B/ggml-model-q4_0.gguf -n 256 --grammar-file grammars/json.gbnf -p 'Request: schedule a call at 8pm; Command:'
```

`grammars/` 폴더에는 몇 가지 샘플 문법이 포함되어 있습니다. 자신의 문법을 작성하려면 [GBNF 가이드](./grammars/README.md)를 참조하십시오.

더 복잡한 JSON 문법을 작성하려면 https://grammar.intrinsiclabs.ai/와 같은 브라우저 앱을 사용할 수도 있습니다. 이 앱은 TypeScript 인터페이스를 작성하여 GBNF 문법으로 컴파일하는 데 사용되며, 로컬로 사용할 수 있도록 저장할 수 있습니다. 앱은 커뮤니티 회원들이 개발하고 유지 관리하고 있으므로 문제나 기능 제안은 [앱 레포지토리](http://github.com/intrinsiclabsai/gbnfgen)에 제출하십시오. 이 레포지토리에는 제출하지 마십시오.

## 빌드

[llama.cpp 로컬 빌드 가이드](./docs/build.md)를 참조하세요.

## 지원되는 백엔드

| Backend | Target devices |
| --- | --- |
| [Metal](./docs/build.md#metal-build) | Apple Silicon |
| [BLAS](./docs/build.md#blas-build) | All |
| [BLIS](./docs/backend/BLIS.md) | All |
| [SYCL](./docs/backend/SYCL.md) | Intel and Nvidia GPU |
| [CUDA](./docs/build.md#cuda) | Nvidia GPU |
| [hipBLAS](./docs/build.md#hipblas) | AMD GPU |
| [Vulkan](./docs/build.md#vulkan) | GPU |

## 도구

### 준비 및 양자화

> [!참고]
> [GGUF-my-repo](https://huggingface.co/spaces/ggml-org/gguf-my-repo)는 설정 없이 모델 가중치를 양자화하는 데 사용할 수 있는 Hugging Face 공간입니다. 6시간마다 `llama.cpp` 메인에서 동기화됩니다.

공식 LLaMA 2 가중치를 얻으려면 <a href="#facebook-llama-2-모델의-획득 및 사용">Facebook LLaMA 2 모델의 획득 및 사용</a> 섹션을 참조하십시오. 또한 Hugging Face에서 다양한 사전 양자화된 `gguf` 모델이 있습니다.

참고: `convert.py`는 `examples/convert_legacy_llama.py`로 이동되었으며, `Llama/Llama2/Mistral` 모델 및 그 유도체 외에는 다른 용도로 사용해서는 안 됩니다. LLaMA 3를 지원하지 않습니다. Hugging Face에서 다운로드한 LLaMA 3을 사용하려면 `convert_hf_to_gguf.py`를 사용하십시오.

모델을 양자화하는 방법에 대해 자세히 알아보려면 [이 문서](./examples/quantize/README.md)를 참조하십시오.

### Perplexity (모델 품질 측정)

`perplexity` 예제를 사용하여 주어진 프롬프트에 대한 perplexity를 측정할 수 있습니다(낮은 perplexity가 좋습니다).
더 자세한 내용은 [https://huggingface.co/docs/transformers/perplexity](https://huggingface.co/docs/transformers/perplexity)를 참조하십시오.

llama.cpp를 사용하여 perplexity를 측정하는 방법에 대해 더 자세히 알고 싶으시면 [이 문서](./examples/perplexity/README.md)를 읽으십시오.

## 기여

- 기여자는 PR을 열 수 있습니다.
- 협력자는 `llama.cpp` 레포지토리의 브랜치에 푸시하고 `master` 브랜치로 PR을 병합할 수 있습니다.
- 협력자는 기여에 따라 초대됩니다.
- 문제 및 PR 관리에 대한 도움은 매우 감사합니다!
- 처음 기여하기에 적합한 작업은 [good first issues](https://github.com/ggerganov/llama.cpp/issues?q=is%3Aissue+is%3Aopen+label%3A%22good+first+issue%22)를 참조하세요.
- 자세한 내용은 [CONTRIBUTING.md](CONTRIBUTING.md)를 참조하세요.
- 이것을 반드시 읽으세요: [Inference at the edge](https://github.com/ggerganov/llama.cpp/discussions/205)
- 관심 있는 분들을 위한 간략한 배경 정보: [Changelog podcast](https://changelog.com/podcast/532)

## 기타 문서

- [main (cli)](./examples/main/README.md)
- [서버](./examples/server/README.md)
- [제패디](./examples/jeopardy/README.md)
- [GBNF 문법](./grammars/README.md)

**개발 문서**

- [빌드 방법](./docs/build.md)
- [Docker 사용](./docs/docker.md)
- [Android에서 빌드](./docs/android.md)
- [성능 문제 해결](./docs/development/token_generation_performance_tips.md)
- [GGML 팁 및 트릭](https://github.com/ggerganov/llama.cpp/wiki/GGML-Tips-&-Tricks)

**기본 논문 및 모델 배경 정보**

모델 생성 품질에 문제가 있는 경우, LLaMA 모델의 한계를 이해하기 위해 다음 링크 및 논문을 훑어보십시오. 특히 적절한 모델 크기를 선택하고 LLaMA 모델과 ChatGPT 사이의 중요하고 미묘한 차이점을 이해할 때 중요합니다.
- LLaMA:
    - [Introducing LLaMA: A foundational, 65-billion-parameter large language model](https://ai.facebook.com/blog/large-language-model-llama-meta-ai/)
    - [LLaMA: Open and Efficient Foundation Language Models](https://arxiv.org/abs/2302.13971)
- GPT-3
    - [Language Models are Few-Shot Learners](https://arxiv.org/abs/2005.14165)
- GPT-3.5 / InstructGPT / ChatGPT:
    - [Aligning language models to follow instructions](https://openai.com/research/instruction-following)
    - [Training language models to follow instructions with human feedback](https://arxiv.org/abs/2203.02155)
