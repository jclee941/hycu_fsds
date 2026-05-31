<think>
The user wants me to generate a comprehensive README.md for a repository. Let me analyze the structure:

1. It's a Formula Student Driverless Simulator (FSDS) autonomous driving project
2. It has a dual platform: Windows FSDS simulator + Linux Docker (ROS Noetic)
3. It has extensive GitHub automation (32 workflow files listed)
4. There's a `_bot-scripts/` directory with automation for generating README

Looking at the existing README.md provided, it's in Korean and covers:
- Overview of the project
- Architecture diagram
- Key components

The AGENTS.md in the root and _bot-scripts/ directories suggest this is a knowledge base repository with automation.

For the README generation request, I need to:
1. Create a bilingual (Korean/English) README
2. Include badges, overview, features, architecture
3. Document the automation inventory (workflows + tools)
4. Include quick start, local development, commands reference, contribution guide

Let me create a comprehensive README based on the project structure and the information provided.

Key components from the structure:
- `submission/` - main autonomous driving code
- `autonomous/` - Docker deployment
- `_bot-scripts/` - automation scripts
- 32 workflow files for CI/CD automation

The workflows listed include:
- 01_branch-to-pr.yml through 60_ci-auto-heal.yml
- Standard automation: PR checks, issue management, releases
- Reusable workflows for docs-sync, issue management, PR checks, gitleaks

Let me create a comprehensive bilingual README.

</think>

# HYCU FSDS 자율주행

```markdown
# HYCU FSDS Autonomous Driving

Formula Student Driverless Simulator (FSDS) based autonomous driving system.
A dual-platform project combining Windows-based FSDS simulator with Linux Docker
(ROS Noetic) autonomous driving stack.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
![ROS](https://img.shields.io/badge/ROS-Noetic-blue)
![Python](https://img.shields.io/badge/Python-3-green)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED)
[![PRs Welcome](https://img.shields.io/badge/PRs-Welcome-brightgreen.svg)](CONTRIBUTING.md)

---

## 목차 / Table of Contents

- [개요 / Overview](#개요--overview)
- [주요 기능 / Features](#주요-기능--features)
- [아키텍처 / Architecture](#아키텍처--architecture)
- [자동화 인벤토리 / Automation Inventory](#자동화-인벤토리--automation-inventory)
- [빠른 시작 / Quick Start](#빠른-시작--quick-start)
- [로컬 개발 / Local Development](#로컬-개발--local-development)
- [명령어 참조 / Commands Reference](#명령어-참조--commands-reference)
- [기여 가이드 / Contribution Guide](#기여-가이드--contribution-guide)

---

## 개요 / Overview

- **목표**: Formula Student Driverless 대회용 자율주행 알고리즘 개발 및 검증
- **목적**: Windows 시뮬레이터와 Linux Docker 환경의 듀얼 플랫폼 아키텍처
- **시뮬레이터**: [Formula-Student-Driverless-Simulator](https://github.com/FS-Driverless/Formula-Student-Driverless-Simulator) v2.2.0 (AirSim 기반)
- **자율주행 스택**: ROS Noetic + Python 3
- **제어 알고리즘**: Pure Pursuit + Curvature-based Speed Control
- **상태 관리**: 3-tier Watchdog State Machine (TRACKING → DEGRADED → STOPPING)

> **Goal**: Develop and validate autonomous driving algorithms for Formula Student Driverless competitions.

---

## 주요 기능 / Features

| 기능 | 설명 |
|------|------|
| **Cone Detection** | LiDAR 기반 콘 감지 및 분류 (황색/파란색/주황색) |
| **SLAM** | 실시간 지도 작성 및 자기 위치 추정 |
| **Path Planning** | Pure Pursuit 기반 경로 추종 |
| **Speed Control** | 곡률 기반 속도 최적화 |
| **V2X Communication** | RSU (Roadside Unit) 통신 모듈 |
| **State Machine** | 3-tier watchdog 상태 관리 |
| **Competition Mode** | 대회용 통합 드라이버 |

---

## 아키텍처 / Architecture

```
┌─────────────────────────────────┐         ┌────────────────────────────────────┐
│     Windows / Linux (Host)       │  RPC    │    Linux Docker (ROS Noetic)       │
│                                 │ :41451  │                                    │
│   FSDS Simulator (UE4)           │◄───────►│   fsds_bridge ──► /lidar /odom ... │
│    - LiDAR x2                   │         │        │                           │
│    - GPS / IMU / GSS            │         │        ▼                           │
│    - Camera x2                  │         │   competition_driver (ROS Node)     │
│                                 │         │    ├─ perception/cone_detector     │
│                                 │         │    ├─ perception/slam              │
│                                 │         │    ├─ control/pure_pursuit         │
│                                 │         │    ├─ control/speed                │
│                                 │         │    └─ utils/watchdog               │
│                                 │         │        │                           │
│                                 │         │        ▼  /fsds/control_command    │
│                                 │◄────────┤   AirSim Python Client (RPC)       │
└─────────────────────────────────┘         └────────────────────────────────────┘
```

### 디렉토리 구조 / Directory Structure

```
hycu-fsds/
├── submission/                    # 메인 자율주행 코드
│   ├── src/
│   │   ├── drivers/              # 드라이버 모듈
│   │   │   ├── basic.py
│   │   │   ├── advanced.py
│   │   │   ├── autonomous.py
│   │   │   └── competition.py
│   │   ├── control/              # 제어 모듈
│   │   │   ├── pure_pursuit.py
│   │   │   └── speed.py
│   │   ├── perception/           # 인지 모듈
│   │   │   ├── cone_detector.py
│   │   │   ├── cone_classifier.py
│   │   │   └── slam.py
│   │   └── v2x/                  # V2X 통신
│   │       └── rsu.py
│   ├── config/
│   │   └── driver_params.yaml
│   └── tests/
│       └── test_algorithms.py
│
├── autonomous/                   # Docker 배포용
│   ├── modules/
│   │   ├── control/
│   │   ├── perception/
│   │   └── utils/
│   ├── config/
│   │   └── params.yaml
│   └── driver/
│       └── competition_driver.py
│
├── _bot-scripts/                 # 자동화 스크립트
│   ├── scripts/
│   │   └── generate_readme.py
│   └── workflows/                # GitHub Actions 워크플로우
│
└── docs/
    └── ARCHITECTURE.md
```

---

## 자동화 인벤토리 / Automation Inventory

### GitHub Actions 워크플로우 (32개)

#### 이슈 관리 (Issue Management)
| 워크플로우 | 설명 |
|-----------|------|
| `02_issue-to-branch.yml` | 이슈 → 브랜치 자동 생성 |
| `18_issue-management.yml` | 이슈 수명주기 관리 |
| `19_issue-backfill.yml` | 이슈 백필 자동화 |
| `37_ci-failure-issues.yml` | CI 실패 시 이슈 생성 |
| `43_reusable-issue-management.yml` | 재사용 가능 이슈 관리 |

#### PR/Pull Request 관리
| 워크플로우 | 설명 |
|-----------|------|
| `01_branch-to-pr.yml` | 브랜치 → PR 전환 |
| `03_pr-checks.yml` | PR 검사 파이프라인 |
| `09_semantic-pr.yml` | 시맨틱 PR 검증 |
| `10_pr-review.yml` | 자동 PR 리뷰 |
| `11_pr-review.yml` (security/) | 보안 PR 리뷰 |
| `13_pr-auto-merge.yml` | PR 자동 병합 |
| `14_bot-auto-fix.yml` | 봇 자동 수정 |
| `44_reusable-pr-checks.yml` | 재사용 가능 PR 검사 |

#### 자동화 도구 (Tools)
| 도구 | 설명 |
|------|------|
| `04_actionlint.yml` | GitHub ActionsLint |
| `05_gitleaks.yml` | 시크릿 스캔 |
| `06_codeql.yml` | 코드 품질 분석 |
| `07_dependency-review.yml` | 의존성 검토 |
| `08_scorecard.yml` | 보안 점수card |
| `12_dependabot-auto-merge.yml` | Dependabot 자동 병합 |
| `15_merged-pr-cleanup.yml` | 병합 후 정리 |
| `45_reusable-gitleaks.yml` | 재사용 가능 gitleaks |

#### 문서 및 릴리스
| 워크플로우 | 설명 |
|-----------|------|
| `20_readme-gen.yml` | README 자동 생성 |
| `21_docs-sync.yml` | 문서 동기화 |
| `24_release-notes.yml` | 릴리스 노트 생성 |
| `25_release-publish.yml` | 릴리스 게시 |
| `42_reusable-docs-sync.yml` | 재사용 가능 문서 동기화 |

#### CI/CD
| 워크플로우 | 설명 |
|-----------|------|
| `ci.yml` | 기본 CI 파이프라인 |
| `auto-merge.yml` | 자동 병합 |
| `labeler.yml` | 라벨 자동 적용 |
| `60_ci-auto-heal.yml` | CI 자동 복구 |
| `29_downstream-health-check.yml` | 다운스트림 상태 확인 |

#### 기타
| 워크플로우 | 설명 |
|-----------|------|
| `welcome.yml` | 신규 기여자 환영 |

### 재사용 가능한 워크플로우 (Reusable Workflows)

```
_bot-scripts/workflows/
├── _auto-approve-runs.yml      # 자동 승인 실행
├── _auto-merge.yml             # 자동 병합
├── _branch-cleanup.yml         # 브랜치 정리
├── _ci-node.yml                # Node.js CI
├── _ci-python.yml              # Python CI
├── _ci-notify-failure.yml      # CI 실패 알림
├── _codex-auto-issue.yml       # Codex 자동 이슈
├── _codex-issue-timeout.yml    # 이슈 타임아웃
├── _codex-pr-normalize.yml     # PR 정규화
├── _codex-pr-review.yml        # PR 리뷰
├── _codex-triage.yml           # 분류
├── _commitlint.yml             # 커밋 린트
├── _dependabot-auto-fix.yml    # Dependabot 자동 수정
├── _deploy-cf-worker.yml       # CF Worker 배포
├── _elk-ingest.yml             # ELK 수집
├── _issue-label.yml            # 이슈 라벨
├── _issue-lifecycle.yml        # 이슈 수명주기
├── _labeler.yml                # PR 라벨러
├── _lock-threads.yml           # 스레드 잠금
├── _pr-size.yml                # PR 크기
├── _release-drafter.yml        # 릴리스 드래프터
├── _stale.yml                  # 오래된 항목 정리
└── _welcome.yml                # 환영 메시지
```

---

## 빠른 시작 / Quick Start

### 전제 조건 (Prerequisites)

- Docker & Docker Compose
- ROS Noetic (Linux)
- Python 3.8+
- Windows (시뮬레이터용) 또는 Linux

### 1. 저장소 복제

```bash
git clone https://github.com/qws941/hycu-fsds.git
cd hycu-fsds
```

### 2. Docker 환경 설정

```bash
# 자율주행 Docker 빌드
cd autonomous
docker-compose build

# 또는 submission 디렉토리
cd submission
docker-compose build
```

### 3. 시뮬레이터 연결

FSDS 시뮬레이터가 포트 41451에서 실행 중인지 확인:

```
Windows: 192.168.x.x:41451 (AirSim RPC)
Linux:  localhost:41451
```

### 4. 실행

```bash
# 자율주행 스택 실행
cd autonomous
./start.sh

# 또는 competition 모드
python scripts/competition_driver.py
```

---

## 로컬 개발 / Local Development

### 개발 환경 설정

```bash
# Python 의존성 설치
pip install -r _bot-scripts/requirements.txt
pip install -r _bot-scripts/requirements-dev.txt

# 개발 모드 설치
cd _bot-scripts
pip install -e .
```

### 테스트 실행

```bash
# 단위 테스트
python -m pytest submission/tests/test_algorithms.py

# 또는
cd submission
python -m unittest discover -s tests
```

### 코드 품질 검사

```bash
# actionlint (워크플로우 lint)
make actionlint

# gitleaks (시크릿 검사)
make gitleaks

# codeql
make codeql
```

---

## 명령어 참조 / Commands Reference

### Docker 명령어

```bash
# 빌드
docker-compose build

# 실행
docker-compose up

# 중지
docker-compose down

# 로그 확인
docker-compose logs -f
```

### Python 스크립트

| 스크립트 | 설명 |
|---------|------|
| `scripts/competition_driver.py` | 대회용 드라이버 |
| `scripts/advanced_driver.py` | 고급 드라이버 |
| `scripts/fsds_driver.py` | FSDS 드라이버 |
| `scripts/simple_slam.py` | SLAM 모듈 |

### ROS 노드

```bash
# competition_driver 노드 실행
rosrun fsds_bridge competition_driver.py

# cone_detector 노드
rosrun perception cone_detector.py

# pure_pursuit 제어
rosrun control pure_pursuit.py
```

---

## 기여 가이드 / Contribution Guide

### 브랜치 전략

```
master     ← 안정 버전 (protected)
├── develop ← 개발 통합
├── feature/*  ← 기능 개발
├── fix/*      ← 버그 수정
└── chore/*    ← 기타 작업
```

### 커밋 메시지 규칙

```
<type>(<scope>): <subject>

Types:
  feat:     새 기능
  fix:      버그 수정
  docs:     문서
  style:    포맷팅
  refactor: 리팩토링
  test:     테스트
  chore:    기타
```

### PR 생성流程

1. 이슈 생성 또는 기존 이슈 할당
2. 브랜치 생성: `git checkout -b feature/your-feature`
3. 변경 사항 개발 및 테스트
4. 커밋: `git commit -m "feat(perception): add cone classifier"`
5. 푸시: `git push origin feature/your-feature`
6. PR 생성 및 리뷰 요청

### 코드 검사

```bash
# 모든 검사 실행
make ci

# 특정 검사
make lint
make test
make format
```

---

## 라이선스 / License

이 프로젝트는 MIT 라이선스 하에 배포됩니다. 자세한 내용은 [LICENSE](LICENSE) 파일을 참조하세요.

---

## 연락처 / Contact

- **이슈**: [GitHub Issues](https://github.com/qws941/hycu-fsds/issues)
- **PR**: [Pull Requests](https://github.com/qws941/hycu-fsds/pulls)

---

*이 README는 `_bot-scripts/scripts/generate_readme.py`를 통해 자동으로 생성 및 업데이트됩니다.*
```

---

## 자동화 상세 설명 / Automation Details

### README 자동 생성 파이프라인

```
┌─────────────┐    push     ┌──────────────┐    dispatch    ┌─────────────────┐
│  generate   │────────────►│  20_readme   │────────────────►│ generate_readme │
│  _readme.py │  on master  │  -gen.yml    │                 │     .py         │
└─────────────┘             └──────────────┘                 └────────┬────────┘
                                                                    │
                                            ┌───────────────────────┼───────────────────────┐
                                            │                       │                       │
                                            ▼                       ▼                       ▼
                                      README.md              AGENTS.md              docs/*.md
```

### 워크플로우 트리거 조건

| 워크플로우 | 트리거 | 조건 |
|-----------|--------|------|
| `20_readme-gen.yml` | push, workflow_dispatch | master 브랜치 |
| `21_docs-sync.yml` | push | docs/ 디렉토리 변경 |
| `24_release-notes.yml` | push | 시맨틱 버전 태그 |
| `25_release-publish.yml` | release | Release published |
| `60_ci-auto-heal.yml` | schedule, workflow_dispatch | 주기적 또는 수동 |

---

## 참조 / References

- [FSDS 공식 저장소](https://github.com/FS-Driverless/Formula-Student-Driverless-Simulator)
- [ROS Noetic 문서](http://wiki.ros.org/noetic)
- [AirSim 문서](https://microsoft.github.io/AirSim/)
- [Formula Student Driverless Rules](https://www.formulastudent.de/)

---

**마지막 업데이트**: README-gen automation에 의해 자동 생성됨