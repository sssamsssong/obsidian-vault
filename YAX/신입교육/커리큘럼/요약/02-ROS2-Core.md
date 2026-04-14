# ROS2 Humble 핵심 가이드

> YAX F1TENTH 교육자료 | Domain 02: ROS2 기초

---

## 1. 개요

### 학습 목표
1. ROS2의 핵심 개념(Node, Topic, Service, Action)을 이해한다
2. ROS2 CLI 도구를 사용하여 시스템을 디버깅할 수 있다
3. Python으로 Publisher/Subscriber 노드를 작성할 수 있다
4. Launch 파일로 여러 노드를 실행할 수 있다

### 예상 소요 시간
- 전체: 4-6시간
- ROS2 설치: 30분
- 핵심 개념: 1시간
- CLI 도구: 1시간
- 패키지 생성: 1시간
- Pub/Sub 실습: 1시간

### 전제 조건
- Domain 01 (Jetson 설정) 완료
- Python 기본 문법 이해
- 터미널 사용 경험

---

## 2. ROS2란?

### 2.1 ROS2 소개

**ROS (Robot Operating System)**는 로봇 소프트웨어 개발을 위한 오픈소스 프레임워크입니다.

| 특징 | 설명 |
|------|------|
| 통신 미들웨어 | 노드 간 메시지 전달 |
| 하드웨어 추상화 | 센서/액추에이터 통합 인터페이스 |
| 패키지 관리 | 재사용 가능한 코드 모듈 |
| 도구 | 시각화, 디버깅, 시뮬레이션 |

### 2.2 ROS1 vs ROS2

| 항목 | ROS1 | ROS2 |
|------|------|------|
| 통신 | roscore 필요 | DDS 기반 (P2P) |
| 실시간성 | 미지원 | 지원 |
| 보안 | 미지원 | DDS Security |
| 멀티플랫폼 | Linux 중심 | Linux/Windows/macOS |
| Python | Python 2 | Python 3 |

### 2.3 왜 F1TENTH에서 ROS2를 사용하는가?

1. **모듈화**: 센서, 제어, 알고리즘을 독립적으로 개발
2. **재사용성**: 기존 패키지 활용 (LiDAR 드라이버, VESC 드라이버 등)
3. **디버깅**: 토픽 모니터링, 시각화 도구
4. **시뮬레이션**: Gazebo, F1TENTH Gym 연동
5. **커뮤니티**: 풍부한 예제와 문서

---

## 3. ROS2 Humble 설치

### 3.1 로케일 설정

```bash
# UTF-8 로케일 확인
locale

# 필요시 설정
sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8
```

### 3.2 레포지토리 추가

```bash
# Universe 레포지토리 활성화
sudo apt install software-properties-common
sudo add-apt-repository universe

# ROS2 GPG 키 추가
sudo apt update && sudo apt install curl -y
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg

# 레포지토리 추가
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
```

### 3.3 ROS2 Desktop 설치

```bash
sudo apt update
sudo apt install ros-humble-desktop -y
```

> ⏱️ 약 10-15분 소요

### 3.4 환경 설정 (.bashrc)

```bash
# ROS2 환경 자동 로드
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

### 3.5 개발 도구 설치

```bash
sudo apt install -y \
    python3-colcon-common-extensions \
    python3-rosdep \
    python3-argcomplete
    
# rosdep 초기화
sudo rosdep init
rosdep update
```

### 3.6 설치 확인

```bash
# ROS2 버전 확인
ros2 --version

# 환경 변수 확인
printenv | grep -i ROS
```

---

## 4. ROS2 핵심 개념

### 4.1 노드 (Node)

**노드**는 ROS2의 기본 실행 단위입니다. 각 노드는 특정 기능을 수행합니다.

```
예시:
- lidar_node: LiDAR 데이터 발행
- controller_node: 조향/속도 제어
- visualizer_node: 데이터 시각화
```

```bash
# 실행 중인 노드 목록
ros2 node list

# 노드 상세 정보
ros2 node info /node_name
```

### 4.2 토픽 (Topic)

**토픽**은 노드 간 비동기 통신 채널입니다. Publisher-Subscriber 패턴을 사용합니다.

```
[LiDAR Node] --(/scan)--> [Controller Node]
    (Publisher)              (Subscriber)
```

```bash
# 토픽 목록
ros2 topic list

# 토픽 메시지 타입 확인
ros2 topic info /scan

# 토픽 데이터 실시간 확인
ros2 topic echo /scan

# 토픽 발행 빈도 확인
ros2 topic hz /scan
```

### 4.3 메시지 (Message)

**메시지**는 토픽으로 전달되는 데이터 구조입니다.

```bash
# 메시지 타입 목록
ros2 interface list

# 메시지 구조 확인
ros2 interface show sensor_msgs/msg/LaserScan
```

**자주 사용하는 메시지 타입:**

| 메시지 | 용도 |
|--------|------|
| `std_msgs/String` | 문자열 |
| `std_msgs/Float64` | 실수 |
| `geometry_msgs/Twist` | 선속도/각속도 |
| `sensor_msgs/LaserScan` | 2D LiDAR |
| `sensor_msgs/Image` | 카메라 이미지 |
| `nav_msgs/Odometry` | 오도메트리 |

### 4.4 서비스 (Service)

**서비스**는 요청-응답 패턴의 동기 통신입니다.

```
[Client] --(Request)--> [Server]
         <--(Response)--
```

```bash
# 서비스 목록
ros2 service list

# 서비스 호출
ros2 service call /service_name service_type "{param: value}"
```

### 4.5 파라미터 (Parameter)

**파라미터**는 노드의 런타임 설정값입니다.

```bash
# 파라미터 목록
ros2 param list /node_name

# 파라미터 값 확인
ros2 param get /node_name param_name

# 파라미터 값 설정
ros2 param set /node_name param_name value
```

---

## 5. ROS2 CLI 실습 (turtlesim)

### 5.1 turtlesim 실행

```bash
# 터미널 1: turtlesim 노드 실행
ros2 run turtlesim turtlesim_node

# 터미널 2: 키보드 조종 노드 실행
ros2 run turtlesim turtle_teleop_key
```

### 5.2 노드 및 토픽 탐색

```bash
# 실행 중인 노드 확인
ros2 node list
# 출력: /turtlesim, /teleop_turtle

# 토픽 확인
ros2 topic list
# 출력: /turtle1/cmd_vel, /turtle1/pose, ...

# cmd_vel 토픽 데이터 확인
ros2 topic echo /turtle1/cmd_vel
```

### 5.3 토픽 직접 발행

```bash
# 터틀 앞으로 이동
ros2 topic pub /turtle1/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 2.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 0.0}}"

# 원 그리기 (지속적 발행)
ros2 topic pub -r 1 /turtle1/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 2.0}, angular: {z: 1.8}}"
```

### 5.4 rqt_graph로 시각화

```bash
# 노드/토픽 연결 시각화
rqt_graph
```

---

## 6. 워크스페이스 & 패키지 생성

### 6.1 워크스페이스 생성

```bash
# 워크스페이스 디렉토리 생성
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws

# 빌드 (빈 워크스페이스)
colcon build

# 환경 로드
source install/setup.bash

# .bashrc에 추가 (선택)
echo "source ~/ros2_ws/install/setup.bash" >> ~/.bashrc
```

### 6.2 패키지 생성

```bash
cd ~/ros2_ws/src

# Python 패키지 생성
ros2 pkg create --build-type ament_python my_robot_pkg

# 또는 의존성과 함께 생성
ros2 pkg create --build-type ament_python --dependencies rclpy std_msgs sensor_msgs my_robot_pkg
```

### 6.3 패키지 구조

```
my_robot_pkg/
├── my_robot_pkg/           # Python 소스 코드
│   └── __init__.py
├── resource/
│   └── my_robot_pkg
├── test/
├── package.xml             # 패키지 메타데이터
├── setup.cfg
└── setup.py                # Python 빌드 설정
```

### 6.4 빌드

```bash
cd ~/ros2_ws

# 전체 빌드
colcon build

# 특정 패키지만 빌드
colcon build --packages-select my_robot_pkg

# 심볼릭 링크 빌드 (Python 수정 시 재빌드 불필요)
colcon build --symlink-install
```

---

## 7. Publisher/Subscriber 구현

### 7.1 Publisher 노드

**파일: `~/ros2_ws/src/my_robot_pkg/my_robot_pkg/simple_publisher.py`**

```python
import rclpy
from rclpy.node import Node
from std_msgs.msg import String

class SimplePublisher(Node):
    def __init__(self):
        super().__init__('simple_publisher')
        
        # Publisher 생성: 토픽명, 메시지타입, 큐사이즈
        self.publisher = self.create_publisher(String, 'my_topic', 10)
        
        # 타이머: 1초마다 publish
        self.timer = self.create_timer(1.0, self.timer_callback)
        self.count = 0
        
        self.get_logger().info('Publisher 노드 시작!')

    def timer_callback(self):
        msg = String()
        msg.data = f'Hello ROS2! Count: {self.count}'
        self.publisher.publish(msg)
        self.get_logger().info(f'Published: {msg.data}')
        self.count += 1

def main(args=None):
    rclpy.init(args=args)
    node = SimplePublisher()
    
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    finally:
        node.destroy_node()
        rclpy.shutdown()

if __name__ == '__main__':
    main()
```

### 7.2 Subscriber 노드

**파일: `~/ros2_ws/src/my_robot_pkg/my_robot_pkg/simple_subscriber.py`**

```python
import rclpy
from rclpy.node import Node
from std_msgs.msg import String

class SimpleSubscriber(Node):
    def __init__(self):
        super().__init__('simple_subscriber')
        
        # Subscriber 생성
        self.subscription = self.create_subscription(
            String,
            'my_topic',
            self.listener_callback,
            10
        )
        
        self.get_logger().info('Subscriber 노드 시작!')

    def listener_callback(self, msg):
        self.get_logger().info(f'Received: {msg.data}')

def main(args=None):
    rclpy.init(args=args)
    node = SimpleSubscriber()
    
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    finally:
        node.destroy_node()
        rclpy.shutdown()

if __name__ == '__main__':
    main()
```

### 7.3 setup.py 수정

**파일: `~/ros2_ws/src/my_robot_pkg/setup.py`**

```python
from setuptools import setup

package_name = 'my_robot_pkg'

setup(
    name=package_name,
    version='0.0.1',
    packages=[package_name],
    data_files=[
        ('share/ament_index/resource_index/packages',
            ['resource/' + package_name]),
        ('share/' + package_name, ['package.xml']),
    ],
    install_requires=['setuptools'],
    zip_safe=True,
    maintainer='YAX',
    maintainer_email='yax@example.com',
    description='YAX F1TENTH ROS2 Package',
    license='MIT',
    tests_require=['pytest'],
    entry_points={
        'console_scripts': [
            'simple_publisher = my_robot_pkg.simple_publisher:main',
            'simple_subscriber = my_robot_pkg.simple_subscriber:main',
        ],
    },
)
```

### 7.4 빌드 및 실행

```bash
cd ~/ros2_ws
colcon build --packages-select my_robot_pkg
source install/setup.bash

# 터미널 1: Publisher 실행
ros2 run my_robot_pkg simple_publisher

# 터미널 2: Subscriber 실행
ros2 run my_robot_pkg simple_subscriber

# 터미널 3: 토픽 확인
ros2 topic echo /my_topic
```

---

## 8. Launch 파일

### 8.1 Launch 파일 생성

**디렉토리 생성:**
```bash
mkdir -p ~/ros2_ws/src/my_robot_pkg/launch
```

**파일: `~/ros2_ws/src/my_robot_pkg/launch/pubsub_launch.py`**

```python
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    return LaunchDescription([
        Node(
            package='my_robot_pkg',
            executable='simple_publisher',
            name='my_publisher',
            output='screen'
        ),
        Node(
            package='my_robot_pkg',
            executable='simple_subscriber',
            name='my_subscriber',
            output='screen'
        ),
    ])
```

### 8.2 setup.py 수정 (launch 파일 포함)

```python
import os
from glob import glob
from setuptools import setup

package_name = 'my_robot_pkg'

setup(
    # ... 기존 내용 ...
    data_files=[
        ('share/ament_index/resource_index/packages',
            ['resource/' + package_name]),
        ('share/' + package_name, ['package.xml']),
        # Launch 파일 추가
        (os.path.join('share', package_name, 'launch'), 
            glob('launch/*.py')),
    ],
    # ... 기존 내용 ...
)
```

### 8.3 Launch 실행

```bash
cd ~/ros2_ws
colcon build --packages-select my_robot_pkg
source install/setup.bash

# Launch 파일 실행
ros2 launch my_robot_pkg pubsub_launch.py
```

### 8.4 파라미터 전달

```python
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    return LaunchDescription([
        Node(
            package='my_robot_pkg',
            executable='simple_publisher',
            name='my_publisher',
            output='screen',
            parameters=[
                {'publish_rate': 2.0},
                {'message_prefix': 'YAX'}
            ]
        ),
    ])
```

---

## 9. 시각화 도구

### 9.1 RViz2

```bash
# RViz2 실행
rviz2

# 설정 파일과 함께 실행
rviz2 -d my_config.rviz
```

**주요 디스플레이:**
- LaserScan: LiDAR 데이터
- Image: 카메라 이미지
- TF: 좌표 프레임
- Marker: 커스텀 시각화

### 9.2 rqt 도구

```bash
# 전체 rqt 실행
rqt

# 개별 플러그인 실행
rqt_graph          # 노드/토픽 연결 그래프
rqt_console        # 로그 메시지 확인
rqt_plot           # 토픽 데이터 플롯
rqt_image_view     # 이미지 토픽 확인
```

---

## 10. 검증 체크리스트

- [ ] `ros2 --version` 출력 확인
- [ ] `ros2 topic list` 동작 확인
- [ ] turtlesim 실행 및 키보드 조종
- [ ] 커스텀 패키지 생성 및 빌드
- [ ] Publisher/Subscriber 노드 동작
- [ ] Launch 파일로 멀티노드 실행
- [ ] rqt_graph로 시스템 구조 확인

---

## 11. 문제 해결

### 패키지를 찾을 수 없음

```bash
# 환경 다시 로드
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash
```

### 빌드 에러

```bash
# 클린 빌드
rm -rf ~/ros2_ws/build ~/ros2_ws/install ~/ros2_ws/log
colcon build
```

### 토픽이 안 보임

```bash
# ROS_DOMAIN_ID 확인
echo $ROS_DOMAIN_ID

# 같은 네트워크인지 확인
ros2 daemon stop
ros2 daemon start
```

---

## 12. 다음 단계

ROS2 기초가 완료되었습니다! 다음 단계로 진행하세요:

➡️ **Domain 03: LiDAR 센서 통합**

---

## 참고 자료

- [ROS2 Humble 공식 문서](https://docs.ros.org/en/humble/)
- [ROS2 Tutorials](https://docs.ros.org/en/humble/Tutorials.html)
- [rclpy API](https://docs.ros2.org/humble/api/rclpy/)

---

*YAX F1TENTH 교육자료 | 작성일: 2026-02-10*
