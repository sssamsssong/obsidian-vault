# Stanley Controller

> YAX F1TENTH 교육자료 | Phase 4: Autonomy
> 
> 크로스트랙 오차 기반 경로 추종

---

## 1. 개요

### 학습 목표
1. Stanley Controller의 기하학적 원리를 이해할 수 있다
2. 크로스트랙 오차(CTE)와 헤딩 오차를 계산할 수 있다
3. Pure Pursuit과 Stanley의 차이점을 설명할 수 있다

### 예상 소요 시간
- 전체: 2.5시간

### 전제 조건
- Pure Pursuit 완료
- 좌표 변환 이해
- 기본 미적분

---

## 2. 알고리즘 개요

### 2.1 Stanley Controller란?

Stanley Controller는 스탠포드 대학의 자율주행 차량 "Stanley"에서 사용된 경로 추종 알고리즘입니다. 2005년 DARPA Grand Challenge 우승 차량에 적용되었습니다.

### 2.2 핵심 아이디어

**두 가지 오차를 동시에 보정:**

1. **헤딩 오차 (ψ_e)**: 차량 방향과 경로 방향의 차이
2. **크로스트랙 오차 (e)**: 차량과 경로 사이의 수직 거리

```
            경로 방향
               ↗
              /
             /  ψ_e (헤딩 오차)
            /   ↙
        ───●────────────── 경로
           ↑
           │ e (크로스트랙 오차)
           │
      ┌────●────┐
      │ F1TENTH │ → 차량 방향
      └─────────┘
```

### 2.3 Stanley 공식

```
δ = ψ_e + arctan(k * e / v)

δ   : 조향각
ψ_e : 헤딩 오차 (경로 방향 - 차량 방향)
k   : 크로스트랙 게인
e   : 크로스트랙 오차
v   : 현재 속도
```

### 2.4 Pure Pursuit vs Stanley

| 항목 | Pure Pursuit | Stanley |
|------|--------------|---------|
| 기준점 | 후륜축 | **전륜축** |
| 오차 보정 | 목표점 각도만 | 헤딩 + CTE |
| 수렴 속도 | 느림 | 빠름 |
| 저속 안정성 | 좋음 | 진동 가능 |
| 고속 안정성 | 보통 | 좋음 |
| 구현 난이도 | 쉬움 | 중간 |

---

## 3. 기하학적 원리

### 3.1 크로스트랙 오차 계산

```
                    가장 가까운 경로점 (P)
                         ●
                        /│
                       / │
                      /  │ e (CTE)
                     /   │
            경로 ───●────┴────●─── 경로
                              
                         ●  전륜축 위치
                    ┌─────────┐
                    │ F1TENTH │
                    └─────────┘
```

```python
def calculate_crosstrack_error(self, front_axle: np.ndarray, 
                               closest_point: np.ndarray,
                               path_heading: float) -> float:
    """
    크로스트랙 오차 계산.
    
    CTE = 전륜축에서 경로까지의 수직 거리
    부호: 경로 왼쪽이면 +, 오른쪽이면 -
    """
    # 전륜축에서 가장 가까운 경로점까지의 벡터
    dx = front_axle[0] - closest_point[0]
    dy = front_axle[1] - closest_point[1]
    
    # 경로 방향에 수직인 방향으로 투영
    # 경로 방향 벡터: (cos(path_heading), sin(path_heading))
    # 수직 방향 벡터: (-sin(path_heading), cos(path_heading))
    
    crosstrack_error = -np.sin(path_heading) * dx + np.cos(path_heading) * dy
    
    return crosstrack_error
```

### 3.2 헤딩 오차 계산

```python
def calculate_heading_error(self, vehicle_heading: float, 
                            path_heading: float) -> float:
    """
    헤딩 오차 계산.
    
    ψ_e = 경로 방향 - 차량 방향
    """
    heading_error = path_heading - vehicle_heading
    
    # -π ~ π 범위로 정규화
    while heading_error > np.pi:
        heading_error -= 2 * np.pi
    while heading_error < -np.pi:
        heading_error += 2 * np.pi
    
    return heading_error
```

### 3.3 전륜축 위치 계산

Stanley는 **전륜축**을 기준으로 합니다.

```python
def get_front_axle(self, pos: np.ndarray, 
                   yaw: float,
                   wheelbase: float) -> np.ndarray:
    """
    전륜축 위치 계산.
    
    후륜축(Odometry 원점)에서 전방으로 휠베이스만큼 이동
    """
    front_x = pos[0] + wheelbase * np.cos(yaw)
    front_y = pos[1] + wheelbase * np.sin(yaw)
    
    return np.array([front_x, front_y])
```

---

## 4. Stanley Controller 노드

### 4.1 전체 구현

```python
#!/usr/bin/env python3
"""
Stanley Controller Node

크로스트랙 오차 기반 경로 추종 알고리즘
"""

import rclpy
from rclpy.node import Node
from nav_msgs.msg import Odometry, Path
from ackermann_msgs.msg import AckermannDriveStamped
from geometry_msgs.msg import PoseStamped
from visualization_msgs.msg import Marker, MarkerArray
import numpy as np
import csv
from tf_transformations import euler_from_quaternion


class StanleyController(Node):
    def __init__(self):
        super().__init__('stanley_controller')
        
        # === 파라미터 ===
        self.declare_parameter('waypoint_file', 'waypoints.csv')
        self.declare_parameter('crosstrack_gain', 1.0)       # k
        self.declare_parameter('softening_constant', 1.0)    # k_soft
        self.declare_parameter('wheelbase', 0.33)
        self.declare_parameter('default_speed', 2.0)
        self.declare_parameter('max_steering', 0.4)
        self.declare_parameter('min_speed', 0.1)  # 저속 발산 방지
        
        # 파라미터 로드
        self.waypoint_file = self.get_parameter('waypoint_file').value
        self.k = self.get_parameter('crosstrack_gain').value
        self.k_soft = self.get_parameter('softening_constant').value
        self.wheelbase = self.get_parameter('wheelbase').value
        self.default_speed = self.get_parameter('default_speed').value
        self.max_steering = self.get_parameter('max_steering').value
        self.min_speed = self.get_parameter('min_speed').value
        
        # 상태
        self.waypoints = None
        self.path_headings = None
        self.current_speed = 0.0
        
        # 웨이포인트 로드
        self.load_waypoints()
        self.calculate_path_headings()
        
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
            '/stanley_viz',
            10
        )
        
        self.path_pub = self.create_publisher(
            Path,
            '/planned_path',
            10
        )
        
        # 타이머
        self.viz_timer = self.create_timer(0.1, self.publish_visualization)
        
        self.get_logger().info('Stanley Controller started')
        self.get_logger().info(f'  Crosstrack gain (k): {self.k}')
        self.get_logger().info(f'  Waypoints: {len(self.waypoints)}')
    
    def load_waypoints(self):
        """CSV에서 웨이포인트 로드."""
        waypoints = []
        
        try:
            with open(self.waypoint_file, 'r') as f:
                reader = csv.reader(f)
                header = next(reader)
                
                for row in reader:
                    if len(row) >= 2:
                        x = float(row[0])
                        y = float(row[1])
                        speed = float(row[2]) if len(row) >= 3 else self.default_speed
                        waypoints.append([x, y, speed])
            
            self.waypoints = np.array(waypoints)
            
        except FileNotFoundError:
            self.get_logger().error(f'Waypoint file not found: {self.waypoint_file}')
            self.waypoints = np.array([[0, 0, 1.0], [1, 0, 1.0]])
    
    def calculate_path_headings(self):
        """
        각 웨이포인트에서의 경로 방향(헤딩) 계산.
        """
        n = len(self.waypoints)
        self.path_headings = np.zeros(n)
        
        for i in range(n):
            # 다음 점을 향한 방향
            next_idx = (i + 1) % n
            dx = self.waypoints[next_idx, 0] - self.waypoints[i, 0]
            dy = self.waypoints[next_idx, 1] - self.waypoints[i, 1]
            self.path_headings[i] = np.arctan2(dy, dx)
    
    def get_yaw_from_quaternion(self, orientation) -> float:
        """Quaternion에서 Yaw 추출."""
        q = [orientation.x, orientation.y, orientation.z, orientation.w]
        _, _, yaw = euler_from_quaternion(q)
        return yaw
    
    def get_front_axle(self, pos: np.ndarray, yaw: float) -> np.ndarray:
        """전륜축 위치 계산."""
        front_x = pos[0] + self.wheelbase * np.cos(yaw)
        front_y = pos[1] + self.wheelbase * np.sin(yaw)
        return np.array([front_x, front_y])
    
    def find_closest_waypoint(self, front_axle: np.ndarray) -> int:
        """전륜축에서 가장 가까운 웨이포인트 인덱스."""
        distances = np.linalg.norm(self.waypoints[:, :2] - front_axle, axis=1)
        return int(np.argmin(distances))
    
    def calculate_crosstrack_error(self, front_axle: np.ndarray,
                                   closest_idx: int) -> float:
        """
        크로스트랙 오차 계산.
        
        경로 접선에 수직인 방향의 거리.
        """
        closest_point = self.waypoints[closest_idx, :2]
        path_heading = self.path_headings[closest_idx]
        
        # 전륜축에서 가장 가까운 점까지의 벡터
        dx = front_axle[0] - closest_point[0]
        dy = front_axle[1] - closest_point[1]
        
        # 경로 수직 방향으로 투영 (오른쪽이 +)
        crosstrack_error = -np.sin(path_heading) * dx + np.cos(path_heading) * dy
        
        return crosstrack_error
    
    def calculate_heading_error(self, vehicle_yaw: float,
                                closest_idx: int) -> float:
        """
        헤딩 오차 계산.
        """
        path_heading = self.path_headings[closest_idx]
        heading_error = path_heading - vehicle_yaw
        
        # 정규화
        while heading_error > np.pi:
            heading_error -= 2 * np.pi
        while heading_error < -np.pi:
            heading_error += 2 * np.pi
        
        return heading_error
    
    def stanley_control(self, heading_error: float,
                        crosstrack_error: float,
                        speed: float) -> float:
        """
        Stanley 조향각 계산.
        
        δ = ψ_e + arctan(k * e / (v + k_soft))
        
        k_soft: 저속에서의 발산 방지
        """
        # 속도가 너무 낮으면 최소값 사용
        v = max(abs(speed), self.min_speed)
        
        # Stanley 공식
        # 헤딩 오차 보정 + 크로스트랙 오차 보정
        crosstrack_term = np.arctan2(self.k * crosstrack_error, v + self.k_soft)
        steering = heading_error + crosstrack_term
        
        # 조향각 제한
        steering = np.clip(steering, -self.max_steering, self.max_steering)
        
        return float(steering)
    
    def odom_callback(self, msg: Odometry):
        """메인 콜백: Stanley Controller 실행."""
        
        if self.waypoints is None or len(self.waypoints) < 2:
            return
        
        # 현재 상태
        pos = np.array([
            msg.pose.pose.position.x,
            msg.pose.pose.position.y
        ])
        yaw = self.get_yaw_from_quaternion(msg.pose.pose.orientation)
        self.current_speed = msg.twist.twist.linear.x
        
        # 전륜축 위치
        front_axle = self.get_front_axle(pos, yaw)
        
        # 가장 가까운 웨이포인트
        closest_idx = self.find_closest_waypoint(front_axle)
        
        # 오차 계산
        crosstrack_error = self.calculate_crosstrack_error(front_axle, closest_idx)
        heading_error = self.calculate_heading_error(yaw, closest_idx)
        
        # Stanley 조향각
        steering = self.stanley_control(
            heading_error, crosstrack_error, self.current_speed
        )
        
        # 목표 속도
        target_speed = self.waypoints[closest_idx, 2]
        
        # 조향각에 따른 감속
        speed_factor = 1.0 - abs(steering) / self.max_steering * 0.3
        speed = target_speed * speed_factor
        
        # 명령 발행
        drive_msg = AckermannDriveStamped()
        drive_msg.header.stamp = self.get_clock().now().to_msg()
        drive_msg.header.frame_id = 'base_link'
        drive_msg.drive.speed = float(speed)
        drive_msg.drive.steering_angle = steering
        
        self.drive_pub.publish(drive_msg)
        
        # 디버그
        self.get_logger().debug(
            f'CTE: {crosstrack_error:.3f}, HE: {np.degrees(heading_error):.1f}°, '
            f'Steer: {np.degrees(steering):.1f}°'
        )
        
        # 저장 (시각화용)
        self.last_front_axle = front_axle
        self.last_closest_idx = closest_idx
        self.last_cte = crosstrack_error
        self.last_he = heading_error
    
    def publish_visualization(self):
        """RViz 시각화."""
        if self.waypoints is None:
            return
        
        marker_array = MarkerArray()
        
        # 경로
        path_marker = Marker()
        path_marker.header.frame_id = 'map'
        path_marker.header.stamp = self.get_clock().now().to_msg()
        path_marker.ns = 'path'
        path_marker.id = 0
        path_marker.type = Marker.LINE_STRIP
        path_marker.action = Marker.ADD
        path_marker.scale.x = 0.05
        path_marker.color.b = 1.0
        path_marker.color.a = 0.8
        
        from geometry_msgs.msg import Point
        for wp in self.waypoints:
            p = Point()
            p.x = wp[0]
            p.y = wp[1]
            path_marker.points.append(p)
        
        # 경로 닫기
        p = Point()
        p.x = self.waypoints[0, 0]
        p.y = self.waypoints[0, 1]
        path_marker.points.append(p)
        
        marker_array.markers.append(path_marker)
        
        # CTE 표시 (전륜축 → 가장 가까운 점)
        if hasattr(self, 'last_front_axle'):
            cte_marker = Marker()
            cte_marker.header.frame_id = 'map'
            cte_marker.header.stamp = self.get_clock().now().to_msg()
            cte_marker.ns = 'cte'
            cte_marker.id = 1
            cte_marker.type = Marker.LINE_STRIP
            cte_marker.action = Marker.ADD
            cte_marker.scale.x = 0.03
            cte_marker.color.r = 1.0
            cte_marker.color.g = 0.5
            cte_marker.color.a = 1.0
            
            # 전륜축
            p1 = Point()
            p1.x = self.last_front_axle[0]
            p1.y = self.last_front_axle[1]
            cte_marker.points.append(p1)
            
            # 가장 가까운 경로점
            p2 = Point()
            p2.x = self.waypoints[self.last_closest_idx, 0]
            p2.y = self.waypoints[self.last_closest_idx, 1]
            cte_marker.points.append(p2)
            
            marker_array.markers.append(cte_marker)
        
        self.marker_pub.publish(marker_array)


def main(args=None):
    rclpy.init(args=args)
    node = StanleyController()
    
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

### 4.2 Launch 파일

```python
# launch/stanley_launch.py
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
            executable='stanley_controller',
            name='stanley_controller',
            parameters=[{
                'waypoint_file': waypoint_file,
                'crosstrack_gain': 1.0,
                'softening_constant': 1.0,
                'wheelbase': 0.33,
                'default_speed': 2.0,
                'max_steering': 0.4,
                'min_speed': 0.1,
            }],
            output='screen'
        )
    ])
```

### 4.3 파라미터 설정

```yaml
# config/stanley_params.yaml
stanley_controller:
  ros__parameters:
    # 웨이포인트
    waypoint_file: "waypoints.csv"
    
    # Stanley 게인
    crosstrack_gain: 1.0       # k: CTE 보정 강도
    softening_constant: 1.0    # k_soft: 저속 안정화
    
    # 차량 파라미터
    wheelbase: 0.33
    
    # 속도/조향
    default_speed: 2.0
    max_steering: 0.4
    min_speed: 0.1
```

---

## 5. 파라미터 튜닝

### 5.1 크로스트랙 게인 (k)

```
k 작음 (0.5):          k 큼 (2.0):
      경로                   경로
   ──────────             ──────────
       ↗                      ↑
      /                       │
     /  느린 수렴              │ 빠른 수렴
    ●                         ●
```

| k 값 | 효과 |
|------|------|
| 0.5 ~ 1.0 | 부드러운 수렴, 느림 |
| 1.0 ~ 2.0 | 균형 잡힌 동작 |
| 2.0+ | 빠른 수렴, 오버슈트 가능 |

### 5.2 소프트닝 상수 (k_soft)

저속에서의 발산을 방지합니다.

```python
# Stanley 공식에서 분모
denominator = v + k_soft

# v = 0.1 m/s, k_soft = 0 → arctan(k*e/0.1) → 급격한 조향
# v = 0.1 m/s, k_soft = 1 → arctan(k*e/1.1) → 안정적 조향
```

| k_soft | 효과 |
|--------|------|
| 0.0 | 저속에서 불안정 |
| 0.5 | 약간의 안정화 |
| 1.0+ | 저속 안정, 고속에서 둔감 |

### 5.3 권장 시작값

**입문자용:**
```yaml
crosstrack_gain: 0.8
softening_constant: 1.5
default_speed: 1.5
```

**중급자용:**
```yaml
crosstrack_gain: 1.2
softening_constant: 1.0
default_speed: 2.5
```

**고급자용:**
```yaml
crosstrack_gain: 2.0
softening_constant: 0.5
default_speed: 4.0
```

### 5.4 튜닝 프로세스

1. **k_soft 먼저**: 1.0 ~ 2.0으로 시작하여 저속 안정 확보
2. **k 조정**: 작은 값(0.5)에서 시작, 수렴 속도 관찰하며 증가
3. **속도 증가**: 안정되면 speed 증가
4. **k_soft 감소**: 고속 응답성 필요시 감소

---

## 6. Pure Pursuit vs Stanley 비교 실험

### 6.1 실험 설계

동일 트랙, 동일 웨이포인트로 두 알고리즘 비교:

| 측정 항목 | Pure Pursuit | Stanley |
|-----------|--------------|---------|
| 평균 CTE | | |
| 최대 CTE | | |
| 랩타임 | | |
| 조향 부드러움 | | |

### 6.2 실험 코드

```python
class ControllerComparison(Node):
    def __init__(self):
        super().__init__('controller_comparison')
        
        self.cte_history = []
        self.steering_history = []
        self.lap_start_time = None
        self.lap_times = []
        
    def record_metrics(self, cte: float, steering: float):
        self.cte_history.append(abs(cte))
        self.steering_history.append(steering)
    
    def calculate_statistics(self):
        return {
            'mean_cte': np.mean(self.cte_history),
            'max_cte': np.max(self.cte_history),
            'steering_variance': np.var(self.steering_history),
            'avg_lap_time': np.mean(self.lap_times) if self.lap_times else 0
        }
```

### 6.3 예상 결과

| 상황 | 권장 알고리즘 |
|------|---------------|
| 좁은 트랙, 높은 정밀도 필요 | Stanley |
| 넓은 트랙, 부드러운 주행 | Pure Pursuit |
| 저속 주행 | Pure Pursuit |
| 고속 레이싱 | Stanley |
| 급커브 많음 | Stanley |

---

## 7. 고급 기법

### 7.1 적응형 게인

속도와 곡률에 따라 k 조정:

```python
def adaptive_crosstrack_gain(self, speed: float, 
                              curvature: float) -> float:
    """
    상황에 따른 적응형 게인.
    
    - 고속: k 감소 (안정성)
    - 급커브: k 증가 (정밀도)
    """
    base_k = self.k
    
    # 속도 보정
    speed_factor = 1.0 / (1.0 + 0.1 * speed)
    
    # 곡률 보정
    curvature_factor = 1.0 + 2.0 * abs(curvature)
    
    return base_k * speed_factor * curvature_factor
```

### 7.2 Feed-forward 조향

경로 곡률을 미리 반영:

```python
def feedforward_steering(self, curvature: float) -> float:
    """
    경로 곡률에 따른 피드포워드 조향.
    
    δ_ff = arctan(L * κ)
    """
    return np.arctan(self.wheelbase * curvature)

def combined_steering(self, feedback: float, 
                      curvature: float) -> float:
    """Stanley + Feedforward."""
    feedforward = self.feedforward_steering(curvature)
    return feedback + feedforward
```

### 7.3 Look-ahead Stanley

약간 앞의 경로를 기준으로 오차 계산:

```python
def lookahead_stanley(self, front_axle: np.ndarray,
                      yaw: float,
                      lookahead_dist: float = 0.5) -> float:
    """
    전방 lookahead 지점의 오차로 조향.
    """
    # Lookahead 지점
    lookahead_point = np.array([
        front_axle[0] + lookahead_dist * np.cos(yaw),
        front_axle[1] + lookahead_dist * np.sin(yaw)
    ])
    
    # Lookahead 지점에서의 CTE와 HE 계산
    closest_idx = self.find_closest_waypoint(lookahead_point)
    cte = self.calculate_crosstrack_error(lookahead_point, closest_idx)
    he = self.calculate_heading_error(yaw, closest_idx)
    
    return self.stanley_control(he, cte, self.current_speed)
```

---

## 8. 실습

### 실습 1: 기본 Stanley 동작

1. Pure Pursuit에서 사용한 웨이포인트 재사용
2. Stanley Controller 실행
3. CTE 수렴 관찰

```bash
ros2 launch f1tenth_autonomy stanley_launch.py
```

### 실습 2: 게인 튜닝

| 실험 | k | k_soft | speed | CTE 결과 |
|------|---|--------|-------|----------|
| 1 | 0.5 | 1.0 | 2.0 | |
| 2 | 1.0 | 1.0 | 2.0 | |
| 3 | 1.5 | 1.0 | 2.0 | |
| 4 | 1.0 | 0.5 | 2.0 | |
| 5 | 1.0 | 2.0 | 2.0 | |

### 실습 3: Pure Pursuit vs Stanley

동일 조건에서 두 알고리즘 비교:

1. 같은 트랙, 같은 웨이포인트
2. 3바퀴 평균 랩타임 측정
3. CTE 로그 기록

### 실습 4: 고속 주행

속도를 단계적으로 증가시키며 안정성 테스트:

- 2.0 m/s → 3.0 m/s → 4.0 m/s → 5.0 m/s
- 각 속도에서 최적 k 값 기록

---

## 9. 문제 해결

### 자주 발생하는 문제

| 증상 | 원인 | 해결책 |
|------|------|--------|
| 저속에서 진동 | k_soft 너무 작음 | k_soft 증가 |
| 수렴 느림 | k 너무 작음 | k 증가 |
| 오버슈트 | k 너무 큼 | k 감소 |
| 코너에서 이탈 | 속도 대비 k 부족 | k 증가 또는 속도 감소 |
| 전륜축 위치 오류 | wheelbase 부정확 | wheelbase 측정 확인 |

### 디버깅 팁

1. **CTE 모니터링**
   ```bash
   ros2 topic echo /stanley_viz
   ```

2. **RViz에서 CTE 시각화**
   - 전륜축 → 가장 가까운 점 라인 표시
   - 라인 길이 = CTE

3. **게인 동적 조정**
   ```bash
   ros2 param set /stanley_controller crosstrack_gain 1.5
   ```

---

## 10. 검증 체크리스트

- [ ] 전륜축 위치 계산 정확
- [ ] CTE 계산 정확 (부호 확인)
- [ ] 헤딩 오차 계산 정확
- [ ] 저속에서 안정 (k_soft 효과)
- [ ] 고속에서 추종 성능
- [ ] 트랙 완주 (3바퀴)
- [ ] Pure Pursuit과 비교 완료

---

*Stanley Controller는 정밀한 경로 추종이 필요한 상황에서 탁월한 성능을 보여줍니다!*
