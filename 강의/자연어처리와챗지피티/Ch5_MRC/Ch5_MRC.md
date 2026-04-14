# Ch 5. Machine Reading Comprehension — 개요

#독해 #QA #MRC

## 이 챕터의 흐름

```
지문(passage) + 질문(question) → 답(answer) 추출
        ↓
  CNN/Daily Mail Dataset (초기 데이터셋)
        ↓
  Attentive Reader 모델 (Stanford / DeepMind)
        ↓
  SQuAD → Open-domain QA
```

## 핵심 개념 목록

- [[attentive_reader]] — Stanford/DeepMind Attentive Reader 구조
- [[squad_qa]] — SQuAD 데이터셋, Span 추출 방식
- [[open_domain_qa]] — 위키피디아 전체에서 답 찾기

## Machine Reading이란?

컴퓨터가 지문을 읽고 질문에 답하는 능력.

```
지문: "The cat sat on the mat in the kitchen."
질문: "Where did the cat sit?"
답:   "on the mat"
```

## 발전 흐름

| 연도 | 시스템 | 특징 |
|------|--------|------|
| 2015 | DeepMind Attentive Reader | CNN/DM 데이터셋, 빈칸 채우기 |
| 2016 | Stanford Attentive Reader | CNN/DM 재분석 |
| 2016 | SQuAD 1.0 | 100K+ 질문, Span 추출 |
| 2019 | SOTA | SQuAD에서 인간 수준(95.1) 돌파 |

## 연결 개념

- 이전 챕터: [[Ch4_Text_Classification]] — 분류 → 이해로 확장
- [[Ch6_Seq2Seq/attention]] — Attentive Reader의 핵심 메커니즘
- [[Ch7_Transformer/self_attention]] — BERT 기반 MRC가 SOTA
