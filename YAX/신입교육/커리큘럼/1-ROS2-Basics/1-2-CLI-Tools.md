# ROS2 CLI 도구 마스터

> YAX F1TENTH 교육자료 | Phase 1: ROS2 Basics
> 
> ros2 명령어 완벽 가이드

---

## 1. 개요

### 학습 목표
1. `ros2` CLI 명령어를 능숙하게 사용할 수 있다
2. 토픽, 노드, 서비스, 파라미터를 CLI로 조작할 수 있다
3. 디버깅을 위한 CLI 도구를 활용할 수 있다

### 예상 소요 시간
- 전체: 1시간
- 기본 명령어: 30분
- 실습: 30분

### 전제 조건
- ROS2 Humble 설치 완료
- ROS2 핵심 개념 이해 (01-Core-Concepts.md)

---

## 2. 환경 준비

### 2.1 ROS2 환경 소싱

```bash
# ROS2 환경 활성화
source /opt/ros/humble/setup.bash

# 확인
echo $ROS_DISTRO  # humble
```

### 2.2 Turtlesim 실행 (실습용)

```bash
# 터미널 1: turtlesim 노드 실행
ros2 run turtlesim turtlesim_node

# 터미널 2: teleop 노드 실행 (선택)
ros2 run turtlesim turtle_teleop_key
```

---

## 3. 노드 (Node) 명령어

### 3.1 노드 목록 확인

```bash
# 실행 중인 모든 노드 목록
ros2 node list

# 예상 출력
# /turtlesim
# /teleop_turtle
```

### 3.2 노드 정보 확인

```bash
# 특정 노드의 상세 정보
ros2 node info /turtlesim

# 출력 내용:
# - Subscribers (구독 중인 토픽)
# - Publishers (발행 중인 토픽)
# - Service Servers (제공하는 서비스)
# - Service Clients (사용하는 서비스)
# - Action Servers
# - Action Clients
```

---

## 4. 토픽 (Topic) 명령어

### 4.1 토픽 목록

```bash
# 모든 토픽 목록
ros2 topic list

# 메시지 타입과 함께
ros2 topic list -t

# 예상 출력
# /turtle1/cmd_vel [geometry_msgs/msg/Twist]
# /turtle1/pose [turtlesim/msg/Pose]
```

### 4.2 토픽 정보

```bash
# 토픽 기본 정보
ros2 topic info /turtle1/pose

# 상세 정보 (QoS 포함)
ros2 topic info /turtle1/pose --verbose
```

### 4.3 토픽 데이터 확인 (Echo)

```bash
# 토픽 데이터 실시간 출력
ros2 topic echo /turtle1/pose

# 한 번만 출력
ros2 topic echo /turtle1/pose --once

# N개만 출력
ros2 topic echo /turtle1/pose -n 5

# 특정 필드만 출력
ros2 topic echo /turtle1/pose --field x
ros2 topic echo /turtle1/pose --field linear_velocity
```

### 4.4 토픽 발행 빈도 확인

```bash
# Hz (초당 메시지 수)
ros2 topic hz /turtle1/pose

# 예상 출력
# average rate: 62.500
# min: 0.016s max: 0.016s std dev: 0.00000s
```

### 4.5 토픽 대역폭 확인

```bash
# 초당 데이터 크기
ros2 topic bw /turtle1/pose
```

### 4.6 토픽에 메시지 발행 (Pub)

```bash
# 기본 발행 (1회)
ros2 topic pub --once /turtle1/cmd_vel geometry_msgs/msg/Twist \
  "{linear: {x: 2.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 1.8}}"

# 지속 발행 (10Hz)
ros2 topic pub -r 10 /turtle1/cmd_vel geometry_msgs/msg/Twist \
  "{linear: {x: 2.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 1.8}}"

# 1Hz로 발행 (기본값)
ros2 topic pub /turtle1/cmd_vel geometry_msgs/msg/Twist \
  "{linear: {x: 1.0}, angular: {z: 0.5}}"
```

### 4.7 메시지 타입 확인

```bash
# 메시지 구조 확인
ros2 interface show geometry_msgs/msg/Twist

# 출력
# Vector3 linear
#   float64 x
#   float64 y
#   float64 z
# Vector3 angular
#   float64 x
#   float64 y
#   float64 z

# 발행용 템플릿 생성
ros2 interface proto geometry_msgs/msg/Twist
```

---

## 5. 서비스 (Service) 명령어

### 5.1 서비스 목록

```bash
# 모든 서비스 목록
ros2 service list

# 타입과 함께
ros2 service list -t
```

### 5.2 서비스 타입 확인

```bash
# 서비스 타입
ros2 service type /clear

# 서비스 인터페이스 확인
ros2 interface show std_srvs/srv/Empty
```

### 5.3 서비스 호출

```bash
# 빈 서비스 호출 (화면 지우기)
ros2 service call /clear std_srvs/srv/Empty

# 인자가 있는 서비스 호출 (거북이 생성)
ros2 service call /spawn turtlesim/srv/Spawn \
  "{x: 5.0, y: 5.0, theta: 0.0, name: 'turtle2'}"

# 펜 색상 변경
ros2 service call /turtle1/set_pen turtlesim/srv/SetPen \
  "{r: 255, g: 0, b: 0, width: 5, off: 0}"
```

---

## 6. 파라미터 (Parameter) 명령어

### 6.1 파라미터 목록

```bash
# 모든 노드의 파라미터
ros2 param list

# 특정 노드의 파라미터
ros2 param list /turtlesim
```

### 6.2 파라미터 값 확인

```bash
# 파라미터 값 가져오기
ros2 param get /turtlesim background_r

# 모든 파라미터 덤프
ros2 param dump /turtlesim
```

### 6.3 파라미터 값 설정

```bash
# 파라미터 설정
ros2 param set /turtlesim background_r 255
ros2 param set /turtlesim background_g 100
ros2 param set /turtlesim background_b 50

# 주의: 일부 파라미터는 동적으로 변경되지 않을 수 있음
```

### 6.4 파라미터 저장/로드

```bash
# 파라미터를 YAML로 저장
ros2 param dump /turtlesim > turtlesim_params.yaml

# 파라미터 로드 (실행 시)
ros2 run turtlesim turtlesim_node --ros-args --params-file turtlesim_params.yaml
```

---

## 7. 액션 (Action) 명령어

### 7.1 액션 목록

```bash
# 모든 액션 목록
ros2 action list

# 타입과 함께
ros2 action list -t
```

### 7.2 액션 정보

```bash
# 액션 정보
ros2 action info /turtle1/rotate_absolute
```

### 7.3 액션 호출

```bash
# 액션 목표 전송
ros2 action send_goal /turtle1/rotate_absolute turtlesim/action/RotateAbsolute \
  "{theta: 1.57}"

# 피드백과 함께
ros2 action send_goal /turtle1/rotate_absolute turtlesim/action/RotateAbsolute \
  "{theta: 1.57}" --feedback
```

---

## 8. 유틸리티 명령어

### 8.1 패키지 명령어

```bash
# 설치된 패키지 목록
ros2 pkg list

# 패키지 경로
ros2 pkg prefix turtlesim

# 패키지 실행 파일
ros2 pkg executables turtlesim
```

### 8.2 인터페이스 명령어

```bash
# 모든 메시지 타입
ros2 interface list -m

# 모든 서비스 타입
ros2 interface list -s

# 모든 액션 타입
ros2 interface list -a

# 특정 패키지의 인터페이스
ros2 interface package sensor_msgs
```

### 8.3 Bag 녹화/재생

```bash
# 토픽 녹화
ros2 bag record /turtle1/pose /turtle1/cmd_vel -o my_bag

# 모든 토픽 녹화
ros2 bag record -a -o all_topics

# 재생
ros2 bag play my_bag

# 정보 확인
ros2 bag info my_bag
```

### 8.4 ROS2 Doctor

```bash
# 시스템 진단
ros2 doctor

# 상세 리포트
ros2 doctor --report
```

---

## 9. 실습 과제

### 실습 1: Turtlesim 조종

1. `turtlesim_node` 실행
2. CLI로 거북이를 원형으로 이동시키기:
   ```bash
   ros2 topic pub /turtle1/cmd_vel geometry_msgs/msg/Twist \
     "{linear: {x: 2.0}, angular: {z: 1.0}}" -r 10
   ```

### 실습 2: 두 번째 거북이 생성

1. `/spawn` 서비스 호출하여 `turtle2` 생성
2. `turtle2`의 토픽 확인
3. `turtle2`를 다른 방향으로 이동

```bash
# 거북이 생성
ros2 service call /spawn turtlesim/srv/Spawn "{x: 2.0, y: 2.0, name: 'turtle2'}"

# turtle2 조종
ros2 topic pub /turtle2/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 1.0}}" --once
```

### 실습 3: 배경색 변경

```bash
# 빨간 배경
ros2 param set /turtlesim background_r 255
ros2 param set /turtlesim background_g 0
ros2 param set /turtlesim background_b 0

# 화면 클리어로 적용
ros2 service call /clear std_srvs/srv/Empty
```

### 실습 4: 토픽 녹화 및 재생

```bash
# 녹화 시작
ros2 bag record /turtle1/pose -o turtle_pose

# 거북이 이동 (다른 터미널)
ros2 topic pub /turtle1/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 1.0}}" -r 10

# 녹화 중지 (Ctrl+C)
# 재생
ros2 bag play turtle_pose
```

---

## 10. 명령어 치트시트

### 자주 사용하는 명령어

| 명령어 | 설명 |
|--------|------|
| `ros2 node list` | 실행 중인 노드 목록 |
| `ros2 topic list` | 토픽 목록 |
| `ros2 topic echo /topic` | 토픽 데이터 확인 |
| `ros2 topic pub /topic type "{}"` | 토픽에 발행 |
| `ros2 topic hz /topic` | 발행 빈도 확인 |
| `ros2 service list` | 서비스 목록 |
| `ros2 service call /srv type "{}"` | 서비스 호출 |
| `ros2 param list` | 파라미터 목록 |
| `ros2 param get /node param` | 파라미터 값 |
| `ros2 param set /node param val` | 파라미터 설정 |
| `ros2 interface show type` | 메시지 구조 |
| `ros2 bag record /topics` | 녹화 |
| `ros2 bag play bag_name` | 재생 |

### Bash 별칭 추천

`~/.bashrc`에 추가:

```bash
alias tl='ros2 topic list'
alias te='ros2 topic echo'
alias nl='ros2 node list'
alias ni='ros2 node info'
alias sl='ros2 service list'
alias pl='ros2 param list'
```

---

## 11. 검증 체크리스트

- [ ] `ros2 topic list` 실행 성공
- [ ] `ros2 topic echo`로 데이터 확인
- [ ] `ros2 topic pub`으로 메시지 발행
- [ ] `ros2 service call`로 서비스 호출
- [ ] `ros2 param get/set`으로 파라미터 조작
- [ ] `ros2 bag record/play`로 녹화/재생

---

*CLI 도구를 마스터하면 ROS2 시스템을 빠르게 디버깅하고 테스트할 수 있습니다!*
