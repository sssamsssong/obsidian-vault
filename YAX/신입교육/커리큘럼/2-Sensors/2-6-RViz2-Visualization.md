# RViz2 시각화

> YAX F1TENTH 교육자료 | Phase 2: Sensors
> 
> 3D 센서 데이터 시각화

---

## 1. 개요

### 학습 목표
1. RViz2의 인터페이스를 이해하고 사용할 수 있다
2. 다양한 센서 데이터를 시각화할 수 있다
3. 커스텀 설정을 저장하고 불러올 수 있다
4. 마커를 사용하여 정보를 표시할 수 있다

### 예상 소요 시간
- 전체: 1시간

### 전제 조건
- LiDAR/카메라 설정 완료
- ROS2 기본 개념 이해

---

## 2. RViz2 기본

### 2.1 실행

```bash
# 기본 실행
rviz2

# 설정 파일과 함께
rviz2 -d my_config.rviz
```

### 2.2 인터페이스

```
┌─────────────────────────────────────────────────────────────┐
│ [File] [Panels] [Help]                          [X]        │
├─────────────────────────────────────────────────────────────┤
│ Displays     │                                  │ Views    │
│ ───────────  │                                  │ ──────── │
│ □ Grid       │                                  │ Orbit    │
│ □ LaserScan  │        3D Viewport               │ TopDown  │
│ □ Image      │                                  │ FPS      │
│ □ TF         │                                  │          │
│              │                                  │          │
│ [Add]        │                                  │          │
├──────────────┴──────────────────────────────────┴──────────┤
│ [Status Bar]                                               │
└─────────────────────────────────────────────────────────────┘
```

### 2.3 주요 설정

| 설정 | 설명 |
|------|------|
| **Fixed Frame** | 기준 좌표계 (base_link, odom, map) |
| **Target Frame** | 카메라가 따라갈 프레임 |
| **Background Color** | 배경색 |

---

## 3. 디스플레이 추가

### 3.1 Grid

기준 좌표계에 그리드 표시.

```
Display Type: Grid
Cell Count: 10
Cell Size: 1.0m
Color: (128, 128, 128)
```

### 3.2 LaserScan

```
Display Type: LaserScan
Topic: /scan
Size (m): 0.05
Color Transformer: Intensity / FlatColor / AxisColor
```

**Color Transformer 옵션**:
- **FlatColor**: 단색
- **Intensity**: 반사 강도 기반
- **AxisColor**: X/Y/Z 축 기반

### 3.3 Image

```
Display Type: Image
Topic: /camera/color/image_raw
Transport Hint: raw / compressed
```

### 3.4 Camera

카메라 이미지를 3D 뷰에 오버레이.

```
Display Type: Camera
Topic: /camera/color/image_raw
Image Rendering: background / overlay
```

### 3.5 PointCloud2

3D LiDAR 데이터 시각화.

```
Display Type: PointCloud2
Topic: /points
Size (m): 0.05
Style: Points / Spheres / Boxes
```

### 3.6 TF

좌표 프레임 시각화.

```
Display Type: TF
Show Names: true
Show Axes: true
Show Arrows: false
```

---

## 4. F1TENTH 설정 예제

### 4.1 기본 설정

```yaml
# f1tenth.rviz
Visualization Manager:
  Global Options:
    Fixed Frame: base_link
    Frame Rate: 30
  
  Displays:
    - Name: Grid
      Class: rviz_default_plugins/Grid
      Cell Count: 20
      Cell Size: 0.5
      
    - Name: LaserScan
      Class: rviz_default_plugins/LaserScan
      Topic: /scan
      Size (m): 0.03
      Color Transformer: AxisColor
      
    - Name: TF
      Class: rviz_default_plugins/TF
      Show Names: true
```

### 4.2 설정 저장/불러오기

```bash
# 저장
# File > Save Config As > f1tenth.rviz

# 불러오기
rviz2 -d ~/f1tenth_ws/src/f1tenth_system/rviz/f1tenth.rviz
```

### 4.3 Launch 파일에서 RViz2 실행

```python
from launch_ros.actions import Node
import os

def generate_launch_description():
    rviz_config = os.path.join(
        get_package_share_directory('f1tenth_system'),
        'rviz',
        'f1tenth.rviz'
    )
    
    return LaunchDescription([
        Node(
            package='rviz2',
            executable='rviz2',
            name='rviz2',
            arguments=['-d', rviz_config]
        )
    ])
```

---

## 5. 마커 (Markers)

### 5.1 마커 타입

| 타입 | 상수 | 용도 |
|------|------|------|
| ARROW | 0 | 방향 표시 |
| CUBE | 1 | 박스 |
| SPHERE | 2 | 구 |
| CYLINDER | 3 | 원기둥 |
| LINE_STRIP | 4 | 연결선 |
| LINE_LIST | 5 | 선 목록 |
| POINTS | 8 | 점들 |
| TEXT_VIEW_FACING | 9 | 텍스트 |

### 5.2 마커 발행 노드

```python
#!/usr/bin/env python3
"""
Marker Publisher Example
"""

import rclpy
from rclpy.node import Node
from visualization_msgs.msg import Marker
from geometry_msgs.msg import Point


class MarkerPublisher(Node):
    def __init__(self):
        super().__init__('marker_publisher')
        
        self.pub = self.create_publisher(Marker, '/visualization_marker', 10)
        self.timer = self.create_timer(0.1, self.publish_marker)
        self.count = 0
    
    def publish_marker(self):
        marker = Marker()
        
        # 헤더
        marker.header.frame_id = 'base_link'
        marker.header.stamp = self.get_clock().now().to_msg()
        
        # ID와 타입
        marker.ns = 'obstacle'
        marker.id = 0
        marker.type = Marker.SPHERE
        marker.action = Marker.ADD
        
        # 위치
        marker.pose.position.x = 1.0
        marker.pose.position.y = 0.0
        marker.pose.position.z = 0.0
        marker.pose.orientation.w = 1.0
        
        # 크기
        marker.scale.x = 0.3
        marker.scale.y = 0.3
        marker.scale.z = 0.3
        
        # 색상 (RGBA)
        marker.color.r = 1.0
        marker.color.g = 0.0
        marker.color.b = 0.0
        marker.color.a = 1.0
        
        # 수명
        marker.lifetime.sec = 0  # 0 = 영구
        
        self.pub.publish(marker)


def main(args=None):
    rclpy.init(args=args)
    node = MarkerPublisher()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()
```

### 5.3 텍스트 마커

```python
def create_text_marker(self, text, x, y):
    marker = Marker()
    marker.header.frame_id = 'base_link'
    marker.header.stamp = self.get_clock().now().to_msg()
    marker.ns = 'text'
    marker.id = 1
    marker.type = Marker.TEXT_VIEW_FACING
    marker.action = Marker.ADD
    
    marker.pose.position.x = x
    marker.pose.position.y = y
    marker.pose.position.z = 0.5
    
    marker.scale.z = 0.2  # 텍스트 높이
    
    marker.color.r = 1.0
    marker.color.g = 1.0
    marker.color.b = 1.0
    marker.color.a = 1.0
    
    marker.text = text
    
    return marker
```

### 5.4 화살표 마커 (방향 표시)

```python
def create_arrow_marker(self, angle):
    marker = Marker()
    marker.header.frame_id = 'base_link'
    marker.header.stamp = self.get_clock().now().to_msg()
    marker.ns = 'direction'
    marker.id = 2
    marker.type = Marker.ARROW
    marker.action = Marker.ADD
    
    # 시작점과 끝점
    start = Point()
    start.x = 0.0
    start.y = 0.0
    start.z = 0.1
    
    end = Point()
    end.x = np.cos(angle)
    end.y = np.sin(angle)
    end.z = 0.1
    
    marker.points = [start, end]
    
    # 화살표 크기
    marker.scale.x = 0.05  # 축 직경
    marker.scale.y = 0.1   # 머리 직경
    marker.scale.z = 0.1   # 머리 길이
    
    # 녹색
    marker.color.r = 0.0
    marker.color.g = 1.0
    marker.color.b = 0.0
    marker.color.a = 1.0
    
    return marker
```

---

## 6. 뷰 설정

### 6.1 기본 뷰

| 뷰 | 설명 | 용도 |
|----|------|------|
| **Orbit** | 3D 회전 | 일반 |
| **TopDownOrtho** | 위에서 내려다봄 | 경로 계획 |
| **FPS** | 1인칭 시점 | 로봇 시점 |
| **ThirdPersonFollower** | 로봇 뒤따라감 | 주행 모니터링 |

### 6.2 뷰 저장

1. 원하는 시점으로 이동
2. Views 패널 > Save Current as New View
3. 이름 지정

---

## 7. 실습

### 실습 1: LiDAR 시각화

1. `rviz2` 실행
2. Fixed Frame을 `laser`로 설정
3. LaserScan 추가 → `/scan` 토픽
4. 색상을 AxisColor로 변경
5. 설정 저장

### 실습 2: 장애물 마커

1. 최소 거리 장애물 위치에 빨간 구 마커 표시
2. 거리 텍스트 마커 추가

---

## 8. 검증 체크리스트

- [ ] RViz2 기본 인터페이스 이해
- [ ] LaserScan 시각화
- [ ] Image 디스플레이 추가
- [ ] TF 프레임 시각화
- [ ] 마커 발행 및 표시
- [ ] 설정 저장/불러오기

---

## 9. Phase 2 완료

축하합니다! Phase 2: 센서 통합을 완료했습니다.


---

*RViz2는 로봇 개발자의 필수 도구입니다!*
