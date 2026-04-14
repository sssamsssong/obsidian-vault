# ROS2 Bag 데이터 수집

> YAX F1TENTH 교육자료 | Phase 5: Data & ML
> 
> rosbag2를 사용한 주행 데이터 녹화

---

## 1. 개요

### 학습 목표
1. rosbag2의 기본 명령어를 사용할 수 있다
2. 필요한 토픽만 선택적으로 녹화할 수 있다
3. 녹화된 데이터를 재생하고 검증할 수 있다

### 예상 소요 시간
- 전체: 3시간

### 전제 조건
- ROS2 Humble 설치
- 센서 드라이버 설정 완료

---

## 2. ROS2 Bag이란?

### 2.1 개념

ROS2 Bag은 **토픽 메시지를 시간순으로 저장**하는 도구입니다.

```
실시간 주행                      나중에 재생
    │                               │
    ▼                               ▼
┌─────────┐     녹화          ┌─────────┐     재생
│ 센서들  │ ──────────►       │  Bag    │ ──────────► 동일한 메시지
│ 노드들  │     rosbag2       │  파일   │     rosbag2
└─────────┘     record        └─────────┘     play
```

### 2.2 용도

| 용도 | 설명 |
|------|------|
| 데이터 수집 | ML 학습용 데이터 |
| 디버깅 | 문제 상황 재현 |
| 알고리즘 테스트 | 동일 데이터로 반복 테스트 |
| 문서화 | 주행 기록 보관 |

### 2.3 파일 형식

ROS2 Humble 기본: **SQLite3 + CDR**

```
my_bag/
├── metadata.yaml     # 메타데이터
└── my_bag_0.db3      # 실제 데이터 (SQLite)
```

---

## 3. 기본 명령어

### 3.1 녹화 (Record)

```bash
# 모든 토픽 녹화
ros2 bag record -a

# 특정 토픽만 녹화
ros2 bag record /camera/image_raw /scan /odom /drive

# 출력 디렉토리 지정
ros2 bag record -o my_driving_data /camera/image_raw /scan

# 압축 녹화 (용량 절약)
ros2 bag record --compression-mode file --compression-format zstd \
    -o compressed_data /camera/image_raw /scan
```

### 3.2 정보 확인 (Info)

```bash
ros2 bag info my_driving_data

# 출력 예시:
# Files:             my_driving_data_0.db3
# Bag size:          1.2 GB
# Storage id:        sqlite3
# Duration:          120.5s
# Start:             Jan 15 2026 14:30:00.123
# End:               Jan 15 2026 14:32:00.623
# Messages:          36150
# Topic information:
#   /camera/image_raw  1800 msgs : sensor_msgs/msg/Image
#   /scan              4800 msgs : sensor_msgs/msg/LaserScan
#   /odom             12000 msgs : nav_msgs/msg/Odometry
#   /drive             1200 msgs : ackermann_msgs/msg/AckermannDriveStamped
```

### 3.3 재생 (Play)

```bash
# 기본 재생
ros2 bag play my_driving_data

# 속도 조절 (0.5배속)
ros2 bag play my_driving_data --rate 0.5

# 반복 재생
ros2 bag play my_driving_data --loop

# 특정 토픽만 재생
ros2 bag play my_driving_data --topics /scan /odom

# 시작 시간 지정 (10초부터)
ros2 bag play my_driving_data --start-offset 10
```

---

## 4. F1TENTH 데이터 수집

### 4.1 필수 토픽

| 토픽 | 타입 | 용도 | 대략적 크기 |
|------|------|------|-------------|
| `/camera/image_raw` | Image | 시각 입력 | ~1MB/frame |
| `/camera/image_raw/compressed` | CompressedImage | 압축 이미지 | ~50KB/frame |
| `/scan` | LaserScan | LiDAR 입력 | ~10KB/scan |
| `/odom` | Odometry | 위치/속도 | ~1KB/msg |
| `/drive` | AckermannDriveStamped | 제어 명령 (레이블) | ~0.5KB/msg |

### 4.2 권장 녹화 설정

```bash
# 카메라 기반 E2E용 (압축 이미지 사용)
ros2 bag record -o e2e_camera \
    /camera/image_raw/compressed \
    /scan \
    /odom \
    /drive

# LiDAR 기반용
ros2 bag record -o e2e_lidar \
    /scan \
    /odom \
    /drive

# 전체 센서 (디버깅용)
ros2 bag record -o full_sensors \
    /camera/image_raw \
    /camera/depth/image_rect_raw \
    /scan \
    /odom \
    /imu \
    /drive
```

### 4.3 녹화 시 주의사항

1. **저장 공간 확인**
   ```bash
   df -h  # 디스크 용량 확인
   # 1분 녹화 예상: ~200MB (압축 이미지)
   # 10분 녹화 예상: ~2GB
   ```

2. **CPU 부하 모니터링**
   ```bash
   htop  # 녹화 중 CPU 사용률 확인
   ```

3. **드롭된 메시지 확인**
   ```bash
   # 녹화 후
   ros2 bag info my_bag | grep -i drop
   ```

---

## 5. 데이터 수집 워크플로우

### 5.1 수집 세션 관리

```bash
#!/bin/bash
# collect_data.sh

# 세션 ID 생성
SESSION_ID=$(date +%Y%m%d_%H%M%S)
BAG_DIR="$HOME/f1tenth_data/raw"
BAG_NAME="${BAG_DIR}/session_${SESSION_ID}"

# 디렉토리 생성
mkdir -p $BAG_DIR

echo "Starting data collection session: $SESSION_ID"
echo "Output: $BAG_NAME"
echo "Press Ctrl+C to stop recording"

# 녹화 시작
ros2 bag record -o $BAG_NAME \
    /camera/image_raw/compressed \
    /scan \
    /odom \
    /drive \
    --max-bag-size 1073741824  # 1GB per file

echo "Recording stopped. Bag saved to: $BAG_NAME"
```

### 5.2 메타데이터 기록

```python
#!/usr/bin/env python3
"""
세션 메타데이터 기록
"""

import json
import datetime
import os

def create_session_metadata(bag_path: str, notes: str = ""):
    metadata = {
        "session_id": os.path.basename(bag_path),
        "timestamp": datetime.datetime.now().isoformat(),
        "track": "morai_oval",  # 트랙 이름
        "driver": "manual",     # manual / autonomous
        "weather": "clear",
        "notes": notes,
        "hardware": {
            "vehicle": "f1tenth_v2",
            "camera": "realsense_d435",
            "lidar": "hokuyo_ust10lx"
        }
    }
    
    metadata_path = f"{bag_path}_metadata.json"
    with open(metadata_path, 'w') as f:
        json.dump(metadata, f, indent=2)
    
    print(f"Metadata saved to: {metadata_path}")
    return metadata_path

# 사용 예
create_session_metadata(
    "/home/yax/f1tenth_data/raw/session_20260115_143000",
    notes="좌회전 코너 집중 수집"
)
```

---

## 6. 데이터 품질 검증

### 6.1 기본 검증

```bash
# 1. 기본 정보 확인
ros2 bag info my_bag

# 2. 메시지 수 확인 (예상 vs 실제)
# 30fps 카메라, 60초 녹화 → 약 1800개 예상

# 3. 토픽별 주파수 확인
ros2 bag play my_bag &
ros2 topic hz /camera/image_raw/compressed
ros2 topic hz /scan
ros2 topic hz /drive
```

### 6.2 시각적 검증

```bash
# RViz로 재생 확인
ros2 bag play my_bag &
rviz2 -d f1tenth_viz.rviz
```

### 6.3 Python 검증 스크립트

```python
#!/usr/bin/env python3
"""
Bag 파일 품질 검증
"""

from rosbags.rosbag2 import Reader
from rosbags.serde import deserialize_cdr
import numpy as np


def validate_bag(bag_path: str):
    """Bag 파일 검증."""
    
    print(f"Validating: {bag_path}")
    
    with Reader(bag_path) as reader:
        # 토픽 정보
        print("\n=== Topics ===")
        for topic, msgtype, count in reader.topics.values():
            print(f"  {topic}: {count} messages ({msgtype})")
        
        # 시간 범위
        duration = (reader.end_time - reader.start_time) / 1e9
        print(f"\n=== Duration ===")
        print(f"  {duration:.2f} seconds")
        
        # 메시지 간격 분석
        print("\n=== Message Intervals ===")
        
        topic_timestamps = {}
        
        for connection, timestamp, rawdata in reader.messages():
            topic = connection.topic
            if topic not in topic_timestamps:
                topic_timestamps[topic] = []
            topic_timestamps[topic].append(timestamp)
        
        for topic, timestamps in topic_timestamps.items():
            if len(timestamps) < 2:
                continue
            
            intervals = np.diff(timestamps) / 1e9  # 초 단위
            mean_hz = 1.0 / np.mean(intervals)
            std_hz = np.std(1.0 / intervals)
            
            print(f"  {topic}:")
            print(f"    Frequency: {mean_hz:.1f} Hz (std: {std_hz:.1f})")
            print(f"    Messages: {len(timestamps)}")
            
            # 누락된 메시지 감지
            expected_interval = 1.0 / mean_hz
            gaps = intervals[intervals > expected_interval * 2]
            if len(gaps) > 0:
                print(f"    WARNING: {len(gaps)} potential dropped messages")
        
        print("\n=== Validation Complete ===")


if __name__ == '__main__':
    import sys
    if len(sys.argv) > 1:
        validate_bag(sys.argv[1])
    else:
        print("Usage: python validate_bag.py <bag_path>")
```

---

## 7. 고급 기능

### 7.1 필터링 및 변환

```bash
# 특정 토픽만 추출
ros2 bag filter input_bag -o output_bag \
    --include-topic /scan --include-topic /drive

# 시간 범위 추출 (10초 ~ 60초)
ros2 bag filter input_bag -o output_bag \
    --start-time 10 --end-time 60
```

### 7.2 토픽 리매핑

```bash
# 재생 시 토픽 이름 변경
ros2 bag play my_bag --remap /camera/image_raw:=/image
```

### 7.3 QoS 설정

```bash
# Best effort QoS로 녹화 (센서 데이터에 적합)
ros2 bag record --qos-profile-overrides-path qos_overrides.yaml \
    /camera/image_raw /scan
```

```yaml
# qos_overrides.yaml
/camera/image_raw:
  reliability: best_effort
  durability: volatile
/scan:
  reliability: best_effort
  durability: volatile
```

---

## 8. 저장 공간 관리

### 8.1 용량 예측

| 토픽 | 주파수 | 크기/메시지 | 1분당 용량 |
|------|--------|-------------|------------|
| 원본 이미지 (640x480) | 30Hz | ~900KB | ~1.6GB |
| 압축 이미지 (JPEG) | 30Hz | ~50KB | ~90MB |
| LiDAR | 40Hz | ~10KB | ~24MB |
| Odometry | 100Hz | ~1KB | ~6MB |
| Drive | 20Hz | ~0.5KB | ~0.6MB |

**권장: 압축 이미지 사용 시 1분당 ~120MB**

### 8.2 자동 정리 스크립트

```bash
#!/bin/bash
# cleanup_old_bags.sh

BAG_DIR="$HOME/f1tenth_data/raw"
DAYS_TO_KEEP=7

echo "Cleaning bags older than $DAYS_TO_KEEP days..."

find $BAG_DIR -name "*.db3" -mtime +$DAYS_TO_KEEP -exec rm -v {} \;
find $BAG_DIR -name "metadata.yaml" -mtime +$DAYS_TO_KEEP -exec rm -v {} \;

# 빈 디렉토리 삭제
find $BAG_DIR -type d -empty -delete

echo "Cleanup complete."
```

---

## 9. 실습

### 실습 1: 첫 녹화

1. MORAI 시뮬레이터 실행
2. 텔레오퍼레이션으로 1분간 주행
3. 데이터 녹화

```bash
# 터미널 1
ros2 launch morai_sim track.launch.py

# 터미널 2
ros2 run teleop_twist_keyboard teleop_twist_keyboard

# 터미널 3
ros2 bag record -o practice_001 \
    /camera/image_raw/compressed /scan /odom /drive
```

### 실습 2: 데이터 검증

1. 녹화 정보 확인
2. 재생하며 RViz 시각화
3. Python 스크립트로 검증

### 실습 3: 대용량 수집

1. 10분 연속 녹화
2. 저장 공간 모니터링
3. 파일 크기 확인

---

## 10. 문제 해결

### 자주 발생하는 문제

| 증상 | 원인 | 해결책 |
|------|------|--------|
| 녹화가 느림 | 디스크 I/O 부족 | SSD 사용, 압축 옵션 |
| 메시지 누락 | CPU 과부하 | 토픽 수 줄이기 |
| 재생 안됨 | QoS 불일치 | `--qos-profile-overrides-path` |
| 파일 손상 | 갑작스런 종료 | Ctrl+C로 정상 종료 |

### 디버깅 명령어

```bash
# 토픽 목록 확인
ros2 topic list

# 토픽 주파수 확인
ros2 topic hz /camera/image_raw

# 메시지 내용 확인
ros2 topic echo /drive --once

# 시스템 리소스 확인
htop
iotop
```

---

## 11. 검증 체크리스트

- [ ] rosbag2 record 명령 이해
- [ ] 필요한 토픽 선택적 녹화
- [ ] bag info로 정보 확인
- [ ] bag play로 재생
- [ ] 압축 녹화 테스트
- [ ] 데이터 품질 검증 스크립트 실행

---

*좋은 데이터가 좋은 모델을 만듭니다. 품질 높은 데이터 수집에 시간을 투자하세요!*
