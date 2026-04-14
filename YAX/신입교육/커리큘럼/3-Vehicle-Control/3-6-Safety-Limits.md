# 안전 제한 설정

> YAX F1TENTH 교육자료 | Phase 3: Vehicle Control
> 
> 안전한 운행을 위한 제한 설정

---

## 1. 개요

### 학습 목표
1. 속도 및 조향 제한을 설정할 수 있다
2. 비상 정지 시스템을 구현할 수 있다
3. 배터리 및 온도 모니터링을 할 수 있다
4. 안전한 테스트 절차를 수립할 수 있다

### 예상 소요 시간
- 전체: 1시간

### 전제 조건
- VESC 드라이버 설정 완료
- 수동 조종 경험

---

## 2. 속도 제한

### 2.1 VESC Tool에서 설정

1. VESC Tool 실행
2. Motor Settings > General > Max ERPM
3. 적절한 값 설정 (예: 20000 ERPM ≈ 4 m/s)

### 2.2 ROS2에서 제한

```python
class SpeedLimiter(Node):
    def __init__(self):
        super().__init__('speed_limiter')
        
        self.declare_parameter('max_speed', 2.0)
        self.declare_parameter('max_steering', 0.4)
        
        self.max_speed = self.get_parameter('max_speed').value
        self.max_steering = self.get_parameter('max_steering').value
        
        self.sub = self.create_subscription(
            AckermannDriveStamped, '/drive_raw', self.callback, 10
        )
        self.pub = self.create_publisher(
            AckermannDriveStamped, '/drive', 10
        )
    
    def callback(self, msg):
        limited = AckermannDriveStamped()
        limited.header = msg.header
        
        # 속도 제한
        limited.drive.speed = np.clip(
            msg.drive.speed,
            -self.max_speed,
            self.max_speed
        )
        
        # 조향 제한
        limited.drive.steering_angle = np.clip(
            msg.drive.steering_angle,
            -self.max_steering,
            self.max_steering
        )
        
        self.pub.publish(limited)
```

### 2.3 속도에 따른 조향 제한

```python
def limit_steering_by_speed(self, speed, steering):
    """고속에서 조향 제한."""
    if abs(speed) > 1.0:
        # 1m/s 이상에서 최대 조향각 감소
        max_steer = self.max_steering * (2.0 / abs(speed))
        max_steer = max(max_steer, 0.1)  # 최소 조향각
        steering = np.clip(steering, -max_steer, max_steer)
    return steering
```

---

## 3. 비상 정지 (Emergency Stop)

### 3.1 소프트웨어 비상 정지

```python
#!/usr/bin/env python3
"""
Emergency Stop Node
"""

import rclpy
from rclpy.node import Node
from std_msgs.msg import Bool
from ackermann_msgs.msg import AckermannDriveStamped


class EmergencyStop(Node):
    def __init__(self):
        super().__init__('emergency_stop')
        
        # 비상 정지 상태
        self.estop_active = False
        
        # 비상 정지 명령 수신
        self.estop_sub = self.create_subscription(
            Bool, '/emergency_stop', self.estop_callback, 10
        )
        
        # 키보드 입력 (Space bar)
        # 별도 키보드 노드 필요
        
        # 제어 명령 필터링
        self.drive_sub = self.create_subscription(
            AckermannDriveStamped, '/drive_cmd', self.drive_callback, 10
        )
        self.drive_pub = self.create_publisher(
            AckermannDriveStamped, '/drive', 10
        )
        
        # 타이머로 정지 명령 발행
        self.timer = self.create_timer(0.02, self.timer_callback)
        
        self.get_logger().info('Emergency stop ready')
    
    def estop_callback(self, msg):
        if msg.data:
            self.activate_estop()
        else:
            self.deactivate_estop()
    
    def activate_estop(self):
        self.estop_active = True
        self.get_logger().warn('EMERGENCY STOP ACTIVATED!')
    
    def deactivate_estop(self):
        self.estop_active = False
        self.get_logger().info('Emergency stop deactivated')
    
    def drive_callback(self, msg):
        if self.estop_active:
            return  # 비상 정지 중 명령 무시
        self.drive_pub.publish(msg)
    
    def timer_callback(self):
        if self.estop_active:
            # 정지 명령 발행
            stop = AckermannDriveStamped()
            stop.header.stamp = self.get_clock().now().to_msg()
            stop.drive.speed = 0.0
            stop.drive.steering_angle = 0.0
            self.drive_pub.publish(stop)
```

### 3.2 비상 정지 트리거

```bash
# 비상 정지 활성화
ros2 topic pub /emergency_stop std_msgs/msg/Bool "data: true" --once

# 비상 정지 해제
ros2 topic pub /emergency_stop std_msgs/msg/Bool "data: false" --once
```

### 3.3 키보드 비상 정지

```python
import sys
import select
import termios
import tty

class KeyboardEstop(Node):
    def __init__(self):
        super().__init__('keyboard_estop')
        self.estop_pub = self.create_publisher(Bool, '/emergency_stop', 10)
        
        # 터미널 설정 저장
        self.settings = termios.tcgetattr(sys.stdin)
    
    def get_key(self):
        tty.setraw(sys.stdin.fileno())
        select.select([sys.stdin], [], [], 0)
        key = sys.stdin.read(1)
        termios.tcsetattr(sys.stdin, termios.TCSADRAIN, self.settings)
        return key
    
    def run(self):
        print('Press SPACE for emergency stop, Q to quit')
        try:
            while True:
                key = self.get_key()
                if key == ' ':
                    msg = Bool()
                    msg.data = True
                    self.estop_pub.publish(msg)
                    print('EMERGENCY STOP!')
                elif key == 'r':
                    msg = Bool()
                    msg.data = False
                    self.estop_pub.publish(msg)
                    print('Reset')
                elif key == 'q':
                    break
        except Exception as e:
            print(e)
        finally:
            termios.tcsetattr(sys.stdin, termios.TCSADRAIN, self.settings)
```

---

## 4. 센서 기반 안전

### 4.1 LiDAR 기반 자동 정지

```python
class LidarSafety(Node):
    def __init__(self):
        super().__init__('lidar_safety')
        
        self.declare_parameter('stop_distance', 0.3)
        self.declare_parameter('slow_distance', 1.0)
        
        self.stop_dist = self.get_parameter('stop_distance').value
        self.slow_dist = self.get_parameter('slow_distance').value
        
        self.scan_sub = self.create_subscription(
            LaserScan, '/scan', self.scan_callback, 10
        )
        self.estop_pub = self.create_publisher(Bool, '/emergency_stop', 10)
        self.speed_limit_pub = self.create_publisher(Float32, '/speed_limit', 10)
    
    def scan_callback(self, msg):
        ranges = np.array(msg.ranges)
        ranges = np.where(np.isinf(ranges), msg.range_max, ranges)
        
        # 전방 거리
        center = len(ranges) // 2
        front = ranges[center - 50:center + 50]
        min_front = np.min(front)
        
        if min_front < self.stop_dist:
            # 비상 정지
            estop = Bool()
            estop.data = True
            self.estop_pub.publish(estop)
            self.get_logger().warn(f'Auto ESTOP! Distance: {min_front:.2f}m')
        elif min_front < self.slow_dist:
            # 속도 제한
            limit = Float32()
            limit.data = 0.5 * (min_front / self.slow_dist)
            self.speed_limit_pub.publish(limit)
```

---

## 5. 배터리 모니터링

### 5.1 VESC 상태 읽기

```python
class BatteryMonitor(Node):
    def __init__(self):
        super().__init__('battery_monitor')
        
        self.declare_parameter('low_voltage', 10.5)  # 3S LiPo: 3.5V/cell
        self.declare_parameter('critical_voltage', 9.9)  # 3.3V/cell
        
        self.low_v = self.get_parameter('low_voltage').value
        self.critical_v = self.get_parameter('critical_voltage').value
        
        self.sub = self.create_subscription(
            VescStateStamped, '/sensors/core', self.state_callback, 10
        )
        self.estop_pub = self.create_publisher(Bool, '/emergency_stop', 10)
    
    def state_callback(self, msg):
        voltage = msg.state.voltage_input
        
        if voltage < self.critical_v:
            self.get_logger().error(f'CRITICAL BATTERY: {voltage:.1f}V')
            estop = Bool()
            estop.data = True
            self.estop_pub.publish(estop)
        elif voltage < self.low_v:
            self.get_logger().warn(f'Low battery: {voltage:.1f}V')
```

---

## 6. 안전 체크리스트

### 6.1 테스트 전

- [ ] 배터리 충전 상태 확인 (> 80%)
- [ ] 물리적 손상 점검
- [ ] 모든 나사 조임 확인
- [ ] 센서 연결 확인

### 6.2 첫 주행

- [ ] 바퀴 들고 모터 테스트
- [ ] 조향 범위 확인
- [ ] 비상 정지 테스트
- [ ] 저속(0.5 m/s) 테스트

### 6.3 자율주행 테스트

- [ ] 넓은 공간 확보
- [ ] 비상 정지 버튼 준비
- [ ] 감시자 배치
- [ ] 녹화 (선택)

---

## 7. Phase 3 완료

축하합니다! Phase 3: 차량 제어를 완료했습니다.

---

*안전은 가장 중요한 우선순위입니다!*
