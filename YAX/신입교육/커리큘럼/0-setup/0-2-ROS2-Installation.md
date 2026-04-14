# ROS2 Humble 설치 가이드

> YAX F1TENTH 교육자료 | Phase 0: Setup
> 
> Jetson Orin Nano (JetPack 6.2.2, Ubuntu 22.04) 환경

---

## 1. 개요

### 학습 목표
1. Jetson Orin Nano에 ROS2 Humble을 설치할 수 있다
2. ROS2 개발 도구(colcon, rosdep)를 설치할 수 있다
3. 워크스페이스를 생성하고 빌드할 수 있다
4. 기본 ROS2 명령어를 실행할 수 있다

### 예상 소요 시간
- 전체: 1-1.5시간
- 패키지 다운로드/설치: 30-40분
- 설정 및 테스트: 20-30분

### 전제 조건
- Jetson Orin Nano 초기 설정 완료 (01-Jetson-Setup.md)
- SSH 접속 가능
- 인터넷 연결 필요

---

## 2. ROS2 Humble 설치

### 2.1 로케일 설정

UTF-8 로케일이 설정되어 있는지 확인합니다.

```bash
# 현재 로케일 확인
locale

# UTF-8 로케일 설정
sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

# 확인
locale
```

### 2.2 소스 리포지토리 설정

```bash
# Universe 리포지토리 활성화
sudo apt install software-properties-common
sudo add-apt-repository universe

# ROS2 GPG 키 추가
sudo apt update && sudo apt install curl -y
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg

# ROS2 리포지토리 추가
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
```

### 2.3 ROS2 Humble 설치

```bash
# 패키지 목록 업데이트
sudo apt update

# ROS2 Humble Desktop 설치 (GUI 도구 포함)
sudo apt install ros-humble-desktop -y

# 설치 확인
source /opt/ros/humble/setup.bash
ros2 --version
```

> **예상 출력**: `ros2 0.9.x` (버전은 다를 수 있음)

### 2.4 환경 설정 (.bashrc)

매번 source 명령어를 입력하지 않도록 .bashrc에 추가합니다.

```bash
# .bashrc에 ROS2 환경 추가
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc

# 현재 세션에 적용
source ~/.bashrc

# 확인
echo $ROS_DISTRO
```

> **예상 출력**: `humble`

---

## 3. 개발 도구 설치

### 3.1 colcon 설치

colcon은 ROS2 패키지를 빌드하는 도구입니다.

```bash
# colcon 설치
sudo apt install python3-colcon-common-extensions -y

# colcon 자동완성 활성화
echo "source /usr/share/colcon_argcomplete/hook/colcon-argcomplete.bash" >> ~/.bashrc
source ~/.bashrc
```

### 3.2 rosdep 초기화

rosdep은 ROS2 패키지의 의존성을 자동으로 설치합니다.

```bash
# rosdep 설치
sudo apt install python3-rosdep2 -y

# rosdep 초기화 (처음 한 번만)
sudo rosdep init
rosdep update
```

### 3.3 추가 개발 도구 설치

```bash
# 빌드 도구
sudo apt install python3-pip python3-vcstool -y

# ROS2 추가 패키지
sudo apt install ros-humble-ament-cmake ros-humble-rosidl-default-generators -y

# 디버깅/시각화 도구
sudo apt install ros-humble-rqt ros-humble-rqt-common-plugins -y
```

---

## 4. 워크스페이스 생성

### 4.1 디렉토리 구조 생성

```bash
# F1TENTH 워크스페이스 생성
mkdir -p ~/f1tenth_ws/src
cd ~/f1tenth_ws

# 디렉토리 구조 확인
tree -L 2
```

예상 구조:
```
~/f1tenth_ws/
└── src/           # 소스 코드
```

### 4.2 빌드 테스트

```bash
cd ~/f1tenth_ws

# 빌드 (src가 비어있어도 가능)
colcon build

# 빌드 후 디렉토리 구조
tree -L 1
```

예상 구조:
```
~/f1tenth_ws/
├── build/         # 빌드 아티팩트
├── install/       # 설치된 패키지
├── log/           # 빌드 로그
└── src/           # 소스 코드
```

### 4.3 워크스페이스 환경 설정

```bash
# 워크스페이스 환경을 .bashrc에 추가
echo "source ~/f1tenth_ws/install/setup.bash" >> ~/.bashrc

# 현재 세션에 적용
source ~/.bashrc
```

---

## 5. 설치 확인

### 5.1 ROS2 명령어 테스트

```bash
# ROS2 버전 확인
ros2 --version

# 사용 가능한 명령어 확인
ros2 --help

# 토픽 목록 (현재는 비어있음)
ros2 topic list
```

### 5.2 Talker-Listener 테스트

두 개의 터미널을 사용합니다. SSH 세션을 두 개 열거나 tmux를 사용하세요.

**터미널 1 (Talker)**:
```bash
ros2 run demo_nodes_cpp talker
```

**터미널 2 (Listener)**:
```bash
ros2 run demo_nodes_cpp listener
```

예상 출력:
```
# 터미널 1 (Talker)
[INFO] [talker]: Publishing: 'Hello World: 1'
[INFO] [talker]: Publishing: 'Hello World: 2'
...

# 터미널 2 (Listener)
[INFO] [listener]: I heard: [Hello World: 1]
[INFO] [listener]: I heard: [Hello World: 2]
...
```

`Ctrl+C`로 종료합니다.

### 5.3 Turtlesim 테스트

그래픽 환경에서 테스트합니다. VNC 또는 모니터 연결 필요.

**터미널 1 (Turtlesim 노드)**:
```bash
ros2 run turtlesim turtlesim_node
```

**터미널 2 (Teleop 노드)**:
```bash
ros2 run turtlesim turtle_teleop_key
```

화살표 키로 거북이를 조종할 수 있습니다.

### 5.4 RQT Graph 확인

```bash
# 새 터미널에서
ros2 run rqt_graph rqt_graph
```

노드 간 연결을 시각적으로 확인할 수 있습니다.

---

## 6. 검증 체크리스트

### 필수 항목
- [ ] `ros2 --version` 출력: `ros2 0.9.x`
- [ ] `echo $ROS_DISTRO` 출력: `humble`
- [ ] `ros2 topic list` 실행 성공 (오류 없음)
- [ ] `colcon build` 성공 (~/f1tenth_ws에서)
- [ ] talker-listener 통신 성공

### 선택 항목
- [ ] turtlesim 실행 및 조종 성공
- [ ] rqt_graph 실행 성공

---

## 7. 문제 해결

### 7.1 패키지를 찾을 수 없음

**증상**:
```
Package 'ros-humble-desktop' has no installation candidate
```

**해결책**:
```bash
# 리포지토리 다시 설정
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository universe

# GPG 키 다시 추가
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg

# 다시 시도
sudo apt update
sudo apt install ros-humble-desktop
```

### 7.2 ros2 명령어를 찾을 수 없음

**증상**:
```
Command 'ros2' not found
```

**해결책**:
```bash
# ROS2 환경 소싱
source /opt/ros/humble/setup.bash

# 영구 적용
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

### 7.3 rosdep init 실패

**증상**:
```
ERROR: default sources list file already exists
```

**해결책**:
```bash
# 이미 초기화된 경우 (정상)
# init은 한 번만 실행하면 됨
rosdep update
```

### 7.4 colcon build 오류

**증상**:
```
colcon: command not found
```

**해결책**:
```bash
sudo apt install python3-colcon-common-extensions
source ~/.bashrc
```

### 7.5 DDS 관련 오류

**증상**:
```
RMW implementation 'rmw_fastrtps_cpp' not found
```

**해결책**:
```bash
sudo apt install ros-humble-rmw-fastrtps-cpp
```

---

## 8. 유용한 ROS2 명령어

```bash
# 노드 목록
ros2 node list

# 토픽 목록
ros2 topic list

# 토픽 정보
ros2 topic info /topic_name

# 토픽 데이터 확인
ros2 topic echo /topic_name

# 서비스 목록
ros2 service list

# 파라미터 목록
ros2 param list

# 패키지 목록
ros2 pkg list

# 패키지 실행
ros2 run <package_name> <executable_name>

# 런치 파일 실행
ros2 launch <package_name> <launch_file>
```


---

*ROS2 Humble 설치가 완료되었습니다! 이제 로봇 소프트웨어 개발을 시작할 준비가 되었습니다.*
