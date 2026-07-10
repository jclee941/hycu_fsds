# Autonomous Racing Stack (FSDS Driverless)

![ROS](https://img.shields.io/badge/ROS-Melodic%2FNoetic-blue.svg)
![Python](https://img.shields.io/badge/Python-3.8%2B-blue.svg)
![Docker](https://img.shields.io/badge/runtime-Docker-blue.svg)
![Status](https://img.shields.io/badge/status-development-yellow.svg)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

## 한국어 요약

Formula Student Driverless(FSDS) 스타일 자율주행 경주차 풀스택입니다.
콘 인식·SLAM·V2X·Pure Pursuit 제어를 한 번에 다루며, `src/autonomous/`는 개발 트리, `submission/`은 대회 제출용 패키지입니다.

## English Summary

A full-stack autonomous race-car system for Formula Student Driverless-style events.
Bundles cone perception, SLAM, V2X, and Pure Pursuit control.
`src/autonomous/` is the development tree; `submission/` is the competition-ready package.

## 상태 / Status

| 항목 / Item | 값 / Value |
| --- | --- |
| Competition class | FSDS / Driverless-style |
| Primary runtime | ROS (Melodic / Noetic) inside Docker |
| Language | Python 3.8+ |
| Sensors | Camera + LiDAR (configurable) |
| Localization | `modules/perception/slam.py` |
| Path tracking | Pure Pursuit + speed profile |
| V2X | RSU telemetry via `v2x/rsu.py` |
| Tests | `src/autonomous/tests/test_algorithms.py` |
| Owner | qws941 — see `OWNERS` |
| License | see `LICENSE` |

## Lap Flow (one pass)

1. `perception/cone_detector.py` → `perception/cone_classifier.py` extract cones.
2. `perception/slam.py` builds the track map and landmark graph.
3. `v2x/rsu.py` consumes race-control telemetry from the road-side unit.
4. `control/pure_pursuit.py` + `control/speed.py` compute steering and throttle.
5. `utils/watchdog.py` guards mission state; `utils/lap_timer.py` logs lap times.
6. `driver/competition_driver.py` orchestrates the lap and exits on `finish`.

## 목차 / Contents

- [패키지 구성 / Package Contents](#패키지-구성--package-contents)
- [먼저 읽을 파일 / First Files to Read](#먼저-읽을-파일--first-files-to-read)
- [빠른 시작 / Quickstart](#빠른-시작--quickstart)
- [설정 / Configuration](#설정--configuration)
- [명령어 / Commands Reference](#명령어--commands-reference)
- [테스트 / Testing](#테스트--testing)
- [로컬 개발 / Local Development](#로컬-개발--local-development)
- [유지보수 / Maintainers](#유지보수--maintainers)
- [추가 문서 / Further Documentation](#추가-문서--further-documentation)

## 패키지 구성 / Package Contents

- `src/autonomous/` — 메인 개발 트리
  - `driver/competition_driver.py` — 한 바퀴 미션 오케스트레이션
  - `modules/perception/` — 콘 감지, 분류, SLAM
  - `modules/control/` — Pure Pursuit, 속도 프로파일
  - `modules/utils/` — 워치독, 랩 타이머
  - `config/params.yaml`, `config/bridge_no_camera.launch` — ROS 설정
  - `tests/test_algorithms.py` — 알고리즘 단위 테스트
- `submission/` — 대회 제출용 패키지
  - `src/drivers/{basic,advanced,autonomous,competition}.py` — 단계별 운전자
  - `src/perception/`, `src/control/`, `src/utils/` — 동일 모듈 미러
  - `src/v2x/rsu.py` — RSU 텔레메트리 수신기
  - `launch/competition.launch`, `Dockerfile` — 실행 컨테이너
  - `autonomous/` — 제출용 자율 컨테이너 번들
- `src/simulator/` — 시뮬레이터 설정 (`settings.json`)
- `scripts/package.sh` — 제출 아티팩트 패키징 스크립트
- `in-memoria.db` — 로컬 SQLite (랩·맵 캐시)
- `docs/SUBMISSION_GUIDE.md` — 대회 제출 절차
- `docs/reference_materials/` — FSDS 설치·SLAM·V2X 강의 노트

## 먼저 읽을 파일 / First Files to Read

| 경로 / Path | 역할 / Role |
| --- | --- |
| `src/autonomous/driver/competition_driver.py` | 미션 시작점 (start.sh가 호출) |
| `src/autonomous/modules/perception/slam.py` | 트랙 맵 빌드 |
| `src/autonomous/modules/control/pure_pursuit.py` | 경로 추종 핵심 |
| `submission/src/drivers/competition.py` | 제출용 운전자 |
| `docs/SUBMISSION_GUIDE.md` | 대회 제출 체크리스트 |

## 빠른 시작 / Quickstart

### 개발 트리 (src/autonomous/)

```bash
cd src/autonomous
docker compose up --build
# 카메라 없이 SLAM만 검증
roslaunch config/bridge_no_camera.launch
```

### 제출용 (submission/)

```bash
cd submission
./dev.sh              # 컨테이너 개발 셸 진입
./run.sh              # 대회 런타임 기동
```

## 설정 / Configuration

- 알고리즘 파라미터: `src/autonomous/config/params.yaml`
- ROS 런치: `src/autonomous/config/bridge_no_camera.launch`
- 시뮬레이터: `src/simulator/settings.json`
- 제출 런치: `submission/launch/competition.launch`

## 명령어 / Commands Reference

| 명령 / Command | 설명 / Description |
| --- | --- |
| `bash src/autonomous/start.sh` | 개발 런타임 시작 |
| `bash src/autonomous/run_all.sh` | 전체 파이프라인 실행 |
| `bash src/autonomous/record_race.sh` | 한 바퀴 기록 |
| `bash src/autonomous/entrypoint.sh` | 컨테이너 진입점 |
| `bash submission/run.sh` | 제출 컨테이너 런타임 |
| `bash submission/dev.sh` | 제출 컨테이너 개발 셸 |
| `bash scripts/package.sh` | 제출 패키지 빌드 |
| `roslaunch config/bridge_no_camera.launch` | 카메라 없이 SLAM 검증 |

## 테스트 / Testing

```bash
cd src/autonomous
pytest tests/test_algorithms.py
```

알고리즘 단위 테스트가 pure pursuit, speed profile, cone detector를 다룹니다.

## 로컬 개발 / Local Development

- Docker 기반 실행을 권장합니다 (호스트 ROS 충돌 회피).
- 시뮬레이터(`src/simulator/`)로 알고리즘을 먼저 검증한 뒤 실제 차량에 이식하세요.
- `docs/reference_materials/`의 강의를 참고해 SLAM/V2X 모듈을 학습할 수 있습니다.
- 새로운 알고리즘은 `modules/` 아래에 추가하고 `tests/test_algorithms.py`로 커버하세요.

## 유지보수 / Maintainers

- Primary owner: qws941 — 자세한 내용은 [`OWNERS`](OWNERS) 참고
- 기여 절차: [`CONTRIBUTING.md`](CONTRIBUTING.md)
- 프로젝트 운영 노트: [`AGENTS.md`](AGENTS.md)

## 추가 문서 / Further Documentation

- [`docs/SUBMISSION_GUIDE.md`](docs/SUBMISSION_GUIDE.md) — 대회 제출 절차
- [`docs/reference_materials/lecture1_fsds_install.txt`](docs/reference_materials/lecture1_fsds_install.txt) — FSDS 설치 가이드
- [`docs/reference_materials/lecture4_slam.ipynb`](docs/reference_materials/lecture4_slam.ipynb) — SLAM 튜토리얼
- [`docs/reference_materials/lecture6_v2x.ipynb`](docs/reference_materials/lecture6_v2x.ipynb) — V2X 튜토리얼
- [`src/simulator/README.md`](src/simulator/README.md) — 시뮬레이터 안내
- [`submission/README.md`](submission/README.md) — 제출 패키지 안내