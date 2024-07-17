# llama.cpp/examples/llama-bench

llama.cpp의 성능 테스트 도구입니다.

## 표지

1. [문법](#문법)
2. [예시](#예시)
    1. [다양한 모델을 사용한 텍스트 생성](#다양한-모델을-사용한-텍스트-생성)
    2. [다양한 배치 크기로 프롬프트 처리](#다양한-배치-크기로-프롬프트-처리)
    3. [다양한 스레드 수](#다양한-스레드-수)
    4. [GPU로 오프로드되는 레이어 수의 차이](#GPU로-오프로드되는-레이어-수의-차이)
3. [출력 형식](#출력-형식)
    1. [Markdown](#markdown)
    2. [CSV](#csv)
    3. [JSON](#json)
    4. [SQL](#sql)

## 구문

```
usage: ./llama-bench [options]

options:
  -h, --help
  -m, --model <filename>              (default: models/7B/ggml-model-q4_0.gguf)
  -p, --n-prompt <n>                  (default: 512)
  -n, --n-gen <n>                     (default: 128)
  -pg <pp,tg>                         (default: 512,128)
  -b, --batch-size <n>                (default: 2048)
  -ub, --ubatch-size <n>              (default: 512)
  -ctk, --cache-type-k <t>            (default: f16)
  -ctv, --cache-type-v <t>            (default: f16)
  -t, --threads <n>                   (default: 16)
  -ngl, --n-gpu-layers <n>            (default: 99)
  -sm, --split-mode <none|layer|row>  (default: layer)
  -mg, --main-gpu <i>                 (default: 0)
  -nkvo, --no-kv-offload <0|1>        (default: 0)
  -fa, --flash-attn <0|1>             (default: 0)
  -mmp, --mmap <0|1>                  (default: 1)
  --numa <distribute|isolate|numactl> (default: disabled)
  -embd, --embeddings <0|1>           (default: 0)
  -ts, --tensor-split <ts0/ts1/..>    (default: 0)
  -r, --repetitions <n>               (default: 5)
  -o, --output <csv|json|md|sql>      (default: md)
  -v, --verbose                       (default: 0)

Multiple values can be given for each parameter by separating them with ',' or by specifying the parameter multiple times.
```

llama-bench는 다음 세 가지 유형의 테스트를 수행할 수 있습니다.

- 프롬프트 처리 (pp): 배치로 프롬프트를 처리합니다 (`-p`)
- 텍스트 생성 (tg): 토큰 시퀀스를 생성합니다 (`-n`)
- 프롬프트 처리 + 텍스트 생성 (pg): 프롬프트를 처리한 후 토큰 시퀀스를 생성합니다 (`-pg`)

`-r`, `-o` 및 `-v`를 제외한 모든 옵션은 여러 번 지정하여 여러 테스트를 실행할 수 있습니다. 각 pp 및 tg 테스트는 지정된 옵션의 모든 조합으로 실행됩니다. 옵션에 대한 여러 값을 지정하려면 값을 쉼표로 구분할 수 있습니다 (예: `-n 16,32`), 또는 옵션을 여러 번 지정할 수 있습니다 (예: `-n 16 -n 32`).

각 테스트는 `-r`에 지정된 횟수만큼 반복되며, 결과는 평균됩니다. 결과는 초당 평균 토큰 (t/s) 및 표준 편차로 제공됩니다. 일부 출력 형식 (예: json)은 또한 각 반복의 개별 결과를 포함합니다.

기타 옵션에 대한 설명은 [주요 예제](../main/README.md)를 참조하십시오.

참고:

- SYCL 백엔드를 사용할 때, 일부 경우에 걸림 문제가 발생할 수 있습니다. `--mmp 0`을 설정하십시오.

## 예시

### 다양한 모델로 텍스트 생성하기

```sh
$ ./llama-bench -m models/7B/ggml-model-q4_0.gguf -m models/13B/ggml-model-q4_0.gguf -p 0 -n 128,256,512
```

| model                          |       size |     params | backend    | ngl | test       |              t/s |
| ------------------------------ | ---------: | ---------: | ---------- | --: | ---------- | ---------------: |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CUDA       |  99 | tg 128     |    132.19 ± 0.55 |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CUDA       |  99 | tg 256     |    129.37 ± 0.54 |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CUDA       |  99 | tg 512     |    123.83 ± 0.25 |
| llama 13B mostly Q4_0          |   6.86 GiB |    13.02 B | CUDA       |  99 | tg 128     |     82.17 ± 0.31 |
| llama 13B mostly Q4_0          |   6.86 GiB |    13.02 B | CUDA       |  99 | tg 256     |     80.74 ± 0.23 |
| llama 13B mostly Q4_0          |   6.86 GiB |    13.02 B | CUDA       |  99 | tg 512     |     78.08 ± 0.07 |

### 다양한 배치 크기로 프롬프트 처리

```sh
$ ./llama-bench -n 0 -p 1024 -b 128,256,512,1024
```

| model                          |       size |     params | backend    | ngl |    n_batch | test       |              t/s |
| ------------------------------ | ---------: | ---------: | ---------- | --: | ---------: | ---------- | ---------------: |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CUDA       |  99 |        128 | pp 1024    |   1436.51 ± 3.66 |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CUDA       |  99 |        256 | pp 1024    |  1932.43 ± 23.48 |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CUDA       |  99 |        512 | pp 1024    |  2254.45 ± 15.59 |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CUDA       |  99 |       1024 | pp 1024    |  2498.61 ± 13.58 |

### 다른 스레드 수

```sh
$ ./llama-bench -n 0 -n 16 -p 64 -t 1,2,4,8,16,32
```

| model                          |       size |     params | backend    |    threads | test       |              t/s |
| ------------------------------ | ---------: | ---------: | ---------- | ---------: | ---------- | ---------------: |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CPU        |          1 | pp 64      |      6.17 ± 0.07 |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CPU        |          1 | tg 16      |      4.05 ± 0.02 |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CPU        |          2 | pp 64      |     12.31 ± 0.13 |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CPU        |          2 | tg 16      |      7.80 ± 0.07 |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CPU        |          4 | pp 64      |     23.18 ± 0.06 |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CPU        |          4 | tg 16      |     12.22 ± 0.07 |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CPU        |          8 | pp 64      |     32.29 ± 1.21 |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CPU        |          8 | tg 16      |     16.71 ± 0.66 |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CPU        |         16 | pp 64      |     33.52 ± 0.03 |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CPU        |         16 | tg 16      |     15.32 ± 0.05 |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CPU        |         32 | pp 64      |     59.00 ± 1.11 |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CPU        |         32 | tg 16      |     16.41 ± 0.79 ||

### GPU로 오프로드된 레이어 수의 차이

```sh
$ ./llama-bench -ngl 10,20,30,31,32,33,34,35
```

| model                          |       size |     params | backend    | ngl | test       |              t/s |
| ------------------------------ | ---------: | ---------: | ---------- | --: | ---------- | ---------------: |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CUDA       |  10 | pp 512     |    373.36 ± 2.25 |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CUDA       |  10 | tg 128     |     13.45 ± 0.93 |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CUDA       |  20 | pp 512     |    472.65 ± 1.25 |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CUDA       |  20 | tg 128     |     21.36 ± 1.94 |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CUDA       |  30 | pp 512     |   631.87 ± 11.25 |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CUDA       |  30 | tg 128     |     40.04 ± 1.82 |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CUDA       |  31 | pp 512     |    657.89 ± 5.08 |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CUDA       |  31 | tg 128     |     48.19 ± 0.81 |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CUDA       |  32 | pp 512     |    688.26 ± 3.29 |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CUDA       |  32 | tg 128     |     54.78 ± 0.65 |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CUDA       |  33 | pp 512     |    704.27 ± 2.24 |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CUDA       |  33 | tg 128     |     60.62 ± 1.76 |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CUDA       |  34 | pp 512     |    881.34 ± 5.40 |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CUDA       |  34 | tg 128     |     71.76 ± 0.23 |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CUDA       |  35 | pp 512     |   2400.01 ± 7.72 |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CUDA       |  35 | tg 128     |    131.66 ± 0.49 |

## 출력 형식

기본적으로 llama-bench는 결과를 markdown 형식으로 출력합니다. `-o` 옵션을 사용하여 다른 형식으로 결과를 출력할 수 있습니다.

### 마크다운

```sh
$ ./llama-bench -o md
```

| model                          |       size |     params | backend    | ngl | test       |              t/s |
| ------------------------------ | ---------: | ---------: | ---------- | --: | ---------- | ---------------: |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CUDA       |  99 | pp 512     |  2368.80 ± 93.24 |
| llama 7B mostly Q4_0           |   3.56 GiB |     6.74 B | CUDA       |  99 | tg 128     |    131.42 ± 0.59 |

### CSV

```sh
$ ./llama-bench -o csv
```

```csv
build_commit,build_number,cuda,metal,gpu_blas,blas,cpu_info,gpu_info,model_filename,model_type,model_size,model_n_params,n_batch,n_threads,f16_kv,n_gpu_layers,main_gpu,mul_mat_q,tensor_split,n_prompt,n_gen,test_time,avg_ns,stddev_ns,avg_ts,stddev_ts
"3469684","1275","1","0","0","1","1","13th Gen Intel(R) Core(TM) i9-13900K","NVIDIA GeForce RTX 3090 Ti","models/7B/ggml-model-q4_0.gguf","llama 7B mostly Q4_0","3825065984","6738415616","512","16","1","99","0","1","0.00","512","0","2023-09-23T12:09:01Z","212155977","732372","2413.341687","8.305961"
"3469684","1275","1","0","0","1","1","13th Gen Intel(R) Core(TM) i9-13900K","NVIDIA GeForce RTX 3090 Ti","models/7B/ggml-model-q4_0.gguf","llama 7B mostly Q4_0","3825065984","6738415616","512","16","1","99","0","1","0.00","0","128","2023-09-23T12:09:02Z","969320879","2728399","132.052051","0.371342"
```

### JSON

```sh
$ ./llama-bench -o json
```

```json
[
  {
    "build_commit": "3469684",
    "build_number": 1275,
    "cuda": true,
    "metal": false,
    "gpu_blas": true,
    "blas": true,
    "cpu_info": "13th Gen Intel(R) Core(TM) i9-13900K",
    "gpu_info": "NVIDIA GeForce RTX 3090 Ti",
    "model_filename": "models/7B/ggml-model-q4_0.gguf",
    "model_type": "llama 7B mostly Q4_0",
    "model_size": 3825065984,
    "model_n_params": 6738415616,
    "n_batch": 512,
    "n_threads": 16,
    "f16_kv": true,
    "n_gpu_layers": 99,
    "main_gpu": 0,
    "mul_mat_q": true,
    "tensor_split": "0.00",
    "n_prompt": 512,
    "n_gen": 0,
    "test_time": "2023-09-23T12:09:57Z",
    "avg_ns": 212365953,
    "stddev_ns": 985423,
    "avg_ts": 2410.974041,
    "stddev_ts": 11.163766,
    "samples_ns": [ 213837238, 211635853, 212328053, 211329715, 212698907 ],
    "samples_ts": [ 2394.34, 2419.25, 2411.36, 2422.75, 2407.16 ]
  },
  {
    "build_commit": "3469684",
    "build_number": 1275,
    "cuda": true,
    "metal": false,
    "gpu_blas": true,
    "blas": true,
    "cpu_info": "13th Gen Intel(R) Core(TM) i9-13900K",
    "gpu_info": "NVIDIA GeForce RTX 3090 Ti",
    "model_filename": "models/7B/ggml-model-q4_0.gguf",
    "model_type": "llama 7B mostly Q4_0",
    "model_size": 3825065984,
    "model_n_params": 6738415616,
    "n_batch": 512,
    "n_threads": 16,
    "f16_kv": true,
    "n_gpu_layers": 99,
    "main_gpu": 0,
    "mul_mat_q": true,
    "tensor_split": "0.00",
    "n_prompt": 0,
    "n_gen": 128,
    "test_time": "2023-09-23T12:09:59Z",
    "avg_ns": 977425219,
    "stddev_ns": 9268593,
    "avg_ts": 130.965708,
    "stddev_ts": 1.238924,
    "samples_ns": [ 984472709, 974901233, 989474741, 970729355, 967548060 ],
    "samples_ts": [ 130.019, 131.295, 129.362, 131.86, 132.293 ]
  }
]
```

### SQL

SQL 출력은 SQLite 데이터베이스로 가져올 수 있습니다. 출력을 `sqlite3` 명령줄 도구로 파이프 라인하여 결과를 데이터베이스에 추가할 수 있습니다.

```sh
$ ./llama-bench -o sql
```

```sql
CREATE TABLE IF NOT EXISTS test (
  build_commit TEXT,
  build_number INTEGER,
  cuda INTEGER,
  metal INTEGER,
  gpu_blas INTEGER,
  blas INTEGER,
  cpu_info TEXT,
  gpu_info TEXT,
  model_filename TEXT,
  model_type TEXT,
  model_size INTEGER,
  model_n_params INTEGER,
  n_batch INTEGER,
  n_threads INTEGER,
  f16_kv INTEGER,
  n_gpu_layers INTEGER,
  main_gpu INTEGER,
  mul_mat_q INTEGER,
  tensor_split TEXT,
  n_prompt INTEGER,
  n_gen INTEGER,
  test_time TEXT,
  avg_ns INTEGER,
  stddev_ns INTEGER,
  avg_ts REAL,
  stddev_ts REAL
);

INSERT INTO test (build_commit, build_number, cuda, metal, gpu_blas, blas, cpu_info, gpu_info, model_filename, model_type, model_size, model_n_params, n_batch, n_threads, f16_kv, n_gpu_layers, main_gpu, mul_mat_q, tensor_split, n_prompt, n_gen, test_time, avg_ns, stddev_ns, avg_ts, stddev_ts) VALUES ('3469684', '1275', '1', '0', '0', '1', '1', '13th Gen Intel(R) Core(TM) i9-13900K', 'NVIDIA GeForce RTX 3090 Ti', 'models/7B/ggml-model-q4_0.gguf', 'llama 7B mostly Q4_0', '3825065984', '6738415616', '512', '16', '1', '99', '0', '1', '0.00', '512', '0', '2023-09-23T12:10:30Z', '212693772', '743623', '2407.240204', '8.409634');
INSERT INTO test (build_commit, build_number, cuda, metal, gpu_blas, blas, cpu_info, gpu_info, model_filename, model_type, model_size, model_n_params, n_batch, n_threads, f16_kv, n_gpu_layers, main_gpu, mul_mat_q, tensor_split, n_prompt, n_gen, test_time, avg_ns, stddev_ns, avg_ts, stddev_ts) VALUES ('3469684', '1275', '1', '0', '0', '1', '1', '13th Gen Intel(R) Core(TM) i9-13900K', 'NVIDIA GeForce RTX 3090 Ti', 'models/7B/ggml-model-q4_0.gguf', 'llama 7B mostly Q4_0', '3825065984', '6738415616', '512', '16', '1', '99', '0', '1', '0.00', '0', '128', '2023-09-23T12:10:31Z', '977925003', '4037361', '130.891159', '0.537692');
```
