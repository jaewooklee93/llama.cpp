# llama.cpp/example/sycl

이 예제 프로그램은 Intel GPU에서 SYCL을 사용하는 llama.cpp를 위한 도구를 제공합니다.

## 도구

|Tool Name| Function|Status|
|-|-|-|
|llama-ls-sycl-device| List all SYCL devices with ID, compute capability, max work group size, ect.|Support|

### llama-ls-sycl-device

모든 SYCL 장치를 ID, 컴퓨트 능력, 최대 작업 그룹 크기 등으로 나열합니다.

1. 모든 대상에 대해 SYCL용 llama.cpp를 빌드합니다.

2. oneAPI 실행 환경을 활성화합니다.

```
source /opt/intel/oneapi/setvars.sh
```

3. 실행

```
./build/bin/llama-ls-sycl-device
```

시작 로그에서 ID를 확인하세요, 예를 들어:

```
found 4 SYCL devices:
  Device 0: Intel(R) Arc(TM) A770 Graphics,	compute capability 1.3,
    max compute_units 512,	max work group size 1024,	max sub group size 32,	global mem size 16225243136
  Device 1: Intel(R) FPGA Emulation Device,	compute capability 1.2,
    max compute_units 24,	max work group size 67108864,	max sub group size 64,	global mem size 67065057280
  Device 2: 13th Gen Intel(R) Core(TM) i7-13700K,	compute capability 3.0,
    max compute_units 24,	max work group size 8192,	max sub group size 64,	global mem size 67065057280
  Device 3: Intel(R) Arc(TM) A770 Graphics,	compute capability 3.0,
    max compute_units 512,	max work group size 1024,	max sub group size 32,	global mem size 16225243136

```

|Attribute|Note|
|-|-|
|compute capability 1.3|Level-zero running time, recommended |
|compute capability 3.0|OpenCL running time, slower than level-zero in most cases|
