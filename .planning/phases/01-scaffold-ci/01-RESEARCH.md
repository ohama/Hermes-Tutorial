# Phase 1: Scaffold & CI — Research

**Researched:** 2026-06-09
**Domain:** mdBook 0.5.3 project structure, book.toml config, GitHub Pages CI/CD via GitHub Actions
**Confidence:** HIGH overall (all critical items verified against official sources)

---

## Summary

This research resolves the concrete, plannable specifics for Phase 1: standing up a buildable
mdBook 0.5.3 project and a GitHub Actions pipeline that auto-deploys to GitHub Pages. Prior
research (STACK.md, ARCHITECTURE.md, PITFALLS.md) already established the correct tool
versions and the high-level approach; this document fills in the exact key names, verified YAML,
and the sequencing constraints that affect planning tasks.

The standard approach is: write `book.toml` with the verified 0.5.x-compatible keys, create a
minimal `src/` tree with a Korean `SUMMARY.md` and one placeholder chapter demonstrating the
`# 검증:` annotation convention, commit `.github/workflows/deploy.yml` using the official
starter workflow (updated to pin mdBook 0.5.3 and use binary install), then create the GitHub
remote, push, and enable Pages → Source = "GitHub Actions" in repo settings.

**Primary recommendation:** Use the pre-built binary install method (not `cargo install`) with
`MDBOOK_VERSION: 0.5.3` pinned in the workflow env, and set `site-url = "/hermes-tutorial/"` as
a parameterized placeholder in `book.toml` — document that the planner must substitute the real
repo name when the remote is created.

---

## Standard Stack

### Core

| Tool | Version | Purpose | Why Standard |
|------|---------|---------|--------------|
| mdBook | 0.5.3 | Static book generator | Current stable (2026-05-19); 0.5.x is the actively maintained branch |
| GitHub Actions | platform | CI build + deploy | Official starter workflow exists; zero hosting cost |
| `actions/checkout` | v4 | Checkout repo in CI | Current stable major version |
| `actions/configure-pages` | v5 | Inject base URL / Pages metadata | Required step before build; auto-injects `site-url` if not set |
| `actions/upload-pages-artifact` | v3 | Package `book/` as Pages artifact | Required intermediary before deploy-pages |
| `actions/deploy-pages` | v5 | Deploy artifact to Pages | Modern approach — no `gh-pages` branch needed |

**Note on action versions:** The official starter workflow at `actions/starter-workflows` pins
`configure-pages@v5`, `upload-pages-artifact@v3`, `deploy-pages@v5`. These are confirmed
current as of research date. The starter workflow still pins **mdBook 0.4.36** (outdated) —
the plan must override this with 0.5.3.

### Supporting

| Tool | Purpose | Notes |
|------|---------|-------|
| Pre-built binary tarball | Faster CI install | Saves ~2-3 min vs. `cargo install`; verified pattern: `mdbook-v0.5.3-x86_64-unknown-linux-gnu.tar.gz` |
| `mdbook build` | Build `src/` → `book/` | Default output dir is `book/`; no flag needed |
| `mdbook serve` | Local dev, live-reload | Port 3000 by default |
| `.gitignore` entry for `book/` | Exclude build output from git | Standard practice; do not commit built output |

### Alternatives Considered

| Recommended | Alternative | Why Not |
|-------------|-------------|---------|
| Pre-built binary download | `cargo install --version 0.5.3 mdbook` | Compiling from source adds 2-3 min CI time; Rust toolchain not needed |
| `actions/deploy-pages@v5` | `peaceiris/actions-gh-pages` | Official action; no extra token config; pages branch approach is legacy |
| `configure-pages@v5` | Skip configure-pages step | `configure-pages` is what creates the `github-pages` environment and outputs `base_path` |

### Installation (local dev)

```bash
# macOS / Linux — pre-built binary (matches CI)
curl -sSL https://github.com/rust-lang/mdBook/releases/download/v0.5.3/mdbook-v0.5.3-x86_64-unknown-linux-gnu.tar.gz | tar -xz
# or macOS arm64:
curl -sSL https://github.com/rust-lang/mdBook/releases/download/v0.5.3/mdbook-v0.5.3-aarch64-apple-darwin.tar.gz | tar -xz
# Move binary to PATH, then:
mdbook build   # outputs to ./book/
mdbook serve   # http://localhost:3000
```

---

## Architecture Patterns

### Recommended Project Structure

```
hermes-tutorial/               # git repo root
├── book.toml                  # mdBook configuration (verified keys below)
├── src/
│   ├── SUMMARY.md             # Navigation index — drives ALL chapter pages
│   ├── README.md              # Book intro → renders as index.html
│   └── 00-intro/
│       └── index.md           # Placeholder chapter with # 검증: annotation
├── .github/
│   └── workflows/
│       └── deploy.yml         # GitHub Actions CI/CD (exact YAML below)
├── .gitignore                 # Must include: book/
└── theme/                     # Optional: add only if custom CSS needed
    └── custom.css
```

### Pattern 1: Verified book.toml for mdBook 0.5.3

**What:** Exact, verified config keys for Korean mdBook. All keys confirmed against official 0.5.x docs.

```toml
[book]
title = "Hermes Agent 한국어 튜토리얼"
authors = ["Hermes Tutorial Contributors"]
description = "Nous Research Hermes Agent 한국어 튜토리얼"
language = "ko"
src = "src"
# NOTE: "multilingual" key was REMOVED in 0.5.0 — do NOT include it

[output.html]
# site-url MUST match the GitHub repo path: /<repo-name>/
# Parameterize this — substitute actual repo name when remote is created
site-url = "/hermes-tutorial/"
git-repository-url = "https://github.com/<owner>/hermes-tutorial"
edit-url-template = "https://github.com/<owner>/hermes-tutorial/edit/main/{path}"
default-theme = "light"
preferred-dark-theme = "navy"
# smart-punctuation defaults to true in 0.5.x — no need to set explicitly
# hash-files defaults to true in 0.5.x — no need to set explicitly

[output.html.search]
enable = true
limit-results = 20
```

**Keys verified present in 0.5.x:** `language`, `src`, `site-url`, `git-repository-url`,
`edit-url-template`, `default-theme`, `preferred-dark-theme`, `smart-punctuation` (default true),
`hash-files` (default true).

**Do NOT include these — they cause errors in 0.5.x:**
- `multilingual` — removed in 0.5.0
- `curly-quotes` — removed in 0.5.0 (use `smart-punctuation` instead)
- `copy-fonts` — removed in 0.5.0

**Unknown fields are now errors in 0.5.0+** — any typo or stale key will break the build.

### Pattern 2: Minimal SUMMARY.md for Phase 1 scaffold

```markdown
# Summary

[소개](README.md)

---

# 기초

- [소개 및 배경](00-intro/index.md)
```

Rules enforced by mdBook:
- Only `#` (h1) headers function as part titles; h2+ are ignored
- Use `-` or `*` for chapters (do not mix within the file)
- Duplicate file path entries cause a build error — run `mdbook build` locally before push
- Draft chapters use empty parens: `- [Draft]()`
- Separators use `---`

### Pattern 3: Placeholder chapter with # 검증: annotation

**File:** `src/00-intro/index.md`

```markdown
# 소개 및 배경

Hermes Agent 한국어 튜토리얼에 오신 것을 환영합니다.

이 튜토리얼은 Nous Research의 Hermes Agent를 한국어로 소개합니다.

## Hermes Agent란?

Hermes Agent는 Nous Research가 개발한 오픈소스 AI 에이전트입니다.

설치 예시:

```bash
# 검증: hermes rolling (설치 시점 최신), 2026-06-09
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

설치 후 버전을 확인합니다:

```bash
# 검증: hermes rolling, 2026-06-09
hermes version
```
```

**Annotation convention:** Every code block that contains a command, config key, or API call
must open with a comment line in the format:
```
# 검증: hermes <version>, YYYY-MM-DD
```
For mdBook/infra commands, use the component name and version (e.g., `# 검증: mdBook 0.5.3, 2026-06-09`).

### Pattern 4: GitHub Actions deploy.yml (verified for 0.5.3)

**File:** `.github/workflows/deploy.yml`

```yaml
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

      - name: Install mdBook (pre-built binary)
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

**Differences from the official starter workflow** (which this plan must apply):
1. Starter workflow pins mdBook `0.4.36` via `cargo install` — change to pre-built binary + `0.5.3`
2. Starter workflow uses `actions/configure-pages@v5` ✓, `upload-pages-artifact@v3` ✓, `deploy-pages@v5` ✓ — keep these
3. Starter workflow uses `$default-branch` placeholder — replace with `main`

### Anti-Patterns to Avoid

- **Using `cargo install mdbook` in CI:** Compiling takes 2-3 min; use pre-built binary
- **Pinning 0.4.36 from starter workflow:** Has removed features and deprecated keys; use 0.5.3
- **Including `multilingual = false` in book.toml:** Causes 0.5.x build error (unknown field)
- **Omitting `site-url`:** CSS/JS and 404 page will 404 on `/<repo>/` sub-path
- **Duplicate SUMMARY.md entries:** Silent build break; always run `mdbook build` locally first

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Binary install in CI | Custom shell script | Official release URL pattern (verified) | Pattern is stable; see exact URL above |
| Pages deployment | Push to `gh-pages` branch | `actions/deploy-pages@v5` | Branch approach is deprecated; requires extra token config |
| Search | Custom JS | mdBook built-in `[output.html.search]` | Built-in, zero config needed; enable with `enable = true` |
| Dark mode | Custom CSS | `preferred-dark-theme = "navy"` | Built-in; triggers via browser `prefers-color-scheme` |

---

## Common Pitfalls

### Pitfall 1: Official starter workflow uses outdated mdBook version

**What goes wrong:** Copy-pasting the GitHub Pages starter workflow directly gives mdBook
0.4.36. This version does not have admonitions, definition lists, or sidebar header nav.
Worse: if book.toml uses 0.5.x-only features or omits deprecated keys, the 0.4.36 build
silently uses different behavior or fails.

**How to avoid:** Override `MDBOOK_VERSION: 0.5.3` in the workflow env and use binary
install, not `cargo install`.

**Warning signs:** CI installs Rust toolchain (visible in job log) instead of downloading tarball.

---

### Pitfall 2: site-url mismatch with actual repo name

**What goes wrong:** If `site-url = "/hermes-tutorial/"` but the GitHub repo is named
`hermes-agent-tutorial`, deployed CSS/JS and the 404 page will 404.

**How to avoid:** The planner must document this as a parameterized placeholder. The plan task
that creates the GitHub remote and sets Pages source must also update `site-url` in `book.toml`
to match the actual repo name, then commit and push.

**Verification:** After first deploy, visit `https://<owner>.github.io/<repo>/404.html` — if
CSS loads, `site-url` is correct.

---

### Pitfall 3: GitHub Pages source must be set BEFORE first push triggers the workflow

**What goes wrong:** If you push the workflow file before enabling "GitHub Actions" as the Pages
source in repo settings, the workflow runs but the `deploy-pages` step fails because the
`github-pages` environment does not exist yet.

**How to avoid:** The sequence is: (1) create GitHub remote repo, (2) enable Pages →
Source = "GitHub Actions" in repo settings, (3) push `main` branch. The `github-pages`
environment is auto-created on first successful deploy, not by the settings change alone.

**Planner note:** This sequencing constraint must appear as an explicit ordered task dependency
in PLAN.md. The repo settings step cannot be automated in the workflow itself.

---

### Pitfall 4: Unknown fields in book.toml cause hard errors in 0.5.x

**What goes wrong:** Any typo or stale key (e.g., `multilingual`, `curly-quotes`, `copy-fonts`,
`google-analytics`) causes `mdbook build` to exit with an error in 0.5.0+.

**How to avoid:** Only include keys from the verified list above. Run `mdbook build` locally
before pushing to catch errors immediately.

**Warning signs:** Error message starts with "unknown field" or "unexpected key".

---

### Pitfall 5: SUMMARY.md duplicate entries

**What goes wrong:** If the same file path appears twice in SUMMARY.md (common during
chapter reorganization), mdBook fails with a build error. The error message is not always
obvious.

**How to avoid:** After any SUMMARY.md edit, run `mdbook build` locally. In Phase 1, the
scaffold is minimal so this is low risk — but the CI pipeline will catch it on every push.

---

## Code Examples

### Verified: Binary install URL pattern for mdBook v0.5.3

```bash
# Source: GitHub API verified — https://api.github.com/repos/rust-lang/mdbook/releases/tags/v0.5.3
# Linux x86_64 (used in CI):
https://github.com/rust-lang/mdBook/releases/download/v0.5.3/mdbook-v0.5.3-x86_64-unknown-linux-gnu.tar.gz

# macOS arm64 (M1/M2/M3 local dev):
https://github.com/rust-lang/mdBook/releases/download/v0.5.3/mdbook-v0.5.3-aarch64-apple-darwin.tar.gz

# macOS x86_64:
https://github.com/rust-lang/mdBook/releases/download/v0.5.3/mdbook-v0.5.3-x86_64-apple-darwin.tar.gz
```

### Verified: Minimum local build test

```bash
# 검증: mdBook 0.5.3, 2026-06-09
mdbook build        # builds to ./book/
mdbook serve        # serves at http://localhost:3000 with live-reload
```

### Verified: Confirm no unknown fields in book.toml

```bash
# Fails loudly with "unknown field" if any deprecated key is present
mdbook build 2>&1 | grep -i "unknown\|error"
```

---

## State of the Art

| Old Approach | Current Approach | Changed | Impact |
|--------------|------------------|---------|--------|
| `curly-quotes = true` | `smart-punctuation = true` (default) | mdBook 0.5.0 | Old key is an error; new key is default — no need to set |
| `copy-fonts = true` | Automatic (no key) | mdBook 0.5.0 | Old key is an error; fonts always copied |
| `multilingual = false` | Removed entirely | mdBook 0.5.0 | Including it causes a hard error |
| `cargo install --version 0.4.36` in CI | Binary download, pin 0.5.3 | mdBook 0.5.x era | Starter workflow is stale; override required |
| `actions/deploy-pages@v1` | `@v5` | GitHub Actions 2024-2025 | v1/v2 deprecated; will error |
| `gh-pages` branch deploy | `actions/deploy-pages@v5` (artifact) | GitHub Pages 2023+ | Branch approach requires extra token; artifact approach is official |

**Deprecated/outdated:**
- `peaceiris/actions-mdbook`: Pinned to old mdBook versions; not maintained for 0.5.x
- `peaceiris/actions-gh-pages`: Still works but legacy; requires `GITHUB_TOKEN` write access to branch
- `actions/upload-artifact` (generic): Use `actions/upload-pages-artifact` specifically for Pages

---

## Open Questions

1. **Owner/repo name not yet fixed**
   - What we know: Repo dir is `hermes-tutorial`; no remote exists yet; `site-url = "/hermes-tutorial/"` is the expected value
   - What's unclear: Will the GitHub remote use the same name? GitHub username is unknown
   - Recommendation: Planner documents `site-url` and `git-repository-url` as placeholders with `<owner>` and `<repo>` markers. The task that creates the remote must include a follow-up step to update book.toml with actual values, commit, and push.

2. **configure-pages vs. manual site-url**
   - What we know: `actions/configure-pages` can inject `site-url` automatically based on Pages settings
   - What's unclear: If `site-url` is set manually in book.toml AND `configure-pages` runs, which wins? Does auto-injection override?
   - Recommendation (LOW confidence): Set `site-url` explicitly in book.toml for local `mdbook serve` correctness; the auto-injection from `configure-pages` should be additive or consistent. If there's a conflict, explicit book.toml wins for local dev. Flag for verification in the plan's test task.

3. **`contents: read` permission in workflow**
   - What we know: `pages: write` and `id-token: write` are confirmed required by `actions/deploy-pages` docs
   - What's unclear: `contents: read` is included in STACK.md's YAML but not listed as required by deploy-pages docs
   - Recommendation: Keep `contents: read` (it's needed for `actions/checkout` which runs in the build job). It's harmless if unnecessary and required for checkout.

4. **Korean language rendering in browsers**
   - What we know: `language = "ko"` in `[book]` sets the HTML `lang` attribute correctly
   - What's unclear: Does mdBook 0.5.3 ship Korean UI strings for navigation labels ("Next", "Previous", etc.)?
   - Recommendation (MEDIUM confidence): The `lang` attribute is set correctly for screen readers and browser translate. Navigation button text localization depends on mdBook's i18n support (which is minimal — only `lang` attribute, not translated strings). Plan should not promise translated UI labels.

---

## Sources

### Primary (HIGH confidence — official sources, directly fetched)

- `https://rust-lang.github.io/mdBook/format/configuration/general.html` — `[book]` config keys
- `https://rust-lang.github.io/mdBook/format/configuration/renderers.html` — `[output.html]` config keys (all keys verified)
- `https://rust-lang.github.io/mdBook/format/summary.html` — SUMMARY.md syntax rules
- `https://raw.githubusercontent.com/rust-lang/mdBook/master/CHANGELOG.md` — 0.5.0 breaking changes (removed keys: `curly-quotes`, `copy-fonts`, `multilingual`, Google Analytics; unknown fields now errors)
- `https://github.com/actions/starter-workflows/blob/main/pages/mdbook.yml` — Official Actions workflow (confirms action versions: checkout@v4, configure-pages@v5, upload-pages-artifact@v3, deploy-pages@v5)
- `https://github.com/actions/deploy-pages` — Confirmed required permissions: `pages: write`, `id-token: write`; current major version is v5
- GitHub API `https://api.github.com/repos/rust-lang/mdbook/releases/tags/v0.5.3` — Verified binary filenames and download URLs

### Secondary (MEDIUM confidence — GitHub search results + community confirmation)

- `book.multilingual` removal confirmed in 0.5.0 via search + changelog
- GitHub Pages sequencing (set source before push) confirmed via multiple community discussions

---

## Metadata

**Confidence breakdown:**
- Standard stack (versions, action names): HIGH — fetched from official starter workflow and GitHub API
- book.toml key names: HIGH — fetched directly from official mdBook 0.5.x docs
- 0.5.0 breaking changes (removed keys): HIGH — fetched from CHANGELOG.md
- GitHub Pages permissions: HIGH — fetched from deploy-pages repo README
- GitHub Pages sequencing (set source before push): MEDIUM — confirmed via community sources, not official docs
- configure-pages auto-injection behavior: LOW — not tested; see Open Question #2
- Korean UI labels in mdBook: MEDIUM — lang attribute confirmed; translated strings unconfirmed

**Research date:** 2026-06-09
**Valid until:** 2026-09-09 (mdBook is stable; GitHub Actions versions change slowly)
