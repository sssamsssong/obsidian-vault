# GLUE 벤치마크

#분류 #GLUE #벤치마크 #NLU

## GLUE란?

**General Language Understanding Evaluation**  
2018년 NYU, University of Washington, DeepMind 협업으로 공개.  
자연어 이해(NLU) 모델의 표준 성능 비교 벤치마크.

## GLUE Tasks 분류

### Single-Sentence Tasks

| 태스크 | 전체 이름 | 내용 |
|--------|-----------|------|
| **CoLA** | Corpus of Linguistic Acceptability | 문법적으로 올바른 문장인지 분류 |
| **SST-2** | Stanford Sentiment Treebank | 영화 리뷰 긍/부정 분류 |

### Similarity & Paraphrase Tasks

| 태스크 | 내용 |
|--------|------|
| **MRPC** | 두 문장이 같은 의미(paraphrase)인지 분류 |
| **QQP** | Quora 질문 쌍의 의미 동일 여부 |
| **STS-B** | 두 문장의 의미적 유사도 점수 (회귀) |

### Inference Tasks

| 태스크 | 내용 |
|--------|------|
| **MNLI** | 전제-가설 논리 관계 (entailment/neutral/contradiction) |
| **QNLI** | 질문-문단에서 답 포함 여부 (SQuAD 변형) |
| **RTE** | Textual Entailment 이진 분류 |

## SuperGLUE

GLUE 공개 후 1년 내에 인간 수준을 넘어서자 더 어려운 버전 출시.

## KLUE

한국어 언어모델 평가 벤치마크. 총 8개 태스크 구성.  
비영어권 국가들의 모국어 NLP 연구 역량 결집 목적.

## 연결 개념

- [[sentiment_analysis]] — SST-2가 대표 감성 분류 태스크
- [[transfer_learning]] — GLUE로 전이학습 모델 평가
- [[Ch7_Transformer]] — Transformer 계열이 GLUE 리더보드 석권
