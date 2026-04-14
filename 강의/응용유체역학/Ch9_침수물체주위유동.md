# Ch9. 침수 물체 주위 유동 (Flow over Immersed Bodies)

## 외부 유동 (External Flows)

- 관심 대상 물체가 유체에 **완전히 둘러싸인** 유동
- 예시: 자동차·항공기 주위 공기 유동(공기역학), 선박 주위 물 유동, 건물 주위 바람
- 이론적(제한적), 실험적, 수치적 접근을 통해 잠수된 물체에 작용하는 **항력(drag)** 과 **양력(lift)** 예측이 목표
- 본 수업에서 $U$는 균일(uniform)하고 정상(steady)한 유동으로 가정

---

## §9.1 일반적인 외부 유동 특성

### 항력과 양력

물체 표면에는 두 가지 응력이 작용:
- **벽면 전단 응력** $\tau_w$: 점성 마찰에 의해 발생
- **압력** $p$: 물체 형상에 의한 압력 분포

**항력 (Drag force):**

$$D = \int (p\cos\theta + \tau_w \sin\theta)\,dA$$

**항력 계수:**

$$C_D = \frac{D}{\frac{1}{2}\rho U^2 \cdot A} \qquad (A: \text{전면적, frontal area})$$

**양력 (Lift force):**

$$L = \int (-p\sin\theta + \tau_w \cos\theta)\,dA$$

**양력 계수:**

$$C_L = \frac{L}{\frac{1}{2}\rho U^2 \cdot A} \qquad (A: \text{평면적, planform area})$$

---

### 전면적 vs 평면적

| 면적 종류 | 정의 | 사용 |
|----------|------|------|
| 전면적 (Frontal area) | 유동 방향에서 본 투영 면적 | 항력 계수 |
| 평면적 (Planform area) | 위에서 본 투영 면적 (날개 등) | 양력 계수 |

---

### 레이놀즈 수에 따른 유동 특성 의존성

$$Re = \frac{\rho U \ell}{\mu} \qquad (\ell: \text{특성 길이})$$

**예시 — 자동차 주위 유동:**
- $U \approx 36\,\text{km/h} = 10\,\text{m/s}$
- $\rho_{\text{air}} \approx 1\,\text{kg/m}^3$, $\mu \approx 65\,\text{Pa·s}$ (주: 실제 공기 $\mu \approx 1.8\times10^{-5}$)
- $\Rightarrow Re \approx 10^6$ → **가장 친숙한 외부 유동 영역** ($Re \gg 1$)

**입자·세포 등 소형 물체:** $Re \ll 1$ (Stokes 영역)
