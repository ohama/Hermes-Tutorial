---
phase: 04-learning-automation
plan: "04"
subsystem: content
tags: [mdbook, hermes, cron, scheduler, automation, skill-injection, delivery]

# Dependency graph
requires:
  - phase: 04-01
    provides: SUMMARY.md Wave-1 소유 + Ch.12 placeholder (src/12-cron/index.md)
  - phase: 04-01
    provides: Phase 4 검증 주석 관례 (# 검증: hermes rolling, 2026-06-10)
provides:
  - src/12-cron/index.md — Ch.12 크론 스케줄러 본문 (233줄)
affects:
  - 04-05 (Ch.13 서브에이전트 — 크론 개념 선행으로 언급)
  - Phase 5 (게이트웨이 배포/설치 상세화 시 Ch.12 forward-reference 활용 가능)

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "hermes cron CLI 정확성 패턴 — 크론 명령은 hermes cron, gateway는 서비스 명령으로만 기술"
    - "workdir 순차 실행 사이드이펙트 명시 패턴 — 병렬/순차 구분 경고 callout"
    - "격리 세션 경고 패턴 — 새로운 에이전트 세션은 메모리/프로파일 모름 강조"

key-files:
  created: []
  modified:
    - src/12-cron/index.md

key-decisions:
  - "hermes cron run 독립 실행 여부 MEDIUM → > 검증 필요 헤지 적용 (Open Question #4 미해결)"
  - "workdir 순차 실행 사이드이펙트를 주의 callout으로 별도 강조 (RESEARCH.md HIGH confidence)"
  - "격리 세션 경고를 주의 1로 최상위 배치 — 가장 흔한 실수이므로"
  - "Task 2(빌드 검증)는 소스 변경 없음 — 커밋 생략 (03-03 패턴 재적용)"

patterns-established:
  - "Phase 4 no-op 빌드 검증 커밋 생략 패턴 — 소스 변경 없는 검증 task는 커밋하지 않음 (03-03에서 확립)"

# Metrics
duration: 2min
completed: 2026-06-10
---

# Phase 4 Plan 04: Ch.12 크론 스케줄러 Summary

**`hermes cron` CLI 전체 하위 명령(create/list/edit/pause/resume/run/remove/status/tick) + 스케줄 형식·스킬 주입·다중 플랫폼 배달·workdir 순차 실행 함정·격리 세션 경고를 한국어 233줄로 작성, mdbook build exit 0**

## Performance

- **Duration:** ~2 min
- **Started:** 2026-06-10T08:42:48Z
- **Completed:** 2026-06-10T08:44:39Z
- **Tasks:** 2 (Task 1 커밋, Task 2 no-op 검증)
- **Files modified:** 1 (src/12-cron/index.md)

## Accomplishments

- `hermes cron` CLI를 정확히 기술하고 `hermes gateway`를 크론 명령으로 오용하지 않음
- workdir 미설정 함정 + workdir 지정 시 순차 실행 사이드이펙트(프로세스 전역 상태 변경) 명시
- 크론 세션이 완전히 새로운 에이전트 세션임을 경고, 스킬 주입과 다중 플랫폼 배달 커버
- `hermes cron run` gateway 독립 실행 여부를 `> 검증 필요`로 헤지
- 모든 명령/설정 코드블록 `# 검증: hermes rolling, 2026-06-10` 주석 적용
- mdbook build exit 0; src/SUMMARY.md 미수정

## Task Commits

1. **Task 1: Ch.12 크론 스케줄러 본문 작성** - `de0808e` (feat)
2. **Task 2: 빌드 검증 + SUMMARY 불변 확인** - no-op (소스 변경 없음, 커밋 생략)

## Files Created/Modified

- `src/12-cron/index.md` — Ch.12 크론 스케줄러 본문 (233줄): hermes cron CLI·스케줄 형식·스킬 주입·다중 플랫폼 배달·no-agent 모드·config.yaml·workdir 함정·격리 세션 경고

## Decisions Made

- `hermes cron run` 독립 실행 여부는 MEDIUM confidence → `> 검증 필요` callout 두 곳에 명시 (Open Question #4)
- workdir 순차 실행 사이드이펙트를 주의 2에 별도 강조 — 독립적 병렬 실행과의 차이를 명확히
- 격리 세션 경고(주의 1)를 최상위 배치 — 가장 흔한 실수이므로 첫 번째 주의로 처리
- Task 2 빌드 검증은 소스 변경 없음 → 커밋 생략 (03-03 패턴 재적용)

## Deviations from Plan

None — plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None — no external service configuration required.

## Next Phase Readiness

- Ch.12 크론 스케줄러 본문 완료; Wave-2 나머지 플랜(04-02, 04-03, 04-05)과 독립 병렬 완료 가능
- `hermes cron run` gateway 독립 실행 여부 로컬 검증 필요 (비블로킹 — 출판 전 업데이트 권장)

---
*Phase: 04-learning-automation*
*Completed: 2026-06-10*
