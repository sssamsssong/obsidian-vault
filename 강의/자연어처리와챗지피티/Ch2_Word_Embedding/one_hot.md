# One-hot 인코딩 (One-hot Encoding)

#임베딩 #표현학습

## 개념

단어를 어휘 크기 $|V|$의 벡터로 표현. 해당 단어 위치만 1, 나머지는 0.

$$v \in \{0, 1\}^{|V|}$$

**예시** (어휘 크기 5):

| 단어 | 벡터 |
|------|------|
| cat  | [1, 0, 0, 0, 0] |
| dog  | [0, 1, 0, 0, 0] |
| bird | [0, 0, 1, 0, 0] |

## 한계

### 1. Curse of Dimensionality (차원의 저주)

- 어휘 크기 $|V|$가 수만~수십만에 달함
- 차원이 높아질수록 데이터가 희소해짐 (대부분 0)
- 가능한 부분공간의 조합이 지수적으로 증가

### 2. Orthogonality (직교성)

모든 단어 벡터가 서로 직교 → 의미 유사도를 전혀 표현 못 함.

$$\text{similarity}(\text{cat}, \text{dog}) = \vec{v}_{cat} \cdot \vec{v}_{dog} = 0$$

cat과 dog은 둘 다 동물이지만, 내적이 0 → "전혀 다르다"고 판단.

### 3. 메모리 효율

- 대부분의 값이 0인 희소 행렬 → 메모리 낭비

## 해결책

→ [[dim_reduction]]: PCA, LDA, Autoencoder로 저차원 밀집 벡터로 변환  
→ [[word2vec_cbow]] / [[word2vec_skipgram]]: 문맥 기반 의미 학습
