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

### 운영 흐름 / Operator Flow

1. 개발자는 `src/autonomous/`에서 알고리즘을 수정·단위 테스트합니다.
2. 검증된 변경 사항은 `submission/` 워크스페이스로 동기화하여 Docker 이미지로 패키징합니다.
3. `submission/run.sh`는 대회용 `competition.launch`를 호출해 SLAM → 콘 감지 → 콘 분류 → 추종 제어 → V2X → 랩 타이머 파이프라인을 기동합니다.
4. 워치독이 각 노드의 건강 상태를 감시하고, 랩 타이머가 규정 준수 여부를 기록합니다.

1. Developers iterate on algorithms under `src/autonomous/` and run unit tests.
2. Validated changes are mirrored into `submission/` and packaged into a Docker image.
3. `submission/run.sh` boots `competition.launch`, which starts the SLAM → cone detection → cone classification → pursuit → V2X → lap-timer pipeline.
4. The watchdog monitors node health while the lap timer records compliance with the event rules.

## 목차 / Table of Contents

- [Purpose / Package Contents](#purpose--package-contents)
- [Status](#status)
- [First Files to Read](#first-files-to-read)
- [API / Entry Points](#api--entry-points)
- [Quickstart / Usage](#quickstart--usage)
- [Architecture](#architecture)
- [Configuration](#configuration)
- [Commands Reference](#commands-reference)
- [Local Development](#local-development)
- [Testing](#testing)
- [Maintainers / Points of Contact](#maintainers--points-of-contact)
- [Contribution Guide](#contribution-guide)
- [Further Documentation](#further-documentation)
- [License](#license)

## Purpose / Package Contents

| 영역 / Area | 경로 / Path | 역할 / Role |
| --- | --- | --- |
| Development workspace | `src/autonomous/` | 알고리즘 개발·시뮬레이션·단위 테스트 / Active algorithm dev, sim runs, unit tests |
| Competition submission | `submission/` | FSG 규격 Docker 제출물 / FSG-compliant Docker image for on-site runs |
| Simulator settings | `src/simulator/` | FSDS 시뮬레이터 환경 설정 / FSDS simulator scene/config |
| Reference docs | `docs/` | 제출 가이드 및 강의 자료 / Submission guide and reference material |
| Packaging scripts | `scripts/` | 보조 빌드 스크립트 / Helper packaging scripts |

이 저장소는 **자율주행 알고리즘 개발 → 시뮬레이션 검증 → 대회 제출** 전 과정을 단일 저장소에서 다루는 것을 목표로 합니다. 동일한 모듈 트리(`perception/`, `control/`, `utils/`, `v2x/`)가 개발·제출 양쪽에 중복 배치되어, 실차 시점의 배포 차이를 최소화합니다.

This repository covers the full lifecycle from algorithm authoring to simulator validation to on-site submission. The same module tree (`perception/`, `control/`, `utils/`, `v2x/`) is mirrored between the dev workspace and the submission image to minimize deployment drift between sim and real car.

## Status

- **Competition readiness:** `src/autonomous/`는 대회 운영 가능한 상태이며, `submission/`은 FSG 제출 규격의 부트/런타임 스크립트를 포함합니다. 현장 도입 전 실제 차량 센서 캘리브레이션 및 트랙 검증을 권장합니다.
- **Maintenance:** 핵심 알고리즘은 적극적으로 유지보수되며, 시뮬레이터 변경 또는 규격 갱신 시 `docs/SUBMISSION_GUIDE.md`를 함께 갱신합니다.
- **Stability:** 노드 간 토픽 계약(`/slam_map`, `/cones`, `/drive`, `/v2x/rsu`)은 안정적이며, 변경 시 `CHANGELOG`에 기록합니다.

- **Competition-ready** with on-site bring-up and calibration still required.
- **Actively maintained;** updates to the FSG rulebook or FSDS trigger docs refresh.
- Topic contracts (`/slam_map`, `/cones`, `/drive`, `/v2x/rsu`) are considered stable.

## First Files to Read

| 순서 / Order | 파일 / File | 이유 / Why read it |
| --- | --- | --- |
| 1 | `README.md` | 저장소 전반의 개요 / High-level overview |
| 2 | `docs/SUBMISSION_GUIDE.md` | 제출 절차·제약 / On-site submission process and constraints |
| 3 | `src/autonomous/AGENTS.md` | 개발 워크스페이스 규칙 / Dev workspace conventions |
| 4 | `submission/AGENTS.md` | 제출 패키지 규칙 / Submission package conventions |
| 5 | `src/autonomous/driver/competition_driver.py` | 차량 엔트리 포인트 / Vehicle entry point |
| 6 | `submission/launch/competition.launch` | 제출 런타임 그래프 / Submission runtime graph |
| 7 | `src/autonomous/config/params.yaml` | 튜닝 가능한 파라미터 / Tunable parameters |
| 8 | `src/simulator/README.md` | FSDS 시뮬레이터 연동 / Simulator integration notes |

## API / Entry Points

### 차량 런타임 / Vehicle Runtime

| 엔트리 / Entry | 경로 / Path | 설명 / Description |
| --- | --- | --- |
| 메인 드라이버 / Main driver | `src/autonomous/driver/competition_driver.py` | 콘 감지→추종→V2X를 오케스트레이션 / Orchestrates perception→control→V2X |
| SLAM 노드 / SLAM node | `src/autonomous/modules/perception/slam.py` | LiDAR 기반 지도 작성 / LiDAR-based mapping |
| 콘 감지 / Cone detection | `src/autonomous/modules/perception/cone_detector.py` | LiDAR 클러스터링으로 콘 후보 추출 / Cone candidates from LiDAR |
| 콘 분류 / Cone classification | `src/autonomous/modules/perception/cone_classifier.py` | 색상/타입 분류 (노란·파란) / Color/type classification |
| Pure Pursuit | `src/autonomous/modules/control/pure_pursuit.py` | 경로 추종 제어 / Path-following controller |
| 속도 제어 / Speed control | `src/autonomous/modules/control/speed.py` | 곡률 기반 속도 프로파일 / Curvature-based speed profile |
| V2X RSU | `src/autonomous/modules/v2x/rsu.py` | 노변 기지국과의 메시지 교환 / RSU message exchange |
| 랩 타이머 / Lap timer | `src/autonomous/modules/utils/lap_timer.py` | 랩 측정·규정 검증 / Lap measurement & rules check |
| 워치독 / Watchdog | `src/autonomous/modules/utils/watchdog.py` | 노드 헬스 모니터링 / Node health monitoring |

### 제출 런타임 / Submission Runtime

| 엔트리 / Entry | 경로 / Path | 설명 / Description |
| --- | --- | --- |
| 부트 스크립트 / Boot script | `submission/run.sh` | Docker 컨테이너 부팅 / Boots the submission container |
| 개발 부트 / Dev boot | `submission/dev.sh` | 인터랙티브 셸 진입 / Interactive shell entry |
| 런치 파일 / Launch file | `submission/launch/competition.launch` | 제출 런타임 토픽 그래프 / Topic graph for on-site runs |
| 드라이버 모듈 / Driver modules | `submission/src/drivers/` | basic · autonomous · advanced · competition |

## Quickstart / Usage

### 1. 시뮬레이터에서 실행 / Run in Simulator (FSDS)

```bash
# 1) FSDS 시뮬레이터를 별도로 실행 (공식 빌드 사용)
#    Launch FSDS separately using its official build.

# 2) 개발 워크스페이스 부팅
cd src/autonomous
docker compose -f docker-compose.yml up
# 또는 / or
./start.sh
```

### 2. 단위 테스트 / Unit Tests

```bash
cd src/autonomous
python -m pytest tests/
```

### 3. 대회 제출 이미지 빌드 / Build Submission Image

```bash
cd submission
docker build -t fsd-submission:latest .
```

### 4. 제출 컨테이너 실행 / Run Submission Container

```bash
cd submission
./run.sh                # 대회 부팅 / competition boot
./dev.sh                # 인터랙티브 디버그 셸 / interactive debug shell
```

### 5. 랩 레코드 / Record a Race Run

```bash
cd src/autonomous
./record_race.sh
./run_all.sh
```

> 자세한 운영 절차는 `docs/SUBMISSION_GUIDE.md`를 참조하세요. See `docs/SUBMISSION_GUIDE.md` for the full on-site procedure.

## Architecture

### 모듈 구성 / Module Layout

| 레이어 / Layer | 모듈 / Modules | 책임 / Responsibility |
| --- | --- | --- |
| Perception | `perception/slam.py`, `perception/cone_detector.py`, `perception/cone_classifier.py` | 지도 작성과 콘 인식 / Mapping and cone recognition |
| Control | `control/pure_pursuit.py`, `control/speed.py` | 조향·속도 명령 생성 / Steering and throttle commands |
| V2X | `v2x/rsu.py` | RSU 메시지 송수신 / RSU message exchange |
| Utils | `utils/lap_timer.py`, `utils/watchdog.py` | 랩 측정·헬스체크 / Lap timing and health checks |
| Driver | `driver/competition_driver.py`, `drivers/{basic,autonomous,advanced,competition}.py` | 런타임 오케스트레이션 / Runtime orchestration |

### 런타임 흐름 / Runtime Flow

1. **센서 입력 / Sensor ingest** — LiDAR/카메라 토픽을 ROS로 수신.
2. **SLAM** — `slam.py`가 차량 주변 지도를 갱신하고 `/slam_map`을 퍼블리시.
3. **Cone detection** — `cone_detector.py`가 LiDAR 클러스터에서 콘 후보를 추출.
4. **Cone classification** — `cone_classifier.py`가 색상(노란·파란)과 타입(좌·우 콘)을 라벨링.
5. **Path planning & control** — `pure_pursuit.py`가 중간 경로 점을 계산하고 `speed.py`가 곡률 기반 속도를 결정.
6. **V2X** — `rsu.py`가 RSU와 위치·상태 메시지를 교환.
7. **Lap timing & watchdog** — `lap_timer.py`가 랩 시간을 기록하고 `watchdog.py`가 노드 헬스를 감시.

## Configuration

| 파일 / File | 용도 / Purpose |
| --- | --- |
| `src/autonomous/config/params.yaml` | 시뮬/개발용 노드 파라미터 (토픽, 임계값) / Dev/sim node parameters |
| `src/autonomous/config/bridge_no_camera.launch` | 카메라 제외 ROS 브리지 / ROS bridge without camera topic |
| `submission/autonomous/config/params.yaml` | 제출 환경 파라미터 / Submission environment parameters |
| `src/simulator/settings.json` | FSDS 시뮬레이터 씬 설정 / FSDS scene configuration |

일반적인 튜닝 항목 / Common tuning keys:

| 키 / Key | 영향 / Effect |
| --- | --- |
| `cone_detector.cluster_tolerance` | 콘 클러스터 민감도 / Clustering sensitivity |
| `cone_classifier.color_threshold` | 노란/파란 콘 분류 임계값 / Yellow vs blue threshold |
| `pure_pursuit.lookahead_distance` | 추종 거리 → 안정성 vs 반응성 / Stability vs reactivity |
| `speed.curvature_gain` | 곡률 기반 감속 강도 / Curvature-based braking |
| `lap_timer.start_finish_zone` | 랩 시작/종료 영역 정의 / Start/finish zone definition |

## Commands Reference

| 명령 / Command | 위치 / Where | 설명 / Description |
| --- | --- | --- |
| `./start.sh` | `src/autonomous/`, `submission/autonomous/` | ROS 런타임 부팅 / Boot ROS runtime |
| `./run_all.sh` | `src/autonomous/`, `submission/autonomous/` | 전체 파이프라인 실행 / Run full pipeline |
| `./record_race.sh` | `src/autonomous/` | 레이스 로그 기록 / Record race logs |
| `./entrypoint.sh` | `src/autonomous/` | 컨테이너 진입점 / Container entrypoint |
| `docker compose up` | `src/autonomous/`, `submission/`, `submission/autonomous/` | compose 기반 기동 / Compose-based boot |
| `python driver/competition_driver.py` | `src/autonomous/` | 드라이버 단독 실행 / Driver standalone |
| `pytest tests/` | `src/autonomous/tests/` | 알고리즘 단위 테스트 / Algorithm unit tests |
| `./package.sh` | `scripts/` | 제출 패키징 도우미 / Submission packaging helper |

## Local Development

1. **환경 준비 / Prereqs** — Docker 20+, Docker Compose, 호스트에 ROS 1 Noetic 의존성(네이티브 실행 시).
2. **개발 워크스페이스 / Dev workspace**

   ```bash
   git clone <repo-url>
   cd <repo>
   cd src/autonomous
   docker compose -f docker-compose.yml run --rm fsd-dev bash
   ```

3. **알고리즘 변경 / Iterate on algorithms** — `modules/perception/`, `modules/control/`, `modules/v2x/`의 스크립트를 수정한 뒤 단위 테스트를 실행합니다.
4. **제출 동기화 / Sync into submission** — 변경이 검증되면 동일 모듈을 `submission/src/`에 미러링하고 `submission/autonomous/config/params.yaml`을 함께 갱신합니다.
5. **이미지 빌드 / Build image** — `cd submission && docker build -t fsd-submission:dev .`
6. **시뮬레이션 회귀 / Sim regression** — FSDS에서 컨테이너를 띄우고 `/slam_map`, `/cones`, `/drive` 토픽을 시각화합니다.

## Testing

| 테스트 / Test | 위치 / Path | 명령 / Command |
| --- | --- | --- |
| 알고리즘 단위 테스트 / Algorithm unit tests | `src/autonomous/tests/test_algorithms.py` | `pytest src/autonomous/tests/` |
| 시뮬레이터 회귀 / Simulator regression | `src/simulator/` (FSDS) | FSDS + `docker compose up` 후 `/cones`, `/drive` 모니터링 |
| 런치 검증 / Launch verification | `submission/launch/` | `roslaunch submission/launch/competition.launch` (dry) |
| Docker 빌드 검증 / Docker build smoke | `submission/Dockerfile`, `submission/autonomous/Dockerfile` | `docker build` 후 `./run.sh` dry-run |

테스트 작성 시 모듈 순수 함수(예: `pure_pursuit`, `cone_classifier`)를 우선 커버하고, 통합 검증은 FSDS에서 수행합니다. 컨테이너 외부에서 실행 가능한 함수는 단위 테스트로 유지합니다.

When adding tests, prioritize pure functions (e.g., `pure_pursuit`, `cone_classifier`) and keep integration coverage to FSDS runs.

## Maintainers / Points of Contact

| 역할 / Role | 위치 / Where |
| --- | --- |
| 저장소 오너 / Repository owners | `OWNERS` |
| 개발 워크스페이스 컨벤션 / Dev workspace conventions | `src/autonomous/AGENTS.md` |
| 제출 패키지 컨벤션 / Submission package conventions | `submission/AGENTS.md` |
| 운영 절차 / Operational procedure | `docs/SUBMISSION_GUIDE.md` |

## Contribution Guide

1. 이슈 또는 PR 생성 전 `OWNERS`와 `AGENTS.md`를 확인해 담당 영역을 파악합니다.
2. 알고리즘 변경은 `src/autonomous/`에서 시작하며, `src/autonomous/tests/`로 단위 테스트를 동반합니다.
3. 검증된 변경은 `submission/` 워크스페이스에 동기화하고 `docs/SUBMISSION_GUIDE.md`에 운영 영향이 있다면 함께 갱신합니다.
4. 커밋 메시지는 변경 범위(예: `perception:`, `control:`, `submission:`) 접두어로 시작합니다.
5. PR은 컨테이너 빌드 + 시뮬레이터 회귀 + 단위 테스트 통과를 게이트로 요구합니다.

1. Read `OWNERS` and `AGENTS.md` to identify the owning area.
2. Land algorithm changes in `src/autonomous/` with matching unit tests.
3. Mirror validated changes into `submission/` and update `docs/SUBMISSION_GUIDE.md` when ops are affected.
4. Prefix commits by area (`perception:`, `control:`, `submission:`, …).
5. PRs must pass container build, simulator regression, and unit tests.

자세한 절차는 `CONTRIBUTING.md`를 참조하세요. See `CONTRIBUTING.md` for the full policy.

## Further Documentation

| 문서 / Document | 경로 / Path |
| --- | --- |
| 제출 가이드 / Submission guide | `docs/SUBMISSION_GUIDE.md` |
| FSDS 설치 참고 / FSDS install reference | `docs/reference_materials/lecture1_fsds_install.txt` |
| SLAM 강의 자료 / SLAM lecture | `docs/reference_materials/lecture4_slam.ipynb` |
| V2X 강의 자료 / V2X lecture | `docs/reference_materials/lecture6_v2x.ipynb` |
| 시뮬레이터 노트 / Simulator notes | `src/simulator/README.md` |
| 개발 워크스페이스 규칙 / Dev workspace rules | `src/autonomous/AGENTS.md` |
| 제출 패키지 규칙 / Submission rules | `submission/AGENTS.md` |
| 제출 런타임 노트 / Submission runtime notes | `submission/README.md` |

## License

이 저장소의 라이선스는 [`LICENSE`](LICENSE) 파일을 참조하세요. 외부 의존성(ROS, FSDS, Python 패키지)은 각 프로젝트의 라이선스를 따릅니다.

See [`LICENSE`](LICENSE) for the repository license. Third-party components (ROS, FSDS, Python packages) retain their respective upstream licenses.