## 생성적 표현 지시 튜닝 (GRIT) 예시
[gritlm]은 프롬프트의 지시에 따라 엠베딩을 생성하거나 "일반적인" 텍스트 생성을 수행할 수 있는 모델입니다.

* 논문: https://arxiv.org/pdf/2402.09906.pdf

### 검색 증강 생성(RAG) 사용 사례
`gritlm`의 하나의 사용 사례는 RAG와 함께 사용하는 것입니다. RAG가 어떻게 작동하는지 기억하면
문서를 사용하여 대규모 언어 모델(LLM)을 기반으로 묶는 것이며, 이러한 문서에 대해 토큰 임베딩을 생성합니다. 이러한 토큰 임베딩을 벡터 데이터베이스에 저장합니다.

쿼리를 수행할 때 LLM을 프롬프트할 때, 먼저 쿼리에 대해 토큰 임베딩을 생성한 다음 가장 유사한 벡터를 검색하기 위해 벡터 데이터베이스를 검색하고, 이러한 문서를 LLM에 맥락으로 전달하도록 반환합니다. 그런 다음 쿼리와 맥락이 LLM에 전달되며, LLM은 쿼리에 대해 다시 토큰 임베딩을 생성해야 합니다. 그러나 gritlm이 사용되면 첫 번째 쿼리가 캐시될 수 있으며, 두 번째 쿼리 토큰화 생성은 전혀 수행할 필요가 없습니다.

### 예제 실행
Grit 모델 다운로드:
```console
$ scripts/hf.sh --repo cohesionet/GritLM-7B_gguf --file gritlm-7b_q4_1.gguf --outdir models
```

다운로드한 모델을 사용하여 예제를 실행하세요:
```console
$ ./llama-gritlm -m models/gritlm-7b_q4_1.gguf

Cosine similarity between "Bitcoin: A Peer-to-Peer Electronic Cash System" and "A purely peer-to-peer version of electronic cash w" is: 0.605
Cosine similarity between "Bitcoin: A Peer-to-Peer Electronic Cash System" and "All text-based language problems can be reduced to" is: 0.103
Cosine similarity between "Generative Representational Instruction Tuning" and "A purely peer-to-peer version of electronic cash w" is: 0.112
Cosine similarity between "Generative Representational Instruction Tuning" and "All text-based language problems can be reduced to" is: 0.547

Oh, brave adventurer, who dared to climb
The lofty peak of Mt. Fuji in the night,
When shadows lurk and ghosts do roam,
And darkness reigns, a fearsome sight.

Thou didst set out, with heart aglow,
To conquer this mountain, so high,
And reach the summit, where the stars do glow,
And the moon shines bright, up in the sky.

Through the mist and fog, thou didst press on,
With steadfast courage, and a steadfast will,
Through the darkness, thou didst not be gone,
But didst climb on, with a steadfast skill.

At last, thou didst reach the summit's crest,
And gazed upon the world below,
And saw the beauty of the night's best,
And felt the peace, that only nature knows.

Oh, brave adventurer, who dared to climb
The lofty peak of Mt. Fuji in the night,
Thou art a hero, in the eyes of all,
For thou didst conquer this mountain, so bright.
```

[gritlm]: https://github.com/ContextualAI/gritlm
