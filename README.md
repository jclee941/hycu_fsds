# Formula Student Driverless — Autonomous Racing Stack

> **FSD (Formula Student Driverless) 자율주행 레이싱 소프트웨어 스택**
> Autonomous racing software stack for the Formula Student Driverless competition

[![ROS 1](https://img.shields.io/badge/ROS-1-noetic-22314E?logo=ros)](https://wiki.ros.org)
[![Docker](https://img.shields.io/badge/Docker-ready-2496ED?logo=docker&logoColor=white)](https://www.docker.com)
[![Python 3](https://img.shields.io/badge/Python-3.x-3776AB?logo=python&logoColor=white)](https://www.python.org)
[![Status](https://img.shields.io/badge/Status-Competition%20Ready-2EA043)]()

## 한눈에 보기 / At a Glance

이 저장소는 FSD 대회를 위한 완전 자율주행 레이싱 차량 소프트웨어 스택입니다. 카메라/LiDAR 기반 SLAM, 콘 감지·분류, Pure Pursuit 추종 제어, 랩 타이머, V2X(RSU) 어댑터, 그리고 대회 규정에 맞는 Docker 제출 패키지를 포함합니다.

This repository delivers a full-stack autonomous racing system for the Formula Student Driverless competition: cone-based perception, SLAM, pure-pursuit control, lap timing, a V2X (RSU) adapter, and a competition-compliant Docker submission image.

| 항목 / Item | 내용 / Description |
| --- | --- |
| 플랫폼 / Platform | ROS 1 (Noetic), Python 3, Docker |
| 대회 / Competition | Formula Student Driverless (FSD) |
| 핵심 노드 / Core nodes | SLAM, 콘 감지·분류, Pure Pursuit, V2X RSU |
| 차량 인터페이스 / Vehicle I/O | `competition_driver.py` (competition entry point) |
| 시뮬레이션 / Simulator | FSDS (Formula Student Driverless Simulator) |
| 제출 형태 / Submission | Docker 이미지 (FSG 규격) |
| 작업 영역 / Workspaces | `src/autonomous/` (개발), `submission/` (대회 제출) |
| 데이터 / Data | `in-memoria.db` (런타임 상태 스냅샷) |
| 라이선스 / License | `LICENSE` 참조 |

### 운영 흐름 / Operating Flow

1. **Perception** — 카메라/LiDAR로 콘(노란/파란) 감지 및 분류, SLAM으로 위치 추정.
2. **Planning** — 감지된 콘 경계로 미니멀 패스 레인을 구성.
3. **Control** — Pure Pursuit으로 조향각 산출, 속도 노드가 종/횡 방향 속도 명령 생성.
4. **V2X** — RSU(Road-Side Unit)로부터 수신한 보조 정보(미션 플래그 등)를 융합.
5. **Driver** — `competition_driver.py`가 위 신호를 단일 명령으로 묶어 차량에 전달.
6. **Telemetry** — `lap_timer.py`와 `watchdog.py`가 랩 타임·안전 상태를 모니터링.

---

## 1. Overview / 개요

FSD 대회는 카메라와 LiDAR로 트랙을 인식하고, 노란색/파란색 콘으로 정의된 미니멀 패스(minimal pass) 레인을 따라 자율 주행을 수행하며, V2X 인프라(RSU: Road-Side Unit)와 통신해 추가 정보를 받는 종목입니다. 본 스택은 그 파이프라인을 ROS 1 노드들로 구현하고, 대회 규정에 맞는 Docker 이미지로 패키징합니다.

The Formula Student Driverless competition asks teams to detect the track, follow a cone-defined lane (yellow/blue), and exchange data with a Road-Side Unit (RSU) over V2X — all without a human driver. This stack implements that pipeline as ROS 1 nodes and packages it as a Docker image that satisfies the competition submission rules.

### 핵심 설계 원칙 / Core design principles

- **모듈화 (Modular)** — perception / control / utils / v2x 디렉터리가 명확히 분리되어 개별 단위 테스트가 가능합니다.
- **재현 가능 (Reproducible)** — 모든 런타임 의존성을 `Dockerfile`에 고정해 동일 시뮬레이션·대회 환경에서 재현됩니다.
- **대회 친화 (Competition-friendly)** — 진입점은 단일 `competition_driver.py`이며, 제출 패키지는 `run.sh`로 즉시 실행됩니다.
- **안전 우선 (Safety-first)** — `watchdog.py`가 상태 이상을 감지하면 차량을 정지 상태로 강제 전환합니다.

---

## 2. Target Users / 대상 사용자

| 사용자 / User | 활용 시나리오 / Scenario |
| --- | --- |
| FSD 팀원 / FSD team member | 시뮬레이터 검증, 실차 통합, 대회 제출 |
| 신규 합류자 / New member | `src/autonomous/`에서 노드 단위로 학습 |
| 심사위원·운영자 / Organizers | `submission/`의 `run.sh`로 즉시 데모 실행 |
| 연구자·학생 / Researchers | 알고리즘 교체(`pure_pursuit.py`, `cone_detector.py` 등) 실험 |

---

## 3. Features / 주요 기능

### Perception (인지)

- **콘 감지 / Cone detection** — `modules/perception/cone_detector.py`가 LiDAR/카메라 포인트에서 콘 후보를 추출.
- **콘 분류 / Cone classification** — `cone_classifier.py`가 색상(노란/파란) 및 신뢰도를 산출.
- **SLAM** — `modules/perception/slam.py`로 트랙 지도 작성 및 자기 위치 추정.

### Control (제어)

- **Pure Pursuit** — `modules/control/pure_pursuit.py`가 레인 중앙 경로에 대한 조향각을 산출.
- **Speed control** — `modules/control/speed.py`가 곡률 기반 목표 속도와 종방향 가감속을 결정.

### V2X (차량-사물 통신)

- **RSU 어댑터 / RSU adapter** — `src/v2x/rsu.py`가 RSU의 미션 토픽(예: 신호등 상태, 트랙 변경)을 수신.

### Utilities (유틸리티)

- **랩 타이머 / Lap timer** — `modules/utils/lap_timer.py`가 스타트/피니시 라인을 감지해 랩 타임을 기록.
- **워치독 / Watchdog** — `modules/utils/watchdog.py`가 노드 헬스체크와 타임아웃을 관리.

### Driver & Composition

- **Competition driver** — `driver/competition_driver.py`가 모든 신호를 단일 토픽 명령으로 합성.
- **Launch composition** — `config/bridge_no_camera.launch`, `submission/launch/competition.launch`로 노드 그래프 부트스트랩.

---

## 4. Architecture / 아키텍처

### 노드 책임 / Node Responsibilities

| 디렉터리 / Path | 책임 / Responsibility |
| --- | --- |
| `src/autonomous/driver/` | 대회 진입점. 모든 모듈 출력을 차량 명령으로 통합 |
| `src/autonomous/modules/perception/` | 콘 감지·분류, SLAM |
| `src/autonomous/modules/control/` | Pure Pursuit, 속도 제어 |
| `src/autonomous/modules/utils/` | 랩 타이머, 워치독 |
| `src/autonomous/config/` | 파라미터(`params.yaml`), 런치 파일 |
| `submission/src/perception/` | 제출용 인지 모듈 (autonomous와 동기 유지) |
| `submission/src/control/` | 제출용 제어 모듈 |
| `submission/src/v2x/` | 제출용 V2X RSU 어댑터 |
| `submission/src/utils/` | 제출용 유틸리티 (lap timer, watchdog) |
| `submission/launch/` | 대회용 런치 파일 (`competition.launch`) |

### 요청 흐름 / Request Flow

1. **입력** — 카메라/이미지 토픽과 LiDAR 포인트클라우드를 수신.
2. **Perception** — 콘 감지·분류 → 레인 경계 포인트.
3. **Planning (in driver)** — 경계로부터 패스 레인의 중심선 생성.
4. **Control** — Pure Pursuit이 조향각, Speed 노드가 속도 명령 산출.
5. **V2X fusion** — RSU 정보를 가중치/플래그로 반영.
6. **Driver output** — `/vesc/commands/...` 형태의 단일 명령 발행.
7. **Watchdog** — 모든 노드의 heartbeat를 모니터링, 이상 시 정지 명령.

---

## 5. Repository Layout / 저장소 구조

> AGENTS.md, OWNERS, CONTRIBUTING.md 같은 거버넌스 문서는 최상위에 있습니다. 실코드는 `src/autonomous/`(개발 작업 공간)와 `submission/`(대회 제출 패키지)에 분리되어 있습니다.

| 경로 / Path | 설명 / Description |
| --- | --- |
| `AGENTS.md`, `OWNERS`, `CONTRIBUTING.md` | 거버넌스 / Governance |
| `LICENSE` | 라이선스 전문 |
| `in-memoria.db` | 런타임 상태 스냅샷 SQLite (디버깅·시각화용) |
| `src/autonomous/` | 개발 작업 공간 (catkin/ROS 워크스페이스) |
| `src/autonomous/Dockerfile` | 개발용 도커 이미지 |
| `src/autonomous/docker-compose.yml` | 개발용 컴포즈 |
| `src/autonomous/driver/competition_driver.py` | 대회 진입 드라이버 |
| `src/autonomous/modules/perception/` | SLAM, 콘 감지·분류 |
| `src/autonomous/modules/control/` | Pure Pursuit, 속도 |
| `src/autonomous/modules/utils/` | 랩 타이머, 워치독 |
| `src/autonomous/config/params.yaml` | 런타임 파라미터 |
| `src/autonomous/config/bridge_no_camera.launch` | 카메라 미사용 브리지 런치 |
| `src/autonomous/tests/test_algorithms.py` | 알고리즘 단위 테스트 |
| `src/autonomous/scripts/start_race.py` | 시뮬레이터에서 레이스 시작 |
| `src/simulator/` | FSDS 설정 (`settings.json`, `README.md`) |
| `scripts/package.sh` | 제출 패키지 빌드 스크립트 |
| `docs/SUBMISSION_GUIDE.md` | 제출 절차 가이드 |
| `docs/reference_materials/` | FSDS 설치·SLAM·V2X 강의 자료 |
| `submission/` | 대회 제출용 패키지 (Docker 이미지 소스) |
| `submission/Dockerfile` | 제출용 도커 이미지 |
| `submission/docker-compose.yml` | 제출용 컴포즈 |
| `submission/run.sh` | 제출 이미지 부트 명령 |
| `submission/dev.sh` | 제출 패키지 개발용 셸 |
| `submission/launch/competition.launch` | 대회 노드 그래프 |
| `submission/src/drivers/` | driver 구현 (basic/advanced/autonomous/competition) |
| `submission/src/perception/`, `control/`, `v2x/`, `utils/` | 모듈 미러 |

---

## 6. Quick Start / 빠른 시작

> 모든 명령은 저장소 루트 기준이며, ROS 1(Noetic) 환경 또는 Docker가 설치되어 있어야 합니다.

### 옵션 A — Docker로 시뮬레이터 실행 (권장)

```bash
# 1) 저장소 클론
git clone <repository-url> fsd-stack && cd fsd-stack

# 2) FSDS 시뮬레이터 설정 확인
cat src/simulator/settings.json

# 3) 개발 작업 공간을 도커로 기동
cd src/autonomous
docker compose up -d

# 4) 컨테이너 내부에서 레이스 시작
docker compose exec autonomous bash
roslaunch config/bridge_no_camera.launch
python3 scripts/start_race.py
```

### 옵션 B — 로컬 ROS 1 환경에서 직접 실행

```bash
# 1) 의존성 설치 (Noetic 기준)
sudo apt install ros-noetic-desktop-full python3-rosdep
sudo rosdep init && rosdep update

# 2) 작업 공간 빌드
cd src/autonomous
source /opt/ros/noetic/setup.bash
catkin_make   # 또는 catkin build

# 3) 런치
source devel/setup.bash
roslaunch config/bridge_no_camera.launch
```

### 제출 이미지로 한 번에 실행

```bash
cd submission
./run.sh
```

---

## 7. Configuration / 설정

주요 설정은 YAML과 런치 파일로 분리되어 있습니다. `src/autonomous/config/params.yaml`을 먼저 확인하세요.

| 항목 / Key | 기본 위치 / Default location | 설명 / Description |
| --- | --- | --- |
| 콘 감지 임계값 / Cone detection thresholds | `src/autonomous/config/params.yaml` | 거리·신뢰도 컷오프 |
| SLAM 파라미터 / SLAM params | `src/autonomous/modules/perception/slam.py` | 매핑 해상도, 루프 클로저 옵션 |
| Pure Pursuit look-ahead | `src/autonomous/modules/control/pure_pursuit.py` | 전방 주시 거리 (곡률에 따라 동적) |
| Speed control limits | `src/autonomous/modules/control/speed.py` | 최대 속도, 가감속 한계 |
| V2X RSU endpoint | `submission/src/v2x/rsu.py` | RSU 호스트·포트·프로토콜 |
| Watchdog timeout | `src/autonomous/modules/utils/watchdog.py` | 노드 헬스체크 주기·타임아웃 |
| Sim 설정 / Sim settings | `src/simulator/settings.json` | FSDS 트랙·차량 정의 |

> **Tip:** 시뮬레이터와 실차에서 같은 파라미터 파일을 사용하면 환경 차이로 인한 재현성 문제가 줄어듭니다.

---

## 8. Commands Reference / 명령어 레퍼런스

| 명령어 / Command | 위치 / Path | 용도 / Purpose |
| --- | --- | --- |
| `roslaunch config/bridge_no_camera.launch` | `src/autonomous/config/` | 카메라 미사용 노드 그래프 부트스트랩 |
| `python3 driver/competition_driver.py` | `src/autonomous/driver/` | 대회 진입 드라이버 단독 실행 |
| `python3 scripts/start_race.py` | `src/autonomous/scripts/` | 시뮬레이터 레이스 시작 |
| `bash start.sh` | `src/autonomous/` | 개발 컨테이너 내부 부트 |
| `bash run_all.sh` | `src/autonomous/` | 전체 노드 일괄 실행 |
| `bash entrypoint.sh` | `src/autonomous/` | 도커 엔트리포인트 |
| `bash record_race.sh` | `src/autonomous/` | 레이스 rosbag 기록 |
| `./run.sh` | `submission/` | 제출 이미지 부트 (심사위원 데모용) |
| `./dev.sh` | `submission/` | 제출 패키지 개발 셸 |
| `bash scripts/package.sh` | `scripts/` | 제출용 Docker 이미지 빌드 |
| `python3 -m pytest tests/` | `src/autonomous/tests/` | 알고리즘 단위 테스트 |

---

## 9. Local Development / 로컬 개발

- 코드 수정은 `src/autonomous/`에서 진행하고, 동일 변경 사항을 `submission/`에 미러링합니다. 두 작업 공간은 같은 모듈 인터페이스(`perception`, `control`, `v2x`, `utils`)를 유지해야 합니다.
- 신규 노드를 추가할 때는 `driver/competition_driver.py`가 발행하는 토픽과 호환되는지 확인하세요.
- 런치 파일 변경 시 카메라 있음/없음 두 변형(`bridge_no_camera.launch` 외)도 함께 점검합니다.
- `in-memoria.db`는 디버깅용 스냅샷이며 커밋 대상에서 제외하는 것을 권장합니다.

---

## 10. Testing / 테스트

- 단위 테스트는 `src/autonomous/tests/test_algorithms.py`에서 실행합니다.

  ```bash
  cd src/autonomous
  python3 -m pytest tests/ -v
  ```

- 통합 테스트는 FSDS 시뮬레이터를 띄운 뒤 `scripts/start_race.py`로 레이스를 시작해 콘 감지→Pure Pursuit→차량 명령 흐름이 끊김 없이 동작하는지 확인합니다.
- 안전 기능(`watchdog.py`)은 노드를 강제 종료한 뒤 정지 명령이 발행되는지 별도 확인합니다.

---

## 11. Simulation / 시뮬레이션

`src/simulator/`는 FSDS(Formula Student Driverless Simulator) 설정 디렉터리입니다. 자세한 내용은 `src/simulator/README.md`를 참조하세요.

- 트랙·차량 정의: `src/simulator/settings.json`
- 강의 자료: `docs/reference_materials/lecture1_fsds_install.txt`

시뮬레이터 기동 → 본 스택 런치 → 레이스 스크립트 실행 순서로 진행합니다.

---

## 12. Submission Package / 제출 패키지

`submission/`은 FSG(Formula Student Germany) 제출 규격에 맞춘 패키지입니다. 자세한 절차는 `docs/SUBMISSION_GUIDE.md`를 참조하세요.

| 파일 / File | 역할 / Role |
| --- | --- |
| `submission/Dockerfile` | 제출용 Docker 이미지 빌드 정의 |
| `submission/docker-compose.yml` | 제출용 컴포즈 |
| `submission/run.sh` | 심사위원 환경에서 단일 명령 부트 |
| `submission/dev.sh` | 개발·디버깅 셸 |
| `submission/launch/competition.launch` | 대회 노드 그래프 |
| `submission/src/` | 모듈 미러 (drivers / perception / control / v2x / utils) |

빌드:

```bash
bash scripts/package.sh
```

---

## 13. Reference Materials / 참고 자료

| 자료 / Material | 경로 / Path |
| --- | --- |
| FSDS 설치 강의 / FSDS install lecture | `docs/reference_materials/lecture1_fsds_install.txt` |
| SLAM 강의 / SLAM lecture | `docs/reference_materials/lecture4_slam.ipynb` |
| V2X 강의 / V2X lecture | `docs/reference_materials/lecture6_v2x.ipynb` |
| 제출 가이드 / Submission guide | `docs/SUBMISSION_GUIDE.md` |

---

## 14. Contributing / 기여

기여 절차는 `CONTRIBUTING.md`를 따릅니다. 코드 리뷰 책임자는 `OWNERS` 파일을 확인하세요. PR 전 다음을 권장합니다.

- `python3 -m pytest src/autonomous/tests/` 통과
- `src/autonomous/`와 `submission/` 모듈 인터페이스 동기화
- 런치 파일과 `params.yaml` 변경 시 동료 검토

---

## 15. License / 라이선스

`LICENSE` 파일을 참조하세요. 대회 제출·재배포 시 라이선스 조건을 확인하시기 바랍니다.
See the `LICENSE` file for the full text. Verify license terms before redistributing the submission image.

---

### Maintainers / Points of Contact

- 저장소 책임자 / Repository owners: `OWNERS`
- 대회/제출 문의 / Competition & submission questions: 팀 내부 Slack 또는 `OWNERS`에 명시된 연락처
- 이슈 / Issue tracking: 저장소 내 Issues 탭 사용
- Further documentation: `docs/SUBMISSION_GUIDE.md`, `src/simulator/README.md`, `AGENTS.md`