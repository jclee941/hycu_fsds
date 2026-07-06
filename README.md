# Formula Student Driverless — Autonomous Racing Stack

[![ROS 1](https://img.shields.io/badge/ROS-1-noetic-22314E?logo=ros)](https://wiki.ros.org)
[![Docker](https://img.shields.io/badge/Docker-ready-2496ED?logo=docker&logoColor=white)](https://www.docker.com)
[![Python 3](https://img.shields.io/badge/Python-3.x-3776AB?logo=python&logoColor=white)](https://www.python.org)
[![Simulator](https://img.shields.io/badge/Sim-FSDS-FF6F00)](https://github.com/FS-Driverless/Formula-Student-Driverless-Simulator)
[![Status](https://img.shields.io/badge/Status-Competition%20Ready-2EA043)]()
[![License](https://img.shields.io/badge/License-See%20LICENSE-blue)](LICENSE)

## 한눈에 보기 / At a Glance

이 저장소는 **Formula Student Driverless (FSD) 자율주행 레이싱 대회**를 위한 풀스택 소프트웨어입니다. 콘 감지·분류, LiDAR 기반 SLAM, Pure Pursuit 추종 제어, 랩 타이머, V2X(RSU) 어댑터, 그리고 FSG 규격의 Docker 제출 패키지까지 한 번에 제공합니다. 개발 작업 영역(`src/autonomous/`)과 대회 제출 패키지(`submission/`)를 분리해 두어, 동일한 알고리즘을 FSDS 시뮬레이터와 실차에서 일관되게 실행·검증할 수 있습니다.

A full-stack autonomous racing system for the Formula Student Driverless competition: cone-based perception, LiDAR SLAM, pure-pursuit control, lap timing, a V2X (RSU) adapter, and a competition-compliant Docker submission image. The `src/autonomous/` workspace hosts active development, while `submission/` packages the same algorithms for the official on-site run.

### 스택 요약 / Stack Snapshot

| 항목 / Item | 내용 / Description |
| --- | --- |
| 플랫폼 / Platform | ROS 1 (Noetic), Python 3, Docker |
| 대회 / Competition | Formula Student Driverless (FSD) |
| 시뮬레이터 / Simulator | FSDS (Formula Student Driverless Simulator) |
| 핵심 노드 / Core nodes | SLAM, Cone Detector, Cone Classifier, Pure Pursuit, Speed, V2X RSU, Lap Timer, Watchdog |
| 차량 진입점 / Vehicle entry point | `src/autonomous/driver/competition_driver.py` |
| 제출 진입점 / Submission entry point | `submission/run.sh` → `launch/competition.launch` |
| 제출 형태 / Submission form | Docker 이미지 (FSG 규격 / FSG-compliant) |
| 라이선스 / License | `LICENSE` 참조 / see `LICENSE` |

## 목차 / Table of Contents

- [개요 / Overview](#개요--overview)
- [주요 기능 / Features](#주요-기능--features)
- [아키텍처 / Architecture](#아키텍처--architecture)
- [빠른 시작 / Quick Start](#빠른-시작--quick-start)
- [설정 / Configuration](#설정--configuration)
- [명령어 / Commands Reference](#명령어--commands-reference)
- [로컬 개발 / Local Development](#로컬-개발--local-development)
- [테스트 / Testing](#테스트--testing)
- [제출 패키징 / Submission Packaging](#제출-패키징--submission-packaging)
- [기여 가이드 / Contributing](#기여-가이드--contributing)
- [유지보수 및 라이선스 / Maintainers & License](#유지보수-및-라이선스--maintainers--license)
- [추가 문서 / Further Documentation](#추가-문서--further-documentation)

## 개요 / Overview

이 프로젝트는 FSD 규정의 **Autonomous 카테고리**(Trackdrive, Skidpad, Autocross, EBS Test)를 소프트웨어 측면에서 완전하게 지원합니다. 카메라·LiDAR 입력을 받아 콘 트랙 경계와 슬라럼 경로를 인식하고, SLAM으로 차량 위치를 추정하며, Pure Pursuit 기반 종·횡 방향 제어 명령을 생성합니다. 모든 결과는 ROS 토픽으로 표준화되어 있어 실제 차량(ROS 1 Noetic)·시뮬레이터(FSDS) 양쪽에서 동일하게 동작합니다.

The project fully supports the FSD **Autonomous disciplines** (Trackdrive, Skidpad, Autocross, EBS Test) from a software perspective. It ingests camera and LiDAR data to detect cone boundaries and slalom paths, estimates vehicle pose via SLAM, and produces longitudinal/lateral control commands using a Pure Pursuit controller. All outputs are published on standard ROS topics, so the same node graph runs identically on a real car (ROS 1 Noetic) and inside the FSDS simulator.

## 주요 기능 / Features

| 영역 / Area | 기능 / Capability |
| --- | --- |
| 인식 / Perception | HSV 기반 콘 색상 분류 (`cone_classifier`), 콘 검출 파이프라인 (`cone_detector`) |
| 위치 추정 / Localization | LiDAR 기반 SLAM (`modules/perception/slam`) |
| 경로 추종 / Path Following | Pure Pursuit (`modules/control/pure_pursuit`) |
| 속도 제어 / Speed Control | 곡률·구간 기반 속도 프로파일 (`modules/control/speed`) |
| 대회 운영 / Race Ops | 랩 타이머 (`utils/lap_timer`), 워치독 (`utils/watchdog`) |
| V2X | RSU 어댑터 (`v2x/rsu`) — 인프라 신호 수신 |
| 시뮬레이션 / Sim | FSDS bridge 런치 (`config/bridge_no_camera.launch`) |
| 제출 / Submission | FSG 규격 Docker 이미지 빌드 및 실행 스크립트 |
| 패키징 / Packaging | `scripts/package.sh`로 의존성 잠금 산출물 생성 |

## 아키텍처 / Architecture

### 노드 그래프 / Node Graph (요약)

| 노드 / Node | 입력 토픽 (예) | 출력 토픽 (예) | 역할 / Role |
| --- | --- | --- | --- |
| `cone_detector` | `/sensors/camera/image_raw` | `/perception/cones` | 콘 후보 추출 / cone candidate extraction |
| `cone_classifier` | `/perception/cones` | `/perception/cones/classified` | 색상(노란/파란) 부착 / color labeling |
| `slam` | `/sensors/lidar/points` | `/localization/pose` | 차량 포즈 추정 / vehicle pose |
| `pure_pursuit` | `/localization/pose`, `/perception/cones/classified` | `/control/steering_cmd` | 조향각 산출 / steering command |
| `speed` | `/localization/pose`, `/perception/cones/classified` | `/control/speed_cmd` | 목표 속도 산출 / target speed |
| `lap_timer` | `/localization/pose`, `/perception/cones/classified` | `/race/lap_time` | 랩 기록 / lap timing |
| `watchdog` | 내부 상태 | `/system/heartbeat` | 데드맨 스위치 / dead-man switch |
| `v2x/rsu` | UDP (V2X) | `/v2x/infrastructure` | 인프라 신호 / infrastructure signals |
| `competition_driver` | 모든 토픽 | 차량 액추에이터 인터페이스 | 오케스트레이션 / orchestration |

요약: 모든 알고리즘 노드는 **ROS 1 토픽**으로만 통신하므로, 시뮬레이터와 실차 간에 노드 그래프 변경 없이 전환됩니다.

All algorithm nodes communicate exclusively over ROS 1 topics, so the same graph runs unchanged between the simulator and the real vehicle.

### 제어 흐름 / Control Flow (번호 순서)

1. **센서 입력** — 카메라 이미지, LiDAR 포인트클라우드, (선택) V2X UDP 패킷이 ROS 토픽으로 수신됩니다.
2. **인식** — `cone_detector`가 콘 후보를 추출하고, `cone_classifier`가 색상 라벨을 부여합니다.
3. **위치 추정** — `slam`이 LiDAR로 차량의 글로벌 포즈를 추정합니다.
4. **경로/속도 계산** — `pure_pursuit`이 조향각을, `speed`가 목표 속도를 산출합니다.
5. **안전 감시** — `watchdog`이 주기적으로 헬스체크하고, 이상 시 정지 명령을 발행합니다.
6. **구동** — `competition_driver`가 종·횡방향 명령을 차량 액추에이터 인터페이스로 전달합니다.
7. **대회 운영** — `lap_timer`가 콘 패턴 변화로 랩 완주를 판정하고 랩 타임을 기록합니다.

## 빠른 시작 / Quick Start

### 1. 사전 요구 사항 / Prerequisites

| 항목 / Item | 권장 버전 / Version |
| --- | --- |
| Docker Engine | 24.x 이상 |
| Docker Compose | v2 |
| ROS (네이티브 실행 시) | Noetic (Ubuntu 20.04) |
| Python (개발 시) | 3.8+ |
| FSDS 시뮬레이터 | 공식 빌드 / official build |

### 2. 개발 워크스페이스 (시뮬레이터) / Dev Workspace (Simulator)

```bash
cd src/autonomous
docker compose up --build
# 또는 / or
./run_all.sh
```

컨테이너가 시작되면 FSDS 브리지가 활성화되고, 알고리즘 노드들이 자동으로 구동됩니다. 시뮬레이터에서 차량을 출발시키면 인식 → 제어 루프가 즉시 동작합니다.

Once the container is up, the FSDS bridge is activated and algorithm nodes start automatically. Trigger the car in the simulator to begin the perception–control loop.

### 3. 대회 제출 이미지 / Competition Submission Image

```bash
cd submission
docker build -t fsd-submission:latest .
./run.sh
# 또는 / or
docker compose up
```

`run.sh`는 `launch/competition.launch`를 호출하여 Trackdrive/Autocross/Skidpad 등 모든 자율주행 과업을 하나의 런치 파일로 실행합니다.

`run.sh` invokes `launch/competition.launch`, which orchestrates Trackdrive / Autocross / Skidpad and other autonomous runs from a single launch file.

### 4. 네이티브 ROS 실행 (선택) / Native ROS Run (Optional)

```bash
# 워크스페이스 준비 / prepare workspace
mkdir -p ~/catkin_ws/src
cp -r src/autonomous ~/catkin_ws/src/
cd ~/catkin_ws
catkin_make
source devel/setup.bash

# 메인 드라이버 실행 / run main driver
roslaunch autonomous competition_driver.py
```

## 설정 / Configuration

| 파일 / File | 용도 / Purpose |
| --- | --- |
| `src/autonomous/config/params.yaml` | 인식·제어·SLAM 튜닝 파라미터 / perception/control/SLAM tunables |
| `src/autonomous/config/bridge_no_camera.launch` | FSDS 브리지 (카메라 제외) / FSDS bridge, camera excluded |
| `submission/launch/competition.launch` | 제출용 통합 런치 / integrated submission launch |
| `src/simulator/settings.json` | FSDS 시뮬레이터 차량·트랙 설정 / FSDS vehicle/track settings |
| `submission/autonomous/config/params.yaml` | 제출 환경 파라미터 / submission-tuned params |

자주 조정하는 키 / Frequently tuned keys:

- `cone_classifier.*` — HSV 색상 임계값
- `pure_pursuit.lookahead` — 전방 주시 거리
- `speed.max_velocity` — 직선부 최대 속도
- `watchdog.timeout_s` — 워치독 트리거 시간

## 명령어 / Commands Reference

| 명령 / Command | 위치 / Location | 설명 / Description |
| --- | --- | --- |
| `./start.sh` | `src/autonomous/` | 시뮬레이션용 빠른 시작 / quick simulator start |
| `./run_all.sh` | `src/autonomous/` | 전체 노드 일괄 실행 / run all nodes |
| `./record_race.sh` | `src/autonomous/` | 시뮬레이션 레코딩 / record race in sim |
| `python3 scripts/start_race.py` | `src/autonomous/` | 레이스 트리거 스크립트 / race trigger script |
| `./run.sh` | `submission/` | 대회 제출 이미지 실행 / run submission image |
| `./dev.sh` | `submission/` | 제출 이미지 개발 모드 진입 / dev mode for submission |
| `docker compose up --build` | 각 `docker-compose.yml` | 컨테이너 빌드·실행 / build & run container |
| `bash scripts/package.sh` | `scripts/` | 의존성 잠금 패키지 산출 / produce locked dependency bundle |

## 로컬 개발 / Local Development

### 워크스페이스 구조 / Workspace Structure

이 저장소는 의도적으로 두 트리를 가집니다:

- `src/autonomous/` — 활성 개발 트리. 알고리즘 변경, 파라미터 튜닝, 시뮬레이터 실험은 여기에서 수행합니다.
- `submission/` — 대회 제출 트리. 변경이 끝나면 알고리즘 모듈을 미러링하여 Docker 이미지로 패키징합니다.

### 개발 워크플로 / Dev Workflow

1. `src/autonomous/`에서 알고리즘 수정 후 `src/autonomous/tests/test_algorithms.py`로 단위 검증.
2. `run_all.sh`로 FSDS 시뮬레이터 통합 검증.
3. 안정화되면 동일 모듈을 `submission/`으로 동기화하고 `docker build`로 재패키징.
4. `submission/dev.sh`로 실제 차량 환경과 동일한 컨테이너에서 회귀 테스트.

### 코드 스타일 / Code Style

- Python: PEP 8, 타입 힌트 권장
- ROS 노드: `init_node` 사용, 토픽명은 `/perception/*`, `/control/*`, `/localization/*` 네임스페이스 준수

## 테스트 / Testing

| 테스트 / Test | 명령 / Command | 대상 / Target |
| --- | --- | --- |
| 알고리즘 단위 테스트 | `pytest src/autonomous/tests/test_algorithms.py` | 콘 인식·제어 알고리즘 |
| 시뮬레이션 통합 | `cd src/autonomous && ./run_all.sh` | FSDS end-to-end |
| 제출 회귀 | `cd submission && ./run.sh` | Docker 환경 일치성 |

테스트는 노드 단위(모듈 함수)와 시스템 단위(시뮬레이터/차량) 모두로 분리되어 있습니다. 새 알고리즘을 추가할 때는 반드시 `tests/test_algorithms.py`에 회귀 케이스를 동봉하세요.

Tests are split into node-level (module functions) and system-level (simulator / vehicle). When adding a new algorithm, include a regression case in `tests/test_algorithms.py`.

## 제출 패키징 / Submission Packaging

### 빌드 / Build

```bash
cd submission
docker build -t <team>-fsd-submission:latest .
```

### 패키지 산출물 / Package Artifact

```bash
cd scripts
bash package.sh
# 산출물: /output/fsd-submission.tar.gz (또는 동등 산출물)
```

산출물은 FSG가 요구하는 형태(단일 이미지 또는 압축 파일)로 정리되며, 의존성과 런치 구성이 잠겨 있습니다.

The artifact is packaged in the form required by FSG (single image or compressed bundle) with dependencies and launch configuration locked.

자세한 절차는 [`docs/SUBMISSION_GUIDE.md`](docs/SUBMISSION_GUIDE.md)를 참조하세요.

For the detailed procedure, see [`docs/SUBMISSION_GUIDE.md`](docs/SUBMISSION_GUIDE.md).

## 기여 가이드 / Contributing

1. 이슈를 먼저 등록하여 변경 범위를 합의합니다.
2. `src/autonomous/`에서 변경하고, `submission/`에는 제출 직전에 동기화합니다.
3. `src/autonomous/tests/`에 회귀 테스트를 추가합니다.
4. PR 본문에 시뮬레이터 검증 결과(랩 타임, 클립 URL 등)를 첨부합니다.
5. `CONTRIBUTING.md`의 PR 템플릿을 따릅니다.

풀 리퀘스트 가이드라인은 [`CONTRIBUTING.md`](CONTRIBUTING.md)를, 책임 영역은 [`OWNERS`](OWNERS)를 참조하세요.

For PR guidelines see [`CONTRIBUTING.md`](CONTRIBUTING.md); for ownership see [`OWNERS`](OWNERS).

## 유지보수 및 라이선스 / Maintainers & License

| 역할 / Role | 참조 / Reference |
| --- | --- |
| 책임자 / Owners | [`OWNERS`](OWNERS) |
| 에이전트 지침 / Agent rules | [`AGENTS.md`](AGENTS.md) |
| 라이선스 / License | [`LICENSE`](LICENSE) |

## 추가 문서 / Further Documentation

| 문서 / Document | 경로 / Path | 용도 / Purpose |
| --- | --- | --- |
| 제출 가이드 / Submission Guide | [`docs/SUBMISSION_GUIDE.md`](docs/SUBMISSION_GUIDE.md) | FSG 제출 절차 |
| 시뮬레이터 안내 / Simulator Notes | [`src/simulator/README.md`](src/simulator/README.md) | FSDS 설정 |
| 제출 패키지 안내 / Submission Notes | [`submission/README.md`](submission/README.md) | 제출 환경 설명 |
| 참조 자료 / Reference Materials | [`docs/reference_materials/`](docs/reference_materials/) | 강의 노트, 설치 스크립트, SLAM/V2X 예제 |
| 기여 가이드 / Contribution Guide | [`CONTRIBUTING.md`](CONTRIBUTING.md) | PR 절차 |