# Ch6. 유체 요소에 대한 보존 방정식

## 개요

**뉴턴 유체, 비압축성 유체에 대한 보존 방정식**
(응력은 변형률에 선형 비례)

---

## ① 공간에서 연속 물리량의 미소 변화

$f(x_1 + \delta x)$의 테일러 전개:

$$f(x_1 + \delta x) = f(x_1) + \frac{1}{1!}\frac{df}{dx}\bigg|_{x=x_1} \cdot \delta x + \frac{1}{2!}\frac{d^2f}{dx^2}\bigg|_{x_1} \cdot \delta x^2 + \cdots$$

**미소 요소**에서는 고차항($\delta x^2, \delta x^3, \ldots$)을 무시할 수 있다:

$$\boxed{f(x_1 + \delta x) \approx f(x_1) + \frac{df}{dx}\bigg|_{x=x_1} \cdot \delta x}$$

---

## ② 질량 보존 (연속 방정식)

### 유도

점 A를 포함하는 검사체적에서 $\hat{x}$ 방향 단위 면적당 질량 유량:

- $\dot{m}_1 = \left[\rho u + \frac{\partial}{\partial x}(\rho u)\cdot\left(-\frac{\delta x}{2}\right)\right] \delta y \cdot \delta z$
- $\dot{m}_2 = \left[\rho u + \frac{\partial}{\partial x}(\rho u)\cdot\left(+\frac{\delta x}{2}\right)\right] \delta y \cdot \delta z$

**$\hat{x}$ 방향 순 질량 유량 = $\dot{m}_2 - \dot{m}_1 = \frac{\partial}{\partial x}(\rho u) \cdot \delta x \cdot \delta y \cdot \delta z$**

$$\Rightarrow \text{순 질량 유량} = \left[\frac{\partial}{\partial x}(\rho u) + \frac{\partial}{\partial y}(\rho v) + \frac{\partial}{\partial z}(\rho w)\right] \delta x \cdot \delta y \cdot \delta z$$

### 질량 보존 법칙

$$\frac{\partial \rho}{\partial t} + \frac{\partial}{\partial x}(\rho u) + \frac{\partial}{\partial y}(\rho v) + \frac{\partial}{\partial z}(\rho w) = 0$$

$$= \frac{\partial \rho}{\partial t} + \nabla \cdot (\rho \vec{V}) = 0$$

**비압축성 유체** ($\rho = \text{const}$):

$$\boxed{\nabla \cdot \vec{V} = 0}$$

> 속도의 발산 → 국소 체적 팽창률

---

## ③ 줄에서의 인장 응력 / 자유 물체 도형

$\tau$: $\hat{x}$ 방향으로 작용하는 단위 면적당 힘

- $\tau_{x+\delta x} = \tau_x + \frac{\partial}{\partial x}\tau_x \cdot \delta x$
- $\tau_{x-\delta x} = \tau_x + \frac{\partial}{\partial x}\tau_x(-\delta x)$

**미소 검사체적의 자유 물체 도형:**

$\hat{x}$-법선 두 면에 작용하는 힘:
- 왼쪽 면: $\tau + \frac{\partial}{\partial x}\tau\left(-\frac{\delta x}{2}\right)$
- 오른쪽 면: $\tau + \frac{\partial}{\partial x}\tau \cdot \frac{\delta x}{2}$

---

## ④ 물질 미분 (Material Derivative)

물질 입자의 물성 변화율:

$$\frac{D}{Dt}(\quad) = \frac{\partial}{\partial t}(\quad) + (\vec{V} \cdot \nabla)(\quad)$$

여기서 $\vec{V} \cdot \nabla = u\frac{\partial}{\partial x} + v\frac{\partial}{\partial y} + w\frac{\partial}{\partial z}$

| 항 | 명칭 |
|------|------|
| $\frac{\partial}{\partial t}$ | 시간 변화 (국소 변화) |
| $(\vec{V}\cdot\nabla)$ | 공간 변화 (대류 미분) |

> 온도장 예로, A 지점에서의 변화 $\left(\frac{\partial T}{\partial t}\right)$는 모든 입자의 변화를 나타내지 못한다.  
> A를 따라가는 particle의 변화가 $\frac{D}{Dt}(\quad)$

### 가속도

$$\vec{a} = \frac{D\vec{V}}{Dt} = \frac{\partial}{\partial t}\vec{V} + (\vec{V}\cdot\nabla)\vec{V}$$

$$= \frac{\partial}{\partial t}\vec{V} + u\frac{\partial}{\partial x}\vec{V} + v\frac{\partial}{\partial y}\vec{V} + w\frac{\partial}{\partial z}\vec{V}$$

**특수 경우 (정상, 1D 벽면 유동):** $v=0, w=0$

$$\vec{a} = \frac{\partial \vec{V}}{\partial t} + u\frac{\partial \vec{V}}{\partial x}$$

---

## ⑤ 선운동량 방정식

### 미소 질량에 대한 뉴턴 제2법칙

$$\delta\vec{F} = \frac{D}{Dt}(\vec{V}\cdot\delta m) = \delta m \cdot \frac{D\vec{V}}{Dt} = \delta m \cdot \vec{a}$$

$$\delta\vec{F} = \delta\vec{F}_b + \delta\vec{F}_s \qquad (\delta\vec{F}_b = \delta m \cdot \vec{g})$$

### 3D 공간에서의 응력 텐서

**응력 표기:** $\tau_{ij}$ 또는 $\sigma_{ij}$
- 첫 번째 첨자 $i$: 면의 법선 방향
- 두 번째 첨자 $j$: 힘의 방향

$$\begin{pmatrix} \sigma_{xx} & \tau_{xy} & \tau_{xz} \\ \tau_{yx} & \sigma_{yy} & \tau_{yz} \\ \tau_{zx} & \tau_{zy} & \sigma_{zz} \end{pmatrix}$$

### 자유 물체 도형 ($\hat{x}$ 방향 표면력만 고려)

6개 면의 기여:
1. $\sigma_{xx} + \frac{\partial \sigma_{xx}}{\partial x}\cdot\frac{\delta x}{2}$
2. $\sigma_{xx} + \frac{\partial \sigma_{xx}}{\partial x}\cdot\left(-\frac{\delta x}{2}\right)$
3. $\tau_{yx} + \frac{\partial \tau_{yx}}{\partial y}\cdot\frac{\delta y}{2}$
4. $\tau_{zx} + \frac{\partial \tau_{zx}}{\partial z}\cdot\frac{\delta z}{2}$

$\hat{x}$ 방향 순 표면력:

$$F = \delta x \cdot \delta y \cdot \delta z \cdot \frac{\partial \sigma_{xx}}{\partial x}$$

### $\hat{x}$ 방향 운동 방정식

$$\rho\left(\frac{\partial u}{\partial t} + u\frac{\partial u}{\partial x} + v\frac{\partial u}{\partial y} + w\frac{\partial u}{\partial z}\right) = \rho g_x + \frac{\partial \sigma_{xx}}{\partial x} + \frac{\partial \tau_{yx}}{\partial y} + \frac{\partial \tau_{zx}}{\partial z}$$

마찬가지로 $\hat{y}$, $\hat{z}$ 방향:

$$\rho\left(\frac{\partial v}{\partial t} + u\frac{\partial v}{\partial x} + v\frac{\partial v}{\partial y} + w\frac{\partial v}{\partial z}\right) = \rho g_y + \frac{\partial \tau_{xy}}{\partial x} + \frac{\partial \sigma_{yy}}{\partial y} + \frac{\partial \tau_{zy}}{\partial z}$$

$$\rho\left(\frac{\partial w}{\partial t} + u\frac{\partial w}{\partial x} + v\frac{\partial w}{\partial y} + w\frac{\partial w}{\partial z}\right) = \rho g_z + \frac{\partial \tau_{xz}}{\partial x} + \frac{\partial \tau_{yz}}{\partial y} + \frac{\partial \sigma_{zz}}{\partial z}$$

---

## ⑥ 구성 방정식 (뉴턴 비압축성 유체)

### 수직 응력 (Normal stress)

$$\sigma_{xx} = -p + 2\mu\frac{\partial u}{\partial x}, \quad \sigma_{yy} = -p + 2\mu\frac{\partial v}{\partial y}, \quad \sigma_{zz} = -p + 2\mu\frac{\partial w}{\partial z}$$

### 전단 응력 (대칭)

$$\tau_{xy} = \tau_{yx} = \mu\left(\frac{\partial u}{\partial y} + \frac{\partial v}{\partial x}\right)$$

$$\tau_{yz} = \tau_{zy} = \mu\left(\frac{\partial v}{\partial z} + \frac{\partial w}{\partial y}\right)$$

$$\tau_{zx} = \tau_{xz} = \mu\left(\frac{\partial w}{\partial x} + \frac{\partial u}{\partial z}\right)$$

### 변형률 텐서

$$\begin{pmatrix} \frac{\partial u}{\partial x} & \frac{\partial u}{\partial y} & \frac{\partial u}{\partial z} \\ \frac{\partial v}{\partial x} & \frac{\partial v}{\partial y} & \frac{\partial v}{\partial z} \\ \frac{\partial w}{\partial x} & \frac{\partial w}{\partial y} & \frac{\partial w}{\partial z} \end{pmatrix}$$

---

## ⑦ 나비에-스토크스 방정식 (운동량 + 구성 방정식)

### 유도 (x 방향)

$$\rho\left(\frac{\partial u}{\partial t} + u\frac{\partial u}{\partial x} + v\frac{\partial u}{\partial y} + w\frac{\partial u}{\partial z}\right)$$

$$= \rho g_x - \frac{\partial p}{\partial x} + \mu\left(\frac{\partial^2 u}{\partial x^2} + \frac{\partial^2 u}{\partial y^2} + \frac{\partial^2 u}{\partial z^2}\right) + \frac{\partial}{\partial x}\left(\frac{\partial u}{\partial x} + \frac{\partial v}{\partial y} + \frac{\partial w}{\partial z}\right)$$

$\nabla\cdot\vec{V}=0$ (비압축성) 적용:

$$= \rho g_x - \frac{\partial p}{\partial x} + \mu\nabla^2 u$$

### 벡터 형태 (나비에-스토크스)

$$\boxed{\rho\left(\frac{\partial \vec{V}}{\partial t} + (\vec{V}\cdot\nabla)\vec{V}\right) = \rho\vec{g} - \nabla p + \mu\nabla^2\vec{V}}$$

| 항 | 물리적 의미 |
|------|-----------------|
| $\rho\vec{g}$ | 체적력 |
| $-\nabla p$ | 열역학적 압력 |
| $\mu\nabla^2\vec{V}$ | 점성 효과 |

---

## ⑧ 에너지 보존 방정식

### 열역학 제1법칙

$$dE = \delta Q - \delta W$$

- $E$: 내부 에너지 + 운동 에너지 + 중력 에너지 + …
- $\delta Q$: 주변으로부터의 열
- $\delta W$: 검사체적이 주변에 하는 일 = 표면력 × 변위

### 변화율 형태

$$\rho\frac{DE}{Dt} = -\rho(\nabla\cdot\vec{V}) + \dot{q}\cdot\nabla\vec{q}$$

여기서:
- $-\rho(\nabla\cdot\vec{V})$: 체적 팽창률
- $\dot{q}$: 열 발생
- $\nabla\cdot\vec{q} = -k\nabla^2 T$: 열전도 유출

### 점성 소산 함수 $\Phi$

$$\Phi = 2\left[\left(\frac{\partial u}{\partial x}\right)^2 + \left(\frac{\partial v}{\partial y}\right)^2 + \left(\frac{\partial w}{\partial z}\right)^2\right] + \left[\left(\frac{\partial v}{\partial x}+\frac{\partial u}{\partial y}\right)^2 + \left(\frac{\partial w}{\partial y}+\frac{\partial v}{\partial z}\right)^2 + \left(\frac{\partial u}{\partial z}+\frac{\partial w}{\partial x}\right)^2\right]-\frac{2}{3}\left(\frac{\partial u}{\partial x}+\frac{\partial v}{\partial y}+\frac{\partial w}{\partial z}\right)^2$$


**비압축성 유체** ($e = cT$, $\nabla\cdot\vec{V}=0$):

$$\boxed{\rho C\frac{DT}{Dt} = k\nabla^2 T + \mu\Phi} \qquad (\Phi \geq 0)$$

---

## ⑨ 유체역학에서의 전형적인 해석 방법

외력을 알고 있을 때 → 유체의 운동 예측

### 두 가지 접근법

1. **유한 체적 해석 (Finite Volume Analysis)**
   - 적분 방정식 사용
   - 물리 법칙 (이동하는 검사체적, 물질)
   - 레이놀즈 수송 정리
   - 검사체적 고정, 모이중압, 내부 각 항 계산

2. **무한소 체적 해석 (Infinite Volume Analysis)**
   - 인접한 유체 입자에 의한 외력
   - 입자 간 상호작용을 기술하는 미분 방정식
   - 풀기 위해 경계 조건 필요

### 비압축성 뉴턴 유체 지배 방정식 요약

| 번호 | 방정식 | 이름 |
|------|--------|------|
| ① | $\nabla\cdot\vec{V}=0$ | 연속 방정식 |
| ② | $\rho\frac{D\vec{V}}{Dt} = -\nabla p + \mu\nabla^2\vec{V} + \rho\vec{g}$ | 나비에-스토크스 |
| ③ | $\rho C\frac{DT}{Dt} = k\nabla^2 T + \mu\Phi$ | 에너지 방정식 |

- $\mu$ = 상수 가정 시: ①②를 풀면 → ③ 풀기 가능
- 많은 경우 $k\nabla^2 T \gg \mu\Phi$ → ③은 ①②와 완전히 분리(decouple)

---

## §6.4 나비에-스토크스 방정식 해석

$$\rho\left(\frac{\partial\vec{V}}{\partial t} + \underbrace{(\vec{V}\cdot\nabla)\vec{V}}_{\text{① 비선형}}\right) = -\nabla p + \mu\nabla^2\vec{V} + \rho\vec{g}$$

비선형 미분 방정식 → **일반 해석해(analytic solution) 존재하지 않음**  
⇒ **전산유체역학(CFD) 필요**

### 레이놀즈 수 (Reynolds Number)

$$\frac{\text{①}}{\text{②}} = \frac{\rho(\vec{V}\cdot\nabla)\vec{V}}{\mu\nabla^2\vec{V}} \sim \frac{\rho V/L}{\mu/L^2} \sim \frac{\rho V L}{\mu} \equiv Re$$

레이놀즈 수 $Re$는 **대류 관성력과 점성력의 비**를 나타내는 무차원 매개변수.

| 영역 | 조건 | 단순화 |
|------|------|--------|
| 저 Re | $Re \ll 1$ (① $\ll$ ②) | $\rho\frac{\partial\vec{V}}{\partial t} = -\nabla p + \mu\nabla^2\vec{V} + \rho\vec{g}$ → 선형 PDE → 해석해 가능 |
| 고 Re | $Re \gg 1$ (① $\gg$ ③) | $\rho\left[\frac{\partial\vec{V}}{\partial t} + (\vec{V}\cdot\nabla)\vec{V}\right] = -\nabla p + \rho\vec{g}$ → **포텐셜 유동** ($\vec{V} = \nabla\phi$) |

---

## §6.4 비점성 유동 (Inviscid Flow)

유동장에서 전단 응력이 상대적으로 무시 가능한 경우,  
$\Rightarrow \mu\frac{\partial u_i}{\partial x_j} \approx 0$

유동장을 **"비점성"**, **"비마찰"**, **"마찰 없음(frictionless)"** 이라고 한다.

### ① 오일러 방정식 (Euler Equation)

$$\rho\frac{D\vec{V}}{Dt} = -\nabla p + \rho\vec{g}$$

### ② 베르누이 방정식 (Bernoulli Equation)

유선(streamline)을 따라 이동하는 유체 입자에 뉴턴 제2법칙 적용:

$$\boxed{P + \frac{1}{2}\rho V^2 + \rho g z = \text{상수} \quad \text{(유선을 따라)}}$$

**유도** (비압축성, 정상 상태):

벡터 항등식: $(\vec{V}\cdot\nabla)\vec{V} = \frac{1}{2}\nabla(\vec{V}\cdot\vec{V}) - \vec{V}\times(\nabla\times\vec{V})$를 이용하면

$$\frac{\nabla p}{\rho} + \frac{1}{2}\nabla V^2 + \rho\nabla z = \vec{V}\times(\nabla\times\vec{V})$$

유선 방향 미소 길이 $d\vec{s}$와의 내적을 취하면:

$$\frac{1}{\rho}dp + \frac{1}{2}dV^2 + g\,dz = 0$$

$$\Rightarrow \int\frac{1}{\rho}dp + \frac{1}{2}V^2 + gz = \text{상수}$$

---

## §6.5 비회전 유동의 특성 (Irrotational Flow)

- **와도(vorticity) = 0**: $\nabla\times\vec{V} = 0$
- 물리적 의미: 유체 입자의 **국소 회전** (각 변형이 아님)
- 전단 응력 없음 → 와도 생성 없음
  (중력과 압력은 와도를 생성할 수 없음)

> "비점성 유체에서, 유동장의 일부가 비회전이라면, 해당 영역에서 출발한 유체 요소는 비회전 상태를 유지한다"

### 속도 포텐셜 $\phi$

비회전 유동에서 $\vec{V} = (u, v, w)$는 다음과 같이 표현 가능:

$$\vec{V} = \nabla\phi = \left(\frac{\partial\phi}{\partial x}, \frac{\partial\phi}{\partial y}, \frac{\partial\phi}{\partial z}\right) \quad \text{($\phi = \phi(x,y,z,t)$)}$$

질량 보존에 대입:

$$\nabla\cdot\vec{V} = 0 = \nabla\cdot(\nabla\phi) = \nabla^2\phi$$

**비점성, 비압축성, 비회전 유동:**

$$\boxed{\nabla^2\phi = 0} \quad \text{(라플라스 방정식)}$$

---

## §6.5 기본 2D 포텐셜 유동

$$\nabla^2\phi = 0 \begin{cases} u = \frac{\partial\phi}{\partial x}, \; v = -\frac{\partial\phi}{\partial y} \\ V_r = \frac{\partial\phi}{\partial r}, \; V_\theta = \frac{1}{r}\frac{\partial\phi}{\partial\theta} \end{cases}$$

$$\nabla^2\psi = 0 \begin{cases} u = \frac{\partial\psi}{\partial y}, \; v = -\frac{\partial\psi}{\partial x} \\ V_r = \frac{1}{r}\frac{\partial\psi}{\partial\theta}, \; V_\theta = -\frac{\partial\psi}{\partial r} \end{cases}$$

### 유선 함수 (Stream Function) §6.2.3

2D 비압축성 유동에서 유선은 $\psi(x,y) = C$로 표현 가능.

유선을 따라: $d\psi = \frac{\partial\psi}{\partial x}dx + \frac{\partial\psi}{\partial y}dy = 0$

**정의:** $\frac{\partial\psi}{\partial y} = u$, $-\frac{\partial\psi}{\partial x} = v$로 놓으면

- 질량 보존: $\nabla\cdot\vec{V} = \frac{\partial u}{\partial x} + \frac{\partial v}{\partial y} = \frac{\partial}{\partial x}\left(\frac{\partial\psi}{\partial y}\right) + \frac{\partial}{\partial y}\left(-\frac{\partial\psi}{\partial x}\right) = 0$ ✓
- 비회전성: $\frac{\partial v}{\partial x} - \frac{\partial u}{\partial y} = 0 \Rightarrow \frac{\partial^2\psi}{\partial x^2} + \frac{\partial^2\psi}{\partial y^2} = \nabla^2\psi = 0$ ✓

### 도식적 표현: 등포텐셜선 ⊥ 유선

등포텐셜선($\phi = \text{const}$)을 따라:

$$\frac{dy}{dx}\bigg|_{\text{포텐셜}} = -\frac{u}{v} \Rightarrow \frac{dy}{dx}\bigg|_{\text{포텐셜}} \cdot \frac{dy}{dx}\bigg|_{\text{유선}} = -1$$

$$\phi(x,y) \leftrightarrow \vec{V}(x,y) \leftrightarrow \psi(x,y)$$

---

### 균일 유동 (Uniform Flow)

$$\phi = U(x\cos\alpha + y\sin\alpha), \quad \psi = U(y\cos\alpha - x\sin\alpha)$$

$$u = \frac{\partial\phi}{\partial x} = U\cos\alpha, \quad v = \frac{\partial\psi}{\partial y} = U\sin\alpha$$

### 소스 (Source) / 싱크 (Sink)

$$\phi = \frac{m}{2\pi}\ln r, \quad \psi = \frac{m}{2\pi}\theta$$

$$V_r = \frac{m}{2\pi r}, \quad V_\theta = 0$$

- $m$: 소스/싱크의 세기
- $m > 0$: 소스 유동
- $m < 0$: 싱크 유동

### 와류 (Vortex)

$$\phi = \frac{\Gamma}{2\pi}\theta, \quad \psi = -\frac{\Gamma}{2\pi}\ln r$$

$$V_r = 0, \quad V_\theta = \frac{\Gamma}{2\pi r}$$

$\Gamma$: **순환(circulation)** $\Gamma \equiv \oint_C \vec{V}\cdot d\vec{s}$

- **비회전 와류**: 와도 = 0, $V_\theta \propto \frac{1}{r}$
- **회전 와류**: 와도 ≠ 0, $V_\theta \propto r$ → 속도 포텐셜 구할 수 없음 ($\phi$ ✗)

### 이중극 (Doublet)

$$\phi = \frac{k\cos\theta}{r}, \quad \psi = -\frac{k\sin\theta}{r}$$

$$V_r = -\frac{k\cos\theta}{r^2}, \quad V_\theta = -\frac{k\sin\theta}{r^2}$$

(소스-싱크 쌍에서 $a \to 0$, $m \to 0$, $ma = \text{const}$인 극한 경우)

---

## §6.6 기본 포텐셜 유동의 중첩

비점성 유동에서 속도 벡터는 **고체 경계와 유선 모두에 접선 방향**이다.

⇒ 속도 포텐셜들을 조합하여 특정 관심 물체 형상에 해당하는 유선을 만들 수 있다면, 그 조합은 해당 물체 주변의 **비점성 유동을 기술**한다.

### ① 반체 (Half Body)

$$\phi = Ur\cos\theta + \frac{m}{2\pi}\ln r \quad \text{(균일 유동 + 소스)}$$

압력: $P(x,y) + \frac{1}{2}\rho|\vec{V}(x,y)|^2 = P_0 + \frac{1}{2}\rho U^2$

정체점(stagnation point): $\vec{V} = 0$

### ② 랭킨 타원체 (Rankine Ovals)

$$\psi = Uy - \frac{m}{2\pi}\tan^{-1}\left(\frac{2ay}{x^2+y^2-a^2}\right)$$

(균일 유동 + 소스-싱크 쌍 → 닫힌 물체)

### ③ 원형 실린더 주위 유동

$$\phi = U\cdot r\left(1+\frac{a^2}{r^2}\right)\cos\theta \quad \text{(균일 유동 + 이중극)}$$

$$V_r = U\left(1-\frac{a^2}{r^2}\right)\cos\theta, \quad V_\theta = -U\left(1+\frac{a^2}{r^2}\right)\sin\theta$$

표면($r=a$)에서: $V_{r,s}=0$, $V_{\theta,s} = -2U\sin\theta$

$$P_s = P_0 + \frac{1}{2}\rho U^2(1-4\sin^2\theta)$$

- **항력**: $F_x = -\int_0^{2\pi}P_s\cos\theta\,ds = 0$ → **달랑베르 역설 (D'Alembert's paradox)**
- **양력**: $F_y \approx 0$

### ④ 회전하는 실린더 주위 유동 (마그누스 효과)

$$\phi = Ur\left(1+\frac{a^2}{r^2}\right)\cos\theta + \frac{\Gamma}{2\pi}\theta \quad \text{(균일 유동 + 이중극 + 비회전 와류)}$$

표면에서:

$$V_{\theta,s} = -2U\sin\theta + \frac{\Gamma}{2\pi a}$$

정체점의 위치는 $\frac{\Gamma}{4\pi Ua}$에 따라 결정:
- $< 1$: 실린더 위 두 개의 정체점
- $= 1$: 아래쪽에서 하나로 합쳐짐
- $> 1$: 정체점이 실린더를 벗어남

$$F_x = 0, \quad F_y = -\rho U\Gamma \quad \text{(마그누스 효과)}$$

---

## 포텐셜 유동 이론의 한계 및 주의사항

1. 주어진 물체의 유선 함수를 구하는 방법은 여러 가지가 있다 (복소 변수 이론).

2. 포텐셜 유동 이론은 점성 효과가 작은 **"가속" 유동**에 대한 근사치만 제공한다.  
   표면 근처의 감속 유동은 **유동 박리(flow separation)**로 이어진다.

3. 실제 점성 유동은 나중에 공부할 **경계층 근사**와 결합된 포텐셜 유동 이론으로 이해할 수 있다.
