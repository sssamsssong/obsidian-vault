# LiDAR 데이터 처리

> YAX F1TENTH 교육자료 | Phase 2: Sensors
> 
> LaserScan 데이터 분석 및 처리

---

## 1. 개요

### 학습 목표
1. LaserScan 메시지 구조를 완벽히 이해할 수 있다
2. LiDAR 데이터를 필터링하고 전처리할 수 있다
3. 특정 영역의 최소/최대 거리를 계산할 수 있다
4. 장애물 감지 노드를 작성할 수 있다

### 예상 소요 시간
- 전체: 2시간

### 전제 조건
- LiDAR 기본 설정 완료 (01-LiDAR-Basics.md)
- Python/NumPy 기초

---

## 2. LaserScan 메시지 구조

### 2.1 메시지 정의

```bash
ros2 interface show sensor_msgs/msg/LaserScan
```

```
std_msgs/Header header
  builtin_interfaces/Time stamp
  string frame_id

float32 angle_min        # 시작 각도 (rad)
float32 angle_max        # 끝 각도 (rad)
float32 angle_increment  # 각도 증분 (rad)

float32 time_increment   # 측정 시간 간격
float32 scan_time        # 전체 스캔 시간

float32 range_min        # 최소 유효 거리 (m)
float32 range_max        # 최대 유효 거리 (m)

float32[] ranges         # 거리 배열 (m)
float32[] intensities    # 강도 배열 (선택)
```

### 2.2 Hokuyo UST-10LX 예시 값

```python
# 일반적인 값
angle_min = -2.356  # -135° (rad)
angle_max = 2.356   # +135° (rad)
angle_increment = 0.00436  # 0.25° (rad)
range_min = 0.06    # 6cm
range_max = 10.0    # 10m
len(ranges) = 1081  # 270° / 0.25° = 1080 + 1
```

### 2.3 각도-인덱스 변환

```python
import numpy as np

def angle_to_index(angle_rad, msg):
    """각도(rad)를 인덱스로 변환."""
    return int((angle_rad - msg.angle_min) / msg.angle_increment)

def index_to_angle(index, msg):
    """인덱스를 각도(rad)로 변환."""
    return msg.angle_min + index * msg.angle_increment

# 예: 전방 (0°) 인덱스
front_index = angle_to_index(0, msg)  # 약 540
```

---

## 3. 데이터 전처리

### 3.1 inf/NaN 처리

LiDAR 데이터에는 무한대(inf)와 NaN 값이 포함될 수 있습니다.

```python
import numpy as np

def preprocess_scan(ranges, range_max):
    """LiDAR 데이터 전처리."""
    ranges = np.array(ranges)
    
    # inf를 range_max로 대체
    ranges = np.where(np.isinf(ranges), range_max, ranges)
    
    # NaN을 range_max로 대체
    ranges = np.where(np.isnan(ranges), range_max, ranges)
    
    # 0 이하 값 처리
    ranges = np.where(ranges <= 0, range_max, ranges)
    
    return ranges
```

### 3.2 노이즈 필터링

```python
from scipy.ndimage import median_filter

def filter_noise(ranges, kernel_size=5):
    """중앙값 필터로 노이즈 제거."""
    return median_filter(ranges, size=kernel_size)
```

### 3.3 이동 평균 필터

```python
def moving_average(ranges, window=3):
    """이동 평균 필터."""
    return np.convolve(ranges, np.ones(window)/window, mode='same')
```

---

## 4. 영역별 데이터 추출

### 4.1 전방 영역

```python
def get_front_ranges(ranges, msg, fov_deg=60):
    """전방 ±fov_deg/2 범위의 데이터 추출."""
    fov_rad = np.radians(fov_deg)
    
    start_angle = -fov_rad / 2
    end_angle = fov_rad / 2
    
    start_idx = angle_to_index(start_angle, msg)
    end_idx = angle_to_index(end_angle, msg)
    
    return ranges[start_idx:end_idx+1]
```

### 4.2 좌/우 영역

```python
def get_left_ranges(ranges, msg, angle_start=45, angle_end=135):
    """좌측 영역 데이터 추출."""
    start_rad = np.radians(angle_start)
    end_rad = np.radians(angle_end)
    
    start_idx = angle_to_index(start_rad, msg)
    end_idx = angle_to_index(end_rad, msg)
    
    return ranges[start_idx:end_idx+1]

def get_right_ranges(ranges, msg, angle_start=-135, angle_end=-45):
    """우측 영역 데이터 추출."""
    start_rad = np.radians(angle_start)
    end_rad = np.radians(angle_end)
    
    start_idx = angle_to_index(start_rad, msg)
    end_idx = angle_to_index(end_rad, msg)
    
    return ranges[start_idx:end_idx+1]
```

---

## 5. 거리 계산

### 5.1 최소 거리

```python
def get_min_distance(ranges, msg, fov_deg=60):
    """전방 최소 거리 계산."""
    front_ranges = get_front_ranges(ranges, msg, fov_deg)
    return np.min(front_ranges)
```

### 5.2 최소 거리와 각도

```python
def get_closest_point(ranges, msg):
    """가장 가까운 점의 거리와 각도."""
    min_idx = np.argmin(ranges)
    min_distance = ranges[min_idx]
    min_angle = index_to_angle(min_idx, msg)
    
    return min_distance, min_angle
```

### 5.3 평균 거리

```python
def get_average_distance(ranges, msg, fov_deg=60):
    """전방 평균 거리 계산."""
    front_ranges = get_front_ranges(ranges, msg, fov_deg)
    return np.mean(front_ranges)
```

---

## 6. 장애물 감지 노드

### 6.1 기본 노드

```python
#!/usr/bin/env python3
"""
Obstacle Detector Node
"""

import rclpy
from rclpy.node import Node
from sensor_msgs.msg import LaserScan
from std_msgs.msg import Float32
import numpy as np


class ObstacleDetector(Node):
    def __init__(self):
        super().__init__('obstacle_detector')
        
        # Parameters
        self.declare_parameter('fov_deg', 60.0)
        self.declare_parameter('warning_distance', 1.0)
        self.declare_parameter('danger_distance', 0.5)
        
        self.fov = self.get_parameter('fov_deg').value
        self.warning_dist = self.get_parameter('warning_distance').value
        self.danger_dist = self.get_parameter('danger_distance').value
        
        # Subscriber
        self.scan_sub = self.create_subscription(
            LaserScan,
            '/scan',
            self.scan_callback,
            10
        )
        
        # Publisher
        self.dist_pub = self.create_publisher(Float32, '/min_distance', 10)
        
        self.get_logger().info('Obstacle detector started')
    
    def scan_callback(self, msg):
        # 전처리
        ranges = np.array(msg.ranges)
        ranges = np.where(np.isinf(ranges), msg.range_max, ranges)
        ranges = np.where(np.isnan(ranges), msg.range_max, ranges)
        
        # 전방 영역 추출
        fov_rad = np.radians(self.fov)
        start_idx = int((-fov_rad/2 - msg.angle_min) / msg.angle_increment)
        end_idx = int((fov_rad/2 - msg.angle_min) / msg.angle_increment)
        
        front_ranges = ranges[start_idx:end_idx+1]
        min_distance = np.min(front_ranges)
        
        # 거리 발행
        dist_msg = Float32()
        dist_msg.data = float(min_distance)
        self.dist_pub.publish(dist_msg)
        
        # 경고
        if min_distance < self.danger_dist:
            self.get_logger().warn(f'DANGER! Distance: {min_distance:.2f}m')
        elif min_distance < self.warning_dist:
            self.get_logger().info(f'Warning: Distance: {min_distance:.2f}m')


def main(args=None):
    rclpy.init(args=args)
    node = ObstacleDetector()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()


if __name__ == '__main__':
    main()
```

### 6.2 실행 및 테스트

```bash
# 빌드
cd ~/f1tenth_ws
colcon build --packages-select my_lidar_pkg

# 실행
ros2 run my_lidar_pkg obstacle_detector

# 다른 터미널에서 확인
ros2 topic echo /min_distance
```

---

## 7. Gap 찾기 (Follow the Gap 준비)

### 7.1 Gap 정의

Gap은 LiDAR 스캔에서 장애물이 없는 열린 공간입니다.

```python
def find_gaps(ranges, threshold=2.0, min_gap_size=10):
    """임계값보다 큰 연속 영역 찾기."""
    gaps = []
    in_gap = False
    gap_start = 0
    
    for i, r in enumerate(ranges):
        if r > threshold:
            if not in_gap:
                gap_start = i
                in_gap = True
        else:
            if in_gap:
                if i - gap_start >= min_gap_size:
                    gaps.append((gap_start, i - 1))
                in_gap = False
    
    # 마지막 gap 처리
    if in_gap and len(ranges) - gap_start >= min_gap_size:
        gaps.append((gap_start, len(ranges) - 1))
    
    return gaps
```

### 7.2 가장 큰 Gap 찾기

```python
def find_largest_gap(ranges, threshold=2.0):
    """가장 큰 gap의 중심 인덱스 반환."""
    gaps = find_gaps(ranges, threshold)
    
    if not gaps:
        return len(ranges) // 2  # 기본값: 전방
    
    # 가장 큰 gap 찾기
    largest_gap = max(gaps, key=lambda g: g[1] - g[0])
    
    # 중심 인덱스
    center_idx = (largest_gap[0] + largest_gap[1]) // 2
    
    return center_idx
```

---

## 8. 실습 과제

### 과제 1: 방향별 최소 거리

전방, 좌측, 우측 각각의 최소 거리를 발행하는 노드를 작성하세요.

토픽:
- `/distance/front`
- `/distance/left`
- `/distance/right`

### 과제 2: 장애물 방향 표시

가장 가까운 장애물의 방향을 문자열로 발행하세요.

- "FRONT": -30° ~ +30°
- "LEFT": +30° ~ +90°
- "RIGHT": -90° ~ -30°
- "CLEAR": 2m 이상

### 과제 3: Gap 시각화

가장 큰 gap의 방향을 `/cmd_vel`로 발행하여 turtlesim에서 시각화하세요.

---

## 9. 검증 체크리스트

- [ ] LaserScan 메시지 구조 이해
- [ ] inf/NaN 전처리 구현
- [ ] 전방 최소 거리 계산 노드 작성
- [ ] `/min_distance` 토픽 발행 확인
- [ ] rqt_plot으로 거리 그래프 확인

---

*LiDAR 데이터 처리는 자율주행의 핵심입니다. 정확한 거리 계산이 안전한 주행의 기반이 됩니다!*
