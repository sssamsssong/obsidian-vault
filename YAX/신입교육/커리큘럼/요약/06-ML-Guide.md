# 머신러닝 & E2E 자율주행 가이드

> YAX F1TENTH 교육자료 | Domain 07: 딥러닝

---

## 1. 개요

### 학습 목표
1. E2E 자율주행의 개념을 이해한다
2. 주행 데이터를 수집하고 전처리할 수 있다
3. PyTorch로 E2E 모델을 학습할 수 있다
4. Jetson에서 실시간 추론을 실행할 수 있다

### 예상 소요 시간: 10-12시간

---

## 2. E2E 자율주행이란?

### 전통적 방법 vs E2E

```
[전통적] 센서 → 인식 → 계획 → 제어
[E2E]    센서 → 신경망 → 제어
```

장점: 수작업 알고리즘 불필요
단점: 해석 어려움, 대량 데이터 필요

---

## 3. 데이터 수집

### 3.1 ROS2 Bag 녹화

```bash
# 주행하며 녹화
ros2 bag record /scan /drive -o training_data
```

### 3.2 필요한 데이터

- **입력 (X)**: LiDAR 스캔 (1080개 거리값)
- **출력 (Y)**: 조향각 (steering_angle)

---

## 4. 데이터 전처리

### 4.1 Bag에서 추출

```python
from rosbags.rosbag2 import Reader
import numpy as np

def extract_data(bag_path):
    scans = []
    steerings = []
    
    with Reader(bag_path) as reader:
        for conn, ts, data in reader.messages():
            if conn.topic == '/scan':
                ranges = np.array(data.ranges)
                ranges = np.clip(ranges, 0, 10) / 10.0  # 정규화
                scans.append(ranges)
            elif conn.topic == '/drive':
                steerings.append(data.drive.steering_angle)
    
    return np.array(scans), np.array(steerings)
```

### 4.2 데이터 분할

```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    scans, steerings, test_size=0.2, random_state=42)
```

---

## 5. PyTorch 모델

### 5.1 Dataset 클래스

```python
from torch.utils.data import Dataset, DataLoader

class DrivingDataset(Dataset):
    def __init__(self, scans, steerings):
        self.scans = torch.FloatTensor(scans)
        self.steerings = torch.FloatTensor(steerings)
    
    def __len__(self):
        return len(self.scans)
    
    def __getitem__(self, idx):
        return self.scans[idx], self.steerings[idx]

train_loader = DataLoader(DrivingDataset(X_train, y_train), 
                          batch_size=32, shuffle=True)
```

### 5.2 모델 정의

```python
import torch.nn as nn

class E2ENet(nn.Module):
    def __init__(self):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(1080, 256),
            nn.ReLU(),
            nn.Dropout(0.2),
            nn.Linear(256, 64),
            nn.ReLU(),
            nn.Linear(64, 1),
            nn.Tanh()  # -1 ~ 1 출력
        )
    
    def forward(self, x):
        return self.net(x)
```

### 5.3 학습

```python
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
model = E2ENet().to(device)
criterion = nn.MSELoss()
optimizer = torch.optim.Adam(model.parameters(), lr=0.001)

for epoch in range(100):
    model.train()
    total_loss = 0
    
    for x, y in train_loader:
        x, y = x.to(device), y.to(device)
        
        optimizer.zero_grad()
        pred = model(x).squeeze()
        loss = criterion(pred, y)
        loss.backward()
        optimizer.step()
        
        total_loss += loss.item()
    
    print(f'Epoch {epoch+1}: Loss = {total_loss/len(train_loader):.4f}')

# 저장
torch.save(model.state_dict(), 'e2e_model.pth')
```

---

## 6. Jetson에서 추론

### 6.1 추론 노드

```python
import torch
import numpy as np
from sensor_msgs.msg import LaserScan
from ackermann_msgs.msg import AckermannDriveStamped

class E2EDriver(Node):
    def __init__(self):
        super().__init__('e2e_driver')
        
        # 모델 로드
        self.device = torch.device('cuda')
        self.model = E2ENet().to(self.device)
        self.model.load_state_dict(torch.load('e2e_model.pth'))
        self.model.eval()
        
        self.scan_sub = self.create_subscription(
            LaserScan, '/scan', self.scan_callback, 10)
        self.drive_pub = self.create_publisher(
            AckermannDriveStamped, '/drive', 10)
    
    def scan_callback(self, msg):
        # 전처리
        ranges = np.array(msg.ranges)
        ranges = np.clip(ranges, 0, 10) / 10.0
        
        # 추론
        with torch.no_grad():
            x = torch.FloatTensor(ranges).unsqueeze(0).to(self.device)
            steering = self.model(x).item()
        
        # 발행
        drive_msg = AckermannDriveStamped()
        drive_msg.drive.speed = 1.5
        drive_msg.drive.steering_angle = steering * 0.4  # 역정규화
        self.drive_pub.publish(drive_msg)
```

### 6.2 TensorRT 최적화 (선택)

```python
from torch2trt import torch2trt

model.eval()
x = torch.randn(1, 1080).cuda()
model_trt = torch2trt(model, [x])

# 저장
torch.save(model_trt.state_dict(), 'e2e_model_trt.pth')
```

---

## 7. 개선 방향

1. **더 많은 데이터**: 다양한 상황, 균형 잡힌 조향각 분포
2. **데이터 증강**: 좌우 반전, 노이즈 추가
3. **CNN 아키텍처**: 1D Conv로 시퀀스 특징 추출
4. **카메라 입력**: RGB 이미지 추가
5. **강화학습**: 시뮬레이터에서 학습

---

## 8. 검증 체크리스트

- [ ] 데이터 5000+ 샘플 수집
- [ ] 모델 학습 완료 (val loss < 0.05)
- [ ] Jetson 실시간 추론 동작
- [ ] E2E로 트랙 완주

---

## 9. 다음 단계

➡️ **Domain 08: F1TENTH 대회 준비**

---

*YAX F1TENTH 교육자료 | 작성일: 2026-02-10*
