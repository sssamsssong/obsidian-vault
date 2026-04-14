# 퍼셉트론 & 신경망 (Perceptron / Neural Networks)

#기초 #신경망 #퍼셉트론

## 생물학적 뉴런 → 인공 뉴런

| 생물학 | 인공 신경망 |
|--------|------------|
| 수상돌기 (Dendrite) | 입력 + 가중치 (Weights) |
| 축삭돌기 (Axon) | 활성화 함수 (Activation Function) |
| 시냅스 강도 | 가중치 $W$ |

## 퍼셉트론 (Perceptron)

신경망의 기본 단위. 단일 뉴런 모델.

$$\hat{y} = g\left( W^T x + b \right)$$

- $x \in \mathbb{R}^m$: 입력 벡터
- $W$: 가중치, $b$: 편향 (bias)
- $g$: 활성화 함수 (비선형성 도입)

## 활성화 함수 (Activation Functions)

> 비선형성이 없으면 아무리 층을 쌓아도 선형 변환과 동일.

| 함수 | 수식 | 특징 |
|------|------|------|
| **Sigmoid** | $\sigma(x) = \frac{1}{1+e^{-x}}$ | 출력 범위 (0,1), 이진 분류 |
| **Tanh** | $\tanh(x)$ | 출력 범위 (-1,1) |
| **ReLU** | $\max(0, x)$ | 가장 많이 사용, 기울기 소실 완화 |
| **Softmax** | $\frac{e^{x_i}}{\sum_j e^{x_j}}$ | 다중 클래스 확률 출력 |

## Multi-Output Perceptron (Dense Layer)

모든 입력이 모든 출력에 연결 → **Fully Connected / Dense Layer**

$$Z = X^T W^{(1)}, \quad \hat{Y} = \text{softmax}(g(Z)^T W^{(2)})$$

## 다층 신경망 (Multi-Layer NN)

```
입력층 → [은닉층1] → [은닉층2] → ... → 출력층
  x          h1           h2              ŷ
```

- **학습 파라미터**: $W = \{W_1, b_1, W_2, b_2, \ldots\}$
- **하이퍼파라미터**: 층 수, 각 층의 뉴런 수 (사전 정의)

## Forward & Backward Propagation

```
Forward:  x → h1 → h2 → ŷ → L
                              ↓
Backward: ∂L/∂W2 ← ∂L/∂h2 ← ∂L/∂h1 ← ∂L/∂W1
```

- **Backpropagation**: 연쇄법칙(chain rule)으로 각 레이어의 gradient 계산
- 계산된 gradient로 [[gradient_descent]] 수행

## 연결 개념

- [[loss_functions]] — 학습 목표
- [[gradient_descent]] — 파라미터 업데이트 방법
- [[regularization]] — 과적합 방지
- [[Ch3_Sequential/rnn]] — Dense Layer의 순차 데이터 한계 극복
