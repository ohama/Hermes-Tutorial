# Project Research Summary

**Project:** Korean mdBook Tutorial — Nous Research Hermes Agent
**Domain:** Developer tutorial (mdBook static site → GitHub Pages)
**Researched:** 2026-06-05
**Confidence:** HIGH

## Executive Summary

This project is a comprehensive Korean-language developer tutorial for Nous Research's Hermes Agent (open-source, MIT, v0.15.x), published as an mdBook static site deployed to GitHub Pages. The deliverable has two distinct stacks: the *tutorial toolchain* (mdBook 0.5.3 + GitHub Actions artifact-based deploy) and the *subject matter* (Hermes Agent install, CLI, and feature set). Both stacks are well-documented with official sources, and the research quality is consistently HIGH across all four areas.

The recommended approach is a dependency-aware 19-chapter structure organized into five logical parts: 기초 (foundations, Ch.0–3), 핵심 개념 (core concepts, Ch.4–8), 학습과 자동화 (learning loop, Ch.9–11), 플랫폼과 확장 (messaging/automation/advanced, Ch.12–16), and 운영 (deployment/security, Ch.17–18). Each chapter must pair conceptual explanation with hands-on CLI steps ("개념 + 실습 혼합형"), include expected output blocks, and end with a troubleshooting subsection. The self-improving Learning Loop (Skills + Memory) is the flagship differentiator and deserves the most tutorial space.

The dominant risk is accuracy drift: Hermes Agent is under fast-moving development, and undocumented or hallucinated CLI commands erode reader trust immediately. The mitigation is strict: every command must be verified against official docs before writing, annotated with a verification date/version, and the Korean UI (`display.language: ko`) must **not** be documented as supported because it is absent from the verified language list (`en|zh|ja|de|es|fr|tr|uk`). Use English UI screenshots and flag the limitation explicitly.

---

## Key Findings

### Recommended Stack

The deliverable toolchain is mdBook 0.5.3 (latest as of 2026-05-19) deployed via the official GitHub Actions starter workflow using `actions/deploy-pages@v5` — not the legacy `gh-pages` branch push approach. mdBook must be installed as a pre-built binary in CI (not via `cargo install`) to avoid 2–3 minutes of compile time. The `book.toml` must set `language = "ko"` and `site-url = "/hermes-tutorial/"` (required for subpath GitHub Pages deployments). The deprecated `curly-quotes` and `copy-fonts` keys must be removed — they cause build failures in 0.5.x.

The subject stack (what the tutorial teaches) requires only `git` as a user prerequisite on Linux/macOS/WSL2; the install script bundles Python 3.11 and Node.js v22 automatically. The CLI surface is large (70+ tools, 28 toolsets, 60+ slash commands) but well-documented in the official CLI reference.

**Core technologies:**

- **mdBook 0.5.3**: Static book generator — official Rust book toolchain, `language = "ko"` support, Korean-friendly
- **GitHub Actions + `actions/deploy-pages@v5`**: CI/CD pipeline — official GitHub-native approach, zero branch management
- **Pre-built mdBook binary**: Faster CI install — saves 2–3 min vs `cargo install`
- **Hermes Agent (rolling)**: Tutorial subject — install via `curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash`
- **`~/.hermes/config.yaml` + `.env`**: Hermes configuration — YAML for settings, `.env` for secrets (strict separation)

### Expected Features (Tutorial Coverage)

**Must have (P1 — tutorial is broken without these):**
- Installation walkthrough for all platforms (Linux/macOS/WSL2/Windows) with OS-split tabs
- Provider setup (Nous Portal OAuth or API key) and 64K context window model requirement
- Persistent Memory system (MEMORY.md, USER.md, SQLite FTS5 session search)
- Context Files hierarchy (`.hermes.md` → `AGENTS.md` → `CLAUDE.md` → `.cursorrules`)
- Skills system (SKILL.md format, agentskills.io standard, auto-generation trigger)
- Self-improving Learning Loop (flagship: 5+ tool calls → skill creation → self-improvement)
- Telegram Gateway setup (highest-value platform; first messaging chapter)
- Tool Gateway (web search, image generation, TTS, browser automation)
- SOUL.md and personality system

**Should have (P2 — makes this tutorial comprehensive):**
- Conversation compression config and cost management
- Approval mode and security model
- Cron scheduler with skill injection and multi-platform delivery
- Docker terminal backend for production isolation
- Subagent delegation and Kanban board
- Voice and TTS setup
- Additional gateways (Discord with Intents callout, Slack, WhatsApp)

**Defer to appendix/advanced (P3):**
- Honcho user modeling (requires external service; HIGH complexity)
- SSH, Modal, Daytona terminal backends
- MCP integration (bidirectional)
- Plugin system and batch processing
- API server mode (OpenAI-compatible endpoint)

### Architecture Approach

Hermes Agent uses a single synchronous `AIAgent` core (`run_agent.py`) shared across all entry points — CLI, Gateway, ACP editor, API server, and Cron. The system prompt is assembled in three tiers: stable identity (SOUL.md) → context/memory layer (context files + MEMORY.md/USER.md) → volatile block (timestamp, active skills). Memory persists across four layers: core Markdown files (~1.3K tokens, always loaded), SQLite FTS5 session history (on-demand recall), the Skills store (procedural memory, trigger-matched), and optional Honcho dialectic modeling. This architecture directly informs chapter ordering: readers must understand the agent loop before skills make sense, and skills before the learning loop makes sense.

The tutorial source mirrors this logical structure: `src/` with numbered chapter directories (`00-intro/` through `18-security/`), a flat `assets/` directory, and one `deploy.yml` workflow file. The mdBook `SUMMARY.md` groups chapters into five labeled parts using `---` separators.

**Major components:**

1. **AIAgent Core** (`run_agent.py`) — synchronous orchestration engine; single class for all entry points
2. **Prompt System** (`prompt_builder.py`, `context_compressor.py`) — 3-tier prompt assembly; compresses at 50% context window
3. **Provider Runtime + Transport Layer** — maps `(provider, model)` tuples to 18+ providers; credential pools, fallback chains
4. **Tool Registry** (`tools/registry.py`) — 70+ tools, `ThreadPoolExecutor` (up to 8 workers), 6 terminal backends
5. **Session DB** (`state.db`) — SQLite + FTS5; session history, memory, skill metadata
6. **Skills Store** (`~/.hermes/skills/`) — agentskills.io-standard Markdown files; auto-generated, self-improving
7. **Gateway** (`gateway/run.py`) — 20+ platform adapters sharing one agent loop; cron scheduler lives here
8. **mdBook + GitHub Actions** — tutorial deliverable: static site CI/CD to GitHub Pages

### Critical Pitfalls

**Tutorial-writing pitfalls (process guards):**

1. **Documenting unverified commands (B-1, CRITICAL)** — Hermes CLI changes across versions; hallucinated or stale commands immediately destroy reader trust. Every command block must be verified on a real install, annotated `# 검증: hermes vX.Y, YYYY-MM-DD`, and no command may be written from inference alone.

2. **Documenting Korean UI support (STACK.md LOW confidence flag)** — `display.language: ko` is NOT in the verified supported language list (`en|zh|ja|de|es|fr|tr|uk`). Tutorial must use English UI screenshots and explicitly note this limitation. Do not promise Korean UI.

3. **Version drift with no maintenance strategy (B-2, CRITICAL)** — Hermes has a fast release cycle. Each chapter must display its tested version; CI should compare against latest release. Links to external docs must use versioned anchors where available.

**Hermes user pitfalls (chapter callouts to include):**

4. **PATH not reloaded after install (A-1, CRITICAL)** — `hermes: command not found` after a successful install. Installation chapter must include `source ~/.bashrc` / `source ~/.zshrc` as an explicit numbered step immediately after the install command.

5. **API keys in `config.yaml` instead of `.env` (A-4, CRITICAL)** — Keys belong in `~/.hermes/.env`; `config.yaml` is for non-secret settings. Warn with a callout box; show the file separation principle early in the configuration chapter.

6. **64K token context window minimum not met (A-8, CRITICAL)** — Fixed overhead of ~13,935 tokens (tool definitions + system prompt) means models below 64K context fail silently or truncate. Model selection chapter must list this requirement prominently and include a recommended model table.

7. **Claude Pro/Max subscription does not equal Anthropic API access (A-6)** — Many developers assume their claude.ai subscription grants API access. Warn explicitly in the provider setup chapter.

8. **Nous Portal Tool Gateway requires paid subscription (A-7)** — Free Nous Portal accounts get inference but not Tool Gateway (web search, image gen, browser). Include a free/paid feature table.

---

## Implications for Roadmap

Based on research, the natural chapter structure is dependency-driven and falls into 7 work phases. Each phase corresponds to the logical tutorial parts identified across FEATURES.md, ARCHITECTURE.md, and the pitfall-to-phase map in PITFALLS.md.

### Phase 1: Project Scaffold and CI/CD

**Rationale:** Nothing else can be written until the mdBook repo exists and deploys cleanly to GitHub Pages. All downstream content phases depend on this working infrastructure.
**Delivers:** `book.toml` with `language = "ko"` and correct `site-url`, skeleton `src/SUMMARY.md` with all 19 chapter stubs, `.github/workflows/deploy.yml` using pre-built mdBook 0.5.3 binary and `actions/deploy-pages@v5`, live GitHub Pages URL.
**Addresses:** Table stakes (copy-pasteable content requires a working site).
**Avoids:** B-12 (`site-url` misconfiguration), B-10 (stale CI Action versions), B-11 (SUMMARY.md duplicate paths), deprecated `curly-quotes`/`copy-fonts` keys.

### Phase 2: Foundations (Ch.0–3 — 기초)

**Rationale:** These four chapters are strict prerequisites for every other chapter. Readers who cannot install Hermes and connect a model get no value from the rest of the tutorial.
**Delivers:** Introduction + what-is-Hermes framing (Ch.0), complete multi-OS install walkthrough with PATH reload step and `sudo` warning (Ch.1), first conversation + TUI/CLI navigation (Ch.2), model selection with 64K requirement and provider setup (Ch.3).
**Implements:** STACK.md install commands (curl/ps1), post-install `hermes setup --portal` flow, `hermes doctor` verification.
**Avoids:** A-1 (PATH), A-2 (sudo), A-3 (Windows native), A-6 (Claude subscription), A-8 (64K context), B-3 (missing prerequisites), B-4 (no expected output).

### Phase 3: Core Concepts (Ch.4–8 — 핵심 개념)

**Rationale:** These five chapters build the conceptual frame that makes all advanced features intelligible. Agent loop → prompt system → memory → tools → backends is the precise dependency order from ARCHITECTURE.md.
**Delivers:** Agent core and conversation loop internals (Ch.4), 3-tier prompt system + context files + SOUL.md + compression (Ch.5), 4-layer memory architecture + FTS5 session search (Ch.6), Tool Gateway with web/image/TTS/browser tools (Ch.7), terminal backends local → Docker → SSH → cloud (Ch.8).
**Addresses:** P1 features: context files, memory, tool gateway, SOUL.md/personalities.
**Avoids:** A-4 (API keys in config.yaml), A-7 (Tool Gateway subscription confusion), A-9 (token cost spike), A-14 (auxiliary model context too small), B-6 (macOS-only testing).

### Phase 4: Learning Loop (Ch.9–11 — 학습과 자동화)

**Rationale:** Skills and the self-improving loop are the flagship differentiator — the feature that makes Hermes worth a comprehensive tutorial. They must follow memory and tools (prerequisite: understanding the agent already executes tools and stores sessions).
**Delivers:** Skills system — SKILL.md format, agentskills.io standard, auto-generation trigger (5+ tool calls), community skills hub (Ch.9); Do→Learn→Improve cycle, skill self-improvement, ~40% task time reduction data point (Ch.10); MCP integration bidirectional (Ch.11).
**Addresses:** P1 features: skills, learning loop. This is the tutorial's value center.
**Avoids:** Teaching Skills before the agent loop (ARCHITECTURE.md anti-pattern 3).

### Phase 5: Platforms and Automation (Ch.12–16 — 플랫폼과 확장)

**Rationale:** Messaging gateways, cron, and subagents all require the agent to be fully working (Phases 2–4). These are the "always-on agent" features that justify running Hermes on a server.
**Delivers:** Messaging gateway architecture + Telegram (Ch.12, lead with highest-value platform), Cron scheduler with skill injection and delivery (Ch.13), Subagent delegation + Kanban (Ch.14), Plugin system (Ch.15), API server mode (Ch.16).
**Addresses:** P2 features: Telegram gateway, cron, subagents. P3: plugins, API server.
**Avoids:** A-10 (Discord Intents — full step with screenshot), A-11 (Telegram duplicate polling), A-12 (approvals off warning), A-20 (cron workdir unset), B-9 (intermediate cliff — cron/subagents after gateway).

### Phase 6: Deployment and Security (Ch.17–18 — 운영)

**Rationale:** Production deployment and security hardening are the final phase because they require understanding all prior systems — you cannot secure what you have not yet built.
**Delivers:** Deployment strategies per scenario ($5 VPS, Docker, SSH, Modal, Daytona) (Ch.17), security model — approval modes, credential management, PII redaction, multi-profile isolation, `hermes doctor` operational runbook (Ch.18).
**Addresses:** P2 features: Docker backend, security model.
**Avoids:** A-12 (approvals.mode: off in production), A-13 (Docker root-owned files), A-15 (macOS launchd PATH), A-2 (sudo install).

### Phase 7: Reference Appendix

**Rationale:** A complete CLI reference, slash command reference, full config.yaml key reference, and consolidated troubleshooting list are high-value for readers returning after initial setup. Deferred to last because content depends on all prior chapters being finalized.
**Delivers:** Full CLI command reference, slash command cheatsheet, config.yaml annotated reference, master troubleshooting index (aggregates all chapter "흔한 오류" sections), glossary of agent-specific terms.
**Addresses:** Table stakes B-5 (no troubleshooting) at scale; improves searchability.

---

### Phase Ordering Rationale

- **Scaffold first (Phase 1):** All content phases write into the mdBook repo; the pipeline must exist before any content has a home.
- **Strict prerequisite order (Phases 2–4):** ARCHITECTURE.md explicitly maps Ch.0→1→2→3→4 as a linear chain before any branching occurs. Installing Hermes and choosing a model must precede understanding the agent loop.
- **Learning Loop after Core Concepts (Phase 4):** Skills make no sense without understanding tool execution and session persistence. FEATURES.md feature dependency map confirms: `[A14 Provider] → ALL`, `[A3 Memory] → [A1 Learning Loop]`, `[A1] → [A2 Skills]`.
- **Platforms after Loop (Phase 5):** Gateways require a working agent; cron requires gateway for delivery; subagents require terminal backends from Phase 3.
- **Security last (Phase 6):** Docker hardening, approval mode, and multi-profile isolation are meaningful only after the reader has deployed something real.
- **Appendix independent (Phase 7):** Can be drafted in parallel with late phases but must be finalized last.

### Research Flags

**Phases needing deeper research during planning:**

- **Phase 4 (Learning Loop):** Honcho integration details (requires external account? exact config keys?) are MEDIUM confidence. Verify before writing Ch.10. ACP vs MCP distinction needs confirmation in current v0.15.x.
- **Phase 5 (Platforms):** Exact Discord OAuth and Privileged Gateway Intents flow may have changed post-2026-03. Verify before writing the Discord subchapter.
- **Phase 6 (Deployment):** Modal and Daytona backend config fields not deeply documented in public sources. Verify exact YAML keys before writing serverless deployment sections.

**Phases with standard patterns (skip additional research-phase):**

- **Phase 1 (Scaffold):** mdBook 0.5.3 + `deploy-pages@v5` workflow is fully documented in the official starter workflow. No ambiguity.
- **Phase 2 (Foundations):** Install commands, `hermes setup --portal`, `hermes doctor` all HIGH confidence from official docs.
- **Phase 3 (Core Concepts):** Agent loop, prompt tiers, memory layers, tool registry all documented in official architecture guide. HIGH confidence.

---

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack (mdBook + GitHub Actions) | HIGH | Official starter workflow verified; all Action versions confirmed; 0.5.x breaking changes catalogued |
| Stack (Hermes CLI + install) | HIGH | Verified against official docs and `cli-commands.md` in repo |
| Features | HIGH | All P1/P2 features sourced from official docs + release notes v0.13–v0.15.2 |
| Architecture | HIGH | Official architecture page + multiple corroborating sources; component boundaries confirmed |
| Pitfalls (Hermes usage) | HIGH | A-1 through A-8 grounded in official docs, GitHub issues, and community troubleshooting guides |
| Pitfalls (tutorial writing) | HIGH | B-series grounded in documentation best-practice research + project constraints |
| Korean UI (`display.language: ko`) | LOW | NOT in verified supported language list; tutorial must use English UI and flag the gap |

**Overall confidence:** HIGH

### Gaps to Address

- **Korean UI support:** `display.language: ko` is unverified. The tutorial must not claim Korean UI support. Use English UI, add a note in the configuration chapter that Korean interface translation is not currently available.
- **Honcho integration:** Whether Honcho requires a separate paid API account is unclear from public docs. Treat as optional advanced feature; verify before writing Ch.6 Honcho sub-section and Ch.10.
- **Subagent RPC protocol:** Internal Python RPC mechanism described but not deeply documented. Write subagent chapter at behavioral/CLI level; avoid claiming specifics about internal wire protocol.
- **Transport layer internals (v0.11.0+):** `agent/transports/` ABC described but not fully documented publicly. Stay at provider/model selection level; do not document transport internals.
- **Hermes version pinning:** Installer uses rolling release. Tutorial cannot pin a global Hermes version. Use per-chapter version annotations (`# 검증: hermes vX.Y, 날짜`) instead.

---

## Sources

### Primary (HIGH confidence)
- `https://github.com/NousResearch/hermes-agent` — README, release notes v0.13–v0.15.2, install instructions
- `https://hermes-agent.nousresearch.com/docs/reference/cli-commands` — Full CLI reference
- `https://hermes-agent.nousresearch.com/docs/user-guide/configuration` — `config.yaml` full schema
- `https://hermes-agent.nousresearch.com/docs/getting-started/installation` — Install prerequisites and paths
- `https://hermes-agent.nousresearch.com/docs/developer-guide/architecture` — Component details, data flow
- `https://rust-lang.github.io/mdBook/` — mdBook 0.5.3 docs, SUMMARY.md format, book.toml options
- `https://github.com/actions/starter-workflows/blob/main/pages/mdbook.yml` — Official Actions workflow
- `https://github.com/rust-lang/mdBook/blob/master/CHANGELOG.md` — 0.5.x breaking changes
- `https://honcho.dev/docs/v3/guides/integrations/hermes` — Honcho dialectic memory integration

### Secondary (MEDIUM confidence)
- `https://lushbinary.com/blog/hermes-agent-developer-guide-setup-skills-self-improving-ai/` — Architecture corroboration
- `https://www.turingpost.com/p/hermes` — Memory layers, learning loop detail
- `https://kisztof.medium.com/hermes-agent-review-nous-researchs-self-improving-ai-agent-e72bc244435a` — Skills system, learning loop
- `https://github.com/mudrii/hermes-agent-docs` — Community documentation
- GitHub Issue #16394 — `hermes setup` silent API key skip behavior
- GitHub Issue #4379 — Token overhead analysis (~13.9K fixed tokens per call)

### Tertiary (LOW confidence)
- `display.language: ko` support — NOT in documented language list; needs verification with Nous Research before any claim is made in tutorial content

---
*Research completed: 2026-06-05*
*Ready for roadmap: yes*
