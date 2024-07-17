
# llama-gguf-hash

모델과 텐서 수준에서의 차이를 감지하기 위해 GGUF 파일을 해싱하는 CLI 도구입니다.

**명령줄 옵션:**

- `--help`: 도움 메시지 표시
- `--xxh64`: 64비트 해시 모드 사용 (기본값)
- `--sha1`: sha1 사용
- `--uuid`: uuid 사용
- `--sha256`: sha256 사용
- `--all`: 모든 해시 사용
- `--no-layer`: 층별 해시 제외
- `--uuid`: UUIDv5 ID 생성
- `-c`, `--check <manifest>`: manifest와 비교

## 소개

대부분의 POSIX 시스템은 sha256sum과 같은 파일 해시 검사 프로그램을 이미 가지고 있지만, 이는 전체 파일을 검사하도록 설계되었습니다. gguf KV 저장소의 메타데이터 내용이 업데이트되었더라도 텐서 데이터의 일관성을 확인하려는 목적에서는 이상적이지 않습니다.

이 프로그램은 '전체 텐서 모델' 해시 외에도 '텐서 레이어별'로 gguf 텐서 유효성을 해시하는 데 설계되었습니다. 전체 텐서 레이어를 먼저 확인하는 것이 목적이며, 일관성 문제가 감지되면 해시를 사용하여 일관성 문제가 있는 특정 텐서 레이어를 식별할 수 있습니다.


**유지보수자를 위한 내용:**
- 개발 및 자동 테스트 중 텐서 일관성 검출
    - xxh64를 사용하여 빠르게 처리합니다.
    - 오류가 있는 텐서 레이어의 위치를 좁히는 데 도움이 되는 텐서 레이어별 해시를 제공합니다.
    - sha1은 느리지만 널리 지원되는 해시 알고리즘입니다.

**모델 제작자를 위한 내용:**
- 모델 텐서 내용에 기반한 옵션적 일관성 UUID 생성
    - UUIDv5는 데이터베이스 키에 유용합니다.
        - llama.cpp UUIDv5 Namespace: `ef001206-dadc-5f6d-a15f-3359e577d4e5`
            - `en.wikipedia.org/wiki/Llama.cpp`의 UUIDv5 URL 네임스페이스를 통해 생성되었습니다.

**모델 사용자를 위한 내용:**
- 메타데이터가 업데이트되었더라도 텐서 레이어 무결성 보장
    - 2024년 현재 여전히 매우 안전하다고 여겨지는 sha256를 사용합니다.

### 디자인 노트

- 프로그램에 인자가 제공되지 않으면 기본적으로 xxhash의 xxh32 모드를 사용하여 해시를 계산합니다. 이는 매우 빠르기 때문에 주로 자동화된 테스트에서 사용하려는 유지보수 담당자를 대상으로 합니다.
- xxhash는 각각 32비트 해시와 128비트 해시를 위한 xxh32와 xxh128를 지원하지만, 2024년 현재 대부분의 컴퓨터가 64비트이기 때문에 64비트 크기의 해시를 계산하는 데 더 적합하기 때문에 64비트 xxhash를 선택했습니다.

## 컴파일 예제

```bash
cmake -B build -DCMAKE_BUILD_TYPE=Debug -DLLAMA_FATAL_WARNINGS=ON
make -C build clean
make -C build llama-gguf-hash VERBOSE=1
./build/bin/llama-gguf-hash test.gguf
./build/bin/llama-gguf-hash --xxh64 test.gguf
./build/bin/llama-gguf-hash --sha1 test.gguf
./build/bin/llama-gguf-hash --uuid test.gguf
./build/bin/llama-gguf-hash --sha256 test.gguf
```

## 생성 및 검증 예시

생성을 위해 다음 명령어를 사용할 수 있습니다.

```bash
./llama-gguf-hash --all test.gguf > test.gguf.manifest
```

다음과 같은 manifest를 생성하는 것은 어떻게 될까요? 여러 해시 유형과 텐서 레이어별 해시를 포함합니다
(UUID는 ID이기 때문에 해시가 아닌 것을 제외합니다)

```bash
xxh64     f66e9cd66a4396a0  test.gguf:tensor_0
sha1      59f79ecefd8125a996fdf419239051a7e99e5f20  test.gguf:tensor_0
sha256    c0510d38fa060c46265e0160a85c7243096b01dd31c2f355bdbb5516b20de1bd  test.gguf:tensor_0
xxh64     7d3a1f9ac04d0537  test.gguf:tensor_1
sha1      4765f592eacf096df4628ba59476af94d767080a  test.gguf:tensor_1
sha256    8514cbcc73692a2c56bd7a33a022edd5ff819614bd23b19915d7224387f397a7  test.gguf:tensor_1
xxh64     a0af5d700049693b  test.gguf:tensor_2
sha1      25cbfbad4513cc348e2c95ebdee69d6ff2fd8753  test.gguf:tensor_2
sha256    947e6b36e20f2cc95e1d2ce1c1669d813d574657ac6b5ac5196158d454d35180  test.gguf:tensor_2
xxh64     e83fddf559d7b6a6  test.gguf:tensor_3
sha1      a9cba73e2d90f2ee3dae2548caa42bef3fe6a96c  test.gguf:tensor_3
sha256    423b044e016d8ac73c39f23f60bf01bedef5ecb03c0230accd824c91fe86f1a1  test.gguf:tensor_3
xxh64     1257733306b7992d  test.gguf:tensor_4
sha1      d7bc61db93bb685ce9d598da89717c66729b7543  test.gguf:tensor_4
sha256    79737cb3912d4201384cf7f16a1a37ff7823f23ea796cb205b6ca361ab9e3ebf  test.gguf:tensor_4
xxh64     d238d16ba4711e58  test.gguf:tensor_5
sha1      0706566c198fe1072f37e0a5135b4b5f23654c52  test.gguf:tensor_5
sha256    60949be8298eced0ecdde64487643d018407bd261691e061d9e9c3dbc9fd358b  test.gguf:tensor_5
xxh64     3fbc3b65ab8c7f39  test.gguf:tensor_6
sha1      73922a0727226a409049f6fc3172a52219ca6f00  test.gguf:tensor_6
sha256    574f4c46ff384a3b9a225eb955d2a871847a2e8b3fa59387a8252832e92ef7b0  test.gguf:tensor_6
xxh64     c22021c29854f093  test.gguf:tensor_7
sha1      efc39cece6a951188fc41e354c73bbfe6813d447  test.gguf:tensor_7
sha256    4c0410cd3c500f078ae5b21e8dc9eb79e29112713b2ab58a882f82a3868d4d75  test.gguf:tensor_7
xxh64     936df61f5d64261f  test.gguf:tensor_8
sha1      c2490296d789a4f34398a337fed8377d943d9f06  test.gguf:tensor_8
sha256    c4401313feeba0261275c3b25bd2d8fe40ce04e0f440c2980ed0e9674c30ff01  test.gguf:tensor_8
xxh64     93fd20c64421c081  test.gguf:tensor_9
sha1      7047ce1e78437a6884337a3751c7ee0421918a65  test.gguf:tensor_9
sha256    23d57cf0d7a6e90b0b3616b41300e0cd354781e812add854a5f95aa55f2bc514  test.gguf:tensor_9
xxh64     5a54d3aad816f302  test.gguf
sha1      d15be52c4ff213e823cb6dd13af7ee2f978e7042  test.gguf
sha256    7dd641b32f59b60dbd4b5420c4b0f6321ccf48f58f6ae201a3dbc4a58a27c6e4  test.gguf
```

그런 다음 기본적으로 가장 높은 보안 강도 해시를 확인하고 그에 대해 검증하는 일반적인 확인 명령을 사용할 수 있습니다.

```bash
$ ./llama-gguf-hash --check test.gguf.manifest test.gguf
manifest  test.gguf.manifest  sha256  sha1  xxh64
sha256    c0510d38fa060c46265e0160a85c7243096b01dd31c2f355bdbb5516b20de1bd  test.gguf:tensor_0  -  Ok
sha256    8514cbcc73692a2c56bd7a33a022edd5ff819614bd23b19915d7224387f397a7  test.gguf:tensor_1  -  Ok
sha256    947e6b36e20f2cc95e1d2ce1c1669d813d574657ac6b5ac5196158d454d35180  test.gguf:tensor_2  -  Ok
sha256    423b044e016d8ac73c39f23f60bf01bedef5ecb03c0230accd824c91fe86f1a1  test.gguf:tensor_3  -  Ok
sha256    79737cb3912d4201384cf7f16a1a37ff7823f23ea796cb205b6ca361ab9e3ebf  test.gguf:tensor_4  -  Ok
sha256    60949be8298eced0ecdde64487643d018407bd261691e061d9e9c3dbc9fd358b  test.gguf:tensor_5  -  Ok
sha256    574f4c46ff384a3b9a225eb955d2a871847a2e8b3fa59387a8252832e92ef7b0  test.gguf:tensor_6  -  Ok
sha256    4c0410cd3c500f078ae5b21e8dc9eb79e29112713b2ab58a882f82a3868d4d75  test.gguf:tensor_7  -  Ok
sha256    c4401313feeba0261275c3b25bd2d8fe40ce04e0f440c2980ed0e9674c30ff01  test.gguf:tensor_8  -  Ok
sha256    23d57cf0d7a6e90b0b3616b41300e0cd354781e812add854a5f95aa55f2bc514  test.gguf:tensor_9  -  Ok
sha256    7dd641b32f59b60dbd4b5420c4b0f6321ccf48f58f6ae201a3dbc4a58a27c6e4  test.gguf  -  Ok

Verification results for test.gguf.manifest - Success
```

혹은 명시적으로 더 빠른 해시를 요청할 수도 있습니다:

```bash
$ ./llama-gguf-hash --check test.gguf.manifest --xxh64 test.gguf
manifest  test.gguf.manifest  sha256  sha1  xxh64
xxh64     f66e9cd66a4396a0  test.gguf:tensor_0  -  Ok
xxh64     7d3a1f9ac04d0537  test.gguf:tensor_1  -  Ok
xxh64     a0af5d700049693b  test.gguf:tensor_2  -  Ok
xxh64     e83fddf559d7b6a6  test.gguf:tensor_3  -  Ok
xxh64     1257733306b7992d  test.gguf:tensor_4  -  Ok
xxh64     d238d16ba4711e58  test.gguf:tensor_5  -  Ok
xxh64     3fbc3b65ab8c7f39  test.gguf:tensor_6  -  Ok
xxh64     c22021c29854f093  test.gguf:tensor_7  -  Ok
xxh64     936df61f5d64261f  test.gguf:tensor_8  -  Ok
xxh64     93fd20c64421c081  test.gguf:tensor_9  -  Ok
xxh64     5a54d3aad816f302  test.gguf  -  Ok

Verification results for test.gguf.manifest - Success
```

혹시 해시가 모두 유효한지 확인하고 싶을 수도 있습니다:

```bash
$./llama-gguf-hash --check test.gguf.manifest --all test.gguf.manifest
manifest  test.gguf.manifest  sha256  sha1  xxh64
xxh64     f66e9cd66a4396a0  test.gguf:tensor_0  -  Ok
sha1      59f79ecefd8125a996fdf419239051a7e99e5f20  test.gguf:tensor_0  -  Ok
sha256    c0510d38fa060c46265e0160a85c7243096b01dd31c2f355bdbb5516b20de1bd  test.gguf:tensor_0  -  Ok
xxh64     7d3a1f9ac04d0537  test.gguf:tensor_1  -  Ok
sha1      4765f592eacf096df4628ba59476af94d767080a  test.gguf:tensor_1  -  Ok
sha256    8514cbcc73692a2c56bd7a33a022edd5ff819614bd23b19915d7224387f397a7  test.gguf:tensor_1  -  Ok
xxh64     a0af5d700049693b  test.gguf:tensor_2  -  Ok
sha1      25cbfbad4513cc348e2c95ebdee69d6ff2fd8753  test.gguf:tensor_2  -  Ok
sha256    947e6b36e20f2cc95e1d2ce1c1669d813d574657ac6b5ac5196158d454d35180  test.gguf:tensor_2  -  Ok
xxh64     e83fddf559d7b6a6  test.gguf:tensor_3  -  Ok
sha1      a9cba73e2d90f2ee3dae2548caa42bef3fe6a96c  test.gguf:tensor_3  -  Ok
sha256    423b044e016d8ac73c39f23f60bf01bedef5ecb03c0230accd824c91fe86f1a1  test.gguf:tensor_3  -  Ok
xxh64     1257733306b7992d  test.gguf:tensor_4  -  Ok
sha1      d7bc61db93bb685ce9d598da89717c66729b7543  test.gguf:tensor_4  -  Ok
sha256    79737cb3912d4201384cf7f16a1a37ff7823f23ea796cb205b6ca361ab9e3ebf  test.gguf:tensor_4  -  Ok
xxh64     d238d16ba4711e58  test.gguf:tensor_5  -  Ok
sha1      0706566c198fe1072f37e0a5135b4b5f23654c52  test.gguf:tensor_5  -  Ok
sha256    60949be8298eced0ecdde64487643d018407bd261691e061d9e9c3dbc9fd358b  test.gguf:tensor_5  -  Ok
xxh64     3fbc3b65ab8c7f39  test.gguf:tensor_6  -  Ok
sha1      73922a0727226a409049f6fc3172a52219ca6f00  test.gguf:tensor_6  -  Ok
sha256    574f4c46ff384a3b9a225eb955d2a871847a2e8b3fa59387a8252832e92ef7b0  test.gguf:tensor_6  -  Ok
xxh64     c22021c29854f093  test.gguf:tensor_7  -  Ok
sha1      efc39cece6a951188fc41e354c73bbfe6813d447  test.gguf:tensor_7  -  Ok
sha256    4c0410cd3c500f078ae5b21e8dc9eb79e29112713b2ab58a882f82a3868d4d75  test.gguf:tensor_7  -  Ok
xxh64     936df61f5d64261f  test.gguf:tensor_8  -  Ok
sha1      c2490296d789a4f34398a337fed8377d943d9f06  test.gguf:tensor_8  -  Ok
sha256    c4401313feeba0261275c3b25bd2d8fe40ce04e0f440c2980ed0e9674c30ff01  test.gguf:tensor_8  -  Ok
xxh64     93fd20c64421c081  test.gguf:tensor_9  -  Ok
sha1      7047ce1e78437a6884337a3751c7ee0421918a65  test.gguf:tensor_9  -  Ok
sha256    23d57cf0d7a6e90b0b3616b41300e0cd354781e812add854a5f95aa55f2bc514  test.gguf:tensor_9  -  Ok
xxh64     5a54d3aad816f302  test.gguf  -  Ok
sha1      d15be52c4ff213e823cb6dd13af7ee2f978e7042  test.gguf  -  Ok
sha256    7dd641b32f59b60dbd4b5420c4b0f6321ccf48f58f6ae201a3dbc4a58a27c6e4  test.gguf  -  Ok

Verification results for test.gguf.manifest - Success
```


## 사용된 암호/해시 라이브러리

이러한 마이크로 C 라이브러리 의존성은 [clib C 패키지 관리자](https://github.com/clibs)를 통해 설치되었습니다.

- https://github.com/Cyan4973/xxHash
- https://github.com/clibs/sha1/
- https://github.com/jb55/sha256.c
