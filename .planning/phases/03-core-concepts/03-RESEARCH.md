# Phase 3: Core Concepts — Research

**Researched:** 2026-06-09
**Domain:** Hermes Agent — Ch.4 에이전트 루프 / Ch.5 컨텍스트 파일 / Ch.6 메모리 / Ch.7 툴 게이트웨이 / Ch.8 터미널 백엔드
**Confidence:** HIGH (loop/prompt/context/memory/tools — all from official docs) / MEDIUM (exact CLI output formats, modal/daytona YAML keys) / LOW (some edge-case config variants)

**Accuracy constraint:** Every command, flag, config key, and expected output must be grounded in official docs or the real repo. Where verification is incomplete, items are marked LOW and flagged `# 검증 필요`. The `# 검증: <source> <version>, YYYY-MM-DD` convention from Phase 2 applies to every code block.

---

## Summary

Phase 3 produces five content chapters (Ch.4–8) that take a reader from "I can run a conversation" (end of Phase 2) to "I understand how the agent loop, prompt system, context files, memory, tools, and backends work — and I can configure them." Research drew on nine primary official doc pages (architecture, agent-loop, prompt-assembly, context-files, memory, memory-providers, tools, toolsets-reference, security), the configuration reference, CLI commands reference, and the session-storage developer guide. The `llms.txt` index revealed two developer-guide pages (`agent-loop`, `prompt-assembly`) not surfaced in prior research — these provide authoritative answers to the "3-tier prompt system" and "agent loop cycle" questions.

**Key corrections/additions to prior established facts:**

1. **Memory file paths corrected:** MEMORY.md and USER.md live at `~/.hermes/memories/MEMORY.md` and `~/.hermes/memories/USER.md` (not `~/.hermes/` directly). SOUL.md stays at `~/.hermes/SOUL.md`.
2. **Prompt system is explicitly 3-tier:** Official prompt-assembly docs confirm a Stable → Context → Volatile three-tier architecture assembled in a documented 10-layer order.
3. **Agent loop named:** Official agent-loop doc names the cycle "Assemble → Call → Parse → Execute → Persist → Loop" — there is no "perceive→decide→act" framing in the official docs.
4. **session_search is an agent tool, not a slash command:** The agent calls it internally when context warrants; readers cannot `/session_search` directly.
5. **FTS5 trigram table (v10+):** `messages_fts_trigram` specifically enables CJK (Korean included) and substring matching — valuable callout for Korean audience.
6. **Tool Gateway paid boundary clarified:** Free accounts get a "free tool pool" (limited allowance auto-surfaced on first use); paid subscription unlocks full Tool Gateway. The doc does not specify the free pool quota.
7. **/personality built-in count corrected:** 14 built-in personalities confirmed (not 13 from prior research) — the teacher/helpful/concise/technical/creative group plus the novelty group.
8. **`agent.disabled_toolsets` key does NOT exist:** Tool disabling uses platform toolsets array or `hermes tools` interactive UI, not an `agent.disabled_toolsets` key.
9. **Compression protect_first_n is hardcoded 3**, not configurable. protect_last_n default 20 IS configurable.
10. **Docker security flags are applied by Hermes automatically** (--cap-drop ALL, --pids-limit 256, etc.) — not the reader's responsibility.

**Primary recommendation:** Each chapter uses the Phase 2 "facts → exact commands → expected output → pitfall callouts" structure. Ch.4 and Ch.5 are conceptual-heavy; pair with diagrams (ASCII-art is sufficient in mdBook). Ch.6–8 are more hands-on. Apply the owner-plan + parallel-body-plan wave structure to avoid SUMMARY.md write races (same pattern as Phase 2).

---

## Chapter Structure Guidance

### Updated SUMMARY.md entries

Current SUMMARY.md (end of Phase 2):

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

After Phase 3, add the "# 핵심 개념" section:

```markdown
# Summary

[소개](README.md)

---

# 기초

- [소개: Hermes란 무엇인가](00-intro/index.md)
- [설치](01-install/index.md)
- [첫 실행](02-first-run/index.md)
- [모델 설정](03-model/index.md)

---

# 핵심 개념

- [에이전트 루프](04-agent-loop/index.md)
- [컨텍스트 파일](05-context-files/index.md)
- [메모리](06-memory/index.md)
- [툴 게이트웨이](07-tools/index.md)
- [터미널 백엔드](08-backends/index.md)
```

### New src/ directories required

```
src/
├── 04-agent-loop/
│   └── index.md
├── 05-context-files/
│   └── index.md
├── 06-memory/
│   └── index.md
├── 07-tools/
│   └── index.md
└── 08-backends/
    └── index.md
```

**mdBook duplicate-path build error warning (Pitfall B-11):** The section separator (`---`) and the new `# 핵심 개념` heading are ADDITIONS — they must not duplicate any existing entry path. The five new paths are entirely new and safe.

### Plan / wave structure (reuse Phase 2 pattern)

- **Plan A (owner):** Updates SUMMARY.md with the new section + 5 entries, creates all five `src/NN/index.md` skeleton files (one-line placeholder). Runs `mdbook build` to verify no duplicate-path error.
- **Plans B–F (body, run in parallel after Plan A):** Each writes the body of one chapter file. Since Plan A already owns the file, Plans B–F overwrite (not create) the skeleton — no write race on SUMMARY.md.

---

## Ch.4: 에이전트 루프 (AIAgent Core Loop)

### Facts to State (HIGH confidence — from official agent-loop and architecture docs)

#### The AIAgent Core

`AIAgent` in `run_agent.py` is the platform-agnostic orchestration engine used by all entry points (CLI, Gateway, ACP, API). It is a **synchronous** engine — one conversation turn runs to completion before the next begins. All entry points are concurrent; the agent itself is not.

#### The Official Cycle Name

The official developer docs name the cycle: **Assemble → Call → Parse → Execute → Persist → Loop**

There is NO "perceive→decide→act" framing in the official Hermes docs. Do not use that framing in the chapter — it will confuse readers who check the official sources.

#### Exact Agent Loop Steps (per `run_conversation()`)

From the agent-loop developer guide:

```
1. Generate task_id (if absent)
2. Append user message to conversation history
3. Build / reuse cached system prompt (if not yet built)
4. Preflight compression check (if >50% context used → compress before API call)
5. Build API messages from conversation history
6. Inject ephemeral prompt layers (budget warnings, etc.)
7. Apply prompt caching markers (Anthropic only)
8. Make interruptible API call (streaming)
9. Parse response:
   - If tool_calls → execute tools, append results, GOTO step 5
   - If text response → persist session, flush memory, return final_response
```

#### Three API Modes (HIGH confidence)

| Mode | Endpoint type | Client library |
|------|--------------|----------------|
| `chat_completions` | OpenAI-compatible (OpenRouter, custom) | `openai.OpenAI` |
| `codex_responses` | OpenAI Codex / Responses API | `openai.OpenAI` with Responses format |
| `anthropic_messages` | Native Anthropic Messages API | `anthropic.Anthropic` adapter |

All three converge on the same internal OpenAI-style message format.

#### Tool Dispatch in the Loop (HIGH confidence)

- **Single tool call:** Direct execution (synchronous)
- **Multiple tool calls:** `ThreadPoolExecutor` (max 8 workers) — concurrent execution
- **Exception — interactive tools:** `clarify`, `todo`, `memory`, `session_search`, `delegate_task` always run sequentially (they require user interaction or maintain state)

Execution flow per tool:
1. Resolve handler from Tool Registry
2. Fire `pre_tool_call` plugin hook
3. Check if dangerous → approval gate
4. Execute with args + task_id
5. Fire `post_tool_call` plugin hook
6. Append tool result to conversation history

#### Loop Termination Conditions

| Condition | What happens |
|-----------|-------------|
| Text response received | Returns `final_response`, persists session |
| 90 iterations budget exhausted | Returns work summary (default `agent.max_turns: 90`) |
| User sends `/stop` | Abandons active API thread, discards pending response |

#### Callback Surfaces (for "심화" section only)

Eight callback hooks fire during execution: `tool_progress_callback`, `thinking_callback`, `reasoning_callback`, `clarify_callback`, `step_callback`, `stream_delta_callback`, `tool_gen_callback`, `status_callback`. These are entry-point-specific (CLI uses them for display; Gateway uses them for platform delivery). Mention in passing only — this is developer-level detail.

#### Fallback Behavior (MEDIUM — mention briefly)

On 429 / 5xx / 401 errors, the agent checks `fallback_providers` list and tries each in order. Auxiliary tasks (vision, compression) have independent fallback chains.

### Exact Commands for Ch.4 (HIGH confidence)

Ch.4 is conceptual — NO new install commands. The chapter's hands-on is observation-only:

```bash
# 검증: hermes rolling, 2026-06-09
# 에이전트 루프를 직접 관찰하려면:
hermes chat -q "현재 디렉터리의 파일을 나열하고 가장 큰 파일을 알려줘"
```

Expected behavior: reader sees tool_progress output showing `terminal` tool being called, result appended, then a text response. This makes the loop concrete without requiring explanation of internals.

```bash
# 검증: hermes rolling, 2026-06-09
# 반복 횟수 제한 설정 (기본값 90):
hermes config set agent.max_turns 50
```

### Expected Outputs (LOW — verify locally)

The tool call output format (the progress display during tool execution) depends on `display.tool_progress` setting and terminal skin. Use placeholder:

```
# [로컬 실행 후 캡처 필요 — 툴 호출 진행 출력]
```

### ASCII Diagram for Ch.4

Include this loop diagram in the chapter (accurate per official docs):

```
사용자 입력
     │
     ▼
task_id 생성
     │
     ▼
시스템 프롬프트 조립 (3티어)
     │
     ├──► 50% 초과? → 컨텍스트 압축 먼저 실행
     │
     ▼
API 메시지 구성 (대화 이력 + 에페머럴 레이어)
     │
     ▼
LLM API 호출 (스트리밍, 인터럽트 가능)
     │
     ├── tool_calls 있음? ──► 툴 디스패치
     │                          ├── 단일: 직접 실행
     │                          └── 복수: ThreadPoolExecutor(≤8)
     │                          │
     │                          └──► 결과를 이력에 추가
     │                                    │
     │                          ◄─────────┘ (루프)
     │
     └── 텍스트 응답 ──► 세션 저장 + 메모리 플러시 → 완료
```

### Pitfall Callouts for Ch.4

**주의 1 — 동기 엔진의 의미 (MEDIUM):**

```
> 참고: AIAgent는 동기(synchronous) 엔진입니다.
> 한 번의 대화 턴이 완전히 완료된 후에야 다음 입력을 받습니다.
> 응답이 느리게 느껴진다면, 도구가 실행 중인 것이며 루프가 작동하고 있는 것입니다.
```

**주의 2 — max_turns 소진 (HIGH):**

```
> 주의: agent.max_turns (기본 90) 를 초과하면 에이전트가 작업 요약을 반환하고 중단됩니다.
> 복잡한 장기 작업의 경우 max_turns를 늘리거나 서브에이전트(delegate_task)를 활용하십시오.
```

---

## Ch.5: 컨텍스트 파일 (Context Files & Prompt System)

### Facts to State

#### The 3-Tier Prompt Architecture (HIGH confidence — from official prompt-assembly doc)

The official name is the **"3-tier cached system prompt"**. The tiers are:

| 티어 | 내용 | 캐시 안정성 |
|------|------|------------|
| **Stable** (안정 계층) | 에이전트 정체성(SOUL.md), 툴 가이던스, 스킬 인덱스, 환경 힌트 | 세션 간 높음 — 캐시 prefix로 유지 |
| **Context** (컨텍스트 계층) | 선택적 system_message + 프로젝트 컨텍스트 파일 (단 1개) | 프로젝트 변경 시에만 바뀜 |
| **Volatile** (휘발성 계층) | MEMORY.md 스냅샷 + USER.md 스냅샷 + 타임스탬프 + 세션 메타 | 매 세션마다 변경 가능 |

**Final assembly order:** `stable` → `context` → `volatile`

#### Full 10-Layer Composition Order (HIGH confidence)

From the prompt-assembly developer guide:

```
1.  에이전트 정체성    — SOUL.md (또는 DEFAULT_AGENT_IDENTITY 폴백)
2.  툴 인식 행동 가이드 — memory 툴 사용법, session_search 프롬프팅
3.  툴 사용 강제      — GPT/Codex 전용 지시어 (실제로 툴을 사용하게 강제)
4.  선택적 system_message — 사용자 설정 오버라이드 (있는 경우)
5.  MEMORY 스냅샷    — ~/.hermes/memories/MEMORY.md (세션 시작 시 동결)
6.  USER 프로파일 스냅샷 — ~/.hermes/memories/USER.md (세션 시작 시 동결)
7.  스킬 인덱스       — 컴팩트 스킬 디렉터리 (skill_view() 지시어 포함)
8.  컨텍스트 파일     — 프로젝트 수준 지시어 (우선순위 1개)
9.  타임스탬프 + 세션 메타 — 현재 시간, 세션 ID
10. 플랫폼 힌트       — CLI 렌더링 가이드
```

**"Not cached" ephemeral layers** (not in the above 10): `ephemeral_system_prompt`, prefill messages, gateway session overlays, plugin `pre_llm_call` context (appended to user message, not system prompt).

**Key insight for Korean readers:** Layers 1–3 and 5–8 form the "stable prefix" for Anthropic prompt caching. This is why the cache is effective — SOUL.md, skills, and memory rarely change.

#### Context File Hierarchy & Precedence (HIGH confidence)

**Priority cascade (first match wins per session — only ONE loads):**

| 우선순위 | 파일 이름 | 탐색 범위 | 용도 |
|----------|-----------|-----------|------|
| 1 | `.hermes.md` / `HERMES.md` | CWD → git root (상위 탐색) | Hermes 전용 프로젝트 지시어 |
| 2 | `AGENTS.md` | CWD 전용 | 일반 에이전트 지시어 |
| 3 | `CLAUDE.md` | CWD 전용 | Claude Code 호환 파일 |
| 4 | `.cursorrules` / `.cursor/rules/*.mdc` | CWD 전용 | Cursor IDE 호환 파일 |

**IMPORTANT:** `.hermes.md` searches UP to the git root; `AGENTS.md`, `CLAUDE.md`, `.cursorrules` are CWD-only.

**Subdirectory AGENTS.md (progressive discovery):**
- Root context file loads at startup (into system prompt)
- As the agent navigates into subdirectories during tool calls (`read_file`, `terminal`, `search_files`), it discovers AGENTS.md files in those subdirs lazily
- Each subdir checked at most once per session
- Discovery walks UP parent dirs (reading `backend/src/main.py` finds `backend/AGENTS.md`)
- Prevents system prompt bloat + preserves prompt cache stability

**SOUL.md distinction:**
- Lives at `~/.hermes/SOUL.md` (or `$HERMES_HOME/SOUL.md`) — NEVER project directory
- Loads as layer 1 (agent identity slot) — before everything else
- Seeds default if absent; never overwritten if present
- Global: same personality across all projects

#### Context File Contents & Processing

- Character limit: 20,000 chars (truncated to 70% head + 20% tail if exceeded)
- Security scan: patterns like "ignore previous instructions" or `curl ... $API_KEY` trigger a block
- YAML frontmatter stripped before injection
- Injected under `# Project Context` header

#### `/personality` Command (HIGH confidence — 14 built-in)

```
/personality [name]
```

**Built-in personalities (14 total):**

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

**Session-scoped:** `/personality` is reset at session end — does NOT modify SOUL.md.

**Custom personalities in config.yaml:**

```yaml
# 검증: hermes rolling, 2026-06-09
agent:
  personalities:
    codereviewer: >
      You are a meticulous code reviewer. Identify bugs, security issues,
      performance concerns, and unclear design choices. Be precise and constructive.
```

Switch with: `/personality codereviewer`

#### .env vs config.yaml Separation (HIGH confidence)

**Resolution order (highest to lowest):**
1. CLI arguments (`--model`, `--backend`)
2. `~/.hermes/config.yaml` (primary non-secret config)
3. `~/.hermes/.env` (secrets)
4. Built-in defaults

**Separation rule:**

| 파일 | 저장 내용 | 예시 |
|------|-----------|------|
| `config.yaml` | 비밀 외 설정 | 모델, 압축 임계값, 디스플레이, 백엔드 타입 |
| `.env` | **비밀만** | API 키, 봇 토큰, OAuth 자격증명 |
| `auth.json` | OAuth 자격증명 | OAuth refresh token |

**`hermes config set` routing:**

```bash
# 검증: hermes rolling, 2026-06-09
hermes config set model anthropic/claude-opus-4      # → config.yaml
hermes config set OPENROUTER_API_KEY sk-or-...       # → .env 자동 라우팅
hermes config set terminal.backend docker            # → config.yaml
```

**Environment variable substitution in config.yaml:**

```yaml
# 검증: hermes rolling, 2026-06-09
auxiliary:
  vision:
    api_key: ${GOOGLE_API_KEY}   # ← 중괄호 필수 (A-19 Pitfall 참조)
```

### Exact Commands for Ch.5 — Hands-on: .hermes.md 만들기

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

**SOUL.md 확인/편집:**

```bash
# 검증: hermes rolling, 2026-06-09
# SOUL.md 현재 내용 확인
cat ~/.hermes/SOUL.md

# SOUL.md 편집 (기본 에디터)
$EDITOR ~/.hermes/SOUL.md
```

**현재 설정 확인:**

```bash
# 검증: hermes rolling, 2026-06-09
hermes config show
```

### Expected Outputs

**`.hermes.md` 로드 확인 (LOW — verify locally):**

```
# [hermes 시작 배너에서 컨텍스트 파일 로드 메시지 확인 필요 — 로컬 캡처 필요]
```

The exact message (if any) indicating a context file was loaded is not documented; verify locally before publishing.

### Pitfall Callouts for Ch.5

**주의 1 — context 파일은 세션 시작 시 한 번만 로드됨 (HIGH):**

```
> 주의: .hermes.md를 실행 중인 세션에서 수정해도 현재 세션에는 반영되지 않습니다.
> 변경 사항은 다음 세션(hermes 재시작)부터 적용됩니다.
```

**주의 2 — 우선순위: 첫 번째 매치만 로드됨 (HIGH):**

```
> 주의: 같은 디렉터리에 .hermes.md와 AGENTS.md가 모두 있으면
> .hermes.md만 로드됩니다 (우선순위 1번이므로).
> 두 파일을 동시에 로드하려면 내용을 .hermes.md로 합쳐야 합니다.
```

**주의 3 — API 키는 config.yaml에 저장하지 말 것 (A-4 반복):**

```
> 경고: ${VAR} 문법으로 참조하더라도 API 키는 config.yaml이 아닌
> .env 파일에 저장해야 합니다. 이 분리는 git 노출 방지를 위해 중요합니다.
```

**주의 4 — 변수 치환 문법 (A-19):**

```
> 주의: config.yaml에서 환경변수를 참조할 때는 반드시 ${VAR_NAME} 형식을 사용하십시오.
> $VAR_NAME (중괄호 없음)은 치환되지 않고 문자열로 그대로 사용됩니다.
```

---

## Ch.6: 메모리 (Memory System)

### Facts to State

#### Four-Layer Memory Architecture (HIGH confidence)

| 레이어 | 저장 위치 | 용량 | 로딩 방식 |
|--------|-----------|------|-----------|
| **Persistent Core Memory** | `~/.hermes/memories/MEMORY.md` + `USER.md` | MEMORY: ~2,200자(~800토큰), USER: ~1,375자(~500토큰) → 합계 ~1,300토큰 | 세션 시작 시 항상 동결 스냅샷으로 로드 |
| **Session History (FTS5)** | `~/.hermes/state.db` (SQLite) | 무제한 | `session_search` 툴로 온디맨드 쿼리 |
| **External Provider** | Honcho/Mem0/Hindsight 등 (선택 사항) | 제공자마다 다름 | Phase 4 범주 — 이 챕터에서는 이름 언급만 |
| **Procedural (Skills)** | `~/.hermes/skills/*.md` | 매칭 시 주입 | Phase 4 범주 |

#### Exact File Paths (HIGH confidence — corrected from prior research)

```
~/.hermes/
├── SOUL.md                    # 에이전트 정체성 (컨텍스트 파일과 다름)
├── memories/
│   ├── MEMORY.md              # 에이전트 메모 (2,200자 한도)
│   └── USER.md                # 사용자 프로파일 (1,375자 한도)
├── state.db                   # SQLite FTS5 세션 이력
└── skills/                    # 절차적 메모리 (Phase 4)
```

**IMPORTANT:** The `/memories/` subdirectory is confirmed by the official memory docs AND by the developer guide schema for state.db (separate path). Do NOT write `~/.hermes/MEMORY.md` — that is incorrect.

#### Built-in Memory Tool (HIGH confidence)

The agent uses the `memory` tool internally (it can also be triggered by asking the agent to remember something). Three operations:

| 동작 | 용도 |
|------|------|
| `add` | 새 메모리 항목 추가 |
| `replace` | 기존 항목 수정 (old_text 부분 문자열 매칭) |
| `remove` | 항목 삭제 (부분 문자열로 식별) |

Entries are separated by `§` (section sign, U+00A7) delimiters within the file.

**What to save vs. skip (official curation heuristics):**

| 저장 대상 | 건너뛸 내용 |
|-----------|-------------|
| 사용자 선호도 | 사소한 사실 |
| 환경 설정 | 코드 덤프 |
| 프로젝트 규약 | 세션 특화 내용 |
| 배운 해결책 / 우회법 | SOUL.md에 이미 있는 내용 |
| 완료된 작업 | 일시적 컨텍스트 |

**Capacity management:** When at limit, the memory tool returns an error. The agent should consolidate at ~80% capacity. The reader can manually edit the files if needed.

**Security:** Memory entries are scanned for prompt injection and credential exfiltration patterns.

#### Memory Flush (MEDIUM — behavior described but exact timing varies)

Memory changes (add/replace/remove) are **persisted to disk immediately** during the session but do NOT appear in the system prompt until the NEXT session (the snapshot is frozen at session start). This is a critical reader gotcha.

#### Session Search — `session_search` Tool (HIGH confidence)

`session_search` is an **agent tool** — the agent calls it internally when context warrants recall. Readers CANNOT invoke it as a slash command. To trigger it, ask the agent a question like "지난주에 우리가 X에 대해 얘기한 내용이 뭐였지?" — the agent will call `session_search` autonomously.

**What it does:**
- Queries `~/.hermes/state.db` via SQLite FTS5
- Three calling patterns: discovery (browse available sessions), scroll (move within a session), browse (search across sessions)
- Returns **raw messages** — no LLM summarization, no truncation
- ~20ms FTS5 query, ~1ms scroll (extremely fast, zero LLM cost)
- Always available — no configuration needed

**FTS5 tables in state.db (confirmed from session-storage developer guide):**

| 테이블 | 목적 |
|--------|------|
| `messages_fts` | 표준 FTS5 전문 검색 |
| `messages_fts_trigram` | 트라이그램 토크나이저 — **한국어(CJK)** 및 부분 문자열 검색 |

**Korean audience callout:** `messages_fts_trigram` was added in v10+ specifically for CJK/substring search. This means Korean content in conversation history is fully searchable. The FTS5 rebuild in v0.15.0 (mentioned in established facts as "4,500x faster") applies to these tables.

**Direct DB query (for advanced readers — verify locally before publishing):**

```bash
# 검증: SQLite 로컬 실행 필요, 2026-06-09
# 예시: 특정 키워드가 포함된 과거 대화 검색
sqlite3 ~/.hermes/state.db \
  "SELECT created_at, role, snippet(messages_fts, 2, '**', '**', '...', 20) 
   FROM messages_fts 
   WHERE messages_fts MATCH 'deploy'
   ORDER BY created_at DESC LIMIT 10"
```

**Note:** The exact column names in the snippet above (`created_at`, `role`) are from the official session-storage schema — HIGH confidence. The FTS5 snippet function syntax is standard SQLite — HIGH confidence.

#### Memory Configuration in config.yaml (HIGH confidence)

```yaml
# 검증: hermes rolling, 2026-06-09
memory:
  memory_enabled: true
  user_profile_enabled: true
  memory_char_limit: 2200     # MEMORY.md 최대 문자 수 (~800토큰)
  user_char_limit: 1375       # USER.md 최대 문자 수 (~500토큰)
```

#### Compression Interaction (HIGH confidence — key cost concern)

Compression does NOT directly affect memory files. However:
- The fixed ~1,300 token overhead from memory is included in EVERY API call
- Compression removes MIDDLE turns but keeps the memory snapshot in the system prompt
- This is why cost-per-call does not shrink below ~1,300 tokens even after compression

**Cost management config (from compression docs):**

```yaml
# 검증: hermes rolling, 2026-06-09
compression:
  enabled: true
  threshold: 0.50      # 컨텍스트의 50% 사용 시 압축 트리거
  target_ratio: 0.20   # 압축 후 최근 20% 유지
  protect_last_n: 20   # 최근 20개 메시지 항상 보존 (기본값, 변경 가능)
  # protect_first_n: 3  ← 하드코딩됨, 설정 불가
```

**Secondary safety net:** Gateway sessions trigger a secondary compressor at 85% (in addition to the 50% primary threshold). This prevents API failures in long gateway sessions.

**Auxiliary compression model:**

```yaml
# 검증: hermes rolling, 2026-06-09
auxiliary:
  compression:
    model: ""        # 빈 칸 = 메인 모델 사용 (저비용 모델 지정 가능)
    provider: "auto"
    base_url: null
```

**Gateway token cost warning (from PITFALLS.md A-9):** Gateway sessions with 2h light use can burn ~4M input tokens. Setting `compression.threshold: 0.30` (instead of default 0.50) and using a cheap auxiliary compression model (e.g., Gemini Flash) significantly reduces this.

#### Slash Commands for Memory (HIGH confidence)

| 슬래시 커맨드 | 기능 |
|--------------|------|
| `/compress` | 수동으로 컨텍스트 압축 실행 |
| `/usage` | 현재 세션 토큰 사용량 표시 |

**Memory management CLI commands:**

```bash
# 검증: hermes rolling, 2026-06-09
hermes memory setup      # 외부 메모리 프로바이더 설정 (Honcho, Mem0 등)
hermes memory status     # 현재 메모리 프로바이더 확인
hermes memory off        # 외부 프로바이더 비활성화 (내장 메모리만 사용)
```

**Direct file viewing (always works):**

```bash
# 검증: hermes rolling, 2026-06-09
cat ~/.hermes/memories/MEMORY.md
cat ~/.hermes/memories/USER.md
```

**Session management CLI:**

```bash
# 검증: hermes rolling, 2026-06-09
hermes sessions list                    # 최근 세션 목록
hermes sessions browse                  # 인터랙티브 세션 탐색기
hermes sessions stats                   # state.db 통계
hermes sessions prune                   # 오래된 세션 삭제
```

### Pitfall Callouts for Ch.6

**주의 1 — 메모리 스냅샷은 세션 시작 시 동결됨 (HIGH — 가장 흔한 혼란):**

```
> 주의: 세션 중 에이전트가 MEMORY.md를 업데이트해도 현재 대화에는 반영되지 않습니다.
> 메모리 변경 사항은 다음 hermes 실행 시부터 시스템 프롬프트에 포함됩니다.
> 이것은 의도된 동작입니다 — 세션 내 컨텍스트 일관성을 보장합니다.
```

**주의 2 — 게이트웨이 토큰 비용 급증 (A-9, CRITICAL — 컨텍스트 압축 미이해):**

```
> 경고: Telegram/Discord 게이트웨이를 통한 2시간 사용으로
> Claude Sonnet 기준 약 400만 토큰 (~$12)이 소비될 수 있습니다 (GitHub Issue 보고).
> 기본 압축 임계값(50%)이 이 비용을 유발합니다.
>
> 비용 절감 방법:
> 1. compression.threshold: 0.30 으로 낮추기
> 2. /compress 로 수동 압축
> 3. 저비용 보조 모델 설정: auxiliary.compression.model: "google/gemini-flash-1.5"
> 4. /usage 로 현재 토큰 사용량 모니터링
```

**주의 3 — session_search는 슬래시 커맨드가 아님 (MEDIUM):**

```
> 참고: session_search는 에이전트가 자동으로 호출하는 내부 툴입니다.
> /session_search 같은 슬래시 커맨드는 없습니다.
> 과거 대화를 검색하려면 에이전트에게 자연어로 물어보세요:
> "지난달에 우리가 배포에 대해 얘기한 내용 기억해?"
```

**주의 4 — 보조 모델 컨텍스트 윈도우 (A-14):**

```
> 주의: auxiliary.compression.model에 메인 모델보다 작은
> 컨텍스트 윈도우의 모델을 지정하면 요약 중 중요 내용이
> 오류 없이 조용히 잘릴 수 있습니다.
> 저비용 요약 모델도 반드시 큰 컨텍스트 윈도우 모델을 선택하십시오
> (예: Gemini Flash — 1M 컨텍스트).
```

---

## Ch.7: 툴 게이트웨이 (Tool Gateway)

### Facts to State

#### Tool Registry Overview (HIGH confidence)

Hermes has **~71 built-in tools** across **30+ toolsets**. The tools registered depends on installed capabilities and active platforms.

**Key toolsets by category (HIGH confidence — from tools-reference):**

| 카테고리 | 툴셋 | 주요 툴 |
|----------|------|---------|
| 파일 & 터미널 | `terminal`, `file` | `terminal`, `process`, `read_file`, `write_file`, `patch`, `search_files` |
| 웹 | `web`, `browser` | `web_search`, `web_extract`, `browser_navigate`, `browser_click`, `browser_snapshot`, `browser_vision` |
| AI | `vision`, `image_gen`, `tts`, `moa` | `vision_analyze`, `image_generate`, `text_to_speech`, `mixture_of_agents` |
| 에이전트 | `delegation`, `clarify`, `todo`, `kanban` | `delegate_task`, `clarify`, `todo`, `kanban_show` |
| 메모리 | `memory`, `session_search`, `skills` | `memory`, `session_search`, `skill_manage`, `skill_view` |
| 자동화 | `cronjob`, `messaging` | `cronjob`, `send_message` |
| 특수 | `homeassistant`, `computer_use`, `spotify` | `ha_call_service`, `computer_use`, `spotify_playback` |
| 플랫폼 | `discord`, `feishu_doc`, `feishu_drive`, `x_search`, `yuanbao` | `discord`, `feishu_doc_read`, `x_search` |

**Kanban toolset is opt-in only** — NOT included even with `all` or `*` wildcard.

#### `hermes tools` Command (HIGH confidence)

```bash
# 검증: hermes rolling, 2026-06-09
hermes tools               # 플랫폼별 인터랙티브 툴 설정 UI (curses 기반)
hermes tools --summary     # 현재 활성화된 툴 요약 출력 후 종료
```

**In-session tool commands:**

```
/tools list                # 현재 세션 활성 툴 목록
/tools disable browser     # 브라우저 툴셋 비활성화
/tools enable homeassistant # Home Assistant 툴셋 활성화
```

**Per-session toolset selection (CLI flag):**

```bash
# 검증: hermes rolling, 2026-06-09
hermes chat --toolsets web,file,terminal   # 특정 툴셋만 활성화
hermes chat --toolsets debugging           # composite toolset (file + terminal + web)
hermes chat --toolsets all                 # 모든 툴셋 (kanban 제외)
```

**Custom toolsets in config.yaml:**

```yaml
# 검증: hermes rolling, 2026-06-09
custom_toolsets:
  data-science:
    - file
    - terminal
    - code_execution
    - web
    - vision
```

**Toolset disabling via config (NOT `agent.disabled_toolsets` — that key does NOT exist):**

```yaml
# 검증: hermes rolling, 2026-06-09
# CLI 기본 플랫폼의 툴셋을 제한하려면 toolsets 배열을 직접 지정:
toolsets:
  - hermes-cli   # 기본 전체 툴셋 대신 필요한 것만 나열
```

#### Tool Concurrency (HIGH confidence — from agent-loop doc)

- **Multiple tool calls → `ThreadPoolExecutor` (max 8 workers)**
- Path-scoped concurrency safety: concurrent tool calls on different file paths are safe; the registry manages ordering for same-path operations
- **Interactive tools always run sequentially**: `clarify`, `todo`, `memory`, `session_search`, `delegate_task`

#### Nous Tool Gateway — Paid Feature (HIGH confidence)

The Nous Tool Gateway routes web search, image generation, TTS, and browser automation through a subscription service (no need for individual API keys per service).

**Free vs. Paid boundary:**

| 기능 | 무료 계정 | 유료 구독 |
|------|-----------|-----------|
| 추론 (LLM) | 가능 (300+ 모델) | 가능 |
| Tool Gateway (웹검색, 이미지, TTS, 브라우저) | **무료 툴 풀** (자동 부여, 한도 있음) | **전체 무제한** |
| 한도 초과 시 | 툴 오류 | 해당 없음 |

**"Free tool pool":** Some accounts receive a limited free allowance automatically surfaced on first use (exact quota not documented — verify on Nous Portal pricing page before publishing this claim).

**Recommendation:** For the tutorial, state the paid boundary clearly with a `> 주의` callout and recommend readers check current pricing at `hermes-agent.nousresearch.com`.

**Nous Portal commands (HIGH confidence):**

```bash
# 검증: hermes rolling, 2026-06-09
hermes setup --portal      # OAuth 로그인 + Nous 공급자 설정 + Tool Gateway 활성화
hermes portal info         # 로그인 상태 + 구독 + 게이트웨이 라우팅 요약
hermes portal tools        # Tool Gateway 카탈로그 (툴별 라우팅 현황)
```

#### Tool Gateway per-tool `use_gateway` Config (HIGH confidence)

```yaml
# 검증: hermes rolling, 2026-06-09
web:
  backend: firecrawl
  use_gateway: true    # true = Nous Gateway 라우팅 / false = 직접 API 키 사용
```

`use_gateway: true` overrides direct keys regardless of which is set.

#### Built-in Tool Gateway Capabilities (HIGH confidence)

**Web search:** Firecrawl (agent-grade, no rate limits via gateway)

**Image generation** (9 models via FAL.ai):

| 모델 ID | 이름 |
|---------|------|
| `fal-ai/flux-2/klein/9b` | FLUX 2 Klein 9B (기본값) |
| `fal-ai/flux-2-pro` | FLUX 2 Pro |
| `fal-ai/z-image/turbo` | Z-Image Turbo |
| `fal-ai/nano-banana-pro` | Nano Banana Pro (Gemini 3 Pro) |
| `fal-ai/gpt-image-1.5` | GPT Image 1.5 |
| `fal-ai/gpt-image-2` | GPT Image 2 |
| `fal-ai/ideogram/v3` | Ideogram V3 |
| `fal-ai/recraft/v4/pro/text-to-image` | Recraft V4 Pro |
| `fal-ai/qwen-image` | Qwen Image |

**Text-to-speech:** OpenAI TTS voices wired into `text_to_speech` tool (Telegram voice notes, audio pipelines)

**Browser automation:** Headless Chromium sessions via Browser Use. Primitives: `browser_navigate`, `browser_click`, `browser_type`, `browser_vision`. No Browserbase account required when using the gateway.

#### Approval Mode (HIGH confidence — from security docs)

```yaml
# 검증: hermes rolling, 2026-06-09
approvals:
  mode: manual         # manual (기본) | smart | off
  timeout: 60          # 승인 대기 시간 (초)
  cron_mode: deny      # 헤드리스 cron: deny | approve
  destructive_slash_confirm: true  # /clear, /new, /reset 전 확인
```

| 모드 | 동작 |
|------|------|
| `manual` | 위험한 명령 전 항상 사용자 승인 요청 (기본, 가장 안전) |
| `smart` | 보조 LLM이 위험도 평가; 저위험 자동 승인, 고위험 자동 거부, 불확실 수동 위임 |
| `off` | 모든 승인 체크 비활성화 (`HERMES_YOLO_MODE=true`와 동일) |

**Hardline blocklist (off 모드에서도 항상 차단):** `rm -rf /`, 포크 폭탄, `mkfs.*` on mounted root 등 — 이 명령들은 어떤 설정에서도 실행 거부.

**Tirith security scanning (always-on by default):**

```yaml
# 검증: hermes rolling, 2026-06-09
security:
  tirith_enabled: true       # 기본값: true
  tirith_timeout: 5          # 타임아웃 (초)
  tirith_fail_open: true     # 스캐너 실패 시 실행 허용
```

Tirith detects: homograph URL spoofing, pipe-to-interpreter patterns (`curl | bash`), terminal injection attacks.

### Pitfall Callouts for Ch.7

**주의 1 — approvals.mode: off 위험성 (A-12, CRITICAL):**

```
> 경고: approvals.mode: off (또는 HERMES_YOLO_MODE=true)는
> 위험한 명령에 대한 모든 안전 프롬프트를 비활성화합니다.
> 로컬 백엔드에서 절대 사용하지 마십시오.
> 컨테이너 백엔드(Docker/Modal/Daytona)에서 컨테이너가 보안 경계 역할을 할 때만 합리적입니다.
> 활성화 시 빨간색 배너 "⚠ YOLO mode"가 표시됩니다.
```

**주의 2 — Tool Gateway 무료/유료 경계 (A-7):**

```
> 주의: Nous Portal 무료 계정은 추론(LLM 대화)에 사용 가능하지만,
> Tool Gateway(웹 검색, 이미지 생성, TTS, 브라우저)는 유료 구독 또는
> 무료 툴 풀 한도 내에서만 사용 가능합니다.
> hermes portal info 로 현재 구독 상태를 확인하십시오.
> 구독 없이 도구 호출 시 "subscription required" 오류가 발생합니다.
```

**주의 3 — agent.disabled_toolsets 설정 키 없음 (MEDIUM — 흔한 오해):**

```
> 참고: 툴셋 비활성화는 config.yaml의 agent.disabled_toolsets 키가 아닌
> toolsets 배열 또는 hermes tools 인터랙티브 UI를 통해 설정합니다.
> 존재하지 않는 설정 키는 조용히 무시됩니다.
```

---

## Ch.8: 터미널 백엔드 (Terminal Backends)

### Facts to State

Ch.8 is an **overview** chapter — deep deployment content is in Phase 5. The goal: reader understands what each backend is for and can switch to Docker for local sandboxing.

#### The Six Backends (HIGH confidence)

| 백엔드 | 격리 수준 | 파일 지속성 | 주요 용도 | 환경 요구사항 |
|--------|-----------|-------------|-----------|---------------|
| `local` | 없음 (호스트 직접) | 호스트 파일시스템 | 개발/테스트 기본값 | 없음 |
| `docker` | 강함 (cap-drop ALL, pids-limit 256) | 컨테이너 `/opt/data` | 프로덕션, 보안 중요 환경 | Docker 데몬 실행 중 |
| `ssh` | 네트워크 경계 | 원격 서버 파일시스템 | 항상 켜진 VPS, 원격 실행 | SSH 접근 가능한 서버 |
| `singularity` | 네임스페이스 격리 | 오버레이 파일시스템 | HPC 클러스터 | `apptainer` 또는 `singularity` in `$PATH` |
| `modal` | 클라우드 샌드박스 | 에페머럴 (세션 간 지속 없음) | 서버리스, 비용 최적화 | `MODAL_TOKEN_ID` 또는 `~/.modal.toml` |
| `daytona` | 관리형 워크스페이스 | 영구 (최대 10GB 디스크) | 클라우드 개발 환경 | `DAYTONA_API_KEY` |

**Modal 특이 사항:** 에페머럴 — 세션 간 파일이 자동으로 유지되지 않음. 서버리스로 유휴 비용 없음 (cold start 있음).

**Daytona 특이 사항:** 유휴 시 히버네이션 → 세션 간 유지 가능, 비용 거의 없음.

#### How to Select / Switch Backend (HIGH confidence)

**방법 1: config.yaml (영구적)**

```yaml
# 검증: hermes rolling, 2026-06-09
terminal:
  backend: docker    # local | docker | ssh | singularity | modal | daytona
```

**방법 2: hermes config set (영구적, CLI에서 즉시)**

```bash
# 검증: hermes rolling, 2026-06-09
hermes config set terminal.backend docker
hermes config set terminal.backend local   # 되돌리기
```

**방법 3: hermes setup terminal (인터랙티브 마법사)**

```bash
# 검증: hermes rolling, 2026-06-09
hermes setup terminal
```

Note: `hermes setup terminal` is documented as configuring "terminal backend and sandbox setup" — exact wizard flow varies by backend. Flag as MEDIUM confidence for exact interaction.

**Docker 상태 확인:**

```bash
# 검증: 로컬 Docker 설치, 2026-06-09
docker info    # Docker 데몬 실행 중인지 확인
```

#### Docker Backend — Hands-on (HIGH confidence config, MEDIUM output)

**Docker backend의 보안 hardening (Hermes가 자동으로 적용):**

```
--cap-drop ALL
--cap-add DAC_OVERRIDE
--cap-add CHOWN
--security-opt no-new-privileges
--pids-limit 256
--tmpfs /tmp:rw,nosuid,size=512m
--tmpfs /var/tmp:rw,noexec,nosuid
```

독자가 직접 설정할 필요 없음 — Hermes가 자동으로 적용.

**Docker backend config.yaml 전체 설정:**

```yaml
# 검증: hermes rolling, 2026-06-09
terminal:
  backend: docker
  docker_image: "nikolaik/python-nodejs:python3.11-nodejs20"  # 기본값
  docker_mount_cwd_to_workspace: false
  docker_run_as_host_user: false    # true: 컨테이너가 호스트 UID로 실행 (파일 소유권 보존)
  docker_forward_env:               # 컨테이너에 전달할 호스트 환경변수
    - "GITHUB_TOKEN"
  docker_env:                       # 컨테이너 전용 환경변수
    DEBUG: "1"
  docker_volumes:                   # 추가 볼륨 마운트
    - "/host/path:/container/path"
  docker_extra_args:                # 추가 docker run 플래그
    - "--gpus=all"
  container_cpu: 1                  # CPU 코어
  container_memory: 5120            # MB (기본 5GB)
  container_disk: 51200             # MB (기본 50GB)
  container_persistent: true        # 세션 간 컨테이너 유지
  docker_persist_across_processes: true
  docker_orphan_reaper: true
```

**최소 Docker 핸즈온:**

```bash
# 검증: hermes rolling + Docker 설치, 2026-06-09
# 1. Docker 백엔드로 전환
hermes config set terminal.backend docker

# 2. hermes 실행 — 첫 툴 실행 시 컨테이너 자동 시작
hermes

# 3. 에이전트에게 파이썬 버전 확인 요청 (컨테이너 내 실행됨)
# 채팅에서: "파이썬 버전을 확인해줘"
# 예상 동작: 에이전트가 terminal 툴로 docker exec 실행
#            출력: Python 3.11.x [GCC ... ] on linux

# 4. 로컬로 되돌리기
hermes config set terminal.backend local
```

**예상 출력 (LOW — 컨테이너 초기화 메시지):**

```
# [첫 Docker 백엔드 실행 시 컨테이너 빌드/풀 메시지 — 로컬 실행 후 캡처 필요]
```

#### SSH Backend (MEDIUM confidence — env var names confirmed, exact interaction unverified)

```bash
# 검증: hermes rolling, 2026-06-09
# SSH 백엔드 설정 (환경변수 방식)
# ~/.hermes/.env에 추가:
TERMINAL_SSH_HOST=your-server.example.com
TERMINAL_SSH_USER=ubuntu
# 선택사항:
# TERMINAL_SSH_PORT=22
# TERMINAL_SSH_KEY=~/.ssh/id_ed25519
```

```yaml
# 검증: hermes rolling, 2026-06-09
terminal:
  backend: ssh
  persistent_shell: true   # SSH ControlMaster 멀티플렉싱 (기본 true)
```

**파일 동기화:** SSH 백엔드는 SSH ControlMaster를 통한 `tar` 파이프로 `~/.hermes` 디렉터리를 원격 호스트와 동기화. 세션 종료 시 수정된 파일을 `~/.hermes/cache/remote-syncs/`로 동기화.

#### Modal Backend (MEDIUM confidence — managed mode is the easy path)

```yaml
# 검증: hermes rolling, 2026-06-09
terminal:
  backend: modal
  container_cpu: 1
  container_memory: 5120    # MB
  container_disk: 51200     # MB
  container_persistent: true
```

**필요 자격증명:** `MODAL_TOKEN_ID` 환경변수 또는 `~/.modal.toml` (modal auth login으로 생성).

**"Managed mode":** NousResearch 게이트웨이를 통해 Modal 리소스를 사전 공급하는 모드 (사용자 자격증명 불필요). `hermes setup terminal` 마법사에서 선택 가능. MEDIUM confidence — exact setup flow for managed mode not fully documented in public sources.

#### Daytona Backend (MEDIUM confidence)

```yaml
# 검증: hermes rolling, 2026-06-09
terminal:
  backend: daytona
  container_cpu: 1
  container_memory: 5120    # MB
  container_disk: 10240     # MB (최대 10GB)
  container_persistent: true
```

**필요 자격증명:** `DAYTONA_API_KEY` 환경변수.

**Daytona SDK가 서버 URL 자동 처리** — 별도 서버 URL 설정 불필요.

#### Singularity/Apptainer Backend (MEDIUM confidence)

```yaml
# 검증: hermes rolling, 2026-06-09
terminal:
  backend: singularity
```

**필요 환경:** `apptainer` 또는 `singularity` 명령이 `$PATH`에 있어야 함. HPC 클러스터 환경 전용.

**스크래치 디렉터리 해결 순서:**
1. `TERMINAL_SCRATCH_DIR` 환경변수
2. `TERMINAL_SANDBOX_DIR/singularity`
3. `/scratch/$USER/hermes-agent` (HPC 관례)
4. `~/.hermes/sandboxes/singularity`

**오버레이 파일시스템:** 읽기 전용 base 이미지 위에 쓰기 가능한 레이어 제공.

### Pitfall Callouts for Ch.8

**주의 1 — Docker 데몬 미실행 (A-13 관련):**

```
> 주의: Docker 백엔드를 사용하려면 Docker 데몬이 실행 중이어야 합니다.
> `docker info` 명령으로 데몬 상태를 먼저 확인하십시오.
> 데몬 미실행 시 터미널 툴 호출마다 "Cannot connect to Docker daemon" 오류 발생.
```

**주의 2 — docker_run_as_host_user와 파일 소유권 (A-13, CRITICAL):**

```
> 주의: docker_run_as_host_user: false (기본값)로 Docker 백엔드를 사용하면
> 컨테이너 내에서 생성된 파일이 root 소유가 될 수 있습니다.
> 호스트에서 일반 사용자로 이 파일을 편집/삭제하려면 sudo가 필요합니다.
>
> 해결책:
>   docker_run_as_host_user: true  ← 컨테이너가 호스트 UID로 실행
>
> 단, 이 설정을 활성화하면 컨테이너 내 패키지 설치(apt install)가
> 권한 오류로 실패할 수 있습니다.
```

**주의 3 — Modal 에페머럴 특성 (LOW — 문서에서 일부 불확실):**

```
> 참고: Modal 백엔드는 기본적으로 에페머럴입니다.
> 세션이 종료되면 컨테이너 내 파일이 사라질 수 있습니다 (스냅샷 미설정 시).
> 세션 간 파일을 유지하려면 container_persistent: true를 설정하십시오.
> [Modal 스냅샷 설정의 정확한 동작은 로컬 검증 필요]
```

**주의 4 — approvals.mode: off는 컨테이너 백엔드에서만 합리적 (A-12):**

```
> 참고: Docker/Modal/Daytona 백엔드를 사용할 경우
> 컨테이너 자체가 보안 경계 역할을 합니다.
> 이 환경에서는 approvals.mode: off가 합리적일 수 있습니다.
> 그러나 local 백엔드에서는 절대 off로 설정하지 마십시오.
```

---

## Architecture Patterns (for planner)

### Chapter file template (same as Phase 2)

```markdown
# Ch.N 챕터 제목

> **이 챕터를 시작하기 전에**
> - 선행 조건 체크리스트

## 개요

[이 챕터에서 할 일 — 1-2단락]

## 개념: [주요 개념]

[다이어그램 또는 설명 — ASCII 다이어그램 권장]

## 실습: 직접 해보기

[명령어 + 예상 출력]

## 심화: 내부 동작 원리

[고급 독자를 위한 내용]

## 흔한 오류 / 주의

[pitfall callouts]

## 다음 단계

다음 챕터: [링크]
```

### Code block annotation convention (MANDATORY — from Phase 2)

Every code block opens with:

```
# 검증: hermes rolling, 2026-06-09
```

Where output is unverified locally:

```
# [로컬 실행 후 캡처 필요 — 출력은 환경에 따라 다름]
```

---

## Open Questions (LOW confidence — verify locally before publishing)

1. **Context file load confirmation message in banner**
   - What we know: `.hermes.md` is loaded and injected at session start
   - What's unclear: Whether the welcome banner explicitly says "Project context loaded" or similar
   - Recommendation: Run `hermes` in a dir with `.hermes.md`, observe banner text, capture

2. **Free tool pool exact quota on Nous Portal**
   - What we know: Free accounts get "a free tool pool" auto-surfaced on first use
   - What's unclear: Exact quota (number of searches, images, etc.)
   - Recommendation: Check https://nousresearch.com/hermes or Nous Portal pricing page before publishing Ch.7 paid-vs-free callout

3. **hermes setup terminal exact interactive flow**
   - What we know: Command exists and configures "terminal backend and sandbox setup"
   - What's unclear: Which backends it prompts for, what questions it asks, expected output
   - Recommendation: Run locally, capture wizard output for Ch.8

4. **Docker first-run output (container build/pull message)**
   - What we know: Hermes starts one long-lived container on first use
   - What's unclear: Exact progress messages (image pull output, container start confirmation)
   - Recommendation: Run `hermes config set terminal.backend docker && hermes`, capture first terminal-tool output

5. **Modal managed mode setup flow**
   - What we know: NousResearch can provision Modal resources (no user credential needed)
   - What's unclear: How this is triggered (via `hermes setup terminal`? separate command?), current availability
   - Recommendation: Flag Modal section as MEDIUM confidence; test locally if Modal account available

6. **exact `memory_char_limit` vs `user_char_limit` config key names**
   - What we know: From official memory docs, these keys exist at exact names listed
   - What's unclear: Whether keys changed in rolling release after docs update
   - Recommendation: Run `hermes config show | grep memory` to confirm

7. **v0.15.0 "4,500x faster FTS5" exact benchmark source**
   - What we know: Prior research stated this; current official docs say "~20ms FTS5 query" which is consistent
   - What's unclear: The 4,500x figure's specific baseline — may be internal benchmark not publicly documented
   - Recommendation: Do not cite the 4,500x figure; use "~20ms FTS5 query" from official docs instead

8. **Singularity/Daytona YAML keys accuracy**
   - What we know: Keys are from official configuration docs (HIGH) and DeepWiki backend analysis (MEDIUM)
   - What's unclear: Whether `modal_image` key is separate from `docker_image` for Modal backend
   - Recommendation: Flag Modal/Daytona YAML sections as MEDIUM confidence; verify before publishing

---

## Sources

### Primary (HIGH confidence — official docs verified)

- `https://hermes-agent.nousresearch.com/docs/developer-guide/agent-loop` — Exact loop steps, 3 API modes, tool dispatch, callbacks, termination, fallback
- `https://hermes-agent.nousresearch.com/docs/developer-guide/prompt-assembly` — 3-tier architecture, exact 10-layer order, cache stability, context file priority
- `https://hermes-agent.nousresearch.com/docs/user-guide/features/context-files` — File hierarchy, SOUL.md distinction, subdirectory loading, security scan
- `https://hermes-agent.nousresearch.com/docs/user-guide/features/personality` — 14 built-in personalities, SOUL.md path/contents, /personality syntax, custom config
- `https://hermes-agent.nousresearch.com/docs/user-guide/features/memory` — MEMORY.md/USER.md paths and limits, memory tool actions, § delimiter, session_search
- `https://hermes-agent.nousresearch.com/docs/user-guide/features/memory-providers` — 9 external providers, hermes memory setup/status/off commands
- `https://hermes-agent.nousresearch.com/docs/reference/tools-reference` — Complete ~71 tool list by toolset
- `https://hermes-agent.nousresearch.com/docs/reference/toolsets-reference` — Toolset categories, enable/disable, kanban opt-in, custom toolsets
- `https://hermes-agent.nousresearch.com/docs/user-guide/features/tool-gateway` — Gateway capabilities, image models, browser tools, per-tool use_gateway
- `https://hermes-agent.nousresearch.com/docs/user-guide/security` — Approval mode (manual/smart/off), Tirith, hardline blocklist, container isolation
- `https://hermes-agent.nousresearch.com/docs/user-guide/configuration` — Config file structure, compression settings, memory limits, terminal backend keys, display
- `https://hermes-agent.nousresearch.com/docs/developer-guide/context-compression-and-caching` — 4-phase compression, protect settings, Anthropic caching, cost tips
- `https://hermes-agent.nousresearch.com/docs/developer-guide/session-storage` — SQLite schema, FTS5 tables, WAL mode, session lineage, CJK trigram
- `https://hermes-agent.nousresearch.com/docs/reference/cli-commands` — hermes tools, hermes memory, hermes sessions, hermes portal subcommands
- `https://hermes-agent.nousresearch.com/docs/integrations/nous-portal` — Portal setup flow, portal info/tools commands

### Secondary (MEDIUM confidence — verified with official source cross-check)

- `https://deepwiki.com/NousResearch/hermes-agent/6.2-backend-implementations` — Docker security flags, SSH ControlMaster, Modal managed mode, Daytona SDK, Singularity scratch dir — corroborates official config docs
- `https://lumadock.com/tutorials/hermes-memory-architecture-explained` — Memory path confirmation (`~/.hermes/memories/`)
- WebSearch: `hermes agent modal daytona singularity backend 2026` — confirmed backend existence and env var names

### Tertiary (LOW confidence — flagged accordingly)

- `https://www.ququ123.top/en/2026/04/hermes-terminal-backends-skill/` — Third-party tutorial; uses non-standard `backends:` YAML key structure (not official `terminal:` structure). DO NOT USE for config examples.

### From Prior Research (HIGH confidence, imported and extended)

- `ARCHITECTURE.md`: AIAgent core, entry points, component roles — all confirmed
- `FEATURES.md`: A3 memory, A5 context files, A6 tool gateway, A9 approval mode — confirmed and extended
- `PITFALLS.md`: A-4, A-7, A-9, A-12, A-13, A-14, A-19 — all mapped to relevant chapters

---

## Metadata

**Confidence breakdown:**

| 챕터/토픽 | 신뢰도 | 근거 |
|-----------|--------|------|
| Ch.4 에이전트 루프 (3 API modes, loop steps, dispatch) | HIGH | Official agent-loop dev guide |
| Ch.4 callback details | MEDIUM | Named in docs; exact signatures not shown |
| Ch.5 3-tier prompt system (names and layer order) | HIGH | Official prompt-assembly dev guide |
| Ch.5 context file hierarchy | HIGH | Official context-files docs + confirmed |
| Ch.5 /personality 14 built-ins | HIGH | Official personality docs |
| Ch.5 config.yaml variable substitution ${VAR} | HIGH | Configuration docs |
| Ch.6 MEMORY.md/USER.md paths | HIGH | Memory docs + lumadock cross-check |
| Ch.6 session_search is agent tool, not slash cmd | HIGH | Memory docs |
| Ch.6 FTS5 trigram table for CJK | HIGH | Session-storage dev guide schema |
| Ch.6 compression config keys | HIGH | Configuration docs |
| Ch.7 ~71 tools, 30+ toolsets | HIGH | Tools-reference official page |
| Ch.7 hermes tools / hermes tools --summary | HIGH | CLI commands reference |
| Ch.7 Tool Gateway free pool boundary | MEDIUM | Portal integration docs (quota not specified) |
| Ch.7 9 image generation models | HIGH | Tool-gateway official docs |
| Ch.7 approval mode config (manual/smart/off) | HIGH | Security docs |
| Ch.7 Tirith scanning details | HIGH | Security docs |
| Ch.8 6 backends existence | HIGH | Configuration docs + DeepWiki |
| Ch.8 Docker config keys | HIGH | Configuration docs |
| Ch.8 Docker security flags (cap-drop etc.) | HIGH | DeepWiki backend analysis (corroborates config) |
| Ch.8 SSH env vars (TERMINAL_SSH_HOST/USER) | HIGH | Configuration docs |
| Ch.8 hermes setup terminal | MEDIUM | CLI reference (exact flow unverified) |
| Ch.8 Modal managed mode | MEDIUM | DeepWiki only; not in primary docs |
| Ch.8 Daytona/Singularity YAML | MEDIUM | Config docs + DeepWiki cross-check |

**Research date:** 2026-06-09
**Valid until:** 2026-07-09 (Hermes is rolling release — re-verify commands 30 days before publish)
**Key risk:** Hermes rolling release. Especially at risk: toolset names, personality list count, compression key names. Re-run `hermes config show` and `hermes tools --summary` locally before marking any chapter publication-ready.
