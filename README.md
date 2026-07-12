# Autonomous Racing Stack (FSD) / 자율주행 레이싱 스택

[![ROS](http://img.shields.io/badge/ROS-Noetic-22314E?style=flat-square&logo=ros)](https://wiki.ros.org)
[![Python](http://img.shields.io/badge/Python-3.8%2B-3776AB?style=flat-square&logo=python)](https://www.python.org)
[![Docker](http://img.shields.io/badge/Docker-Ready-2496ED?style=flat-square&logo=docker)](https://www.docker.com)
[![Platform](http://img.shields.io/badge/Platform-FSD%20%2F%20FSDS-EA3324?style=flat-square)]()

Formula Student Driverless(FSD) 규격의 자율주행 레이싱 스택입니다.
원격 인지(cone detection, SLAM), 경로 추종(pure pursuit), 속도 제어, 랩 타이밍, V2X(RSU) 모듈을
한 번의 `docker compose up`으로 실행할 수 있도록 구성되어 있습니다.

A Formula Student Driverless autonomous racing stack.
Combines perception (cone detection, SLAM), path tracking (pure pursuit), speed control,
lap timing and V2X (RSU) modules so a full race run launches with one `docker compose up`.

---

## 한눈에 보기 / At a Glance

| 항목 / Item | 값 / Value |
| --- | --- |
| 대상 대회 / Target comp. | Formula Student Driverless (CV / DV / EC) |
| 시뮬레이터 / Simulator | FSDS (Formula Student Driverless Simulator) |
| 주요 언어 / Language | Python 3.8+, ROS launch YAML |
| 실행 환경 / Runtime | Docker / docker compose, ROS Noetic |
| 진입점 / Entry point | `submission/run.sh` 또는 `src/autonomous/start.sh` |
| 컨테이너 빌드 / Build | `submission/Dockerfile`, `src/autonomous/Dockerfile` |
| 제출 패키지 / Submission | `submission/` (Docker 기반 재현 가능) |
| 문서 / Docs | `docs/SUBMISSION_GUIDE.md` |
| 라이선스 / License | `LICENSE` (저장소 참조) |
| 상태 / Status | 실험적 / Experimental — 대회 제출용 |

## 실행 흐름 요약 / Runtime Flow

1. `submission/run.sh` 또는 `src/autonomous/start.sh` 실행
2. Docker 이미지가 빌드되고 ROS 컨테이너가 기동
3. `competition.launch` 또는 `bridge_no_camera.launch` 로 노드 그래프 활성화
4. 인지 노드(cone_detector, cone_classifier, slam)가 트랙을 추정
5. 제어 노드(pure_pursuit, speed)가 스티어링·스로틀 명령 생성
6. `competition_driver.py` 가 시뮬레이터/차량과 인터페이스
7. `lap_timer.py` + `watchdog.py` 가 랩·안전 상태를 모니터링
8. V2X/RSU 통신이 필요한 경우 `v2x/rsu.py` 가 활성

---

## 목차 / Table of Contents

1. [목적 / Purpose](#목적--purpose)
2. [구성 / Package Contents](#구성--package-contents)
3. [상태 / Status](#상태--status)
4. [먼저 읽을 파일 / First Files to Read](#먼저-읽을-파일--first-files-to-read)
5. [진입점 / Entry Points](#진입점--entry-points)
6. [빠른 시작 / Quickstart](#빠른-시작--quickstart)
7. [명령어 레퍼런스 / Commands Reference](#명령어-레퍼런스--commands-reference)
8. [아키텍처 / Architecture](#아키텍처--architecture)
9. [설정 / Configuration](#설정--configuration)
10. [로컬 개발 / Local Development](#로컬-개발--local-development)
11. [테스트 / Testing](#테스트--testing)
12. [기여 / Contribution Guide](#기여--contribution-guide)
13. [유지보수자 / Maintainers](#유지보수자--maintainers)
14. [추가 문서 / Further Documentation](#추가-문서--further-documentation)
15. [라이선스 / License](#라이선스--license)

---

## 목적 / Purpose

이 저장소는 FSD(Formula Student Driverless) 규칙에 맞는 자율주행 레이싱 소프트웨어를
제공합니다. 인지 → 위치 추정 → 경로 계획 → 제어 → 랩 타이밍까지의 전체 루프를
ROS 노드 단위로 모듈화하여 시뮬레이터(FSDS)와 실차에서 동일하게 실행할 수 있도록 설계했습니다.

This repository delivers a complete autonomous racing loop (perception → localization →
planning → control → lap timing) modularized as ROS nodes so the same stack runs on
both the FSDS simulator and a real vehicle.

### 사용 대상 / Who Uses It

- FSD/FSG 팀의 자율주행 SW 개발자
- 대회 제출용 Docker 패키지를 빠르게 만들고 싶은 팀
- SLAM·pure pursuit·V2X 단일 모듈만 실험해 보고 싶은 연구자

## 구성 / Package Contents

| 경로 / Path | 역할 / Role |
| --- | --- |
| `src/autonomous/` | 개발용 워크스페이스 (driver, perception, control, utils) |
| `src/autonomous/driver/` | 시뮬레이터/차량과의 입출력 어댑터 |
| `src/autonomous/modules/perception/` | cone detection, classifier, SLAM |
| `src/autonomous/modules/control/` | pure pursuit, speed controller |
| `src/autonomous/modules/utils/` | lap timer, watchdog |
| `src/autonomous/config/` | launch 파일, 파라미터 YAML |
| `src/simulator/` | FSDS 환경 설정 (`settings.json`) |
| `submission/` | 대회 제출용 Docker 패키지 (재현 가능 빌드) |
| `submission/src/` | 제출 트리에 포함되는 정제된 노드 집합 |
| `submission/launch/` | 대회 평가용 launch 파일 |
| `docs/SUBMISSION_GUIDE.md` | 제출 절차 가이드 |
| `docs/reference_materials/` | 강의 노트, SLAM·V2X 참고 자료 |
| `scripts/package.sh` | 제출 산출물 패키징 |
| `in-memoria.db` | 학습/메모리 캐시 데이터베이스 |

## 상태 / Status

- 실험적 단계입니다. 대회 제출용으로 검증되었으나, 노드별 단위 테스트는 일부만 존재합니다.
- Experimental. Validated for competition submission; full per-node test coverage is partial.
- `src/autonomous/tests/` 에 알고리즘 단위 테스트가 있으며, 통합 테스트는 시뮬레이터 환경에서
  수동으로 수행합니다.
- 실차 적용 시 별도의 안전 리뷰가 필요합니다.

## 먼저 읽을 파일 / First Files to Read

| 순서 / Order | 파일 / File | 이유 / Why |
| --- | --- | --- |
| 1 | `docs/SUBMISSION_GUIDE.md` | 제출 절차와 규약 확인 |
| 2 | `src/autonomous/config/params.yaml` | 조정 가능한 파라미터 확인 |
| 3 | `src/autonomous/driver/competition_driver.py` | 메인 루프 구조 |
| 4 | `submission/launch/competition.launch` | 대회 평가 시 활성 노드 |
| 5 | `src/autonomous/AGENTS.md` | 자율 워크스페이스 규약 |

## 진입점 / Entry Points

| 진입점 / Entry | 용도 / Purpose | 호출 예 / Example |
| --- | --- | --- |
| `submission/run.sh` | 대회 제출 시 컨테이너 실행 | `./submission/run.sh` |
| `submission/dev.sh` | 개발 모드 (소스 마운트) | `./submission/dev.sh` |
| `src/autonomous/start.sh` | 개발 워크스페이스 부팅 | `./src/autonomous/start.sh` |
| `src/autonomous/run_all.sh` | 전체 노드 일괄 기동 | `./src/autonomous/run_all.sh` |
| `src/autonomous/scripts/start_race.py` | 레이스 시퀀서 | `python3 start_race.py` |
| `src/autonomous/driver/competition_driver.py` | 단일 노드 메인 | `rosrun ... competition_driver.py` |
| `scripts/package.sh` | 제출 산출물 빌드 | `bash scripts/package.sh` |

## 빠른 시작 / Quickstart

### 사전 요구 / Prerequisites

- Docker 20+, docker compose v2
- NVIDIA Container Toolkit (GPU 사용 시)
- Linux 호스트 권장 (Ubuntu 20.04 검증)
- ROS Noetic (호스트에서 직접 빌드 시)

### 컨테이너로 실행 / Run in Container

```bash
git clone <this-repo>
cd <this-repo>
docker compose -f submission/docker-compose.yml build
bash submission/run.sh
```

### 로컬 ROS 워크스페이스로 실행 / Local ROS Workspace

```bash
cd src/autonomous
catkin_make || colcon build
source devel/setup.bash
bash start.sh
```

## 명령어 레퍼런스 / Commands Reference

| 명령 / Command | 설명 / Description |
| --- | --- |
| `docker compose -f submission/docker-compose.yml up` | 제출 스택 기동 |
| `docker compose -f src/autonomous/docker-compose.yml up` | 개발 스택 기동 |
| `bash submission/run.sh` | 평가용 레이스 실행 |
| `bash src/autonomous/run_all.sh` | 개발 모드 일괄 실행 |
| `bash src/autonomous/record_race.sh` | 레이스 데이터 기록 |
| `bash scripts/package.sh` | 제출 아카이브 생성 |
| `python3 -m pytest src/autonomous/tests` | 단위 테스트 실행 |

## 아키텍처 / Architecture

노드 그래프는 ROS launch 파일에 의해 결정되며, 데이터 흐름은 아래와 같습니다.

1. **Input 어댑터** — `competition_driver.py` 가 시뮬레이터/차량 토픽 구독
2. **인지 (Perception)** — `cone_detector.py` → `cone_classifier.py` → `slam.py`
3. **위치 추정 / 맵 빌드** — SLAM 출력이 트랙 중심선과 경계 콘 맵 생성
4. **제어 (Control)** — `pure_pursuit.py` 가 경로 추종, `speed.py` 가 속도 결정
5. **명령 출력** — `competition_driver.py` 가 스티어링·스로틀·브레이크 발행
6. **모니터링** — `lap_timer.py` 가 랩 기록, `watchdog.py` 가 안전 상태 감시
7. **V2X (선택)** — `v2x/rsu.py` 가 외부 인프라(RSU)와 데이터 송수신

### 모듈 책임 / Module Responsibilities

| 모듈 / Module | 책임 / Responsibility | 위치 / Location |
| --- | --- | --- |
| `cone_detector` | LiDAR/카메라 입력에서 콘 후보 추출 | `src/autonomous/modules/perception/` |
| `cone_classifier` | 콘 색상/종류 분류 (대/소, 좌/우 경계) | `src/autonomous/modules/perception/` |
| `slam` | 콘 기반 SLAM, 차량 위치 추정 | `src/autonomous/modules/perception/` |
| `pure_pursuit` | 곡률 기반 스티어링 | `src/autonomous/modules/control/` |
| `speed` | 구간별 목표 속도 생성 | `src/autonomous/modules/control/` |
| `lap_timer` | 랩 카운트 및 시간 기록 | `src/autonomous/modules/utils/` |
| `watchdog` | 노드 헬스 체크 및 페일세이프 | `src/autonomous/modules/utils/` |
| `rsu` | V2X 인프라 통신 | `src/autonomous/modules/v2x/` (제출 패키지) |
| `competition_driver` | 시뮬레이터/차량 IO 어댑터 | `src/autonomous/driver/` |

## 설정 / Configuration

주요 설정은 `src/autonomous/config/params.yaml` 에 있습니다.
대표 키는 다음과 같습니다.

| 키 / Key | 의미 / Meaning | 비고 / Notes |
| --- | --- | --- |
| `vehicle.*` | 차량 치수, 휠베이스 등 | pure pursuit 입력 |
| `perception.*` | 콘 감지 임계값, ROI | 튜닝 대상 |
| `control.lookahead` | pure pursuit 전방주시 거리 | m 단위 |
| `control.target_speed` | 기본 목표 속도 | m/s |
| `slam.*` | SLAM 파라미터 (loop closure 등) | 대회 트랙 의존 |
| `v2x.enabled` | RSU 통신 사용 여부 | bool |
| `safety.watchdog_timeout` | 노드 헬스 타임아웃 | 초 |

튜닝 후에는 `submission/` 디렉토리에 변경된 YAML을 반영하고 다시 빌드하세요.

## 로컬 개발 / Local Development

1. `submission/dev.sh` 로 컨테이너에 소스 디렉토리를 마운트합니다.
2. 변경한 Python 노드는 컨테이너 재시작 없이 `rosnode restart` 또는 launch 재실행으로 반영합니다.
3. `in-memoria.db` 는 로컬 캐시이므로 버전 관리에서 제외하거나 정기 백업합니다.
4. 디버깅 시 `rqt_graph`, `rviz` 로 토픽 흐름을 시각화합니다.

## 테스트 / Testing

- 단위 테스트: `pytest src/autonomous/tests/test_algorithms.py`
- 통합 테스트: FSDS 시뮬레이터에서 `bash src/autonomous/run_all.sh` 실행 후 콘솔 출력 확인
- 회귀 테스트: `scripts/package.sh` 로 패키징이 성공하는지 확인 (제출 직전 필수)

## 기여 / Contribution Guide

1. 이슈 등록 → 브랜치 생성 → PR 제출의 흐름을 따릅니다.
2. `CONTRIBUTING.md` 의 코딩 컨벤션과 커밋 메시지 규칙을 따릅니다.
3. PR에는 변경 노드, 테스트 결과, 대회 영향도를 명시합니다.
4. 대회 직전 일정에는 별도의 freeze 정책이 적용될 수 있습니다.

## 유지보수자 / Maintainers

`OWNERS` 파일에 명시된 팀원이 리뷰 및 머지 권한을 가집니다.
조직 변경 시 `OWNERS` 파일을 우선 갱신하고 README의 다음 섹션을 같이 업데이트합니다.

| 역할 / Role | 책임 / Responsibility |
| --- | --- |
| Perception 리드 | cone detection / SLAM |
| Control 리드 | pure pursuit / speed |
| Integration 리드 | driver, launch, Docker |
| Submission 리드 | 대회 제출 패키지, 문서 |

## 추가 문서 / Further Documentation

| 문서 / Doc | 위치 / Path |
| --- | --- |
| 제출 가이드 | `docs/SUBMISSION_GUIDE.md` |
| FSDS 설치 노트 | `docs/reference_materials/lecture1_fsds_install.txt` |
| SLAM 강의 노트 | `docs/reference_materials/lecture4_slam.ipynb` |
| V2X 강의 노트 | `docs/reference_materials/lecture6_v2x.ipynb` |
| 자율 워크스페이스 규약 | `src/autonomous/AGENTS.md` |
| 제출 트리 규약 | `submission/AGENTS.md` |
| 시뮬레이터 설정 | `src/simulator/README.md` |

## 라이선스 / License

저장소 루트의 `LICENSE` 파일을 참조하십시오.
대부분의 모듈은 팀 내부 사용을 전제로 하며, 외부 배포 시에는 라이선스 조건을 확인하세요.