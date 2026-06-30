# Formula Student Driverless — Autonomous Racing Stack

> **FSD (Formula Student Driverless) 자율주행 레이싱 소프트웨어 스택**
> Autonomous racing software stack for the Formula Student Driverless competition

이 저장소는 FSD 대회를 위한 완전 자율주행 레이싱 차량 소프트웨어를 제공합니다. SLAM, 콘 감지/분류, Pure Pursuit 추종 제어, 랩 타이머, V2X(차량-사물 통신) 어댑터, 그리고 대회용 Docker 제출 패키지를 포함합니다.

This repository provides a full-stack autonomous driving system for the Formula Student Driverless competition. It bundles SLAM, cone detection/classification, pure-pursuit lane following, lap timing, a V2X adapter, and a competition-ready Docker submission package.

---

## Table of Contents / 목차

1. [Overview / 개요](#overview--개요)
2. [Target Users / 대상 사용자](#target-users--대상-사용자)
3. [Features / 주요 기능](#features--주요-기능)
4. [Architecture / 아키텍처](#architecture--아키텍처)
5. [Repository Layout / 저장소 구조](#repository-layout--저장소-구조)
6. [Quick Start / 빠른 시작](#quick-start--빠른-시작)
7. [Configuration / 설정](#configuration--설정)
8. [Commands Reference / 명령어 레퍼런스](#commands-reference--명령어-레퍼런스)
9. [Local Development / 로컬 개발](#local-development--로컬-개발)
10. [Testing / 테스트](#testing--테스트)
11. [Simulation / 시뮬레이션](#simulation--시뮬레이션)
12. [Submission Package / 제출 패키지](#submission-package--제출-패키지)
13. [Reference Materials / 참고 자료](#reference-materials--참고-자료)
14. [Contributing / 기여](#contributing--기여)
15. [License / 라이선스](#license--라이선스)

---

## Overview / 개요

FSD 대회는 카메라와 LiDAR로 트랙을 인식하고, 노란색/파란색 콘으로 정의된 미니멀 패스 레인을 따라 자율 주행을 수행하며, V2X 인프라(RSU)와 통신해 추가 정보를 받는 종목입니다. 본 스택은 그 파이프라인을 ROS 1 기반 노드들로 구현하며, 대회 규정에 맞는 Docker 이미지 형태로 패키징됩니다.

The Formula Student Driverless competition asks teams to detect the track, follow a cone-defined lane (yellow/blue), and exchange data with a Roadside Unit (RSU) over V2X — all without a human driver. This stack implements that pipeline as ROS 1 nodes and packages it as a Docker image that satisfies competition submission rules.

### 핵심 설계 원칙 / Core design principles

- **모듈화 (Modular)** — perception, control, utils, v2x를 독립 패키지로 분리
  Each subsystem (perception, control, utils, v2x) lives in its own module.
- **대회 즉시 실행 가능 (Competition-ready)** — `submission/` 디렉터리에 Docker 제출 패키지 포함
  A self-contained Docker submission package lives in `submission/`.
- **시뮬레이션 우선 (Simulation-first)** — 로컬 시뮬레이터에서 알고리즘 반복 검증
  Algorithms are iterated against the local simulator before track deployment.
- **관측 가능성 (Observable)** — watchdog, 랩 타이머, race recorder가 상태를 지속 모니터링
  Watchdog, lap timer, and race recorder continuously surface runtime state.

---

## Target Users / 대상 사용자

| 사용자 / Audience | 용도 / Use case |
|---|---|
| FSD 동아리 팀원 | 대회 차량에 탑재할 자율주행 스택 개발 및 튜닝 |
| 캡틴 / 팀 리드 | `submission/` 패키지로 대회 운영 측에 제출 |
| 멘토 / 심사위원 | 알고리즘 단위 테스트(`tests/`)와 시뮬레이터 재현으로 동작 검증 |
| 신규 합류자 | `docs/reference_materials/` 강의 노트와 코드를 함께 학습 |

---

## Features / 주요 기능

### Perception / 인지

- **콘 감지 (Cone Detection)** — 카메라/LiDAR 입력에서 콘 후보를 추출
- **콘 분류 (Cone Classification)** — 색상/형상 기반으로 노란색·파란색 콘 라벨링
- **SLAM** — 차량의 위치 추정과 트랙 점유 격자 맵 동시 구축

### Control / 제어

- **Pure Pursuit** — 콘으로 정의된 중심 경로를 따라가는 스티어링 목표치 산출
- **Speed Control** — 곡률/구간에 따른 스로틀·브레이크 명령 생성

### Utilities / 유틸리티

- **Lap Timer** — 시작선 통과 이벤트 기반 랩 타임 기록
- **Watchdog** — 노드 헬스체크와 타임아웃 시 안전 정지
- **Race Recorder** — 레이스 데이터를 ROS bag으로 기록

### V2X / 인프라 통신

- **RSU Adapter** — 노변 기지국(RSU)과 데이터 송수신 어댑터

### Submission / 제출

- **Docker 이미지** — 대회 규격에 맞춘 컨테이너 패키지
- **Launch 파일** — 카메라 없는 브리지 모드와 풀 스택 모드 분리

---

## Architecture / 아키텍처

스택은 ROS 1 노드 그래프 위에서 동작하며, 데이터는 다음 단계로 흐릅니다.

The stack runs as a ROS 1 node graph. Data flows through the following stages:

| 단계 / Stage | 모듈 / Module | 책임 / Responsibility |
|---|---|---|
| 1. Sense | `perception/cone_detector` | 카메라/LiDAR에서 콘 후보 추출 |
| 2. Classify | `perception/cone_classifier` | 노란색/파란색 라벨 부여 |
| 3. Localize | `perception/slam` | 차량 자세 추정 + 트랙 맵 생성 |
| 4. Plan | `control/pure_pursuit` | 중심 경로 기반 스티어링 목표 산출 |
| 5. Actuate | `control/speed` | 스로틀/브레이크 명령 생성 |
| 6. Monitor | `utils/watchdog` | 노드 헬스체크 + 타임아웃 정지 |
| 7. Time | `utils/lap_timer` | 랩 타임 측정 및 기록 |
| 8. Communicate | `v2x/rsu` | RSU와 V2X 메시지 교환 |

### 요청 흐름 / Request flow

1. 센서 드라이버가 `/camera`, `/lidar`, `/imu` 등 토픽 발행
2. `cone_detector`가 콘 후보를 `/cones/raw`로 발행
3. `cone_classifier`가 색상 라벨을 붙여 `/cones/classified`로 발행
4. `slam`이 `/odom`, `/map`을 발행하고 트랙 점유 격자를 갱신
5. `competition_driver`가 위 토픽들을 구독해 미들레인 중심 경로 산출
6. `pure_pursuit`가 스티어링, `speed`가 스로틀/브레이크 명령을 `/cmd_vel`로 발행
7. `watchdog`이 모든 노드의 heartbeat를 감시, 미응답 시 정지 명령
8. `lap_timer`가 시작선 통과를 감지해 랩 타임 누적
9. `rsu` 어댑터가 대회 인프라(RSU)와 보조 데이터 송수신

### 런타임 토폴로지 / Runtime topology

| 컨테이너 / Container | 역할 / Role | 의존 / Depends on |
|---|---|---|
| `autonomous` | 메인 자율주행 노드 그래프 | ROS master, 차량 인터페이스 |
| `bridge_no_camera` | 카메라 제외 센서 브리지 | ROS master |
| (옵션) `simulator` | 시뮬레이션 데이터 송출 | ROS master |

자세한 노드/토픽 매핑은 `submission/launch/competition.launch`를 참고하십시오.
See `submission/launch/competition.launch` for the full node and topic mapping.

---

## Repository Layout / 저장소 구조

```text
.
├── AGENTS.md                       # 저장소 운영 가이드 (참고용)
├── CONTRIBUTING.md                 # 기여 절차
├── LICENSE                         # 라이선스 전문
├── OWNERS                          # 책임자 명단
├── README.md                       # 본 문서
├── in-memoria.db                   # 로컬 상태 데이터베이스
├── scripts/
│   └── package.sh                  # 제출용 Docker 이미지 빌드 스크립트
├── docs/
│   ├── SUBMISSION_GUIDE.md         # 대회 제출 절차 안내
│   └── reference_materials/        # 강의 노트/실습 자료
├── src/
│   ├── autonomous/                 # 로컬 자율주행 스택
│   │   ├── Dockerfile
│   │   ├── docker-compose.yml
│   │   ├── entrypoint.sh
│   │   ├── start.sh
│   │   ├── run_all.sh
│   │   ├── record_race.sh
│   │   ├── config/
│   │   │   ├── bridge_no_camera.launch
│   │   │   └── params.yaml
│   │   ├── driver/
│   │   │   └── competition_driver.py
│   │   ├── modules/
│   │   │   ├── perception/
│   │   │   │   ├── cone_classifier.py
│   │   │   │   ├── cone_detector.py
│   │   │   │   └── slam.py
│   │   │   ├── control/
│   │   │   │   ├── pure_pursuit.py
│   │   │   │   └── speed.py
│   │   │   └── utils/
│   │   │       ├── lap_timer.py
│   │   │       └── watchdog.py
│   │   ├── scripts/
│   │   │   └── start_race.py
│   │   └── tests/
│   │       └── test_algorithms.py
│   └── simulator/                  # 시뮬레이터 설정
│       ├── README.md
│       └── settings.json
└── submission/                     # 대회 제출 패키지
    ├── AGENTS.md
    ├── Dockerfile
    ├── README.md
    ├── dev.sh
    ├── run.sh
    ├── docker-compose.yml
    ├── launch/
    │   └── competition.launch
    ├── src/
    │   ├── drivers/
    │   │   ├── advanced.py
    │   │   ├── autonomous.py
    │   │   ├── basic.py
    │   │   └── competition.py
    │   ├── perception/             # (src/autonomous/modules/perception 미러)
    │   ├── control/                # (src/autonomous/modules/control 미러)
    │   ├── utils/                  # (src/autonomous/modules/utils 미러)
    │   └── v2x/
    │       └── rsu.py
    └── autonomous/                 # submission 내부 자율주행 컨테이너
        ├── Dockerfile
        ├── docker-compose.yml
        ├── entrypoint.sh
        ├── start.sh
        ├── run_all.sh
        ├── config/params.yaml
        ├── driver/competition_driver.py
        └── modules/                # submission 모듈 미러
```

---

## Quick Start / 빠른 시작

### 사전 요구 사항 / Prerequisites

| 항목 / Item | 버전 / Version | 용도 / Purpose |
|---|---|---|
| Docker Engine | 20.10+ | 제출 컨테이너 실행 |
| Docker Compose | v2 | 멀티 컨테이너 오케스트레이션 |
| ROS 1 (Melodic/Noetic) | 개발 시 | 노트북에서 직접 디버깅 시 |
| Python | 3.8+ | 알고리즘/스크립트 |
| GPU (선택 / optional) | CUDA 호환 | 콘 분류 가속 |

### 로컬 시뮬레이터 실행 / Run local simulator

```bash
# 1. 시뮬레이터 디렉터리로 이동
cd src/simulator

# 2. 시뮬레이터 실행 (시뮬레이터별 절차는 src/simulator/README.md 참고)
# 2. Launch the simulator (see src/simulator/README.md for vendor-specific steps)
```

### 자율주행 스택 실행 / Launch autonomous stack

```bash
# src/autonomous 안에서 docker compose로 전체 스택 기동
cd src/autonomous
docker compose up -d

# 로그 확인
docker compose logs -f autonomous

# 정지
docker compose down
```

### 제출 패키지 빌드 / Build submission package

```bash
# 대회 규격 Docker 이미지 빌드
./scripts/package.sh

# 또는 submission 디렉터리에서 직접
cd submission
docker compose build
```

---

## Configuration / 설정

| 파일 / File | 위치 / Location | 설명 / Description |
|---|---|---|
| `params.yaml` | `src/autonomous/config/` | 콘 감지 임계값, Pure Pursuit 룩어헤드, 속도 프로파일 등 핵심 파라미터 |
| `params.yaml` | `submission/autonomous/config/` | 제출용 컨테이너 기본 파라미터 |
| `bridge_no_camera.launch` | `src/autonomous/config/` | 카메라 토픽 제외한 센서 브리지 ROS launch |
| `competition.launch` | `submission/launch/` | 대회 모드 풀 스택 ROS launch |
| `settings.json` | `src/simulator/` | 시뮬레이터 트랙/차량/센서 설정 |
| `docker-compose.yml` | `src/autonomous/`, `submission/` | 컨테이너 서비스 정의 및 볼륨 매핑 |

### 주요 튜닝 파라미터 / Key tunable parameters

| 파라미터 / Parameter | 기본값(예) / Example default | 의미 / Meaning |
|---|---|---|
| `perception.cone_min_area` | 80 px² | 콘 후보 최소 면적 |
| `perception.classifier.confidence_threshold` | 0.65 | 색상 분류 신뢰도 임계값 |
| `control.pure_pursuit.lookahead` | 2.5 m | 룩어헤드 거리 |
| `control.speed.max_throttle` | 0.6 | 최대 스로틀 |
| `control.speed.curvature_slowdown` | 0.4 | 곡률 감속 계수 |
| `utils.watchdog.heartbeat_timeout` | 1.0 s | 노드 헬스비트 타임아웃 |
| `utils.lap_timer.start_line_topic` | `/start_line` | 랩 시작선 트리거 토픽 |

> **주의 / Note** — 실제 기본값은 `params.yaml` 파일을 직접 확인하십시오. 위 값은 예시입니다.
> Always confirm the actual defaults in `params.yaml`. Values above are illustrative.

---

## Commands Reference / 명령어 레퍼런스

### 셸 스크립트 / Shell scripts

| 스크립트 / Script | 위치 / Path | 목적 / Purpose |
|---|---|---|
| `entrypoint.sh` | `src/autonomous/`, `submission/autonomous/` | 컨테이너 시작 시 ROS 환경 소싱 및 노드 실행 |
| `start.sh` | `src/autonomous/`, `submission/autonomous/` | 단일 노드 그래프 기동 |
| `run_all.sh` | `src/autonomous/`, `submission/autonomous/` | 전체 자율주행 스택 일괄 기동 |
| `record_race.sh` | `src/autonomous/` | ROS bag으로 레이스 데이터 녹화 |
| `start_race.py` | `src/autonomous/scripts/` | 대회 시작 신호 처리 Python 진입점 |
| `dev.sh` | `submission/` | 개발 모드(호스트 마운트) 진입 |
| `run.sh` | `submission/` | 제출 모드 실행 |
| `package.sh` | `scripts/` | 제출용 Docker 이미지 패키징 |

### Docker / Compose

| 명령어 / Command | 설명 / Description |
|---|---|
| `docker compose up -d` | 스택을 백그라운드로 기동 |
| `docker compose logs -f autonomous` | 메인 자율주행 컨테이너 로그 스트림 |
| `docker compose exec autonomous bash` | 컨테이너 내부 셸 진입 |
| `docker compose down` | 스택 종료 및 컨테이너 정리 |
| `docker compose build` | 이미지 재빌드 (코드 변경 후) |
| `docker compose restart <service>` | 개별 서비스 재시작 |

### ROS 1 디버깅 / ROS 1 debugging

| 명령어 / Command | 용도 / Use |
|---|---|
| `rostopic list` | 활성 토픽 목록 |
| `rostopic echo /cones/classified` | 분류된 콘 토픽 실시간 출력 |
| `rosnode list` / `rosnode info <node>` | 노드 상태/연결 확인 |
| `roslaunch submission/launch/competition.launch` | 대회 launch 직접 실행 |
| `rosbag record -a` | 전체 토픽 녹화 |
| `rqt_graph` | 노드 그래프 시각화 |

---

## Local Development / 로컬 개발

### 워크플로 / Workflow

1. **알고리즘 변경** — `src/autonomous/modules/` 아래 파일 편집
2. **단위 테스트** — `tests/test_algorithms.py` 실행으로 회귀 확인
3. **시뮬레이터 검증** — `src/simulator/`에서 가상 트랙으로 종단간 검증
4. **제출 패키지 반영** — 변경분을 `submission/src/`로 동기화
5. **Docker 재빌드** — `docker compose build` 후 `docker compose up -d`
6. **현장 시험** — 실차에 컨테이너 배포 후 watchdog/log 모니터링

### 개발 모드 진입 / Enter development mode

```bash
cd submission
./dev.sh
# 호스트의 src/가 컨테이너 /workspace로 마운트되어 즉시 코드 반영됨
# Host src/ is mounted to /workspace inside the container for hot reload
```

### 코드 스타일 / Code style

- Python: PEP 8, 들여쓰기 4 spaces
- ROS 노드: `if __name__ == '__main__'` 패턴, 명시적 `rospy.init_node`
- 모듈 의존성: perception → control → utils 단방향 유지 권장
- 파라미터: 코드 하드코딩 금지, `params.yaml`에서 로드

---

## Testing / 테스트

### 단위 테스트 실행 / Run unit tests

```bash
cd src/autonomous
python3 -m pytest tests/test_algorithms.py -v
```

### 테스트 범위 / Coverage areas

| 영역 / Area | 검증 항목 / What is verified |
|---|---|
| Cone classifier | 색상 라벨 정확도, 신뢰도 임계값 경계 |
| Cone detector | 면적/종횡비 필터, 노이즈 제거 |
| Pure pursuit | 룩어헤드 거리별 스티어링 각도 |
| Speed control | 곡률 감속 곡선, 최대 스로틀 클램프 |
| SLAM | 루프 클로저 일관성 (간이 자검증) |
| Lap timer | 시작선 트리거 이벤트 카운팅 |
| Watchdog | 타임아웃 시 안전 정지 명령 발행 |

### 시뮬레이터 기반 종단간 검증 / End-to-end simulator validation

```bash
# 1) 시뮬레이터 기동
cd src/simulator && <simulator-binary> &

# 2) 자율주행 스택 기동
cd ../autonomous && docker compose up -d

# 3) 모니터링
docker compose logs -f autonomous
rostopic hz /cmd_vel
```

---

## Simulation / 시뮬레이션

`src/simulator/`는 차량 동역학과 트랙을 가상으로 제공하는 시뮬레이터의 설정 디렉터리입니다. 자세한 사용법은 `src/simulator/README.md`를 참고하십시오.

`src/simulator/` is the configuration directory for the simulator that provides virtual vehicle dynamics and track environments. Refer to `src/simulator/README.md` for details.

| 항목 / Item | 경로 / Path |
|---|---|
| 시뮬레이터 설정 | `src/simulator/settings.json` |
| 시뮬레이터 안내 문서 | `src/simulator/README.md` |
| 강의 노트 (설치) | `docs/reference_materials/lecture1_fsds_install.txt` |
| 강의 노트 (SLAM) | `docs/reference_materials/lecture4_slam.ipynb` |
| 강의 노트 (V2X) | `docs/reference_materials/lecture6_v2x.ipynb` |

---

## Submission Package / 제출 패키지

`submission/`은 대회 운영 측에 그대로 제출 가능한 형태의 패키지입니다.

The `submission/` directory is a ready-to-submit package for the competition organizers.

| 구성 / Component | 파일 / File | 역할 / Role |
|---|---|---|
| 제출 Dockerfile | `submission/Dockerfile` | 대회 규격 컨테이너 빌드 |
| Compose 정의 | `submission/docker-compose.yml` | 제출 컨테이너 서비스 |
| 런치 파일 | `submission/launch/competition.launch` | 대회 모드 노드 그래프 |
| 개발 진입 | `submission/dev.sh` | 호스트 마운트 개발 모드 |
| 제출 실행 | `submission/run.sh` | 제출 컨테이너 실행 |
| 드라이버 계층 | `submission/src/drivers/{basic,autonomous,advanced,competition}.py` | 자율/수동/고급 드라이버 토글 |
| 미러 모듈 | `submission/src/{perception,control,utils,v2x}/` | `src/autonomous/modules/` 동기화본 |
| 자율 컨테이너 | `submission/autonomous/` | 제출 환경 내부 자율주행 컨테이너 |

### 제출 절차 / Submission steps

1. `./scripts/package.sh` 실행으로 Docker 이미지 생성
2. `submission/` 디렉터리를 대회 운영 측에 전달
3. 운영 측은 `docker compose up -d`로 스택 기동
4. 시작 신호 입력 시 `start_race.py`가 차량에 미션 시작 명령 전달

자세한 절차는 [`docs/SUBMISSION_GUIDE.md`](docs/SUBMISSION_GUIDE.md)를 참조하십시오.
See [`docs/SUBMISSION_GUIDE.md`](docs/SUBMISSION_GUIDE.md) for the full procedure.

---

## Reference Materials / 참고 자료

| 자료 / Material | 경로 / Path | 주제 / Topic |
|---|---|---|
| 제출 가이드 | `docs/SUBMISSION_GUIDE.md` | 대회 제출 절차 |
| 강의 1 — FSDS 설치 | `docs/reference_materials/lecture1_fsds_install.txt` | FSD 시뮬레이터 설치 절차 |
| 강의 4 — SLAM | `docs/reference_materials/lecture4_slam.ipynb` | SLAM 이론/실습 |
| 강의 6 — V2X | `docs/reference_materials/lecture6_v2x.ipynb` | V2X/RSU 이론/실습 |

---

## Contributing / 기여

1. `CONTRIBUTING.md`의 절차와 코드 리뷰 가이드라인을 숙지하십시오
2. 기능 브랜치를 생성하고 변경 사항을 커밋합니다
3. `src/autonomous/tests/`에 회귀 테스트를 추가/갱신합니다
4. 변경된 모듈이 `submission/src/`에도 반영되었는지 확인합니다
5. Pull Request를 열고 OWNERS의 리뷰를 요청합니다
6. CI의 단위 테스트와 시뮬레이터 검증이 모두 통과해야 병합됩니다

1. Read the procedure and review guidelines in `CONTRIBUTING.md`.
2. Create a feature branch and commit your changes.
3. Add or update regression tests in `src/autonomous/tests/`.
4. Make sure mirrored modules in `submission/src/` are in sync.
5. Open a Pull Request and request review from OWNERS.
6. CI unit tests and simulator validation must pass before merge.

---

## License / 라이선스

본 저장소는 [`LICENSE`](LICENSE) 파일에 명시된 조건 하에 배포됩니다. 자세한 내용은 해당 파일을 참고하십시오.

This repository is distributed under the terms described in the [`LICENSE`](LICENSE) file. Refer to that file for details.