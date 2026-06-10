# 컨텍스트 파일

> **이 챕터를 시작하기 전에**
> - [Ch.4 에이전트 루프](../04-agent-loop/index.md)를 완료하고 3티어 프롬프트 시스템(Stable / Context / Volatile)의 개념을 이해하고 있어야 한다.
> - 이 챕터는 **개념 + 실습** 구성이다. 에이전트가 프로젝트별 지시어를 어떻게 받아들이는지, 어떤 파일이 우선하는지 이해하는 것이 목표다.

---

## 개요

3티어 프롬프트 시스템에서 **Context 티어**는 프로젝트 수준의 지시어를 에이전트에 전달하는 슬롯이다. 이 슬롯에 주입되는 파일이 **컨텍스트 파일**이다.

컨텍스트 파일을 사용하면 에이전트가 새 세션을 시작할 때마다 "이 프로젝트는 Python 3.11을 사용하고, 커밋 메시지는 한국어로 작성한다"는 식의 프로젝트별 규칙을 자동으로 인식한다. 매번 직접 설명하지 않아도 된다.

이 챕터에서는 다음을 이해한다:

- `.hermes.md` → `AGENTS.md` → `CLAUDE.md` → `.cursorrules` 우선순위 캐스케이드
- SOUL.md의 역할과 컨텍스트 파일과의 차이
- 컨텍스트 파일의 처리 규칙(20,000자 한도, 보안 스캔, YAML frontmatter 제거)
- `/personality` 명령으로 세션 페르소나 변경
- `.env`(비밀)와 `config.yaml`(설정) 분리와 `${VAR}` 치환 문법

---

## 개념: 컨텍스트 파일 우선순위 캐스케이드

Hermes는 세션 시작 시 프로젝트 디렉터리에서 아래 순서로 파일을 탐색하고, **첫 번째로 발견된 파일 하나만** Context 티어에 로드한다. 두 개 이상의 파일이 있어도 첫 매치 이후 탐색을 중단한다.

| 우선순위 | 파일 이름 | 탐색 범위 | 용도 |
|----------|-----------|-----------|------|
| 1 | `.hermes.md` / `HERMES.md` | CWD → git root (상위 탐색) | Hermes 전용 프로젝트 지시어 |
| 2 | `AGENTS.md` | CWD 전용 | 일반 에이전트 지시어 |
| 3 | `CLAUDE.md` | CWD 전용 | Claude Code 호환 파일 |
| 4 | `.cursorrules` / `.cursor/rules/*.mdc` | CWD 전용 | Cursor IDE 호환 파일 |

**중요 차이점:** `.hermes.md`는 CWD에서 git root까지 **상위 디렉터리를 탐색**한다. 반면 `AGENTS.md`, `CLAUDE.md`, `.cursorrules`는 CWD 전용이다.

### 하위 디렉터리 AGENTS.md — 점진적 발견

루트의 컨텍스트 파일은 세션 시작 시 시스템 프롬프트에 로드된다. 에이전트가 툴 호출(`read_file`, `terminal`, `search_files` 등)로 하위 디렉터리로 진입하면, 해당 디렉터리의 `AGENTS.md`를 **lazy하게 발견**한다. 각 하위 디렉터리는 세션당 한 번만 확인하고, 부모 디렉터리 방향으로 상위 탐색도 한다. 이 방식은 시스템 프롬프트 팽창을 막고 프롬프트 캐시 안정성을 보존한다.

---

## 개념: SOUL.md — 전역 에이전트 정체성

**SOUL.md는 컨텍스트 파일과 다른 별개의 파일이다.** 혼동하기 쉬우므로 명확히 구분해야 한다.

| 항목 | SOUL.md | 컨텍스트 파일 |
|------|---------|--------------|
| 위치 | `~/.hermes/SOUL.md` (또는 `$HERMES_HOME/SOUL.md`) | 프로젝트 디렉터리 |
| 범위 | **전역** — 모든 프로젝트에서 동일한 페르소나 | 프로젝트별 |
| 로드 시점 | 레이어 1 (정체성 슬롯) — 모든 것보다 먼저 | 레이어 8 (컨텍스트 슬롯) |
| 부재 시 | 기본값 시드 | 로드 없음 |
| 수정 시 | 재시작 없이는 반영 안 됨 | 재시작 없이는 반영 안 됨 |

SOUL.md는 **Stable 티어 레이어 1**에 로드되어 에이전트의 기본 정체성을 정의한다. 파일이 없으면 기본값이 사용되고, 있으면 덮어쓰지 않는다. SOUL.md는 **프로젝트 디렉터리에 두지 않는다** — 항상 `~/.hermes/SOUL.md`다.

---

## 개념: 컨텍스트 파일 처리 규칙

로드된 컨텍스트 파일은 다음 처리 과정을 거쳐 시스템 프롬프트에 주입된다:

- **20,000자 한도:** 초과 시 앞부분 70% + 뒷부분 20%를 잘라 삽입(나머지 10% 중간 부분 제거)
- **보안 스캔:** `ignore previous instructions`나 `curl ... $API_KEY` 같은 패턴이 감지되면 주입이 차단됨
- **YAML frontmatter 제거:** `---` 블록으로 감싼 frontmatter는 주입 전 자동으로 제거됨
- **주입 위치:** `# Project Context` 헤더 아래 레이어 8 위치에 삽입됨

---

## 실습: .hermes.md 만들기

가장 권장하는 컨텍스트 파일 방식은 `.hermes.md`다. 최우선 탐색 순위를 가지고 git root까지 상위 탐색하므로 모노레포 환경에서도 유용하다.

**1단계: .hermes.md 파일 생성**

```bash
# 검증: hermes rolling, 2026-06-09
# 1. 프로젝트 디렉터리로 이동
cd ~/my-project

# 2. .hermes.md 파일 생성
cat > .hermes.md << 'EOF'
# My Project Instructions

이 프로젝트는 Python 3.11을 사용합니다.
모든 코드 변경 후 pytest를 실행하십시오.
커밋 메시지는 한국어로 작성하십시오.
EOF

# 3. hermes 실행 (새 세션)
hermes

# 결과: 에이전트가 프로젝트 지시어를 자동으로 인식하여 따릅니다.
# [hermes 시작 시 "Project context loaded" 또는 유사한 메시지 출력 — 로컬 캡처 필요]
```

> **검증 필요:** 컨텍스트 파일 로드를 알리는 정확한 배너 메시지(있다면)는 공개 문서에 명시되어 있지 않다. 로컬에서 `hermes`를 실행한 후 실제 배너 텍스트를 확인하고 위 placeholder를 교체해야 한다.

**SOUL.md 확인/편집**

```bash
# 검증: hermes rolling, 2026-06-09
# SOUL.md 현재 내용 확인
cat ~/.hermes/SOUL.md

# SOUL.md 편집 (기본 에디터)
$EDITOR ~/.hermes/SOUL.md
```

**현재 설정 확인**

```bash
# 검증: hermes rolling, 2026-06-09
hermes config show
```

---

## 개념: /personality 명령

`/personality`는 현재 세션의 에이전트 응답 스타일을 변경하는 슬래시 커맨드다.

**문법:**

```
/personality [name]
```

### 14개 내장 페르소나

| 이름 | 스타일 |
|------|--------|
| `helpful` | 친화적인 범용 어시스턴트 |
| `concise` | 간결하고 직관적인 응답 |
| `technical` | 상세하고 정확한 기술 전문가 |
| `creative` | 혁신적, 창의적 사고 |
| `teacher` | 예시로 설명하는 인내심 있는 교사 |
| `kawaii` | 귀여운 표현, 반짝임, 열정 |
| `catgirl` | 냥~하는 표현의 네코짱 |
| `pirate` | 기술에 밝은 해적 선장 Hermes |
| `shakespeare` | 극적인 문체의 시인 |
| `surfer` | 쿨한 서퍼 브로 바이브 |
| `noir` | 하드보일드 탐정 서술 |
| `uwu` | 최고조의 귀여움 + uwu 말투 |
| `philosopher` | 깊은 사색 |
| `hype` | 최대 에너지와 열정 |

**세션 스코프:** `/personality`는 세션이 종료되면 리셋된다. SOUL.md를 수정하지 않는다. 영구적인 스타일 변경은 SOUL.md 편집으로 해야 한다.

### 커스텀 페르소나 추가

`~/.hermes/config.yaml`에 `agent.personalities` 키로 커스텀 페르소나를 정의할 수 있다:

```yaml
# 검증: hermes rolling, 2026-06-09
agent:
  personalities:
    codereviewer: >
      You are a meticulous code reviewer. Identify bugs, security issues,
      performance concerns, and unclear design choices. Be precise and constructive.
```

전환: `/personality codereviewer`

---

## 개념: .env vs config.yaml 분리

Hermes는 설정을 **역할에 따라 두 파일로 엄격히 분리**한다.

### 설정 해석 우선순위 (높음 → 낮음)

1. CLI 인수 (`--model`, `--backend`)
2. `~/.hermes/config.yaml` (비밀 외 설정)
3. `~/.hermes/.env` (비밀)
4. 내장 기본값

### 파일별 저장 내용

| 파일 | 저장 내용 | 예시 |
|------|-----------|------|
| `config.yaml` | 비밀 외 설정 | 모델, 압축 임계값, 디스플레이, 백엔드 타입 |
| `.env` | **비밀만** | API 키, 봇 토큰, OAuth 자격증명 |
| `auth.json` | OAuth 자격증명 | OAuth refresh token |

### hermes config set 자동 라우팅

`hermes config set` 명령은 키의 성격에 따라 저장 위치를 자동으로 결정한다:

```bash
# 검증: hermes rolling, 2026-06-09
hermes config set model anthropic/claude-opus-4      # → config.yaml
hermes config set OPENROUTER_API_KEY sk-or-...       # → .env 자동 라우팅
hermes config set terminal.backend docker            # → config.yaml
```

### 환경변수 치환 문법

`config.yaml` 안에서 환경변수를 참조하려면 반드시 `${VAR_NAME}` 형식을 사용해야 한다:

```yaml
# 검증: hermes rolling, 2026-06-09
auxiliary:
  vision:
    api_key: ${GOOGLE_API_KEY}   # ← 중괄호 필수 (아래 주의 4 참조)
```

---

## 흔한 오류 / 주의

> **주의: 컨텍스트 파일은 세션 시작 시 한 번만 로드됩니다.**
> `.hermes.md`를 실행 중인 세션에서 수정해도 현재 세션에는 반영되지 않습니다.
> 변경 사항은 다음 세션(`hermes` 재시작)부터 적용됩니다.

> **주의: 우선순위에서 첫 번째 매치만 로드됩니다.**
> 같은 디렉터리에 `.hermes.md`와 `AGENTS.md`가 모두 있으면 `.hermes.md`만 로드됩니다.
> 두 파일을 동시에 로드하려면 내용을 `.hermes.md`로 병합해야 합니다.

> **경고: API 키는 config.yaml에 저장하지 마십시오.**
> `${VAR}` 문법으로 참조하더라도 API 키 자체는 config.yaml이 아닌 `.env` 파일에 저장해야 합니다.
> 이 분리는 git 노출 방지를 위해 중요합니다.

> **주의: config.yaml에서 환경변수를 참조할 때는 반드시 `${VAR_NAME}` 형식을 사용하십시오.**
> `$VAR_NAME` (중괄호 없음)은 치환되지 않고 문자열로 그대로 사용됩니다.

---

## 다음 단계

- 이전: [Ch.4 에이전트 루프](../04-agent-loop/index.md)
- 다음: [Ch.6 메모리](../06-memory/index.md)
