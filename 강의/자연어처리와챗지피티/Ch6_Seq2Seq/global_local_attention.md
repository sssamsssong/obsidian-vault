# Global vs Local Attention

#생성 #Attention #Luong

## 논문

*Effective Approaches to Attention-based Neural Machine Translation* (Luong et al., 2015)

---

## Global Attention

디코더의 매 스텝마다 **인코더의 모든 위치**를 참조.

```
디코더 스텝 i
    ↓
모든 인코더 위치 h1, h2, ..., hn 과 score 계산
    ↓
α_i1, α_i2, ..., α_in (전체 합 = 1)
    ↓
c_i = Σ α_ij · hj
```

**장점:** 어떤 위치도 놓치지 않음  
**단점:** 긴 시퀀스에서 계산 비용이 큼

---

## Local Attention

디코더의 매 스텝마다 **일부 인코더 위치만** 참조.

```
1. 집중할 인코더 위치 p_t 예측
2. [p_t - D, p_t + D] 범위(윈도우)만 참조
3. 해당 범위 내에서만 α 계산
```

**장점:** 계산 효율적, 노이즈 감소  
**단점:** 윈도우 밖의 중요한 정보를 놓칠 수 있음

---

## 비교

| | Global Attention | Local Attention |
|--|-----------------|-----------------|
| **참조 범위** | 전체 인코더 | 윈도우 내 일부 |
| **계산 비용** | $O(n)$ | $O(D)$, D ≪ n |
| **긴 문장** | 비효율적 | 효율적 |
| **정보 손실** | 없음 | 윈도우 밖 손실 |

---

## Transformer와의 관계

[[Ch7_Transformer/self_attention|Self-Attention]]은 Global Attention을 **병렬화**한 것.
모든 위치를 동시에 참조하되 행렬 연산으로 효율화.

## 연결 개념

- [[attention]] — Attention의 기본 개념
- [[Ch7_Transformer/self_attention]] — Attention의 완전한 병렬화
- [[Ch7_Transformer/multi_head_attention]] — 여러 관점의 Attention 동시 수행
