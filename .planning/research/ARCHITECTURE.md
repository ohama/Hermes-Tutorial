# Architecture Research

**Domain:** Korean developer tutorial (mdBook) for Nous Research Hermes Agent
**Researched:** 2026-06-05
**Confidence:** HIGH (A — Hermes architecture) / HIGH (B — mdBook/Pages structure)

---

## Part A: Hermes Agent Architecture

### System Overview

```
┌──────────────────────────────────────────────────────────────────────┐
│                         ENTRY POINTS                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────┐  ┌─────────────┐  │
│  │  CLI / TUI   │  │   Gateway    │  │   ACP    │  │  Batch/API  │  │
│  │  (cli.py)    │  │(gateway/run) │  │ (editor) │  │  (v0.4.0+)  │  │
│  └──────┬───────┘  └──────┬───────┘  └────┬─────┘  └──────┬──────┘  │
└─────────┼─────────────────┼───────────────┼───────────────┼──────────┘
          │                 │               │               │
          └─────────────────┴───────────────┴───────────────┘
                                    │
                    ┌───────────────▼───────────────┐
                    │        AIAgent Core            │
                    │      (run_agent.py)            │
                    │  Orchestration Engine:         │
                    │  provider selection,           │
                    │  prompt construction,          │
                    │  tool execution,               │
                    │  retries / fallback,           │
                    │  callbacks, compression,       │
                    │  persistence                   │
                    └──┬──────┬──────┬───────┬───────┘
                       │      │      │       │
          ┌────────────┘  ┌───┘  ┌───┘   ┌──┘
          ▼               ▼      ▼        ▼
┌──────────────┐ ┌───────────┐ ┌────────────┐ ┌──────────────────────┐
│  Prompt      │ │ Provider  │ │  Tool      │ │  Session / Memory    │
│  System      │ │ Runtime   │ │  Registry  │ │  (SQLite + FTS5)     │
│(prompt_buil- │ │(providers/│ │(tools/reg- │ │  state.db            │
│ der.py,      │ │resolver)  │ │ istry.py)  │ │  MEMORY.md / USER.md │
│context_comp- │ │18+ provid-│ │70+ tools   │ │  skills/ directory   │
│ressor.py,    │ │ers, OAuth,│ │28 toolsets │ │  Honcho (optional)   │
│prompt_cach-  │ │cred pools,│ │6 backends  │ │                      │
│ ing.py)      │ │alias res.)│ │ThreadPool  │ │                      │
└──────────────┘ └───────────┘ └────────────┘ └──────────────────────┘
```

### Component Responsibilities

| Component | Responsibility | Communicates With | Source |
|-----------|---------------|-------------------|--------|
| **CLI / TUI** (`cli.py`) | Interactive terminal session; multiline editing, slash-command autocomplete, conversation history, interrupt/redirect | → AIAgent core | Entry point |
| **Gateway** (`gateway/run.py`) | Long-running process; 20+ platform adapters (Telegram, Discord, Slack, WhatsApp, Signal, Matrix, Email, etc.); unified session routing, user auth allowlists, DM pairing, slash commands, hook system, cron delivery | → AIAgent core, → Cron, ↔ Platform APIs | Messaging layer |
| **ACP Adapter** | Agent Communication Protocol; exposes Hermes over stdio/JSON-RPC for VS Code, Zed, JetBrains | → AIAgent core | Editor integration |
| **API Server** (v0.4.0+) | OpenAI-compatible `/v1/chat/completions` endpoint; SQLite-backed persistence, CORS, `Idempotency-Key`, real-time tool streaming, `X-Hermes-Session-Id` continuity | → AIAgent core | Programmatic access |
| **AIAgent Core** (`run_agent.py`) | Central synchronous orchestration engine; single class serves all entry points; manages full conversation loop | ← All entry points, → Prompt System, Provider Runtime, Tool Registry, Session DB | Core |
| **Prompt System** (`prompt_builder.py`, `context_compressor.py`, `prompt_caching.py`) | Assembles tiered system prompt (stable identity → context/memory → volatile timestamp); compresses at 50% context window; applies Anthropic prefix caching | ← AIAgent, ↔ Session DB | Prompt layer |
| **Provider Runtime** (`providers/`) | Maps `(provider, model)` tuples to `(api_mode, api_key, base_url)` for 18+ providers; OAuth flows, credential pool rotation, failover chains | ← AIAgent, → External LLM APIs | Model routing |
| **Transport Layer** (`agent/transports/`, v0.11.0+) | `ProviderTransport` ABC with implementations: `AnthropicTransport`, `ChatCompletionsTransport`, `ResponsesApiTransport`, `BedrockTransport`; handles message conversion, tool mapping, streaming | ← Provider Runtime | Wire protocol |
| **Tool Registry** (`tools/registry.py`) | Central registry of 70+ tools across ~28 toolsets; self-registration at import time; concurrent execution via `ThreadPoolExecutor` (max 8 workers); path-scoped concurrency safety | ← AIAgent, → Terminal Backends | Tool dispatch |
| **Terminal Backends** | Six execution environments: local, Docker, SSH, Singularity, Modal (serverless), Daytona | ← Tool Registry | Execution isolation |
| **Session DB** (SQLite + FTS5) | `~/.hermes/state.db`; stores session history, memory, skill metadata; FTS5 full-text search; session lineage (`parent_session_id`); per-platform isolation; atomic writes | ← AIAgent, ← Prompt System | Persistence |
| **Memory System** (`MEMORY.md`, `USER.md`) | Persistent core memory (~1.3k tokens frozen at session start); pluggable backends (v0.7.0+); Honcho dialectic user modeling (optional); `session_search` tool for FTS5 recall | ↔ AIAgent, ↔ Session DB | Memory layer |
| **Skills Store** (`~/.hermes/skills/`) | Markdown files with YAML frontmatter; auto-generated after 5+ tool-call tasks; self-improving during execution; 118 bundled skills (v0.10.0); agentskills.io standard | ← AIAgent (triggered), → Prompt System (injected) | Procedural memory |
| **Cron Scheduler** | Built-in scheduler; JSON-stored jobs; multiple schedule formats; spawns fresh AIAgent per job tick; skill injection; platform delivery | ← Gateway, → AIAgent core | Automation |
| **Subagent System** | Isolated subagents with independent conversations, terminals, Python RPC scripts; parallel workstreams without context bleed | ← AIAgent (spawns), → Tool Registry | Parallelism |
| **Plugin System** (v0.3.0+) | Three discovery sources: `~/.hermes/plugins/`, project plugins, pip entry points; custom tools, lifecycle hooks (`pre_llm_call`, `post_llm_call`, `on_session_start`, `on_session_end`), memory providers, context engines | ← AIAgent, → Tool Registry | Extensibility |
| **MCP Layer** | Bidirectional: (1) MCP client — connects any MCP server for extended tools; (2) MCP server (`hermes mcp serve`) — exposes Hermes sessions to Claude Desktop, Cursor, VS Code, Zed via stdio/JSON-RPC | ← Tool Registry (client), ← ACP (server) | Protocol bridge |

---

### Data Flow: CLI Conversation

```
User input (terminal)
    │
    ▼
HermesCLI.process_input()
    │
    ▼
AIAgent.run_conversation()
    │
    ├──► prompt_builder.py
    │       ├─ Load stable identity layer (SOUL.md / personality)
    │       ├─ Inject context files (project .hermes/ context)
    │       ├─ Inject memory (MEMORY.md + USER.md, ≤1.3k tokens)
    │       ├─ Retrieve relevant session fragments (FTS5 search)
    │       └─ Append volatile block (timestamp, active skills)
    │
    ├──► context_compressor.py  (if >50% context window used)
    │       └─ Summarize middle turns, protect first + last 20 turns
    │
    ├──► Provider Runtime (provider/model resolution)
    │       └─ Transport Layer (AnthropicTransport / ChatCompletions / etc.)
    │               └──► External LLM API call (streaming)
    │
    ├──► Tool dispatch loop  (if LLM returns tool calls)
    │       ├─ ThreadPoolExecutor (≤8 workers, path-scoped concurrency)
    │       ├─ Approval gate (if approval mode enabled)
    │       ├─ Terminal backend execution (local/Docker/SSH/Modal/…)
    │       └─ Results reinserted into conversation history
    │
    ├──► Skill creation check  (if ≥5 tool calls in turn)
    │       └─ LLM extracts reusable pattern → Markdown skill file saved
    │
    ├──► Response display  (token-by-token streaming)
    │
    └──► SessionDB.save()  (SQLite + FTS5 index update)
```

### Data Flow: Gateway Message (Telegram/Discord/etc.)

```
Platform event (incoming message)
    │
    ▼
Platform Adapter (gateway/run.py)
    │
    ├──► User authorization check (allowlist / DM pairing)
    ├──► Slash command dispatch (if /command)
    ├──► Session resolution (per-platform isolation, history load)
    │
    ▼
AIAgent (fresh instance, history injected)
    │  (same loop as CLI above)
    ▼
Response delivery → Platform API (message send)
```

### Data Flow: Cron Job

```
Scheduler tick (built-in cron, Gateway process)
    │
    ▼
Load due jobs (JSON store)
    │
    ▼
Fresh AIAgent (isolated session)
    ├──► Skill injection (configured for job)
    ├──► Task execution (tool loop)
    └──► Delivery to target platform
```

### Learning Loop (Self-Improvement Cycle)

```
Task execution (≥5 tool calls)
    │
    ▼
Post-task pattern extraction (LLM analysis)
    │
    ▼
Skill Markdown file created / updated
(~/.hermes/skills/<name>.md  with YAML frontmatter)
    │
    ▼
Skill injected into future sessions
    │
    ▼
Skill self-improves during execution (refinement)
    │
    ▼
Memory flush: LLM decides what persists in MEMORY.md / USER.md
    │
    ▼
Session indexed in SQLite FTS5 (searchable across future sessions)
    │
    ▼
Honcho user model updated (optional — longer-term behavioral patterns)
```

---

### Memory Architecture (Four Layers)

| Layer | Storage Location | Token Budget | Retrieval Mechanism |
|-------|-----------------|--------------|---------------------|
| Persistent Core Memory | `~/.hermes/MEMORY.md` + `USER.md` | ~1.3k tokens (frozen at session start) | Always loaded |
| Session History | `~/.hermes/state.db` (SQLite FTS5) | Unbounded | `session_search` tool — keyword FTS5 + LLM summarization |
| User Model | Honcho dialectic (optional plugin) | N/A | Cross-session behavioral patterns |
| Procedural Memory (Skills) | `~/.hermes/skills/*.md` | Injected on match | Skill matching at session start / on demand |

### MCP Integration Role

MCP plays a **bidirectional** role in Hermes:

**As MCP Client** (extend Hermes with external tools):
- Connect any MCP server via `hermes tools` → MCP section
- Use cases: GitHub automation, knowledge servers, internal databases/APIs
- OAuth PKCE flows for remote MCP server auth
- Credential filtering for security

**As MCP Server** (expose Hermes to other tools):
- `hermes mcp serve` — runs Hermes over stdio/JSON-RPC
- Compatible clients: Claude Desktop, Cursor, VS Code, Zed, JetBrains
- Allows IDEs to use Hermes as an intelligent backend agent
- Session continuity preserved across MCP calls

---

## Part B: Tutorial Information Architecture

### Recommended Chapter Sequence (dependency-aware)

The ordering below respects conceptual prerequisites — each chapter assumes the prior ones.

```
Chapter 0: 소개 및 배경 (Introduction)
  └── 선행 요건: 없음
  └── 내용: Hermes란 무엇인가, Nous Research 소개, OpenClaw와의 차이,
            이 튜토리얼의 범위와 학습 목표

Chapter 1: 설치 및 환경 설정 (Installation & Setup)
  └── 선행 요건: Ch.0
  └── 내용: install.sh (macOS/Linux), install.ps1 (Windows), WSL2, Termux,
            hermes setup 마법사, ~/.hermes/ 디렉터리 구조,
            .env vs config.yaml 역할 분리

Chapter 2: 첫 번째 대화 (First Conversation)
  └── 선행 요건: Ch.1
  └── 내용: hermes / hermes --tui 실행, CLI TUI 인터페이스 조작법,
            /help 슬래시 커맨드, hermes --continue, hermes doctor,
            스트리밍 응답 이해

Chapter 3: 모델 선택과 전환 (Model Selection & Switching)
  └── 선행 요건: Ch.2
  └── 내용: hermes model, Nous Portal / OpenRouter / OpenAI / Anthropic /
            로컬 엔드포인트, Provider Runtime 내부 동작,
            64k 토큰 최소 컨텍스트 요건, 모델 교체 시 코드 변경 불필요한 이유

Chapter 4: AIAgent 코어와 에이전트 루프 (Agent Core & Loop)
  └── 선행 요건: Ch.3
  └── 내용: run_agent.py AIAgent 클래스, 동기 오케스트레이션 엔진,
            대화 루프 단계(프롬프트 구성 → LLM 호출 → 툴 디스패치 → 저장),
            Observable execution, Interruptible design

Chapter 5: 프롬프트 시스템과 컨텍스트 파일 (Prompt System & Context Files)
  └── 선행 요건: Ch.4
  └── 내용: 3-tier 프롬프트 계층(stable → context → volatile),
            SOUL.md 퍼스널리티, 프로젝트 .hermes/ 컨텍스트 파일,
            context_compressor.py (50% 임계값), prompt_caching.py,
            압축(Compression) 동작 원리

Chapter 6: 메모리 시스템 (Memory System)
  └── 선행 요건: Ch.5
  └── 내용: 4계층 메모리 구조, MEMORY.md + USER.md (≤1.3k tokens),
            SQLite FTS5 session_search, Honcho 통합(옵션),
            hermes memory setup, 메모리 플러시 동작

Chapter 7: 툴 게이트웨이 (Tool Gateway)
  └── 선행 요건: Ch.4
  └── 내용: tools/registry.py, 70+ 내장 툴 / 28 툴셋,
            hermes tools 설정, ThreadPoolExecutor 병렬 실행(≤8 workers),
            승인 모드(Approval Mode), 경로 스코프 동시성 안전,
            웹 검색·이미지 생성·TTS·클라우드 브라우저

Chapter 8: 터미널 백엔드 (Terminal Backends)
  └── 선행 요건: Ch.7
  └── 내용: 6개 백엔드 (local, Docker, SSH, Singularity, Modal, Daytona),
            백엔드 선택 기준, 서버리스 옵션의 유휴 비용,
            컨테이너 강화 및 네임스페이스 격리

Chapter 9: 스킬 시스템 (Skills System)
  └── 선행 요건: Ch.6, Ch.7
  └── 내용: 절차적 기억 개념, 스킬 자동 생성 트리거(≥5 tool calls),
            ~/.hermes/skills/*.md 형식(Markdown + YAML frontmatter),
            skill_manage 툴, hermes skills, 118 번들 스킬(v0.10.0),
            agentskills.io 표준, Skills Hub 설치

Chapter 10: 학습 루프 (Learning Loop)
  └── 선행 요건: Ch.9
  └── 내용: Do → Learn → Improve 사이클 전체,
            스킬 자가 개선 메커니즘, Honcho 사용자 모델링 심화,
            성능 개선 효과(~40% 작업 시간 단축 실험 사례),
            세션 간 지식 축적 흐름

Chapter 11: MCP 통합 (MCP Integration)
  └── 선행 요건: Ch.7, Ch.9
  └── 내용: MCP(Model Context Protocol) 개념,
            Hermes를 MCP 클라이언트로 사용 (외부 MCP 서버 연결),
            Hermes를 MCP 서버로 노출 (hermes mcp serve),
            Claude Desktop / Cursor / VS Code / Zed 연동,
            OAuth PKCE, 자격증명 필터링

Chapter 12: 메시징 게이트웨이 (Messaging Gateway)
  └── 선행 요건: Ch.4, Ch.8
  └── 내용: hermes gateway setup, 지원 플랫폼 20+
            (Telegram, Discord, Slack, WhatsApp, Signal, Matrix, Email 등),
            단일 게이트웨이 프로세스 아키텍처,
            플랫폼 간 세션 연속성, 사용자 인가(allowlist / DM pairing),
            음성 메모 전사(voice transcription), /status 슬래시 커맨드

Chapter 13: 크론 스케줄러 (Cron Scheduler)
  └── 선행 요건: Ch.12
  └── 내용: 내장 cron 개념, 자연어 스케줄 생성,
            JSON 저장 형식, 다중 스케줄 포맷,
            스킬 주입 실행, 플랫폼 배달,
            자동화 사용 사례 (일일 보고서, 야간 백업, 주간 감사)

Chapter 14: 서브에이전트와 병렬 작업 (Subagents & Parallel Work)
  └── 선행 요건: Ch.13, Ch.9
  └── 내용: 서브에이전트 개념, 독립 대화/터미널/Python RPC,
            컨텍스트 블리드 없는 병렬 워크스트림,
            Python 스크립트에서 툴 RPC 호출,
            다단계 파이프라인 통합 패턴

Chapter 15: 플러그인과 확장 (Plugins & Extensions)
  └── 선행 요건: Ch.7, Ch.11
  └── 내용: 플러그인 세 가지 디스커버리 소스,
            ~/.hermes/plugins/ 드롭인 방식,
            커스텀 툴 작성, 라이프사이클 훅,
            메모리 프로바이더 교체, 대시보드 탭 추가

Chapter 16: API 서버 모드 (API Server Mode)
  └── 선행 요건: Ch.4, Ch.11
  └── 내용: OpenAI 호환 /v1/chat/completions,
            X-Hermes-Session-Id 세션 연속성,
            Idempotency-Key, CORS 설정,
            실시간 툴 진행 스트리밍(v0.7.0+)

Chapter 17: 배포 (Deployment)
  └── 선행 요건: Ch.8, Ch.12, Ch.13
  └── 내용: 배포 시나리오별 전략 (개인 서버, Docker, SSH 원격,
            Singularity HPC, Modal 서버리스, Daytona),
            서버리스 환경 hibernation과 비용 최적화,
            프로파일 격리, GitHub Actions CI/CD

Chapter 18: 보안과 운영 (Security & Operations)
  └── 선행 요건: Ch.17
  └── 내용: PII 자동 삭제(privacy.redact_pii), 시크릿 레댁션,
            승인 시스템, 워크스페이스 스코프 파일 접근,
            OAuth PKCE, 다중 프로파일 격리,
            hermes doctor, hermes logs --follow, 문제 해결
```

### Chapter Prerequisite Graph (abbreviated)

```
Ch.0 → Ch.1 → Ch.2 → Ch.3 → Ch.4
                               │
                    ┌──────────┼──────────┐
                    ▼          ▼          ▼
                  Ch.5       Ch.7       Ch.12
                    │          │          │
                    ▼          ▼          ▼
                  Ch.6       Ch.8       Ch.13
                    │          │          │
                    └──┬───────┘       Ch.14
                       ▼
                     Ch.9
                       │
                  ┌────┴────┐
                  ▼         ▼
               Ch.10      Ch.11
                            │
                    ┌───────┴──────┐
                    ▼              ▼
                 Ch.15          Ch.16
                            ┌────────┐
                            ▼        ▼
                          Ch.17   Ch.18
```

---

## Part C: mdBook Source Layout & GitHub Pages Deploy Structure

### Recommended mdBook Source Layout

```
hermes-tutorial/           # Git repo root
├── book.toml              # mdBook configuration
├── src/                   # All book content lives here
│   ├── SUMMARY.md         # Chapter index (drives navigation)
│   ├── README.md          # Book introduction (maps to index.html)
│   ├── 00-intro/
│   │   └── index.md
│   ├── 01-install/
│   │   ├── index.md
│   │   ├── macos-linux.md
│   │   └── windows.md
│   ├── 02-first-conversation/
│   │   └── index.md
│   ├── 03-model-selection/
│   │   └── index.md
│   ├── 04-agent-core/
│   │   └── index.md
│   ├── 05-prompt-system/
│   │   └── index.md
│   ├── 06-memory/
│   │   └── index.md
│   ├── 07-tool-gateway/
│   │   └── index.md
│   ├── 08-terminal-backends/
│   │   └── index.md
│   ├── 09-skills/
│   │   └── index.md
│   ├── 10-learning-loop/
│   │   └── index.md
│   ├── 11-mcp/
│   │   └── index.md
│   ├── 12-messaging-gateway/
│   │   ├── index.md
│   │   ├── telegram.md
│   │   ├── discord.md
│   │   └── slack.md
│   ├── 13-cron/
│   │   └── index.md
│   ├── 14-subagents/
│   │   └── index.md
│   ├── 15-plugins/
│   │   └── index.md
│   ├── 16-api-server/
│   │   └── index.md
│   ├── 17-deployment/
│   │   ├── index.md
│   │   ├── docker.md
│   │   ├── ssh.md
│   │   └── serverless.md
│   ├── 18-security/
│   │   └── index.md
│   └── assets/            # Shared images / diagrams
│       ├── architecture-overview.png
│       └── learning-loop.png
├── .github/
│   └── workflows/
│       └── deploy.yml     # GitHub Actions auto-deploy
└── theme/                 # Optional: custom CSS / JS overrides
    └── custom.css
```

### SUMMARY.md Structure Skeleton

```markdown
# Summary

[소개](README.md)

---

# 기초

- [소개 및 배경](00-intro/index.md)
- [설치 및 환경 설정](01-install/index.md)
  - [macOS / Linux](01-install/macos-linux.md)
  - [Windows / WSL2](01-install/windows.md)
- [첫 번째 대화](02-first-conversation/index.md)
- [모델 선택과 전환](03-model-selection/index.md)

---

# 핵심 개념

- [AIAgent 코어와 에이전트 루프](04-agent-core/index.md)
- [프롬프트 시스템과 컨텍스트 파일](05-prompt-system/index.md)
- [메모리 시스템](06-memory/index.md)
- [툴 게이트웨이](07-tool-gateway/index.md)
- [터미널 백엔드](08-terminal-backends/index.md)

---

# 학습과 자동화

- [스킬 시스템](09-skills/index.md)
- [학습 루프](10-learning-loop/index.md)
- [MCP 통합](11-mcp/index.md)

---

# 플랫폼과 확장

- [메시징 게이트웨이](12-messaging-gateway/index.md)
  - [Telegram](12-messaging-gateway/telegram.md)
  - [Discord](12-messaging-gateway/discord.md)
  - [Slack](12-messaging-gateway/slack.md)
- [크론 스케줄러](13-cron/index.md)
- [서브에이전트](14-subagents/index.md)
- [플러그인과 확장](15-plugins/index.md)
- [API 서버 모드](16-api-server/index.md)

---

# 운영

- [배포](17-deployment/index.md)
  - [Docker](17-deployment/docker.md)
  - [SSH 원격](17-deployment/ssh.md)
  - [서버리스 (Modal / Daytona)](17-deployment/serverless.md)
- [보안과 운영](18-security/index.md)
```

### book.toml Configuration

```toml
[book]
title = "Hermes Agent 한국어 튜토리얼"
authors = ["Hermes Tutorial Contributors"]
language = "ko"
multilingual = false
src = "src"

[output.html]
site-url = "/hermes-tutorial/"   # Must match GitHub Pages repo path
git-repository-url = "https://github.com/<owner>/hermes-tutorial"
edit-url-template = "https://github.com/<owner>/hermes-tutorial/edit/main/{path}"
default-theme = "light"
preferred-dark-theme = "navy"

[output.html.search]
enable = true
limit-results = 20
```

Note: `site-url` **must** be set to `/<repo-name>/` when hosting on `https://<user>.github.io/<repo-name>/`; otherwise the 404 page cannot load static assets.

### GitHub Actions Deploy Workflow (`.github/workflows/deploy.yml`)

```yaml
name: Deploy mdBook to GitHub Pages

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pages: write
      id-token: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Install latest mdbook
        run: |
          tag=$(curl 'https://api.github.com/repos/rust-lang/mdbook/releases/latest' | jq -r '.tag_name')
          url="https://github.com/rust-lang/mdbook/releases/download/${tag}/mdbook-${tag}-x86_64-unknown-linux-gnu.tar.gz"
          mkdir mdbook
          curl -sSL $url | tar -xz --directory=./mdbook
          echo `pwd`/mdbook >> $GITHUB_PATH

      - name: Build Book
        run: mdbook build

      - name: Setup Pages
        uses: actions/configure-pages@v4

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: 'book'

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

**Repository Settings required**: Settings → Pages → Source → select **"GitHub Actions"** (not "Deploy from a branch").

### Build Output Structure

```
book/                      # Generated by mdbook build (gitignored)
├── index.html
├── print.html
├── 404.html
├── searchindex.js
├── 00-intro/
│   └── index.html
├── 01-install/
│   ├── index.html
│   ├── macos-linux.html
│   └── windows.html
... (mirrors src/ structure as HTML)
└── assets/
    └── (images copied verbatim)
```

---

## Architectural Patterns for Tutorial Content

### Pattern 1: 개념 + 실습 혼합형 (Concept + Hands-on Mixed)

**What:** Each chapter opens with a conceptual diagram/explanation of the component, then immediately moves to hands-on CLI steps that exercise the concept.
**When to use:** All chapters (project requirement).
**Trade-offs:** Slightly longer chapters, but readers learn by doing — reduces drop-off.

**Chapter template:**
```markdown
# 챕터 제목

## 개념: [Component Name]이란?
[다이어그램 또는 설명]

## 실습: 직접 해보기

### 사전 준비
[prerequisites]

### 단계별 실행
\`\`\`bash
hermes <command>
\`\`\`
[expected output explanation]

## 심화: 내부 동작 원리
[for readers who want to go deeper]
```

### Pattern 2: Accuracy-Gated Content

**What:** Every CLI command, file path, config key, and API endpoint cites its source (official docs page or repo file path).
**When to use:** Always — accuracy is a core project constraint.
**Trade-offs:** More research time per chapter, but prevents tutorial rot and reader frustration.

### Pattern 3: Progressive Disclosure

**What:** Core concepts (Ch.0–4) are kept lean; advanced internals (agent transports, plugin hooks, API server) appear only after foundations are solid.
**When to use:** Ordering of chapters as defined above.
**Trade-offs:** Advanced readers may skim early chapters; provide jump links.

---

## Anti-Patterns

### Anti-Pattern 1: Inventing CLI Commands

**What people do:** Write `hermes memory flush` or `hermes skills list` without verifying the actual command syntax.
**Why it's wrong:** Hermes CLI commands change across versions; invented commands break the accuracy constraint and erode reader trust.
**Do this instead:** Verify every command against the official docs at `hermes-agent.nousresearch.com/docs` or the `cli.py` source before writing it.

### Anti-Pattern 2: Explaining Architecture Without Source Grounding

**What people do:** Describe internal data flow based on inference rather than the official architecture page or source files.
**Why it's wrong:** Hermes evolves quickly (v0.1 → v0.11+ in one year); inferred architecture may lag reality.
**Do this instead:** Cite `hermes-agent.nousresearch.com/docs/developer-guide/architecture` for all architecture claims; flag LOW confidence where docs are absent.

### Anti-Pattern 3: Chapter Order That Skips Prerequisites

**What people do:** Introduce Skills before explaining the AIAgent loop, or Cron before Gateway.
**Why it's wrong:** Skills make no sense without understanding the tool execution loop; Cron requires Gateway to deliver results.
**Do this instead:** Follow the dependency-aware chapter order (Ch.0→18) defined above.

### Anti-Pattern 4: Flat Chapter Structure Without Sections

**What people do:** Dump all 18 topics as a single flat list in SUMMARY.md.
**Why it's wrong:** Makes navigation overwhelming; readers can't locate their current stage.
**Do this instead:** Group into logical parts (기초 / 핵심 개념 / 학습과 자동화 / 플랫폼과 확장 / 운영) with separators in SUMMARY.md.

---

## Integration Points

### External Services

| Service | Integration Pattern | Notes |
|---------|---------------------|-------|
| Nous Portal | `hermes model` → OAuth flow | Subscription-based; preferred for bundled tools |
| OpenRouter | API key in `~/.hermes/.env` | 200+ models; easy fallback |
| OpenAI / Anthropic | API key in `~/.hermes/.env` | Direct provider; no intermediary |
| Telegram / Discord / Slack | `hermes gateway setup` → bot token | Per-platform bot setup required |
| MCP Servers (external) | `hermes tools` → MCP section, OAuth PKCE | Any MCP-compatible server |
| Modal / Daytona | Terminal backend config | Serverless; requires account |
| agentskills.io Skills Hub | `hermes skills install <name>` | Community skills marketplace |

### Internal Boundaries

| Boundary | Communication | Notes |
|----------|---------------|-------|
| CLI / Gateway / ACP → AIAgent | Direct function call (single process) | Platform-agnostic core by design |
| AIAgent → Tool Registry | Self-registration + dispatch call | Loose coupling via registry pattern |
| AIAgent → Provider Runtime | `(provider, model)` tuple resolution | No hard-coding; swap without code change |
| AIAgent → Session DB | SQLite writes (atomic) | Per-platform isolation; contention handled |
| Cron → Gateway → AIAgent | Scheduler tick → Gateway → spawn | Cron lives inside Gateway process |
| Plugin hooks → AIAgent | `pre_llm_call` / `post_llm_call` callbacks | Optional; zero-cost when absent |
| Subagent → Parent Agent | Python RPC | Isolated process; results returned via RPC |

---

## Sources

| Source | URL | Confidence | Notes |
|--------|-----|------------|-------|
| Official Architecture Docs | https://hermes-agent.nousresearch.com/docs/developer-guide/architecture | HIGH | Primary source for component details |
| GitHub Repo (README) | https://github.com/NousResearch/hermes-agent | HIGH | Repo structure, feature descriptions |
| Turing Post Analysis | https://www.turingpost.com/p/hermes | HIGH | Memory layers, learning loop detail |
| Lushbinary Developer Guide | https://lushbinary.com/blog/hermes-agent-developer-guide-setup-skills-self-improving-ai/ | MEDIUM | Component overview, data flow |
| Kisztof Medium Review | https://kisztof.medium.com/hermes-agent-review-nous-researchs-self-improving-ai-agent-e72bc244435a | MEDIUM | Skills system, learning loop, data flow |
| i-scoop Overview | https://www.i-scoop.eu/hermes-agent-from-nous-research/ | MEDIUM | System overview |
| mudrii hermes-agent-docs | https://github.com/mudrii/hermes-agent-docs | MEDIUM | Community documentation |
| mdBook Official Docs | https://rust-lang.github.io/mdBook/guide/creating.html | HIGH | Source layout, SUMMARY.md format |
| mdBook CI/CD Wiki | https://github.com/rust-lang/mdBook/wiki/Automated-Deployment:-GitHub-Actions | HIGH | Exact GitHub Actions YAML |

---

## Confidence Assessment

| Area | Confidence | Reason |
|------|------------|--------|
| AIAgent core & data flow | HIGH | Official architecture docs page + multiple corroborating sources |
| Component boundaries | HIGH | Official docs + repo structure confirmed |
| Skills system mechanics | HIGH | Multiple detailed sources, v0.10.0 specifics confirmed |
| Memory architecture (4 layers) | HIGH | Turing Post + official docs consistent |
| MCP bidirectional role | HIGH | Official docs + community sources |
| Messaging gateway platforms | HIGH | Official docs list 20+ platforms |
| Subagent/RPC mechanism | MEDIUM | Described but internal RPC protocol not deeply documented |
| Plugin hook signatures | MEDIUM | Documented but full API surface not fully sourced |
| Exact CLI command syntax | MEDIUM | Core commands confirmed; edge cases may shift across versions |
| mdBook SUMMARY.md format | HIGH | Official mdBook docs |
| GitHub Actions deploy YAML | HIGH | mdBook wiki exact YAML retrieved |

## Gaps / LOW-Confidence Areas to Flag

- **Transport layer internals (v0.11.0+)**: `agent/transports/` ABC and individual transport implementations described but not deeply documented in public docs. Flag as MEDIUM in chapters covering provider resolution.
- **Honcho integration depth**: Honcho is described as optional; exact configuration and whether it requires a separate Honcho API account is unclear from public docs. Flag this in Ch.6 and Ch.10.
- **Exact `hermes skills` CLI subcommands**: `skill_manage` tool confirmed; exact CLI syntax for listing/installing/removing skills should be verified against current docs before writing Ch.9.
- **ACP vs MCP distinction**: ACP (editor protocol) and MCP (Model Context Protocol) overlap in function but are documented as separate integrations. Confirm current status of ACP in latest version before writing Ch.11/16.
- **API server auth**: CORS and `Idempotency-Key` documented; full auth/security model for API server mode needs deeper verification for Ch.16.

---
*Architecture research for: Korean mdBook tutorial — Hermes Agent (Nous Research)*
*Researched: 2026-06-05*
