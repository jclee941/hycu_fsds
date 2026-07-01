# Formula Student Driverless — Autonomous Racing Stack

> **FSD (Formula Student Driverless) 자율주행 레이싱 소프트웨어 스택**
> Autonomous racing software stack for the Formula Student Driverless competition

이 저장소는 FSD 대회를 위한 완전 자율주행 레이싱 차량 소프트웨어를 제공합니다. SLAM, 콘 감지/분류, Pure Pursuit 추종 제어, 랩 타이머, V2X(차량-사물 통신) 어댑터, 그리고 대회용 Docker 제출 패키지를 포함합니다.

This repository provides a full-stack autonomous driving system for the Formula Student Driverless competition. It bundles SLAM, cone detection/classification, pure-pursuit lane following, lap timing, a V2X adapter, and a competition-ready Docker submission package.

---

## Table of Contents / 목차

1. [Overview / 개요](#overview--개요)
2. [Target Users / 대상 사용자](#target-users--대상-사용자)
3. [Features / 주요 기능](#features--주요-기능)
4. [Architecture / 아키텍처](#architecture--아키텍처)
5. [Repository Layout / 저장소 구조](#repository-layout--저소-구조)
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

FSD 대회는 카메라와 LiDAR로 트랙을 인식하고, 노란색/파란색 콘으로 정의된 미니멀 패스 레인을 따라 자율 주행을 수행하며, V2X 인프라(RSU)와 통신해 추가 정보를 받는 종목입니다. 본 스택은 그 파이프라인을 ROS 1 기반 노드들로 구현하며, 대회 규정에 맞는 Docker 이미지 형태로 패키징됩니다.

The Formula Student Driverless competition asks teams to detect the track, follow a cone-defined lane (yellow/blue), and exchange data with a Roadside Unit (RSU) over V2X — all without a human driver. This stack implements that pipeline as ROS 1 nodes and packages it as a Docker image that satisfies competition submission rules.

### 핵심 설계 원칙 / Core design principles

- **모듈화 (Modular)** — perception, control, utils, v2x를 독립 패키지로 분리하여 단독 테스트 및 교체 가능.
- **대회 규정 준수 (Competition-compliant)** — 제출 패키지는 단일 `docker compose up` 명령으로 부팅 가능한 단일 컨테이너 구조.
- **시뮬레이션 친화 (Simulation-friendly)** — 실제 차량과 시뮬레이터 모두에서 동일 노드 구동이 가능하도록 launch 파일과 driver 추상화.
- **투명성 (Transparent)** — 랩 타이머, 워치독, 의사결정 로그를 외부에서 관측 가능.

---

## Target Users / 대상 사용자

| Audience / 대상 | Use case / 사용 시나리오 |
|---|---|
| Formula Student Driverless 팀 (학생) | 대회 출전용 자율주행 스택 빌드/튜닝/제출 |
| 대학 캡스톤 / ROS 교육 과정 | SLAM·Pure Pursuit·V2X 실습 예제 |
| 시뮬레이션 연구자 | `src/simulator` 환경에서 알고리즘 사전 검증 |
| 대회 심사위원 (FSCC/FS-AI) | 제출 패키지 재현성 및 동작 검증 |

---

## Features / 주요 기능

| Module / 모듈 | 책임 / Responsibility | Entry point / 진입점 |
|---|---|---|
| Perception / 인지 | 카메라/LiDAR 콘 감지·분류, SLAM | `src/autonomous/modules/perception/` |
| Control / 제어 | Pure Pursuit 경로 추종, 속도 결정 | `src/autonomous/modules/control/` |
| Utils / 유틸 | 랩 타이머, 워치독 | `src/autonomous/modules/utils/` |
| Driver / 드라이버 | 대회 운영 흐름(state machine) | `src/autonomous/driver/competition_driver.py` |
| V2X / 차량-사물 통신 | RSU 데이터 송수신 어댑터 | `submission/src/v2x/rsu.py` |
| Submission / 제출 | 단일 컨테이너 대회 제출 패키지 | `submission/Dockerfile`, `submission/docker-compose.yml` |

상위 레벨 기능 요약 / High-level feature highlights:

- ROS 1 launch 기반 부팅 (`bridge_no_camera.launch`, `competition.launch`).
- 노란/파란 콘의 색상 기반 미니멀 패스 라인 추정.
- Pure Pursuit 추종 + 곡률 기반 속도 계획.
- RSU 메시지 송수신을 위한 V2X 어댑터.
- Lap timer / watchdog로 대회 운영 중 안전 상태 유지.
- 시뮬레이터(`src/simulator`)와 실제 차량 동일한 launch 시퀀스.

---

## Architecture / 아키텍처

본 스택은 크게 **autonomous 노드 셋**과 **submission 패키지** 두 개의 배포 산출물로 구성됩니다. 두 패키지는 동일한 모듈 디렉터리 레이아웃(`perception / control / utils / v2x`)을 공유합니다.

### 컴포넌트 매핑 / Component map

| Layer / 계층 | ROS package / ROS 패키지 | 역할 / Role |
|---|---|---|
| Driver | `drivers/competition.py` | 대회 state machine (FS-AI: AS, EB, FS, FT, …) |
| Driver | `drivers/autonomous.py` / `basic.py` / `advanced.py` | 단계별 자율주행 프로파일 |
| Perception | `perception/cone_detector.py` | 카메라/LiDAR 콘 위치 추정 |
| Perception | `perception/cone_classifier.py` | 노란/파란 색상 분류 |
| Perception | `perception/slam.py` | SLAM 기반 차량 위치 추정 |
| Control | `control/pure_pursuit.py` | Pure Pursuit 추종 |
| Control | `control/speed.py` | 곡률 기반 속도 계획 |
| V2X | `v2x/rsu.py` | RSU 메시지 송수신 |
| Utils | `utils/lap_timer.py` | 랩 측정 |
| Utils | `utils/watchdog.py` | 상태 워치독 |

### 런타임 요청 흐름 / Runtime request flow

1. `entrypoint.sh`가 ROS 마스터 부팅 → `competition.launch` 실행.
2. `competition_driver.py`가 대회 state를 초기화(`AS`).
3. `cone_detector`가 카메라/시뮬레이터 토픽 수신 → 콘 위치 publish.
4. `cone_classifier`가 색상 라벨 부착.
5. `slam`이 차량 위치(`/odom`)를 publish.
6. `pure_pursuit` + `speed`가 콘 중심선과 곡률로부터 `/cmd_vel` 산출.
7. `v2x/rsu`가 RSU로부터 신호 수신 시 driver에 이벤트 전달.
8. `lap_timer`가 lap time 측정, `watchdog`가 비정상 상태 감지 시 EB 전이.

### 빌드 산출물 / Build artifacts

| Artifact / 산출물 | 위치 | 비고 |
|---|---|---|
| 메인 자율주행 이미지 | `src/autonomous/Dockerfile` | 개발·테스트용 |
| 대회 제출 이미지 | `submission/Dockerfile` | 단일 컨테이너, FS 규격 |
| Compose 정의 | `submission/docker-compose.yml`, `src/autonomous/docker-compose.yml` | |
| Launch 정의 | `submission/launch/competition.launch`, `src/autonomous/config/bridge_no_camera.launch` | |

---

## Repository Layout / 저장소 구조

```text
.
├── AGENTS.md
├── CONTRIBUTING.md
├── LICENSE
├── OWNERS
├── README.md
├── in-memoria.db
├── scripts/
│   └── package.sh
├── docs/
│   ├── SUBMISSION_GUIDE.md
│   └── reference_materials/
│       ├── lecture1_fsds_install.txt
│       ├── lecture4_slam.ipynb
│       └── lecture6_v2x.ipynb
├── src/
│   ├── autonomous/
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
│   │   │   ├── perception/ (cone_classifier, cone_detector, slam)
│   │   │   ├── control/    (pure_pursuit, speed)
│   │   │   └── utils/      (lap_timer, watchdog)
│   │   └── tests/test_algorithms.py
│   └── simulator/
│       ├── README.md
│       └── settings.json
└── submission/
    ├── AGENTS.md
    ├── Dockerfile
    ├── README.md
    ├── dev.sh
    ├── docker-compose.yml
    ├── run.sh
    ├── launch/competition.launch
    ├── src/
    │   ├── drivers/      (advanced, autonomous, basic, competition)
    │   ├── perception/   (cone_classifier, cone_detector, slam)
    │   ├── v2x/rsu.py
    │   ├── utils/        (lap_timer, watchdog)
    │   └── control/      (pure_pursuit, speed)
    └── autonomous/
        ├── Dockerfile
        ├── docker-compose.yml
        ├── entrypoint.sh
        ├── run_all.sh
        ├── start.sh
        ├── config/params.yaml
        ├── driver/competition_driver.py
        └── modules/
            ├── perception/ (cone_classifier, cone_detector)
            └── (…)
```

---

## Quick Start / 빠른 시작

### 사전 요구사항 / Prerequisites

- Docker 20.10+ 및 Docker Compose v2
- (시뮬레이션 실행 시) FSDS 또는 호환 Gazebo 환경
- 8GB 이상 RAM, NVIDIA GPU 권장

### 대회 제출 패키지 실행 / Run the submission package

```bash
cd submission
docker compose up --build
```

### 개발용 자율주행 이미지 실행 / Run the dev autonomous image

```bash
cd src/autonomous
docker compose up --build
```

### 시뮬레이터에서 부팅 / Boot in simulator

1. `src/simulator/settings.json`을 FSDS 클라이언트에 로드.
2. 별도 터미널에서 `src/autonomous` 이미지 부팅 → `/cmd_vel`을 시뮬레이터로 라우팅.

---

## Configuration / 설정

| File / 파일 | 용도 / Purpose |
|---|---|
| `src/autonomous/config/params.yaml` | 인지/제어 노드 파라미터 (콘 임계값, 속도 한계 등) |
| `src/autonomous/config/bridge_no_camera.launch` | 카메라 없이 시뮬 데이터만 사용하는 launch |
| `submission/launch/competition.launch` | 대회 제출 시 단일 컨테이너 launch 정의 |
| `src/simulator/settings.json` | FSDS 트랙·차량 설정 |
| `submission/src/drivers/competition.py` (내부 상수) | FS-AI 상태 머신 상수 |

`params.yaml`에서 주로 조정되는 키:

| Key | 의미 / Meaning | 기본값 권장 |
|---|---|---|
| `cone.detection.threshold` | 콘 감지 신뢰도 임계값 | 0.5 |
| `cone.classifier.yellow_hsv` | 노란 콘 HSV 범위 | 튜닝 필요 |
| `control.pure_pursuit.lookahead` | 전방 주시 거리(m) | 1.5 |
| `control.speed.max_velocity` | 최대 속도(m/s) | 대회 규정에 따름 |
| `v2x.rsu.endpoint` | RSU 주소 | 시뮬 환경에서는 localhost |

---

## Commands Reference / 명령어 레퍼런스

| Command | 설명 / Description |
|---|---|
| `cd submission && docker compose up --build` | 대회 제출 스택 부팅 |
| `cd src/autonomous && docker compose up --build` | 개발용 자율주행 스택 부팅 |
| `bash submission/run.sh` | 제출 컨테이너 내부에서 실행되는 메인 스크립트 |
| `bash src/autonomous/start.sh` | 자율주행 컨테이너 부팅 스크립트 |
| `bash src/autonomous/run_all.sh` | 전체 노드 일괄 실행 |
| `bash src/autonomous/record_race.sh` | 레이스 로그 기록 (`*.bag`) |
| `bash src/autonomous/scripts/start_race.py` | Race orchestrator (Python 진입점) |
| `bash submission/dev.sh` | 제출 컨테이너를 개발 모드로 진입 |
| `bash scripts/package.sh` | 대회 제출용 tar/이미지 패키징 |

---

## Local Development / 로컬 개발

### 컨테이너 내부 진입 / Enter the container

```bash
docker compose -f src/autonomous/docker-compose.yml run --rm autonomous bash
```

### 노드 단독 실행 / Run a single node

```bash
source /opt/ros/<distro>/setup.bash
rosrun perception cone_detector _params:=src/autonomous/config/params.yaml
```

### launch 디버깅 / Launch debugging

```bash
roslaunch src/autonomous/config/bridge_no_camera.launch --screen
```

### 로그 확인 / Inspect logs

- 컨테이너 로그: `docker compose logs -f`
- 토픽 모니터링: `rqt` 또는 `rostopic list / echo`

### 알고리즘 변경 절차 / Changing an algorithm

1. `src/autonomous/modules/<area>/<file>.py` 수정.
2. `src/autonomous/tests/test_algorithms.py`에서 단위 검증.
3. `params.yaml`로 임계값 튜닝.
4. 동일 로직을 `submission/src/<area>/`에 동기화.
5. `submission/dev.sh`로 대회 launch에서 재검증.

---

## Testing / 테스트

| File / 파일 | Coverage / 범위 |
|---|---|
| `src/autonomous/tests/test_algorithms.py` | 알고리즘 단위 테스트 (perception/control/utils) |

실행 방법 / How to run:

```bash
cd src/autonomous
python -m pytest tests/
```

또는 컨테이너 내부에서:

```bash
docker compose -f src/autonomous/docker-compose.yml run --rm autonomous pytest tests/
```

---

## Simulation / 시뮬레이션

`src/simulator/`는 FSDS(Full Self-Driving Simulator) 연동을 위한 설정 디렉터리입니다.

| File / 파일 | 역할 / Role |
|---|---|
| `src/simulator/README.md` | 시뮬레이터 사용 안내 |
| `src/simulator/settings.json` | 트랙·차량·센서 구성 |

시뮬레이션 모드에서는 `bridge_no_camera.launch`를 사용해 카메라 토픽을 시뮬레이터 데이터로 대체할 수 있습니다.

---

## Submission Package / 제출 패키지

`submission/`은 대회 규정을 만족하는 단일 컨테이너 산출물입니다.

| File / 파일 | 역할 / Role |
|---|---|
| `submission/Dockerfile` | 대회 제출 이미지 빌드 정의 |
| `submission/docker-compose.yml` | 단일 컨테이너 compose 정의 |
| `submission/run.sh` | 부팅 시 실행되는 메인 엔트리 |
| `submission/dev.sh` | 개발자 디버그 셸 진입 |
| `submission/launch/competition.launch` | 대회 launch 시퀀스 |
| `submission/autonomous/Dockerfile` | 제출 컨테이너 내부에서 사용하는 서브 이미지 |
| `docs/SUBMISSION_GUIDE.md` | 대회 제출 절차 가이드 |

자세한 절차는 [`docs/SUBMISSION_GUIDE.md`](docs/SUBMISSION_GUIDE.md)를 참조하세요.

---

## Reference Materials / 참고 자료

| Resource / 자료 | 경로 / Path |
|---|---|
| FSDS 설치 강의 노트 | `docs/reference_materials/lecture1_fsds_install.txt` |
| SLAM 강의 노트북 | `docs/reference_materials/lecture4_slam.ipynb` |
| V2X 강의 노트북 | `docs/reference_materials/lecture6_v2x.ipynb` |
| 제출 가이드 | `docs/SUBMISSION_GUIDE.md` |
| 시뮬레이터 사용법 | `src/simulator/README.md` |
| 제출 패키지 README | `submission/README.md` |

---

## Contributing / 기여

기여 절차는 [`CONTRIBUTING.md`](CONTRIBUTING.md)를 참조하세요. 주요 원칙:

1. 기능 브랜치 분기 후 PR 생성.
2. PR에는 알고리즘 변경 시 `src/autonomous/tests/test_algorithms.py` 갱신 포함.
3. 알고리즘 변경 시 `src/autonomous/modules/`와 `submission/src/`를 동시에 동기화.
4. 파라미터 변경 시 `params.yaml`을 명시적으로 갱신.

Maintainers는 [`OWNERS`](OWNERS) 파일을 참조하세요.

---

## License / 라이선스

[`LICENSE`](LICENSE) 파일에 명시된 라이선스를 따릅니다.