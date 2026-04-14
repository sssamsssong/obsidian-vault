# Word2Vec: Skip-gram

#임베딩 #Word2Vec #Skipgram

## 개념

**중심 단어(target)로 주변 단어(context)를 예측**하는 모델.
CBOW의 반대 방향.

```
중심 단어 → [숨겨진 표현] → 주변 단어들 예측
   w(t)    →    h         →  w(t-2), w(t-1), w(t+1), w(t+2)
```

## 구조

$$x \in \mathbb{R}^{|V|} \xrightarrow{W} h \in \mathbb{R}^{d} \xrightarrow{W'} \hat{y}_1, \hat{y}_2, \ldots, \hat{y}_C$$

- 중심 단어 하나로 여러 문맥 단어를 동시에 예측
- 각 문맥 위치마다 독립적인 예측

## CBOW와 차이

[[word2vec_cbow]] 참고.

| | Skip-gram | CBOW |
|--|-----------|------|
| **방향** | 중심 → 문맥 | 문맥 → 중심 |
| **희소 단어 표현** | 우수 | 보통 |
| **소량 데이터** | 잘 작동 | 데이터 많아야 유리 |
| **학습 속도** | 느림 | 빠름 |

## 비용 문제와 해결책

매번 전체 어휘 $|V|$에 대한 Softmax 계산 → 매우 비쌈.

→ [[neg_sampling]] 으로 해결

## 연결 개념

- [[word2vec_cbow]] — 반대 방향 모델
- [[neg_sampling]] — 학습 효율화
- [[Ch3_Sequential/rnn]] — 학습된 임베딩이 RNN의 입력으로 사용
