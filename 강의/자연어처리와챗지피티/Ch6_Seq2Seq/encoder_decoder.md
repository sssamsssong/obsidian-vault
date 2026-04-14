# Encoder-Decoder 모델

#생성 #Seq2Seq #EncoderDecoder

## 구조

가변 길이 입력 시퀀스 → 고정 벡터 → 가변 길이 출력 시퀀스

```
입력: x1, x2, ..., xn
         ↓
    [Encoder RNN]
         ↓
    context vector c = F(h1, ..., hn)   ← 보통 마지막 은닉 상태 h_n
         ↓
    [Decoder RNN]
         ↓
출력: y1, y2, ..., ym
```

## Encoder

- 입력 시퀀스를 읽어 **고정 크기 문맥 벡터(context vector)** $c$로 압축
- 보통 마지막 은닉 상태 사용: $c = h_n$

$$h_t = \text{RNN}(x_t, h_{t-1})$$

## Decoder

- 문맥 벡터 $c$와 이전 출력을 입력으로 다음 단어 생성
- `<EOS>` 토큰이 나올 때까지 생성 반복

$$s_t = \text{RNN}(y_{t-1}, s_{t-1}, c)$$
$$P(y_t | y_{<t}, x) = \text{softmax}(W s_t)$$

## 학습 (Training)

Teacher Forcing: 실제 정답 시퀀스를 디코더 입력으로 사용

$$\text{minimize} - \frac{1}{T} \sum_{t=1}^{T} \log P(y_t | y_{<t}, x)$$

## 추론 (Inference)

이전 시점의 **예측값**을 다음 입력으로 사용 (Teacher Forcing 없음)

→ 디코딩 전략: [[beam_search]]

## 한계: 고정 벡터 병목

- 아무리 긴 입력도 하나의 벡터 $c$로 압축
- 긴 문장일수록 정보 손실
- 인코더 초반부 정보가 희미해짐

→ 해결책: [[attention]]

## 연결 개념

- [[beam_search]] — 디코딩 전략
- [[attention]] — 고정 벡터 한계 극복
- [[Ch3_Sequential/lstm]] — Encoder/Decoder의 기본 구성 요소
