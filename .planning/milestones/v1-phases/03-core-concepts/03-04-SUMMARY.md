---
phase: 03-core-concepts
plan: 04
subsystem: content
tags: [hermes, tools, toolsets, gateway, approval-mode, tirith, mdbook]

# Dependency graph
requires:
  - phase: 03-01
    provides: SUMMARY.md with 핵심 개념 section and Ch.7 entry; src/07-tools/index.md placeholder
provides:
  - Ch.7 툴 게이트웨이 full body (~71 tools / 30+ toolsets catalog, hermes tools UI, Tool Gateway paid boundary, use_gateway config, approval mode, Tirith)
affects:
  - future phases referencing Ch.7 as cross-link source
  - 03-05 (Ch.8 터미널 백엔드 — already completed in wave 2)

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Ch.7 uses toolsets array (not agent.disabled_toolsets) for disabling — documented as non-existent key warning"
    - "Free pool quota left as 검증 필요 callout — no invented number"
    - "kanban opt-in exclusion from all/* documented explicitly"

key-files:
  created:
    - src/07-tools/index.md
  modified: []

key-decisions:
  - "agent.disabled_toolsets key documented only as non-existent (주의 3 callout) — never as real config"
  - "Nous Tool Gateway free pool quota flagged 검증 필요 per Open Question #2 — no specific number asserted"
  - "kanban toolset opt-in exclusion from all/* explicitly called out in the registry table note"
  - "use_gateway: true overrides direct API key — documented as override priority"
  - "Tirith fail_open: true default documented — scanner failure allows execution"

patterns-established:
  - "All code/config blocks open with # 검증: hermes rolling, 2026-06-09"
  - "LOW/MEDIUM confidence outputs use [로컬 실행 후 캡처 필요] + > 검증 필요 callout"
  - "Non-existent config keys documented only in 주의 callout with explicit 없음/없다 language"

# Metrics
duration: 2min
completed: 2026-06-10
---

# Phase 3 Plan 04: 툴 게이트웨이 Summary

**Ch.7 툴 게이트웨이 본문 — ~71개 툴/30+ 툴셋 카탈로그, hermes tools UI, Nous Tool Gateway 유료 경계, use_gateway per-tool 설정, 승인 모드(manual/smart/off), Tirith 스캔 — agent.disabled_toolsets 존재하지 않음 명시, 무료 풀 수치 검증 필요 callout**

## Performance

- **Duration:** ~2 min
- **Started:** 2026-06-10T07:53:29Z
- **Completed:** 2026-06-10T07:55:40Z
- **Tasks:** 2 (Task 1: 본문 작성 / Task 2: 빌드 검증)
- **Files modified:** 1

## Accomplishments

- Ch.7 full body written (284 lines) with all required sections from plan
- `agent.disabled_toolsets` correctly documented as non-existent key (주의 3 callout only)
- Nous Tool Gateway free pool quota left as `검증 필요` (Open Question #2 — no invented number)
- `kanban` toolset opt-in exclusion from `all`/`*` wildcard documented
- `mdbook build` exit 0, no errors, `src/SUMMARY.md` untouched

## Task Commits

Each task was committed atomically:

1. **Task 1: Ch.7 툴 게이트웨이 본문 작성** - `9eba0bb` (feat)
2. **Task 2: 빌드 검증** - (verification-only, no file change — covered by Task 1 commit)

**Plan metadata:** (this SUMMARY commit)

## Files Created/Modified

- `src/07-tools/index.md` — Ch.7 툴 게이트웨이 전체 본문 (284줄); 툴 카탈로그 표 / hermes tools UI / 커스텀 툴셋 / use_gateway / FAL.ai 9 모델 표 / 승인 모드 표 / Tirith config / 3개 주의 callout

## Decisions Made

- `agent.disabled_toolsets`를 동작하는 config 키로 절대 문서화하지 않음 — 주의 3 callout에서만 "존재하지 않는 키" 문맥으로 언급 (RESEARCH.md correction #8 준수)
- 무료 툴 풀 수치 단정 안 함 — `> 검증 필요` callout + Nous Portal 가격 페이지 확인 안내 (RESEARCH.md Open Question #2 준수)
- 세션 내 슬래시 커맨드(`/tools list` 등)는 bash 코드 펜스가 아닌 일반 코드 펜스로 처리 (실행 명령이 아님)
- `# 검증: hermes rolling, 2026-06-09` 주석을 모든 bash/yaml 코드블록 첫 줄에 적용

## Deviations from Plan

None — plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None — content-only chapter, no external service configuration required.

## Next Phase Readiness

- Ch.7 완료 — Wave 2 (03-02 ~ 03-05) 모두 완료되면 Phase 3 전체 준비됨
- `src/08-backends/index.md` (Ch.8) 는 03-05에서 이미 완료됨
- Phase 3 완료 후 Phase 4 (스킬 시스템) 진행 가능

---
*Phase: 03-core-concepts*
*Completed: 2026-06-10*
