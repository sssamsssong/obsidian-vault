# RNN 응용 태스크

#순차모델 #응용 #NLP

## Many-to-One: 감성 분석 (Sentiment Analysis)

입력 시퀀스 → 하나의 클래스 레이블

```
"This movie is fun..." → [RNN] → h_T → softmax → {positive, negative}
```

- 마지막 은닉 상태 $h_T$가 전체 시퀀스를 요약
- → [[Ch4_Text_Classification/sentiment_analysis]] 에서 상세 학습

## Many-to-Many (동기): POS 태깅

각 입력 단어에 대해 즉시 출력 (입출력 길이 동일)

```
"The  cat  sat  on  the  mat"
  ↓    ↓    ↓    ↓    ↓    ↓
  DT   NN   VBD  IN   DT   NN
```

$$\hat{y}_t = \text{softmax}(W_{hy} h_t), \quad \text{class} \in \{NN, VB, \ldots, JJ\}$$

## Many-to-Many (비동기): 기계 번역

입력 시퀀스를 모두 읽은 후 출력 시퀀스 생성 → **Encoder-Decoder** 구조 필요

```
"고양이가 앉았다" → [Encoder RNN] → context → [Decoder RNN] → "The cat sat"
```

→ [[Ch6_Seq2Seq/encoder_decoder]] 에서 상세 학습

## One-to-Many: 이미지 캡셔닝

하나의 입력(이미지)에서 시퀀스(설명) 생성

```
[이미지] → CNN (인코딩) → [RNN 디코더] → "A dog is running on the grass"
```

- CNN으로 이미지를 고정 벡터로 인코딩
- RNN으로 단어를 순차 생성

## 연결 개념

- [[rnn]] — 이 모든 태스크의 기반 구조
- [[lstm]] — 장거리 의존성이 필요한 태스크에서 RNN 대체
- [[Ch4_Text_Classification/sentiment_analysis]] — 감성 분석 상세
- [[Ch6_Seq2Seq/encoder_decoder]] — 기계번역 상세
