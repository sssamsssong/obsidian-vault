# 데이터 전처리

> YAX F1TENTH 교육자료 | Phase 5: Data & ML
> 
> 학습 성능을 높이는 데이터 전처리 기법

---

## 1. 개요

### 학습 목표
1. 이미지 정규화와 크기 조정을 적용할 수 있다
2. 데이터 증강으로 학습 데이터를 확장할 수 있다
3. 레이블 분포 문제를 해결할 수 있다

### 예상 소요 시간
- 전체: 3시간

### 전제 조건
- 데이터 파이프라인 완료
- NumPy, OpenCV 기초

---

## 2. 전처리의 중요성

### 2.1 왜 전처리가 필요한가?

| 문제 | 해결책 |
|------|--------|
| 다양한 이미지 크기 | 리사이즈 |
| 픽셀 값 범위 불일치 | 정규화 |
| 데이터 부족 | 데이터 증강 |
| 레이블 불균형 | 리샘플링 |
| 조명 변화 | 색상 증강 |

### 2.2 전처리 파이프라인

```
원본 이미지 → 크기 조정 → 정규화 → 증강 → 텐서 변환
    ↓
 640x480      200x66     [0,1]    flip,    torch.Tensor
                                  color
```

---

## 3. 이미지 전처리

### 3.1 크기 조정

```python
import cv2
import numpy as np

def resize_image(image: np.ndarray, 
                 target_size: tuple = (200, 66)) -> np.ndarray:
    """
    이미지 크기 조정.
    
    NVIDIA PilotNet 기준: 200x66
    """
    return cv2.resize(image, target_size, interpolation=cv2.INTER_LINEAR)
```

### 3.2 ROI (Region of Interest) 추출

```python
def crop_roi(image: np.ndarray,
             top_crop: float = 0.4,
             bottom_crop: float = 0.1) -> np.ndarray:
    """
    관심 영역만 추출 (하늘, 차량 앞부분 제거).
    
    top_crop: 상단 제거 비율
    bottom_crop: 하단 제거 비율
    """
    h, w = image.shape[:2]
    
    top = int(h * top_crop)
    bottom = int(h * (1 - bottom_crop))
    
    return image[top:bottom, :, :]
```

### 3.3 정규화

```python
def normalize_image(image: np.ndarray) -> np.ndarray:
    """
    이미지 정규화 [0, 255] → [0, 1] 또는 [-1, 1].
    """
    # 방법 1: [0, 1] 스케일링
    normalized = image.astype(np.float32) / 255.0
    
    # 방법 2: [-1, 1] 스케일링 (일부 모델에서 선호)
    # normalized = (image.astype(np.float32) / 127.5) - 1.0
    
    return normalized


def standardize_image(image: np.ndarray,
                      mean: tuple = (0.485, 0.456, 0.406),
                      std: tuple = (0.229, 0.224, 0.225)) -> np.ndarray:
    """
    ImageNet 통계로 표준화 (전이학습 시 사용).
    """
    image = image.astype(np.float32) / 255.0
    
    for c in range(3):
        image[:, :, c] = (image[:, :, c] - mean[c]) / std[c]
    
    return image
```

### 3.4 색상 공간 변환

```python
def convert_colorspace(image: np.ndarray, 
                       colorspace: str = 'YUV') -> np.ndarray:
    """
    색상 공간 변환.
    
    YUV: 밝기와 색상 분리 (PilotNet에서 사용)
    HSV: 색상 기반 필터링에 유용
    """
    if colorspace == 'YUV':
        return cv2.cvtColor(image, cv2.COLOR_BGR2YUV)
    elif colorspace == 'HSV':
        return cv2.cvtColor(image, cv2.COLOR_BGR2HSV)
    elif colorspace == 'GRAY':
        return cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
    else:
        return image
```

---

## 4. 데이터 증강

### 4.1 기하학적 증강

```python
def random_flip(image: np.ndarray, 
                steering: float,
                probability: float = 0.5) -> tuple:
    """
    좌우 반전 (조향각도 반전).
    
    중요: 조향각 부호를 반대로!
    """
    if np.random.random() < probability:
        image = cv2.flip(image, 1)  # 수평 반전
        steering = -steering
    
    return image, steering


def random_translate(image: np.ndarray,
                     steering: float,
                     x_range: float = 50,
                     y_range: float = 10,
                     steering_per_pixel: float = 0.002) -> tuple:
    """
    랜덤 평행이동.
    
    가로 이동 시 조향각 보정.
    """
    h, w = image.shape[:2]
    
    tx = np.random.uniform(-x_range, x_range)
    ty = np.random.uniform(-y_range, y_range)
    
    M = np.float32([[1, 0, tx], [0, 1, ty]])
    image = cv2.warpAffine(image, M, (w, h))
    
    # 조향각 보정
    steering = steering + tx * steering_per_pixel
    
    return image, steering


def random_rotation(image: np.ndarray,
                    steering: float,
                    max_angle: float = 5,
                    steering_per_degree: float = 0.01) -> tuple:
    """
    랜덤 회전.
    """
    h, w = image.shape[:2]
    
    angle = np.random.uniform(-max_angle, max_angle)
    
    M = cv2.getRotationMatrix2D((w/2, h/2), angle, 1.0)
    image = cv2.warpAffine(image, M, (w, h))
    
    # 조향각 보정
    steering = steering + angle * steering_per_degree
    
    return image, steering
```

### 4.2 색상 증강

```python
def random_brightness(image: np.ndarray,
                      factor_range: tuple = (0.5, 1.5)) -> np.ndarray:
    """
    랜덤 밝기 조절.
    """
    factor = np.random.uniform(*factor_range)
    
    hsv = cv2.cvtColor(image, cv2.COLOR_BGR2HSV)
    hsv[:, :, 2] = np.clip(hsv[:, :, 2] * factor, 0, 255)
    
    return cv2.cvtColor(hsv, cv2.COLOR_HSV2BGR)


def random_shadow(image: np.ndarray) -> np.ndarray:
    """
    랜덤 그림자 추가.
    """
    h, w = image.shape[:2]
    
    # 랜덤 사다리꼴 영역
    x1 = np.random.randint(0, w)
    x2 = np.random.randint(0, w)
    
    hsv = cv2.cvtColor(image, cv2.COLOR_BGR2HSV)
    
    # 마스크 생성
    mask = np.zeros((h, w), dtype=np.float32)
    pts = np.array([[x1, 0], [x2, 0], [x2, h], [x1, h]])
    cv2.fillPoly(mask, [pts], 1.0)
    
    # 그림자 적용
    shadow_factor = np.random.uniform(0.3, 0.7)
    hsv[:, :, 2] = hsv[:, :, 2] * (1 - mask * (1 - shadow_factor))
    
    return cv2.cvtColor(hsv.astype(np.uint8), cv2.COLOR_HSV2BGR)


def random_contrast(image: np.ndarray,
                    factor_range: tuple = (0.8, 1.2)) -> np.ndarray:
    """
    랜덤 대비 조절.
    """
    factor = np.random.uniform(*factor_range)
    
    mean = np.mean(image, axis=(0, 1), keepdims=True)
    image = (image - mean) * factor + mean
    
    return np.clip(image, 0, 255).astype(np.uint8)
```

### 4.3 증강 파이프라인

```python
class DataAugmentation:
    def __init__(self, 
                 flip_prob: float = 0.5,
                 translate_prob: float = 0.3,
                 brightness_prob: float = 0.5,
                 shadow_prob: float = 0.3):
        self.flip_prob = flip_prob
        self.translate_prob = translate_prob
        self.brightness_prob = brightness_prob
        self.shadow_prob = shadow_prob
    
    def __call__(self, image: np.ndarray, 
                 steering: float) -> tuple:
        """
        증강 파이프라인 적용.
        """
        # 좌우 반전
        if np.random.random() < self.flip_prob:
            image = cv2.flip(image, 1)
            steering = -steering
        
        # 평행이동
        if np.random.random() < self.translate_prob:
            image, steering = random_translate(image, steering)
        
        # 밝기
        if np.random.random() < self.brightness_prob:
            image = random_brightness(image)
        
        # 그림자
        if np.random.random() < self.shadow_prob:
            image = random_shadow(image)
        
        return image, steering
```

---

## 5. LiDAR 전처리

### 5.1 범위 필터링

```python
def filter_lidar_range(ranges: np.ndarray,
                       min_range: float = 0.1,
                       max_range: float = 10.0) -> np.ndarray:
    """
    유효 범위 외 값 처리.
    """
    ranges = np.where(ranges < min_range, max_range, ranges)
    ranges = np.where(ranges > max_range, max_range, ranges)
    ranges = np.where(np.isinf(ranges), max_range, ranges)
    ranges = np.where(np.isnan(ranges), max_range, ranges)
    
    return ranges
```

### 5.2 정규화

```python
def normalize_lidar(ranges: np.ndarray,
                    max_range: float = 10.0) -> np.ndarray:
    """
    LiDAR 범위 정규화 [0, 1].
    """
    return ranges / max_range
```

### 5.3 다운샘플링

```python
def downsample_lidar(ranges: np.ndarray,
                     target_size: int = 180) -> np.ndarray:
    """
    LiDAR 포인트 수 줄이기.
    
    1081 → 180 (6배 감소)
    """
    indices = np.linspace(0, len(ranges) - 1, target_size).astype(int)
    return ranges[indices]
```

### 5.4 FOV 추출

```python
def extract_fov(ranges: np.ndarray,
                angles: np.ndarray,
                fov_deg: float = 180) -> np.ndarray:
    """
    전방 시야각만 추출.
    """
    fov_rad = np.radians(fov_deg / 2)
    
    mask = (angles >= -fov_rad) & (angles <= fov_rad)
    
    return ranges[mask]
```

---

## 6. 레이블 처리

### 6.1 레이블 분포 분석

```python
import matplotlib.pyplot as plt
import pandas as pd

def analyze_label_distribution(labels_path: str):
    """
    조향각 분포 분석.
    """
    df = pd.read_csv(labels_path)
    
    plt.figure(figsize=(12, 4))
    
    # 히스토그램
    plt.subplot(1, 2, 1)
    plt.hist(df['steering'], bins=50, edgecolor='black')
    plt.xlabel('Steering Angle')
    plt.ylabel('Count')
    plt.title('Steering Distribution')
    
    # 시간에 따른 조향각
    plt.subplot(1, 2, 2)
    plt.plot(df['steering'].values[:1000])
    plt.xlabel('Frame')
    plt.ylabel('Steering')
    plt.title('Steering over Time')
    
    plt.tight_layout()
    plt.show()
    
    # 통계
    print(f"Mean: {df['steering'].mean():.4f}")
    print(f"Std: {df['steering'].std():.4f}")
    print(f"Min: {df['steering'].min():.4f}")
    print(f"Max: {df['steering'].max():.4f}")
```

### 6.2 레이블 불균형 해결

```python
def balance_steering_labels(df: pd.DataFrame,
                            n_bins: int = 23,
                            samples_per_bin: int = 200) -> pd.DataFrame:
    """
    조향각 분포 균형 맞추기.
    
    문제: 직진(조향각 ≈ 0) 데이터가 너무 많음
    해결: 빈도가 높은 구간에서 랜덤 샘플링
    """
    # 조향각 구간 나누기
    bins = np.linspace(df['steering'].min(), 
                       df['steering'].max(), 
                       n_bins + 1)
    
    df['bin'] = pd.cut(df['steering'], bins, labels=False)
    
    balanced_dfs = []
    
    for bin_idx in range(n_bins):
        bin_df = df[df['bin'] == bin_idx]
        
        if len(bin_df) > samples_per_bin:
            # 오버샘플링된 구간: 랜덤 샘플링
            bin_df = bin_df.sample(samples_per_bin)
        
        balanced_dfs.append(bin_df)
    
    result = pd.concat(balanced_dfs, ignore_index=True)
    result = result.drop('bin', axis=1)
    
    return result.sample(frac=1).reset_index(drop=True)  # 셔플
```

### 6.3 레이블 스무딩

```python
def smooth_steering_labels(steering: np.ndarray,
                           window_size: int = 3) -> np.ndarray:
    """
    조향각 스무딩 (노이즈 제거).
    """
    kernel = np.ones(window_size) / window_size
    smoothed = np.convolve(steering, kernel, mode='same')
    return smoothed
```

---

## 7. PyTorch 변환

### 7.1 Transform 클래스

```python
import torch
from torchvision import transforms


class F1TenthTransform:
    """F1TENTH 데이터용 전처리 변환."""
    
    def __init__(self, 
                 target_size: tuple = (200, 66),
                 augment: bool = True):
        self.target_size = target_size
        self.augment = augment
        
        # 증강
        self.augmentation = DataAugmentation() if augment else None
        
        # PyTorch 변환
        self.to_tensor = transforms.ToTensor()
        self.normalize = transforms.Normalize(
            mean=[0.485, 0.456, 0.406],
            std=[0.229, 0.224, 0.225]
        )
    
    def __call__(self, image: np.ndarray, 
                 steering: float) -> tuple:
        """
        전처리 적용.
        
        Args:
            image: BGR 이미지 (H, W, 3)
            steering: 조향각
            
        Returns:
            image_tensor: (3, H, W) 텐서
            steering_tensor: (1,) 텐서
        """
        # BGR → RGB
        image = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)
        
        # 증강 (학습 시에만)
        if self.augment and self.augmentation:
            image, steering = self.augmentation(image, steering)
        
        # ROI 추출
        image = crop_roi(image)
        
        # 크기 조정
        image = cv2.resize(image, self.target_size)
        
        # 텐서 변환 [0, 255] → [0, 1]
        image_tensor = self.to_tensor(image)
        
        # 정규화
        image_tensor = self.normalize(image_tensor)
        
        # 조향각 텐서
        steering_tensor = torch.tensor([steering], dtype=torch.float32)
        
        return image_tensor, steering_tensor
```

### 7.2 LiDAR Transform

```python
class LiDARTransform:
    """LiDAR 데이터용 전처리."""
    
    def __init__(self,
                 target_size: int = 180,
                 max_range: float = 10.0):
        self.target_size = target_size
        self.max_range = max_range
    
    def __call__(self, ranges: np.ndarray) -> torch.Tensor:
        """
        LiDAR 전처리.
        
        Args:
            ranges: (N,) numpy 배열
            
        Returns:
            (target_size,) 텐서
        """
        # 범위 필터링
        ranges = filter_lidar_range(ranges, max_range=self.max_range)
        
        # 다운샘플링
        ranges = downsample_lidar(ranges, self.target_size)
        
        # 정규화
        ranges = normalize_lidar(ranges, self.max_range)
        
        # 텐서 변환
        return torch.tensor(ranges, dtype=torch.float32)
```

---

## 8. 완전한 전처리 파이프라인

```python
class F1TenthPreprocessor:
    """F1TENTH 데이터 전처리 파이프라인."""
    
    def __init__(self,
                 image_size: tuple = (200, 66),
                 lidar_size: int = 180,
                 augment: bool = True):
        
        self.image_transform = F1TenthTransform(
            target_size=image_size,
            augment=augment
        )
        
        self.lidar_transform = LiDARTransform(
            target_size=lidar_size
        )
    
    def process_sample(self,
                       image_path: str,
                       lidar_path: str,
                       steering: float,
                       speed: float) -> dict:
        """
        단일 샘플 전처리.
        """
        # 이미지 로드 및 전처리
        image = cv2.imread(image_path)
        image_tensor, steering_tensor = self.image_transform(image, steering)
        
        # LiDAR 로드 및 전처리
        lidar_data = np.load(lidar_path)
        lidar_tensor = self.lidar_transform(lidar_data['ranges'])
        
        return {
            'image': image_tensor,
            'lidar': lidar_tensor,
            'steering': steering_tensor,
            'speed': torch.tensor([speed], dtype=torch.float32)
        }
```

---

## 9. 실습

### 실습 1: 이미지 전처리 시각화

```python
def visualize_preprocessing(image_path: str):
    """전처리 단계 시각화."""
    image = cv2.imread(image_path)
    image_rgb = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)
    
    fig, axes = plt.subplots(2, 3, figsize=(15, 8))
    
    # 원본
    axes[0, 0].imshow(image_rgb)
    axes[0, 0].set_title('Original')
    
    # ROI
    roi = crop_roi(image_rgb)
    axes[0, 1].imshow(roi)
    axes[0, 1].set_title('ROI Cropped')
    
    # 리사이즈
    resized = cv2.resize(roi, (200, 66))
    axes[0, 2].imshow(resized)
    axes[0, 2].set_title('Resized (200x66)')
    
    # 밝기 증강
    bright = random_brightness(image_rgb)
    axes[1, 0].imshow(bright)
    axes[1, 0].set_title('Brightness Augmented')
    
    # 그림자 증강
    shadow = random_shadow(image_rgb)
    axes[1, 1].imshow(shadow)
    axes[1, 1].set_title('Shadow Augmented')
    
    # 반전
    flipped = cv2.flip(image_rgb, 1)
    axes[1, 2].imshow(flipped)
    axes[1, 2].set_title('Flipped')
    
    plt.tight_layout()
    plt.show()
```

### 실습 2: 레이블 분포 분석

```python
# labels.csv 분석
analyze_label_distribution('data/processed/session_001/labels.csv')

# 균형 맞추기 전/후 비교
df = pd.read_csv('data/processed/session_001/labels.csv')
balanced_df = balance_steering_labels(df)

print(f"Before: {len(df)} samples")
print(f"After: {len(balanced_df)} samples")
```

### 실습 3: 증강 효과 비교

증강 있을 때/없을 때 학습 결과 비교

---

## 10. 검증 체크리스트

- [ ] 이미지 정규화 이해
- [ ] ROI 추출 적용
- [ ] 데이터 증강 구현
- [ ] LiDAR 전처리 적용
- [ ] 레이블 분포 분석
- [ ] PyTorch Transform 작성

---

*좋은 전처리는 학습 성능의 절반을 결정합니다!*
