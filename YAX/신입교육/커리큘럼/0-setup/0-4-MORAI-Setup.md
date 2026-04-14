# MORAI 시뮬레이터 설정 가이드

> YAX F1TENTH 교육자료 | Phase 0: Setup
> 
> MORAI 시뮬레이터 설치 및 ROS2 연동

---

## 1. 개요

### 학습 목표
1. MORAI 시뮬레이터의 개념과 용도를 이해할 수 있다
2. Windows PC에 MORAI를 설치하고 라이선스를 활성화할 수 있다
3. MORAI와 ROS2를 연동하여 시뮬레이션 데이터를 수신할 수 있다
4. 기본 시뮬레이션 시나리오를 실행할 수 있다

### 예상 소요 시간
- 전체: 1-1.5시간
- 설치 및 라이선스: 30분
- ROS2 연동 설정: 30분
- 테스트: 15분

### 전제 조건
- Windows 10/11 PC (GPU 권장)
- MORAI 라이선스 (크루에서 제공)
- Jetson에 ROS2 설치 완료
- 동일 네트워크에 Jetson과 PC 연결

---

## 2. MORAI 시뮬레이터 소개

### 2.1 MORAI란?

MORAI는 한국의 모라이(주)에서 개발한 자율주행 시뮬레이터입니다.

**주요 특징**:
- 고품질 3D 그래픽 환경
- 다양한 센서 시뮬레이션 (LiDAR, 카메라, GPS, IMU)
- ROS/ROS2 연동 지원
- 실시간 물리 엔진
- 한국 도로 환경 지원

### 2.2 F1TENTH 프로젝트에서의 역할

```
┌─────────────────────────────────────────────────────────────┐
│  개발 워크플로우                                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  [1단계] MORAI 시뮬레이션                                     │
│      └── 알고리즘 개발 및 테스트                              │
│      └── 안전한 환경에서 무한 반복                            │
│                                                             │
│  [2단계] 실차 테스트                                          │
│      └── 저속에서 검증                                       │
│      └── 점진적 속도 증가                                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 2.3 시스템 요구사항

| 항목 | 최소 사양 | 권장 사양 |
|------|----------|----------|
| OS | Windows 10 64-bit | Windows 11 64-bit |
| CPU | Intel i5 | Intel i7 / AMD Ryzen 7 |
| RAM | 8GB | 16GB+ |
| GPU | GTX 1060 | RTX 3060+ |
| Storage | 20GB SSD | 50GB+ SSD |
| Network | 100Mbps | 1Gbps |

---

## 3. MORAI 설치

### 3.1 다운로드

1. MORAI 공식 사이트 접속: https://www.morai.ai/
2. 계정 생성 또는 로그인
3. 다운로드 페이지에서 최신 버전 다운로드

> **참고**: 정확한 다운로드 절차는 MORAI 공식 문서를 참조하세요.

### 3.2 설치

1. 다운로드한 설치 파일 실행
2. 설치 마법사 지시에 따라 진행
3. 설치 경로: `C:\MORAI\` (기본값 권장)

### 3.3 라이선스 활성화

1. MORAI Launcher 실행
2. 라이선스 키 입력 (크루에서 제공)
3. 활성화 확인

```
라이선스 관련 문의: [크루 리더에게 문의]
```

---

## 4. MORAI 기본 사용법

### 4.1 시뮬레이터 실행

1. MORAI Launcher 실행
2. 맵 선택 (F1TENTH 트랙 또는 테스트 환경)
3. 차량 모델 선택
4. "Start Simulation" 클릭

### 4.2 UI 구성

```
┌─────────────────────────────────────────────────────────────┐
│  [메뉴바]  File | Edit | View | Simulation | Help           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                    [3D 뷰포트]                               │
│                                                             │
│                    시뮬레이션 화면                           │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  [센서 패널]        [차량 상태]        [네트워크 상태]        │
└─────────────────────────────────────────────────────────────┘
```

### 4.3 기본 조작

| 키 | 동작 |
|----|------|
| W/S | 전진/후진 |
| A/D | 좌/우 조향 |
| Space | 브레이크 |
| R | 차량 리셋 |
| F1 | 도움말 |

---

## 5. 네트워크 설정

### 5.1 네트워크 구성

MORAI(Windows)와 Jetson이 통신하려면 같은 네트워크에 있어야 합니다.

```
┌─────────────────────────────────────────────────────────────┐
│                      [WiFi 라우터]                          │
│                           │                                 │
│              ┌────────────┴────────────┐                    │
│              │                         │                    │
│         [Windows PC]              [Jetson]                  │
│         MORAI 실행                ROS2 노드                 │
│         IP: 192.168.1.10         IP: 192.168.1.20          │
└─────────────────────────────────────────────────────────────┘
```

### 5.2 IP 주소 확인

**Windows PC**:
```powershell
ipconfig
# IPv4 Address 확인 (예: 192.168.1.10)
```

**Jetson**:
```bash
ip addr show
# inet 주소 확인 (예: 192.168.1.20)
```

### 5.3 연결 테스트

**Windows에서 Jetson ping**:
```powershell
ping 192.168.1.20
```

**Jetson에서 Windows ping**:
```bash
ping 192.168.1.10
```

### 5.4 방화벽 설정

MORAI와 ROS2 통신을 위해 방화벽에서 포트를 열어야 할 수 있습니다.

**Windows**:
1. Windows Defender 방화벽 > 고급 설정
2. 인바운드 규칙 > 새 규칙
3. 포트 선택 > TCP/UDP > 특정 포트 (예: 7777, 7778)
4. 연결 허용

---

## 6. ROS2 Bridge 설정

### 6.1 MORAI ROS2 Bridge 개념

```
┌──────────────┐     UDP/TCP     ┌──────────────┐
│    MORAI     │ ◄─────────────► │  ROS2 Bridge │
│  Simulator   │                 │    (Jetson)  │
└──────────────┘                 └──────────────┘
       │                                │
       │                                ▼
       │                         ┌──────────────┐
       │                         │  ROS2 Topics │
       │                         │  /scan       │
       │                         │  /image      │
       │                         │  /odom       │
       │                         └──────────────┘
```

### 6.2 MORAI ROS2 패키지 설치

Jetson에서:

```bash
cd ~/f1tenth_ws/src

# MORAI ROS2 패키지 클론 (예시)
# 정확한 레포지토리는 MORAI 공식 문서 참조
git clone https://github.com/MORAI-Autonomous/MORAI-ROS2.git

# 의존성 설치
cd ~/f1tenth_ws
rosdep install --from-paths src --ignore-src -r -y

# 빌드
colcon build
source install/setup.bash
```

> **참고**: MORAI ROS2 패키지의 정확한 설치 방법은 MORAI 공식 문서를 참조하세요.

### 6.3 Bridge 설정 파일

`~/f1tenth_ws/src/morai_config/config/bridge_config.yaml` (예시):

```yaml
# MORAI Bridge Configuration
morai:
  host: "192.168.1.10"  # Windows PC IP
  port: 7777
  
ros2:
  namespace: "morai"
  
sensors:
  lidar:
    enabled: true
    topic: "/scan"
  camera:
    enabled: true
    topic: "/image_raw"
  imu:
    enabled: true
    topic: "/imu"
```

### 6.4 Bridge 실행

```bash
# MORAI Bridge 노드 실행
ros2 launch morai_ros2 bridge.launch.py

# 또는 단일 노드 실행
ros2 run morai_ros2 morai_bridge_node
```

---

## 7. 연동 테스트

### 7.1 MORAI 시뮬레이션 시작

1. Windows에서 MORAI 실행
2. F1TENTH 맵 선택
3. 시뮬레이션 시작
4. 네트워크 연결 상태 확인 (녹색 = 연결됨)

### 7.2 ROS2 토픽 확인

Jetson에서:

```bash
# 토픽 목록 확인
ros2 topic list

# 예상 출력
# /morai/scan
# /morai/image_raw
# /morai/odom
# /morai/cmd_vel
```

### 7.3 센서 데이터 확인

```bash
# LiDAR 데이터 확인
ros2 topic echo /morai/scan

# 이미지 데이터 확인 (메타데이터만)
ros2 topic info /morai/image_raw

# 오도메트리 확인
ros2 topic echo /morai/odom
```

### 7.4 RViz2로 시각화

```bash
# RViz2 실행
rviz2

# Add > By topic > /morai/scan > LaserScan
# Fixed Frame을 "base_link" 또는 "laser"로 설정
```

### 7.5 제어 명령 전송

```bash
# 간단한 제어 명령 (전진)
ros2 topic pub /morai/cmd_vel geometry_msgs/msg/Twist \
  "{linear: {x: 1.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 0.0}}" -r 10
```

---

## 8. 검증 체크리스트

### MORAI 설치
- [ ] MORAI Launcher 실행 성공
- [ ] 라이선스 활성화 완료
- [ ] 시뮬레이션 시작 가능

### 네트워크
- [ ] Windows ↔ Jetson ping 성공
- [ ] 방화벽 설정 완료

### ROS2 연동
- [ ] MORAI ROS2 Bridge 실행 성공
- [ ] `ros2 topic list`에서 MORAI 토픽 확인
- [ ] LiDAR 데이터 수신 확인
- [ ] RViz2에서 센서 데이터 시각화

---

## 9. 문제 해결

### 9.1 MORAI 실행 안됨

**증상**: 실행 시 검은 화면 또는 크래시

**해결책**:
- GPU 드라이버 업데이트
- DirectX 최신 버전 설치
- 관리자 권한으로 실행

### 9.2 네트워크 연결 실패

**증상**: Ping 실패

**해결책**:
- 같은 WiFi/LAN 연결 확인
- 방화벽 비활성화 후 테스트
- IP 주소 확인

### 9.3 ROS2 토픽 안 보임

**증상**: `ros2 topic list`에 MORAI 토픽 없음

**해결책**:
```bash
# DDS 통신 확인
export ROS_DOMAIN_ID=0  # 양쪽에서 같은 ID 사용

# 네트워크 인터페이스 확인
export ROS_LOCALHOST_ONLY=0

# 멀티캐스트 허용
# Windows 방화벽에서 UDP 허용
```

### 9.4 센서 데이터 지연

**증상**: 데이터가 늦게 도착

**해결책**:
- 유선 이더넷 연결 권장
- QoS 설정 조정 (Best Effort)
- 네트워크 대역폭 확인

---

## 10. 유용한 팁

### 10.1 시뮬레이션 녹화

MORAI에서 주행 데이터를 녹화하고 재생할 수 있습니다.

### 10.2 다양한 시나리오

- 다양한 트랙 레이아웃 테스트
- 장애물 추가
- 날씨/조명 조건 변경

### 10.3 멀티 차량

여러 차량을 동시에 시뮬레이션하여 레이싱 시나리오 테스트 가능.


---

*MORAI 시뮬레이터 설정이 완료되었습니다! 이제 안전하게 자율주행 알고리즘을 개발하고 테스트할 수 있습니다.*
