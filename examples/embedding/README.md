# llama.cpp/example/embedding

이 예제는 llama.cpp를 사용하여 주어진 텍스트의 고차원 임베딩 벡터를 생성하는 방법을 보여줍니다.

## 빠른 시작

바로 시작하려면 다음 명령을 실행하고, 사용 중인 모델의 경로를 올바르게 지정하십시오.

### Unix 기반 시스템 (Linux, macOS 등):

```bash
./llama-embedding -m ./path/to/model --log-disable -p "Hello World!" 2>/dev/null
```

### Windows:

```powershell
llama-embedding.exe -m ./path/to/model --log-disable -p "Hello World!" 2>$null
```

위 명령어를 실행하면 공백으로 구분된 부동 소수점 값이 출력됩니다.

## 추가 매개변수
### --embd-normalize $integer$
| $integer$ | description         | formula |
|-----------|---------------------|---------|
| $-1$      | none                |
| $0$       | max absolute int16  | $\Large{{32760 * x_i} \over\max \lvert x_i\rvert}$
| $1$       | taxicab             | $\Large{x_i \over\sum \lvert x_i\rvert}$
| $2$       | euclidean (default) | $\Large{x_i \over\sqrt{\sum x_i^2}}$
| $>2$      | p-norm              | $\Large{x_i \over\sqrt[p]{\sum \lvert x_i\rvert^p}}$

### --embd-output-format $'string'$
| $'string'$ | description                  |  |
|------------|------------------------------|--|
| ''         | same as before               | (default)
| 'array'    | single embeddings            | $[[x_1,...,x_n]]$
|            | multiple embeddings          | $[[x_1,...,x_n],[x_1,...,x_n],...,[x_1,...,x_n]]$
| 'json'     | openai style                 |
| 'json+'    | add cosine similarity matrix |

### --embd-separator "string"
| $"string"$   | |
|--------------|-|
| "\n"         | (default)
| "<#embSep#>" | for exemple
| "<#sep#>"    | other exemple

## examples
### Unix 기반 시스템 (Linux, macOS 등):

```bash
./embedding -p 'Castle<#sep#>Stronghold<#sep#>Dog<#sep#>Cat' --embd-separator '<#sep#>' --embd-normalize 2  --embd-output-format '' -m './path/to/model.gguf' --n-gpu-layers 99 --log-disable 2>/dev/null
```

### Windows:

```powershell
embedding.exe -p 'Castle<#sep#>Stronghold<#sep#>Dog<#sep#>Cat' --embd-separator '<#sep#>' --embd-normalize 2  --embd-output-format '' -m './path/to/model.gguf' --n-gpu-layers 99 --log-disable 2>/dev/null
```
