# Positional Encoding (위치 인코딩)

#트랜스포머 #위치인코딩

## 필요성

Transformer는 입력을 **순서와 무관하게 한꺼번에** 처리.  
→ RNN과 달리 순서 정보가 자동으로 반영되지 않음.  
→ 위치 정보를 별도로 주입해야 함.

## 방법

각 단어 임베딩에 위치 벡터를 **더함(add)**.

$$\text{입력} = \text{Word Embedding} + \text{Positional Encoding}$$

- 가장 아래 인코더에서 **1회만** 적용
- 나머지 인코더 블록은 하위 블록의 출력을 그대로 입력으로 사용

## Sinusoidal Positional Encoding (원 논문)

$$PE_{(pos, 2i)} = \sin\left(\frac{pos}{10000^{2i/d_{model}}}\right)$$
$$PE_{(pos, 2i+1)} = \cos\left(\frac{pos}{10000^{2i/d_{model}}}\right)$$

- $pos$: 단어의 위치 (0, 1, 2, ...)
- $i$: 임베딩 차원 인덱스
- $d_{model}$: 임베딩 차원 수 (원 논문: 512)

## 좋은 Positional Encoding의 조건

1. 모든 위치에서 인코딩 벡터의 크기가 동일 → 워드 임베딩 정보 훼손 방지
2. 충분히 구별 가능해야 함 → 다른 위치는 다른 벡터
3. 모델이 상대적 위치 관계를 추론할 수 있어야 함

## 연결 개념

- [[self_attention]] — Positional Encoding이 더해진 벡터가 입력
- [[Ch2_Word_Embedding/word2vec_cbow]] — 더해지는 대상인 단어 임베딩
- [[Ch7_Transformer]] — Transformer 전체 구조에서의 위치
