# 레이싱 전략

> YAX F1TENTH 교육자료 | Phase 6: Competition
> 
> F1TENTH 대회에서 승리하기 위한 전략

---

## 1. 개요

### 학습 목표
1. F1TENTH 대회 형식과 규칙을 이해할 수 있다
2. 상황별 레이싱 전략을 수립할 수 있다
3. 헤드투헤드 레이스에서의 전술을 적용할 수 있다

### 예상 소요 시간
- 전체: 2시간

### 전제 조건
- 성능 튜닝 완료
- 안정적인 자율주행 가능

---

## 2. F1TENTH 대회 형식

### 2.1 대회 종류

| 종류 | 설명 | 평가 기준 |
|------|------|-----------|
| **타임 트라이얼** | 단독 주행, 최단 시간 | 랩타임 |
| **헤드투헤드** | 1:1 대결 | 먼저 결승선 통과 |
| **레이스** | 다중 차량 경쟁 | 순위 |

### 2.2 일반 규칙

- 충돌 시 페널티 또는 실격
- 트랙 이탈 시 페널티
- 일정 바퀴 수 또는 시간 제한
- 차량 사양 제한 (하드웨어 동일)

### 2.3 전형적인 대회 일정

```
Day 1: 기술 검사 + 연습 세션
  └─► 차량 규격 확인
  └─► 트랙 탐색

Day 2: 예선 (타임 트라이얼)
  └─► 그리드 순서 결정
  └─► 알고리즘 미세 튜닝

Day 3: 본선 (헤드투헤드 또는 레이스)
  └─► 토너먼트 진행
  └─► 결승전
```

---

## 3. 타임 트라이얼 전략

### 3.1 핵심 원칙

```
최단 시간 = 최단 거리 + 최대 속도 + 안정성
```

### 3.2 레이싱 라인

```
             ┌──────────────────────────┐
             │                          │
             │    2. 에이펙스 (정점)    │
             │         ●                │
            ╱│        ╱ ╲               │╲
           ╱ │       ╱   ╲              │ ╲
    1. 아웃│  ▼     ╱     ╲     ▼       │  │3. 아웃
          ╲ ───────●───────●────────   ╱
           ╲              탈출         ╱
            ╲_________________________╱

1. 코너 진입: 바깥쪽에서 시작
2. 에이펙스: 안쪽 정점 통과  
3. 코너 탈출: 바깥쪽으로 가속
```

### 3.3 속도 관리

```python
# 코너별 속도 전략
def corner_speed_strategy(curvature: float,
                          track_grip: float = 1.0) -> dict:
    """
    곡률에 따른 속도 전략.
    """
    if curvature < 0.1:
        # 직선/완만한 커브
        return {
            'entry_speed': 'max',
            'apex_speed': 'max',
            'exit_speed': 'max'
        }
    elif curvature < 0.3:
        # 중간 커브
        return {
            'entry_speed': 'brake_late',
            'apex_speed': 'maintain',
            'exit_speed': 'accelerate'
        }
    else:
        # 급커브
        return {
            'entry_speed': 'brake_early',
            'apex_speed': 'minimum',
            'exit_speed': 'accelerate_hard'
        }
```

### 3.4 리스크 관리

| 접근법 | 장점 | 단점 |
|--------|------|------|
| 보수적 | 완주 확실 | 느린 랩타임 |
| 공격적 | 빠른 랩타임 | 충돌 위험 |
| **균형** | 안정적 + 빠름 | 경험 필요 |

**권장**: 처음 2-3바퀴는 보수적, 이후 점진적 공격

---

## 4. 헤드투헤드 전략

### 4.1 시작 위치에 따른 전략

```
선두 출발:                후발 출발:
┌─────────────────┐      ┌─────────────────┐
│       ●         │      │                 │
│    (선두)       │      │       ●         │
│                 │      │    (후발)       │
│       ○         │      │       ○         │
│    (상대)       │      │    (상대/선두)  │
└─────────────────┘      └─────────────────┘

전략: 레이싱 라인 유지     전략: 추월 기회 탐색
      간격 유지                  실수 유도
      실수 최소화                슬립스트림 활용
```

### 4.2 추월 전략

```python
# 추월 의사결정
def should_overtake(gap: float,
                    relative_speed: float,
                    corner_ahead: bool,
                    track_position: str) -> bool:
    """
    추월 시도 여부 결정.
    
    Args:
        gap: 상대와의 거리 (m)
        relative_speed: 상대 대비 속도 (m/s)
        corner_ahead: 전방 코너 여부
        track_position: 현재 트랙 위치
    """
    # 직선에서 속도 우위가 있을 때
    if not corner_ahead and relative_speed > 0.5:
        if gap < 1.0:  # 충분히 가까우면
            return True
    
    # 코너 진입 시 인사이드 확보 가능할 때
    if corner_ahead and track_position == 'inside':
        if gap < 0.5:
            return True
    
    return False
```

### 4.3 방어 전략

```
방어 라인 유지:
              코너
           ┌────────┐
           │   ╱    │
    상대 → │  ╱     │ ← 인사이드 차단
           │ ╱ ●    │
           │╱  (나)  │
           └────────┘

직선에서:
           ─────────────────
           ●  ← 중앙 라인 유지
           ─────────────────
              상대가 좌우 선택 강요
```

### 4.4 FTG vs 추월

Follow the Gap이 상대 차량을 "장애물"로 인식:

```python
class OvertakeAwareFTG:
    """추월 인식 FTG."""
    
    def __init__(self):
        self.opponent_tracker = OpponentTracker()
    
    def find_gap(self, ranges, opponent_pos=None):
        """상대 위치를 고려한 Gap 탐색."""
        
        if opponent_pos is not None:
            # 상대 차량 영역을 장애물로 마킹
            opponent_angles = self.calculate_opponent_angles(opponent_pos)
            ranges = self.mask_opponent_region(ranges, opponent_angles)
        
        # 일반 FTG 로직
        return super().find_gap(ranges)
```

---

## 5. 다중 차량 레이스

### 5.1 포지션별 전략

| 포지션 | 목표 | 전략 |
|--------|------|------|
| 1위 | 유지 | 레이싱 라인 사수, 안정 주행 |
| 2-3위 | 추월 | 1위 실수 대기, 기회 포착 |
| 중위권 | 순위 상승 | 리스크 감수, 공격적 추월 |
| 하위권 | 생존 | 충돌 회피, 점진적 상승 |

### 5.2 다중 장애물 처리

```python
def handle_multiple_opponents(self, ranges, opponents):
    """
    여러 상대 차량 처리.
    """
    # 모든 상대를 장애물로 마킹
    for opp in opponents:
        ranges = self.add_safety_bubble(ranges, opp.position)
    
    # 가장 넓은 안전 Gap 찾기
    gap_start, gap_end = self.find_max_gap(ranges)
    
    # Gap이 너무 좁으면 감속
    gap_width = self.calculate_gap_width(gap_start, gap_end)
    if gap_width < self.min_safe_gap:
        self.reduce_speed()
    
    return self.steer_to_gap(gap_start, gap_end)
```

---

## 6. 트랙 분석

### 6.1 트랙 학습

```python
class TrackAnalyzer:
    """트랙 특성 분석."""
    
    def __init__(self):
        self.sectors = []
        self.corners = []
        self.straights = []
    
    def analyze_from_waypoints(self, waypoints):
        """웨이포인트에서 트랙 특성 추출."""
        
        curvatures = self.calculate_curvatures(waypoints)
        
        current_sector = {'type': None, 'start': 0}
        
        for i, k in enumerate(curvatures):
            if k < 0.05:  # 직선
                sector_type = 'straight'
            elif k < 0.2:  # 완만한 커브
                sector_type = 'gentle_curve'
            else:  # 급커브
                sector_type = 'sharp_corner'
            
            if sector_type != current_sector['type']:
                # 새 섹터 시작
                if current_sector['type'] is not None:
                    current_sector['end'] = i
                    self.sectors.append(current_sector.copy())
                
                current_sector = {'type': sector_type, 'start': i}
        
        return self.sectors
    
    def get_sector_strategy(self, sector_idx):
        """섹터별 전략 반환."""
        sector = self.sectors[sector_idx]
        
        if sector['type'] == 'straight':
            return {'speed': 'max', 'line': 'center'}
        elif sector['type'] == 'gentle_curve':
            return {'speed': 'high', 'line': 'racing'}
        else:
            return {'speed': 'moderate', 'line': 'apex'}
```

### 6.2 코너 분류

```
타입 1: 헤어핀 (180도)
  └─► 강한 감속, 정밀한 에이펙스

타입 2: 90도 코너
  └─► 중간 감속, 아웃-인-아웃

타입 3: 시케인 (연속 S자)
  └─► 리듬 유지, 중앙 라인

타입 4: 고속 스위퍼
  └─► 최소 감속, 부드러운 조향
```

---

## 7. 심리전과 마인드셋

### 7.1 대회 전 준비

```markdown
## 체크리스트

### 하드웨어
- [ ] 배터리 완충
- [ ] 모든 연결 확인
- [ ] 예비 부품 준비

### 소프트웨어
- [ ] 최신 코드 배포
- [ ] 파라미터 백업
- [ ] 비상 정지 테스트

### 전략
- [ ] 트랙 분석 완료
- [ ] 파라미터 세트 준비 (보수적/공격적)
- [ ] 팀 역할 분담
```

### 7.2 실시간 의사결정

```python
# 상황별 대응
RACE_SCENARIOS = {
    'leading': {
        'action': 'maintain_gap',
        'risk_level': 'low',
        'focus': 'consistency'
    },
    'chasing': {
        'action': 'pressure',
        'risk_level': 'medium',
        'focus': 'opportunities'
    },
    'behind_after_incident': {
        'action': 'recover',
        'risk_level': 'high',
        'focus': 'aggressive_but_smart'
    },
    'technical_issues': {
        'action': 'survive',
        'risk_level': 'low',
        'focus': 'finish_race'
    }
}
```

### 7.3 팀워크

| 역할 | 담당 | 책임 |
|------|------|------|
| 드라이버 | 알고리즘 개발자 | 실시간 튜닝 |
| 스트래티지스트 | 분석 담당 | 전략 수립, 상대 분석 |
| 엔지니어 | 하드웨어 담당 | 기술 지원, 문제 해결 |
| 스팟터 | 모니터링 담당 | 트랙 상황 전달 |

---

## 8. 비상 상황 대응

### 8.1 일반적인 문제

| 상황 | 증상 | 대응 |
|------|------|------|
| 센서 오류 | 비정상 데이터 | 백업 알고리즘 전환 |
| 네트워크 끊김 | 원격 모니터링 불가 | 자율 모드 신뢰 |
| 충돌 후 | 방향 틀어짐 | 캘리브레이션 확인 |
| 배터리 저전압 | 성능 저하 | 속도 제한, 완주 우선 |

### 8.2 비상 정지

```python
class EmergencyStop:
    """비상 정지 시스템."""
    
    def __init__(self, node):
        self.node = node
        self.e_stop_active = False
        
        # 비상 정지 조건
        self.collision_threshold = 0.1  # m
        self.stuck_timeout = 3.0  # s
        
    def check_emergency(self, state):
        """비상 상황 확인."""
        
        # 1. 충돌 임박
        if state['min_range'] < self.collision_threshold:
            return self.trigger_stop("Collision imminent")
        
        # 2. 차량 정체
        if state['speed'] < 0.1 and state['time_stuck'] > self.stuck_timeout:
            return self.trigger_stop("Vehicle stuck")
        
        # 3. 센서 오류
        if state['sensor_error']:
            return self.trigger_stop("Sensor failure")
        
        return False
    
    def trigger_stop(self, reason):
        """비상 정지 실행."""
        self.e_stop_active = True
        self.node.get_logger().error(f"E-STOP: {reason}")
        
        # 정지 명령 발행
        stop_msg = AckermannDriveStamped()
        stop_msg.drive.speed = 0.0
        stop_msg.drive.steering_angle = 0.0
        self.node.drive_pub.publish(stop_msg)
        
        return True
```

---

## 9. 실습

### 실습 1: 트랙 분석

1. 시뮬레이터 트랙 탐색
2. 섹터 분류 및 특성 기록
3. 섹터별 속도 프로파일 설계

### 실습 2: 모의 레이스

1. 팀원과 헤드투헤드 모의 진행
2. 추월/방어 시도
3. 전략 토론

### 실습 3: 비상 상황 시뮬레이션

1. 의도적 장애물 배치
2. 비상 정지 테스트
3. 복구 절차 연습

---

## 10. 검증 체크리스트

- [ ] 타임 트라이얼 전략 이해
- [ ] 헤드투헤드 전술 숙지
- [ ] 트랙 분석 방법론
- [ ] 비상 대응 계획
- [ ] 팀 역할 분담

---

*레이싱은 기술과 전략의 조합입니다. 준비된 자가 승리합니다!*
