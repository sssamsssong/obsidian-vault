# LiDAR 센서 가이드

> YAX F1TENTH 교육자료 | Domain 03: 2D LiDAR

---

## 1. 개요

### 학습 목표
1. LiDAR의 작동 원리를 이해한다
2. Hokuyo LiDAR를 ROS2에 연결할 수 있다
3. LaserScan 메시지를 분석하고 처리할 수 있다
4. RViz2에서 LiDAR 데이터를 시각화할 수 있다

### 예상 소요 시간
- 전체: 3-4시간
- LiDAR 원리: 30분
- 하드웨어 연결: 30분
- 드라이버 설치: 30분
- 데이터 처리: 1.5시간

### 필요 장비
- Hokuyo UST-10LX LiDAR
- USB 케이블
- Jetson Orin Nano (ROS2 설치 완료)

---

## 2. LiDAR 원리

### 2.1 ToF (Time of Flight) 방식

LiDAR는 레이저 빛을 발사하고 반사되어 돌아오는 시간을 측정합니다.

```
거리 = (빛의 속도 × 왕복 시간) / 2
```

### 2.2 2D vs 3D LiDAR

| 특성 | 2D LiDAR | 3D LiDAR |
|------|----------|----------|
| 스캔 방식 | 수평 1개 평면 | 다중 레이어 |
| 출력 | LaserScan | PointCloud2 |
| 가격 | $500-2000 | $5000+ |
| 용도 | F1TENTH, 실내 로봇 | 자율주행차 |

### 2.3 주요 사양 이해

- **측정 거리**: 최소~최대 감지 거리
- **각도 범위 (FOV)**: 스캔 범위 (270°, 360° 등)
- **각도 해상도**: 각 측정 간 각도 간격
- **스캔 주파수**: 초당 스캔 횟수 (Hz)

---

## 3. Hokuyo UST-10LX 소개

### 3.1 하드웨어 사양

| 항목 | 값 |
|------|-----|
| 측정 거리 | 0.06m ~ 10m |
| 각도 범위 | 270° |
| 각도 해상도 | 0.25° (1080 포인트) |
| 스캔 주파수 | 40Hz |
| 정확도 | ±40mm |
| 인터페이스 | USB 2.0 / Ethernet |
| 전원 | 5V DC (USB 공급) |

### 3.2 F1TENTH 장착 위치

차량 전방 중앙, 지면에서 약 10cm 높이에 수평으로 장착합니다.

---

## 4. 하드웨어 연결

### 4.1 USB 연결

1. LiDAR USB 케이블을 Jetson에 연결
2. 연결 확인:

```bash
# USB 디바이스 확인
lsusb
# 출력: Hokuyo Data Flex for USB

# 시리얼 포트 확인
ls /dev/ttyACM*
# 출력: /dev/ttyACM0
```

### 4.2 권한 설정

```bash
# 현재 사용자를 dialout 그룹에 추가
sudo usermod -a -G dialout $USER

# 재로그인 필요
logout
# 또는
sudo reboot
```

---

## 5. udev 규칙 설정

재부팅 후에도 고정된 디바이스 이름을 사용하려면:

```bash
# udev 규칙 파일 생성
sudo nano /etc/udev/rules.d/99-hokuyo.rules
```

**파일 내용:**
```
KERNEL=="ttyACM[0-9]*", ACTION=="add", ATTRS{idVendor}=="15d1", MODE="0666", GROUP="dialout", SYMLINK+="sensors/hokuyo"
```

```bash
# 규칙 적용
sudo udevadm control --reload-rules
sudo udevadm trigger

# 확인
ls -l /dev/sensors/
# 출력: hokuyo -> ../ttyACM0
```

---

## 6. ROS2 드라이버 설치

### 6.1 urg_node 설치

```bash
sudo apt update
sudo apt install ros-humble-urg-node -y
```

### 6.2 드라이버 실행

```bash
# 기본 실행
ros2 run urg_node urg_node_driver --ros-args -p serial_port:=/dev/ttyACM0

# 또는 심볼릭 링크 사용
ros2 run urg_node urg_node_driver --ros-args -p serial_port:=/dev/sensors/hokuyo
```

### 6.3 토픽 확인

```bash
# 토픽 목록
ros2 topic list
# 출력: /scan

# 토픽 정보
ros2 topic info /scan
# Type: sensor_msgs/msg/LaserScan

# 데이터 확인
ros2 topic echo /scan --once
```

---

## 7. LaserScan 메시지 이해

### 7.1 메시지 구조

```bash
ros2 interface show sensor_msgs/msg/LaserScan
```

```
std_msgs/Header header
    builtin_interfaces/Time stamp
    string frame_id

float32 angle_min        # 시작 각도 (rad)
float32 angle_max        # 끝 각도 (rad)  
float32 angle_increment  # 각도 간격 (rad)
float32 time_increment   # 측정 간 시간
float32 scan_time        # 스캔 총 시간
float32 range_min        # 최소 측정 거리 (m)
float32 range_max        # 최대 측정 거리 (m)
float32[] ranges         # 거리 배열 (m)
float32[] intensities    # 강도 배열 (선택)
```

### 7.2 ranges 배열 해석

```python
# ranges[i]는 angle_min + i * angle_increment 방향의 거리
# 예: Hokuyo UST-10LX
# angle_min = -2.356 rad (-135°)
# angle_max = 2.356 rad (135°)
# angle_increment = 0.00436 rad (0.25°)
# len(ranges) = 1080

# 정면(0°) 인덱스 계산
front_idx = int((0 - angle_min) / angle_increment)
# front_idx ≈ 540
```

### 7.3 특수 값

```python
import math

# inf: 최대 거리 초과 (물체 없음)
if math.isinf(ranges[i]):
    print("물체 없음")

# nan: 측정 실패
if math.isnan(ranges[i]):
    print("측정 실패")
```

---

## 8. RViz2에서 시각화

### 8.1 RViz2 실행

```bash
rviz2
```

### 8.2 LaserScan 디스플레이 추가

1. 좌측 "Add" 버튼 클릭
2. "By topic" 탭 선택
3. `/scan` → `LaserScan` 선택
4. "OK" 클릭

### 8.3 설정 조정

- **Fixed Frame**: `laser` (또는 LiDAR frame_id)
- **Topic**: `/scan`
- **Size**: 0.05 (포인트 크기)
- **Color Transformer**: Intensity 또는 FlatColor

### 8.4 설정 저장

```bash
# RViz 설정 저장
# File → Save Config As → ~/ros2_ws/config/lidar.rviz

# 저장된 설정으로 실행
rviz2 -d ~/ros2_ws/config/lidar.rviz
```

---

## 9. LiDAR 데이터 처리 (Python)

### 9.1 기본 Subscriber 노드

**파일: `lidar_processor.py`**

```python
import rclpy
from rclpy.node import Node
from sensor_msgs.msg import LaserScan
import numpy as np

class LidarProcessor(Node):
    def __init__(self):
        super().__init__('lidar_processor')
        
        self.subscription = self.create_subscription(
            LaserScan,
            '/scan',
            self.scan_callback,
            10
        )
        
        self.get_logger().info('LiDAR Processor 시작!')

    def scan_callback(self, msg):
        # numpy 배열로 변환
        ranges = np.array(msg.ranges)
        
        # inf, nan 제거
        valid_ranges = ranges[np.isfinite(ranges)]
        
        if len(valid_ranges) > 0:
            min_dist = np.min(valid_ranges)
            avg_dist = np.mean(valid_ranges)
            
            self.get_logger().info(
                f'최소 거리: {min_dist:.2f}m, 평균 거리: {avg_dist:.2f}m'
            )

def main(args=None):
    rclpy.init(args=args)
    node = LidarProcessor()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

### 9.2 최근접 장애물 찾기

```python
def find_closest_obstacle(self, msg):
    ranges = np.array(msg.ranges)
    
    # 유효한 측정값만 사용
    valid_mask = np.isfinite(ranges) & (ranges > msg.range_min)
    
    if not np.any(valid_mask):
        return None, None
    
    # 최소 거리 인덱스
    valid_ranges = np.where(valid_mask, ranges, np.inf)
    min_idx = np.argmin(valid_ranges)
    min_dist = ranges[min_idx]
    
    # 각도 계산
    angle = msg.angle_min + min_idx * msg.angle_increment
    angle_deg = np.degrees(angle)
    
    return min_dist, angle_deg
```

### 9.3 전방 90도 범위 필터링

```python
def get_front_ranges(self, msg):
    """전방 ±45도 범위의 거리값 반환"""
    ranges = np.array(msg.ranges)
    
    # 각도 배열 생성
    angles = np.linspace(msg.angle_min, msg.angle_max, len(ranges))
    
    # 전방 ±45도 (±0.785 rad) 필터
    front_mask = np.abs(angles) < np.radians(45)
    
    front_ranges = ranges[front_mask]
    front_angles = angles[front_mask]
    
    return front_ranges, front_angles
```

### 9.4 충돌 경고 시스템

```python
class CollisionWarning(Node):
    def __init__(self):
        super().__init__('collision_warning')
        
        self.subscription = self.create_subscription(
            LaserScan, '/scan', self.scan_callback, 10)
        
        self.warning_distance = 0.5  # 50cm
        self.danger_distance = 0.3   # 30cm

    def scan_callback(self, msg):
        ranges = np.array(msg.ranges)
        
        # 전방 범위만 확인
        angles = np.linspace(msg.angle_min, msg.angle_max, len(ranges))
        front_mask = np.abs(angles) < np.radians(30)
        front_ranges = ranges[front_mask]
        
        # 유효한 최소 거리
        valid = front_ranges[np.isfinite(front_ranges)]
        if len(valid) == 0:
            return
            
        min_dist = np.min(valid)
        
        if min_dist < self.danger_distance:
            self.get_logger().error(f'🚨 위험! {min_dist:.2f}m')
        elif min_dist < self.warning_distance:
            self.get_logger().warn(f'⚠️ 주의! {min_dist:.2f}m')
```

---

## 10. Launch 파일

**파일: `lidar_launch.py`**

```python
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    return LaunchDescription([
        # LiDAR 드라이버
        Node(
            package='urg_node',
            executable='urg_node_driver',
            name='hokuyo_lidar',
            parameters=[{
                'serial_port': '/dev/sensors/hokuyo',
                'frame_id': 'laser',
            }],
            output='screen'
        ),
        
        # LiDAR 프로세서
        Node(
            package='my_robot_pkg',
            executable='lidar_processor',
            name='lidar_processor',
            output='screen'
        ),
    ])
```

---

## 11. 검증 체크리스트

- [ ] LiDAR USB 연결 확인 (`ls /dev/ttyACM*`)
- [ ] udev 규칙 설정 (`ls /dev/sensors/`)
- [ ] urg_node 실행 성공
- [ ] `/scan` 토픽 발행 확인
- [ ] RViz2에서 포인트 시각화
- [ ] 최근접 장애물 거리 출력 동작

---

## 12. 문제 해결

### LiDAR가 인식되지 않을 때

```bash
# USB 연결 확인
lsusb | grep Hokuyo

# 권한 확인
ls -l /dev/ttyACM0
# crw-rw---- 1 root dialout ...

# dialout 그룹 확인
groups
```

### 데이터가 이상할 때

- LiDAR 렌즈 청소
- 반사율 높은 물체 확인 (유리, 거울)
- 케이블 연결 상태 확인

### 프레임 레이트가 낮을 때

```bash
# 스캔 주파수 확인
ros2 topic hz /scan
# 40Hz 근처여야 정상
```

---

## 참고 자료

- [urg_node GitHub](https://github.com/ros-drivers/urg_node)
- [sensor_msgs/LaserScan](https://docs.ros2.org/humble/api/sensor_msgs/msg/LaserScan.html)
- [Hokuyo 공식 사이트](https://www.hokuyo-aut.jp/)

---

*YAX F1TENTH 교육자료 | 작성일: 2026-02-10*
