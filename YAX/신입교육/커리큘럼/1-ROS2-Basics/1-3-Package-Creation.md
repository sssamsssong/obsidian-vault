# ROS2 패키지 생성 가이드

> YAX F1TENTH 교육자료 | Phase 1: ROS2 Basics
> 
> Python 패키지 생성 및 구조 이해

---

## 1. 개요

### 학습 목표
1. ROS2 패키지의 구조를 이해할 수 있다
2. Python 패키지를 생성할 수 있다
3. 패키지를 빌드하고 실행할 수 있다

### 예상 소요 시간
- 전체: 1시간
- 개념 이해: 20분
- 패키지 생성 실습: 40분

### 전제 조건
- ROS2 환경 설정 완료
- CLI 도구 사용법 이해 (02-CLI-Tools.md)

---

## 2. ROS2 패키지란?

### 2.1 패키지 정의
패키지는 ROS2 코드를 구성하는 기본 단위입니다. 관련 있는 노드, 라이브러리, 설정 파일을 하나의 패키지로 묶습니다.

### 2.2 패키지의 역할
- 코드 모듈화 및 재사용
- 의존성 관리
- 빌드 및 배포 단위

### 2.3 빌드 타입

| 타입 | 언어 | 빌드 도구 |
|------|------|-----------|
| `ament_python` | Python | setuptools |
| `ament_cmake` | C++ | CMake |
| `ament_cmake_python` | Python + C++ | CMake |

F1TENTH에서는 주로 **Python** 패키지를 사용합니다.

---

## 3. 워크스페이스 구조

### 3.1 전체 구조

```
~/f1tenth_ws/                    # 워크스페이스 루트
├── src/                         # 소스 폴더
│   ├── my_package/              # 패키지 1
│   │   ├── my_package/          # Python 모듈
│   │   ├── resource/
│   │   ├── test/
│   │   ├── package.xml
│   │   └── setup.py
│   └── another_package/         # 패키지 2
├── build/                       # 빌드 아티팩트 (자동 생성)
├── install/                     # 설치된 패키지 (자동 생성)
└── log/                         # 빌드 로그 (자동 생성)
```

### 3.2 Python 패키지 구조

```
my_package/
├── my_package/                  # Python 모듈 (패키지 이름과 동일)
│   ├── __init__.py             # 패키지 초기화 파일
│   ├── my_node.py              # 노드 스크립트
│   └── utils.py                # 유틸리티 함수
├── resource/
│   └── my_package              # 마커 파일 (빈 파일)
├── test/                        # 테스트 폴더
│   ├── test_copyright.py
│   └── test_flake8.py
├── package.xml                  # 패키지 메타데이터
├── setup.py                     # Python 설치 설정
└── setup.cfg                    # 설치 설정 (선택)
```

---

## 4. 패키지 생성

### 4.1 워크스페이스 준비

```bash
# 워크스페이스로 이동
cd ~/f1tenth_ws/src

# ROS2 환경 소싱
source /opt/ros/humble/setup.bash
```

### 4.2 패키지 생성 명령어

```bash
# Python 패키지 생성
ros2 pkg create --build-type ament_python my_first_pkg

# 의존성과 함께 생성
ros2 pkg create --build-type ament_python my_first_pkg \
  --dependencies rclpy std_msgs

# 노드 이름까지 지정
ros2 pkg create --build-type ament_python my_first_pkg \
  --dependencies rclpy std_msgs \
  --node-name my_node
```

### 4.3 생성된 파일 확인

```bash
cd my_first_pkg
tree .
```

출력:
```
my_first_pkg/
├── my_first_pkg/
│   ├── __init__.py
│   └── my_node.py        # --node-name 옵션 사용 시
├── resource/
│   └── my_first_pkg
├── test/
│   ├── test_copyright.py
│   ├── test_flake8.py
│   └── test_pep257.py
├── package.xml
├── setup.cfg
└── setup.py
```

---

## 5. 패키지 파일 이해

### 5.1 package.xml

패키지의 메타데이터를 정의합니다.

```xml
<?xml version="1.0"?>
<?xml-model href="http://download.ros.org/schema/package_format3.xsd" schematypens="http://www.w3.org/2001/XMLSchema"?>
<package format="3">
  <name>my_first_pkg</name>
  <version>0.0.1</version>
  <description>My first ROS2 package</description>
  
  <maintainer email="your_email@example.com">Your Name</maintainer>
  <license>MIT</license>

  <!-- 빌드 도구 -->
  <buildtool_depend>ament_python</buildtool_depend>

  <!-- 실행 의존성 -->
  <exec_depend>rclpy</exec_depend>
  <exec_depend>std_msgs</exec_depend>
  <exec_depend>sensor_msgs</exec_depend>
  <exec_depend>geometry_msgs</exec_depend>

  <!-- 테스트 의존성 -->
  <test_depend>ament_copyright</test_depend>
  <test_depend>ament_flake8</test_depend>
  <test_depend>ament_pep257</test_depend>
  <test_depend>python3-pytest</test_depend>

  <export>
    <build_type>ament_python</build_type>
  </export>
</package>
```

### 5.2 setup.py

Python 패키지 설치 설정입니다.

```python
from setuptools import find_packages, setup

package_name = 'my_first_pkg'

setup(
    name=package_name,
    version='0.0.1',
    packages=find_packages(exclude=['test']),
    data_files=[
        ('share/ament_index/resource_index/packages',
            ['resource/' + package_name]),
        ('share/' + package_name, ['package.xml']),
        # Launch 파일 포함 (선택)
        # ('share/' + package_name + '/launch', ['launch/my_launch.py']),
        # Config 파일 포함 (선택)
        # ('share/' + package_name + '/config', ['config/params.yaml']),
    ],
    install_requires=['setuptools'],
    zip_safe=True,
    maintainer='Your Name',
    maintainer_email='your_email@example.com',
    description='My first ROS2 package',
    license='MIT',
    tests_require=['pytest'],
    entry_points={
        'console_scripts': [
            # 'executable_name = package_name.module_name:main'
            'my_node = my_first_pkg.my_node:main',
        ],
    },
)
```

### 5.3 setup.cfg

```ini
[develop]
script_dir=$base/lib/my_first_pkg

[install]
install_scripts=$base/lib/my_first_pkg
```

---

## 6. 간단한 노드 작성

### 6.1 노드 코드 작성

`my_first_pkg/my_first_pkg/my_node.py`:

```python
#!/usr/bin/env python3
"""
My First ROS2 Node
"""

import rclpy
from rclpy.node import Node


class MyFirstNode(Node):
    """Simple ROS2 node example."""
    
    def __init__(self):
        super().__init__('my_first_node')
        self.get_logger().info('Hello from my_first_node!')
        
        # 타이머 생성 (1초마다 콜백 실행)
        self.timer = self.create_timer(1.0, self.timer_callback)
        self.count = 0
    
    def timer_callback(self):
        """Timer callback function."""
        self.count += 1
        self.get_logger().info(f'Timer callback: {self.count}')


def main(args=None):
    """Main entry point."""
    rclpy.init(args=args)
    
    node = MyFirstNode()
    
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

### 6.2 setup.py에 진입점 추가

`entry_points`에 노드를 등록합니다:

```python
entry_points={
    'console_scripts': [
        'my_node = my_first_pkg.my_node:main',
    ],
},
```

---

## 7. 빌드 및 실행

### 7.1 빌드

```bash
# 워크스페이스 루트로 이동
cd ~/f1tenth_ws

# 빌드
colcon build

# 특정 패키지만 빌드
colcon build --packages-select my_first_pkg

# 심볼릭 링크 설치 (Python 개발 시 편리)
colcon build --symlink-install
```

### 7.2 환경 소싱

```bash
# 워크스페이스 환경 소싱
source install/setup.bash
```

### 7.3 실행

```bash
# 노드 실행
ros2 run my_first_pkg my_node

# 예상 출력
# [INFO] [my_first_node]: Hello from my_first_node!
# [INFO] [my_first_node]: Timer callback: 1
# [INFO] [my_first_node]: Timer callback: 2
# ...
```

### 7.4 확인

```bash
# 다른 터미널에서
ros2 node list
# /my_first_node
```

---

## 8. 의존성 추가

### 8.1 package.xml에 추가

새로운 의존성이 필요하면 `package.xml`에 추가:

```xml
<exec_depend>sensor_msgs</exec_depend>
<exec_depend>geometry_msgs</exec_depend>
<exec_depend>nav_msgs</exec_depend>
```

### 8.2 rosdep으로 설치

```bash
cd ~/f1tenth_ws
rosdep install --from-paths src --ignore-src -r -y
```

---

## 9. F1TENTH 패키지 예제

### 9.1 gap_follow 패키지 구조 예시

```
gap_follow/
├── gap_follow/
│   ├── __init__.py
│   ├── gap_follow_node.py     # 메인 노드
│   └── utils.py               # 유틸리티 함수
├── launch/
│   └── gap_follow.launch.py   # 런치 파일
├── config/
│   └── params.yaml            # 파라미터 파일
├── resource/
│   └── gap_follow
├── test/
├── package.xml
├── setup.py
└── setup.cfg
```

### 9.2 package.xml

```xml
<package format="3">
  <name>gap_follow</name>
  <version>0.1.0</version>
  <description>Follow the Gap algorithm for F1TENTH</description>
  
  <maintainer email="yax@yonsei.ac.kr">YAX Team</maintainer>
  <license>MIT</license>

  <buildtool_depend>ament_python</buildtool_depend>

  <exec_depend>rclpy</exec_depend>
  <exec_depend>std_msgs</exec_depend>
  <exec_depend>sensor_msgs</exec_depend>
  <exec_depend>ackermann_msgs</exec_depend>
  <exec_depend>nav_msgs</exec_depend>

  <export>
    <build_type>ament_python</build_type>
  </export>
</package>
```

---

## 10. 실습 과제

### 과제: hello_f1tenth 패키지 생성

1. 패키지 생성:
   ```bash
   cd ~/f1tenth_ws/src
   ros2 pkg create --build-type ament_python hello_f1tenth \
     --dependencies rclpy std_msgs
   ```

2. 노드 작성 (`hello_f1tenth/hello_f1tenth/hello_node.py`):
   ```python
   import rclpy
   from rclpy.node import Node
   from std_msgs.msg import String

   class HelloNode(Node):
       def __init__(self):
           super().__init__('hello_node')
           self.publisher = self.create_publisher(String, 'greeting', 10)
           self.timer = self.create_timer(1.0, self.timer_callback)

       def timer_callback(self):
           msg = String()
           msg.data = 'Hello, F1TENTH!'
           self.publisher.publish(msg)
           self.get_logger().info(f'Published: {msg.data}')

   def main(args=None):
       rclpy.init(args=args)
       node = HelloNode()
       rclpy.spin(node)
       node.destroy_node()
       rclpy.shutdown()
   ```

3. setup.py 수정
4. 빌드 및 실행
5. `ros2 topic echo /greeting`으로 확인

---

## 11. 검증 체크리스트

- [ ] Python 패키지 생성 성공
- [ ] package.xml 내용 이해
- [ ] setup.py entry_points 설정
- [ ] `colcon build` 성공
- [ ] `ros2 run` 실행 성공
- [ ] `ros2 node list`에 노드 표시

---

*패키지 구조를 이해하면 체계적으로 ROS2 프로젝트를 관리할 수 있습니다!*
