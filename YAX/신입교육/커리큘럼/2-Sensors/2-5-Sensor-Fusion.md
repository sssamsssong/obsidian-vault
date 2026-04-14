# 센서 융합 기초

> YAX F1TENTH 교육자료 | Phase 2: Sensors
> 
> 다중 센서 데이터 통합

---

## 1. 개요

### 학습 목표
1. 센서 융합의 개념과 필요성을 이해할 수 있다
2. 다중 센서 데이터를 시간 동기화할 수 있다
3. LiDAR와 카메라 데이터를 함께 활용할 수 있다

### 예상 소요 시간
- 전체: 1.5시간

### 전제 조건
- LiDAR 처리 완료 (02-LiDAR-Processing.md)
- 카메라 처리 완료 (04-Camera-Processing.md)

---

## 2. 센서 융합이란?

### 2.1 정의

센서 융합(Sensor Fusion)은 여러 센서의 데이터를 결합하여 더 정확하고 신뢰성 있는 정보를 얻는 기술입니다.

### 2.2 왜 필요한가?

| 센서 | 장점 | 단점 |
|------|------|------|
| **LiDAR** | 정확한 거리, 조명 무관 | 색상/질감 없음 |
| **카메라** | 풍부한 시각 정보 | 거리 정보 부족, 조명 영향 |
| **IMU** | 빠른 반응, 드리프트 감지 | 누적 오차 |
| **Odometry** | 연속적 이동 추적 | 휠 슬립 오차 |

**융합 결과**: 각 센서의 장점을 결합하고 단점을 보완

### 2.3 융합 수준

```
┌─────────────────────────────────────────────────────────────┐
│  Low-Level Fusion (데이터 수준)                              │
│  - 원시 데이터 결합                                          │
│  - 예: LiDAR 포인트에 카메라 색상 매핑                        │
├─────────────────────────────────────────────────────────────┤
│  Mid-Level Fusion (특징 수준)                                │
│  - 추출된 특징 결합                                          │
│  - 예: 장애물 경계 + 색상 특징                               │
├─────────────────────────────────────────────────────────────┤
│  High-Level Fusion (결정 수준)                               │
│  - 각 센서의 결과 결합                                       │
│  - 예: LiDAR 장애물 감지 + 카메라 객체 인식                   │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. 시간 동기화

### 3.1 문제점

센서들은 서로 다른 시간에 데이터를 발행합니다:
- LiDAR: 40Hz (25ms 간격)
- 카메라: 30Hz (33ms 간격)
- IMU: 100Hz (10ms 간격)

### 3.2 message_filters 사용

```python
from message_filters import Subscriber, ApproximateTimeSynchronizer
from sensor_msgs.msg import LaserScan, Image

class SensorFusion(Node):
    def __init__(self):
        super().__init__('sensor_fusion')
        
        # 개별 Subscriber
        self.scan_sub = Subscriber(self, LaserScan, '/scan')
        self.image_sub = Subscriber(self, Image, '/camera/color/image_raw')
        
        # 시간 동기화
        self.sync = ApproximateTimeSynchronizer(
            [self.scan_sub, self.image_sub],
            queue_size=10,
            slop=0.1  # 100ms 허용 오차
        )
        self.sync.registerCallback(self.synced_callback)
    
    def synced_callback(self, scan_msg, image_msg):
        """동기화된 LiDAR + 카메라 데이터 처리."""
        scan_time = scan_msg.header.stamp
        image_time = image_msg.header.stamp
        
        time_diff = abs(
            (scan_time.sec + scan_time.nanosec * 1e-9) -
            (image_time.sec + image_time.nanosec * 1e-9)
        )
        
        self.get_logger().info(f'Time diff: {time_diff*1000:.1f}ms')
        
        # 동기화된 데이터 처리
        self.process_fused_data(scan_msg, image_msg)
```

### 3.3 ExactTime vs ApproximateTime

| 방식 | 설명 | 용도 |
|------|------|------|
| `ExactTimeSynchronizer` | 정확히 같은 타임스탬프 | 동일 소스, 시뮬레이션 |
| `ApproximateTimeSynchronizer` | 근사 타임스탬프 (slop 허용) | 실제 센서 |

---

## 4. 좌표 변환 (TF2)

### 4.1 TF2 개념

서로 다른 위치에 장착된 센서들의 데이터를 하나의 좌표계로 변환합니다.

```
               [base_link]
                    │
        ┌───────────┼───────────┐
        │           │           │
   [laser]     [camera]      [imu]
```

### 4.2 TF Listener

```python
from tf2_ros import TransformListener, Buffer
from geometry_msgs.msg import TransformStamped

class TFExample(Node):
    def __init__(self):
        super().__init__('tf_example')
        
        self.tf_buffer = Buffer()
        self.tf_listener = TransformListener(self.tf_buffer, self)
    
    def get_transform(self, target_frame, source_frame):
        """좌표 변환 가져오기."""
        try:
            transform = self.tf_buffer.lookup_transform(
                target_frame,
                source_frame,
                rclpy.time.Time(),  # 최신
                timeout=rclpy.duration.Duration(seconds=1.0)
            )
            return transform
        except Exception as e:
            self.get_logger().error(f'TF error: {e}')
            return None
```

### 4.3 포인트 변환

```python
from tf2_geometry_msgs import do_transform_point
from geometry_msgs.msg import PointStamped

def transform_point(self, point, target_frame):
    """포인트를 다른 프레임으로 변환."""
    point_stamped = PointStamped()
    point_stamped.header.frame_id = 'laser'
    point_stamped.point = point
    
    transform = self.get_transform(target_frame, 'laser')
    if transform:
        transformed = do_transform_point(point_stamped, transform)
        return transformed.point
    return None
```

---

## 5. LiDAR + 카메라 융합 예제

### 5.1 기본 구조

```python
#!/usr/bin/env python3
"""
LiDAR + Camera Fusion Node
"""

import rclpy
from rclpy.node import Node
from sensor_msgs.msg import LaserScan, Image
from message_filters import Subscriber, ApproximateTimeSynchronizer
from cv_bridge import CvBridge
import numpy as np
import cv2


class LidarCameraFusion(Node):
    def __init__(self):
        super().__init__('lidar_camera_fusion')
        self.bridge = CvBridge()
        
        # Subscribers
        self.scan_sub = Subscriber(self, LaserScan, '/scan')
        self.image_sub = Subscriber(self, Image, '/camera/color/image_raw')
        
        # Time synchronization
        self.sync = ApproximateTimeSynchronizer(
            [self.scan_sub, self.image_sub],
            queue_size=10,
            slop=0.1
        )
        self.sync.registerCallback(self.fusion_callback)
        
        # Publisher
        self.fused_pub = self.create_publisher(Image, '/fused_image', 10)
        
        self.get_logger().info('Fusion node started')
    
    def fusion_callback(self, scan_msg, image_msg):
        # LiDAR 처리
        ranges = np.array(scan_msg.ranges)
        ranges = np.where(np.isinf(ranges), scan_msg.range_max, ranges)
        
        min_distance = np.min(ranges)
        min_angle = scan_msg.angle_min + np.argmin(ranges) * scan_msg.angle_increment
        
        # 이미지 처리
        cv_image = self.bridge.imgmsg_to_cv2(image_msg, 'bgr8')
        
        # 융합: 거리 정보를 이미지에 표시
        h, w = cv_image.shape[:2]
        
        # 거리 텍스트 표시
        text = f'Min: {min_distance:.2f}m at {np.degrees(min_angle):.0f}deg'
        cv2.putText(cv_image, text, (10, 30), 
                    cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 255, 0), 2)
        
        # 경고 표시
        if min_distance < 0.5:
            cv2.rectangle(cv_image, (0, 0), (w, h), (0, 0, 255), 10)
            cv2.putText(cv_image, 'DANGER!', (w//2-80, h//2),
                        cv2.FONT_HERSHEY_SIMPLEX, 2, (0, 0, 255), 3)
        
        # 발행
        fused_msg = self.bridge.cv2_to_imgmsg(cv_image, 'bgr8')
        fused_msg.header = image_msg.header
        self.fused_pub.publish(fused_msg)


def main(args=None):
    rclpy.init(args=args)
    node = LidarCameraFusion()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()
```

---

## 6. 융합 전략

### 6.1 보완적 융합

각 센서의 장점 활용:
- LiDAR: 정확한 거리 정보
- 카메라: 객체 인식, 차선 감지

### 6.2 중복적 융합

동일 정보의 신뢰도 향상:
- LiDAR 장애물 + 카메라 Depth 장애물
- 교차 검증으로 오탐 감소

### 6.3 F1TENTH 적용

```
┌─────────────────────────────────────────────────────────────┐
│                    센서 융합 파이프라인                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  [LiDAR] ──► 장애물 거리/각도 ──┐                            │
│                                 ├──► 융합 판단 ──► 제어 명령  │
│  [Camera] ──► 장애물 감지 ──────┘                            │
│                                                             │
│  예: LiDAR 전방 1m 내 장애물 + 카메라 확인 = 정지/회피        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 7. 실습 과제

### 과제: 다중 센서 경고 시스템

LiDAR와 카메라를 모두 사용하여:
1. LiDAR: 1m 이내 장애물 감지
2. 카메라: 이미지에 장애물 방향 화살표 표시
3. 양쪽 모두 감지 시 경고 레벨 상승

---

## 8. 검증 체크리스트

- [ ] message_filters로 동기화 구현
- [ ] LiDAR + Camera 콜백 동시 수신
- [ ] 융합 데이터 발행
- [ ] TF2 기본 개념 이해

---

*센서 융합은 강력한 자율주행 시스템의 핵심입니다!*
