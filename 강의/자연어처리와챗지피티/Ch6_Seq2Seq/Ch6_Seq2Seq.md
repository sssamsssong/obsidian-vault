# Ch 6. Sequence-to-Sequence & 기계번역 — 개요

#생성 #Seq2Seq #기계번역 #Attention

## 이 챕터의 흐름

```
RNN 언어모델 (Many-to-Many)
        ↓
Encoder-Decoder 구조 (가변 길이 입출력)
        ↓
Greedy Decoding vs Beam Search
        ↓
Attention (고정 벡터 병목 해결)
        ↓
Global vs Local Attention
```

## 핵심 개념 목록

- [[encoder_decoder]] — Seq2Seq 기본 구조, 학습/추론
- [[beam_search]] — Greedy vs Beam Search 디코딩
- [[attention]] — Attention 메커니즘, 핵심 아이디어
- [[global_local_attention]] — Attention 유형 비교

## 기계번역의 도전

```
영어: "the black cat drank milk"     (5 words)
프랑스어: "le chat noir a bu du lait" (7 words)
```

- 입출력 길이가 다름 → 고정 크기 출력 불가
- 단어 순서가 다를 수 있음
- 문법 구조가 언어마다 상이

## 발전 흐름

```
RNN LM → Encoder-Decoder → + Attention → Transformer (Ch7)
```

Attention이 이 챕터의 핵심이며 [[Ch7_Transformer]]의 직접적 기반.

## 연결 개념

- 이전 챕터: [[Ch3_Sequential/lstm]] — Encoder-Decoder의 기본 구성 요소
- 다음 챕터: [[Ch7_Transformer]] — Attention만으로 RNN 대체
