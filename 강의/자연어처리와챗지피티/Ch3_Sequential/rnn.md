# RNN (Recurrent Neural Network)

#순차모델 #RNN

## 핵심 아이디어

각 타임스텝의 출력을 **다음 타임스텝의 입력으로 재귀적으로 사용**.
→ 이전 스텝의 정보를 은닉 상태(hidden state)로 전달.

## 수식

$$h_t = \tanh(W_{hh} h_{t-1} + W_{xh} x_t + b_{hh})$$
$$\hat{y}_t = \text{softmax}(W_{hy} h_t)$$

- $x_t \in \mathbb{R}^{|V|}$: $t$ 시점의 입력 (one-hot 또는 임베딩)
- $h_t \in \mathbb{R}^d$: $t$ 시점의 은닉 상태 (이전 정보 누적)
- $W_{hh}, W_{xh}$: **모든 타임스텝에서 공유**되는 파라미터

## Computational Graph

```
x1 → [RNN] → h1 → [RNN] → h2 → [RNN] → h3 → ...
               ↑              ↑
           (h0=0)           (h1)
```

가중치 $W_{hh}, W_{xh}$는 모든 스텝에서 동일하게 재사용.

## 입출력 패턴

| 패턴 | 구조 | 예시 |
|------|------|------|
| **Many-to-One** | 시퀀스 → 하나 | 감성 분류 (마지막 $h_T$ 사용) |
| **One-to-Many** | 하나 → 시퀀스 | 이미지 캡셔닝 |
| **Many-to-Many** | 시퀀스 → 시퀀스 | 기계번역, POS 태깅 |

## RNN의 한계

### Vanishing / Exploding Gradient

역전파(BPTT: Backpropagation Through Time) 시 기울기가:
- **소실**: 길수록 초기 스텝의 기울기 → 0 (장기 의존성 학습 불가)
- **폭발**: 기울기가 지수적으로 증가

→ 해결책: [[lstm]]

## 연결 개념

- [[lstm]] — RNN의 장기 의존성 문제 해결
- [[rnn_applications]] — 실제 태스크 적용
- [[Ch2_Word_Embedding/word2vec_cbow]] — 임베딩이 $x_t$로 입력
- [[Ch6_Seq2Seq/encoder_decoder]] — RNN을 Encoder-Decoder로 확장
