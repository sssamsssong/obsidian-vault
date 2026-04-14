# Multi-Head Attention (멀티헤드 어텐션)

#트랜스포머 #Multi-head-Attention

## 개념

하나의 [[self_attention]] 대신 **여러 개의 Attention을 병렬 수행**.
같은 토큰에 대해 **다양한 관점(head)**에서 표현.

```
"The animal didn't cross the street because it was too tired."

Head 1: "it" → "animal" (지시 대상)
Head 2: "it" → "tired"  (상태 관계)
Head 3: "it" → "street" (장소 관계)
```

---

## 계산 과정

### 1. 각 헤드별 독립적 Attention 계산

$$\text{head}_i = \text{Attention}(Q W_i^Q,\ K W_i^K,\ V W_i^V)$$

- 원 논문: 헤드 수 $h = 8$, $d_k = d_v = d_{model}/h = 64$
- 각 헤드는 별도의 $W^Q_i, W^K_i, W^V_i$ 파라미터 보유

### 2. Concatenation

$$\text{MultiHead}(Q, K, V) = \text{Concat}(\text{head}_1, \ldots, \text{head}_h) W^O$$

- 8개 헤드 결과를 이어 붙임: $8 \times 64 = 512$차원
- $W^O \in \mathbb{R}^{hd_v \times d_{model}}$로 원래 차원으로 변환

---

## 구조 비교

```
Single-head Attention:
  x → [Attention] → z

Multi-head Attention (h=8):
  x → [Head1] ─┐
  x → [Head2] ─┤
  x → [Head3] ─┤→ Concat → W^O → z
  ...           │
  x → [Head8] ─┘
```

---

## 왜 여러 헤드가 필요한가?

단일 Attention은 하나의 관점만 포착 가능.  
Multi-head는 **문법적, 의미적, 지시적** 등 여러 관계를 동시에 학습.

---

## 연결 개념

- [[self_attention]] — 각 헤드 내부의 기본 계산
- [[ffn_residual]] — Multi-head Attention 이후 적용되는 레이어
- [[masked_attention]] — 디코더에서의 변형된 Multi-head Attention
