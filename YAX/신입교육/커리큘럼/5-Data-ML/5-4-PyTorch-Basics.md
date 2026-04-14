# PyTorch 기초

> YAX F1TENTH 교육자료 | Phase 5: Data & ML
> 
> 딥러닝 프레임워크 PyTorch 입문

---

## 1. 개요

### 학습 목표
1. Tensor 연산의 기본을 이해할 수 있다
2. Dataset과 DataLoader를 구현할 수 있다
3. 신경망 모델을 정의하고 학습할 수 있다

### 예상 소요 시간
- 전체: 4시간

### 전제 조건
- Python 중급
- NumPy 기초
- 미분/경사하강법 개념

---

## 2. PyTorch 소개

### 2.1 왜 PyTorch인가?

| 특징 | 설명 |
|------|------|
| 동적 그래프 | 디버깅 용이 |
| Pythonic | NumPy와 유사한 인터페이스 |
| GPU 가속 | CUDA 지원 |
| 연구 친화적 | 빠른 프로토타이핑 |
| 생태계 | torchvision, torchaudio 등 |

### 2.2 설치

```bash
# CUDA 12.x 버전
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121

# CPU 전용
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cpu

# 확인
python -c "import torch; print(torch.__version__); print(torch.cuda.is_available())"
```

---

## 3. Tensor 기초

### 3.1 텐서 생성

```python
import torch
import numpy as np

# 직접 생성
t1 = torch.tensor([1, 2, 3])
t2 = torch.tensor([[1, 2], [3, 4]], dtype=torch.float32)

# NumPy에서 변환
arr = np.array([1, 2, 3])
t3 = torch.from_numpy(arr)
t4 = torch.tensor(arr)  # 복사본 생성

# 특수 텐서
zeros = torch.zeros(3, 4)
ones = torch.ones(3, 4)
rand = torch.rand(3, 4)        # [0, 1) 균등분포
randn = torch.randn(3, 4)      # 표준정규분포
arange = torch.arange(0, 10, 2)  # [0, 2, 4, 6, 8]

# 기존 텐서와 같은 형태
like_zeros = torch.zeros_like(t2)
like_rand = torch.rand_like(t2)
```

### 3.2 텐서 속성

```python
t = torch.rand(3, 4, 5)

print(t.shape)       # torch.Size([3, 4, 5])
print(t.size())      # torch.Size([3, 4, 5])
print(t.ndim)        # 3
print(t.dtype)       # torch.float32
print(t.device)      # cpu 또는 cuda:0
print(t.numel())     # 60 (원소 개수)
```

### 3.3 GPU 이동

```python
# GPU 사용 가능 여부
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
print(f"Using device: {device}")

# GPU로 이동
t = torch.rand(3, 4)
t_gpu = t.to(device)
t_gpu = t.cuda()  # 직접 지정

# CPU로 이동
t_cpu = t_gpu.cpu()

# NumPy 변환 (CPU에서만 가능)
arr = t_cpu.numpy()
```

### 3.4 기본 연산

```python
a = torch.rand(3, 4)
b = torch.rand(3, 4)

# 산술 연산
c = a + b
c = torch.add(a, b)
c = a - b
c = a * b  # 원소별 곱
c = a / b

# 행렬 곱
m1 = torch.rand(3, 4)
m2 = torch.rand(4, 5)
result = torch.matmul(m1, m2)  # (3, 5)
result = m1 @ m2               # 동일

# 통계
print(a.mean())
print(a.sum())
print(a.max())
print(a.min())
print(a.std())

# 차원별 연산
print(a.mean(dim=0))  # 열 평균, shape: (4,)
print(a.mean(dim=1))  # 행 평균, shape: (3,)
```

### 3.5 형태 변환

```python
t = torch.rand(12)

# Reshape
t2 = t.view(3, 4)
t2 = t.reshape(3, 4)
t2 = t.view(-1, 4)  # 자동 계산

# 차원 추가/제거
t3 = t.unsqueeze(0)  # (1, 12)
t3 = t.unsqueeze(1)  # (12, 1)
t4 = t3.squeeze()    # (12,)

# 전치
m = torch.rand(3, 4)
mt = m.T              # (4, 3)
mt = m.transpose(0, 1)

# 차원 순서 변경
t = torch.rand(2, 3, 4)
t = t.permute(2, 0, 1)  # (4, 2, 3)
```

---

## 4. 자동 미분 (Autograd)

### 4.1 기본 개념

```python
# requires_grad=True: 이 텐서에 대한 기울기 추적
x = torch.tensor([2.0], requires_grad=True)

# 계산
y = x ** 2 + 3 * x + 1  # y = x² + 3x + 1

# 역전파
y.backward()

# 기울기 확인 (dy/dx = 2x + 3 = 7 at x=2)
print(x.grad)  # tensor([7.])
```

### 4.2 신경망에서의 사용

```python
# 가중치
W = torch.randn(3, 4, requires_grad=True)
b = torch.randn(4, requires_grad=True)

# 입력
x = torch.randn(2, 3)

# 순전파
y = x @ W + b

# 손실 계산
loss = y.sum()

# 역전파
loss.backward()

# 기울기 확인
print(W.grad.shape)  # (3, 4)
print(b.grad.shape)  # (4,)
```

### 4.3 기울기 추적 비활성화

```python
# 추론 시 (기울기 불필요)
with torch.no_grad():
    output = model(input)

# 또는
model.eval()  # 드롭아웃, 배치정규화 비활성화
output = model(input)

# detach: 그래프에서 분리
x_detached = x.detach()
```

---

## 5. Dataset과 DataLoader

### 5.1 커스텀 Dataset

```python
from torch.utils.data import Dataset, DataLoader
import pandas as pd
import cv2


class F1TenthDataset(Dataset):
    """F1TENTH 주행 데이터셋."""
    
    def __init__(self, 
                 data_dir: str,
                 split: str = 'train',
                 transform=None):
        """
        Args:
            data_dir: 데이터 디렉토리 경로
            split: 'train', 'val', 'test'
            transform: 전처리 변환
        """
        self.data_dir = data_dir
        self.transform = transform
        
        # 레이블 로드
        labels_path = os.path.join(data_dir, 'labels.csv')
        self.df = pd.read_csv(labels_path)
        
        # Split 파일 로드
        split_path = os.path.join(data_dir, 'splits', f'{split}.txt')
        if os.path.exists(split_path):
            with open(split_path, 'r') as f:
                valid_images = set(f.read().strip().split('\n'))
            self.df = self.df[self.df['image'].isin(valid_images)]
        
        self.df = self.df.reset_index(drop=True)
    
    def __len__(self):
        """데이터셋 크기."""
        return len(self.df)
    
    def __getitem__(self, idx):
        """
        단일 샘플 반환.
        
        Args:
            idx: 인덱스
            
        Returns:
            dict: {'image': tensor, 'steering': tensor, ...}
        """
        row = self.df.iloc[idx]
        
        # 이미지 로드
        image_path = os.path.join(self.data_dir, 'images', row['image'])
        image = cv2.imread(image_path)
        image = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)
        
        # 레이블
        steering = row['steering']
        speed = row['speed']
        
        # 변환 적용
        if self.transform:
            image, steering = self.transform(image, steering)
        else:
            image = torch.tensor(image, dtype=torch.float32).permute(2, 0, 1) / 255.0
            steering = torch.tensor([steering], dtype=torch.float32)
        
        return {
            'image': image,
            'steering': steering,
            'speed': torch.tensor([speed], dtype=torch.float32)
        }
```

### 5.2 DataLoader

```python
# 데이터셋 생성
train_dataset = F1TenthDataset(
    data_dir='data/processed/session_001',
    split='train',
    transform=F1TenthTransform(augment=True)
)

val_dataset = F1TenthDataset(
    data_dir='data/processed/session_001',
    split='val',
    transform=F1TenthTransform(augment=False)
)

# DataLoader 생성
train_loader = DataLoader(
    train_dataset,
    batch_size=32,
    shuffle=True,
    num_workers=4,
    pin_memory=True  # GPU 사용 시 속도 향상
)

val_loader = DataLoader(
    val_dataset,
    batch_size=32,
    shuffle=False,
    num_workers=4,
    pin_memory=True
)

# 사용
for batch in train_loader:
    images = batch['image']        # (32, 3, 66, 200)
    steerings = batch['steering']  # (32, 1)
    
    print(f"Batch shape: {images.shape}")
    break
```

### 5.3 데이터 확인

```python
def visualize_batch(loader, n_samples: int = 4):
    """배치 시각화."""
    import matplotlib.pyplot as plt
    
    batch = next(iter(loader))
    images = batch['image']
    steerings = batch['steering']
    
    fig, axes = plt.subplots(1, n_samples, figsize=(15, 4))
    
    for i in range(n_samples):
        img = images[i].permute(1, 2, 0).numpy()
        
        # 역정규화 (ImageNet 기준)
        mean = np.array([0.485, 0.456, 0.406])
        std = np.array([0.229, 0.224, 0.225])
        img = img * std + mean
        img = np.clip(img, 0, 1)
        
        axes[i].imshow(img)
        axes[i].set_title(f"Steering: {steerings[i].item():.3f}")
        axes[i].axis('off')
    
    plt.tight_layout()
    plt.show()
```

---

## 6. 신경망 정의

### 6.1 nn.Module

```python
import torch.nn as nn
import torch.nn.functional as F


class SimpleCNN(nn.Module):
    """간단한 CNN 모델."""
    
    def __init__(self):
        super().__init__()
        
        # Convolutional layers
        self.conv1 = nn.Conv2d(3, 24, kernel_size=5, stride=2)
        self.conv2 = nn.Conv2d(24, 36, kernel_size=5, stride=2)
        self.conv3 = nn.Conv2d(36, 48, kernel_size=5, stride=2)
        self.conv4 = nn.Conv2d(48, 64, kernel_size=3)
        self.conv5 = nn.Conv2d(64, 64, kernel_size=3)
        
        # Fully connected layers
        self.fc1 = nn.Linear(64 * 1 * 18, 100)
        self.fc2 = nn.Linear(100, 50)
        self.fc3 = nn.Linear(50, 10)
        self.fc4 = nn.Linear(10, 1)
        
        # Dropout
        self.dropout = nn.Dropout(0.5)
    
    def forward(self, x):
        """
        순전파.
        
        Args:
            x: (N, 3, 66, 200) 입력 이미지
            
        Returns:
            (N, 1) 조향각 예측
        """
        # Conv layers with ReLU
        x = F.relu(self.conv1(x))
        x = F.relu(self.conv2(x))
        x = F.relu(self.conv3(x))
        x = F.relu(self.conv4(x))
        x = F.relu(self.conv5(x))
        
        # Flatten
        x = x.view(x.size(0), -1)
        
        # FC layers
        x = F.relu(self.fc1(x))
        x = self.dropout(x)
        x = F.relu(self.fc2(x))
        x = self.dropout(x)
        x = F.relu(self.fc3(x))
        x = self.fc4(x)
        
        return x
```

### 6.2 모델 확인

```python
model = SimpleCNN()

# 모델 구조 출력
print(model)

# 파라미터 수 계산
total_params = sum(p.numel() for p in model.parameters())
trainable_params = sum(p.numel() for p in model.parameters() if p.requires_grad)
print(f"Total parameters: {total_params:,}")
print(f"Trainable parameters: {trainable_params:,}")

# 출력 형태 확인
dummy_input = torch.randn(1, 3, 66, 200)
output = model(dummy_input)
print(f"Output shape: {output.shape}")  # (1, 1)
```

### 6.3 nn.Sequential

```python
# Sequential로 간단하게 정의
model = nn.Sequential(
    nn.Conv2d(3, 24, 5, 2),
    nn.ReLU(),
    nn.Conv2d(24, 36, 5, 2),
    nn.ReLU(),
    nn.Conv2d(36, 48, 5, 2),
    nn.ReLU(),
    nn.Flatten(),
    nn.Linear(48 * 5 * 22, 100),
    nn.ReLU(),
    nn.Dropout(0.5),
    nn.Linear(100, 1)
)
```

---

## 7. 학습 루프

### 7.1 기본 학습

```python
def train_one_epoch(model, loader, optimizer, criterion, device):
    """1 에포크 학습."""
    model.train()
    total_loss = 0
    
    for batch in loader:
        images = batch['image'].to(device)
        targets = batch['steering'].to(device)
        
        # Forward
        outputs = model(images)
        loss = criterion(outputs, targets)
        
        # Backward
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()
        
        total_loss += loss.item()
    
    return total_loss / len(loader)


def validate(model, loader, criterion, device):
    """검증."""
    model.eval()
    total_loss = 0
    
    with torch.no_grad():
        for batch in loader:
            images = batch['image'].to(device)
            targets = batch['steering'].to(device)
            
            outputs = model(images)
            loss = criterion(outputs, targets)
            
            total_loss += loss.item()
    
    return total_loss / len(loader)
```

### 7.2 전체 학습 스크립트

```python
def train(config):
    """학습 메인 함수."""
    device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
    print(f"Using device: {device}")
    
    # 데이터
    train_dataset = F1TenthDataset(
        config['data_dir'], 
        split='train',
        transform=F1TenthTransform(augment=True)
    )
    val_dataset = F1TenthDataset(
        config['data_dir'], 
        split='val',
        transform=F1TenthTransform(augment=False)
    )
    
    train_loader = DataLoader(
        train_dataset, 
        batch_size=config['batch_size'],
        shuffle=True, 
        num_workers=4,
        pin_memory=True
    )
    val_loader = DataLoader(
        val_dataset, 
        batch_size=config['batch_size'],
        shuffle=False, 
        num_workers=4,
        pin_memory=True
    )
    
    # 모델
    model = SimpleCNN().to(device)
    
    # 손실 함수 & 옵티마이저
    criterion = nn.MSELoss()
    optimizer = torch.optim.Adam(model.parameters(), lr=config['lr'])
    scheduler = torch.optim.lr_scheduler.ReduceLROnPlateau(
        optimizer, mode='min', patience=3, factor=0.5
    )
    
    # 학습 루프
    best_val_loss = float('inf')
    
    for epoch in range(config['epochs']):
        train_loss = train_one_epoch(model, train_loader, optimizer, criterion, device)
        val_loss = validate(model, val_loader, criterion, device)
        
        scheduler.step(val_loss)
        
        print(f"Epoch {epoch+1}/{config['epochs']} - "
              f"Train Loss: {train_loss:.4f}, Val Loss: {val_loss:.4f}")
        
        # 최고 모델 저장
        if val_loss < best_val_loss:
            best_val_loss = val_loss
            torch.save(model.state_dict(), 'best_model.pth')
            print(f"  Saved best model (val_loss: {val_loss:.4f})")
    
    return model


# 설정
config = {
    'data_dir': 'data/processed/session_001',
    'batch_size': 32,
    'lr': 1e-4,
    'epochs': 50
}

# 학습 실행
model = train(config)
```

---

## 8. 모델 저장/로드

### 8.1 저장

```python
# 가중치만 저장 (권장)
torch.save(model.state_dict(), 'model_weights.pth')

# 전체 모델 저장
torch.save(model, 'model_full.pth')

# 체크포인트 저장 (학습 재개용)
checkpoint = {
    'epoch': epoch,
    'model_state_dict': model.state_dict(),
    'optimizer_state_dict': optimizer.state_dict(),
    'train_loss': train_loss,
    'val_loss': val_loss,
}
torch.save(checkpoint, 'checkpoint.pth')
```

### 8.2 로드

```python
# 가중치 로드
model = SimpleCNN()
model.load_state_dict(torch.load('model_weights.pth'))
model.eval()

# 전체 모델 로드
model = torch.load('model_full.pth')

# 체크포인트 로드 (학습 재개)
checkpoint = torch.load('checkpoint.pth')
model.load_state_dict(checkpoint['model_state_dict'])
optimizer.load_state_dict(checkpoint['optimizer_state_dict'])
start_epoch = checkpoint['epoch']
```

---

## 9. 평가 및 시각화

### 9.1 예측 시각화

```python
def visualize_predictions(model, loader, device, n_samples=5):
    """예측 결과 시각화."""
    model.eval()
    
    batch = next(iter(loader))
    images = batch['image'].to(device)
    targets = batch['steering'].cpu().numpy()
    
    with torch.no_grad():
        predictions = model(images).cpu().numpy()
    
    fig, axes = plt.subplots(1, n_samples, figsize=(15, 4))
    
    for i in range(n_samples):
        img = images[i].cpu().permute(1, 2, 0).numpy()
        
        # 역정규화
        mean = np.array([0.485, 0.456, 0.406])
        std = np.array([0.229, 0.224, 0.225])
        img = img * std + mean
        img = np.clip(img, 0, 1)
        
        axes[i].imshow(img)
        axes[i].set_title(f"True: {targets[i][0]:.3f}\nPred: {predictions[i][0]:.3f}")
        axes[i].axis('off')
    
    plt.tight_layout()
    plt.show()
```

### 9.2 학습 곡선

```python
def plot_learning_curves(train_losses, val_losses):
    """학습 곡선 시각화."""
    plt.figure(figsize=(10, 5))
    plt.plot(train_losses, label='Train Loss')
    plt.plot(val_losses, label='Val Loss')
    plt.xlabel('Epoch')
    plt.ylabel('Loss')
    plt.title('Learning Curves')
    plt.legend()
    plt.grid(True)
    plt.show()
```

---

## 10. 실습

### 실습 1: Tensor 연습

```python
# 3x4 랜덤 텐서 생성
t = torch.rand(3, 4)

# 평균, 표준편차 계산
print(f"Mean: {t.mean()}")
print(f"Std: {t.std()}")

# GPU로 이동 (가능한 경우)
if torch.cuda.is_available():
    t_gpu = t.cuda()
    print(f"Device: {t_gpu.device}")
```

### 실습 2: 커스텀 Dataset

1. F1TenthDataset 클래스 구현
2. DataLoader 생성
3. 배치 시각화

### 실습 3: 모델 학습

1. SimpleCNN 모델 정의
2. 10 에포크 학습
3. 학습 곡선 시각화

---

## 11. 검증 체크리스트

- [ ] Tensor 생성 및 연산
- [ ] GPU 이동 이해
- [ ] autograd 기본 이해
- [ ] Dataset 클래스 구현
- [ ] DataLoader 사용
- [ ] nn.Module 모델 정의
- [ ] 학습 루프 구현
- [ ] 모델 저장/로드

---

*PyTorch는 연구와 프로덕션 모두에서 강력한 도구입니다!*
