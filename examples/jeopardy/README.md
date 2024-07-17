# llama.cpp/example/jeopardy

이것은 aigoopy/llm-jeopardy/의 거의 직접적인 포트이며, 그래프 뷰어가 추가되었습니다.

 jeopardy 테스트는 다른 모델의 사실 지식을 비교하고 서로 비교하는 데 사용할 수 있습니다. 다른 테스트와 달리 논리적 추론, 창의성, 글쓰기 능력 등을 테스트하는 것이 아닙니다.


1단계: jeopardy.sh를 열고 다음을 수정하십시오:
```
MODEL=(path to your model)
MODEL_NAME=(name of your model)
prefix=(basically, if you use vicuna it's Human: , if you use something else it might be User: , etc)
opts=(add -instruct here if needed for your model, or anything else you want to test out)
```
2단계: llama.cpp 폴더에서 `jeopardy.sh`를 실행합니다.

3단계: 필요한 결과가 나올 때까지 1단계와 2단계를 반복합니다.

4단계: `graph.py`를 실행하고 지시 사항을 따릅니다. 마지막으로 최종 그래프를 생성합니다.

참고: Human 막대는 전체 원본 100개의 샘플 질문을 기반으로 합니다. 질문 수 또는 질문을 수정하면 유효하지 않습니다.
