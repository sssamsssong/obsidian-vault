# 손실 함수 (Loss Functions)

#기초 #손실함수

## 개념

모델의 예측 $\hat{y}$와 실제 정답 $y$ 사이의 차이를 수치화한 것.
학습의 목표는 이 손실을 최소화하는 파라미터 $W^*$를 찾는 것.

$$W^* = \underset{W}{\arg\min}\ L(f(x; W),\ y)$$

## Empirical Loss (경험적 손실)

전체 데이터셋에 대한 평균 손실.

$$L = \frac{1}{N} \sum_{i} L(\hat{y}_i, y_i)$$

---

## MSE (Mean Squared Error)

**사용 시점:** 연속값 예측 (회귀)

$$L_{MSE} = \frac{1}{N} \sum_{i}(\hat{y}_i - y_i)^2$$

- 예측값과 실제값의 차이를 제곱해 평균
- 이상치(outlier)에 민감

---

## Binary Cross Entropy

**사용 시점:** 이진 분류 (출력이 0~1 확률)

$$L = -\frac{1}{N} \sum_{i} \left[ y_i \log \hat{y}_i + (1 - y_i) \log(1 - \hat{y}_i) \right]$$

- 출력층에 **Sigmoid** 활성화 함수와 함께 사용
- $\hat{y} \geq 0.5$ → 1로 분류, $\hat{y} < 0.5$ → 0으로 분류

---

## (General) Cross Entropy

**사용 시점:** 다중 클래스 분류

$$L = -\frac{1}{N} \sum_{i} \sum_{c} y_{ic} \log \hat{y}_{ic}$$

- 출력층에 **Softmax** 활성화 함수와 함께 사용
- 클래스 수가 3개 이상일 때 사용

---

## 비교 요약

| 손실 함수 | 태스크 | 출력 활성화 |
|-----------|--------|-------------|
| MSE | 회귀 | 없음 / Linear |
| Binary Cross Entropy | 이진 분류 | Sigmoid |
| Cross Entropy | 다중 분류 | Softmax |

## 연결 개념

- [[gradient_descent]] — 이 손실을 최소화하는 방법
- [[perceptron_nn]] — 손실을 계산하는 모델 구조
- [[Ch3_Sequential/rnn]] — RNN에서도 동일한 손실 함수 사용
