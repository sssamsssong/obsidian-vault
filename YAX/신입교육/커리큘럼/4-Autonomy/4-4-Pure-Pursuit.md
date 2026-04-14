# Pure Pursuit 알고리즘

> YAX F1TENTH 교육자료 | Phase 4: Autonomy
> 
> 웨이포인트 기반 경로 추종

---

## 1. 개요

### 학습 목표
1. Pure Pursuit 알고리즘의 기하학적 원리를 이해할 수 있다
2. 웨이포인트를 생성하고 로드할 수 있다
3. Lookahead distance를 조절하여 성능을 최적화할 수 있다

### 예상 소요 시간
- 전체: 3시간

### 전제 조건
- Follow the Gap 완료
- Odometry 이해
- 기본 삼각함수

---

## 2. 알고리즘 개요

### 2.1 Pure Pursuit이란?

Pure Pursuit은 차량이 **경로 상의 목표점(Lookahead Point)** 을 향해 조향하는 경로 추종 알고리즘입니다.

```
                 목표점 (Lookahead Point)
                    ●
                   /
                  / Ld (Lookahead Distance)
                 /
                /  α (alpha)
               /
         ┌────●────┐
         │ F1TENTH │
         └─────────┘
              ↑
           현재 위치
```

### 2.2 FTG vs Pure Pursuit

| 항목 | Follow the Gap | Pure Pursuit |
|------|----------------|--------------|
| 입력 | LiDAR (실시간) | 웨이포인트 + Odometry |
| 목표 | 빈 공간 찾기 | 경로 추종 |
| 지도 | 불필요 | 필요 (웨이포인트) |
| 장점 | 동적 장애물 대응 | 최적 경로 유지 |
| 단점 | 비효율적 경로 | 장애물 대응 별도 필요 |

### 2.3 언제 사용하는가?

- 트랙 레이아웃을 알고 있을 때
- 최적 레이싱 라인을 따라가고 싶을 때
- 일관된 주행 패턴이 필요할 때
- 타임 어택 / 대회 상황

---

## 3. 기하학적 원리

### 3.1 핵심 공식

Pure Pursuit의 조향각 공식:

```
              2 * L * sin(α)
δ = arctan( ───────────────── )
                   Ld

δ  : 조향각 (steering angle)
L  : 휠베이스 (wheelbase) = 0.33m (F1TENTH)
α  : 현재 방향과 목표점 사이 각도
Ld : Lookahead distance
```

### 3.2 공식 유도

```
                    목표점
                      ●
                     /|
                    / |
                Ld /  | Ld * sin(α)
                  /   |
                 / α  |
      ──────────●─────┴──────────
               차량     
```

1. 차량에서 목표점까지 거리: `Ld`
2. 목표점의 측면 오프셋: `Ld * sin(α)`
3. Ackermann 조향 기하학 적용
4. 곡률 `κ = 2 * sin(α) / Ld`
5. 조향각 `δ = arctan(L * κ)`

### 3.3 Lookahead Distance 영향

| Ld | 효과 |
|----|------|
| 작음 (0.5-1.0m) | 정확한 추종, 진동 가능, 급격한 조향 |
| 큼 (2.0-3.0m) | 부드러운 주행, 코너 커팅, 지연 |

```
Ld 작음:                    Ld 큼:
     경로                        경로
  ●────●────●                 ●────●────●
   ↖  ↑  ↗                       ↗
    \ | /                        /
     \|/                        /
      ●  (급격한 조향)           ● (부드러운 조향)
```

---

## 4. 웨이포인트 시스템

### 4.1 웨이포인트란?

경로를 나타내는 좌표점들의 배열입니다.

```
CSV 형식:
x, y, (speed)
0.0, 0.0, 2.0
0.5, 0.1, 2.0
1.0, 0.3, 2.5
1.5, 0.6, 3.0
...
```

### 4.2 웨이포인트 기록 노드

수동 주행 중 경로를 기록합니다.

```python
#!/usr/bin/env python3
"""
Waypoint Recorder

수동 주행 중 경로를 기록하여 CSV로 저장합니다.
"""

import rclpy
from rclpy.node import Node
from nav_msgs.msg import Odometry
from geometry_msgs.msg import Twist
import numpy as np
import csv
from tf_transformations import euler_from_quaternion


class WaypointRecorder(Node):
    def __init__(self):
        super().__init__('waypoint_recorder')
        
        # 파라미터
        self.declare_parameter('output_file', 'waypoints.csv')
        self.declare_parameter('min_distance', 0.3)  # 최소 기록 간격 (m)
        self.declare_parameter('include_speed', True)
        
        self.output_file = self.get_parameter('output_file').value
        self.min_distance = self.get_parameter('min_distance').value
        self.include_speed = self.get_parameter('include_speed').value
        
        # 상태
        self.waypoints = []
        self.last_pos = None
        self.current_speed = 0.0
        
        # Subscribers
        self.odom_sub = self.create_subscription(
            Odometry,
            '/odom',
            self.odom_callback,
            10
        )
        
        self.cmd_sub = self.create_subscription(
            Twist,
            '/cmd_vel',
            self.cmd_callback,
            10
        )
        
        self.get_logger().info(f'Waypoint Recorder started')
        self.get_logger().info(f'  Output: {self.output_file}')
        self.get_logger().info(f'  Min distance: {self.min_distance}m')
        self.get_logger().info('Press Ctrl+C to save and exit')
    
    def cmd_callback(self, msg: Twist):
        """현재 속도 저장."""
        self.current_speed = msg.linear.x
    
    def odom_callback(self, msg: Odometry):
        """위치 기록."""
        pos = np.array([
            msg.pose.pose.position.x,
            msg.pose.pose.position.y
        ])
        
        # 첫 번째 점이거나 최소 거리 이상 이동했을 때
        if self.last_pos is None:
            self.record_waypoint(pos)
        elif np.linalg.norm(pos - self.last_pos) >= self.min_distance:
            self.record_waypoint(pos)
    
    def record_waypoint(self, pos: np.ndarray):
        """웨이포인트 추가."""
        if self.include_speed:
            wp = [pos[0], pos[1], max(0.5, self.current_speed)]
        else:
            wp = [pos[0], pos[1]]
        
        self.waypoints.append(wp)
        self.last_pos = pos.copy()
        
        self.get_logger().info(
            f'Waypoint {len(self.waypoints)}: '
            f'({pos[0]:.2f}, {pos[1]:.2f})'
        )
    
    def save_waypoints(self):
        """CSV로 저장."""
        if len(self.waypoints) == 0:
            self.get_logger().warn('No waypoints to save!')
            return
        
        with open(self.output_file, 'w', newline='') as f:
            writer = csv.writer(f)
            
            # 헤더
            if self.include_speed:
                writer.writerow(['x', 'y', 'speed'])
            else:
                writer.writerow(['x', 'y'])
            
            # 데이터
            for wp in self.waypoints:
                writer.writerow(wp)
        
        self.get_logger().info(
            f'Saved {len(self.waypoints)} waypoints to {self.output_file}'
        )


def main(args=None):
    rclpy.init(args=args)
    node = WaypointRecorder()
    
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        node.save_waypoints()
    finally:
        node.destroy_node()
        rclpy.shutdown()


if __name__ == '__main__':
    main()
```

### 4.3 웨이포인트 시각화

RViz에서 웨이포인트를 시각화합니다.

```python
from visualization_msgs.msg import Marker, MarkerArray
from geometry_msgs.msg import Point

def publish_waypoints_marker(self):
    """웨이포인트를 RViz 마커로 시각화."""
    marker_array = MarkerArray()
    
    # 경로 라인
    line_marker = Marker()
    line_marker.header.frame_id = 'map'
    line_marker.header.stamp = self.get_clock().now().to_msg()
    line_marker.ns = 'waypoints_line'
    line_marker.id = 0
    line_marker.type = Marker.LINE_STRIP
    line_marker.action = Marker.ADD
    line_marker.scale.x = 0.05  # 라인 두께
    line_marker.color.r = 0.0
    line_marker.color.g = 1.0
    line_marker.color.b = 0.0
    line_marker.color.a = 0.8
    
    for wp in self.waypoints:
        p = Point()
        p.x = wp[0]
        p.y = wp[1]
        p.z = 0.0
        line_marker.points.append(p)
    
    marker_array.markers.append(line_marker)
    
    # 현재 목표점 표시
    if hasattr(self, 'current_target'):
        target_marker = Marker()
        target_marker.header.frame_id = 'map'
        target_marker.header.stamp = self.get_clock().now().to_msg()
        target_marker.ns = 'target_point'
        target_marker.id = 1
        target_marker.type = Marker.SPHERE
        target_marker.action = Marker.ADD
        target_marker.pose.position.x = self.current_target[0]
        target_marker.pose.position.y = self.current_target[1]
        target_marker.scale.x = 0.2
        target_marker.scale.y = 0.2
        target_marker.scale.z = 0.2
        target_marker.color.r = 1.0
        target_marker.color.g = 0.0
        target_marker.color.b = 0.0
        target_marker.color.a = 1.0
        
        marker_array.markers.append(target_marker)
    
    self.marker_pub.publish(marker_array)
```

---

## 5. Pure Pursuit 노드

### 5.1 전체 구현

```python
#!/usr/bin/env python3
"""
Pure Pursuit Node

웨이포인트 기반 경로 추종 알고리즘
"""

import rclpy
from rclpy.node import Node
from nav_msgs.msg import Odometry
from ackermann_msgs.msg import AckermannDriveStamped
from visualization_msgs.msg import Marker, MarkerArray
from geometry_msgs.msg import Point
import numpy as np
import csv
from tf_transformations import euler_from_quaternion


class PurePursuit(Node):
    def __init__(self):
        super().__init__('pure_pursuit')
        
        # === 파라미터 ===
        self.declare_parameter('waypoint_file', 'waypoints.csv')
        self.declare_parameter('lookahead_distance', 1.5)
        self.declare_parameter('lookahead_gain', 0.5)  # 속도 비례 Ld
        self.declare_parameter('min_lookahead', 0.5)
        self.declare_parameter('max_lookahead', 3.0)
        self.declare_parameter('wheelbase', 0.33)
        self.declare_parameter('default_speed', 2.0)
        self.declare_parameter('max_steering', 0.4)
        self.declare_parameter('use_waypoint_speed', True)
        
        # 파라미터 로드
        self.waypoint_file = self.get_parameter('waypoint_file').value
        self.base_lookahead = self.get_parameter('lookahead_distance').value
        self.lookahead_gain = self.get_parameter('lookahead_gain').value
        self.min_lookahead = self.get_parameter('min_lookahead').value
        self.max_lookahead = self.get_parameter('max_lookahead').value
        self.wheelbase = self.get_parameter('wheelbase').value
        self.default_speed = self.get_parameter('default_speed').value
        self.max_steering = self.get_parameter('max_steering').value
        self.use_waypoint_speed = self.get_parameter('use_waypoint_speed').value
        
        # 상태 변수
        self.waypoints = None
        self.current_idx = 0
        self.current_speed = 0.0
        self.current_target = None
        
        # 웨이포인트 로드
        self.load_waypoints()
        
        # === Subscribers ===
        self.odom_sub = self.create_subscription(
            Odometry,
            '/odom',
            self.odom_callback,
            10
        )
        
        # === Publishers ===
        self.drive_pub = self.create_publisher(
            AckermannDriveStamped,
            '/drive',
            10
        )
        
        self.marker_pub = self.create_publisher(
            MarkerArray,
            '/waypoints_viz',
            10
        )
        
        # 시각화 타이머
        self.viz_timer = self.create_timer(0.1, self.publish_markers)
        
        self.get_logger().info('Pure Pursuit node started')
        self.get_logger().info(f'  Loaded {len(self.waypoints)} waypoints')
        self.get_logger().info(f'  Base lookahead: {self.base_lookahead}m')
    
    def load_waypoints(self):
        """CSV에서 웨이포인트 로드."""
        waypoints = []
        
        try:
            with open(self.waypoint_file, 'r') as f:
                reader = csv.reader(f)
                header = next(reader)  # 헤더 스킵
                
                for row in reader:
                    if len(row) >= 2:
                        x = float(row[0])
                        y = float(row[1])
                        speed = float(row[2]) if len(row) >= 3 else self.default_speed
                        waypoints.append([x, y, speed])
            
            self.waypoints = np.array(waypoints)
            
        except FileNotFoundError:
            self.get_logger().error(f'Waypoint file not found: {self.waypoint_file}')
            self.waypoints = np.array([[0, 0, 1.0]])
        except Exception as e:
            self.get_logger().error(f'Error loading waypoints: {e}')
            self.waypoints = np.array([[0, 0, 1.0]])
    
    def get_yaw_from_quaternion(self, orientation) -> float:
        """Quaternion에서 Yaw 추출."""
        q = [
            orientation.x,
            orientation.y,
            orientation.z,
            orientation.w
        ]
        _, _, yaw = euler_from_quaternion(q)
        return yaw
    
    def calculate_lookahead(self) -> float:
        """
        속도에 비례하는 동적 Lookahead 계산.
        
        Ld = base + gain * speed
        """
        ld = self.base_lookahead + self.lookahead_gain * abs(self.current_speed)
        return np.clip(ld, self.min_lookahead, self.max_lookahead)
    
    def find_lookahead_point(self, pos: np.ndarray, 
                             yaw: float) -> tuple:
        """
        Lookahead point 탐색.
        
        전략:
        1. 가장 가까운 웨이포인트 찾기
        2. 그 이후 웨이포인트 중 Ld 거리에 있는 것 선택
        
        Returns:
            (target_point, target_speed, target_idx)
        """
        ld = self.calculate_lookahead()
        
        # 모든 웨이포인트까지의 거리
        wp_positions = self.waypoints[:, :2]
        distances = np.linalg.norm(wp_positions - pos, axis=1)
        
        # 가장 가까운 점 찾기
        closest_idx = np.argmin(distances)
        
        # closest_idx 이후부터 Ld 거리 이상인 첫 번째 점 찾기
        n_waypoints = len(self.waypoints)
        
        for i in range(n_waypoints):
            idx = (closest_idx + i) % n_waypoints
            dist = distances[idx]
            
            if dist >= ld:
                # 차량 전방에 있는지 확인
                wp = wp_positions[idx]
                wp_angle = np.arctan2(wp[1] - pos[1], wp[0] - pos[0])
                angle_diff = self.normalize_angle(wp_angle - yaw)
                
                # 전방 ±90도 이내
                if abs(angle_diff) < np.pi / 2:
                    target = self.waypoints[idx]
                    return target[:2], target[2], idx
        
        # 못 찾으면 가장 먼 점
        farthest_idx = np.argmax(distances)
        target = self.waypoints[farthest_idx]
        return target[:2], target[2], farthest_idx
    
    def normalize_angle(self, angle: float) -> float:
        """각도를 -pi ~ pi 범위로 정규화."""
        while angle > np.pi:
            angle -= 2 * np.pi
        while angle < -np.pi:
            angle += 2 * np.pi
        return angle
    
    def calculate_steering(self, pos: np.ndarray, 
                          yaw: float,
                          target: np.ndarray) -> float:
        """
        Pure Pursuit 조향각 계산.
        
        δ = arctan(2 * L * sin(α) / Ld)
        """
        # 목표점까지의 거리
        dx = target[0] - pos[0]
        dy = target[1] - pos[1]
        ld = np.sqrt(dx**2 + dy**2)
        
        if ld < 0.01:  # 너무 가까우면
            return 0.0
        
        # 목표점 방향 각도
        target_angle = np.arctan2(dy, dx)
        
        # 차량 기준 상대 각도 (alpha)
        alpha = self.normalize_angle(target_angle - yaw)
        
        # Pure Pursuit 공식
        steering = np.arctan2(2 * self.wheelbase * np.sin(alpha), ld)
        
        # 조향 제한
        steering = np.clip(steering, -self.max_steering, self.max_steering)
        
        return float(steering)
    
    def odom_callback(self, msg: Odometry):
        """메인 콜백: Pure Pursuit 실행."""
        
        if self.waypoints is None or len(self.waypoints) == 0:
            return
        
        # 현재 위치와 방향
        pos = np.array([
            msg.pose.pose.position.x,
            msg.pose.pose.position.y
        ])
        yaw = self.get_yaw_from_quaternion(msg.pose.pose.orientation)
        self.current_speed = msg.twist.twist.linear.x
        
        # 목표점 찾기
        target, target_speed, target_idx = self.find_lookahead_point(pos, yaw)
        self.current_target = target
        self.current_idx = target_idx
        
        # 조향각 계산
        steering = self.calculate_steering(pos, yaw, target)
        
        # 속도 결정
        if self.use_waypoint_speed:
            speed = target_speed
        else:
            speed = self.default_speed
        
        # 조향각에 따른 감속
        speed *= (1.0 - abs(steering) / self.max_steering * 0.3)
        
        # 명령 발행
        drive_msg = AckermannDriveStamped()
        drive_msg.header.stamp = self.get_clock().now().to_msg()
        drive_msg.header.frame_id = 'base_link'
        drive_msg.drive.speed = float(speed)
        drive_msg.drive.steering_angle = steering
        
        self.drive_pub.publish(drive_msg)
        
        # 디버그
        self.get_logger().debug(
            f'Target: {target_idx}, Steer: {steering:.2f}, Speed: {speed:.2f}'
        )
    
    def publish_markers(self):
        """RViz 시각화."""
        if self.waypoints is None:
            return
        
        marker_array = MarkerArray()
        
        # 경로 라인
        line_marker = Marker()
        line_marker.header.frame_id = 'map'
        line_marker.header.stamp = self.get_clock().now().to_msg()
        line_marker.ns = 'path'
        line_marker.id = 0
        line_marker.type = Marker.LINE_STRIP
        line_marker.action = Marker.ADD
        line_marker.scale.x = 0.05
        line_marker.color.g = 1.0
        line_marker.color.a = 0.8
        
        for wp in self.waypoints:
            p = Point()
            p.x = wp[0]
            p.y = wp[1]
            line_marker.points.append(p)
        
        # 경로 닫기 (루프)
        p = Point()
        p.x = self.waypoints[0, 0]
        p.y = self.waypoints[0, 1]
        line_marker.points.append(p)
        
        marker_array.markers.append(line_marker)
        
        # 목표점
        if self.current_target is not None:
            target_marker = Marker()
            target_marker.header.frame_id = 'map'
            target_marker.header.stamp = self.get_clock().now().to_msg()
            target_marker.ns = 'target'
            target_marker.id = 1
            target_marker.type = Marker.SPHERE
            target_marker.action = Marker.ADD
            target_marker.pose.position.x = self.current_target[0]
            target_marker.pose.position.y = self.current_target[1]
            target_marker.scale.x = 0.3
            target_marker.scale.y = 0.3
            target_marker.scale.z = 0.3
            target_marker.color.r = 1.0
            target_marker.color.a = 1.0
            
            marker_array.markers.append(target_marker)
        
        self.marker_pub.publish(marker_array)


def main(args=None):
    rclpy.init(args=args)
    node = PurePursuit()
    
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
# launch/pure_pursuit_launch.py
from launch import LaunchDescription
from launch_ros.actions import Node
from ament_index_python.packages import get_package_share_directory
import os

def generate_launch_description():
    pkg_dir = get_package_share_directory('f1tenth_autonomy')
    waypoint_file = os.path.join(pkg_dir, 'config', 'waypoints.csv')
    
    return LaunchDescription([
        Node(
            package='f1tenth_autonomy',
            executable='pure_pursuit',
            name='pure_pursuit',
            parameters=[{
                'waypoint_file': waypoint_file,
                'lookahead_distance': 1.5,
                'lookahead_gain': 0.5,
                'min_lookahead': 0.5,
                'max_lookahead': 3.0,
                'wheelbase': 0.33,
                'default_speed': 2.0,
                'max_steering': 0.4,
                'use_waypoint_speed': True,
            }],
            output='screen'
        )
    ])
```

### 5.3 파라미터 설정

```yaml
# config/pure_pursuit_params.yaml
pure_pursuit:
  ros__parameters:
    # 웨이포인트
    waypoint_file: "waypoints.csv"
    use_waypoint_speed: true
    
    # Lookahead
    lookahead_distance: 1.5    # 기본 Ld
    lookahead_gain: 0.5        # 속도 비례 계수
    min_lookahead: 0.5         # 최소 Ld
    max_lookahead: 3.0         # 최대 Ld
    
    # 차량 파라미터
    wheelbase: 0.33            # F1TENTH 휠베이스
    
    # 속도/조향
    default_speed: 2.0
    max_steering: 0.4
```

---

## 6. 동적 Lookahead

### 6.1 속도 비례 Lookahead

속도가 높을수록 먼 점을 바라봅니다.

```python
def calculate_lookahead(self, current_speed: float) -> float:
    """
    Ld = base + gain * |v|
    
    예:
    - 정지: Ld = 1.5m
    - 2 m/s: Ld = 1.5 + 0.5 * 2 = 2.5m
    - 4 m/s: Ld = 1.5 + 0.5 * 4 = 3.5m (max로 제한)
    """
    ld = self.base_lookahead + self.lookahead_gain * abs(current_speed)
    return np.clip(ld, self.min_lookahead, self.max_lookahead)
```

### 6.2 곡률 기반 Lookahead

코너에서는 가깝게, 직선에서는 멀리:

```python
def calculate_adaptive_lookahead(self, pos: np.ndarray, 
                                  current_idx: int) -> float:
    """
    경로 곡률에 따른 적응형 Lookahead.
    """
    # 현재 근처 웨이포인트들로 곡률 추정
    n = len(self.waypoints)
    
    # 3개 점으로 곡률 계산
    p1 = self.waypoints[(current_idx - 2) % n, :2]
    p2 = self.waypoints[current_idx, :2]
    p3 = self.waypoints[(current_idx + 2) % n, :2]
    
    curvature = self.calculate_curvature(p1, p2, p3)
    
    # 곡률이 크면(급커브) Ld 감소
    # 곡률이 작으면(직선) Ld 증가
    curvature_factor = 1.0 / (1.0 + 10 * curvature)
    
    ld = self.min_lookahead + (self.max_lookahead - self.min_lookahead) * curvature_factor
    return ld

def calculate_curvature(self, p1, p2, p3):
    """세 점을 지나는 원의 곡률."""
    a = np.linalg.norm(p2 - p1)
    b = np.linalg.norm(p3 - p2)
    c = np.linalg.norm(p3 - p1)
    
    # 삼각형 넓이 (Heron's formula)
    s = (a + b + c) / 2
    area_sq = s * (s - a) * (s - b) * (s - c)
    
    if area_sq <= 0:
        return 0.0
    
    area = np.sqrt(area_sq)
    
    # 곡률 = 4 * 넓이 / (a * b * c)
    if a * b * c == 0:
        return 0.0
    
    return 4 * area / (a * b * c)
```

---

## 7. 파라미터 튜닝

### 7.1 주요 파라미터 영향

| 파라미터 | 증가 시 | 감소 시 |
|----------|---------|---------|
| `lookahead_distance` | 부드러움, 코너 커팅 | 정확함, 진동 가능 |
| `lookahead_gain` | 고속에서 안정 | 고속에서 급조향 |
| `default_speed` | 빠름, 추종 오차 증가 | 느림, 정확 |

### 7.2 권장 시작값

**입문자용:**
```yaml
lookahead_distance: 2.0
lookahead_gain: 0.3
min_lookahead: 1.0
max_lookahead: 3.0
default_speed: 1.5
```

**중급자용:**
```yaml
lookahead_distance: 1.5
lookahead_gain: 0.5
min_lookahead: 0.5
max_lookahead: 3.0
default_speed: 2.5
```

**고급자용:**
```yaml
lookahead_distance: 1.0
lookahead_gain: 0.4
min_lookahead: 0.3
max_lookahead: 2.5
default_speed: 4.0
```

### 7.3 튜닝 프로세스

1. **안정 확보**: 큰 Ld로 시작 (2.0m+)
2. **Ld 감소**: 트랙 잘 따라가면 0.2m씩 감소
3. **속도 증가**: 안정되면 speed 증가
4. **gain 조정**: 고속에서 불안정하면 gain 증가

---

## 8. FTG + Pure Pursuit 조합

### 8.1 하이브리드 접근

- **Pure Pursuit**: 기본 경로 추종
- **FTG**: 장애물 감지 시 회피

```python
class HybridController(Node):
    def __init__(self):
        super().__init__('hybrid_controller')
        
        self.pure_pursuit = PurePursuit()
        self.ftg = FollowTheGap()
        
        self.obstacle_threshold = 1.5  # 장애물 임계 거리
        
    def control_callback(self, odom_msg, scan_msg):
        # 전방 장애물 체크
        front_ranges = scan_msg.ranges[len(scan_msg.ranges)//3:2*len(scan_msg.ranges)//3]
        min_front = np.min(front_ranges)
        
        if min_front < self.obstacle_threshold:
            # 장애물 가까움 → FTG
            steering, speed = self.ftg.compute(scan_msg)
            mode = "FTG"
        else:
            # 장애물 없음 → Pure Pursuit
            steering, speed = self.pure_pursuit.compute(odom_msg)
            mode = "PP"
        
        self.get_logger().debug(f'Mode: {mode}, Min front: {min_front:.2f}')
        return steering, speed
```

---

## 9. 실습

### 실습 1: 웨이포인트 기록

1. 텔레오퍼레이션으로 트랙 한 바퀴 주행
2. 웨이포인트 기록
3. CSV 파일 확인

```bash
# 터미널 1: 시뮬레이터
ros2 launch morai_sim track.launch.py

# 터미널 2: 텔레오퍼레이션
ros2 run teleop_twist_keyboard teleop_twist_keyboard

# 터미널 3: 웨이포인트 기록
ros2 run f1tenth_autonomy waypoint_recorder --ros-args \
  -p output_file:=my_track.csv \
  -p min_distance:=0.3
```

### 실습 2: Pure Pursuit 실행

```bash
ros2 launch f1tenth_autonomy pure_pursuit_launch.py \
  waypoint_file:=my_track.csv
```

### 실습 3: Lookahead 실험

| 실험 | Ld (base) | gain | 결과 |
|------|-----------|------|------|
| 1 | 2.0 | 0.0 | |
| 2 | 1.5 | 0.0 | |
| 3 | 1.0 | 0.0 | |
| 4 | 1.5 | 0.3 | |
| 5 | 1.5 | 0.5 | |

### 실습 4: 레이싱 라인 최적화

1. 느린 속도로 기본 라인 기록
2. RViz에서 시각화
3. 코너 안쪽으로 수정
4. 랩타임 비교

---

## 10. 문제 해결

### 자주 발생하는 문제

| 증상 | 원인 | 해결책 |
|------|------|--------|
| 경로 이탈 | Ld 너무 큼 | Ld 감소 |
| 진동 | Ld 너무 작음 | Ld 증가 |
| 코너 커팅 | Ld 고정값 | 동적 Ld 사용 |
| 코너에서 느림 | 웨이포인트 속도 낮음 | 웨이포인트 속도 조정 |
| 시작 못함 | 첫 웨이포인트 너무 멀음 | 차량 위치 조정 |

### 디버깅 팁

1. **RViz 시각화 필수**
   - 웨이포인트 경로 표시
   - 현재 목표점 표시
   - Lookahead 원 표시

2. **Odometry 확인**
   ```bash
   ros2 topic echo /odom
   ```

3. **조향각 모니터링**
   ```bash
   ros2 topic echo /drive
   ```

---

## 11. 검증 체크리스트

- [ ] 웨이포인트 로드 성공
- [ ] Lookahead point 올바르게 선택
- [ ] 조향각 계산 정확
- [ ] 직선 구간 안정 주행
- [ ] 코너 추종 성공
- [ ] 트랙 완주 (3바퀴)
- [ ] 랩타임 측정

---

*Pure Pursuit은 F1TENTH 대회 우승팀들이 가장 많이 사용하는 경로 추종 알고리즘입니다!*
