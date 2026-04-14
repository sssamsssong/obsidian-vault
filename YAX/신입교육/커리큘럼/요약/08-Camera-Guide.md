# RealSense 카메라 가이드

> YAX F1TENTH 교육자료 | Domain 04: 비전 센서

---

## 1. 개요

### 학습 목표
1. RGB-D 카메라의 원리를 이해한다
2. RealSense 카메라를 ROS2에 연결할 수 있다
3. OpenCV로 이미지를 처리할 수 있다
4. Depth 데이터로 거리를 측정할 수 있다

### 예상 소요 시간: 3-4시간

### 필요 장비
- Intel RealSense D435/D455
- USB 3.0 케이블
- Jetson Orin Nano (ROS2 설치 완료)

---

## 2. Intel RealSense 소개

### 2.1 RGB-D 카메라란?

RGB (컬러) + Depth (깊이) 정보를 동시에 제공하는 카메라입니다.

### 2.2 RealSense D435 사양

| 항목 | 값 |
|------|-----|
| RGB 해상도 | 1920x1080 @ 30fps |
| Depth 해상도 | 1280x720 @ 30fps |
| Depth 범위 | 0.2m ~ 10m |
| FOV (Depth) | 87° x 58° |
| 인터페이스 | USB 3.0 |

---

## 3. RealSense SDK 설치 (Jetson)

```bash
# 의존성 설치
sudo apt install -y libssl-dev libusb-1.0-0-dev libudev-dev pkg-config libgtk-3-dev

# RealSense SDK 설치
sudo apt install -y ros-humble-librealsense2

# 확인
realsense-viewer
```

---

## 4. ROS2 Wrapper 설치

```bash
sudo apt install ros-humble-realsense2-camera -y
```

---

## 5. 카메라 실행

### 5.1 기본 실행

```bash
ros2 launch realsense2_camera rs_launch.py
```

### 5.2 파라미터와 함께 실행

```bash
ros2 launch realsense2_camera rs_launch.py \
    depth_module.profile:=640x480x30 \
    rgb_camera.profile:=640x480x30 \
    enable_color:=true \
    enable_depth:=true
```

### 5.3 토픽 확인

```bash
ros2 topic list | grep camera
# /camera/color/image_raw
# /camera/depth/image_rect_raw
# /camera/color/camera_info
```

---

## 6. OpenCV 연동 (cv_bridge)

### 6.1 설치

```bash
sudo apt install ros-humble-cv-bridge -y
pip3 install opencv-python
```

### 6.2 이미지 처리 노드

```python
import rclpy
from rclpy.node import Node
from sensor_msgs.msg import Image
from cv_bridge import CvBridge
import cv2

class CameraProcessor(Node):
    def __init__(self):
        super().__init__('camera_processor')
        self.bridge = CvBridge()
        
        self.subscription = self.create_subscription(
            Image,
            '/camera/color/image_raw',
            self.image_callback,
            10
        )

    def image_callback(self, msg):
        # ROS Image → OpenCV
        cv_image = self.bridge.imgmsg_to_cv2(msg, 'bgr8')
        
        # OpenCV 처리 (예: 그레이스케일)
        gray = cv2.cvtColor(cv_image, cv2.COLOR_BGR2GRAY)
        
        # 화면에 표시
        cv2.imshow('Camera', cv_image)
        cv2.waitKey(1)

def main(args=None):
    rclpy.init(args=args)
    node = CameraProcessor()
    rclpy.spin(node)
    cv2.destroyAllWindows()
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

---

## 7. Depth 이미지 처리

### 7.1 거리 측정

```python
from sensor_msgs.msg import Image
import numpy as np

def depth_callback(self, msg):
    # Depth 이미지 변환 (16비트, mm 단위)
    depth_image = self.bridge.imgmsg_to_cv2(msg, 'passthrough')
    
    # 중앙 픽셀 거리
    h, w = depth_image.shape
    center_x, center_y = w // 2, h // 2
    
    distance_mm = depth_image[center_y, center_x]
    distance_m = distance_mm / 1000.0
    
    self.get_logger().info(f'중앙 거리: {distance_m:.2f}m')
```

### 7.2 특정 영역 평균 거리

```python
def get_region_distance(self, depth_image, x, y, size=10):
    """(x, y) 주변 size x size 영역의 평균 거리"""
    region = depth_image[y-size:y+size, x-size:x+size]
    
    # 0이 아닌 값만 사용 (유효한 측정)
    valid = region[region > 0]
    
    if len(valid) > 0:
        return np.mean(valid) / 1000.0  # mm → m
    return None
```

---

## 8. 센서 융합 (LiDAR + Camera)

### 8.1 message_filters로 동기화

```python
from message_filters import ApproximateTimeSynchronizer, Subscriber
from sensor_msgs.msg import Image, LaserScan

class SensorFusion(Node):
    def __init__(self):
        super().__init__('sensor_fusion')
        
        # Subscriber 생성
        self.image_sub = Subscriber(self, Image, '/camera/color/image_raw')
        self.scan_sub = Subscriber(self, LaserScan, '/scan')
        
        # 동기화 (0.1초 허용)
        self.sync = ApproximateTimeSynchronizer(
            [self.image_sub, self.scan_sub],
            queue_size=10,
            slop=0.1
        )
        self.sync.registerCallback(self.sync_callback)

    def sync_callback(self, image_msg, scan_msg):
        self.get_logger().info('Camera + LiDAR 동기화 수신!')
        # 동기화된 데이터 처리
```

---

## 9. Jetson 성능 최적화

```bash
# 해상도/프레임 낮추기
ros2 launch realsense2_camera rs_launch.py \
    depth_module.profile:=424x240x15 \
    rgb_camera.profile:=424x240x15
```

---

## 10. 검증 체크리스트

- [ ] realsense-viewer 실행 확인
- [ ] `/camera/color/image_raw` 토픽 수신
- [ ] `/camera/depth/image_rect_raw` 토픽 수신
- [ ] OpenCV로 이미지 표시
- [ ] 거리 측정 동작

---

## 11. 다음 단계

➡️ **Domain 05: VESC 모터 제어**

---

*YAX F1TENTH 교육자료 | 작성일: 2026-02-10*
