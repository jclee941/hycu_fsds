# HYCU FSDS Autonomous Driving

# HYCU FSDS 자율주행

> Formula Student Driverless Simulator 기반 자율주행 시스템  
> Formula Student Driverless Simulator (FSDS) Based Autonomous Driving System

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![ROS: Noetic](https://img.shields.io/badge/ROS-Noetic-blue)](http://wiki.ros.org/noetic)
[![Python: 3.8+](https://img.shields.io/badge/Python-3.8+-green)](https://www.python.org/)
[![Docker](https://img.shields.io/badge/Docker-Ready-blue)](https://www.docker.com/)

---

## 목차 (Table of Contents)

- [개요 (Overview)](#개요-overview)
- [주요 기능 (Key Features)](#주요-기능-key-features)
- [시스템 아키텍처 (Architecture)](#시스템-아키텍처-architecture)
- [자동화 인벤토리 (Automation Inventory)](#자동화-인벤토리-automation-inventory)
- [빠른 시작 (Quick Start)](#빠른-시작-quick-start)
- [로컬 개발 (Local Development)](#로컬-개발-local-development)
- [명령어 참고서 (Commands Reference)](#명령어-참고서-commands-reference)
- [기여 가이드 (Contribution)](#기여-가이드-contribution)

---

## 개요 (Overview)

본 프로젝트는 **Formula Student Driverless Simulator (FSDS)** 기반으로 개발된 자율주행 시스템입니다. Windows 환경의 시뮬레이터와 Linux (ROS Noetic) Docker 기반 자율주행 스택을 결합한 이중 플랫폼架构으로,cone 检测 (콘 감지), SLAM, 경로 계획 및 제어 기능을 통합합니다.

This project is an autonomous driving system based on the **Formula Student Driverless Simulator (FSDS)**. It combines a Windows-based simulator with a Linux (ROS Noetic) Docker-based autonomous driving stack, integrating cone detection, SLAM, path planning, and control functions.

---

## 주요 기능 (Key Features)

### 자율주행 모듈 (Autonomous Driving Modules)

| 모듈 (Module) | 설명 (Description) |
|---------------|-------------------|
| **Perception** | Cone detection, classification, SLAM |
| **Control** | Pure pursuit, speed control |
| **Drivers** | Basic, Advanced, Competition, Autonomous modes |
| **V2X** | RSU (Roadside Unit) communication |

### 개발 인프라 (Development Infrastructure)

| 기능 (Feature) | 설명 (Description) |
|---------------|-------------------|
| **이중 플랫폼** | Windows FSDS + Linux Docker (ROS Noetic) |
| **Docker 지원** | Full containerized development & deployment |
| **테스트 자동화** | Algorithm testing with CI/CD integration |
| **자동화 워크플로우** | 32개의 GitHub Actions 워크플로우 |

---

## 시스템 아키텍처 (Architecture)

```
┌─────────────────────────────────────────────────────────────────┐
│                      FSDS (Windows)                              │
│                   시뮬레이터 환경                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │   Cones     │  │     Car      │  │    Track     │          │
│  │  (감지 대상)  │  │  (차량 모델)  │  │   (트랙)      │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                   ROS Noetic (Linux Docker)                      │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                    Perception Layer                       │   │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐        │   │
│  │  │Cone Detector│ │Classifier   │ │    SLAM     │        │   │
│  │  └─────────────┘ └─────────────┘ └─────────────┘        │   │
│  └──────────────────────────────────────────────────────────┘   │
│                              │                                   │
│                              ▼                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                     Control Layer                         │   │
│  │  ┌─────────────────┐        ┌─────────────────┐          │   │
│  │  │  Pure Pursuit  │        │ Speed Controller│          │   │
│  │  └─────────────────┘        └─────────────────┘          │   │
│  └──────────────────────────────────────────────────────────┘   │
│                              │                                   │
│                              ▼                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                     Driver Layer                          │   │
│  │  ┌────────┐ ┌────────┐ ┌───────────┐ ┌───────────┐      │   │
│  │  │  Basic │ │Advanced│ │Competition│ │Autonomous │      │   │
│  │  └────────┘ └────────┘ └───────────┘ └───────────┘      │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### 디렉토리 구조 (Directory Structure)

```
/
├── submission/                    # 메인 자율주행 코드
│   ├── src/
│   │   ├── drivers/              # 드라이버 모듈 (basic, advanced, autonomous, competition)
│   │   ├── control/              # 제어 모듈 (pure_pursuit, speed)
│   │   ├── perception/           # 인식 모듈 (cone_detector, classifier, slam)
│   │   └── v2x/                  # V2X 통신 (rsu)
│   ├── config/                   # 파라미터 설정
│   ├── tests/                    # 단위 테스트
│   ├── launch/                   # ROS 런치 파일
│   └── scripts/                  # 실행 스크립트
│
├── autonomous/                   # Docker 배포용
│   ├── modules/                 # 모듈화된 코드
│   ├── config/                   # 설정 파일
│   └── driver/                  # 드라이버
│
├── _bot-scripts/                 # 자동화 스크립트
│   ├── scripts/                  # 유틸리티 스크립트
│   └── (GitHub Actions 워크플로우정의)
│
└── .github/workflows/            # GitHub Actions 워크플로우 (32개)
```

---

## 자동화 인벤토리 (Automation Inventory)

### GitHub Actions 워크플로우 (32 워크플로우)

#### 1. PR & Branch 자동화

| 워크플로우 | 설명 |
|-----------|------|
| `01_branch-to-pr.yml` | 브랜치 → PR 자동 생성 |
| `02_issue-to-branch.yml` | 이슈 → 브랜치 자동 생성 |
| `03_pr-checks.yml` | PR 검증을 위한 통합 체크 |
| `09_semantic-pr.yml` | Semantic PR 리ント |
| `10_pr-review.yml` | 자동 PR 리뷰 |
| `13_pr-auto-merge.yml` | PR 자동 병합 |
| `14_bot-auto-fix.yml` | 봇 자동 수정 |
| `15_merged-pr-cleanup.yml` | 병합 후 브랜치 정리 |
| `security/11_pr-review.yml` | 보안 리뷰 워크플로우 |

#### 2. CI/CD & 품질검증

| 워크플로우 | 설명 |
|-----------|------|
| `04_actionlint.yml` | GitHub Actions YAML 린트 |
| `05_gitleaks.yml` | 시크릿/민감정보 스캔 |
| `06_codeql.yml` | CodeQL 정적 분석 |
| `07_dependency-review.yml` | 의존성 보안 리뷰 |
| `08_scorecard.yml` | OpenSSF 보안 점수 |
| `44_reusable-pr-checks.yml` | 재사용可能な PR 체크 |
| `45_reusable-gitleaks.yml` | 재사용 가능한 Gitleaks |
| `ci.yml` | 기본 CI 파이프라인 |
| `60_ci-auto-heal.yml` | CI 자동 복구 |

#### 3. 이슈 관리

| 워크플로우 | 설명 |
|-----------|------|
| `18_issue-management.yml` | 이슈 자동 관리 |
| `19_issue-backfill.yml` | 이슈 백필 |
| `37_ci-failure-issues.yml` | CI 실패 → 이슈 자동 생성 |
| `43_reusable-issue-management.yml` | 재사용 가능한 이슈 관리 |
| `labeler.yml` | 자동 라벨링 |
| `welcome.yml` | 신규 기여자 환영 |
| `auto-merge.yml` | 자동 병합 |

#### 4. 문서 & README

| 워크플로우 | 설명 |
|-----------|------|
| `20_readme-gen.yml` | README 자동 생성 |
| `21_docs-sync.yml` | 문서 동기화 |
| `42_reusable-docs-sync.yml` | 재사용 가능한 문서 동기화 |

#### 5. 릴리스 & 배포

| 워크플로우 | 설명 |
|-----------|------|
| `24_release-notes.yml` | 릴리스 노트 생성 |
| `25_release-publish.yml` | 릴리스 게시 |
| `12_dependabot-auto-merge.yml` | Dependabot 자동 병합 |

#### 6. 하위 프로젝트 건강도

| 워크플로우 | 설명 |
|-----------|------|
| `29_downstream-health-check.yml` | 하위 프로젝트 건강도 체크 |

#### 7. 재사용 가능한 워크플로우 (Reusable)

| 워크플로우 | 설명 |
|-----------|------|
| `42_reusable-docs-sync.yml` | 문서 동기화 템플릿 |
| `43_reusable-issue-management.yml` | 이슈 관리 템플릿 |
| `44_reusable-pr-checks.yml` | PR 체크 템플릿 |
| `45_reusable-gitleaks.yml` | Gitleaks 템플릿 |

---

### 자동화 도구 (Automation Tools)

| 도구 | 용도 |
|------|------|
| **actionlint** | GitHub Actions YAML 검증 |
| **gitleaks** | 민감정보 스캔 |
| **CodeQL** | 정적 코드 분석 |
| **OSV/Dependabot** | 의존성 취약점 스캔 |
| **OpenSSF Scorecard** | 보안 점수 산출 |
| **semantic-pull-request** | conventional commits 검증 |
| **github/codeql-action** | CodeQL 분석 실행 |
| **step-security/harden-runner** | 러너 보안 강화 |
| **docker/build-push-action** | Docker 이미지 빌드/푸시 |

---

## 빠른 시작 (Quick Start)

### 사전 조건 (Prerequisites)

- **OS:** Ubuntu 20.04+ (Linux) / Windows 10+ (FSDS Simulator)
- **Docker:** 20.10+
- **Docker Compose:** 1.29+
- **Python:** 3.8+
- **ROS:** Noetic (Linux 환경)

### 1. 저장소 복제 (Clone)

```bash
git clone https://github.com/qws941/HYCU-FSDS.git
cd HYCU-FSDS
```

### 2. Docker 기반 실행 (Recommended)

```bash
# 전체 자동주행 시스템 실행
cd submission
docker-compose up --build

# 또는 자율주행 모듈만 실행
cd autonomous
docker-compose up --build
```

### 3. 로컬 개발 환경 설정

```bash
# Python 가상환경 생성
python3 -m venv venv
source venv/bin/activate  # Linux
# or
.\venv\Scripts\activate   # Windows

# 의존성 설치
pip install -r requirements.txt
```

---

## 로컬 개발 (Local Development)

### 개발 환경 설정

```bash
# submission 디렉토리에서 개발
cd submission

# Python 패키지 설치 (개발 모드)
pip install -e .

# 테스트 실행
python -m pytest tests/

# 개별 모듈 실행
python scripts/competition_driver.py
```

### Docker 환경에서 개발

```bash
# development.sh 실행
cd submission
chmod +x dev.sh
./dev.sh
```

### 테스트

```bash
# 단위 테스트 실행
cd submission
python -m pytest tests/test_algorithms.py -v

# 특정 모듈 테스트
python -c "from src.perception import cone_detector; print('OK')"
```

---

## 명령어 참고서 (Commands Reference)

### Docker 명령어

| 명령어 | 설명 |
|--------|------|
| `docker-compose up --build` | 전체 시스템 빌드 및 실행 |
| `docker-compose down` | 컨테이너 중지 및 정리 |
| `docker-compose logs -f` | 로그 실시간 확인 |
| `docker exec -it <container> bash` | 컨테이너 내부 접속 |

### Python 스크립트

| 스크립트 | 설명 |
|---------|------|
| `scripts/simple_slam.py` | SLAM 모듈 실행 |
| `scripts/fsds_driver.py` | FSDS 드라이버 실행 |
| `scripts/competition_driver.py` | 대회용 드라이버 실행 |
| `scripts/advanced_driver.py` | 고급 드라이버 실행 |

### ROS 명령어

| 명령어 | 설명 |
|--------|------|
| `roslaunch submission competition.launch` | 대회 런치파일 실행 |
| `rosrun rqt_graph rqt_graph` | 노드 그래프 확인 |
| `rostopic list` | 토픽 목록 확인 |
| `rostopic echo /cone_detections` | 콘 감지 토픽 구독 |

---

## 기여 가이드 (Contribution)

### 브랜치 전략

```
main (production)
├── develop (integration)
│   ├── feature/xxx
│   ├── fix/xxx
│   └── docs/xxx
└── release/x.x.x
```

### 커밋 메시지 규약

```
<type>(<scope>): <subject>

Types: feat, fix, docs, style, refactor, test, chore
```

**예시:**

```
feat(perception): add cone classification model
fix(control): correct pure pursuit calculation
docs(readme): update installation guide
```

### Pull Request 과정

1. **브랜치 생성:** `git checkout -b feature/your-feature`
2. **변경사항 커밋:** `git commit -m "feat(scope): description"`
3. **푸시:** `git push origin feature/your-feature`
4. **PR 생성:** GitHub에서 PR 생성 (자동 워크플로우가 실행됩니다)
5. **리뷰 요청:** 유지관리자에게 리뷰 요청

### 자동화 워크플로우 가이드

| 워크플로우 | 언제 실행되는가 |
|-----------|----------------|
| `03_pr-checks.yml` | 모든 PR에 자동 실행 |
| `09_semantic-pr.yml` | PR 제목/본문 검증 |
| `10_pr-review.yml` | AI 기반 코드 리뷰 |
| `20_readme-gen.yml` | master 브랜치 푸시 시 |
| `05_gitleaks.yml` | PR/푸시 시 |
| `06_codeql.yml` | PR/푸시 시 |

### 이슈 리포팅

버그 발견 시:

1. 기존 이슈 검색
2. 없으면 새 이슈 생성 (`bug_report` 템플릿 사용)
3. 재현 방법, 환경 정보 포함

---

## 라이선스 (License)

MIT License - 자세한 내용은 [LICENSE](LICENSE) 파일을 참조하세요.

---

## 연락처 (Contact)

- **Website:** <https://bot.jclee.me>
- **Documentation:** <https://cliproxy.jclee.me>

---

*이 프로젝트는 GitHub Actions 자동화 워크플로우를 통해 지속적으로 관리되고 있습니다.*
*This project is continuously maintained through GitHub Actions automation workflows.*

```

---

**참고 (Notes):**

- `in-memoria.db` 파일은 프로젝트 히스토리 추적용으로 보입니다
- `_bot-scripts/` 디렉토리의 Docker 파일들은 GitHub App/Actions 자동화를 위한 것으로 사용자가 직접 실행할 필요가 없습니다
- 자동화 워크플로우 중 `14_bot-auto-fix.yml`는 PR-Agent를 활용하여 자동 코드 수정을 수행합니다
- 프로젝트의 모든 문서는 `20_readme-gen.yml` 워크플로우를 통해 master 브랜치 푸시 시 자동으로 생성/갱신됩니다
