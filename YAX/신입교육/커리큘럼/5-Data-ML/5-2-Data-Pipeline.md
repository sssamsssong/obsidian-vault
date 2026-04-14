# 데이터 파이프라인

> YAX F1TENTH 교육자료 | Phase 5: Data & ML
> 
> ROS2 Bag에서 학습 데이터셋 추출

---

## 1. 개요

### 학습 목표
1. Bag 파일에서 이미지/LiDAR 데이터를 추출할 수 있다
2. 센서 데이터와 제어 명령을 시간 동기화할 수 있다
3. 학습에 적합한 형식으로 데이터를 저장할 수 있다

### 예상 소요 시간
- 전체: 4시간

### 전제 조건
- ROS2 Bag 수집 완료
- Python 기초

---

## 2. 파이프라인 개요

```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐     ┌──────────────┐
│  ROS2 Bag   │────►│   추출기     │────►│  동기화기   │────►│  데이터셋    │
│  (.db3)     │     │  (Extract)   │     │  (Sync)     │     │  (Dataset)   │
└─────────────┘     └──────────────┘     └─────────────┘     └──────────────┘
      │                    │                    │                    │
      │                    ▼                    ▼                    ▼
      │              이미지 저장          타임스탬프 매칭        images/
      │              LiDAR 저장           레이블 생성           labels.csv
      │              메타데이터                                  metadata.json
```

---

## 3. rosbags 라이브러리

### 3.1 설치

```bash
pip install rosbags
```

### 3.2 기본 사용법

```python
from rosbags.rosbag2 import Reader
from rosbags.serde import deserialize_cdr

# Bag 읽기
with Reader('my_bag') as reader:
    # 연결 정보 출력
    for connection in reader.connections:
        print(f"Topic: {connection.topic}")
        print(f"  Type: {connection.msgtype}")
        print(f"  Count: {connection.msgcount}")
    
    # 메시지 순회
    for connection, timestamp, rawdata in reader.messages():
        msg = deserialize_cdr(rawdata, connection.msgtype)
        print(f"{timestamp}: {connection.topic}")
```

---

## 4. 이미지 추출

### 4.1 원본 이미지 추출

```python
#!/usr/bin/env python3
"""
Bag에서 이미지 추출
"""

import os
import numpy as np
import cv2
from rosbags.rosbag2 import Reader
from rosbags.serde import deserialize_cdr


def extract_images(bag_path: str, output_dir: str, topic: str = '/camera/image_raw'):
    """
    Bag에서 이미지 추출.
    
    Args:
        bag_path: Bag 파일 경로
        output_dir: 출력 디렉토리
        topic: 이미지 토픽 이름
    """
    os.makedirs(output_dir, exist_ok=True)
    
    image_data = []  # (timestamp, filename) 저장
    
    with Reader(bag_path) as reader:
        # 토픽 연결 찾기
        connections = [c for c in reader.connections if c.topic == topic]
        
        if not connections:
            print(f"Topic {topic} not found!")
            return []
        
        conn = connections[0]
        print(f"Extracting images from: {topic}")
        print(f"  Message type: {conn.msgtype}")
        print(f"  Total messages: {conn.msgcount}")
        
        for idx, (connection, timestamp, rawdata) in enumerate(
            reader.messages(connections=[conn])
        ):
            msg = deserialize_cdr(rawdata, connection.msgtype)
            
            # 메시지 타입에 따른 처리
            if 'CompressedImage' in connection.msgtype:
                # 압축 이미지
                img_array = np.frombuffer(msg.data, np.uint8)
                img = cv2.imdecode(img_array, cv2.IMREAD_COLOR)
            else:
                # 원본 이미지
                if msg.encoding == 'rgb8':
                    img = np.frombuffer(msg.data, np.uint8).reshape(
                        msg.height, msg.width, 3
                    )
                    img = cv2.cvtColor(img, cv2.COLOR_RGB2BGR)
                elif msg.encoding == 'bgr8':
                    img = np.frombuffer(msg.data, np.uint8).reshape(
                        msg.height, msg.width, 3
                    )
                else:
                    print(f"Unknown encoding: {msg.encoding}")
                    continue
            
            # 파일명 생성
            filename = f"img_{idx:06d}.jpg"
            filepath = os.path.join(output_dir, filename)
            
            # 저장
            cv2.imwrite(filepath, img)
            image_data.append((timestamp, filename))
            
            if idx % 100 == 0:
                print(f"  Extracted: {idx}/{conn.msgcount}")
    
    print(f"Extracted {len(image_data)} images to {output_dir}")
    return image_data


if __name__ == '__main__':
    extract_images(
        'session_001',
        'output/images',
        '/camera/image_raw/compressed'
    )
```

### 4.2 Depth 이미지 추출

```python
def extract_depth_images(bag_path: str, output_dir: str, 
                         topic: str = '/camera/depth/image_rect_raw'):
    """Depth 이미지 추출 (16-bit PNG)."""
    
    os.makedirs(output_dir, exist_ok=True)
    depth_data = []
    
    with Reader(bag_path) as reader:
        connections = [c for c in reader.connections if c.topic == topic]
        
        if not connections:
            return []
        
        for idx, (connection, timestamp, rawdata) in enumerate(
            reader.messages(connections=connections)
        ):
            msg = deserialize_cdr(rawdata, connection.msgtype)
            
            # 16-bit depth
            depth = np.frombuffer(msg.data, np.uint16).reshape(
                msg.height, msg.width
            )
            
            filename = f"depth_{idx:06d}.png"
            filepath = os.path.join(output_dir, filename)
            
            cv2.imwrite(filepath, depth)
            depth_data.append((timestamp, filename))
    
    return depth_data
```

---

## 5. LiDAR 데이터 추출

### 5.1 LaserScan 추출

```python
def extract_lidar(bag_path: str, output_dir: str, 
                  topic: str = '/scan'):
    """
    LiDAR 데이터 추출.
    
    Returns:
        List of (timestamp, filename, ranges, angles)
    """
    os.makedirs(output_dir, exist_ok=True)
    lidar_data = []
    
    with Reader(bag_path) as reader:
        connections = [c for c in reader.connections if c.topic == topic]
        
        if not connections:
            return []
        
        for idx, (connection, timestamp, rawdata) in enumerate(
            reader.messages(connections=connections)
        ):
            msg = deserialize_cdr(rawdata, connection.msgtype)
            
            # LaserScan 데이터
            ranges = np.array(msg.ranges, dtype=np.float32)
            
            # 각도 배열 계산
            angles = np.linspace(
                msg.angle_min, msg.angle_max, len(ranges)
            ).astype(np.float32)
            
            # NPZ로 저장
            filename = f"lidar_{idx:06d}.npz"
            filepath = os.path.join(output_dir, filename)
            
            np.savez_compressed(filepath, 
                               ranges=ranges, 
                               angles=angles,
                               angle_min=msg.angle_min,
                               angle_max=msg.angle_max,
                               range_min=msg.range_min,
                               range_max=msg.range_max)
            
            lidar_data.append((timestamp, filename))
    
    return lidar_data
```

### 5.2 LiDAR 시각화

```python
def visualize_lidar(npz_path: str):
    """LiDAR 데이터 시각화."""
    import matplotlib.pyplot as plt
    
    data = np.load(npz_path)
    ranges = data['ranges']
    angles = data['angles']
    
    # 극좌표 플롯
    fig, ax = plt.subplots(subplot_kw={'projection': 'polar'})
    ax.scatter(angles, ranges, s=1, c='blue')
    ax.set_rmax(10.0)
    ax.set_title('LiDAR Scan')
    plt.show()
```

---

## 6. 제어 명령 추출 (레이블)

### 6.1 Drive 메시지 추출

```python
def extract_drive_commands(bag_path: str, 
                           topic: str = '/drive'):
    """
    제어 명령 추출 (학습 레이블).
    
    Returns:
        List of (timestamp, steering, speed)
    """
    commands = []
    
    with Reader(bag_path) as reader:
        connections = [c for c in reader.connections if c.topic == topic]
        
        if not connections:
            return []
        
        for connection, timestamp, rawdata in reader.messages(connections=connections):
            msg = deserialize_cdr(rawdata, connection.msgtype)
            
            steering = msg.drive.steering_angle
            speed = msg.drive.speed
            
            commands.append((timestamp, steering, speed))
    
    return commands
```

### 6.2 Odometry 추출

```python
def extract_odometry(bag_path: str, 
                     topic: str = '/odom'):
    """Odometry 데이터 추출."""
    from tf_transformations import euler_from_quaternion
    
    odom_data = []
    
    with Reader(bag_path) as reader:
        connections = [c for c in reader.connections if c.topic == topic]
        
        for connection, timestamp, rawdata in reader.messages(connections=connections):
            msg = deserialize_cdr(rawdata, connection.msgtype)
            
            x = msg.pose.pose.position.x
            y = msg.pose.pose.position.y
            
            q = msg.pose.pose.orientation
            _, _, yaw = euler_from_quaternion([q.x, q.y, q.z, q.w])
            
            vx = msg.twist.twist.linear.x
            
            odom_data.append((timestamp, x, y, yaw, vx))
    
    return odom_data
```

---

## 7. 시간 동기화

### 7.1 동기화의 중요성

센서들의 주파수가 다르기 때문에 동기화가 필요합니다:

```
카메라:    ──●───────●───────●───────●───────●──  (30Hz)
LiDAR:     ─●──●──●──●──●──●──●──●──●──●──●──●─  (40Hz)
Drive:     ────●─────────●─────────●─────────●─  (20Hz)

동기화 후:
           ──●───────●───────●───────●───────●──
             ↓       ↓       ↓       ↓       ↓
           이미지에 가장 가까운 LiDAR와 Drive 매칭
```

### 7.2 시간 기반 동기화

```python
def synchronize_data(image_data: list, 
                     lidar_data: list,
                     drive_data: list,
                     max_time_diff: float = 0.05):  # 50ms
    """
    이미지 타임스탬프 기준으로 데이터 동기화.
    
    Args:
        image_data: [(timestamp, filename), ...]
        lidar_data: [(timestamp, filename), ...]
        drive_data: [(timestamp, steering, speed), ...]
        max_time_diff: 최대 허용 시간 차이 (초)
    
    Returns:
        동기화된 데이터 리스트
    """
    max_diff_ns = max_time_diff * 1e9  # 나노초 변환
    
    # 타임스탬프 배열 생성
    lidar_timestamps = np.array([d[0] for d in lidar_data])
    drive_timestamps = np.array([d[0] for d in drive_data])
    
    synchronized = []
    
    for img_ts, img_file in image_data:
        # 가장 가까운 LiDAR 찾기
        lidar_idx = np.argmin(np.abs(lidar_timestamps - img_ts))
        lidar_diff = abs(lidar_timestamps[lidar_idx] - img_ts)
        
        if lidar_diff > max_diff_ns:
            continue  # 너무 차이 나면 스킵
        
        # 가장 가까운 Drive 명령 찾기
        drive_idx = np.argmin(np.abs(drive_timestamps - img_ts))
        drive_diff = abs(drive_timestamps[drive_idx] - img_ts)
        
        if drive_diff > max_diff_ns:
            continue
        
        # 동기화된 데이터 저장
        synchronized.append({
            'timestamp': img_ts,
            'image': img_file,
            'lidar': lidar_data[lidar_idx][1],
            'steering': drive_data[drive_idx][1],
            'speed': drive_data[drive_idx][2]
        })
    
    print(f"Synchronized {len(synchronized)} samples "
          f"(from {len(image_data)} images)")
    
    return synchronized
```

---

## 8. 전체 파이프라인

### 8.1 통합 추출 스크립트

```python
#!/usr/bin/env python3
"""
Bag → Dataset 변환 파이프라인
"""

import os
import json
import pandas as pd
from pathlib import Path


class BagExtractor:
    def __init__(self, bag_path: str, output_dir: str):
        self.bag_path = bag_path
        self.output_dir = Path(output_dir)
        
        # 출력 디렉토리 생성
        self.images_dir = self.output_dir / 'images'
        self.lidar_dir = self.output_dir / 'lidar'
        
        self.images_dir.mkdir(parents=True, exist_ok=True)
        self.lidar_dir.mkdir(parents=True, exist_ok=True)
    
    def extract_all(self):
        """전체 추출 실행."""
        
        print("=" * 50)
        print(f"Extracting from: {self.bag_path}")
        print(f"Output to: {self.output_dir}")
        print("=" * 50)
        
        # 1. 이미지 추출
        print("\n[1/4] Extracting images...")
        image_data = extract_images(
            self.bag_path,
            str(self.images_dir),
            '/camera/image_raw/compressed'
        )
        
        # 2. LiDAR 추출
        print("\n[2/4] Extracting LiDAR...")
        lidar_data = extract_lidar(
            self.bag_path,
            str(self.lidar_dir),
            '/scan'
        )
        
        # 3. Drive 명령 추출
        print("\n[3/4] Extracting drive commands...")
        drive_data = extract_drive_commands(self.bag_path, '/drive')
        
        # 4. 동기화
        print("\n[4/4] Synchronizing...")
        synchronized = synchronize_data(image_data, lidar_data, drive_data)
        
        # 5. CSV 저장
        self.save_labels(synchronized)
        
        # 6. 메타데이터 저장
        self.save_metadata(len(synchronized), len(image_data))
        
        print("\n" + "=" * 50)
        print("Extraction complete!")
        print(f"  Total samples: {len(synchronized)}")
        print(f"  Labels saved to: {self.output_dir / 'labels.csv'}")
        print("=" * 50)
        
        return synchronized
    
    def save_labels(self, synchronized: list):
        """레이블 CSV 저장."""
        df = pd.DataFrame(synchronized)
        df.to_csv(self.output_dir / 'labels.csv', index=False)
    
    def save_metadata(self, n_samples: int, n_raw_images: int):
        """메타데이터 저장."""
        metadata = {
            'bag_path': str(self.bag_path),
            'n_samples': n_samples,
            'n_raw_images': n_raw_images,
            'sync_rate': n_samples / n_raw_images if n_raw_images > 0 else 0,
            'output_dir': str(self.output_dir)
        }
        
        with open(self.output_dir / 'metadata.json', 'w') as f:
            json.dump(metadata, f, indent=2)


def main():
    import argparse
    
    parser = argparse.ArgumentParser(description='Extract dataset from ROS2 bag')
    parser.add_argument('bag_path', help='Path to bag file/directory')
    parser.add_argument('output_dir', help='Output directory')
    
    args = parser.parse_args()
    
    extractor = BagExtractor(args.bag_path, args.output_dir)
    extractor.extract_all()


if __name__ == '__main__':
    main()
```

### 8.2 사용 예시

```bash
# 단일 bag 변환
python bag_extractor.py session_001 ./data/processed/session_001

# 여러 bag 일괄 변환
for bag in raw/session_*; do
    name=$(basename $bag)
    python bag_extractor.py $bag ./data/processed/$name
done
```

---

## 9. 데이터셋 분할

### 9.1 Train/Val/Test 분할

```python
def split_dataset(data_dir: str, 
                  train_ratio: float = 0.8,
                  val_ratio: float = 0.1,
                  test_ratio: float = 0.1,
                  seed: int = 42):
    """
    데이터셋을 Train/Val/Test로 분할.
    """
    import random
    
    random.seed(seed)
    
    # 레이블 로드
    labels_path = os.path.join(data_dir, 'labels.csv')
    df = pd.read_csv(labels_path)
    
    # 인덱스 셔플
    indices = list(range(len(df)))
    random.shuffle(indices)
    
    # 분할
    n_train = int(len(indices) * train_ratio)
    n_val = int(len(indices) * val_ratio)
    
    train_indices = indices[:n_train]
    val_indices = indices[n_train:n_train + n_val]
    test_indices = indices[n_train + n_val:]
    
    # 분할 파일 저장
    splits_dir = os.path.join(data_dir, 'splits')
    os.makedirs(splits_dir, exist_ok=True)
    
    for name, idx_list in [('train', train_indices), 
                           ('val', val_indices), 
                           ('test', test_indices)]:
        filepath = os.path.join(splits_dir, f'{name}.txt')
        with open(filepath, 'w') as f:
            for idx in idx_list:
                f.write(f"{df.iloc[idx]['image']}\n")
        print(f"{name}: {len(idx_list)} samples")
    
    return train_indices, val_indices, test_indices
```

### 9.2 시퀀스 기반 분할

연속 프레임이 train/val에 섞이지 않도록:

```python
def split_by_sequence(data_dir: str,
                      sequence_length: int = 100):
    """
    시퀀스 단위로 분할 (데이터 누수 방지).
    """
    df = pd.read_csv(os.path.join(data_dir, 'labels.csv'))
    
    n_sequences = len(df) // sequence_length
    sequence_indices = list(range(n_sequences))
    random.shuffle(sequence_indices)
    
    # 시퀀스 기준 분할
    n_train_seq = int(n_sequences * 0.8)
    n_val_seq = int(n_sequences * 0.1)
    
    train_seqs = sequence_indices[:n_train_seq]
    val_seqs = sequence_indices[n_train_seq:n_train_seq + n_val_seq]
    test_seqs = sequence_indices[n_train_seq + n_val_seq:]
    
    # 시퀀스 → 개별 인덱스 변환
    def seqs_to_indices(seqs):
        indices = []
        for s in seqs:
            start = s * sequence_length
            end = min(start + sequence_length, len(df))
            indices.extend(range(start, end))
        return indices
    
    return (seqs_to_indices(train_seqs),
            seqs_to_indices(val_seqs),
            seqs_to_indices(test_seqs))
```

---

## 10. 데이터셋 구조

### 10.1 최종 디렉토리 구조

```
data/processed/session_001/
├── images/
│   ├── img_000000.jpg
│   ├── img_000001.jpg
│   └── ...
├── lidar/
│   ├── lidar_000000.npz
│   ├── lidar_000001.npz
│   └── ...
├── labels.csv
├── metadata.json
└── splits/
    ├── train.txt
    ├── val.txt
    └── test.txt
```

### 10.2 labels.csv 형식

```csv
timestamp,image,lidar,steering,speed
1705312200123456789,img_000000.jpg,lidar_000000.npz,0.05,2.3
1705312200156789012,img_000001.jpg,lidar_000001.npz,0.08,2.4
...
```

---

## 11. 실습

### 실습 1: 이미지 추출

1. 수집한 Bag에서 이미지 추출
2. 추출된 이미지 확인

### 실습 2: 전체 파이프라인

1. 전체 추출 스크립트 실행
2. labels.csv 확인
3. 데이터셋 분할

### 실습 3: 데이터 시각화

```python
# 이미지와 레이블 함께 시각화
import matplotlib.pyplot as plt

df = pd.read_csv('data/processed/session_001/labels.csv')

for i in range(5):
    row = df.iloc[i]
    img = cv2.imread(f"data/processed/session_001/images/{row['image']}")
    img = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
    
    plt.figure(figsize=(10, 4))
    plt.imshow(img)
    plt.title(f"Steering: {row['steering']:.3f}, Speed: {row['speed']:.2f}")
    plt.show()
```

---

## 12. 검증 체크리스트

- [ ] rosbags 라이브러리 사용
- [ ] 이미지 추출 성공
- [ ] LiDAR 추출 성공
- [ ] 시간 동기화 이해
- [ ] labels.csv 생성
- [ ] Train/Val/Test 분할

---

*데이터 파이프라인은 ML 프로젝트의 기반입니다. 재사용 가능한 코드로 구축하세요!*
