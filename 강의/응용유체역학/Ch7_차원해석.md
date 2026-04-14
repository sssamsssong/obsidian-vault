# Ch7. 차원 해석, 상사성 및 모델링

## 개요

$\Delta P_\ell = f(D, \rho, \mu, V)$처럼 물리량 사이의 관계를 차원 해석으로 정리한다.

단순 변수 실험 (각 변수별 그래프 4개) → 차원 해석으로 **무차원 파이 그룹 1개**로 압축:

$$\frac{D\Delta P_\ell}{\rho V^2} = \phi\left(\frac{\rho V D}{\mu}\right)$$

→ 실험 효율 극대화 (비차원화 파이)

---

## 버킹엄 파이 정리 (Buckingham Pi Theorem)

$k$개 변수, $r$개 기본 차원이면:

$$\Pi_1 = \phi(\Pi_2, \ldots, \Pi_{k-r}) \qquad (k-r \text{개의 } \Pi \text{그룹})$$

### 풀이 절차

1. 변수 $k$개 선택, 기본 차원 $r$개 파악
2. 반복 변수(repeating variables) $r$개 선택
3. 나머지 각 변수와 반복 변수를 조합하여 $\Pi_i$ 구성
4. 각 $\Pi_i$의 지수를 연립방정식으로 결정

---

## 예제 1: 진자 (Pendulum)

$$\omega = f(\ell, m, g)$$

$k=4$, $r=3$ (L, M, T) → $\Pi$ 그룹 1개

$$\Pi = \omega^a \cdot \ell^b \cdot m^c \cdot g^d$$

차원 분석:

$$[T^{-1}]^a [L]^b [M]^c [LT^{-2}]^d = L^0 M^0 T^0$$

- $b+d=0$, $c=0$, $a+2d=0$ → $d=-\frac{1}{2}$, $b=\frac{1}{2}$

$$\Pi = \omega \cdot \ell^{1/2} \cdot m^0 \cdot g^{-1/2} = C$$

$$\boxed{\omega = C\sqrt{\frac{g}{\ell}}}$$

---

## 예제 2: 파이프 내 압력 강하

$$\Delta P_\ell = f(D, \rho, \mu, V)$$

**차원 체계:** $F$-$L$-$T$

| 변수              | 차원                         |
| --------------- | -------------------------- |
| $\Delta P_\ell$ | $F \cdot L^{-3}$           |
| $D$             | $L$                        |
| $\rho$          | $F \cdot L^{-4} \cdot T^2$ |
| $\mu$           | $F \cdot L^{-2} \cdot T$   |
| $V$             | $L \cdot T^{-1}$           |

$k=5$, $r=3$ (F, L, T) → **$\Pi$ 그룹 2개**

### $\Pi_1$ 계산

$$\Pi_1 = \Delta P_\ell \cdot D^a V^b \rho^c$$

지수 조건: $c=-1$, $b=-2$, $a=1$

$$\Pi_1 = \frac{\Delta P_\ell \cdot D}{\rho V^2}$$

### $\Pi_2$ 계산

$$\Pi_2 = \mu \cdot D^a V^b \rho^c$$

지수 조건: $c=-1$, $b=-1$, $a=-1$

$$\Pi_2 = \frac{\mu}{\rho V D}$$

### 결과

$$\frac{D\Delta P_\ell}{\rho V^2} = \phi\left(\frac{\mu}{\rho V D}\right) = \phi\left(\frac{1}{Re}\right)$$

실험 결과 예시: $\Pi_1 = 0.115 \cdot \Pi_2^{-0.25}$

---

## 예제 3: 구(Sphere)에 작용하는 항력

$$F_D = f(D, V, \mu) \quad k-r = 4-3 = 1$$

$$\Pi_1 = \frac{F_D}{\mu V D} = C \Rightarrow F_D = C\cdot\mu V D$$

---

## 예제 4: 간판(Sign Board)에 작용하는 항력

$$F_D = f(w, h, V, \rho, \mu) \quad 6-3=3 \text{ 그룹}$$

$$\frac{F_D}{w^2\rho V^2} = \phi\left(\frac{w}{h}, \frac{\rho V w}{\mu}\right)$$

---

## 상사성 및 모델링 (Similitude and Modeling)

모델(model)과 실물(prototype) 사이의 유사성 조건.

### 기하학적 상사 (Geometric Similarity)

$$\frac{w_m}{h_m} = \frac{w}{h}$$

### 동역학적 상사 (Dynamic Similarity, 레이놀즈 수 일치)

$$\frac{\rho_m V_m w_m}{\mu_m} = \frac{\rho V w}{\mu} = Re$$

### 힘 스케일링

$$\frac{F_{D,m}}{w_m^2 \rho_m V_m^2} = \frac{F_D}{w^2\rho V^2}$$

$$\Rightarrow F_D = \left(\frac{w}{w_m}\right)^2 \cdot \left(\frac{\rho}{\rho_m}\right) \cdot \left(\frac{V}{V_m}\right)^2 \cdot F_{D,m}$$

### 파이 정리의 일반적 적용

$$\phi(\Pi_1, \Pi_2, \Pi_3)_{\text{모델}} = \phi(\Pi_1', \Pi_2', \Pi_3')_{\text{실물}}$$

모델과 실물의 파이 그룹을 일치시키면 → 상사성 달성
