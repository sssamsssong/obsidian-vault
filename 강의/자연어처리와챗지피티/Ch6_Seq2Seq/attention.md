# Attention 메커니즘

#생성 #Attention #핵심개념

## 등장 배경

[[encoder_decoder]]의 고정 벡터 병목 문제 해결.
디코더가 매 스텝마다 **인코더의 모든 은닉 상태를 참조**할 수 있게 함.

> RNN → LSTM → **LSTM + Attention** → Transformer

---

## 핵심 아이디어

"디코더의 현재 상태와 가장 관련 있는 인코더 위치에 더 집중하라"

```
디코더 스텝 i에서:
  1. 인코더 각 위치 j와의 관련도(score) 계산
  2. Softmax로 정규화 → attention weight α_ij
  3. 인코더 상태들의 가중 합산 → context vector c_i
  4. c_i를 디코더 출력 계산에 활용
```

---

## 수식

### Score 함수 $s(i, j)$

디코더 스텝 $i$와 인코더 스텝 $j$의 관련도.

| 방법 | 수식 |
|------|------|
| **Dot product** | $s_i(j) = h_j^T s_i$ |
| **General** | $s_i(j) = h_j^T W s_i$ |
| **Concat (Bahdanau)** | $s_i(j) = v^T \tanh(W[h_j; s_i])$ |

### Attention Weight

$$\alpha_{ij} = \frac{\exp(s(i,j))}{\sum_{k} \exp(s(i,k))}$$

### Context Vector

$$c_i = \sum_j \alpha_{ij} h_j$$

---

## 시각적 직관

```
디코더: "chat" 생성 중

인코더 위치:  "the"  "black"  "cat"  "drank"  "milk"
Attention:    0.05    0.10    0.80    0.03     0.02
                                ↑ 높은 집중도
```

"cat"을 번역하는 순간 인코더의 "cat" 위치에 집중.

---

## 장점

- 긴 시퀀스에서도 정보 손실 없음
- 어떤 소스 위치에 집중했는지 해석 가능 (interpretability)
- 번역 정렬 시각화 가능

## 연결 개념

- [[global_local_attention]] — Attention 유형 비교
- [[encoder_decoder]] — Attention이 해결하는 문제
- [[Ch7_Transformer/self_attention]] — Attention만으로 구성된 모델
- [[Ch5_MRC/attentive_reader]] — MRC에서의 Attention 활용
