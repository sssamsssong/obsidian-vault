# MPC (Model Predictive Control) 소개

> YAX F1TENTH 교육자료 | Phase 4: Autonomy
> 
> 모델 예측 제어의 기초

---

## 1. 개요

### 학습 목표
1. MPC의 기본 개념과 동작 원리를 이해할 수 있다
2. 차량 동역학 모델의 종류를 구분할 수 있다
3. MPC가 기존 제어기보다 우수한 상황을 설명할 수 있다

### 예상 소요 시간
- 전체: 3시간 (개념 이해 중심)

### 전제 조건
- Pure Pursuit, Stanley Controller 완료
- 기본 선형대수 (행렬 연산)
- Python 중급

### 주의

> **이 문서는 MPC의 개념적 이해를 위한 입문 자료입니다.**
> 
> MPC는 고급 주제로, 실제 구현과 튜닝에는 상당한 수학적 배경과 경험이 필요합니다.
> F1TENTH 대회 참가를 위해 반드시 MPC가 필요한 것은 아닙니다.
> Pure Pursuit + FTG 조합으로도 우수한 성적을 거둘 수 있습니다.

---

## 2. MPC란?

### 2.1 기본 개념

**MPC (Model Predictive Control)** 는 미래를 **예측**하고 **최적화**하는 제어 기법입니다.

```
현재 ────────────────────────────────────────► 미래

  │                    예측 구간 (Horizon)
  │    ┌─────────────────────────────────┐
  │    │                                 │
  ●────●────●────●────●────●────●────●────●
  t    t+1  t+2  t+3  t+4  t+5  t+6  t+7  t+N
  │
  │ 현재 상태에서 N 스텝 미래까지 예측
  │ → 최적 입력 시퀀스 계산
  │ → 첫 번째 입력만 적용
  │ → 다음 시간에 반복 (Receding Horizon)
```

### 2.2 MPC vs 기존 제어기

| 특성 | PID / Pure Pursuit / Stanley | MPC |
|------|------------------------------|-----|
| 시간 관점 | 현재 오차만 고려 | 미래 N 스텝 예측 |
| 제약 조건 | 사후 처리 (클리핑) | 최적화에 포함 |
| 최적성 | 휴리스틱 | 수학적 최적 |
| 계산량 | 매우 낮음 | 높음 |
| 구현 난이도 | 쉬움 | 어려움 |

### 2.3 MPC의 장점

1. **제약 조건 직접 처리**
   - 조향각 한계
   - 속도 한계
   - 가속도 한계
   - 트랙 경계

2. **미래 예측**
   - 코너 미리 감속
   - 경로 이탈 예방

3. **다목표 최적화**
   - 경로 추종 + 속도 최대화 + 편안함

### 2.4 MPC의 단점

1. **계산 비용**: 실시간 최적화 필요
2. **모델 의존성**: 정확한 차량 모델 필요
3. **튜닝 복잡성**: 많은 파라미터
4. **수렴 보장 없음**: 비선형 문제에서

---

## 3. 차량 동역학 모델

MPC의 핵심은 **정확한 예측**이며, 이를 위해 차량 모델이 필요합니다.

### 3.1 Kinematic Bicycle Model (기구학 모델)

저속에서 적합한 단순 모델:

```
                    ● 전륜
                   /│
                  / │
                 /  │ L (휠베이스)
                /   │
               / β  │
          v  ↗      │
            ●───────┘ 후륜
           (x, y, θ)
```

**상태 변수:**
- `x, y`: 위치
- `θ`: 헤딩 (yaw)
- `v`: 속도

**입력:**
- `δ`: 조향각
- `a`: 가속도

**이산 시간 모델:**
```python
def kinematic_model(state, control, dt, L=0.33):
    """
    Kinematic Bicycle Model.
    
    state: [x, y, theta, v]
    control: [delta, a]
    """
    x, y, theta, v = state
    delta, a = control
    
    # 다음 상태 계산
    x_next = x + v * np.cos(theta) * dt
    y_next = y + v * np.sin(theta) * dt
    theta_next = theta + v / L * np.tan(delta) * dt
    v_next = v + a * dt
    
    return np.array([x_next, y_next, theta_next, v_next])
```

### 3.2 Dynamic Bicycle Model (동역학 모델)

고속, 드리프트 상황에서 필요:

```
                    Fyf (전륜 측력)
                     ↑
                    ● 전륜
                   /
                  / 슬립각
                 /
               ●───────→ Fxr (후륜 구동력)
              ↓
             Fyr (후륜 측력)
```

**추가 상태 변수:**
- `vy`: 측면 속도
- `r`: 요 레이트 (dθ/dt)

**추가 파라미터:**
- 타이어 강성 (Cf, Cr)
- 차량 질량 (m)
- 관성 모멘트 (Iz)
- 무게 중심 위치 (lf, lr)

### 3.3 모델 선택 가이드

| 상황 | 권장 모델 |
|------|-----------|
| 저속 (< 3 m/s) | Kinematic |
| 중속 (3-5 m/s) | Kinematic 가능 |
| 고속 (> 5 m/s) | Dynamic |
| 드리프트 | Dynamic 필수 |
| 실시간 제약 | Kinematic |

---

## 4. MPC 문제 정의

### 4.1 비용 함수 (Cost Function)

최소화하고자 하는 목표:

```
J = Σ (상태 비용 + 입력 비용 + 변화율 비용)
    t=0 to N

J = Σ [ Q*(x_t - x_ref)² + R*u_t² + Rd*(u_t - u_{t-1})² ]
```

**구성 요소:**

1. **상태 비용 (Q)**: 경로 추종 오차
   - 위치 오차: x, y 차이
   - 헤딩 오차: θ 차이
   - 속도 오차: v 차이

2. **입력 비용 (R)**: 제어 입력 크기
   - 조향각 δ
   - 가속도 a

3. **변화율 비용 (Rd)**: 입력 변화량
   - 급격한 조향 방지
   - 부드러운 주행

```python
def cost_function(states, controls, reference, Q, R, Rd):
    """
    MPC 비용 함수.
    """
    cost = 0.0
    N = len(states)
    
    for t in range(N):
        # 상태 오차
        state_error = states[t] - reference[t]
        cost += state_error.T @ Q @ state_error
        
        # 입력 비용
        cost += controls[t].T @ R @ controls[t]
        
        # 변화율 비용
        if t > 0:
            delta_u = controls[t] - controls[t-1]
            cost += delta_u.T @ Rd @ delta_u
    
    return cost
```

### 4.2 제약 조건 (Constraints)

**입력 제약:**
```python
# 조향각 제한
-0.4 <= delta <= 0.4  # rad

# 가속도 제한
-5.0 <= a <= 3.0      # m/s²
```

**상태 제약:**
```python
# 속도 제한
0.0 <= v <= 6.0       # m/s

# 트랙 경계 (예: 단순화)
y_min <= y <= y_max
```

**변화율 제약:**
```python
# 조향 변화율 제한
-0.2 <= delta_t - delta_{t-1} <= 0.2  # rad per step
```

### 4.3 최적화 문제

```
minimize    J(x, u)
subject to  x_{t+1} = f(x_t, u_t)    (동역학)
            u_min <= u <= u_max       (입력 제약)
            x_min <= x <= x_max       (상태 제약)
```

---

## 5. MPC 알고리즘 흐름

### 5.1 Receding Horizon 제어

```
시간 t=0:
  현재 상태 x_0 측정
  → N 스텝 미래 예측: x_1, x_2, ..., x_N
  → 최적 입력 시퀀스: u_0*, u_1*, ..., u_{N-1}*
  → 첫 번째 입력 u_0* 적용
  
시간 t=1:
  현재 상태 x_1 측정 (실제 값)
  → N 스텝 미래 예측: x_2, x_3, ..., x_{N+1}
  → 최적 입력 시퀀스: u_1*, u_2*, ..., u_N*
  → 첫 번째 입력 u_1* 적용
  
... 반복
```

### 5.2 의사 코드

```python
class MPC:
    def __init__(self, horizon, dt, model):
        self.N = horizon     # 예측 구간
        self.dt = dt         # 시간 간격
        self.model = model   # 차량 모델
        
    def solve(self, current_state, reference_trajectory):
        """
        MPC 최적화 문제 풀이.
        
        Returns:
            optimal_control: 최적 제어 입력 (첫 번째만 사용)
        """
        # 1. 초기 추측 (Warm start)
        initial_guess = self.get_initial_guess()
        
        # 2. 최적화 문제 설정
        problem = self.setup_problem(current_state, reference_trajectory)
        
        # 3. 풀이
        solution = self.optimizer.solve(problem, initial_guess)
        
        # 4. 첫 번째 입력 반환
        return solution.controls[0]
    
    def control_loop(self):
        """메인 제어 루프."""
        while running:
            # 현재 상태 측정
            state = self.get_current_state()
            
            # 참조 경로 생성
            reference = self.get_reference_trajectory(state)
            
            # MPC 풀이
            control = self.solve(state, reference)
            
            # 제어 입력 적용
            self.apply_control(control)
            
            # 대기
            time.sleep(self.dt)
```

---

## 6. 간소화된 MPC 구현

### 6.1 선형 MPC (Linear MPC)

비선형 모델을 현재 상태에서 **선형화**하여 사용:

```python
import numpy as np
from scipy.optimize import minimize

class SimpleMPC:
    def __init__(self):
        # 파라미터
        self.N = 10           # 예측 구간
        self.dt = 0.1         # 100ms
        self.L = 0.33         # 휠베이스
        
        # 가중치
        self.Q = np.diag([10.0, 10.0, 1.0, 1.0])  # 상태 [x, y, θ, v]
        self.R = np.diag([1.0, 0.1])              # 입력 [δ, a]
        self.Rd = np.diag([10.0, 1.0])            # 변화율
        
        # 제약
        self.delta_max = 0.4
        self.a_max = 3.0
        self.a_min = -5.0
        self.v_max = 5.0
    
    def predict(self, state, controls):
        """N 스텝 미래 예측."""
        states = [state]
        
        for u in controls:
            next_state = self.kinematic_model(states[-1], u)
            states.append(next_state)
        
        return np.array(states[1:])  # 현재 상태 제외
    
    def kinematic_model(self, state, control):
        """Kinematic Bicycle Model."""
        x, y, theta, v = state
        delta, a = control
        
        x_next = x + v * np.cos(theta) * self.dt
        y_next = y + v * np.sin(theta) * self.dt
        theta_next = theta + v / self.L * np.tan(delta) * self.dt
        v_next = np.clip(v + a * self.dt, 0, self.v_max)
        
        return np.array([x_next, y_next, theta_next, v_next])
    
    def cost(self, controls_flat, current_state, reference):
        """비용 함수."""
        controls = controls_flat.reshape(self.N, 2)
        
        # 예측
        predicted = self.predict(current_state, controls)
        
        cost = 0.0
        
        for t in range(self.N):
            # 상태 오차
            error = predicted[t] - reference[t]
            cost += error @ self.Q @ error
            
            # 입력 비용
            cost += controls[t] @ self.R @ controls[t]
            
            # 변화율 비용
            if t > 0:
                delta_u = controls[t] - controls[t-1]
                cost += delta_u @ self.Rd @ delta_u
        
        return cost
    
    def solve(self, current_state, reference):
        """최적화 문제 풀이."""
        # 초기 추측
        u0 = np.zeros(self.N * 2)
        
        # 제약 조건
        bounds = []
        for _ in range(self.N):
            bounds.append((-self.delta_max, self.delta_max))  # δ
            bounds.append((self.a_min, self.a_max))           # a
        
        # 최적화
        result = minimize(
            self.cost,
            u0,
            args=(current_state, reference),
            method='SLSQP',
            bounds=bounds,
            options={'maxiter': 50}
        )
        
        # 첫 번째 입력 반환
        optimal_controls = result.x.reshape(self.N, 2)
        return optimal_controls[0]
```

### 6.2 ROS2 노드

```python
#!/usr/bin/env python3
"""
Simple MPC Node (Educational Purpose)
"""

import rclpy
from rclpy.node import Node
from nav_msgs.msg import Odometry
from ackermann_msgs.msg import AckermannDriveStamped
import numpy as np
from tf_transformations import euler_from_quaternion


class SimpleMPCNode(Node):
    def __init__(self):
        super().__init__('simple_mpc')
        
        self.mpc = SimpleMPC()
        self.waypoints = self.load_waypoints('waypoints.csv')
        
        self.odom_sub = self.create_subscription(
            Odometry, '/odom', self.odom_callback, 10
        )
        
        self.drive_pub = self.create_publisher(
            AckermannDriveStamped, '/drive', 10
        )
        
        self.get_logger().info('Simple MPC started')
    
    def load_waypoints(self, filename):
        # 웨이포인트 로드 (생략)
        pass
    
    def get_reference_trajectory(self, current_pos):
        """
        현재 위치에서 N 스텝 앞의 참조 경로 생성.
        """
        reference = []
        
        # 가장 가까운 웨이포인트 찾기
        closest_idx = self.find_closest_waypoint(current_pos)
        
        for i in range(self.mpc.N):
            idx = (closest_idx + i) % len(self.waypoints)
            wp = self.waypoints[idx]
            
            # [x, y, θ, v] 형태의 참조 상태
            if idx + 1 < len(self.waypoints):
                dx = self.waypoints[idx+1, 0] - wp[0]
                dy = self.waypoints[idx+1, 1] - wp[1]
                theta_ref = np.arctan2(dy, dx)
            else:
                theta_ref = 0
            
            v_ref = wp[2] if len(wp) > 2 else 2.0
            
            reference.append([wp[0], wp[1], theta_ref, v_ref])
        
        return np.array(reference)
    
    def odom_callback(self, msg):
        # 현재 상태
        x = msg.pose.pose.position.x
        y = msg.pose.pose.position.y
        q = msg.pose.pose.orientation
        _, _, theta = euler_from_quaternion([q.x, q.y, q.z, q.w])
        v = msg.twist.twist.linear.x
        
        current_state = np.array([x, y, theta, v])
        
        # 참조 경로
        reference = self.get_reference_trajectory(np.array([x, y]))
        
        # MPC 풀이
        try:
            optimal_control = self.mpc.solve(current_state, reference)
            delta, a = optimal_control
        except Exception as e:
            self.get_logger().warn(f'MPC failed: {e}')
            delta, a = 0.0, 0.0
        
        # 속도 계산 (현재 속도 + 가속도 * dt)
        speed = max(0, v + a * self.mpc.dt)
        
        # 발행
        drive_msg = AckermannDriveStamped()
        drive_msg.header.stamp = self.get_clock().now().to_msg()
        drive_msg.drive.speed = float(speed)
        drive_msg.drive.steering_angle = float(delta)
        
        self.drive_pub.publish(drive_msg)
```

---

## 7. MPC 라이브러리

실제 프로젝트에서는 검증된 라이브러리 사용을 권장합니다.

### 7.1 CasADi

**특징:**
- 자동 미분
- 다양한 솔버 지원
- Python/C++ 인터페이스

```python
import casadi as ca

def create_mpc_problem():
    # 상태 및 입력 정의
    x = ca.MX.sym('x', 4)  # [x, y, θ, v]
    u = ca.MX.sym('u', 2)  # [δ, a]
    
    # 동역학 모델
    L = 0.33
    dt = 0.1
    
    x_next = ca.vertcat(
        x[0] + x[3] * ca.cos(x[2]) * dt,
        x[1] + x[3] * ca.sin(x[2]) * dt,
        x[2] + x[3] / L * ca.tan(u[0]) * dt,
        x[3] + u[1] * dt
    )
    
    f = ca.Function('f', [x, u], [x_next])
    
    return f
```

### 7.2 ACADOS

**특징:**
- 고성능 실시간 솔버
- 임베디드 시스템 지원
- ROS2 통합 용이

### 7.3 do-mpc

**특징:**
- Python 기반
- 교육 목적에 적합
- 쉬운 설정

```python
import do_mpc

# 모델 설정
model = do_mpc.model.Model('continuous')

# 상태 변수
x = model.set_variable('_x', 'x')
y = model.set_variable('_x', 'y')
theta = model.set_variable('_x', 'theta')
v = model.set_variable('_x', 'v')

# 입력 변수
delta = model.set_variable('_u', 'delta')
a = model.set_variable('_u', 'a')

# 동역학
L = 0.33
model.set_rhs('x', v * np.cos(theta))
model.set_rhs('y', v * np.sin(theta))
model.set_rhs('theta', v / L * np.tan(delta))
model.set_rhs('v', a)

model.setup()

# MPC 설정
mpc = do_mpc.controller.MPC(model)
```

---

## 8. 튜닝 가이드

### 8.1 주요 파라미터

| 파라미터 | 설명 | 영향 |
|----------|------|------|
| N (Horizon) | 예측 구간 | 길면 최적, 계산 증가 |
| dt | 시간 간격 | 작으면 정밀, 계산 증가 |
| Q | 상태 가중치 | 높으면 경로 추종 중시 |
| R | 입력 가중치 | 높으면 부드러운 제어 |
| Rd | 변화율 가중치 | 높으면 급격한 변화 억제 |

### 8.2 권장 시작값

```python
# F1TENTH 용
N = 10          # 10 스텝 (1초 예측)
dt = 0.1        # 100ms

Q = np.diag([
    10.0,   # x 위치
    10.0,   # y 위치
    1.0,    # 헤딩
    1.0     # 속도
])

R = np.diag([
    1.0,    # 조향
    0.1     # 가속도
])

Rd = np.diag([
    10.0,   # 조향 변화율
    1.0     # 가속도 변화율
])
```

### 8.3 튜닝 순서

1. **계산 시간 확인**: N, dt 조정하여 실시간 가능 여부 확인
2. **경로 추종**: Q의 x, y 가중치 조정
3. **부드러움**: Rd 증가로 급격한 입력 억제
4. **성능**: Q, R 미세 조정

---

## 9. MPC vs 기존 제어기 실습

### 실습 1: 계산 시간 측정

```python
import time

# 여러 N 값으로 계산 시간 측정
for N in [5, 10, 15, 20]:
    mpc = SimpleMPC()
    mpc.N = N
    
    start = time.time()
    for _ in range(100):
        mpc.solve(current_state, reference)
    elapsed = (time.time() - start) / 100
    
    print(f'N={N}: {elapsed*1000:.1f} ms per solve')
```

### 실습 2: 성능 비교

| 알고리즘 | 평균 CTE | 최대 CTE | 랩타임 | 계산시간 |
|----------|----------|----------|--------|----------|
| Pure Pursuit | | | | |
| Stanley | | | | |
| MPC (N=5) | | | | |
| MPC (N=10) | | | | |

### 실습 3: 제약 조건 실험

- 조향각 제한: ±0.4 → ±0.3 → ±0.2
- 속도 제한: 5.0 → 4.0 → 3.0
- 각 조건에서 경로 추종 품질 비교

---

## 10. 실제 적용 시 고려사항

### 10.1 계산 자원

| 플랫폼 | 권장 N | 솔버 |
|--------|--------|------|
| Jetson Orin Nano | 5-10 | IPOPT, OSQP |
| Jetson AGX Orin | 10-20 | ACADOS |
| PC (실시간 아님) | 20+ | CasADi |

### 10.2 Warm Start

이전 해를 초기 추측으로 사용하여 수렴 속도 향상:

```python
def solve_with_warmstart(self, current_state, reference):
    if self.last_solution is not None:
        # 이전 해 시프트
        initial_guess = np.roll(self.last_solution, -2)
        initial_guess[-2:] = initial_guess[-4:-2]  # 마지막 복사
    else:
        initial_guess = np.zeros(self.N * 2)
    
    result = self.optimize(initial_guess, current_state, reference)
    self.last_solution = result
    return result[0:2]
```

### 10.3 Fallback 전략

MPC 실패 시 대체 제어기:

```python
def control(self, state, reference):
    try:
        return self.mpc.solve(state, reference)
    except Exception:
        # MPC 실패 시 Stanley로 fallback
        return self.stanley.compute(state, reference)
```

---

## 11. 다음 단계

### MPC 심화 학습

1. **비선형 MPC (NMPC)**: 비선형 모델 직접 사용
2. **적응형 MPC**: 온라인 모델 파라미터 추정
3. **강건 MPC**: 불확실성 고려
4. **학습 기반 MPC**: Neural Network와 결합

### 추천 자료

- **교재**: "Model Predictive Control" by Rawlings & Mayne
- **강의**: MIT OCW 2.154J Control of Manufacturing Processes
- **튜토리얼**: do-mpc documentation
- **코드**: F1TENTH official repository (MPC examples)

---

## 12. 검증 체크리스트

- [ ] Kinematic model 이해
- [ ] 비용 함수 구성 요소 설명 가능
- [ ] 제약 조건 설정 이해
- [ ] 간소화된 MPC 코드 실행
- [ ] 계산 시간 측정
- [ ] Pure Pursuit/Stanley와 비교

---

## 13. 정리

### MPC 핵심 포인트

1. **예측**: 미래 N 스텝을 모델 기반으로 예측
2. **최적화**: 비용 함수 최소화
3. **제약**: 물리적 한계 직접 반영
4. **Receding Horizon**: 매 스텝 다시 풀이

### 언제 MPC를 사용할까?

| 상황 | 권장 |
|------|------|
| 빠른 개발 필요 | Pure Pursuit / Stanley |
| 높은 정밀도 필요 | MPC |
| 제한된 계산 자원 | Pure Pursuit / Stanley |
| 복잡한 제약 조건 | MPC |
| 고속 레이싱 | MPC + 동적 모델 |

---

*MPC는 자율주행의 최전선 기술입니다. 기초를 다진 후 점진적으로 심화하세요!*
