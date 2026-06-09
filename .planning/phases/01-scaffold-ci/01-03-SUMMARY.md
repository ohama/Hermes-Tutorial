---
phase: 01-scaffold-ci
plan: 03
subsystem: infra
tags: [github-pages, mdbook, github-actions, ci-cd, deploy, korean]

# Dependency graph
requires:
  - phase: 01-scaffold-ci plan 01
    provides: book.toml, src/SUMMARY.md, Korean chapter scaffold
  - phase: 01-scaffold-ci plan 02
    provides: .github/workflows/deploy.yml GitHub Actions pipeline
provides:
  - Live site at https://ohama.github.io/Hermes-Tutorial/ (Korean mdBook, GitHub Pages)
  - Reconciled book.toml (site-url="/Hermes-Tutorial/", git-repository-url, edit-url-template) for owner ohama/Hermes-Tutorial
  - Verified push-to-main → auto-deploy pipeline end-to-end (build 7s + deploy 10s, both green)
  - Phase 1 all success criteria confirmed
affects:
  - all later phases (every push to main auto-deploys new chapters to the live site)
  - 02+ (live URL = https://ohama.github.io/Hermes-Tutorial/ is the canonical reference for readers)

# Tech tracking
tech-stack:
  added: []
  patterns:
    - GitHub Pages enabled with build_type=workflow (via gh api) BEFORE push — mandatory ordering (Pitfall 3)
    - book.toml site-url must match repo name with leading+trailing slash (e.g. "/Hermes-Tutorial/")
    - Workflow run verification via gh run view to confirm both build and deploy jobs green

key-files:
  created: []
  modified:
    - book.toml

key-decisions:
  - "owner=ohama, repo=Hermes-Tutorial — canonical reference for all later phases"
  - "book.toml site-url set to /Hermes-Tutorial/ (leading + trailing slash); git-repository-url and edit-url-template updated to https://github.com/ohama/Hermes-Tutorial"
  - "GitHub Pages source enabled via gh api (build_type=workflow) BEFORE push — non-negotiable ordering per Pitfall 3"
  - "Node.js 20 deprecation warning from checkout@v4/configure-pages@v5/upload-artifact@v4 is non-blocking; forced to Node 24 starting 2026-06-16 — deferred maintenance item"

patterns-established:
  - "Enable GitHub Pages source (build_type=workflow) via gh api BEFORE first push — prevents missing github-pages environment error"
  - "Verify site-url correctness by confirming hashed CSS loads from subpath (e.g. /Hermes-Tutorial/general-0392ca55.css HTTP 200)"

# Metrics
duration: ~30min (including GitHub human-action steps and live verification)
completed: 2026-06-09
---

# Phase 1 Plan 03: Final Verification and Live Deploy Summary

**mdBook 0.5.3 Korean tutorial deployed live at https://ohama.github.io/Hermes-Tutorial/ via push-to-main GitHub Actions pipeline; book.toml reconciled to ohama/Hermes-Tutorial; all Phase 1 success criteria confirmed**

## Performance

- **Duration:** ~30 min (including human-action checkpoint for GitHub account steps)
- **Started:** 2026-06-09 (Task 1 local build check)
- **Completed:** 2026-06-09 (workflow run 27192158593 green; live site verified)
- **Tasks:** 2 (1 autonomous + 1 human-action checkpoint)
- **Files modified:** 1 (book.toml)

## Accomplishments

- Local `mdbook build` confirmed exit 0 on the full pre-push repo state (scaffold + workflow); book/index.html and book/00-intro/index.html both present
- book.toml reconciled: site-url="/Hermes-Tutorial/", git-repository-url and edit-url-template updated to https://github.com/ohama/Hermes-Tutorial
- GitHub Pages source enabled (build_type=workflow) via gh api BEFORE push — mandatory ordering preserved
- `git push -u origin main` triggered workflow run 27192158593; build job passed (7s), deploy job passed (10s) — both green
- Live site https://ohama.github.io/Hermes-Tutorial/ returns HTTP 200; Korean title "Hermes Agent 한국어 튜토리얼" renders; sidebar TOC shows "소개" / "소개 및 배경"
- Hashed CSS (general-0392ca55.css) served correctly from subpath — confirms site-url "/Hermes-Tutorial/" is correct (per research Pitfall 2 verification method)
- 00-intro/index.html also HTTP 200 — placeholder chapter deployed successfully

## Task Commits

1. **Task 1: Autonomous final local build check** — no commit (book/ is gitignored; no source changes; build is verification only)
2. **Task 2 (human-action): book.toml reconciliation + remote setup + deploy** — `cc2701b` (chore(01): reconcile book.toml site-url to ohama/Hermes-Tutorial)

**Plan metadata:** (docs commit follows — this summary)

## Files Created/Modified

- `book.toml` — site-url="/Hermes-Tutorial/", git-repository-url="https://github.com/ohama/Hermes-Tutorial", edit-url-template updated; committed cc2701b

## Decisions Made

- **owner=ohama, repo=Hermes-Tutorial** — canonical values for all downstream phases and reader-facing URLs
- **GitHub Pages source enabled via `gh api` before push** — avoids missing github-pages environment failure (Pitfall 3 from research)
- **Node.js 20 deprecation warning** (checkout@v4, configure-pages@v5, upload-artifact@v4) treated as non-blocking known concern; actions forced to Node 24 runtime from 2026-06-16; no action required until that date (action versions already pinned)

## Deviations from Plan

None — plan executed exactly as written. The human-action checkpoint was resolved by the orchestrator acting on behalf of the user with their GitHub account, which is the intended flow.

## Authentication Gates

Task 2 was a `checkpoint:human-action` gate requiring GitHub account access:

1. GitHub remote created at https://github.com/ohama/Hermes-Tutorial
2. book.toml reconciled and committed before push
3. GitHub Pages source enabled (build_type=workflow) via gh api
4. `git push -u origin main` succeeded; workflow deployed on first push

This is expected and documented — the plan explicitly marks Task 2 as requiring a human GitHub account.

## Issues Encountered

None. Workflow succeeded on the first push with no retries required. Pitfall 3 ordering (enable Pages before push) was followed correctly.

## User Setup Required

None — the live site is fully deployed and auto-deploys on every push to main. No additional configuration required.

## Known Concerns (Non-Blocking)

- **Node.js 20 deprecation in GitHub Actions:** checkout@v4, configure-pages@v5, and upload-artifact@v4 emit annotation warnings; GitHub will force-migrate to Node 24 runtime on 2026-06-16. No immediate action needed; versions are already pinned and will continue to work post-migration. Future maintenance: update to action versions that natively target Node 24 when available.

## Phase 1 Success Criteria — All Met

| Criterion | Status | Evidence |
|-----------|--------|----------|
| SC-1: Local `mdbook build` produces book/ | PASS | Task 1 exit 0; book/index.html + book/00-intro/index.html present |
| SC-2: git push triggers Actions and auto-deploys | PASS | Workflow run 27192158593; build ✓ 7s + deploy ✓ 10s |
| SC-3: Deployed site reachable with Korean TOC | PASS | https://ohama.github.io/Hermes-Tutorial/ HTTP 200; Korean title + sidebar TOC "소개"/"소개 및 배경" |
| SC-4: # 검증: annotation chapter pattern in place | PASS | 01-01 established this convention; present in placeholder chapter |
| INFRA-02: site-url matches real repo | CLOSED | site-url="/Hermes-Tutorial/", git-repository-url/edit-url-template updated to ohama/Hermes-Tutorial |

## Next Phase Readiness

- Live site operational: https://ohama.github.io/Hermes-Tutorial/
- Every push to main auto-deploys new chapters — Phase 2+ authors can push and see results live
- owner=ohama, repo=Hermes-Tutorial established as canonical reference
- Concern: Node.js 20 deprecation (2026-06-16) is low-urgency maintenance; does not block Phase 2

---
*Phase: 01-scaffold-ci*
*Completed: 2026-06-09*
