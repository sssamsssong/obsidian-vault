# FFN & Residual Connection & Layer Norm

#트랜스포머 #FFN #Residual #LayerNorm

## 인코더 블록 구조

각 인코더 블록은 두 개의 서브레이어로 구성.

```
입력 x
  ↓
[Multi-Head Self-Attention]
  ↓
Add & Norm  (Residual + LayerNorm)
  ↓
[Feed-Forward Network]
  ↓
Add & Norm  (Residual + LayerNorm)
  ↓
출력
```

---

## Position-wise Feed-Forward Network (FFN)

각 위치(토큰)에 대해 **독립적으로** 적용되는 2층 완전 연결 네트워크.

$$\text{FFN}(x) = \max(0,\ x W_1 + b_1) W_2 + b_2$$

- 원 논문 차원: $d_{model} = 512 \rightarrow d_{ff} = 2048 \rightarrow d_{model} = 512$
- 활성화 함수: ReLU
- Self-Attention이 위치 간 관계를 학습한다면, FFN은 **각 위치의 표현을 깊게 변환**

---

## Residual Connection (잔차 연결)

서브레이어의 입력을 출력에 **더함(Add)**.

$$\text{Output} = \text{LayerNorm}(x + \text{Sublayer}(x))$$

**효과:**
- 기울기 소실 방지 → 깊은 네트워크 학습 가능
- 원본 정보가 직접 전달 → 학습 안정화

---

## Layer Normalization

배치가 아닌 **레이어(feature) 차원**에서 정규화.

$$\text{LayerNorm}(x) = \gamma \cdot \frac{x - \mu}{\sigma} + \beta$$

- $\mu, \sigma$: 해당 레이어의 평균/분산
- $\gamma, \beta$: 학습 가능한 스케일/시프트 파라미터

---

## 디코더에서도 동일 적용

디코더 블록도 3개의 서브레이어 각각에 Residual + LayerNorm 적용.

## 연결 개념

- [[multi_head_attention]] — FFN 이전 단계
- [[masked_attention]] — 디코더 블록의 첫 번째 서브레이어
- [[Ch1_ML_Basics/regularization]] — 정규화의 다른 형태 (Dropout과 함께 사용)
