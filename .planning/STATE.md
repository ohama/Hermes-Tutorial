# Project State

## Project Reference

See: .planning/PROJECT.md (updated 2026-06-05)

**Core value:** 개발자가 따라 하면 Hermes Agent를 설치·실행하고, 스킬·게이트웨이·배포까지 실제로 동작시킬 수 있다. 모든 명령어·API는 공식 소스 근거 필수.
**Current focus:** Phase 1 — Scaffold & CI

## Current Position

Phase: 1 of 6 (Scaffold & CI) — **COMPLETE**
Plan: 3 of 3 in current phase — **ALL DONE**
Status: Phase 1 complete
Last activity: 2026-06-09 — Completed 01-03-PLAN.md (final verification + live deploy to GitHub Pages)

Progress: [███░░░░░░░] ~17% (3/~18 plans estimated)

## Performance Metrics

**Velocity:**
- Total plans completed: 1
- Average duration: 2 min
- Total execution time: ~2 min

**By Phase:**

| Phase | Plans | Total | Avg/Plan |
|-------|-------|-------|----------|
| 01-scaffold-ci | 3/3 (DONE) | ~33 min | ~11 min |

**Recent Trend:**
- Last 5 plans: 01-01 (2 min), 01-02 (1 min), 01-03 (~30 min incl. human-action GitHub steps)
- Trend: 01-03 was longer due to human-action checkpoint (GitHub account + live deploy verification)

*Updated after each plan completion*

## Accumulated Context

### Decisions

Decisions are logged in PROJECT.md Key Decisions table.
Recent decisions affecting current work:

- Init: mdBook 0.5.3 + `actions/deploy-pages@v5` 공식 스타터 워크플로우 사용 (pre-built 바이너리, cargo install 금지)
- Init: 한국어 UI (`display.language: ko`) 미지원 확인 — 영어 UI 사용 명시, 해당 내용 Ch.3에 포함
- Init: 코드블록 버전·검토일 주석 관례 (`# 검증: hermes vX.Y, YYYY-MM-DD`) Phase 1부터 적용
- 01-01: mdBook 0.5.3 was already on PATH (pre-built binary); installation step skipped per plan's skip-if-present instruction
- 01-01: `<owner>` placeholders left in git-repository-url/edit-url-template — plan 01-03 reconciles once remote exists
- 01-01: book.toml must NOT include multilingual/curly-quotes/copy-fonts/google-analytics/smart-punctuation/hash-files (removed in 0.5.0, cause hard errors)
- 01-02: deploy.yml uses pre-built binary install (not cargo install); pins deploy-pages@v5; permissions pages:write + id-token:write required by deploy-pages
- 01-02: YAML validation via PyYAML (python3 yaml.safe_load) — confirmed available on system
- 01-03: owner=ohama, repo=Hermes-Tutorial; remote=https://github.com/ohama/Hermes-Tutorial; live URL=https://ohama.github.io/Hermes-Tutorial/
- 01-03: book.toml site-url="/Hermes-Tutorial/", git-repository-url/edit-url-template reconciled to ohama/Hermes-Tutorial (commit cc2701b)
- 01-03: GitHub Pages source must be enabled (build_type=workflow via gh api) BEFORE first push — ordering non-negotiable (Pitfall 3)
- 01-03: Workflow run 27192158593 green (build 7s + deploy 10s); live site HTTP 200, Korean TOC renders

### Pending Todos

None yet.

### Blockers/Concerns

- **[Node.js 20 deprecation — LOW, 2026-06-16]** GitHub Actions annotations warn that checkout@v4, configure-pages@v5, and upload-artifact@v4 use Node.js 20 (deprecated); GitHub forces Node 24 runtime from 2026-06-16. Non-blocking — actions continue to work post-migration. Future maintenance: update to action versions natively targeting Node 24 when available. Does NOT block Phase 2.
- Phase 4: Honcho integration 세부 사항 (외부 계정 필요 여부, 정확한 config 키) MEDIUM confidence — Ch.10 작성 전 별도 검증 필요
- Phase 5: Discord OAuth + Privileged Gateway Intents 흐름 2026-03 이후 변경 가능 — Discord 서브챕터 작성 전 검증 필요
- Phase 5: Modal/Daytona 백엔드 YAML 키 공개 문서 미확인 — serverless 배포 섹션 작성 전 검증 필요

## Session Continuity

Last session: 2026-06-09 (Plan 01-03 executed; book.toml reconciled cc2701b; live deploy verified — workflow run 27192158593 green; https://ohama.github.io/Hermes-Tutorial/ HTTP 200)
Stopped at: Plan 01-03 complete — Phase 1 ALL 3/3 plans done; site live and auto-deploying
Resume file: None
Next workflow trigger: `/gsd:plan-phase 2` (begin Phase 2 — first content chapters)
