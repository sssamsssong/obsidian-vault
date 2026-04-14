# SQuAD & Span 추출 QA

#독해 #SQuAD #QA

## SQuAD (Stanford Question Answering Dataset)

논문: *SQuAD: 100,000+ Questions for Machine Comprehension of Text* (2016)

- 위키피디아 지문 + 크라우드소싱 질문 100K+
- 답이 지문 내 **연속된 구간(span)** 으로 존재
- 빈칸 채우기 방식보다 훨씬 자연스러운 QA

## Cloze-style vs Span Extraction 비교

| 방식 | 설명 | 한계 |
|------|------|------|
| **Cloze-style** (CNN/DM) | 빈칸에 단어 채우기 | 인공적, 어휘 제한 |
| **Span Extraction** (SQuAD) | 지문에서 답 구간 선택 | 자연스러운 QA |

## Span 추출 방법

```
지문 토큰: t1, t2, ..., tn
질문:      q1, q2, ..., qm

모델이 예측하는 것:
  - 답의 시작 위치: start_idx
  - 답의 끝 위치:   end_idx

답 = 지문[start_idx : end_idx]
```

## SQuAD용 Attentive Reader

1. 질문과 지문을 함께 인코딩
2. 각 지문 위치에 대해 **시작/끝 확률** 계산
3. 가장 높은 확률의 구간 선택

## Common Failure Cases

- **Paraphrase**: 질문과 다른 표현으로 답이 쓰인 경우
- **Multi-sentence reasoning**: 여러 문장을 연결해야 하는 경우
- **Negation**: "X는 Y가 아니다" 형태

## 연결 개념

- [[attentive_reader]] — SQuAD 이전 MRC 모델
- [[open_domain_qa]] — SQuAD를 오픈 도메인으로 확장
- [[Ch6_Seq2Seq/attention]] — Span 추출에서 Attention 활용
