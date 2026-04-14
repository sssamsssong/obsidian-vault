# F1TENTH 플랫폼 종합 가이드

> YAX F1TENTH 교육자료 | Domain 08: 대회 준비

---

## 1. F1TENTH이란?

F1TENTH는 1:10 스케일 자율주행 레이싱 플랫폼으로, 전 세계 대학에서 자율주행 연구와 교육에 활용됩니다.

### 공식 사이트
- 홈페이지: https://f1tenth.org/
- 문서: https://f1tenth.readthedocs.io/
- GitHub: https://github.com/f1tenth

---

## 2. 하드웨어 구성

| 컴포넌트 | 사양 |
|----------|------|
| 섀시 | Traxxas Slash 4x4 (1:10) |
| 컴퓨터 | Jetson Orin Nano 8GB |
| LiDAR | Hokuyo UST-10LX |
| 모터 컨트롤러 | VESC 6+ |
| 카메라 | Intel RealSense D435 |
| 배터리 | 3S LiPo |

---

## 3. 소프트웨어 스택

```
f1tenth_system/
├── f1tenth_stack/       # 메인 스택
│   ├── bringup/         # 시작 런치 파일
│   └── config/          # 설정 파일
├── vesc/                # VESC 드라이버
├── urg_node/            # LiDAR 드라이버
└── ackermann_mux/       # 명령 다중화
```

### 빠른 시작

```bash
# 설치
cd ~/ros2_ws/src
git clone https://github.com/f1tenth/f1tenth_system
cd ~/ros2_ws && colcon build

# 실행
ros2 launch f1tenth_stack bringup_launch.py
```

---

## 4. F1TENTH Gym (시뮬레이터)

```bash
# 설치
git clone https://github.com/f1tenth/f1tenth_gym
cd f1tenth_gym && pip install -e .
```

```python
import gymnasium as gym

env = gym.make('f1tenth_gym:f1tenth-v0', map='./maps/berlin')
obs, info = env.reset()

for _ in range(1000):
    action = [[1.0, 0.0]]  # [speed, steering]
    obs, reward, done, truncated, info = env.step(action)
    env.render()
```

---

## 5. 대회 준비

### 5.1 경기 방식

1. **Time Trial**: 단독 주행, 최고 랩타임
2. **Head-to-Head**: 1:1 경주

### 5.2 레이싱 전략

- **Time Trial**: 최적 레이싱 라인, 코너 진입 속도
- **Head-to-Head**: 추월, 방어, 충돌 회피

### 5.3 체크리스트

- [ ] 하드웨어 점검 (배터리, 연결)
- [ ] 소프트웨어 테스트 (킬스위치)
- [ ] 알고리즘 파라미터 확정
- [ ] 예비 부품 준비

---

## 6. 성능 튜닝

### 랩타임 개선

1. **속도 프로파일**: 직선 ↑, 코너 ↓
2. **레이싱 라인**: 코너 안쪽으로
3. **파라미터 튜닝**: lookahead, 속도 한계

### A/B 테스트

```python
# 설정 A vs 설정 B 비교
# 각각 5회 주행 → 평균 랩타임 비교
```

---

## 7. 알고리즘 선택

| 상황 | 추천 알고리즘 |
|------|---------------|
| 안전 중시 | Follow the Gap |
| 속도 중시 | Pure Pursuit + 최적 경로 |
| 최고 성능 | MPC |
| 실험적 | E2E (ML) |

---

## 8. 디버깅 & 모니터링

```bash
# 실시간 토픽 모니터링
ros2 topic echo /scan
ros2 topic echo /odom
ros2 topic echo /drive

# 데이터 녹화
ros2 bag record -a -o debug_data

# 시각화
rviz2
rqt_graph
```

---

## 9. 커뮤니티 리소스

| 리소스 | URL |
|--------|-----|
| 포럼 | https://f1tenth.discourse.group/ |
| GitHub | https://github.com/f1tenth |
| 논문 | https://arxiv.org/abs/2402.18558 |
| 트랙 | https://github.com/f1tenth/f1tenth_racetracks |

---

## 10. YAX 팀 가이드

### 팀 워크플로우

1. **주간 미팅**: 진행 상황 공유
2. **Git 협업**: feature 브랜치 → PR → 리뷰
3. **문서화**: 변경사항 기록

### Git 브랜치 전략

```
main              # 안정 버전
├── develop       # 개발 버전
├── feature/ftg   # 기능 개발
└── feature/pp    # 기능 개발
```

---

## 11. FAQ

**Q: 차량이 진동할 때?**
→ PID 게인 낮추기, 조향 필터 추가

**Q: 속도가 안 나올 때?**
→ VESC 속도 제한 확인, 배터리 충전 상태

**Q: 코너에서 불안정할 때?**
→ 코너 진입 속도 ↓, lookahead ↑

---

## 12. 마무리

6개월간의 YAX F1TENTH 여정을 마쳤습니다!

### 학습 경로 요약

```
Domain 01: Jetson 설정
    ↓
Domain 02: ROS2 기초
    ↓
Domain 03: LiDAR
    ↓
Domain 04: Camera
    ↓
Domain 05: VESC 제어
    ↓
Domain 06: 자율주행 알고리즘
    ↓
Domain 07: ML & E2E
    ↓
Domain 08: 대회 준비 ← 현재
```

**수고하셨습니다! 🏎️**

---

*YAX F1TENTH 교육자료 | 작성일: 2026-02-10*
