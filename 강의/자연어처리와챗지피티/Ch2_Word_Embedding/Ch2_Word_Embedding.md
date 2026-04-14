# Ch 2. Word Embedding — 개요

#임베딩 #NLP #Word2Vec

## 이 챕터의 흐름

```
One-hot 표현의 한계
    ├── 고차원 희소 벡터 (Curse of Dimensionality)
    └── 의미 관계 없음 (Orthogonality)
            ↓
        차원 축소
    ├── [[dim_reduction]] — PCA, LDA, Autoencoder
    └── [[word2vec_cbow]] / [[word2vec_skipgram]] — 분포 가설 기반 학습
```

## 핵심 개념 목록

- [[one_hot]] — One-hot 인코딩과 그 한계
- [[dim_reduction]] — PCA, LDA, Autoencoder
- [[word2vec_cbow]] — CBOW 모델
- [[word2vec_skipgram]] — Skip-gram 모델
- [[neg_sampling]] — Hierarchical Softmax, Negative Sampling

## 왜 Word Embedding인가?

| 표현 방식 | 특징 |
|-----------|------|
| **One-hot** | 고차원, 희소, 의미 없음 |
| **Word Embedding** | 저차원, 밀집, 의미 유사도 반영 |

분포 가설(Distributional Hypothesis):
> "비슷한 문맥에서 등장하는 단어는 비슷한 의미를 가진다"
> — J.R. Firth (1957)

## 연결 개념

- 이전 챕터: [[Ch1_ML_Basics/perceptron_nn]] — NN 구조를 임베딩 학습에 활용
- 다음 챕터: [[Ch3_Sequential]] — 임베딩 벡터를 순차 모델의 입력으로 사용
