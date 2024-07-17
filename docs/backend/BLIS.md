BLIS 설치 가이드
------------------------

BLIS는 고성능 BLAS 유사 밀도 선형대수 라이브러리에 대한 휴대용 소프트웨어 프레임워크입니다. 2023년 James H. Wilkinson 수치 소프트웨어상과 2020년 SIAM Supercomputing 활동 그룹 최우수 논문상을 포함하여 수상 및 인정을 받았습니다. BLIS는 새로운 BLAS 유사 API와 기존 BLAS 루틴 호출에 대한 호환성 레이어를 제공합니다. 객체 기반 API, 타입 API, BLAS 및 CBLAS 호환성 레이어와 같은 기능을 제공합니다.

프로젝트 URL: https://github.com/flame/blis

### 준비:

BLIS 컴파일:

```bash
git clone https://github.com/flame/blis
cd blis
./configure --enable-cblas -t openmp,pthreads auto
# will install to /usr/local/ by default.
make -j
```

BLIS 설치:

```bash
sudo make install
```

openmp를 사용하는 것을 권장합니다. 사용 중인 코어를 수정하기 쉬워서요.

### llama.cpp 컴파일

Makefile:

```bash
make GGML_BLIS=1 -j
# make GGML_BLIS=1 llama-benchmark-matmult
```

CMake:

```bash
mkdir build
cd build
cmake -DGGML_BLAS=ON -DGGML_BLAS_VENDOR=FLAME ..
make -j
```

### llama.cpp 실행

BLIS 문서에 따르면, 다음과 같은 환경 변수를 설정하여 openmp의 동작을 수정할 수 있습니다.

```bash
export GOMP_CPU_AFFINITY="0-19"
export BLIS_NUM_THREADS=14
```

그리고 바이너리 파일을 일반적으로 실행합니다.


### Intel 특정 문제

일부 사용자는 `libimf.so`가 찾을 수 없다고 하는 오류 메시지를 받을 수 있습니다.
[stackoverflow 페이지](https://stackoverflow.com/questions/70687930/intel-oneapi-2022-libimf-so-no-such-file-or-directory-during-openmpi-compila)를 참조하십시오.

### 참고 자료:

1. https://github.com/flame/blis#getting-started
2. https://github.com/flame/blis/blob/master/docs/Multithreading.md
