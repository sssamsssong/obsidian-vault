# 개발 환경 구축 가이드

> YAX F1TENTH 교육자료 | Phase 0: Setup
> 
> VS Code Remote SSH 및 개발 도구 설정

---

## 1. 개요

### 학습 목표
1. VS Code Remote SSH로 Jetson에 원격 접속할 수 있다
2. SSH 키를 설정하여 비밀번호 없이 접속할 수 있다
3. 개발에 필요한 도구들을 설치할 수 있다
4. 효율적인 개발 워크플로우를 구축할 수 있다

### 예상 소요 시간
- 전체: 1시간
- VS Code 설정: 20분
- SSH 키 설정: 15분
- 개발 도구 설치: 25분

### 전제 조건
- Jetson 초기 설정 완료 (01-Jetson-Setup.md)
- ROS2 설치 완료 (02-ROS2-Installation.md)
- SSH 접속 가능

---

## 2. VS Code 설치 및 Remote SSH 설정

### 2.1 VS Code 설치 (개인 PC)

**Windows**:
1. https://code.visualstudio.com/ 접속
2. Windows 버전 다운로드 및 설치

**Mac**:
```bash
brew install --cask visual-studio-code
```

### 2.2 Remote SSH 확장 설치

1. VS Code 실행
2. 좌측 사이드바에서 Extensions 아이콘 클릭 (또는 `Ctrl+Shift+X`)
3. "Remote - SSH" 검색
4. Microsoft에서 제공하는 "Remote - SSH" 설치

### 2.3 Jetson에 연결

1. `Ctrl+Shift+P`로 Command Palette 열기
2. "Remote-SSH: Connect to Host..." 선택
3. SSH 연결 정보 입력:
   ```
   jetson@<JETSON_IP>
   ```
   예: `jetson@192.168.1.100`
4. 비밀번호 입력

### 2.4 연결 확인

연결 성공 시:
- VS Code 좌측 하단에 `SSH: <JETSON_IP>` 표시
- Terminal에서 Jetson 명령어 실행 가능

---

## 3. SSH 키 설정 (비밀번호 없이 접속)

매번 비밀번호를 입력하지 않도록 SSH 키를 설정합니다.

### 3.1 SSH 키 생성 (개인 PC)

**Windows (PowerShell 또는 Git Bash)**:
```powershell
# SSH 키 생성
ssh-keygen -t ed25519 -C "your_email@example.com"

# 엔터 3번 (기본 경로, 빈 비밀번호)
```

**Mac/Linux**:
```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

### 3.2 공개키를 Jetson에 복사

**Windows (PowerShell)**:
```powershell
# 공개키 내용 확인
type $env:USERPROFILE\.ssh\id_ed25519.pub

# 출력된 내용을 복사
```

**Mac/Linux**:
```bash
# 자동 복사
ssh-copy-id jetson@<JETSON_IP>
```

### 3.3 Jetson에 공개키 추가

SSH로 Jetson 접속 후:
```bash
# authorized_keys 파일에 공개키 추가
mkdir -p ~/.ssh
chmod 700 ~/.ssh
echo "<복사한_공개키_내용>" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

### 3.4 SSH 설정 파일 (선택사항)

개인 PC의 `~/.ssh/config` 파일에 추가:

```
Host jetson
    HostName <JETSON_IP>
    User jetson
    IdentityFile ~/.ssh/id_ed25519
```

이제 `ssh jetson`만으로 접속 가능합니다.

### 3.5 연결 테스트

```bash
# 비밀번호 없이 접속 확인
ssh jetson

# 또는
ssh jetson@<JETSON_IP>
```

---

## 4. VS Code 추천 확장 프로그램

### 4.1 필수 확장

Jetson에 연결된 상태에서 설치:

| 확장 | 용도 |
|------|------|
| Python | Python 개발, 디버깅 |
| Pylance | Python 타입 체크, 자동완성 |
| C/C++ | C++ 개발 (선택) |
| CMake Tools | CMake 프로젝트 지원 |

### 4.2 ROS2 개발용 확장

| 확장 | 용도 |
|------|------|
| ROS | ROS2 명령어, 런치 파일 지원 |
| XML | XML 파일 편집 (launch, urdf) |
| YAML | YAML 파일 편집 (config) |

### 4.3 생산성 확장

| 확장 | 용도 |
|------|------|
| GitLens | Git 기록 시각화 |
| Todo Tree | TODO 주석 관리 |
| Error Lens | 인라인 오류 표시 |

### 4.4 확장 설치 명령

VS Code Command Palette (`Ctrl+Shift+P`):
```
ext install ms-python.python
ext install ms-python.vscode-pylance
ext install ms-vscode.cmake-tools
ext install ms-iot.vscode-ros
```

---

## 5. Jetson 개발 도구 설치

### 5.1 기본 개발 도구

```bash
# 필수 도구
sudo apt update
sudo apt install -y \
    git \
    curl \
    wget \
    vim \
    nano \
    htop \
    tree

# Python 개발 도구
sudo apt install -y \
    python3-pip \
    python3-venv

# 빌드 도구
sudo apt install -y \
    build-essential \
    cmake
```

### 5.2 tmux 설정 (터미널 멀티플렉서)

tmux는 여러 터미널을 관리하고 세션을 유지합니다.

```bash
# tmux 설치
sudo apt install -y tmux

# tmux 설정 파일 생성
cat > ~/.tmux.conf << 'EOF'
# 마우스 지원
set -g mouse on

# 256 색상
set -g default-terminal "screen-256color"

# 창 분할 단축키
bind | split-window -h
bind - split-window -v

# 상태바
set -g status-bg black
set -g status-fg white
EOF
```

**tmux 기본 사용법**:
```bash
# 새 세션 시작
tmux new -s f1tenth

# 세션 분리 (detach): Ctrl+B, D
# 세션 재접속
tmux attach -t f1tenth

# 창 분할: Ctrl+B, | (가로) 또는 Ctrl+B, - (세로)
# 창 이동: Ctrl+B, 화살표
```

### 5.3 Git 설정

```bash
# 사용자 정보 설정
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"

# 기본 브랜치 이름
git config --global init.defaultBranch main

# 유용한 별칭
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
```

### 5.4 Python 가상환경 (선택)

프로젝트별 Python 환경 분리:

```bash
# 가상환경 생성
cd ~/f1tenth_ws
python3 -m venv venv

# 활성화
source venv/bin/activate

# 비활성화
deactivate
```

---

## 6. 효율적인 개발 워크플로우

### 6.1 VS Code 워크스페이스 설정

1. VS Code에서 `~/f1tenth_ws` 폴더 열기
2. File > Save Workspace As... > `f1tenth.code-workspace`

```json
// f1tenth.code-workspace
{
    "folders": [
        {
            "path": "."
        }
    ],
    "settings": {
        "python.defaultInterpreterPath": "/usr/bin/python3",
        "python.analysis.extraPaths": [
            "/opt/ros/humble/lib/python3.10/site-packages"
        ],
        "files.associations": {
            "*.launch": "xml",
            "*.launch.py": "python"
        }
    }
}
```

### 6.2 VS Code 터미널 활용

- `` Ctrl+` ``: 터미널 열기/닫기
- `Ctrl+Shift+5`: 터미널 분할
- 여러 터미널에서 동시 작업 가능

### 6.3 빌드 단축키 설정

`.vscode/tasks.json`:
```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "colcon build",
            "type": "shell",
            "command": "cd ~/f1tenth_ws && colcon build",
            "group": {
                "kind": "build",
                "isDefault": true
            }
        }
    ]
}
```

`Ctrl+Shift+B`로 빌드 실행.

### 6.4 디버깅 설정

`.vscode/launch.json`:
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Python: Current File",
            "type": "python",
            "request": "launch",
            "program": "${file}",
            "console": "integratedTerminal"
        },
        {
            "name": "ROS2: Launch",
            "type": "ros",
            "request": "launch",
            "target": "${workspaceFolder}/src/<package>/launch/<launch_file>.py"
        }
    ]
}
```

---

## 7. 검증 체크리스트

### VS Code
- [ ] VS Code Remote SSH로 Jetson 연결 성공
- [ ] SSH 키 설정으로 비밀번호 없이 접속
- [ ] 터미널에서 `ros2 topic list` 실행 가능

### 개발 도구
- [ ] git 설정 완료 (`git config --list`로 확인)
- [ ] tmux 실행 가능
- [ ] Python 확장 설치 및 자동완성 동작

### 워크플로우
- [ ] ~/f1tenth_ws 폴더를 VS Code에서 열 수 있음
- [ ] 터미널에서 `colcon build` 실행 가능

---

## 8. 문제 해결

### 8.1 Remote SSH 연결 실패

**증상**: "Could not establish connection"

**해결책**:
```bash
# Jetson에서 SSH 서비스 확인
sudo systemctl status ssh

# 재시작
sudo systemctl restart ssh

# 방화벽 확인
sudo ufw status
sudo ufw allow ssh
```

### 8.2 SSH 키 인증 실패

**증상**: 여전히 비밀번호 요청

**해결책**:
```bash
# Jetson에서 권한 확인
ls -la ~/.ssh/
# authorized_keys는 600, .ssh 폴더는 700이어야 함

chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

### 8.3 VS Code 확장 설치 안됨

**증상**: 원격에서 확장 설치 실패

**해결책**:
1. VS Code 재시작
2. 원격 연결 해제 후 재연결
3. 확장 탭에서 "Install in SSH: jetson" 클릭

### 8.4 Python 자동완성 안됨

**증상**: ROS2 모듈 import 오류

**해결책**:
```json
// settings.json에 추가
{
    "python.analysis.extraPaths": [
        "/opt/ros/humble/lib/python3.10/site-packages",
        "~/f1tenth_ws/install/<package>/lib/python3.10/site-packages"
    ]
}
```


---

*효율적인 개발 환경을 구축했습니다! 이제 편하게 코딩할 준비가 되었습니다.*
