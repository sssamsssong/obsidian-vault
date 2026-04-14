# 서비스와 액션

> YAX F1TENTH 교육자료 | Phase 1: ROS2 Basics
> 
> 요청-응답 통신 패턴

---

## 1. 개요

### 학습 목표
1. Service와 Topic의 차이를 이해할 수 있다
2. Service Server/Client를 작성할 수 있다
3. Action의 개념을 이해하고 사용할 수 있다

### 예상 소요 시간
- 전체: 2시간
- Service: 1시간
- Action: 1시간

### 전제 조건
- Publisher/Subscriber 이해 (04-Publisher-Subscriber.md)

---

## 2. Service vs Topic

### 2.1 비교

| 특성 | Topic | Service |
|------|-------|---------|
| 통신 방식 | 단방향 (발행) | 양방향 (요청-응답) |
| 동기성 | 비동기 | 동기 |
| 연결 | 1:N, N:1 | 1:1 |
| 용도 | 연속 데이터 스트림 | 일회성 요청 |

### 2.2 언제 Service를 사용하는가?

- 일회성 요청-응답이 필요할 때
- 결과가 즉시 필요할 때
- 예: 파라미터 설정, 상태 조회, 시스템 리셋

```
┌────────────┐   Request    ┌────────────┐
│   Client   │ ───────────► │   Server   │
│            │              │            │
│            │ ◄─────────── │            │
└────────────┘   Response   └────────────┘
```

---

## 3. Service Server 작성

### 3.1 기본 구조

```python
#!/usr/bin/env python3
"""
Service Server Node
"""

import rclpy
from rclpy.node import Node
from example_interfaces.srv import AddTwoInts


class AddTwoIntsServer(Node):
    """Service server that adds two integers."""
    
    def __init__(self):
        super().__init__('add_two_ints_server')
        
        # Service Server 생성
        self.srv = self.create_service(
            AddTwoInts,           # 서비스 타입
            'add_two_ints',       # 서비스 이름
            self.add_callback     # 콜백 함수
        )
        
        self.get_logger().info('Service server ready')
    
    def add_callback(self, request, response):
        """Service callback function."""
        response.sum = request.a + request.b
        self.get_logger().info(
            f'Request: {request.a} + {request.b} = {response.sum}'
        )
        return response


def main(args=None):
    rclpy.init(args=args)
    node = AddTwoIntsServer()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()


if __name__ == '__main__':
    main()
```

### 3.2 std_srvs 사용 예제

```python
from std_srvs.srv import Empty, SetBool, Trigger

# Empty 서비스 (인자 없음)
class ResetServer(Node):
    def __init__(self):
        super().__init__('reset_server')
        self.srv = self.create_service(Empty, 'reset', self.reset_callback)
    
    def reset_callback(self, request, response):
        self.get_logger().info('Reset called!')
        # 리셋 로직
        return response

# SetBool 서비스 (bool 설정)
class EnableServer(Node):
    def __init__(self):
        super().__init__('enable_server')
        self.enabled = False
        self.srv = self.create_service(SetBool, 'enable', self.enable_callback)
    
    def enable_callback(self, request, response):
        self.enabled = request.data
        response.success = True
        response.message = f'Enabled: {self.enabled}'
        return response

# Trigger 서비스 (성공/실패 반환)
class TriggerServer(Node):
    def __init__(self):
        super().__init__('trigger_server')
        self.srv = self.create_service(Trigger, 'trigger', self.trigger_callback)
    
    def trigger_callback(self, request, response):
        # 어떤 작업 수행
        response.success = True
        response.message = 'Action completed'
        return response
```

---

## 4. Service Client 작성

### 4.1 동기 호출 (Synchronous)

```python
#!/usr/bin/env python3
"""
Service Client Node - Synchronous
"""

import rclpy
from rclpy.node import Node
from example_interfaces.srv import AddTwoInts


class AddTwoIntsClient(Node):
    """Service client that requests addition."""
    
    def __init__(self):
        super().__init__('add_two_ints_client')
        
        # Client 생성
        self.client = self.create_client(AddTwoInts, 'add_two_ints')
        
        # 서비스 대기
        while not self.client.wait_for_service(timeout_sec=1.0):
            self.get_logger().info('Waiting for service...')
        
        self.get_logger().info('Service available')
    
    def call_service(self, a, b):
        """Call the service synchronously."""
        request = AddTwoInts.Request()
        request.a = a
        request.b = b
        
        # 동기 호출
        future = self.client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        
        return future.result()


def main(args=None):
    rclpy.init(args=args)
    
    client = AddTwoIntsClient()
    
    # 서비스 호출
    result = client.call_service(5, 3)
    print(f'Result: {result.sum}')
    
    client.destroy_node()
    rclpy.shutdown()


if __name__ == '__main__':
    main()
```

### 4.2 비동기 호출 (Asynchronous)

```python
class AsyncClient(Node):
    def __init__(self):
        super().__init__('async_client')
        self.client = self.create_client(AddTwoInts, 'add_two_ints')
    
    def send_request(self, a, b):
        """Send async request."""
        request = AddTwoInts.Request()
        request.a = a
        request.b = b
        
        # 비동기 호출
        future = self.client.call_async(request)
        future.add_done_callback(self.response_callback)
    
    def response_callback(self, future):
        """Handle response."""
        result = future.result()
        self.get_logger().info(f'Result: {result.sum}')
```

---

## 5. 커스텀 서비스 정의

### 5.1 서비스 파일 생성

`srv/CalculateDistance.srv`:

```
# Request
float64 x1
float64 y1
float64 x2
float64 y2
---
# Response
float64 distance
bool success
string message
```

### 5.2 CMakeLists.txt (cmake 패키지의 경우)

```cmake
find_package(rosidl_default_generators REQUIRED)

rosidl_generate_interfaces(${PROJECT_NAME}
  "srv/CalculateDistance.srv"
)
```

### 5.3 package.xml에 추가

```xml
<buildtool_depend>rosidl_default_generators</buildtool_depend>
<exec_depend>rosidl_default_runtime</exec_depend>
<member_of_group>rosidl_interface_packages</member_of_group>
```

---

## 6. Action 개념

### 6.1 Action이란?

Action은 장시간 실행되는 비동기 작업을 위한 통신 패턴입니다.

**구성 요소**:
- **Goal**: 요청 (목표)
- **Feedback**: 진행 상황 (중간 결과)
- **Result**: 최종 결과

```
┌────────────┐   Goal        ┌────────────┐
│   Client   │ ───────────►  │   Server   │
│            │               │            │
│            │ ◄─ Feedback ─ │  (작업중)   │
│            │ ◄─ Feedback ─ │            │
│            │ ◄─ Result ─── │            │
└────────────┘               └────────────┘
```

### 6.2 Service vs Action

| 특성 | Service | Action |
|------|---------|--------|
| 피드백 | 없음 | 있음 (진행 상황) |
| 취소 | 불가 | 가능 |
| 시간 | 짧은 작업 | 긴 작업 |
| 예시 | 파라미터 설정 | 목표점 이동, 경로 계획 |

---

## 7. Action Server/Client 예제

### 7.1 Fibonacci Action 사용

```python
#!/usr/bin/env python3
"""
Action Client Example - Fibonacci
"""

import rclpy
from rclpy.action import ActionClient
from rclpy.node import Node
from example_interfaces.action import Fibonacci


class FibonacciClient(Node):
    def __init__(self):
        super().__init__('fibonacci_client')
        self._client = ActionClient(self, Fibonacci, 'fibonacci')
    
    def send_goal(self, order):
        goal_msg = Fibonacci.Goal()
        goal_msg.order = order
        
        self._client.wait_for_server()
        
        self._send_goal_future = self._client.send_goal_async(
            goal_msg,
            feedback_callback=self.feedback_callback
        )
        self._send_goal_future.add_done_callback(self.goal_response_callback)
    
    def feedback_callback(self, feedback_msg):
        feedback = feedback_msg.feedback
        self.get_logger().info(f'Feedback: {feedback.partial_sequence}')
    
    def goal_response_callback(self, future):
        goal_handle = future.result()
        if not goal_handle.accepted:
            self.get_logger().info('Goal rejected')
            return
        
        self.get_logger().info('Goal accepted')
        
        self._get_result_future = goal_handle.get_result_async()
        self._get_result_future.add_done_callback(self.result_callback)
    
    def result_callback(self, future):
        result = future.result().result
        self.get_logger().info(f'Result: {result.sequence}')


def main(args=None):
    rclpy.init(args=args)
    client = FibonacciClient()
    client.send_goal(10)
    rclpy.spin(client)


if __name__ == '__main__':
    main()
```

### 7.2 Turtlesim RotateAbsolute Action

```python
from turtlesim.action import RotateAbsolute

class TurtleRotateClient(Node):
    def __init__(self):
        super().__init__('turtle_rotate_client')
        self._client = ActionClient(
            self, 
            RotateAbsolute, 
            'turtle1/rotate_absolute'
        )
    
    def rotate(self, theta):
        goal = RotateAbsolute.Goal()
        goal.theta = theta  # radians
        
        self._client.wait_for_server()
        future = self._client.send_goal_async(
            goal,
            feedback_callback=self.feedback_cb
        )
        future.add_done_callback(self.goal_response_cb)
    
    def feedback_cb(self, feedback):
        remaining = feedback.feedback.remaining
        self.get_logger().info(f'Remaining: {remaining:.2f} rad')
```

---

## 8. CLI로 서비스/액션 테스트

### 8.1 서비스 테스트

```bash
# 서비스 목록
ros2 service list

# 서비스 타입
ros2 service type /add_two_ints

# 서비스 호출
ros2 service call /add_two_ints example_interfaces/srv/AddTwoInts "{a: 5, b: 3}"

# turtlesim 예제
ros2 service call /clear std_srvs/srv/Empty
ros2 service call /spawn turtlesim/srv/Spawn "{x: 5.0, y: 5.0, name: 't2'}"
```

### 8.2 액션 테스트

```bash
# 액션 목록
ros2 action list

# 액션 정보
ros2 action info /turtle1/rotate_absolute

# 액션 목표 전송
ros2 action send_goal /turtle1/rotate_absolute \
  turtlesim/action/RotateAbsolute "{theta: 1.57}"

# 피드백과 함께
ros2 action send_goal /turtle1/rotate_absolute \
  turtlesim/action/RotateAbsolute "{theta: 3.14}" --feedback
```

---

## 9. 실습 과제

### 과제 1: 제곱 서비스

**Server**: 숫자를 받아 제곱값 반환
**Client**: 1~10 숫자의 제곱값 요청

### 과제 2: 차량 활성화 서비스

**Server**: SetBool 서비스로 차량 활성화/비활성화
- 활성화 시: Publisher 시작
- 비활성화 시: Publisher 중지

---

## 10. 검증 체크리스트

- [ ] Service Server 작성 및 실행
- [ ] Service Client 작성 및 호출
- [ ] CLI로 서비스 호출 성공
- [ ] Action 개념 이해
- [ ] Action Client로 목표 전송

---

*Service와 Action은 로봇 시스템의 제어와 조율에 필수적입니다!*
