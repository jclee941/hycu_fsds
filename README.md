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
| 핵심 노드 / Core nodes | SLAM, Cone Detector/Classifier, Pure Pursuit, Speed, V2X RSU, Lap Timer, Watchdog |
| 차량 진입점 / Vehicle entry point | `competition_driver.py` |
| 제출 형태 / Submission form | Docker 이미지 (FSG 규격 / FSG-compliant) |
| 라이선스 / License | `LICENSE` 참조 / see `LICENSE` |

### 실행 흐름 요약 / Compact Run Flow

1. Docker 컨테이너 부팅 → `entrypoint.sh` 실행 / container boots, runs `entrypoint.sh`
2. `roslaunch competition.launch` (또는 `bridge_no_camera.launch`) 로 노드 기동 / launch ROS nodes
3. `competition_driver.py` 가 차량 인터페이스와 메인 루프 시작 / driver starts vehicle interface + main loop
4. Perception(콘) → SLAM → Pure Pursuit → Vehicle 로 데이터 라우팅 / perception → SLAM → control → vehicle
5. Lap Timer 기록, Watchdog 감시, V2X RSU 가 인프라 통신 / lap timer logs, watchdog supervises, V2X RSU communicates

## 상태 / Status

| 영역 / Area | 상태 / Status |
| --- | --- |
| 대회 준비도 / Competition readiness | Competition-ready |
| 시뮬레이터 검증 / Simulator validation | FSDS end-to-end 연동 / integrated |
| Docker 제출 이미지 / Docker submission image | `submission/Dockerfile` 제공 / provided |
| 현장 차량 검증 / On-site vehicle runs | 팀 내부 테스트 이력 / per team history |
| 문서화 / Documentation | `docs/SUBMISSION_GUIDE.md` 참조 / see submission guide |

## 주요 기능 / Features

- **콘 감지·분류 / Cone detection & classification** — HSV + 윤곽 기반의 단일 콘 위치 추정 및 색상 분류 / HSV/contour-based cone detection and color classification
- **SLAM** — LiDAR 기반 동시 위치 추정·지도 작성 / LiDAR-based simultaneous localization and mapping
- **Pure Pursuit** — 곡선 경로 추종 / geometric path following
- **Speed Control** — 코너·직선 속도 프로파일 / corner vs. straight speed profiles
- **Lap Timer** — 시작/종료선 감지 및 랩 타임 기록 / start/finish line detection + lap logging
- **Watchdog** — 노드 헬스 체크 및 안전 정지 / node health supervision + safe stop
- **V2X RSU** — 인프라(RSU) 메시지 어댑터 / roadside-unit message adapter
- **FSDS 통합 / FSDS integration** — 시뮬레이터 기반 end-to-end 학습·디버깅 / sim-based learning and debugging
- **Docker 제출 / Docker submission** — FSG 규격 on-site 제출 이미지 / FSG-compliant on-site submission image

## 아키텍처 / Architecture

### 계층 구성 / Layered View

| 계층 / Layer | 모듈 / Module | 주요 책임 / Responsibility |
| --- | --- | --- |
| 차량 I/O / Vehicle I/O | `competition_driver.py` | 조향·스로틀·브레이크 인터페이스 및 메인 루프 / steering, throttle, brake, main loop |
| 인지 / Perception | `cone_detector.py`, `cone_classifier.py` | 카메라 입력 → 콘 위치·색상 / camera input → cone positions and colors |
| 위치 추정 / Localization | `slam.py` | LiDAR 스캔 → 차량 포즈 + 콘 맵 / LiDAR scans → pose and cone map |
| 제어 / Control | `pure_pursuit.py`, `speed.py` | 경로 추종 + 속도 명령 / path following and speed commands |
| 인프라 통신 / V2X | `rsu.py` | RSU 메시지 송수신 / send/receive RSU messages |
| 안전·계측 / Safety & Telemetry | `watchdog.py`, `lap_timer.py` | 헬스 체크 + 랩 기록 / health check + lap logging |
| 런치 / Launch | `competition.launch`, `bridge_no_camera.launch` | 노드 토폴로지 / node topology |
| 제출 / Submission | `submission/Dockerfile`, `run.sh` | FSG 규격 패키징 / FSG-compliant packaging |

### 데이터 흐름 / Data Flow

1. 카메라 + LiDAR 가 동시 입력 / camera + LiDAR ingest concurrently
2. `cone_detector` 가 콘 후보 추출 → `cone_classifier` 가 색상 확정 / detector → classifier pipeline
3. `slam` 이 LiDAR + 콘 관측을 융합해 포즈·맵 갱신 / SLAM fuses LiDAR + cone observations
4. `pure_pursuit` + `speed` 가 경로·속도 명령 생성 / pure-pursuit + speed emit control commands
5. `competition_driver` 가 차량에 명령 전달, `watchdog` 가 헬스 감시 / driver commands vehicle; watchdog supervises
6. `lap_timer` 가 랩 기록, `rsu` 가 인프라 메시지 송수신 / lap timer logs; RSU exchanges infrastructure messages

## 패키지 구성 / Package Contents

| 경로 / Path | 설명 / Description |
| --- | --- |
| `src/autonomous/` | 개발 작업 영역 (catkin 워크스페이스) / active development workspace |
| `src/autonomous/driver/competition_driver.py` | 차량 인터페이스 + 메인 루프 진입점 / vehicle interface + main loop |
| `src/autonomous/modules/perception/` | `cone_detector.py`, `cone_classifier.py`, `slam.py` |
| `src/autonomous/modules/control/` | `pure_pursuit.py`, `speed.py` |
| `src/autonomous/modules/utils/` | `lap_timer.py`, `watchdog.py` |
| `src/autonomous/config/` | `params.yaml`, `bridge_no_camera.launch` |
| `src/autonomous/scripts/start_race.py` | FSDS 레이스 트리거 / race trigger |
| `src/autonomous/tests/test_algorithms.py` | 알고리즘 단위 테스트 / algorithm unit tests |
| `src/simulator/` | FSDS 시뮬레이터 설정 (`settings.json`) |
| `submission/` | FSG 규격 Docker 제출 패키지 / FSG-compliant submission package |
| `submission/launch/competition.launch` | 제출용 ROS 런치 / submission ROS launch |
| `submission/src/` | 제출용 소스 (동일 알고리즘 사본) / submission source mirror |
| `scripts/package.sh` | 제출 패키징 헬퍼 / submission packaging helper |
| `docs/SUBMISSION_GUIDE.md` | 제출 절차 가이드 / submission procedure |
| `docs/reference_materials/` | FS DS 설치, SLAM, V2X 강의 노트 / reference lecture notes |
| `LICENSE`, `OWNERS`, `CONTRIBUTING.md` | 거버넌스 / governance |

## 먼저 읽을 파일 / First Files to Read

운영자·신규 합류자가 가장 먼저 봐야 할 파일 / files to read first:

1. `src/autonomous/driver/competition_driver.py` — 차량 진입점과 메인 루프 / vehicle entry point and main loop
2. `src/autonomous/modules/perception/cone_detector.py` — 콘 감지 파이프라인 / cone detection pipeline
3. `src/autonomous/modules/control/pure_pursuit.py` — 추종 제어 / path following
4. `src/autonomous/config/params.yaml` — 튜닝 파라미터 / tuning parameters
5. `docs/SUBMISSION_GUIDE.md` — 제출 절차 / submission procedure
6. `submission/README.md` — 제출 패키지 구조 / submission package structure

## API / 엔트리 포인트 / Entry Points

| 엔트리 / Entry | 역할 / Role |
| --- | --- |
| `competition_driver.py` | 메인 차량 루프 (steering + throttle + brake) |
| `start_race.py` | FSDS 레이스 트리거 / race trigger |
| `competition.launch` | 제출용 ROS 런치 / submission ROS launch |
| `bridge_no_camera.launch` | 카메라 비활성 디버그 런치 / debug launch without camera |
| `entrypoint.sh` | Docker 컨테이너 시작 훅 / container start hook |
| `run_all.sh` / `start.sh` | 노드 일괄/개별 기동 / bulk and per-node startup |
| `record_race.sh` | 레이스 로그·비디오 기록 / race log and video recording |

## 빠른 시작 / Quickstart

### 1. 시뮬레이터 실행 / Run in FSDS

```bash
# 사전 설치 / prerequisites: ROS Noetic, catkin_tools, FSDS
cd src/autonomous
catkin_make
source devel/setup.bash

# 런치 + 드라이버 / launch + driver
roslaunch bridge_no_camera.launch
rosrun driver competition_driver.py
```

### 2. Docker 제출 이미지 빌드 및 실행 / Build & Run Submission Image

```bash
cd submission
docker compose build
./run.sh          # 제출 이미지 실행 / run submission image
./dev.sh          # 개발용 인터랙티브 셸 / interactive dev shell
```

자세한 절차는 `docs/SUBMISSION_GUIDE.md` 참조 / see `docs/SUBMISSION_GUIDE.md` for the full procedure.

## 설정 / Configuration

| 파일 / File | 용도 / Purpose |
| --- | --- |
| `src/autonomous/config/params.yaml` | 노드 파라미터 (임계값, 속도 제한 등) / node parameters (thresholds, speed limits) |
| `src/autonomous/config/bridge_no_camera.launch` | 디버그용 런치 / debug launch |
| `submission/launch/competition.launch` | 제출용 런치 / submission launch |
| `src/simulator/settings.json` | FSDS 시뮬레이터 설정 / simulator settings |

> 실제 차량 IP·컨테이너 번호 등 환경 의존 값은 `params.yaml` 또는 환경 변수로 주입합니다. / Inject environment-dependent values (vehicle IPs, container numbers) via `params.yaml` or environment variables.

## 명령어 레퍼런스 / Commands Reference

| 명령 / Command | 설명 / Description |
| --- | --- |
| `./run_all.sh` | 전체 노드 일괄 기동 / start all nodes |
| `./start.sh` | 기본 노드 기동 / start default node set |
| `./record_race.sh` | 레이스 로그·비디오 기록 / record race log and video |
| `./entrypoint.sh` | Docker 컨테이너 진입 / container entrypoint |
| `python3 scripts/start_race.py` | 레이스 트리거 / race trigger |
| `bash scripts/package.sh` | 제출 패키지 빌드 / build submission package |
| `roslaunch competition.launch` | 제출 ROS 런치 / submission launch |
| `catkin_make` | 워크스페이스 빌드 / build workspace |

## 로컬 개발 / Local Development

```bash
# 의존성 / dependencies
sudo apt install ros-noetic-desktop-full python3-catkin-tools

# 빌드 / build
cd src/autonomous
catkin_make
source devel/setup.bash

# 개발 워크플로우 / workflow
# 1. perception / control 모듈 수정 후 catkin_make 재실행
# 2. FSDS 로 end-to-end 확인
# 3. src/autonomous/tests/ 통과 확인
# 4. 동일 알고리즘을 submission/src/ 에 동기화 후 Docker 검증
```

## 테스트 / Testing

| 단계 / Stage | 도구 / Tool |
| --- | --- |
| 단위 테스트 / Unit | `src/autonomous/tests/test_algorithms.py` |
| 통합 테스트 / Integration | FSDS end-to-end 시뮬레이션 / FSDS end-to-end runs |
| 제출 검증 / Submission | `docs/SUBMISSION_GUIDE.md` 체크리스트 / checklist |

## 기여 / Contributing

기여 절차는 `CONTRIBUTING.md` 를 따릅니다. PR 전 다음을 권장합니다 / before opening a PR:

- `src/autonomous/tests/` 통과 / pass unit tests
- 변경 알고리즘을 `submission/src/` 에 동기화 / sync changed algorithms into `submission/src/`
- `params.yaml` 변경 시 노트 첨부 / document any `params.yaml` changes

## 유지보수자 / 연락처 / Maintainers & Contact

`OWNERS` 파일 참조 / see `OWNERS` for the current maintainer list and contact points.

## 라이선스 / License

`LICENSE` 파일 참조 / see `LICENSE` for full terms.

## 추가 문서 / Further Documentation

- 제출 절차 / Submission procedure: [`docs/SUBMISSION_GUIDE.md`](./docs/SUBMISSION_GUIDE.md)
- 시뮬레이터 가이드 / Simulator guide: [`src/simulator/README.md`](./src/simulator/README.md)
- 제출 패키지 / Submission package: [`submission/README.md`](./submission/README.md)
- 워크스페이스 가이드 / Workspace notes: [`src/autonomous/AGENTS.md`](./src/autonomous/AGENTS.md), [`submission/AGENTS.md`](./submission/AGENTS.md)
- 강의 노트 / Lecture notes: [`docs/reference_materials/`](./docs/reference_materials/)
- 거버넌스 / Governance: [`CONTRIBUTING.md`](./CONTRIBUTING.md), [`OWNERS`](./OWNERS)