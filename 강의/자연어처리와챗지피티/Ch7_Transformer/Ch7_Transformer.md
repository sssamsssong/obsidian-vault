# Ch 7. Transformer — 개요

#트랜스포머 #Attention #Transformer

## 발전 계보

```
RNN → LSTM → LSTM + Attention → Transformer (Only Attention)
```

RNN을 완전히 제거하고 **Attention만으로** 언어 모델 구성.

## 핵심 개념 목록

- [[positional_encoding]] — 위치 정보 주입 방법
- [[self_attention]] — Self-Attention 단계별 계산
- [[multi_head_attention]] — 여러 관점의 병렬 Attention
- [[ffn_residual]] — FFN, Residual Connection, Layer Norm
- [[masked_attention]] — 디코더의 Masked Multi-head Attention

## Transformer의 특징

| 특징 | 설명 |
|------|------|
| **병렬 처리** | RNN과 달리 순차 처리 불필요 → 학습 빠름 |
| **장거리 의존성** | 거리에 무관하게 모든 위치 직접 참조 |
| **멀티모달** | 언어 외 이미지(ViT) 등 다양한 데이터 처리 가능 |

## 전체 구조

```
입력 텍스트
    ↓
[입력 임베딩 + Positional Encoding]
    ↓
[Encoder Block × 6]
  - Self-Attention
  - FFN
  - Residual + LayerNorm
    ↓
[Decoder Block × 6]
  - Masked Self-Attention
  - Encoder-Decoder Attention
  - FFN
  - Residual + LayerNorm
    ↓
[Linear + Softmax]
    ↓
출력 텍스트
```

## Complexity 비교

| 모델 | Sequential 연산 | 장거리 경로 길이 |
|------|----------------|----------------|
| RNN | $O(n)$ | $O(n)$ |
| Transformer | $O(1)$ | $O(1)$ |

## 연결 개념

- 이전 챕터: [[Ch6_Seq2Seq/attention]] — Transformer Attention의 직접 전신
- [[Ch4_Text_Classification/glue_tasks]] — BERT(Transformer)가 GLUE 평가 기준 석권
