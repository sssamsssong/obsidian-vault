# Word2Vec: CBOW

#임베딩 #Word2Vec #CBOW

## 분포 가설 (Distributional Hypothesis)

> "비슷한 문맥에서 등장하는 단어는 비슷한 의미를 가진다"

```
... and military officials, [President Trump] said yesterday ...
                              ↑ 주변 단어들로 예측
```

---

## CBOW (Continuous Bag of Words)

**주변 단어(context)로 중심 단어(target)를 예측**하는 모델.

```
문맥 단어들 → [숨겨진 표현] → 중심 단어 예측
w(t-2), w(t-1), w(t+1), w(t+2)  →  w(t)
```

### 학습 과정

```
Step 1. 입력: 주변 단어들의 one-hot 벡터
Step 2. W[V×d] 행렬로 각 단어의 hidden representation 조회
Step 3. 여러 문맥 단어 표현을 평균내어 하나의 벡터로 합산 (Encode)
Step 4. W'[d×V] 행렬로 어휘 전체에 대한 점수 계산 (Decode)
Step 5. Softmax → 확률 분포
Step 6. Cross Entropy Loss + Backprop
```

### 행렬 구조

$$\text{입력: } x \in \mathbb{R}^{|V|} \xrightarrow{W_{|V| \times d}} h \in \mathbb{R}^{d} \xrightarrow{W'_{d \times |V|}} \hat{y} \in \mathbb{R}^{|V|}$$

- $W$: **임베딩 행렬** — 학습 후 단어 벡터로 사용
- $d$: 임베딩 차원 (보통 100~300)

---

## CBOW vs Skip-gram 비교

| | CBOW | [[word2vec_skipgram\|Skip-gram]] |
|--|------|---------|
| **입력 → 출력** | 문맥 → 중심 단어 | 중심 단어 → 문맥 |
| **학습 속도** | 빠름 | 느림 |
| **빈번한 단어** | 정확도 높음 | 비슷 |
| **희소 단어** | 약함 | 잘 표현 |
| **데이터 양** | 많을 때 유리 | 적어도 잘 동작 |

---

## 연결 개념

- [[one_hot]] — 입력으로 사용되는 표현
- [[dim_reduction]] — 유사한 목적의 차원 축소
- [[word2vec_skipgram]] — CBOW의 반대 방향 모델
- [[neg_sampling]] — Softmax 계산 비용 절감 기법
- [[Ch1_ML_Basics/perceptron_nn]] — CBOW의 기반 신경망 구조
