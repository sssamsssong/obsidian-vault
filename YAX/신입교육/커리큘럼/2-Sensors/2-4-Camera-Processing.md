# 카메라 데이터 처리

> YAX F1TENTH 교육자료 | Phase 2: Sensors
> 
> RGB 및 Depth 이미지 처리

---

## 1. 개요

### 학습 목표
1. ROS2 이미지 메시지를 처리할 수 있다
2. OpenCV와 cv_bridge를 사용할 수 있다
3. Depth 이미지에서 거리를 계산할 수 있다
4. 간단한 이미지 처리를 수행할 수 있다

### 예상 소요 시간
- 전체: 1.5시간

### 전제 조건
- RealSense 카메라 설정 완료 (03-Camera-Setup.md)
- Python OpenCV 기초

---

## 2. 필수 패키지 설치

```bash
# cv_bridge (ROS2 ↔ OpenCV 변환)
sudo apt install ros-humble-cv-bridge

# Python OpenCV
pip3 install opencv-python

# 이미지 전송 최적화
sudo apt install ros-humble-image-transport
sudo apt install ros-humble-compressed-image-transport
```

---

## 3. 이미지 메시지 구조

### 3.1 sensor_msgs/Image

```bash
ros2 interface show sensor_msgs/msg/Image
```

```
std_msgs/Header header
uint32 height          # 이미지 높이
uint32 width           # 이미지 너비
string encoding        # 픽셀 인코딩 (rgb8, bgr8, 16UC1, etc.)
uint8 is_bigendian     # 엔디안
uint32 step            # 행당 바이트 수
uint8[] data           # 이미지 데이터
```

### 3.2 주요 인코딩

| 인코딩 | 설명 | 용도 |
|--------|------|------|
| `rgb8` | 8비트 RGB | 컬러 이미지 |
| `bgr8` | 8비트 BGR | OpenCV 기본 |
| `mono8` | 8비트 그레이스케일 | 흑백 이미지 |
| `16UC1` | 16비트 단일 채널 | Depth 이미지 |
| `32FC1` | 32비트 float | Depth (미터 단위) |

---

## 4. cv_bridge 사용법

### 4.1 ROS → OpenCV 변환

```python
from cv_bridge import CvBridge
import cv2

class ImageProcessor(Node):
    def __init__(self):
        super().__init__('image_processor')
        self.bridge = CvBridge()
        
        self.sub = self.create_subscription(
            Image,
            '/camera/color/image_raw',
            self.image_callback,
            10
        )
    
    def image_callback(self, msg):
        # ROS Image → OpenCV (numpy array)
        cv_image = self.bridge.imgmsg_to_cv2(msg, desired_encoding='bgr8')
        
        # OpenCV로 처리
        gray = cv2.cvtColor(cv_image, cv2.COLOR_BGR2GRAY)
        
        # 화면에 표시 (디버깅용)
        cv2.imshow('Image', cv_image)
        cv2.waitKey(1)
```

### 4.2 OpenCV → ROS 변환

```python
def publish_processed_image(self, cv_image):
    # OpenCV → ROS Image
    ros_image = self.bridge.cv2_to_imgmsg(cv_image, encoding='bgr8')
    self.image_pub.publish(ros_image)
```

---

## 5. RGB 이미지 처리

### 5.1 기본 이미지 처리 노드

```python
#!/usr/bin/env python3
"""
RGB Image Processor
"""

import rclpy
from rclpy.node import Node
from sensor_msgs.msg import Image
from cv_bridge import CvBridge
import cv2
import numpy as np


class RGBProcessor(Node):
    def __init__(self):
        super().__init__('rgb_processor')
        self.bridge = CvBridge()
        
        # Subscriber
        self.sub = self.create_subscription(
            Image,
            '/camera/color/image_raw',
            self.image_callback,
            10
        )
        
        # Publisher
        self.pub = self.create_publisher(
            Image,
            '/camera/processed',
            10
        )
        
        self.get_logger().info('RGB processor started')
    
    def image_callback(self, msg):
        # ROS → OpenCV
        cv_image = self.bridge.imgmsg_to_cv2(msg, 'bgr8')
        
        # 처리: 예시로 엣지 검출
        gray = cv2.cvtColor(cv_image, cv2.COLOR_BGR2GRAY)
        edges = cv2.Canny(gray, 50, 150)
        
        # 결과를 컬러로 변환
        edges_color = cv2.cvtColor(edges, cv2.COLOR_GRAY2BGR)
        
        # OpenCV → ROS
        processed_msg = self.bridge.cv2_to_imgmsg(edges_color, 'bgr8')
        processed_msg.header = msg.header
        self.pub.publish(processed_msg)


def main(args=None):
    rclpy.init(args=args)
    node = RGBProcessor()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()
```

### 5.2 색상 감지

```python
def detect_color(self, cv_image, color='red'):
    """특정 색상 영역 감지."""
    hsv = cv2.cvtColor(cv_image, cv2.COLOR_BGR2HSV)
    
    if color == 'red':
        lower = np.array([0, 100, 100])
        upper = np.array([10, 255, 255])
    elif color == 'green':
        lower = np.array([40, 100, 100])
        upper = np.array([80, 255, 255])
    elif color == 'blue':
        lower = np.array([100, 100, 100])
        upper = np.array([140, 255, 255])
    
    mask = cv2.inRange(hsv, lower, upper)
    return mask
```

---

## 6. Depth 이미지 처리

### 6.1 Depth 데이터 읽기

```python
def depth_callback(self, msg):
    # ROS → OpenCV (16비트 단일 채널)
    depth_image = self.bridge.imgmsg_to_cv2(msg, desired_encoding='passthrough')
    
    # depth_image: uint16, 단위는 밀리미터
    # 미터로 변환
    depth_meters = depth_image.astype(np.float32) / 1000.0
    
    # 중앙 픽셀 거리
    center_y = depth_image.shape[0] // 2
    center_x = depth_image.shape[1] // 2
    center_distance = depth_meters[center_y, center_x]
    
    self.get_logger().info(f'Center distance: {center_distance:.2f}m')
```

### 6.2 특정 영역 평균 거리

```python
def get_region_distance(self, depth_meters, x, y, size=10):
    """(x, y) 주변 영역의 평균 거리."""
    region = depth_meters[y-size:y+size, x-size:x+size]
    
    # 유효한 값만 사용 (0과 inf 제외)
    valid = region[(region > 0) & (region < 10)]
    
    if len(valid) > 0:
        return np.mean(valid)
    else:
        return float('inf')
```

### 6.3 Depth 시각화

```python
def visualize_depth(self, depth_image):
    """Depth 이미지를 컬러맵으로 시각화."""
    # 정규화 (0-255)
    depth_normalized = cv2.normalize(
        depth_image, None, 0, 255, cv2.NORM_MINMAX, cv2.CV_8U
    )
    
    # 컬러맵 적용
    depth_colormap = cv2.applyColorMap(depth_normalized, cv2.COLORMAP_JET)
    
    return depth_colormap
```

---

## 7. RGB-D 통합 처리

### 7.1 동기화된 처리

```python
from message_filters import Subscriber, ApproximateTimeSynchronizer

class RGBDProcessor(Node):
    def __init__(self):
        super().__init__('rgbd_processor')
        self.bridge = CvBridge()
        
        # 동기화된 Subscriber
        self.rgb_sub = Subscriber(self, Image, '/camera/color/image_raw')
        self.depth_sub = Subscriber(self, Image, '/camera/depth/image_rect_raw')
        
        # 시간 동기화 (0.1초 허용)
        self.ts = ApproximateTimeSynchronizer(
            [self.rgb_sub, self.depth_sub],
            queue_size=10,
            slop=0.1
        )
        self.ts.registerCallback(self.synced_callback)
    
    def synced_callback(self, rgb_msg, depth_msg):
        # 동기화된 RGB와 Depth 처리
        rgb_image = self.bridge.imgmsg_to_cv2(rgb_msg, 'bgr8')
        depth_image = self.bridge.imgmsg_to_cv2(depth_msg, 'passthrough')
        
        # 통합 처리 예: 특정 거리 이내의 영역만 표시
        depth_meters = depth_image.astype(np.float32) / 1000.0
        mask = (depth_meters > 0.3) & (depth_meters < 2.0)
        
        # 마스크 적용
        result = rgb_image.copy()
        result[~mask] = 0
        
        cv2.imshow('Filtered', result)
        cv2.waitKey(1)
```

---

## 8. 실습 과제

### 과제 1: 중앙 거리 표시

RGB 이미지 중앙에 십자선과 해당 위치의 Depth 거리를 표시하는 노드를 작성하세요.

### 과제 2: 근거리 경고

1m 이내의 물체가 감지되면 이미지에 빨간 테두리를 표시하세요.

### 과제 3: 바닥 제거

Depth 이미지에서 바닥 영역을 제거하고 장애물만 표시하세요.

---

## 9. 검증 체크리스트

- [ ] cv_bridge로 이미지 변환 성공
- [ ] RGB 이미지 처리 및 발행
- [ ] Depth 이미지에서 거리 계산
- [ ] rqt_image_view로 처리 결과 확인

---

*카메라 데이터는 시각 기반 자율주행의 핵심입니다!*
