# Beam Search

#생성 #디코딩 #BeamSearch

## Greedy Decoding의 문제

매 스텝 가장 확률 높은 단어만 선택 → 전체적으로 최적이 아닐 수 있음.

```
스텝1: "le" (0.6) ← 선택
스텝2: "chat" | "le" (0.3)

vs

스텝1: "the" (0.4)
스텝2: "cat" | "the" (0.7) ← 전체 확률은 더 높음 (0.28)
```

## Beam Search

상위 $k$개(beam width) 후보를 동시에 유지하며 탐색.

```
beam_size = 2 예시:

t=1: ["le"(0.6), "the"(0.4)]
         ↓
t=2: "le" 에서: ["le chat"(0.6×0.5), "le noir"(0.6×0.3)]
     "the" 에서: ["the cat"(0.4×0.7), "the black"(0.4×0.4)]
     → 상위 2개 유지: ["the cat"(0.28), "le chat"(0.30)]
         ↓
... EOS까지 반복
```

### 알고리즘

1. 초기: 빈 시퀀스 1개로 시작
2. 매 스텝: 각 후보에서 가능한 다음 단어 확장
3. 전체 후보 중 누적 확률 상위 $k$개만 유지
4. 모든 후보가 `<EOS>`에 도달하면 종료

## Beam Size 영향

| Beam Size | 특징 |
|-----------|------|
| **1** | Greedy Search와 동일 |
| **작을수록** | 빠르지만 품질 낮음 |
| **클수록** | 느리지만 품질 높음 |
| **∞** | 완전 탐색 (비현실적) |

실제로는 beam size = 4~10 사용.

## 연결 개념

- [[encoder_decoder]] — Beam Search가 적용되는 디코딩 단계
- [[attention]] — Attention이 있으면 Beam Search 품질 크게 향상
