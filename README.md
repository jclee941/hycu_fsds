# HYCU FSDS 자율주행

Formula Student Driverless Simulator (FSDS) 기반 자율주행 시스템.
Windows에서 실행되는 FSDS 시뮬레이터와 Linux Docker(ROS Noetic) 환경의
자율주행 스택을 결합한 듀얼 플랫폼 프로젝트.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
![ROS](https://img.shields.io/badge/ROS-Noetic-blue)
![Python](https://img.shields.io/badge/Python-3-green)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED)

---

## 개요

- **목표**: Formula Student Driverless 대회용 자율주행 알고리즘 개발 및 검증
- **시뮬레이터**: [Formula-Student-Driverless-Simulator](https://github.com/FS-Driverless/Formula-Student-Driverless-Simulator) v2.2.0 (AirSim 기반)
- **자율주행 스택**: ROS Noetic + Python 3 (Pure Pursuit + Curvature-based Speed Control)
- **상태 관리**: 3-tier Watchdog State Machine (TRACKING → DEGRADED → STOPPING)

## 아키텍처

```
┌────────────────────────────┐         ┌────────────────────────────────────┐
│  Windows / Linux (Host)    │  RPC    │  Linux Docker (ROS Noetic)         │
│                            │ :41451  │                                    │
│  FSDS Simulator (UE4)      │◄───────►│  fsds_bridge ──► /lidar /odom ...  │
│   - LiDAR x2               │         │       │                            │
│   - GPS / IMU / GSS        │         │       ▼                            │
│   - Camera x2              │         │  competition_driver (ROS Node)     │
│                            │         │   ├─ perception/cone_detector      │
│                            │         │   ├─ perception/slam               │
│                            │         │   ├─ control/pure_pursuit          │
│                            │         │   ├─ control/speed                 │
│                            │         │   └─ utils/watchdog                │
│                            │         │       │                            │
│                            │         │       ▼  /fsds/control_command     │
│                            │◄────────┤  AirSim Python Client (RPC)        │
└────────────────────────────┘         └────────────────────────────────────┘
```

## 프로젝트 구조

```
hycu_fsds/
├── src/
│   ├── simulator/              # Windows FSDS 시뮬레이터 설정
│   │   ├── settings.json       # AirSim 차량/센서 정의
│   │   └── README.md
│   │
│   └── autonomous/             # ★ 메인 개발 디렉토리
│       ├── driver/
│       │   └── competition_driver.py   # 메인 ROS 노드 (1044L)
│       ├── modules/                    # Pure Python (ROS import 금지)
│       │   ├── control/
│       │   │   ├── pure_pursuit.py     # 조향 알고리즘 (96L)
│       │   │   └── speed.py            # 속도 제어 (190L)
│       │   ├── perception/
│       │   │   ├── cone_detector.py    # 콘 검출 (Grid BFS, 227L)
│       │   │   ├── cone_classifier.py  # 청색/황색 분류 (300L)
│       │   │   └── slam.py             # 점유 격자 SLAM (262L)
│       │   └── utils/
│       │       ├── watchdog.py         # 상태 머신 (294L)
│       │       └── lap_timer.py        # 랩타임 (207L)
│       ├── config/
│       │   ├── params.yaml             # 모든 튜닝 파라미터 (84L)
│       │   └── bridge_no_camera.launch # 카메라 미사용 ROS launch
│       ├── tests/
│       │   └── test_algorithms.py      # 단위 테스트 (846L, 29 methods)
│       ├── Dockerfile                  # ROS Noetic + 의존성 (pinned)
│       ├── docker-compose.yml          # 3 services
│       ├── .env                        # FSDS_HOST_IP / FSDS_PORT
│       ├── start.sh                    # 원클릭 실행 스크립트
│       ├── entrypoint.sh
│       └── run_all.sh
│
├── fsds_docker/                # ⚠️  DEPRECATED (legacy 참고용)
├── submission/                 # 자동 생성 배포 패키지 (직접 수정 금지)
├── scripts/
│   └── package.sh              # submission/ 자동 패키징
├── docs/
│   ├── SUBMISSION_GUIDE.md
│   └── reference_materials/    # 강의자료, SLAM/V2X 노트북
├── dist/                       # 배포 산출물
├── .github/                    # 30+ workflow (CI / 자동 머지 등)
├── AGENTS.md                   # 프로젝트 컨벤션·아키텍처 (필독)
├── LICENSE                     # MIT
└── README.md                   # 이 파일
```

> `submission/`은 `scripts/package.sh`로 자동 생성됩니다. 직접 편집하지 마세요.
> `fsds_docker/`는 deprecated 상태로, 신규 작업은 모두 `src/autonomous/`에서 진행합니다.

## 사전 요구사항

### Windows (시뮬레이터 호스트)
- Windows 10/11 (NVIDIA GPU 권장)
- [FSDS v2.2.0](https://github.com/FS-Driverless/Formula-Student-Driverless-Simulator/releases) 다운로드 및 압축 해제

### Linux (자율주행 스택)
- Ubuntu 20.04+ (또는 호환 배포판)
- Docker Engine 20.10+ / Docker Compose v2
- NVIDIA GPU + nvidia-container-toolkit (Linux에서 시뮬레이터까지 실행할 경우)
- X11 디스플레이 (`$DISPLAY` 설정)

## 빠른 시작

### 1. Windows에서 시뮬레이터 실행 (Cross-platform 모드)

```powershell
# AirSim 설정 폴더에 settings.json 복사
copy src\simulator\settings.json %USERPROFILE%\Documents\AirSim\settings.json

# FSDS.exe 실행
.\FSDS.exe
```

### 2. Linux에서 자율주행 스택 실행

```bash
cd src/autonomous

# (A) 자동: Linux에서 시뮬레이터까지 함께 실행
./start.sh                  # 기본 맵: TrainingMap
./start.sh CompetitionMap1  # 다른 맵 선택

# (B) 수동: Windows에서 시뮬레이터를 돌리는 경우
echo "FSDS_HOST_IP=<Windows IP>" > .env
echo "FSDS_PORT=41451" >> .env
docker compose up -d --build
```

### 3. 드라이버 실행

```bash
docker exec -it fsds_autonomous bash
python3 /root/catkin_ws/src/autonomous/driver/competition_driver.py

# 옵션: 랩 수 지정
python3 /root/catkin_ws/src/autonomous/driver/competition_driver.py _target_laps:=1
```

## 설정

주요 파라미터는 모두 [`src/autonomous/config/params.yaml`](src/autonomous/config/params.yaml)에서 관리합니다.

| 파라미터 | 기본값 | 설명 |
|---|---|---|
| `max_speed` | 6.0 m/s | 최대 속도 |
| `max_throttle` | 0.5 | 스로틀 출력 한계 [0-1] |
| `max_steering` | 0.5 rad | 최대 조향각 |
| `lookahead_base` | 3.5 m | Pure Pursuit 기준 거리 |
| `lookahead_speed_gain` | 0.4 | `lookahead = base + speed × gain` |
| `cones_range_cutoff` | 20.0 m | 콘 검출 거리 |
| `target_laps` | 1 | 종료 랩 수 (0 = 무한) |
| `degraded_timeout` | 3.0 s | TRACKING → DEGRADED 전환 |
| `stopping_timeout` | 5.0 s | DEGRADED → STOPPING 전환 |

런타임 변경:
```bash
rosparam set /competition_driver/max_speed 8.0
```

## Docker 서비스

`src/autonomous/docker-compose.yml`이 정의하는 3개 서비스:

| 서비스 | 역할 | Health Check |
|---|---|---|
| `roscore` | ROS Master | `rostopic list` |
| `fsds_bridge` | FSDS ↔ ROS RPC 브릿지 | `roscore` healthy 의존 |
| `autonomous` | 드라이버 컨테이너 | `sleep infinity` (수동 모드) |

```bash
# 로그 확인
docker compose logs -f autonomous

# 컨테이너 진입
docker exec -it fsds_autonomous bash

# 종료
docker compose down
```

## 개발 가이드

### 모듈 책임 분리

| 모듈 | 목적 | ROS import |
|---|---|---|
| `modules/control/` | 순수 수학 (Pure Pursuit / 속도) | ❌ numpy 만 |
| `modules/perception/` | 센서 처리 (콘 검출 / SLAM) | ❌ numpy 만 |
| `modules/utils/` | 상태 머신 / 타이머 | ❌ pure Python |
| `driver/` | ROS glue / 콜백 / 메인 루프 | ✅ |

> `modules/`는 의도적으로 ROS-free 입니다. 단위 테스트와 재사용성을 보장하기 위함입니다.

### Anti-patterns

| 금지 | 대신 |
|---|---|
| `modules/control/`에 ROS import | numpy array로 데이터 전달 |
| 모듈 내 전역 상태 | 명시적 인자 / 반환값 |
| `sys.path.append()` | `entrypoint.sh`에서 `PYTHONPATH` 설정 |
| 콜백 내 blocking I/O | `threading.Lock`으로 핸드오프 |

### 테스트

```bash
# 컨테이너 내부에서
python3 -m pytest tests/ -v
```

테스트는 `unittest` 기반이며, 사전에 `sys.modules['rospy'] = MagicMock()`으로
ROS를 mocking 하여 호스트에서도 실행 가능합니다.

### 커밋·CI

- Conventional Commits 강제 (`.github/workflows/commitlint.yml`)
- PR 라벨링·자동 머지·릴리즈 드래프터 워크플로우 활성화
- 자세한 컨벤션은 [`AGENTS.md`](AGENTS.md) 참고

## 사용 가능한 맵

`start.sh <맵이름>` 형태로 선택:
- `TrainingMap` (기본)
- `CompetitionMap1`
- `CompetitionMap2`
- `CompetitionMap3`
- `Skidpad`

## 트러블슈팅

| 증상 | 원인 / 해결 |
|---|---|
| `Connection refused 41451` | Windows 방화벽 / `FSDS_HOST_IP` 확인, FSDS 먼저 실행 |
| `cameralauncher crash` | 기본 launch가 `bridge_no_camera.launch`로 카메라 미사용 |
| `llvmpipe / 소프트웨어 렌더링` | `start.sh`가 `__NV_PRIME_RENDER_OFFLOAD=1` 강제, GPU 드라이버 확인 |
| `xhost / DISPLAY 오류` | `xhost +local:root` 후 `DISPLAY=:0` 설정 |
| 컨테이너에서 모듈 import 실패 | `PYTHONPATH=/root/catkin_ws/src/autonomous` 확인 |

## 라이선스

MIT License — 자세한 내용은 [LICENSE](LICENSE) 참고.

## 참고

- [Formula Student Driverless Simulator](https://github.com/FS-Driverless/Formula-Student-Driverless-Simulator)
- [AGENTS.md](AGENTS.md) — 본 프로젝트의 상세 컨벤션 / 아키텍처
- [docs/SUBMISSION_GUIDE.md](docs/SUBMISSION_GUIDE.md) — 대회 제출 가이드


<!-- LLM review probe 1777811213 — auto-removed after verification -->
