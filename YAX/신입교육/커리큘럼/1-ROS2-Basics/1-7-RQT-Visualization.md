# RQT 시각화 도구

> YAX F1TENTH 교육자료 | Phase 1: ROS2 Basics
> 
> ROS2 시스템 모니터링 및 디버깅

---

## 1. 개요

### 학습 목표
1. RQT의 다양한 플러그인을 사용할 수 있다
2. rqt_graph로 시스템 구조를 시각화할 수 있다
3. rqt_plot으로 데이터를 그래프로 확인할 수 있다
4. rqt_console로 로그를 모니터링할 수 있다

### 예상 소요 시간
- 전체: 1시간

### 전제 조건
- ROS2 기본 개념 이해
- 노드, 토픽 개념 숙지

---

## 2. RQT 소개

### 2.1 RQT란?

RQT는 ROS2의 GUI 기반 도구 모음입니다. Qt 프레임워크로 만들어진 플러그인 시스템입니다.

### 2.2 설치 확인

```bash
# RQT 패키지 설치
sudo apt install ros-humble-rqt ros-humble-rqt-common-plugins

# RQT 실행
rqt
```

### 2.3 주요 플러그인

| 플러그인 | 용도 |
|----------|------|
| rqt_graph | 노드/토픽 연결 시각화 |
| rqt_plot | 데이터 실시간 그래프 |
| rqt_console | 로그 메시지 모니터링 |
| rqt_topic | 토픽 발행/구독 |
| rqt_service_caller | 서비스 호출 |
| rqt_reconfigure | 파라미터 동적 변경 |
| rqt_image_view | 이미지 토픽 시각화 |

---

## 3. rqt_graph - 시스템 구조 시각화

### 3.1 실행

```bash
# 방법 1: 직접 실행
ros2 run rqt_graph rqt_graph

# 방법 2: RQT에서 플러그인 선택
rqt
# Plugins > Introspection > Node Graph
```

### 3.2 화면 구성

```
┌─────────────────────────────────────────────────────────────┐
│ [Nodes only ▼] [All ▼] [Refresh] [Save]                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│    ┌──────────┐         ┌──────────┐         ┌──────────┐  │
│    │ /lidar   │──/scan──►│ /gap_   │──/drive─►│ /vesc   │  │
│    │  _driver │         │  follow  │         │ _driver  │  │
│    └──────────┘         └──────────┘         └──────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 3.3 주요 기능

- **Nodes only**: 노드만 표시 (토픽 숨김)
- **Nodes/Topics (all)**: 모든 노드와 토픽 표시
- **Hide**: 특정 노드/토픽 숨기기
- **Refresh**: 그래프 새로고침
- **Save**: 이미지로 저장

### 3.4 활용 팁

```bash
# Turtlesim으로 테스트
ros2 run turtlesim turtlesim_node &
ros2 run turtlesim turtle_teleop_key &
ros2 run rqt_graph rqt_graph
```

---

## 4. rqt_plot - 데이터 시각화

### 4.1 실행

```bash
ros2 run rqt_plot rqt_plot
```

### 4.2 토픽 추가

1. Topic 입력창에 토픽 경로 입력
2. 필드까지 지정 (예: `/turtle1/pose/x`)
3. "+" 버튼 클릭

```
/turtle1/pose/x
/turtle1/pose/y
/turtle1/pose/theta
```

### 4.3 명령줄에서 직접 지정

```bash
ros2 run rqt_plot rqt_plot /turtle1/pose/x /turtle1/pose/y
```

### 4.4 그래프 설정

- **X축 범위**: 시간 범위 조정
- **Y축 범위**: 자동/수동 설정
- **색상**: 각 데이터 시리즈 색상 변경

### 4.5 F1TENTH 활용 예

```bash
# LiDAR 최소 거리 모니터링 (커스텀 노드 필요)
ros2 run rqt_plot rqt_plot /min_distance

# 속도 명령 모니터링
ros2 run rqt_plot rqt_plot /cmd_vel/linear/x /cmd_vel/angular/z

# 오도메트리 위치
ros2 run rqt_plot rqt_plot /odom/pose/pose/position/x /odom/pose/pose/position/y
```

---

## 5. rqt_console - 로그 모니터링

### 5.1 실행

```bash
ros2 run rqt_console rqt_console
```

### 5.2 로그 레벨

| 레벨 | 색상 | 용도 |
|------|------|------|
| DEBUG | 회색 | 상세 디버깅 정보 |
| INFO | 흰색 | 일반 정보 |
| WARN | 노란색 | 경고 |
| ERROR | 빨간색 | 오류 |
| FATAL | 빨간색 (굵게) | 치명적 오류 |

### 5.3 필터링

- **노드별 필터**: 특정 노드의 로그만 표시
- **레벨 필터**: WARN 이상만 표시
- **메시지 필터**: 키워드 검색

### 5.4 로그 레벨 변경

```bash
# 노드의 로그 레벨 동적 변경
ros2 service call /node_name/set_logger_level \
  rcl_interfaces/srv/SetLoggerLevel \
  "{logger_name: 'node_name', level: 10}"  # 10=DEBUG
```

---

## 6. rqt_topic - 토픽 관리

### 6.1 실행

```bash
# RQT에서
# Plugins > Topics > Topic Monitor
```

### 6.2 기능

- 토픽 목록 확인
- 발행 빈도 (Hz) 확인
- 메시지 구조 확인
- 데이터 실시간 확인

---

## 7. rqt_service_caller - 서비스 호출

### 7.1 실행

```bash
# RQT에서
# Plugins > Services > Service Caller
```

### 7.2 사용법

1. 서비스 목록에서 선택
2. 요청 파라미터 입력
3. "Call" 버튼 클릭
4. 응답 확인

### 7.3 예제

```
Service: /spawn
Request:
  x: 5.0
  y: 5.0
  theta: 0.0
  name: "turtle2"
```

---

## 8. rqt_reconfigure - 동적 파라미터

### 8.1 실행

```bash
ros2 run rqt_reconfigure rqt_reconfigure
```

### 8.2 기능

- 실행 중인 노드의 파라미터 수정
- 슬라이더로 값 조정
- 실시간 적용

### 8.3 활용

```
# turtlesim 배경색 변경
/turtlesim:
  - background_r: 0-255
  - background_g: 0-255
  - background_b: 0-255
```

---

## 9. rqt_image_view - 이미지 시각화

### 9.1 실행

```bash
ros2 run rqt_image_view rqt_image_view
```

### 9.2 사용법

1. 드롭다운에서 이미지 토픽 선택
2. 이미지 스트림 확인

### 9.3 F1TENTH 활용

```bash
# RealSense 이미지 확인
# Topic: /camera/color/image_raw
# Topic: /camera/depth/image_rect_raw
```

---

## 10. 통합 대시보드 구성

### 10.1 여러 플러그인 동시 사용

1. RQT 실행: `rqt`
2. Plugins 메뉴에서 원하는 플러그인 추가
3. 창 배치 조정 (드래그)
4. Perspectives > Create Perspective로 저장

### 10.2 F1TENTH 대시보드 예

```
┌─────────────────────────────────────────────────────────────┐
│                    F1TENTH Dashboard                        │
├─────────────────────┬───────────────────────────────────────┤
│                     │                                       │
│   [Node Graph]      │         [rqt_plot]                   │
│                     │   - /cmd_vel/linear/x                 │
│   lidar → gap →     │   - /cmd_vel/angular/z                │
│           vesc      │   - /min_distance                     │
│                     │                                       │
├─────────────────────┼───────────────────────────────────────┤
│                     │                                       │
│  [Console]          │  [Image View]                         │
│  - WARN/ERROR logs  │  - /camera/image                      │
│                     │                                       │
└─────────────────────┴───────────────────────────────────────┘
```

---

## 11. 실습 과제

### 과제 1: Turtlesim 모니터링

1. turtlesim 노드 실행
2. rqt_graph로 구조 확인
3. rqt_plot으로 pose 모니터링
4. turtle 이동 → 그래프 변화 관찰

### 과제 2: 로그 필터링

1. rqt_console 실행
2. 여러 노드 실행
3. 특정 노드의 WARN 이상 메시지만 필터링

### 과제 3: 대시보드 생성

1. RQT에서 3개 이상 플러그인 배치
2. Perspective로 저장
3. 다음 실행 시 복원

---

## 12. 검증 체크리스트

- [ ] rqt_graph로 노드 연결 확인
- [ ] rqt_plot으로 데이터 그래프 표시
- [ ] rqt_console로 로그 필터링
- [ ] rqt_reconfigure로 파라미터 변경
- [ ] 커스텀 대시보드 구성

---

## 13. Phase 1 완료

축하합니다! Phase 1: ROS2 기초를 완료했습니다.

---

*RQT 도구들을 활용하면 복잡한 로봇 시스템도 쉽게 디버깅하고 모니터링할 수 있습니다!*
