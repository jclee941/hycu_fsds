# Autonomous Racing Stack (FSDriverless)

- License: see [LICENSE](LICENSE)
- Maintainers: [OWNERS](OWNERS)
- Contributing: [CONTRIBUTING.md](CONTRIBUTING.md)
- Submission guide: [docs/SUBMISSION_GUIDE.md](docs/SUBMISSION_GUIDE.md)
- Simulator settings: [src/simulator/settings.json](src/simulator/settings.json)

## 한 줄 요약

Formula Student Driverless(FSD) 규격 경주를 위한 자율주행 스택입니다.
콘 감지·분류, SLAM, pure pursuit 추종, 속도 제어, V2X(RSU) 통신을 ROS 노드로 묶고, Docker로 패키징해 FSDS 시뮬레이터와 실 차량에 그대로 배포할 수 있게 구성했습니다.

## One-line summary

Autonomous racing stack for Formula Student Driverless (FSD)–style events.
It packages cone detection, classification, SLAM, pure-pursuit control, speed control, and V2X (RSU) communication as ROS nodes, and ships them as reproducible Docker images for the FSDS simulator and real cars.

## 빠른 상태 / Status at a glance

| 항목 / Item | 값 / Value |
| --- | --- |
| 제품 성격 / Product | 자율주행 레이스 스택 (FSD 규격) / Autonomous racing stack |
| 대상 환경 / Target | ROS + Docker (FSDS 시뮬레이터, F1TENTH 스케일 차량) |
| 메인 진입점 / Entry point | `submission/run.sh` (제출용), `src/autonomous/start.sh` (개발용) |
| 시뮬레이터 설정 / Simulator | [src/simulator/settings.json](src/simulator/settings.json) |
| 테스트 / Tests | [src/autonomous/tests/test_algorithms.py](src/autonomous/tests/test_algorithms.py) |
| 운영 가이드 / Runbook | [docs/SUBMISSION_GUIDE.md](docs/SUBMISSION_GUIDE.md) |
| 유지보수 / Maintainers | [OWNERS](OWNERS) |
| 라이선스 / License | [LICENSE](LICENSE) |

## 흐름 한 눈에 / Flow at a glance

1. FSDS 시뮬레이터 또는 실 차량이 라이다(`/scan`), 카메라(`/camera`), 휠 오도메트리(`/odom`)를 발행합니다.
2. `perception/cone_detector`가 콘을 클러스터링하고, `cone_classifier`가 노랑/파랑 색을 입힙니다.
3. `perception/slam`이 차량 포즈와 맵을 갱신해 `/slam_out_pose`를 노출합니다.
4. `control/pure_pursuit`이 콘 중심선을 따라 경로를 만들고, `control/speed`가 구간별 목표 속도를 정합니다.
5. `v2x/rsu`가 roadside unit 메시지를 받아 코스 폐쇄·플래그 같은 외부 신호를 반영합니다.
6. `driver/competition_driver`가 위 결과를 종합해 `/cmd_vel`로 차량에 명령을 전달합니다.
7. `utils/watchdog`이 응답을 감시하고 이상 시 비상 정지, `utils/lap_timer`가 랩 타임과 통과 순서를 기록합니다.

## 목차 / Contents

- [제품 목적 / Purpose](#제품-목적--purpose)
- [패키지 구성 / Package contents](#패키지-구성--package-contents)
- [운영 상태 / Status](#운영-상태--status)
- [먼저 읽을 파일 / First files to read](#먼저-읽을-파일--first-files-to-read)
- [API와 진입점 / API and entry points](#api와-진입점--api-and-entry-points)
- [빠른 시작 / Quickstart](#빠른-시작--quickstart)
- [설정 / Configuration](#설정--configuration)
- [명령어 / Commands reference](#명령어--commands-reference)
- [아키텍처 / Architecture](#아키텍처--architecture)
- [로컬 개발 / Local development](#로컬-개발--local-development)
- [테스트 / Testing](#테스트--testing)
- [기여 / Contributing](#기여--contributing)
- [유지보수 / Maintainers](#유지보수--maintainers)
- [추가 문서 / Further documentation](#추가-문서--further-documentation)
- [라이선스 / License](#라이선스--license)

## 제품 목적 / Purpose

이 저장소는 FSD(Formula Student Driverless) 규격의 자율주행 경주 스택을 한 곳에 묶어 둔 작업본입니다.
콘이 놓인 미지의 트랙을 라이다와 카메라로 인식하고, SLAM으로 위치를 추정하며, pure pursuit과 속도 제어로 차량을 주행시킵니다.
V2X(RSU) 모듈을 포함해 코스 폐쇄 신호 같은 외부 이벤트에 반응할 수 있고, 동일한 코드를 시뮬레이터와 실 차량에서 그대로 실행할 수 있도록 Docker로 패키징되어 있습니다.

## 패키지 구성 / Package contents

저장소는 **개발용 스택**과 **대회 제출용 패키지** 두 갈래로 구성되어 있습니다.

- `src/autonomous/` — 개발용 자율주행 스택 소스.
  - `driver/competition_driver.py` — 메인 루프. 인식·제어·V2X 결과를 모아 차량 명령으로 보냅니다.
  - `modules/perception/` — `cone_detector.py`, `cone_classifier.py`, `slam.py`.
  - `modules/control/` — `pure_pursuit.py`, `speed.py`.
  - `modules/utils/` — `lap_timer.py`, `watchdog.py`.
  - `config/bridge_no_camera.launch`, `config/params.yaml` — ROS launch와 파라미터.
  - `Dockerfile`, `docker-compose.yml`, `entrypoint.sh`, `start.sh`, `run_all.sh`, `record_race.sh` — 컨테이너 자산.
  - `scripts/start_race.py` — 대회 레이스 트리거(독립 실행 가능).
  - `tests/test_algorithms.py` — 알고리즘 단위 테스트.
- `src/simulator/` — FSDS 시뮬레이터 설정(`settings.json`)과 보조 README.
- `submission/` — 대회 제출용 패키지. 노드 구조가 `src/` 아래로 평탄화되어 있고, `run.sh` / `dev.sh`로 실행합니다.
  - `src/drivers/` — `basic.py`, `autonomous.py`, `advanced.py`, `competition.py` 등 시나리오별 드라이버.
  - `src/perception/`, `src/control/`, `src/utils/`, `src/v2x/` — 제출용 모듈 사본. V2X에는 `rsu.py`가 포함됩니다.
  - `launch/competition.launch` — 제출 시 단일 진입 launch.
  - `autonomous/` — `src/autonomous/`와 동일한 구조의 사본. 제출 환경에서 개발·튜닝을 이어가기 위한 자리입니다.
- `docs/SUBMISSION_GUIDE.md` — 대회 제출 절차와 운영 가이드.
- `docs/reference_materials/` — FSDS 설치, SLAM, V2X 강의 노트(`.txt`, `.ipynb`).
- `scripts/package.sh` — 제출 산출물 패키징 스크립트.
- `in-memoria.db` — 런타임 상태(예: 맵 캐시, V2X 수신 로그)를 보관하는 SQLite 파일.
- `AGENTS.md`, `CONTRIBUTING.md`, `OWNERS`, `LICENSE` — 거버넌스 문서.

## 운영 상태 / Status

- 스택의 성격은 **연구·경쟁용**입니다. 대회 시즌에 맞춰 활발히 변경되며, 안정성 보장은 시뮬레이터 통과 수준입니다.
- 공개 API와 토픽 계약은 대회 규격 변경에 따라 바뀔 수 있습니다. 운영 버전은 `docs/SUBMISSION_GUIDE.md`에 명시된 릴리스 태그를 기준으로 고정해 주세요.
- 시뮬레이터 결과와 실 차량 결과는 라이다 모델, 조명, 트랙 표면에 따라 차이가 큽니다. 튜닝 전에는 반드시 시뮬레이터에서 회귀 테스트를 먼저 돌려 주세요.
- 프로덕션 배포용이 아니므로, 외부 사용 시 안전 장치(워치독·비상 정지·원격 킬 스위치) 동작을 별도 검증해야 합니다.

## 먼저 읽을 파일 / First files to read

순서대로 보면 스택 전체를 빠르게 잡을 수 있습니다.

1. [docs/SUBMISSION_GUIDE.md](docs/SUBMISSION_GUIDE.md) — 대회 운영 절차, 평가 항목, 제출 흐름.
2. [src/autonomous/driver/competition_driver.py](src/autonomous/driver/competition_driver.py) — 노드들이 어떻게 묶여 동작하는지 보여 주는 메인 루프.
3. [src/autonomous/config/params.yaml](src/autonomous/config/params.yaml) — 감지 임계값, 속도 제한, 안전 한계 같은 핵심 튜닝 파라미터.
4. [src/autonomous/modules/perception/cone_detector.py](src/autonomous/modules/perception/cone_detector.py) — 라이다 기반 콘 감지 파이프라인.
5. [src/autonomous/modules/control/pure_pursuit.py](src/autonomous/modules/control/pure_pursuit.py) — 경로 추종 알고리즘.
6. [src/simulator/README.md](src/simulator/README.md) — FSDS에서 시뮬레이션을 돌리는 절차.

## API와 진입점 / API and entry points

**ROS 노드(공통 계약)**

| 노드 / Node | 입력 / Input | 출력 / Output | 책임 / Responsibility |
| --- | --- | --- | --- |
| `perception/cone_detector` | `/scan`, `/camera` | `/perception/cones`, `/perception/cones_visual` | 콘 클러스터링과 위치 추정 |
| `perception/cone_classifier` | `/camera` | `/perception/cone_colors` | 노랑/파랑 색상 분류 |
| `perception/slam` | `/scan`, `/odom` | `/slam_out_pose`, `/slam_map` | 차량 포즈와 맵 갱신 |
| `control/pure_pursuit` | `/slam_out_pose`, `/perception/cones` | `/control/path` | 중심선 경로 생성 |
| `control/speed` | `/control/path`, `/slam_out_pose` | `/control/target_speed` | 구간별 목표 속도 산출 |
| `v2x/rsu` | `v2x/in`, `/slam_out_pose` | `/v2x/event`, `/v2x/course_status` | RSU 이벤트 반영 |
| `utils/lap_timer` | `/slam_out_pose`, `/perception/cones` | `/race/lap_time`, `/race/status` | 랩 기록·통과 순서 |
| `utils/watchdog` | `/cmd_vel`, `/slam_out_pose` | `/safety/stop`, `/safety/heartbeat` | 워치독·비상 정지 |
| `driver/competition_driver` | 위 모든 토픽 | `/cmd_vel` | 명령 통합 및 비상 정지 |

**컨테이너 진입점**

- 개발: `src/autonomous/start.sh`, `src/autonomous/run_all.sh`.
- 제출: `submission/run.sh`. ROS launch는 `submission/launch/competition.launch` 한 곳에서 시작됩니다.
- 시뮬 + 드라이버 동시 실행: `src/autonomous/record_race.sh`.

## 빠른 시작 / Quickstart

개발 머신에서 시뮬레이터와 드라이버를 함께 실행하는 가장 짧은 경로입니다.

1. 저장소를 받고, 호스트에 ROS + Docker 환경을 준비합니다.
2. 트랙 설정이 필요하면 [src/simulator/settings.json](src/simulator/settings.json)을 확인합니다.
3. 개발 컨테이너를 띄웁니다.
   - `cd src/autonomous && docker compose up --build`
   - 또는 호스트에서 ROS를 직접 실행할 경우 `./start.sh`로 launch를 올립니다.
4. FSDS 시뮬레이터를 별도 프로세스로 실행합니다.
   설치 절차는 [docs/reference_materials/lecture1_fsds_install.txt](docs/reference_materials/lecture1_fsds_install.txt)를 참고하세요.
5. 시뮬레이션과 드라이버가 연결되면 `/race/status` 토픽으로 랩 진행 상황이 보입니다.

실제 대회 제출 절차(컨테이너 빌드, 산출물 압축, 업로드)는 [docs/SUBMISSION_GUIDE.md](docs/SUBMISSION_GUIDE.md)를 그대로 따라 주세요.

## 설정 / Configuration

주요 설정은 [src/autonomous/config/params.yaml](src/autonomous/config/params.yaml)에 모여 있습니다.

- `perception` — 라이다 클러스터링 거리, 콘 최소/최대 크기, 분류 신뢰도 임계값.
- `slam` — 맵 해상도, 업데이트 빈도, 루프 클로저 사용 여부.
- `control` — pure pursuit lookahead 거리, 최대 조향각, 속도별 가감속 게인.
- `safety` — 워치독 타임아웃, 비상 정지 거리, 통신 두절 시 동작.
- `v2x` — RSU 수신 포트, 메시지 만료 시간, 코스 폐쇄 시 감속 곡선.

제출용 패키지에서도 동일한 키를 `submission/autonomous/config/params.yaml`에서 사용합니다.
한쪽을 고치면 다른 쪽도 함께 맞춰 주세요.

환경 의존 값(IP 주소, RSU 호스트, 컨테이너 번호 등)은 코드에 하드코딩하지 말고 파라미터 파일이나 환경 변수로 주입하는 것을 원칙으로 합니다.

## 명령어 / Commands reference

| 명령 / Command | 용도 / Purpose |
| --- | --- |
| `src/autonomous/start.sh` | 개발용 ROS launch 실행 |
| `src/autonomous/run_all.sh` | 인식·제어·V2X 노드 일괄 실행 |
| `src/autonomous/record_race.sh` | 시뮬 + 드라이버 동시 실행, 주행 기록 |
| `src/autonomous/scripts/start_race.py` | 대회 레이스 트리거(독립 실행) |
| `submission/run.sh` | 제출용 단일 진입점(컨테이너 내부) |
| `submission/dev.sh` | 제출 컨테이너를 개발 모드로 진입 |
| `scripts/package.sh` | 제출 산출물 압축 |
| `docker compose up --build` (`src/autonomous`) | 개발 스택 컨테이너 빌드 및 실행 |
| `docker compose up --build` (`submission`) | 제출 스택 컨테이너 빌드 및 실행 |
| `pytest src/autonomous/tests` | 알고리즘 단위 테스트 실행 |

## 아키텍처 / Architecture

스택은 ROS 노드 기반의 데이터 흐름 파이프라인입니다.
토픽은 환경 → 인식 → 위치/매핑 → 경로/속도 → 외부 신호 → 통합 → 안전 → 차량 순으로 흐릅니다.

| 단계 / Stage | 노드 / Node | 책임 / Responsibility |
| --- | --- | --- |
| 1. 환경 / Environment | FSDS / 차량 | 라이다·카메라·휠 오도메트리 발행 |
| 2. 인식 / Perception | `perception/cone_detector` | 콘 클러스터링과 위치 추정 |
| 3. 인식 / Perception | `perception/cone_classifier` | 노랑/파랑 색상 분류 |
| 4. 위치 / Localization | `perception/slam` | 차량 포즈와 맵 갱신 |
| 5. 경로 / Planning | `control/pure_pursuit` | 콘 중심선 경로 생성 |
| 6. 속도 / Speed | `control/speed` | 구간별 목표 속도 산출 |
| 7. 외부 신호 / External | `v2x/rsu` | RSU 이벤트 반영 |
| 8. 통합 / Integration | `driver/competition_driver` | 명령 통합 및 비상 정지 |
| 9. 안전 / Safety | `utils/watchdog` | 워치독·비상 정지 |
| 10. 계측 / Telemetry | `utils/lap_timer` | 랩 기록·상태 노출 |

**1랩 요청 흐름**

1. 차량이 `/scan`을 발행합니다.
2. `cone_detector`가 콘 목록을 만들고, `cone_classifier`가 색을 입힙니다.
3. `slam`이 포즈를 갱신하고, `pure_pursuit`이 중심선 경로를 만듭니다.
4. `speed`가 곡률에 따라 목표 속도를 정합니다.
5. `competition_driver`가 `/cmd_vel`로 차량에 명령을 내보내고, `watchdog`이 응답을 감시합니다.
6. `lap_timer`가 콘 통과 순서를 확인해 랩 완주 여부를 기록합니다.

## 로컬 개발 / Local development

- 시뮬레이터와 드라이버는 별도 프로세스로 띄우는 편이 디버깅에 유리합니다. 동시 실행이 꼭 필요할 때만 `record_race.sh`를 사용하세요.
- 파라미터를 바꿀 때는 `src/autonomous/config/params.yaml`과 `submission/autonomous/config/params.yaml`을 함께 갱신해 두세요. 둘은 대회 시즌 동안 동일하게 유지해야 합니다.
- `docs/reference_materials/`의 노트북(`.ipynb`)은 SLAM과 V2X 모듈의 설계 배경을 설명합니다. 새 알고리즘을 추가할 때 같은 형식으로 짧은 노트를 함께 두면 리뷰가 빨라집니다.
- 런타임 상태(맵 캐시, V2X 로그 등)는 `in-memoria.db`에 저장됩니다. 로컬 개발에서 초기화하려면 실행을 멈추고 파일을 삭제하면 됩니다.
- IP 주소, 컨테이너 번호, 원격 RSU 호스트 같은 환경 의존 값은 코드에 하드코딩하지 말고 파라미터 또는 환경 변수로 주입해 주세요.

## 테스트 / Testing

- **단위 테스트**: `pytest src/autonomous/tests` — pure pursuit, 속도 프로파일, 워치독 로직 등을 검증합니다.
- **시뮬레이션 회귀**: FSDS에서 알려진 트랙을 한 바퀴 돌리고 `/race/status`의 랩 타임과 콘 통과 순서가 기대치와 일치하는지 확인합니다.
- **안전 점검**: `utils/watchdog`을 별도로 강제 트립시키는 테스트 케이스를 두는 것을 권장합니다. 노트북이 끊겼을 때 `/cmd_vel`이 0으로 떨어지는지 확인합니다.
- **V2X 점검**: RSU 이벤트(코스 폐쇄, 플래그)를 시뮬레이터에서 수동으로 발생시켜 `/v2x/event`가 전파되는지 확인합니다.

## 기여 / Contributing

- 작업 시작 전 [CONTRIBUTING.md](CONTRIBUTING.md)와 [OWNERS](OWNERS)를 확인해 코드 오너십과 리뷰 경로를 잡아 주세요.
- 알고리즘 변경 시에는 동일 트랙에서의 시뮬레이션 회귀 결과를 PR 설명에 첨부해 주세요.
- 대회 시즌에는 `params.yaml` 변경이 곧 점수에 영향을 주므로, 사소한 튜닝도 PR로 분리해 추적 가능하게 유지합니다.
- 새 모듈을 추가할 때 `docs/reference_materials/`에 짧은 노트(설계 의도, 가정, 한계)를 함께 두는 것을 권장합니다.

## 유지보수 / Maintainers

- 저장소 오너와 도메인 리뷰어는 [OWNERS](OWNERS) 파일에 정리되어 있습니다.
- 거버넌스 규칙과 외부 기여 절차는 [CONTRIBUTING.md](CONTRIBUTING.md)를 참고해 주세요.
- 운영 중 이슈는 저장소 이슈 트래커를 사용하고, 보안 관련 보고는 메인테이너에게 직접 연락합니다.

## 추가 문서 / Further documentation

- [docs/SUBMISSION_GUIDE.md](docs/SUBMISSION_GUIDE.md) — 대회 제출 절차.
- [docs/reference_materials/lecture1_fsds_install.txt](docs/reference_materials/lecture1_fsds_install.txt) — FSDS 설치.
- [docs/reference_materials/lecture4_slam.ipynb](docs/reference_materials/lecture4_slam.ipynb) — SLAM 모듈 설계 노트.
- [docs/reference_materials/lecture6_v2x.ipynb](docs/reference_materials/lecture6_v2x.ipynb) — V2X/RSU 설계 노트.
- [src/simulator/README.md](src/simulator/README.md) — 시뮬레이터 실행 메모.

## 라이선스 / License

[LICENSE](LICENSE) 파일을 참고해 주세요. 대회 제출물에 라이선스 헤더를 함께 싣는 것은 각 시즌 운영팀의 요구 사항을 따릅니다.