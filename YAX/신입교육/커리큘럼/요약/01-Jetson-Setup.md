# Jetson Orin Nano 설정 가이드

> YAX F1TENTH 교육자료 | Domain 01: 임베디드 시스템

---

## 1. 개요

### 학습 목표
1. Jetson Orin Nano에 JetPack 6.2.2를 설치할 수 있다
2. SSH를 통해 원격으로 Jetson에 접속할 수 있다
3. VS Code Remote SSH로 개발 환경을 구축할 수 있다

### 예상 소요 시간
- 전체: 2-3시간
- 이미지 플래싱: 30분
- 초기 설정: 30분
- 원격 접속 설정: 1시간

### 필요 장비
- Jetson Orin Nano Developer Kit (8GB)
- microSD 카드 (64GB 이상, UHS-I 권장)
- USB 키보드/마우스
- HDMI 모니터
- 전원 어댑터 (DC 5V 4A)
- 개인 노트북 (Windows/Mac/Linux)
- SD 카드 리더기

---

## 2. Jetson Orin Nano 소개

### 2.1 하드웨어 사양

| 항목 | Jetson Orin Nano 8GB |
|------|---------------------|
| GPU | 1024-core NVIDIA Ampere (32 Tensor Cores) |
| CPU | 6-core Arm Cortex-A78AE (1.5GHz) |
| RAM | 8GB LPDDR5 |
| Storage | microSD / NVMe SSD |
| AI Performance | 40 TOPS |
| Power | 7W - 15W |

### 2.2 F1TENTH에서의 역할
- 센서 데이터 처리 (LiDAR, 카메라)
- 자율주행 알고리즘 실행
- ROS2 노드 실행
- 딥러닝 추론 (CUDA/TensorRT)

### 2.3 Jetson 제품군 비교

| 모델 | AI 성능 | 가격대 | F1TENTH 적합도 |
|------|---------|--------|----------------|
| Jetson Nano | 472 GFLOPS | $ | 입문용 |
| Jetson Orin Nano | 40 TOPS | $$ | **권장** |
| Jetson Orin NX | 100 TOPS | $$$ | 고급 |
| Jetson AGX Orin | 275 TOPS | $$$$ | 오버스펙 |

---

## 3. JetPack 6.2.2 설치

### 3.1 준비물 확인
- [ ] microSD 카드 (64GB+)
- [ ] SD 카드 리더기
- [ ] Balena Etcher 설치 (https://www.balena.io/etcher/)
- [ ] JetPack 이미지 다운로드

### 3.2 이미지 다운로드

1. NVIDIA 개발자 사이트 접속:
   ```
   https://developer.nvidia.com/embedded/jetpack
   ```

2. "Jetson Orin Nano Developer Kit" 선택

3. SD Card Image 다운로드 (약 15GB)
   - 파일명: `jetson-orin-nano-devkit-sd-card-image.zip`

### 3.3 Balena Etcher로 플래싱

1. Balena Etcher 실행

2. "Flash from file" 클릭 → 다운로드한 zip 파일 선택

3. "Select target" 클릭 → SD 카드 선택
   > ⚠️ 주의: 다른 드라이브를 선택하지 않도록 주의!

4. "Flash!" 클릭 → 약 15-20분 소요

5. 완료 후 SD 카드 안전하게 제거

### 3.4 첫 부팅 및 초기 설정

1. **하드웨어 연결**
   ```
   1. SD 카드를 Jetson에 삽입
   2. HDMI 모니터 연결
   3. USB 키보드/마우스 연결
   4. 전원 어댑터 연결 (자동 부팅)
   ```

2. **초기 설정 마법사**
   - 언어: English (권장) 또는 한국어
   - 키보드 레이아웃: Korean (101/104 key)
   - 시간대: Asia/Seoul
   - 사용자 이름: `nvidia` (권장)
   - 비밀번호: 기억하기 쉬운 것으로 설정
   - 파티션: 기본값 사용

3. **설정 완료 후 재부팅**

---

## 4. 시스템 초기 설정

### 4.1 패키지 업데이트
```bash
sudo apt update && sudo apt upgrade -y
```
> 약 10-15분 소요

### 4.2 필수 패키지 설치
```bash
sudo apt install -y \
    build-essential \
    python3-pip \
    python3-dev \
    git \
    curl \
    wget \
    htop \
    nano \
    vim
```

### 4.3 스왑 메모리 설정

Jetson Orin Nano는 8GB RAM이지만, 대규모 빌드 시 부족할 수 있습니다.

```bash
# 현재 스왑 확인
free -h

# 8GB 스왑 파일 생성
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# 재부팅 후에도 유지
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# 확인
free -h
```

### 4.4 전력 모드 설정

```bash
# 현재 전력 모드 확인
sudo nvpmodel -q

# 사용 가능한 모드 목록
# 0: 15W (기본, 권장)
# 1: 7W (저전력)

# 15W 모드로 설정 (최대 성능)
sudo nvpmodel -m 0

# 최대 클럭으로 고정 (성능 테스트 시)
sudo jetson_clocks
```

---

## 5. 네트워크 설정

### 5.1 WiFi 연결

1. 우측 상단 네트워크 아이콘 클릭
2. WiFi 네트워크 선택
3. 비밀번호 입력

또는 터미널에서:
```bash
# 사용 가능한 WiFi 목록
nmcli device wifi list

# WiFi 연결
nmcli device wifi connect "SSID이름" password "비밀번호"
```

### 5.2 IP 주소 확인
```bash
# IP 주소 확인
hostname -I

# 또는 상세 정보
ip addr show wlan0
```

### 5.3 고정 IP 설정 (선택)

안정적인 SSH 접속을 위해 권장:

```bash
# 네트워크 설정 편집
sudo nano /etc/netplan/01-network-manager-all.yaml
```

```yaml
network:
  version: 2
  renderer: NetworkManager
  wifis:
    wlan0:
      dhcp4: no
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
      access-points:
        "SSID이름":
          password: "비밀번호"
```

```bash
# 적용
sudo netplan apply
```

### 5.4 Hostname 변경
```bash
# 현재 hostname 확인
hostname

# hostname 변경
sudo hostnamectl set-hostname yax-jetson-01

# 재부팅 후 적용
sudo reboot
```

---

## 6. SSH 원격 접속

### 6.1 SSH 서버 확인

JetPack에는 SSH가 기본 설치되어 있습니다.

```bash
# SSH 상태 확인
sudo systemctl status ssh

# 실행 중이 아니면 시작
sudo systemctl enable ssh
sudo systemctl start ssh
```

### 6.2 다른 PC에서 SSH 접속

**Windows (PowerShell 또는 CMD):**
```bash
ssh nvidia@192.168.1.100
```

**Mac/Linux:**
```bash
ssh nvidia@192.168.1.100
```

### 6.3 SSH 키 설정 (비밀번호 없이 접속)

**개인 노트북에서:**
```bash
# SSH 키 생성 (이미 있으면 스킵)
ssh-keygen -t rsa -b 4096

# 공개키를 Jetson에 복사
ssh-copy-id nvidia@192.168.1.100

# 이제 비밀번호 없이 접속 가능
ssh nvidia@192.168.1.100
```

### 6.4 SSH 설정 파일 (편의성)

**개인 노트북의 `~/.ssh/config` 파일:**
```
Host jetson
    HostName 192.168.1.100
    User nvidia
    IdentityFile ~/.ssh/id_rsa
```

이제 간단하게 접속:
```bash
ssh jetson
```

---

## 7. VS Code Remote SSH 설정

### 7.1 VS Code 확장 설치

1. VS Code 실행
2. 확장(Extensions) 탭 열기 (Ctrl+Shift+X)
3. "Remote - SSH" 검색
4. "Remote - SSH" (Microsoft) 설치

### 7.2 SSH Config 설정

1. F1 또는 Ctrl+Shift+P 눌러 명령 팔레트 열기
2. "Remote-SSH: Open SSH Configuration File..." 선택
3. `~/.ssh/config` 선택
4. 아래 내용 추가:

```
Host jetson
    HostName 192.168.1.100
    User nvidia
    IdentityFile ~/.ssh/id_rsa
```

### 7.3 원격 연결

1. F1 → "Remote-SSH: Connect to Host..."
2. "jetson" 선택
3. 새 VS Code 창에서 Jetson에 연결됨
4. 폴더 열기: `/home/nvidia/`

### 7.4 추천 확장 프로그램 (Jetson에 설치)

원격 연결 후 Jetson에 설치:
- Python (Microsoft)
- Pylance
- ROS (Microsoft)
- C/C++ (Microsoft)

---

## 8. 시스템 모니터링

### 8.1 jtop 설치 및 사용

`jtop`은 Jetson 전용 모니터링 도구입니다.

```bash
# 설치
sudo pip3 install -U jetson-stats

# 재부팅 필요
sudo reboot

# 실행
jtop
```

**jtop 화면 설명:**
- GPU: GPU 사용률 및 온도
- CPU: 각 코어별 사용률
- MEM: RAM 사용량
- SWAP: 스왑 사용량
- POWER: 전력 소비량
- TEMP: 각 부품 온도

### 8.2 tegrastats 명령어

```bash
# 실시간 상태 모니터링
sudo tegrastats

# 1초 간격으로 출력
sudo tegrastats --interval 1000
```

**출력 예시:**
```
RAM 2840/7620MB (lfb 512x4MB) SWAP 0/4096MB CPU [25%@1420,off,off,20%@1420,22%@1420,18%@1420] GR3D_FREQ 76% PVA0_FREQ 115 VIC_FREQ 115 APE 174 MTS fg 0% bg 0% AO@42.5C GPU@40C PMIC@50C AUX@41C CPU@42.5C thermal@41.7C VDD_IN 5765mW VDD_CPU_GPU_CV 1548mW VDD_SOC 1497mW
```

### 8.3 GPU/CPU 상태 확인

```bash
# GPU 정보
nvidia-smi

# CPU 정보
lscpu

# 온도 확인
cat /sys/devices/virtual/thermal/thermal_zone*/temp
```

---

## 9. CUDA 확인

### 9.1 CUDA 버전 확인

```bash
# CUDA 버전
nvcc --version

# 출력 예시:
# nvcc: NVIDIA (R) Cuda compiler driver
# Cuda compilation tools, release 12.6, V12.6.xx
```

### 9.2 간단한 CUDA 테스트

```bash
# CUDA 샘플 디렉토리로 이동
cd /usr/local/cuda/samples/1_Utilities/deviceQuery

# 빌드 (처음 한 번만)
sudo make

# 실행
./deviceQuery
```

**정상 출력:**
```
CUDA Device Query (Runtime API) version (CUDART static linking)

Detected 1 CUDA Capable device(s)

Device 0: "Orin"
  CUDA Driver Version / Runtime Version          12.6 / 12.6
  CUDA Capability Major/Minor version number:    8.7
  Total amount of global memory:                 7620 MBytes
  ...
  Result = PASS
```

### 9.3 Python에서 CUDA 확인

```python
import torch
print(f"PyTorch version: {torch.__version__}")
print(f"CUDA available: {torch.cuda.is_available()}")
print(f"CUDA version: {torch.version.cuda}")
print(f"Device name: {torch.cuda.get_device_name(0)}")
```

---

## 10. 검증 체크리스트

모든 항목이 완료되었는지 확인하세요:

- [ ] Jetson이 정상 부팅됨
- [ ] WiFi 연결됨
- [ ] SSH 접속 성공 (`ssh nvidia@<IP>`)
- [ ] VS Code Remote SSH 연결 성공
- [ ] `nvidia-smi` 출력 확인
- [ ] `nvcc --version` 출력 확인
- [ ] `jtop` 실행 확인
- [ ] 스왑 메모리 설정 확인 (`free -h`)

---

## 11. 문제 해결 (Troubleshooting)

### 부팅이 안 될 때

1. **전원 확인**: 5V 4A 이상의 어댑터 사용
2. **SD 카드 확인**: 다른 SD 카드로 재시도
3. **이미지 재플래싱**: Balena Etcher로 다시 플래싱

### SSH 연결이 안 될 때

1. **IP 주소 확인**: Jetson에서 `hostname -I` 실행
2. **방화벽 확인**: 
   ```bash
   sudo ufw status
   sudo ufw allow ssh
   ```
3. **SSH 서비스 확인**:
   ```bash
   sudo systemctl restart ssh
   ```

### WiFi가 불안정할 때

1. **안테나 확인**: 외부 안테나 연결 확인
2. **5GHz 사용**: 2.4GHz보다 5GHz가 안정적
3. **드라이버 업데이트**:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

### 화면이 안 나올 때

1. **HDMI 케이블 확인**: 다른 케이블로 테스트
2. **모니터 해상도**: 다른 모니터로 테스트
3. **Headless 모드**: SSH로 접속 후 설정

### 용량 부족 시

```bash
# 불필요한 패키지 제거
sudo apt autoremove -y
sudo apt clean

# 용량 확인
df -h
```

---

## 12. 다음 단계

Jetson 설정이 완료되었습니다! 다음 단계로 진행하세요:

➡️ **Domain 02: ROS2 설치 및 기초**

```bash
# 다음 단계 미리보기
# ROS2 Humble 설치
sudo apt install ros-humble-desktop
```

---

## 참고 자료

- [NVIDIA Jetson Orin Nano 공식 문서](https://developer.nvidia.com/embedded/learn/get-started-jetson-orin-nano-devkit)
- [JetPack SDK Documentation](https://docs.nvidia.com/jetson/jetpack/)
- [Jetson Linux Developer Guide](https://docs.nvidia.com/jetson/archives/r36.2/DeveloperGuide/)
- [jetson-stats (jtop) GitHub](https://github.com/rbonghi/jetson_stats)

---

*YAX F1TENTH 교육자료 | 작성일: 2026-02-10*
