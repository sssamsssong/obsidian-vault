# 성능 튜닝

> YAX F1TENTH 교육자료 | Phase 6: Competition
> 
> 랩타임 단축을 위한 최적화 기법

---

## 1. 개요

### 학습 목표
1. 시스템 성능 병목을 분석할 수 있다
2. 알고리즘 파라미터를 체계적으로 튜닝할 수 있다
3. 하드웨어/소프트웨어 최적화를 적용할 수 있다

### 예상 소요 시간
- 전체: 4시간

### 전제 조건
- Phase 4 자율주행 알고리즘 완료
- 기본 주행 가능 상태

---

## 2. 성능 튜닝 개요

### 2.1 성능 구성 요소

```
랩타임 = f(최대 속도, 코너링 효율, 안정성, 시스템 지연)

┌──────────────┐
│   최대 속도   │ ← 직선 구간 속도
├──────────────┤
│  코너링 효율  │ ← 코너 진입/탈출 속도
├──────────────┤
│    안정성    │ ← 충돌 없는 주행
├──────────────┤
│  시스템 지연  │ ← 센서→제어 응답 시간
└──────────────┘
```

### 2.2 튜닝 우선순위

| 순서 | 항목 | 이유 |
|------|------|------|
| 1 | 안정성 확보 | 충돌 시 모든 것 무의미 |
| 2 | 시스템 지연 최소화 | 고속 주행의 기본 조건 |
| 3 | 경로 최적화 | 주행 거리 단축 |
| 4 | 속도 증가 | 안정성 범위 내 최대화 |

---

## 3. 시스템 지연 분석

### 3.1 지연 측정

```python
#!/usr/bin/env python3
"""
시스템 지연 측정 노드
"""

import rclpy
from rclpy.node import Node
from sensor_msgs.msg import LaserScan, Image
from ackermann_msgs.msg import AckermannDriveStamped
import time


class LatencyMeasurement(Node):
    def __init__(self):
        super().__init__('latency_measurement')
        
        self.scan_times = []
        self.drive_times = []
        
        self.scan_sub = self.create_subscription(
            LaserScan, '/scan', self.scan_callback, 10
        )
        
        self.drive_sub = self.create_subscription(
            AckermannDriveStamped, '/drive', self.drive_callback, 10
        )
        
        # 10초마다 통계 출력
        self.timer = self.create_timer(10.0, self.print_stats)
    
    def scan_callback(self, msg):
        now = time.time()
        msg_time = msg.header.stamp.sec + msg.header.stamp.nanosec * 1e-9
        latency = (now - msg_time) * 1000  # ms
        self.scan_times.append(latency)
    
    def drive_callback(self, msg):
        now = time.time()
        msg_time = msg.header.stamp.sec + msg.header.stamp.nanosec * 1e-9
        latency = (now - msg_time) * 1000  # ms
        self.drive_times.append(latency)
    
    def print_stats(self):
        if self.scan_times:
            import numpy as np
            scan_arr = np.array(self.scan_times[-100:])
            print(f"\nLiDAR Latency: {np.mean(scan_arr):.1f} ± {np.std(scan_arr):.1f} ms")
        
        if self.drive_times:
            drive_arr = np.array(self.drive_times[-100:])
            print(f"Drive Latency: {np.mean(drive_arr):.1f} ± {np.std(drive_arr):.1f} ms")
```

### 3.2 지연 요소

| 구성 요소 | 예상 지연 | 최적화 방법 |
|-----------|-----------|-------------|
| LiDAR 수집 | 25 ms (40Hz) | 하드웨어 의존 |
| 카메라 수집 | 33 ms (30fps) | 해상도 감소 |
| 전처리 | 5-10 ms | 최적화 코드 |
| 알고리즘 | 5-20 ms | 효율적 구현 |
| 제어 출력 | 1-2 ms | - |

**목표**: 총 지연 < 50ms (20Hz 제어)

### 3.3 지연 최적화

```python
# 1. ROS2 QoS 최적화
from rclpy.qos import QoSProfile, ReliabilityPolicy, HistoryPolicy

fast_qos = QoSProfile(
    reliability=ReliabilityPolicy.BEST_EFFORT,
    history=HistoryPolicy.KEEP_LAST,
    depth=1  # 최신 메시지만
)

self.scan_sub = self.create_subscription(
    LaserScan, '/scan', self.callback, fast_qos
)

# 2. 콜백 최적화
def callback(self, msg):
    # 무거운 연산 피하기
    # numpy 벡터화 사용
    ranges = np.array(msg.ranges)  # 리스트 → numpy 한 번에
```

---

## 4. 알고리즘 파라미터 튜닝

### 4.1 체계적 튜닝 프로세스

```
1. 기준선 설정 (Baseline)
   └─► 안전한 파라미터로 완주 가능한 상태

2. 단일 파라미터 탐색
   └─► 한 번에 하나씩 변경, 효과 측정

3. 조합 탐색
   └─► 상호작용 확인

4. 검증
   └─► 여러 바퀴 일관성 확인
```

### 4.2 Follow the Gap 튜닝

```python
# 튜닝 가능 파라미터
ftg_params = {
    'bubble_radius': [0.2, 0.25, 0.3, 0.35, 0.4],
    'max_speed': [2.0, 2.5, 3.0, 3.5, 4.0],
    'fov_deg': [160, 180, 200],
    'strategy': ['farthest', 'center']
}

# 튜닝 기록 템플릿
"""
| 실험 | bubble | speed | fov | strategy | 랩타임 | 충돌 | 비고 |
|------|--------|-------|-----|----------|--------|------|------|
| 1    | 0.3    | 2.0   | 180 | farthest | 45.2s  | 0    | 기준선 |
| 2    | 0.3    | 2.5   | 180 | farthest | 40.1s  | 0    | 속도↑ |
| 3    | 0.25   | 2.5   | 180 | farthest | 39.5s  | 1    | 버블↓ |
"""
```

### 4.3 Pure Pursuit 튜닝

```python
pp_params = {
    'lookahead_distance': [0.8, 1.0, 1.2, 1.5, 2.0],
    'lookahead_gain': [0.3, 0.4, 0.5],
    'speed': [2.0, 2.5, 3.0, 3.5, 4.0]
}

# 상황별 권장값
"""
직선 위주 트랙: Ld = 1.5-2.0m, gain = 0.5
코너 위주 트랙: Ld = 0.8-1.2m, gain = 0.3
혼합 트랙: 동적 Ld 사용
"""
```

### 4.4 자동 튜닝 스크립트

```python
#!/usr/bin/env python3
"""
파라미터 그리드 탐색
"""

import itertools
import subprocess
import pandas as pd

def run_experiment(params: dict) -> dict:
    """단일 실험 실행."""
    # 파라미터 설정 및 노드 실행
    # 랩타임 측정
    # 결과 반환
    pass

def grid_search(param_grid: dict, n_laps: int = 3):
    """그리드 탐색."""
    
    # 모든 조합 생성
    keys = list(param_grid.keys())
    values = list(param_grid.values())
    combinations = list(itertools.product(*values))
    
    results = []
    
    for combo in combinations:
        params = dict(zip(keys, combo))
        print(f"Testing: {params}")
        
        lap_times = []
        crashes = 0
        
        for lap in range(n_laps):
            result = run_experiment(params)
            if result['completed']:
                lap_times.append(result['lap_time'])
            else:
                crashes += 1
        
        results.append({
            **params,
            'avg_lap_time': np.mean(lap_times) if lap_times else float('inf'),
            'std_lap_time': np.std(lap_times) if lap_times else 0,
            'crashes': crashes
        })
    
    # 결과 저장
    df = pd.DataFrame(results)
    df.to_csv('tuning_results.csv', index=False)
    
    # 최적 파라미터
    best = df.loc[df['avg_lap_time'].idxmin()]
    print(f"\nBest parameters:\n{best}")
    
    return df
```

---

## 5. 경로 최적화

### 5.1 레이싱 라인

```
일반 경로:                최적 레이싱 라인:
                          
    ┌─────────────┐           ┌─────────────┐
    │             │           │      ╲      │
    │     ╭───╮   │           │       ╲     │
    │    ╱     ╲  │           │        ╲    │
   ─┤   ╱       ╲ ├─         ─┤         ╲   ├─
    │  ╱         ╲│           │──────────╲  │
    │ ╱           │           │           ╲ │
    └─────────────┘           └─────────────┘
    
    코너 중앙 통과              아웃-인-아웃
```

### 5.2 웨이포인트 최적화

```python
def optimize_waypoints(waypoints: np.ndarray,
                       track_width: float = 1.0,
                       smoothing: float = 0.5) -> np.ndarray:
    """
    레이싱 라인 최적화.
    
    1. 코너 감지
    2. 아웃-인-아웃 적용
    3. 곡률 스무딩
    """
    optimized = waypoints.copy()
    
    # 곡률 계산
    curvatures = calculate_curvature(waypoints)
    
    for i in range(1, len(waypoints) - 1):
        if curvatures[i] > 0.1:  # 코너
            # 내측으로 이동
            normal = get_normal_vector(waypoints, i)
            optimized[i] += normal * track_width * 0.3
    
    # 스무딩
    optimized = smooth_path(optimized, smoothing)
    
    return optimized


def smooth_path(waypoints: np.ndarray, 
                weight: float = 0.5,
                iterations: int = 100) -> np.ndarray:
    """경로 스무딩."""
    smoothed = waypoints.copy()
    
    for _ in range(iterations):
        for i in range(1, len(waypoints) - 1):
            smoothed[i] = (
                weight * waypoints[i] + 
                (1 - weight) * 0.5 * (smoothed[i-1] + smoothed[i+1])
            )
    
    return smoothed
```

### 5.3 속도 프로파일

```python
def generate_speed_profile(waypoints: np.ndarray,
                           max_speed: float = 5.0,
                           max_lateral_accel: float = 5.0) -> np.ndarray:
    """
    곡률 기반 속도 프로파일.
    
    v_max = sqrt(a_lateral / curvature)
    """
    curvatures = calculate_curvature(waypoints)
    
    speeds = np.zeros(len(waypoints))
    
    for i, k in enumerate(curvatures):
        if k > 0.01:  # 곡률이 있으면
            speed = np.sqrt(max_lateral_accel / k)
        else:  # 직선
            speed = max_speed
        
        speeds[i] = min(speed, max_speed)
    
    # 가감속 한계 적용
    speeds = apply_acceleration_limits(speeds, waypoints)
    
    return speeds
```

---

## 6. 하드웨어 최적화

### 6.1 Jetson 최적화

```bash
# 최대 성능 모드
sudo nvpmodel -m 0
sudo jetson_clocks

# 불필요한 서비스 중지
sudo systemctl stop lightdm
sudo systemctl stop snapd

# CPU 주파수 고정
sudo cpufreq-set -g performance
```

### 6.2 센서 설정

```yaml
# LiDAR 설정
lidar:
  scan_frequency: 40  # Hz (최대)
  angle_range: 270    # degrees
  
# 카메라 설정
camera:
  resolution: [640, 480]  # 높으면 지연 증가
  fps: 30
  exposure: auto
```

### 6.3 무선 통신 최적화

```bash
# WiFi 전력 관리 비활성화
sudo iw dev wlan0 set power_save off

# SSH 연결 유지
# ~/.ssh/config
Host jetson
    ServerAliveInterval 30
    ServerAliveCountMax 10
```

---

## 7. 디버깅 및 모니터링

### 7.1 실시간 모니터링

```python
#!/usr/bin/env python3
"""
성능 모니터링 대시보드
"""

import rclpy
from rclpy.node import Node
import curses


class PerformanceMonitor(Node):
    def __init__(self, stdscr):
        super().__init__('performance_monitor')
        self.stdscr = stdscr
        
        self.lap_times = []
        self.current_lap_start = None
        self.speed_history = []
        
        # 구독
        # ...
        
        self.timer = self.create_timer(0.1, self.update_display)
    
    def update_display(self):
        self.stdscr.clear()
        
        self.stdscr.addstr(0, 0, "=== F1TENTH Performance Monitor ===")
        self.stdscr.addstr(2, 0, f"Current Lap: {self.current_lap_time():.2f}s")
        self.stdscr.addstr(3, 0, f"Best Lap: {min(self.lap_times) if self.lap_times else 0:.2f}s")
        self.stdscr.addstr(4, 0, f"Avg Speed: {np.mean(self.speed_history[-100:]):.2f} m/s")
        
        self.stdscr.refresh()
```

### 7.2 텔레메트리 로깅

```python
import csv
from datetime import datetime

class TelemetryLogger:
    def __init__(self, log_dir: str):
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        self.filepath = f"{log_dir}/telemetry_{timestamp}.csv"
        
        self.file = open(self.filepath, 'w', newline='')
        self.writer = csv.writer(self.file)
        self.writer.writerow([
            'timestamp', 'x', 'y', 'yaw', 'speed', 
            'steering', 'lap_progress', 'lap_time'
        ])
    
    def log(self, data: dict):
        self.writer.writerow([
            data['timestamp'],
            data['x'], data['y'], data['yaw'],
            data['speed'], data['steering'],
            data['lap_progress'], data['lap_time']
        ])
    
    def close(self):
        self.file.close()
```

---

## 8. 튜닝 워크시트

### 8.1 체크리스트

```markdown
## 기준선 설정
- [ ] 완주 가능한 안전한 파라미터
- [ ] 기준 랩타임 측정 (3회 평균)
- [ ] 시스템 지연 측정

## 시스템 최적화
- [ ] Jetson 최대 성능 모드
- [ ] 불필요한 프로세스 종료
- [ ] ROS2 QoS 최적화

## 알고리즘 튜닝
- [ ] 개별 파라미터 영향 분석
- [ ] 최적 조합 탐색
- [ ] 일관성 검증

## 경로 최적화
- [ ] 레이싱 라인 웨이포인트
- [ ] 속도 프로파일
- [ ] 코너별 설정

## 최종 검증
- [ ] 10바퀴 연속 주행
- [ ] 충돌 없음 확인
- [ ] 랩타임 편차 < 5%
```

### 8.2 기록 템플릿

```markdown
## 세션 #__

날짜: 
트랙: 
조건: 

### 파라미터
| 항목 | 값 |
|------|-----|
| 알고리즘 | |
| bubble_radius | |
| max_speed | |
| lookahead | |

### 결과
| 랩 | 시간 | 충돌 | 비고 |
|----|------|------|------|
| 1 | | | |
| 2 | | | |
| 3 | | | |

평균: 
최고: 

### 관찰
-
-
```

---

## 9. 실습

### 실습 1: 지연 측정

1. 시스템 지연 측정 노드 실행
2. 병목 구간 식별

### 실습 2: 파라미터 튜닝

1. 기준선 랩타임 측정
2. 속도 파라미터 단계적 증가
3. 최적 조합 찾기

### 실습 3: 레이싱 라인

1. 수동 주행으로 레이싱 라인 기록
2. 웨이포인트 최적화 적용
3. 랩타임 비교

---

## 10. 검증 체크리스트

- [ ] 시스템 지연 < 50ms
- [ ] 10바퀴 연속 완주
- [ ] 랩타임 편차 < 5%
- [ ] 파라미터 문서화
- [ ] 튜닝 결과 기록

---

*0.1초의 개선이 순위를 바꿉니다. 체계적으로 튜닝하세요!*
