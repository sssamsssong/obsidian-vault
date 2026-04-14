# Publisher/Subscriber 프로그래밍

> YAX F1TENTH 교육자료 | Phase 1: ROS2 Basics
> 
> 토픽 기반 통신 구현

---

## 1. 개요

### 학습 목표
1. Publisher 노드를 작성할 수 있다
2. Subscriber 노드를 작성할 수 있다
3. 커스텀 메시지를 사용할 수 있다
4. QoS를 설정할 수 있다

### 예상 소요 시간
- 전체: 2시간
- Publisher 작성: 40분
- Subscriber 작성: 40분
- 통합 실습: 40분

### 전제 조건
- 패키지 생성 방법 이해 (03-Package-Creation.md)

---

## 2. Publisher/Subscriber 개념

### 2.1 통신 패턴

```
┌────────────┐     /topic     ┌────────────┐
│ Publisher  │ ─────────────► │ Subscriber │
│   Node     │                │    Node    │
└────────────┘                └────────────┘
      │                             ▲
      │         Message             │
      └─────────────────────────────┘
```

### 2.2 특징
- **비동기 통신**: Publisher는 Subscriber의 상태와 무관하게 발행
- **1:N 통신**: 하나의 토픽에 여러 Subscriber 가능
- **N:1 통신**: 여러 Publisher가 같은 토픽에 발행 가능
- **느슨한 결합**: Publisher와 Subscriber가 서로를 알 필요 없음

### 2.3 F1TENTH에서의 활용

| 토픽 | Publisher | Subscriber | 메시지 |
|------|-----------|------------|--------|
| `/scan` | LiDAR 드라이버 | 자율주행 노드 | LaserScan |
| `/cmd_vel` | 자율주행 노드 | VESC 드라이버 | Twist |
| `/odom` | VESC 드라이버 | SLAM, 경로계획 | Odometry |

---

## 3. Publisher 노드 작성

### 3.1 기본 구조

```python
#!/usr/bin/env python3
"""
Simple Publisher Node
"""

import rclpy
from rclpy.node import Node
from std_msgs.msg import String


class SimplePublisher(Node):
    """Publishes String messages to a topic."""
    
    def __init__(self):
        super().__init__('simple_publisher')
        
        # Publisher 생성
        # create_publisher(메시지타입, 토픽이름, 큐크기)
        self.publisher_ = self.create_publisher(String, 'chatter', 10)
        
        # 타이머 생성 (0.5초마다 콜백 호출)
        timer_period = 0.5  # seconds
        self.timer = self.create_timer(timer_period, self.timer_callback)
        
        self.count = 0
        self.get_logger().info('Publisher node started')
    
    def timer_callback(self):
        """Timer callback - publishes a message."""
        msg = String()
        msg.data = f'Hello World: {self.count}'
        
        self.publisher_.publish(msg)
        self.get_logger().info(f'Publishing: "{msg.data}"')
        
        self.count += 1


def main(args=None):
    rclpy.init(args=args)
    
    node = SimplePublisher()
    
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

### 3.2 다양한 메시지 타입 발행

```python
from std_msgs.msg import Int32, Float64, Bool
from geometry_msgs.msg import Twist, Point
from sensor_msgs.msg import LaserScan

# Int32 발행
self.int_pub = self.create_publisher(Int32, 'int_topic', 10)
msg = Int32()
msg.data = 42
self.int_pub.publish(msg)

# Twist 발행 (속도 명령)
self.cmd_pub = self.create_publisher(Twist, 'cmd_vel', 10)
twist = Twist()
twist.linear.x = 1.0  # 전진 속도
twist.angular.z = 0.5  # 회전 속도
self.cmd_pub.publish(twist)

# Point 발행
self.point_pub = self.create_publisher(Point, 'point', 10)
point = Point()
point.x = 1.0
point.y = 2.0
point.z = 0.0
self.point_pub.publish(point)
```

---

## 4. Subscriber 노드 작성

### 4.1 기본 구조

```python
#!/usr/bin/env python3
"""
Simple Subscriber Node
"""

import rclpy
from rclpy.node import Node
from std_msgs.msg import String


class SimpleSubscriber(Node):
    """Subscribes to String messages from a topic."""
    
    def __init__(self):
        super().__init__('simple_subscriber')
        
        # Subscriber 생성
        # create_subscription(메시지타입, 토픽이름, 콜백함수, 큐크기)
        self.subscription = self.create_subscription(
            String,
            'chatter',
            self.listener_callback,
            10
        )
        self.subscription  # prevent unused variable warning
        
        self.get_logger().info('Subscriber node started')
    
    def listener_callback(self, msg):
        """Callback function when message is received."""
        self.get_logger().info(f'Received: "{msg.data}"')


def main(args=None):
    rclpy.init(args=args)
    
    node = SimpleSubscriber()
    
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

### 4.2 센서 데이터 구독

```python
from sensor_msgs.msg import LaserScan
import numpy as np

class LidarSubscriber(Node):
    def __init__(self):
        super().__init__('lidar_subscriber')
        
        self.subscription = self.create_subscription(
            LaserScan,
            '/scan',
            self.scan_callback,
            10
        )
    
    def scan_callback(self, msg):
        """Process LiDAR data."""
        # LaserScan 메시지 구조
        # msg.ranges: 거리 배열
        # msg.angle_min: 시작 각도
        # msg.angle_max: 끝 각도
        # msg.angle_increment: 각도 증분
        
        ranges = np.array(msg.ranges)
        
        # inf 값 처리
        ranges = np.where(np.isinf(ranges), msg.range_max, ranges)
        
        # 최소 거리 찾기
        min_distance = np.min(ranges)
        min_index = np.argmin(ranges)
        
        # 각도 계산
        angle = msg.angle_min + min_index * msg.angle_increment
        
        self.get_logger().info(
            f'Closest obstacle: {min_distance:.2f}m at {np.degrees(angle):.1f}°'
        )
```

---

## 5. Publisher + Subscriber 통합 노드

### 5.1 데이터 처리 노드

```python
#!/usr/bin/env python3
"""
Node that subscribes to LiDAR data and publishes velocity commands.
"""

import rclpy
from rclpy.node import Node
from sensor_msgs.msg import LaserScan
from geometry_msgs.msg import Twist
import numpy as np


class ReactiveController(Node):
    """Reactive obstacle avoidance controller."""
    
    def __init__(self):
        super().__init__('reactive_controller')
        
        # Subscriber
        self.scan_sub = self.create_subscription(
            LaserScan,
            '/scan',
            self.scan_callback,
            10
        )
        
        # Publisher
        self.cmd_pub = self.create_publisher(Twist, '/cmd_vel', 10)
        
        # Parameters
        self.safe_distance = 1.0  # meters
        self.max_speed = 2.0     # m/s
        
        self.get_logger().info('Reactive controller started')
    
    def scan_callback(self, msg):
        """Process scan and publish velocity."""
        ranges = np.array(msg.ranges)
        ranges = np.where(np.isinf(ranges), msg.range_max, ranges)
        
        # 전방 거리 (중앙 부분)
        num_ranges = len(ranges)
        center_idx = num_ranges // 2
        front_ranges = ranges[center_idx - 10:center_idx + 10]
        front_distance = np.min(front_ranges)
        
        # 속도 결정
        twist = Twist()
        
        if front_distance > self.safe_distance:
            # 안전: 전진
            twist.linear.x = self.max_speed
            twist.angular.z = 0.0
        else:
            # 장애물: 정지 또는 회전
            twist.linear.x = 0.0
            twist.angular.z = 1.0  # 좌회전
        
        self.cmd_pub.publish(twist)
        
        self.get_logger().info(
            f'Front: {front_distance:.2f}m -> Speed: {twist.linear.x:.2f}'
        )


def main(args=None):
    rclpy.init(args=args)
    node = ReactiveController()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()
```

---

## 6. QoS (Quality of Service) 설정

### 6.1 QoS 개념

QoS는 통신의 품질을 설정합니다.

| 정책 | 옵션 | 설명 |
|------|------|------|
| **Reliability** | RELIABLE / BEST_EFFORT | 메시지 전달 보장 여부 |
| **Durability** | VOLATILE / TRANSIENT_LOCAL | 새 구독자에게 이전 메시지 전달 |
| **History** | KEEP_LAST / KEEP_ALL | 메시지 보관 정책 |
| **Depth** | 숫자 | 큐 크기 |

### 6.2 센서 데이터용 QoS

센서 데이터는 보통 **BEST_EFFORT**를 사용합니다:

```python
from rclpy.qos import QoSProfile, ReliabilityPolicy, HistoryPolicy

# 센서 데이터용 QoS
sensor_qos = QoSProfile(
    reliability=ReliabilityPolicy.BEST_EFFORT,
    history=HistoryPolicy.KEEP_LAST,
    depth=10
)

# Subscriber에 적용
self.scan_sub = self.create_subscription(
    LaserScan,
    '/scan',
    self.scan_callback,
    sensor_qos
)
```

### 6.3 미리 정의된 QoS 프로파일

```python
from rclpy.qos import qos_profile_sensor_data, qos_profile_system_default

# 센서 데이터용 (BEST_EFFORT)
self.sub = self.create_subscription(
    LaserScan, '/scan', self.callback,
    qos_profile_sensor_data
)

# 기본 설정 (RELIABLE)
self.sub = self.create_subscription(
    String, '/topic', self.callback,
    qos_profile_system_default
)
```

---

## 7. 파라미터 사용

### 7.1 파라미터 선언 및 사용

```python
class ParameterizedNode(Node):
    def __init__(self):
        super().__init__('parameterized_node')
        
        # 파라미터 선언
        self.declare_parameter('speed', 1.0)
        self.declare_parameter('topic_name', 'cmd_vel')
        
        # 파라미터 값 가져오기
        self.speed = self.get_parameter('speed').value
        topic = self.get_parameter('topic_name').value
        
        self.publisher = self.create_publisher(Twist, topic, 10)
        
        self.get_logger().info(f'Speed: {self.speed}, Topic: {topic}')
```

### 7.2 실행 시 파라미터 전달

```bash
ros2 run my_pkg my_node --ros-args -p speed:=2.0 -p topic_name:=drive
```

---

## 8. 실습 과제

### 과제 1: 카운터 노드

**Publisher**: 1초마다 증가하는 숫자를 발행
**Subscriber**: 숫자를 받아서 제곱값 출력

### 과제 2: 속도 증폭기

**Subscriber**: `/cmd_vel` 구독
**Publisher**: 속도를 2배로 증폭하여 `/cmd_vel_amplified`에 발행

```python
# 힌트
def cmd_callback(self, msg):
    amplified = Twist()
    amplified.linear.x = msg.linear.x * 2.0
    amplified.angular.z = msg.angular.z * 2.0
    self.amp_pub.publish(amplified)
```

### 과제 3: LiDAR 필터

**Subscriber**: `/scan` 구독
**Publisher**: 전방 90도 범위만 필터링하여 `/scan_filtered`에 발행

---

## 9. 패키지 완성

### 9.1 setup.py entry_points

```python
entry_points={
    'console_scripts': [
        'publisher = my_pkg.publisher_node:main',
        'subscriber = my_pkg.subscriber_node:main',
        'reactive = my_pkg.reactive_controller:main',
    ],
},
```

### 9.2 빌드 및 실행

```bash
cd ~/f1tenth_ws
colcon build --packages-select my_pkg
source install/setup.bash

# 터미널 1
ros2 run my_pkg publisher

# 터미널 2
ros2 run my_pkg subscriber
```

---

## 10. 검증 체크리스트

- [ ] Publisher 노드 작성 및 실행 성공
- [ ] Subscriber 노드 작성 및 실행 성공
- [ ] `ros2 topic echo`로 메시지 확인
- [ ] `rqt_graph`로 연결 확인
- [ ] QoS 설정 적용
- [ ] 파라미터 사용

---

*Publisher/Subscriber는 ROS2의 가장 기본적인 통신 패턴입니다. 이것을 마스터하면 대부분의 로봇 시스템을 구현할 수 있습니다!*
