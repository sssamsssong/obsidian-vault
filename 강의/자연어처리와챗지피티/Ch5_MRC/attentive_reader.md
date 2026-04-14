# Attentive Reader

#독해 #Attention #Attentive_Reader

## CNN/Daily Mail 데이터셋

DeepMind가 구축한 초기 MRC 데이터셋.

- 뉴스 기사를 지문으로, 요약문의 특정 단어를 빈칸으로 만들어 질문 생성
- 빈칸에 들어갈 개체명(entity)을 맞추는 Cloze-style QA

## DeepMind Attentive Reader (2015)

논문: *Teaching Machines to Read and Comprehend*

```
질문 인코딩: q → [BiRNN] → u (질문 벡터)
지문 인코딩: d1,d2,...,dn → [BiRNN] → h1,h2,...,hn

Attention:
  a_i = softmax(u · h_i)   ← 질문과 각 지문 단어의 관련도
  o = Σ a_i · h_i          ← 가중 합산 (지문 요약 벡터)

출력: o로 답 단어 예측
```

## Stanford Attentive Reader (2016)

논문: *A Thorough Examination of the CNN/DM Reading Comprehension Task*

- DeepMind 모델을 재분석 및 개선
- 더 단순화된 구조로도 유사한 성능 달성
- 데이터셋의 한계(빈칸 방식) 지적

## Attention 메커니즘의 역할

단순 Encoder-Decoder와 달리, **지문의 어느 부분에 집중할지** 동적으로 결정.

```
질문: "Where did the cat sit?"
지문: "The cat sat [on the mat] in the kitchen."
                      ↑ 높은 attention 가중치
```

→ [[Ch6_Seq2Seq/attention]] 에서 더 일반적인 Attention 메커니즘 학습

## 연결 개념

- [[squad_qa]] — 더 발전된 MRC 데이터셋과 Span 추출
- [[Ch6_Seq2Seq/attention]] — Attention의 일반적 정의
- [[Ch3_Sequential/lstm]] — BiRNN 인코더의 기반
