# llama.cpp/example/main-cmake-pkg

이 프로그램은 재사용 가능한 CMake 패키지를 사용하여 [llama-cli](../main)를 빌드합니다. [llama.cpp](https://github.com/ggerganov/llama.cpp)를 소스 트리 외부에 있는 프로젝트에 편리하게 포함하는 데 `find_package()` CMake 명령을 사용하는 방법을 보여줍니다.

## 빌드

이 예제가 "소스 트리 외부"이기 때문에, 먼저 CMake을 사용하여 llama.cpp를 빌드/설치하는 것이 중요합니다. 여기 예시가 제공되지만, 자세한 빌드 지침은 [llama.cpp 빌드 지침](../..)을 참조하십시오.

### 고려 사항

하드웨어 가속 라이브러리가 사용될 때 (예: CUDA, Metal 등), CMake은 관련된 CMake 패키지를 찾을 수 있어야 합니다.

### llama.cpp를 빌드하고 C:\LlamaCPP 디렉토리에 설치하기

```cmd
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp
cmake -B build -DBUILD_SHARED_LIBS=OFF -G "Visual Studio 17 2022" -A x64
cmake --build build --config Release
cmake --install build --prefix C:/LlamaCPP
```

### llama-cli-cmake-pkg 빌드하기


```cmd
cd ..\examples\main-cmake-pkg
cmake -B build -DBUILD_SHARED_LIBS=OFF -DCMAKE_PREFIX_PATH="C:/LlamaCPP/lib/cmake/Llama" -G "Visual Studio 17 2022" -A x64
cmake --build build --config Release
cmake --install build --prefix C:/MyLlamaApp
```
