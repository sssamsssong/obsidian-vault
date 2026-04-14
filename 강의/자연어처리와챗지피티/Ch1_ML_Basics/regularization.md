# 정규화 (Regularization)

#기초 #정규화 #과적합

## 과적합 (Overfitting)

모델이 훈련 데이터에는 잘 맞지만 새로운 데이터에는 성능이 떨어지는 현상.

```
훈련 Loss ↓↓  but  검증 Loss ↑↑  →  Overfitting
```

## 1. L2 정규화 (Weight Decay)

손실 함수에 가중치의 크기를 페널티로 추가.

$$L_{reg} = L_{original} + \lambda \sum_{i} W_i^2$$

- $\lambda$: 정규화 강도 (하이퍼파라미터)
- 가중치가 너무 커지지 않도록 억제
- 특징이 많고 각각이 조금씩 기여할 때 효과적

## 2. 드롭아웃 (Dropout)

훈련 중 각 뉴런을 확률 $p$로 랜덤하게 꺼버림.

```
훈련 시: 각 뉴런을 확률 p로 출력 → 0
테스트 시: 모든 뉴런 사용 (dropout 비활성화)
```

**효과:**
- 뉴런 간 공동 적응(co-adaptation) 방지
- 더 강건한(robust) 특징 학습 유도
- 일종의 앙상블(ensemble) 효과

## 3. Early Stopping

검증 손실이 더 이상 줄지 않을 때 학습 조기 종료.

```
Validation Loss
     │
     │\
     │ \      ← 이 지점에서 멈춤 (best model)
     │  \___/‾‾‾  ← 다시 올라가면 과적합 시작
     └────────── Epochs
```

## 비교

| 방법 | 핵심 아이디어 | 언제 쓸까 |
|------|---------------|-----------|
| L2 | 가중치 크기 제한 | 특징이 많을 때 |
| Dropout | 뉴런 랜덤 비활성화 | 깊은 네트워크 |
| Early Stopping | 최적 시점에 중단 | 항상 사용 권장 |

## 연결 개념

- [[gradient_descent]] — 정규화 항이 추가된 손실을 최소화
- [[perceptron_nn]] — 정규화가 적용되는 파라미터 $W$
