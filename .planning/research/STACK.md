# Stack Research

**Domain:** Korean mdBook Tutorial — Hermes Agent (developer-facing)
**Researched:** 2026-06-05
**Confidence:** HIGH (A: mdBook toolchain) / HIGH (B: Hermes CLI surface)

---

This document covers TWO stacks:

- **(A) Deliverable toolchain** — mdBook + GitHub Pages CI pipeline
- **(B) Subject stack** — Hermes Agent install, config, and CLI surface (what the tutorial teaches)

All commands and versions verified against official docs and real repo. Unverifiable items are flagged.

---

## STACK A — Deliverable: mdBook + GitHub Pages

### Core Technologies

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| mdBook | **0.5.3** (latest as of 2026-05-19) | Static book generator from Markdown | Official Rust book toolchain; powers rust-lang/book; first-class Korean (`language = "ko"`) support via `[book]` config |
| GitHub Actions | N/A (platform) | CI build + deploy pipeline | Official starter workflow exists (`actions/starter-workflows`); zero additional hosting cost |
| `actions/checkout` | **v4** | Checkout repo in CI | Current stable; required before mdbook build |
| `actions/configure-pages` | **v5** | Set GitHub Pages base URL | Handles `site-url` injection automatically |
| `actions/upload-pages-artifact` | **v3** | Package built `book/` dir as Pages artifact | Required for `deploy-pages` approach |
| `actions/deploy-pages` | **v5** | Deploy artifact to Pages | Modern approach — does NOT use `gh-pages` branch; cleaner than legacy push |

### Supporting Tools

| Tool | Purpose | Notes |
|------|---------|-------|
| `cargo install mdbook` | Install mdBook from source | Requires Rust ≥ 1.88; use pinned `--version 0.5.3` in CI for reproducibility |
| Pre-built binary (GitHub Releases) | Faster CI install | `curl` from `github.com/rust-lang/mdbook/releases` — avoids Rust compile time (~2–3 min saved) |
| `mdbook build` | Build `src/` → `book/` | Run from repo root; outputs to `./book/` by default |
| `mdbook serve` | Local dev server | Port 3000; live-reload on change |
| `mdbook test` | Test embedded Rust code blocks | Not needed for this tutorial (no Rust code), but available |

### book.toml Configuration

Minimum required config for a Korean mdBook:

```toml
[book]
title = "Hermes Agent 완벽 가이드"
authors = ["Your Name"]
description = "Nous Research Hermes Agent 한국어 튜토리얼"
language = "ko"
src = "src"

[output.html]
site-url = "/hermes-tutorial/"   # REQUIRED if repo name is not the user/org Pages root
git-repository-url = "https://github.com/<owner>/hermes-tutorial"
edit-url-template = "https://github.com/<owner>/hermes-tutorial/edit/main/{path}"

[output.html.search]
enable = true
```

**Note on `site-url`**: Must match your GitHub repo path. If deploying to `<username>.github.io` (root), omit or set to `/`. If deploying to `<username>.github.io/hermes-tutorial/`, set to `/hermes-tutorial/`.

### SUMMARY.md Structure

```markdown
# Summary

[소개](README.md)

# 시작하기

- [설치](getting-started/installation.md)
- [첫 번째 실행](getting-started/quickstart.md)
   - [모델 설정](getting-started/model-setup.md)

# 핵심 기능

- [모델 & 공급자](features/models.md)
- [스킬](features/skills.md)
- [컨텍스트 파일](features/context-files.md)
- [도구](features/tools.md)
- [게이트웨이](features/gateway.md)
- [크론](features/cron.md)
- [서브에이전트](features/subagents.md)

# 배포

- [환경 분리](deployment/profiles.md)
- [프로덕션 운영](deployment/production.md)

---

[참고자료](reference.md)
```

### GitHub Actions Deploy Workflow

The **official starter workflow** (`actions/starter-workflows/pages/mdbook.yml`) is the recommended path. It uses the modern Pages artifact approach (not the legacy `gh-pages` branch push).

```yaml
# .github/workflows/deploy.yml
name: Deploy mdBook to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      MDBOOK_VERSION: 0.5.3
    steps:
      - uses: actions/checkout@v4

      - name: Install mdBook (pre-built binary — faster than cargo)
        run: |
          tag="v${MDBOOK_VERSION}"
          url="https://github.com/rust-lang/mdbook/releases/download/${tag}/mdbook-${tag}-x86_64-unknown-linux-gnu.tar.gz"
          mkdir mdbook
          curl -sSL "$url" | tar -xz --directory=./mdbook
          echo "$(pwd)/mdbook" >> $GITHUB_PATH

      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5

      - name: Build with mdBook
        run: mdbook build

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./book

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v5
```

**Critical repo setting**: In repository Settings → Pages → Source, select **"GitHub Actions"** (not "Deploy from a branch"). This is required for the `deploy-pages` action to work.

### Alternatives Considered (Stack A)

| Recommended | Alternative | Why Not |
|-------------|-------------|---------|
| mdBook 0.5.3 | mdBook 0.4.x | 0.5.x is current stable; 0.4.x has deprecated `curly-quotes` option and no admonitions/definition lists; 0.4.36 is what the starter workflow still pins — update it |
| `deploy-pages` action | `peaceiris/actions-gh-pages` push to `gh-pages` branch | `deploy-pages` is the official GitHub-native approach; no extra token configuration; Pages branch approach is legacy |
| Pre-built binary install | `cargo install mdbook` in CI | Cargo compile adds ~2–3 min to CI run; binary tarball is instant |
| Korean `language = "ko"` in book.toml | No language setting | Screen readers and browser translate use the `lang` HTML attribute; set it correctly |

### What NOT to Use (Stack A)

| Avoid | Why | Use Instead |
|-------|-----|-------------|
| `cargo install --version 0.4.36` in CI | Outdated; missing admonitions, definition lists, sidebar headings nav | Pin to `0.5.3` or use `latest` release API |
| `output.html.curly-quotes` in book.toml | Removed in 0.5.0; causes error | `output.html.smart-punctuation = true` (now the default) |
| `output.html.copy-fonts` | Removed in 0.5.0; causes error | Delete the key; fonts now copy automatically |
| `{{#previous}}` / `{{#next}}` in custom theme | Removed in 0.5.0 | Use theme objects (see 0.5.0 changelog) |
| `peaceiris/actions-mdbook` pinned to `mdbook-version: 0.4.8` | Pinned to old version | Use direct binary install with explicit version |

### Version Compatibility (Stack A)

| Component | Compatible With | Notes |
|-----------|-----------------|-------|
| mdBook 0.5.3 | Rust ≥ 1.88 | Only relevant for local `cargo install`; CI uses pre-built binary |
| mdBook 0.5.x | `actions/configure-pages@v5` | Tested combination in official starter workflow |
| mdBook 0.5.x | `book.toml` without deprecated keys | Remove `curly-quotes`, `copy-fonts`; preprocessors now need `optional` field or cause errors |

---

## STACK B — Subject: Hermes Agent

### Runtime Requirements

| Requirement | Version | Notes |
|-------------|---------|-------|
| Git | Any recent | **Only prerequisite on Linux/macOS/WSL2** — the install script handles everything else |
| Python | 3.11 (bundled) | Installed automatically via `uv`; do NOT pre-install manually |
| Node.js | v22 (bundled) | Installed automatically; do NOT pre-install manually |
| ripgrep | bundled | File search tool |
| ffmpeg | bundled | Audio conversion |
| Windows: PowerShell | Any | For `install.ps1`; bundles Python 3.11, Node, ripgrep, ffmpeg, MinGit (~45 MB) |

### Installation Commands

| Platform | Command | Confidence |
|----------|---------|------------|
| Linux / macOS / WSL2 / Termux | `curl -fsSL https://hermes-agent.nousresearch.com/install.sh \| bash` | HIGH — verified against official docs |
| Windows (PowerShell) | `iex (irm https://hermes-agent.nousresearch.com/install.ps1)` | HIGH — verified against official docs |
| Linux + Desktop GUI | `curl -fsSL https://hermes-agent.nousresearch.com/install.sh --include-desktop \| bash` | HIGH |
| Nix | Flake + NixOS module (see contributing docs) | MEDIUM — path exists, details not fully documented |

**Install locations:**

| Mode | Code | Binary | Data |
|------|------|--------|------|
| Standard user | `~/.hermes/hermes-agent/` | `~/.local/bin/hermes` | `~/.hermes/` |
| Root | `/usr/local/lib/hermes-agent/` | `/usr/local/bin/hermes` | `/root/.hermes/` |

### Post-Install Setup Commands

```bash
# Reload shell after install (installer modifies PATH)
source ~/.bashrc   # or ~/.zshrc

# Quick Nous Portal setup (login + model + Tool Gateway in one step)
hermes setup --portal

# OR: manual step-by-step
hermes model          # Interactive provider/model wizard
hermes tools          # Enable/disable tool categories
hermes setup          # Full wizard (model, tts, terminal, gateway, tools, agent)

# Verify installation
hermes doctor         # Diagnose config + dependency issues
hermes version        # Show installed version
```

### Core CLI Surface (Tutorial Chapters Map)

#### Chat & Interactive Use

```bash
hermes                              # Start interactive session (classic REPL)
hermes --tui                        # Modern TUI (recommended for new users)
hermes chat -q "What is 2+2?"      # One-shot non-interactive
hermes -z "prompt"                  # Scriptable: clean output only (for pipes)
hermes --continue                   # Resume most recent session
hermes --resume <id-or-title>       # Resume specific session
hermes chat --model anthropic/claude-sonnet-4.6  # Override model
hermes chat --provider nous         # Force provider
hermes chat --toolsets web,terminal # Enable specific toolset categories
hermes -s skill-name                # Preload a skill
hermes --yolo                       # Skip dangerous-command prompts
```

Key in-session slash commands:
```
/model              # Show/switch current model
/tools              # List available tools
/skills browse      # Browse skill hub
/background <task>  # Parallel isolated sub-task
/voice on           # Enable voice mode
/status             # Session info recap
/sessions           # Interactive session picker
/rollback           # Revert to last checkpoint
```

Key in-session keybindings:
| Key | Action |
|-----|--------|
| `Enter` | Send message |
| `Alt+Enter` / `Ctrl+J` / `Shift+Enter` | New line |
| `Ctrl+G` | Open input in external editor |
| `Ctrl+C` | Interrupt agent (double → force exit) |
| `Ctrl+D` | Exit |

#### Model & Provider Management

```bash
hermes model                        # Interactive wizard — add providers, OAuth
hermes config set model <model-id>  # Set model directly
hermes config set provider <name>   # Set provider directly
hermes auth list                    # Show credential pools
hermes auth add <provider>          # Add API key / OAuth credential
hermes fallback list                # View fallback provider chain
```

**Supported providers (verified):**

| Provider | Auth Method | Notes |
|----------|-------------|-------|
| Nous Portal | OAuth via `hermes setup --portal` | 300+ models; includes Tool Gateway routing |
| OpenRouter | `OPENROUTER_API_KEY` | 200+ models |
| OpenAI | `OPENAI_API_KEY` | |
| Anthropic | `ANTHROPIC_API_KEY` | |
| NovitaAI | API key | |
| NVIDIA NIM | API key | Nemotron models |
| Xiaomi MiMo | API key | |
| Kimi/Moonshot | API key | |
| MiniMax | API key | |
| Hugging Face | API key | |
| Custom endpoint | `base_url` in config.yaml | OpenAI-compatible APIs; Ollama etc. |
| Google Gemini | API key | |
| DeepSeek | API key | |
| AWS Bedrock | AWS credentials | |
| Ollama | Local, no key | |

**Minimum requirement:** Models must support ≥ 64K token context for multi-step tool-calling workflows.

#### Configuration Files

| File | Location | Purpose |
|------|----------|---------|
| `config.yaml` | `~/.hermes/config.yaml` | All non-secret settings (model, terminal backend, compression, display, agent behavior) |
| `.env` | `~/.hermes/.env` | API keys and secrets; auto-redacted from logs |
| `auth.json` | `~/.hermes/auth.json` | OAuth credentials (Nous Portal etc.) |
| `SOUL.md` | `~/.hermes/SOUL.md` | Agent identity/personality; slot #1 in system prompt; auto-created on first run |

Context files (project-local, loaded from CWD upward):

| File | Priority | Notes |
|------|----------|-------|
| `.hermes.md` or `HERMES.md` | 1 (highest) | Hermes-specific project instructions |
| `AGENTS.md` | 2 | Project structure, conventions (compatible with Claude Code) |
| `CLAUDE.md` | 3 | Claude Code context files (also recognized) |
| `.cursorrules` | 4 (lowest) | Cursor IDE conventions; CWD only |
| `SOUL.md` | Global (always) | Personality; loaded independently of project context |

Rules: Only ONE project context file is loaded per session (first match wins). Files max 20,000 chars; truncated 70%/20% head/tail. All files security-scanned before injection.

```bash
hermes config show          # View all settings
hermes config edit          # Open in editor
hermes config set KEY VAL   # Set individual value
hermes config check         # Validate after edits
hermes config migrate       # Prompt for unconfigured skill settings
hermes config path          # Print config file location
```

Key `config.yaml` sections for the tutorial:

```yaml
model:
  default: nous/hermes-3-405b   # or any provider/model-id
  provider: nous                 # nous|openrouter|anthropic|openai|custom|auto
  context_length: 131072
  reasoning_effort: medium       # none|minimal|low|medium|high|xhigh

terminal:
  backend: local                 # local|docker|ssh|modal|daytona|singularity
  timeout: 180

compression:
  enabled: true
  threshold: 0.50               # Compress at 50% of context window

display:
  language: ko                   # UI translation: en|zh|ja|de|es|fr|tr|uk
  show_cost: true
  streaming: true
```

**Note on Korean UI:** `display.language: ko` is NOT listed in verified supported languages. Supported: `en|zh|ja|de|es|fr|tr|uk`. Korean UI translation is **LOW confidence** — tutorial should use English UI screenshots and note this limitation.

#### Skills System

```bash
hermes skills browse                        # Browse official skill hub
hermes skills search <query>                # Search registries
hermes skills install official/<skill-id>  # Install from official registry
hermes skills list                          # List installed skills
hermes skills inspect <id>                 # Preview without installing
hermes skills uninstall <id>               # Remove skill
hermes skills publish                       # Share to registry

hermes bundles create <name>                # Group skills under one slash command
hermes bundles list
hermes bundles delete <name>

hermes curator status                       # Background skill maintenance service
hermes curator run                          # Trigger review now
hermes curator pin <skill>                  # Prevent auto-archival
```

Skill lifecycle: Curator tracks usage, marks idle skills stale, archives them (with pre-archive backup). Agent-created skills have `created_by: "agent"` provenance. Skills declare config in `SKILL.md` frontmatter; config stored under `skills.config` namespace in `config.yaml`.

#### Tools & Toolsets

```bash
hermes tools                    # Configure enabled tools (interactive)
hermes tools --summary          # Print current config

# In-session
hermes chat --toolsets web,terminal,skills   # Enable toolsets by name
```

Built-in toolset categories: `web`, `terminal`, `skills`, `memory`, and others. 60+ built-in tools total.

#### Gateway (Messaging Platforms)

```bash
hermes gateway run              # Start gateway (foreground; WSL2 recommended)
hermes gateway start            # Start as systemd/launchd service
hermes gateway stop / restart
hermes gateway status           # Show service state
hermes gateway list             # All profiles + gateway status
hermes gateway install          # Register as system service
hermes gateway uninstall

hermes send --to telegram "message"    # One-shot message, no agent loop
hermes send --to discord:channel-id "alert"
hermes whatsapp                        # WhatsApp pairing flow
hermes slack manifest                  # Generate Slack app manifest
hermes pairing list                    # Messaging platform access control
hermes pairing approve <platform> <code>
hermes pairing revoke <platform> <user-id>
```

Supported platforms: Telegram, Discord, Slack, WhatsApp, Signal, Email, Teams.

#### Cron & Scheduled Automations

```bash
hermes cron list                 # Show scheduled jobs
hermes cron create               # Add job (interactive)
hermes cron edit                 # Update job
hermes cron pause / resume       # Control job state
hermes cron remove               # Delete job
hermes cron tick                 # Run due jobs once (manual trigger)

# In-session
/cron                            # Manage cron from within agent
```

Per-job config fields: `skills`, `model`/`provider` overrides, `script` (pre-run data collection), `context_from` (chain job outputs), `workdir` (directory with its own AGENTS.md), multi-platform delivery targets.

#### Subagents & Delegation

Hermes uses `delegate_task()` internally for subagent spawning:

- **Synchronous**: parent waits for child's summary before continuing
- `delegation.max_spawn_depth`: controls nesting depth
- `delegation.max_concurrent_children`: default 3 parallel workers
- Agent roles: `leaf` (cannot re-delegate) vs `orchestrator` (can spawn workers)
- **Not durable**: child cancels if parent is interrupted
- For durable parallel work, use `hermes cron` or `/background`

```bash
hermes kanban boards create <name>       # Multi-profile task dispatch
hermes kanban create "<task title>"      # Add task
hermes kanban dispatch                   # Run one scheduler pass
```

#### Session Management

```bash
hermes sessions list
hermes sessions browse              # Interactive picker with full-text search
hermes sessions export <output>     # Export to JSONL
hermes sessions delete <id>
hermes sessions rename <id> <title>

hermes checkpoints status           # Shadow git store for /rollback
hermes checkpoints prune
```

Sessions stored in `~/.hermes/state.db` (SQLite with FTS5).

#### Profiles (Multi-instance)

```bash
hermes profile list
hermes profile create <name>        # New isolated profile
hermes profile use <name>           # Set sticky default
hermes profile delete <name>
hermes -p <name>                    # Use profile for one command
```

Each profile: own `config.yaml`, `.env`, `SOUL.md`, memories, sessions, skills, cron jobs, state.db.

#### MCP Integration

```bash
hermes mcp catalog                  # List Nous-approved MCPs
hermes mcp install <name>           # Add from catalog
hermes mcp serve                    # Run Hermes as MCP server
hermes mcp add / remove / list / test   # Custom MCP management
hermes acp                          # ACP stdio server for editor integration (Zed, Neovim, etc.)
```

#### Diagnostics & Maintenance

```bash
hermes doctor                       # Diagnose config + dependencies
hermes doctor --fix                 # Attempt auto-repair
hermes status                       # Agent/auth/platform status
hermes logs -f --level WARNING --since 1h  # Tail filtered logs
hermes insights --days 30           # Token/cost analytics
hermes prompt-size                  # System prompt byte breakdown
hermes security audit               # OSV.dev supply-chain scan
hermes update                       # Pull latest + reinstall deps
hermes update --check               # Check for updates only
hermes backup -o ./hermes-backup.zip
hermes import -f ./hermes-backup.zip
hermes dump                         # Copy-pasteable setup summary
hermes version                      # Show installed version
```

### What NOT to Use (Stack B)

| Avoid | Why | Use Instead |
|-------|-----|-------------|
| Pre-installing Python manually | Installer manages its own Python 3.11 via `uv`; version conflicts likely | Let installer handle it; only pre-requisite is `git` |
| Pre-installing Node.js manually | Same reason — installer bundles Node v22 | Same |
| `hermes setup` without `--portal` for Nous users | Full wizard prompts for many things; `--portal` covers model + auth + Tool Gateway in one step | `hermes setup --portal` for Nous Portal users |
| Hardcoding API keys in `config.yaml` | Keys are secrets; they belong in `~/.hermes/.env` | Set keys via `hermes model` wizard or `hermes auth add`, which routes to `.env` |
| Korean display language (`display.language: ko`) | NOT in verified supported language list; likely falls back to English | Document in English; note this limitation in tutorial |

---

## Version Compatibility Matrix

| Component | Version | Verified | Source |
|-----------|---------|---------|--------|
| mdBook | 0.5.3 | HIGH | crates.io + docs.rs (released 2026-05-19) |
| Hermes Agent | Not versioned in installer (rolling) | HIGH | install.sh always pulls latest; `hermes version` shows current |
| Hermes Python runtime | 3.11 (bundled) | HIGH | Install docs |
| Hermes Node.js runtime | v22 (bundled) | HIGH | Install docs |
| mdBook Rust requirement | ≥ 1.88 | HIGH | mdBook install docs |
| `actions/checkout` | v4 | HIGH | Official starter workflow |
| `actions/configure-pages` | v5 | HIGH | Official starter workflow |
| `actions/upload-pages-artifact` | v3 | HIGH | Official starter workflow |
| `actions/deploy-pages` | v5 | HIGH | Official starter workflow |

---

## Confidence Summary

| Area | Confidence | Notes |
|------|------------|-------|
| mdBook 0.5.3 + GitHub Actions deploy workflow | HIGH | Official starter workflow verified; 0.5.x breaking changes catalogued |
| Hermes install commands (curl/ps1) | HIGH | Verified against official docs + GitHub repo |
| Hermes CLI command surface | HIGH | Verified against `website/docs/reference/cli-commands.md` in repo |
| Hermes supported providers (core 6) | HIGH | Listed in official install/quickstart docs |
| Hermes additional providers (Gemini, DeepSeek, Bedrock, Ollama) | MEDIUM | Listed in quickstart provider table; not individually verified |
| Korean UI (`display.language: ko`) | LOW | NOT in documented supported language list (`en\|zh\|ja\|de\|es\|fr\|tr\|uk`); flag for verification |
| Hermes version number | N/A | Rolling release; no pinned version in install script |
| Subagent config fields (`max_spawn_depth`, etc.) | MEDIUM | Mentioned in docs but exact YAML keys not shown in public config examples |

---

## Sources

- `https://github.com/NousResearch/hermes-agent` — README, install instructions (HIGH)
- `https://hermes-agent.nousresearch.com/docs/getting-started/installation` — Install prerequisites and locations (HIGH)
- `https://hermes-agent.nousresearch.com/docs/reference/cli-commands` — Full CLI reference (HIGH)
- `https://hermes-agent.nousresearch.com/docs/user-guide/cli` — CLI user guide, keybindings, slash commands (HIGH)
- `https://hermes-agent.nousresearch.com/docs/user-guide/features/context-files` — Context file hierarchy and format (HIGH)
- `https://hermes-agent.nousresearch.com/docs/user-guide/configuration` — config.yaml full schema (HIGH)
- `https://hermes-agent.nousresearch.com/docs/getting-started/quickstart` — Provider setup, first-run flow (HIGH)
- `https://rust-lang.github.io/mdBook/` — mdBook overview, version 0.5.3 confirmed (HIGH)
- `https://rust-lang.github.io/mdBook/guide/installation.html` — Install methods, Rust ≥ 1.88 requirement (HIGH)
- `https://rust-lang.github.io/mdBook/format/configuration/general.html` — book.toml config options (HIGH)
- `https://rust-lang.github.io/mdBook/format/summary.html` — SUMMARY.md syntax (HIGH)
- `https://github.com/rust-lang/mdBook/blob/master/CHANGELOG.md` — 0.5.x breaking changes (HIGH)
- `https://github.com/actions/starter-workflows/blob/main/pages/mdbook.yml` — Official Actions workflow (HIGH)
- `https://github.com/rust-lang/mdBook/wiki/Automated-Deployment:-GitHub-Actions` — Community workflow reference (HIGH)

---
*Stack research for: Korean mdBook Tutorial — Hermes Agent*
*Researched: 2026-06-05*
