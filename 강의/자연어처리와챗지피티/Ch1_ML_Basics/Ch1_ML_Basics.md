# Ch 1. Machine Learning Basics — 개요

#기초 #머신러닝 #신경망

## 이 챕터의 흐름

```
데이터 (Task/Experience)
    → 모델 (Perceptron / NN)
        → 손실 측정 (Loss)
            → 최적화 (Gradient Descent)
                → 일반화 (Regularization)
```

## 핵심 개념 목록

- [[loss_functions]] — MSE, Cross Entropy, Binary Cross Entropy
- [[gradient_descent]] — 경사하강법, Loss 최적화
- [[perceptron_nn]] — 퍼셉트론, 활성화 함수, Dense Layer, Backprop
- [[regularization]] — L2, Dropout, Early Stopping

## 머신러닝의 4요소

| 요소 | 설명 |
|------|------|
| **Task** | 모델이 수행할 작업 (분류, 회귀 등) |
| **Experience** | 학습 데이터 $D = \{(x_i, y_i)\}$ |
| **Performance Measure** | 손실 함수 $L(\hat{y}, y)$ |
| **Program (Model)** | $\hat{y} = f(x; W)$ |

## AI 계층 구조

```
AI
└── Machine Learning
    └── Representation Learning
        └── Deep Learning
```

## 연결 개념

- 다음 챕터: [[Ch2_Word_Embedding]] — 여기서 배운 NN 구조를 텍스트에 적용
- [[Ch3_Sequential/rnn]] — Dense Layer의 한계를 극복한 순차 모델
