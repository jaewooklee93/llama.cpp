## GGUF split 예제

GGUF 파일을 분할/병합하는 CLI입니다.

**명령줄 옵션:**

- `--split`: GGUF를 여러 개의 GGUF로 분할하는 기본 작업.
- `--split-max-size`: 분할당 최대 크기 (M 또는 G), 예: `500M` 또는 `2G`.
- `--split-max-tensors`: 각 분할당 최대 텐서 수: 기본값(128)
- `--merge`: 여러 개의 GGUF를 하나의 GGUF로 병합합니다.
