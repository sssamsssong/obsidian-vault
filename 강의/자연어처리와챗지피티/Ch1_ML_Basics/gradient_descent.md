# 경사하강법 (Gradient Descent)

#기초 #최적화

## 개념

손실 함수 $L(W)$를 최소화하는 파라미터 $W$를 찾는 방법.
손실 함수의 **기울기(gradient)** 반대 방향으로 $W$를 조금씩 이동.

## 업데이트 규칙

$$W \leftarrow W - \eta \cdot \nabla_W L$$

- $\eta$ (eta): **학습률 (learning rate)** — 얼마나 크게 이동할지
- $\nabla_W L$: 파라미터 $W$에 대한 손실의 편미분

## 알고리즘 흐름

```
1. W를 랜덤 초기화
2. Forward pass: ŷ = f(x; W) 계산
3. Loss 계산: L(ŷ, y)
4. Backward pass (Backprop): ∇L 계산
5. W 업데이트: W ← W - η·∇L
6. L이 수렴할 때까지 2~5 반복
```

## 학습률의 영향

| 학습률 | 문제 |
|--------|------|
| 너무 크면 | Loss가 발산 (overshooting) |
| 너무 작으면 | 수렴이 느림, 지역 최솟값에 갇힐 수 있음 |
| 적절하면 | 안정적으로 전역 최솟값에 수렴 |

## Gradient Descent 변형

| 방법 | 설명 |
|------|------|
| **Batch GD** | 전체 데이터로 한 번에 gradient 계산 |
| **Stochastic GD (SGD)** | 데이터 1개씩 업데이트 → 빠르지만 불안정 |
| **Mini-batch GD** | 소규모 배치로 업데이트 → 실제로 가장 많이 사용 |

## Loss Landscape 직관

```
Loss
 │      *        ← 시작점 (랜덤)
 │       \
 │        \
 │    *    \     ← 업데이트
 │     \    \
 │      *    *   ← 지역 최솟값
 └────────────── W
```

## 연결 개념

- [[loss_functions]] — 최소화할 대상
- [[perceptron_nn]] — Backprop을 통해 gradient 계산
- [[regularization]] — 과적합 방지를 위해 손실에 페널티 추가
