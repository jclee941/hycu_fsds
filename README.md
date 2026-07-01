# Formula Student Driverless — Autonomous Racing Stack

> **FSD (Formula Student Driverless) 자율주행 레이싱 소프트웨어 스택**
> Autonomous racing software stack for the Formula Student Driverless competition

[![ROS 1](https://img.shields.io/badge/ROS-1-noetic-22314E?logo=ros)](https://wiki.ros.org)
[![Docker](https://img.shields.io/badge/Docker-ready-2496ED?logo=docker&logoColor=white)](https://www.docker.com)
[![Python 3](https://img.shields.io/badge/Python-3.x-3776AB?logo=python&logoColor=white)](https://www.python.org)
[![Simulator](https://img.shields.io/badge/Sim-FSDS-FF6F00)](https://github.com/FS-Driverless/Formula-Student-Driverless-Simulator)
[![Status](https://img.shields.io/badge/Status-Competition%20Ready-2EA043)]()
[![License](https://img.shields.io/badge/License-See%20LICENSE-blue)](LICENSE)

## 한눈에 보기 / At a Glance

이 저장소는 FSD(Formula Student Driverless) 대회를 위한 완전 자율주행 레이싱 차량 소프트웨어 스택입니다.
카메라·LiDAR 기반 SLAM, 콘 감지·분류, Pure Pursuit 추종 제어, 랩 타이머, V2X(RSU) 어댑터, 그리고 FSG 규격의 Docker 제출 패키지를 함께 제공합니다.
개발 작업 영역(`src/autonomous/`)과 대회 제출 패키지(`submission/`)를 분리해 두어, 동일 알고리즘을 시뮬레이터와 실차에서 일관되게 실행할 수 있습니다.

This repository delivers a full-stack autonomous racing system for the Formula Student Driverless competition:
cone-based perception, SLAM, pure-pursuit control, lap timing, a V2X (RSU) adapter, and a competition-compliant
Docker submission image. The `src/autonomous/` workspace hosts active development, while `submission/` packages
the same algorithms for the official on-site run.

### 스택 요약 / Stack Snapshot

| 항목 / Item | 내용 / Description |
| --- | --- |
| 플랫폼 / Platform | ROS 1 (Noetic), Python 3, Docker |
| 대회 / Competition | Formula Student Driverless (FSD) |
| 핵심 노드 / Core nodes | SLAM, 콘 감지·분류, Pure Pursuit, V2X RSU, Lap Timer, Watchdog |
| 차량 인터페이스 / Vehicle I/O | `competition_driver.py` (단일 진입점 / single entry point) |
| 시뮬레이터 / Simulator | FSDS (Formula Student Driverless Simulator) |
| 제출 형태 / Submission | Docker 이미지 (FSG 규격 / FSG-compliant) |
| 작업 영역 / Workspaces | `src/autonomous/` (개발 / dev), `submission/` (제출 / submit) |
| 데이터 / Data | `in-memoria.db` (런타임 상태 스냅샷 / runtime state snapshot) |
| 라이선스 / License | [`LICENSE`](LICENSE) 참조 / see [`LICENSE`](LICENSE) |

### 운영 흐름 / Operating Flow

1. **Perception** — 카메라·LiDAR로 콘(노란·파란)을 감지·분류하고 SLAM으로 차량 자세를 추정합니다.
2. **Planning** — 감지된 콘 경계로 미니멀 레인 중심 경로를 구성합니다.
3. **Control** — Pure Pursuit이 조향각을, 속도 노드가 종·횡 방향 속도 명령을 생성합니다.
4. **V2X** — RSU(Road-Side Unit)로부터 수신한 보조 신호(미션 플래그 등)를 융합합니다.
5. **Driver** — `competition_driver.py`가 모든 신호를 단일 명령으로 묶어 차량 인터페이스에 전달합니다.
6. **Telemetry** — `lap_timer.py`와 `watchdog.py`가 랩 타임과 안전 상태를 모니터링합니다.

---

## 1. Purpose / 목적

| 대상 / Audience | 얻을 수 있는 것 / What you get |
| --- | --- |
| 팀원 / Team members | 대회 제출용 Docker 이미지와 로컬 시뮬레이션 개발 환경 |
| 신규 합류자 / New contributors | SLAM·Perception·Control 모듈을 독립적으로 실행 가능한 모듈형 구조 |
| 심사위원 / Judges & reviewers | FSG 규격을 만족하는 재현 가능한 실행 진입점(`run.sh`, `start.sh`) |
| 운영자 / Operators | 시뮬레이터(FSDS)와 실차에서 동일한 ROS 토픽 인터페이스 사용 |

이 저장소는 **대회를 위한 실전 소프트웨어**입니다. 시뮬레이터 검증 → 대회 제출 이미지 빌드 → 현장 부팅까지의 흐름을 한 곳에서 다룹니다.
연구용 프레임워크가 아니므로, 알고리즘 튜닝은 모듈 단위로 진행하며 대회 규칙(`docs/SUBMISSION_GUIDE.md`)을 우선합니다.

---

## 2. Package Contents / 패키지 구성

실제 최상위 디렉터리 구조를 반영합니다.

| 경로 / Path | 역할 / Role |
| --- | --- |
| `src/autonomous/` | 메인 개발 작업 영역. ROS 패키지, Docker 빌드, 시뮬레이터 런치 파일 |
| `src/autonomous/driver/` | 대회용 차량 드라이버 진입점 (`competition_driver.py`) |
| `src/autonomous/modules/perception/` | 콘 감지·분류, SLAM |
| `src/autonomous/modules/control/` | Pure Pursuit 추종, 속도 명령 |
| `src/autonomous/modules/utils/` | 랩 타이머, 워치독 |
| `src/autonomous/config/` | ROS 파라미터(`params.yaml`), 시뮬레이터 런치 |
| `src/autonomous/scripts/` | `start_race.py` 등 보조 스크립트 |
| `src/autonomous/tests/` | 알고리즘 단위 테스트 (`test_algorithms.py`) |
| `src/autonomous/Dockerfile` | 개발용 컨테이너 이미지 |
| `src/autonomous/docker-compose.yml` | 시뮬레이터 연동 compose |
| `src/autonomous/start.sh`, `run_all.sh`, `record_race.sh`, `entrypoint.sh` | 운영 셸 진입점 |
| `src/simulator/` | FSDS 설정 (`settings.json`) |
| `scripts/package.sh` | 대회 제출 패키징 스크립트 |
| `submission/` | FSG 규격 대회 제출 패키지 (독립 작업 영역) |
| `submission/launch/competition.launch` | 제출용 단일 ROS 런치 |
| `submission/src/drivers/` | basic / advanced / autonomous / competition 드라이버 |
| `submission/src/perception/`, `control/`, `utils/`, `v2x/` | 제출용 모듈(개발 영역과 1:1 대응) |
| `submission/src/v2x/rsu.py` | RSU 메시지 수신·파싱 어댑터 |
| `submission/autonomous/` | 제출 패키지 내부의 Docker·시작 스크립트 모음 |
| `submission/run.sh`, `dev.sh` | 대회 환경 표준 진입점 |
| `docs/SUBMISSION_GUIDE.md` | FSG 제출 절차·규격 요약 |
| `docs/reference_materials/` | FSDS 설치 가이드, SLAM·V2X 참고 자료 |
| `in-memoria.db` | 런타임 상태 스냅샷 (디버깅·재현용) |
| `AGENTS.md`, `submission/AGENTS.md`, `src/autonomous/AGENTS.md` | 에이전트 작업 지침 |
| `CONTRIBUTING.md`, `OWNERS`, `LICENSE` | 기여·소유·라이선스 정책 |

---

## 3. Status / 현황

| 영역 / Area | 상태 / Status | 비고 / Notes |
| --- | --- | --- |
| 대회 제출 / Competition submission | Ready | FSG 규격 Docker 이미지 빌드 가능 |
| 시뮬레이터 / Simulator (FSDS) | Verified | `src/simulator/settings.json` + `bridge_no_camera.launch` |
| 콘 감지 / Cone detection | Implemented | `modules/perception/cone_detector.py`, `cone_classifier.py` |
| SLAM | Implemented | `modules/perception/slam.py` |
| Pure Pursuit 제어 / Control | Implemented | `modules/control/pure_pursuit.py`, `speed.py` |
| V2X (RSU) | Implemented | `submission/src/v2x/rsu.py` |
| 랩 타이머 / Lap timer | Implemented | `modules/utils/lap_timer.py` |
| 안전 워치독 / Watchdog | Implemented | `modules/utils/watchdog.py` |
| 단위 테스트 / Unit tests | Present | `src/autonomous/tests/test_algorithms.py` |
| 프로덕션 차량 검증 / Production-vehicle validation | Pending | 현장 주행 캘리브레이션 필요 |

---

## 4. First Files to Read / 먼저 읽을 파일

새 합류자는 다음 순서로 읽는 것을 권장합니다. / New contributors should read in this order:

1. [`README.md`](README.md) — 저장소 개요 (현재 문서 / this file).
2. [`docs/SUBMISSION_GUIDE.md`](docs/SUBMISSION_GUIDE.md) — FSG 제출 규격과 마감 일정.
3. [`src/autonomous/AGENTS.md`](src/autonomous/AGENTS.md) — 개발 작업 영역의 운영 규칙.
4. [`submission/README.md`](submission/README.md) — 제출 패키지의 진입점과 실행 절차.
5. [`src/autonomous/config/params.yaml`](src/autonomous/config/params.yaml) — 튜닝 가능한 ROS 파라미터.
6. [`src/autonomous/driver/competition_driver.py`](src/autonomous/driver/competition_driver.py) — 시스템 전체 신호 흐름의 진입점.
7. [`docs/reference_materials/lecture1_fsds_install.txt`](docs/reference_materials/lecture1_fsds_install.txt) — FSDS 설치 절차.

---

## 5. API & Entry Points / API와 진입점

### 5.1 차량 드라이버 진입점 / Vehicle driver entry

| 진입점 / Entry | 위치 / Location | 역할 / Role |
| --- | --- | --- |
| `competition_driver.py` | `src/autonomous/driver/`, `submission/autonomous/driver/` | 모든 노드를 묶는 단일 명령 생성기 |
| `drivers/competition.py` | `submission/src/drivers/` | 제출 패키지용 대회 드라이버 어댑터 |
| `drivers/autonomous.py` | `submission/src/drivers/` | 자율 모드 진입점 |
| `drivers/advanced.py`, `drivers/basic.py` | `submission/src/drivers/` | 단계별 드라이버 (튜닝·디버그용) |

### 5.2 ROS 런치 / Launches

| 런치 / Launch | 위치 / Location | 용도 / Purpose |
| --- | --- | --- |
| `competition.launch` | `submission/launch/` | 대회 제출 단일 런치 (FSG 요구) |
| `bridge_no_camera.launch` | `src/autonomous/config/` | FSDS ↔ ROS 브리지 (카메라 비활성) |
| `params.yaml` | `src/autonomous/config/`, `submission/autonomous/config/` | ROS 파라미터 |

### 5.3 핵심 모듈 / Core modules

| 모듈 / Module | 제공 기능 / Capability | 호출 표면 / Surface |
| --- | --- | --- |
| `perception/cone_detector.py` | 이미지·점군에서 콘 후보 추출 | ROS 토픽 publisher |
| `perception/cone_classifier.py` | 노란/파란 콘 분류 | ROS 토픽 subscriber/publisher |
| `perception/slam.py` | 차량 자세·맵 추정 | ROS 토픽 publisher |
| `control/pure_pursuit.py` | 조향각 산출 | ROS 토픽 subscriber/publisher |
| `control/speed.py` | 종·횡 방향 속도 명령 | ROS 토픽 subscriber/publisher |
| `v2x/rsu.py` | RSU 메시지 수신·파싱 | ROS 토픽 subscriber |
| `utils/lap_timer.py` | 랩 타임 기록·리셋 | ROS 서비스 / 토픽 |
| `utils/watchdog.py` | 안전 상태 모니터링 | ROS 토픽 |

### 5.4 운영 셸 진입점 / Shell entry points

| 스크립트 / Script | 위치 / Location | 용도 / Purpose |
| --- | --- | --- |
| `start.sh` | `src/autonomous/`, `submission/autonomous/` | 로컬 개발용 부팅 |
| `run_all.sh` | `src/autonomous/`, `submission/autonomous/` | 전체 노드 일괄 실행 |
| `entrypoint.sh` | `src/autonomous/` | Docker 컨테이너 진입점 |
| `record_race.sh` | `src/autonomous/` | rosbag 레코딩 |
| `scripts/start_race.py` | `src/autonomous/scripts/` | Python 기반 레이스 시작 |
| `run.sh` | `submission/` | FSG 제출 이미지 표준 실행 |
| `dev.sh` | `submission/` | 제출 패키지 개발 모드 |
| `scripts/package.sh` | `scripts/` | 대회 제출 패키징 |

---

## 6. Quickstart / 빠른 시작

> **사전 요구사항 / Prerequisites:** Ubuntu 20.04, ROS Noetic, Docker + docker-compose, Python 3.x, FSDS.

### 6.1 시뮬레이터에서 실행 / Run in simulator

```bash
# 1) 저장소 클론
git clone <repository-url> fsd-stack && cd fsd-stack

# 2) FSDS 설정 (시뮬레이터는 별도 호스트에서 실행)
#    자세한 절차: docs/reference_materials/lecture1_fsds_install.txt

# 3) 개발용 Docker 이미지 빌드
docker compose -f src/autonomous/docker-compose.yml build

# 4) 시뮬레이터와 ROS 브리지 실행
docker compose -f src/autonomous/docker-compose.yml up
# 또는 호스트에서 직접:
bash src/autonomous/start.sh
```

### 6.2 로컬 ROS 워크스페이스로 실행 / Run as local ROS workspace

```bash
# Noetic 환경
source /opt/ros/noetic/setup.bash

# 의존성 설치
sudo apt install ros-noetic-ackermann-msgs ros-noetic-cv-bridge \
                 ros-noetic-pcl-ros ros-noetic-rosbag

# 개발 작업 영역 빌드
catkin_make -C src/autonomous/

# 전체 노드 실행
bash src/autonomous/run_all.sh
```

### 6.3 대회 제출 패키지 빌드 / Build competition submission

```bash
# 제출용 Docker 이미지 빌드 (FSG 규격)
docker build -t fsd-submission -f submission/Dockerfile submission/

# 표준 실행
docker run --rm --privileged --net=host \
  -v /dev:/dev fsd-submission ./run.sh

# 개발 모드 (호스트 소스 마운트)
docker compose -f submission/docker-compose.yml up
```

자세한 절차는 [`docs/SUBMISSION_GUIDE.md`](docs/SUBMISSION_GUIDE.md)와 [`submission/README.md`](submission/README.md) 참조.

---

## 7. Configuration / 설정

주요 튜닝 지점은 `params.yaml`에集中在되어 있습니다. / Main tuning points are centralized in `params.yaml`.

| 파일 / File | 책임 / Responsibility |
| --- | --- |
| `src/autonomous/config/params.yaml` | 개발 작업 영역 ROS 파라미터 |
| `submission/autonomous/config/params.yaml` | 제출 패키지 ROS 파라미터 (동일 스키마 유지) |
| `src/simulator/settings.json` | FSDS 시뮬레이터 설정 (맵, 차량, 카메라, LiDAR) |
| `submission/src/v2x/rsu.py` 상수 | V2X RSU 채널 ID, 메시지 포맷 |
| `src/autonomous/modules/utils/watchdog.py` | 안전 임계값 (속도·조향 한계) |

제출 패키지의 파라미터는 **대회 직전에 잠그고(lock)** 동결합니다. 그 전까지는 `src/autonomous/config/params.yaml`에서 자유롭게 튜닝합니다.

---

## 8. Commands Reference / 명령어 레퍼런스

| 명령 / Command | 목적 / Purpose |
| --- | --- |
| `bash src/autonomous/start.sh` | 개발용 부팅 (시뮬레이터 연결) |
| `bash src/autonomous/run_all.sh` | 전체 노드 일괄 실행 |
| `bash src/autonomous/record_race.sh` | rosbag 레코딩 |
| `python3 src/autonomous/scripts/start_race.py` | 레이스 시작 시퀀스 |
| `docker compose -f src/autonomous/docker-compose.yml up` | 개발용 컨테이너 기동 |
| `docker compose -f submission/docker-compose.yml up` | 제출 패키지 개발 모드 |
| `bash submission/run.sh` | FSG 제출 이미지 표준 실행 |
| `bash submission/dev.sh` | 제출 패키지 개발 모드 셸 |
| `bash scripts/package.sh` | 대회 제출 패키징 |
| `rostopic list` | 활성 토픽 확인 |
| `rqt_graph` | 노드·토픽 그래프 시각화 |
| `rosbag play <bag>` | 저장된 레이스 재현 |

---

## 9. Local Development / 로컬 개발

### 9.1 개발 워크플로우 / Workflow

1. `src/autonomous/`에서 기능 브랜치를 생성합니다.
2. 모듈 단위로 작업합니다 — `perception/`, `control/`, `utils/`, `v2x/`는 서로 독립적인 책임 영역입니다.
3. `params.yaml` 변경은 대회 직전 동결을 고려해 신중히 합니다.
4. 변경 사항은 동일 인터페이스로 `submission/src/`에 1:1 반영합니다.
5. `src/autonomous/tests/test_algorithms.py`를 실행해 회귀를 확인합니다.

### 9.2 시뮬레이터 ↔ 실차 동기화 / Simulator ↔ vehicle parity

- 동일한 ROS 토픽 이름과 메시지 타입을 사용합니다.
- 알고리즘 코드는 `src/autonomous/`에서 개발·검증하고, 동등한 모듈을 `submission/src/`로 복제합니다.
- 대회 직전에는 `submission/run.sh`로 실차 부팅 시퀀스를 사전 검증합니다.

### 9.3 데이터 / Data

- `in-memoria.db`는 런타임 상태 스냅샷입니다. 디버깅과 재현에 사용되며, **새 기능의 영속 저장소로 사용하지 마세요.**
- rosbag은 `record_race.sh`로 별도 디렉터리에 저장합니다.

---

## 10. Testing / 테스트

| 테스트 / Test | 명령 / Command |
| --- | --- |
| 알고리즘 단위 테스트 | `python3 -m pytest src/autonomous/tests/test_algorithms.py` |
| 시뮬레이터 통합 | `bash src/autonomous/start.sh` 후 콘·경로 출력 확인 |
| 제출 이미지 스모크 테스트 | `docker run --rm fsd-submission ./run.sh --self-test` |

추가로 rqt, rviz를 활용해 토픽과 경로를 시각적으로 검증하는 것을 권장합니다.

---

## 11. Architecture / 아키텍처

모듈형 ROS 노드 구조이며, `competition_driver.py`가 단일 진입점입니다.

### 11.1 모듈 책임 / Module responsibilities

| 계층 / Layer | 모듈 / Module | 입력 / Input | 출력 / Output |
| --- | --- | --- | --- |
| Perception | `cone_detector.py` | 카메라/점군 | 콘 후보 |
| Perception | `cone_classifier.py` | 콘 후보 | 노란/파란 콘 |
| Perception | `slam.py` | LiDAR/IMU | 자세·맵 |
| Planning (in-driver) | `competition_driver.py` | 콘·자세 | 경로 중심선 |
| Control | `pure_pursuit.py` | 경로 | 조향각 |
| Control | `speed.py` | 경로·상태 | 속도 명령 |
| V2X | `rsu.py` | RSU 메시지 | 미션 플래그 |
| Utility | `lap_timer.py` | 시작/종료 신호 | 랩 타임 |
| Utility | `watchdog.py` | 센서·명령 | 안전 플래그 |

### 11.2 신호 흐름 요약 / Signal flow

1. `cone_detector` → 콘 후보 토픽 → `cone_classifier` → 분류된 콘 토픽.
2. `slam` → `/odom`, `/map` → `competition_driver`에 차량 자세 제공.
3. `competition_driver`가 콘·자세를 융합해 경로 중심선을 만들고 `pure_pursuit`, `speed`에 전달.
4. `pure_pursuit` + `speed` → 조향·속도 명령 → 차량 인터페이스.
5. `rsu` → 미션 플래그 → `competition_driver`가 경로 선택·미션 전환에 반영.
6. `lap_timer`와 `watchdog`는 전체 사이클을 모니터링.

### 11.3 워크스페이스 분리 이유 / Workspace split

| 영역 / Area | 목적 / Purpose |
| --- | --- |
| `src/autonomous/` | 자유로운 개발·튜닝, 시뮬레이터 검증 |
| `submission/` | FSG 규격 제출, 대회 환경에서만 수정 최소화 |

두 영역의 알고리즘 코드는 동일 인터페이스를 유지해야 합니다.

---

## 12. Maintainers & Points of Contact / 유지보수와 연락처

| 역할 / Role | 위치 / Location |
| --- | --- |
| 저장소 소유자 / Repository owners | [`OWNERS`](OWNERS) |
| 기여 정책 / Contribution policy | [`CONTRIBUTING.md`](CONTRIBUTING.md) |
| 에이전트 운영 규칙 / Agent rules | [`AGENTS.md`](AGENTS.md), [`src/autonomous/AGENTS.md`](src/autonomous/AGENTS.md), [`submission/AGENTS.md`](submission/AGENTS.md) |

팀 내부 커뮤니케이션 채널과 대회의 주요 연락처는 [`docs/SUBMISSION_GUIDE.md`](docs/SUBMISSION_GUIDE.md)에 정리해 두세요.

---

## 13. Further Documentation / 추가 문서

| 문서 / Document | 위치 / Location |
| --- | --- |
| 대회 제출 절차 / Submission procedure | [`docs/SUBMISSION_GUIDE.md`](docs/SUBMISSION_GUIDE.md) |
| FSDS 설치 / FSDS install guide | [`docs/reference_materials/lecture1_fsds_install.txt`](docs/reference_materials/lecture1_fsds_install.txt) |
| SLAM 참고 / SLAM reference | [`docs/reference_materials/lecture4_slam.ipynb`](docs/reference_materials/lecture4_slam.ipynb) |
| V2X 참고 / V2X reference | [`docs/reference_materials/lecture6_v2x.ipynb`](docs/reference_materials/lecture6_v2x.ipynb) |
| 시뮬레이터 설정 / Simulator config | [`src/simulator/README.md`](src/simulator/README.md), [`src/simulator/settings.json`](src/simulator/settings.json) |
| 제출 패키지 가이드 / Submission package guide | [`submission/README.md`](submission/README.md) |
| 기여 정책 / Contribution policy | [`CONTRIBUTING.md`](CONTRIBUTING.md) |
| 라이선스 / License | [`LICENSE`](LICENSE) |

---

## 14. License / 라이선스

본 저장소는 [`LICENSE`](LICENSE) 파일에 명시된 조건 하에 배포됩니다. 대회 제출 패키지의 자산(이미지, 영상, 지도 등)은 팀 내부 자산이므로 외부 공유 시 별도 검토가 필요합니다.