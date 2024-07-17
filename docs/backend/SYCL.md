# SYCL를 사용한 llama.cpp

- [개요](#개요)
- [권장 버전](#권장-버전)
- [뉴스](#뉴스)
- [운영체제](#운영체제)
- [하드웨어](#하드웨어)
- [Docker](#docker)
- [Linux](#linux)
- [Windows](#windows)
- [환경 변수](#환경-변수)
- [알려진 문제](#알려진-문제)
- [Q&A](#qa)
- [TODO](#todo)

## 배경

**SYCL**은 CPU, GPU 및 FPGA와 같은 다양한 하드웨어 가속기에서 코드를 작성하는 개발자 생산성을 향상시키기 위해 설계된 고수준 병렬 프로그래밍 모델입니다. CPU, GPU 및 FPGA와 같은 다양한 하드웨어에서 동일한 코드를 실행할 수 있도록 설계된 단일 소스 언어이며 C++17 기반입니다.

**oneAPI**는 Intel CPU, GPU 및 FPGA를 포함한 여러 아키텍처를 지원하는 오픈 이코시스템 및 표준 기반 명세입니다. oneAPI 이코시스템의 핵심 구성 요소는 다음과 같습니다.

- **DPCPP** *(데이터 병렬 C++)*: oneAPI SYCL 구현의 주요 구성 요소로, icpx/icx 컴파일러를 포함합니다.
- **oneAPI 라이브러리**: 여러 영역을 대상으로 하는 최적화된 라이브러리 세트 *(예: oneMKL - 수학 커널 라이브러리)*.
- **oneAPI LevelZero**: Intel iGPU 및 dGPU에 대한 미세 조정 제어를 위한 고성능 저수준 인터페이스.
- **Nvidia & AMD 플러그인**: Nvidia 및 AMD GPU 대상의 SYCL에 대한 oneAPI의 DPCPP 지원을 확장하는 플러그인입니다.

### Llama.cpp + SYCL

Llama.cpp SYCL 백엔드는 **Intel GPU**를 먼저 지원하도록 설계되었습니다. SYCL의 크로스 플랫폼 기능을 기반으로 **Nvidia GPU**(*AMD GPU 예정*)도 지원할 수 있습니다.

**Intel CPU**를 대상으로 할 때는 llama.cpp의 [Intel oneMKL](README.md#intel-onemkl) 백엔드를 사용하는 것이 좋습니다.

OpenBLAS, cuBLAS 등 다른 llama.cpp BLAS 기반 경로와 유사한 디자인을 가지고 있습니다. 초기 작업에서는 oneAPI의 [SYCLomatic](https://github.com/oneapi-src/SYCLomatic) 오픈 소스 마이그레이션 도구(상용 출시 [Intel® DPC++ 호환성 도구](https://www.intel.com/content/www/us/en/developer/tools/oneapi/dpc-compatibility-tool.html))가 이 목적으로 사용되었습니다.

## 권장 버전

온라인 CI가 없어 SYCL 백엔드가 일부 PR로 인해 오류가 발생합니다.

다음 버전은 좋은 품질로 검증되었습니다.

|Commit ID|Tag|Release|Verified  Platform|
|-|-|-|-|
|fb76ec31a9914b7761c1727303ab30380fd4f05c|b3038 |[llama-b3038-bin-win-sycl-x64.zip](https://github.com/ggerganov/llama.cpp/releases/download/b3038/llama-b3038-bin-win-sycl-x64.zip) |Arc770/Linux/oneAPI 2024.1<br>MTL Arc GPU/Windows 11/oneAPI 2024.1|


## 뉴스

- 2024.5
  - 성능 향상: Arc770에서 llama-2-7b.Q4_0의 토큰/초 처리 속도가 34 -> 37로 증가.
  - Arch Linux에서 성공적으로 검증.

- 2024.4
  - 데이터 유형 지원: GGML_TYPE_IQ4_NL, GGML_TYPE_IQ4_XS, GGML_TYPE_IQ3_XXS, GGML_TYPE_IQ3_S, GGML_TYPE_IQ2_XXS, GGML_TYPE_IQ2_XS, GGML_TYPE_IQ2_S, GGML_TYPE_IQ1_S, GGML_TYPE_IQ1_M.

- 2024.3
  - Windows의 이진 파일 배포.
  - 블로그 게시: **Intel GPU를 사용하여 LLM 실행**: [intel.com](https://www.intel.com/content/www/us/en/developer/articles/technical/run-llm-on-all-gpus-using-llama-cpp-artical.html) 또는 [medium.com](https://medium.com/@jianyu_neo/run-llm-on-all-intel-gpus-using-llama-cpp-fd2e2dcbd9bd).
  - 새로운 기준선 준비: [tag b2437](https://github.com/ggerganov/llama.cpp/tree/b2437).
  - 여러 카드 지원: **--split-mode**: [none|layer]; [row]는 개발 중입니다.
  - **--main-gpu**로 메인 GPU 할당 지원, $GGML_SYCL_DEVICE 대체.
  - level-zero를 사용하여 모든 GPU를 감지하고 최상위 **Max compute units**를 동일하게 지원.
  - 지원되는 연산자
    - hardsigmoid
    - hardswish
    - pool2d

- 2024.1
  - Intel GPU를 위한 SYCL 백엔드 생성.
  - Windows 빌드 지원

## 운영체제

| OS      | Status  | Verified                                       |
|---------|---------|------------------------------------------------|
| Linux   | Support | Ubuntu 22.04, Fedora Silverblue 39, Arch Linux |
| Windows | Support | Windows 11                                     |


## 하드웨어

### Intel GPU

**확인된 기기**

| Intel GPU                     | Status  | Verified Model                        |
|-------------------------------|---------|---------------------------------------|
| Intel Data Center Max Series  | Support | Max 1550, 1100                        |
| Intel Data Center Flex Series | Support | Flex 170                              |
| Intel Arc Series              | Support | Arc 770, 730M, Arc A750               |
| Intel built-in Arc GPU        | Support | built-in Arc GPU in Meteor Lake       |
| Intel iGPU                    | Support | iGPU in i5-1250P, i7-1260P, i7-1165G7 |

*참고 사항:*

- **메모리**
  - 장대 모델을 실행할 때 장치 메모리는 제한 요소입니다. `./bin/llama-cli`를 실행하면 로그에 로드된 모델 크기, *`llm_load_tensors: buffer_size`*가 표시됩니다.

  - 호스트에서 GPU 공유 메모리가 모델 크기에 충분한지 확인하십시오. 예를 들어, *llama-2-7b.Q4_0*은 통합 GPU에 최소 8.0GB, 별도 GPU에 최소 4.0GB가 필요합니다.

- **실행 유닛 (EU)**
  - iGPU가 80개 미만의 EU를 갖고 있다면, 실질적인 사용에 적합한 속도로 인프런스가 실행되지 않습니다.

### 다른 벤더 GPU

**검증된 기기**

| Nvidia GPU               | Status  | Verified Model |
|--------------------------|---------|----------------|
| Ampere Series            | Support | A100, A4000    |
| Ampere Series *(Mobile)* | Support | RTX 40 Series  |

## Docker
Docker 빌드 옵션은 현재 *intel GPU* 대상에만 제한됩니다.

### 이미지 빌드
```sh
# Using FP16
docker build -t llama-cpp-sycl --build-arg="GGML_SYCL_F16=ON" -f .devops/llama-cli-intel.Dockerfile .
```

*참고사항*:

기본 FP32 *(FP16 대안보다 느림)*로 빌드하려면 이전 명령어에서 `--build-arg="GGML_SYCL_F16=ON"` 인수를 제거할 수 있습니다.

또는 `.devops/llama-server-intel.Dockerfile` 을 사용하여 *"서버"* 대안을 빌드할 수 있습니다.

### 컨테이너 실행

```sh
# First, find all the DRI cards
ls -la /dev/dri
# Then, pick the card that you want to use (here for e.g. /dev/dri/card1).
docker run -it --rm -v "$(pwd):/app:Z" --device /dev/dri/renderD128:/dev/dri/renderD128 --device /dev/dri/card1:/dev/dri/card1 llama-cpp-sycl -m "/app/models/YOUR_MODEL_FILE" -p "Building a website can be done in 10 simple steps:" -n 400 -e -ngl 33
```

*참고사항:*
- Docker는 네이티브 Linux에서 성공적으로 테스트되었습니다. WSL 지원은 아직 확인되지 않았습니다.
- **호스트** 머신에 Intel GPU 드라이버를 설치해야 할 수 있습니다. (자세한 내용은 [Linux 구성](#linux)을 참조하세요.)

## Linux

### I. 환경 설정

1. **GPU 드라이버 설치**

  - **인텔 GPU**

인텔 데이터 센터 GPU 드라이버 설치 가이드 및 다운로드 페이지는 여기에서 찾을 수 있습니다: [인텔 dGPU 드라이버 가져오기](https://dgpu-docs.intel.com/driver/installation.html#ubuntu-install-steps).

*참고*: 클라이언트 GPU *(iGPU & Arc A 시리즈)* 에 대해서는 [클라이언트 iGPU 드라이버 설치](https://dgpu-docs.intel.com/driver/client/overview.html)를 참조하십시오.

설치 후 사용자를 `video` 및 `render` 그룹에 추가하십시오.

```sh
sudo usermod -aG render $USER
sudo usermod -aG video $USER
```

*참고*: 변경 사항이 적용되려면 로그아웃/재로그인하세요.

`clinfo`를 통해 설치 여부를 확인하세요:

```sh
sudo apt install clinfo
sudo clinfo -l
```

샘플 출력:

```sh
Platform #0: Intel(R) OpenCL Graphics
 `-- Device #0: Intel(R) Arc(TM) A770 Graphics

Platform #0: Intel(R) OpenCL HD Graphics
 `-- Device #0: Intel(R) Iris(R) Xe Graphics [0x9a49]
```

- **Nvidia GPU**

Nvidia GPU를 SYCL을 통해 타겟하려면 CUDA/CUBLAS 기본 요구 사항 *- README.md#cuda 에서 찾을 수 있습니다 *- 을 설치해야 합니다.

2. **Intel® oneAPI Base Toolkit 설치**

- **Intel GPU를 위한 경우**

기본 툴킷은 공식 [Intel® oneAPI Base Toolkit](https://www.intel.com/content/www/us/en/developer/tools/oneapi/base-toolkit.html) 페이지에서 얻을 수 있습니다.

Linux용 툴킷 다운로드 및 설치 지침을 따르십시오. 설치 경로 (*`/opt/intel/oneapi` 기본값*)를 포함하여 기본 설치 값을 변경하지 않는 것이 좋습니다.

다음 지침/코드 스니펫은 기본 설치 값을 가정합니다. 필요에 따라 필요한 변경 사항이 반영되어 있는지 확인하십시오.

성공적인 설치 후 SYCL은 사용 가능한 Intel 기기에 대해 활성화되며, Intel GPU에 대한 oneAPI MKL과 같은 관련 라이브러리가 함께 제공됩니다.

- **Nvidia GPU에 대한 지원 추가**

**oneAPI 플러그인**: Nvidia GPU에서 SYCL 지원을 활성화하려면 [Codeplay oneAPI 플러그인 for Nvidia GPUs](https://developer.codeplay.com/products/oneapi/nvidia/download)을 설치하십시오. 사용자는 또한 플러그인 버전이 설치된 기본 툴킷과 일치하도록 해야 합니다 *(이전 단계)*.  seamless "oneAPI on Nvidia GPU" 설정을 위해.


**oneMKL for cuBlas**: 현재 oneMKL 출시판 *(oneAPI base-toolkit에 포함됨)*에는 cuBLAS 백엔드가 포함되어 있지 않습니다. 따라서 Nvidia GPU에서 실행하려면 *cuBLAS* 백엔드를 사용하도록 구성한 upstream [oneMKL](https://github.com/oneapi-src/oneMKL)의 소스 코드를 빌드해야 합니다.

```sh
git clone https://github.com/oneapi-src/oneMKL
cd oneMKL
cmake -B buildWithCublas -DCMAKE_CXX_COMPILER=icpx -DCMAKE_C_COMPILER=icx -DENABLE_MKLGPU_BACKEND=OFF -DENABLE_MKLCPU_BACKEND=OFF -DENABLE_CUBLAS_BACKEND=ON -DTARGET_DOMAINS=blas
cmake --build buildWithCublas --config Release
```


3. **설치 및 환경 확인**

기계에서 사용 가능한 SYCL 장치를 확인하려면 `sycl-ls` 명령을 사용하십시오.
```sh
source /opt/intel/oneapi/setvars.sh
sycl-ls
```

- **인텔 GPU**

인텔 GPU를 대상으로 할 때, 사용자는 사용 가능한 SYCL 장치 중 하나 이상의 레벨 제로 장치를 기대해야 합니다. 예를 들어 아래 샘플 출력에서 [`ext_oneapi_level_zero:gpu:0`]과 같이 적어도 하나의 GPU가 존재하는지 확인하십시오.

```
[opencl:acc:0] Intel(R) FPGA Emulation Platform for OpenCL(TM), Intel(R) FPGA Emulation Device OpenCL 1.2  [2023.16.10.0.17_160000]
[opencl:cpu:1] Intel(R) OpenCL, 13th Gen Intel(R) Core(TM) i7-13700K OpenCL 3.0 (Build 0) [2023.16.10.0.17_160000]
[opencl:gpu:2] Intel(R) OpenCL Graphics, Intel(R) Arc(TM) A770 Graphics OpenCL 3.0 NEO  [23.30.26918.50]
[ext_oneapi_level_zero:gpu:0] Intel(R) Level-Zero, Intel(R) Arc(TM) A770 Graphics 1.3 [1.3.26918]
```

- **Nvidia GPU**

Nvidia GPU를 사용하는 사용자는 아래와 같이 적어도 하나의 SYCL-CUDA 장치 [`ext_oneapi_cuda:gpu`]를 기대할 수 있습니다.
```
[opencl:acc:0] Intel(R) FPGA Emulation Platform for OpenCL(TM), Intel(R) FPGA Emulation Device OpenCL 1.2  [2023.16.12.0.12_195853.xmain-hotfix]
[opencl:cpu:1] Intel(R) OpenCL, Intel(R) Xeon(R) Gold 6326 CPU @ 2.90GHz OpenCL 3.0 (Build 0) [2023.16.12.0.12_195853.xmain-hotfix]
[ext_oneapi_cuda:gpu:0] NVIDIA CUDA BACKEND, NVIDIA A100-PCIE-40GB 8.0 [CUDA 12.2]
```

### II. llama.cpp 빌드

#### Intel GPU
```sh
# Export relevant ENV variables
source /opt/intel/oneapi/setvars.sh

# Build LLAMA with MKL BLAS acceleration for intel GPU

# Option 1: Use FP32 (recommended for better performance in most cases)
cmake -B build -DGGML_SYCL=ON -DCMAKE_C_COMPILER=icx -DCMAKE_CXX_COMPILER=icpx

# Option 2: Use FP16
cmake -B build -DGGML_SYCL=ON -DCMAKE_C_COMPILER=icx -DCMAKE_CXX_COMPILER=icpx -DGGML_SYCL_F16=ON

# build all binary
cmake --build build --config Release -j -v
```

#### Nvidia GPU
```sh
# Export relevant ENV variables
export LD_LIBRARY_PATH=/path/to/oneMKL/buildWithCublas/lib:$LD_LIBRARY_PATH
export LIBRARY_PATH=/path/to/oneMKL/buildWithCublas/lib:$LIBRARY_PATH
export CPLUS_INCLUDE_DIR=/path/to/oneMKL/buildWithCublas/include:$CPLUS_INCLUDE_DIR
export CPLUS_INCLUDE_DIR=/path/to/oneMKL/include:$CPLUS_INCLUDE_DIR

# Build LLAMA with Nvidia BLAS acceleration through SYCL

# Option 1: Use FP32 (recommended for better performance in most cases)
cmake -B build -DGGML_SYCL=ON -DGGML_SYCL_TARGET=NVIDIA -DCMAKE_C_COMPILER=icx -DCMAKE_CXX_COMPILER=icpx

# Option 2: Use FP16
cmake -B build -DGGML_SYCL=ON -DGGML_SYCL_TARGET=NVIDIA -DCMAKE_C_COMPILER=icx -DCMAKE_CXX_COMPILER=icpx -DGGML_SYCL_F16=ON

# build all binary
cmake --build build --config Release -j -v

```

### III. 인프런 실행

1. 모델 가져오기 및 준비

모델 준비 방법은 일반적인 [*Prepare and Quantize*](README.md#prepare-and-quantize) 가이드를 참조하거나, [llama-2-7b.Q4_0.gguf](https://huggingface.co/TheBloke/Llama-2-7B-GGUF/blob/main/llama-2-7b.Q4_0.gguf) 모델을 예시로 다운로드할 수 있습니다.

2. oneAPI 실행 환경 활성화

```sh
source /opt/intel/oneapi/setvars.sh
```

3. 기기 정보 목록

`sycl-ls`와 유사하게, 사용 가능한 SYCL 기기를 다음과 같이 문의할 수 있습니다:

```sh
./build/bin/llama-ls-sycl-device
```
1 *intel CPU*와 1 *intel GPU* 시스템에서 이와 같은 로그의 예시는 다음과 같습니다:
```
found 6 SYCL devices:
|  |                  |                                             |Compute   |Max compute|Max work|Max sub|               |
|ID|       Device Type|                                         Name|capability|units      |group   |group  |Global mem size|
|--|------------------|---------------------------------------------|----------|-----------|--------|-------|---------------|
| 0|[level_zero:gpu:0]|               Intel(R) Arc(TM) A770 Graphics|       1.3|        512|    1024|     32|    16225243136|
| 1|[level_zero:gpu:1]|                    Intel(R) UHD Graphics 770|       1.3|         32|     512|     32|    53651849216|
| 2|    [opencl:gpu:0]|               Intel(R) Arc(TM) A770 Graphics|       3.0|        512|    1024|     32|    16225243136|
| 3|    [opencl:gpu:1]|                    Intel(R) UHD Graphics 770|       3.0|         32|     512|     32|    53651849216|
| 4|    [opencl:cpu:0]|         13th Gen Intel(R) Core(TM) i7-13700K|       3.0|         24|    8192|     64|    67064815616|
| 5|    [opencl:acc:0]|               Intel(R) FPGA Emulation Device|       1.2|         24|67108864|     64|    67064815616|
```

| Attribute              | Note                                                        |
|------------------------|-------------------------------------------------------------|
| compute capability 1.3 | Level-zero driver/runtime, recommended                      |
| compute capability 3.0 | OpenCL driver/runtime, slower than level-zero in most cases |

4. 인퍼런스 실행

두 가지 장치 선택 모드가 있습니다.

- 단일 장치: 사용자가 지정한 하나의 장치 대상을 사용합니다.
- 여러 장치: 가장 큰 Max compute-units를 가진 장치를 자동으로 선택합니다.

| Device selection | Parameter                              |
|------------------|----------------------------------------|
| Single device    | --split-mode none --main-gpu DEVICE_ID |
| Multiple devices | --split-mode layer (default)           |

예시:

- 기기 0 사용:

```sh
ZES_ENABLE_SYSMAN=1 ./build/bin/llama-cli -m models/llama-2-7b.Q4_0.gguf -p "Building a website can be done in 10 simple steps:" -n 400 -e -ngl 33 -sm none -mg 0
```
혹은 스크립트로 실행:

```sh
./examples/sycl/run_llama2.sh 0
```

- 여러 개의 장치를 사용하세요:

```sh
ZES_ENABLE_SYSMAN=1 ./build/bin/llama-cli -m models/llama-2-7b.Q4_0.gguf -p "Building a website can be done in 10 simple steps:" -n 400 -e -ngl 33 -sm layer
```

그렇지 않으면 스크립트를 실행할 수 있습니다:

```sh
./examples/sycl/run_llama2.sh
```

*참고사항:*

- 실행 시, 출력 로그에서 선택된 장치 ID를 확인하십시오. 예를 들어 다음과 같이 표시될 수 있습니다.

```sh
detect 1 SYCL GPUs: [0] with top Max compute units:512
```
 아니면
```sh
use 1 SYCL GPUs: [0] with Max compute units:512
```

## Windows

### I. 환경 설정

1. GPU 드라이버 설치

Intel GPU 드라이버 지침 및 다운로드 페이지는 여기에서 찾을 수 있습니다: [Intel GPU 드라이버 가져오기](https://www.intel.com/content/www/us/en/products/docs/discrete-gpus/arc/software/drivers.html).

2. Visual Studio 설치

이미 최신 버전의 Microsoft Visual Studio가 있는 경우, 이 단계를 건너뛸 수 있습니다. 그렇지 않은 경우, [Microsoft Visual Studio](https://visualstudio.microsoft.com/)의 공식 다운로드 페이지를 참조하십시오.

3. Intel® oneAPI Base Toolkit 설치

기본 툴킷은 공식 [Intel® oneAPI Base Toolkit](https://www.intel.com/content/www/us/en/developer/tools/oneapi/base-toolkit.html) 페이지에서 얻을 수 있습니다.

Windows용 툴킷 다운로드 및 설치 지침을 따르십시오. 설치 경로(*(`C:\Program Files (x86)\Intel\oneAPI` 기본값)*)를 기본값으로 유지하는 것이 좋습니다.

다음 지침/코드 스니펫은 기본 설치 값을 가정합니다. 그렇지 않은 경우, 적용 가능한 경우 필요한 변경 사항이 반영되었는지 확인하십시오.

b. oneAPI 실행 환경 활성화:

- 검색창에 "oneAPI"를 입력한 후 `Intel oneAPI command prompt for Intel 64 for Visual Studio 2022` 앱을 열습니다.

- 명령 프롬프트에서 다음을 사용하여 실행 환경을 활성화합니다:
```
"C:\Program Files (x86)\Intel\oneAPI\setvars.bat" intel64
```

c. 설치 확인

oneAPI 명령 프롬프트에서 다음 명령을 실행하여 사용 가능한 SYCL 장치를 출력합니다.

```
sycl-ls
```

하나 이상의 *level-zero* GPU 장치가 **[ext_oneapi_level_zero:gpu]**로 표시되어야 합니다. 아래는 *intel Iris Xe* GPU를 Level-zero SYCL 장치로 감지하는 출력 예입니다.

출력 (예시):
```
[opencl:acc:0] Intel(R) FPGA Emulation Platform for OpenCL(TM), Intel(R) FPGA Emulation Device OpenCL 1.2  [2023.16.10.0.17_160000]
[opencl:cpu:1] Intel(R) OpenCL, 11th Gen Intel(R) Core(TM) i7-1185G7 @ 3.00GHz OpenCL 3.0 (Build 0) [2023.16.10.0.17_160000]
[opencl:gpu:2] Intel(R) OpenCL Graphics, Intel(R) Iris(R) Xe Graphics OpenCL 3.0 NEO  [31.0.101.5186]
[ext_oneapi_level_zero:gpu:0] Intel(R) Level-Zero, Intel(R) Iris(R) Xe Graphics 1.3 [1.3.28044]
```

4. 빌드 도구 설치

a. Windows용 cmake 다운로드 및 설치: https://cmake.org/download/ (CMake는 Visual Studio Installer에서도 설치할 수 있습니다)
b. 새로운 Visual Studio는 Ninja를 기본으로 설치합니다. (설치되지 않았다면, 수동으로 설치하십시오: https://ninja-build.org/)


### II. llama.cpp 빌드

oneAPI 명령줄 창에서 llama.cpp 메인 디렉토리로 이동하고 다음을 실행합니다.

```
@call "C:\Program Files (x86)\Intel\oneAPI\setvars.bat" intel64 --force

# Option 1: Use FP32 (recommended for better performance in most cases)
cmake -B build -G "Ninja" -DGGML_SYCL=ON -DCMAKE_C_COMPILER=cl -DCMAKE_CXX_COMPILER=icx  -DCMAKE_BUILD_TYPE=Release

# Option 2: Or FP16
cmake -B build -G "Ninja" -DGGML_SYCL=ON -DCMAKE_C_COMPILER=cl -DCMAKE_CXX_COMPILER=icx  -DCMAKE_BUILD_TYPE=Release -DGGML_SYCL_F16=ON

cmake --build build --config Release -j
```

그렇지 않으면, 이전 지침을 캡슐화한 `win-build-sycl.bat` 래퍼를 실행합니다:
```sh
.\examples\sycl\win-build-sycl.bat
```

또는 CMake 프리셋을 사용하여 빌드하세요:
```sh
cmake --preset x64-windows-sycl-release
cmake --build build-x64-windows-sycl-release -j --target llama-cli

cmake -DGGML_SYCL_F16=ON --preset x64-windows-sycl-release
cmake --build build-x64-windows-sycl-release -j --target llama-cli

cmake --preset x64-windows-sycl-debug
cmake --build build-x64-windows-sycl-debug -j --target llama-cli
```

또는 Visual Studio를 사용하여 llama.cpp 폴더를 CMake 프로젝트로 열 수 있습니다. 프로젝트를 컴파일하기 전에 sycl CMake 프리셋 (`x64-windows-sycl-release` 또는 `x64-windows-sycl-debug`)을 선택하십시오.

*참고 사항:* 

- 최소한의 실험적 설정의 경우, 사용자는 `cmake --build build --config Release -j --target llama-cli`를 통해만 인프런 실행 파일을 빌드할 수 있습니다.

### III. 인프런 실행

1. 모델 가져오기 및 준비

모델 준비 방법은 일반 [*Prepare and Quantize*](README#prepare-and-quantize) 가이드를 참조하거나, [llama-2-7b.Q4_0.gguf](https://huggingface.co/TheBloke/Llama-2-7B-GGUF/blob/main/llama-2-7b.Q4_0.gguf) 모델을 예시로 다운로드할 수 있습니다.

2. oneAPI 실행 환경 활성화

oneAPI 명령줄 창에서 다음을 실행하고 llama.cpp 디렉토리로 이동합니다.
```
"C:\Program Files (x86)\Intel\oneAPI\setvars.bat" intel64
```

3. 기기 정보 목록

`sycl-ls`와 유사하게, 사용 가능한 SYCL 기기를 다음과 같이 문의할 수 있습니다:

```
build\bin\ls-sycl-device.exe
```

이 명령어의 출력은 1 *intel CPU*와 1 *intel GPU*가 있는 시스템에서 다음과 같이 표시됩니다.
```
found 6 SYCL devices:
|  |                  |                                             |Compute   |Max compute|Max work|Max sub|               |
|ID|       Device Type|                                         Name|capability|units      |group   |group  |Global mem size|
|--|------------------|---------------------------------------------|----------|-----------|--------|-------|---------------|
| 0|[level_zero:gpu:0]|               Intel(R) Arc(TM) A770 Graphics|       1.3|        512|    1024|     32|    16225243136|
| 1|[level_zero:gpu:1]|                    Intel(R) UHD Graphics 770|       1.3|         32|     512|     32|    53651849216|
| 2|    [opencl:gpu:0]|               Intel(R) Arc(TM) A770 Graphics|       3.0|        512|    1024|     32|    16225243136|
| 3|    [opencl:gpu:1]|                    Intel(R) UHD Graphics 770|       3.0|         32|     512|     32|    53651849216|
| 4|    [opencl:cpu:0]|         13th Gen Intel(R) Core(TM) i7-13700K|       3.0|         24|    8192|     64|    67064815616|
| 5|    [opencl:acc:0]|               Intel(R) FPGA Emulation Device|       1.2|         24|67108864|     64|    67064815616|

```

| Attribute              | Note                                                      |
|------------------------|-----------------------------------------------------------|
| compute capability 1.3 | Level-zero running time, recommended                      |
| compute capability 3.0 | OpenCL running time, slower than level-zero in most cases |


4. 인퍼런스 실행

두 가지 장치 선택 모드가 있습니다.

- 단일 장치: 사용자가 지정한 하나의 장치를 사용합니다.
- 여러 장치: 가장 큰 컴퓨트 유닛을 가진 동일한 장치를 자동으로 선택합니다.

| Device selection | Parameter                              |
|------------------|----------------------------------------|
| Single device    | --split-mode none --main-gpu DEVICE_ID |
| Multiple devices | --split-mode layer (default)           |

예시:

- 기기 0 사용:

```
build\bin\llama-cli.exe -m models\llama-2-7b.Q4_0.gguf -p "Building a website can be done in 10 simple steps:\nStep 1:" -n 400 -e -ngl 33 -s 0 -sm none -mg 0
```

- 여러 개의 장치를 사용하세요:

```
build\bin\llama-cli.exe -m models\llama-2-7b.Q4_0.gguf -p "Building a website can be done in 10 simple steps:\nStep 1:" -n 400 -e -ngl 33 -s 0 -sm layer
```
그렇지 않으면 다음 래퍼 스크립트를 실행하십시오:

```
.\examples\sycl\win-run-llama2.bat
```

참고:

- 실행 시, 출력 로그에서 선택된 장치(들) ID(들)를 확인하십시오. 예를 들어 다음과 같이 표시될 수 있습니다:

```sh
detect 1 SYCL GPUs: [0] with top Max compute units:512
```
 아니면
```sh
use 1 SYCL GPUs: [0] with Max compute units:512
```

## 환경 변수

#### 빌드

| Name               | Value                             | Function                                    |
|--------------------|-----------------------------------|---------------------------------------------|
| GGML_SYCL          | ON (mandatory)                    | Enable build with SYCL code path.           |
| GGML_SYCL_TARGET   | INTEL *(default)* \| NVIDIA       | Set the SYCL target device type.            |
| GGML_SYCL_F16      | OFF *(default)* \|ON *(optional)* | Enable FP16 build with SYCL code path.      |
| CMAKE_C_COMPILER   | icx                               | Set *icx* compiler for SYCL code path.      |
| CMAKE_CXX_COMPILER | icpx *(Linux)*, icx *(Windows)*   | Set `icpx/icx` compiler for SYCL code path. |

#### 실행 시간

| Name              | Value            | Function                                                                                                                  |
|-------------------|------------------|---------------------------------------------------------------------------------------------------------------------------|
| GGML_SYCL_DEBUG   | 0 (default) or 1 | Enable log function by macro: GGML_SYCL_DEBUG                                                                             |
| ZES_ENABLE_SYSMAN | 0 (default) or 1 | Support to get free memory of GPU by sycl::aspect::ext_intel_free_memory.<br>Recommended to use when --split-mode = layer |

## 알려진 문제

- `Split-mode:[row]` 는 지원되지 않습니다.

## 질문 및 답변

- 오류:  `error while loading shared libraries: libsycl.so.7: cannot open shared object file: No such file or directory`.

  - 가능한 원인: oneAPI 설치가 되어 있지 않거나 ENV 변수가 설정되지 않았습니다.
  - 해결 방법: *oneAPI 기본 툴킷*을 설치하고 `source /opt/intel/oneapi/setvars.sh`를 통해 ENV를 활성화합니다.

- 일반 컴파일러 오류:

  - **build** 폴더를 삭제하거나 깨끗한 빌드를 시도합니다.

- Linux에서 GPU 드라이버를 설치한 후에도 `[ext_oneapi_level_zero:gpu]`를 볼 수 없습니다.

  `sudo sycl-ls`로 확인해주세요.

  만약 목록에 존재한다면, 사용자에게 video/render 그룹을 추가한 후 **로그아웃/로그인** 또는 시스템을 다시 시작하세요:

  ```
  sudo usermod -aG render $USER
  sudo usermod -aG video $USER
  ```
  그렇지 않다면, GPU 드라이버 설치 단계를 다시 한번 확인해주세요.

### **GitHub 기여**:
문제/PR 제목에 **[SYCL]** 접두사/태그를 추가하여 SYCL 팀이 지연 없이 확인/처리할 수 있도록 도와주세요.

## TODO

- 여러 카드 실행에 대한 행 레이어 분할 지원.
