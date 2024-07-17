# MobileVLM

현재 이 구현은 [MobileVLM-1.7B](https://huggingface.co/mtgv/MobileVLM-1.7B) / [MobileVLM_V2-1.7B](https://huggingface.co/mtgv/MobileVLM_V2-1.7B) 변이를 지원합니다.

더 자세한 내용은 [Meituan-AutoML/MobileVLM](https://github.com/Meituan-AutoML/MobileVLM)을 참조하십시오.

이 구현은 llava 기반이며, llava 및 mobileVLM과 호환됩니다. 사용법은 llava와 거의 동일합니다.

참고: **MobileVLM** 및 **MobileVLM_V2** 모델의 모델 추론 전체 과정은 동일하지만, 모델 변환 과정은 약간 다릅니다. 따라서 **MobileVLM-1.7B**를 예시로 사용하여 다른 변환 단계를 보여드리겠습니다.

## 사용법
cmake로 빌드하거나 `make llama-llava-cli`를 실행하여 빌드합니다.

빌드 후, `./llama-llava-cli`를 실행하여 사용법을 확인합니다. 예를 들어:

```sh
./llama-llava-cli -m MobileVLM-1.7B/ggml-model-q4_k.gguf \
    --mmproj MobileVLM-1.7B/mmproj-model-f16.gguf \
    --image path/to/an/image.jpg \
    -p "A chat between a curious user and an artificial intelligence assistant. The assistant gives helpful, detailed, and polite answers to the user's questions. USER: <image>\nWho is the author of this book? Answer the question using a single word or phrase. ASSISTANT:"
```

## 모델 변환

1. `mobileVLM-1.7B` 및 `clip-vit-large-patch14-336`을 로컬로 복제합니다.

```sh
git clone https://huggingface.co/mtgv/MobileVLM-1.7B

git clone https://huggingface.co/openai/clip-vit-large-patch14-336
```

2. `llava_surgery.py`를 사용하여 LLaVA 모델을 LLaMA와 다중 모델 프로젝터 구성 요소로 분할합니다:

```sh
python ./examples/llava/llava_surgery.py -m path/to/MobileVLM-1.7B
```

3. `convert_image_encoder_to_gguf.py`를 사용하여 `--projector-type ldp` (**V2**의 경우 `--projector-type ldpv2`)로 LLaVA 이미지 인코더를 GGUF로 변환합니다.

```sh
python ./examples/llava/convert_image_encoder_to_gguf \
    -m path/to/clip-vit-large-patch14-336 \
    --llava-projector path/to/MobileVLM-1.7B/llava.projector \
    --output-dir path/to/MobileVLM-1.7B \
    --projector-type ldp
```

```sh
python ./examples/llava/convert_image_encoder_to_gguf \
    -m path/to/clip-vit-large-patch14-336 \
    --llava-projector path/to/MobileVLM-1.7B_V2/llava.projector \
    --output-dir path/to/MobileVLM-1.7B_V2 \
    --projector-type ldpv2
```

4. `examples/convert_legacy_llama.py`를 사용하여 LLaVA의 LLaMA 부분을 GGUF로 변환합니다.

```sh
python ./examples/convert_legacy_llama.py path/to/MobileVLM-1.7B
```

5. `quantize`를 사용하여 LLaMA 부분의 DataType를 `fp16`에서 `q4_k`로 변환합니다.
```sh
./llama-quantize path/to/MobileVLM-1.7B/ggml-model-f16.gguf path/to/MobileVLM-1.7B/ggml-model-q4_k.gguf q4_k_s
```

이제 LLaMA 부분과 이미지 인코더가 모두 `MobileVLM-1.7B` 디렉토리에 있습니다.

## 안드로이드 컴파일 및 실행
### 컴파일
`examples/llava/android/build_64.sh`를 참조하세요
```sh
mkdir examples/llava/android/build_64
cd examples/llava/android/build_64
../build_64.sh
```
### Android에서 실행
`android/adb_run.sh`를 참조하여 리소스의 `name`과 `path`를 수정하세요.

## Snapdragon 888 칩셋에서 안드로이드에서의 일부 결과
### case 1
**입력**
```sh
/data/local/tmp/llama-llava-cli \
    -m /data/local/tmp/ggml-model-q4_k.gguf \
    --mmproj /data/local/tmp/mmproj-model-f16.gguf \
    -t 4 \
    --image /data/local/tmp/demo.jpg \
    -p "A chat between a curious user and an artificial intelligence assistant. The assistant gives helpful, detailed, and polite answers to the user's questions. USER: <image>\nWho is the author of this book? \nAnswer the question using a single word or phrase. ASSISTANT:"
```
**출력**
```sh
encode_image_with_clip: image encoded in 21148.71 ms by CLIP (  146.87 ms per image patch)
 Susan Wise Bauer
llama_print_timings:        load time =   23574.72 ms
llama_print_timings:      sample time =       1.24 ms /     6 runs   (    0.21 ms per token,  4850.44 tokens per second)
llama_print_timings: prompt eval time =   12460.15 ms /   246 tokens (   50.65 ms per token,    19.74 tokens per second)
llama_print_timings:        eval time =     424.86 ms /     6 runs   (   70.81 ms per token,    14.12 tokens per second)
llama_print_timings:       total time =   34731.93 ms
```
### 사례 2
**입력**
```sh
/data/local/tmp/llama-llava-cli \
    -m /data/local/tmp/ggml-model-q4_k.gguf \
    --mmproj /data/local/tmp/mmproj-model-f16.gguf \
    -t 4 \
    --image /data/local/tmp/cat.jpeg \
    -p "A chat between a curious user and an artificial intelligence assistant. The assistant gives helpful, detailed, and polite answers to the user's questions. USER: <image>\nWhat is in the image? ASSISTANT:"
```
**출력**
```sh
encode_image_with_clip: image encoded in 21149.51 ms by CLIP (  146.87 ms per image patch)
 The image depicts a cat sitting in the grass near some tall green plants.
llama_print_timings:        load time =   23257.32 ms
llama_print_timings:      sample time =       5.25 ms /    18 runs   (    0.29 ms per token,  3430.53 tokens per second)
llama_print_timings: prompt eval time =   11900.73 ms /   232 tokens (   51.30 ms per token,    19.49 tokens per second)
llama_print_timings:        eval time =    1279.03 ms /    18 runs   (   71.06 ms per token,    14.07 tokens per second)
llama_print_timings:       total time =   34570.79 ms
```


## Snapdragon 778G 칩으로 실행한 안드로이드 결과물
### MobileVLM-1.7B 사례
#### llava-cli release-b2005
**입력**
```sh
/data/local/tmp/llama-llava-cli \
    -m /data/local/tmp/ggml-model-q4_k.gguf \
    --mmproj /data/local/tmp/mmproj-model-f16.gguf \
    -t 4 \
    --image /data/local/tmp/many_llamas.jpeg \
    -p "A chat between a curious user and an artificial intelligence assistant. The assistant gives helpful, detailed, and polite answers to the user's questions. USER: <image>\nWhat's that? ASSISTANT:"
```
**출력**
```sh
encode_image_with_clip: image encoded in 18728.52 ms by CLIP (  130.06 ms per image patch)
system_prompt: A chat between a curious user and an artificial intelligence assistant. The assistant gives helpful, detailed, and polite answers to the user's questions. USER:
user_prompt: \nWhat's that? ASSISTANT:

 A group of llamas are standing in a green pasture.

llama_print_timings:        load time =   20357.33 ms
llama_print_timings:      sample time =       2.96 ms /    14 runs   (    0.21 ms per token,  4734.53 tokens per second)
llama_print_timings: prompt eval time =    8119.49 ms /   191 tokens (   42.51 ms per token,    23.52 tokens per second)
llama_print_timings:        eval time =    1005.75 ms /    14 runs   (   71.84 ms per token,    13.92 tokens per second)
llama_print_timings:       total time =   28038.34 ms /   205 tokens
```
#### llava-cli 최신 버전
**입력**

위와 동일합니다.

**출력**(아마도 훨씬 느리겠습니다)
```sh
encode_image_with_clip: image embedding created: 144 tokens

encode_image_with_clip: image encoded in 288268.88 ms by CLIP ( 2001.87 ms per image patch)
system_prompt: A chat between a curious user and an artificial intelligence assistant. The assistant gives helpful, detailed, and polite answers to the user's questions. USER:
user_prompt: \nWhat's that? ASSISTANT:

 It is a group of sheep standing together in a grass field.

llama_print_timings:        load time =  818120.91 ms
llama_print_timings:      sample time =       3.44 ms /    14 runs   (    0.25 ms per token,  4067.40 tokens per second)
llama_print_timings: prompt eval time =  529274.69 ms /   191 tokens ( 2771.07 ms per token,     0.36 tokens per second)
llama_print_timings:        eval time =   43894.02 ms /    13 runs   ( 3376.46 ms per token,     0.30 tokens per second)
llama_print_timings:       total time =  865441.76 ms /   204 tokens
```
### MobileVLM_V2-1.7B 사례
#### llava-cli release-2005b
**입력**

위와 동일합니다.

**출력**
```sh
encode_image_with_clip: image encoded in 20609.61 ms by CLIP (  143.12 ms per image patch)
system_prompt: A chat between a curious user and an artificial intelligence assistant. The assistant gives helpful, detailed, and polite answers to the user's questions. USER:
user_prompt: \nWhat's that? ASSISTANT:

 This image captures a lively scene of 20 llamas in motion on an expansive, grassy field. The llama is scattered across the landscape with some standing and others sitting down as if taking rest or observing their surroundings from different vantage points within this verdant setting.

The background offers glimpses into a picturesque town nestled amidst hills under an overcast sky, adding depth to the scene while also emphasizing that distance between these llama and human-made structures like houses or roads in which they roam freely without any barriers around them. The image is framed by text at both right angles on white backgrounds against a contrasting blue backdrop with green foliage, further drawing attention to the llamas amidst their natural habitat while also inviting viewers into this picturesque landscape within town limits of Alta Llama

llama_print_timings:        load time =   22406.77 ms
llama_print_timings:      sample time =      49.26 ms /   186 runs   (    0.26 ms per token,  3776.27 tokens per second)
llama_print_timings: prompt eval time =    9044.54 ms /   191 tokens (   47.35 ms per token,    21.12 tokens per second)
llama_print_timings:        eval time =   14497.49 ms /   186 runs   (   77.94 ms per token,    12.83 tokens per second)
llama_print_timings:       total time =   44411.01 ms /   377 tokens
```

## Orin 컴파일 및 실행
### 컴파일
```sh
make GGML_CUDA=1 CUDA_DOCKER_ARCH=sm_87 GGML_CUDA_F16=1 -j 32
```
### Orin에서 실행하기
### case 1
**입력**
```sh
./llama-llava-cli \
    -m /data/local/tmp/ggml-model-q4_k.gguf \
    --mmproj /data/local/tmp/mmproj-model-f16.gguf \
    --image /data/local/tmp/demo.jpeg \
    -p "A chat between a curious user and an artificial intelligence assistant. The assistant gives helpful, detailed, and polite answers to the user's questions. USER: <image>\nWho is the author of this book? \nAnswer the question using a single word or phrase. ASSISTANT:" \
    --n-gpu-layers 999
```
**출력**
```sh

encode_image_with_clip: image encoded in   296.62 ms by CLIP (    2.06 ms per image patch)

 Susan Wise Bauer

llama_print_timings:        load time =    1067.64 ms
llama_print_timings:      sample time =       1.53 ms /     6 runs   (    0.25 ms per token,  3934.43 tokens per second)
llama_print_timings: prompt eval time =     306.84 ms /   246 tokens (    1.25 ms per token,   801.72 tokens per second)
llama_print_timings:        eval time =      91.50 ms /     6 runs   (   15.25 ms per token,    65.58 tokens per second)
llama_print_timings:       total time =    1352.63 ms /   252 tokens
```

### 사례 2
**입력**
```sh
./llama-llava-cli \
    -m /data/local/tmp/ggml-model-q4_k.gguf \
    --mmproj /data/local/tmp/mmproj-model-f16.gguf \
    -p "A chat between a curious user and an artificial intelligence assistant. The assistant gives helpful, detailed, and polite answers to the user's questions. USER: <image>\nWhat is in the image? ASSISTANT:" \
    --n-gpu-layers 999

```
**출력**
```sh
encode_image_with_clip: image encoded in   302.15 ms by CLIP (    2.10 ms per image patch)

 The image features a cat lying in the grass.

llama_print_timings:        load time =    1057.07 ms
llama_print_timings:      sample time =       3.27 ms /    11 runs   (    0.30 ms per token,  3360.83 tokens per second)
llama_print_timings: prompt eval time =     213.60 ms /   232 tokens (    0.92 ms per token,  1086.14 tokens per second)
llama_print_timings:        eval time =     166.65 ms /    11 runs   (   15.15 ms per token,    66.01 tokens per second)
llama_print_timings:       total time =    1365.47 ms /   243 tokens
```

## Intel(R) Core(TM) i7-10750H에서 실행하기
### 운영 체제
Ubuntu22.04
### 컴파일
```sh
make -j32
```
### MobileVLM-1.7B 사례
**입력**
```sh
-m /path/to/ggml-model-q4_k.gguf \
    --mmproj /path/to/mmproj-model-f16.gguf \
    --image /path/to/many_llamas.jpeg
    -p "A chat between a curious user and an artificial intelligence assistant. The assistant gives helpful, detailed, and polite answers to the user's questions. USER: <image>\nWhat's that? ASSISTANT:" \
```
**출력**
```sh
encode_image_with_clip: image embedding created: 144 tokens

encode_image_with_clip: image encoded in  2730.94 ms by CLIP (   18.96 ms per image patch)
system_prompt: A chat between a curious user and an artificial intelligence assistant. The assistant gives helpful, detailed, and polite answers to the user's questions. USER:
user_prompt: \nWhat's that?ASSISTANT:

 A group of llamas are walking together in a field.

llama_print_timings:        load time =    5506.60 ms
llama_print_timings:      sample time =       0.44 ms /    13 runs   (    0.03 ms per token, 29545.45 tokens per second)
llama_print_timings: prompt eval time =    2031.58 ms /   190 tokens (   10.69 ms per token,    93.52 tokens per second)
llama_print_timings:        eval time =     438.92 ms /    12 runs   (   36.58 ms per token,    27.34 tokens per second)
llama_print_timings:       total time =    5990.25 ms /   202 tokens
```

### MobileVLM_V2-1.7B 사례
**입력**

위와 동일합니다.

**출력**
```sh
encode_image_with_clip: image embedding created: 144 tokens

encode_image_with_clip: image encoded in  3223.89 ms by CLIP (   22.39 ms per image patch)
system_prompt: A chat between a curious user and an artificial intelligence assistant. The assistant gives helpful, detailed, and polite answers to the user's questions. USER:
user_prompt: \nWhat's that?ASSISTANT:

 The image captures a tranquil scene in a park, where a group of approximately 20 llamas are gathered. The llamas, a mix of white and black, are standing in a line, their black and white patterns contrasting with the lush green grass of the park. The lamas are arranged in a line, suggesting a social order.

The park itself is lush and green, with trees dotting the landscape in the background. A sign reading "Llamas Tico  Ana" is also visible in the image, possibly indicating the location or the breed of the llamas. The image seems to be taken from a distance, providing a wide view of the scene and the surrounding environment.

The llamas' positions relative to each other, the sign, and the trees create a harmonious composition. The image does not contain any discernible text. The overall scene is one of peace and natural beauty, with the llamas in their natural habitat, surrounded by the vibrant colors and lush greenery of the park.

llama_print_timings:        load time =    6642.61 ms
llama_print_timings:      sample time =       8.15 ms /   223 runs   (    0.04 ms per token, 27358.61 tokens per second)
llama_print_timings: prompt eval time =    2475.07 ms /   190 tokens (   13.03 ms per token,    76.77 tokens per second)
llama_print_timings:        eval time =    8760.60 ms /   222 runs   (   39.46 ms per token,    25.34 tokens per second)
llama_print_timings:       total time =   15513.95 ms /   412 tokens
```

## Intel(R) Core(TM) Ultra7 115H에서 실행하기
### 운영 체제
Windows11
### 컴파일
```sh
make -j32
```
### MobileVLM-1.7B 사례
**입력**
```sh
-m /path/to/ggml-model-q4_k.gguf \
    --mmproj /path/to/tmp/mmproj-model-f16.gguf \
    -p "A chat between a curious user and an artificial intelligence assistant. The assistant gives helpful, detailed, and polite answers to the user's questions. USER: <image>\nWhat's that? ASSISTANT:" \
```
**출력**
```sh
encode_image_with_clip: image encoded in  4902.81 ms by CLIP (   34.05 ms per image patch)
system_prompt: A chat between a curious user and an artificial intelligence assistant. The assistant gives helpful, detailed, and polite answers to the user's questions. USER:
user_prompt: \nWhat's that? ASSISTANT:

 The image features a group of brown and white llamas standing in a grassy field.

llama_print_timings:        load time =    7441.06 ms
llama_print_timings:      sample time =       0.72 ms /    19 runs   (    0.04 ms per token, 26279.39 tokens per second)
llama_print_timings: prompt eval time =    2090.71 ms /   191 tokens (   10.95 ms per token,    91.36 tokens per second)
llama_print_timings:        eval time =     512.35 ms /    18 runs   (   28.46 ms per token,    35.13 tokens per second)
llama_print_timings:       total time =    7987.23 ms /   209 tokens
```

### MobileVLM_V2-1.7B 사례
**입력**

위와 동일합니다.

**출력**
```sh
encode_image_with_clip: image encoded in  4682.44 ms by CLIP (   32.52 ms per image patch)
system_prompt: A chat between a curious user and an artificial intelligence assistant. The assistant gives helpful, detailed, and polite answers to the user's questions. USER:
user_prompt: \nWhat's that? ASSISTANT:

 This image captures a lively scene of a group of 14 llamas in a grassy field. The llamas, with their distinctive black and white coats, are standing and walking in a line, seemingly engaged in a social activity. One
 of them, possibly the first in the line, has its back turned, perhaps observing something in the distance.

The llama in the front of the line stands out due to its black and white coloring, which is quite unusual for llama patterns. The llama in the front also seems to be more aware of its surroundings, as it faces the camera, giving a sense of engagement with the viewer.

The image is taken from the side of the llama, providing a clear view of the llama in the front and its companions. The lameness in the llama in
 front is not visible, indicating that it might not be the main focus of the photo.

The background of the image features a grassy field, with a fence and a tree visible in the distance. The tree appears to be bare, suggesting that it might be during a time of year when most trees are dormant or have shed their leaves.


llama_print_timings:        load time =    7015.35 ms
llama_print_timings:      sample time =      10.61 ms /   256 runs   (    0.04 ms per token, 24119.09 tokens per second)
llama_print_timings: prompt eval time =    2052.45 ms /   191 tokens (   10.75 ms per token,    93.06 tokens per second)
llama_print_timings:        eval time =    7259.43 ms /   255 runs   (   28.47 ms per token,    35.13 tokens per second)
llama_print_timings:       total time =   14371.19 ms /   446 tokens
```

## TODO

- [x] 새로운 연산자( `depthwise`, `hardswish`, `hardsigmoid` )에 대한 CPU 이외의 백엔드 지원
- [ ] LDP 프로젝터 성능 최적화

      - 불필요한 메모리 재배열을 피하기 위해 구조 정의 최적화, `ggml_permute_cpy` 사용량 감소;
      - 연산자 구현 (ARM CPU/NVIDIA GPU) 최적화: depthwise conv, hardswish, hardsigmoid 등
- [x] Jetson Orin에서 MobileVLM 실행
- [ ] MobileVLM-3B와 같은 더 많은 모델 변형 지원.


## 기여자
```sh
zhangjidong05, yangyang260, huyiming03, chenxiaotao03, ZiangWu-77
```
