# Formula Student Driverless — Autonomous Racing Stack

> **FSD (Formula Student Driverless) 자율주행 레이싱 소프트웨어 스택**
> Autonomous racing software stack for the Formula Student Driverless competition

이 저장소는 FSD 대회를 위한 완전 자율주행 레이싱 차량 소프트웨어를 제공합니다. SLAM, 콘 감지/분류, Pure Pursuit 추종 제어, 랩 타이머, V2X(차량-사물 통신) 어댑터, 그리고 대회 규정에 맞는 Docker 제출 패키지를 포함합니다.

This repository provides a full-stack autonomous driving system for the Formula Student Driverless competition. It bundles SLAM, cone detection/classification, pure-pursuit lane following, lap timing, a V2X (RSU) adapter, and a competition-ready Docker submission package.

---

## Table of Contents / 목차

1. [Overview / 개요](#overview--개요)
2. [Target Users / 대상 사용자](#target-users--대상-사용자)
3. [Features / 주요 기능](#features--주요-기능)
4. [Architecture / 아키텍처](#architecture--아키텍처)
5. [Repository Layout / 저장소 구조](#repository-layout--저장소-구조)
6. [Quick Start / 빠른 시작](#quick-start--빠른-시작)
7. [Configuration / 설정](#configuration--설정)
8. [Commands Reference / 명령어 레퍼런스](#commands-reference--명령어-레퍼런스)
9. [Local Development / 로컬 개발](#local-development--로컬-개발)
10. [Testing / 테스트](#testing--테스트)
11. [Simulation / 시뮬레이션](#simulation--시뮬레이션)
12. [Submission Package / 제출 패키지](#submission-package--제출-패키지)
13. [Reference Materials / 참고 자료](#reference-materials--참고-자료)
14. [Contributing / 기여](#contributing--기여)
15. [License / 라이선스](#license--라이선스)

---

## Overview / 개요

FSD 대회는 카메라와 LiDAR로 트랙을 인식하고, 노란색/파란색 콘으로 정의된 미니멀 패스 레인을 따라 자율 주행을 수행하며, V2X 인프라(RSU: Road-Side Unit)와 통신해 추가 정보를 받는 종목입니다. 본 스택은 그 파이프라인을 ROS 1 기반 노드들로 구현하며, 대회 규정에 맞는 Docker 이미지 형태로 패키징됩니다.

The Formula Student Driverless competition asks teams to detect the track, follow a cone-defined lane (yellow/blue), and exchange data with a Road-Side Unit (RSU) over V2X — all without a human driver. This stack implements that pipeline as ROS 1 nodes and packages it as a Docker image that satisfies the competition submission rules.

### 핵심 설계 원칙 / Core design principles

- **모듈화 (Modular)** — perception, control, utils, v2x를 독립 패키지로 분리합니다. Each subsystem is its own Python package under `modules/`.
- **설정 외부화 (Configurable)** — 토픽 이름, 차량 파라미터, 콘 감지 임계값은 `params.yaml`에서 변경합니다. Topic names, vehicle parameters, and detector thresholds are externalized.
- **대회 제출 친화적 (Competition-ready)** — `submission/`은 단일 `docker compose up`으로 부팅되는 자급식 패키지입니다. The submission directory boots standalone with one `docker compose up`.
- **실차/시뮬레이션 듀얼 타겟 (Dual target)** — 같은 알고리즘을 시뮬레이터와 실차에서 모두 구동할 수 있습니다. Same algorithm runs in simulator and on real hardware.

---

## Target Users / 대상 사용자

| Audience / 대상 | Use case / 용도 |
|---|---|
| FSD 팀원 (Trackdrive / Skidpad / Autocross) | 콘 레인 자율주행 차량의 백엔드 스택 구성 / Run the autonomous stack on the actual car |
| 대회 심사위원 / 동료 팀 | Docker 제출 패키지의 동작 검증 / Validate a team's Docker submission |
| 자율주행 학습자 | ROS 1 + OpenCV 기반의 미니멀 자율주행 파이프라인 학습 / Learn a minimal ROS 1 + OpenCV pipeline |
| 시뮬레이션 개발자 | LGSVL/ROS 기반 시뮬레이터 환경에서 알고리즘 검증 / Iterate algorithms in sim before track time |

---

## Features / 주요 기능

| 기능 / Feature | 설명 / Description | Entry point |
|---|---|---|
| 콘 감지 (Cone detection) | 카메라/점군에서 콘 후보 추출 / Extract cone candidates from image / point cloud | `modules/perception/cone_detector.py` |
| 콘 분류 (Cone classification) | HSV 기반 노란색/파란색 콘 색상 분류 / Classify cones as yellow or blue | `modules/perception/cone_classifier.py` |
| SLAM | LiDAR + 휠 오도메트리 기반 지도 작성 / Map building from LiDAR + odometry | `modules/perception/slam.py` |
| Pure Pursuit 추종 (Lane following) | 콘 중심선 기반 조향각 산출 / Steering from the cone centerline | `modules/control/pure_pursuit.py` |
| 속도 제어 (Speed control) | 곡률 기반 속도 프로파일 / Curvature-aware speed profile | `modules/control/speed.py` |
| 랩 타이머 (Lap timer) | 출발선 통과 감지 및 랩 타임 기록 / Detect start/finish crossing, log lap time | `modules/utils/lap_timer.py` |
| 워치독 (Watchdog) | 센서/제어 헬스 체크 및 안전 정지 / Health-check & safe stop | `modules/utils/watchdog.py` |
| V2X 어댑터 (V2X adapter) | RSU 메시지 송수신 (제출 패키지 한정) / Talk to the Road-Side Unit (submission only) | `submission/src/v2x/rsu.py` |
| 대회 드라이버 (Competition driver) | 미션 디스패치 및 코스 전환 / Mission dispatch and course switching | `driver/competition_driver.py` |
| Docker 제출 패키지 | 대회 규정 준수 단일 이미지 / Single image compliant with rules | `submission/Dockerfile` |

---

## Architecture / 아키텍처

### 컴포넌트 다이어그램 / Component diagram

| 계층 / Layer | 모듈 / Module | 책임 / Responsibility |
|---|---|---|
| Sensing (입력) | Camera, LiDAR, IMU, GNSS, Wheel Odometry | 토픽 발행 / Publish raw sensor topics |
| Perception | `cone_detector`, `cone_classifier`, `slam` | 콘 위치/색상/지도 산출 / Detect cones, classify, localize |
| V2X | `rsu` | 인프라 메시지 수신 / Receive RSU messages |
| Control | `pure_pursuit`, `speed` | 조향/속도 명령 발행 / Publish `cmd_vel` (steer/speed) |
| Mission | `competition_driver` | 코스 전환, 미션 디스패치 / Dispatch missions |
| Utility | `lap_timer`, `watchdog` | 랩 기록, 안전 보장 / Log laps, enforce safety |
| Output (출력) | VESC, Steering actuator, E-stop | 차량 구동 / Drive the actuators |

### 데이터 흐름 / Data flow

1. **Sensors** publish raw topics (`/camera/image_raw`, `/lidar/points`, `/odom`, …) on ROS 1 bus.
2. **Perception nodes** consume those topics and publish `cones` (yellow/blue positions) and `map` updates.
3. **V2X adapter** receives RSU messages and publishes them as `v2x/*` topics.
4. **Control nodes** subscribe to `cones` + `odom` and publish `cmd_vel` (steer, speed, brake).
5. **Competition driver** subscribes to mission selector + cones + odom to choose between Trackdrive / Skidpad / Autocross logic.
6. **Watchdog** monitors all critical topics; on timeout it commands a safe stop.
7. **Lap timer** consumes `cones` and `odom` to detect start/finish crossings.

---

## Repository Layout / 저장소 구조

```text
/
├── AGENTS.md
├── CONTRIBUTING.md
├── LICENSE
├── OWNERS
├── README.md
├── in-memoria.db
├── src/
│   ├── autonomous/                   # 자율주행 스택 소스 (개발 트리)
│   │   ├── Dockerfile
│   │   ├── docker-compose.yml
│   │   ├── entrypoint.sh
│   │   ├── record_race.sh
│   │   ├── run_all.sh
│   │   ├── start.sh
│   │   ├── scripts/
│   │   │   └── start_race.py
│   │   ├── config/
│   │   │   ├── bridge_no_camera.launch
│   │   │   └── params.yaml
│   │   ├── driver/
│   │   │   └── competition_driver.py
│   │   ├── modules/
│   │   │   ├── perception/
│   │   │   │   ├── cone_classifier.py
│   │   │   │   ├── cone_detector.py
│   │   │   │   └── slam.py
│   │   │   ├── utils/
│   │   │   │   ├── lap_timer.py
│   │   │   │   └── watchdog.py
│   │   │   └── control/
│   │   │       ├── pure_pursuit.py
│   │   │       └── speed.py
│   │   └── tests/
│   │       └── test_algorithms.py
│   └── simulator/                    # 시뮬레이터 자산
│       ├── README.md
│       └── settings.json
├── scripts/
│   └── package.sh                    # Docker 이미지 빌드/패키징
├── docs/
│   ├── SUBMISSION_GUIDE.md
│   └── reference_materials/          # 강의 노트/Jupyter 자료
└── submission/                       # 대회 제출용 자급식 패키지
    ├── AGENTS.md
    ├── Dockerfile
    ├── docker-compose.yml
    ├── run.sh
    ├── dev.sh
    ├── README.md
    ├── launch/
    │   └── competition.launch
    └── src/
        ├── drivers/                  # basic / advanced / autonomous / competition
        ├── perception/               # cone_detector / cone_classifier / slam
        ├── v2x/                      # RSU 어댑터
        ├── utils/                    # lap_timer / watchdog
        └── control/                  # pure_pursuit / speed
```

---

## Quick Start / 빠른 시작

### 사전 요구 사항 / Prerequisites

| 항목 / Item | 버전 / Version |
|---|---|
| Docker Engine | 20.10+ |
| Docker Compose | v2+ |
| Host OS (개발용) | Ubuntu 20.04 / 22.04 |
| (실차) ROS 1 | Noetic |
| (실차) GPU | NVIDIA + nvidia-container-toolkit (선택 / optional) |

### 1. 저장소 클론 / Clone

```bash
git clone <this-repo-url> fsd-stack
cd fsd-stack
```

### 2. 제출 패키지 부팅 (가장 빠름) / Boot the submission package

```bash
cd submission
docker compose up --build
```

내부적으로 `roscore`, `bridge_no_camera.launch`, `competition_driver.py`가 차례로 기동합니다. It brings up `roscore`, the launch file, and the competition driver in order.

### 3. 개발 트리에서 실행 / Run from the dev tree

```bash
cd src/autonomous
docker compose up --build
# 별도 터미널에서:
docker compose exec fsd bash
roslaunch config/bridge_no_camera.launch
rosrun driver competition_driver.py
```

### 4. (시뮬레이터 사용 시) / With simulator

```bash
# 터미널 A: 시뮬레이터 부팅
cd src/simulator
# 시뮬레이터별 실행 방법은 src/simulator/README.md 참조
# Terminal B: 스택 부팅
cd src/autonomous && ./start.sh
```

자세한 트랙 데이 가이드는 `docs/SUBMISSION_GUIDE.md`를 참고하세요. For the full track-day walkthrough, see `docs/SUBMISSION_GUIDE.md`.

---

## Configuration / 설정

### `src/autonomous/config/params.yaml` (예시 키 / example keys)

| 키 / Key | 의미 / Meaning | 기본값 / Default |
|---|---|---|
| `topics.camera` | 입력 이미지 토픽 / Raw image topic | `/camera/image_raw` |
| `topics.lidar` | 입력 LiDAR 토픽 / Raw LiDAR topic | `/lidar/points` |
| `topics.cmd_vel` | 출력 차량 명령 토픽 / Output command topic | `/cmd_vel` |
| `perception.hsv.yellow_h_low` | 노란 콘 Hue 하한 / Yellow cone hue low | `20` |
| `perception.hsv.yellow_h_high` | 노란 콘 Hue 상한 / Yellow cone hue high | `35` |
| `perception.hsv.blue_h_low` | 파란 콘 Hue 하한 / Blue cone hue low | `95` |
| `perception.hsv.blue_h_high` | 파란 콘 Hue 상한 / Blue cone hue high | `130` |
| `control.lookahead` | Pure Pursuit 룩어헤드 거리 [m] / Pure-pursuit lookahead | `1.5` |
| `control.max_speed` | 최대 속도 [m/s] / Max speed | `8.0` |
| `control.min_speed` | 최소 속도 [m/s] / Min speed | `1.5` |
| `watchdog.timeout_s` | 워치독 타임아웃 [s] / Watchdog timeout | `1.0` |

> 실제 키 이름은 저장소의 `params.yaml`을 우선하세요. Always defer to the keys actually present in `params.yaml` of this repo.

### `submission/src/` 패키지 import 경로

| 패키지 / Package | Python import |
|---|---|
| Perception | `from perception.cone_detector import detect_cones` |
| Control | `from control.pure_pursuit import compute_steer` |
| V2X | `from v2x.rsu import RsuClient` |
| Utils | `from utils.watchdog import Watchdog` |

---

## Commands Reference / 명령어 레퍼런스

| 명령어 / Command | 설명 / Description |
|---|---|
| `./src/autonomous/start.sh` | 개발 환경에서 스택 부팅 / Boot the stack in dev mode |
| `./src/autonomous/run_all.sh` | 모든 노드 + 레코더 일괄 실행 / Bring up nodes + recorder |
| `./src/autonomous/record_race.sh` | `rosbag`으로 레코드하면서 주행 / Drive while recording a rosbag |
| `./src/autonomous/scripts/start_race.py` | 미션 셀렉터를 인자로 받아 시작 / Start with a mission selector |
| `cd submission && docker compose up --build` | 대회 제출 이미지 빌드 후 실행 / Build & run submission image |
| `./submission/run.sh` | 제출 컨테이너에서 대회 절차 실행 / Run the competition procedure |
| `./submission/dev.sh` | 제출 컨테이너에 개발 셸 진입 / Drop into a dev shell inside the submission container |
| `./scripts/package.sh` | 대회 규정용 단일 이미지 패키징 / Package a single image per competition rules |
| `roslaunch config/bridge_no_camera.launch` | ROS ↔ 시뮬레이터 브리지 / Bridge ROS ↔ simulator |
| `roslaunch launch/competition.launch` | 제출 패키지 런치 / Submission launch file |

---

## Local Development / 로컬 개발

### 개발 워크플로우 / Workflow

1. `src/autonomous/`에서 알고리즘을 수정합니다. Edit algorithms under `src/autonomous/`.
2. `src/autonomous/tests/test_algorithms.py`로 단위 테스트를 실행합니다. Run unit tests.
3. `src/simulator/`와 연동해 회귀를 확인합니다. Regress against `src/simulator/`.
4. 안정화되면 동일 코드를 `submission/src/`에 동기화합니다. Once stable, mirror the change into `submission/src/`.
5. `submission/`에서 `docker compose up --build`로 대회 환경 그대로 검증합니다. Validate with `docker compose up --build`.

### 컨테이너 개발 셸 / Container dev shell

```bash
cd submission
./dev.sh
# 컨테이너 내부:
roslaunch launch/competition.launch
rosrun drivers competition.py
```

### 시뮬레이터 ↔ ROS 브리지 / Simulator ↔ ROS bridge

`src/autonomous/config/bridge_no_camera.launch`는 카메라/점군/오도메트리 토픽을 시뮬레이터와 ROS 1 사이에 연결합니다. 카메라가 없는 실차 검증 시 동일한 런치를 재사용할 수 있습니다. The bridge launch wires simulator topics to ROS 1 and is reused for camera-less hardware validation.

---

## Testing / 테스트

| 테스트 / Test | 위치 / Location | 실행 / How to run |
|---|---|---|
| 알고리즘 단위 테스트 | `src/autonomous/tests/test_algorithms.py` | `cd src/autonomous && python -m pytest tests/` |
| 통합(시뮬) | `src/simulator/` + `src/autonomous/run_all.sh` | `cd src/autonomous && ./run_all.sh` |
| 제출 패키지 회귀 | `submission/` | `cd submission && docker compose up --build` |

테스트는 콘 분류의 정확도, Pure Pursuit 추종 오차, 워치독 타임아웃 동작을 검증합니다. Tests cover classifier accuracy, pursuit tracking error, and watchdog timeout behavior.

---

## Simulation / 시뮬레이션

`src/simulator/`는 시뮬레이터 자산과 환경 설정을 보관합니다. 자세한 시뮬레이터 조작법은 `src/simulator/README.md`와 `docs/reference_materials/`의 Jupyter 노트북을 참고하세요. See `src/simulator/README.md` and the Jupyter notebooks under `docs/reference_materials/` for simulator usage.

---

## Submission Package / 제출 패키지

`submission/`은 대회 규정에서 요구하는 "단일 부팅 가능한 Docker 이미지" 형태의 자급식 패키지입니다. 별도 빌드 도구나 외부 의존 없이 `docker compose up`만으로 전체 파이프라인이 실행됩니다. The submission package is a self-contained Docker image that boots the full pipeline with a single `docker compose up`.

| 항목 / Item | 위치 / Path |
|---|---|
| Dockerfile | `submission/Dockerfile` |
| Compose | `submission/docker-compose.yml` |
| 런치 / Launch | `submission/launch/competition.launch` |
| 드라이버 / Drivers | `submission/src/drivers/{basic,advanced,autonomous,competition}.py` |
| V2X (RSU) | `submission/src/v2x/rsu.py` |
| 실행 진입점 / Entry scripts | `submission/run.sh`, `submission/dev.sh` |

> 대회마다 패키징 세부 규칙이 다를 수 있으므로 `docs/SUBMISSION_GUIDE.md`를 반드시 확인하세요. Always read `docs/SUBMISSION_GUIDE.md` for the latest rules per event.

---

## Reference Materials / 참고 자료

| 자료 / Material | 경로 / Path | 용도 / Use |
|---|---|---|
| FSDS 설치 강의 노트 | `docs/reference_materials/lecture1_fsds_install.txt` | FSDS 시뮬레이터 설치 / Install FSDS |
| SLAM 강의 노트북 | `docs/reference_materials/lecture4_slam.ipynb` | SLAM 이론 + 실습 / SLAM theory & practice |
| V2X 강의 노트북 | `docs/reference_materials/lecture6_v2x.ipynb` | V2X/RSU 연동 / V2X integration |
| 제출 가이드 | `docs/SUBMISSION_GUIDE.md` | 대회 제출 절차 / Submission procedure |

---

## Contributing / 기여

기여 절차는 `CONTRIBUTING.md`를 참고하세요. For the contribution procedure, see `CONTRIBUTING.md`. 저장소 메인테이너는 `OWNERS` 파일에 명시되어 있습니다. Repository maintainers are listed in `OWNERS`.

### 권장 브랜치 전략 / Branching

- `master` — 안정화 버전, 대회 제출용 / Stable, competition-ready
- `feat/<topic>` — 기능 단위 작업 / Per-feature work
- `fix/<topic>` — 버그 수정 / Bug fix
- `sim/<topic>` — 시뮬레이션 전용 / Simulator-only

### 커밋 메시지 / Commit messages

Conventional Commits (`feat:`, `fix:`, `docs:`, `test:`, `refactor:`)를 권장합니다. Conventional Commits recommended.

---

## License / 라이선스

`LICENSE` 파일을 참고하세요. See the `LICENSE` file for the full text.