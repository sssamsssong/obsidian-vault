# Self-Attention (셀프 어텐션)

#트랜스포머 #Self-Attention #핵심개념

## 개념

같은 시퀀스 내의 모든 토큰이 **서로를 참조**하며 표현을 업데이트.
"이 단어를 표현할 때 시퀀스의 어떤 단어에 집중할까?"

**예시:**
```
"The animal didn't cross the street because it was too tired."
                                              ↑
                                    "it"이 "animal"을 참조
```

---

## 6단계 계산 과정

### Step 1: Q, K, V 벡터 생성

각 입력 벡터 $x_i$에 세 가지 행렬을 곱해 Query, Key, Value 생성.

$$Q_i = x_i W^Q, \quad K_i = x_i W^K, \quad V_i = x_i W^V$$

- $W^Q, W^K, W^V \in \mathbb{R}^{d_{model} \times d_k}$  (원 논문: $d_k = 64$)
- Q: "나는 무엇을 찾고 있나?"
- K: "나는 어떤 정보를 제공할 수 있나?"
- V: "내가 실제로 제공하는 정보"

### Step 2: Score 계산

Query와 모든 Key의 내적으로 관련도 계산.

$$\text{score}(i, j) = Q_i \cdot K_j$$

### Step 3: Scaling

기울기 소실 방지를 위해 $\sqrt{d_k}$로 나눔.

$$\text{scaled score} = \frac{Q_i \cdot K_j}{\sqrt{d_k}}$$

### Step 4: Softmax

점수를 확률 분포로 변환.

$$\alpha_{ij} = \text{softmax}\left(\frac{Q_i \cdot K_j}{\sqrt{d_k}}\right)$$

### Step 5 & 6: 가중 합산

$$z_i = \sum_j \alpha_{ij} V_j$$

---

## 행렬 연산 (병렬화)

모든 토큰을 동시에 처리.

$$\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right) V$$

---

## 연결 개념

- [[multi_head_attention]] — Self-Attention을 여러 개 병렬로 실행
- [[masked_attention]] — 디코더에서 미래 토큰을 차단한 Self-Attention
- [[positional_encoding]] — Self-Attention의 입력에 위치 정보 추가
- [[Ch6_Seq2Seq/attention]] — Seq2Seq Attention과의 차이 (self vs cross)
