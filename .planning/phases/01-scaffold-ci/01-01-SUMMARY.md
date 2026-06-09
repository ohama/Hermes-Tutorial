---
phase: 01-scaffold-ci
plan: 01
subsystem: infra
tags: [mdbook, book.toml, korean, gitignore, scaffold]

# Dependency graph
requires: []
provides:
  - mdBook 0.5.3 buildable project at repo root
  - book.toml with language="ko" and site-url="/hermes-tutorial/"
  - Korean SUMMARY.md navigation index
  - src/README.md book intro page (renders as index.html)
  - src/00-intro/index.md placeholder chapter with # 검증: annotation convention
  - .gitignore excluding book/ build output
affects:
  - 01-scaffold-ci/01-02 (GitHub Actions CI/CD pipeline depends on this scaffold)
  - 01-scaffold-ci/01-03 (remote URL reconciliation updates book.toml git-repository-url/edit-url-template)
  - all content phases (02-06) build chapters on top of this src/ structure

# Tech tracking
tech-stack:
  added: [mdBook 0.5.3]
  patterns:
    - "# 검증: <component> <version>, YYYY-MM-DD code-block annotation convention"
    - "Only verified 0.5.x book.toml keys — unknown fields are hard errors in 0.5.x"

key-files:
  created:
    - book.toml
    - .gitignore
    - src/SUMMARY.md
    - src/README.md
    - src/00-intro/index.md
  modified: []

key-decisions:
  - "mdBook 0.5.3 pre-built binary used (already on PATH); cargo install not needed"
  - "<owner> placeholders left in git-repository-url and edit-url-template — reconciled in plan 01-03"
  - "book/ excluded via .gitignore (build output not committed)"

patterns-established:
  - "code-block annotation: every code block with command/config/API call opens with # 검증: <component> <version>, YYYY-MM-DD"
  - "book.toml: no multilingual/curly-quotes/copy-fonts/google-analytics/smart-punctuation/hash-files — all removed in 0.5.0"

# Metrics
duration: 2min
completed: 2026-06-09
---

# Phase 1 Plan 1: mdBook Scaffold Summary

**mdBook 0.5.3 project initialized with book.toml (language="ko", site-url="/hermes-tutorial/"), Korean SUMMARY.md, book intro, and placeholder chapter demonstrating the `# 검증:` code-block annotation convention — builds to book/index.html with zero errors**

## Performance

- **Duration:** 2 min
- **Started:** 2026-06-09T07:47:09Z
- **Completed:** 2026-06-09T07:48:47Z
- **Tasks:** 3
- **Files modified:** 5

## Accomplishments

- book.toml created with all verified 0.5.x keys; no forbidden removed keys (multilingual/curly-quotes/copy-fonts/google-analytics)
- Korean src/SUMMARY.md + src/README.md intro + src/00-intro/index.md placeholder chapter with 4 `# 검증:` annotated code blocks
- `mdbook build` exits 0, produces book/index.html, no unknown/error lines in output

## Task Commits

Each task was committed atomically:

1. **Task 1: Write book.toml and .gitignore** - `e1916e8` (chore)
2. **Task 2: Write Korean SUMMARY.md, README intro, and placeholder chapter** - `6f4dd44` (feat)
3. **Task 3: Install mdBook 0.5.3 locally and verify the build** - `cce217f` (chore)

**Plan metadata:** (docs: complete plan — committed after SUMMARY.md creation)

## Files Created/Modified

- `book.toml` - mdBook 0.5.3 config: language="ko", site-url="/hermes-tutorial/", search enabled; <owner> placeholders for git-repository-url/edit-url-template
- `.gitignore` - excludes book/ build output
- `src/SUMMARY.md` - Korean navigation index: [소개](README.md) + 기초 section with 00-intro/index.md
- `src/README.md` - Book intro page (renders as book/index.html)
- `src/00-intro/index.md` - Placeholder chapter with 4 `# 검증:` annotated code blocks demonstrating convention

## Decisions Made

- mdBook 0.5.3 was already on PATH (pre-built binary); installation step skipped per plan's skip-if-present instruction
- `<owner>` placeholders left in git-repository-url and edit-url-template per plan constraint; plan 01-03 reconciles these once a remote exists
- book/ excluded via .gitignore; confirmed not committed after build

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness

- Buildable mdBook 0.5.3 scaffold complete; INFRA-01, INFRA-02, INFRA-04 satisfied
- Plan 01-02 can now add GitHub Actions CI/CD workflow targeting this scaffold
- Plan 01-03 can reconcile <owner> placeholders in book.toml once remote is established

---
*Phase: 01-scaffold-ci*
*Completed: 2026-06-09*
