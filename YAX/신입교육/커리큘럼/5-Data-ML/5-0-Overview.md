# Phase 5: 데이터 수집 및 머신러닝

> YAX F1TENTH 교육자료 | Phase 5 개요
> 
> End-to-End 자율주행을 위한 데이터 파이프라인과 딥러닝

---

## 1. Phase 5 개요

### 목표

이 Phase에서는 **데이터 기반 자율주행**의 기초를 학습합니다:

1. ROS2 Bag을 사용한 주행 데이터 수집
2. 데이터 전처리 파이프라인 구축
3. PyTorch로 End-to-End 모델 학습
4. Jetson에서 모델 배포 및 추론

### 왜 머신러닝인가?

| 기존 방식 (Rule-based) | ML 방식 (Data-driven) |
|------------------------|----------------------|
| 알고리즘 직접 설계 | 데이터에서 학습 |
| 모든 상황 예측 필요 | 다양한 상황 일반화 |
| 파라미터 수동 튜닝 | 자동 최적화 |
| 복잡한 환경 어려움 | 복잡한 패턴 학습 가능 |

### 학습 경로

```
┌─────────────────────────────────────────────────────────────┐
│  01-ROS2-Bag: 데이터 수집                                    │
│  - rosbag2 기본 사용법                                       │
│  - 센서 데이터 녹화                                          │
│  - 데이터 재생 및 검증                                       │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  02-Data-Pipeline: 데이터 파이프라인                         │
│  - Bag → Dataset 변환                                        │
│  - 데이터 동기화                                             │
│  - 저장 형식 선택                                            │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  03-Data-Preprocessing: 전처리                               │
│  - 이미지 정규화/증강                                        │
│  - LiDAR 데이터 처리                                         │
│  - 레이블 생성                                               │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  04-PyTorch-Basics: PyTorch 기초                             │
│  - Tensor 연산                                               │
│  - Dataset/DataLoader                                        │
│  - 모델 정의 및 학습                                         │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  05-E2E-Model: End-to-End 모델                               │
│  - CNN 기반 조향 예측                                        │
│  - 모델 아키텍처                                             │
│  - 학습 및 평가                                              │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  06-Model-Deployment: 모델 배포                              │
│  - TensorRT 변환                                             │
│  - Jetson 추론                                               │
│  - ROS2 노드 통합                                            │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. 학습 목표

### Phase 5 완료 시 할 수 있는 것

1. **데이터 수집**
   - rosbag2로 주행 데이터 녹화
   - 카메라, LiDAR, 제어 명령 동시 기록
   - 데이터 품질 검증

2. **데이터 처리**
   - Bag 파일에서 이미지/LiDAR 추출
   - 시간 동기화
   - 학습용 데이터셋 구성

3. **모델 학습**
   - PyTorch로 CNN 모델 구현
   - GPU 학습 (PC 또는 서버)
   - 과적합 방지 기법 적용

4. **배포**
   - TensorRT로 모델 최적화
   - Jetson에서 실시간 추론
   - ROS2 노드로 통합

---

## 3. 하드웨어 요구사항

### 데이터 수집용 (F1TENTH 차량)

| 구성 요소 | 사양 | 용도 |
|-----------|------|------|
| Jetson Orin Nano | 8GB | 센서 데이터 수집 |
| Intel RealSense D435 | 640x480 @ 30fps | 이미지 입력 |
| Hokuyo UST-10LX | 1081 points @ 40Hz | LiDAR 입력 |
| SSD | 256GB+ | 데이터 저장 |

### 학습용 (PC/서버)

| 구성 요소 | 최소 사양 | 권장 사양 |
|-----------|-----------|-----------|
| GPU | GTX 1060 6GB | RTX 3080+ |
| RAM | 16GB | 32GB+ |
| Storage | SSD 256GB | NVMe 1TB+ |
| CUDA | 11.8+ | 12.x |

### 추론용 (Jetson)

| 구성 요소 | 사양 |
|-----------|------|
| Jetson Orin Nano | 8GB |
| JetPack | 6.2.2 |
| TensorRT | 10.x |
| CUDA | 12.6 |

---

## 4. 소프트웨어 환경

### Python 패키지

```bash
# 기본 환경
pip install numpy pandas matplotlib

# PyTorch (CUDA 12.x)
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121

# 데이터 처리
pip install opencv-python pillow

# ROS2 관련
pip install rosbags  # Bag 파일 읽기

# 시각화
pip install tensorboard wandb
```

### Jetson 환경

```bash
# JetPack 6.2.2에 포함된 PyTorch 사용
# 또는 NVIDIA 공식 wheel 설치

# TensorRT Python
pip install tensorrt
```

---

## 5. 프로젝트 구조

```
f1tenth_ml/
├── data/
│   ├── raw/                    # 원본 bag 파일
│   │   ├── session_001/
│   │   ├── session_002/
│   │   └── ...
│   ├── processed/              # 처리된 데이터
│   │   ├── images/
│   │   ├── lidar/
│   │   └── labels.csv
│   └── splits/                 # Train/Val/Test 분할
│       ├── train.txt
│       ├── val.txt
│       └── test.txt
│
├── src/
│   ├── data/
│   │   ├── bag_extractor.py    # Bag → 데이터 추출
│   │   ├── dataset.py          # PyTorch Dataset
│   │   └── augmentation.py     # 데이터 증강
│   │
│   ├── models/
│   │   ├── pilotnet.py         # NVIDIA PilotNet
│   │   ├── resnet_steering.py  # ResNet 기반
│   │   └── lidar_net.py        # LiDAR 전용
│   │
│   ├── training/
│   │   ├── train.py            # 학습 스크립트
│   │   ├── evaluate.py         # 평가 스크립트
│   │   └── config.yaml         # 학습 설정
│   │
│   └── inference/
│       ├── trt_converter.py    # TensorRT 변환
│       └── ros_inference.py    # ROS2 추론 노드
│
├── notebooks/
│   ├── 01_data_exploration.ipynb
│   ├── 02_model_training.ipynb
│   └── 03_evaluation.ipynb
│
├── models/                     # 저장된 모델
│   ├── checkpoints/
│   └── exported/
│
├── configs/
│   └── train_config.yaml
│
└── requirements.txt
```

---

## 6. End-to-End 자율주행 개념

### 6.1 전통적 방식 vs E2E

**전통적 파이프라인:**
```
센서 → 인지 → 판단 → 제어
       │      │      │
       │      │      └─ PID, MPC
       │      └─ 경로 계획
       └─ 객체 감지, SLAM
```

**End-to-End:**
```
센서 → [Neural Network] → 제어
       
       하나의 모델이 센서에서 직접 조향 예측
```

### 6.2 E2E의 장단점

| 장점 | 단점 |
|------|------|
| 단순한 파이프라인 | 해석 어려움 (블랙박스) |
| 자동 특징 학습 | 많은 데이터 필요 |
| 복잡한 상황 처리 가능 | 예측 불가능한 실패 |
| 빠른 개발 | 안전성 검증 어려움 |

### 6.3 F1TENTH에서의 E2E

```
┌─────────────┐
│   카메라    │───┐
│  640x480    │   │
└─────────────┘   │     ┌──────────────┐     ┌──────────┐
                  ├────►│  CNN Model   │────►│ steering │
┌─────────────┐   │     │  (PilotNet)  │     │  speed   │
│   LiDAR     │───┘     └──────────────┘     └──────────┘
│  1081 pts   │
└─────────────┘

입력: 이미지 (또는 LiDAR 또는 둘 다)
출력: 조향각 (+ 속도)
```

---

## 7. 학습 일정

### 권장 학습 순서

| 주차       | 내용                         | 예상 시간 |
| -------- | -------------------------- | ----- |
| Week 1   | [[5-1-ROS2-Bag]]           | 3시간   |
| Week 1   | [[5-2-Data-Pipeline]]      | 4시간   |
| Week 2   | [[5-3-Data-Preprocessing]] | 3시간   |
| Week 2   | [[5-4-PyTorch-Basics]]     | 4시간   |
| Week 3   | [[5-5-E2E-Model]]          | 5시간   |
| Week 3-4 | [[5-6-Model-Deployment]]   | 4시간   |

**총 예상 시간: 23시간 (3-4주)**

---

## 8. 사전 지식 점검

### 필수 지식

- [ ] Python 프로그래밍 (중급)
- [ ] NumPy 배열 연산
- [ ] 기본 선형대수 (행렬 곱셈)
- [ ] ROS2 기본 (Phase 1 완료)
- [ ] 센서 데이터 이해 (Phase 2 완료)

### 권장 지식

- [ ] 딥러닝 기초 (CNN 개념)
- [ ] PyTorch 경험
- [ ] Linux 터미널 사용

### 자가 진단

```python
# 이 코드를 이해할 수 있어야 합니다
import numpy as np

# 배열 생성 및 연산
a = np.random.randn(3, 224, 224)  # 3채널 이미지
b = a.transpose(1, 2, 0)          # HWC로 변환
c = (b - b.mean()) / b.std()      # 정규화

# 조건부 인덱싱
mask = c > 0
c[mask] = 1.0
```

---

## 9. 시작하기

### 환경 설정

```bash
# 1. 작업 디렉토리 생성
mkdir -p ~/f1tenth_ml/{data,src,models,notebooks}
cd ~/f1tenth_ml

# 2. 가상환경 생성
python3 -m venv venv
source venv/bin/activate

# 3. 패키지 설치
pip install numpy pandas matplotlib opencv-python
pip install torch torchvision --index-url https://download.pytorch.org/whl/cu121
pip install rosbags tensorboard
```

### 첫 번째 데이터 수집

```bash
# ROS2 환경에서
source /opt/ros/humble/setup.bash

# 모든 토픽 녹화
ros2 bag record -a -o my_first_bag

# 또는 특정 토픽만
ros2 bag record /camera/image_raw /scan /drive -o driving_data
```

---

## 10. 다음 단계

환경 설정이 완료되면 다음 문서로 진행합니다:
- [01-ROS2-Bag.md](5-1-ROS2-Bag.md) - ROS2 Bag을 사용한 데이터 수집
---

*머신러닝은 자율주행의 미래입니다. 한 걸음씩 나아가봅시다!*
