# 학습 효율화: Negative Sampling & Hierarchical Softmax

#임베딩 #Word2Vec #최적화

## 문제: Softmax 계산 비용

전체 어휘에 대한 Softmax 정규화 인수(normalization factor) 계산이 매우 비쌈.

$$P(w|c) = \frac{\exp(v_w \cdot v_c)}{\sum_{w' \in V} \exp(v_{w'} \cdot v_c)}$$

분모의 $\sum_{w' \in V}$ 계산이 $|V|$번의 연산 필요 → 수십만 단어면 매 스텝 엄청난 비용.

---

## 1. Hierarchical Softmax

단어들을 **이진 트리(binary tree)의 잎(leaf)**에 배치.
확률 계산을 트리 경로 탐색으로 대체.

```
          ROOT
         /    \
        /      \
      [0]      [1]
      / \      / \
   [00] [01] [10] [11]
    cat  dog bird  ...
```

- 계산 복잡도: $O(|V|)$ → $O(\log |V|)$
- 빈도 높은 단어를 트리 상단에 배치하면 더 효율적

---

## 2. Negative Sampling

정답 단어(positive)와 함께 **소수의 오답 단어(negative)** 만 샘플링해 학습.

```
원래: 전체 어휘 |V|개에 대해 손실 계산
개선: 정답 1개 + 랜덤 부정 샘플 k개 (보통 5~20개)만 계산
```

### 손실 함수

$$L = \log \sigma(v_{w_O} \cdot v_{w_I}) + \sum_{k=1}^{K} \mathbb{E}_{w_k \sim P_n(w)} \left[ \log \sigma(-v_{w_k} \cdot v_{w_I}) \right]$$

- 정답 단어: 내적값이 높아지도록 학습
- 부정 샘플: 내적값이 낮아지도록 학습

### 부정 샘플 선택

빈도 기반 확률로 샘플링 (빈도 높은 단어가 더 자주 선택):

$$P(w) \propto f(w)^{3/4}$$

---

## 비교

| 방법 | 복잡도 | 특징 |
|------|--------|------|
| Full Softmax | $O(\|V\|)$ | 정확하지만 느림 |
| Hierarchical Softmax | $O(\log \|V\|)$ | 트리 구조 필요 |
| Negative Sampling | $O(k)$ | 간단, 실제로 잘 동작 |

## 연결 개념

- [[word2vec_cbow]] — 이 기법이 적용되는 모델
- [[word2vec_skipgram]] — 주로 Skip-gram에서 활용
- [[Ch1_ML_Basics/loss_functions]] — Cross Entropy를 대체하는 학습 목표
