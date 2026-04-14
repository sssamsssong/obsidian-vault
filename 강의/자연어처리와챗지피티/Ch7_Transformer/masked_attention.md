# Masked Multi-Head Attention

#트랜스포머 #Masked-Attention #디코더

## 개념

디코더의 Self-Attention에서 **현재 위치보다 미래 토큰을 참조하지 못하도록 마스킹**.

> 인코더: 입력 전체를 한꺼번에 알고 있음 → 양방향 참조 가능  
> 디코더: 단어를 순차 생성 → 아직 생성 안 된 미래 토큰 참조 불가

---

## 마스킹 방법

Score 행렬에서 미래 위치를 $-\infty$로 설정 → Softmax 후 0이 됨.

```
위치:  1    2    3    4    5
  1 [ 가능  -∞   -∞   -∞   -∞ ]
  2 [ 가능  가능  -∞   -∞   -∞ ]
  3 [ 가능  가능  가능  -∞   -∞ ]
  4 [ 가능  가능  가능  가능  -∞ ]
  5 [ 가능  가능  가능  가능  가능]
```

---

## 디코더의 3개 서브레이어

```
디코더 입력 (생성 중인 타겟 시퀀스)
    ↓
[1. Masked Multi-Head Self-Attention]
    ← 미래 토큰 마스킹 → 자기회귀(autoregressive) 생성
    ↓
Add & Norm
    ↓
[2. Encoder-Decoder Attention (Cross-Attention)]
    ← 인코더 출력 K, V + 디코더 상태 Q
    ↓
Add & Norm
    ↓
[3. Feed-Forward Network]
    ↓
Add & Norm
```

---

## Encoder vs Decoder Attention 비교

| | 인코더 Self-Attention | 디코더 Masked Self-Attention | Cross-Attention |
|--|----------------------|------------------------------|-----------------|
| **Q** | 인코더 토큰 | 디코더 토큰 | 디코더 토큰 |
| **K, V** | 인코더 토큰 | 디코더 토큰 (과거만) | 인코더 출력 |
| **마스킹** | 없음 | 미래 마스킹 | 없음 |

---

## 병렬 학습의 핵심

추론 시에는 순차 생성이지만, **학습 시에는 마스킹 덕분에 병렬 처리 가능**.  
타겟 시퀀스 전체를 한 번에 입력하고, 마스킹으로 미래 정보를 가림.

## 연결 개념

- [[self_attention]] — 마스킹이 없는 기본 Self-Attention
- [[multi_head_attention]] — Masked Attention도 Multi-head로 수행
- [[ffn_residual]] — Masked Attention 이후 적용되는 레이어
- [[Ch6_Seq2Seq/encoder_decoder]] — 디코더 개념의 기원
