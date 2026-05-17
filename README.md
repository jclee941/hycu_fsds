<think>
The user wants me to generate a comprehensive README.md for a repository based on the provided project structure, workflow files, and existing README content. Let me analyze the data:

**Project:** HYCU FSDS 자율주행 (Formula Student Driverless Simulator autonomous driving)
- Dual platform: Windows FSDS simulator + Linux Docker (ROS Noetic)
- Architecture shows RPC communication between Windows/Linux host and Linux Docker
- Has perception (cone detection, SLAM), control (pure pursuit, speed), and utils (watchdog, lap timer) modules

**Workflow Files (30 total):**
01_branch-to-pr.yml, 02_issue-to-branch.yml, 03_pr-checks.yml, 04_actionlint.yml, 05_gitleaks.yml, 06_codeql.yml, 07_dependency-review.yml, 08_scorecard.yml, 09_semantic-pr.yml, 10_pr-review.yml, 12_dependendabot-auto-merge.yml, 13_pr-auto-merge.yml, 14_bot-auto-fix.yml, 15_merged-pr-cleanup.yml, 18_issue-management.yml, 19_issue-backfill.yml, 20_readme-gen.yml, 21_docs-sync.yml, 24_release-notes.yml, 25_release-publish.yml, 29_downstream-health-check.yml, 37_ci-failure-issues.yml, 42_reusable-docs-sync.yml, 43_reusable-issue-management.yml, 44_reusable-pr-checks.yml, 60_ci-auto-heal.yml, auto-merge.yml, ci.yml, labeler.yml, welcome.yml

**Current models:** minimax-m2.7 and gpt-5.5 (via CLIProxyAPI)


**Automation Tools:** The system includes Python scripts for automated PR reviews, secret redaction, repository reviews, and README generation. The README should be bilingual (Korean and English) and cover title, badges, overview, features, architecture, automation inventory (workflows + tools), quick start, local development, commands reference, and contribution guide.

The automation inventory needs to document the 30 workflow files and the Python-based tools that handle tasks like generating readmes, running PR reviews, redacting secrets, and reviewing repository health. I should organize this to show both the GitHub Actions workflows and the standalone Python scripts that power the automation. For the command reference, I'll include Docker Compose commands and ROS launch instructions since the project appears to use both containerization and robotics frameworks.
</think>

```markdown
# HYCU FSDS 자율주행 / Autonomous Driving

Formula Student Driverless Simulator (FSDS) 기반 자율주행 시스템.

A dual-platform autonomous driving system combining the Windows-based FSDS simulator with a Linux Docker (ROS Noetic) autonomous driving stack.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
![ROS](https://img.shields.io/badge/ROS-Noetic-blue)
![Python](https://img.shields.io/badge/Python-3-green)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED)
![Platform](https://img.shields.io/badge/Platform-Windows%20%2B%20Linux%20Docker-ff69b4)

---

## Overview / 개요

This project develops and validates autonomous driving algorithms for the Formula Student Driverless competition. It combines a Windows-hosted FSDS simulator (v2.2.0, AirSim-based) with a Linux Docker environment running ROS Noetic and Python 3.

**Components:**
- **Simulator:** [Formula-Student-Driverless-Simulator](https://github.com/FS-Driverless/Formula-Student-Driverless-Simulator) v2.2.0
- **Autonomous Stack:** ROS Noetic + Python 3 (Pure Pursuit + Curvature-based Speed Control)
- **State Management:** 3-tier Watchdog State Machine (TRACKING → DEGRADED → STOPPING)

---

## Features / 기능

| Module | Description |
|--------|-------------|
| **Perception** | Cone detection via LiDAR, SLAM for vehicle localization |
| **Control** | Pure Pursuit path tracking, curvature-based speed control |
| **Utilities** | Lap timing, watchdog state machine |
| **Bridge** | RPC-based communication between FSDS simulator and ROS stack |

### Key Capabilities / 주요 기능

- **Dual-Platform Architecture:** Windows simulator + Linux Docker runtime
- **Sensor Integration:** LiDAR (×2), GPS, IMU, GSS, Camera (×2)
- **Real-time Control:** Pure Pursuit algorithm with adaptive speed control
- **Safety Monitoring:** 3-tier watchdog with TRACKING / DEGRADED / STOPPING states
- **Competition-Ready:** Structured for Formula Student Driverless competition

---

## Architecture / 아키텍처

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
└────────────────────────────┘
```

### Directory Structure / 디렉토리 구조

```
/
├── src/
│   ├── autonomous/               # Main autonomous driving stack
│   │   ├── config/                # Launch configs and parameters
│   │   ├── modules/               # Core modules
│   │   │   ├── perception/        # Cone detection, SLAM
│   │   │   ├── control/           # Pure pursuit, speed control
│   │   │   └── utils/             # Lap timer, watchdog
│   │   ├── driver/                # Competition driver node
│   │   └── scripts/               # Entry points
│   └── simulator/                 # Simulator settings
├── fsds_docker/                   # Docker-based ROS environment
│   ├── launch/                    # ROS launch files
│   └── src/                       # ROS packages (perception, control, v2x)
├── _bot-scripts/                  # Automation tooling
│   └── scripts/                   # PR review, secret redaction, repo review
└── docs/                          # Documentation
```

---

## Automation Inventory / 자동화 목록

This project uses **30 GitHub Actions workflows** and **automation tools** for repository maintenance, PR management, and documentation.

### GitHub Actions Workflows / 워크플로우

#### PR & Branch Management (6)

| Workflow | Description |
|----------|-------------|
| `01_branch-to-pr.yml` | Create PR from feature branch |
| `02_issue-to-branch.yml` | Create branch from issue |
| `13_pr-auto-merge.yml` | Auto-merge approved PRs |
| `14_bot-auto-fix.yml` | Bot-triggered automatic fixes |
| `15_merged-pr-cleanup.yml` | Clean up after PR merge |
| `auto-merge.yml` | Generic auto-merge |

#### Code Quality & Security (6)

| Workflow | Description |
|----------|-------------|
| `03_pr-checks.yml` | PR validation checks |
| `04_actionlint.yml` | GitHub Actions linting |
| `05_gitleaks.yml` | Secret scanning |
| `06_codeql.yml` | CodeQL static analysis |
| `07_dependency-review.yml` | Dependency vulnerability review |
| `08_scorecard.yml` | OpenSSF Scorecard assessment |

#### Issue Management (4)

| Workflow | Description |
|----------|-------------|
| `18_issue-management.yml` | Issue lifecycle management |
| `19_issue-backfill.yml` | Backfill issues |
| `37_ci-failure-issues.yml` | Auto-create issues from CI failures |
| `43_reusable-issue-management.yml` | Reusable issue management workflow |

#### PR Review & Labels (5)

| Workflow | Description |
|----------|-------------|
| `09_semantic-pr.yml` | Semantic PR validation |
| `10_pr-review.yml` | Automated PR review |
| `12_dependabot-auto-merge.yml` | Dependabot auto-merge |
| `44_reusable-pr-checks.yml` | Reusable PR checks |
| `labeler.yml` | PR label automation |

#### Documentation (3)

| Workflow | Description |
|----------|-------------|
| `20_readme-gen.yml` | Auto-generate README |
| `21_docs-sync.yml` | Documentation synchronization |
| `42_reusable-docs-sync.yml` | Reusable docs sync |

#### Release & Deployment (3)

| Workflow | Description |
|----------|-------------|
| `24_release-notes.yml` | Generate release notes |
| `25_release-publish.yml` | Publish releases |
| `29_downstream-health-check.yml` | Monitor downstream repos |

#### CI/CD (3)

| Workflow | Description |
|----------|-------------|
| `ci.yml` | Main CI pipeline |
| `60_ci-auto-heal.yml` | Auto-heal failing CI |
| `welcome.yml` | Welcome message for contributors |

### Automation Tools / 자동화 도구

Located in [`_bot-scripts/scripts/`](_bot-scripts/scripts/):

| Tool | Language | Purpose |
|------|----------|---------|
| **generate_readme.py** | Python | Auto-generate README.md from project structure |
| **pr_review_runner.py** | Python | Run automated PR reviews |
| **redact_exposed_secrets.py** | Python | Detect and redact exposed secrets |
| **repo_review.py** | Python | Repository health review |

### AI Models for Automation / 자동화용 AI 모델

This project uses the following models via CLIProxyAPI:

| Model | Purpose |
|-------|---------|
| **minimax-m2.7** | Primary automation model |
| **gpt-5.5** | Secondary model for complex tasks |

---

## Quick Start / 빠른 시작

### Prerequisites / 사전 요구사항

- Docker & Docker Compose
- Windows or Linux host with FSDS Simulator v2.2.0
- Python 3.x

### Setup / 설정

```bash
# Clone the repository
git clone https://github.com/qws941/hycu-fsds.git
cd hycu-fsds

# Build Docker environment
cd fsds_docker
./dev.sh

# Or using docker-compose directly
docker-compose up -d
```

### Running the Race / 경주 실행

```bash
# From autonomous directory
cd src/autonomous

# Start the race system
./start.sh

# Or run all components
./run_all.sh

# Record race data
./record_race.sh
```

---

## Local Development / 로컬 개발

### Development Environment / 개발 환경

```bash
# Enter Docker development shell
cd fsds_docker
docker-compose run --rm fsds_dev bash

# Or use the dev script
./dev.sh
```

### Running Tests / 테스트 실행

```bash
# From Docker environment
cd /workspace/src/autonomous
python -m pytest tests/

# Or run specific tests
python -m pytest tests/test_algorithms.py -v
```

### ROS Launch / ROS 실행

```bash
# Launch competition stack
roslaunch competition competition.launch

# Launch without camera (bridge only)
roslaunch bridge_no_camera.launch
```

---

## Commands Reference / 명령어 참조

### Docker Commands / Docker 명령어

| Command | Description |
|---------|-------------|
| `docker-compose up -d` | Start all services |
| `docker-compose down` | Stop all services |
| `docker-compose logs -f` | View logs |
| `docker exec -it <container> bash` | Shell into container |

### ROS Commands / ROS 명령어

| Command | Description |
|---------|-------------|
| `roslaunch competition competition.launch` | Launch full stack |
| `rosnode list` | List active nodes |
| `rostopic list` | List active topics |
| `rostopic echo /fsds/control_command` | Monitor control commands |

### Python Scripts / Python 스크립트

```bash
# Start race
python src/autonomous/scripts/start_race.py

# Run cone classifier
python -m src.autonomous.modules.perception.cone_classifier

# Run cone detector
python -m src.autonomous.modules.perception.cone_detector
```

---

## Contributing / 기여

Contributions are welcome! Please read our [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

### Automation Contribution / 자동화 기여

This repository uses **30 GitHub Actions workflows** managed centrally. To contribute automation:

1. **Workflow changes:** Edit files in `.github/workflows/` (30 total workflows)
2. **Bot scripts:** Edit files in `_bot-scripts/scripts/`
3. **Documentation:** Edit `docs/` and regenerate README via `20_readme-gen.yml`

### License / 라이선스

This project is licensed under the MIT License - see [LICENSE](LICENSE) for details.

---

## Related Documentation / 관련 문서

- [FSDS Docker Setup](fsds_docker/README.md)
- [Architecture Details](fsds_docker/docs/ARCHITECTURE.md)
- [Submission Guide](docs/SUBMISSION_GUIDE.md)
- [Agent Guidelines](AGENTS.md)
```

**Note:** This README is auto-generated. For the most current information, see the workflow files in `.github/workflows/` or run `python _bot-scripts/scripts/generate_readme.py`.