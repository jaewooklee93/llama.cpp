# llama.cpp/examples/lookup

프롬프트 검색 디코딩의 시연

https://github.com/apoorvumang/prompt-lookup-decoding

검색 디코딩의 핵심 매개변수는 `ngram_min`, `ngram_max` 및 `n_draft`입니다. 첫 두 개는 프롬프트에서 일치하는 ngram의 크기를 결정합니다. 마지막은 일치하는 경우 다음 토큰을 몇 개 작성할지 지정합니다.

더 자세한 내용은 다음을 참조하십시오.

https://github.com/ggerganov/llama.cpp/pull/4484
https://github.com/ggerganov/llama.cpp/issues/4226
