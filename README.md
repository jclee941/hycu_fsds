# qws941 — Formula Student Driverless Autonomous Stack

> ROS 1 + FSDS 기반 자율주행 레이싱 스택 · 컨테이너 실행 가능한 대회 제출용 코드베이스

이 저장소는 Formula Student Driverless(FS-D) 대회를 위한 자율주행 소프트웨어 스택을 제공합니다. F1TENTH/FSDS 시뮬레이터에서 동작하는 인지(SLAM·콘 탐지·V2X), 경로 추종(pure pursuit), 속도 제어, 그리고 대회 제출용 Docker 패키지를 한 워크스페이스에 통합합니다.

---

## 한눈에 보기

| 항목 | 내용 |
| --- | --- |
| 대상 대회 | Formula Student Driverless (FS-D) |
| 시뮬레이터 | FSDS(F1TENTH/Full Scale DS) |
| 미들웨어 | ROS 1 (Noetic) |
| 핵심 언어 | Python 3 |
| 컨테이너 | Docker · docker compose |
| 실행 진입점 | `src/fsds_docker/run.sh`, `src/autonomous/start.sh` |
| 운영 진입점 | `src/fsds_docker/scripts/competition_driver.py`, `src/autonomous/driver/competition_driver.py` |
| 제출 산출물 | `scripts/package.sh` → Docker 이미지 + 번들 |

| 흐름 단계 | 모듈 | 역할 |
| --- | --- | --- |
| 1. 인지 (Perception) | `perception/cone_detector.py`, `perception/cone_classifier.py` | 카메라/라이다로 콘(cones) 위치·색상 추정 |
| 2. 위치 추정 | `perception/slam.py`, `v2x/rsu.py` | SLAM 맵 구축, RSU 인프라 신호 수신 |
| 3. 경로 계획 | `control/pure_pursuit.py` | 중심선 추종 및 조향각 산출 |
| 4. 속도 제어 | `control/speed.py` | 곡률 기반 목표 속도 및 종방향 제어 |
| 5. 런타임 보호 | `utils/watchdog.py`, `utils/lap_timer.py` | 런타임 이상 감시, 랩 타임 기록 |
| 6. 대회 드라이버 | `drivers/competition.py`, `drivers/advanced.py`, `drivers/autonomous.py`, `drivers/basic.py` | 시뮬레이터/실차 인터페이스 오케스트레이션 |

---

## 목차 (Table of Contents)

1. [프로젝트 소개 (Purpose)](#1-프로젝트-소개)
2. [저장소 구성 (Package Contents)](#2-저장소-구성)
3. [상태 및 지원 범위 (Status)](#3-상태-및-지원-범위)
4. [먼저 읽을 파일 (First Files to Read)](#4-먼저-읽을-파일)
5. [진입점 및 API (Entry Points)](#5-진입점-및-api)
6. [아키텍처 (Architecture)](#6-아키텍처)
7. [빠른 시작 (Quickstart)](#7-빠른-시작)
8. [명령어 레퍼런스 (Commands Reference)](#8-명령어-레퍼런스)
9. [설정 (Configuration)](#9-설정)
10. [로컬 개발 (Local Development)](#10-로컬-개발)
11. [테스트 (Testing)](#11-테스트)
12. [제출 패키징 (Submission)](#12-제출-패키징)
13. [기여 안내 (Contributing)](#13-기여-안내)
14. [유지보수자 (Maintainers)](#14-유지보수자)
15. [라이선스 및 추가 문서 (License & Further Docs)](#15-라이선스-및-추가-문서)

---

## 1. 프로젝트 소개

**qws941**는 Formula Student Driverless 클래스에서 사용 가능한 자율주행 소프트웨어 스택입니다. 이 저장소는 다음 두 가지 사용 시나리오를 지원합니다.

| 사용 시나리오 | 진입 워크스페이스 | 설명 |
| --- | --- | --- |
| 시뮬레이션 기반 개발 | `src/fsds_docker/` | FSDS 시뮬레이터와 동일한 컨테이너 안에서 인지·제어 알고리즘을 반복 개발·검증 |
| 실차/최종 통합 | `src/autonomous/` | 실차 런처(`bridge_no_camera.launch`)와 운영 스크립트로 대회 환경에서 실행 |

대상 사용자는 FS-D 팀의 자율주행 SW 엔지니어, 대회 참가 학생, 그리고 FSDS 기반 알고리즘을 검증하는 연구자입니다. 사용자는 콘 슬램(cone SLAM), pure pursuit 추종, V2X 신호 수신을 결합해 트랙 한 바퀴 완주 미션을 구현·개선할 수 있습니다.

---

## 2. 저장소 구성

| 경로 | 역할 |
| --- | --- |
| `src/autonomous/` | 실차/런치 기반 자율주행 워크스페이스 (ROS 패키지, 드라이버, 런처, 컨피그) |
| `src/autonomous/driver/` | 대회용 오케스트레이션 드라이버 (`competition_driver.py`) |
| `src/autonomous/modules/` | 인지·제어·유틸 모듈(perception / control / utils) |
| `src/autonomous/config/` | ROS 파라미터, 런치 파일 |
| `src/autonomous/scripts/` | 레이스 시작/녹화 스크립트 |
| `src/autonomous/tests/` | 알고리즘 단위 테스트 |
| `src/autonomous/Dockerfile`, `docker-compose.yml` | 실차/통합용 컨테이너 정의 |
| `src/fsds_docker/` | FSDS 시뮬레이터 통합 워크스페이스 (Docker, 런처, 드라이버, V2X) |
| `src/fsds_docker/scripts/` | 다단계 드라이버(advanced / competition / fsds / simple_slam) |
| `src/fsds_docker/src/drivers/` | 기본·자율·대회·고급 드라이버 모듈 |
| `src/fsds_docker/src/perception/` | 콘 탐지/분류, SLAM |
| `src/fsds_docker/src/control/` | pure pursuit, speed |
| `src/fsds_docker/src/v2x/` | RSU 통신 모듈 |
| `src/fsds_docker/launch/` | ROS 런치 파일 (`competition.launch`) |
| `src/fsds_docker/config/` | 드라이버 파라미터 YAML |
| `src/fsds_docker/tests/` | 알고리즘 단위 테스트 |
| `src/simulator/` | 시뮬레이터 설정 파일 (`settings.json`) |
| `src/submission/` | 대회 제출용 Dockerfile |
| `scripts/package.sh` | 제출용 산출물 패키징 스크립트 |
| `docs/` | 제출 가이드, 강의 참고 자료 |
| `docs/reference_materials/` | 강의 노트/주피터 노트북(`lecture1`, `lecture4`, `lecture6`) |
| `in-memoria.db` | 로컬 캐시/메모 DB(개발용 상태 파일) |
| `CONTRIBUTING.md`, `LICENSE`, `OWNERS` | 거버넌스 문서 |

---

## 3. 상태 및 지원 범위

| 영역 | 상태 |
| --- | --- |
| 대회 제출 | 운영 가능(production-ready). 제출 산출물은 `scripts/package.sh`로 빌드 |
| 알고리즘(인지/제어) | 단위 테스트 포함(`tests/test_algorithms.py`) |
| 컨테이너화 | FSDS용 / 자율주행용 두 개의 Docker Compose 제공 |
| 시뮬레이터 통합 | FSDS 공식 이미지와 `dev.sh`로 인터랙티브 개발 지원 |
| 지원 OS | Linux (Docker 실행 환경 기준) |
| 지원 ROS | ROS 1 Noetic 기준 |
| 알려진 한계 | 본 저장소는 ROS 2를 다루지 않음 |

---

## 4. 먼저 읽을 파일

새로운 팀원이 코드를 파악할 때 아래 순서로 읽으면 전체 동작을 빠르게 이해할 수 있습니다.

1. `src/autonomous/AGENTS.md` — 자율주행 워크스페이스 운영 규칙
2. `src/fsds_docker/README.md` — FSDS 컨테이너 사용법
3. `src/fsds_docker/docs/ARCHITECTURE.md` — 시스템 아키텍처
4. `src/fsds_docker/scripts/competition_driver.py` — 대회 모드 메인 루프
5. `src/autonomous/driver/competition_driver.py` — 실차/통합 메인 루프
6. `src/fsds_docker/src/control/pure_pursuit.py` — 경로 추종 알고리즘
7. `src/fsds_docker/src/perception/cone_detector.py` — 콘 탐지 알고리즘
8. `docs/SUBMISSION_GUIDE.md` — 대회 제출 절차

---

## 5. 진입점 및 API

### 5.1 운영 진입점

| 진입점 | 종류 | 설명 |
| --- | --- | --- |
| `src/fsds_docker/run.sh` | 셸 스크립트 | FSDS 시뮬레이터 + 자율 스택을 도커로 실행 |
| `src/fsds_docker/dev.sh` | 셸 스크립트 | 개발자용 인터랙티브 컨테이너 진입 |
| `src/autonomous/start.sh` | 셸 스크립트 | 자율주행 통합 컨테이너 시작 |
| `src/autonomous/run_all.sh` | 셸 스크립트 | 전체 파이프라인 일괄 실행 |
| `src/autonomous/scripts/start_race.py` | 파이썬 스크립트 | 레이스 시작 오케스트레이션 |
| `src/autonomous/record_race.sh` | 셸 스크립트 | 레이스 로그/녹화 |
| `src/autonomous/driver/competition_driver.py` | 파이썬 드라이버 | 대회 미션 메인 루프 |
| `src/fsds_docker/scripts/competition_driver.py` | 파이썬 드라이버 | 시뮬레이션 대회 미션 메인 루프 |
| `src/fsds_docker/scripts/fsds_driver.py` | 파이썬 드라이버 | 기본 FSDS 드라이버 |
| `src/fsds_docker/scripts/advanced_driver.py` | 파이썬 드라이버 | 고급 알고리즘 드라이버 |
| `src/fsds_docker/scripts/simple_slam.py` | 파이썬 스크립트 | SLAM 단독 실행 |

### 5.2 ROS 런치 / 파라미터 진입점

| 진입점 | 경로 | 설명 |
| --- | --- | --- |
| `competition.launch` | `src/fsds_docker/launch/` | FSDS 대회 모드 런치 |
| `bridge_no_camera.launch` | `src/autonomous/config/` | 실차 브리지(카메라 제외) 런치 |
| `params.yaml` | `src/autonomous/config/` | 자율주행 기본 파라미터 |
| `driver_params.yaml` | `src/fsds_docker/config/` | 드라이버 파라미터 |

### 5.3 모듈 API (Python 패키지)

| 모듈 | 주요 심볼 | 용도 |
| --- | --- | --- |
| `perception.cone_detector` | 콘 탐지 함수 | 라이다/이미지에서 콘 위치 산출 |
| `perception.cone_classifier` | 콘 분류 함수 | 색상(파랑/노랑) 분류 |
| `perception.slam` | SLAM 추정기 | 콘 기반 맵 구축 |
| `v2x.rsu` | RSU 클라이언트 | 인프라 신호 수신 |
| `control.pure_pursuit` | 경로 추종기 | 조향각 산출 |
| `control.speed` | 속도 제어기 | 곡률 기반 속도 |
| `utils.lap_timer` | 랩 타이머 | 랩 측정 |
| `utils.watchdog` | 워치독 | 런타임 이상 감시 |
| `drivers.basic / autonomous / competition / advanced` | 드라이버 클래스 | 시뮬레이터/실차 통합 |

---

## 6. 아키텍처

전체 시스템은 인지 → 위치 추정 → 계획 → 제어 → 보호 → 드라이버 순으로 흐릅니다.

| 단계 | 입력 | 처리 | 출력 |
| --- | --- | --- | --- |
| 1. 센서 입력 | 라이다/카메라/GPS | ROS 토픽 수신 | 원시 포인트/이미지 |
| 2. 콘 탐지 | 라이다 포인트 | `cone_detector` | 콘 후보 위치 |
| 3. 콘 분류 | 콘 후보 | `cone_classifier` | 색상 라벨 |
| 4. SLAM | 콘 분류 결과 | `slam` | 트랙 맵 + 차량 pose |
| 5. V2X | RSU 메시지 | `v2x.rsu` | 인프라 신호(신호등, 구역) |
| 6. 경로 계획 | 맵 + pose | `pure_pursuit` | 목표 조향각 |
| 7. 속도 결정 | 곡률 + 구역 | `speed` | 목표 속도 |
| 8. 보호 계층 | 모든 단계 | `watchdog`, `lap_timer` | 안전 정지 / 랩 기록 |
| 9. 드라이버 | 제어 명령 | `competition`/`advanced` | 시뮬레이터 또는 차량 액추에이터 명령 |

상세 아키텍처는 `src/fsds_docker/docs/ARCHITECTURE.md`를 참고하십시오.

---

## 7. 빠른 시작

### 7.1 사전 요구 사항

| 항목 | 버전/조건 |
| --- | --- |
| Docker Engine | 20.10 이상 권장 |
| docker compose | v2 이상 |
| 호스트 OS | Linux(권장) 또는 WSL2 |
| GPU | 선택. 학습/시각화 가속이 필요한 경우 NVIDIA Container Toolkit |
| 디스크 여유 공간 | 최소 15 GB(FSDS 이미지 포함) |

### 7.2 시뮬레이션 환경에서 실행(FSDS)

```bash
# 저장소 클론
git clone <repository-url> qws941
cd qws941

# FSDS 컨테이너 빌드 및 진입
cd src/fsds_docker
docker compose build
./run.sh            # 대회 미션 시뮬레이션 실행
# 또는 개발용으로 진입
./dev.sh
```

### 7.3 자율주행 통합 스택 실행(실차/통합)

```bash
cd src/autonomous
docker compose build
./start.sh          # 통합 스택 시작
./run_all.sh        # 전체 파이프라인 일괄 실행
```

### 7.4 ROS 런치 직접 호출

```bash
# FSDS 대회 런치
roslaunch src/fsds_docker/launch/competition.launch

# 실차 브리지 런치(카메라 제외)
roslaunch src/autonomous/config/bridge_no_camera.launch
```

### 7.5 제출 패키지 빌드

```bash
# 저장소 루트에서 실행
./scripts/package.sh
```

빌드 결과는 대회 제출용 Docker 이미지로 생성되며, `docs/SUBMISSION_GUIDE.md`의 절차에 따라 업로드합니다.

---

## 8. 명령어 레퍼런스

| 명령어 | 위치 | 목적 |
| --- | --- | --- |
| `./scripts/package.sh` | 루트 | 제출용 Docker 이미지/번들 생성 |
| `cd src/fsds_docker && docker compose build` | FSDS | FSDS 컨테이너 빌드 |
| `cd src/fsds_docker && ./run.sh` | FSDS | 시뮬레이션 대회 실행 |
| `cd src/fsds_docker && ./dev.sh` | FSDS | 인터랙티브 개발 셸 |
| `python src/fsds_docker/scripts/competition_driver.py` | FSDS | 대회 드라이버 단독 실행 |
| `python src/fsds_docker/scripts/simple_slam.py` | FSDS | SLAM만 단독 실행 |
| `cd src/autonomous && docker compose build` | 자율 | 자율주행 컨테이너 빌드 |
| `cd src/autonomous && ./start.sh` | 자율 | 스택 시작 |
| `cd src/autonomous && ./run_all.sh` | 자율 | 전체 파이프라인 실행 |
| `cd src/autonomous && ./record_race.sh` | 자율 | 레이스 로그/녹화 |
| `python src/autonomous/driver/competition_driver.py` | 자율 | 대회 드라이버 직접 실행 |
| `roslaunch src/fsds_docker/launch/competition.launch` | FSDS | ROS 대회 런치 |
| `roslaunch src/autonomous/config/bridge_no_camera.launch` | 자율 | 실차 브리지 런치 |

---

## 9. 설정

| 설정 파일 | 적용 대상 | 주요 키 |
| --- | --- | --- |
| `src/autonomous/config/params.yaml` | 자율 스택 전반 | 토픽 이름, 제어 게인, 안전 한계 |
| `src/autonomous/config/bridge_no_camera.launch` | 실차 브리지 | 센서 토픽 리매핑 |
| `src/fsds_docker/config/params.yaml` | FSDS 스택 | 시뮬레이터 파라미터 |
| `src/fsds_docker/config/driver_params.yaml` | FSDS 드라이버 | 경로 추종/속도 게인 |
| `src/simulator/settings.json` | FSDS 시뮬레이터 | 맵, 차량, 물리 파라미터 |

설정을 변경한 후에는 컨테이너를 재빌드하거나 ROS 파라미터 서버(`rosparam load`)로 런타임에 주입할 수 있습니다.

---

## 10. 로컬 개발

| 작업 | 절차 |
| --- | --- |
| 컨테이너 안에서 개발 | `./dev.sh`로 진입 후 `src/` 트리 수정 → 호스트와 볼륨 공유 |
| 알고리즘 반복 테스트 | `tests/test_algorithms.py`를 컨테이너 내부에서 pytest로 실행 |
| 컨피그 핫리로드 | ROS 파라미터 서버 또는 YAML 재생성 |
| 로그 확인 | `record_race.sh` 결과, ROS 로그(`/root/.ros/log`) |
| DB 캐시 활용 | `in-memoria.db`는 로컬 메모/캐시 파일로, 삭제 시 재구성됨 |

### 10.1 워크플로 권장 순서

1. `dev.sh`로 컨테이너 진입
2. 알고리즘 수정 후 단위 테스트 실행
3. `competition_driver.py`로 미션 단축 검증
4. `competition.launch`로 전체 ROS 그래프 검증
5. 파라미터 튜닝 후 `record_race.sh`로 랩 기록 비교
6. 안정화되면 `scripts/package.sh`로 제출 이미지 빌드

---

## 11. 테스트

| 테스트 파일 | 커버 영역 | 실행 방법 |
| --- | --- | --- |
| `src/autonomous/tests/test_algorithms.py` | pure pursuit, speed, cone classifier 등 핵심 알고리즘 | 컨테이너 내부에서 `pytest` |
| `src/fsds_docker/tests/test_algorithms.py` | FSDS 환경 알고리즘 동일 항목 | 컨테이너 내부에서 `pytest` |

테스트 추가 시 같은 디렉터리에 `test_*.py` 명명 규칙을 따르고, 외부 ROS 의존 없이 동작하는 순수 함수 위주로 작성합니다.

---

## 12. 제출 패키징

`scripts/package.sh`는 대회 제출에 필요한 Docker 이미지를 빌드하고, 실행에 필요한 설정 파일·런처를 한 번에 묶어 산출물을 생성합니다.

| 단계 | 설명 |
| --- | --- |
| 1. 사전 검사 | `src/submission/Dockerfile`, 컨피그 존재 확인 |
| 2. 이미지 빌드 | 대회 제출용 Dockerfile로 이미지 생성 |
| 3. 번들링 | 런처, 컨피그, README를 단일 디렉터리로 복사 |
| 4. 산출물 | 대회 심사 시스템이 소비 가능한 디렉터리/이미지 |

상세 절차는 `docs/SUBMISSION_GUIDE.md`를 따릅니다.

---

## 13. 기여 안내

기여 절차는 `CONTRIBUTING.md`를 따릅니다. 일반적인 가이드라인은 다음과 같습니다.

| 항목 | 규칙 |
| --- | --- |
| 브랜치 모델 | 기능 브랜치 → PR → 리뷰 → 머지 |
| 커밋 메시지 | 변경 의도와 영향 범위를 명확히 기술 |
| 코드 스타일 | PEP 8, ROS 패키지 명명 규칙 준수 |
| 테스트 | 알고리즘 변경 시 단위 테스트 동반 |
| 문서 | 사용자 가용 동작이 바뀌면 README/ARCHITECTURE 동시 갱신 |
| OWNERS | 책임 영역별 OWNERS 지정에 따라 리뷰어 자동 할당 |

---

## 14. 유지보수자

| 항목 | 출처 |
| --- | --- |
| 코드 오너십 | 저장소 루트의 `OWNERS` 파일 |
| 운영 규칙 | `src/autonomous/AGENTS.md`, `src/fsds_docker/AGENTS.md`, `src/submission/AGENTS.md` |
| 책임 분리 | 인지(perception), 제어(control), V2X, 드라이버, 인프라(Docker) 단위로 OWNERS 구분 |

담당자 변경 시 `OWNERS` 파일과 각 워크스페이스의 `AGENTS.md`를 함께 갱신합니다.

---

## 15. 라이선스 및 추가 문서

| 문서 | 경로 |
| --- | --- |
| 라이선스 | `LICENSE` |
| 기여 가이드 | `CONTRIBUTING.md` |
| 제출 가이드 | `docs/SUBMISSION_GUIDE.md` |
| FSDS 아키텍처 | `src/fsds_docker/docs/ARCHITECTURE.md` |
| FSDS 워크스페이스 사용법 | `src/fsds_docker/README.md` |
| 자율 워크스페이스 운영 규칙 | `src/autonomous/AGENTS.md` |
| 강의 참고 자료 | `docs/reference_materials/` (`lecture1_fsds_install.txt`, `lecture4_slam.ipynb`, `lecture6_v2x.ipynb`) |