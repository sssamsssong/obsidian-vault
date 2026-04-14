# Open-domain QA

#독해 #OpenDomainQA #검색

## 개념

특정 지문이 주어지지 않고, **대규모 문서 집합(예: 위키피디아 전체)**에서 답을 찾는 QA.

논문: *Reading Wikipedia to Answer Open-Domain Questions* (DrQA, 2017)

```
질문: "Where was Marie Curie born?"
           ↓
   [Document Retriever]  ← 위키피디아 전체 검색
           ↓
   관련 문서 top-k 선택
           ↓
   [Document Reader]     ← Span 추출 MRC
           ↓
   답: "Warsaw"
```

## 2단계 파이프라인 (DrQA)

### 1. Document Retriever

- TF-IDF 기반 문서 검색
- 질문과 관련된 위키피디아 문서 top-k 반환

### 2. Document Reader

- [[squad_qa]] 방식의 Span 추출 모델
- 검색된 문서들에서 최종 답 구간 선택

## SQuAD QA와의 차이

| | SQuAD QA | Open-domain QA |
|--|----------|----------------|
| **지문 제공** | 항상 제공 | 없음, 직접 검색 |
| **검색 단계** | 불필요 | 필수 |
| **난이도** | 낮음 | 높음 |
| **실용성** | 낮음 | 높음 |

## 연결 개념

- [[squad_qa]] — Document Reader의 기반 기술
- [[attentive_reader]] — Open-domain QA의 전신
- [[Ch7_Transformer]] — BERT 기반 Reader가 현재 SOTA
