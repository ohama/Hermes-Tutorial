# Feature Research

**Domain:** Korean mdBook developer tutorial — Nous Research Hermes Agent
**Researched:** 2026-06-05
**Confidence:** HIGH (A-side: Hermes features) / HIGH (B-side: tutorial craft)

---

## Part A — Hermes Agent Feature Catalog

Hermes Agent is Nous Research's open-source, self-hosted, self-improving AI agent (MIT license, released February 2026, currently v0.15.x). Every feature below is sourced from the official repository, official docs at hermes-agent.nousresearch.com, and release notes. Unverifiable claims are explicitly flagged LOW confidence.

---

### A1 — Self-Improving Learning Loop (The Core Differentiator)

**What it is:** After completing a complex task (heuristic: 5+ tool calls), the agent automatically creates a Markdown skill document capturing the workflow, commands, decision branches, pitfalls, and verification steps. On subsequent similar tasks, that skill is loaded into context. Every 15 tasks, a "periodic nudge" evaluates overall skill quality and prunes/updates stale entries.

**Why it matters:** The agent compounds capability through use. Unlike static agents, Hermes improves its own procedures over time without manual intervention.

**How it works (5-stage loop):**
1. Task completes → pattern extraction phase
2. Agent identifies reusable multi-step workflow
3. Skill file written to `~/.hermes/skills/` as `SKILL.md`-format document
4. Future similar tasks trigger semantic match → skill loaded into context
5. Post-task reconciliation updates skill if new edge cases found

**Complexity:** HIGH — requires understanding skills, memory, and the agent loop together.
**Learning order dependency:** Must understand tool execution and memory first (A3, A4).
**Tutorial space needed:** Full chapter — this is the flagship feature.
**Confidence:** HIGH

---

### A2 — Skills System (agentskills.io Standard)

**What it is:** Skills are Markdown files with YAML frontmatter describing reusable procedures. They follow the agentskills.io open standard (launched December 2025 by Anthropic, widely adopted by March 2026). Any skill built for Hermes works across 25+ compatible agent platforms.

**SKILL.md structure:**
```yaml
---
name: [skill-identifier]
description: [brief explanation]
triggers: [activation phrases]
version: [number]
---
```
Body: prerequisites, numbered steps with commands, decision branches, known issues, verification methods.

**Progressive disclosure loading:** Only skill names and descriptions (index, ~500–1,000 tokens) are loaded into the system prompt. Full skill content is loaded on-demand when a trigger matches. 200 skills cost almost nothing until activated.

**Skill types:**
- Auto-generated (by the agent after complex tasks)
- Self-improving (agent refines on subsequent use, increments version)
- Manually authored (developer writes)
- Community skills via HermesHub / agentskills.io catalog (Skills Hub ships 19,932+ entries as of v0.15.1)

**Conditional activation:** Triggers and description determine relevance; context-aware matching before full content loads. Custom skills take priority over bundled skills with identical triggers.

**Skill bundles (v0.15.0+):** Multiple skills packaged under a single slash command for one-shot activation.

**Complexity:** MEDIUM — format is simple Markdown, but understanding trigger/loading mechanics and self-improvement requires solid grounding.
**Learning order dependency:** Learning loop (A1). Can be introduced in parallel with memory (A3).
**Tutorial space needed:** Full chapter with hands-on skill authoring exercise.
**Confidence:** HIGH

---

### A3 — Persistent Memory System

**What it is:** Two bounded Markdown files injected into the system prompt at session start, plus a full-text search layer over all past conversations.

**Components:**

| Component | File/Store | Capacity | Purpose |
|-----------|-----------|----------|---------|
| Agent notes | `~/.hermes/memories/MEMORY.md` | ~2,200 chars / ~800 tokens | Environment facts, conventions, learned workarounds |
| User profile | `~/.hermes/memories/USER.md` | ~1,375 chars / ~500 tokens | User preferences, communication style, expectations |
| Session history | `~/.hermes/state.db` (SQLite FTS5) | Unbounded | Full-text search across all past conversations |

**FTS5 session search (v0.15.0 rebuild: 4,500x faster, zero LLM cost):** Returns raw messages without LLM summarization. Enables recall of conversations from weeks prior.

**Memory tool actions:** `add`, `replace` (substring match), `remove`. Agent uses `§` delimiters between entries. Usage percentage shown in system prompt header.

**Curation heuristics — save:** user preferences, environment facts, project conventions, workarounds, completed work. Skip: trivial facts, code dumps, session-specific content, content already in SOUL.md.

**Capacity management:** Returns error when at limit; agent should consolidate at 80% capacity.

**Security:** Memory entries scanned for prompt injection and credential exfiltration patterns.

**External memory providers (pluggable backends):** Honcho, Mem0, Hindsight, OpenViking, Holographic, RetainDB, ByteRover, Supermemory — configured via `hermes memory setup`.

**Complexity:** LOW-MEDIUM — core concept is simple; FTS5 and external providers add depth.
**Learning order dependency:** None — can be taught early (chapter 3–4).
**Tutorial space needed:** One chapter covering both local memory and session search; external providers in advanced section.
**Confidence:** HIGH

---

### A4 — Honcho User Modeling (Dialectic Memory)

**What it is:** An optional AI-native memory backend (developed by Plastic Labs) that builds dynamic user profiles through dialectic LLM reasoning rather than static file storage. Runs alongside built-in memory files.

**Dialectic process (3-pass):**
1. **Initial assessment** — cold-start (new user) or warm (session-aware) prompts
2. **Self-audit** — identifies gaps in initial assessment
3. **Reconciliation** — resolves contradictions, synthesizes findings
Supports 1–3 configurable passes per invocation; auto-terminates early on strong signal.

**Two-layer context injection:**
- **Base layer:** session summaries, user representation, peer card (refreshed per `contextCadence`)
- **Dialectic layer:** synthesized reasoning about current user state (refreshed per `dialecticCadence`)

**Config knobs:**

| Setting | Purpose | Default |
|---------|---------|---------|
| `contextCadence` | Base context refresh frequency | every 1 turn |
| `dialecticCadence` | LLM reasoning invocation frequency | every 2 turns |
| `dialecticDepth` | Passes per invocation | 1 |

**Recall modes:** hybrid (auto-inject + tools), context (inject only), tools (explicit model calls).
**Tools added when active:** `honcho_profile`, `honcho_search`, `honcho_context`, `honcho_reasoning`, `honcho_conclude`.

**Complexity:** HIGH — requires understanding of external service setup, API keys, and the dialectic concept.
**Learning order dependency:** Base memory (A3) must be understood first.
**Tutorial space needed:** Sub-section within Memory chapter or dedicated advanced chapter.
**Confidence:** HIGH (official docs + Honcho integration guide)

---

### A5 — Context Files

**What it is:** Project-specific instruction files automatically discovered and injected into the agent's system prompt at session start. Provides per-project or per-directory behavioral customization without editing global config.

**Priority cascade (first match wins):**
`.hermes.md` → `AGENTS.md` → `CLAUDE.md` → `.cursorrules`

**Loading behavior:**
- Top-level file: loaded upfront into system prompt
- Subdirectory AGENTS.md files: discovered lazily via `subdirectory_hints.py` during tool calls, injected into tool results — NOT loaded upfront

**Context references (`@` syntax):** Type `@` + reference to inject files, folders, git diffs, and URLs directly into messages with automatic inline expansion.

**SOUL.md distinction:** `SOUL.md` is the global agent identity (in `$HERMES_HOME`, not project dir). Context files are project-scoped.

**Complexity:** LOW — simple file placement, but understanding priority cascade matters.
**Learning order dependency:** Basic setup (install + config). Should be taught early alongside SOUL.md.
**Tutorial space needed:** Focused sub-chapter within Configuration.
**Confidence:** HIGH

---

### A6 — Tool Gateway (Nous Tool Gateway)

**What it is:** A routing layer (available to paid Nous Portal subscribers) that proxies web search, image generation, TTS, and browser automation through a subscription rather than requiring individual API keys per service.

**Tools routed through gateway:**
- **Web search:** Multiple backends — SearxNG, Firecrawl, Tavily, Exa, Parallel (configurable per `web.backend`)
- **Image generation:** FLUX 2 Klein/Pro, GPT-Image 1.5/2, Nano Banana Pro, Ideogram V3, Recraft V4 Pro, Qwen, Z-Image Turbo (via FAL.ai)
- **TTS (text-to-speech):** 10 providers — Edge TTS (free), ElevenLabs, OpenAI TTS, MiniMax, Mistral Voxtral, Google Gemini, xAI, NeuTTS, KittenTTS, Piper
- **Browser automation:** Browserbase cloud, Browser Use cloud, local Chrome/Brave/Chromium/Edge via CDP, local Chromium (CamoFox)

**Per-tool `use_gateway` control:** Each tool can independently route via gateway or direct API.

**Complexity:** LOW for basic usage; MEDIUM for custom provider configuration.
**Learning order dependency:** LLM provider setup (A14) recommended first.
**Tutorial space needed:** Dedicated "Tools & Tool Gateway" chapter covering each tool category.
**Confidence:** HIGH

---

### A7 — Conversation Compression

**What it is:** Automatic or manual summarization of conversation history to reduce token count when approaching context limits.

**Trigger modes:**
- **Automatic:** Triggers at configurable threshold (`compression.threshold`, default 0.50 = 50% context used)
- **Manual:** `/compress` slash command

**Config:**
```yaml
compression:
  enabled: true
  threshold: 0.50      # trigger at 50% context window fill
  target_ratio: 0.20   # compress down to 20% of window
  protect_last_n: 20   # always preserve last 20 messages
```

**Auxiliary model:** Compression can use a different, cheaper model than the main agent (configured via `auxiliary.compression.*`).

**Complexity:** LOW — config-driven, `/compress` is a one-command fix.
**Learning order dependency:** None, but makes more sense after understanding context window concepts.
**Tutorial space needed:** Brief section within Configuration or Tips & Tricks chapter.
**Confidence:** HIGH

---

### A8 — Personalities & SOUL.md

**What it is:** Two-layer system for agent identity — a persistent global persona file (SOUL.md) and session-level personality overlays (`/personality` command).

**SOUL.md:**
- Stored at `~/.hermes/SOUL.md` (or `$HERMES_HOME/SOUL.md`)
- Occupies slot #1 in system prompt (the agent identity position)
- Auto-created with defaults if absent; never overwritten if present
- Only loaded from HERMES_HOME (not project directory) — ensures consistent personality across all projects
- Contains: tone/style, directness level, uncertainty handling, stylistic avoidances
- NOT for project-specific instructions (those go in AGENTS.md)

**`/personality` overlays (session-level):**
- Built-in: `helpful`, `concise`, `technical`, `creative`, `teacher`
- Novelty: `kawaii`, `catgirl`, `pirate`, `shakespeare`, `surfer`, `noir`, `uwu`, `philosopher`, `hype`
- Custom: define in `config.yaml` under `agent.personalities`
- Cleared at session end; do not alter SOUL.md

**Complexity:** LOW — simple file editing and command usage.
**Learning order dependency:** Basic setup. Best introduced early in tutorial (chapter 2–3).
**Tutorial space needed:** One focused section; pair with Context Files.
**Confidence:** HIGH

---

### A9 — Approval Mode

**What it is:** A safety layer that controls whether the agent must get user confirmation before executing potentially dangerous terminal commands.

**Three modes:**
- **`manual`** — Prompts user before every flagged command (default/safest)
- **`smart`** — Uses auxiliary LLM to assess risk; auto-approves safe operations, prompts only for risky ones
- **`off`** — Disables all approval checks (use only in trusted isolated environments)

**Tirith security scanning:** Examines terminal commands pre-execution; configurable timeout with fail-open behavior (command proceeds if scanner times out).

**Config:**
```yaml
approvals:
  mode: manual   # manual | smart | off
```

**Complexity:** LOW — config-driven, but students need to understand the risk model.
**Learning order dependency:** Terminal backends (A15) — approval mode is most relevant when discussing what the agent executes.
**Tutorial space needed:** Section within Security chapter.
**Confidence:** HIGH

---

### A10 — Messaging Gateways

**What it is:** A unified gateway process that bridges 22+ messaging platforms to the same agent core. All subsystems (gateway, CLI, ACP server, cron scheduler) share one agent loop, ensuring consistent behavior across all entry points.

**Supported platforms (as of v0.15.x):**
Telegram, Discord, Slack, WhatsApp, Signal, Matrix, Microsoft Teams, Email, SMS, DingTalk, Feishu/Lark, WeCom (Enterprise WeChat), Google Chat, LINE, SimpleX Chat, Mattermost, QQBot (added v0.14.0), BlueBubb + others.

**Key capabilities:**
- DM pairing for platform authentication
- Context persists across platforms (start on Telegram, continue on CLI)
- `/handoff` command for live session transfer between platforms (v0.14.0)
- Native button UI for clarification prompts on Telegram/Discord
- Voice channel conversations on Discord
- Session persistence across gateway restarts (v0.13.0)

**Authentication:** Each platform has its own credential setup flow (`hermes gateway`, `hermes whatsapp`, `hermes slack`, etc.)

**Complexity:** MEDIUM — each platform has different credential/setup complexity.
**Learning order dependency:** Basic agent setup must work first. Recommend teaching 2–3 platforms (Telegram + one more) before covering all.
**Tutorial space needed:** Full chapter — architecture overview + per-platform setup guides for highest-value platforms.
**Confidence:** HIGH

---

### A11 — Voice Memo Transcription

**What it is:** Voice messages received via messaging gateways (WhatsApp, Telegram, etc.) are automatically transcribed to text and processed as regular text messages by the agent.

**TTS providers (10 options):** Edge TTS (free), ElevenLabs, OpenAI TTS, MiniMax, Mistral Voxtral, Google Gemini, xAI, NeuTTS, KittenTTS, Piper.

**Voice input (CLI):** Microphone recording via configurable hotkey (`voice.record_key: ctrl+b`), with `auto_tts: false` by default.

**Full voice mode:** Real-time interaction across CLI and messaging platforms including Discord voice channel conversations.

**Complexity:** LOW for transcription alone; MEDIUM for full bidirectional voice setup.
**Learning order dependency:** Messaging gateways (A10) for platform voice memos; basic setup for CLI voice.
**Tutorial space needed:** Sub-section within Gateways chapter.
**Confidence:** HIGH (docs confirmed); voice memo transcription listed in feature overview.

---

### A12 — Cron Scheduler

**What it is:** A built-in scheduler that runs tasks automatically on your server with full access to the agent's memory and skills. Delivers results via any configured messaging platform.

**Key features:**
- Natural language or standard cron expression scheduling
- Skill attachment (attach a skill to provide workflow guidance)
- Multi-platform result delivery
- Managed via `hermes cron` CLI

**Example use cases:** Daily summaries delivered to Telegram, weekly report generation, periodic web monitoring, automated data processing.

**Complexity:** LOW-MEDIUM — cron expressions are standard; integration with skills and gateways adds depth.
**Learning order dependency:** Basic agent setup, gateway setup (A10), skills (A2).
**Tutorial space needed:** Dedicated section within an Automation chapter.
**Confidence:** HIGH

---

### A13 — Subagent Spawning (Delegation)

**What it is:** The `delegate_task` tool spawns child agent instances with isolated context, restricted toolsets, and their own terminal sessions. Enables parallel workstreams.

**How it works:**
- Child agents start with completely blank conversation history
- Parent passes only `goal` and `context` parameters explicitly
- Up to 3 concurrent subagents by default (configurable, no hard ceiling)
- Only child's final summary returns to parent — efficient token usage
- Results sorted by task index regardless of completion order

**Restrictions on leaf subagents (default):** Cannot access `delegate_task`, `clarify`, `memory`, `send_message`, or `execute_code`.

**Orchestrator children:** Can delegate further but face depth limits (`max_spawn_depth: 1` default) to prevent runaway recursion.

**Synchronous blocking:** Parent blocks until all children complete. New message or `/stop` cancels active children.

**Multi-agent Kanban board (v0.13.0+):** Full multi-agent platform with durability and persistence across the Kanban workflow.

**Complexity:** HIGH — context isolation model is non-obvious; orchestrator depth limits require careful understanding.
**Learning order dependency:** Tool execution, terminal backends (A15), basic agent loop.
**Tutorial space needed:** Advanced chapter — subagent patterns, use cases, limitations.
**Confidence:** HIGH

---

### A14 — Multi-Model Support & Provider Routing

**What it is:** Model-agnostic design supporting 300+ models across multiple providers. Fine-grained routing controls which provider handles each request type.

**Supported providers:** Nous Portal (300+ models), OpenRouter (200+ models), OpenAI, Anthropic (native), NovitaAI, NVIDIA NIM, xAI/Grok (SuperGrok OAuth, 1M context), Kimi/Moonshot, MiniMax, Hugging Face, Xiaomi MiMo, custom OpenAI-compatible endpoints.

**Model selection:** `/model` command in-session; `hermes model` CLI; `config.yaml` default.

**Auxiliary task routing:** Vision, web extraction, and compression can use different (cheaper) models than the primary agent.

**Provider features:**
- Fallback providers: automatic failover on errors
- Credential pools: distribute API calls across multiple keys with rotation strategies (round_robin, fill_first, least_used, random)
- Prompt caching: cross-session 1-hour prefix cache on Claude (Anthropic, OpenRouter, Nous Portal) — always-on
- Provider routing: sorting, whitelists, blacklists, priority ordering

**Complexity:** LOW for basic setup; MEDIUM for routing and credential pooling.
**Learning order dependency:** None — this is the first thing to configure.
**Tutorial space needed:** Chapter 1 (Getting Started) + brief advanced section on routing.
**Confidence:** HIGH

---

### A15 — Terminal Backends (Execution Environments)

**What it is:** Six execution environments controlling where and how the agent runs shell commands. Each offers different isolation, persistence, and infrastructure tradeoffs.

| Backend | Isolation | Persistence | Use Case | Complexity |
|---------|----------|-------------|----------|------------|
| **Local** | None | Native FS | Development/testing | Trivial |
| **Docker** | Strong (cap drops, `--pids-limit 256`) | Container FS | Production, security-sensitive | Low |
| **SSH** | Network boundary | Remote server FS | Remote execution, always-on VPS | Low |
| **Singularity/Apptainer** | Namespace isolation | HPC cluster | Academic/HPC environments | Medium |
| **Modal** | Cloud sandbox | Configurable snapshots | Serverless, cost optimization | Medium |
| **Daytona** | Managed workspace | Stop/resume (10GB disk) | Cloud dev environments | Medium |

**Lazy-installed backends (v0.14.0):** Non-default backends are installed only when first selected, reducing base install footprint.

**Config:** `config.yaml` terminal section; switchable mid-session.

**Complexity:** LOW for local/SSH; MEDIUM for Docker/cloud backends.
**Learning order dependency:** Basic installation. Docker should be covered before production deployment.
**Tutorial space needed:** Section within Getting Started (local default) + advanced chapter covering Docker, SSH, and cloud options.
**Confidence:** HIGH

---

### A16 — MCP Integration (Model Context Protocol)

**What it is:** Bidirectional Model Context Protocol support. Hermes can act as an MCP client (connecting to external MCP servers) or as an MCP server (exposing Hermes to IDEs and tools).

**MCP Client mode:** Connect to any MCP server via stdio or HTTP transport for external tools (databases, APIs, file systems) without native Hermes development. MCP catalog with interactive picker (v0.15.0).

**MCP Server mode (v0.6.0+):** Expose Hermes as an OpenAI-compatible HTTP endpoint. Compatible with Open WebUI, LobeChat, LibreChat, etc.

**ACP (Agent Communication Protocol):** IDE integration with VS Code, Zed, JetBrains for in-editor chat and file management.

**Complexity:** MEDIUM — requires understanding of MCP concepts and server setup.
**Learning order dependency:** Basic setup, tool system (A6).
**Tutorial space needed:** Section within Integrations chapter.
**Confidence:** HIGH

---

### A17 — Checkpoints & Rollback

**What it is:** Automatic filesystem snapshots of the working directory taken before any file modifications. Enables `/rollback` to recover from mistakes.

**How it works:** Before any destructive file operation, Hermes saves a checkpoint. The `/rollback` slash command restores the most recent checkpoint.

**Complexity:** LOW — transparent by default; `/rollback` is a one-command operation.
**Learning order dependency:** Basic tool execution.
**Tutorial space needed:** Brief section, good to mention early for safety reassurance.
**Confidence:** HIGH

---

### A18 — Plugins & Extension System

**What it is:** Three plugin types extending Hermes beyond built-in capabilities: general plugins (custom tools), memory provider plugins, and context engine plugins. Managed via unified UI.

**Event hooks:** Custom code at lifecycle points — gateway hooks (logging, alerts, webhooks) and plugin hooks (tool interception, metrics).

**Batch processing:** Parallel processing across hundreds/thousands of prompts for training data generation (ShareGPT format trajectory export for fine-tuning/RL experiments).

**Complexity:** HIGH — requires Python development.
**Learning order dependency:** Full understanding of tools, skills, and memory.
**Tutorial space needed:** Advanced/Developer chapter.
**Confidence:** HIGH

---

### Feature Dependency Map

```
[A14 Multi-Model / Provider Setup]
    └──required for──> ALL FEATURES (agent cannot run without LLM)

[A15 Terminal Backends]
    └──required for──> [A13 Subagents] (each child needs a terminal)
    └──required for──> [A12 Cron] (cron jobs execute in terminal)

[A3 Memory System]
    └──enhances──> [A1 Learning Loop] (skills reference memory)
    └──required for──> [A4 Honcho] (Honcho supplements, not replaces)

[A1 Learning Loop]
    └──produces──> [A2 Skills] (loop auto-creates skills)
    └──requires understanding──> [A3 Memory] (same mental model)

[A2 Skills]
    └──enhances──> [A12 Cron] (attach skills to scheduled jobs)
    └──enhances──> [A13 Subagents] (skills guide subagent behavior)

[A5 Context Files]
    └──complements──> [A8 Personalities/SOUL.md] (different scopes)

[A10 Gateways]
    └──required for──> [A11 Voice Memo] (memos come via gateway)
    └──required for──> [A12 Cron] (deliver results via gateway)

[A6 Tool Gateway]
    └──requires──> [A14 Provider Setup] (Nous Portal subscription)

[A9 Approval Mode]
    └──configures behavior of──> [A15 Terminal Backends]

[A16 MCP]
    └──extends──> [A6 Tool Gateway] (MCP servers add more tools)
```

---

## Part B — What Makes a Strong Developer Tutorial (mdBook)

### B1 — Table Stakes (Missing = Tutorial Feels Broken)

| Feature | Why Expected | Complexity to Implement | Notes |
|---------|--------------|------------------------|-------|
| Clear prerequisites section | Readers need to know entry requirements | LOW | List OS, Python, API keys, minimum context |
| Copy-pasteable commands | Manual transcription causes errors | LOW | Every command in a fenced code block |
| Expected output blocks | Readers need to verify their result matches | LOW | Use `# expected output:` comments or separate blocks |
| Installation walkthrough (all OSes) | macOS / Linux / WSL2 differ | MEDIUM | Test each path; note Windows native limitations |
| Version pinning | Tutorials go stale without version context | LOW | Document which Hermes version was tested |
| Logical chapter progression | Learning should build on prior knowledge | MEDIUM | Respect feature dependency order |
| Troubleshooting section per chapter | First-time users will hit errors | MEDIUM | Cover top 3–5 errors per chapter with fixes |
| Section summaries / what-you-learned | Reinforces retention | LOW | 3–5 bullet recap at chapter end |
| Working code/config examples | Abstract docs are not enough | MEDIUM | All YAML/config snippets must be real and tested |

### B2 — Differentiators (Competitive Advantage for This Tutorial)

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| Korean-native explanations | No awkward translated phrasing | HIGH | Write concepts in Korean idiom, not translated English |
| Concept + Hands-on pairing per chapter | 혼합형 format beats either alone | MEDIUM | Theory → immediate practice in same chapter |
| Annotated config files | Comments explaining WHY each setting exists | LOW | Add `# 이 설정은 X 때문에 필요합니다` style comments |
| Real-world use case scenarios | Abstract features become concrete | MEDIUM | Each chapter opens with "when would you use this?" |
| Progressive complexity | Beginner → intermediate → advanced within each feature | MEDIUM | Chapter sections: 기초 → 응용 → 심화 |
| Feature interaction examples | Show how skills + cron + gateways work together | HIGH | Late chapters covering combined workflows |
| Korean developer context examples | Use Korean-specific services (Kakao, Naver) where relevant | MEDIUM | Platform-specific setup for Korean users |
| Verified, tested commands | Run every command on fresh install before publishing | HIGH | Include version/date tested in chapter header |
| Side-by-side comparison with alternatives | OpenClaw, standard CLI agents | MEDIUM | Helps readers understand positioning |

### B3 — Anti-Features (Avoid These)

| Anti-Feature | Why Problematic | What to Do Instead |
|--------------|----------------|-------------------|
| Walls of prose without code | Readers lose engagement; hard to follow | Interleave explanation and code; max 2–3 paragraphs before next code block |
| Unverified commands | "command not found" destroys trust | Test every command on a clean environment before publishing |
| Screenshots of terminal output | Go stale instantly; not searchable; not copy-pasteable | Use fenced code blocks with `# expected output:` comment |
| Covering every option equally | Creates cognitive overload | Opinionated defaults first; advanced options in sidebar/appendix |
| Assuming global prior knowledge | Readers may be new to agents | Define agent-specific jargon (LLM, MCP, FTS5, dialectic) when first used |
| Configuration-first ordering | Config means nothing without conceptual frame | Concept → minimal working example → then full config options |
| Skipping error cases | Real tutorials help when things go wrong | Every setup chapter needs a "흔한 오류" (common errors) section |
| Feature dump without workflow | Listing all 40 tools without showing use | Introduce tools in context of a real task |
| Ignoring model cost implications | Readers may run up unexpected API bills | Note token costs and recommend cheapest viable model for learning |
| Chapters with no exercises | Passive reading does not produce competence | Each chapter ends with at least one exercise with expected output |

---

## Proposed Tutorial Chapter Groupings

Based on feature dependency order and Korean developer tutorial conventions:

### Group 1 — 시작하기 (Getting Started)
- Prerequisites & Concepts (what is Hermes, what problem it solves)
- Installation (Linux/macOS/WSL2 paths)
- First run & provider setup (A14 — must come first)
- Basic CLI usage + slash commands
- SOUL.md & Personalities (A8 — early personality, low barrier)

**Table stakes for tutorial structure:** This group must be complete before any other chapter makes sense.

### Group 2 — 핵심 기능 (Core Features)
- Persistent Memory (A3 — MEMORY.md, USER.md, FTS5)
- Context Files (A5 — .hermes.md, AGENTS.md hierarchy)
- Configuration deep dive (A7 Compression, A9 Approval Mode, backends intro)
- Tools & Tool Gateway (A6 — search, image gen, TTS, browser)

### Group 3 — 자기학습 루프 (The Learning Loop)
- How Skills Work — agentskills.io standard, SKILL.md format (A2)
- The Self-Improving Learning Loop (A1 — flagship feature)
- Writing Custom Skills + community skills (HermesHub)
- Skill Bundles (v0.15.0+)

### Group 4 — 메시징 & 자동화 (Messaging & Automation)
- Gateway Architecture overview
- Telegram setup (A10 — start with highest-value platform)
- Additional gateways: Discord, Slack, WhatsApp, Signal (A10)
- Voice & TTS (A11)
- Cron Scheduler (A12)

### Group 5 — 고급 기능 (Advanced Features)
- Terminal Backends — Docker, SSH, cloud (A15)
- Subagent Delegation & Kanban (A13)
- Honcho User Modeling (A4)
- MCP Integration (A16)
- Plugin System & Batch Processing (A18)

### Group 6 — 배포 & 운영 (Deployment & Operations)
- Deployment options ($5 VPS, Docker, Modal, Daytona)
- Security best practices (Approval Mode, credential management)
- Monitoring & logs
- Updating & maintenance

### Group 7 — 부록 (Appendix)
- CLI command reference
- Slash command reference
- Configuration reference (all config.yaml keys)
- Troubleshooting master list

---

## Feature Prioritization Matrix (Tutorial Coverage)

| Feature | Learner Value | Implementation Cost (tutorial) | Priority |
|---------|--------------|-------------------------------|----------|
| Installation + Provider Setup (A14) | HIGH | MEDIUM | P1 |
| Persistent Memory (A3) | HIGH | LOW | P1 |
| Skills & Learning Loop (A1, A2) | HIGH | HIGH | P1 |
| Telegram Gateway (A10) | HIGH | MEDIUM | P1 |
| Context Files (A5) | HIGH | LOW | P1 |
| SOUL.md / Personalities (A8) | MEDIUM | LOW | P1 |
| Tool Gateway — search (A6) | HIGH | LOW | P1 |
| Compression (A7) | MEDIUM | LOW | P2 |
| Approval Mode (A9) | MEDIUM | LOW | P2 |
| Cron Scheduler (A12) | HIGH | MEDIUM | P2 |
| Docker Backend (A15) | HIGH | MEDIUM | P2 |
| Voice & TTS (A11) | MEDIUM | MEDIUM | P2 |
| Subagent Delegation (A13) | HIGH | HIGH | P2 |
| Honcho (A4) | MEDIUM | HIGH | P3 |
| SSH / Modal / Daytona backends (A15) | MEDIUM | MEDIUM | P3 |
| MCP Integration (A16) | MEDIUM | MEDIUM | P3 |
| Plugin System (A18) | LOW | HIGH | P3 |
| Additional Gateways (A10) | MEDIUM | MEDIUM | P3 |

**Priority key:**
- P1: Must cover for tutorial to be useful
- P2: Should cover — differentiates this tutorial as comprehensive
- P3: Include in advanced/appendix chapters; defer if scope pressure

---

## Sources

- [Hermes Agent GitHub (NousResearch/hermes-agent)](https://github.com/NousResearch/hermes-agent) — README, release notes (v0.13.0–v0.15.2), directory structure. **Confidence: HIGH**
- [Official Documentation — hermes-agent.nousresearch.com/docs](https://hermes-agent.nousresearch.com/docs/) — Features overview, configuration, personality, memory, delegation pages. **Confidence: HIGH**
- [Honcho integration guide — honcho.dev/docs/v3/guides/integrations/hermes](https://honcho.dev/docs/v3/guides/integrations/hermes) — Dialectic memory details. **Confidence: HIGH**
- [agentskills.io / Anthropic Agent Skills standard](https://simonwillison.net/2025/Dec/19/agent-skills/) — Open standard launched December 2025. **Confidence: HIGH**
- [Lushbinary Hermes developer guide](https://lushbinary.com/blog/hermes-agent-developer-guide-setup-skills-self-improving-ai/) — Architecture corroboration. **Confidence: MEDIUM** (third-party)
- [Release tracker — github.com/NousResearch/hermes-agent/releases](https://github.com/NousResearch/hermes-agent/releases) — v0.13–v0.15.2 changelogs. **Confidence: HIGH**

---

*Feature research for: Korean mdBook developer tutorial — Nous Research Hermes Agent*
*Researched: 2026-06-05*
*Confidence: HIGH (A-features grounded in official repo/docs) | HIGH (B-tutorial craft from established best practices)*
