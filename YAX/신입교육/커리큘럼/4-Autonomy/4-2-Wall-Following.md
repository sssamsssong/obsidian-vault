# Wall Following 알고리즘

> YAX F1TENTH 교육자료 | Phase 4: Autonomy
> 
> 벽 따라가기 알고리즘 구현

---

## 1. 개요

### 학습 목표
1. Wall Following 알고리즘의 원리를 이해할 수 있다
2. PID 제어기를 사용하여 벽과의 거리를 유지할 수 있다
3. 실차에서 안정적으로 동작하도록 튜닝할 수 있다

### 예상 소요 시간
- 전체: 2시간

### 전제 조건
- LiDAR 데이터 처리 이해
- PID 제어 기초

---

## 2. 알고리즘 개요

### 2.1 개념

벽과 일정한 거리를 유지하면서 주행합니다.

```
         벽
    ═════════════════════════
                              
         d_desired            
       ◄──────►               
                              
    ┌───────────┐             
    │  F1TENTH  │  ──►        
    └───────────┘             
                              
    ═════════════════════════
         벽
```

### 2.2 왜 Wall Following인가?

- 단순하고 직관적
- 지도 필요 없음
- 안정적인 동작
- 자율주행 입문에 적합

---

## 3. 거리 측정

### 3.1 두 점을 이용한 거리 계산

```
                 벽
    ═══════════════════════
           │
           │ b
           │
     ●─────┘
     a \
        \α
         \
          ● 차량
```

두 LiDAR 빔(a, b)을 사용하여 벽까지의 수직 거리를 계산합니다.

```python
def get_distance_to_wall(self, ranges, angle_a, angle_b):
    """
    두 빔을 사용하여 벽까지의 수직 거리 계산.
    
    Args:
        ranges: LiDAR 거리 배열
        angle_a: 첫 번째 빔 각도 (deg)
        angle_b: 두 번째 빔 각도 (deg), 보통 90도
    
    Returns:
        wall_distance: 벽까지 수직 거리
        alpha: 벽과의 각도
    """
    theta = np.radians(angle_b - angle_a)
    
    # 각도에 해당하는 인덱스
    idx_a = self.angle_to_index(np.radians(angle_a))
    idx_b = self.angle_to_index(np.radians(angle_b))
    
    a = ranges[idx_a]
    b = ranges[idx_b]
    
    # 벽과의 각도 계산
    alpha = np.arctan2(a * np.cos(theta) - b, a * np.sin(theta))
    
    # 현재 수직 거리
    wall_distance = b * np.cos(alpha)
    
    return wall_distance, alpha
```

### 3.2 미래 거리 예측

전방 lookahead 거리에서의 예상 벽 거리:

```python
def predict_distance(self, current_distance, alpha, lookahead):
    """
    lookahead 거리 후의 예상 벽 거리.
    """
    return current_distance + lookahead * np.sin(alpha)
```

---

## 4. PID 제어기

### 4.1 PID 기본

```
error = desired_distance - actual_distance

P: proportional (현재 오차에 비례)
I: integral (누적 오차에 비례)
D: derivative (오차 변화율에 비례)

steering = Kp * error + Ki * integral + Kd * derivative
```

### 4.2 구현

```python
class PIDController:
    def __init__(self, kp, ki, kd, min_out=-1.0, max_out=1.0):
        self.kp = kp
        self.ki = ki
        self.kd = kd
        self.min_out = min_out
        self.max_out = max_out
        
        self.integral = 0.0
        self.prev_error = 0.0
    
    def compute(self, error, dt):
        # Proportional
        p_term = self.kp * error
        
        # Integral (안티 와인드업)
        self.integral += error * dt
        self.integral = np.clip(self.integral, -1.0, 1.0)
        i_term = self.ki * self.integral
        
        # Derivative
        derivative = (error - self.prev_error) / dt if dt > 0 else 0.0
        d_term = self.kd * derivative
        
        self.prev_error = error
        
        # 출력 합산 및 제한
        output = p_term + i_term + d_term
        return np.clip(output, self.min_out, self.max_out)
    
    def reset(self):
        self.integral = 0.0
        self.prev_error = 0.0
```

---

## 5. Wall Following 노드

### 5.1 전체 구현

```python
#!/usr/bin/env python3
"""
Wall Following Node
"""

import rclpy
from rclpy.node import Node
from sensor_msgs.msg import LaserScan
from ackermann_msgs.msg import AckermannDriveStamped
import numpy as np


class WallFollowing(Node):
    def __init__(self):
        super().__init__('wall_following')
        
        # Parameters
        self.declare_parameter('desired_distance', 1.0)
        self.declare_parameter('lookahead', 1.0)
        self.declare_parameter('speed', 1.5)
        self.declare_parameter('side', 'right')  # 'left' or 'right'
        
        self.declare_parameter('kp', 1.0)
        self.declare_parameter('ki', 0.0)
        self.declare_parameter('kd', 0.1)
        
        self.desired_dist = self.get_parameter('desired_distance').value
        self.lookahead = self.get_parameter('lookahead').value
        self.speed = self.get_parameter('speed').value
        self.side = self.get_parameter('side').value
        
        kp = self.get_parameter('kp').value
        ki = self.get_parameter('ki').value
        kd = self.get_parameter('kd').value
        
        self.pid = PIDController(kp, ki, kd, min_out=-0.4, max_out=0.4)
        
        # LiDAR 파라미터 (초기화 시 업데이트)
        self.angle_min = 0
        self.angle_increment = 0
        
        # Timing
        self.prev_time = self.get_clock().now()
        
        # Subscriber
        self.scan_sub = self.create_subscription(
            LaserScan, '/scan', self.scan_callback, 10
        )
        
        # Publisher
        self.drive_pub = self.create_publisher(
            AckermannDriveStamped, '/drive', 10
        )
        
        self.get_logger().info(f'Wall following started: {self.side} side')
    
    def angle_to_index(self, angle_rad):
        """각도를 LiDAR 인덱스로 변환."""
        return int((angle_rad - self.angle_min) / self.angle_increment)
    
    def get_wall_distance(self, ranges):
        """벽까지의 거리와 각도 계산."""
        if self.side == 'right':
            angle_a = -45  # 우측 45도
            angle_b = -90  # 우측 90도
        else:  # left
            angle_a = 45   # 좌측 45도
            angle_b = 90   # 좌측 90도
        
        theta = np.radians(abs(angle_b - angle_a))
        
        idx_a = self.angle_to_index(np.radians(angle_a))
        idx_b = self.angle_to_index(np.radians(angle_b))
        
        # 유효 범위 체크
        if idx_a < 0 or idx_a >= len(ranges) or idx_b < 0 or idx_b >= len(ranges):
            return self.desired_dist, 0.0
        
        a = ranges[idx_a]
        b = ranges[idx_b]
        
        # inf 처리
        if np.isinf(a) or np.isinf(b):
            return self.desired_dist, 0.0
        
        # 벽과의 각도
        alpha = np.arctan2(a * np.cos(theta) - b, a * np.sin(theta))
        
        # 현재 수직 거리
        current_dist = b * np.cos(alpha)
        
        # 미래 거리 예측
        future_dist = current_dist + self.lookahead * np.sin(alpha)
        
        return future_dist, alpha
    
    def scan_callback(self, msg):
        # LiDAR 파라미터 업데이트
        self.angle_min = msg.angle_min
        self.angle_increment = msg.angle_increment
        
        # 전처리
        ranges = np.array(msg.ranges)
        ranges = np.where(np.isinf(ranges), msg.range_max, ranges)
        
        # 벽 거리 계산
        wall_dist, alpha = self.get_wall_distance(ranges)
        
        # 오차 계산
        error = self.desired_dist - wall_dist
        if self.side == 'right':
            error = -error  # 우측 벽: 반대 방향
        
        # 시간 계산
        current_time = self.get_clock().now()
        dt = (current_time - self.prev_time).nanoseconds / 1e9
        self.prev_time = current_time
        
        # PID 제어
        steering = self.pid.compute(error, dt)
        
        # 속도 조절 (조향각에 따라)
        speed = self.speed * (1.0 - abs(steering) / 0.4 * 0.3)
        
        # 발행
        drive_msg = AckermannDriveStamped()
        drive_msg.header.stamp = current_time.to_msg()
        drive_msg.header.frame_id = 'base_link'
        drive_msg.drive.speed = speed
        drive_msg.drive.steering_angle = steering
        
        self.drive_pub.publish(drive_msg)
        
        # 디버그 출력
        self.get_logger().debug(
            f'Dist: {wall_dist:.2f}, Error: {error:.2f}, Steer: {steering:.2f}'
        )


def main(args=None):
    rclpy.init(args=args)
    node = WallFollowing()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()


if __name__ == '__main__':
    main()
```

---

## 6. 파라미터 튜닝

### 6.1 PID 튜닝 가이드

| 증상 | 조치 |
|------|------|
| 반응이 느림 | Kp 증가 |
| 진동함 | Kp 감소, Kd 증가 |
| 오버슈트 | Kd 증가 |
| 정상상태 오차 | Ki 증가 (조심스럽게) |

### 6.2 권장 시작값

```yaml
wall_following:
  ros__parameters:
    desired_distance: 0.8    # 벽으로부터 0.8m
    lookahead: 1.0           # 1m 전방 예측
    speed: 1.0               # 1 m/s
    side: "right"            # 우측 벽
    
    kp: 1.0
    ki: 0.0
    kd: 0.1
```

### 6.3 단계별 튜닝

1. Ki = 0, Kd = 0 으로 시작
2. Kp를 진동 직전까지 증가
3. Kd를 추가하여 진동 억제
4. 필요시 Ki 소량 추가

---

## 7. 실습

### 실습 1: 기본 동작

1. MORAI 시뮬레이션에서 테스트
2. 우측 벽 따라가기 성공
3. 좌측 벽으로 전환

### 실습 2: 파라미터 실험

| 실험 | Kp | Ki | Kd | 결과 |
|------|----|----|----|----|
| 1 | 0.5 | 0 | 0 | |
| 2 | 1.0 | 0 | 0 | |
| 3 | 1.5 | 0 | 0 | |
| 4 | 1.0 | 0 | 0.1 | |
| 5 | 1.0 | 0 | 0.2 | |

### 실습 3: 코너 주행

- 급격한 코너에서의 동작 확인
- 필요시 속도 조절

---

## 8. 검증 체크리스트

- [ ] 벽 거리 계산 정확성
- [ ] PID 제어기 동작
- [ ] 직선 구간 안정적 주행
- [ ] 코너 통과
- [ ] 파라미터 튜닝 완료

---

*Wall Following은 자율주행의 첫 번째 도전입니다!*
