---
phase: 04-learning-automation
plan: 05
subsystem: content
tags: [hermes, subagents, delegate_task, kanban, multi-agent, parallel, sqlite, mdbook]

# Dependency graph
requires:
  - phase: 04-01
    provides: SUMMARY.md Wave-1 owner, Ch.10~13 스켈레톤 파일 생성
provides:
  - Ch.13 서브에이전트 본문 (delegate_task + 동시성/격리/leaf 제약 + Kanban 내구 시스템 + 비교표)
affects: [05-deployment-gateways, future-phases-referencing-subagents]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Kanban 내구 시스템 기술 패턴: SQLite+CLI+dashboard+dispatcher 전체 기술, 단순 toolset으로 축소 금지"
    - "Phase 5 전방 참조 = plain prose only (파일 미존재 링크 금지)"
    - "에이전트 내부 툴 설명 코드블록은 검증 주석 예외 (실행 명령 아님)"

key-files:
  created: []
  modified:
    - src/13-subagents/index.md

key-decisions:
  - "delegate_task Python 시그니처 코드블록은 실행 명령이 아닌 설명용임을 주석으로 명시 — '# 에이전트 내부 사용 — 독자는 자연어로 위임 요청'"
  - "Task 2(빌드 검증)는 파일 변경 없음 — no-op 커밋 생략, mdbook build exit 0 확인으로 완료"
  - "Kanban을 단순 opt-in toolset이 아닌 SQLite+CLI+dashboard+dispatcher 내구 시스템으로 명확히 기술"
  - "Phase 5 전방 참조는 plain prose — 존재하지 않는 14+ 챕터 파일 링크 없음"

patterns-established:
  - "에이전트 내부 호출 툴 설명: '독자는 자연어로 위임 요청' 패턴 (Phase 3 toolset 설명과 일관)"
  - "다중 시스템 비교표 패턴: Kanban vs delegate_task 표 형식 (용도/실행모델/지속성/인간참여/재개)"

# Metrics
duration: 2min
completed: 2026-06-10
---

# Phase 4 Plan 05: Ch.13 서브에이전트 Summary

**`delegate_task` 병렬 추론 위임(격리/동시성 3개/leaf 제약)과 Kanban 내구 SQLite 다중 에이전트 시스템(kanban.db + CLI + hermes dashboard + hermes gateway start) 본문, 선택 기준 비교표 포함**

## Performance

- **Duration:** 2min
- **Started:** 2026-06-10T08:43:07Z
- **Completed:** 2026-06-10T08:45:31Z
- **Tasks:** 2
- **Files modified:** 1

## Accomplishments

- Ch.13 서브에이전트 본문 303줄 작성 (placeholder 1줄 → 완전한 챕터)
- delegate_task 설명: 컨텍스트 격리, 동시성(기본 3개), ThreadPoolExecutor, max_spawn_depth, leaf 차단 툴 5개
- Kanban 내구 시스템: SQLite kanban.db, 상태 머신(triage→todo→ready→running→blocked/done/archived), CLI, hermes dashboard, hermes gateway start dispatcher
- Kanban vs delegate_task 선택 기준 비교표 (6행)
- 다중 에이전트 협업 패턴 5종 (fan-out, pipeline, voting, long-running journals, human-in-the-loop)
- 주의 callout 5개; Phase 5 전방 참조는 plain prose only; mdbook build exit 0

## Task Commits

Each task was committed atomically:

1. **Task 1: Ch.13 서브에이전트 본문 작성** - `ff85b94` (feat)
2. **Task 2: 빌드 검증 + SUMMARY 불변 확인** - no-op (파일 변경 없음, 커밋 생략)

**Plan metadata:** (this commit — docs: complete Ch.13 plan)

## Files Created/Modified

- `src/13-subagents/index.md` — Ch.13 서브에이전트 본문 전체 (delegate_task + Kanban 시스템 + 비교표)

## Decisions Made

- `delegate_task` Python 시그니처 코드블록은 실행 명령이 아닌 에이전트 내부 설명용임을 주석으로 명시 — `# 에이전트 내부 사용 — 독자는 자연어로 위임 요청`. 이 코드블록에는 `# 검증:` 주석을 붙이지 않음 (실행 명령이 아니므로).
- Task 2(빌드 검증)는 파일 변경 없음 — 03-03과 동일 패턴으로 no-op 커밋 생략, mdbook build exit 0만 확인.
- Kanban toolset 활성화 설명에서 `HERMES_KANBAN_TASK` 환경변수 자동 활성화 메커니즘을 명시 — RESEARCH.md HIGH confidence 근거.

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness

- Phase 4 Wave-2 계획 5개 중 04-05가 완료됨. Ch.13 서브에이전트가 mdBook에서 정상 렌더링.
- Phase 5(배포/게이트웨이) 시작 가능: Ch.13 "다음 단계"에서 gateway 운영이 Phase 5로 plain prose 연결됨.
- 04-02/03/04 완료 여부 확인 후 Phase 4 phase-complete 가능.

---
*Phase: 04-learning-automation*
*Completed: 2026-06-10*
