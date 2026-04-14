# LSTM (Long Short-Term Memory)

#순차모델 #LSTM #장기의존성

## 등장 배경

[[rnn]]의 **Vanishing Gradient** 문제 해결을 위해 설계.
장기(long-term) 의존성을 효과적으로 학습.

> RNN → LSTM → LSTM + Attention → Transformer
> (점진적 발전 흐름)

## 핵심 구조: 셀 상태 (Cell State)

RNN에 없는 **셀 상태 $C_t$** 를 추가. 장기 기억을 별도의 경로로 유지.

```
     C_{t-1} ──────────────────────────────► C_t
                  │           │
              [forget]     [input]
                  │           │
h_{t-1}, x_t ─────────────────────────────► h_t
                              │
                           [output]
```

## 4개의 게이트

### 1. Forget Gate (망각 게이트)

$$f_t = \sigma(W_f [h_{t-1}, x_t] + b_f)$$

이전 셀 상태에서 **얼마나 잊을지** 결정. 0이면 완전히 삭제, 1이면 완전히 유지.

### 2. Input Gate (입력 게이트)

$$i_t = \sigma(W_i [h_{t-1}, x_t] + b_i)$$
$$\tilde{C}_t = \tanh(W_C [h_{t-1}, x_t] + b_C)$$

**새 정보를 얼마나 추가할지** 결정.

### 3. Cell State 업데이트

$$C_t = f_t \odot C_{t-1} + i_t \odot \tilde{C}_t$$

- 이전 셀 상태의 일부를 잊고 + 새 정보를 추가

### 4. Output Gate (출력 게이트)

$$o_t = \sigma(W_o [h_{t-1}, x_t] + b_o)$$
$$h_t = o_t \odot \tanh(C_t)$$

셀 상태에서 **얼마를 출력할지** 결정.

## RNN vs LSTM 비교

| | RNN | LSTM |
|--|-----|------|
| **장기 의존성** | 취약 | 강함 |
| **파라미터 수** | 적음 | 많음 (게이트 4개) |
| **학습 속도** | 빠름 | 느림 |
| **메모리** | $h_t$ 하나 | $h_t$ + $C_t$ |

## 연결 개념

- [[rnn]] — LSTM이 해결하는 문제의 근원
- [[Ch6_Seq2Seq/attention]] — LSTM + Attention으로 더 발전
- [[Ch7_Transformer]] — Attention만으로 LSTM을 대체
