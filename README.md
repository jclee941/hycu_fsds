# Formula Student Driverless — Autonomous Racing Stack

> **FSD (Formula Student Driverless) 자율주행 레이싱 소프트웨어 스택**
> Autonomous racing software stack for the Formula Student Driverless competition

ROS 1 기반의 자

율주행 레이싱 소프트웨어 스택입니다. 콘 감지·분류, SLAM, Pure Pursuit 추종 제어, 랩 타이머, V2X(차량-사물 통신) 어댑터, 대회 규정 준수 Docker 제출 패키지를 한 번에 제공합니다.

This is a ROS 1-based autonomous racing software stack. It bundles cone detection/classification, SLAM, pure-pursuit lane following, lap timing, a V2X (RSU) adapter, and a competition-rule-compliant Docker submission package.

---

## Status / 운영 상태

| 항목 / Item | 상태 / Status | 비고 / Notes |
|---|---|---|
| Production readiness / 프로덕션 준비 | Competition-ready | FSD 규정 준수 Docker 제출 이미지 |
| ROS distribution / ROS 배포판 | ROS 1 (Noetic 호환) | `bridge_no_camera.launch` 사용 |
| Language / 언어 | Python 3 | `src/autonomous/modules/`, `submission/src/` |
| Container runtime / 컨테이너 런타임 | Docker, docker-compose | `Dockerfile` × 2, `docker-compose.yml` × 2 |
| Simulation / 시뮬레이터 연동 | 지원 | `src/simulator/` (settings.json) |
| V2X / RSU 어댑터 | 지원 | `submission/src/v2x/rsu.py` |
| Testing / 테스트 스크립트 | 부분 제공 | `src/autonomous/tests/test_algorithms.py` |
| Deprecated modules / 폐기 예정 모듈 | 없음 | — |

## Quick-glance Flow / 빠른 흐름 요약

운영자가 한 줄 요약으로 이해할 수 있는 흐름입니다.

| 단계 / Stage | 경로 / Path | 목적 / Purpose |
|---|---|---|
| 1. Build / 빌드 | `submission/Dockerfile` | 대회 규정에 맞는 단일 이미지 빌드 |
| 2. Launch / 실행 | `submission/launch/competition.launch` | 대회용 ROS 노드 그래프 기동 |
| 3. Drive / 주행 | `submission/src/drivers/competition.py` | Perception → Control 토픽 연결 및 차량 제어 |
| 4. Simulate / 시뮬레이션 | `src/simulator/settings.json` + `src/autonomous/scripts/start_race.py` | 시뮬레이터에서 통합 시험 |
| 5. Package / 패키징 | `scripts/package.sh` | 제출용 아티팩트 생성 |

핵심 토픽 그래프 (Compact request flow):

1. **Perception** — `cone_detector.py`, `cone_classifier.py`가 카메라/LiDAR 입력으로 콘 위치·색상 발행
2. **Localization** — `slam.py`가 SLAM으로 차량 위치 추정
3. **Planning/Control** — `pure_pursuit.py`, `speed.py`가 목표 경로 추종 및 속도 명령 생성
4. **V2X** — `v2x/rsu.py`가 RSU 메시지 수신·송신
5. **Drivers** — `competition.py` / `competition_driver.py`가 위 노드를 묶어 `/cmd_vel` 등 차량 명령 발행
6. **Utils** — `lap_timer.py`, `watchdog.py`가 랩 측정 및 안전 정지 보장

---

## Package Contents / 패키지 구성

이 저장소가 실제로 제공하는 산출물입니다.

| 구분 / Group | 경로 / Path | 역할 / Role |
|---|---|---|
| 자율주행 노드 / Autonomous nodes | `src/autonomous/modules/` | perception, control, utils 모듈 (Python) |
| 자율주행 진입점 / Driver entry | `src/autonomous/driver/competition_driver.py` | 로컬/시뮬레이션용 메인 드라이버 |
| 시뮬레이터 통합 / Simulator | `src/autonomous/scripts/start_race.py` | 시뮬레이션 기반 레이스 실행 |
| ROS 설정 / ROS configuration | `src/autonomous/config/bridge_no_camera.launch`, `params.yaml` | ROS launch 및 파라미터 |
| 시뮬레이터 설정 / Simulator settings | `src/simulator/settings.json` | 시뮬레이터 구성 값 |
| 대회 제출 패키지 / Competition submission | `submission/` | 규격화된 Docker 제출 트리 |
| 대회 드라이버 / Competition drivers | `submission/src/drivers/` | `basic.py`, `advanced.py`, `autonomous.py`, `competition.py` |
| V2X 어댑터 / V2X | `submission/src/v2x/rsu.py` | RSU 통신 어댑터 |
| 스크립트 / Scripts | `scripts/package.sh` | 패키징 유틸리티 |
| 문서 / Docs | `docs/SUBMISSION_GUIDE.md`, `docs/reference_materials/` | 제출 가이드, 강의 자료 |
| 거버넌스 / Governance | `LICENSE`, `OWNERS`, `CONTRIBUTING.md` | 라이선스, 책임자, 기여 절차 |

## First Files to Read / 먼저 읽을 파일

운영자가 우선적으로 살펴봐야 할 파일 목록입니다.

| 우선 / Priority | 파일 / File | 이유 / Why |
|---|---|---|
| 1 | [`docs/SUBMISSION_GUIDE.md`](docs/SUBMISSION_GUIDE.md) | 대회 규정, 제출 절차 |
| 2 | [`submission/README.md`](submission/README.md) | 제출 패키지 내부 설명 |
| 3 | [`src/autonomous/AGENTS.md`](src/autonomous/AGENTS.md) | 자율주행 하위 시스템 가이드 |
| 4 | [`submission/AGENTS.md`](submission/AGENTS.md) | 제출 패키지 작업 지침 |
| 5 | [`submission/launch/competition.launch`](submission/launch/competition.launch) | 기동되는 ROS 노드 목록 |
| 6 | [`submission/src/drivers/competition.py`](submission/src/drivers/competition.py) | 실제 주행 드라이버 진입점 |

---

## Features / 주요 기능

| 기능 / Feature | 모듈 / Module | 설명 / Description |
|---|---|---|
| 콘 감지 / Cone detection | `modules/perception/cone_detector.py` | LiDAR/카메라 입력에서 콘 후보 추출 |
| 콘 분류 / Cone classification | `modules/perception/cone_classifier.py` | 노란/파란 콘 색상 분류 |
| SLAM / Localization | `modules/perception/slam.py` | SLAM 기반 차량 위치 추정 |
| 경로 추종 / Lane following | `modules/control/pure_pursuit.py` | Pure Pursuit 알고리즘 |
| 속도 제어 / Speed control | `modules/control/speed.py` | 곡률 기반 속도 명령 생성 |
| 랩 타이밍 / Lap timing | `modules/utils/lap_timer.py` | 시작선 통과 시 랩 기록 |
| 안전 감시 / Watchdog | `modules/utils/watchdog.py` | 타임아웃 시 차량 정지 |
| V2X / RSU | `v2x/rsu.py` | Road-Side Unit 양방향 통신 |
| 드라이버 진입점 / Driver entry | `drivers/competition.py` 등 | 대회/시뮬레이션 진입점 |

---

## Architecture / 아키텍처

| 계층 / Layer | 컴포넌트 / Component | 책임 / Responsibility |
|---|---|---|
| Drivers / 드라이버 | `drivers/basic.py`, `drivers/advanced.py`, `drivers/autonomous.py`, `drivers/competition.py` | ROS 토픽 구독 및 통합 |
| Perception / 인식 | `perception/cone_detector.py`, `perception/cone_classifier.py`, `perception/slam.py` | 센서 → 트랙/위치 토픽 |
| Control / 제어 | `control/pure_pursuit.py`, `control/speed.py` | 경로 → 차량 명령 |
| V2X | `v2x/rsu.py` | RSU 메시지 송수신 |
| Utils / 유틸 | `utils/lap_timer.py`, `utils/watchdog.py` | 랩 측정, 안전 감시 |
| ROS Launch | `launch/competition.launch`, `config/bridge_no_camera.launch` | 노드 그래프 기동 |
| Container / 컨테이너 | `Dockerfile`, `docker-compose.yml`, `entrypoint.sh`, `run.sh` | 실행 환경 패키징 |

요청 흐름 (Compact numbered flow):

1. 카메라/LiDAR 토픽이 `cone_detector` → `cone_classifier`로 들어와 콘 배열 토픽 발행
2. `slam`이 LiDAR/IMU 기반으로 차량 위치 `/odom` 등 발행
3. `pure_pursuit` + `speed`가 콘 배열·위치·속도를 입력받아 `/cmd_vel` 발행
4. `rsu`가 RSU 정보를 받아 driver에 전달
5. 드라이버가 모든 토픽을 통합해 최종 명령 발행
6. `lap_timer`, `watchdog`가 랩 기록과 안전 정지 보장
7. `competition.launch`가 위 모든 노드를 단일 그래프로 기동

> 노드 간 통신 토픽의 정확한 이름은 `competition.launch` 및 `params.yaml`을 확인하세요.

---

## Repository Layout / 저장소 구조

실제 최상위 디렉터리만 반영합니다.

```text
.
├── AGENTS.md
├── CONTRIBUTING.md
├── LICENSE
├── OWNERS
├── README.md
├── in-memoria.db
├── src/
│   ├── autonomous/
│   │   ├── AGENTS.md
│   │   ├── Dockerfile
│   │   ├── docker-compose.yml
│   │   ├── entrypoint.sh
│   │   ├── record_race.sh
│   │   ├── run_all.sh
│   │   ├── start.sh
│   │   ├── scripts/start_race.py
│   │   ├── config/
│   │   │   ├── bridge_no_camera.launch
│   │   │   └── params.yaml
│   │   ├── driver/competition_driver.py
│   │   ├── modules/
│   │   │   ├── perception/{cone_classifier,cone_detector,slam}.py
│   │   │   ├── control/{pure_pursuit,speed}.py
│   │   │   └── utils/{lap_timer,watchdog}.py
│   │   └── tests/test_algorithms.py
│   └── simulator/
│       ├── README.md
│       └── settings.json
├── scripts/package.sh
├── docs/
│   ├── SUBMISSION_GUIDE.md
│   └── reference_materials/
│       ├── lecture1_fsds_install.txt
│       ├── lecture4_slam.ipynb
│       └── lecture6_v2x.ipynb
└── submission/
    ├── AGENTS.md
    ├── Dockerfile
    ├── README.md
    ├── dev.sh
    ├── docker-compose.yml
    ├── run.sh
    ├── launch/competition.launch
    └── src/
        ├── drivers/{basic,advanced,autonomous,competition}.py
        ├── perception/{cone_classifier,cone_detector,slam}.py
        ├── v2x/rsu.py
        ├── control/{pure_pursuit,speed}.py
        └── utils/{lap_timer,watchdog}.py
```

---

## Quick Start / 빠른 시작

### 1. 시뮬레이터 기반 실행 / Simulator-based run

시뮬레이터 설정은 [`src/simulator/settings.json`](src/simulator/settings.json)을 사용합니다.

```bash
# 자율주행 스택 진입
cd src/autonomous

# ROS 1 + 시뮬레이터 환경에서
./start.sh                # 컨테이너 시작
./run_all.sh              # 전체 노드 실행
# 또는 레이스 기록
./record_race.sh
```

### 2. 대회 제출 이미지 실행 / Run competition submission image

```bash
cd submission
./dev.sh                  # 개발 컨테이너 진입
# 또는
./run.sh                  # 대회 모드 실행
```

`docker-compose.yml`과 `entrypoint.sh`는 해당 디렉터리에 함께 제공됩니다.

### 3. 패키징 / Package

```bash
bash scripts/package.sh
```

자세한 절차는 [`docs/SUBMISSION_GUIDE.md`](docs/SUBMISSION_GUIDE.md)를 따르세요.

---

## Configuration / 설정

| 설정 항목 / Setting | 위치 / Location | 비고 / Notes |
|---|---|---|
| ROS launch (대회) | [`submission/launch/competition.launch`](submission/launch/competition.launch) | 대회 시 사용되는 단일 launch |
| ROS bridge (no camera) | [`src/autonomous/config/bridge_no_camera.launch`](src/autonomous/config/bridge_no_camera.launch) | 카메라 없는 브리지 시연 |
| ROS 파라미터 | [`src/autonomous/config/params.yaml`](src/autonomous/config/params.yaml) | 튜닝 가능한 노드 파라미터 |
| 시뮬레이터 | [`src/simulator/settings.json`](src/simulator/settings.json) | 시뮬레이터 구성 |

> 비밀번호, 토큰, 차량 CAN ID 등 민감한 값은 `params.yaml`에 직접 두지 말고 별도 환경 변수/시크릿으로 주입하세요.

---

## Commands Reference / 명령어 레퍼런스

| 명령어 / Command | 설명 / Description |
|---|---|
| `./start.sh` (in `src/autonomous/`) | 자율주행 컨테이너 시작 |
| `./run_all.sh` (in `src/autonomous/`) | 전체 노드 일괄 실행 |
| `./record_race.sh` (in `src/autonomous/`) | 레이스 데이터 기록 |
| `python scripts/start_race.py` | 시뮬레이터 기반 레이스 시작 |
| `./dev.sh` (in `submission/`) | 제출 패키지 개발 컨테이너 진입 |
| `./run.sh` (in `submission/`) | 대회 모드 실행 |
| `bash scripts/package.sh` | 제출 아티팩트 패키징 |
| `docker compose up` | `docker-compose.yml` 기반 기동 (각 디렉터리에서) |

---

## Local Development / 로컬 개발

| 작업 / Task | 절차 / Procedure |
|---|---|
| 가상환경 / venv | Python 3.x 가상환경 권장 (ROS 1 의존성 분리) |
| ROS 환경 / ROS env | 별도 devcontainer 또는 ROS Noetic 컨테이너 사용 |
| 모듈 변경 / Edit module | `src/autonomous/modules/` 또는 `submission/src/` 하위 파일 수정 후 `./run_all.sh` 재실행 |
| 새 드라이버 / Add driver | `submission/src/drivers/`에 `*.py` 추가 후 `competition.launch` 참조 갱신 |
| 파라미터 튜닝 / Tune params | `src/autonomous/config/params.yaml` 수정 |

## Testing / 테스트

| 테스트 종류 / Type | 위치 / Location | 실행 / Run |
|---|---|---|
| 알고리즘 단위 테스트 / Algorithm tests | [`src/autonomous/tests/test_algorithms.py`](src/autonomous/tests/test_algorithms.py) | `python -m pytest src/autonomous/tests/` |
| 시뮬레이션 통합 / Sim integration | `src/autonomous/scripts/start_race.py` | 시뮬레이터에서 노드 통합 검증 |
| 제출 패키지 검증 / Submission validation | `submission/run.sh`, `bash scripts/package.sh` | 대회 절차 회귀 검증 |

CI는 본 저장소에 기본 제공되지 않을 수 있으므로, 저장소 외부 CI를 연결하는 경우 `scripts/` 단계와 pytest를 함께 트리거하세요.

---

## Simulation / 시뮬레이션

[`src/simulator/`](src/simulator/)는 시뮬레이터용 설정(`settings.json`)과 간단한 `README.md`를 제공합니다. `start_race.py`와 결합하여 인식·제어 파이프라인을 시뮬레이터에서 검증합니다.

| 항목 / Item | 위치 / Location |
|---|---|
| 시뮬레이터 설정 | [`src/simulator/settings.json`](src/simulator/settings.json) |
| 레이스 진입 스크립트 | [`src/autonomous/scripts/start_race.py`](src/autonomous/scripts/start_race.py) |
| 시뮬레이터 보조 문서 | [`src/simulator/README.md`](src/simulator/README.md) |

---

## Submission Package / 제출 패키지

[`submission/`](submission/)은 대회 규격에 맞춘 단일 진입 패키지입니다.

| 구성 / Component | 파일 / File | 설명 / Description |
|---|---|---|
| Docker 빌드 / Docker build | [`submission/Dockerfile`](submission/Dockerfile) | 단일 이미지 빌드 |
| Compose / Compose | [`submission/docker-compose.yml`](submission/docker-compose.yml) | 컨테이너 오케스트레이션 |
| 실행 / Run | [`submission/run.sh`](submission/run.sh) | 대회 모드 실행 스크립트 |
| 개발 진입 / Dev shell | [`submission/dev.sh`](submission/dev.sh) | 개발용 셸 |
| Launch / ROS launch | [`submission/launch/competition.launch`](submission/launch/competition.launch) | 대회 노드 그래프 |
| 드라이버 / Drivers | [`submission/src/drivers/`](submission/src/drivers/) | `basic`, `advanced`, `autonomous`, `competition` |
| V2X | [`submission/src/v2x/rsu.py`](submission/src/v2x/rsu.py) | RSU 어댑터 |
| 작업 지침 / Agent notes | [`submission/AGENTS.md`](submission/AGENTS.md) | 제출 패키지 작업 규칙 |

## Reference Materials / 참고 자료

| 자료 / Material | 경로 / Path | 용도 / Use |
|---|---|---|
| FSDS 설치 강의 / FSDS install | [`docs/reference_materials/lecture1_fsds_install.txt`](docs/reference_materials/lecture1_fsds_install.txt) | Formula Student Driverless Simulator 설치 |
| SLAM 강의 / SLAM lecture | [`docs/reference_materials/lecture4_slam.ipynb`](docs/reference_materials/lecture4_slam.ipynb) | SLAM 개념/튜토리얼 |
| V2X 강의 / V2X lecture | [`docs/reference_materials/lecture6_v2x.ipynb`](docs/reference_materials/lecture6_v2x.ipynb) | V2X/RSU 개요 |
| 제출 가이드 / Submission guide | [`docs/SUBMISSION_GUIDE.md`](docs/SUBMISSION_GUIDE.md) | 대회 제출 절차 |

---

## Contributing / 기여

기여 절차는 [`CONTRIBUTING.md`](CONTRIBUTING.md)를 따릅니다. 책임자는 [`OWNERS`](OWNERS) 파일을 참조하세요.

| 항목 / Item | 참조 / Reference |
|---|---|
| 기여 가이드 | [`CONTRIBUTING.md`](CONTRIBUTING.md) |
| 작업 지침 (루트) | [`AGENTS.md`](AGENTS.md) |
| 자율주행 작업 지침 | [`src/autonomous/AGENTS.md`](src/autonomous/AGENTS.md) |
| 제출 패키지 작업 지침 | [`submission/AGENTS.md`](submission/AGENTS.md) |

## Maintainers / Points of Contact / 유지보수자 · 연락처

| 역할 / Role | 참조 / Reference |
|---|---|
| Owners / 책임자 명단 | [`OWNERS`](OWNERS) |
| 기여 정책 / Contribution policy | [`CONTRIBUTING.md`](CONTRIBUTING.md) |
| 작업 규칙 / Work rules | `AGENTS.md` (루트 및 하위 디렉터리) |

## License / 라이선스

본 저장소는 [`LICENSE`](LICENSE) 파일에 명시된 라이선스를 따릅니다.

---

## Further Documentation / 추가 문서

| 문서 / Doc | 경로 / Path |
|---|---|
| 제출 절차 | [`docs/SUBMISSION_GUIDE.md`](docs/SUBMISSION_GUIDE.md) |
| 자율주행 하위 시스템 노트 | [`src/autonomous/AGENTS.md`](src/autonomous/AGENTS.md) |
| 제출 패키지 내부 문서 | [`submission/README.md`](submission/README.md) |
| 시뮬레이터 메모 | [`src/simulator/README.md`](src/simulator/README.md) |

> **Tip**: 실제 차량에 적용하기 전에 시뮬레이터에서 `start_race.py` → `record_race.sh` 순으로 회귀 시험을 수행하고, `lap_timer` 및 `watchdog` 동작을 반드시 확인하세요.