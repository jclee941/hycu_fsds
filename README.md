# 자율주행 경주 스택 — FSDS 기반 Formula Student Driverless

[![License: MIT](LICENSE)](LICENSE) [![Docs: Submission Guide](docs/SUBMISSION_GUIDE.md)](docs/SUBMISSION_GUIDE.md) [![Simulator](src/simulator/README.md)](src/simulator/README.md) [![Reference Materials](docs/reference_materials/)](docs/reference_materials/)

FSDS(Formula Student Driverless Simulator) 환경에서 콘 인식 → SLAM → Pure Pursuit 추종 → V2X → 랩 타이밍까지 한 번에 검증·기록·제출할 수 있는 자율주행 경주용 풀스택 워크스페이스입니다.

This workspace packages a Formula Student Driverless–style autonomous racing stack for the FSDS simulator: cone perception, SLAM, pursuit control, V2X, lap timing, and a Docker-ready submission image, all runnable from a single `docker compose up`.

## 한눈에 보기 (Status at a Glance)

| 항목 | 값 |
| --- | --- |
| 목적 | FSDS 시뮬레이터 위 자율주행 경주 알고리즘 검증·대회 제출 |
| 핵심 언어 | Python 3 (ROS / ROS bridge 노드) |
| 시뮬레이터 | FSDS (F1TENTH / Formula Student Driverless Simulator) |
| 핵심 노드 | `cone_detector`, `cone_classifier`, `slam`, `pure_pursuit`, `speed`, `lap_timer`, `watchdog`, `rsu` |
| 런타임 진입점 | `src/autonomous/run_all.sh`, `src/autonomous/start.sh`, `src/submission/run.sh` |
| 컨테이너 진입점 | `src/autonomous/entrypoint.sh`, `src/submission/Dockerfile` |
| 대회 드라이버 | `src/autonomous/driver/competition_driver.py` |
| 패키징 스크립트 | `scripts/package.sh` |
| 제출 가이드 | `docs/SUBMISSION_GUIDE.md` |
| 메모리/상태 | `in-memoria.db` (랩·이벤트 누적) |
| 라이선스 | `LICENSE` |
| 책임자 | `OWNERS` |
| 기여 가이드 | `CONTRIBUTING.md` |

## 런타임 흐름 (Operator Flow)

1. FSDS 시뮬레이터가 LiDAR / 카메라 / 차량 상태 토픽을 발행합니다.
2. `cone_detector`가 LiDAR 클러스터에서 콘 후보를 추출하고, `cone_classifier`가 색상(파란색/노란색/주황색)을 분류합니다.
3. `slam` 모듈이 콘 맵을 누적해 트랙 지도를 작성하고 중심 라인을 추정합니다.
4. `pure_pursuit`이 목표 경로를 추종하고, `speed`가 코너 감속·직선 가속을 결정합니다.
5. `lap_timer`가 랩 타임을 측정하고 `watchdog`가 정지·이탈·센서 무응답을 감지해 안전 정지합니다.
6. `competition_driver`가 전체 파이프라인을 오케스트레이션하고, `rsu`(V2X)가 인프라 신호를 수신합니다.
7. 결과는 `in-memoria.db`에 누적되며 `record_race.sh`로 랩 단위 기록이 남습니다.
8. `scripts/package.sh`가 대회 제출용 아카이브를 생성합니다.

## 목차

- [패키지 구성](#패키지-구성)
- [아키텍처](#아키텍처)
- [빠른 시작](#빠른-시작)
- [설정](#설정)
- [명령어 레퍼런스](#명령어-레퍼런스)
- [대회 제출](#대회-제출)
- [로컬 개발](#로컬-개발)
- [테스트](#테스트)
- [유지보수·연락처](#유지보수연락처)
- [추가 문서](#추가-문서)

## 패키지 구성

이 저장소는 개발용(`src/autonomous`)과 제출용(`src/submission`) 두 트리를 같은 레이아웃으로 유지합니다. 두 트리는 동일한 노드 구성을 가지며, 제출용은 `v2x/` 모듈과 대회 런처를 추가로 포함합니다.

| 경로 | 역할 |
| --- | --- |
| `src/autonomous/driver/competition_driver.py` | 대회용 메인 드라이버 (전 노드 오케스트레이터) |
| `src/autonomous/modules/perception/` | `cone_detector.py`, `cone_classifier.py`, `slam.py` |
| `src/autonomous/modules/control/` | `pure_pursuit.py`, `speed.py` |
| `src/autonomous/modules/utils/` | `lap_timer.py`, `watchdog.py` |
| `src/autonomous/scripts/start_race.py` | 레이스 시작/리셋 보조 스크립트 |
| `src/autonomous/config/` | ROS 런처(`bridge_no_camera.launch`)·튜닝 파라미터(`params.yaml`) |
| `src/autonomous/tests/` | 알고리즘 단위 테스트 (`test_algorithms.py`) |
| `src/simulator/` | FSDS 시뮬레이터 설정(`settings.json`) 및 가이드 |
| `src/submission/` | 대회 제출용 패키지 (`drivers/`, `perception/`, `control/`, `v2x/`, `utils/`) |
| `src/submission/drivers/` | 단계별 드라이버: `basic.py`, `advanced.py`, `autonomous.py`, `competition.py` |
| `src/submission/v2x/rsu.py` | RSU(Roadside Unit) 인프라 신호 수신 |
| `scripts/package.sh` | 제출 아카이브 패키징 |
| `docs/SUBMISSION_GUIDE.md` | 대회 제출 절차 |
| `docs/reference_materials/` | FSDS 설치·SLAM·V2X 강의 노트 |
| `in-memoria.db` | 랩·이벤트 누적 데이터베이스 |

## 아키텍처

시스템은 “인지 → 지도 → 제어 → 안전 → 기록 → 외부 협력” 파이프라인으로 구성됩니다. 각 책임은 단일 Python 노드 또는 모듈에 격리되어 있어, 대회 등급 드라이버와 단계별 학습용 드라이버가 같은 노드를 재사용합니다.

| 계층 | 노드/모듈 | 책임 | 입력 | 출력 |
| --- | --- | --- | --- | --- |
| Perception | `cone_detector` | LiDAR 클러스터에서 콘 후보 추출 | LiDAR 스캔 | 콘 후보 포인트 |
| Perception | `cone_classifier` | 콘 색상 분류(파랑/노랑/주황) | 이미지 또는 후보 | 라벨링된 콘 |
| Perception | `slam` | 콘 맵 누적·중심선 추정 | 라벨링된 콘 | 트랙 맵·중심 경로 |
| Control | `pure_pursuit` | 경로 추종 조향각 산출 | 중심 경로·자차 위치 | 조향 명령 |
| Control | `speed` | 직선/코너 기반 목표 속도 산출 | 경로 곡률·자차 속도 | 속도 명령 |
| Utils | `lap_timer` | 시작선 통과 기반 랩 측정 | 자차 위치 | 랩 타임 |
| Utils | `watchdog` | 정지·이탈·센서 무응답 감지 | 차량 상태·토픽 Heartbeat | 안전 정지 신호 |
| V2X | `rsu` | 인프라(RSU) 신호 수신·디코딩 | V2X 메시지 | 트래픽·신호 정보 |
| Orchestration | `competition_driver` | 전체 노드 부팅·토픽 연결·레코드 | 토픽 그래프 | 통합 차량 명령 |

런처와 컨테이너는 위 계층을 한 그래프로 묶어 부팅합니다. 제출 트리에는 같은 모듈이 `v2x/`와 `drivers/competition.py`와 함께 포함되어 단일 이미지로 실행됩니다.

## 빠른 시작

### 1. 시뮬레이터 준비

`src/simulator/settings.json`을 FSDS에 맞게 조정하고, `docs/reference_materials/lecture1_fsds_install.txt`의 설치 절차대로 시뮬레이터를 띄웁니다.

### 2. 개발 모드 (호스트에서 직접 실행)

```bash
# 저장소 루트에서
git clone <repo-url> fsds-autonomous
cd fsds-autonomous

# 시뮬레이터가 떠 있는 상태에서 자율주행 스택 실행
./src/autonomous/run_all.sh
# 또는 레이스 단위로만 시작
./src/autonomous/start.sh
```

### 3. Docker로 실행

```bash
cd src/autonomous
docker compose up --build
# 또는 제출 이미지 그대로
cd ../submission
docker compose up --build
```

`src/autonomous/entrypoint.sh`가 ROS 환경 로딩 → 런처 실행 순서로 부팅을 마무리합니다.

### 4. 레이스 기록

```bash
./src/autonomous/record_race.sh
# 결과는 in-memoria.db에 누적
```

## 설정

| 파일 | 용도 |
| --- | --- |
| `src/autonomous/config/params.yaml` | 콘 인식·SLAM·제어 튜닝 파라미터 |
| `src/autonomous/config/bridge_no_camera.launch` | ROS ↔ FSDS 브리지(카메라 제외) 런처 |
| `src/submission/launch/competition.launch` | 대회용 통합 런처 |
| `src/simulator/settings.json` | FSDS 시뮬레이터 차량·트랙·센서 설정 |
| `src/submission/autonomous/config/params.yaml` | 제출 빌드용 파라미터 (개발 트리와 동기 유지) |

튜닝 시 `params.yaml`의 키 순서는 노드 초기화 순서와 일치합니다. 키 변경 후에는 반드시 `./src/autonomous/tests/test_algorithms.py`로 단위 회귀를 확인하세요.

## 명령어 레퍼런스

| 명령어 | 위치 | 설명 |
| --- | --- | --- |
| `run_all.sh` | `src/autonomous/` | 인지·제어·기록 노드를 한 번에 기동 |
| `start.sh` | `src/autonomous/` | 레이스 1회 시작(부팅·리셋) |
| `record_race.sh` | `src/autonomous/` | 랩 단위 기록 모드로 실행 |
| `entrypoint.sh` | `src/autonomous/` | 컨테이너 부팅 시 ROS 환경 로딩·런처 실행 |
| `start_race.py` | `src/autonomous/scripts/` | 레이스 시작 보조 스크립트(Python) |
| `run.sh` | `src/submission/` | 제출 이미지의 메인 실행 스크립트 |
| `dev.sh` | `src/submission/` | 제출 트리에서 개발 모드 실행 |
| `package.sh` | `scripts/` | 대회 제출 아카이브 생성 |

## 대회 제출

1. `docs/SUBMISSION_GUIDE.md`의 체크리스트를 따라 환경 변수·파라미터·라이선스를 확인합니다.
2. `src/submission/`에서 `docker compose up --build`로 전체 스택이 동작하는지 검증합니다.
3. `scripts/package.sh`를 실행해 제출 아카이브를 생성합니다.
4. `src/submission/dev.sh`로 추가 디버깅이 필요할 때 빠르게 호스트 모드로 전환합니다.

> 제출 트리는 독립적으로 실행 가능하도록 자체 `Dockerfile`, `docker-compose.yml`, `launch/` 디렉터리를 가집니다. 운영 시에는 `competition_driver.py`를 메인으로 사용하고, 단계별 검증은 `drivers/basic.py` → `advanced.py` → `autonomous.py` 순으로 진행하세요.

## 로컬 개발

- **Python 모듈 경로**: `src/autonomous/modules/` 또는 `src/submission/src/`를 `PYTHONPATH`에 추가하면 IDE에서 직접 임포트할 수 있습니다.
- **코드 스타일**: PEP 8 + ROS 노드 명명 규칙(스네이크 케이스)을 따릅니다.
- **ROS 워크스페이스로 사용**: `src/autonomous`를 그대로 ROS 2 오버레이의 일부로 두고, `src/autonomous/config/bridge_no_camera.launch`를 실행하세요.
- **V2X 디버깅**: `src/submission/v2x/rsu.py`의 핸들러를 단독 실행해 RSU 메시지 포맷을 확인할 수 있습니다.
- **상태 확인**: `in-memoria.db`를 SQLite 브라우저로 열어 랩 타임·이벤트 로그를 확인합니다.

## 테스트

| 테스트 | 위치 | 명령어 |
| --- | --- | --- |
| 알고리즘 단위 테스트 | `src/autonomous/tests/test_algorithms.py` | `python -m pytest src/autonomous/tests/test_algorithms.py` |
| 레이스 리허설 | `src/autonomous/record_race.sh` | `./src/autonomous/record_race.sh` |
| 제출 빌드 스모크 | `src/submission/` | `docker compose up --build` |

테스트는 `cone_detector`, `pure_pursuit`, `speed`의 결정성(같은 입력 → 같은 출력)을 중점적으로 검증합니다. 새 노드를 추가할 때는 동일한 단위 테스트 패턴을 따르세요.

## 유지보수·연락처

| 역할 | 위치 |
| --- | --- |
| 코드 오너십 | `OWNERS` |
| 기여 절차 | `CONTRIBUTING.md` |
| 라이선스 | `LICENSE` |
| 보조 자동화 메모 | `AGENTS.md` (저장소 운영 메모; 제품 기능 아님) |

## 추가 문서

| 문서 | 용도 |
| --- | --- |
| `docs/SUBMISSION_GUIDE.md` | 대회 제출 체크리스트·제약 조건 |
| `src/simulator/README.md` | FSDS 시뮬레이터 설정 가이드 |
| `docs/reference_materials/lecture1_fsds_install.txt` | FSDS 설치 절차 |
| `docs/reference_materials/lecture4_slam.ipynb` | SLAM 튜토리얼 노트북 |
| `docs/reference_materials/lecture6_v2x.ipynb` | V2X 튜토리얼 노트북 |

### English (Quick Reference)

- **Purpose**: End-to-end autonomous racing stack for the FSDS simulator, including cone perception, SLAM, pursuit control, V2X, lap timing, and a Dockerized competition submission.
- **Entry points**: `./src/autonomous/run_all.sh` for development, `./src/submission/run.sh` for the submission image, `docker compose up` in either tree for containerized runs.
- **Drivers**: `basic → advanced → autonomous → competition`, all sharing the same perception/control nodes.
- **Packaging**: `scripts/package.sh` produces the final competition archive.
- **Testing**: `pytest src/autonomous/tests/test_algorithms.py` for unit coverage; `record_race.sh` for end-to-end race rehearsal.