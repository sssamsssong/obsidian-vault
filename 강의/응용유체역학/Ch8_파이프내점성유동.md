# Ch8. 파이프 내 점성 유동

## §8.1 파이프 유동의 일반적 특성

### ① 층류 vs 난류

레이놀즈의 실험적 관찰:

| 영역                               | Re     | 유동 형태             |
| -------------------------------- | ------ | ----------------- |
| $Re \lesssim 2000$               | 낮은 $V$ | 층류 (Laminar)      |
| $2000 \lesssim Re \lesssim 4000$ | 중간     | 전이 (Transitional) |
| $Re \gtrsim 4000$                | 높은 $V$ | 난류 (Turbulent)    |

여기서 $Re \equiv \frac{\rho V D}{\mu}$

### ② 난류의 특성

- 불규칙적, 비정상, 비1차원
- 속도가 시간에 따라 모든 방향으로 변동

### 수돗물로 컵 채우기 예제

$$Re = \frac{\rho V D}{\mu} \Rightarrow V_c \sim \frac{Re\cdot\mu D}{\rho}$$

충전 시간 $t_c \sim \frac{V}{\dot{Q}} \sim \frac{\rho V}{\text{Re}\mu D} \sim \frac{10^3 \times 2\times10^{-4}}{10^{-3}\times10^{-2}\times2\times10^3} \sim 10\text{s}$

($V\sim200\text{mL}$, $D\sim1\text{cm}$, $\mu\sim10^{-3}\text{Pa·s}$, $\rho\sim10^3\text{kg/m}^3$, 층류 가정 → $Re<2000$)

만약 $t < t_c$ → 난류?

### ③ 입구 영역 유동 (Entrance Region Flow)

벽면에서 경계층이 성장하며, 처음에는 **비점성 코어(inviscid core)** (점성 효과 무시 가능)가 존재.

- **입구 영역** → **완전 발달 영역**
- 입구 길이 $\ell_e$:

$$\frac{\ell_e}{D} = 0.06\,Re \quad \text{(층류)}$$

$$\frac{\ell_e}{D} \approx 4.4\,Re^{1/6} \quad \text{(난류)}$$

($Re=3000$ → $\ell_e \approx 120D$ 층류, $\approx 16D$ 난류)

### 압력 분포

$\frac{\partial P}{\partial x}$ 존재 → $\Delta P$는 점성력을 극복하고 유체 내부를 구동하는 데 필요.
![[Pasted image 20260414222541.png]]
---

## §8.2 완전 발달 층류 유동

큰 가정 없이 이루어지는 몇 안 되는 이론 해석 중 하나. 파이프 유동 이해의 기본 틀.

**속도 프로파일 유도 방법 3가지:**
1. 유체 요소에 대한 뉴턴 제2법칙
2. 나비에-스토크스 방정식
3. 차원 해석

### ① 뉴턴 제2법칙 유도

정상, 완전 발달 유동에서:

$$\frac{D\vec{V}}{Dt} = \frac{\partial\vec{V}}{\partial t} + (\vec{V}\cdot\nabla)\vec{V} = 0$$

힘 평형 ($\Sigma F = 0$):

$$p_1\pi r^2 - (p_1 - \Delta p)\pi r^2 - 2\pi r\ell\cdot\tau = 0$$

$$\Rightarrow \frac{\Delta p}{\ell} = \frac{2\tau}{r} = -\frac{2\mu}{r}\frac{du}{dr}$$

경계 조건 $u(r=D/2)=0$으로 적분:

$$u = C - \frac{\Delta p}{4\mu\ell}r^2 \Rightarrow C = \frac{\Delta p D^2}{16\mu\ell}$$

$$\boxed{u = \frac{\Delta p D^2}{16\mu\ell}\left(1-\left(\frac{2r}{D}\right)^2\right)} \quad \text{(포물선 프로파일)}$$

**체적 유량 (포아죄유 법칙, Poiseuille's Law):**

$$Q = \int u\,dA = \frac{\pi D^4}{128\mu\ell}\Delta p$$

**경사진 파이프:**

$$Q = \frac{\pi D^4}{128\mu\ell}(\Delta p - \rho g\ell\sin\theta)$$

### ② 나비에-스토크스 유도 (복습)

$\vec{V} = u(r)\hat{i}$, $\vec{g} = -\rho g\sin\theta\,\hat{i}$로 놓으면:

$$\frac{\partial p}{\partial x} + \rho g\sin\theta = \frac{\mu}{r}\frac{\partial}{\partial r}\left(r\frac{\partial u}{\partial r}\right)$$

경계 조건: $u(r=0)<\infty$, $u(r=D/2)=0$

### ③ 차원 해석

$$\Delta p = f(V, \ell, \mu, D) \quad \Rightarrow \quad k-r = 5-3=2$$

$$\frac{D\Delta p}{\mu V} = \phi\left(\frac{\ell}{D}\right)$$

실험적으로 $\Delta p \propto \ell$ → $\frac{D\Delta p}{\mu V} = C\cdot\frac{\ell}{D}$ ($C=32$)

$$\frac{\Delta p}{\ell} = \frac{32\mu V}{D^2}$$

**무차원화 (다르시-바이스바흐, Darcy-Weisbach):**

$$\frac{\Delta p}{\frac{1}{2}\rho V^2} = \frac{64}{Re}\cdot\frac{\ell}{D} \Rightarrow \frac{\Delta p}{\frac{1}{2}\rho V^2}\bigg/\frac{\ell}{D} = \frac{64}{Re} \equiv f \quad \text{(다르시 마찰계수)}$$

### 에너지 고려

수정 베르누이 방정식 (Eq. 5.89):

$$\frac{p_1}{\gamma} + \alpha_1\frac{V_1^2}{2g} + z_1 = \frac{p_2}{\gamma} + \alpha_2\frac{V_2^2}{2g} + z_2 + h_L$$

운동 에너지 보정 계수 $\alpha$:

| 유동 형태    | $\alpha$ | 프로파일  |
| -------- | -------- | ----- |
| 균일 유동    | 1        | 균일    |
| 난류       | 1.08     | 거의 균일 |
| 포물선 (층류) | 2        | 포물선   |

$$\alpha = \frac{\int\left(\frac{1}{2}\rho V^2\right)\vec{V}\cdot\hat{n}\,dA}{\frac{1}{2}\dot{m}V^2}$$

완전 발달 유동 ($\alpha_1 = \alpha_2$, $V_1 = V_2$):

$$h_L = \frac{p_1-p_2}{\gamma} + z_1 - z_2 = \frac{\Delta p - \rho g\sin\theta}{\gamma}$$

---

## §8.3 완전 발달 난류 유동

### 난류 유동의 특성

- 시간에 따라 불규칙적으로 변동
- 국소화된 불규칙 3D 유동 (와류 운동, eddy motion)
- 열과 운동량의 효과적인 혼합
  (분자 충돌에 의존하는 확산과 달리)

### 수학적 기술

$$u = \bar{u} + u', \quad \bar{u} = \frac{1}{T}\int_{t_0}^{t_0+T}u\,dt, \quad \overline{u'} = 0$$

**난류 강도(Turbulent Intensity):**

$$\beta = \frac{\sqrt{\overline{(u')^2}}}{\bar{u}}$$

- 강 유동: $\beta \sim 1$
- 잘 설계된 시스템: $\beta \sim 10^{-3}$

### 난류 전단 응력

**층류:** $\tau = \mu\frac{du}{dy}$ (무작위 분자 충돌에 의해 발생)

**난류:**

$$\tau = \mu\frac{d\bar{u}}{dy} + (-\rho\overline{u'v'})$$

여기서 $-\rho\overline{u'v'}$ = **난류 전단 응력 (레이놀즈 응력, Reynolds stress)**

- 불규칙 국소화 유동(와류 유동)을 통한 운동량 교환에 의해 발생
- $u'v' < 0 \Rightarrow -\overline{u'v'} > 0$
- 높은 운동량 영역 → 낮은 운동량 영역으로 운동량 전달

### 난류 유동의 구조

$$\tau = \tau_{\text{층류}} + \tau_{\text{난류}}$$

| 층 | 설명 |
|----|------|
| 점성 저층 (Viscous sublayer, $r \approx R$) | $\tau_{\text{층류}} \gg \tau_{\text{난류}}$ |
| 중간층 (Overlap layer) | 전이 구간 |
| 외부층 (Outer layer) | $\tau_{\text{층류}} \ll \tau_{\text{난류}}$ |

**점성 저층:**

$$\frac{\bar{u}}{u^*} = \frac{1}{\nu/u^*}y, \quad 0 \leq \frac{y}{\nu/u^*} \leq 5$$

여기서:
- $u^* \equiv \sqrt{\tau_w/\rho}$: 마찰 속도 (friction velocity)
- $y \equiv R-r$: 벽면으로부터의 거리
- $\nu = \mu/\rho$: 동점성 계수 (kinematic viscosity)

**경험적 모델 (멱함수 법칙):**

$$\frac{\bar{u}}{V_c} = \left(1-\frac{r}{R}\right)^n, \quad n=n(Re)$$

→ 층류의 포물선 프로파일에 비해 **평탄한 속도 분포**

### 난류 전단 응력 모델링

$$\tau_{\text{난류}} = \mathcal{Z}\frac{d\bar{u}}{dy} \quad (\mathcal{Z}: \text{와점성계수(eddy viscosity), 미지})$$

**프란틀 혼합 길이 모델(Prandtl's mixing length):**

$$\tau_{\text{난류}} = \rho \ell_m^2\left(\frac{d\bar{u}}{dy}\right)^2$$

여기서 $\ell_m$: 혼합 길이 = 서로 다른 속도를 가진 두 인접 영역 사이의 거리.

---

## §8.4 파이프 유동 손실

### 에너지 방정식

$$P_2 + \frac{\alpha_2}{2}\rho V_2^2 + \rho g z_2 = P_1 + \frac{\alpha_1}{2}\rho V_1^2 + \rho g z_1 - \Delta\mathcal{E}$$

**수두 손실 (Head Loss):**

$$h_L \equiv \frac{\Delta\mathcal{E}}{\gamma} = \left(\frac{p}{\gamma}+\frac{\alpha}{2g}V^2+z\right)_1 - \left(\quad\right)_2$$

$$h_L = h_{L,\text{주손실}} + h_{L,\text{부손실}}$$

| 종류 | 발생 원인 |
|------|----------|
| 주손실 (Major loss) | 직관 (straight pipe) |
| 부손실 (Minor loss) | 각종 부품 (밸브, 벤드 등) |

### 주손실 (Major Losses)

수평 원형 직관에서 정상, 비압축성, 난류 유동:

$$\Delta p = f(V, D, \ell, \varepsilon, \mu, \rho) \quad \text{(변수 7개, 기본 차원 3개)}$$

$$\frac{\Delta p}{\frac{1}{2}\rho V^2} = \phi\left(\frac{\rho V D}{\mu}, \frac{\ell}{D}, \frac{\varepsilon}{D}\right) = \frac{\ell}{D}\phi\left(Re, \frac{\varepsilon}{D}\right) = f\cdot\frac{\ell}{D}$$

**다르시-바이스바흐 방정식:**

$$h_{L,\text{주}} = \frac{\Delta p}{\gamma} = f\cdot\frac{\ell}{D}\cdot\frac{V^2}{2g}$$

등직경 파이프 ($V_1=V_2$, $z_1=z_2$):

$$p_1-p_2 = \underbrace{(z_2-z_1)\rho g}_{\text{고도 변화}} + \underbrace{f\cdot\frac{\ell}{D}\cdot\frac{V^2}{2g}}_{\text{수두 손실}}$$

### 마찰계수 $f$

| 영역 | $f$ |
|------|-----|
| 층류 | $f = \frac{64}{Re}$ |
| 난류 | $f = f\left(Re, \frac{\varepsilon}{D}\right)$ → **무디 선도 (Moody chart)** |

**무디 선도:** $\varepsilon/D$를 매개변수로 하는 $f$ vs. $Re$ 그래프.

---

## 예제 8.4 (20°C 물)

주어진 값: $\rho = 998\,\text{kg/m}^3$, $\nu = 1.004\times10^{-6}\,\text{m}^2/\text{s}$, $Q = 4\times10^{-2}\,\text{m}^3/\text{s}$, $D = 0.1\,\text{m}$

**(a) 점성 저층의 두께:**

$$\frac{\delta_s u^*}{\nu} = 5 \Rightarrow \delta_s = \frac{5\nu}{u^*} = 5\sqrt{\frac{\nu^2}{\tau_w/\rho}}$$

벽면 전단력: $\tau\cdot2\pi R\ell = \Delta p\cdot\pi\left(\frac{D}{2}\right)^2 \Rightarrow \tau = \frac{\Delta p\cdot D}{4\ell}$

**(b) 평균 속도:**

$$V = \frac{Q}{A} = \sim, \quad Re = \frac{VD}{\nu} = \sim$$

$n$ 주어질 때: $\frac{\bar{u}}{V_c} = \left(1-\frac{r}{R}\right)^n$

$$Q = V_c\int_0^R\left(1-\frac{r}{R}\right)^n 2\pi r\,dr = 2\pi R V_c\frac{R^2}{(n+1)(2n+1)}$$

$$V_c = \frac{(n+1)(2n+1)}{2n^2}V$$

**(c) 난류 전단 응력 vs 층류 전단 응력:**

$$\tau = \tau_w\cdot\frac{r}{D/2} = \tau_{\text{층류}} + \tau_{\text{난류}}$$

$$\mu\frac{d\bar{u}}{dr} = \mu\frac{V_c}{nR}\left(1-\frac{r}{R}\right)^{\frac{1-n}{n}} = \sim$$

$$\frac{\tau_{\text{난류}}}{\tau_{\text{층류}}} \approx 1270 \Rightarrow \tau_{\text{난류}} \gg \tau_{\text{층류}}$$

---

## §8.4 파이프 유동 손실 (계속) — 부손실 & 마찰계수 상세

### 마찰계수 $f$ — 무디 선도 상세

**층류:** $f = \frac{64}{Re}$ → $\log f = \log 64 - \log Re$ (직선)

**난류 영역 구분:**

| 영역 | 특성 |
|------|------|
| 완전 난류 (Completely turbulent) | $f = \phi\!\left(\frac{\varepsilon}{D}\right)$ (Re 무관) |
| 전이 구간 (Transition) | $f = f\!\left(Re,\,\frac{\varepsilon}{D}\right)$ |
| 매끄러운 관 (Smooth/glass) | $\varepsilon/D \to 0$ |

**콜브룩 공식 (Colebrook formula):**

$$\frac{1}{\sqrt{f}} = -2.0\log\!\left(\frac{\varepsilon/D}{3.7} + \frac{2.51}{Re\sqrt{f}}\right)$$

> 특정 $Re$ 영역에 대한 다른 상관식(correlation)들도 존재함.

---

### 부손실 (Minor Losses)

파이프 시스템 내 **추가 부품**(밸브, 벤드, 티 등)에 의해 발생하는 수두 손실.

$$h_{L,\text{부}} = K_L \cdot \frac{V^2}{2g} \qquad \left(K_L = \frac{\Delta p}{\frac{1}{2}\rho V^2} : \text{손실 계수}\right)$$

**등가 길이(Equivalent length)로 표현:**

$$h_{L,\text{부}} = f \cdot \frac{\ell_{eq}}{D} \cdot \frac{V^2}{2g} = K_L \cdot \frac{V^2}{2g} \qquad \left(\ell_{eq} \equiv \frac{D \cdot K_L}{f}\right)$$

$\ell_{eq}$: 같은 손실을 유발하는 등가 직관 길이

---

#### ① 밸브 (Valve)

$$K_L = f(\text{geometry},\, Re) \approx f(\text{geometry})$$

형상에 주로 의존 ($Re$ 의존성은 약함).

---

#### ② 파이프 입구 (Pipe Entrance)

입구 형상에 따라 $K_L$ 크게 달라짐:

| 입구 형상 | $K_L$ |
|---------|-------|
| 날카로운 모서리 (sharp) | ~0.5 |
| 약간 둥근 형태 | 감소 |
| 완전히 둥근 형태 (rounded) | 크게 감소 |

- 가속 구간에서 상당한 운동에너지 손실 ($\frac{1}{2}\rho V^2 \to P$)
- 분리 유동(separated flow) 발생 → 손실 증가
- 이상 흐름: $p + \frac{1}{2}\rho V^2 = \text{const}$이어야 하나, 손실로 인해 압력이 더 낮아짐

---

#### ③ 파이프 출구 — 탱크 연결 (Pipe Exit to Tank)

$$p_1 + \tfrac{1}{2}\rho V_1^2 = p_2 + \tfrac{1}{2}\rho V_2^2 + \Delta\mathcal{E}, \quad V_2 \approx 0,\; p_1 \approx p_2$$

$$h_{L,\text{부}} = \frac{\Delta\mathcal{E}}{\gamma} = \frac{V_1^2}{2g} \quad \therefore K_L = 1$$

---

#### ④ 급격한 확대 (Sudden Expansion)

**에너지 방정식:**

$$p_1 + \tfrac{1}{2}\rho V_1^2 = p_3 + \tfrac{1}{2}\rho V_3^2 + \Delta\mathcal{E}$$

$$h_{L,\text{부}} = \frac{\Delta\mathcal{E}}{\gamma} = \frac{p_1 - p_3}{\rho g} + \frac{V_1^2 - V_3^2}{2g}$$

$$K_L = \frac{h_{L,\text{부}}}{\frac{V_1^2}{2g}} = \frac{2}{\rho V_1^2}(p_1 - p_3) + \left(1 - \left(\frac{V_3}{V_1}\right)^2\right)$$

**검사체적 운동량 방정식 적용** (정상, $A_1 V_1 = A_3 V_3$):

$$-\rho V_1 V_3 + \rho V_3^2 = p_1 - p_3 \quad \Rightarrow \quad p_1 - p_3 = \rho V_3(V_3 - V_1)$$

$$\boxed{K_L = \left(1 - \frac{V_3}{V_1}\right)^2 = \left(1 - \frac{A_1}{A_3}\right)^2}$$

---

#### ⑤ 원뿔형 디퓨저 (Conical Diffuser) — 감속

$$K_L = \phi\!\left(\theta,\, \frac{A_1}{A_2}\right)$$

- **최적 각도**: $\theta \approx 8°$ (손실 최소, 단 매우 길어짐)
- $\theta = 90°$, $180°$: 손실 급격히 증가

---

#### ⑥ 원뿔형 축소 / 노즐 (Conical Contraction / Nozzle)

$$K_L \approx 0.02 \quad (\theta \lesssim 30°)$$

매우 효율적인 가속이 가능 → $K_L$이 매우 작음.

---

#### ⑦ 벤드 (Bend)

$$K_L = \phi\!\left(\frac{R}{D},\, \frac{\varepsilon}{D}\right)$$

원심력 불균형으로 인해 **이차 유동(secondary flow)** 발생 (파이프 단면 내 쌍 와류 형성).

---

### 비원형 파이프 (Non-circular Pipes)

원형 파이프 결과를 실용적으로 적용:
→ $D$ 대신 **수력 직경(Hydraulic diameter)** $D_h$ 사용

$$D_h = \frac{4A}{P}$$

여기서 $A$: 단면적, $P$: 젖은 둘레(wetted perimeter)

> §8.5 예제 — 자습 / §8.6 — skip
