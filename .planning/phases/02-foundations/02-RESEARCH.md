# Phase 2: Foundations — Research

**Researched:** 2026-06-09
**Domain:** Hermes Agent — Ch.0 소개 / Ch.1 설치 / Ch.2 첫 실행 / Ch.3 모델 설정
**Confidence:** HIGH (install commands, CLI surface, provider list) / MEDIUM (doctor output format, exact TUI appearance) / LOW (hermes version exact output, some provider-specific details)

**Accuracy constraint:** Every command, flag, config key, and expected output must be grounded in official docs or the real repo. Where verification is incomplete, items are marked LOW and flagged for local verification before publishing.

---

## Summary

Phase 2 produces four content chapters (Ch.0–3) that take a reader from "never heard of Hermes" to "running a first conversation with a configured model." Research drew on the official installation docs, quickstart guide, CLI reference, configuration guide, and skills/learning-loop docs — all at `hermes-agent.nousresearch.com/docs`. Existing STACK.md and PITFALLS.md are confirmed accurate and are extended here with chapter-specific detail.

**Critical correction to existing STATE.md:** `display.language: ko` IS supported as of the current docs. The supported set is `en|zh|zh-hant|ja|de|es|fr|tr|uk|af|ko|it|ga|pt|ru|hu`. However, the docs clarify it only translates "a small set of static user-facing messages" (approval prompt, a handful of gateway slash-command replies). Agent responses, logs, tool output, and error messages remain English. Ch.3 should explain this accurately — not "Korean unsupported," but "Korean UI partial: only static prompts translate; all substantive output is English."

**Primary recommendation:** Each chapter follows a "facts to state → exact commands → expected output → pitfall callouts" structure. Commands are HIGH confidence from official docs; expected outputs are MEDIUM and must be verified locally before publishing. Use the `# 검증: hermes rolling, YYYY-MM-DD` annotation on every code block.

---

## Chapter Structure Guidance

### Recommended src/ directory layout

```
src/
├── README.md                    # 현재 [소개] 항목 — 유지
├── SUMMARY.md                   # 아래 참조
├── 00-intro/
│   └── index.md                 # REPURPOSE: placeholder → Ch.0 소개 본문으로 교체
├── 01-install/
│   └── index.md                 # NEW: Ch.1 설치
├── 02-first-run/
│   └── index.md                 # NEW: Ch.2 첫 실행
└── 03-model/
    └── index.md                 # NEW: Ch.3 모델 설정
```

**Decision on 00-intro:** 기존 `00-intro/index.md`는 `# 검증:` 주석 관례 데모용 placeholder다. Ch.0 소개 내용으로 **교체(overwrite)** 한다. 새 디렉터리를 만들거나 경로를 바꾸면 SUMMARY.md 중복 경로 빌드 오류(Pitfall B-11)가 생기므로 same path reuse가 안전하다.

### Updated SUMMARY.md entries

Current SUMMARY.md:
```markdown
# Summary

[소개](README.md)

---

# 기초

- [소개 및 배경](00-intro/index.md)
```

After Phase 2, it should become:
```markdown
# Summary

[소개](README.md)

---

# 기초

- [소개: Hermes란 무엇인가](00-intro/index.md)
- [설치](01-install/index.md)
- [첫 실행](02-first-run/index.md)
- [모델 설정](03-model/index.md)
```

**Warning:** Do NOT keep the existing `[소개 및 배경](00-intro/index.md)` entry and add a new `[소개: Hermes란 무엇인가](00-intro/index.md)` — that is a duplicate-path build error (Pitfall B-11). The planner must replace the existing entry text in place.

---

## Ch.0: 소개 — Hermes란 무엇인가

### Facts to State (HIGH confidence, verified)

**Core identity:** Hermes is a general-purpose autonomous agent built by Nous Research. It is not an IDE copilot or a chatbot — it is a long-running agent designed to "grow with you" over time. It runs on the command line and connects to any of 40+ model providers.

**The two must-convey concepts before installing:**

1. **Multi-model, no lock-in:** Hermes routes to 40+ providers (Nous Portal, OpenRouter, Anthropic, OpenAI, Gemini, DeepSeek, Ollama, etc.). Switching provider is `hermes model` — no code changes. This matters so readers understand why model setup (Ch.3) is a first-class concern, not an afterthought.

2. **The Learning Loop (Do → Learn → Improve):** This is Hermes's flagship differentiator. It is not just a reactive chatbot:
   - **Do:** Execute tasks using built-in tools (terminal, web, file search, etc.)
   - **Learn:** After completing a complex task (5+ tool calls, or discovering a new solution), Hermes auto-creates a **skill** — a reusable documented procedure stored in `~/.hermes/skills/`
   - **Improve:** Future sessions load relevant skills at startup; the agent gets faster and more accurate on familiar problems over time. Skills self-improve as the agent encounters variations.
   
   Source: official skills docs + GitHub README. This is the reason the tutorial exists — a reader who understands the loop understands why context files, memory, cron, and subagents (later chapters) all connect.

**Other key facts to mention in Ch.0:**
- Multi-platform access: same agent reachable via CLI, Telegram, Discord, Slack, WhatsApp, Signal (covered in Phase 5 — just name-drop in Ch.0)
- Deployable on a $5 VPS or serverless (Modal/Daytona), not laptop-bound
- Compatible with agentskills.io open standard for skill portability

**What Ch.0 should NOT do:** Do not explain install commands (Ch.1), model setup (Ch.3), or any CLI commands in detail. Ch.0 is orientation only.

### How to use this tutorial (tutorial map)

Ch.0 should end with a "튜토리얼 읽는 법" section mapping chapters to the learning path:

| 챕터 | 목표 |
|------|------|
| Ch.0 (지금) | Hermes 철학 이해 |
| Ch.1 설치 | macOS/Linux/WSL2/Windows 설치 + 검증 |
| Ch.2 첫 실행 | 첫 대화 완료, CLI 탐색 |
| Ch.3 모델 설정 | 제공자 연결, API 키, 모델 선택 |
| Phase 3+ | 컨텍스트·메모리·도구·게이트웨이·배포 |

### Pitfall callouts for Ch.0

None required in Ch.0 itself (no commands run yet). However, Ch.0 may include a brief `> 참고` note: "Hermes UI는 부분 한국어를 지원합니다 (`display.language: ko`). 승인 프롬프트 등 일부 정적 메시지만 번역되며, 에이전트 응답·로그·도구 출력은 영어입니다. 이 튜토리얼은 영어 UI를 기준으로 설명합니다."

---

## Ch.1: 설치

### Exact Commands (HIGH confidence, verified against official docs)

#### macOS / Linux / WSL2 / Android (Termux)

```bash
# 검증: hermes rolling (설치 시점 최신), 2026-06-09
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

#### Windows (Native PowerShell)

```powershell
# 검증: hermes rolling (설치 시점 최신), 2026-06-09
iex (irm https://hermes-agent.nousresearch.com/install.ps1)
```

**macOS/Windows Desktop installer (MEDIUM confidence):** The official install page also mentions a "Hermes Desktop" GUI installer downloadable from `hermes-agent.nousresearch.com/desktop`. This is separate from the CLI installer and is described as "recommended" for macOS and Windows in the docs. The planner should flag this as requiring local verification — it is a separate product track and may not be the right focus for a developer-oriented tutorial. **Recommendation:** Mention the Desktop option in a callout but make curl/ps1 the primary tutorial path.

#### Termux (Android) — MEDIUM confidence

Termux follows the same `curl | bash` command as Linux. However, PITFALLS.md A-17 notes that standard install fails on Termux due to voice dependencies. The installer may require `--skip-voice` or a `.[termux]` extras flag. **Planner: flag this as LOW confidence — verify locally or note as "별도 안내" with a link to the official Termux section of install docs rather than transcribing an unverified command.**

### Prerequisites

State clearly before any install command:

| 요구사항 | 플랫폼 | 비고 |
|----------|--------|------|
| Git | macOS / Linux / WSL2 / Termux | 유일한 수동 설치 요구사항 |
| PowerShell | Windows | 기본 포함 |
| (자동 설치) Python 3.11 | 모두 | 직접 설치 금지 — 인스톨러가 uv로 관리 |
| (자동 설치) Node.js v22 | 모두 | 직접 설치 금지 |
| (자동 설치) ripgrep, ffmpeg | 모두 | 직접 설치 금지 |

### Install Locations (HIGH confidence)

| 설치 방식 | 코드 위치 | 바이너리 | 데이터 |
|-----------|-----------|---------|--------|
| 사용자 설치 (표준) | `~/.hermes/hermes-agent/` | `~/.local/bin/hermes` | `~/.hermes/` |
| root 설치 (비권장) | `/usr/local/lib/hermes-agent/` | `/usr/local/bin/hermes` | `/root/.hermes/` |

### PATH Reload Step (HIGH confidence — #1 first-run failure)

This must appear immediately after the install command, before any other step:

```bash
# 검증: bash/zsh shell reload, 2026-06-09
# bash 사용자:
source ~/.bashrc

# zsh 사용자 (macOS 기본):
source ~/.zshrc
```

**Alternative (if source fails or shell unknown):**

```bash
# 검증: manual PATH export, 2026-06-09
export PATH="$HOME/.local/bin:$PATH"
```

This export only lasts the current session. The permanent fix is adding the line to `~/.bashrc` or `~/.zshrc`.

**Recommended verification step after PATH reload:**

```bash
# 검증: hermes rolling, 2026-06-09
which hermes
```

Expected output (HIGH confidence): `/home/<username>/.local/bin/hermes` (Linux) or `/Users/<username>/.local/bin/hermes` (macOS). If empty, PATH was not reloaded.

### hermes doctor (MEDIUM confidence — output format unverified locally)

The command exists and is documented:

```bash
# 검증: hermes rolling, 2026-06-09
hermes doctor
```

What it checks (from official docs): "config and dependency issues." The `--fix` flag attempts auto-repair.

**What the planner CANNOT transcribe verbatim (LOW confidence):** The exact output format — pass/fail icons, check categories, specific error messages. The official docs do not show a sample output. The hermes dump command shows one data point: `version: 0.8.0 (2026.4.8) [af4abd2f]` — suggesting doctor may show similar version context.

**Recommendation for chapter:** Show the command, explain it "checks your config and dependencies and suggests fixes," but use a placeholder for expected output:

```
[hermes doctor 실제 출력 — 로컬 실행 후 캡처 필요]
```

With a `> 검증 필요` callout. Do NOT invent output.

**hermes doctor --fix:**

```bash
# 검증: hermes rolling, 2026-06-09
hermes doctor --fix
```

Attempts automatic repair. Mention this for the "PATH repair" scenario.

### hermes version (MEDIUM confidence — format partially verified)

```bash
# 검증: hermes rolling, 2026-06-09
hermes version
```

From `hermes dump` output mentioned in CLI docs: format is `version: X.Y.Z (YYYY.M.D) [git-hash]`. Example: `version: 0.8.0 (2026.4.8) [af4abd2f]`. The exact current version will differ — this is a rolling release. The chapter should show the command and note "출력 예시: `version: 0.8.0 (2026.4.8) [af4abd2f]` (실행 시점에 따라 다름)".

### Pitfall Callouts for Ch.1 (from PITFALLS.md)

**주의 1 — PATH 미반영 (A-1, CRITICAL):**

```
> 주의: "hermes: command not found" 오류가 나타나면 PATH가 아직 반영되지 않은 것입니다.
> `source ~/.bashrc` (또는 `~/.zshrc`)를 실행하거나 새 터미널 탭을 여십시오.
> `which hermes`로 인식 여부를 확인하세요.
```

**주의 2 — sudo 사용 금지 (A-2, CRITICAL):**

```
> 경고: 설치 스크립트를 `sudo`로 실행하지 마십시오.
> sudo 설치 시 바이너리가 `/usr/local/bin`에 들어가 설정 파일이 root 소유가 됩니다.
> 이미 sudo로 설치했다면: `sudo rm /usr/local/bin/hermes` 후 표준 방법으로 재설치하십시오.
```

**주의 3 — Windows 네이티브 제한 (A-3):**

```
> 참고 (Windows 사용자): PowerShell 설치 방법은 CLI와 게이트웨이를 지원하지만,
> 브라우저 대시보드 채팅 창과 일부 POSIX PTY 기능이 제한됩니다.
> 모든 기능을 사용하려면 WSL2 환경을 권장합니다.
```

**주의 4 — Termux 별도 경로 (A-17, MEDIUM confidence):**

```
> 참고 (Android/Termux): 표준 curl|bash 설치가 음성 의존성 오류로 실패할 수 있습니다.
> Termux 설치는 공식 문서의 별도 안내를 참조하십시오. [링크]
```

---

## Ch.2: 첫 실행

### What `hermes` Launches (HIGH confidence)

Running `hermes` without flags launches the **classic CLI mode** — a full terminal user interface (TUI) using prompt_toolkit. It is NOT a web UI.

Running `hermes --tui` launches the **modern TUI** with modal overlays, mouse selection, and non-blocking input.

**Both interfaces share sessions, slash commands, and all features.** The docs recommend the TUI (`hermes --tui`) for new users. The tutorial may show both commands and let the reader choose.

### Welcome Banner Contents (HIGH confidence)

After launching, the banner displays:
- Current model name
- Working directory
- Available tools
- Installed skills

Below the banner: a persistent status bar showing model, token usage (current/max), context fill percentage (color-coded), estimated cost, compression count, active background tasks, and elapsed session time.

**Expected output for tutorial:** The planner CANNOT transcribe the exact banner text verbatim — it depends on the reader's model and installed skills. Use a schematic placeholder:

```
╔══════════════════════════════════════╗
║  Hermes Agent  │  <모델명>           ║
║  📁 ~/         │  tools: N  skills: M ║
╚══════════════════════════════════════╝

[상태표시줄: model | N/M tokens | X% | $0.00 | 0 bg | 0s]

> (입력 프롬프트)
```

Mark this as: `[실제 배너는 모델과 설치 환경에 따라 다름 — 로컬 실행 후 스크린샷 교체 필요]`

### First Conversation Steps (HIGH confidence for steps; output must be verified locally)

1. Run `hermes` or `hermes --tui`
2. Wait for welcome banner
3. Type a simple test message and press Enter:

```
이 레포의 주요 파일을 5줄로 요약해줘.
```

or (English, safe for any model):

```
What files are in my current directory?
```

4. Observe: agent replies, tools activate if needed (terminal for `ls`, file read, etc.)
5. Verify: more than one turn works (multi-turn = success indicator per official docs)

**Verification criteria (from official quickstart):** "The banner shows your chosen model/provider. Hermes replies without error. It can use a tool if needed. The conversation continues normally for more than one turn."

### Session Resume (HIGH confidence)

```bash
# 검증: hermes rolling, 2026-06-09
# 가장 최근 세션 재개
hermes --continue
```

### Top-Level Commands to Show in Ch.2 (HIGH confidence, all verified)

Present these as a "주요 명령어 일람" reference table:

| 명령어 | 설명 |
|--------|------|
| `hermes` | 기본 CLI 모드로 시작 |
| `hermes --tui` | 모던 TUI 모드로 시작 |
| `hermes --continue` | 가장 최근 세션 재개 |
| `hermes chat -q "질문"` | 비대화형 단일 질의 |
| `hermes model` | 공급자·모델 설정 마법사 |
| `hermes setup` | 전체 설정 마법사 |
| `hermes setup --portal` | Nous Portal OAuth 원클릭 설정 |
| `hermes tools` | 도구 설정 (대화형) |
| `hermes tools --summary` | 현재 도구 상태 출력 |
| `hermes gateway` | 메시징 플랫폼 서비스 관리 |
| `hermes doctor` | 설치·설정 진단 |
| `hermes doctor --fix` | 자동 복구 시도 |
| `hermes version` | 버전 정보 출력 |
| `hermes config show` | 현재 설정 전체 출력 |
| `hermes sessions list` | 세션 목록 |

**Note:** There is no bare `hermes help` command documented as a top-level command in the CLI reference. The global `--help` / `-h` flag exists (`hermes --help`). Do NOT write `hermes help` as if it's a standalone command — use `hermes --help`.

```bash
# 검증: hermes rolling, 2026-06-09
hermes --help
```

### Key In-Session Slash Commands (HIGH confidence)

| 슬래시 커맨드 | 설명 |
|--------------|------|
| `/help` | 사용 가능한 커맨드 목록 |
| `/model` | 현재 모델 표시 / 전환 (설정된 공급자 간) |
| `/tools` | 도구 목록 |
| `/skills browse` | 스킬 허브 탐색 |
| `/voice on` | 음성 모드 활성화 |
| `/reasoning high` | 추론 수준 조정 |
| `/title` | 세션 이름 지정 |
| `/status` | 세션 정보 요약 |
| `/compress` | 수동 컨텍스트 압축 |
| `/rollback` | 마지막 체크포인트로 되돌리기 |

Commands are case-insensitive (`/HELP` = `/help`).

### Keybindings (HIGH confidence)

| 키 | 동작 |
|----|------|
| `Enter` | 메시지 전송 |
| `Alt+Enter` / `Ctrl+J` / `Shift+Enter` | 줄바꿈 (다음 줄) |
| `Ctrl+G` | 외부 에디터에서 입력 편집 |
| `Ctrl+C` | 에이전트 중단 (두 번 연속 → 강제 종료) |
| `Ctrl+D` | 종료 |

### Pitfall Callouts for Ch.2

**주의 — 공급자 미설정 시 첫 실행 실패:**

```
> 주의: hermes를 실행하면 모델/공급자가 설정되어 있지 않은 경우
> "No provider configured" 오류가 나타날 수 있습니다.
> 이 경우 Ctrl+D로 종료하고 먼저 Ch.3 모델 설정을 완료하십시오.
```

**실제 출력 캡처 필수 (for planner):**

The banner, status bar, and first tool-call output MUST be captured from a real local run before publishing. Do NOT transcribe guessed output. Mark all output blocks with `[로컬 실행 후 캡처 필요]` during drafting.

---

## Ch.3: 모델 설정

### hermes model Flow (HIGH confidence, structure verified)

```bash
# 검증: hermes rolling, 2026-06-09
hermes model
```

This launches an **interactive wizard** for:
- Adding new providers
- Setting up API keys
- Running OAuth flows

The wizard must be run from the terminal **outside** an active chat session. The in-session `/model` command only switches between already-configured providers.

### hermes setup --portal (HIGH confidence)

```bash
# 검증: hermes rolling, 2026-06-09
hermes setup --portal
```

Does three things simultaneously:
1. OAuth login to Nous Portal (browser opens)
2. Sets Nous as the provider
3. Enables Tool Gateway

This is the recommended path for new users. After completion, run `hermes` and the banner should show the Nous model.

### Verified Providers for Ch.3 (HIGH confidence — from official quickstart provider table)

For a beginner-focused Ch.3, recommend focusing on 3–4 providers that are easiest for Korean developers. The full provider list has 40+; show a curated subset and reference the full list.

**Tier 1 — Recommend showing in detail:**

| 공급자 | 인증 방식 | 특징 |
|--------|-----------|------|
| Nous Portal | OAuth (`hermes setup --portal`) | 가장 쉬운 시작; 300+ 모델; Tool Gateway 포함 |
| OpenRouter | API 키 (`OPENROUTER_API_KEY`) | 200+ 모델; 단일 키로 다양한 모델 접근 |
| Anthropic | API 키 (`ANTHROPIC_API_KEY`) | Claude 모델; **Claude Pro/Max 구독과 별개** (경고 필수) |
| OpenAI | API 키 (환경변수명 MEDIUM confidence — verify) | GPT 모델 |

**Tier 2 — Mention but don't detail:**

Google Gemini (OAuth or `GOOGLE_API_KEY`/`GEMINI_API_KEY`), DeepSeek (`DEEPSEEK_API_KEY`), NVIDIA NIM (`NVIDIA_API_KEY`), Ollama (로컬, 키 불필요), Custom Endpoint (OpenAI-compatible).

**Note on OpenAI environment variable name (LOW confidence):** The provider table fetched shows the env var name for OpenAI is NOT explicitly listed in the portion retrieved. The standard convention is `OPENAI_API_KEY`. This is MEDIUM confidence — verify against `hermes model` wizard output before publishing.

### API Key Storage (HIGH confidence)

```
비밀 (API 키)  → ~/.hermes/.env
비밀 외 설정   → ~/.hermes/config.yaml
OAuth 자격증명 → ~/.hermes/auth.json
```

The `hermes config set` command automatically routes values to the correct file. Users should NOT manually edit `config.yaml` for API keys.

To verify where a key was stored:

```bash
# 검증: hermes rolling, 2026-06-09
cat ~/.hermes/.env
```

### config.yaml — Model Section (HIGH confidence)

```yaml
# 검증: hermes rolling, 2026-06-09
model:
  default: nous/hermes-3-405b   # 또는 provider/model-id 형식
  provider: nous                 # nous|openrouter|anthropic|openai|custom|auto
  context_length: 131072
  reasoning_effort: medium       # none|minimal|low|medium|high|xhigh
```

Show readers how to inspect:

```bash
# 검증: hermes rolling, 2026-06-09
hermes config show
```

### 64K Context Requirement (HIGH confidence)

State explicitly:

"Hermes Agent는 **최소 64,000 토큰**의 컨텍스트 윈도우를 가진 모델이 필요합니다. 이보다 작은 모델은 시작 시 거부됩니다."

Why: Hermes has ~13,935 tokens of fixed overhead per call (tool definitions ~8,759 tokens + system prompt ~5,176 tokens). Models with context < 64K leave nearly no room for actual conversation.

From official docs verbatim: "Hermes Agent requires a model with at least 64,000 tokens of context. Models with smaller windows cannot maintain enough working memory for multi-step tool-calling workflows."

Recommended models that exceed 64K: "Most hosted models (Claude, GPT, Gemini, Qwen, DeepSeek) meet this easily." For Ollama: requires explicit `-c 65536` parameter.

**For Ollama users:**

```bash
# 검증: ollama (버전 로컬 확인 필요), 2026-06-09
# 예시: llama3.2를 64K 컨텍스트로 실행
ollama run llama3.2 --context-size 65536
```

(LOW confidence — verify this is the correct Ollama flag syntax)

### display.language Korean Support (UPDATED — corrected from STATE.md)

**Corrected finding:** `ko` IS in the supported language list per current official docs.

Full supported list: `en | zh | zh-hant | ja | de | es | fr | tr | uk | af | ko | it | ga | pt | ru | hu`

BUT — the scope is narrow. From official docs: "this setting specifically translates a small set of static user-facing messages — the CLI approval prompt, a handful of gateway slash-command replies." It does NOT affect:
- Agent responses
- Logs
- Tool output
- Error messages (these remain English)

**What Ch.3 should say:**

```yaml
# 검증: hermes rolling, 2026-06-09
display:
  language: ko    # 승인 프롬프트 등 일부 정적 메시지를 한국어로 번역
                  # 에이전트 응답·로그·도구 출력은 영어 유지
```

Add a `> 참고` callout: "한국어(ko) 설정은 CLI 승인 프롬프트와 일부 게이트웨이 슬래시 커맨드 응답만 번역합니다. 에이전트의 실제 응답과 오류 메시지는 영어입니다."

### Pitfall Callouts for Ch.3

**주의 1 — Claude Pro/Max 구독 ≠ API 접근 (A-6, CRITICAL):**

```
> 경고 (Anthropic Claude 공급자 선택 시):
> Claude Pro 또는 Max 구독(claude.ai)은 Hermes와 같은 서드파티 도구의 API 접근을
> 허용하지 않습니다. Hermes에서 Claude를 사용하려면 console.anthropic.com에서
> 별도 API 계정과 API 키(sk-ant-...)를 발급받아야 합니다.
> 대안: OpenRouter를 통해 Claude 모델에 접근할 수도 있습니다.
```

**주의 2 — hermes setup Silent API Key Skip (A-5, CRITICAL):**

```
> 주의: ~/.hermes/.env에 해당 공급자 키 항목이 이미 존재하면 (값이 잘못되어도)
> hermes setup의 API 키 입력 프롬프트가 조용히 건너뛰어집니다 (GitHub Issue #16394).
> 녹색 체크마크가 표시되어도 실제로는 잘못된 키가 남아있을 수 있습니다.
> 해결: ~/.hermes/.env를 직접 편집하여 해당 키 줄을 삭제 후 hermes setup 재실행.
> 또는: hermes config set PROVIDER_API_KEY NEW_KEY 로 직접 덮어쓰기.
```

**주의 3 — API 키를 config.yaml에 저장하지 말 것 (A-4):**

```
> 경고: API 키는 반드시 ~/.hermes/.env에 저장해야 합니다.
> config.yaml에 키를 직접 입력하면 git 추적 시 키가 노출될 위험이 있습니다.
> hermes model 마법사나 hermes auth add를 사용하면 자동으로 .env에 저장됩니다.
```

**주의 4 — 64K 미만 모델 선택 시 (A-8):**

```
> 주의: 컨텍스트 윈도우가 64,000 토큰 미만인 모델은 Hermes가 시작 시 거부합니다.
> 무료 또는 소형 모델 선택 시 공급자의 모델 사양에서 context window를 확인하십시오.
> 고정 오버헤드(약 13,935 토큰)가 매 API 호출마다 소비됩니다.
```

**주의 5 — /model과 hermes model의 차이 (A-16):**

```
> 참고: 채팅 세션 내 /model 명령은 이미 설정된 공급자 간 전환만 가능합니다.
> 새 공급자를 추가하려면 세션을 종료하고 터미널에서 hermes model을 실행하십시오.
```

---

## Architecture Patterns (for planner)

### Chapter file template

Each chapter file (`index.md`) should follow this structure:

```markdown
# Ch.N 제목

> **이 챕터를 시작하기 전에**
> - [선행 조건 체크리스트]

## 개요

[1-2단락: 이 챕터에서 할 일]

## [주요 섹션들]

[명령어·설명·예상 출력]

## 흔한 오류 / 주의

[pitfall callouts — 주의/경고 admonition blocks]

## 다음 단계

다음 챕터: [링크]
```

### Code block annotation convention (from Phase 1, mandatory)

Every command code block must start with:

```
# 검증: hermes rolling (설치 시점 최신), 2026-06-09
```

For mdBook 0.5.3 config blocks:

```
# 검증: mdBook 0.5.3, 2026-06-09
```

Where output is unverified locally, add instead:

```
# [로컬 실행 후 캡처 필요 — 출력은 환경에 따라 다름]
```

---

## Open Questions (LOW confidence items requiring local verification before publishing)

1. **hermes doctor exact output format**
   - What we know: command exists, checks config/deps, --fix does auto-repair
   - What's unclear: exact check categories, pass/fail icons/text, sample output
   - Recommendation: Run locally, capture output, insert verbatim in Ch.1

2. **hermes version exact output on current rolling build**
   - What we know: format is `version: X.Y.Z (YYYY.M.D) [git-hash]` (from hermes dump data point)
   - What's unclear: current version number, whether output has additional fields
   - Recommendation: Run locally, capture, insert in Ch.1

3. **hermes welcome banner exact appearance**
   - What we know: shows model, working dir, tools, skills; status bar with token/cost info
   - What's unclear: exact layout, emoji usage, color coding
   - Recommendation: Capture screenshot for Ch.2; describe schematically in text

4. **OpenAI environment variable name for API key**
   - What we know: standard convention is OPENAI_API_KEY
   - What's unclear: whether hermes uses a different internal name (e.g., HERMES_OPENAI_API_KEY)
   - Recommendation: Check hermes model wizard output or ~/.hermes/.env after setup; HIGH probability it's OPENAI_API_KEY

5. **Termux install special flags**
   - What we know: standard curl|bash fails due to voice dependencies (Pitfall A-17)
   - What's unclear: exact workaround flag (--skip-voice? .[termux] extras?)
   - Recommendation: Verify against official Termux docs section; do NOT guess the flag

6. **hermes --tui appearance vs classic**
   - What we know: modal overlays, mouse selection, non-blocking input
   - What's unclear: exact visual difference, which to recommend for readers
   - Recommendation: Run both locally, decide which to use as primary screenshot

7. **Nous Portal Tool Gateway free vs paid boundary**
   - What we know: Tool Gateway is included in hermes setup --portal; free vs paid split exists (Pitfall A-7)
   - What's unclear: current pricing tier details as of 2026-06
   - Recommendation: Check Nous Portal pricing page before publishing Ch.3 Nous Portal section

8. **display.language: ko scope verification**
   - What we know (updated): ko IS listed in supported values; scope is "static user-facing messages" only
   - What's unclear: which exact static messages translate (approval prompt confirmed; others unclear)
   - Recommendation: Set display.language: ko, run hermes, observe which strings translate vs stay English

---

## Sources

### Primary (HIGH confidence — verified)
- `https://hermes-agent.nousresearch.com/docs/getting-started/installation` — install commands, locations, prerequisites, PATH reload, sudo notes
- `https://hermes-agent.nousresearch.com/docs/getting-started/quickstart` — first-run flow, setup --portal, provider list, 64K requirement, success criteria
- `https://hermes-agent.nousresearch.com/docs/reference/cli-commands` — top-level commands, flags, subcommands, hermes version format hint
- `https://hermes-agent.nousresearch.com/docs/user-guide/configuration` — display.language supported values (including ko), model config structure
- `https://hermes-agent.nousresearch.com/docs/user-guide/cli` — welcome banner contents, slash commands, keybindings, TUI vs classic
- `https://hermes-agent.nousresearch.com/docs/user-guide/features/skills` — Do-Learn-Improve loop technical description, skill auto-creation triggers
- `https://github.com/NousResearch/hermes-agent` — core philosophy, differentiators, positioning

### Secondary (MEDIUM confidence — from official docs, some details unverified locally)
- `hermes dump` output format (version string format) — inferred from CLI reference docs
- Provider env variable names (most confirmed; OpenAI OPENAI_API_KEY inferred from convention)

### From Prior Research (HIGH confidence, imported)
- STACK.md: all CLI commands table, config.yaml full structure, supported providers table
- PITFALLS.md: A-1 through A-8 mapped to chapters; B-11 SUMMARY.md duplicate path warning

---

## Metadata

**Confidence breakdown:**
- Ch.0 philosophy/framing: HIGH — verified against README and official marketing docs
- Ch.1 install commands: HIGH — exact commands from official install docs
- Ch.1 hermes doctor: MEDIUM — command confirmed, output format unverified
- Ch.2 CLI commands list: HIGH — from official CLI reference
- Ch.2 expected output: LOW-MEDIUM — must capture locally
- Ch.3 provider list (core 4): HIGH — from official quickstart provider table
- Ch.3 hermes model flow: HIGH — from CLI reference
- Ch.3 display.language ko: HIGH for existence; LOW for exact scope of translation

**Research date:** 2026-06-09
**Valid until:** 2026-07-09 (Hermes is rolling release — re-verify commands 30 days before any major publish/update)
**Key risk:** Hermes rolling release means CLI flags and config keys can change without versioned docs. All commands should be re-verified locally before each chapter is marked publication-ready.
