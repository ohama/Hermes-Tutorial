# Project State

## Project Reference

See: .planning/PROJECT.md (updated 2026-06-05)

**Core value:** 개발자가 따라 하면 Hermes Agent를 설치·실행하고, 스킬·게이트웨이·배포까지 실제로 동작시킬 수 있다. 모든 명령어·API는 공식 소스 근거 필수.
**Current focus:** Phase 1 — Scaffold & CI

## Current Position

Phase: 1 of 6 (Scaffold & CI)
Plan: 2 of 3 in current phase
Status: In progress
Last activity: 2026-06-09 — Completed 01-02-PLAN.md (GitHub Actions deploy.yml CI/CD pipeline)

Progress: [██░░░░░░░░] ~11% (2/~18 plans estimated)

## Performance Metrics

**Velocity:**
- Total plans completed: 1
- Average duration: 2 min
- Total execution time: ~2 min

**By Phase:**

| Phase | Plans | Total | Avg/Plan |
|-------|-------|-------|----------|
| 01-scaffold-ci | 2/3 | ~3 min | 1.5 min |

**Recent Trend:**
- Last 5 plans: 01-01 (2 min), 01-02 (1 min)
- Trend: fast (no external deps, pure file write + validation)

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

### Pending Todos

None yet.

### Blockers/Concerns

- Phase 4: Honcho integration 세부 사항 (외부 계정 필요 여부, 정확한 config 키) MEDIUM confidence — Ch.10 작성 전 별도 검증 필요
- Phase 5: Discord OAuth + Privileged Gateway Intents 흐름 2026-03 이후 변경 가능 — Discord 서브챕터 작성 전 검증 필요
- Phase 5: Modal/Daytona 백엔드 YAML 키 공개 문서 미확인 — serverless 배포 섹션 작성 전 검증 필요

## Session Continuity

Last session: 2026-06-09 (Plan 01-02 executed by gsd-executor agent; deploy.yml written + YAML validated at 07:51:20Z)
Stopped at: Plan 01-02 complete — 2/3 plans in Phase 1 done; deploy.yml committed at 658d01c
Resume file: None
Next workflow trigger: Execute 01-03-PLAN.md (push to GitHub remote + enable GitHub Pages)
