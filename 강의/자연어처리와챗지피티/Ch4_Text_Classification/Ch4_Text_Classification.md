# Ch 4. Text Classification — 개요

#분류 #NLP #감성분석

## 이 챕터의 흐름

```
텍스트 분류 (Text Classification)
    ├── [[sentiment_analysis]] — 감성 분석
    ├── [[glue_tasks]] — 벤치마크 (GLUE, SuperGLUE, KLUE)
    ├── [[transfer_learning]] — 전이학습, Cross-lingual
    └── [[eval_metrics]] — Precision, Recall, F1
```

## 텍스트 분류란?

텍스트 입력 → 미리 정의된 클래스(범주) 중 하나로 분류.

**예시:**
- 스팸 메일 분류: {스팸, 정상}
- 감성 분류: {긍정, 부정}
- 뉴스 카테고리 분류: {정치, 경제, 스포츠, ...}

## 핵심 개념 목록

- [[sentiment_analysis]] — 감성 분석 정의, Lexicon vs ML 기반
- [[glue_tasks]] — GLUE / SuperGLUE / KLUE 벤치마크
- [[transfer_learning]] — Cross-lingual, Multi-task Learning
- [[eval_metrics]] — 분류 성능 평가 지표

## 연결 개념

- 이전 챕터: [[Ch3_Sequential/rnn]] — 분류기의 인코더로 사용
- 다음 챕터: [[Ch5_MRC]] — 단순 분류를 넘어 지문 이해 + QA
- [[Ch7_Transformer]] — BERT 등 Transformer 기반 분류가 GLUE 석권
