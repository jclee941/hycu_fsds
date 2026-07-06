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

### 실행 흐름 요약 / Compact Run Flow

1. Docker 컨테이너 부팅 → `entrypoint.sh` 실행 / container boots, runs `entrypoint.sh`
2. `start.sh` → `run_all.sh` 순으로 ROS 노드 가동 / `start.sh` then `run_all.sh` bring up ROS nodes
3. SLAM이 LiDAR 스캔으로 맵 생성, Cone Detector/Classifier가 콘 위치/색상 추정 / SLAM builds the track, cone pipeline estimates cone pose & color
4. Pure Pursuit + Speed 모듈이 경로 추종 및 속도 명령 생성 / Pure Pursuit + Speed emit steering/throttle commands
5. Lap Timer가 랩 시간 기록, V2X RSU가 인프라 메시지 송수신, Watchdog가 헬스 체크 / Lap Timer logs times, V2X RSU exchanges infra messages, Watchdog monitors health
6. `record_race.sh` 또는 시뮬레이터 종료 신호로 미션 종료 / Mission ends via `record_race.sh` or simulator stop

## 목차 / Table of Contents

- [Features](#features)
- [Architecture](#architecture)
- [Package Contents](#package-contents)
- [Status](#status)
- [First Files to Read](#first-files-to-read)
- [API / Entry Points](#api--entry-points)
- [Quickstart](#quickstart)
- [Configuration](#configuration)
- [Commands Reference](#commands-reference)
- [Local Development](#local-development)
- [Testing](#testing)
- [Submitting to FSG](#submitting-to-fsg)
- [Maintainers](#maintainers)
- [Contributing](#contributing)
- [Further Documentation](#further-documentation)

## Features

### Perception / 인지

| 모듈 / Module | 역할 / Role |
| --- | --- |
| `cone_detector.py` | LiDAR 클러스터링으로 콘 후보 추출 / cluster LiDAR returns into cone candidates |
| `cone_classifier.py` | 색상/종류(대형·소형) 분류, FSD 트랙 규칙 매핑 / classify color & size (large/small) and map to FSD rules |
| `slam.py` | LiDAR SLAM으로 글로벌 맵과 차량 포즈 추정 / LiDAR SLAM for global map & ego-pose |

### Control / 제어

| 모듈 / Module | 역할 / Role |
| --- | --- |
| `pure_pursuit.py` | 중심선 기반 조향각 산출 / steering from track centerline |
| `speed.py` | 곡률·구간별 속도 프로파일 생성 / curvature-aware speed profile |

### Mission / 미션 지원

| 모듈 / Module | 역할 / Role |
| --- | --- |
| `lap_timer.py` | 시작선 통과 감지, 랩 시간 로깅 / start-line crossing detection & lap logging |
| `watchdog.py` | 노드 헬스 체크, 데드락 감시 / node health & deadlock supervision |
| `rsu.py` (V2X) | RSU 인프라 메시지 송수신 / RSU infrastructure message exchange |

### Drivers / 차량 드라이버

| 파일 / File | 용도 / Use |
| --- | --- |
| `basic.py` | 최소 기능 드라이버 (디버깅) / minimal driver for debugging |
| `advanced.py` | 확장 드라이버 / extended driver |
| `autonomous.py` | 표준 자율주행 드라이버 / standard autonomous driver |
| `competition.py` | FSG 대회 규격 드라이버 / FSG competition-grade driver |

## Architecture

자율주행 파이프라인은 감지(Perception) → 위치추정(Localization) → 계획(Planning) → 제어(Control) → 미션(Mission) → 안전(Watchdog) 순서로 구성됩니다.

| 단계 / Stage | 입력 / Input | 출력 / Output | 책임 모듈 / Module |
| --- | --- | --- | --- |
| Perception | LiDAR, (선택) Camera | Cone list (x,y,color,size) | `cone_detector`, `cone_classifier` |
| Localization | LiDAR, IMU | Pose, Map | `slam` |
| Planning | Cone list, Pose | Centerline path | `pure_pursuit` |
| Control | Path, Pose | Steering / Throttle / Brake | `pure_pursuit`, `speed` |
| Mission | Pose, Timer | Lap time, mission state | `lap_timer`, `rsu` |
| Safety | All topics | Heartbeat, E-Stop | `watchdog` |

ROS 토픽 흐름은 FSDS 시뮬레이터와 동일한 메시지 계약(`/scan`, `/odom`, `/cmd_vel` 등)을 따르며, 상세 내용은 `docs/reference_materials/`의 강의를 참고하세요.

## Package Contents

| 경로 / Path | 설명 / Description |
| --- | --- |
| `src/autonomous/` | 활성 개발 워크스페이스 / active development workspace |
| `src/autonomous/driver/competition_driver.py` | 대회 차량 진입점 / competition vehicle entry point |
| `src/autonomous/modules/perception/` | 콘 감지·분류·SLAM / cone perception & SLAM |
| `src/autonomous/modules/control/` | Pure Pursuit / Speed |
| `src/autonomous/modules/utils/` | Lap Timer, Watchdog |
| `src/autonomous/config/params.yaml` | 튜닝 파라미터 / tuning parameters |
| `src/autonomous/config/bridge_no_camera.launch` | ROS-FSDS 브리지 (카메라 제외) / bridge launch |
| `src/autonomous/Dockerfile` | 개발용 이미지 / dev image |
| `src/autonomous/docker-compose.yml` | 개발용 compose / dev compose |
| `src/autonomous/tests/test_algorithms.py` | 알고리즘 단위 테스트 / algorithm unit tests |
| `src/simulator/` | FSDS 시뮬레이터 설정 / FSDS simulator settings |
| `submission/` | FSG 제출 패키지 / FSG submission package |
| `submission/run.sh` | 제출 이미지 실행 스크립트 / submission run script |
| `submission/dev.sh` | 제출 이미지 개발 모드 진입 / dev mode entry |
| `submission/launch/competition.launch` | 대회 런치 파일 / competition launch file |
| `submission/Dockerfile` | 제출용 이미지 빌드 / submission image build |
| `submission/src/v2x/rsu.py` | V2X RSU 어댑터 (제출 전용) / V2X RSU adapter |
| `scripts/package.sh` | 제출 패키징 도구 / submission packaging tool |
| `docs/SUBMISSION_GUIDE.md` | 제출 절차 안내 / submission walkthrough |
| `docs/reference_materials/` | FSDS 강의 자료 / FSDS lecture materials |
| `in-memoria.db` | 로컬 노트/메모 데이터베이스 / local memo database |
| `CONTRIBUTING.md` | 기여 가이드 / contribution guide |
| `OWNERS` | 코드 오너십 / code ownership |
| `LICENSE` | 라이선스 본문 / license text |

## Status

| 항목 / Item | 상태 / Status |
| --- | --- |
| 대회 준비도 / Competition readiness | Competition-ready (FSD 규격 / FSG rules) |
| 시뮬레이터 연동 / Simulator integration | FSDS validated |
| Docker 이미지 / Docker image | 빌드 가능, 제출 규격 충족 / buildable, FSG-compliant |
| 테스트 커버리지 / Test coverage | 핵심 알고리즘 단위 테스트 (`tests/test_algorithms.py`) |
| 활성 유지보수 / Active maintenance | See `OWNERS` |
| Deprecation | 없음 / None |

## First Files to Read

운영자/신규 기여자가 가장 먼저 봐야 할 파일 / read these first:

| 순서 / Order | 파일 / File | 이유 / Why |
| --- | --- | --- |
| 1 | `docs/SUBMISSION_GUIDE.md` | 제출 전체 흐름 / end-to-end submission flow |
| 2 | `src/autonomous/driver/competition_driver.py` | 차량 진입점 / vehicle entry point |
| 3 | `src/autonomous/config/params.yaml` | 튜닝 파라미터 / tuning parameters |
| 4 | `submission/launch/competition.launch` | 제출 런치 토폴로지 / submission launch topology |
| 5 | `src/autonomous/modules/perception/slam.py` | SLAM 진입점 / SLAM entry point |
| 6 | `src/autonomous/tests/test_algorithms.py` | 알고리즘 단위 테스트 / algorithm unit tests |

## API / Entry Points

| 진입점 / Entry Point | 종류 / Kind | 책임 / Responsibility |
| --- | --- | --- |
| `competition_driver.py` | ROS 노드 / ROS node | FSD 차량 메인 드라이버 / main FSD vehicle driver |
| `cone_detector.py` | ROS 노드 / ROS node | 콘 후보 발행 / publishes cone candidates |
| `cone_classifier.py` | ROS 노드 / ROS node | 콘 분류 결과 발행 / publishes classified cones |
| `slam.py` | ROS 노드 / ROS node | 맵/포즈 발행 / publishes map & pose |
| `pure_pursuit.py` | ROS 노드 / ROS node | 조향 명령 발행 / publishes steering commands |
| `speed.py` | ROS 노드 / ROS node | 속도/스로틀·브레이크 명령 / speed/throttle/brake |
| `lap_timer.py` | ROS 노드 / ROS node | 랩 시간 로깅 / logs lap times |
| `watchdog.py` | ROS 노드 / ROS node | 헬스/데드락 감시 / health & deadlock monitor |
| `rsu.py` | ROS 노드 / ROS node | V2X 인프라 메시지 / V2X infra messages |
| `scripts/package.sh` | CLI / CLI tool | 제출 패키징 / submission packaging |

## Quickstart

### 1. 저장소 클론 / Clone

```bash
git clone <repo-url> fs-driverless
cd fs-driverless
```

### 2. 개발 컨테이너 실행 / Run dev container

```bash
cd src/autonomous
docker compose up -d
docker compose exec autonomous bash
```

컨테이너 내부에서 ROS 환경이 자동 소싱되며, `start.sh`로 전체 파이프라인을 띄울 수 있습니다.

Inside the container, ROS is auto-sourced; use `start.sh` to bring up the full pipeline.

### 3. FSDS 시뮬레이터 연결 / Connect FSDS

`bridge_no_camera.launch`로 ROS-FSDS 브리지를 시작하고 시뮬레이터와 동기화합니다.

Launch the ROS↔FSDS bridge (no camera) and sync with the simulator:

```bash
roslaunch src/autonomous/config/bridge_no_camera.launch
```

### 4. 미션 실행 / Run a mission

```bash
./src/autonomous/start.sh           # 노드 일괄 기동 / bring up all nodes
./src/autonomous/record_race.sh     # 미션 기록 / record mission
```

## Configuration

| 파일 / File | 역할 / Role | 비고 / Notes |
| --- | --- | --- |
| `src/autonomous/config/params.yaml` | 차량/알고리즘 튜닝 파라미터 / vehicle & algorithm tuning | 핵심 설정 / primary tuning surface |
| `src/autonomous/config/bridge_no_camera.launch` | ROS-FSDS 브리지 / bridge | 카메라 토픽 제외 / camera excluded |
| `submission/launch/competition.launch` | 제출 런치 / submission launch | 대회용 토폴로지 / competition topology |
| `src/simulator/settings.json` | FSDS 시뮬레이터 설정 / FSDS simulator settings | 차량/트랙 / vehicle & track |
| `submission/docker-compose.yml` | 제출 컨테이너 정의 / submission container | 운영 시 / runtime |
| `src/autonomous/docker-compose.yml` | 개발 컨테이너 정의 / dev container | 개발 시 / dev time |

## Commands Reference

| 명령 / Command | 위치 / Location | 용도 / Purpose |
| --- | --- | --- |
| `./start.sh` | `src/autonomous/` | 개발 노드 기동 / launch dev nodes |
| `./run_all.sh` | `src/autonomous/` | 전체 파이프라인 기동 / full pipeline |
| `./record_race.sh` | `src/autonomous/` | 미션 기록 시작 / start mission recording |
| `./entrypoint.sh` | `src/autonomous/` | 컨테이너 부팅 훅 / container boot hook |
| `roslaunch .../bridge_no_camera.launch` | `src/autonomous/config/` | 시뮬레이터 브리지 / simulator bridge |
| `./run.sh` | `submission/` | 제출 이미지 실행 / run submission image |
| `./dev.sh` | `submission/` | 제출 이미지 개발 모드 / dev mode entry |
| `./scripts/package.sh` | `scripts/` | 제출 패키징 / package submission |
| `docker compose up` | `src/autonomous/` 또는 `submission/` | Compose 기동 / bring up compose |
| `python -m unittest src/autonomous/tests/test_algorithms.py` | repo root | 알고리즘 단위 테스트 / unit tests |

## Local Development

1. FSDS 시뮬레이터를 별도로 실행합니다.
2. 개발 워크스페이스를 컨테이너에서 띄웁니다: `cd src/autonomous && docker compose up -d`.
3. 컨테이너에 진입한 뒤 `start.sh`로 노드를 띄우고, 알고리즘 변경 시 `tests/test_algorithms.py`로 회귀 테스트를 수행합니다.
4. 동일 알고리즘이 `submission/` 트리에도 복제되어 있으므로, 알고리즘을 양쪽에 동기화해야 합니다(상세는 `docs/SUBMISSION_GUIDE.md`).

Workflow:

1. Run FSDS simulator separately.
2. Bring up the dev container: `cd src/autonomous && docker compose up -d`.
3. Enter the container, run `start.sh`, iterate on algorithms, then regress with `tests/test_algorithms.py`.
4. Mirror algorithm changes into `submission/` (see `docs/SUBMISSION_GUIDE.md`).

### 디렉터리 동기화 가이드 / Directory Sync Notes

`src/autonomous/modules/`와 `submission/src/`는 동일한 모듈을 유지합니다. 새 모듈을 추가할 때는 양쪽에 동일한 인터페이스로 배치하세요.

The `src/autonomous/modules/` and `submission/src/` trees mirror each other. Add new modules to both with identical interfaces.

## Testing

```bash
cd src/autonomous
python -m unittest tests/test_algorithms.py
```

추가로 시뮬레이터 통합 테스트는 FSDS를 실행한 뒤 미션 시나리오로 수동 검증합니다.

Additionally, run end-to-end simulator integration manually via FSDS mission scenarios.

## Submitting to FSG

자세한 절차는 `docs/SUBMISSION_GUIDE.md`를 따르세요.

| 단계 / Step | 명령 / Command |
| --- | --- |
| 1. 패키징 / Package | `./scripts/package.sh` |
| 2. 제출 이미지 빌드 / Build submission image | `cd submission && docker compose build` |
| 3. 로컬 dry-run / Local dry-run | `cd submission && ./run.sh` |
| 4. 대회 제출 / Submit | `docs/SUBMISSION_GUIDE.md`의 "Submit" 섹션 참조 / see "Submit" section |

## Maintainers

- 코드 오너십: `OWNERS` 파일 참조 / see `OWNERS`.
- 이슈/문의: 저장소 이슈 트래커 사용 / use the repository issue tracker.
- 운영 문의 / Operational contact: `OWNERS`에 명시된 담당자 / contacts listed in `OWNERS`.

## Contributing

기여 절차와 PR 규칙은 `CONTRIBUTING.md`를 따릅니다. 알고리즘 변경 시 `src/autonomous/`와 `submission/` 양쪽에 동일하게 반영하고, `tests/test_algorithms.py`로 회귀를 검증해 주세요.

See `CONTRIBUTING.md`. When changing algorithms, mirror updates into both `src/autonomous/` and `submission/`, and add regression coverage in `tests/test_algorithms.py`.

## Further Documentation

| 문서 / Document | 경로 / Path |
| --- | --- |
| 제출 가이드 / Submission guide | `docs/SUBMISSION_GUIDE.md` |
| FSDS 설치 강의 / FSDS install lecture | `docs/reference_materials/lecture1_fsds_install.txt` |
| SLAM 강의 / SLAM lecture | `docs/reference_materials/lecture4_slam.ipynb` |
| V2X 강의 / V2X lecture | `docs/reference_materials/lecture6_v2x.ipynb` |
| 시뮬레이터 설정 / Simulator settings | `src/simulator/README.md`, `src/simulator/settings.json` |
| 라이선스 / License | `LICENSE` |
| 기여 가이드 / Contributing | `CONTRIBUTING.md` |
| 오너십 / Ownership | `OWNERS` |