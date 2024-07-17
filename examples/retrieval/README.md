# llama.cpp/examples/retrieval

코사인 유사도 기반 간단한 검색 기술의 시연

자세한 내용:
https://github.com/ggerganov/llama.cpp/pull/6193

### 사용 방법

`retieval.cpp`는 다음과 같은 매개변수를 가지고 있습니다.
- `--context-file`: 잠금을 위한 파일 - 여러 번 지정하여 여러 파일을 잠금
- `--chunk-size`: 잠금될 각 텍스트 조각의 최소 크기
- `--chunk-separator`: 조각을 구분하는 문자열. 기본값은 줄바꿈

`retrieval` 예제는 다음과 같이 테스트할 수 있습니다.

```bash
make -j && ./llama-retrieval --model ./models/bge-base-en-v1.5-f16.gguf --top-k 3 --context-file README.md --context-file License --chunk-size 100 --chunk-separator .
```

이 코드는 주어진 모든 파일을 조각화하고 임베딩하고, 쿼리 입력을 요청하는 루프를 시작합니다.

```
Enter query:
```

각 쿼리 입력에 대해, 파일 이름, 파일 내 덩어리 위치 및 원본 텍스트와 함께 상위 k개의 덩어리가 표시됩니다.

```
Enter query: describe the mit license
batch_decode: n_tokens = 6, n_seq = 1
Top 3 similar chunks:
filename: README.md
filepos: 119
similarity: 0.762334
textdata:
png)

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](https://opensource.org/licenses/MIT)

[Roadmap](https://github.
--------------------
filename: License
filepos: 0
similarity: 0.725146
textdata:
MIT License

Copyright (c) 2023 Georgi Gerganov

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.
--------------------
filename: README.md
filepos: 9178
similarity: 0.621722
textdata:
com/cztomsik/ava) (MIT)
- [ptsochantaris/emeltal](https://github.com/ptsochantaris/emeltal)
- [pythops/tenere](https://github.
--------------------
```
