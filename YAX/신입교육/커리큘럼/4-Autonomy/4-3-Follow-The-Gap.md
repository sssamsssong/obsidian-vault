# Follow the Gap 알고리즘

> YAX F1TENTH 교육자료 | Phase 4: Autonomy
> 
> LiDAR 기반 반응형 장애물 회피

---

## 1. 개요

### 학습 목표
1. Follow the Gap (FTG) 알고리즘의 원리를 이해할 수 있다
2. LiDAR 데이터를 전처리하여 안전 영역을 탐지할 수 있다
3. 실시간 장애물 회피 시스템을 구현할 수 있다

### 예상 소요 시간
- 전체: 3시간

### 전제 조건
- Wall Following 완료
- LiDAR 데이터 처리 이해
- NumPy 기초

---

## 2. 알고리즘 개요

### 2.1 Follow the Gap이란?

FTG는 LiDAR 스캔에서 **가장 넓은 빈 공간(Gap)** 을 찾아 그 방향으로 주행하는 반응형 알고리즘입니다.

```
         장애물              장애물
           ██                ██
           ██                ██
           ██    Gap!        ██
           ██  ◄────────►    ██
           ██                ██
           ██                ██
                   ↑
              ┌─────────┐
              │ F1TENTH │
              └─────────┘
```

### 2.2 왜 FTG인가?

| 장점 | 설명 |
|------|------|
| 지도 불필요 | 실시간 LiDAR 데이터만 사용 |
| 동적 장애물 대응 | 움직이는 장애물도 회피 |
| 구현 간단 | 5단계 알고리즘 |
| 속도 우수 | 적극적인 빈 공간 활용 |

### 2.3 Wall Following vs FTG

| 항목 | Wall Following | Follow the Gap |
|------|----------------|----------------|
| 목표 | 벽과 거리 유지 | 빈 공간으로 이동 |
| 장애물 | 대응 불가 | 회피 가능 |
| 속도 | 보수적 | 공격적 |
| 난이도 | 쉬움 | 중간 |

---

## 3. 알고리즘 5단계

FTG는 다음 5단계로 구성됩니다:

```
┌─────────────────────────────────────────────────────────────┐
│  Step 1: 전처리 (Preprocessing)                              │
│  - inf, nan 값 처리                                          │
│  - 노이즈 필터링                                             │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 2: 최근접점 탐지 (Find Closest Point)                  │
│  - 가장 가까운 장애물 인덱스 찾기                            │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 3: 안전 버블 생성 (Create Safety Bubble)               │
│  - 최근접점 주변을 0으로 마스킹                              │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 4: 최대 Gap 탐색 (Find Max Gap)                        │
│  - 0이 아닌 연속 구간 중 가장 긴 것                          │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 5: 최적점 선택 및 조향 (Select Best Point)             │
│  - Gap 내에서 가장 먼 점으로 조향                            │
└─────────────────────────────────────────────────────────────┘
```

---

## 4. 단계별 구현

### 4.1 Step 1: 전처리

LiDAR 데이터의 무효값을 처리합니다.

```python
def preprocess_lidar(self, ranges: np.ndarray) -> np.ndarray:
    """
    LiDAR 데이터 전처리.
    
    Args:
        ranges: 원본 LiDAR 거리 배열
        
    Returns:
        전처리된 거리 배열
    """
    proc = np.array(ranges, dtype=np.float32)
    
    # inf 값을 최대 범위로 대체
    proc[np.isinf(proc)] = self.max_range
    
    # nan 값을 0으로 대체 (장애물 취급)
    proc[np.isnan(proc)] = 0.0
    
    # 최소값 이하를 0으로 (너무 가까운 것은 장애물)
    proc[proc < self.min_range] = 0.0
    
    return proc
```

### 4.2 Step 2: 최근접점 탐지

가장 가까운 장애물을 찾습니다.

```python
def find_closest_point(self, ranges: np.ndarray) -> int:
    """
    가장 가까운 점의 인덱스 반환.
    
    Args:
        ranges: 전처리된 거리 배열
        
    Returns:
        최근접점 인덱스
    """
    # 0이 아닌 값 중 최소값의 인덱스
    valid_ranges = np.where(ranges > 0, ranges, np.inf)
    return int(np.argmin(valid_ranges))
```

### 4.3 Step 3: 안전 버블 생성

최근접점 주변에 "버블"을 만들어 해당 영역을 장애물로 표시합니다.

```
        전                    후
  
   3.2  2.8  0.5  2.1  3.5    3.2  2.8  0.0  0.0  0.0  2.1  3.5
              ↑                         └─────────┘
           최근접점                    안전 버블 (0으로 마스킹)
```

```python
def create_safety_bubble(self, ranges: np.ndarray, 
                         closest_idx: int,
                         angle_increment: float) -> np.ndarray:
    """
    최근접점 주변에 안전 버블 생성.
    
    Args:
        ranges: 거리 배열
        closest_idx: 최근접점 인덱스
        angle_increment: LiDAR 각도 해상도 (rad)
        
    Returns:
        버블이 적용된 거리 배열
    """
    result = ranges.copy()
    
    # 버블 반경에 해당하는 각도 계산
    closest_dist = ranges[closest_idx]
    if closest_dist <= 0:
        closest_dist = 0.1  # 0으로 나누기 방지
    
    # 버블 반경 / 거리 = 버블이 차지하는 각도 (라디안)
    bubble_angle = np.arctan2(self.bubble_radius, closest_dist)
    
    # 각도를 인덱스로 변환
    bubble_size = int(bubble_angle / angle_increment)
    
    # 버블 범위 계산
    start_idx = max(0, closest_idx - bubble_size)
    end_idx = min(len(ranges), closest_idx + bubble_size + 1)
    
    # 버블 영역을 0으로 마스킹
    result[start_idx:end_idx] = 0.0
    
    return result
```

### 4.4 Step 4: 최대 Gap 탐색

0이 아닌 연속 구간 중 가장 긴 것을 찾습니다.

```
     ranges: [0, 0, 3.2, 2.8, 2.5, 0, 0, 4.1, 3.9, 4.0, 4.2, 0]
                  └────────────┘     └──────────────────┘
                     Gap 1 (3)            Gap 2 (4) ← 최대 Gap
```

```python
def find_max_gap(self, ranges: np.ndarray) -> Tuple[int, int]:
    """
    가장 넓은 Gap의 시작/끝 인덱스 반환.
    
    Args:
        ranges: 버블이 적용된 거리 배열
        
    Returns:
        (start_idx, end_idx) 튜플
    """
    # 유효한 인덱스들 (0보다 큰 값)
    valid_indices = np.where(ranges > self.gap_threshold)[0]
    
    if len(valid_indices) == 0:
        # Gap이 없으면 전방 중앙
        mid = len(ranges) // 2
        return mid, mid
    
    # 연속된 인덱스 그룹으로 분리
    gaps = np.split(valid_indices, 
                    np.where(np.diff(valid_indices) != 1)[0] + 1)
    
    # 가장 긴 Gap 선택
    longest_gap = max(gaps, key=len)
    
    return int(longest_gap[0]), int(longest_gap[-1])
```

### 4.5 Step 5: 최적점 선택

Gap 내에서 가장 먼 점을 선택합니다.

```python
def find_best_point(self, ranges: np.ndarray, 
                    start_idx: int, end_idx: int) -> int:
    """
    Gap 내에서 최적 조향점 선택.
    
    선택 전략:
    - 'farthest': 가장 먼 점 (기본값, 공격적)
    - 'center': Gap 중앙 (안정적)
    
    Args:
        ranges: 거리 배열
        start_idx: Gap 시작 인덱스
        end_idx: Gap 끝 인덱스
        
    Returns:
        최적점 인덱스
    """
    if self.strategy == 'farthest':
        # Gap 내 가장 먼 점
        gap_ranges = ranges[start_idx:end_idx + 1]
        best_local = np.argmax(gap_ranges)
        return start_idx + best_local
    
    else:  # 'center'
        # Gap 중앙
        return (start_idx + end_idx) // 2
```

---

## 5. 완전한 ROS2 노드

### 5.1 FTG 노드 구현

```python
#!/usr/bin/env python3
"""
Follow the Gap Node

LiDAR 기반 반응형 장애물 회피 알고리즘
"""

import rclpy
from rclpy.node import Node
from sensor_msgs.msg import LaserScan
from ackermann_msgs.msg import AckermannDriveStamped
import numpy as np
from typing import Tuple


class FollowTheGap(Node):
    def __init__(self):
        super().__init__('follow_the_gap')
        
        # === 파라미터 선언 ===
        self.declare_parameter('bubble_radius', 0.3)
        self.declare_parameter('gap_threshold', 0.1)
        self.declare_parameter('max_speed', 3.0)
        self.declare_parameter('min_speed', 1.0)
        self.declare_parameter('max_steering', 0.4)
        self.declare_parameter('min_range', 0.1)
        self.declare_parameter('max_range', 10.0)
        self.declare_parameter('fov_deg', 180.0)  # 시야각 (도)
        self.declare_parameter('strategy', 'farthest')  # 'farthest' or 'center'
        
        # 파라미터 로드
        self.bubble_radius = self.get_parameter('bubble_radius').value
        self.gap_threshold = self.get_parameter('gap_threshold').value
        self.max_speed = self.get_parameter('max_speed').value
        self.min_speed = self.get_parameter('min_speed').value
        self.max_steering = self.get_parameter('max_steering').value
        self.min_range = self.get_parameter('min_range').value
        self.max_range = self.get_parameter('max_range').value
        self.fov_deg = self.get_parameter('fov_deg').value
        self.strategy = self.get_parameter('strategy').value
        
        # === Subscriber / Publisher ===
        self.scan_sub = self.create_subscription(
            LaserScan,
            '/scan',
            self.scan_callback,
            10
        )
        
        self.drive_pub = self.create_publisher(
            AckermannDriveStamped,
            '/drive',
            10
        )
        
        self.get_logger().info('Follow the Gap node started')
        self.get_logger().info(f'  Bubble radius: {self.bubble_radius}m')
        self.get_logger().info(f'  Max speed: {self.max_speed}m/s')
        self.get_logger().info(f'  Strategy: {self.strategy}')
    
    def get_fov_indices(self, msg: LaserScan) -> Tuple[int, int]:
        """
        시야각에 해당하는 LiDAR 인덱스 범위 계산.
        """
        fov_rad = np.radians(self.fov_deg)
        half_fov = fov_rad / 2
        
        # 전방 중심 기준 좌우 시야각
        start_angle = -half_fov
        end_angle = half_fov
        
        start_idx = int((start_angle - msg.angle_min) / msg.angle_increment)
        end_idx = int((end_angle - msg.angle_min) / msg.angle_increment)
        
        # 범위 제한
        start_idx = max(0, start_idx)
        end_idx = min(len(msg.ranges) - 1, end_idx)
        
        return start_idx, end_idx
    
    def preprocess_lidar(self, ranges: np.ndarray) -> np.ndarray:
        """LiDAR 데이터 전처리."""
        proc = np.array(ranges, dtype=np.float32)
        proc[np.isinf(proc)] = self.max_range
        proc[np.isnan(proc)] = 0.0
        proc[proc < self.min_range] = 0.0
        proc[proc > self.max_range] = self.max_range
        return proc
    
    def find_closest_point(self, ranges: np.ndarray) -> int:
        """최근접점 인덱스 반환."""
        valid_ranges = np.where(ranges > 0, ranges, np.inf)
        return int(np.argmin(valid_ranges))
    
    def create_safety_bubble(self, ranges: np.ndarray, 
                             closest_idx: int,
                             angle_increment: float) -> np.ndarray:
        """안전 버블 생성."""
        result = ranges.copy()
        
        closest_dist = max(ranges[closest_idx], 0.1)
        bubble_angle = np.arctan2(self.bubble_radius, closest_dist)
        bubble_size = int(bubble_angle / angle_increment)
        
        start_idx = max(0, closest_idx - bubble_size)
        end_idx = min(len(ranges), closest_idx + bubble_size + 1)
        
        result[start_idx:end_idx] = 0.0
        return result
    
    def find_max_gap(self, ranges: np.ndarray) -> Tuple[int, int]:
        """최대 Gap 탐색."""
        valid_indices = np.where(ranges > self.gap_threshold)[0]
        
        if len(valid_indices) == 0:
            mid = len(ranges) // 2
            return mid, mid
        
        gaps = np.split(valid_indices, 
                        np.where(np.diff(valid_indices) != 1)[0] + 1)
        longest_gap = max(gaps, key=len)
        
        return int(longest_gap[0]), int(longest_gap[-1])
    
    def find_best_point(self, ranges: np.ndarray, 
                        start_idx: int, end_idx: int) -> int:
        """Gap 내 최적점 선택."""
        if self.strategy == 'farthest':
            gap_ranges = ranges[start_idx:end_idx + 1]
            best_local = np.argmax(gap_ranges)
            return start_idx + best_local
        else:
            return (start_idx + end_idx) // 2
    
    def calculate_steering(self, best_idx: int, 
                          fov_start: int, fov_end: int,
                          angle_min: float, 
                          angle_increment: float) -> float:
        """
        최적점의 조향각 계산.
        """
        # 인덱스 → 각도 변환
        best_angle = angle_min + best_idx * angle_increment
        
        # 조향각 제한
        steering = np.clip(best_angle, -self.max_steering, self.max_steering)
        
        return float(steering)
    
    def calculate_speed(self, steering: float, 
                        min_distance: float) -> float:
        """
        조향각과 최소 거리에 따른 속도 계산.
        """
        # 조향각이 클수록 감속
        steering_factor = 1.0 - (abs(steering) / self.max_steering) * 0.5
        
        # 장애물이 가까울수록 감속
        distance_factor = min(1.0, min_distance / 2.0)
        
        speed = self.max_speed * steering_factor * distance_factor
        speed = max(self.min_speed, speed)
        
        return float(speed)
    
    def scan_callback(self, msg: LaserScan):
        """메인 콜백: FTG 알고리즘 실행."""
        
        # 1. 시야각 범위 계산
        fov_start, fov_end = self.get_fov_indices(msg)
        
        # 2. 전처리 (시야각 범위만)
        full_ranges = self.preprocess_lidar(msg.ranges)
        ranges = full_ranges[fov_start:fov_end + 1]
        
        if len(ranges) == 0:
            return
        
        # 3. 최근접점 탐지
        closest_idx = self.find_closest_point(ranges)
        min_distance = ranges[closest_idx] if ranges[closest_idx] > 0 else self.max_range
        
        # 4. 안전 버블 생성
        ranges = self.create_safety_bubble(ranges, closest_idx, msg.angle_increment)
        
        # 5. 최대 Gap 탐색
        gap_start, gap_end = self.find_max_gap(ranges)
        
        # 6. 최적점 선택
        best_idx = self.find_best_point(ranges, gap_start, gap_end)
        
        # 7. 조향각 계산 (전체 인덱스로 변환)
        global_best_idx = fov_start + best_idx
        steering = self.calculate_steering(
            global_best_idx, fov_start, fov_end,
            msg.angle_min, msg.angle_increment
        )
        
        # 8. 속도 계산
        speed = self.calculate_speed(steering, min_distance)
        
        # 9. 명령 발행
        drive_msg = AckermannDriveStamped()
        drive_msg.header.stamp = self.get_clock().now().to_msg()
        drive_msg.header.frame_id = 'base_link'
        drive_msg.drive.speed = speed
        drive_msg.drive.steering_angle = steering
        
        self.drive_pub.publish(drive_msg)
        
        # 디버그
        self.get_logger().debug(
            f'Gap: [{gap_start}, {gap_end}], Best: {best_idx}, '
            f'Steer: {steering:.2f}, Speed: {speed:.2f}'
        )


def main(args=None):
    rclpy.init(args=args)
    node = FollowTheGap()
    
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    finally:
        node.destroy_node()
        rclpy.shutdown()


if __name__ == '__main__':
    main()
```

### 5.2 Launch 파일

```python
# launch/ftg_launch.py
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    return LaunchDescription([
        Node(
            package='f1tenth_autonomy',
            executable='follow_the_gap',
            name='follow_the_gap',
            parameters=[{
                'bubble_radius': 0.3,
                'gap_threshold': 0.1,
                'max_speed': 3.0,
                'min_speed': 1.0,
                'max_steering': 0.4,
                'fov_deg': 180.0,
                'strategy': 'farthest',
            }],
            output='screen'
        )
    ])
```

### 5.3 파라미터 설정 파일

```yaml
# config/ftg_params.yaml
follow_the_gap:
  ros__parameters:
    # 안전 설정
    bubble_radius: 0.3       # 안전 버블 반경 (m)
    gap_threshold: 0.1       # 유효 Gap 임계값 (m)
    min_range: 0.1           # LiDAR 최소 유효 거리
    max_range: 10.0          # LiDAR 최대 유효 거리
    
    # 속도 설정
    max_speed: 3.0           # 최대 속도 (m/s)
    min_speed: 1.0           # 최소 속도 (m/s)
    
    # 조향 설정
    max_steering: 0.4        # 최대 조향각 (rad)
    fov_deg: 180.0           # 시야각 (도)
    
    # 전략
    strategy: "farthest"     # 'farthest' or 'center'
```

---

## 6. 파라미터 튜닝

### 6.1 주요 파라미터 영향

| 파라미터 | 증가 시 | 감소 시 |
|----------|---------|---------|
| `bubble_radius` | 더 안전, 좁은 공간 통과 어려움 | 공격적, 충돌 위험 |
| `fov_deg` | 넓은 탐지, 측면 반응 | 전방 집중, 빠른 처리 |
| `max_speed` | 빠름, 반응 시간 부족 | 느림, 안정적 |
| `strategy=farthest` | 공격적 경로 | - |
| `strategy=center` | - | 안정적 경로 |

### 6.2 권장 시작값

**입문자용 (안전):**
```yaml
bubble_radius: 0.4
max_speed: 2.0
fov_deg: 160.0
strategy: "center"
```

**중급자용 (균형):**
```yaml
bubble_radius: 0.3
max_speed: 3.0
fov_deg: 180.0
strategy: "farthest"
```

**고급자용 (공격적):**
```yaml
bubble_radius: 0.2
max_speed: 4.0
fov_deg: 200.0
strategy: "farthest"
```

### 6.3 단계별 튜닝 프로세스

1. **안전 우선**: bubble_radius 크게, max_speed 낮게 시작
2. **속도 증가**: 안정되면 max_speed를 0.5씩 증가
3. **버블 축소**: 좁은 공간 통과 필요시 bubble_radius 감소
4. **전략 변경**: center → farthest로 변경하여 공격적 주행

---

## 7. 고급 기법

### 7.1 다중 버블

여러 장애물에 대해 각각 버블 생성:

```python
def create_multiple_bubbles(self, ranges: np.ndarray, 
                            angle_increment: float,
                            num_bubbles: int = 3) -> np.ndarray:
    """
    가장 가까운 N개 장애물에 버블 생성.
    """
    result = ranges.copy()
    temp_ranges = ranges.copy()
    
    for _ in range(num_bubbles):
        closest_idx = self.find_closest_point(temp_ranges)
        if temp_ranges[closest_idx] <= 0:
            break
        
        result = self.create_safety_bubble(
            result, closest_idx, angle_increment
        )
        temp_ranges[closest_idx] = 0  # 다음 최근접점 찾기 위해
    
    return result
```

### 7.2 Gap 가중치

Gap 선택 시 거리와 각도를 고려:

```python
def score_gaps(self, ranges: np.ndarray, 
               gaps: list,
               center_idx: int) -> list:
    """
    각 Gap에 점수 부여.
    
    점수 = Gap 너비 * 평균 거리 * 중앙 선호도
    """
    scored_gaps = []
    
    for gap in gaps:
        width = len(gap)
        avg_distance = np.mean(ranges[gap[0]:gap[-1]+1])
        
        # 중앙에 가까울수록 높은 점수
        gap_center = (gap[0] + gap[-1]) // 2
        center_factor = 1.0 - abs(gap_center - center_idx) / center_idx * 0.3
        
        score = width * avg_distance * center_factor
        scored_gaps.append((gap, score))
    
    return sorted(scored_gaps, key=lambda x: x[1], reverse=True)
```

### 7.3 이동 평균 필터

LiDAR 노이즈 감소:

```python
def smooth_ranges(self, ranges: np.ndarray, 
                  window_size: int = 5) -> np.ndarray:
    """
    이동 평균으로 노이즈 필터링.
    """
    kernel = np.ones(window_size) / window_size
    smoothed = np.convolve(ranges, kernel, mode='same')
    return smoothed
```

---

## 8. 실습

### 실습 1: 기본 FTG 동작

1. MORAI 시뮬레이터에서 노드 실행
2. 장애물 없는 트랙에서 기본 동작 확인
3. rqt_plot으로 steering, speed 모니터링

```bash
# 터미널 1: 시뮬레이터
ros2 launch morai_sim track.launch.py

# 터미널 2: FTG 노드
ros2 launch f1tenth_autonomy ftg_launch.py

# 터미널 3: 모니터링
ros2 topic echo /drive
```

### 실습 2: 장애물 회피

1. 시뮬레이터에 장애물 배치
2. 회피 동작 확인
3. bubble_radius 조절하며 영향 관찰

### 실습 3: 파라미터 실험

| 실험 | bubble_radius | max_speed | strategy | 결과 |
|------|---------------|-----------|----------|------|
| 1 | 0.4 | 2.0 | center | |
| 2 | 0.3 | 2.0 | center | |
| 3 | 0.3 | 3.0 | center | |
| 4 | 0.3 | 3.0 | farthest | |
| 5 | 0.2 | 4.0 | farthest | |

### 실습 4: 랩타임 측정

1. 트랙 3바퀴 완주
2. 각 파라미터 조합별 랩타임 기록
3. 충돌 없이 가장 빠른 설정 찾기

---

## 9. 문제 해결

### 자주 발생하는 문제

| 증상 | 원인 | 해결책 |
|------|------|--------|
| 벽에 충돌 | bubble_radius 너무 작음 | bubble_radius 증가 |
| 좁은 공간 통과 못함 | bubble_radius 너무 큼 | bubble_radius 감소 |
| 진동 (좌우 흔들림) | 속도 대비 반응 빠름 | max_speed 감소 또는 fov_deg 감소 |
| 반응 느림 | fov_deg 너무 작음 | fov_deg 증가 |
| Gap 못 찾음 | gap_threshold 너무 높음 | gap_threshold 감소 |
| 멈춤 | 모든 방향에 장애물 | min_speed 설정, 비상 정지 로직 |

### 디버깅 팁

1. **LiDAR 데이터 시각화**
   ```bash
   ros2 run rviz2 rviz2
   # LaserScan 토픽 추가
   ```

2. **Gap 시각화**
   - Marker 토픽으로 Gap 영역 표시
   - 선택된 최적점 표시

3. **로그 레벨 변경**
   ```bash
   ros2 run f1tenth_autonomy follow_the_gap --ros-args --log-level debug
   ```

---

## 10. 검증 체크리스트

- [ ] 전처리: inf/nan 값 정상 처리
- [ ] 최근접점: 올바르게 탐지
- [ ] 안전 버블: 장애물 주변 마스킹
- [ ] Gap 탐색: 가장 넓은 Gap 선택
- [ ] 조향: Gap 방향으로 조향
- [ ] 속도 조절: 조향각에 따른 감속
- [ ] 장애물 회피: 동적 장애물 대응
- [ ] 트랙 완주: 충돌 없이 완주

---

*Follow the Gap은 F1TENTH 대회에서 가장 많이 사용되는 알고리즘입니다!*
