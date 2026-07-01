# Formula Student Driverless — Autonomous Racing Stack

> **FSD (Formula Student Driverless) 자율주행 레이싱 소프트웨어 스택**
> Autonomous racing software stack for the Formula Student Driverless competition

이 저장소는 FSD 대회를 위한 완전 자율주행 레이싱 차량 소프트웨어를 제공합니다. SLAM, 콘 감지/분류, Pure Pursuit 추종 제어, 랩 타이머, V2X(차량-사물 통신) 어댑터, 그리고 대회 규정에 맞는 Docker 제출 패키지를 포함합니다.

This repository provides a full-stack autonomous driving system for the Formula Student Driverless competition. It bundles SLAM, cone detection/classification, pure-pursuit lane following, lap timing, a V2X (RSU) adapter, and a competition-ready Docker submission package.

---

## 목차 / Table of Contents

1. [개요 / Overview](#개요--overview)
2. [대상 사용자 / Target Users](#대상-사용자--target-users)
3. [주요 기능 / Features](#주요-기능--features)
4. [아키텍처 / Architecture](#아키텍처--architecture)
5. [저장소 구조 / Repository Layout](#저장소-구조--repository-layout)
6. [빠른 시작 / Quick Start](#빠른-시작--quick-start)
7. [설정 / Configuration](#설정--configuration)
8. [명령어 레퍼런스 / Commands Reference](#명령어-레퍼런스--commands-reference)
9. [로컬 개발 / Local Development](#로컬-개발--local-development)
10. [테스트 / Testing](#테스트--testing)
11. [시뮬레이션 / Simulation](#시뮬레이션--simulation)
12. [제출 패키지 / Submission Package](#제출-패키지--submission-package)
13. [참고 자료 / Reference Materials](#참고-자료--reference-materials)
14. [기여 / Contributing](#기여--contributing)
15. [라이선스 / License](#라이선스--license)

---

## 개요 / Overview

FSD 대회는 카메라와 LiDAR로 트랙을 인식하고, 노란색/파란색 콘으로 정의된 미니멀 패스 레인을 따라 자율 주행을 수행하며, V2X 인프라(RSU: Road-Side Unit)와 통신해 추가 정보를 받는 종목입니다. 본 스택은 그 파이프라인을 ROS 1 기반 노드들로 구현하며, 대회 규정에 맞는 Docker 이미지 형태로 패키징됩니다.

The Formula Student Driverless competition asks teams to detect the track, follow a cone-defined lane (yellow/blue), and exchange data with a Road-Side Unit (RSU) over V2X — all without a human driver. This stack implements that pipeline as ROS 1 nodes and packages it as a Docker image that satisfies the competition submission rules.

### 핵심 설계 원칙 / Core design principles

| 원칙 | 설명 / Description |
| --- | --- |
| 모듈화 / Modular | perception, control, v2x, utils 모듈을 독립 패키지로 분리 — perception, control, v2x and utils live as independent packages |
| 재현 가능 / Reproducible | 모든 의존성을 Docker로 고정 — all dependencies pinned via Docker images |
| 대회 규정 준수 / Competition-compliant | 오프라인 실행, 단일 진입점, 리소스 제한 내 구동 — runs offline, single entry point, bounded resources |
| 안전 페일 / Safe-fail | 워치독 노드로 비정상 상태 감지 후 차량 정지 — watchdog halts the vehicle on anomalies |
| 이중 트랙 / Dual pipeline | 개발용 `src/autonomous/` 와 제출용 `submission/` 두 트랙 유지 — dev (`src/autonomous/`) and submission (`submission/`) tracks |

### 빠른 상태 요약 / Quick status

| 항목 / Item | 값 / Value |
| --- | --- |
| 대회 트랙 / Competition class | Formula Student Driverless (FS-AI / FS-DV / FS-ASC 합본) |
| 미들웨어 / Middleware | ROS 1 (Melodic/Noetic 가정) — assumed Melodic/Noetic |
| 런타임 / Runtime | Docker (CPU only, ROS base) |
| 입력 센서 / Sensors | 카메라, LiDAR, 휠 오도메트리 — camera, LiDAR, wheel odometry |
| 출력 / Outputs | Ackermann 조향·스로틀, 콘 트랙 맵, 랩 타임 — Ackermann steer/throttle, cone map, lap time |
| 외부 통신 / External comms | V2X (RSU 어댑터) |
| 상태 / Status | 대회 제출용 프로토타입 — competition submission prototype |

---

## 대상 사용자 / Target Users

- **FSD 대회 참가 팀 / FSD competing teams** — 본 스택을 베이스라인으로 사용하고 차량 플랫폼에 맞춰 튜닝 — adopt as a baseline and tune to the team's car
- **자율주행 알고리즘 연구자 / Autonomous-driving researchers** — perception/control 알고리즘을 분리해 단독 실험 — isolate perception/control modules for experiments
- **ROS 1 학습자 / ROS 1 learners** — 실제 대회 도메인 예제로 학습 — learn through a real competition domain
- **심사위원/리뷰어 / Judges & reviewers** — `submission/` 패키지로 규격 준수 여부 검증 — verify rule compliance via `submission/`

---

## 주요 기능 / Features

| 영역 / Area | 기능 / Capability | 코드 위치 / Source |
| --- | --- | --- |
| Perception — 콘 감지 / Cone detection | 카메라/LiDAR에서 콘 후보 추출 — extract cone candidates from camera/LiDAR | `src/autonomous/modules/perception/cone_detector.py`, `submission/src/perception/cone_detector.py` |
| Perception — 콘 분류 / Cone classification | 노란/파란 대형/소형 콘 분류 — classify yellow/blue large/small cones | `src/autonomous/modules/perception/cone_classifier.py`, `submission/src/perception/cone_classifier.py` |
| Perception — SLAM | 콘 기반 트랙 SLAM 및 로컬라이제이션 — cone-based track SLAM and localization | `src/autonomous/modules/perception/slam.py`, `submission/src/perception/slam.py` |
| Control — Pure Pursuit | 콘 중심선을 따라가는 조향 추종 — centerline tracking via pure pursuit | `src/autonomous/modules/control/pure_pursuit.py`, `submission/src/control/pure_pursuit.py` |
| Control — 속도 / Speed | 구간별 속도 프로파일 — segment-based speed profile | `src/autonomous/modules/control/speed.py`, `submission/src/control/speed.py` |
| Utils — 랩 타이머 / Lap timer | 시작선 통과 시 랩 타임 기록 — record lap times on start-line crossings | `src/autonomous/modules/utils/lap_timer.py`, `submission/src/utils/lap_timer.py` |
| Utils — 워치독 / Watchdog | 비정상 상태에서 정지 명령 발행 — issue stop on anomalies | `src/autonomous/modules/utils/watchdog.py`, `submission/src/utils/watchdog.py` |
| V2X / RSU | Road-Side Unit 어댑터 — Road-Side Unit adapter | `submission/src/v2x/rsu.py` |
| Drivers | basic / advanced / autonomous / competition 드라이버 — driver variants | `submission/src/drivers/` |
| 제출 패키지 / Submission | 규격 맞춤 Docker 이미지 및 런처 — competition-shaped Docker image and launch | `submission/Dockerfile`, `submission/launch/competition.launch` |

---

## 아키텍처 / Architecture

### 컴포넌트 흐름 / Component flow

1. **센서 입력** — 카메라 이미지, LiDAR 스캔, 휠 오도메트리가 ROS 토픽으로 유입 — camera, LiDAR scan, wheel odometry enter as ROS topics.
2. **Perception 노드** — `cone_detector` → `cone_classifier` → `slam` 순으로 콘 위치/색상을 정리 — `cone_detector` → `cone_classifier` → `slam` produce cone positions and colors.
3. **Planning/Control 노드** — `pure_pursuit` 가 SLAM 결과를 따라 목표 조향 곡선을 생성, `speed` 가 구간 속도 결정 — `pure_pursuit` produces steering, `speed` decides throttle/brake.
4. **드라이버 노드** — `competition_driver` (또는 제출용 `drivers/competition.py`)가 Ackermann 명령을 차량으로 송신 — `competition_driver` sends Ackermann commands to the vehicle.
5. **V2X** — `v2x/rsu.py` 가 RSU 신호를 수신해 콘/속도 결정에 반영 — `v2x/rsu.py` ingests RSU data and feeds it into planning.
6. **Utils** — `lap_timer` 가 랩 타임을 기록, `watchdog` 가 모든 단계 헬스를 감시 — `lap_timer` records laps, `watchdog` monitors health.

### 노드-주제 매핑 / Node-topic map

| 노드 / Node | 입력 / Inputs | 출력 / Outputs | 비고 / Notes |
| --- | --- | --- | --- |
| `cone_detector` | `/camera/image_raw`, `/scan` | `/perception/cones_3d` | 후보 콘 추출 — candidate cones |
| `cone_classifier` | `/perception/cones_3d`, `/camera/image_raw` | `/perception/cones_classified` | 색상/사이즈 라벨 부착 — color/size labels |
| `slam` | `/perception/cones_classified`, `/odom` | `/slam/track_map`, `/slam/pose` | 트랙 맵 + 차량 포즈 — track map + pose |
| `pure_pursuit` | `/slam/track_map`, `/slam/pose` | `/control/trajectory` | 조향 경로 — steering path |
| `speed` | `/slam/track_map`, `/control/trajectory` | `/control/velocity` | 목표 속도 — target velocity |
| `competition_driver` | `/control/trajectory`, `/control/velocity` | `/ackermann_cmd` | 차량 명령 발행 — vehicle commands |
| `v2x/rsu` | UDP/TCP (RSU) | `/v2x/events` | 외부 신호 분배 — external signal fan-out |
| `lap_timer` | `/slam/pose`, `/v2x/events` | `/lap/time`, `/lap/count` | 랩 측정 — lap measurement |
| `watchdog` | 모든 노드 헬스 — all node healths | `/emergency_stop` | 이상 시 정지 — emergency stop |

> 토픽명은 `src/autonomous/config/params.yaml` 및 `submission/src/...` 의 런처에서 조정 가능 — topic names are tunable via the launchers and `params.yaml`.

---

## 저장소 구조 / Repository Layout

```text
.
├── AGENTS.md
├── CONTRIBUTING.md
├── LICENSE
├── OWNERS
├── README.md
├── in-memoria.db                # 캐시/메모 데이터베이스 — cache/memo DB
├── src/
│   ├── autonomous/              # 개발용 자율주행 노드 트리 — dev track
│   │   ├── AGENTS.md
│   │   ├── Dockerfile
│   │   ├── docker-compose.yml
│   │   ├── entrypoint.sh
│   │   ├── record_race.sh
│   │   ├── run_all.sh
│   │   ├── start.sh
│   │   ├── scripts/start_race.py
│   │   ├── config/              # bridge_no_camera.launch, params.yaml
│   │   ├── driver/competition_driver.py
│   │   ├── modules/
│   │   │   ├── perception/      # cone_detector, cone_classifier, slam
│   │   │   ├── control/         # pure_pursuit, speed
│   │   │   └── utils/           # lap_timer, watchdog
│   │   └── tests/test_algorithms.py
│   └── simulator/               # 시뮬레이터 설정 — simulator settings
│       ├── README.md
│       └── settings.json
├── scripts/package.sh           # 제출용 패키징 스크립트 — packaging script
├── docs/
│   ├── SUBMISSION_GUIDE.md
│   └── reference_materials/     # 강의 노트 — lecture notes (FS-DS install, SLAM, V2X)
├── submission/                  # 대회 제출용 패키지 — competition submission
│   ├── AGENTS.md
│   ├── Dockerfile
│   ├── README.md
│   ├── dev.sh
│   ├── docker-compose.yml
│   ├── run.sh
│   ├── launch/competition.launch
│   ├── src/
│   │   ├── drivers/             # basic, advanced, autonomous, competition
│   │   ├── perception/          # cone_detector, cone_classifier, slam
│   │   ├── control/             # pure_pursuit, speed
│   │   ├── v2x/                 # rsu
│   │   └── utils/               # lap_timer, watchdog
│   └── autonomous/              # 제출 트랙의 보조 컨테이너 — auxiliary container
│       ├── Dockerfile
│       ├── docker-compose.yml
│       ├── entrypoint.sh
│       ├── run_all.sh
│       ├── start.sh
│       ├── config/params.yaml
│       ├── driver/competition_driver.py
│       └── modules/perception/  # cone_detector, cone_classifier
```

> `submission/` 안에는 `src/autonomous` 의 로직이 다시 한번 미러링되어 있어, 대회 환경에서 단독으로 실행 가능한 형태를 갖추고 있습니다. `submission/` mirrors the `src/autonomous` logic so that the package runs standalone in the competition sandbox.

---

## 빠른 시작 / Quick Start

### 1. 사전 요구 사항 / Prerequisites

| 항목 / Item | 버전/설명 / Version or note |
| --- | --- |
| Docker Engine | 20.10+ 권장 — recommended |
| Docker Compose | v2 (`docker compose ...`) |
| 호스트 OS | Linux (Ubuntu 20.04/22.04 검증) — Ubuntu 20.04/22.04 verified |
| 디스크 / Disk | 컨테이너 이미지 + ROS base 약 5 GB 이상 — at least ~5 GB |
| GPU | 불필요 (CPU only 스택) — not required |

### 2. 저장소 클론 / Clone

```bash
git clone <repository-url> fsd-stack
cd fsd-stack
```

### 3. 개발 트랙 기동 / Run the dev track

```bash
cd src/autonomous
docker compose up --build
```

컨테이너가 올라가면 ROS 마스터와 노드들이 자동으로 실행됩니다. 토픽 목록은 `docker compose exec fsd-autonomous rostopic list` 로 확인하세요.

Once the container is up, the ROS master and nodes start automatically. Inspect topics via `docker compose exec fsd-autonomous rostopic list`.

### 4. 시뮬레이터 연결 / Connect the simulator

`src/simulator/README.md` 의 안내에 따라 시뮬레이터를 별도 프로세스로 띄우고, 동일 호스트 네트워크 또는 토픽 브릿지로 연결합니다. 카메라 입력이 없을 때는 `config/bridge_no_camera.launch` 로 더미 토픽을 게시할 수 있습니다.

Follow `src/simulator/README.md` to launch the simulator as a separate process and bridge it via host networking or topic relay. When no camera is available, `config/bridge_no_camera.launch` publishes dummy topics.

---

## 설정 / Configuration

| 파일 / File | 용도 / Purpose |
| --- | --- |
| `src/autonomous/config/params.yaml` | 알고리즘 하이퍼파라미터 (콘 임계값, 속도, 추종 게인) — algorithm hyperparameters |
| `src/autonomous/config/bridge_no_camera.launch` | 카메라 없을 때 더미 토픽 게시 — dummy topics without camera |
| `src/simulator/settings.json` | 시뮬레이터 트랙/차량 파라미터 — simulator track/vehicle parameters |
| `submission/launch/competition.launch` | 제출 트랙의 단일 진입 런처 — single entry launch for submission |
| `submission/autonomous/config/params.yaml` | 제출 트랙 하이퍼파라미터 — submission hyperparameters |
| `in-memoria.db` | 학습/메모 캐시 — memo cache database |

> 대회 환경에서 토픽 이름과 노드 명은 심사 도구가 검사할 수 있습니다. 변경 시 `submission/launch/competition.launch` 와 `params.yaml` 을 함께 갱신하세요 — under competition inspection, topic and node names may be audited. Keep the launcher and `params.yaml` in sync when changing them.

---

## 명령어 레퍼런스 / Commands Reference

| 명령 / Command | 위치 / Path | 설명 / Description |
| --- | --- | --- |
| `docker compose up --build` | `src/autonomous/` | 개발 트랙 전체 기동 — start dev track |
| `docker compose exec fsd-autonomous bash` | `src/autonomous/` | 컨테이너 진입 — enter the container |
| `./run_all.sh` | `src/autonomous/`, `submission/autonomous/` | 시뮬레이션 + 노드 일괄 기동 — run sim + nodes together |
| `./start.sh` | `src/autonomous/`, `submission/autonomous/` | 단일 진입 실행 스크립트 — single entry script |
| `./entrypoint.sh` | `src/autonomous/` | 컨테이너 부팅 훅 — container entrypoint |
| `./record_race.sh` | `src/autonomous/` | rosbag 기록 모드 — rosbag recording mode |
| `python3 scripts/start_race.py` | `src/autonomous/scripts/` | Python 기반 레이스 시작 — Python race launcher |
| `./scripts/package.sh` | `scripts/` | 제출용 Docker 이미지/번들 패키징 — build submission bundle |
| `./dev.sh` | `submission/` | 제출 패키지 개발 모드 실행 — dev-mode run |
| `./run.sh` | `submission/` | 제출 패키지 단일 진입 실행 — submission entrypoint |
| `roslaunch competition.launch` | `submission/launch/` | 대회 런처 — competition launcher |
| `roslaunch bridge_no_camera.launch` | `src/autonomous/config/` | 더미 카메라 토픽 게시 — publish dummy camera topics |
| `rostopic list` | 컨테이너 내부 / inside container | 활성 토픽 확인 — list active topics |
| `rosnode list` | 컨테이너 내부 / inside container | 활성 노드 확인 — list active nodes |

---

## 로컬 개발 / Local Development

| 단계 / Step | 작업 / Action |
| --- | --- |
| 1 | `src/autonomous/` 의 `Dockerfile` 에 새 의존성 추가 — add dependencies in `src/autonomous/Dockerfile` |
| 2 | `modules/<area>/` 아래 새 노드 작성 — author a new node under `modules/<area>/` |
| 3 | `config/params.yaml` 에 파라미터 노출 — expose parameters in `config/params.yaml` |
| 4 | 컨테이너 재빌드: `docker compose build` — rebuild with `docker compose build` |
| 5 | `tests/test_algorithms.py` 에 단위 테스트 추가 — add unit tests in `tests/test_algorithms.py` |
| 6 | `src/autonomous/AGENTS.md` 의 모듈 규칙 준수 — follow the per-module rules in `src/autonomous/AGENTS.md` |
| 7 | 변경 사항을 `submission/` 트리에 동기화 (규격 호환성 유지) — mirror changes into `submission/` to keep spec parity |

---

## 테스트 / Testing

| 명령 / Command | 설명 / Description |
| --- | --- |
| `docker compose run --rm fsd-autonomous pytest src/autonomous/tests/` | 알고리즘 단위 테스트 — algorithm unit tests |
| `python3 -m unittest src/autonomous/tests/test_algorithms.py` | 컨테이너 외부에서 직접 실행 — direct run outside container (requires ROS env) |
| `./record_race.sh && rosbag play` | rosbag 재생으로 통합 회귀 테스트 — regression via recorded rosbags |
| `./run_all.sh` | 시뮬레이터 기반 종단간 시뮬레이션 — end-to-end with simulator |

CI 환경에서 알고리즘 회귀를 막으려면 `tests/` 하위의 단위 테스트를 항상 통과시킨 뒤 PR 을 올려 주세요.

To guard against algorithm regressions in CI, ensure `tests/` unit tests pass before opening a PR.

---

## 시뮬레이션 / Simulation

`src/simulator/` 디렉터리는 FSD 규격 트랙을 시뮬레이션하기 위한 설정 파일을 보관합니다. `settings.json` 의 차량/트랙 파라미터를 환경에 맞춰 조정한 뒤 시뮬레이터를 별도 프로세스로 실행하고, 본 스택과 동일 호스트 네트워크 또는 토픽 브릿지로 연결합니다. 상세 절차는 [`src/simulator/README.md`](src/simulator/README.md) 를 참조하세요.

The `src/simulator/` directory holds settings for a FSD-shaped track. Tweak `settings.json` for your environment, run the simulator as a separate process, and connect it via host networking or a topic bridge. See [`src/simulator/README.md`](src/simulator/README.md) for details.

| 항목 / Item | 기본값 / Default | 위치 / Path |
| --- | --- | --- |
| 트랙 길이 / Track length | 환경별 상이 — varies | `src/simulator/settings.json` |
| 콘 색상 / Cone colors | 노랑/파랑 — yellow/blue | `src/simulator/settings.json` |
| 차량 토픽 / Vehicle topics | ROS 표준 — ROS standard | 시뮬레이터 기본값 — simulator defaults |

---

## 제출 패키지 / Submission Package

`submission/` 디렉터리는 FSD 대회 심사 환경에서 그대로 실행될 수 있도록 패키징된 형태입니다.

The `submission/` directory is the competition-ready packaging of the stack.

| 산출물 / Artifact | 설명 / Description |
| --- | --- |
| `submission/Dockerfile` | 대회 규격 베이스 이미지 — competition-shaped base image |
| `submission/docker-compose.yml` | 단일 서비스 compose — single-service compose |
| `submission/launch/competition.launch` | 단일 진입 ROS 런처 — single-entry ROS launcher |
| `submission/src/drivers/` | basic / advanced / autonomous / competition 4종 드라이버 — 4 driver variants |
| `submission/src/v2x/rsu.py` | RSU 어댑터 — RSU adapter |
| `submission/run.sh` | 심사 스크립트가 호출하는 단일 진입점 — single entry point invoked by judges |
| `submission/dev.sh` | 개발 모드 진입점 — dev-mode entry point |
| `submission/autonomous/` | 보조 컨테이너 빌드 자산 — auxiliary container assets |

### 빌드 절차 / Build procedure

1. `submission/` 디렉터리로 이동 — `cd submission/`
2. `./dev.sh` 로 로컬 검증 — validate locally with `./dev.sh`
3. `./scripts/package.sh` 로 제출 번들 생성 — produce submission bundle with `./scripts/package.sh`
4. 산출물을 [`docs/SUBMISSION_GUIDE.md`](docs/SUBMISSION_GUIDE.md) 의 절차에 따라 제출 — submit per [`docs/SUBMISSION_GUIDE.md`](docs/SUBMISSION_GUIDE.md)

> 심사 환경은 보통 인터넷이 차단됩니다. 빌드 시점의 베이스 이미지에 모든 의존성을 미리 포함해 주세요 — the judging sandbox is usually offline. Bake all dependencies into the base image at build time.

---

## 참고 자료 / Reference Materials

| 자료 / Material | 위치 / Path | 용도 / Purpose |
| --- | --- | --- |
| 제출 가이드 / Submission guide | [`docs/SUBMISSION_GUIDE.md`](docs/SUBMISSION_GUIDE.md) | 대회 제출 절차 요약 — submission procedure summary |
| FSDS 설치 노트 / FSDS install notes | [`docs/reference_materials/lecture1_fsds_install.txt`](docs/reference_materials/lecture1_fsds_install.txt) | Full Self-Driving Simulator 설치 절차 — FSDS install steps |
| SLAM 강의 노트 / SLAM lecture | [`docs/reference_materials/lecture4_slam.ipynb`](docs/reference_materials/lecture4_slam.ipynb) | 콘 기반 SLAM 이론 — cone-based SLAM theory |
| V2X 강의 노트 / V2X lecture | [`docs/reference_materials/lecture6_v2x.ipynb`](docs/reference_materials/lecture6_v2x.ipynb) | V2X/RSU 프로토콜 — V2X/RSU protocols |
| 시뮬레이터 문서 / Simulator docs | [`src/simulator/README.md`](src/simulator/README.md) | 시뮬레이터 사용법 — simulator usage |
| 제출 트랙 노트 / Submission notes | [`submission/README.md`](submission/README.md) | 제출 패키지 상세 — submission package details |
| 모듈 규칙 / Module rules | `src/autonomous/AGENTS.md`, `submission/AGENTS.md` | 각 트랙의 모듈별 규칙 — per-track module rules |

---

## 기여 / Contributing

기여 절차는 [`CONTRIBUTING.md`](CONTRIBUTING.md) 를 따릅니다. 주요 원칙:

Follow [`CONTRIBUTING.md`](CONTRIBUTING.md). Key principles:

- **두 트랙 동기화** — `src/autonomous/` 와 `submission/` 양쪽에 동일 변경 적용 — apply identical changes in both `src/autonomous/` and `submission/`
- **노드 인터페이스 고정** — 토픽 이름/메시지 타입 변경 시 런처와 `params.yaml` 함께 갱신 — keep launchers and `params.yaml` in sync when changing topics or messages
- **테스트 우선** — `tests/test_algorithms.py` 통과 후 PR — pass `tests/test_algorithms.py` before opening a PR
- **문서 동행** — 토픽 변경 시 `docs/SUBMISSION_GUIDE.md` 도 갱신 — update `docs/SUBMISSION_GUIDE.md` when topics change

책임자 목록은 [`OWNERS`](OWNERS) 파일을 참조하세요. 모듈별 세부 규칙은 각 `AGENTS.md` 에 정의되어 있습니다.

Owners are listed in [`OWNERS`](OWNERS). Per-module rules live in each `AGENTS.md`.

---

## 라이선스 / License

본 저장소는 [`LICENSE`](LICENSE) 파일의 조항에 따라 배포됩니다. 대회 제출 시 라이선스 고지 의무가 있는지 [`docs/SUBMISSION_GUIDE.md`](docs/SUBMISSION_GUIDE.md) 에서 확인해 주세요.

This repository is distributed under the terms of [`LICENSE`](LICENSE). Check [`docs/SUBMISSION_GUIDE.md`](docs/SUBMISSION_GUIDE.md) for any attribution obligations required by the competition rules.