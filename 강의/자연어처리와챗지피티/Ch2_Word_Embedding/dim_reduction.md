# 차원 축소 (Dimensionality Reduction)

#임베딩 #차원축소 #Autoencoder

## 개념

고차원 데이터를 **유사한 정보를 유지하면서** 저차원으로 변환하는 과정.

$$\text{고차원 } x \in \mathbb{R}^{|V|} \longrightarrow \text{저차원 } z \in \mathbb{R}^{d},\quad d \ll |V|$$

---

## PCA (Principal Component Analysis)

- 데이터의 **분산이 최대**가 되는 방향의 부분공간으로 투영
- 동시에 투영 거리(오차)를 최소화 (least-squares sense)
- **선형** 변환

```
원본 데이터 (2D)        PCA 후 (1D)
      *   *
   *    *         →    ——●——●——●——
      *   *
```

**특징:** 비지도 학습, 해석 가능, 선형 가정

---

## LDA (Linear Discriminant Analysis)

- **클래스 정보를 활용**하여 클래스 간 분리를 최대화하는 방향으로 투영
- PCA와 달리 **지도 학습** 방식
- 분류 태스크 전처리에 효과적

---

## Autoencoder

[[Ch1_ML_Basics/perceptron_nn]] 구조를 활용한 **비선형** 차원 축소.

```
입력 x (고차원)
    ↓
  Encoder: g(x) → z (저차원, latent code)
    ↓
  Decoder: f(z) → x̂ (재구성)
    ↓
  Loss: MSE(x̂, x) 최소화
```

### 구조

| 파트 | 역할 |
|------|------|
| **Encoder** | 고차원 입력 → 저차원 잠재 표현 $z$ |
| **Decoder** | 저차원 $z$ → 원본 복원 $\hat{x}$ |
| **Loss** | $L = \| f(g(x)) - x \|^2$ (MSE) |

### 핵심 원리

- 병목(bottleneck) 구조 강제 → 가장 중요한 특징만 압축
- 재구성 오차를 줄이면서 자연스럽게 유의미한 표현 학습
- 비선형 변환 가능 → PCA보다 표현력 높음

---

## 비교

| 방법 | 학습 방식 | 변환 종류 | 특징 |
|------|-----------|-----------|------|
| PCA | 비지도 | 선형 | 해석 용이, 빠름 |
| LDA | 지도 | 선형 | 분류 목적 |
| Autoencoder | 비지도 | 비선형 | 표현력 높음, 학습 필요 |

## 연결 개념

- [[one_hot]] — 이 방법들이 해결하려는 문제
- [[word2vec_cbow]] — 차원 축소를 임베딩 학습과 결합
- [[Ch1_ML_Basics/perceptron_nn]] — Autoencoder의 기반 구조
