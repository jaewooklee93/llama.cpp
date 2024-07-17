# 토큰 생성 성능 문제 해결

## GPU 사용 확인
CUDA를 사용하여 모델이 GPU에서 실행되도록 컴파일했는지 확인하십시오. [이 가이드](/docs/build.md#cuda)에 따라 올바른 환경 변수를 사용하여 llama를 컴파일했는지 확인하십시오. 이렇게 하면 llama가 `-ngl N` (또는 `--n-gpu-layers N`) 플래그를 받아들일 수 있습니다. llama를 실행할 때 `N`을 매우 큰 값으로 설정할 수 있으며, 이는 GPU에 최대한 많은 레이어를 오프로드하는 것을 의미합니다. GPU에 설정된 레이어 수보다 적더라도 그렇습니다. 예를 들어:
```shell
./llama-cli -m "path/to/model.gguf" -ngl 200000 -p "Please sir, may I have some "
```

llama를 실행할 때, 인퍼런스 작업을 시작하기 전에 cuBLAS가 GPU로 작업을 오프로드하는지 여부를 보여주는 진단 정보를 출력합니다. 다음과 같은 줄을 찾으세요:
```shell
llama_model_load_internal: [cublas] offloading 60 layers to GPU
llama_model_load_internal: [cublas] offloading output layer to GPU
llama_model_load_internal: [cublas] total VRAM used: 17223 MB
... rest of inference
```

이 줄들이 보인다면 GPU가 사용되고 있습니다.

## CPU 과부하 확인
llama는 `-t N` (또는 `--threads N`) 매개변수를 허용합니다. 이 매개변수가 너무 크지 않도록 하는 것이 매우 중요합니다. 토큰 생성이 매우 느리면 이 숫자를 1로 설정해 보세요. 이것이 토큰 생성 속도를 상당히 향상시킨다면, CPU가 과부하되어 있으며, 컴퓨터의 물리적 CPU 코어 수로 이 매개변수를 명시적으로 설정해야 합니다 (GPU를 사용하더라도). 망설이신다면, 1부터 시작하여 성능 병목 현상에 도달할 때까지 두 배로 늘리고, 그 후에는 숫자를 줄여보세요.

# 런타임 플래그가 추론 속도 벤치마크에 미치는 영향의 예시
다음 머신에서 테스트되었습니다.
GPU: A6000 (48GB VRAM)
CPU: 7개의 물리적 코어
RAM: 32GB

모델: `TheBloke_Wizard-Vicuna-30B-Uncensored-GGML/Wizard-Vicuna-30B-Uncensored.q4_0.gguf` (30B 파라미터, 4비트 양자화, GGML)

실행 명령어: `./llama-cli -m "모델 경로.gguf" -p "10가지 최고의 민족 음식에 대한 매우 자세한 설명과 레시피가 뒤따릅니다: " -n 1000 [추가 벤치마크 플래그]`

결과:

| command | tokens/second (higher is better) |
| - | - |
| -ngl 2000000 | N/A (less than 0.1) |
| -t 7 | 1.7 |
| -t 1 -ngl 2000000 | 5.5 |
| -t 7 -ngl 2000000 | 8.7 |
| -t 4 -ngl 2000000 | 9.1 |
