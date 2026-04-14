# 감성 분석 (Sentiment Analysis)

#분류 #감성분석

## 정의

텍스트에 담긴 **의견, 감성, 평가, 태도** 등 주관적 정보를 분석하는 태스크.
텍스트 분류의 대표적 예시.

**활용:** 고객 피드백 분석, 콜센터 메시지, 제품 리뷰, SNS 여론 분석

## 접근 방법

### 1. Lexicon-based Approach

- 감성 사전(sentiment lexicon)을 구축
- 키워드 등장 여부로 긍/부정 판단
- 규칙 기반 → 새로운 표현에 취약

### 2. Machine Learning Approach

- 텍스트를 벡터로 표현 후 분류기 학습
- RNN/LSTM → 시퀀스 전체 문맥 파악
- Transformer → SOTA 성능

## 감성(Sentiment) vs 감정(Emotion)

| 구분 | 감성 (Sentiment) | 감정 (Emotion) |
|------|-----------------|----------------|
| 정의 | 주관적 태도, 평가 | 내면의 심리 상태 |
| 예시 | 긍정/부정/중립 | 기쁨/슬픔/분노/놀람 |
| 클래스 수 | 보통 2~5개 | 보통 6~8개 |

## RNN 기반 감성 분석

```
"This movie is fun..." 
  → [임베딩] → [RNN/LSTM] → h_T → [Dense] → softmax → {positive, negative}
```

## 연결 개념

- [[glue_tasks]] — SST-2가 대표적인 감성 분류 벤치마크
- [[eval_metrics]] — 성능 평가 방법
- [[Ch3_Sequential/rnn_applications]] — RNN 기반 감성 분류 구조
- [[Ch7_Transformer]] — BERT 기반 감성 분류 (SOTA)
