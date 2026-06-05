# Pitfalls Research

**Domain:** Korean mdBook tutorial — Nous Research Hermes Agent (for developers)
**Researched:** 2026-06-05
**Confidence:** HIGH (A-section: grounded in official docs + GitHub issues + community reports; B-section: grounded in documentation best-practice research)

---

## Dual-Scope Overview

This file covers **two distinct pitfall categories**:

- **Part A — Hermes Agent Usage Pitfalls:** What readers of the tutorial will get wrong when installing/using Hermes Agent. These seed `주의 (caution)` callouts and troubleshooting sections in each chapter.
- **Part B — Tutorial-Writing Pitfalls:** What goes wrong when producing this kind of comprehensive developer tutorial. These are internal guard-rails for the authors/contributors of this mdBook.

---

# PART A: Hermes Agent Usage Pitfalls

## Critical Pitfalls (A)

### A-1: curl|bash 설치 후 PATH 미반영 — "hermes: command not found"

**What goes wrong:**
설치 스크립트는 `~/.local/bin`에 바이너리를 넣지만, 현재 열려 있는 셸 세션에는 PATH 변경이 반영되지 않는다. 독자는 설치가 성공했는데도 `hermes: command not found` 오류를 만난다.

**Why it happens:**
설치 스크립트는 `.bashrc` / `.zshrc`에 export 구문을 추가하지만, 현재 셸 프로세스는 이미 로드된 환경을 사용하고 있다.

**How to avoid:**
설치 직후 반드시 `source ~/.bashrc` 또는 `source ~/.zshrc`를 실행하거나 새 터미널 탭을 열도록 튜토리얼에 명시한다. `hermes doctor`로 설치 상태를 진단할 수 있음을 안내한다.

**Warning signs:**
- `which hermes` 가 아무것도 출력하지 않음
- `$PATH`에 `~/.local/bin`이 없음

**Phase to address:**
설치 챕터 (1장 설치 및 초기 설정) — 설치 커맨드 바로 다음 단계로 반드시 포함.

---

### A-2: sudo로 설치 — 권한 충돌

**What goes wrong:**
독자가 `sudo`로 설치 스크립트를 실행하면 바이너리가 `/usr/local/bin`에 설치되고, 이후 `~/.local/bin`에 다시 설치하면 두 경로 모두에 버전이 존재하여 예상치 못한 버전이 실행된다. 또한 설정 파일이 root 소유가 되어 일반 사용자로 실행 시 권한 오류가 발생한다.

**Why it happens:**
리눅스 설치 관행상 시스템 도구는 sudo로 설치하는 습관이 있는 개발자들이 반사적으로 sudo를 붙인다.

**How to avoid:**
튜토리얼 내 설치 커맨드에 `sudo 없이` 실행해야 한다는 것을 굵은 글씨나 경고 박스로 강조한다. 이미 sudo로 설치한 경우 복구 방법(`sudo rm /usr/local/bin/hermes` 후 재설치)도 제공한다.

**Warning signs:**
- `hermes` 실행 시 "Permission denied" 오류
- `ls -la $(which hermes)`가 root 소유를 보여줌

**Phase to address:**
설치 챕터 — 설치 커맨드 직전 경고 callout.

---

### A-3: Windows에서 네이티브 실행 시도 — 대시보드 기능 누락

**What goes wrong:**
Windows 사용자가 PowerShell 설치 방법으로 설치하면 CLI와 게이트웨이는 작동하지만, 브라우저 대시보드의 채팅 창이 작동하지 않는다. POSIX PTY(의사 터미널)가 필요한 기능이 제한된다.

**Why it happens:**
공식 문서에는 "Windows: WSL2 필요"라고 적혀 있지만, PowerShell 설치 방법이 먼저 나오기 때문에 독자가 전체 기능을 기대하고 실망한다.

**How to avoid:**
튜토리얼 OS 선택 섹션에서 Windows 독자에게 WSL2 환경을 권장하고, 네이티브 PowerShell 설치는 "제한된 기능"임을 명확히 표시한다. WSL2 설치 가이드 링크를 제공한다.

**Warning signs:**
- 채팅 창이 빈 화면으로 로드됨
- "PTY error" 또는 "terminal backend unavailable" 메시지

**Phase to address:**
설치 챕터 OS 선택 섹션, Windows/WSL2 별도 트랙 분기.

---

### A-4: API 키를 config.yaml에 저장 — 보안 및 동작 이상

**What goes wrong:**
독자가 API 키를 `~/.hermes/.env` 대신 `~/.hermes/config.yaml`에 직접 입력한다. 기능은 작동할 수 있지만 git으로 관리되는 디렉터리에 키가 노출될 위험이 있고, `hermes config set KEY VAL` 명령이 올바른 파일(`.env`)에 라우팅하는 것을 우회하게 된다.

**Why it happens:**
개발자들은 설정 파일을 하나로 통합하려는 경향이 있다. config.yaml이 더 구조화되어 있어 모든 설정을 넣으려 한다.

**How to avoid:**
설정 챕터에서 파일 분리 원칙을 명확히 설명: "비밀 = `.env`, 나머지 = `config.yaml`". `hermes config set API_KEY sk-...` 명령이 자동으로 `.env`에 라우팅됨을 보여준다.

**Warning signs:**
- `git diff ~/.hermes/` 에서 API 키가 보임
- `hermes setup` 재실행 시 키가 이미 있다고 나오지만 작동하지 않음

**Phase to address:**
설정 챕터 — 파일 구조 설명 섹션.

---

### A-5: hermes setup의 무음 API 키 건너뜀 (Silent Skip)

**What goes wrong:**
`~/.hermes/.env`에 해당 공급자의 키 항목이 이미 존재하면(값이 유효하지 않아도) `hermes setup` 및 `hermes model`의 API 키 프롬프트가 조용히 건너뛰어진다. 잘못된 키를 입력한 독자는 녹색 체크마크를 보고 성공했다고 착각하지만, 실제 API 호출은 계속 실패한다.

**Why it happens:**
설정 마법사는 값의 존재 여부만 확인하고 유효성 검사를 하지 않는다 (GitHub Issue #16394).

**How to avoid:**
키 입력 오류 발생 시 복구 방법을 튜토리얼에 포함: `~/.hermes/.env`를 직접 편집하여 잘못된 키 줄을 삭제한 후 `hermes setup` 재실행. 또는 `hermes config set PROVIDER_API_KEY NEW_KEY`로 덮어쓰기.

**Warning signs:**
- `hermes setup` 완료 후에도 "Invalid API key" 또는 "Authentication failed" 오류
- 모든 설정에 녹색 체크마크가 표시되지만 채팅이 작동하지 않음

**Phase to address:**
API 키 설정 챕터, 트러블슈팅 챕터.

---

### A-6: Anthropic Claude API — 구독 계정(Pro/Max)으로 Hermes 사용 불가

**What goes wrong:**
2026년 4월 기준, Anthropic은 Claude Pro/Max 구독 계정에서 서드파티 도구(Hermes 포함)의 API 접근을 차단했다. 독자가 Claude를 사용하려면 별도 Anthropic API 계정과 사용량 기반 결제가 필요하다.

**Why it happens:**
Claude 구독(claude.ai)과 Anthropic API는 별개 제품이다. 많은 개발자가 "Claude 구독이 있으면 API도 쓸 수 있다"고 오해한다.

**How to avoid:**
튜토리얼에서 Claude 공급자 선택 시 명확히 설명: "Claude Pro/Max 구독 ≠ API 접근". [console.anthropic.com](https://console.anthropic.com)에서 별도 API 키 발급 필요. 대안으로 OpenRouter를 통한 Claude 모델 접근도 안내한다.

**Warning signs:**
- "Forbidden" 또는 "401 Unauthorized" 오류와 함께 구독 관련 메시지
- API 키가 `sk-ant-`로 시작하지만 Hermes가 거부

**Phase to address:**
공급자 설정 챕터 — Claude 섹션에 경고 박스.

---

### A-7: Nous Portal Tool Gateway — 무료 계정과 유료 구독 혼동

**What goes wrong:**
Nous Portal 무료 계정은 추론(inference)에는 사용 가능하지만, Tool Gateway(웹 검색, 이미지 생성, 브라우저 자동화 등)는 유료 구독에서만 제공된다. 독자가 `hermes setup --portal`로 연결은 성공했지만 도구가 작동하지 않아 혼란을 겪는다.

**Why it happens:**
"Nous Portal"이라는 단일 진입점 뒤에 무료/유료 기능이 섞여 있어 경계가 불명확하다.

**How to avoid:**
Nous Portal 챕터에서 무료 vs. 유료 기능 표를 제공한다. 구독 만료 시 Tool Gateway 도구가 조용히 실패한다는 것도 안내한다(`use_gateway: false`로 설정하거나 직접 API 키로 폴백).

**Warning signs:**
- `hermes tools`에서 특정 도구가 "unavailable" 또는 "subscription required"
- 웹 검색이나 이미지 생성 호출 시 오류

**Phase to address:**
공급자 설정 챕터 — Nous Portal 섹션, 도구 설정 챕터.

---

### A-8: 최소 컨텍스트 윈도우 미충족 — 64K 토큰 미만 모델

**What goes wrong:**
컨텍스트 윈도우가 64,000 토큰 미만인 모델(무료 또는 저비용 모델)을 선택하면 Hermes가 올바르게 작동하지 않는다. 도구 정의만 약 8,759 토큰 + 시스템 프롬프트 5,176 토큰 = 약 13,935 토큰의 고정 오버헤드가 발생하기 때문에, 작은 컨텍스트 모델에서는 실제 대화에 사용 가능한 토큰이 극도로 제한된다.

**Why it happens:**
독자가 비용 절감을 위해 무료/소형 모델을 선택하거나, 공급자 모델 목록에서 작은 컨텍스트 모델을 고르는 경우.

**How to avoid:**
모델 선택 챕터에서 "최소 64K 토큰 컨텍스트 윈도우" 요구사항을 강조하고 추천 모델 목록을 제공한다. 고정 오버헤드 (~14K)가 존재함을 간략히 설명한다.

**Warning signs:**
- 짧은 대화에서도 "Context length exceeded" 오류
- 메시지 응답 중 갑자기 끊김

**Phase to address:**
모델 선택 챕터.

---

## Moderate Pitfalls (A)

### A-9: 게이트웨이 토큰 비용 급증 — 컨텍스트 압축 미이해

**What goes wrong:**
Telegram/Discord 게이트웨이를 통한 약 2시간의 가벼운 사용으로 약 400만 입력 토큰이 소비되어 Claude Sonnet 기준 약 $12의 비용이 발생한다는 보고가 있다. 기본 압축 임계값이 50%이므로, 컨텍스트의 절반이 찰 때까지 압축이 시작되지 않는다. 또한 각 API 호출마다 ~13,935 토큰의 고정 오버헤드가 추가된다.

**Prevention:**
토큰 비용 챕터에서 `/usage`로 사용량 확인 방법, `/compress`로 수동 압축, `compression.threshold`를 낮추는 방법(예: 0.3), 그리고 게이트웨이 사용 시 저렴한 보조 모델(예: Gemini Flash)을 압축 요약 모델로 설정하는 방법을 안내한다.

**Phase to address:**
비용 관리 챕터 또는 게이트웨이 챕터의 고급 설정 섹션.

---

### A-10: Discord 봇 — Privileged Gateway Intents 미설정

**What goes wrong:**
Discord 봇 설정 가이드 대부분이 "Privileged Gateway Intents" 활성화를 빠뜨린다. Hermes가 온라인 상태로 보이지만 모든 메시지를 무시한다. 특히 "Message Content Intent"를 활성화하지 않으면 봇이 채널 메시지를 읽지 못한다.

**Prevention:**
Discord 게이트웨이 챕터에서 Discord Developer Portal의 Bot 탭에서 다음을 활성화하는 단계를 스크린샷과 함께 포함:
- Server Members Intent
- Message Content Intent
활성화 후 게이트웨이 재시작 필요.

**Phase to address:**
Discord 게이트웨이 설정 챕터.

---

### A-11: Telegram 봇 토큰 — 동일 토큰 중복 폴링

**What goes wrong:**
동일한 Telegram 봇 토큰을 사용하는 두 개의 Hermes 프로세스(또는 두 프로필)가 실행되면, 하나의 프로세스만 각 메시지를 받는다. 결과적으로 메시지가 간헐적으로 처리되거나 완전히 무시된다.

**Prevention:**
게이트웨이 시작 전 `hermes gateway status`로 이미 실행 중인 인스턴스 확인. 봇 토큰은 프로필 간 공유 불가(공식 문서 명시). 로그 경로: `~/.hermes/logs/gateway.log`.

**Phase to address:**
Telegram 게이트웨이 챕터, 트러블슈팅 챕터.

---

### A-12: approvals.mode: off — 보안 경고 없이 명령 실행

**What goes wrong:**
`approvals.mode: off` (또는 구 방식인 `HERMES_YOLO_MODE=1`)을 설정하면 위험한 명령에 대한 모든 안전 프롬프트가 비활성화된다. 독자가 "편의를 위해" 이 설정을 켜고 그대로 프로덕션 환경에 두는 경우가 많다.

**Prevention:**
보안 챕터에서 이 설정의 위험성을 경고 박스로 강조. Docker/Modal/Daytona 백엔드를 사용할 경우 컨테이너 자체가 보안 경계가 되므로 승인 모드를 끄는 것이 합리적이나, 로컬 백엔드에서는 절대 권장하지 않음을 명시. 기본값(manual)을 그대로 유지하도록 권장.

**Phase to address:**
보안 챕터, 배포 챕터.

---

### A-13: Docker 백엔드 — root 소유 파일 생성

**What goes wrong:**
`docker_run_as_host_user` 설정 없이 Docker 백엔드를 사용하면 컨테이너 내에서 생성된 파일이 root 소유가 되어, 호스트에서 일반 사용자로 해당 파일을 편집하거나 삭제하려면 sudo가 필요하다.

**Prevention:**
Docker 챕터에서 `docker_run_as_host_user: true` 설정 방법을 안내. 단, 이 설정 시 컨테이너 내 패키지 설치(apt install 등)가 제한될 수 있음도 함께 안내.

**Phase to address:**
Docker 배포 챕터.

---

### A-14: 보조 모델 컨텍스트 윈도우 부족 — 무음 컨텍스트 손실

**What goes wrong:**
`auxiliary.compression.model`로 메인 모델보다 작은 컨텍스트 윈도우를 가진 요약 모델을 설정하면, 요약 과정에서 컨텍스트가 조용히 잘린다. 오류 없이 요약이 완료되지만 중요한 정보가 유실된다.

**Prevention:**
설정 챕터에서 요약 모델은 메인 모델과 동일하거나 더 큰 컨텍스트 윈도우가 필요하다는 규칙을 설명. 저비용 요약 모델(예: Gemini Flash)은 큰 컨텍스트 윈도우를 가진 모델로 선택.

**Phase to address:**
고급 설정 챕터 — 컨텍스트 압축 섹션.

---

### A-15: macOS launchd 게이트웨이 — Homebrew/nvm 도구 누락

**What goes wrong:**
macOS에서 `hermes gateway install`로 launchd 서비스를 설치한 후 Homebrew나 nvm으로 새 도구를 설치하면, launchd는 기존 PATH를 사용하기 때문에 새 도구를 찾지 못한다. 인터랙티브 셸에서는 작동하는 도구가 게이트웨이에서는 실패한다.

**Prevention:**
macOS 게이트웨이 챕터에서 새 도구 설치 후 `hermes gateway install`을 재실행하여 PATH를 갱신해야 함을 안내.

**Phase to address:**
macOS 배포 챕터, 게이트웨이 트러블슈팅 챕터.

---

### A-16: /model은 이미 설정된 공급자 사이에서만 전환

**What goes wrong:**
채팅 세션 내 `/model` 명령은 새 공급자를 추가하지 못하고 이미 설정된 공급자들 사이에서만 전환한다. 새 공급자를 추가하려면 세션을 종료하고 터미널에서 `hermes model`을 실행해야 한다.

**Prevention:**
모델 선택 챕터에서 `/model` (세션 내)과 `hermes model` (터미널)의 차이를 명확히 설명.

**Phase to address:**
모델 선택 챕터, 빠른 참조 섹션.

---

### A-17: Termux — 표준 설치 스크립트 사용 불가

**What goes wrong:**
Android Termux에서 표준 curl|bash 설치 방법을 시도하면 음성(voice) 의존성 때문에 설치가 실패한다. Termux에는 `.[termux]` 추가 패키지를 사용한 별도 설치 경로가 필요하다.

**Prevention:**
설치 챕터에서 Termux를 별도 탭 또는 섹션으로 분리. 공식 문서의 Termux 매뉴얼 설치 경로로 안내.

**Phase to address:**
설치 챕터 — 플랫폼 선택 섹션.

---

## Minor Pitfalls (A)

### A-18: GitHub 접근 차단 환경에서 설치 실패

**What goes wrong:**
기업 방화벽이나 특정 국가/지역에서 `raw.githubusercontent.com` 접근이 차단되어 설치 스크립트 다운로드가 실패한다.

**Prevention:**
트러블슈팅 챕터에 대안 안내: GitHub 미러 설정 또는 GitHub 릴리스 페이지에서 수동 다운로드 후 로컬 실행.

**Phase to address:**
설치 챕터 트러블슈팅 섹션.

---

### A-19: config.yaml의 변수 치환 미사용 — ${VAR_NAME} 문법

**What goes wrong:**
독자가 `config.yaml`에서 환경변수를 참조할 때 `$VAR_NAME` 문법을 사용하면 치환이 되지 않고 문자열 그대로 사용된다. `${VAR_NAME}`(중괄호 필수)을 사용해야 한다.

**Prevention:**
설정 챕터에서 올바른 문법 예시 제공.

**Phase to address:**
고급 설정 챕터.

---

### A-20: cron 작업 디렉터리 미설정 — 예측 불가능한 파일 경로

**What goes wrong:**
cron 기반 예약 작업에 `terminal.cwd`를 명시하지 않으면 작업 디렉터리가 Hermes 설치 디렉터리가 되어 파일 생성/읽기 경로가 예상과 다를 수 있다.

**Prevention:**
cron/자동화 챕터에서 각 cron 작업에 `workdir` 명시를 권장.

**Phase to address:**
자동화/cron 챕터.

---

---

# PART B: Tutorial-Writing Pitfalls

## Critical Pitfalls (B)

### B-1: 검증되지 않은 명령어 문서화 — 가장 큰 신뢰 파괴자

**What goes wrong:**
튜토리얼에 실제로 실행해보지 않은 커맨드, 존재하지 않는 옵션, 오래된 플래그를 그대로 게시한다. 독자가 복사-붙여넣기하면 오류가 발생하고 신뢰를 잃는다. 이 프로젝트의 **최대 리스크**: Hermes Agent가 활발히 개발 중이므로 CLI 인터페이스와 설정 키가 릴리스마다 변경된다.

**Why it happens:**
- LLM 지원 작성 시 환각된 커맨드 생성
- 초기 버전에서 작성 후 버전 업 시 미검증
- OS별 차이 미고려

**How to avoid:**
- 모든 코드 블록의 커맨드는 실제 환경에서 직접 실행하여 검증
- 코드 블록 상단 주석에 검증 날짜와 버전 기록: `# 검증: hermes v0.15, 2026-05-01`
- CI에서 자동 검증 가능한 커맨드는 자동화
- LLM으로 내용 생성 시 모든 커맨드를 별도로 실제 환경 검증

**Phase to address:**
모든 챕터 작성 단계 — "커맨드 검증" 체크리스트를 각 PR 요구사항으로 포함.

---

### B-2: 버전 드리프트 — 작성 후 구식이 되는 문서

**What goes wrong:**
튜토리얼 게시 후 Hermes Agent가 업데이트되면 커맨드, 설정 키, UI가 변경되지만 튜토리얼은 그대로다. 독자는 구식 정보를 따라 시간을 낭비한다.

**Why it happens:**
Hermes Agent는 활발히 개발 중이며 릴리스 주기가 빠르다 (`hermes-agent/releases` 참조). mdBook은 정적 사이트라 콘텐츠 만료를 자동으로 감지하지 않는다.

**How to avoid:**
- 각 챕터 상단에 "이 챕터 기준 버전: vX.Y.Z" 표시
- 변경 가능성이 높은 항목(설정 키 이름, 모델 이름, URL)은 주석으로 "버전 의존"이라 표시
- GitHub Pages 배포 CI에 Hermes 최신 릴리스와의 버전 비교 단계 추가
- 챕터별 "마지막 검토일" 메타데이터 유지

**Phase to address:**
모든 챕터 — 버전 태그는 콘텐츠 작성 직전에 기록, 릴리스 시 검토.

---

### B-3: 사전 요구사항 누락 — "왜 안 되지?" 페이지

**What goes wrong:**
독자가 튜토리얼 중간에서 "Python 3.11 필요", "Git 설치 필요", "Docker 데몬 실행 중이어야 함" 같은 요구사항을 처음 만난다. 처음부터 알았다면 미리 준비했을 것을 뒤늦게 알게 되어 시간을 낭비한다.

**How to avoid:**
각 챕터 앞에 "이 챕터를 시작하기 전에 (Before You Begin)" 체크리스트 포함. 설치 챕터에는 전체 사전 요구사항 목록을 게시(OS, Python 버전, Node.js, Docker 여부, 네트워크 요구사항).

**Phase to address:**
설치 챕터 서두, 각 실습 챕터 서두.

---

### B-4: 예상 출력 없음 — 독자가 성공 여부를 모름

**What goes wrong:**
커맨드 실행 후 어떤 출력이 나와야 하는지 보여주지 않으면 독자는 "이게 맞게 된 건지"를 알 수 없다. 특히 설치, API 연결, 게이트웨이 시작 단계에서 중요하다.

**How to avoid:**
주요 커맨드 다음에 "예상 출력 (Expected Output)" 코드 블록 추가. 출력이 환경마다 다를 수 있는 부분(타임스탬프, 경로 등)은 `[...]`으로 표시.

**Phase to address:**
설치 챕터, 설정 챕터, 게이트웨이 챕터 — 각 검증 단계마다.

---

### B-5: 트러블슈팅 섹션 부재 — 막히면 포기

**What goes wrong:**
성공 경로만 기록하고 오류 시나리오를 다루지 않으면, 독자는 일반적인 오류(위 A섹션의 모든 항목)를 만날 때 다른 곳을 찾아야 한다. 공식 FAQ가 영어만 제공되므로 한국어 트러블슈팅 가이드의 가치가 특히 크다.

**How to avoid:**
각 챕터 끝 또는 별도 "트러블슈팅" 챕터에서 해당 챕터에서 발생 가능한 오류와 해결책을 제공. 오류 메시지를 직접 검색 가능하도록 원문 그대로 포함.

**Phase to address:**
모든 실습 챕터, 별도 트러블슈팅 레퍼런스 챕터.

---

## Moderate Pitfalls (B)

### B-6: 단일 OS 가정 — macOS에서만 테스트된 튜토리얼

**What goes wrong:**
저자가 macOS에서만 튜토리얼을 작성/검증하면, Linux (Ubuntu/Debian), Windows (WSL2), Termux 독자가 OS별 차이로 막힌다. 예: Homebrew 경로(`/opt/homebrew` vs `/usr/local`), 셸 설정 파일명(`.zshrc` vs `.bashrc`), 패키지 매니저 차이.

**How to avoid:**
mdBook 탭 기능(`{{#tabs}}`)을 활용하여 OS별 커맨드를 탭으로 분리. 최소한 Linux와 macOS를 병행 검증.

**Phase to address:**
설치 챕터부터 OS 분기 구조 확립.

---

### B-7: 커맨드에 설명 없음 — 복붙 자동기계 양산

**What goes wrong:**
독자가 커맨드의 의미를 이해하지 못하고 복사-붙여넣기만 하면, 환경이 조금 달라지거나 오류가 발생했을 때 적응하지 못한다. 또한 보안 관련 커맨드(sudo, curl|bash)의 위험성을 인지하지 못한다.

**How to avoid:**
커맨드 블록 앞뒤에 간결한 설명 추가. 특히 플래그 의미 설명(`-fsSL`: follow redirect, silent, show error, location). `curl|bash` 신뢰 문제도 직접 언급하고 소스 확인 방법 안내.

**Phase to address:**
설치 챕터 (curl|bash 설명), 이후 각 챕터.

---

### B-8: 버전 미지정 링크 — 랜딩 페이지 변경 시 링크 파괴

**What goes wrong:**
공식 문서, GitHub, API 레퍼런스로의 링크가 특정 버전 앵커 없이 최신 페이지를 가리키면, 해당 페이지 구조가 바뀔 때 독자가 다른 내용을 보게 된다.

**How to avoid:**
가능한 경우 버전 태그가 있는 URL 사용 (예: GitHub에서 특정 태그/커밋 링크). 변경 가능성이 높은 외부 링크에는 "작성 시점 기준 링크" 주석 추가.

**Phase to address:**
모든 챕터 — 링크 추가 시점에 버전 기록.

---

### B-9: 중급 독자 공백 (Tutorial Cliff)

**What goes wrong:**
설치 + 첫 채팅(101)에서 바로 고급 배포/cron/subagent(400)로 점프하면 중급 독자(실제 업무에 쓰려는 개발자)가 따라가지 못한다.

**How to avoid:**
학습 경로를 명시: 기초(설치→첫 채팅→기본 설정) → 중급(게이트웨이→도구→메모리/스킬) → 고급(배포→비용최적화→자동화). 각 챕터에 "다음 단계" 링크 포함.

**Phase to address:**
목차(SUMMARY.md) 구성 단계, 서론 챕터.

---

### B-10: mdBook GitHub Pages CI — 오래된 Actions 버전

**What goes wrong:**
2025년 3월 GitHub이 일부 Action을 deprecated하여 기존 mdBook 배포 워크플로우 파일이 실패할 수 있다. `actions/deploy-pages@v1`, `actions/upload-artifact@v2` 등이 영향을 받는다.

**How to avoid:**
CI 워크플로우 작성 시 최신 stable 버전의 Actions 사용 확인. Dependabot으로 Actions 버전 자동 업데이트 설정 권장. mdBook 공식 CI 문서의 예시를 참조하되, GitHub Marketplace에서 최신 버전 확인.

**Phase to address:**
GitHub Pages 배포 챕터.

---

### B-11: mdBook SUMMARY.md 중복 항목 — 빌드 실패

**What goes wrong:**
`SUMMARY.md`에 동일한 파일 경로가 두 번 등록되면 mdBook 빌드가 실패한다. 챕터 추가/재구성 시 흔히 발생.

**How to avoid:**
챕터 추가/삭제 시 `SUMMARY.md` 중복 검사. CI 파이프라인에 `mdbook build` 단계를 포함하여 빌드 실패를 조기 감지.

**Phase to address:**
GitHub Pages 배포 챕터, 기여 가이드.

---

### B-12: site-url 미설정 — 서브경로 배포 시 CSS/JS 누락

**What goes wrong:**
GitHub Pages에서 커스텀 도메인 없이 배포하면 URL이 `https://username.github.io/hermes-tutorial/` 형태가 된다. `book.toml`에 `site-url`을 설정하지 않으면 404 페이지와 일부 정적 파일이 잘못된 경로를 참조한다.

**How to avoid:**
`book.toml`에 다음 설정 추가:
```toml
[output.html]
site-url = "/hermes-tutorial/"
```

**Phase to address:**
GitHub Pages 배포 챕터.

---

---

## Technical Debt Patterns

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|----------|-------------------|----------------|-----------------|
| 커맨드 검증 없이 게시 | 작성 속도 향상 | 독자 신뢰 파괴, 이슈 폭증 | Never |
| OS 분기 없이 macOS만 테스트 | 작성 단순화 | Linux/WSL2 독자 이탈 | MVP 단계에서만, 반드시 후속 보완 |
| 버전 태그 없이 링크 삽입 | 링크 추가 속도 | 링크 파괴, 구식 정보 | Never (GitHub 링크는 항상 태그/SHA 사용) |
| 트러블슈팅 섹션 생략 | 챕터 길이 단축 | 독자 이탈, 지원 요청 증가 | Never (최소 3개 일반 오류는 포함) |
| 예상 출력 없이 커맨드 제시 | 작성 단순화 | "성공인지 모르겠음" 독자 경험 | 출력이 완전히 환경 종속인 경우에만 생략 가능 |

---

## Integration Gotchas

| Integration | Common Mistake | Correct Approach |
|-------------|----------------|------------------|
| OpenRouter | OpenAI 키로 OpenRouter 엔드포인트 접근 | 공급자별 키는 별개; OpenRouter 전용 키 발급 |
| Anthropic API | Claude Pro/Max 구독 = API 접근이라 오해 | console.anthropic.com에서 별도 API 키 발급 |
| Telegram BotFather | 토큰에 포함된 콜론(`:`)이 env 파서에서 오파싱 | `.env`에 따옴표 없이 그대로 저장 |
| Discord Developer Portal | Message Content Intent 미활성화 | Bot 탭에서 Privileged Gateway Intents 3종 확인 |
| Nous Portal Tool Gateway | 무료 계정으로 도구 사용 시도 | 유료 구독 필요, 대안으로 직접 API 키 사용 |
| Docker 백엔드 | Docker 데몬 미실행 상태에서 Hermes 시작 | `docker info`로 데몬 상태 확인 후 시작 |
| Modal 백엔드 | Tool Gateway에 포함된다고 오해 | Modal은 별도 add-on, 독립 설정 필요 |

---

## Performance Traps

| Trap | Symptoms | Prevention | When It Breaks |
|------|----------|------------|----------------|
| 고정 토큰 오버헤드 (13.9K/call) | 짧은 대화에서도 예상보다 높은 비용 | 불필요한 도구셋 비활성화 | 소형 컨텍스트 모델 (<64K) 사용 시 즉시 |
| 게이트웨이 무제한 세션 축적 | 메모리 사용 증가, 느린 시작 | `hermes sessions clean` 정기 실행 | 수백 시간 사용 후 |
| 압축 임계값 50% 기본값 | 비용 급증 | `compression.threshold: 0.3`으로 낮추기 | 장기 대화/게이트웨이 사용 시 |
| 스트리밍 + 프롬프트 캐싱 동시 사용 | 캐싱 효과 없음, 비용 증가 | 둘 중 하나 선택 | 항상 (둘은 상호 배타적) |

---

## Security Mistakes

| Mistake | Risk | Prevention |
|---------|------|------------|
| `approvals.mode: off` 로컬 환경에서 사용 | 악의적 프롬프트가 시스템 커맨드 실행 | 로컬에서는 `manual` 유지, Docker 백엔드에서만 off 고려 |
| `GATEWAY_ALLOW_ALL_USERS=true` 프로덕션 사용 | 모든 사용자가 에이전트에 접근 | allowlist 또는 DM 페어링 사용 |
| `docker_forward_env`에 비밀 키 포함 | 컨테이너 내 코드가 키 탈취 가능 | 필요한 변수만 선택적으로 포함 |
| gateway를 root 계정으로 실행 | 에이전트 탈출 시 전체 시스템 장악 | 전용 비권한 계정으로 실행 |
| Telegram 봇 토큰을 스크린샷/공개 이슈에 노출 | 토큰 탈취, 봇 하이재킹 | 즉시 BotFather에서 토큰 재발급 |

---

## "Looks Done But Isn't" Checklist

- [ ] **설치 완료:** `hermes` 실행 시 환영 메시지가 나오는가? — `hermes --version`으로 확인
- [ ] **API 연결:** 실제 채팅 응답이 오는가? — `hermes doctor`로 확인
- [ ] **게이트웨이 시작:** 봇에 DM을 보내 응답이 오는가? — 로그 `~/.hermes/logs/gateway.log` 확인
- [ ] **Discord Intents:** Message Content Intent 활성화 여부 — Discord Developer Portal에서 확인
- [ ] **비용 모니터링:** `/usage` 커맨드로 현재 세션 토큰 사용량 확인 가능한가?
- [ ] **GitHub Pages 배포:** 사이트가 올바른 URL(`/hermes-tutorial/`)로 접근 가능한가?
- [ ] **site-url 설정:** 404 페이지에서 CSS가 로드되는가?

---

## Recovery Strategies

| Pitfall | Recovery Cost | Recovery Steps |
|---------|---------------|----------------|
| sudo 설치로 인한 권한 충돌 | LOW | `sudo rm /usr/local/bin/hermes` → 표준 설치 재실행 |
| 잘못된 API 키 silent skip | LOW | `~/.hermes/.env`에서 해당 키 줄 삭제 → `hermes setup` 재실행 |
| Discord 봇 무반응 | LOW | Developer Portal에서 Intents 활성화 → 게이트웨이 재시작 |
| config.yaml에 API 키 노출 | MEDIUM | 키 즉시 무효화 → 새 키 발급 → `.env`로 이동 → git history 정리 |
| 버전 드리프트로 커맨드 오류 | HIGH | 해당 챕터 전체 재검증 및 업데이트 |
| mdBook 빌드 실패 (SUMMARY.md 중복) | LOW | `SUMMARY.md` 중복 항목 제거 → 로컬 빌드 확인 |

---

## Pitfall-to-Phase Mapping

| Pitfall | Prevention Phase | Verification |
|---------|------------------|--------------|
| A-1: PATH 미반영 | 1장 설치 챕터 | `which hermes` 출력 예시 포함 |
| A-2: sudo 설치 | 1장 설치 챕터 (커맨드 직전) | 경고 callout 존재 여부 |
| A-3: Windows 네이티브 제한 | 1장 OS 선택 섹션 | WSL2 안내 링크 포함 여부 |
| A-4: API 키 wrong file | 2장 설정 챕터 | `.env` vs `config.yaml` 구분 설명 여부 |
| A-5: Silent skip | 2장 설정 챕터 + 트러블슈팅 | 복구 방법 포함 여부 |
| A-6: Claude API 구독 혼동 | 2장 공급자 설정 챕터 | Claude 섹션 경고 박스 |
| A-7: Tool Gateway 혼동 | 3장 Nous Portal 챕터 | 무료/유료 기능 표 포함 |
| A-8: 최소 컨텍스트 미충족 | 2장 모델 선택 챕터 | 64K 요구사항 명시 |
| A-9: 토큰 비용 급증 | 5장 비용 관리 챕터 | `/usage`, `/compress` 사용법 포함 |
| A-10: Discord Intents | 4장 Discord 게이트웨이 챕터 | 스크린샷 포함 단계 안내 |
| A-11: Telegram 중복 폴링 | 4장 Telegram 챕터 + 트러블슈팅 | 중복 프로세스 확인 방법 |
| A-12: approvals off | 6장 보안 챕터 | 경고 박스 포함 |
| A-13: Docker root 파일 | 6장 Docker 배포 챕터 | `docker_run_as_host_user` 설명 |
| A-14: 보조 모델 컨텍스트 부족 | 5장 고급 설정 챕터 | 요약 모델 선택 가이드 |
| A-15: macOS launchd PATH | 6장 macOS 배포 챕터 | 재설치 필요성 안내 |
| B-1: 미검증 커맨드 | 모든 챕터 (PR 체크리스트) | 커맨드 검증 날짜 주석 |
| B-2: 버전 드리프트 | 모든 챕터 (버전 태그) | 각 챕터 버전 표시 존재 여부 |
| B-3: 사전 요구사항 누락 | 각 챕터 서두 | "Before You Begin" 체크리스트 |
| B-4: 예상 출력 없음 | 주요 커맨드 포함 챕터 | 예상 출력 블록 존재 여부 |
| B-5: 트러블슈팅 부재 | 각 챕터 + 별도 레퍼런스 챕터 | 최소 3개 오류 시나리오 |
| B-10: 오래된 CI Actions | GitHub Pages 배포 챕터 | Actions 버전 최신 여부 |
| B-11: SUMMARY.md 중복 | mdBook 설정 챕터 | 로컬 빌드 CI 단계 |
| B-12: site-url 미설정 | GitHub Pages 배포 챕터 | `book.toml` 설정 확인 |

---

## Sources

- [Hermes Agent GitHub — NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent)
- [Hermes Agent 공식 문서](https://hermes-agent.nousresearch.com/docs/)
- [FAQ & Troubleshooting — 공식 문서](https://hermes-agent.nousresearch.com/docs/reference/faq)
- [Configuration — 공식 문서](https://hermes-agent.nousresearch.com/docs/user-guide/configuration)
- [Security — 공식 문서](https://hermes-agent.nousresearch.com/docs/user-guide/security)
- [Tool Gateway — 공식 문서](https://hermes-agent.nousresearch.com/docs/user-guide/features/tool-gateway)
- [Quickstart — 공식 문서](https://github.com/NousResearch/hermes-agent/blob/main/website/docs/getting-started/quickstart.md)
- [GitHub Issue #16394 — hermes setup silent API key skip](https://github.com/NousResearch/hermes-agent/issues/16394)
- [GitHub Issue #4379 — Token overhead analysis (73% fixed overhead ~13.9K tokens)](https://github.com/NousResearch/hermes-agent/issues/4379)
- [Hermes Agent Troubleshooting Guide — bswen.com](https://docs.bswen.com/blog/2026-04-21-hermes-agent-troubleshooting-guide/)
- [Hermes Agent Gateway Troubleshooting — hermes-agent.ai](https://hermes-agent.ai/blog/hermes-agent-gateway-troubleshooting)
- [Hermes Agent Discord Setup Guide — hermes-agent.ai](https://hermes-agent.ai/blog/hermes-agent-discord-setup)
- [Hermes Agent Telegram Setup — hermes-agent.ai](https://hermes-agent.ai/blog/hermes-agent-telegram-setup)
- [The 7 Deadly Sins of Developer Documentation — bekahhw.com](https://bekahhw.com/common-dev-content-pitfalls)
- [10 Common Developer Documentation Mistakes — document360.com](https://document360.com/blog/developer-documentation-mistakes/)
- [mdBook CI/CD 공식 문서 — rust-lang.github.io](https://rust-lang.github.io/mdBook/continuous-integration.html)
- [GitHub Pages 배포 설정 — GitHub Docs](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site)
- [Hermes Agent v2026.4.16 Tool Gateway 구독 추가 — fintechextra.com](https://fintechextra.com/news/hermes-agent-v2026416-adds-subscription-based-tool-gateway-access-126)

---
*Pitfalls research for: Korean mdBook Tutorial — Nous Research Hermes Agent*
*Researched: 2026-06-05*
