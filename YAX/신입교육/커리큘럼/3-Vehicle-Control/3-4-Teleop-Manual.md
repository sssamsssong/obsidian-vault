# 수동 조종 (Teleop)

> YAX F1TENTH 교육자료 | Phase 3: Vehicle Control
> 
> 조이스틱으로 차량 조종하기

---

## 1. 개요

### 학습 목표
1. 조이스틱을 ROS2와 연결할 수 있다
2. Teleop 노드를 설정하고 실행할 수 있다
3. 안전하게 차량을 수동 조종할 수 있다

### 예상 소요 시간
- 전체: 1.5시간

### 전제 조건
- VESC ROS2 드라이버 설정 완료

---

## 2. 조이스틱 연결

### 2.1 지원 조이스틱

| 조이스틱 | 연결 | 호환성 |
|---------|------|--------|
| Logitech F710 | USB 동글 | 권장 |
| Xbox Controller | USB/Bluetooth | 좋음 |
| PS4 DualShock | Bluetooth | 좋음 |
| 일반 USB 게임패드 | USB | 대부분 지원 |

### 2.2 연결 확인

```bash
# 연결된 입력 장치 확인
ls /dev/input/js*

# 또는
cat /proc/bus/input/devices | grep -A 5 "Joystick"
```

### 2.3 조이스틱 테스트

```bash
# jstest 설치
sudo apt install joystick

# 테스트
jstest /dev/input/js0
```

축과 버튼을 움직여 값이 변하는지 확인합니다.

---

## 3. Joy 패키지 설치

### 3.1 설치

```bash
sudo apt install ros-humble-joy ros-humble-teleop-twist-joy
```

### 3.2 Joy 노드 실행

```bash
ros2 run joy joy_node
```

### 3.3 Joy 토픽 확인

```bash
ros2 topic echo /joy

# 예상 출력
# axes: [0.0, 0.0, 1.0, 0.0, 0.0, 1.0, 0.0, 0.0]
# buttons: [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
```

---

## 4. Teleop 노드 설정

### 4.1 teleop_twist_joy 사용

```bash
ros2 launch teleop_twist_joy teleop-launch.py joy_config:='xbox'
```

### 4.2 커스텀 Teleop 노드

`teleop_node.py`:

```python
#!/usr/bin/env python3
"""
F1TENTH Teleop Node
"""

import rclpy
from rclpy.node import Node
from sensor_msgs.msg import Joy
from ackermann_msgs.msg import AckermannDriveStamped
import numpy as np


class F1TenthTeleop(Node):
    def __init__(self):
        super().__init__('f1tenth_teleop')
        
        # Parameters
        self.declare_parameter('max_speed', 2.0)
        self.declare_parameter('max_steering', 0.4)
        self.declare_parameter('speed_axis', 1)      # 왼쪽 스틱 Y
        self.declare_parameter('steering_axis', 3)   # 오른쪽 스틱 X
        self.declare_parameter('deadman_button', 4)  # LB 버튼
        
        self.max_speed = self.get_parameter('max_speed').value
        self.max_steering = self.get_parameter('max_steering').value
        self.speed_axis = self.get_parameter('speed_axis').value
        self.steering_axis = self.get_parameter('steering_axis').value
        self.deadman_button = self.get_parameter('deadman_button').value
        
        # Subscriber
        self.joy_sub = self.create_subscription(
            Joy, '/joy', self.joy_callback, 10
        )
        
        # Publisher
        self.drive_pub = self.create_publisher(
            AckermannDriveStamped, '/drive', 10
        )
        
        # State
        self.last_speed = 0.0
        self.last_steering = 0.0
        
        self.get_logger().info('F1TENTH Teleop started')
        self.get_logger().info(f'Hold button {self.deadman_button} (LB) to drive')
    
    def joy_callback(self, msg):
        drive_msg = AckermannDriveStamped()
        drive_msg.header.stamp = self.get_clock().now().to_msg()
        drive_msg.header.frame_id = 'base_link'
        
        # 데드맨 버튼 확인
        if len(msg.buttons) > self.deadman_button and \
           msg.buttons[self.deadman_button] == 1:
            
            # 속도 (왼쪽 스틱 Y)
            speed_input = msg.axes[self.speed_axis]
            drive_msg.drive.speed = speed_input * self.max_speed
            
            # 조향 (오른쪽 스틱 X)
            steering_input = msg.axes[self.steering_axis]
            drive_msg.drive.steering_angle = steering_input * self.max_steering
            
            self.last_speed = drive_msg.drive.speed
            self.last_steering = drive_msg.drive.steering_angle
        else:
            # 데드맨 버튼 해제 → 정지
            drive_msg.drive.speed = 0.0
            drive_msg.drive.steering_angle = 0.0
        
        self.drive_pub.publish(drive_msg)


def main(args=None):
    rclpy.init(args=args)
    node = F1TenthTeleop()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()


if __name__ == '__main__':
    main()
```

### 4.3 Launch 파일

```python
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    return LaunchDescription([
        # Joy 노드
        Node(
            package='joy',
            executable='joy_node',
            name='joy_node',
            parameters=[{
                'device_id': 0,
                'deadzone': 0.1,
                'autorepeat_rate': 20.0
            }]
        ),
        # Teleop 노드
        Node(
            package='f1tenth_system',
            executable='teleop_node',
            name='teleop',
            parameters=[{
                'max_speed': 2.0,
                'max_steering': 0.4,
                'deadman_button': 4
            }]
        )
    ])
```

---

## 5. 조이스틱 매핑

### 5.1 일반적인 Xbox 컨트롤러 매핑

```
Axes:
  0: 왼쪽 스틱 X (좌우)
  1: 왼쪽 스틱 Y (상하)
  2: LT (트리거)
  3: 오른쪽 스틱 X (좌우)
  4: 오른쪽 스틱 Y (상하)
  5: RT (트리거)
  6: D-pad X
  7: D-pad Y

Buttons:
  0: A
  1: B
  2: X
  3: Y
  4: LB (데드맨)
  5: RB
  6: Back
  7: Start
  8: Xbox
  9: 왼쪽 스틱 클릭
  10: 오른쪽 스틱 클릭
```

### 5.2 조이스틱 확인 스크립트

```python
#!/usr/bin/env python3
"""
Joystick Mapping Helper
"""

import rclpy
from rclpy.node import Node
from sensor_msgs.msg import Joy


class JoyMapper(Node):
    def __init__(self):
        super().__init__('joy_mapper')
        self.sub = self.create_subscription(Joy, '/joy', self.callback, 10)
        self.last_axes = None
        self.last_buttons = None
    
    def callback(self, msg):
        # 변경된 축 표시
        if self.last_axes is not None:
            for i, (old, new) in enumerate(zip(self.last_axes, msg.axes)):
                if abs(new - old) > 0.1:
                    print(f'Axis {i}: {new:.2f}')
        
        # 변경된 버튼 표시
        if self.last_buttons is not None:
            for i, (old, new) in enumerate(zip(self.last_buttons, msg.buttons)):
                if new != old:
                    print(f'Button {i}: {new}')
        
        self.last_axes = list(msg.axes)
        self.last_buttons = list(msg.buttons)


def main(args=None):
    rclpy.init(args=args)
    node = JoyMapper()
    print('Move joystick axes and press buttons to see mapping...')
    rclpy.spin(node)


if __name__ == '__main__':
    main()
```

---

## 6. 안전 기능

### 6.1 데드맨 스위치

데드맨 버튼을 놓으면 즉시 정지:

```python
# joy_callback에서
if not msg.buttons[self.deadman_button]:
    self.stop()
```

### 6.2 속도 제한

```python
def limit_speed(self, speed):
    return np.clip(speed, -self.max_speed, self.max_speed)
```

### 6.3 부드러운 가속

```python
def smooth_speed(self, target_speed, current_speed, max_accel=2.0, dt=0.05):
    """급가속 방지."""
    speed_diff = target_speed - current_speed
    max_change = max_accel * dt
    
    if abs(speed_diff) > max_change:
        return current_speed + np.sign(speed_diff) * max_change
    return target_speed
```

---

## 7. 실습

### 실습 1: 기본 조종

1. VESC 드라이버 실행
2. Joy + Teleop 실행
3. 데드맨 버튼을 누른 상태에서 조종
4. 전진, 후진, 좌/우회전 연습

### 실습 2: 장애물 코스

간단한 장애물 코스를 만들고 충돌 없이 통과하세요.

### 실습 3: 파라미터 조정

- 최대 속도 조정
- 조향 감도 조정
- 데드존 조정

---

## 8. 문제 해결

### 조이스틱 인식 안됨

```bash
# 권한 문제
sudo chmod 666 /dev/input/js0

# 또는 사용자를 input 그룹에 추가
sudo usermod -a -G input $USER
# 재로그인 필요
```

### 축/버튼 매핑 문제

joy_mapper 스크립트로 실제 매핑을 확인하고 파라미터를 조정하세요.

---

## 9. 검증 체크리스트

- [ ] 조이스틱 연결 확인
- [ ] `/joy` 토픽 수신
- [ ] Teleop 노드 동작
- [ ] 데드맨 스위치 동작
- [ ] 안전한 수동 주행

---

*수동 조종은 자율주행 개발의 첫 단계입니다!*
