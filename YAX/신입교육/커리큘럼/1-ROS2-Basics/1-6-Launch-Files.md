# Launch 파일 작성

> YAX F1TENTH 교육자료 | Phase 1: ROS2 Basics
> 
> 여러 노드를 한 번에 실행하기

---

## 1. 개요

### 학습 목표
1. Launch 파일의 개념과 용도를 이해할 수 있다
2. Python Launch 파일을 작성할 수 있다
3. 파라미터와 인자를 전달할 수 있다
4. 조건부 실행과 그룹핑을 할 수 있다

### 예상 소요 시간
- 전체: 1.5시간
- 기본 문법: 30분
- 고급 기능: 30분
- 실습: 30분

### 전제 조건
- ROS2 패키지 구조 이해
- 노드 실행 경험

---

## 2. Launch 파일이란?

### 2.1 용도

Launch 파일은 여러 노드를 한 번에 실행하고 설정하는 스크립트입니다.

**일반 실행**:
```bash
# 터미널 1
ros2 run lidar_driver lidar_node
# 터미널 2
ros2 run camera_driver camera_node
# 터미널 3
ros2 run autonomy gap_follow
# ...
```

**Launch 파일 사용**:
```bash
ros2 launch f1tenth_system f1tenth.launch.py
```

### 2.2 장점
- 여러 노드 동시 실행
- 파라미터 중앙 관리
- 네임스페이스 설정
- 조건부 실행
- 재사용성

---

## 3. 기본 Launch 파일

### 3.1 Python Launch 파일 구조

`launch/my_launch.py`:

```python
from launch import LaunchDescription
from launch_ros.actions import Node


def generate_launch_description():
    """Generate launch description."""
    return LaunchDescription([
        # 노드 1
        Node(
            package='turtlesim',
            executable='turtlesim_node',
            name='turtle1',
            output='screen'
        ),
        # 노드 2
        Node(
            package='turtlesim',
            executable='turtle_teleop_key',
            name='teleop',
            output='screen',
            prefix='xterm -e'  # 새 터미널에서 실행
        ),
    ])
```

### 3.2 패키지에 Launch 파일 추가

**setup.py 수정**:

```python
import os
from glob import glob

setup(
    # ... 기존 설정 ...
    data_files=[
        ('share/ament_index/resource_index/packages',
            ['resource/' + package_name]),
        ('share/' + package_name, ['package.xml']),
        # Launch 파일 추가
        (os.path.join('share', package_name, 'launch'),
            glob('launch/*.py')),
    ],
)
```

### 3.3 실행

```bash
# 빌드
cd ~/f1tenth_ws
colcon build --packages-select my_pkg

# 실행
ros2 launch my_pkg my_launch.py
```

---

## 4. 파라미터 전달

### 4.1 직접 파라미터 설정

```python
Node(
    package='my_pkg',
    executable='my_node',
    name='my_node',
    parameters=[
        {'speed': 2.0},
        {'distance_threshold': 0.5},
        {'topic_name': '/scan'}
    ]
)
```

### 4.2 YAML 파일로 파라미터 전달

`config/params.yaml`:
```yaml
my_node:
  ros__parameters:
    speed: 2.0
    distance_threshold: 0.5
    topic_name: "/scan"
```

Launch 파일:
```python
import os
from ament_index_python.packages import get_package_share_directory

def generate_launch_description():
    config = os.path.join(
        get_package_share_directory('my_pkg'),
        'config',
        'params.yaml'
    )
    
    return LaunchDescription([
        Node(
            package='my_pkg',
            executable='my_node',
            name='my_node',
            parameters=[config]
        )
    ])
```

---

## 5. Launch 인자 (Arguments)

### 5.1 인자 선언 및 사용

```python
from launch import LaunchDescription
from launch.actions import DeclareLaunchArgument
from launch.substitutions import LaunchConfiguration
from launch_ros.actions import Node


def generate_launch_description():
    # 인자 선언
    speed_arg = DeclareLaunchArgument(
        'speed',
        default_value='1.0',
        description='Maximum speed'
    )
    
    use_sim_arg = DeclareLaunchArgument(
        'use_sim',
        default_value='false',
        description='Use simulation'
    )
    
    # 인자 값 가져오기
    speed = LaunchConfiguration('speed')
    use_sim = LaunchConfiguration('use_sim')
    
    return LaunchDescription([
        speed_arg,
        use_sim_arg,
        Node(
            package='my_pkg',
            executable='my_node',
            parameters=[{'speed': speed}]
        )
    ])
```

### 5.2 실행 시 인자 전달

```bash
ros2 launch my_pkg my_launch.py speed:=2.5 use_sim:=true
```

---

## 6. 토픽/서비스 리매핑

### 6.1 리매핑 (Remapping)

```python
Node(
    package='my_pkg',
    executable='my_node',
    name='my_node',
    remappings=[
        ('/input_scan', '/scan'),
        ('/output_cmd', '/drive'),
        ('odom', '/vesc/odom')
    ]
)
```

이렇게 하면:
- 노드가 구독하는 `/input_scan`이 실제로는 `/scan`에 연결
- 노드가 발행하는 `/output_cmd`가 실제로는 `/drive`에 발행

---

## 7. 네임스페이스

### 7.1 노드 네임스페이스

```python
Node(
    package='my_pkg',
    executable='my_node',
    namespace='robot1',  # /robot1/my_node
    name='my_node'
)
```

토픽도 네임스페이스가 적용됨:
- `/my_topic` → `/robot1/my_topic`

### 7.2 그룹으로 네임스페이스 적용

```python
from launch_ros.actions import PushRosNamespace
from launch.actions import GroupAction

def generate_launch_description():
    robot1_group = GroupAction([
        PushRosNamespace('robot1'),
        Node(package='lidar', executable='lidar_node'),
        Node(package='camera', executable='camera_node'),
    ])
    
    robot2_group = GroupAction([
        PushRosNamespace('robot2'),
        Node(package='lidar', executable='lidar_node'),
        Node(package='camera', executable='camera_node'),
    ])
    
    return LaunchDescription([robot1_group, robot2_group])
```

---

## 8. 조건부 실행

### 8.1 IfCondition / UnlessCondition

```python
from launch.conditions import IfCondition, UnlessCondition
from launch.substitutions import LaunchConfiguration

def generate_launch_description():
    use_sim = LaunchConfiguration('use_sim')
    
    return LaunchDescription([
        DeclareLaunchArgument('use_sim', default_value='false'),
        
        # use_sim=true일 때만 실행
        Node(
            package='gazebo',
            executable='gazebo',
            condition=IfCondition(use_sim)
        ),
        
        # use_sim=false일 때만 실행
        Node(
            package='lidar_driver',
            executable='lidar_node',
            condition=UnlessCondition(use_sim)
        ),
    ])
```

---

## 9. 다른 Launch 파일 포함

### 9.1 IncludeLaunchDescription

```python
from launch.actions import IncludeLaunchDescription
from launch.launch_description_sources import PythonLaunchDescriptionSource

def generate_launch_description():
    lidar_launch = IncludeLaunchDescription(
        PythonLaunchDescriptionSource([
            get_package_share_directory('urg_node'),
            '/launch/urg_node_launch.py'
        ]),
        launch_arguments={
            'sensor_interface': 'serial'
        }.items()
    )
    
    return LaunchDescription([lidar_launch])
```

---

## 10. F1TENTH Launch 파일 예제

### 10.1 전체 시스템 Launch

```python
"""
F1TENTH Full System Launch
"""

import os
from ament_index_python.packages import get_package_share_directory
from launch import LaunchDescription
from launch.actions import DeclareLaunchArgument, IncludeLaunchDescription
from launch.conditions import IfCondition
from launch.substitutions import LaunchConfiguration
from launch.launch_description_sources import PythonLaunchDescriptionSource
from launch_ros.actions import Node


def generate_launch_description():
    # Arguments
    use_sim = LaunchConfiguration('use_sim')
    
    declare_use_sim = DeclareLaunchArgument(
        'use_sim',
        default_value='false',
        description='Use simulation instead of real hardware'
    )
    
    # Config files
    pkg_share = get_package_share_directory('f1tenth_system')
    params_file = os.path.join(pkg_share, 'config', 'params.yaml')
    
    # LiDAR driver
    lidar_node = Node(
        package='urg_node',
        executable='urg_node_driver',
        name='lidar',
        parameters=[params_file],
        condition=UnlessCondition(use_sim)
    )
    
    # VESC driver
    vesc_node = Node(
        package='vesc_driver',
        executable='vesc_driver_node',
        name='vesc',
        parameters=[params_file],
        condition=UnlessCondition(use_sim)
    )
    
    # Autonomy node
    gap_follow_node = Node(
        package='gap_follow',
        executable='gap_follow_node',
        name='gap_follow',
        parameters=[params_file],
        remappings=[
            ('/scan', '/lidar/scan'),
            ('/drive', '/vesc/drive')
        ]
    )
    
    return LaunchDescription([
        declare_use_sim,
        lidar_node,
        vesc_node,
        gap_follow_node,
    ])
```

---

## 11. 디버깅 팁

### 11.1 Launch 출력 확인

```bash
# 상세 출력
ros2 launch my_pkg my_launch.py --show-args

# 런치 그래프 출력
ros2 launch my_pkg my_launch.py --print
```

### 11.2 노드 로그 레벨

```python
Node(
    package='my_pkg',
    executable='my_node',
    arguments=['--ros-args', '--log-level', 'debug']
)
```

---

## 12. 실습 과제

### 과제: F1TENTH 기본 Launch 파일

다음을 포함하는 Launch 파일을 작성하세요:

1. `use_sim` 인자 (기본값: false)
2. 시뮬레이션 모드일 때: turtlesim 실행
3. 실제 모드일 때: (placeholder 노드)
4. 공통: 텔레옵 노드

---

## 13. 검증 체크리스트

- [ ] 기본 Launch 파일 작성
- [ ] 여러 노드 동시 실행
- [ ] 파라미터 전달 (직접 / YAML)
- [ ] 인자 사용
- [ ] 리매핑 설정
- [ ] 조건부 실행

---
*Launch 파일로 복잡한 로봇 시스템을 한 번에 시작할 수 있습니다!*
