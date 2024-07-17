
## 안드로이드

Llama.cpp를 안드로이드 기기에서 실행하는 방법에 대한 정보입니다.

### 필요한 것들

* 안드로이드 기기
* Termux 앱
* Llama.cpp 코드
* 모델 가중치 파일

### 설치 및 설정

1. Termux 앱을 설치하고 실행합니다.
2. 다음 명령어를 사용하여 필요한 패키지를 설치합니다.

```bash
apt update
apt upgrade
 apt install git python3 python3-pip
```
3. Llama.cpp 코드를 Termux에 클론합니다.

```bash
cd /data/data/com.termux/files/usr/bin
git clone https://github.com/ggerganov/llama.cpp.git
```
4. 모델 가중치 파일을 Termux에 복사합니다.
5. Termux에서 Llama.cpp 코드를 빌드합니다.

```bash
cd llama.cpp
make
```
6. 빌드된 Llama.cpp 실행 파일을 사용하여 안드로이드 기기에서 모델을 실행합니다.

### 사용 방법

Llama.cpp를 실행하면 명령 프롬프트에서 사용자 입력을 받고 텍스트를 생성합니다.

### 추가 정보

* 안드로이드 기기의 사양에 따라 실행 속도가 다를 수 있습니다.
* 더 많은 정보는 Llama.cpp 공식 문서를 참조하십시오.

https://github.com/ggerganov/llama.cpp


# 안드로이드

## Termux를 사용한 Android 빌드
[Termux](https://github.com/termux/termux-app#installation)는 루트 권한 없이 안드로이드 기기에서 `llama.cpp`를 실행하는 방법입니다.
```
apt update && apt upgrade -y
apt install git make cmake
```

최적의 성능을 위해 모델을 `~/` 디렉토리 안으로 옮기는 것이 좋습니다:
```
cd storage/downloads
mv model.gguf ~/
```

`llama.cpp`를 빌드하려면 [코드를 가져오세요](https://github.com/ggerganov/llama.cpp#get-the-code) & [Linux 빌드 지침](https://github.com/ggerganov/llama.cpp#build)을 따르세요.

## Android NDK를 사용하여 프로젝트 빌드
[Android NDK](https://developer.android.com/ndk)를 가져오고 CMake를 사용하여 빌드합니다.

컴퓨터에서 다음 명령을 실행하여 모바일 기기에 NDK를 다운로드하지 않도록 합니다. 또는 Termux에서도 이 작업을 수행할 수 있습니다.
```
$ mkdir build-android
$ cd build-android
$ export NDK=<your_ndk_directory>
$ cmake -DCMAKE_TOOLCHAIN_FILE=$NDK/build/cmake/android.toolchain.cmake -DANDROID_ABI=arm64-v8a -DANDROID_PLATFORM=android-23 -DCMAKE_C_FLAGS=-march=armv8.4a+dotprod ..
$ make
```

기기에서 [termux](https://github.com/termux/termux-app#installation)를 설치하고 `termux-setup-storage`를 실행하여 SD 카드에 액세스하세요 (Android 11 이상이면 명령어를 두 번 실행하세요).

마지막으로, 빌드된 `llama` 바이너리와 모델 파일을 기기 저장소로 복사합니다. Android sdcard의 파일 권한을 변경할 수 없기 때문에, 실행 가능한 파일을 `/data/data/com.termux/files/home/bin` 경로로 복사한 후 Termux에서 다음 명령어를 실행하여 실행 권한을 추가하세요.

( `adb push`를 사용하여 빌드된 실행 파일을 `/sdcard/llama.cpp/bin` 경로로 전송했다고 가정)
```
$cp -r /sdcard/llama.cpp/bin /data/data/com.termux/files/home/
$cd /data/data/com.termux/files/home/bin
$chmod +x ./*
```

모델 [llama-2-7b-chat.Q4_K_M.gguf](https://huggingface.co/TheBloke/Llama-2-7B-Chat-GGUF/blob/main/llama-2-7b-chat.Q4_K_M.gguf)을 다운로드하고 `/sdcard/llama.cpp/` 로 이동시킨 후 `/data/data/com.termux/files/home/model/` 로 이동시킵니다.

```
$mv /sdcard/llama.cpp/llama-2-7b-chat.Q4_K_M.gguf /data/data/com.termux/files/home/model/
```

이제 대화를 시작할 수 있습니다:
```
$cd /data/data/com.termux/files/home/bin
$./llama-cli -m ../model/llama-2-7b-chat.Q4_K_M.gguf -n 128 -cml
```

Pixel 5 휴대폰에서 실행되는 상호 작용형 세션의 데모입니다:

https://user-images.githubusercontent.com/271616/225014776-1d567049-ad71-4ef2-b050-55b0b3b9274c.mp4
