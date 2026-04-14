# VESC 모터 제어 가이드

> YAX F1TENTH 교육자료 | Domain 05: 차량 제어

---

## 1. 개요

### 학습 목표
1. VESC의 역할과 설정 방법을 이해한다
2. VESC Tool로 모터를 설정할 수 있다
3. ROS2로 차량을 제어할 수 있다
4. 조이스틱으로 수동 주행할 수 있다

### 예상 소요 시간: 4-5시간

### ⚠️ 안전 주의사항
- **바퀴를 들어올린 상태**에서 모터 테스트
- **비상 정지** 준비 (배터리 분리)
- **개방된 공간**에서 주행 테스트
- **보호 장비** 착용 권장

---

## 2. VESC 소개

### 2.1 VESC란?
VESC (Vedder Electronic Speed Controller)는 BLDC 모터용 오픈소스 모터 컨트롤러입니다.

### 2.2 F1TENTH 연결 구조

```
[배터리] → [VESC] → [모터 (구동)]
              ↓
           [서보 (조향)]
              ↓
         [Jetson USB]
```

---

## 3. VESC Tool 설정

### 3.1 VESC Tool 설치 (노트북)

1. https://vesc-project.com/vesc_tool 접속
2. 계정 생성 후 다운로드
3. 설치 및 실행

### 3.2 VESC 연결

1. 배터리 연결 (VESC 전원)
2. USB로 노트북 연결
3. VESC Tool → AutoConnect

### 3.3 펌웨어 업데이트

1. Firmware 탭
2. "Show non-default firmwares" 체크
3. `VESC_servoout.bin` 선택 (서보 출력용)
4. 다운로드 → 완료

### 3.4 모터 설정 (FOC)

1. Motor Settings → FOC 탭
2. 화살표 순서대로 4개 버튼 클릭
3. **모터가 회전** (바퀴 들어올리기!)
4. 완료 후 "Write Motor Configuration"

### 3.5 Openloop 파라미터

```
Motor Settings → Sensorless:
- Openloop Hysteresis: 0.01
- Openloop Time: 0.01
→ Write Motor Configuration
```

---

## 4. ROS2 드라이버 설치

### 4.1 f1tenth_system 클론

```bash
cd ~/ros2_ws/src
git clone https://github.com/f1tenth/f1tenth_system.git

cd ~/ros2_ws
rosdep install -i --from-path src --rosdistro humble -y
colcon build
source install/setup.bash
```

### 4.2 udev 규칙

```bash
sudo nano /etc/udev/rules.d/99-vesc.rules
```

```
KERNEL=="ttyACM[0-9]*", ACTION=="add", ATTRS{idVendor}=="0483", ATTRS{idProduct}=="5740", MODE="0666", GROUP="dialout", SYMLINK+="sensors/vesc"
```

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```

---

## 5. AckermannDrive 메시지

### 5.1 메시지 구조

```python
from ackermann_msgs.msg import AckermannDriveStamped

msg = AckermannDriveStamped()
msg.header.stamp = self.get_clock().now().to_msg()
msg.drive.speed = 1.0           # m/s
msg.drive.steering_angle = 0.2  # rad (좌회전 +, 우회전 -)
```

### 5.2 값 범위

| 항목 | 범위 | 단위 |
|------|------|------|
| speed | -2.0 ~ 2.0 | m/s |
| steering_angle | -0.4 ~ 0.4 | rad |

---

## 6. 차량 제어 노드

```python
import rclpy
from rclpy.node import Node
from ackermann_msgs.msg import AckermannDriveStamped

class VehicleController(Node):
    def __init__(self):
        super().__init__('vehicle_controller')
        self.publisher = self.create_publisher(
            AckermannDriveStamped, '/drive', 10)
        
    def drive(self, speed, steering):
        msg = AckermannDriveStamped()
        msg.header.stamp = self.get_clock().now().to_msg()
        msg.drive.speed = speed
        msg.drive.steering_angle = steering
        self.publisher.publish(msg)
        
    def stop(self):
        self.drive(0.0, 0.0)

def main():
    rclpy.init()
    controller = VehicleController()
    
    # 1초간 전진
    controller.drive(1.0, 0.0)
    import time
    time.sleep(1.0)
    controller.stop()
    
    controller.destroy_node()
    rclpy.shutdown()
```

---

## 7. 수동 조종 (Teleop)

### 7.1 조이스틱 연결 (Logitech F710)

```bash
# 조이스틱 확인
ls /dev/input/js*

# joy 패키지 설치
sudo apt install ros-humble-joy ros-humble-teleop-twist-joy -y
```

### 7.2 Teleop 실행

```bash
# VESC 드라이버 + Teleop
ros2 launch f1tenth_stack bringup_launch.py
```

### 7.3 조작법

- **LB (L1)**: Deadman 스위치 (누르고 있어야 동작)
- **왼쪽 스틱 상/하**: 속도
- **오른쪽 스틱 좌/우**: 조향

---

## 8. 오도메트리 캘리브레이션

### 8.1 속도 캘리브레이션

```bash
# 1m/s 명령 → 실제 이동 거리 측정
# speed_to_erpm_gain 조정
```

### 8.2 조향 캘리브레이션

```bash
# steering_angle = 0 일 때 직진 확인
# steering_angle_to_servo_offset 조정
```

---

## 9. 안전 설정

### 9.1 속도 제한

**vesc.yaml:**
```yaml
speed_min: -2.0
speed_max: 2.0
```

### 9.2 비상 정지

```python
def emergency_stop(self):
    for _ in range(10):
        self.drive(0.0, 0.0)
        time.sleep(0.01)
```

---

## 10. 검증 체크리스트

- [ ] VESC Tool 연결 성공
- [ ] 모터 회전 확인
- [ ] 서보(조향) 동작 확인
- [ ] ROS2 `/drive` 토픽으로 제어
- [ ] 조이스틱 Teleop 동작

---

*YAX F1TENTH 교육자료 | 작성일: 2026-02-10*
