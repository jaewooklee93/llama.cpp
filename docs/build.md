# llama.cpp를 로컬로 빌드하기

**코드를 가져오기:**

```bash
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp
```

llama.cpp를 빌드하려면 네 가지 다른 옵션이 있습니다.

- `make` 사용:
  - Linux 또는 MacOS에서:

      ```bash
      make
      ```

  - Windows에서:

    1. [w64devkit](https://github.com/skeeto/w64devkit/releases)의 최신 Fortran 버전을 다운로드합니다.
    2. `w64devkit`를 PC에 압축 해제합니다.
    3. `w64devkit.exe`를 실행합니다.
    4. `cd` 명령을 사용하여 `llama.cpp` 폴더로 이동합니다.
    5. 여기에서 다음을 실행할 수 있습니다:
        ```bash
        make
        ```

  - 참고사항:
    - `Q4_0_4_4` 양자화 유형 빌드를 위해 `GGML_NO_LLAMAFILE=1` 플래그를 추가합니다. 예를 들어 `make GGML_NO_LLAMAFILE=1`을 사용합니다.
    - 컴파일 속도를 높이려면 `-j` 인수를 추가하여 여러 작업을 병렬로 실행합니다. 예를 들어 `make -j 8`은 8개의 작업을 병렬로 실행합니다.
    - 반복적인 컴파일 속도를 높이려면 [ccache](https://ccache.dev/)를 설치합니다.
    - 디버그 빌드를 위해 `make LLAMA_DEBUG=1`을 실행합니다.

- `CMake` 사용:

  ```bash
  cmake -B build
  cmake --build build --config Release
  ```

  **참고사항**:

    - `Q4_0_4_4` 양자화 유형 빌드를 위해 `-DGGML_LLAMAFILE=OFF` cmake 옵션을 추가합니다. 예를 들어 `cmake -B build -DGGML_LLAMAFILE=OFF`을 사용합니다.
    - 컴파일 속도를 높이려면 `-j` 인수를 추가하여 여러 작업을 병렬로 실행합니다. 예를 들어 `cmake --build build --config Release -j 8`은 8개의 작업을 병렬로 실행합니다.
    - 반복적인 컴파일 속도를 높이려면 [ccache](https://ccache.dev/)를 설치합니다.
    - 디버그 빌드를 위해 다음 두 가지 경우가 있습니다:

      1. 단일 구성 생성기 (예: 기본값은 `Unix Makefiles`; 이들은 `--config` 플래그를 무시합니다):

      ```bash
      cmake -B build -DCMAKE_BUILD_TYPE=Debug
      cmake --build build
      ```

      2. 다중 구성 생성기 (`-G` 매개변수가 Visual Studio, XCode...로 설정됨):

      ```bash
      cmake -B build -G "Xcode"
      cmake --build build --config Debug
      ```

- `gmake` (FreeBSD) 사용:

    1. FreeBSD의 [DRM](https://wiki.freebsd.org/Graphics)을 설치하고 활성화합니다.
    2. 사용자를 **video** 그룹에 추가합니다.
    3. 컴파일 의존성을 설치합니다.

        ```bash
        sudo pkg install gmake automake autoconf pkgconf llvm15 openblas

        gmake CC=/usr/local/bin/clang15 CXX=/usr/local/bin/clang++15 -j4
        ```

## Metal 빌드

MacOS에서 Metal은 기본적으로 활성화되어 있습니다. Metal을 사용하면 계산이 GPU에서 실행됩니다.
컴파일 시 Metal 빌드를 비활성화하려면 `GGML_NO_METAL=1` 플래그 또는 `GGML_METAL=OFF` cmake 옵션을 사용하십시오.

Metal 지원으로 빌드된 경우, `--n-gpu-layers|-ngl 0` 명령줄 인수로 GPU 추론을 명시적으로 비활성화할 수 있습니다.

## BLAS 빌드

BLAS 지원을 사용하여 프로그램을 빌드하면 배치 크기가 32보다 큰 경우 (기본값은 512) 프롬프트 처리 성능이 향상될 수 있습니다. CPU만 사용하는 BLAS 구현을 사용하면 일반 생성 성능에 영향을 미치지 않습니다. GPU가 사용되는 BLAS 구현(예: cuBLAS, hipBLAS)을 사용하면 생성 성능이 향상될 수 있습니다. 현재 빌드 및 사용할 수 있는 여러 가지 다른 BLAS 구현이 있습니다:

### 가속 프레임워크:

이 기능은 Mac PC에서만 사용 가능하며, 기본적으로 활성화되어 있습니다. 일반적인 지침을 사용하여 빌드를 진행하면 됩니다.

### OpenBLAS:

CPU만 사용하여 BLAS 가속을 제공합니다. 머신에 OpenBLAS가 설치되어 있는지 확인하십시오.

- `make`를 사용하는 경우:
  - Linux에서:
    ```bash
    make GGML_OPENBLAS=1
    ```

  - Windows에서:

    1. [w64devkit](https://github.com/skeeto/w64devkit/releases)의 최신 Fortran 버전을 다운로드합니다.
    2. [OpenBLAS for Windows](https://github.com/xianyi/OpenBLAS/releases)의 최신 버전을 다운로드합니다.
    3. PC에 `w64devkit`를 추출합니다.
    4. 다운로드한 OpenBLAS 압축 파일에서 `lib` 폴더 내의 `libopenblas.a`를 `w64devkit\x86_64-w64-mingw32\lib` 안으로 복사합니다.
    5. 동일한 OpenBLAS 압축 파일에서 `include` 폴더의 내용을 `w64devkit\x86_64-w64-mingw32\include` 안으로 복사합니다.
    6. `w64devkit.exe`를 실행합니다.
    7. `cd` 명령을 사용하여 `llama.cpp` 폴더로 이동합니다.
    8. 여기에서 다음을 실행할 수 있습니다.

        ```bash
        make GGML_OPENBLAS=1
        ```

- Linux에서 `CMake`를 사용하는 경우:

    ```bash
    cmake -B build -DGGML_BLAS=ON -DGGML_BLAS_VENDOR=OpenBLAS
    cmake --build build --config Release
    ```

### BLIS

더 자세한 내용은 [BLIS.md](./backend/BLIS.md)를 참조하세요.

### SYCL

SYCL은 다양한 하드웨어 가속기에서 프로그래밍 생산성을 향상시키기 위한 고수준 프로그래밍 모델입니다.

SYCL 기반의 llama.cpp는 **Intel GPU** (데이터 센터 Max 시리즈, Flex 시리즈, Arc 시리즈, 내장 GPU 및 iGPU)를 **지원**하는 데 사용됩니다.

자세한 내용은 [SYCL을 사용한 llama.cpp](./backend/SYCL.md)를 참조하십시오.

### Intel oneMKL

oneAPI 컴파일러를 사용하여 빌드하면 avx512 및 avx512_vnni를 지원하지 않는 인텔 프로세서에서 avx_vnni 명령 집합이 사용 가능합니다. 이 빌드 구성은 **인텔 GPU를 지원하지 않습니다**. 인텔 GPU 지원을 원하시면 [llama.cpp for SYCL](./backend/SYCL.md)를 참조하십시오.

- 수동 oneAPI 설치 사용:
  기본적으로 `GGML_BLAS_VENDOR`는 `Generic`로 설정되어 있습니다. 이미 인텔 환경 스크립트를 소스하고 cmake에서 `-DGGML_BLAS=ON`을 지정한 경우, mkl 버전의 Blas가 자동으로 선택됩니다. 그렇지 않은 경우 oneAPI를 설치하고 아래 단계를 따르십시오:
    ```bash
    source /opt/intel/oneapi/setvars.sh # oneapi-basekit docker 이미지의 경우 이 단계를 건너뛸 수 있습니다. 수동 설치에만 필요합니다.
    cmake -B build -DGGML_BLAS=ON -DGGML_BLAS_VENDOR=Intel10_64lp -DCMAKE_C_COMPILER=icx -DCMAKE_CXX_COMPILER=icpx -DGGML_NATIVE=ON
    cmake --build build --config Release
    ```

- oneAPI docker 이미지 사용:
  환경 변수를 소스하고 oneAPI를 수동으로 설치하지 않고 싶다면, 인텔 docker 컨테이너를 사용하여 코드를 빌드할 수도 있습니다: [oneAPI-basekit](https://hub.docker.com/r/intel/oneapi-basekit). 그런 다음 위에 주어진 명령을 사용할 수 있습니다.

[인텔 CPU에서 LLaMA2를 최적화하고 실행하기](https://www.intel.com/content/www/us/en/content-details/791610/optimizing-and-running-llama2-on-intel-cpu.html)에서 자세한 내용을 확인하십시오.

### CUDA

Nvidia GPU의 CUDA 코어를 사용하여 GPU 가속을 제공합니다. CUDA 툴킷이 설치되어 있는지 확인하십시오. Linux 배포판의 패키지 관리자(예: `apt install nvidia-cuda-toolkit`) 또는 [CUDA 툴킷](https://developer.nvidia.com/cuda-downloads)에서 다운로드할 수 있습니다.

Jetson 사용자의 경우 Jetson Orin을 사용하는 경우 다음을 참조하십시오: [Offical Support](https://www.jetson-ai-lab.com/tutorial_text-generation.html).  Jetson nano/TX2와 같은 오래된 모델을 사용하는 경우 컴파일 전에 추가 작업이 필요합니다.

- `make`를 사용하는 경우:
  ```bash
  make GGML_CUDA=1
  ```
- `CMake`를 사용하는 경우:

  ```bash
  cmake -B build -DGGML_CUDA=ON
  cmake --build build --config Release
  ```

환경 변수 [`CUDA_VISIBLE_DEVICES`](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#env-vars)를 사용하여 사용할 GPU(s)를 지정할 수 있습니다. 성능을 조정하기 위한 다음 컴파일 옵션도 사용할 수 있습니다:

| Option                        | Legal values           | Default | Description                                                                                                                                                                                                                                                                             |
|-------------------------------|------------------------|---------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| GGML_CUDA_FORCE_DMMV          | Boolean                | false   | Force the use of dequantization + matrix vector multiplication kernels instead of using kernels that do matrix vector multiplication on quantized data. By default the decision is made based on compute capability (MMVQ for 6.1/Pascal/GTX 1000 or higher). Does not affect k-quants. |
| GGML_CUDA_DMMV_X              | Positive integer >= 32 | 32      | Number of values in x direction processed by the CUDA dequantization + matrix vector multiplication kernel per iteration. Increasing this value can improve performance on fast GPUs. Power of 2 heavily recommended. Does not affect k-quants.                                         |
| GGML_CUDA_MMV_Y               | Positive integer       | 1       | Block size in y direction for the CUDA mul mat vec kernels. Increasing this value can improve performance on fast GPUs. Power of 2 recommended.                                                                                                                                         |
| GGML_CUDA_FORCE_MMQ           | Boolean                | false   | Force the use of custom matrix multiplication kernels for quantized models instead of FP16 cuBLAS even if there is no int8 tensor core implementation available (affects V100, RDNA3). MMQ kernels are enabled by default on GPUs with int8 tensor core support. With MMQ force enabled, speed for large batch sizes will be worse but VRAM consumption will be lower.                       |
| GGML_CUDA_FORCE_CUBLAS        | Boolean                | false   | Force the use of FP16 cuBLAS instead of custom matrix multiplication kernels for quantized models                                                                                                                                                                                       |
| GGML_CUDA_F16                 | Boolean                | false   | If enabled, use half-precision floating point arithmetic for the CUDA dequantization + mul mat vec kernels and for the q4_1 and q5_1 matrix matrix multiplication kernels. Can improve performance on relatively recent GPUs.                                                           |
| GGML_CUDA_KQUANTS_ITER        | 1 or 2                 | 2       | Number of values processed per iteration and per CUDA thread for Q2_K and Q6_K quantization formats. Setting this value to 1 can improve performance for slow GPUs.                                                                                                                     |
| GGML_CUDA_PEER_MAX_BATCH_SIZE | Positive integer       | 128     | Maximum batch size for which to enable peer access between multiple GPUs. Peer access requires either Linux or NVLink. When using NVLink enabling peer access for larger batch sizes is potentially beneficial.                                                                         |
| GGML_CUDA_FA_ALL_QUANTS       | Boolean                | false   | Compile support for all KV cache quantization type (combinations) for the FlashAttention CUDA kernels. More fine-grained control over KV cache size but compilation takes much longer.                                                                                                  |

### hipBLAS

이는 HIP 지원 AMD GPU에서 BLAS 가속을 제공합니다.
ROCm이 설치되어 있어야 합니다.
Linux 배포판의 패키지 관리자 또는 [ROCm 빠른 시작 (Linux)](https://rocm.docs.amd.com/projects/install-on-linux/en/latest/tutorial/quick-start.html#rocm-install-quick)에서 다운로드할 수 있습니다.

- `make`를 사용하는 경우:
  ```bash
  make GGML_HIPBLAS=1
  ```
- Linux용 `CMake` 사용 (gfx1030 호환 AMD GPU를 가정하는 경우):
  ```bash
  HIPCXX="$(hipconfig -l)/clang" HIP_PATH="$(hipconfig -R)" \
      cmake -S . -B build -DGGML_HIPBLAS=ON -DAMDGPU_TARGETS=gfx1030 -DCMAKE_BUILD_TYPE=Release \
      && cmake --build build --config Release -- -j 16
  ```
  Linux에서는 CPU와 통합 GPU 간에 메인 메모리를 공유하는 통합 메모리 아키텍처 (UMA)를 사용하여 `-DGGML_HIP_UMA=ON`을 설정할 수 있습니다.
  그러나 이는 통합 GPU가 아닌 비통합 GPU의 성능을 저하시키지만 (통합 GPU를 사용할 수 있게 함)

  다음과 같은 오류가 발생하면 주의하십시오.
  ```
  clang: error: cannot find ROCm device library; provide its path via '--rocm-path' or '--rocm-device-lib-path', or pass '-nogpulib' to build without ROCm device library
  ```
  `HIP_PATH` 아래에 `oclc_abi_version_400.bc` 파일이 있는 디렉토리를 찾으십시오. 그런 다음 다음을 명령의 시작 부분에 추가하십시오.
  `HIP_DEVICE_LIB_PATH=<찾은 디렉토리>`, 예를 들어:
  ```bash
  HIPCXX="$(hipconfig -l)/clang" HIP_PATH="$(hipconfig -p)" \
  HIP_DEVICE_LIB_PATH=<찾은 디렉토리> \
      cmake -S . -B build -DGGML_HIPBLAS=ON -DAMDGPU_TARGETS=gfx1030 -DCMAKE_BUILD_TYPE=Release \
      && cmake --build build -- -j 16
  ```

- `make`를 사용하는 경우 (gfx1030 대상 예시, 16개의 CPU 스레드로 빌드):
  ```bash
  make -j16 GGML_HIPBLAS=1 GGML_HIP_UMA=1 AMDGPU_TARGETS=gfx1030
  ```

- Windows용 `CMake` 사용 (x64 Native Tools Command Prompt for VS를 사용하고 gfx1100 호환 AMD GPU를 가정하는 경우):
  ```bash
  set PATH=%HIP_PATH%\bin;%PATH%
  cmake -S . -B build -G Ninja -DAMDGPU_TARGETS=gfx1100 -DGGML_HIPBLAS=ON -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++ -DCMAKE_BUILD_TYPE=Release
  cmake --build build
  ```
  `AMDGPU_TARGETS`가 원하는 컴파일 대상 GPU 아키텍처로 설정되어 있어야 합니다. 위의 예시는 `gfx1100`을 사용하며 이는 Radeon RX 7900XTX/XT/GRE에 해당합니다. 대상 목록은 [여기](https://llvm.org/docs/AMDGPUUsage.html#processors)에서 찾을 수 있습니다.
  GPU 버전 문자열을 `rocminfo | grep gfx | head -1 | awk '{print $2}'`에서 가장 중요한 버전 정보를 `gfx1035`와 같은 목록과 일치시켜 찾습니다.


환경 변수 [`HIP_VISIBLE_DEVICES`](https://rocm.docs.amd.com/en/latest/understand/gpu_isolation.html#hip-visible-devices)를 사용하여 사용할 GPU(s)를 지정할 수 있습니다.
GPU가 공식적으로 지원되지 않는 경우 환경 변수 [`HSA_OVERRIDE_GFX_VERSION`]을 유사한 GPU로 설정하여 사용할 수 있습니다. 예를 들어 RDNA2 (예: gfx1030, gfx1031 또는 gfx1035)에 대해 10.3.0 또는 RDNA3에 대해 11.0.0을 사용할 수 있습니다.
다음 컴파일 옵션도 사용하여 성능을 조정할 수 있습니다 (네, 이들은 CUDA를 가리키며 HIP는 cuBLAS 버전과 동일한 코드를 사용하기 때문입니다):

| Option                 | Legal values           | Default | Description                                                                                                                                                                                                                                    |
|------------------------|------------------------|---------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| GGML_CUDA_DMMV_X       | Positive integer >= 32 | 32      | Number of values in x direction processed by the HIP dequantization + matrix vector multiplication kernel per iteration. Increasing this value can improve performance on fast GPUs. Power of 2 heavily recommended. Does not affect k-quants. |
| GGML_CUDA_MMV_Y        | Positive integer       | 1       | Block size in y direction for the HIP mul mat vec kernels. Increasing this value can improve performance on fast GPUs. Power of 2 recommended. Does not affect k-quants.                                                                       |
| GGML_CUDA_KQUANTS_ITER | 1 or 2                 | 2       | Number of values processed per iteration and per HIP thread for Q2_K and Q6_K quantization formats. Setting this value to 1 can improve performance for slow GPUs.                                                                             |

### Vulkan

**Windows**

#### w64devkit

[w64devkit](https://github.com/skeeto/w64devkit/releases)를 다운로드하고 압축 해제하세요.

[Vulkan SDK](https://vulkan.lunarg.com/sdk/home#windows)를 다운로드하고 설치하세요. 구성 요소를 선택할 때는 Vulkan SDK Core만 필요합니다.

`w64devkit.exe`를 실행하고 Vulkan 의존성을 복사하는 다음 명령을 실행하세요:
```sh
SDK_VERSION=1.3.283.0
cp /VulkanSDK/$SDK_VERSION/Bin/glslc.exe $W64DEVKIT_HOME/bin/
cp /VulkanSDK/$SDK_VERSION/Lib/vulkan-1.lib $W64DEVKIT_HOME/x86_64-w64-mingw32/lib/
cp -r /VulkanSDK/$SDK_VERSION/Include/* $W64DEVKIT_HOME/x86_64-w64-mingw32/include/
cat > $W64DEVKIT_HOME/x86_64-w64-mingw32/lib/pkgconfig/vulkan.pc <<EOF
Name: Vulkan-Loader
Description: Vulkan Loader
Version: $SDK_VERSION
Libs: -lvulkan-1
EOF

```
`llama.cpp` 디렉토리로 이동하고 `make GGML_VULKAN=1`을 실행합니다.

#### MSYS2
[MSYS2](https://www.msys2.org/)를 설치한 후 UCRT 터미널에서 다음 명령을 실행하여 의존성을 설치합니다.
  ```sh
  pacman -S git \
      mingw-w64-ucrt-x86_64-gcc \
      mingw-w64-ucrt-x86_64-cmake \
      mingw-w64-ucrt-x86_64-vulkan-devel \
      mingw-w64-ucrt-x86_64-shaderc
  ```
`llama.cpp` 디렉토리로 이동하여 CMake을 사용하여 빌드합니다.
```sh
cmake -B build -DGGML_VULKAN=ON
cmake --build build --config Release
```

**Docker를 사용하여**:

Vulkan SDK를 설치할 필요가 없습니다. 컨테이너 내부에서 설치됩니다.

```sh
# Build the image
docker build -t llama-cpp-vulkan -f .devops/llama-cli-vulkan.Dockerfile .

# Then, use it:
docker run -it --rm -v "$(pwd):/app:Z" --device /dev/dri/renderD128:/dev/dri/renderD128 --device /dev/dri/card1:/dev/dri/card1 llama-cpp-vulkan -m "/app/models/YOUR_MODEL_FILE" -p "Building a website can be done in 10 simple steps:" -n 400 -e -ngl 33
```

**Docker 없이**:

첫째, [Vulkan SDK](https://vulkan.lunarg.com/doc/view/latest/linux/getting_started_ubuntu.html)를 설치했는지 확인해야 합니다.

예를 들어, Ubuntu 22.04 (jammy)에서 아래 명령어를 사용하십시오.

```bash
wget -qO - https://packages.lunarg.com/lunarg-signing-key-pub.asc | apt-key add -
wget -qO /etc/apt/sources.list.d/lunarg-vulkan-jammy.list https://packages.lunarg.com/vulkan/lunarg-vulkan-jammy.list
apt update -y
apt-get install -y vulkan-sdk
# To verify the installation, use the command below:
vulkaninfo
```

또는 패키지 관리자가 적절한 라이브러리를 제공할 수도 있습니다.
예를 들어 Ubuntu 22.04의 경우 `libvulkan-dev`를 설치할 수 있습니다.
Fedora 40의 경우 `vulkan-devel`, `glslc` 및 `glslang` 패키지를 설치할 수 있습니다.

그런 다음 아래 cmake 명령어를 사용하여 llama.cpp를 빌드하십시오.

```bash
cmake -B build -DGGML_VULKAN=1
cmake --build build --config Release
# Test the output binary (with "-ngl 33" to offload all layers to GPU)
./bin/llama-cli -m "PATH_TO_MODEL" -p "Hi you how are you" -n 50 -e -ngl 33 -t 4

# You should see in the output, ggml_vulkan detected your GPU. For example:
# ggml_vulkan: Using Intel(R) Graphics (ADL GT2) | uma: 1 | fp16: 1 | warp size: 32
```

### Android

Android 빌드 가이드를 읽으려면 [여기를 클릭하세요](./android.md)
