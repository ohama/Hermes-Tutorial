# Phase 4: Learning & Automation — Research

**Researched:** 2026-06-10
**Domain:** Hermes Agent — Ch.9 스킬 시스템 / Ch.10 학습 루프 / Ch.11 MCP 연동 / Ch.12 크론 스케줄러 / Ch.13 서브에이전트
**Confidence:** HIGH (skills, cron, MCP client, subagents — all from official docs) / MEDIUM (hermes mcp serve exact tool list, Honcho config file paths) / LOW (some skill auto-gen observation specifics)

**Accuracy constraint:** Every command, flag, config key, and expected output grounded in official docs retrieved 2026-06-10.
Convention (MANDATORY): every command/config code block opens with `# 검증: hermes rolling, 2026-06-10`.
LOW/MEDIUM specifics → `[로컬 실행 후 캡처 필요]` placeholder, `> 검증 필요` callout — never fabricate.

---

## Summary

Phase 4 produces five chapters (Ch.9–13) covering the flagship self-improvement loop and its automation ecosystem. All five major open flags from STATE.md have been resolved with varying confidence:

1. **Skills system (Ch.9):** Fully documented — SKILL.md format, exact CLI, community hub, auto-gen trigger. HIGH confidence.
2. **Learning loop (Ch.10):** Do→Learn→Improve mechanics confirmed; observation flow verified via Curator docs. The "periodic nudge every 15 tasks" from prior research has been corrected: the curator runs on an **inactivity trigger** (~every 7 days by default), NOT every 15 tasks. Skill creation on ≥5 tool calls is confirmed.
3. **MCP (Ch.11):** Bidirectional flow fully confirmed with exact commands. **ACP vs MCP is now RESOLVED: they are separate integrations.** `hermes mcp serve` exposes Hermes as a stdio MCP server to Claude Desktop/Cursor/etc. ACP (`hermes acp`) separately exposes Hermes as an editor-native agent for VS Code/Zed/JetBrains.
4. **Cron (Ch.12):** Exact CLI fully confirmed — `hermes cron` is the command (NOT `hermes gateway`). Workdir pitfall documented precisely.
5. **Subagents/Kanban (Ch.13):** `delegate_task` fully confirmed. Kanban is a **separate, distinct system** from `delegate_task` — it is a durable SQLite-backed task queue (NOT just an opt-in toolset as prior research suggested). Both are documented.

**Key corrections to established facts (prior research):**
- Bundled skills count: **~89** (v0.15.x), NOT 118 (that was v0.10.0). Optional skills catalog adds ~90 more.
- Skills community hub count 19,932+: **NOT verified from current docs** — sources list multiple hubs (official, skills.sh, ClawHub, LobeHub) but no single aggregate count found. Remove this number from Ch.9; use "다양한 커뮤니티 스킬 허브" instead.
- Curator runs on inactivity trigger (≥2h idle + ≥7-day interval), NOT a fixed "every 15 tasks" periodic nudge.
- `hermes mcp serve` exposes **10 platform messaging tools** (not a full Hermes session relay as previously described). This is important for Ch.11 accuracy.
- ACP and MCP remain **separate** integrations — ACP is for editor embedding; MCP is for tool server connections.

**Primary recommendation:** Owner plan writes SUMMARY.md + Ch.9 skeleton. Parallel plans write Ch.10–13 body. Use Phase 3 owner-plan pattern exactly.

---

## Chapter Structure Guidance

### SUMMARY.md additions (after Phase 3 `# 핵심 개념` section)

Current SUMMARY.md ends at:
```markdown
# 핵심 개념

- [에이전트 루프](04-agent-loop/index.md)
- [컨텍스트 파일](05-context-files/index.md)
- [메모리](06-memory/index.md)
- [툴 게이트웨이](07-tools/index.md)
- [터미널 백엔드](08-backends/index.md)
```

Phase 4 adds a new `# 학습과 자동화` section:

```markdown
---

# 학습과 자동화

- [스킬 시스템](09-skills/index.md)
- [학습 루프](10-learning-loop/index.md)
- [MCP 연동](11-mcp/index.md)
- [크론 스케줄러](12-cron/index.md)
- [서브에이전트](13-subagents/index.md)
```

### New src/ directories required

```
src/
├── 09-skills/
│   └── index.md
├── 10-learning-loop/
│   └── index.md
├── 11-mcp/
│   └── index.md
├── 12-cron/
│   └── index.md
└── 13-subagents/
    └── index.md
```

**mdBook duplicate-path error prevention (Pitfall B-11):** All five paths are brand new — no existing entries in SUMMARY.md use these paths. Safe to add.

### Plan / wave structure (reuse Phase 3 pattern)

- **Plan A (owner):** Updates SUMMARY.md + appends `# 학습과 자동화` section + 5 new entries. Creates all five `src/NN/index.md` skeleton files (one-line placeholder). Runs `mdbook build` to verify no duplicate-path error. Also writes the body of Ch.9.
- **Plans B–E (body, run in parallel after Plan A):** Each writes the body of one chapter (Ch.10 / Ch.11 / Ch.12 / Ch.13). Since Plan A already owns all skeleton files, Plans B–E overwrite skeletons — no SUMMARY.md write race.

---

## Ch.9: 스킬 시스템 (Skills System)

**Prerequisites:** Ch.6 (메모리), Ch.7 (툴 게이트웨이) — skills are procedural memory; agent must understand the tool loop.

### Facts to State (HIGH confidence — from official skills docs retrieved 2026-06-10)

#### What Skills Are

Skills are Markdown files with YAML frontmatter describing reusable procedures. They represent **procedural memory** — capturing workflows, commands, decision branches, pitfalls, and verification steps for reuse across sessions.

Hermes follows the **agentskills.io open standard** — skills are portable across 25+ compatible agent platforms.

#### Storage Location (HIGH confidence)

```
~/.hermes/skills/
├── <category>/
│   └── <skill-name>/
│       ├── SKILL.md              ← 핵심 파일
│       ├── references/           ← API 문서, 예시 (선택)
│       ├── templates/            ← 설정 템플릿 (선택)
│       ├── scripts/              ← 설정 스크립트 (선택)
│       └── assets/               ← 이미지 등 (선택)
```

Additional external directories can be configured:
```yaml
# 검증: hermes rolling, 2026-06-10
skills:
  external_dirs:
    - "/path/to/additional/skills"
```

#### SKILL.md Format (HIGH confidence — from official skills docs)

```yaml
# 검증: hermes rolling, 2026-06-10
---
name: my-skill
description: Brief description of what this skill does
version: 1.0.0
platforms: [macos, linux]        # 선택: OS 제한
metadata:
  hermes:
    tags: [automation, web]
    category: productivity
    config:
      output_format: markdown    # 비밀이 아닌 설정 선언
required_environment_variables:   # 자격증명 프롬프트
  - MY_API_KEY
fallback_for_toolsets: []        # 조건부 활성화
requires_toolsets: [web]         # 필요 툴셋 제약
---

## When to Use

[이 스킬을 언제 사용하는지 설명]

## Procedure

1. [단계 1 — 명령어 또는 액션]
2. [단계 2]
3. [분기: 조건 A면 → 단계 3a, 조건 B면 → 단계 3b]

## Pitfalls

- [알려진 문제 1]
- [알려진 문제 2]

## Verification

[결과가 올바른지 확인하는 방법]
```

**Mandatory body sections:** When to Use, Procedure, Pitfalls, Verification.

#### Progressive Disclosure Loading (HIGH confidence)

Only skill names and descriptions (the "skill index") are loaded into the system prompt. Full skill content loads **on-demand** when a trigger matches. 200 skills consume almost nothing until activated.

Skills appear in the system prompt's **Stable tier, Layer 7** (skill index) — confirmed from Phase 3 prompt-assembly research.

#### Auto-Generation Trigger (HIGH confidence)

The agent creates skills automatically after:
- Completing a **complex task with 5+ tool calls** successfully
- Discovering a **non-trivial workflow** worth preserving

The docs state: "After a complex multi-step task, Hermes will often **offer to save** the approach as a skill." This means the agent **proposes** saving; the user accepts or declines. This is important for Ch.10's hands-on — the reader needs to accept the proposal.

#### Bundled Skills (CORRECTED — HIGH confidence)

- **~89 bundled skills** across 17 categories (as of v0.15.x docs — NOT 118 as previously stated; 118 was the v0.10.0 count)
- **~90+ optional skills** in a separate catalog across 18 categories
- Both live in `~/.hermes/skills/` after installation

Notable bundled skill categories: GitHub, Research (arXiv), DevOps (Kanban), Software Development (debugging, TDD), Productivity (Google Workspace, Notion), Note-Taking (Obsidian).

#### CLI Commands for Skills Management (HIGH confidence)

```bash
# 검증: hermes rolling, 2026-06-10
hermes skills list                    # 설치된 스킬 목록
hermes skills browse                  # 허브 스킬 탐색 (인터랙티브)
hermes skills search <query>          # 이름/태그로 검색
hermes skills inspect <identifier>    # 설치 전 미리보기
hermes skills install <identifier>    # 보안 스캔 후 설치
hermes skills check                   # 업스트림 업데이트 확인
hermes skills update                  # 업데이트된 스킬 재설치
```

**In-session slash command:**

```
/skills install official/creative/songwriting-and-ai-music
```

#### Community Skill Installation (HIGH confidence)

```bash
# 검증: hermes rolling, 2026-06-10
# 방법 1: 공식 선택 스킬 설치
hermes skills install official/research/arxiv

# 방법 2: URL로 직접 설치
hermes skills install https://sharethis.chat/SKILL.md

# 방법 3: URL에 이름 지정
hermes skills install https://example.com/SKILL.md --name my-skill

# 제거
hermes skills uninstall <skill-name>
```

**Community skill hubs:** official optional catalog, skills.sh directory, ClawHub, LobeHub, GitHub repositories, browse.sh. No single aggregate count is reliably documented — do NOT state "19,932+" in the tutorial.

#### `skill_manage` Tool — Agent-Directed Skill Updates (HIGH confidence)

The agent uses this internally:

| 액션 | 용도 |
|------|------|
| `create` | 새 스킬 파일 생성 |
| `patch` | 타겟 변경 (토큰 절약, 선호 방식) |
| `edit` | 전체 파일 재작성 |
| `delete` | 스킬 삭제 |
| `write_file` | 지원 파일 작성 |
| `remove_file` | 지원 파일 삭제 |

Readers do NOT need to call `skill_manage` directly — they can ask the agent to improve a skill in natural language.

#### Skill Activation (HIGH confidence)

Skills activate as **slash commands** once placed in `~/.hermes/skills/`:

```
/skill-name [task description]
```

Plugin-namespaced: `/plugin:skill-name`

### Exact Commands for Ch.9 Hands-On

**1. 스킬 목록 확인:**

```bash
# 검증: hermes rolling, 2026-06-10
hermes skills list
```

Expected output shape (LOW — verify locally):
```
# [로컬 실행 후 캡처 필요 — 설치된 스킬 목록 출력]
```

**2. 수동으로 스킬 만들기:**

```bash
# 검증: hermes rolling, 2026-06-10
mkdir -p ~/.hermes/skills/productivity/my-deploy-skill
cat > ~/.hermes/skills/productivity/my-deploy-skill/SKILL.md << 'EOF'
---
name: my-deploy-skill
description: 로컬 Python 프로젝트를 서버에 배포하는 절차
version: 1.0.0
metadata:
  hermes:
    tags: [deploy, python]
    category: productivity
---

## When to Use

Python 프로젝트를 원격 서버에 배포할 때 사용합니다.

## Procedure

1. `pytest`로 테스트를 먼저 실행합니다.
2. `git push origin main`으로 코드를 푸시합니다.
3. SSH로 서버에 접속하여 `git pull`을 실행합니다.
4. `systemctl restart myapp`으로 서비스를 재시작합니다.

## Pitfalls

- 테스트가 실패하면 배포를 중단합니다.
- SSH 키가 서버에 등록되어 있어야 합니다.

## Verification

서비스 상태: `systemctl status myapp`으로 active(running) 확인.
EOF

# hermes 재시작 후 /my-deploy-skill 로 활성화 가능
```

**3. 커뮤니티 스킬 설치:**

```bash
# 검증: hermes rolling, 2026-06-10
hermes skills inspect official/research/arxiv
hermes skills install official/research/arxiv
```

### Pitfall Callouts for Ch.9

**주의 1 — 스킬 변경은 재시작 후 반영 (MEDIUM):**
```
> 참고: ~/.hermes/skills/에 스킬 파일을 추가하거나 수정하면
> 현재 실행 중인 hermes 세션에는 즉시 반영되지 않을 수 있습니다.
> hermes를 재시작하면 스킬 인덱스가 갱신됩니다.
> [로컬에서 정확한 동작 확인 필요]
```

**주의 2 — bundled 스킬 버전 (HIGH — correction from prior research):**
```
> 참고: 번들 스킬 수는 버전마다 달라집니다.
> v0.10.0에는 118개, v0.15.x 현재에는 약 89개의 번들 스킬이 있습니다.
> hermes skills list로 현재 설치된 스킬 수를 항상 확인하십시오.
```

**주의 3 — 보안 스캔 (HIGH):**
```
> 참고: hermes skills install은 설치 전 자동으로 보안 스캔을 수행합니다.
> 스캔 결과에 경고가 있어도 --force 플래그로 강제 설치할 수 있지만,
> 위험하지 않은 항목에 대해서만 사용하십시오.
```

---

## Ch.10: 학습 루프 (Learning Loop — Do→Learn→Improve)

**Prerequisites:** Ch.9 (스킬 시스템) — 학습 루프는 스킬 생성/개선이 핵심.

### Facts to State

#### The Learning Loop Concept (HIGH confidence)

The Do→Learn→Improve cycle is the flagship feature of Hermes:

```
Do:      복잡한 작업 수행 (도구 5번 이상 호출)
          │
          ▼
Learn:   에이전트가 워크플로우를 분석하고 스킬 파일 생성 제안
          │
          ▼ (사용자 수락)
Improve: 유사한 미래 작업에 스킬이 자동 주입됨
          │
          ▼
Curator: 백그라운드에서 주기적으로 스킬 품질 검토 및 개선 (자동)
```

#### Step 1: Skill Auto-Generation (HIGH confidence)

**Trigger:** ≥5 tool calls in a single task turn.

**What happens:** Agent proposes saving the approach as a skill. Reader accepts with a confirmation in the chat. The skill is written to `~/.hermes/skills/<category>/<name>/SKILL.md`.

**Observation method for tutorial hands-on:**
1. Ask Hermes a multi-step task: "이 프로젝트의 모든 Python 파일에서 미사용 import를 찾아 리포트를 만들어줘"
2. Agent executes multiple tools (read_file, terminal, write_file, etc.)
3. After completion, agent offers: "이 워크플로우를 스킬로 저장할까요?" (exact wording not verified — see below)
4. Accept → skill appears in `~/.hermes/skills/`

**Expected output (LOW — verify locally):**
```
# [로컬 실행 후 캡처 필요 — 스킬 저장 제안 메시지 및 스킬 파일 생성 확인]
```

> 검증 필요: 스킬 저장 제안의 정확한 메시지 문구 및 수락 방법을 로컬 실행 후 캡처하십시오.

**Verification after generation:**
```bash
# 검증: hermes rolling, 2026-06-10
hermes skills list
ls ~/.hermes/skills/
```

#### Step 2: Skill Self-Improvement in Use (HIGH confidence)

When a skill is triggered on a subsequent task, the agent:
1. Loads the full skill content into context
2. Executes the procedure
3. If it discovers new edge cases or better approaches, it can `patch` the skill automatically

The `patch_count` metadata tracks how many times a skill has been improved.

Readers can observe this by checking:
```bash
# 검증: hermes rolling, 2026-06-10
cat ~/.hermes/skills/.usage.json     # use_count, patch_count 확인
```

> 검증 필요: `.usage.json`의 정확한 위치와 형식을 로컬 실행 후 확인하십시오.

#### Step 3: Curator — Background Quality Review (HIGH confidence — CORRECTS prior research)

**CORRECTION:** The prior "every 15 tasks periodic nudge" description was inaccurate. The actual mechanism:

- Triggered by **inactivity** (not a task count): `min_idle_hours` (default 2h) must have passed AND interval `interval_hours` (default 7 days = 168h) must have elapsed
- First run deferred by one full interval on new installs (reader will not see it in a first session)
- Runs a **background AIAgent fork** — isolated, no conversation impact

```yaml
# 검증: hermes rolling, 2026-06-10
curator:
  enabled: true
  interval_hours: 168        # 7일 (기본값)
  stale_after_days: 30       # 30일 미사용 → stale
  archive_after_days: 90     # 90일 미사용 → archived
```

**Curator CLI:**
```bash
# 검증: hermes rolling, 2026-06-10
hermes curator status           # 마지막 실행, 스킬 카운트, 최근 미사용 스킬 표시
hermes curator run              # 즉시 실행
hermes curator run --background # 백그라운드 실행
hermes curator run --dry-run    # 변경 없이 미리보기
hermes curator rollback         # 마지막 실행 전 상태로 복원
```

**Run reports saved at:** `~/.hermes/logs/curator/<timestamp>/REPORT.md`

**Backup before each run:** `~/.hermes/skills/.curator_backups/`

**Archive location:** `~/.hermes/skills/.archive/` (복구 가능)

#### Honcho — Advanced Aside (RESOLVED FLAG — HIGH confidence for key facts)

**Resolution:** Honcho **REQUIRES an external account and API key** from honcho.dev. It is NOT a free, zero-friction add-on. For the tutorial, treat as an "심화" (advanced aside) — not mandatory hands-on.

**Key facts for the chapter (do not expand into full hands-on):**

| 항목 | 내용 |
|------|------|
| 외부 계정 필요? | **예 — honcho.dev에서 계정 필요** |
| API 키 저장 위치 | `~/.hermes/.env`에 `HONCHO_API_KEY=xxx` |
| config.yaml 설정 | `memory.provider: honcho` (단 1줄) |
| 추가 설정 파일 위치 | `~/.honcho/config.json` (global) 또는 `$HERMES_HOME/honcho.json` |
| 설치 커맨드 | `hermes memory setup honcho` |
| 자기 호스팅 | 가능 (AUTH_USE_AUTH=false로 인증 건너뛰기 가능) |

**Honcho-specific config keys (MEDIUM confidence — from official Honcho integration docs):**

| 키 | 기본값 | 설명 |
|----|--------|------|
| `contextCadence` | `1` | base layer 갱신 최소 주기 (턴 수) |
| `dialecticCadence` | `2` | dialectic LLM 호출 최소 주기 (턴 수) |
| `dialecticDepth` | `1` | 호출당 reasoning 패스 수 (1–3 범위) |

**Tools added when Honcho is active:** `honcho_profile`, `honcho_search`, `honcho_context`, `honcho_reasoning`, `honcho_conclude`

**Recommended chapter treatment:** 1-paragraph mention in Ch.10's "심화" section with code block showing minimum setup. Link to official docs. Do NOT make it required hands-on.

```yaml
# 검증: honcho.dev 계정 필요, hermes rolling, 2026-06-10
# 최소 Honcho 설정 (심화 — 계정 필요)
memory:
  provider: honcho
# ~/.hermes/.env에 추가:
# HONCHO_API_KEY=your-key-here
```

### Pitfall Callouts for Ch.10

**주의 1 — 스킬 저장 제안을 놓치지 말 것 (HIGH):**
```
> 참고: 에이전트가 스킬 저장을 제안할 때 응답하지 않으면 해당 워크플로우는 저장되지 않습니다.
> 작업 완료 후 에이전트의 메시지를 주의 깊게 확인하십시오.
```

**주의 2 — Curator는 즉시 실행되지 않음 (HIGH — corrects prior research):**
```
> 참고: Curator는 "15번째 작업마다"가 아닌 비활동 기반 트리거로 실행됩니다.
> 기본 설정에서는 마지막 활동 후 최소 2시간 경과 AND 7일 간격이 지나야 실행됩니다.
> hermes curator run 으로 즉시 수동 실행할 수 있습니다.
```

**주의 3 — Honcho는 외부 계정이 필요함 (HIGH — resolves open flag):**
```
> 주의: Honcho 연동은 honcho.dev에서 별도 계정과 API 키가 필요합니다.
> 무료 계정으로 시작할 수 있지만, 클라우드 서비스 의존성이 추가됩니다.
> 자기 호스팅(self-hosted) 옵션도 지원됩니다.
```

---

## Ch.11: MCP 연동 (MCP Integration — Bidirectional)

**Prerequisites:** Ch.7 (툴 게이트웨이), Ch.9 (스킬 시스템).

### Facts to State

#### ACP vs MCP — RESOLVED FLAG (HIGH confidence)

**These are TWO separate integrations.** Both exist in current Hermes.

| 항목 | ACP (Agent Client Protocol) | MCP (Model Context Protocol) |
|------|------------------------------|-------------------------------|
| **목적** | 에디터 임베딩 — Hermes를 에디터 네이티브 코딩 에이전트로 실행 | 외부 툴 서버 연결 + Hermes를 MCP 서버로 노출 |
| **방향** | 에디터 → Hermes | 양방향 |
| **지원 에디터** | VS Code, Zed, JetBrains | Claude Desktop, Cursor, VS Code (MCP 클라이언트), Zed |
| **명령어** | `hermes acp` / `hermes-acp` | `hermes mcp serve` (서버) / config.yaml `mcp_servers` (클라이언트) |
| **설치 추가** | `pip install -e '.[acp]'` | 기본 설치에 포함 |
| **프로토콜** | stdio/JSON-RPC, ACP 전용 렌더링 | stdio 또는 HTTP |

**Chapter 11 scope:** MCP only (bidirectional). ACP deserves a brief mention and forward reference to a future chapter or link to docs.

#### Bidirectional MCP: Two Modes

**Mode 1 — Hermes as MCP CLIENT (외부 MCP 서버 연결)**

Connect any MCP server to Hermes for external tools (GitHub, databases, file systems, APIs).

**Command approach:**
```bash
# 검증: hermes rolling, 2026-06-10
hermes mcp                          # 인터랙티브 MCP 카탈로그 피커
hermes mcp catalog                  # 카탈로그 텍스트 목록
hermes mcp install <name>           # 큐레이션 카탈로그에서 설치
hermes mcp add <name> --preset <type>  # 사전 설정된 전송 방식으로 추가
hermes mcp configure <name>         # 설치 후 툴 선택 업데이트
hermes mcp login <server>           # OAuth 원격 서버 인증 완료
```

**Config approach (stdio 서버 — 로컬 프로세스):**
```yaml
# 검증: hermes rolling, 2026-06-10
mcp_servers:
  filesystem:
    command: "npx"
    args: ["-y", "@modelcontextprotocol/server-filesystem", "/home/user/my-project"]
    env:
      SOME_VAR: "value"
    tools:
      include: [read_file, list_directory]    # 허용 툴 화이트리스트
      exclude: [write_file]                   # 제외 (include가 있으면 무시됨)
    timeout: 30                               # 툴 실행 제한 시간 (초)
    supports_parallel_tool_calls: true        # 병렬 실행 허용
```

**Config approach (HTTP 서버 — 원격):**
```yaml
# 검증: hermes rolling, 2026-06-10
mcp_servers:
  remote_api:
    url: "https://mcp.example.com/mcp"
    headers:
      Authorization: "Bearer ${MY_API_TOKEN}"   # .env에서 참조
    ssl_verify: true
```

**Config approach (OAuth 인증 HTTP 서버):**
```yaml
# 검증: hermes rolling, 2026-06-10
mcp_servers:
  linear:
    url: "https://mcp.linear.app/mcp"
    auth: oauth
    # hermes mcp login linear 로 PKCE OAuth 완료
    # 토큰 캐시: ~/.hermes/mcp-tokens/linear.json
```

**OAuth PKCE:** Hermes handles discovery, dynamic client registration, PKCE, token exchange, refresh, and caching automatically. Tokens cached at `~/.hermes/mcp-tokens/<server>.json`.

**Tool naming:** MCP tools register as `mcp_<server>_<tool>` (hyphens/dots → underscores).

**Reload without restart:**
```
/reload-mcp
```

**Mode 2 — Hermes as MCP SERVER (Hermes를 외부 MCP 클라이언트에 노출)**

```bash
# 검증: hermes rolling, 2026-06-10
hermes mcp serve              # stdio MCP 서버 실행
hermes mcp serve --verbose    # 디버그 로깅
```

**IMPORTANT LIMITATION (HIGH confidence):** `hermes mcp serve` exposes **10 platform messaging tools** (Telegram, Discord, Slack 등 플랫폼 메시지 관리 도구). It reads conversation data from Hermes's session store. It is **NOT** a full "run Hermes as your agent" relay — it is specifically a messaging platform integration tool.

> 검증 필요: `hermes mcp serve`가 노출하는 정확한 10개 툴 이름을 로컬 실행 후 확인하십시오.

**Claude Desktop 연동 (HIGH confidence):**
```json
// ~/.claude/claude_desktop_config.json
// 검증: hermes rolling, 2026-06-10
{
  "mcpServers": {
    "hermes": {
      "command": "hermes",
      "args": ["mcp", "serve"]
    }
  }
}
```

**Cursor 연동:** Same pattern as Claude Desktop (exact Cursor config file path may vary — verify locally).

**VS Code / Zed:** Both support MCP clients — use the same `hermes mcp serve` command. Editor-specific UI configuration varies.

> 검증 필요: Cursor/VS Code/Zed의 정확한 MCP 클라이언트 설정 파일 경로 및 UI를 로컬에서 확인하십시오.

#### Full Config Reference Keys (HIGH confidence)

```yaml
# 검증: hermes rolling, 2026-06-10
mcp_servers:
  my-server:
    # 연결 (stdio 또는 HTTP 중 하나)
    command: "..."            # stdio 전용
    args: [...]               # stdio 전용
    env: {}                   # stdio 전용
    url: "https://..."        # HTTP 전용
    headers: {}               # HTTP 전용

    # 보안
    ssl_verify: true
    client_cert: "/path/cert.pem"
    client_key: "/path/key.pem"
    auth: oauth               # OAuth 2.1 PKCE

    # 런타임 제어
    enabled: true             # false면 건너뜀
    timeout: 30               # 툴 실행 시간 제한 (초)
    connect_timeout: 10       # 초기 연결 시간 제한

    # 툴 필터링
    supports_parallel_tool_calls: false
    tools:
      include: [tool1, tool2] # 화이트리스트 (설정 시 exclude 무시)
      exclude: [tool3]        # 블랙리스트
      resources: false        # 리소스 유틸리티 툴 비활성화
      prompts: false          # 프롬프트 유틸리티 툴 비활성화
```

**Precedence rule:** `include` takes precedence over `exclude` when both are set.

### Pitfall Callouts for Ch.11

**주의 1 — hermes mcp serve는 전체 에이전트 릴레이가 아님 (HIGH — 가장 중요한 오해 방지):**
```
> 주의: hermes mcp serve는 Hermes 에이전트 전체를 MCP로 노출하는 것이 아닙니다.
> 현재 구현은 플랫폼 메시징 툴(Telegram, Discord 등) 10개를 stdio MCP 서버로 노출합니다.
> Hermes를 에디터에서 완전히 실행하려면 ACP(hermes acp)를 사용하십시오.
> [정확한 노출 툴 수는 로컬 실행으로 확인 필요]
```

**주의 2 — ACP와 MCP는 별개 (HIGH):**
```
> 참고: ACP(Agent Client Protocol)와 MCP(Model Context Protocol)는 별도 통합입니다.
> ACP: Hermes를 VS Code/Zed/JetBrains의 에디터 네이티브 에이전트로 실행
> MCP: 외부 툴 서버를 Hermes에 연결하거나 Hermes를 MCP 서버로 노출
> 혼동하지 마십시오.
```

**주의 3 — include 우선순위 (HIGH):**
```
> 참고: mcp_servers 설정에서 tools.include와 tools.exclude를 동시에 설정하면
> include가 우선합니다. exclude는 무시됩니다.
> 화이트리스트 방식(include만 사용)이 보안상 권장됩니다.
```

**주의 4 — MCP 보안 (HIGH — A-12 관련):**
```
> 경고: MCP 서버 연결은 에이전트의 툴 표면을 확장합니다.
> tools.include 화이트리스트를 즉시 설정하여 필요한 툴만 허용하십시오.
> 신뢰할 수 없는 MCP 서버는 연결하지 마십시오.
> OAuth 원격 서버의 토큰은 ~/.hermes/mcp-tokens/에 캐시됩니다.
```

---

## Ch.12: 크론 스케줄러 (Cron Scheduler)

**Prerequisites:** Ch.9 (스킬 시스템 — 스킬 주입 이해), Ch.7 (툴 게이트웨이). Gateway 설정은 결과 배달에 필요하지만, 크론 자체는 gateway 없이도 local/files 배달로 동작함.

### Facts to State

#### Architecture (HIGH confidence — from ARCHITECTURE.md + official cron docs)

The cron scheduler lives **inside** the gateway process (`gateway/run.py`). However:
- `hermes cron` CLI commands work **independently** of whether gateway is running
- Jobs are stored in **JSON** in `~/.hermes/cron/` 
- Jobs execute in **fresh isolated AIAgent sessions** — they have zero knowledge of the operator's current conversation
- Output saved to `~/.hermes/cron/output/<job_id>/<timestamp>.md`
- Gateway ticks every **60 seconds**, checking `next_run_at`
- File locks prevent double-execution if tick intervals overlap

**RESOLVED: The cron CLI is `hermes cron` — NOT `hermes gateway`.**

#### Exact CLI Commands (HIGH confidence)

```bash
# 검증: hermes rolling, 2026-06-10
# 작업 관리
hermes cron list                              # 모든 작업 보기
hermes cron create "schedule" "prompt"        # 작업 추가
hermes cron edit <job_id> --schedule "..."    # 스케줄 수정
hermes cron pause <job_id_or_name>            # 일시 중지
hermes cron resume <job_id_or_name>           # 재개
hermes cron run <job_id_or_name>              # 즉시 수동 트리거
hermes cron remove <job_id_or_name>           # 삭제
hermes cron status                            # 전체 상태
hermes cron tick                              # 수동 tick (디버그용)
```

**Gateway service (결과 배달에 필요):**
```bash
# 검증: hermes rolling, 2026-06-10
hermes gateway install              # 사용자 서비스로 설치 (launchd/systemd)
sudo hermes gateway install --system # 시스템 서비스로 설치
hermes gateway                      # 포그라운드 실행
```

#### Schedule Formats (HIGH confidence)

| 형식 | 예시 | 동작 |
|------|------|------|
| 상대 지연 | `30m`, `2h`, `1d` | 1회 실행 |
| 인터벌 | `every 30m`, `every 2h` | 영구 반복 |
| Cron 표현식 | `0 9 * * *` (매일 오전 9시) | 영구 반복 |
| ISO 타임스탬프 | `2026-03-15T09:00:00` | 1회 지정 시간 |

#### Creating a Job — Full Examples (HIGH confidence)

**기본 예시:**
```bash
# 검증: hermes rolling, 2026-06-10
# 매일 오전 9시 요약 보고서 — Telegram으로 배달
hermes cron create "0 9 * * *" \
  "오늘의 주요 뉴스를 검색하고 한국어로 요약해서 알려줘" \
  --deliver telegram

# 2시간마다 블로그 모니터링 — 스킬 주입
hermes cron create "every 2h" \
  "새 블로그 포스트 확인 및 요약" \
  --skill blogwatcher \
  --deliver telegram \
  --name "blog-monitor"

# 특정 작업 디렉터리 설정 (A-20 Pitfall 방지)
hermes cron create "every 1d at 09:00" \
  "PR 감사 보고서 작성" \
  --workdir /home/me/projects/acme \
  --deliver "discord:#engineering"
```

#### Skill Injection (HIGH confidence)

Skills are injected into fresh isolated sessions **before** the prompt executes:

```bash
# 검증: hermes rolling, 2026-06-10
# 단일 스킬 주입
hermes cron create "every 1h" "피드 요약" --skill blogwatcher

# 복수 스킬 주입 (순서대로 로드)
hermes cron create "every 1h" "통합 브리핑" \
  --skill blogwatcher --skill maps
```

Skills provide reusable workflow guidance to the fresh cron session — this is why skills must be understood before this chapter.

#### Multi-Platform Delivery (HIGH confidence)

| 배달 대상 | 문법 |
|----------|------|
| 기본 (origin) | `"origin"` |
| 로컬 파일 | `"local"` → `~/.hermes/cron/output/` |
| Telegram | `"telegram"` |
| Discord 채널 | `"discord:#engineering"` |
| 복수 대상 | `"telegram,discord"` |
| 모든 설정된 플랫폼 | `"all"` |
| 복합 | `"origin,all"` |

Responses deliver automatically — no `send_message` call needed in the prompt.

#### No-Agent Mode (HIGH confidence)

For watchdog scripts that don't need LLM reasoning:

```bash
# 검증: hermes rolling, 2026-06-10
hermes cron create "every 5m" \
  --no-agent \
  --script memory-watchdog.sh \
  --deliver telegram \
  --name "memory-watchdog"
```

Script behavior:
- 빈 stdout → 조용히 종료 (watchdog 패턴)
- 비 0 exit → 에러 알림 배달
- 마지막 줄에 `{"wakeAgent": false}` → 조용히 종료

No token consumption.

#### Job Chaining with `context_from` (HIGH confidence)

```bash
# 에이전트 내부 cronjob 툴 사용 (파이썬 시그니처):
# cronjob(
#   action="create",
#   prompt="...",
#   schedule="30 7 * * *",
#   context_from="collector-job-id",   # 이전 작업 출력을 컨텍스트로 전달
#   name="AI News Triage"
# )
```

#### Config.yaml Cron Settings (HIGH confidence)

```yaml
# 검증: hermes rolling, 2026-06-10
cron:
  wrap_response: false              # 래퍼 헤더/푸터 억제
  script_timeout_seconds: 300       # 사전 실행 스크립트 시간 제한
```

#### Workdir Pitfall (HIGH confidence — A-20 from PITFALLS.md, now with specifics)

**The pitfall:** cron 작업에 `--workdir`를 지정하지 않으면, 작업이 Hermes 설치 디렉터리를 작업 디렉터리로 사용합니다. 파일 생성/읽기 경로가 예상과 다를 수 있습니다.

**The sequential side-effect:** workdir 지정 시 해당 작업은 **순차 실행**됩니다 (병렬 실행 불가). 이는 workdir 전환이 프로세스 전체 상태 변경이기 때문입니다. workdir 없는 작업들은 여전히 병렬 실행됩니다.

**Effects of workdir:**
- AGENTS.md, CLAUDE.md, .cursorrules 자동 로드
- terminal 및 file 작업의 기준 디렉터리
- 반드시 절대 경로; 상대 경로 거부됨
- 편집 시 `--workdir ""`로 해제

### Pitfall Callouts for Ch.12

**주의 1 — 크론 세션은 메모리가 없음 (HIGH — 가장 흔한 실수):**
```
> 경고: cron 작업은 완전히 새로운 에이전트 세션에서 실행됩니다.
> 현재 대화 내용, 메모리, 사용자 프로파일을 알지 못합니다.
> 필요한 모든 컨텍스트(URL, 설정, 기호)를 프롬프트에 명시적으로 포함시키십시오.
> 예시: "https://example.com/feed 에서 새 포스트를 확인해줘" (URL 명시)
```

**주의 2 — workdir 미설정 시 파일 경로 예측 불가 (A-20, HIGH):**
```
> 주의: --workdir를 지정하지 않으면 작업이 Hermes 설치 디렉터리에서 실행됩니다.
> 상대 경로로 파일을 생성하거나 읽는 작업에서 예상치 못한 경로 문제가 발생합니다.
> 파일 I/O가 있는 모든 cron 작업에는 반드시 --workdir /절대/경로 를 지정하십시오.
>
> 추가 주의: workdir 지정 시 해당 작업은 순차 실행됩니다 (병렬 실행 중단).
```

**주의 3 — macOS launchd PATH 문제 (A-15, HIGH):**
```
> 주의 (macOS): hermes gateway install 로 launchd 서비스를 설치한 후
> Homebrew나 nvm으로 새 도구를 설치하면, launchd의 PATH에 새 도구가 없습니다.
> 새 도구 설치 후 hermes gateway install을 재실행하여 PATH를 갱신하십시오.
```

**주의 4 — gateway 없이 cron 실행 불가 (MEDIUM):**
```
> 참고: hermes cron create 로 작업을 등록할 수 있지만,
> 실제 스케줄 실행은 gateway 프로세스가 필요합니다.
> hermes gateway 또는 hermes gateway install로 게이트웨이를 먼저 시작하십시오.
> hermes cron run <name> 으로 즉시 수동 실행은 gateway 없이도 가능합니다 (검증 필요).
```

> 검증 필요: `hermes cron run`이 gateway 프로세스 없이 독립적으로 실행 가능한지 로컬 확인.

---

## Ch.13: 서브에이전트 (Subagents & Parallel Work)

**Prerequisites:** Ch.9 (스킬 시스템), Ch.12 (크론 스케줄러 — 자동화 개념 이해).

### Facts to State

#### Two Distinct Systems (HIGH confidence — RESOLVES prior flag)

Chapter 13 covers **two separate but related features:**

1. **`delegate_task` (서브에이전트 위임):** Blocking RPC call that spawns isolated child agents for parallel reasoning tasks.
2. **Kanban (다중 에이전트 칸반 보드):** Durable SQLite-backed task queue for long-running, resumable, human-in-the-loop work.

Prior research described Kanban as merely "opt-in `kanban` toolset" — this is INCOMPLETE. Kanban is a full multi-agent coordination system with its own database, CLI, dashboard, and dispatch mechanism.

#### `delegate_task` — Subagent Spawning (HIGH confidence)

**Tool signature (agent calls this internally):**
```python
# 에이전트 내부 사용 — 독자는 자연어로 위임 요청
delegate_task(
    goal="목표 설명",
    context="서브에이전트가 알아야 할 모든 컨텍스트",
    toolsets=["web", "file"],
    max_iterations=30,         # 선택
    model="anthropic/claude-...",   # 선택: 모델 오버라이드
    provider="anthropic"       # 선택: 프로바이더 오버라이드
)

# 병렬 배치 모드
delegate_task(tasks=[
    {
        "goal": "WebAssembly 서버 사이드 현황 조사",
        "context": "runtimes, WASI, cloud/edge 사례 중점",
        "toolsets": ["web"]
    },
    {
        "goal": "RISC-V 서버 칩 채택 현황 조사",
        "context": "서버 칩, 클라우드 프로바이더 채택 현황",
        "toolsets": ["web"]
    },
    {
        "goal": "실용적인 양자 컴퓨팅 응용 사례 조사",
        "context": "에러 교정, 실제 사용 사례, 주요 기업",
        "toolsets": ["web"]
    }
])
```

**Context isolation model (HIGH confidence):**
- 서브에이전트는 **완전히 새로운 대화**로 시작 — 부모 대화 이력 없음
- 부모는 `goal`과 `context`로 모든 필요 정보를 명시적으로 전달해야 함
- 자식의 **최종 요약만** 부모에게 반환 — 중간 과정은 반환하지 않음
- 결과는 **입력 순서**로 정렬 (완료 순서와 무관)

**Concurrency (HIGH confidence):**
- 기본 최대 동시 서브에이전트: **3개** (구성 가능, 상한 없음)
- ThreadPoolExecutor로 병렬 실행
- 부모는 모든 자식이 완료될 때까지 블록됨
- 새 메시지 전송 또는 `/stop` → 활성 자식 취소

**Leaf subagent restrictions (HIGH confidence):**

다음 툴은 leaf 서브에이전트에서 차단됨:
- `delegation` (재귀 위임 방지)
- `clarify` (사용자 상호작용 불가)
- `memory` (메모리 격리)
- `code_execution`
- `send_message`

**Orchestrator pattern (HIGH confidence):**
```yaml
# 검증: hermes rolling, 2026-06-10
delegation:
  max_spawn_depth: 2          # 위임 중첩 허용 레벨 (기본 1)
  max_concurrent_children: 30 # 호출당 병렬 워커 수
```

`role="orchestrator"` 자식에게 위임 능력 유지. 기본 `max_spawn_depth: 1` (평면 구조).

**When to use `delegate_task` (HIGH confidence — from delegation patterns guide):**
- 추론 집약적 서브태스크 (디버깅, 코드 리뷰, 연구 합성)
- 중간 데이터로 컨텍스트가 넘칠 작업
- 독립적인 병렬 워크스트림

**Do NOT use `delegate_task` for:**
- 단일 툴 호출 → 직접 툴 사용
- 기계적 다단계 작업 → `execute_code`
- 사용자 상호작용 필요 → 서브에이전트는 `clarify` 불가
- 내구적 장기 작업 → Kanban 또는 `cronjob` 사용

#### Kanban Multi-Agent System (HIGH confidence — CORRECTS prior research)

**What it is:** A durable, SQLite-backed task queue + state machine for multi-agent work. NOT just a toolset toggle.

**Database:** `~/.hermes/kanban.db` (또는 named board: `~/.hermes/kanban/boards/<slug>/kanban.db`)

**Tables:** `tasks`, `task_links`, `task_comments`, `task_runs`, `task_events` (append-only event log for auditing)

**Task states:** `triage` → `todo` → `ready` → `running` → `blocked` / `done` / `archived`

**Initialization:**
```bash
# 검증: hermes rolling, 2026-06-10
hermes kanban init           # 선택적 — 첫 kanban 명령 시 자동 초기화
hermes dashboard             # 브라우저 대시보드 열기 (http://127.0.0.1:9119)
```

**Core CLI (HIGH confidence):**
```bash
# 검증: hermes rolling, 2026-06-10
hermes kanban create "태스크 제목" --assignee profile --tenant project
hermes kanban show <id>            # 태스크 상세
hermes kanban complete <id>        # 완료 처리
hermes kanban block <id>           # 차단 표시 (human-in-the-loop)
hermes kanban unblock <id>         # 차단 해제
hermes kanban decompose <id>       # 서브태스크로 분해
hermes kanban runs <id>            # 실행 이력
hermes kanban watch --kinds completed,gave_up,timed_out  # 이벤트 모니터링
```

**Dispatch (다중 프로파일 워커 실행):**
```bash
# 검증: hermes rolling, 2026-06-10
hermes gateway start    # 임베디드 dispatcher가 모든 프로파일 태스크를 처리
```

**Kanban vs `delegate_task` — 선택 기준 (HIGH confidence):**

| 항목 | Kanban | `delegate_task` |
|------|--------|-----------------|
| 실행 모델 | fire-and-forget 작업 큐 | 블로킹 함수 호출 |
| 부모 동작 | 태스크 생성 후 즉시 반환 | 자식 완료까지 블록 |
| 지속성 | SQLite 내구적 행 | 컨텍스트 압축 시 소실 |
| 인간 참여 | 지원 (comment/block/unblock) | 미지원 |
| 재개 가능성 | block → unblock → retry | 없음 — 실패는 최종 |
| 용도 | 장기, 복잡, human-in-the-loop 작업 | 짧은 추론 답변 |

**Worker tools (kanban toolset — 워커 에이전트가 사용):**
`kanban_show`, `kanban_list`, `kanban_complete`, `kanban_block`, `kanban_heartbeat`, `kanban_comment`, `kanban_create`, `kanban_link`, `kanban_unblock`

**Enabling kanban toolset for orchestrator profiles:**
```yaml
# 검증: hermes rolling, 2026-06-10
# config.yaml (orchestrator 프로파일)
toolsets:
  - hermes-cli
  - kanban            # 명시적으로 활성화 (기본 비활성)
```

Note: Kanban activates automatically for dispatcher-spawned workers via `HERMES_KANBAN_TASK` environment variable. Regular chat sessions need explicit profile configuration.

**Multi-Agent Coordination Patterns:**
- **Fan-out:** 병렬 형제 태스크
- **Pipeline:** 순차 역할 체인 (scout → editor → writer)
- **Voting/quorum:** 다수 에이전트 → 집계자
- **Long-running journals:** 메모리를 축적하는 영속 어시스턴트
- **Human-in-the-loop:** 사용자 입력 전 워커 블록

### Pitfall Callouts for Ch.13

**주의 1 — 서브에이전트 컨텍스트 블리드 없음 (HIGH — 가장 흔한 오해):**
```
> 주의: 서브에이전트는 부모의 대화 이력을 전혀 알지 못합니다.
> goal과 context 파라미터에 필요한 모든 정보를 명시적으로 전달해야 합니다.
> "알고 있는 내용" 참조 없이 완전히 자기 완결적인 컨텍스트를 작성하십시오.
```

**주의 2 — delegate_task 비용 (HIGH):**
```
> 주의: 각 서브에이전트는 독립적인 LLM 세션입니다.
> 3개 병렬 서브에이전트 = 최소 3×기본 오버헤드(~14K 토큰/세션).
> 단순 반복 작업이나 단일 툴 호출에는 서브에이전트보다 직접 툴을 사용하십시오.
```

**주의 3 — max_spawn_depth 기본값 1 (HIGH):**
```
> 참고: 기본 max_spawn_depth: 1 에서 서브에이전트는 추가 서브에이전트를 생성할 수 없습니다.
> 중첩 위임이 필요한 경우 delegation.max_spawn_depth: 2 이상으로 설정하십시오.
> 단, 깊은 중첩은 지수적 비용 증가를 초래합니다.
```

**주의 4 — Kanban은 delegate_task 대체재가 아님 (HIGH):**
```
> 참고: Kanban과 delegate_task는 상호 보완적이지 대체 관계가 아닙니다.
> 짧은 병렬 추론 → delegate_task
> 장기, 재개 가능, 인간 참여 작업 → Kanban
```

**주의 5 — Kanban 툴셋은 기본 비활성 (HIGH — corrects prior "opt-in" description):**
```
> 참고: Kanban 툴셋은 일반 채팅 세션에서 기본적으로 비활성화되어 있습니다.
> gateway dispatcher가 자동으로 활성화하거나, 프로파일 설정에서 명시적으로 추가해야 합니다.
> hermes dashboard 로 브라우저 대시보드를 열면 진행 상황을 시각적으로 확인할 수 있습니다.
```

---

## Resolved vs. Deferred Flags Summary

| 플래그 | 상태 | 결과 |
|--------|------|------|
| **Honcho 외부 계정 필요 여부** | RESOLVED — HIGH | 예, honcho.dev 계정 + API 키 필요. 자기 호스팅 가능. Ch.10 심화 aside로 처리. |
| **ACP vs MCP 구분** | RESOLVED — HIGH | 별개 통합. ACP = 에디터 임베딩. MCP = 툴 서버 연결 + Hermes MCP 서버. `hermes mcp serve`는 messaging tools 10개 노출. |
| **정확한 cron CLI** | RESOLVED — HIGH | `hermes cron create/list/edit/pause/resume/run/remove/status`. NOT `hermes gateway`. |
| **서브에이전트 생성 메커니즘** | RESOLVED — HIGH | `delegate_task` 툴 (에이전트 내부 호출). 최대 3개 동시, 구성 가능. |
| **Kanban toolset 상세** | RESOLVED — HIGH | Kanban은 단순 toolset이 아닌 완전한 다중 에이전트 협업 시스템. 별도 SQLite DB, CLI, dashboard, dispatcher. |

---

## Open Questions for Local Verification

1. **스킬 저장 제안 메시지의 정확한 문구**
   - What we know: Agent "often offers to save the approach as a skill" after ≥5 tool calls
   - What's unclear: Exact UI text, whether it's a prompt in-chat or a separate dialog
   - Recommendation: Run a complex multi-step task, observe exactly what the agent says, capture for Ch.10 hands-on

2. **`hermes skills list` 출력 형식**
   - What we know: Command confirmed
   - What's unclear: Column layout, whether it shows version/usage stats
   - Recommendation: Run locally, capture output for Ch.9

3. **`hermes mcp serve`가 노출하는 정확한 10개 툴 이름**
   - What we know: Exposes 10 platform messaging tools; reads from session store
   - What's unclear: Exact tool names (e.g., `send_telegram_message`?)
   - Recommendation: Run `hermes mcp serve --verbose`, connect an MCP client, list available tools

4. **`hermes cron run <name>`이 gateway 없이 독립 실행 가능한지**
   - What we know: `hermes cron run` command exists for immediate manual trigger
   - What's unclear: Whether this requires the gateway process to be running
   - Recommendation: Test with gateway stopped; if it fails, update Ch.12 pitfall callout

5. **`~/.hermes/skills/.usage.json` 정확한 경로와 형식**
   - What we know: skill usage tracking exists (`use_count`, `patch_count`, etc.)
   - What's unclear: Exact file location (`.usage.json` in skills root?)
   - Recommendation: After running skills, check `ls ~/.hermes/skills/` for hidden files

6. **스킬 인덱스가 시스템 프롬프트에 표시되는 정확한 형식**
   - What we know: Names and descriptions only (progressive disclosure); Layer 7 of prompt
   - What's unclear: Whether the index is a YAML list, plain text, or structured table in the prompt
   - Recommendation: Enable prompt debugging (`hermes --debug-prompt` if it exists) or check source code

7. **Cursor/VS Code/Zed의 MCP 클라이언트 설정 파일 위치**
   - What we know: Same `hermes mcp serve` command works; Claude Desktop config path confirmed
   - What's unclear: Cursor's JSON config path, VS Code extension settings format, Zed MCP config syntax
   - Recommendation: Test with each editor, capture exact configuration steps

8. **`hermes curator status` 출력 형식**
   - What we know: Displays last run, counts, pinned list, 5 least-recently-used skills
   - What's unclear: Exact format, whether useful for tutorial screenshot
   - Recommendation: Run `hermes curator status`, capture output for Ch.10

---

## Architecture Patterns (for planner)

### Chapter file template (same as Phase 3)

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

### Code block annotation convention (MANDATORY — inherited from Phase 2/3)

Every code block opens with:
```
# 검증: hermes rolling, 2026-06-10
```

Where output is unverified locally:
```
# [로컬 실행 후 캡처 필요 — 출력은 환경에 따라 다름]
```

---

## Sources

### Primary (HIGH confidence — official docs retrieved 2026-06-10)

- `https://hermes-agent.nousresearch.com/docs/user-guide/features/skills` — SKILL.md format, storage, auto-gen trigger, CLI commands, self-improvement
- `https://hermes-agent.nousresearch.com/docs/guides/work-with-skills` — Hands-on skill writing, community install commands
- `https://hermes-agent.nousresearch.com/docs/user-guide/features/curator` — Curator inactivity trigger, curator CLI, skill lifecycle states
- `https://hermes-agent.nousresearch.com/docs/user-guide/features/cron` — Complete cron CLI, schedule formats, skill injection, delivery, workdir behavior
- `https://hermes-agent.nousresearch.com/docs/guides/automate-with-cron` — Real-world cron patterns, memory constraint
- `https://hermes-agent.nousresearch.com/docs/user-guide/features/mcp` — hermes mcp serve, Claude Desktop config, catalog commands
- `https://hermes-agent.nousresearch.com/docs/reference/mcp-config-reference` — Full config.yaml keys for mcp_servers
- `https://hermes-agent.nousresearch.com/docs/guides/use-mcp-with-hermes` — MCP client setup steps
- `https://hermes-agent.nousresearch.com/docs/user-guide/features/acp` — ACP vs MCP distinction, supported editors
- `https://hermes-agent.nousresearch.com/docs/user-guide/features/delegation` — delegate_task parameters, context isolation, concurrency, restrictions
- `https://hermes-agent.nousresearch.com/docs/guides/delegation-patterns` — When to delegate, parallel example, orchestrator pattern
- `https://hermes-agent.nousresearch.com/docs/user-guide/features/kanban` — Kanban architecture, toolset enablement, vs delegate_task
- `https://hermes-agent.nousresearch.com/docs/user-guide/features/kanban-tutorial` — CLI commands, dashboard, multi-profile dispatch
- `https://hermes-agent.nousresearch.com/docs/user-guide/features/honcho` — External account requirement, config keys, setup command, self-hosted option
- `https://hermes-agent.nousresearch.com/docs/reference/skills-catalog` — ~89 bundled skills count correction
- `https://hermes-agent.nousresearch.com/docs/reference/optional-skills-catalog` — ~90 optional skills
- `https://hermes-agent.nousresearch.com/docs/llms.txt` — Doc index, all feature page URLs

### From Prior Research (imported and verified)

- `FEATURES.md`: A2 skills, A12 cron, A13 subagents, A16 MCP — verified and corrected where needed
- `ARCHITECTURE.md`: Cron inside gateway process, subagent Python RPC, MCP layer roles — confirmed
- `PITFALLS.md`: A-20 cron workdir, A-15 macOS launchd PATH — confirmed and expanded
- `03-RESEARCH.md`: Phase 3 established facts for skills Layer 7 placement, kanban opt-in status (now corrected to "full separate system") — Phase 3 Chapter template reused

---

## Metadata

**Confidence breakdown:**

| 챕터/토픽 | 신뢰도 | 근거 |
|-----------|--------|------|
| Ch.9 SKILL.md 형식 | HIGH | Official skills docs |
| Ch.9 skills CLI 커맨드 | HIGH | Official skills docs |
| Ch.9 번들 스킬 수 ~89개 | HIGH | Skills catalog reference |
| Ch.9 커뮤니티 허브 집계 수 | LOW | Multiple hubs — no single count available |
| Ch.10 Do→Learn→Improve 사이클 | HIGH | Skills + Curator official docs |
| Ch.10 Curator 비활동 트리거 | HIGH | Curator docs (corrects prior research) |
| Ch.10 Curator CLI 커맨드 | HIGH | Curator docs |
| Ch.10 Honcho 외부 계정 필요 | HIGH | Honcho integration docs |
| Ch.10 Honcho config 키 | MEDIUM | Honcho integration docs (파일 경로 구조 주의) |
| Ch.11 MCP 클라이언트 설정 | HIGH | MCP feature + config reference + guide |
| Ch.11 hermes mcp serve | HIGH | MCP feature docs |
| Ch.11 hermes mcp serve 노출 툴 | MEDIUM | "10 messaging tools" — exact names unverified |
| Ch.11 ACP vs MCP 구분 | HIGH | ACP + MCP docs 별도 존재 확인 |
| Ch.12 hermes cron CLI | HIGH | Cron feature + guide docs |
| Ch.12 스케줄 형식 | HIGH | Cron docs |
| Ch.12 Workdir 순차 실행 사이드이펙트 | HIGH | Cron docs |
| Ch.12 hermes cron run 독립 실행 가능성 | MEDIUM | Docs mentions manual trigger but gateway dependency unclear |
| Ch.13 delegate_task 파라미터 | HIGH | Delegation feature docs |
| Ch.13 Kanban 아키텍처 | HIGH | Kanban feature + tutorial docs |
| Ch.13 Kanban vs delegate_task 비교 | HIGH | Kanban docs explicit comparison |

**Research date:** 2026-06-10
**Valid until:** 2026-07-10 (Hermes is rolling release — re-verify commands 30 days before publish)
**Key risk:** Hermes rolling release. Especially at risk: exact `hermes mcp serve` tool list, `hermes skills list` output format, Honcho config file paths. Run all commands locally before marking chapters publication-ready.
