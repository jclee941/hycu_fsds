# Autonomous Racing Stack (Formula Student Driverless) / 자율주행 레이싱 스택

[![ROS](http://img.shields.io/badge/ROS-Melodic%2FNoetic-22314E.svg)](https://www.ros.org/)
[![Docker](http://img.shields.io/badge/Docker-Ready-2496ED.svg)](https://www.docker.com/)
[![Python](http://img.shields.io/badge/Python-3.x-3776AB.svg)](https://www.python.org/)
[![License](https://img.shields.io/badge/License-See%20LICENSE-blue.svg)](./LICENSE)
[![Maintained](https://img.shields.io/badge/Maintained-Yes-success.svg)](./OWNERS)

## 한국어 요약

이 저장소는 **Formula Student Driverless(FSD)** 대회를 위한 자율주행 레이싱 스택이다.
`src/autonomous/`는 FSDS(Formula Student Driverless Simulator) 기반의 시뮬레이션
실행 환경을, `submission/`은 실차 배포용 경량 Docker 스택을, `src/simulator/`는
시뮬레이터 설정을 제공한다. 콘지킴(cone) 인지, SLAM, Pure Pursuit 경로 추종,
속도 프로파일링, V2X(RSU) 통신, 랩 타이머를 모듈 단위로 분리해 두었으며,
ROS launch 파일과 Docker Compose로 일관된 실행 경로를 보장한다.

- 시뮬레이션에서 알고리즘을 검증한 뒤 동일 코드를 실차 이미지로 패키징한다.
- 제출용 이미지는 워치독(watchdog)과 랩 타이머로 운영 안전성을 확보한다.
- 강의 노트와 레퍼런스 자료를 `docs/reference_materials/`에서 함께 제공한다.

## English Summary

A Formula Student Driverless (FSD) autonomous racing stack. `src/autonomous/`
provides the FSDS-based simulation runner, `submission/` is the lightweight
on-vehicle Docker image, and `src/simulator/` holds simulator settings. The
software is split into perception (cone detection/classification, SLAM),
control (pure pursuit, speed profile), V2X (RSU), and utility (lap timer,
watchdog) modules, all wired through ROS launch files and Docker Compose.

## 한눈에 보기 / At a Glance

| 항목 | 상태 | 위치 |
|---|---|---|
| 시뮬레이션 실행 | 지원 | `src/autonomous/` |
| 시뮬레이터 설정 | 제공 | `src/simulator/` |
| 실차 제출 이미지 | 지원 | `submission/` |
| 콘 인지 (Detection) | 구현됨 | `*/modules/perception/cone_detector.py` |
| 콘 분류 (Color/Class) | 구현됨 | `*/modules/perception/cone_classifier.py` |
| SLAM | 구현됨 | `*/modules/perception/slam.py` |
| 경로 추종 (Pure Pursuit) | 구현됨 | `*/modules/control/pure_pursuit.py` |
| 속도 제어 | 구현됨 | `*/modules/control/speed.py` |
| V2X / RSU | 구현됨 | `submission/src/v2x/rsu.py` |
| 랩 타이머 | 구현됨 | `*/modules/utils/lap_timer.py` |
| 워치독 | 구현됨 | `*/modules/utils/watchdog.py` |
| 알고리즘 단위 테스트 | 포함 | `src/autonomous/tests/` |
| 제출 가이드 | 포함 | `docs/SUBMISSION_GUIDE.md` |
| 강의 자료 | 포함 | `docs/reference_materials/` |
| 운영자(메인테이너) | OWNERS | `./OWNERS` |

## 실행 흐름 한 줄 요약

1. 시뮬레이션 검증 → `cd src/autonomous && docker-compose up`
2. 알고리즘 단위 검증 → `src/autonomous/tests/test_algorithms.py`
3. 실차 이미지 빌드/실행 → `cd submission && ./run.sh` 또는 `./dev.sh`
4. 패키징 → `./scripts/package.sh`

## 목차 / Table of Contents

- [패키지 구성 / Package Contents](#패키지-구성--package-contents)
- [아키텍처 / Architecture](#아키텍처--architecture)
- [빠른 시작 / Quickstart](#빠른-시작--quickstart)
- [명령어 / Commands](#명령어--commands)
- [설정 / Configuration](#설정--configuration)
- [로컬 개발 / Local Development](#로컬-개발--local-development)
- [테스트 / Testing](#테스트--testing)
- [기여 / Contributing](#기여--contributing)
- [운영자 / Maintainers](#운영자--maintainers)
- [라이선스 / License](#라이선스--license)
- [추가 문서 / Further Documentation](#추가-문서--further-documentation)

## 패키지 구성 / Package Contents

| 경로 | 용도 |
|---|---|
| `src/autonomous/` | FSDS 시뮬레이션 자율주행 런타임(Docker, launch, driver, modules) |
| `src/autonomous/driver/` | 대회용 메인 드라이버 (`competition_driver.py`) |
| `src/autonomous/modules/perception/` | 콘 검출·분류, SLAM |
| `src/autonomous/modules/control/` | Pure Pursuit, 속도 |
| `src/autonomous/modules/utils/` | 랩 타이머, 워치독 |
| `src/autonomous/config/` | ROS launch, 파라미터 YAML |
| `src/autonomous/tests/` | 알고리즘 단위 테스트 |
| `src/simulator/` | FSDS 시뮬레이터 설정 |
| `submission/` | 실차 제출용 경량 스택(Docker, launch, src 모듈 미러) |
| `submission/drivers/` | Basic/Advanced/Autonomous/Competition 드라이버 |
| `submission/v2x/` | RSU 통신 모듈 |
| `docs/` | 제출 가이드, 레퍼런스 강의 자료 |
| `scripts/` | 패키징 스크립트 |
| `AGENTS.md`, `CONTRIBUTING.md`, `OWNERS`, `LICENSE` | 거버넌스/기여/운영 문서 |

> 모듈 트리는 `src/autonomous/` 와 `submission/src/` 양쪽에 동일한 인터페이스로
> 미러링되어 있어, 시뮬레이션과 실차 간 코드 차이를 최소화한다.

## 아키텍처 / Architecture

전체 파이프라인은 **인지 → 측위(Localization) → 계획/제어 → 운영(Watchdog)** 의
단방향 흐름이며, V2X 채널은 병렬 입력으로 결합된다.

| 단계 | 모듈 | 책임 |
|---|---|---|
| 입력 | 시뮬레이터 또는 차량 센서 | 카메라, LiDAR, 휠 엔코더, V2X |
| 인지 | `perception/cone_detector.py` | 콘 후보 추출 |
| 인지 | `perception/cone_classifier.py` | 색상/클래스 분류 |
| 측위 | `perception/slam.py` | 트랙 내 자기 위치 추정 |
| 통신 | `v2x/rsu.py` | 인프라(RSU) 메시지 수신 |
| 제어 | `control/pure_pursuit.py` | 경로 추종 스티어링 |
| 제어 | `control/speed.py` | 속도 프로파일 |
| 운영 | `utils/watchdog.py` | 정지 감시·안전 정지 |
| 운영 | `utils/lap_timer.py` | 랩 측정·로그 |
| 통합 | `driver/competition_driver.py` | 위 모듈 오케스트레이션 |

런타임 진입점은 시뮬레이션/실차 두 가지다.

| 진입점 | 실행 방법 | 비고 |
|---|---|---|
| 시뮬레이션 | `src/autonomous/start.sh` 또는 `docker-compose up` | FSDS 연동 |
| 실차 제출 | `submission/run.sh` | 라즈베리파이급 경량 이미지 |
| 개발 셸 | `submission/dev.sh` | 컨테이너 내부 진입 |

## 빠른 시작 / Quickstart

### 사전 요구 사항

- Docker 20.10+ 및 Docker Compose v2
- NVIDIA 런타임(시뮬레이션 GPU 가속 시)
- 호스트에 FSDS(FSDS Docker 이미지 권장)
- ROS 1 지식(Melodic/Noetic)

### 시뮬레이션 실행

```bash
git clone <repo-url> fsd-stack
cd fsd-stack/src/autonomous
docker-compose up --build
# 또는 직접 실행
./start.sh
```

### 단위 테스트

```bash
cd src/autonomous
python -m pytest tests/ -v
```

### 실차 이미지 빌드/실행

```bash
cd submission
./run.sh          # 실차 런타임 컨테이너 실행
./dev.sh          # 개발용 셸 진입 (디버깅)
docker-compose up --build
```

### 패키징

```bash
./scripts/package.sh    # 제출 아티팩트 생성
```

## 명령어 / Commands

| 스크립트 | 목적 |
|---|---|
| `src/autonomous/start.sh` | 시뮬레이션 메인 런처 |
| `src/autonomous/run_all.sh` | 전체 시나리오 실행 |
| `src/autonomous/record_race.sh` | 레이스 데이터 기록/리플레이 |
| `src/autonomous/scripts/start_race.py` | 레이스 시작 트리거 |
| `src/autonomous/entrypoint.sh` | 컨테이너 초기화 |
| `submission/run.sh` | 실차 런타임 컨테이너 실행 |
| `submission/dev.sh` | 개발용 셸 |
| `scripts/package.sh` | 제출물 패키징 |

## 설정 / Configuration

| 설정 파일 | 용도 |
|---|---|
| `src/autonomous/config/params.yaml` | 시뮬레이션 파라미터 (속도, 게인 등) |
| `src/autonomous/config/bridge_no_camera.launch` | ROS ↔ 시뮬레이터 브리지(카메라 제외) |
| `submission/config/params.yaml` | 실차 파라미터 |
| `submission/launch/competition.launch` | 대회 런치 파일 |
| `src/simulator/settings.json` | FSDS 시뮬레이터 설정 |

## 로컬 개발 / Local Development

1. 워크스페이스는 `src/autonomous` 와 `submission` 두 개의 독립 ROS 패키지로
   구성된다. IDE는 VSCode + ROS 확장을 권장한다.
2. 코드 스타일은 PEP 8이며, 신규 모듈은 `modules/` 하위에 도메인별 패키지로
   추가한다.
3. 시뮬레이션 결과를 빠르게 반복하려면 FSDS GUI와 `competition_driver.py`를
   함께 띄우고, 변경 후 `pytest`로 회귀를 확인한다.
4. 실차와 동일한 인터페이스를 유지하기 위해 `submission/src/modules/` 와
   `src/autonomous/modules/` 의 함수 시그니처를 동기화한다.
5. 로컬 IP, 네트워크 엔드포인트 등 환경 의존 값은 YAML로 외부화하고 커밋에서
   제외한다.

## 테스트 / Testing

| 범위 | 도구 | 위치 |
|---|---|---|
| 알고리즘 단위 | `pytest` | `src/autonomous/tests/test_algorithms.py` |
| 시나리오 검증 | 수동 + 레코드 재생 | `record_race.sh` |

테스트 추가 시 동일 디렉터리의 기존 케이스 네이밍 컨벤션을 따른다.

## 기여 / Contributing

기여 절차는 `CONTRIBUTING.md`를 따른다. 일반적으로:

1. 이슈 등록 → 기능 브랜치 생성 → PR 제출
2. CI 통과 및 리뷰 1인 이상 승인 후 머지
3. 거버넌스 변경은 `AGENTS.md` / `OWNERS` 업데이트 동반

운영·자동화 정책 변경은 저장소 OWNERS의 승인을 필수로 한다.

## 운영자 / Maintainers

책임자 목록은 [`OWNERS`](./OWNERS) 파일을 참조한다. 도메인별 책임 영역:

| 영역 | 책임 |
|---|---|
| 인지 (Perception) | OWNERS 참조 |
| 제어 (Control) | OWNERS 참조 |
| V2X | OWNERS 참조 |
| 인프라/Docker | OWNERS 참조 |
| 제출/운영 | OWNERS 참조 |

## 지원 / Help

- 사용 가이드: [`docs/SUBMISSION_GUIDE.md`](./docs/SUBMISSION_GUIDE.md)
- 레퍼런스 강의: [`docs/reference_materials/`](./docs/reference_materials/)
- 시뮬레이션 런너: [`src/simulator/README.md`](./src/simulator/README.md)
- 제출 패키지: [`submission/README.md`](./submission/README.md)
- 기여 절차: [`CONTRIBUTING.md`](./CONTRIBUTING.md)
- 거버넌스: [`AGENTS.md`](./AGENTS.md)

## 라이선스 / License

이 저장소는 [`LICENSE`](./LICENSE) 파일에 명시된 라이선스를 따른다.
사용·배포·수정 전 라이선스 전문을 확인할 것.

## 추가 문서 / Further Documentation

- [`docs/SUBMISSION_GUIDE.md`](./docs/SUBMISSION_GUIDE.md) — 대회 제출 절차
- [`docs/reference_materials/lecture1_fsds_install.txt`](./docs/reference_materials/lecture1_fsds_install.txt) — FSDS 설치
- [`docs/reference_materials/lecture4_slam.ipynb`](./docs/reference_materials/lecture4_slam.ipynb) — SLAM 강의
- [`docs/reference_materials/lecture6_v2x.ipynb`](./docs/reference_materials/lecture6_v2x.ipynb) — V2X 강의
- [`AGENTS.md`](./AGENTS.md) — 저장소 거버넌스/정책