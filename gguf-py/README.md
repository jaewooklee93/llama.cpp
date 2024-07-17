## gguf

[GGML](https://github.com/ggerganov/ggml) (GGML Universal File) 형식으로 이진 파일을 작성하는 Python 패키지입니다.

사용 예시는 [convert_hf_to_gguf.py](https://github.com/ggerganov/llama.cpp/blob/master/convert_hf_to_gguf.py)를 참조하세요.

## 설치
```sh
pip install gguf
```

## API 예시/간단한 도구

[examples/writer.py](https://github.com/ggerganov/llama.cpp/blob/master/gguf-py/examples/writer.py) — 현재 디렉토리에 `example.gguf` 파일을 생성하여 GGUF 파일을 생성하는 방법을 보여줍니다. 이 파일은 모델로 사용할 수 없습니다.

[scripts/gguf_dump.py](https://github.com/ggerganov/llama.cpp/blob/master/gguf-py/scripts/gguf_dump.py) — GGUF 파일의 메타데이터를 콘솔에 출력합니다.

[scripts/gguf_set_metadata.py](https://github.com/ggerganov/llama.cpp/blob/master/gguf-py/scripts/gguf_set_metadata.py) — 키에 따라 GGUF 파일의 간단한 메타데이터 값을 변경할 수 있습니다.

[scripts/gguf_convert_endian.py](https://github.com/ggerganov/llama.cpp/blob/master/gguf-py/scripts/gguf_convert_endian.py) — GGUF 파일의 엔디언을 변환할 수 있습니다.

[scripts/gguf_new_metadata.py](https://github.com/ggerganov/llama.cpp/blob/master/gguf-py/scripts/gguf_new_metadata.py) — 추가된/수정된/제거된 메타데이터 값을 가진 GGUF 파일을 복사합니다.

## 개발
이 패키지의 개발에 참여하는 유지보수자는 수정 가능한 모드로 설치하는 것이 좋습니다:

```sh
cd /path/to/llama.cpp/gguf-py

pip install --editable .
```

**참고**: 이는 Pip 설치를 업그레이드해야 할 수 있습니다. 편집 가능한 설치가 현재 `setup.py`를 필요로 한다는 메시지가 표시될 수 있습니다.
이 경우, Pip를 최신 버전으로 업그레이드하십시오:

```sh
pip install --upgrade pip
```

## CI를 사용한 자동 게시

지정된 형식의 태그가 생성되면 자동으로 릴리스를 생성하는 GitHub 워크플로가 있습니다.

1. `pyproject.toml`에서 버전을 업데이트합니다.
2. `gguf-vx.x.x`라는 이름의 태그를 생성합니다. 여기서 `x.x.x`는 의미 있는 버전 번호입니다.

```sh
git tag -a gguf-v1.0.0 -m "Version 1.0 release"
```

3. 태그를 푸시합니다.

```sh
git push origin --tags
```

## 수동 게시
모든 이유로 패키지를 수동으로 게시하려면 `twine` 과 `build` 가 설치되어 있어야 합니다:

```sh
pip install build twine
```

새 버전을 출시하려면 다음 단계를 따르세요.

1. `pyproject.toml`에서 버전을 업데이트합니다.
2. 패키지를 빌드합니다:

```sh
python -m build
```

3. 생성된 배포 아카이브를 업로드합니다.

```sh
python -m twine upload dist/*
```

## TODO
- [ ] 이 패키지에 명령줄 입력점으로 변환 스크립트를 포함합니다.
