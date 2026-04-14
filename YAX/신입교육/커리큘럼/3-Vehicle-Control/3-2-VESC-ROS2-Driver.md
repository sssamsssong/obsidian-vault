# VESC ROS2 드라이버

> YAX F1TENTH 교육자료 | Phase 3: Vehicle Control
> 
> ROS2로 VESC 제어하기

---

## 1. 개요

### 학습 목표
1. VESC ROS2 드라이버를 설치하고 설정할 수 있다
2. AckermannDrive 메시지를 이해할 수 있다
3. ROS2 토픽으로 차량을 제어할 수 있다
4. 오도메트리 데이터를 확인할 수 있다

### 예상 소요 시간
- 전체: 2시간

### 전제 조건
- VESC Tool 설정 완료 (01-VESC-Setup.md)
- ROS2 기본 개념 이해

---

## 2. 드라이버 설치

### 2.1 F1TENTH VESC 드라이버 클론

```bash
cd ~/f1tenth_ws/src

# F1TENTH VESC 드라이버
git clone https://github.com/f1tenth/vesc.git

# 의존성 설치
cd ~/f1tenth_ws
rosdep install --from-paths src --ignore-src -r -y

# 빌드
colcon build --packages-select vesc vesc_driver vesc_msgs vesc_ackermann
source install/setup.bash
```

### 2.2 패키지 구성

```
vesc/
├── vesc_driver/         # VESC 통신 드라이버
├── vesc_msgs/           # 메시지 정의
├── vesc_ackermann/      # Ackermann 변환
└── vesc_interface/      # 공통 인터페이스
```

---

## 3. AckermannDrive 메시지

### 3.1 메시지 구조

```bash
ros2 interface show ackermann_msgs/msg/AckermannDriveStamped
```

```
std_msgs/Header header
AckermannDrive drive
  float32 steering_angle      # 조향각 (rad)
  float32 steering_angle_velocity
  float32 speed               # 속도 (m/s)
  float32 acceleration
  float32 jerk
```

### 3.2 사용 예

```python
from ackermann_msgs.msg import AckermannDriveStamped

msg = AckermannDriveStamped()
msg.header.stamp = self.get_clock().now().to_msg()
msg.header.frame_id = 'base_link'
msg.drive.speed = 1.0           # 1 m/s 전진
msg.drive.steering_angle = 0.2  # 좌회전 약 11.5°
```

### 3.3 조향각 변환

```python
import numpy as np

# 라디안 ↔ 도
degrees = np.degrees(radians)
radians = np.radians(degrees)

# 예: 20도 = 0.349 rad
steering_rad = np.radians(20)  # 0.349
```

---

## 4. 드라이버 설정

### 4.1 파라미터 파일

`config/vesc_config.yaml`:

```yaml
vesc_driver:
  ros__parameters:
    port: "/dev/ttyACM0"            # VESC USB 포트
    speed_to_erpm_gain: 4614.0      # 속도→ERPM 변환 계수
    speed_to_erpm_offset: 0.0
    steering_to_servo_gain: -1.0    # 조향→서보 변환 계수
    steering_to_servo_offset: 0.5   # 서보 중립 (0.0~1.0)
    
    # 오도메트리
    tachometer_ticks_to_meters_gain: 0.00006  # 틱→미터
    publish_odom: true
    odom_frame: "odom"
    base_frame: "base_link"
```

### 4.2 Launch 파일

`launch/vesc_driver.launch.py`:

```python
from launch import LaunchDescription
from launch_ros.actions import Node
from ament_index_python.packages import get_package_share_directory
import os

def generate_launch_description():
    config = os.path.join(
        get_package_share_directory('vesc_driver'),
        'config',
        'vesc_config.yaml'
    )
    
    return LaunchDescription([
        Node(
            package='vesc_driver',
            executable='vesc_driver_node',
            name='vesc_driver',
            parameters=[config],
            output='screen'
        )
    ])
```

### 4.3 실행

```bash
ros2 launch vesc_driver vesc_driver.launch.py
```

---

## 5. 토픽 구조

### 5.1 발행 토픽 (VESC → ROS2)

| 토픽 | 타입 | 설명 |
|------|------|------|
| `/sensors/core` | VescStateStamped | VESC 상태 |
| `/odom` | Odometry | 오도메트리 |
| `/sensors/servo_position_command` | Float64 | 현재 서보 위치 |

### 5.2 구독 토픽 (ROS2 → VESC)

| 토픽 | 타입 | 설명 |
|------|------|------|
| `/drive` | AckermannDriveStamped | Ackermann 명령 |
| `/commands/motor/speed` | Float64 | 직접 속도 명령 (ERPM) |
| `/commands/servo/position` | Float64 | 직접 서보 명령 (0.0~1.0) |

---

## 6. CLI 테스트

### 6.1 토픽 확인

```bash
# 토픽 목록
ros2 topic list

# VESC 상태 확인
ros2 topic echo /sensors/core

# 오도메트리 확인
ros2 topic echo /odom
```

### 6.2 직접 제어 테스트

```bash
# 서보 중립 (0.5)
ros2 topic pub /commands/servo/position std_msgs/msg/Float64 "data: 0.5" --once

# 서보 좌회전 (0.3)
ros2 topic pub /commands/servo/position std_msgs/msg/Float64 "data: 0.3" --once

# 서보 우회전 (0.7)
ros2 topic pub /commands/servo/position std_msgs/msg/Float64 "data: 0.7" --once
```

> ⚠️ 주의: 모터 테스트는 바퀴를 들어올린 상태에서!

```bash
# 저속 전진 (ERPM)
ros2 topic pub /commands/motor/speed std_msgs/msg/Float64 "data: 1000.0" --once

# 정지
ros2 topic pub /commands/motor/speed std_msgs/msg/Float64 "data: 0.0" --once
```

### 6.3 Ackermann 명령

```bash
# 저속 전진, 직진
ros2 topic pub /drive ackermann_msgs/msg/AckermannDriveStamped \
  "{header: {frame_id: 'base_link'}, drive: {speed: 0.5, steering_angle: 0.0}}" --once

# 정지
ros2 topic pub /drive ackermann_msgs/msg/AckermannDriveStamped \
  "{header: {frame_id: 'base_link'}, drive: {speed: 0.0, steering_angle: 0.0}}" --once
```

---

## 7. 파라미터 튜닝

### 7.1 speed_to_erpm_gain 계산

```python
# ERPM = RPM × 극 쌍 수 (pole pairs)
# 속도 = RPM × 휠 둘레 / 60 / 기어비

# F1TENTH 기본값
pole_pairs = 7
wheel_diameter = 0.1  # 10cm
gear_ratio = 2.6

wheel_circumference = np.pi * wheel_diameter  # 0.314m
speed_per_rpm = wheel_circumference / 60 / gear_ratio

speed_to_erpm_gain = 1 / speed_per_rpm * pole_pairs
# 약 4614
```

### 7.2 steering_to_servo 캘리브레이션

1. VESC Tool에서 서보 범위 확인 (min, max)
2. 조향각과 서보 값 매핑
3. 선형 변환 계수 계산

```python
# servo = gain * steering_angle + offset
# 예: steering -0.4 ~ +0.4 rad → servo 0.3 ~ 0.7

steering_range = 0.8  # -0.4 ~ +0.4
servo_range = 0.4     # 0.3 ~ 0.7

steering_to_servo_gain = -servo_range / steering_range  # 부호 주의
steering_to_servo_offset = 0.5  # 중립
```

---

## 8. Ackermann 변환 노드

### 8.1 별도 변환 노드 사용

```bash
ros2 launch vesc_ackermann ackermann_to_vesc_node.launch.py
```

### 8.2 변환 파라미터

```yaml
ackermann_to_vesc_node:
  ros__parameters:
    speed_to_erpm_gain: 4614.0
    speed_to_erpm_offset: 0.0
    steering_angle_to_servo_gain: -1.2217
    steering_angle_to_servo_offset: 0.5
    
    # 입력/출력 토픽
    ackermann_cmd_topic: "/drive"
    speed_cmd_topic: "/commands/motor/speed"
    servo_cmd_topic: "/commands/servo/position"
```

---

## 9. 오도메트리 설정

### 9.1 오도메트리 파라미터

```yaml
vesc_driver:
  ros__parameters:
    publish_odom: true
    odom_frame: "odom"
    base_frame: "base_link"
    
    # 틱당 이동 거리 (캘리브레이션 필요)
    tachometer_ticks_to_meters_gain: 0.00006
```

### 9.2 오도메트리 확인

```bash
ros2 topic echo /odom

# 예상 출력
# pose:
#   pose:
#     position: {x: 1.23, y: 0.45, z: 0.0}
#     orientation: {x: 0, y: 0, z: 0.1, w: 0.99}
# twist:
#   twist:
#     linear: {x: 0.5, y: 0, z: 0}
#     angular: {x: 0, y: 0, z: 0.1}
```

---

## 10. 실습 과제

### 과제 1: 기본 제어 노드

조이스틱 없이 키보드로 간단한 제어:
- 'w': 전진
- 's': 후진
- 'a': 좌회전
- 'd': 우회전
- 'space': 정지

### 과제 2: 속도 ramp

급가속 방지를 위한 속도 램프 노드:
- 목표 속도까지 점진적으로 증가
- 최대 가속도 제한

---

## 11. 검증 체크리스트

- [ ] VESC 드라이버 설치 및 빌드
- [ ] `/drive` 토픽 발행으로 제어
- [ ] 서보 동작 확인
- [ ] 모터 동작 확인 (바퀴 들고)
- [ ] 오도메트리 수신

---

*ROS2로 차량을 제어할 준비가 되었습니다!*
