```markdown
# HYCU FSDS 자율주행

Formula Student Driverless Simulator(FSDS) 기반 자율주행 시스템입니다. Windows에서 실행되는 FSDS 시뮬레이터(Unreal Engine 4)와 Linux Docker(ROS Noetic) 환경의 자율주행 스택을 결합한 듀얼 플랫폼 프로젝트입니다.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
![ROS](https://img.shields.io/badge/ROS-Noetic-blue)
![Python](https://img.shields.io/badge/Python-3-green)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED)

---

## 개요

- **목표**: Formula Student Driverless 대회용 자율주행 알고리즘 개발 및 검증
- **시뮬레이터**: [Formula-Student-Driverless-Simulator](https://github.com/FS-Driverless/Formula-Student-Driverless-Simulator) v2.2.0 (AirSim 기반)
- **자율주행 스택**: ROS Noetic + Python 3
- **제어 알고리즘**: Pure Pursuit 경로 추적 + 곡률 기반 속도 제어
- **안전 시스템**: 3단계 워치독 상태 머신 (TRACKING → DEGRADED → STOPPING)

## 주요 기능

- **FSDS 연동**: AirSim RPC(41451) 기반 센서 데이터(LiDAR ×2, Camera ×2, GPS/IMU/GSS) 수신 및 제어 명령 전송
- **인지(Perception)**: LiDAR/Camera 기반 콘(cone) 탐지 및 분류, SLAM
- **제어(Control)**: Pure Pursuit + 곡률 기반 속도 프로파일
- **V2X 지원**: RSU(Roadside Unit) 연동 및 차량-인프라 통신
- **멀티 드라이버 모드**: Basic / Advanced / Autonomous / Competition 단계별 주행 모듈
- **컨테이너화**: Docker Compose 기반 일관된 개발·제출 환경 제공
- **CI/CD 자동화**: GitHub Actions/App 기반 리포지토리 검증 및 리뷰 봇

## 시스템 아키텍처

```
┌────────────────────────────┐         ┌────────────────────────────────────┐
│  Windows / Linux (Host)    │  RPC    │  Linux Docker (ROS Noetic)         │
│                            │ :41451  │                                    │
│  FSDS Simulator (UE4)      │◄───────►│  fsds_bridge ──► /lidar /odom ...  │
│   - LiDAR x2               │         │       │                            │
│   - GPS / IMU / GSS        │         │       ▼                            │
│   - Camera x2              │         │  competition_driver (ROS Node)     │
│                            │         │   ├─ perception/cone_detector      │
│                            │         │   ├─ perception/slam               │
│                            │         │   ├─ control/pure_pursuit          │
│                            │         │   ├─ control/speed                 │
│                            │         │   └─ utils/watchdog                │
│                            │         │       │                            │
│                            │         │       ▼  /fsds/control_command     │
│                            │◄────────┤  AirSim Python Client (RPC)        │
└────────────────────────────┘         └────────────────────────────────────┘
```

## 설치 방법

### 필수 조건

- **Windows 10/11**: FSDS Simulator 실행 환경
- **Docker 20.10+** 및 **Docker Compose**
- **Git**

### 빠른 시작

```bash
# 1. 저장소 클론
git clone <repository-url>
cd hycu-fsds-autonomous

# 2. 자율주행 스택 Docker 이미지 빌드
cd src/autonomous
docker-compose build

# 3. 제출용 이미지 빌드 (대회 제출 시)
cd ../../submission
docker-compose build
```

> **참고**: FSDS 시뮬레이터는 Windows 환경에서 별도로 설치해야 합니다. 설치 가이드는 [FSDS 공식 문서](https://github.com/FS-Driverless/Formula-Student-Driverless-Simulator)를 참고하세요.

## 사용 방법

### 1. 시뮬레이터 실행 (Windows)

FSDS Simulator를 실행하고 원하는 맵(트랙)을 로드합니다.

### 2. 자율주행 스택 실행 (개발/테스트)

```bash
cd src/autonomous

# 전체 시스템 일괄 실행
./run_all.sh

# 또는 개별 단계별 실행
./start.sh          # ROS 마스터 및 브리지 실행
./record_race.sh    # 주행 데이터 기록
```

### 3. 단위 테스트

```bash
cd src/autonomous
python -m pytest tests/test_algorithms.py -v
```

### 4. 대회 제출용 빌드 및 실행

```bash
cd submission

# 개발 모드 (로컬 테스트)
./dev.sh

# 제출 모드 (최종 검증)
./run.sh
```

## 프로젝트 구조

```
hycu-fsds-autonomous/
├── src/
│   ├── simulator/              # FSDS 시뮬레이터 설정 및 에셋 구성
│   └── autonomous/             # 메인 자율주행 스택 (ROS/Docker)
│       ├── config/             # ROS launch, 파라미터(params.yaml)
│       ├── driver/             # 대회용 드라이버 노드
│       ├── modules/            # 핵심 알고리즘 패키지
│       │   ├── control/        # Pure Pursuit, 속도 제어
│       │   ├── perception/     # 콘 탐지/분류, SLAM
│       │   └── utils/          # 랩타이머, 워치독
│       ├── scripts/            # 레이스 실행 유틸리티
│       └── tests/              # 알고리즘 단위 테스트
├── submission/                 # 대회 제출용 코드 및 Dockerfile
│   ├── src/
│   │   ├── control/
│   │   ├── drivers/            # basic / advanced / autonomous / competition
│   │   ├── utils/
│   │   └── v2x/                # RSU 통신 모듈
│   └── Dockerfile
├── docs/                       # 문서 및 참고 자료(강의 노트, 가이드)
│   ├── SUBMISSION_GUIDE.md
│   └── reference_materials/
├── _bot-scripts/               # GitHub Actions/App 자동화 스크립트
├── scripts/                    # 패키징 및 배포 스크립트
├── CONTRIBUTING.md             # 기여 가이드
└── AGENTS.md                   # 팀원 및 역할 정의
```

## 기여 방법

이 프로젝트에 기여하려면 다음 문서를 참고해 주세요.

- **[CONTRIBUTING.md](CONTRIBUTING.md)**: 커밋 컨벤션, 브랜치 전략, PR 절차
- **[AGENTS.md](AGENTS.md)**: 팀 구성 및 모듈별 담당자 정보
- **코드 리뷰**: 모든 PR은 `_bot-scripts`의 자동화 봇을 통한 검증과 팀원 리뷰를 거칩니다.

버그 리포트나 기능 제안은 GitHub Issues를 통해 등록해 주세요.

## 라이선스

이 프로젝트는 [MIT License](LICENSE)를 따릅니다.
```