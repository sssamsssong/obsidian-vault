# 오도메트리 캘리브레이션

> YAX F1TENTH 교육자료 | Phase 3: Vehicle Control
> 
> 정확한 위치 추정을 위한 캘리브레이션

---

## 1. 개요

### 학습 목표
1. 오도메트리의 원리를 이해할 수 있다
2. 휠 오도메트리를 캘리브레이션할 수 있다
3. 오도메트리 오차를 측정하고 보정할 수 있다

### 예상 소요 시간
- 전체: 1.5시간

### 전제 조건
- VESC 드라이버 동작
- 수동 조종 가능

---

## 2. 오도메트리란?

### 2.1 정의

오도메트리(Odometry)는 휠 회전을 측정하여 로봇의 이동 거리와 위치를 추정하는 방법입니다.

### 2.2 VESC 오도메트리 소스

- **Tachometer**: 모터 회전 측정 (전기적 회전 수)
- **Hall Sensors / Sensorless**: 위치 피드백

### 2.3 오도메트리 메시지

```bash
ros2 interface show nav_msgs/msg/Odometry
```

```
std_msgs/Header header
string child_frame_id
geometry_msgs/PoseWithCovariance pose    # 위치 및 방향
geometry_msgs/TwistWithCovariance twist  # 속도
```

---

## 3. 캘리브레이션 파라미터

### 3.1 주요 파라미터

| 파라미터 | 설명 | 단위 |
|----------|------|------|
| `speed_to_erpm_gain` | 속도 → ERPM 변환 | ERPM/(m/s) |
| `tachometer_ticks_to_meters` | 타코미터 틱 → 거리 | m/tick |
| `wheelbase` | 휠베이스 | m |

### 3.2 계산

```python
# 기본 파라미터
wheel_diameter = 0.1      # 휠 직경 (m)
motor_pole_pairs = 7      # 모터 극 쌍 수
gear_ratio = 2.6          # 기어비
tachometer_ticks_per_rev = 42  # 1회전당 틱 수

# 휠 둘레
wheel_circumference = np.pi * wheel_diameter  # 0.314 m

# 모터 1회전당 휠 이동 거리
meters_per_motor_rev = wheel_circumference / gear_ratio

# 틱당 이동 거리
ticks_per_motor_rev = tachometer_ticks_per_rev * motor_pole_pairs
meters_per_tick = meters_per_motor_rev / ticks_per_motor_rev
# 약 0.00043 m/tick
```

---

## 4. 직선 주행 캘리브레이션

### 4.1 절차

1. 정확히 측정된 직선 거리 준비 (예: 5m)
2. 시작점에서 오도메트리 리셋
3. 직선 주행
4. 실제 거리와 오도메트리 비교
5. 보정 계수 계산

### 4.2 측정 방법

```python
#!/usr/bin/env python3
"""
Odometry Calibration Tool - Linear
"""

import rclpy
from rclpy.node import Node
from nav_msgs.msg import Odometry
import numpy as np


class LinearCalibration(Node):
    def __init__(self):
        super().__init__('linear_calibration')
        
        self.sub = self.create_subscription(
            Odometry, '/odom', self.odom_callback, 10
        )
        
        self.start_x = None
        self.start_y = None
        self.current_x = 0.0
        self.current_y = 0.0
        
        self.get_logger().info('Linear calibration ready')
        self.get_logger().info('Press Enter to start, then again to stop')
    
    def odom_callback(self, msg):
        self.current_x = msg.pose.pose.position.x
        self.current_y = msg.pose.pose.position.y
    
    def start_measurement(self):
        self.start_x = self.current_x
        self.start_y = self.current_y
        self.get_logger().info(f'Start: ({self.start_x:.3f}, {self.start_y:.3f})')
    
    def end_measurement(self):
        dx = self.current_x - self.start_x
        dy = self.current_y - self.start_y
        distance = np.sqrt(dx**2 + dy**2)
        self.get_logger().info(f'End: ({self.current_x:.3f}, {self.current_y:.3f})')
        self.get_logger().info(f'Measured distance: {distance:.3f} m')
        return distance


def main():
    rclpy.init()
    node = LinearCalibration()
    
    input('Press Enter to start measurement...')
    node.start_measurement()
    
    input('Drive straight, then press Enter to stop...')
    measured = node.end_measurement()
    
    actual = float(input('Enter actual distance (m): '))
    
    correction = actual / measured
    print(f'\nCorrection factor: {correction:.4f}')
    print(f'Multiply tachometer_ticks_to_meters by {correction:.4f}')
    
    node.destroy_node()
    rclpy.shutdown()


if __name__ == '__main__':
    main()
```

### 4.3 보정 적용

```yaml
vesc_driver:
  ros__parameters:
    # 기존 값 × 보정 계수
    tachometer_ticks_to_meters_gain: 0.000065  # 조정된 값
```

---

## 5. 회전 캘리브레이션

### 5.1 절차

1. 제자리에서 360도 회전
2. 오도메트리의 회전 각도 확인
3. 휠베이스 또는 트랙 폭 보정

### 5.2 측정 방법

```python
#!/usr/bin/env python3
"""
Odometry Calibration Tool - Angular
"""

import rclpy
from rclpy.node import Node
from nav_msgs.msg import Odometry
from tf_transformations import euler_from_quaternion
import numpy as np


class AngularCalibration(Node):
    def __init__(self):
        super().__init__('angular_calibration')
        
        self.sub = self.create_subscription(
            Odometry, '/odom', self.odom_callback, 10
        )
        
        self.start_yaw = None
        self.current_yaw = 0.0
        self.total_rotation = 0.0
        self.last_yaw = None
    
    def odom_callback(self, msg):
        q = msg.pose.pose.orientation
        _, _, yaw = euler_from_quaternion([q.x, q.y, q.z, q.w])
        
        if self.last_yaw is not None:
            # 각도 차이 계산 (래핑 처리)
            diff = yaw - self.last_yaw
            if diff > np.pi:
                diff -= 2 * np.pi
            elif diff < -np.pi:
                diff += 2 * np.pi
            self.total_rotation += diff
        
        self.last_yaw = yaw
        self.current_yaw = yaw
    
    def start_measurement(self):
        self.start_yaw = self.current_yaw
        self.total_rotation = 0.0
        self.get_logger().info(f'Start yaw: {np.degrees(self.start_yaw):.1f}°')
    
    def end_measurement(self):
        degrees = np.degrees(self.total_rotation)
        self.get_logger().info(f'Total rotation: {degrees:.1f}°')
        return degrees


def main():
    rclpy.init()
    node = AngularCalibration()
    
    input('Press Enter to start rotation measurement...')
    node.start_measurement()
    
    input('Rotate 360°, then press Enter...')
    measured = node.end_measurement()
    
    actual = 360.0
    correction = actual / abs(measured)
    print(f'\nAngular correction factor: {correction:.4f}')
    
    node.destroy_node()
    rclpy.shutdown()
```

---

## 6. 휠베이스 측정

### 6.1 물리적 측정

```
        ┌────────────────────┐
        │                    │
        │                    │
   ●────┤                    ├────●  ← 전륜 축
        │                    │
        │         L          │  ← 휠베이스
        │                    │
   ●────┤                    ├────●  ← 후륜 축
        │                    │
        └────────────────────┘

L = 전륜 축 중심에서 후륜 축 중심까지 거리
```

### 6.2 F1TENTH 기본값

```python
WHEELBASE = 0.324  # m (약 32.4 cm)
TRACK_WIDTH = 0.22  # m (약 22 cm)
```

### 6.3 오차 원인

- 측정 오차
- 서스펜션 처짐
- 타이어 마모
- 노면 조건

---

## 7. 오도메트리 정확도 테스트

### 7.1 사각형 주행 테스트

```
    Start/End
        ●─────────────────●
        │                 │
        │                 │
    2m  │                 │
        │                 │
        │                 │
        ●─────────────────●
              2m
```

시작점으로 정확히 돌아오는지 확인.

### 7.2 오차 측정

```python
# 시작점과 끝점 비교
x_error = end_x - start_x
y_error = end_y - start_y
theta_error = end_theta - start_theta

position_error = np.sqrt(x_error**2 + y_error**2)
```

### 7.3 허용 오차

- 직선 주행: < 2% 거리 오차
- 회전: < 5° / 360° 오차
- 사각형 복귀: < 10cm 위치 오차

---

## 8. 파라미터 조정

### 8.1 설정 파일

```yaml
vesc_driver:
  ros__parameters:
    # 캘리브레이션된 값
    speed_to_erpm_gain: 4614.0
    speed_to_erpm_offset: 0.0
    
    # 오도메트리
    tachometer_ticks_to_meters_gain: 0.000065  # 캘리브레이션 결과
    
    # 프레임
    odom_frame: "odom"
    base_frame: "base_link"
    
    publish_odom: true
    publish_tf: true
```

### 8.2 TF 발행

```
      [odom]
         │
         ▼
    [base_link]
```

---

## 9. 검증 체크리스트

- [ ] 직선 주행 오차 < 2%
- [ ] 360° 회전 오차 < 5°
- [ ] 사각형 복귀 오차 < 10cm
- [ ] `/odom` 토픽 정상 발행
- [ ] RViz2에서 TF 확인

---

*정확한 오도메트리는 자율주행의 기반입니다!*
