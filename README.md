# Formula Student Driverless — Autonomous Racing Stack

> Formula Student Driverless (FSD) 자율주행 레이싱 소프트웨어 스택
> Autonomous racing software stack for the Formula Student Driverless competition

이 저장소는 FSD(Formula Student Driverless) 대회를 위한 완전 자율주행 레이싱 차량 소프트웨어를 제공합니다. SLAM, 콘(코너링 마커) 감지/분류, Pure Pursuit 추종 제어, 랩 타이머, V2X(차량-사물 통신) 어댑터, 그리고 대회용 Docker 제출 패키지를 포함합니다.

This repository provides a full-stack autonomous driving system for the Formula Student Driverless competition. It bundles SLAM, cone detection/classification, pure-pursuit lane following, lap timing, a V2X adapter, and a competition-ready Docker submission package.

---

## Table of Contents / 목차

1. [Overview / 개요](#overview--개요)
2. [Features / 주요 기능](#features--주요-기능)
3. [Architecture / 아키텍처](#architecture--아키텍처)
4. [Repository Layout / 저장소 구조](#repository-layout--저장소-구조)
5. [Quick Start / 빠른 시작](#quick-start--빠른-시작)
6. [Configuration / 설정](#configuration--설정)
7. [Commands Reference / 명령어 레퍼런스](#commands-reference--명령어-레퍼런스)
8. [Local Development / 로컬 개발](#local-development--로컬-개발)
9. [Testing / 테스트](#testing--테스트)
10. [Simulation / 시뮬레이션](#simulation--시뮬레이션)
11. [Submission Package / 제출 패키지](#submission-package--제출-패키지)
12. [Contributing / 기여](#contributing--기여)
13. [License / 라이선스](#license--라이선스)

---

## Overview / 개요

FSD 대회는 카메라와 LiDAR로 트랙을 인식하고, 미니멀 패스(노란색/파란색 콘)로 정의된 레인을 따라 자율 주행을 수행하며, V2X 인프라(RSU)와 통신해 추가 정보를 받는 종목입니다. 본 스택은 그 파이프라인을 ROS 1 기반 노드들로 구현합니다.

The Formula Student Driverless competition asks teams to detect the track, follow a cone-defined lane (yellow/blue), and exchange data with a Roadside Unit (RSU) over V2X — all without a human driver. This stack implements that pipeline as a set of ROS 1 nodes.

대상 사용자 / Target users:

- FSD 대회에 참가하는 팀의 자율주행 SW 엔지니어
- Autonomous-driving SW engineers on FSD teams
- SLAM / computer vision / control 알고리즘을 실제 차량에 배포하려는 연구자
- Researchers who want to deploy SLAM / CV / control algorithms on a real race car
- 본 코드를 베이스라인으로 포크해 팀 고유 알고리즘을 시험하려는 학생 팀
- Student teams forking the baseline to test their own algorithms

엔트리 포인트 / Entry points:

- Development runtime: `src/autonomous/start.sh`, `src/autonomous/run_all.sh`
- Competition submission: `submission/run.sh`, `submission/dev.sh`
- Launch file (ROS): `submission/launch/competition.launch`
- Competition driver: `submission/src/drivers/competition.py`, `src/autonomous/driver/competition_driver.py`

---

## Features / 주요 기능

- **Cone Detection** — 카메라 영상에서 콘을 감지 (`modules/perception/cone_detector.py`, `submission/src/perception/cone_detector.py`).
- **Cone Classification** — 노란색/파란색 콘을 분류해 좌/우 레인 경계를 결정 (`modules/perception/cone_classifier.py`).
- **SLAM** — LiDAR + IMU 기반 동시 위치추정 및 지도작성 (`modules/perception/slam.py`).
- **Pure Pursuit Control** — 기하학 기반 조향 추종 (`modules/control/pure_pursuit.py`).
- **Speed Control** — 곡률 기반 속도 계획 (`modules/control/speed.py`).
- **Lap Timer** — Start/Finish 라인 통과 시 랩 타임 기록 (`modules/utils/lap_timer.py`).
- **Watchdog** — 비정상 상태 감시 및 안전 정지 (`modules/utils/watchdog.py`).
- **V2X / RSU** — 로드사이드 유닛과의 통신 어댑터 (`submission/src/v2x/rsu.py`).
- **Multiple Driver Modes** — `basic`, `advanced`, `autonomous`, `competition` 4단계 드라이버 (`submission/src/drivers/`).
- **Competition Driver** — 대회 규정(FSG Driverless)에 맞춘 통합 드라이버 (`src/autonomous/driver/competition_driver.py`).
- **Dockerized Runtime** — 개발용과 제출용 분리된 Docker 이미지.
- **Head-to-head Race Support** — `start_race.py`로 다중 차량 레이스 시퀀스 실행.
- **Simulator Bridge** — 카메라이 없는 환경에서 시뮬레이터와 연동 (`config/bridge_no_camera.launch`).

---

## Architecture / 아키텍처

자율주행 파이프라인은 일반적으로 `Perception → Localization → Planning → Control → Actuation` 흐름을 따릅니다. 본 스택은 이를 ROS 토픽으로 연결합니다.

The autonomous driving pipeline follows a standard `Perception → Localization → Planning → Control → Actuation` flow, wired together through ROS topics.

```mermaid
flowchart LR
    subgraph Sensors
        CAM["Camera<br/>/camera/image_raw"]
        LIDAR["LiDAR<br/>/scan"]
        IMU["IMU<br/>/imu"]
    end

    subgraph Perception
        CD["cone_detector"]
        CC["cone_classifier"]
        SLAM["slam"]
    end

    subgraph Planning
        PP["pure_pursuit"]
        SP["speed"]
    end

    subgraph Driver
        BD["drivers/basic"]
        AD["drivers/advanced"]
        AU["drivers/autonomous"]
        CD2["drivers/competition"]
    end

    subgraph V2X
        RSU["v2x/rsu"]
    end

    subgraph Vehicle
        VESC["VESC<br/>/commands/motor &amp; /commands/servo"]
    end

    subgraph Utils
        LT["lap_timer"]
        WD["watchdog"]
    end

    CAM --&gt; CD
    CAM --&gt; CC
    LIDAR --&gt; SLAM
    IMU --&gt; SLAM
    CD --&gt; CC
    CC --&gt; PP
    SLAM --&gt; PP
    CC --&gt; SP
    PP --&gt; CD2
    SP --&gt; CD2
    CD2 --&gt; VESC
    RSU --&gt; CD2
    LT --&gt; CD2
    WD --&gt; VESC
    BD -. fallback .-&gt; VESC
    AD -. fallback .-&gt; VESC
    AU -. fallback .-&gt; VESC
```

핵심 토픽 흐름 / Core topic flow:

1. `cone_detector`가 카메라 입력에서 콘 후보를 추출.
2. `cone_classifier`가 색상 분류 후 좌/우 레인 경계 발행.
3. `slam`이 LiDAR/IMU로 차량 위치 및 콘 지도 발행.
4. `pure_pursuit`이 중심선과 곡률을 계산.
5. `speed`가 곡률 기반 목표 속도 계산.
6. `drivers/competition`이 모든 입력을 합쳐 VESC 모터/서보 명령 발행.
7. `watchdog`이 이상 상태를 감지하면 즉시 안전 정지.
8. `lap_timer`가 랩 타임 기록.
9. `v2x/rsu`가 외부 인프라 정보를 받아 의사결정에 반영.

---

## Repository Layout / 저장소 구조

```
/
├── AGENTS.md
├── CONTRIBUTING.md
├── LICENSE
├── OWNERS
├── README.md
├── in-memoria.db              # 로컬 학습/메모리 캐시 (개발용)
├── src/
│   ├── autonomous/            # 활성 개발 브랜치
│   │   ├── AGENTS.md
│   │   ├── Dockerfile
│   │   ├── docker-compose.yml
│   │   ├── entrypoint.sh
│   │   ├── record_race.sh
│   │   ├── run_all.sh
│   │   ├── start.sh
│   │   ├── scripts/start_race.py
│   │   ├── config/
│   │   │   ├── bridge_no_camera.launch
│   │   │   └── params.yaml
│   │   ├── driver/competition_driver.py
│   │   ├── modules/
│   │   │   ├── perception/   # cone_detector, cone_classifier, slam
│   │   │   ├── control/      # pure_pursuit, speed
│   │   │   └── utils/        # lap_timer, watchdog
│   │   └── tests/test_algorithms.py
│   └── simulator/             # 시뮬레이터 설정
│       ├── README.md
│       └── settings.json
├── scripts/
│   └── package.sh             # 제출 패키지 빌드 스크립트
├── docs/
│   ├── SUBMISSION_GUIDE.md
│   └── reference_materials/   # 강의 노트/노트북 (FSDS 설치, SLAM, V2X)
├── submission/                # 대회 제출용 패키지
│   ├── AGENTS.md
│   ├── Dockerfile
│   ├── README.md
│   ├── dev.sh
│   ├── docker-compose.yml
│   ├── run.sh
│   ├── launch/competition.launch
│   └── src/
│       ├── drivers/           # basic, advanced, autonomous, competition
│       ├── perception/        # cone_detector, cone_classifier, slam
│       ├── control/           # pure_pursuit, speed
│       ├── v2x/rsu.py
│       ├── utils/             # lap_timer, watchdog
│       └── autonomous/        # 제출용 자율 패키지 미러
│           ├── Dockerfile
│           ├── docker-compose.yml
│           ├── entrypoint.sh
│           ├── run_all.sh
│           ├── start.sh
│           ├── config/params.yaml
│           ├── driver/competition_driver.py
│           └── modules/perception/...
```

`src/autonomous/`는 활발한 개발을 위한 워크스페이스이고, `submission/`은 대회 규격에 맞게 패키징된 변형입니다. 두 디렉터리 모두 동일한 모듈 구조(`perception`, `control`, `v2x`, `utils`, `drivers`)를 따릅니다.

`src/autonomous/` is the active development workspace; `submission/` is a competition-packaged variant. Both share the same module layout (`perception`, `control`, `v2x`, `utils`, `drivers`).

---

## Quick Start / 빠른 시작

### Prerequisites / 사전 요구사항

- Docker 20.10+ and Docker Compose v2
- 호스트가 Linux이고 USB 시리얼/카메라 디바이스 접근 가능
- (선택) ROS 1 Noetic — 컨테이너 외부에서 토픽을 디버깅하려는 경우
- (Optional) ROS 1 Noetic — only for host-side topic inspection

### 1) Clone / 클론

```bash
git clone <your-fork-url> fsd-stack
cd fsd-stack
```

### 2) Run the autonomous stack / 자율주행 스택 실행

개발 워크스페이스:

```bash
cd src/autonomous
docker compose up --build
```

또는 호스트에서 직접:

```bash
cd src/autonomous
./start.sh           # 단일 런
./run_all.sh         # 전체 파이프라인 (추천)
```

### 3) Run the competition submission / 대회 제출 패키지 실행

```bash
cd submission
./dev.sh             # 개발 모드 (디버깅 로그 ON)
./run.sh             # 대회 모드 (대회 규정 준수)
```

### 4) Record a race / 레이스 기록

```bash
cd src/autonomous
./record_race.sh
# 또는 다중 차량 레이스:
python3 scripts/start_race.py
```

`record_race.sh`는 ROS bag을 저장하고, `start_race.py`는 head-to-head 레이스 시퀀스를 트리거합니다.

---

## Configuration / 설정

주요 설정 파일은 YAML 형식입니다. ROS 파라미터 서버에서 로드됩니다.

| File / 파일 | Purpose / 용도 |
|------------|---------------|
| `src/autonomous/config/params.yaml` | 개발 워크스페이스의 모든 노드 파라미터 |
| `submission/autonomous/config/params.yaml` | 제출 패키지용 파라미터 미러 |
| `simulator/settings.json` | 시뮬레이터 트랙/차량/센서 설정 |
| `submission/launch/competition.launch` | 대회 런치 (토픽, 노드, 리매핑) |
| `src/autonomous/config/bridge_no_camera.launch` | 카메라가 없을 때 시뮬레이터로 브릿지 |

예시 파라미터 키 / Example keys (representative — 실제 키는 `params.yaml` 참조):

```yaml
perception:
  cone_detector:
    model_path: /opt/fsd/models/yolov8n-cone.engine
    conf_threshold: 0.35
    input_topic: /camera/image_raw
  cone_classifier:
    yellow_hsv_lower: [20, 100, 100]
    yellow_hsv_upper: [35, 255, 255]
control:
  pure_pursuit:
    lookahead_m: 2.5
    wheelbase_m: 1.55
  speed:
    max_speed_mps: 12.0
    min_speed_mps: 2.0
    curvature_gain: 4.0
safety:
  watchdog:
    heartbeat_timeout_ms: 250
    e_stop_topic: /emergency_stop
v2x:
  rsu:
    host: <rsu-host>
    port: 5555
    enabled: true
```

> 실제 키 이름은 사용 중인 모델 버전과 대회 규정 버전에 따라 달라질 수 있습니다. 반드시 `params.yaml`을 직접 확인하세요.
> Actual key names depend on the model and rulebook version. Always verify against `params.yaml`.

---

## Commands Reference / 명령어 레퍼런스

### Autonomous development / 자율주행 개발

| Script / 스크립트 | Description / 설명 |
|------------------|-------------------|
| `src/autonomous/start.sh` | 메인 런치 실행 (단일 실행) |
| `src/autonomous/run_all.sh` | 전체 파이프라인 실행 (추천) |
| `src/autonomous/record_race.sh` | ROS bag 녹화 모드로 실행 |
| `src/autonomous/scripts/start_race.py` | head-to-head 레이스 시퀀스 시작 |
| `src/autonomous/entrypoint.sh` | 컨테이너 진입 시 자동 실행 |

### Submission / 제출 패키지

| Script / 스크립트 | Description / 설명 |
|------------------|-------------------|
| `submission/run.sh` | 대회 모드 런 |
| `submission/dev.sh` | 개발 모드 런 (verbose 로그) |
| `scripts/package.sh` | 제출용 아카이브 빌드 |

### Docker Compose / Docker Compose

```bash
# 개발 워크스페이스
cd src/autonomous
docker compose up --build            # 포어그라운드
docker compose up --build -d         # 백그라운드
docker compose logs -f autonomous    # 로그 스트림
docker compose down                  # 정리

# 제출 패키지
cd submission
docker compose up --build
```

### ROS introspection / ROS 디버깅 (호스트에서)

```bash
docker compose exec autonomous bash
source /opt/ros/noetic/setup.bash
rosnode list
rostopic list
rostopic echo /vesc/odom
rqt_graph
```

---

## Local Development / 로컬 개발

### Recommended workflow / 권장 워크플로

1. `src/autonomous/`에서 알고리즘을 수정합니다.
2. `tests/test_algorithms.py`로 단위 테스트를 실행합니다.
3. `record_race.sh`로 bag을 수집해 시뮬레이터에서 회귀 테스트를 수행합니다.
4. 안정화된 변경만 `submission/`으로 동기화합니다 (`scripts/package.sh`로 패키징).

1. Modify algorithms under `src/autonomous/`.
2. Run unit tests via `tests/test_algorithms.py`.
3. Record bags with `record_race.sh` and replay in the simulator.
4. Once stable, sync to `submission/` and run `scripts/package.sh` to build a competition archive.

### Code conventions / 코드 컨벤션

- Python 3, PEP 8, 4-space indent.
- ROS 노드는 `if __name__ == "__main__"` 패턴 + `rospy.init_node(...)`.
- 새 토픽은 `params.yaml`에 명시적으로 등록.
- 안전(safety) 관련 코드는 `watchdog`와 반드시 연동.

### Adding a new driver mode / 새 드라이버 모드 추가

1. `submission/src/drivers/`에 `<name>.py` 생성.
2. `BaseDriver` 또는 기존 `basic.py`를 상속.
3. `submission/launch/competition.launch`에 노드 등록.
4. `params.yaml`에 토픽/파라미터 추가.
5. `tests/`에 단위 테스트 추가.

---

## Testing / 테스트

```bash
cd src/autonomous
python3 -m pytest tests/ -v
```

또는 ROS 환경 안에서:

```bash
cd src/autonomous
docker compose run --rm autonomous bash -lc "source /opt/ros/noetic/setup.bash && python3 -m pytest tests/ -v"
```

테스트는 알고리즘 단위(perception, control, utils)를 다룹니다. 통합 테스트는 시뮬레이터에서 ROS bag을 재생해 수행합니다.

The test suite covers unit-level algorithms (perception, control, utils). Integration tests replay recorded ROS bags through the simulator.

---

## Simulation / 시뮬레이션

`src/simulator/`는 트랙, 차량 동역학, 센서 모델을 정의합니다.

The `src/simulator/` directory defines the track, vehicle dynamics, and sensor models.

```bash
# 시뮬레이터 설정 확인
cat src/simulator/settings.json

# 카메라 없이 시뮬레이터만으로 실행
roslaunch src/autonomous/config/bridge_no_camera.launch
```

자세한 내용은 `src/simulator/README.md`를 참조하세요.

See `src/simulator/README.md` for details.

---

## Submission Package / 제출 패키지

`submission/`은 대회 운영 측에 제출하는 형태 그대로 패키징된 디렉터리입니다. ROS 워크스페이스, Docker 설정, 런치 파일, 그리고 소스 트리 미러가 포함됩니다.

The `submission/` directory is the ready-to-submit variant, containing a ROS workspace, Docker setup, launch files, and a mirrored source tree.

자세한 절차는 `docs/SUBMISSION_GUIDE.md`를 참조하세요.

For step-by-step instructions, see `docs/SUBMISSION_GUIDE.md`.

요약 / Summary:

1. `submission/dev.sh`로 로컬에서 충분히 시험.
2. `scripts/package.sh`로 아카이브 생성.
3. `docs/SUBMISSION_GUIDE.md`의 체크리스트에 따라 제출.

---

## Contributing / 기여

기여 절차는 [`CONTRIBUTING.md`](CONTRIBUTING.md)를 따릅니다. 큰 변경 전에는 `AGENTS.md`와 `submission/AGENTS.md`를 먼저 확인하세요.

Please follow [`CONTRIBUTING.md`](CONTRIBUTING.md). Read `AGENTS.md` and `submission/AGENTS.md` before opening large changes.

기본 원칙 / Ground rules:

- Pull request 한 개당 한 가지 변경.
- 새 기능은 `src/autonomous/`에서 먼저 개발, 검증 후 `submission/`에 동기화.
- 안전(safety)에 영향을 주는 변경은 별도 라벨과 리뷰어 1인 이상 필요.
- 대회 규정(FSG Driverless Rulebook) 위반 여부를 PR 본문에 명시.

---

## Reference Materials / 참고 자료

`docs/reference_materials/`에 학습용 자료가 있습니다:

- `lecture1_fsds_install.txt` — FSDS(FSD Simulator) 설치 가이드
- `lecture4_slam.ipynb` — SLAM 강의 노트북
- `lecture6_v2x.ipynb` — V2X 강의 노트북

`docs/reference_materials/` contains educational material:

- `lecture1_fsds_install.txt` — FSD Simulator install guide
- `lecture4_slam.ipynb` — SLAM lecture notebook
- `lecture6_v2x.ipynb` — V2X lecture notebook

---

## License / 라이선스

본 저장소는 [`LICENSE`](LICENSE) 파일에 명시된 라이선스를 따릅니다. 대회에 제출하는 경우, 외부 코드/모델의 라이선스 호환성을 PR 본문에 명시해야 합니다.

This repository is distributed under the license stated in [`LICENSE`](LICENSE). When submitting to a competition, you must declare the license compatibility of any third-party code or models in the PR body.

---

## Maintainers / 유지보수

- 코드 오너십은 [`OWNERS`](OWNERS) 파일을 참조.
- See [`OWNERS`](OWNERS) for current code ownership.