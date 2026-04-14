# Ackermann 조향 이론

> YAX F1TENTH 교육자료 | Phase 3: Vehicle Control
> 
> 차량 조향의 기초 이론

---

## 1. 개요

### 학습 목표
1. Ackermann 조향 기하학을 이해할 수 있다
2. 조향각과 회전 반경의 관계를 계산할 수 있다
3. Bicycle 모델을 이해할 수 있다
4. 조향 제한을 고려한 제어를 구현할 수 있다

### 예상 소요 시간
- 전체: 1시간

### 전제 조건
- 기본 삼각함수 이해
- Python NumPy 기초

---

## 2. Ackermann 조향이란?

### 2.1 개념

Ackermann 조향은 차량이 회전할 때 모든 바퀴가 동일한 중심점을 기준으로 회전하도록 하는 조향 기하학입니다.

### 2.2 왜 필요한가?

```
    잘못된 조향 (평행)          올바른 조향 (Ackermann)
    
    ┌──────────┐               ┌──────────┐
    │←  ←      │               │←    ↖    │
    │          │               │          │
    └──────────┘               └──────────┘
    
    바퀴가 미끄러짐             모든 바퀴가 같은 중심으로
                               회전 → 미끄러짐 없음
```

### 2.3 기하학

```
                    L (휠베이스)
         ┌───────────────────────┐
         │                       │
    ┌────┤                       ├────┐
    │ δi │         *             │ δo │
    └────┤       (ICR)           ├────┘
         │         │             │
         │         │ R           │
    ●────┤         │             ├────●
         │         │             │
         └─────────┴─────────────┘
                   ↑
              회전 중심 (ICR)

    δi: 내측 바퀴 조향각 (더 큼)
    δo: 외측 바퀴 조향각 (더 작음)
    R: 회전 반경
    L: 휠베이스
```

---

## 3. 수학적 관계

### 3.1 기본 공식

**평균 조향각과 회전 반경**:

```
R = L / tan(δ)

여기서:
R: 회전 반경 (m)
L: 휠베이스 (m)
δ: 평균 조향각 (rad)
```

### 3.2 Python 구현

```python
import numpy as np

class AckermannModel:
    def __init__(self, wheelbase=0.324):
        """
        Args:
            wheelbase: 전륜과 후륜 사이 거리 (m)
        """
        self.L = wheelbase
    
    def get_turn_radius(self, steering_angle):
        """조향각으로부터 회전 반경 계산."""
        if abs(steering_angle) < 0.001:
            return float('inf')  # 직진
        return self.L / np.tan(steering_angle)
    
    def get_steering_angle(self, turn_radius):
        """회전 반경으로부터 조향각 계산."""
        if abs(turn_radius) < 0.001:
            return 0.0
        return np.arctan(self.L / turn_radius)
```

### 3.3 F1TENTH 파라미터

```python
# F1TENTH 기본 파라미터
WHEELBASE = 0.324      # 휠베이스 (m)
TRACK_WIDTH = 0.22     # 트랙 폭 (m)
MAX_STEERING = 0.4189  # 최대 조향각 ≈ 24° (rad)
```

---

## 4. Bicycle 모델

### 4.1 단순화

Bicycle 모델은 Ackermann 차량을 단순화한 모델입니다. 전륜 2개를 1개로, 후륜 2개를 1개로 취급합니다.

```
            δ (조향각)
            │
      ●─────┤  ← 전륜 (가상)
            │
            │ L (휠베이스)
            │
      ●─────┘  ← 후륜 (가상)
```

### 4.2 운동학 방정식

```
ẋ = v × cos(θ)
ẏ = v × sin(θ)
θ̇ = v × tan(δ) / L

여기서:
(x, y): 후륜 중심 위치
θ: 차량 방향 (yaw)
v: 속도
δ: 조향각
L: 휠베이스
```

### 4.3 Python 구현

```python
class BicycleModel:
    def __init__(self, wheelbase=0.324, dt=0.01):
        self.L = wheelbase
        self.dt = dt
        
        # 상태: [x, y, theta]
        self.state = np.array([0.0, 0.0, 0.0])
    
    def update(self, speed, steering_angle):
        """상태 업데이트 (오일러 적분)."""
        x, y, theta = self.state
        
        # 운동학 방정식
        x_dot = speed * np.cos(theta)
        y_dot = speed * np.sin(theta)
        theta_dot = speed * np.tan(steering_angle) / self.L
        
        # 적분
        self.state[0] += x_dot * self.dt
        self.state[1] += y_dot * self.dt
        self.state[2] += theta_dot * self.dt
        
        # theta 정규화 (-π ~ π)
        self.state[2] = np.arctan2(
            np.sin(self.state[2]),
            np.cos(self.state[2])
        )
        
        return self.state.copy()
    
    def get_position(self):
        return self.state[:2]
    
    def get_heading(self):
        return self.state[2]
```

---

## 5. 조향 제한

### 5.1 물리적 제한

```python
# F1TENTH 조향 제한
MAX_STEERING_ANGLE = np.radians(24)  # ±24°

def clip_steering(steering_angle):
    """조향각을 물리적 한계 내로 제한."""
    return np.clip(
        steering_angle,
        -MAX_STEERING_ANGLE,
        MAX_STEERING_ANGLE
    )
```

### 5.2 최소 회전 반경

```python
# 최대 조향각에서의 최소 회전 반경
min_radius = WHEELBASE / np.tan(MAX_STEERING_ANGLE)
# 약 0.73m
```

### 5.3 속도에 따른 조향 제한

고속에서 급조향은 위험합니다.

```python
def get_max_steering_for_speed(speed, max_lateral_accel=5.0):
    """
    속도에 따른 최대 조향각 계산.
    횡가속도 = v² / R 제한
    """
    if speed < 0.1:
        return MAX_STEERING_ANGLE
    
    # 횡가속도 제한에서 최소 반경
    min_radius = (speed ** 2) / max_lateral_accel
    
    # 해당 반경을 위한 조향각
    max_steering = np.arctan(WHEELBASE / min_radius)
    
    return min(max_steering, MAX_STEERING_ANGLE)
```

---

## 6. 조향각 vs 서보 위치

### 6.1 변환

VESC에서 서보는 0.0~1.0 값을 받습니다.

```python
def steering_to_servo(steering_angle, 
                      servo_min=0.15, 
                      servo_max=0.85,
                      servo_center=0.5):
    """조향각(rad)을 서보 위치로 변환."""
    # 정규화: -MAX ~ +MAX → -1 ~ +1
    normalized = steering_angle / MAX_STEERING_ANGLE
    
    # 서보 범위로 매핑
    if normalized >= 0:
        servo = servo_center + normalized * (servo_max - servo_center)
    else:
        servo = servo_center + normalized * (servo_center - servo_min)
    
    return np.clip(servo, servo_min, servo_max)

def servo_to_steering(servo_position,
                      servo_min=0.15,
                      servo_max=0.85,
                      servo_center=0.5):
    """서보 위치를 조향각(rad)으로 변환."""
    if servo_position >= servo_center:
        normalized = (servo_position - servo_center) / (servo_max - servo_center)
    else:
        normalized = (servo_position - servo_center) / (servo_center - servo_min)
    
    return normalized * MAX_STEERING_ANGLE
```

---

## 7. 실습

### 실습 1: 회전 반경 계산

다양한 조향각에 대한 회전 반경을 계산하고 표로 정리하세요.

| 조향각 (도) | 조향각 (rad) | 회전 반경 (m) |
|------------|-------------|---------------|
| 5 | 0.087 | ? |
| 10 | 0.175 | ? |
| 15 | 0.262 | ? |
| 24 (max) | 0.419 | ? |

### 실습 2: 시뮬레이션

Bicycle 모델을 사용하여 원형 경로를 시뮬레이션하세요.

```python
model = BicycleModel()
speed = 1.0  # m/s
steering = np.radians(15)  # 15도

trajectory = []
for _ in range(1000):
    state = model.update(speed, steering)
    trajectory.append(state[:2].copy())

# 시각화
import matplotlib.pyplot as plt
traj = np.array(trajectory)
plt.plot(traj[:, 0], traj[:, 1])
plt.axis('equal')
plt.show()
```

---

## 8. 검증 체크리스트

- [ ] Ackermann 조향 원리 이해
- [ ] 조향각-회전반경 관계 이해
- [ ] Bicycle 모델 운동학 이해
- [ ] 조향 제한 필요성 이해
- [ ] 조향각-서보 변환 이해

---

*Ackermann 조향을 이해하면 차량 제어와 경로 계획의 기초가 됩니다!*
