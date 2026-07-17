# Autonomous Racing Stack

Korean-first summary: 이 저장소는 Formula Student Driverless(FSDS) 및 유사 규격의 자율주행 레이싱 차량을 위한 ROS 기반 소프트웨어 스택입니다. 콘(cone) 인지·SLAM·pure pursuit 경로 추종·속도 제어·V2X(노변 장치) 모듈을 포함하며, 시뮬레이터 연동과 대회 제출용 Docker 패키징을 함께 제공합니다.

English (secondary): A ROS-based autonomous racing stack aimed at Formula Student Driverless-style competitions. It bundles cone perception, SLAM, pure-pursuit control, speed control, and V2X (RSU) modules, with a simulator harness and a competition-ready Docker image for on-track deployment.

## 한눈에 보기 / At a Glance

| 영역 / Area | 상태 / Status | 대표 진입점 / Entry point | 운영 책임 / Owner |
| --- | --- | --- | --- |
| 인지 / Perception (cones, SLAM) | 구현됨 (테스트 일부) | `src/autonomous/modules/perception/` | `OWNERS` |
| 제어 / Control (path, speed) | 구현됨 | `src/autonomous/modules/control/` | `OWNERS` |
| V2X (RSU) | 구현됨 (제출 빌드 한정) | `submission/src/v2x/rsu.py` | `OWNERS` |
| 드라이버 변형 / Driver variants | 4종 (basic / advanced / autonomous / competition) | `submission/src/drivers/` | `OWNERS` |
| 시뮬레이터 / Simulator | 설정 파일만 / Settings only | `src/simulator/settings.json` | `OWNERS` |
| 대회 제출 빌드 / Submission build | Docker 제공 / Docker provided | `submission/Dockerfile`, `submission/run.sh` | `OWNERS` |

## 컴팩트 흐름 / Compact Flow

1. **인지 / Perception** — 카메라·LiDAR 입력 → `cone_detector` → `cone_classifier` → 콘 색상별 분류.
2. **위치 추정 / Localization** — `slam` 모듈이 콘 맵을 작성하고 차량 포즈 추정.
3. **경로 산출 / Path** — 인지된 콘으로 트랙 중심선 계산.
4. **제어 / Control** — `pure_pursuit`이 조향, `speed` 모듈이 구동·제동 명령 산출.
5. **안전 / Safety** — `watchdog`이 비정상 상태 감시, `lap_timer`가 랩 측정.
6. **V2X** — `submission/src/v2x/rsu.py`가 노변 장치 메시지 송수신 (제출 빌드 한정).
7. **드라이버 / Driver** — `submission/src/drivers/competition.py`가 위 모듈을 묶어 실제 레이스 구동.

## 목차 / Contents

- [Purpose](#purpose)
- [Package Contents](#package-contents)
- [First Files to Read](#first-files-to-read)
- [API and Entry Points](#api-and-entry-points)
- [Quickstart](#quickstart)
- [Architecture](#architecture)
- [Configuration](#configuration)
- [Commands Reference](#commands-reference)
- [Local Development](#local-development)
- [Testing](#testing)
- [Maintainers](#maintainers)
- [Further Documentation](#further-documentation)
- [Contributing](#contributing)
- [License](#license)

## Purpose

이 프로젝트는 자율주행 레이싱 대회 출전용 차량 스택을 한 저장소에서 개발·검증·제출할 수 있도록 구성되어 있습니다. 핵심 사용자는 팀의 인지/제어 엔지니어, 시뮬레이터 운용자, 대회 제출 빌드 관리자입니다.

What you can do with it:

- 콘 마핑 기반 트랙에서 자율 주행을 시뮬레이션하고 알고리즘을 단위 테스트합니다.
- 동일 코드베이스로 대회 제출용 Docker 이미지를 빌드하고 트랙에 배포합니다.
- 여러 드라이버 모드(basic → advanced → autonomous → competition)를 비교 실험합니다.
- V2X 노변 장치 연동을 포함한 확장 시나리오를 검증합니다.

## Package Contents

| 경로 / Path | 역할 / Role |
| --- | --- |
| `src/autonomous/` | 개발용 자율주행 워크스페이스 (ROS 패키지, Dockerfile, 테스트) |
| `src/autonomous/driver/competition_driver.py` | 개발 빌드용 메인 드라이버 루프 |
| `src/autonomous/modules/perception/` | 콘 검출·분류·SLAM |
| `src/autonomous/modules/control/` | pure pursuit·속도 제어 |
| `src/autonomous/modules/utils/` | 랩 타이머·워치독 |
| `src/autonomous/tests/` | 알고리즘 단위 테스트 |
| `src/simulator/` | 시뮬레이터 설정 (`settings.json`) |
| `submission/` | 대회 제출용 빌드 (드라이버 4종, V2X, Docker) |
| `submission/src/drivers/` | basic·advanced·autonomous·competition 드라이버 |
| `submission/src/v2x/` | RSU(노변 장치) 모듈 |
| `submission/launch/competition.launch` | ROS 런치 파일 |
| `scripts/package.sh` | 제출 패키징 스크립트 |
| `docs/SUBMISSION_GUIDE.md` | 대회 제출 절차 가이드 |
| `docs/reference_materials/` | 강의 노트/설치 자료(레퍼런스) |

## First Files to Read

운영자/신규 합류자가 먼저 봐야 할 파일 순서입니다.

1. `README.md` (이 문서) — 전체 구조 파악.
2. `docs/SUBMISSION_GUIDE.md` — 대회 제출 흐름.
3. `submission/README.md` — 제출 빌드 진입점.
4. `src/autonomous/driver/competition_driver.py` — 메인 루프.
5. `src/autonomous/config/params.yaml` — 튜닝 파라미터.
6. `submission/launch/competition.launch` — ROS 토픽/노드 매핑.

## API and Entry Points

| 진입점 / Entry | 종류 / Kind | 용도 / Purpose |
| --- | --- | --- |
| `src/autonomous/driver/competition_driver.py` | Python 모듈 | 개발 빌드 메인 루프 |
| `submission/src/drivers/competition.py` | Python 모듈 | 제출 빌드 메인 드라이버 |
| `submission/src/drivers/autonomous.py` | Python 모듈 | 자율 모드 드라이버 |
| `submission/src/drivers/advanced.py` | Python 모듈 | 확장 모드 드라이버 |
| `submission/src/drivers/basic.py` | Python 모듈 | 기본 모드 드라이버 |
| `submission/src/perception/*` | Python 패키지 | 콘 인지·SLAM |
| `submission/src/control/*` | Python 패키지 | 경로 추종·속도 |
| `submission/src/v2x/rsu.py` | Python 모듈 | 노변 장치 연동 |
| `submission/launch/competition.launch` | ROS 런치 | 토픽·노드 부트스트랩 |
| `src/autonomous/config/bridge_no_camera.launch` | ROS 런치 | 카메라 미사용 시 브리지 |
| `src/autonomous/scripts/start_race.py` | Python 스크립트 | 레이스 시작 보조 |

## Quickstart

### 1. 개발 워크스페이스 실행

```bash
# 개발 빌드용 컨테이너 기동
docker compose -f src/autonomous/docker-compose.yml up
```

### 2. 대회 제출 빌드 패키징

```bash
# 제출용 이미지 빌드 및 실행
./scripts/package.sh
docker compose -f submission/docker-compose.yml up
```

### 3. 차량 온보드 배포

```bash
docker compose -f submission/docker-compose.yml up
./submission/run.sh
```

상세 절차는 `docs/SUBMISSION_GUIDE.md`를 따릅니다.

## Architecture

ROS 토픽 중심의 모듈형 파이프라인입니다.

| 계층 / Layer | 모듈 / Module | 입력 / Input | 출력 / Output |
| --- | --- | --- | --- |
| Sensor | 카메라·LiDAR 드라이버 | 차량 HW | 원시 토픽 |
| Perception | `cone_detector`, `cone_classifier` | 이미지·포인트클라우드 | 콘 위치·색상 |
| Localization | `slam` | 콘 위치, IMU/오도메트리 | 차량 포즈, 트랙 맵 |
| Planning | (드라이버 내부) | 트랙 맵 | 목표 경로 |
| Control | `pure_pursuit`, `speed` | 목표 경로, 포즈 | 조향·구동·제동 |
| Safety | `watchdog`, `lap_timer` | 시스템 상태 | 비상 정지, 랩 기록 |
| V2X | `rsu` (제출 빌드) | RSU 메시지 | 보조 경로·속도 |
| Driver | `competition` / `autonomous` / `advanced` / `basic` | 위 모듈들 | 최종 제어 명령 |

레이스 1사이클의 요청 흐름:

1. 센서가 카메라/LiDAR 데이터를 ROS 토픽으로 발행합니다.
2. 인지 모듈이 콘 검출·분류 결과를 퍼블리시합니다.
3. SLAM이 트랙 맵과 차량 포즈를 갱신합니다.
4. 드라이버가 경로·속도 명령을 생성합니다.
5. `pure_pursuit`과 `speed`가 차량 제어 명령을 발행합니다.
6. `watchdog`이 사이클 시간을 감시하고 이상 시 안전 정지합니다.
7. V2X가 활성화된 제출 빌드는 RSU 메시지를 보조 입력으로 사용합니다.

## Configuration

주요 설정 파일과 책임:

| 파일 / File | 책임 / Responsibility |
| --- | --- |
| `src/autonomous/config/params.yaml` | 인지/제어 튜닝 파라미터 |
| `submission/config/params.yaml` | 제출 빌드 튜닝 파라미터 |
| `src/simulator/settings.json` | 시뮬레이터 환경 설정 |
| `submission/launch/competition.launch` | ROS 노드/토픽 부트스트랩 |

설정 변경 후에는 해당 빌드의 Docker 이미지를 다시 빌드해야 반영됩니다.

## Commands Reference

| 명령 / Command | 위치 / Path | 용도 / Purpose |
| --- | --- | --- |
| `start.sh` | `src/autonomous/start.sh` | 개발 컨테이너 시작 |
| `run_all.sh` | `src/autonomous/run_all.sh` | 전체 모듈 실행 |
| `entrypoint.sh` | `src/autonomous/entrypoint.sh` | 컨테이너 진입 훅 |
| `record_race.sh` | `src/autonomous/record_race.sh` | 레이스 로그 기록 |
| `scripts/start_race.py` | `src/autonomous/scripts/start_race.py` | 레이스 트리거 |
| `package.sh` | `scripts/package.sh` | 제출 패키징 |
| `run.sh` | `submission/run.sh` | 제출 빌드 실행 |
| `dev.sh` | `submission/dev.sh` | 제출 빌드 개발 모드 |
| `pytest` (또는 동등 도구) | `src/autonomous/tests/` | 단위 테스트 |

## Local Development

- 워크스페이스: `src/autonomous/`
- 권장 흐름:
  1. `src/autonomous/config/params.yaml`에서 파라미터 조정.
  2. `src/autonomous/tests/`에서 단위 테스트 실행.
  3. 시뮬레이터 연동은 `src/simulator/settings.json`을 사용.
  4. 통합 검증은 Docker compose로 동일한 토픽 환경 구성.

## Testing

- 단위 테스트: `src/autonomous/tests/test_algorithms.py`
- 실행 예시(컨테이너 내부):

```bash
python -m pytest src/autonomous/tests
```

- 시뮬레이터 통합: `src/simulator/README.md` 참조.

## Maintainers

- 운영 책임자: 저장소 루트의 `OWNERS` 파일 참조.
- 문의 채널: GitHub Issues.
- 제출 절차 책임: `docs/SUBMISSION_GUIDE.md`의 담당자 섹션 참조.

## Further Documentation

- `docs/SUBMISSION_GUIDE.md` — 대회 제출 절차.
- `src/simulator/README.md` — 시뮬레이터 사용법.
- `submission/README.md` — 제출 빌드 운영.
- `docs/reference_materials/lecture1_fsds_install.txt` — FSDS 설치 레퍼런스.
- `docs/reference_materials/lecture4_slam.ipynb` — SLAM 강의 노트.
- `docs/reference_materials/lecture6_v2x.ipynb` — V2X 강의 노트.

## Contributing

기여 절차는 저장소 루트의 `CONTRIBUTING.md`를 따릅니다. 인지/제어 알고리즘 변경 시 단위 테스트와 시뮬레이터 회귀 결과를 PR에 첨부해 주세요.

## License

저장소 루트의 `LICENSE` 파일을 따릅니다.