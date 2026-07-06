# FSDS 자율주행 스택

ROS 1 기반 Formula Student Driverless Simulator(FSDS) 자율주행 레이스 스택. 콘(cone) 검출·SLAM·Pure Pursuit 추종·V2X 통신을 한 Docker 실행 환경으로 묶고, 대회 제출용 패키지까지 함께 제공한다.

| ![ROS](https://img.shields.io/badge/ROS-Noetic-22314E?logo=ros) | ![Python](https://img.shields.io/badge/Python-3.8%2B-3776AB?logo=python) | ![Docker](https://img.shields.io/badge/Docker-ready-2496ED?logo=docker) | ![Simulator](https://img.shields.io/badge/FSDS-LFS--19.04-orange) | ![Mode](https://img.shields.io/badge/Mode-Autonomous%20%7C%20Submission-success) |
| --- | --- | --- | --- | --- |

## 한 줄 요약

FSDS 시뮬레이터에서 콘 트랙을 인식해 자율 주행을 수행하고, 대회 제출용 Docker 이미지로 패키징하는 Python/ROS 스택.

## 빠른 상태

| 항목 | 값 |
| --- | --- |
| 대상 플랫폼 | Ubuntu 20.04 + ROS Noetic |
| 시뮬레이터 | Formula Student Driverless Simulator (FSDS) |
| 언어 | Python 3 |
| 핵심 노드 그룹 | perception, control, v2x, utils |
| 실행 모드 | `autonomous` (개발/시뮬레이션), `submission` (대회 제출) |
| 컨테이너 | Docker, docker-compose |
| 문서 | `docs/SUBMISSION_GUIDE.md` |
| 라이선스 | `LICENSE` |

## 실행 흐름 요약

1. FSDS 시뮬레이터에서 트랙을 로드하고 차량을 스폰한다.
2. `src/autonomous` 또는 `src/submission`에서 docker-compose를 기동한다.
3. `competition.launch` 또는 `bridge_no_camera.launch`가 perception / control / v2x 노드를 로드한다.
4. `competition_driver.py`가 콘 검출 → SLAM → Pure Pursuit → 속도 제어를 순차 구동한다.
5. `lap_timer`가 랩 타임을 기록하고 `watchdog`가 비정상 상태를 감시한다.
6. `scripts/package.sh`로 대회 제출용 아카이브를 생성한다.

## 목차

- [프로젝트 개요](#프로젝트-개요)
- [패키지 구성](#패키지-구성)
- [상태 및 지원 범위](#상태-및-지원-범위)
- [먼저 읽을 파일](#먼저-읽을-파일)
- [진입점 (Entry Points)](#진입점-entry-points)
- [아키텍처](#아키텍처)
- [빠른 시작](#빠른-시작)
- [명령어 참조](#명령어-참조)
- [구성 (Configuration)](#구성-configuration)
- [로컬 개발](#로컬-개발)
- [테스트](#테스트)
- [기여 가이드](#기여-가이드)
- [유지보수자 및 연락처](#유지보수자-및-연락처)
- [추가 문서](#추가-문서)
- [라이선스](#라이선스)

## 프로젝트 개요

이 저장소는 FSDS(FSD Simulator) 위에서 동작하는 **Formula Student Driverless** 자율주행 파이프라인이다. 시뮬레이션 주행뿐 아니라 실차 제출 패키지까지 동일한 코드 베이스에서 만들 수 있도록 두 가지 실행 모드를 함께 둔다.

| 모드 | 경로 | 용도 |
| --- | --- | --- |
| `autonomous` | `src/autonomous/` | 로컬 개발, 시뮬레이터 연동, 영상 기록 |
| `submission` | `src/submission/` | 대회 제출용 패키지, 실제 차량/하드웨어 연동 |

## 패키지 구성

```
./
├── README.md
├── LICENSE
├── OWNERS
├── CONTRIBUTING.md
├── AGENTS.md
├── in-memoria.db            # 로컬 개발용 SQLite 캐시
├── src/
│   ├── autonomous/          # 개발 모드 스택 (Docker, ROS launch, 모듈)
│   │   ├── Dockerfile
│   │   ├── docker-compose.yml
│   │   ├── entrypoint.sh
│   │   ├── start.sh
│   │   ├── run_all.sh
│   │   ├── record_race.sh
│   │   ├── scripts/start_race.py
│   │   ├── config/
│   │   │   ├── bridge_no_camera.launch
│   │   │   └── params.yaml
│   │   ├── driver/competition_driver.py
│   │   ├── modules/perception/{cone_detector,cone_classifier,slam}.py
│   │   ├── modules/control/{pure_pursuit,speed}.py
│   │   ├── modules/utils/{lap_timer,watchdog}.py
│   │   └── tests/test_algorithms.py
│   ├── submission/          # 대회 제출 패키지
│   │   ├── Dockerfile
│   │   ├── docker-compose.yml
│   │   ├── dev.sh
│   │   ├── run.sh
│   │   ├── launch/competition.launch
│   │   ├── src/drivers/{basic,advanced,autonomous,competition}.py
│   │   ├── src/perception/{cone_detector,cone_classifier,slam}.py
│   │   ├── src/v2x/rsu.py
│   │   ├── src/control/{pure_pursuit,speed}.py
│   │   ├── src/utils/{lap_timer,watchdog}.py
│   │   └── autonomous/      # 제출 내부 자율주행 컨테이너
│   └── simulator/           # FSDS 시뮬레이터 설정
│       ├── README.md
│       └── settings.json
├── scripts/package.sh       # 대회 제출 아카이브 생성기
├── docs/
│   ├── SUBMISSION_GUIDE.md
│   └── reference_materials/ # FSDS 설치·SLAM·V2X 강의 자료
└── submission/              # 루트의 제출 디렉토리(문서/스크립트 보조)
    ├── AGENTS.md
    ├── README.md
    └── Dockerfile
```

## 상태 및 지원 범위

| 영역 | 상태 | 비고 |
| --- | --- | --- |
| 시뮬레이션 자율주행 | 안정 | FSDS 연동 검증됨 |
| 대회 제출 패키지 | 안정 | `docs/SUBMISSION_GUIDE.md` 절차 사용 |
| V2X (RSU) | 실험적 | `src/submission/src/v2x/rsu.py` 참조 |
| 카메라 없는 브리지 모드 | 안정 | `bridge_no_camera.launch` |
| 단위 테스트 | 부분 | `src/autonomous/tests/test_algorithms.py` |

이 저장소는 **deprecated가 아니며**, FSD 대회 시즌에 맞춰 운영되는 competition-ready 코드다. 단, V2X 노드는 실차/하드웨어 의존성이 있어 시뮬레이션에서는 동작이 제한될 수 있다.

## 먼저 읽을 파일

| 순서 | 파일 | 이유 |
| --- | --- | --- |
| 1 | `docs/SUBMISSION_GUIDE.md` | 대회 제출 절차 요약 |
| 2 | `src/autonomous/config/params.yaml` | 모든 노드의 튜닝 파라미터 |
| 3 | `src/autonomous/driver/competition_driver.py` | 파이프라인 오케스트레이터 |
| 4 | `src/autonomous/modules/perception/cone_detector.py` | 콘 검출 진입점 |
| 5 | `src/submission/launch/competition.launch` | 제출 모드의 ROS 노드 구성 |
| 6 | `src/simulator/README.md` | FSDS 설정 방법 |
| 7 | `scripts/package.sh` | 제출 아카이브 생성 방법 |

## 진입점 (Entry Points)

| 진입점 | 종류 | 설명 |
| --- | --- | --- |
| `src/autonomous/driver/competition_driver.py` | Python | 자율주행 메인 루프 (perception → control) |
| `src/submission/src/drivers/competition.py` | Python | 제출 모드용 드라이버 |
| `src/submission/src/drivers/advanced.py` | Python | 확장 드라이버 (대회 고급 카테고리) |
| `src/submission/src/drivers/autonomous.py` | Python | 자율주행 드라이버 래퍼 |
| `src/submission/src/drivers/basic.py` | Python | 베이스라인 드라이버 |
| `src/submission/launch/competition.launch` | ROS launch | 제출 노드 그래프 기동 |
| `src/autonomous/config/bridge_no_camera.launch` | ROS launch | 카메라 입력 없이 SLAM/제어만 검증 |
| `scripts/package.sh` | Shell | 제출 아카이브 패키징 |

## 아키텍처

| 계층 | 모듈 | 책임 |
| --- | --- | --- |
| 입력 | FSDS 시뮬레이터 / 카메라 토픽 | 차량 상태, 이미지, LiDAR |
| Perception | `cone_detector`, `cone_classifier`, `slam` | 콘 위치 추정, 트랙 경계 인식, 자기 위치 추정 |
| V2X | `v2x/rsu` | RSU(노변 기지국) 메시지 송수신 |
| Planning/Control | `pure_pursuit`, `speed` | 경로 추종 및 속도 명령 생성 |
| Driver | `competition_driver`, `drivers/*` | 위 모듈들을 묶어 차량 명령 발행 |
| Utility | `lap_timer`, `watchdog` | 랩 기록, 장애 감시 |
| Runtime | Docker + docker-compose | 재현 가능한 실행 환경 제공 |

요청 흐름:

1. FSDS가 `/cam`, `/odom` 등 ROS 토픽을 발행한다.
2. `cone_detector`가 이미지에서 콘 후보를 추출하고 `cone_classifier`가 색상(파랑/노랑/주황)을 판별한다.
3. `slam`이 LiDAR/관성 정보를 결합해 차량의 트랙상 위치를 갱신한다.
4. `pure_pursuit`이 콘 중심 경로를 따라 조향각을 계산하고 `speed`가 직선/곡률을 고려해 속도를 결정한다.
5. `lap_timer`가 시작선 통과를 감지해 랩 타임을 기록하고, `watchdog`가 일정 시간 입력이 없으면 안전 정지한다.
6. `competition_driver`가 위 결과를 묶어 `/cmd_vel` 또는 차량 제어 토픽으로 송출한다.

## 빠른 시작

### 사전 요구사항

- Docker 20.10 이상, docker-compose v2
- FSDS 설치 (자세한 절차: `docs/reference_materials/lecture1_fsds_install.txt`)
- NVIDIA GPU 권장 (시뮬레이터 가속용)

### 1) 저장소 클론 후 시뮬레이터 실행

FSDS를 설치한 뒤 `src/simulator/settings.json`을 시뮬레이터 디렉토리에 둔다.

### 2) 자율주행(개발) 모드 실행

```bash
cd src/autonomous
docker-compose up --build
```

내부적으로 `entrypoint.sh` → `start.sh` 또는 `run_all.sh` 순으로 기동되며, `bridge_no_camera.launch`로 ROS 노드가 로드된다. 영상을 남기려면 별도로 `record_race.sh`를 사용한다.

### 3) 대회 제출 모드 실행

```bash
cd src/submission
./run.sh           # 대회 운영 시
./dev.sh           # 로컬 디버깅 시
docker-compose up  # 직접 컨테이너를 띄울 때
```

### 4) 제출용 아카이브 생성

```bash
bash scripts/package.sh
```

생성된 아카이브는 대회 플랫폼이 요구하는 형식으로 묶인다. 자세한 내용은 `docs/SUBMISSION_GUIDE.md`를 따른다.

## 명령어 참조

| 명령어 | 위치 | 설명 |
| --- | --- | --- |
| `docker-compose up --build` | `src/autonomous/` | 개발 컨테이너 빌드 및 기동 |
| `bash start.sh` | `src/autonomous/` | 컨테이너 내부에서 ROS 노드만 기동 |
| `bash run_all.sh` | `src/autonomous/` | 시뮬레이터 + ROS 노드 일괄 기동 |
| `bash record_race.sh` | `src/autonomous/` | 주행 영상을 rosbag으로 기록 |
| `python3 scripts/start_race.py` | `src/autonomous/scripts/` | 레이스 시작 훅 |
| `roslaunch competition.launch` | `src/submission/launch/` | 제출 노드 그래프 기동 |
| `./run.sh` | `src/submission/` | 대회 운영 스크립트 |
| `./dev.sh` | `src/submission/` | 로컬 개발 스크립트 |
| `bash scripts/package.sh` | 저장소 루트 | 제출 아카이브 패키징 |

## 구성 (Configuration)

주요 설정은 `src/autonomous/config/params.yaml`에 있다. 시뮬레이터 토픽 이름, 콘 색상 임계값, Pure Pursuit lookahead 거리, 속도 리미트, watchdog 타임아웃 등을 조정한다. `bridge_no_camera.launch`는 카메라 토픽을 더미로 대체해 비전 비의존 경로로 SLAM/제어만 빠르게 검증할 때 사용한다.

| 파라미터 그룹 | 예시 키 | 기본 위치 |
| --- | --- | --- |
| Perception | 콘 HSV 임계값, ROI 비율 | `params.yaml` |
| Control | lookahead 거리, 최대 조향각 | `params.yaml` |
| Speed | 직선/곡선 속도, 감속 비율 | `params.yaml` |
| Lap Timer | 시작선 좌표, 최소 통과 시간 | `params.yaml` |
| Watchdog | 입력 타임아웃, 안전 정지 속도 | `params.yaml` |

## 로컬 개발

| 단계 | 방법 |
| --- | --- |
| 컨테이너 진입 | `docker-compose exec autonomous bash` |
| 단일 노드 실행 | `rosrun <pkg> <node>` (예: `rosrun perception cone_detector.py`) |
| 파라미터 덮어쓰기 | `roslaunch --args _param:=value competition.launch` |
| 시뮬레이터 설정 변경 | `src/simulator/settings.json` 수정 후 재기동 |
| 캐시 초기화 | `in-memoria.db` 삭제 (SQLite 캐시) |

## 테스트

| 테스트 종류 | 위치 | 실행 |
| --- | --- | --- |
| 알고리즘 단위 테스트 | `src/autonomous/tests/test_algorithms.py` | `python3 -m unittest src/autonomous/tests/test_algorithms.py` |
| 카메라 없는 통합 테스트 | `bridge_no_camera.launch` 사용 | `roslaunch bridge_no_camera.launch` |
| 시뮬레이션 E2E | FSDS + `run_all.sh` | `bash src/autonomous/run_all.sh` |
| 제출 드라이버 스모크 | `src/submission/src/drivers/basic.py` | `rosrun drivers basic.py` |

## 기여 가이드

1. 이슈를 먼저 등록하고 작업 범위를 합의한다.
2. `main`에서 기능 브랜치를 분기한다 (예: `feat/cone-color-tuning`).
3. 단위 테스트를 추가하거나 기존 테스트를 갱신한다.
4. PR 본문에 시뮬레이션 결과(랩 타임, 콘 인식률)를 첨부한다.
5. `OWNERS` 파일의 리뷰어 중 최소 1인의 승인을 받아 병합한다.

자세한 규칙은 저장소 루트의 `CONTRIBUTING.md`를 따른다.

## 유지보수자 및 연락처

| 역할 | 위치 |
| --- | --- |
| 메인테이너 목록 | `OWNERS` |
| 대회 운영 이슈 | `docs/SUBMISSION_GUIDE.md` 참조 후 이슈 등록 |
| FSDS 설치/환경 문제 | `docs/reference_materials/lecture1_fsds_install.txt` |

## 추가 문서

| 문서 | 위치 |
| --- | --- |
| 대회 제출 가이드 | `docs/SUBMISSION_GUIDE.md` |
| FSDS 설치 가이드 | `docs/reference_materials/lecture1_fsds_install.txt` |
| SLAM 강의 노트북 | `docs/reference_materials/lecture4_slam.ipynb` |
| V2X 강의 노트북 | `docs/reference_materials/lecture6_v2x.ipynb` |
| 시뮬레이터 설정 메모 | `src/simulator/README.md` |
| 제출 패키지 메모 | `submission/README.md` |
| 자율 모드 노트 | `src/autonomous/AGENTS.md` |
| 제출 모드 노트 | `submission/AGENTS.md` |
| 에이전트 운영 규약 | `AGENTS.md` |

## 라이선스

`LICENSE` 파일의 조항을 따른다. 대회 제출 시 외부 라이브러리 고지 의무가 있으면 `docs/SUBMISSION_GUIDE.md`를 함께 확인할 것.