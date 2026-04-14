# 자율주행 알고리즘 가이드

> YAX F1TENTH 교육자료 | Domain 06: 자율주행

---

## 1. 개요

### 학습 목표
1. 반응형 자율주행 알고리즘을 구현할 수 있다
2. Follow the Gap으로 장애물을 회피할 수 있다
3. Pure Pursuit으로 경로를 추적할 수 있다
4. 파라미터를 튜닝하여 성능을 개선할 수 있다

### 예상 소요 시간: 8-10시간

---

## 2. 알고리즘 분류

| 유형 | 알고리즘 | 특징 |
|------|----------|------|
| 반응형 | Wall Following, FTG | 지도 불필요, 실시간 반응 |
| 경로 추적 | Pure Pursuit, Stanley | 웨이포인트 필요, 정확한 추적 |
| 최적 제어 | MPC, LQR | 계산량 많음, 최적 성능 |

---

## 3. Follow the Gap (FTG)

### 3.1 알고리즘 단계

1. **전처리**: inf/nan 처리
2. **최근접점 찾기**: 가장 가까운 장애물
3. **안전 버블**: 최근접점 주변 0으로 설정
4. **최대 Gap 찾기**: 가장 넓은 빈 공간
5. **조향각 계산**: Gap 중심으로 조향

### 3.2 Python 구현

```python
import numpy as np
import rclpy
from rclpy.node import Node
from sensor_msgs.msg import LaserScan
from ackermann_msgs.msg import AckermannDriveStamped

class FollowTheGap(Node):
    def __init__(self):
        super().__init__('follow_the_gap')
        
        self.scan_sub = self.create_subscription(
            LaserScan, '/scan', self.scan_callback, 10)
        self.drive_pub = self.create_publisher(
            AckermannDriveStamped, '/drive', 10)
        
        # 파라미터
        self.bubble_radius = 0.3   # 안전 버블 (m)
        self.max_speed = 2.0       # 최대 속도
        self.min_speed = 0.5       # 최소 속도
        
    def preprocess(self, ranges):
        """inf, nan 처리"""
        proc = np.array(ranges)
        proc[np.isinf(proc)] = 10.0
        proc[np.isnan(proc)] = 0.0
        return proc
    
    def find_closest(self, ranges):
        """최근접점 인덱스"""
        return np.argmin(ranges)
    
    def create_bubble(self, ranges, idx, angle_inc):
        """안전 버블 생성"""
        bubble_angle = np.arctan(self.bubble_radius / max(ranges[idx], 0.1))
        bubble_size = int(bubble_angle / angle_inc)
        
        start = max(0, idx - bubble_size)
        end = min(len(ranges), idx + bubble_size)
        ranges[start:end] = 0
        return ranges
    
    def find_max_gap(self, ranges):
        """최대 Gap 찾기"""
        # 0이 아닌 연속 구간 찾기
        nonzero = np.nonzero(ranges > 0.1)[0]
        if len(nonzero) == 0:
            return 0, len(ranges) - 1
        
        # 연속 구간 중 가장 긴 것
        gaps = np.split(nonzero, np.where(np.diff(nonzero) != 1)[0] + 1)
        longest = max(gaps, key=len)
        
        return longest[0], longest[-1]
    
    def get_best_point(self, ranges, start, end):
        """Gap 내 최적점 (가장 먼 점)"""
        return start + np.argmax(ranges[start:end+1])
    
    def scan_callback(self, msg):
        ranges = self.preprocess(msg.ranges)
        
        # 1. 최근접점 찾기
        closest_idx = self.find_closest(ranges)
        
        # 2. 안전 버블 생성
        ranges = self.create_bubble(ranges.copy(), closest_idx, msg.angle_increment)
        
        # 3. 최대 Gap 찾기
        start, end = self.find_max_gap(ranges)
        
        # 4. 최적점 선택
        best_idx = self.get_best_point(ranges, start, end)
        
        # 5. 조향각 계산
        steering = msg.angle_min + best_idx * msg.angle_increment
        
        # 6. 속도 계산 (조향각에 따라 감속)
        speed = self.max_speed - (self.max_speed - self.min_speed) * abs(steering) / 0.4
        
        # 7. 발행
        drive_msg = AckermannDriveStamped()
        drive_msg.drive.speed = float(speed)
        drive_msg.drive.steering_angle = float(steering)
        self.drive_pub.publish(drive_msg)

def main():
    rclpy.init()
    node = FollowTheGap()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

---

## 4. Pure Pursuit

### 4.1 알고리즘 개요

미리 정의된 경로(웨이포인트) 위의 목표점을 향해 조향합니다.

### 4.2 핵심 공식

```
steering_angle = arctan(2 * L * sin(alpha) / lookahead_distance)

L: 휠베이스 (0.33m)
alpha: 현재 방향과 목표점 사이 각도
```

### 4.3 Python 구현

```python
import numpy as np
from nav_msgs.msg import Odometry
from ackermann_msgs.msg import AckermannDriveStamped
from tf_transformations import euler_from_quaternion

class PurePursuit(Node):
    def __init__(self):
        super().__init__('pure_pursuit')
        
        self.odom_sub = self.create_subscription(
            Odometry, '/odom', self.odom_callback, 10)
        self.drive_pub = self.create_publisher(
            AckermannDriveStamped, '/drive', 10)
        
        # 웨이포인트 로드
        self.waypoints = self.load_waypoints('waypoints.csv')
        self.lookahead = 1.5  # Lookahead distance
        self.wheelbase = 0.33
        self.speed = 2.0
        
    def load_waypoints(self, filename):
        """CSV에서 웨이포인트 로드: x, y, (선택: speed)"""
        return np.loadtxt(filename, delimiter=',')
    
    def find_lookahead_point(self, pos):
        """현재 위치에서 lookahead 거리의 웨이포인트 찾기"""
        distances = np.linalg.norm(self.waypoints[:, :2] - pos, axis=1)
        
        # lookahead 거리보다 먼 첫 번째 점
        for i, d in enumerate(distances):
            if d >= self.lookahead:
                return self.waypoints[i]
        
        return self.waypoints[-1]
    
    def get_yaw(self, orientation):
        """Quaternion → Yaw"""
        q = [orientation.x, orientation.y, orientation.z, orientation.w]
        _, _, yaw = euler_from_quaternion(q)
        return yaw
    
    def odom_callback(self, msg):
        # 현재 위치
        pos = np.array([msg.pose.pose.position.x, msg.pose.pose.position.y])
        yaw = self.get_yaw(msg.pose.pose.orientation)
        
        # 목표점 찾기
        target = self.find_lookahead_point(pos)
        
        # 목표점까지의 각도
        dx = target[0] - pos[0]
        dy = target[1] - pos[1]
        alpha = np.arctan2(dy, dx) - yaw
        
        # 조향각 계산
        steering = np.arctan2(2 * self.wheelbase * np.sin(alpha), self.lookahead)
        steering = np.clip(steering, -0.4, 0.4)
        
        # 발행
        drive_msg = AckermannDriveStamped()
        drive_msg.drive.speed = self.speed
        drive_msg.drive.steering_angle = float(steering)
        self.drive_pub.publish(drive_msg)
```

### 4.4 웨이포인트 생성

수동 주행하며 경로 기록:

```python
class WaypointRecorder(Node):
    def __init__(self):
        super().__init__('waypoint_recorder')
        self.odom_sub = self.create_subscription(
            Odometry, '/odom', self.odom_callback, 10)
        self.waypoints = []
        self.last_pos = None
        self.min_distance = 0.5  # 50cm 간격
        
    def odom_callback(self, msg):
        pos = np.array([msg.pose.pose.position.x, msg.pose.pose.position.y])
        
        if self.last_pos is None or np.linalg.norm(pos - self.last_pos) > self.min_distance:
            self.waypoints.append(pos)
            self.last_pos = pos
            self.get_logger().info(f'Waypoint {len(self.waypoints)}: {pos}')
    
    def save(self, filename):
        np.savetxt(filename, self.waypoints, delimiter=',')
```

---

## 5. 알고리즘 비교

| 알고리즘 | 속도 | 안정성 | 난이도 | 용도 |
|----------|------|--------|--------|------|
| Wall Following | ★★ | ★★★★ | ★★ | 연습 |
| Follow the Gap | ★★★ | ★★★ | ★★★ | 장애물 회피 |
| Pure Pursuit | ★★★★ | ★★★ | ★★★ | 트랙 주행 |
| MPC | ★★★★★ | ★★★★ | ★★★★★ | 대회 |

---

## 6. 파라미터 튜닝

### FTG 튜닝

| 파라미터 | 효과 |
|----------|------|
| bubble_radius ↑ | 더 안전, 느려짐 |
| max_speed ↑ | 빨라짐, 불안정 |

### Pure Pursuit 튜닝

| 파라미터 | 효과 |
|----------|------|
| lookahead ↑ | 부드러움, 코너 커팅 |
| lookahead ↓ | 정확함, 진동 가능 |

---

## 7. 검증 체크리스트

- [ ] FTG로 장애물 회피 동작
- [ ] Pure Pursuit으로 트랙 완주
- [ ] 랩타임 측정
- [ ] 파라미터 튜닝으로 성능 개선

---

*YAX F1TENTH 교육자료 | 작성일: 2026-02-10*
