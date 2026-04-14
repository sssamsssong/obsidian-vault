# Ch 3. Sequential Modeling — 개요

#순차모델 #RNN #LSTM

## 이 챕터의 흐름

```
Vanilla NN의 한계 (고정 입력 크기, 순서 무시)
            ↓
    RNN (순차 의존성 모델링)
            ↓
    LSTM (장기 의존성 해결)
```

## 핵심 개념 목록

- [[rnn]] — Recurrent Neural Network 구조와 수식
- [[lstm]] — Long Short-Term Memory, Vanishing Gradient 해결
- [[rnn_applications]] — 감성 분석, POS 태깅, 이미지 캡셔닝, 기계번역

## Vanilla NN의 한계

| 문제 | 설명 |
|------|------|
| 고정 입력 크기 | 가변 길이 시퀀스 처리 불가 |
| 순서 정보 무시 | 단순 평균은 순서를 버림 |
| 시간적 의존성 없음 | 이전 스텝의 정보를 활용 못 함 |

## 시퀀스 데이터의 예

- 영화 리뷰 → 감성 분석
- 영어 문장 → 한국어 번역
- 이미지 → 설명 문장
- 단어 → 품사 태깅

## 연결 개념

- 이전 챕터: [[Ch2_Word_Embedding]] — 시퀀스의 각 토큰을 임베딩으로 표현
- 다음 챕터: [[Ch4_Text_Classification]] — RNN을 분류에 활용
- 연결 챕터: [[Ch6_Seq2Seq]] — RNN을 Encoder-Decoder로 확장
