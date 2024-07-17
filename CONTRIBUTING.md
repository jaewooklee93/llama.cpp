# Pull requests

- PR을 병합하기 전에 항상 squash-merge를 사용합니다.
- 마지막 커밋에 다음 형식을 사용합니다: `<모듈> : <커밋 제목> (#<문제 번호>)`. 예를 들어: `utils : utils.py에서의 오타 수정 (#1234)`
- 테스트를 수행합니다:
  - [`tests`](tests) 폴더의 명령어를 사용합니다. 예를 들어, `./tests/test-backend-ops` 명령을 실행하면 GGML 라이브러리의 다양한 백엔드 구현을 테스트합니다.
  - [전체 CI를 로컬 머신에서 실행](ci/README.md)합니다.
- pull request가 문서 변경만 포함하는 경우 (예: README 업데이트, 새로운 wiki 페이지 추가) `[no ci]`를 커밋 제목에 추가하십시오. 이렇게 하면 불필요한 CI 확인이 건너뛰어 빌드 시간이 단축됩니다.
- PR의 복잡도를 평가하십시오 (예: `리뷰 복잡도 : 낮음`, `리뷰 복잡도 : 중간`, `리뷰 복잡도 : 높음`). 이는 유지보수 담당자들이 PR을 분류하는 데 도움이 됩니다.
  - PR 템플릿에는 `[ ]`로 표시된 리뷰 복잡도 확인란이 있으며 [편의를 위해](https://docs.github.com/en/get-started/writing-on-github/working-with-advanced-formatting/about-task-lists) `[X]`로 표시할 수 있습니다.

# 코딩 가이드라인

- 타이어드 파티  의존성, 추가 파일, 추가 헤더 등을 추가하지 않도록 합니다.
- 다른 운영 체제 및 아키텍처와의 호환성을 항상 고려합니다.
- 화려한 모던 STL 구조를 피하고 기본 `for` 루프를 사용합니다. 템플릿을 피하고 간단하게 유지합니다.
- 코드 스타일에 대한 엄격한 규칙은 없지만 코드에서 사용되는 패턴을 따르십시오 ( 들여쓰기, 공백 등). 수직 정렬은 코드를 더 읽기 쉽고 대량 편집을 용이하게 합니다.
- 꼬리 공백을 정리하고, 들여쓰기는 4개의 공백을 사용하고, 괄호는 같은 줄에 배치합니다. `void * ptr`, `int & a` 와 같이 합니다.
- 이름 지정은 일반적으로 공통 접두어를 최적화합니다. (https://github.com/ggerganov/ggml/pull/302#discussion_r1243240963 참조)
- 텐서 는 행 주문으로 데이터를 저장합니다. 0차원을 열, 1차원을 행, 2차원을 행렬로 참조합니다.
- 행렬 곱셈은 비정상적입니다: [`C = ggml_mul_mat(ctx, A, B)`](https://github.com/ggerganov/llama.cpp/blob/880e352277fc017df4d5794f0c21c44e1eae2b84/ggml.h#L1058-L1064) 는 $C^T = A B^T \Leftrightarrow C = B A^T$를 의미합니다.

![matmul](media/matmul.png)

