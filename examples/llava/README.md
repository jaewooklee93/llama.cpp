# LLaVA

현재 이 구현은 [llava-v1.5](https://huggingface.co/liuhaotian/llava-v1.5-7b) 변형 및
llava-1.6 [llava-v1.6](https://huggingface.co/collections/liuhaotian/llava-16-65b9e40155f60fd046a5ccf2) 변형을 지원합니다.

사전 변환된 [7b](https://huggingface.co/mys/ggml_llava-v1.5-7b)
및 [13b](https://huggingface.co/mys/ggml_llava-v1.5-13b)
모델이 사용 가능합니다.
llava-1.6의 경우 다양한 준비된 gguf 모델도 사용 가능합니다 [7b-34b](https://huggingface.co/cmp-nct/llava-1.6-gguf)

API가 확인되면 더 많은 모델이 지원/업로드될 예정입니다.

## 사용법
cmake로 빌드하거나 `make llama-llava-cli`를 실행하여 빌드합니다.

빌드 후, `./llama-llava-cli`를 실행하여 사용법을 확인합니다. 예를 들어:

```sh
./llama-llava-cli -m ../llava-v1.5-7b/ggml-model-f16.gguf --mmproj ../llava-v1.5-7b/mmproj-model-f16.gguf --image path/to/an/image.jpg
```

**참고**: 더 나은 품질을 위해 0.1과 같은 낮은 온도를 권장합니다. 명령에 `--temp 0.1`을 추가하여 이를 수행하십시오.
**참고**: GPU 오프로드를 위해서는 일반과 같이 `-ngl` 플래그를 사용해야 합니다.

## LLaVA 1.5

1. LLaVA와 CLIP 모델을 복제합니다 ([가용 옵션](https://github.com/haotian-liu/LLaVA/blob/main/docs/MODEL_ZOO.md)). 예를 들어:

```sh
git clone https://huggingface.co/liuhaotian/llava-v1.5-7b

git clone https://huggingface.co/openai/clip-vit-large-patch14-336
```

2. 필요한 Python 패키지를 설치하세요:

```sh
pip install -r examples/llava/requirements.txt
```

3. `llava_surgery.py`를 사용하여 LLaVA 모델을 LLaMA와 다중 모델 프로젝터 구성 요소로 분리합니다.

```sh
python ./examples/llava/llava_surgery.py -m ../llava-v1.5-7b
```

4. `convert_image_encoder_to_gguf.py`를 사용하여 LLaVA 이미지 인코더를 GGUF로 변환합니다.

```sh
python ./examples/llava/convert_image_encoder_to_gguf.py -m ../clip-vit-large-patch14-336 --llava-projector ../llava-v1.5-7b/llava.projector --output-dir ../llava-v1.5-7b
```

5. `examples/convert_legacy_llama.py`를 사용하여 LLaVA의 LLaMA 부분을 GGUF로 변환합니다.

```sh
python ./examples/convert_legacy_llama.py ../llava-v1.5-7b --skip-unknown
```

이제 LLaMA 부분과 이미지 인코더가 모두 `llava-v1.5-7b` 디렉토리에 있습니다.

## LLaVA 1.6 gguf 변환
1) 먼저 LLaVA 1.6 모델을 복제합니다:
```console
git clone https://huggingface.co/liuhaotian/llava-v1.6-vicuna-7b
```

2) 필요한 Python 패키지를 설치하세요:

```sh
pip install -r examples/llava/requirements.txt
```

3) `llava_surgery_v2.py`를 사용하세요. 이 스크립트는 llava-1.5 변형 pytorch 모델과 safetensor 모델을 모두 지원합니다:
```console
python examples/llava/llava_surgery_v2.py -C -m ../llava-v1.6-vicuna-7b/
```
- 모델 디렉토리에 llava.projector와 llava.clip 파일이 있습니다.

4) llava.clip 파일을 하위 디렉토리(예: vit)로 복사하고 pytorch_model.bin으로 이름을 바꾸고 디렉토리에 맞는 vit 구성 파일을 추가하세요:
```console
mkdir vit
cp ../llava-v1.6-vicuna-7b/llava.clip vit/pytorch_model.bin
cp ../llava-v1.6-vicuna-7b/llava.projector vit/
curl -s -q https://huggingface.co/cmp-nct/llava-1.6-gguf/raw/main/config_vit.json -o vit/config.json
```

5) 시각적 gguf 모델 생성:
```console
python ./examples/llava/convert_image_encoder_to_gguf.py -m vit --llava-projector vit/llava.projector --output-dir vit --clip-model-is-vision
```
- llava-1.5와 유사하지만, 인코더에 CLIP의 순수 비전 모델 부분을 사용하고 있다는 것을 알려줍니다.

6) 모델을 gguf 형식으로 변환합니다:
```console
python ./examples/convert_legacy_llama.py ../llava-v1.6-vicuna-7b/ --skip-unknown
```

7) 마지막으로 1.6 모델 버전을 사용하여 llava cli를 실행할 수 있습니다:
```console
./llama-llava-cli -m ../llava-v1.6-vicuna-7b/ggml-model-f16.gguf --mmproj vit/mmproj-model-f16.gguf --image some-image.jpg -c 4096
```

**참고** llava-1.6는 llava-1.5보다 더 많은 맥락이 필요합니다. 최소 3000개가 필요합니다 (단순히 -c 4096으로 실행하세요)
**참고** llava-1.6는 배치 프롬프트 처리에서 큰 이점을 얻습니다 (기본값이 작동합니다)

## llava-cli 템플릿 및 llava-1.6 프롬프팅

llava-1.5 모델은 모두 동일한 vicuna 프롬프트를 사용합니다. 여기서는 `-p "전체 설명을 제공하세요."`와 같이 이미지 질문을 추가할 수 있습니다.
llava-1.5 모델 중 vicuna가 아닌 (mistral 및 Yi) 모델의 경우 시스템 프롬프트와 사용자 프롬프트를 모두 조정해야 합니다. 이를 위해 llava-cli에는 기본적인 템플릿 시스템이 있습니다.

**Mistral을 사용하는 경우 및 llava-cli binary를 사용하는 경우:**
다음을 추가하세요: `-p "<이미지>\n사용자:\n전체 설명을 제공하세요.\n보조자:\n"`
llava-1.6의 mistral 템플릿은 시스템 출력이 없고 사용자/보조자 역할이 있는 것으로 보입니다.

**34B 모델의 경우 다음이 작동해야 합니다:**
다음을 추가하세요: `-e -p <|im_start|>system\n질문에 답하세요.<|im_end|><|im_start|>user\n<이미지>\n전체 설명을 제공하세요.<|im_end|><|im_start|>assistant\n`


## llava-1.5 또는 llava-1.6 모드인지 확인하는 방법

llava-cli를 실행하면 프롬프트가 처리되기 직전에 시각적 정보가 표시됩니다.

**Llava-1.5:**
`encode_image_with_clip: image embedding created: 576 tokens`

**Llava-1.6 (576 이상):**
`encode_image_with_clip: image embedding created: 2880 tokens`


또는 프롬프트에 사용된 토큰 수에 주의를 기울이면 됩니다. llava-1.6의 경우 1000개 이상의 토큰이 사용된 것을 확인할 수 있습니다.




## TODO

- [x] 이미지 인코딩 부분에 대한 CPU 외 백엔드 지원
- [ ] 다양한 샘플링 방법 지원
- [ ] 더 많은 모델 변형체 지원
