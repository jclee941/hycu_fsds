# FSD 자율주행 시스템 (Formula Student Driverless Stack)

![ROS2](https://img.shields.io/badge/ROS2-Humble-F74C00)
![Python](https://img.shields.io/badge/Python-3.10-3776AB)
![Docker](https://img.shields.io/badge/Docker-ready-2496ED)
![Status](https://img.shields.io/badge/Status-Competition--Ready-2EA043)
![License](https://img.shields.io/badge/License-Check%20LICENSE-blue)

## 한국어 요약

Formula Student Driverless Simulator(FSDS) 대회를 위한 ROS2 기반 자율주행 스택입니다.
콘 인식, SLAM, Pure Pursuit 경로 추종, 속도 제어, V2X(RSU), 랩 타이머·워치독을 한 묶음으로
제공하며, `src/autonomous/`(개발·시뮬레이터)와 `submission/`(대회 제출) 두 런타임으로
동작합니다.

## English Summary

ROS2-based autonomous driving stack for the Formula Student Driverless (FSD) competition.
It packages cone perception, SLAM, pure-pursuit path following, speed control, V2X (RSU),
and lap-timer/watchdog utilities into two runtimes: `src/autonomous/` for development and
`submission/` for the official competition submission.

## Quick Status

| 항목 | 값 |
| --- | --- |
| 대상 대회 | Formula Student Driverless (FSDS) |
| 미들웨어 | ROS2 (Humble) |
| 개발 런타임 | `src/autonomous/` |
| 제출 런타임 | `submission/` |
| 시뮬레이터 | `src/simulator/settings.json` |
| 컨테이너 | Docker / docker-compose |
| 단위 테스트 | `src/autonomous/tests/` |
| 제출 진입점 | `submission/run.sh` |
| 개발 진입점 | `src/autonomous/run_all.sh` |
| 라이선스 | `LICENSE` 참조 |

## 운영 흐름 (Compact Flow)

1. 시뮬레이터(FSDS) 또는 차량에서 카메라·LiDAR 토픽 수신
2. `perception/cone_detector.py`로 콘 후보 검출
3. `perception/cone_classifier.py`로 색상(파랑·노랑·주황) 분류
4. `perception/slam.py`로 트랙 지도와 차량 위치 추정
5. `control/pure_pursuit.py` + `control/speed.py`로 조향·속도 명령 생성
6. `v2x/rsu.py`로 외부 인프라 신호 수신
7. `utils/lap_timer.py` + `utils/watchdog.py`로 랩 카운트와 페일세이프 수행
8. `driver/competition_driver.py`가 위 모듈 전체를 오케스트레이션

## Table of Contents

1. [Purpose & Package Contents](#purpose--package-contents)
2. [First Files to Read](#first-files-to-read)
3. [Architecture](#architecture)
4. [Quick Start](#quick-start)
5. [Configuration](#configuration)
6. [Entry Points & Commands](#entry-points--commands)
7. [Local Development](#local-development)
8. [Testing](#testing)
9. [Contribution Guide](#contribution-guide)
10. [Maintainers & Contact](#maintainers--contact)
11. [License & Further Docs](#license--further-docs)

## Purpose & Package Contents

FSD 자율주행 알고리즘을 ROS2 노드 단위로 모듈화하고, 대회 제출에 그대로 쓸 수 있는
Docker 이미지로 패키징한 저장소입니다.

| 경로 | 역할 |
| --- | --- |
| `src/autonomous/` | 개발·시뮬레이터 구동 런타임 |
| `src/autonomous/modules/perception/` | 콘 검출·분류·SLAM |
| `src/autonomous/modules/control/` | Pure Pursuit, 속도 |
| `src/autonomous/modules/utils/` | 랩 타이머, 워치독 |
| `src/autonomous/driver/` | 대회용 통합 드라이버 |
| `src/autonomous/config/` | launch, params 설정 |
| `src/simulator/` | FSDS 시뮬레이터 설정 |
| `submission/` | 대회 제출 패키지 |
| `submission/src/drivers/` | basic/advanced/autonomous/competition |
| `submission/src/v2x/` | RSU 통신 |
| `submission/launch/` | ROS2 런처 |
| `docs/SUBMISSION_GUIDE.md` | 대회 제출 절차 |
| `docs/reference_materials/` | FSDS·SLAM·V2X 참고 |
| `scripts/package.sh` | 제출물 패키징 |
| `in-memoria.db` | 런타임 지식 베이스 |

## First Files to Read

새 기여자가 가장 먼저 봐야 하는 파일과 그 이유는 다음과 같습니다.

| 파일 | 왜 먼저 읽어야 하는가 |
| --- | --- |
| `src/autonomous/driver/competition_driver.py` | 전체 노드 오케스트레이션 진입점 |
| `src/autonomous/modules/perception/cone_detector.py` | 인식 파이프라인의 핵심 |
| `src/autonomous/modules/control/pure_pursuit.py` | 경로 추종 알고리즘 |
| `src/autonomous/config/params.yaml` | 튜닝 가능한 모든 파라미터 |
| `submission/launch/competition.launch` | 제출 시 토픽·노드 매핑 |
| `docs/SUBMISSION_GUIDE.md` | 대회 제출 절차 |

## Architecture

| 레이어 | 모듈 | 책임 |
| --- | --- | --- |
| 입력 | FSDS / 차량 센서 | 카메라·LiDAR 토픽 |
| 인식 | `perception/cone_detector.py` | 콘 후보 검출 |
| 인식 | `perception/cone_classifier.py` | 색상 분류 |
| 인식 | `perception/slam.py` | 트랙 지도 작성 |
| 계획·제어 | `control/pure_pursuit.py` | 조향 각도 산출 |
| 계획·제어 | `control/speed.py` | 목표 속도 산출 |
| 통신 | `v2x/rsu.py` | 외부 인프라 메시지 |
| 운영 | `utils/lap_timer.py` | 랩 카운트·시간 |
| 운영 | `utils/watchdog.py` | 이상 정지·복구 |
| 오케스트레이션 | `driver/competition_driver.py` | 노드 전체 조율 |

두 런타임의 진입점은 다음과 같이 구분됩니다.

| 모드 | 위치 | 진입점 |
| --- | --- | --- |
| 개발/시뮬레이션 | `src/autonomous/` | `run_all.sh`, `start.sh` |
| 대회 제출 | `submission/` | `run.sh`, `launch/competition.launch` |

## Quick Start

### 사전 요구사항

- Docker 24+ / docker-compose v2
- ROS2 Humble (컨테이너 외부 디버깅 시)
- FSDS 시뮬레이터 (로컬 구동 시)

### 개발 모드(시뮬레이터)

```bash
docker compose -f src/autonomous/docker-compose.yml up --build
docker compose -f src/autonomous/docker-compose.yml exec autonomous bash
./src/autonomous/run_all.sh
```

### 대회 제출 모드

```bash
docker compose -f submission/docker-compose.yml up --build
./submission/run.sh
```

### 시뮬레이터 단독 실행

```bash
# FSDS는 src/simulator/settings.json을 읽어 트랙·차량을 구성
./src/autonomous/record_race.sh   # rosbag 녹화 예시
```

## Configuration

| 파일 | 용도 |
| --- | --- |
| `src/autonomous/config/params.yaml` | 개발 런타임 파라미터 |
| `submission/autonomous/config/params.yaml` | 제출 런타임 파라미터 |
| `submission/launch/competition.launch` | ROS2 노드·토픽 매핑 |
| `src/autonomous/config/bridge_no_camera.launch` | 카메라 제외 브릿지 런처 |
| `src/simulator/settings.json` | FSDS 트랙·차량 설정 |

자주 조정하는 키 예시:

| 키 | 설명 | 단위 |
| --- | --- | --- |
| `max_speed` | 직선 목표 속도 | m/s |
| `lookahead` | Pure Pursuit 전방주시 거리 | m |
| `cone_color_thresholds` | 콘 색상 분류 임계값 | HSV |
| `watchdog_timeout` | 워치독 비활성 타임아웃 | s |

## Entry Points & Commands

| 명령 | 위치 | 설명 |
| --- | --- | --- |
| `docker compose up --build` | `src/autonomous/`, `submission/` | 런타임 빌드·기동 |
| `./submission/run.sh` | `submission/` | 제출 런타임 실행 |
| `./src/autonomous/run_all.sh` | `src/autonomous/` | 개발 런타임 전체 실행 |
| `./src/autonomous/start.sh` | `src/autonomous/` | 단일 노드 기동 |
| `./src/autonomous/record_race.sh` | `src/autonomous/` | 레이스 rosbag 녹화 |
| `./scripts/package.sh` | `scripts/` | 제출물 tar/zip 패키징 |
| `python3 src/autonomous/tests/test_algorithms.py` | `src/autonomous/tests/` | 알고리즘 단위 테스트 |
| `bash submission/dev.sh` | `submission/` | 제출 패키지 개발 셸 |

## Local Development

1. `src/autonomous/modules/`에서 알고리즘 수정
2. `src/autonomous/tests/`에서 단위 검증
3. `docker compose -f src/autonomous/docker-compose.yml build`로 이미지 갱신
4. FSDS와 함께 end-to-end 실행 후 동작 확인
5. 안정화된 로직을 `submission/src/`에 동일하게 동기화
6. `scripts/package.sh`로 제출물 패키징 후 `docs/SUBMISSION_GUIDE.md` 절차 적용

## Testing

| 항목 | 경로 | 내용 |
| --- | --- | --- |
| 알고리즘 단위 테스트 | `src/autonomous/tests/test_algorithms.py` | Pure Pursuit·속도 곡선 |
| 레이스 녹화 검증 | `src/autonomous/record_race.sh` | rosbag 비교 |
| 제출 dry-run | `submission/run.sh` | 컨테이너 내부 dry-run |

## Contribution Guide

1. 작업 항목(한국어/영어 무관)을 이슈로 먼저 등록
2. 기능 브랜치 생성 후 `src/autonomous/tests/` 통과 확인
3. `submission/src/` 측 동일 코드 경로에 동기화
4. `CONTRIBUTING.md`의 코드 스타일·커밋 컨벤션 준수
5. PR 제출 후 `OWNERS`의 리뷰 대기

## Maintainers & Contact

| 항목 | 위치 |
| --- | --- |
| 메인테이너 목록 | [`OWNERS`](OWNERS) |
| 기여 절차 | [`CONTRIBUTING.md`](CONTRIBUTING.md) |
| 대회 제출 절차 | [`docs/SUBMISSION_GUIDE.md`](docs/SUBMISSION_GUIDE.md) |

## License & Further Docs

| 항목 | 링크 |
| --- | --- |
| 라이선스 | [`LICENSE`](LICENSE) |
| 시뮬레이터 안내 | [`src/simulator/README.md`](src/simulator/README.md) |
| 제출 런타임 안내 | [`submission/README.md`](submission/README.md) |
| FSDS 설치 참고 | [`docs/reference_materials/lecture1_fsds_install.txt`](docs/reference_materials/lecture1_fsds_install.txt) |
| SLAM 참고 | [`docs/reference_materials/lecture4_slam.ipynb`](docs/reference_materials/lecture4_slam.ipynb) |
| V2X 참고 | [`docs/reference_materials/lecture6_v2x.ipynb`](docs/reference_materials/lecture6_v2x.ipynb) |