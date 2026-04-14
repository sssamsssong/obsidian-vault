# 대회 최종 준비

> YAX F1TENTH 교육자료 | Phase 6: Competition
> 
> F1TENTH 대회 참가를 위한 최종 점검

---

## 1. 개요

### 학습 목표
1. 대회 전 완벽한 준비 상태를 갖출 수 있다
2. 현장에서 발생할 수 있는 문제에 대비할 수 있다
3. 팀으로서 효과적으로 협력할 수 있다

### 예상 소요 시간
- 전체: 2시간 (문서) + 실제 준비 시간

### 전제 조건
- Phase 1-5 완료
- 성능 튜닝 및 레이싱 전략 수립

---

## 2. 대회 2주 전

### 2.1 시스템 점검

```markdown
## 하드웨어 체크리스트

### 차량 본체
- [ ] 섀시 균열/손상 없음
- [ ] 바퀴 정렬 상태 양호
- [ ] 베어링 상태 확인
- [ ] 서스펜션 점검
- [ ] 모든 나사 조임 상태

### 전자장치
- [ ] Jetson Orin Nano 동작 확인
- [ ] LiDAR 정상 동작 (40Hz)
- [ ] 카메라 정상 동작 (30fps)
- [ ] VESC 동작 확인
- [ ] 모든 케이블 연결 상태

### 전원
- [ ] 배터리 상태 (사이클 수, 용량)
- [ ] 충전기 동작 확인
- [ ] 전압/전류 모니터링 가능
- [ ] 예비 배터리 준비

### 무선 통신
- [ ] WiFi 연결 안정성
- [ ] SSH 접속 확인
- [ ] 비상 정지 리모컨 동작
```

### 2.2 소프트웨어 점검

```markdown
## 소프트웨어 체크리스트

### 시스템
- [ ] OS 최신 상태
- [ ] ROS2 정상 동작
- [ ] 모든 패키지 빌드 성공
- [ ] 의존성 충돌 없음

### 알고리즘
- [ ] 메인 알고리즘 동작 확인
- [ ] 백업 알고리즘 준비
- [ ] 파라미터 파일 정리
- [ ] Launch 파일 테스트

### 데이터
- [ ] 최신 웨이포인트 백업
- [ ] 파라미터 세트 저장
- [ ] 로그 저장 경로 확인
```

### 2.3 예비 부품 목록

```markdown
## 예비 부품

### 필수
- [ ] 예비 배터리 2개
- [ ] 예비 LiDAR (가능하면)
- [ ] USB 케이블 (각종)
- [ ] 점프 케이블
- [ ] 나사/너트 세트

### 도구
- [ ] 육각 렌치 세트
- [ ] 드라이버 세트
- [ ] 멀티미터
- [ ] 절연 테이프
- [ ] 케이블 타이

### 기타
- [ ] 노트북 (개발용)
- [ ] 충전기
- [ ] 무선 마우스/키보드
- [ ] 모니터 (HDMI 연결용)
```

---

## 3. 대회 1주 전

### 3.1 최종 테스트

```python
#!/usr/bin/env python3
"""
시스템 자가 진단 스크립트
"""

import subprocess
import os


def run_diagnostics():
    results = {}
    
    # 1. 센서 확인
    print("=== Sensor Check ===")
    
    # LiDAR
    lidar_topic = subprocess.run(
        ['ros2', 'topic', 'hz', '/scan', '--window', '10'],
        capture_output=True, timeout=15
    )
    results['lidar'] = '40Hz' in lidar_topic.stdout.decode()
    print(f"LiDAR: {'OK' if results['lidar'] else 'FAIL'}")
    
    # Camera
    camera_topic = subprocess.run(
        ['ros2', 'topic', 'hz', '/camera/image_raw', '--window', '10'],
        capture_output=True, timeout=15
    )
    results['camera'] = '30' in camera_topic.stdout.decode()
    print(f"Camera: {'OK' if results['camera'] else 'FAIL'}")
    
    # 2. 모터 확인
    print("\n=== Motor Check ===")
    # VESC 연결 확인
    vesc_check = os.path.exists('/dev/ttyACM0')
    results['vesc'] = vesc_check
    print(f"VESC: {'OK' if vesc_check else 'FAIL'}")
    
    # 3. 메모리/저장공간
    print("\n=== System Resources ===")
    df = subprocess.run(['df', '-h', '/'], capture_output=True)
    print(df.stdout.decode())
    
    free = subprocess.run(['free', '-h'], capture_output=True)
    print(free.stdout.decode())
    
    # 결과 요약
    print("\n=== Summary ===")
    all_pass = all(results.values())
    print(f"Overall: {'ALL PASS' if all_pass else 'SOME FAILURES'}")
    
    return results


if __name__ == '__main__':
    run_diagnostics()
```

### 3.2 파라미터 세트 준비

```yaml
# config/competition_params.yaml

# 세트 1: 안전 우선 (예선용)
safe_mode:
  max_speed: 2.5
  bubble_radius: 0.35
  lookahead: 1.5
  steering_limit: 0.35

# 세트 2: 균형 (일반 경주용)  
balanced_mode:
  max_speed: 3.5
  bubble_radius: 0.28
  lookahead: 1.2
  steering_limit: 0.4

# 세트 3: 공격적 (리드 시 또는 결승)
aggressive_mode:
  max_speed: 4.5
  bubble_radius: 0.22
  lookahead: 1.0
  steering_limit: 0.4

# 세트 4: 비상 (문제 발생 시)
emergency_mode:
  max_speed: 1.5
  bubble_radius: 0.5
  lookahead: 2.0
  steering_limit: 0.3
```

### 3.3 문서화

```markdown
## 차량 운영 매뉴얼

### 시작 절차
1. 배터리 연결
2. Jetson 전원 ON
3. SSH 접속: `ssh yax@192.168.1.100`
4. 센서 노드 시작: `ros2 launch f1tenth_stack sensors.launch.py`
5. 자율주행 노드 시작: `ros2 launch f1tenth_stack autonomy.launch.py`

### 파라미터 변경
```bash
# 실시간 변경
ros2 param set /autonomy max_speed 3.0

# 파일에서 로드
ros2 launch f1tenth_stack autonomy.launch.py \
    config_file:=competition_params.yaml \
    mode:=balanced_mode
```

### 비상 정지
- 리모컨 빨간 버튼
- 또는: `ros2 topic pub /e_stop std_msgs/Bool "data: true"`

### 문제 해결
- 센서 오류 → 케이블 확인, 재시작
- 네트워크 끊김 → WiFi 채널 변경
- 모터 무반응 → VESC 재연결
```

---

## 4. 대회 전날

### 4.1 패킹 체크리스트

```markdown
## 대회 패킹 리스트

### 차량
- [ ] F1TENTH 차량 (완전 조립 상태)
- [ ] 보호 케이스/가방

### 전원
- [ ] 주 배터리 (완충)
- [ ] 예비 배터리 2개 (완충)
- [ ] 배터리 충전기
- [ ] 멀티탭

### 컴퓨터
- [ ] 개발용 노트북
- [ ] 노트북 충전기
- [ ] USB 허브
- [ ] HDMI 케이블
- [ ] 휴대용 모니터 (선택)

### 도구
- [ ] 공구 세트
- [ ] 멀티미터
- [ ] 케이블/테이프

### 문서
- [ ] 운영 매뉴얼 (인쇄)
- [ ] 파라미터 세트 (인쇄)
- [ ] 비상 연락처

### 개인
- [ ] 신분증
- [ ] 팀 유니폼/네임택
- [ ] 노트/펜
```

### 4.2 최종 확인

```bash
#!/bin/bash
# final_check.sh

echo "=== F1TENTH 최종 점검 ==="

# 1. 시스템 상태
echo -e "\n[1/5] System Status"
uptime
free -h | head -2

# 2. 디스크 공간
echo -e "\n[2/5] Disk Space"
df -h / | tail -1

# 3. 네트워크
echo -e "\n[3/5] Network"
ip addr show wlan0 | grep inet

# 4. ROS2 패키지
echo -e "\n[4/5] ROS2 Packages"
source /opt/ros/humble/setup.bash
source ~/f1tenth_ws/install/setup.bash
ros2 pkg list | grep f1tenth

# 5. 센서 토픽
echo -e "\n[5/5] Topics"
timeout 5 ros2 topic list

echo -e "\n=== 점검 완료 ==="
```

---

## 5. 대회 당일

### 5.1 아침 루틴

```
07:00  기상, 아침 식사
08:00  장비 최종 점검
08:30  이동
09:00  현장 도착, 부스 설치
09:30  기술 검사 대기
10:00  연습 세션 시작
```

### 5.2 현장 설정

```markdown
## 부스 설정

### 작업 공간
1. 테이블 위 정리
   - 노트북 (중앙)
   - 공구 (오른쪽)
   - 여분 부품 (왼쪽)

2. 충전 스테이션
   - 멀티탭 연결
   - 배터리 충전기 설치
   - 충전 중인 배터리 표시

3. 통신 확인
   - WiFi 연결 테스트
   - SSH 접속 확인
   - 팀 통신 수단 (무전기/메신저)
```

### 5.3 기술 검사 대응

```markdown
## 기술 검사 항목

### 차량 사양
- 크기: 1/10 스케일 규격 내
- 무게: 제한 이내
- 센서: 허용된 센서만 사용

### 안전
- 비상 정지 버튼 동작
- 배터리 보호 (LiPo bag)
- 날카로운 모서리 없음

### 소프트웨어
- 자율주행 시연
- 비상 정지 테스트
- 원격 제어 가능
```

### 5.4 경주 중 역할 분담

```
┌─────────────────────────────────────────────────┐
│                    레이스 중                      │
├─────────────────────────────────────────────────┤
│                                                 │
│  [드라이버]         [엔지니어]      [스트래티지스트] │
│  - 실시간 모니터링   - 하드웨어 대기  - 상대 분석    │
│  - 파라미터 조정    - 문제 해결     - 전략 조언    │
│  - 로그 확인       - 배터리 관리   - 다음 라운드 준비│
│                                                 │
│  ──────────────────────────────────────────────  │
│                                                 │
│                   [관전자/기록]                   │
│                   - 사진/영상 촬영                │
│                   - SNS 업데이트                 │
│                   - 결과 기록                    │
│                                                 │
└─────────────────────────────────────────────────┘
```

---

## 6. 트러블슈팅 가이드

### 6.1 일반적인 문제

| 증상 | 가능한 원인 | 해결책 |
|------|-------------|--------|
| 부팅 안됨 | 배터리 방전 | 충전된 배터리 교체 |
| SSH 안됨 | IP 변경 | `nmap` 스캔, 직접 모니터 연결 |
| LiDAR 없음 | USB 연결 | 케이블 교체, 포트 변경 |
| 모터 무반응 | VESC 오류 | VESC Tool로 진단 |
| 진동 심함 | 바퀴 불량 | 바퀴 교체, 밸런싱 |

### 6.2 소프트웨어 문제

```bash
# 노드가 죽었을 때
ros2 node list  # 실행 중인 노드 확인
ros2 launch ... # 재시작

# 토픽이 안 나올 때
ros2 topic list
ros2 topic echo /scan --once

# 빌드 오류
cd ~/f1tenth_ws
rm -rf build install log
colcon build --symlink-install

# 파라미터 초기화
ros2 param dump /autonomy > backup_params.yaml
ros2 param load /autonomy default_params.yaml
```

### 6.3 응급 수리

```markdown
## 응급 수리 가이드

### 케이블 단선
1. 손상 부위 확인
2. 예비 케이블로 교체
3. 없으면 절연 테이프로 임시 연결

### 나사 풀림
1. 해당 나사 확인
2. 적절한 공구로 조임
3. 나사못풀림 방지제 도포

### 센서 탈락
1. 마운트 상태 확인
2. 예비 마운트 또는 케이블 타이로 고정
3. 캘리브레이션 재확인

### 배터리 부풀음
⚠️ 즉시 사용 중지!
1. 안전한 곳에 격리
2. 예비 배터리 사용
3. 대회 운영진에게 보고
```

---

## 7. 대회 후

### 7.1 정리

```markdown
## 대회 후 체크리스트

### 현장 정리
- [ ] 모든 장비 수거
- [ ] 작업 공간 청소
- [ ] 잊은 물건 없는지 확인

### 차량 관리
- [ ] 배터리 분리 (장기 보관 시)
- [ ] 먼지/이물질 청소
- [ ] 파손 부위 기록

### 데이터 백업
- [ ] 경기 로그 저장
- [ ] 최종 파라미터 백업
- [ ] 사진/영상 수집
```

### 7.2 회고

```markdown
## 대회 회고 템플릿

### 기본 정보
- 대회명: 
- 일시: 
- 장소: 
- 최종 순위: 

### 잘된 점
1. 
2. 
3. 

### 개선할 점
1. 
2. 
3. 

### 기술적 교훈
- 알고리즘:
- 하드웨어:
- 튜닝:

### 다음 대회 목표
1. 
2. 

### 감사한 점/특이사항

```

---

## 8. YAX 팀 가이드

### 8.1 팀 규칙

```markdown
## YAX F1TENTH 팀 규칙

1. **안전 최우선**
   - 항상 안전 장비 착용
   - 위험 상황 즉시 보고

2. **협력과 소통**
   - 모든 결정은 팀원과 논의
   - 진행 상황 공유

3. **기록 유지**
   - 모든 실험 기록
   - 코드 변경 커밋

4. **존중**
   - 모든 팀원의 의견 존중
   - 건설적인 피드백
```

### 8.2 역할별 가이드

```markdown
## 역할별 책임

### 팀 리더
- 일정 관리
- 대외 소통
- 의사결정 조율

### 알고리즘 개발자
- 자율주행 알고리즘 개발
- 파라미터 튜닝
- 성능 분석

### 하드웨어 담당
- 차량 조립/유지보수
- 센서 캘리브레이션
- 전원 관리

### 문서/홍보 담당
- 진행 상황 기록
- SNS 운영
- 대회 보고서 작성
```

---

## 9. 자주 묻는 질문

### Q: 처음 대회인데 무엇부터 해야 하나요?
A: 안정적으로 완주하는 것을 첫 번째 목표로 삼으세요. 보수적인 파라미터로 시작하고, 상황을 보며 조정하세요.

### Q: 예선에서 안 좋은 결과가 나왔어요.
A: 당황하지 마세요. 본선까지 시간이 있습니다. 데이터를 분석하고, 필요하면 전략을 수정하세요.

### Q: 차량이 갑자기 고장났어요.
A: 침착하게 증상을 파악하고, 예비 부품으로 교체하세요. 해결이 어려우면 다른 팀에 도움을 요청할 수도 있습니다.

### Q: 상대팀이 압도적으로 빠릅니다.
A: 그들에게 배우세요! 레이스 후 친하게 지내며 노하우를 물어보세요. F1TENTH 커뮤니티는 열려 있습니다.

---

## 10. 최종 체크리스트

```markdown
## 대회 준비 최종 점검

### 2주 전
- [ ] 하드웨어 전체 점검
- [ ] 소프트웨어 빌드 확인
- [ ] 예비 부품 목록 작성

### 1주 전
- [ ] 자가 진단 스크립트 실행
- [ ] 파라미터 세트 준비
- [ ] 운영 매뉴얼 최종화

### 전날
- [ ] 모든 장비 패킹
- [ ] 배터리 완충
- [ ] 이동 경로 확인

### 당일 아침
- [ ] 장비 동작 확인
- [ ] 팀원 역할 재확인
- [ ] 긍정적 마인드셋!

### 현장 도착 후
- [ ] 부스 설정
- [ ] WiFi 연결
- [ ] 기술 검사 준비
- [ ] 연습 세션 계획
```

---

## 11. 마무리

### YAX 팀에게

```
F1TENTH 대회는 단순한 경쟁이 아닙니다.

여러분이 배운 모든 것을 시험하는 기회이자,
같은 열정을 가진 사람들을 만나는 장이며,
다음 단계로 나아가는 발판입니다.

승패와 관계없이,
이 경험이 여러분의 미래를 밝혀줄 것입니다.

화이팅! 🏎️
```

---

*이 문서로 YAX F1TENTH 커리큘럼의 모든 Phase가 완료됩니다.*
*준비가 되었다면, 레이스에 나서세요!*
