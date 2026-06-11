---
phase: 06-reference
plan: "03"
subsystem: documentation
tags: [mdbook, troubleshooting, korean, reference, index]

# Dependency graph
requires:
  - phase: 06-01
    provides: src/21-troubleshooting/index.md skeleton + SUMMARY.md 레퍼런스 섹션
provides:
  - Ch.21 마스터 트러블슈팅 색인 본문 (230줄) — 37개 오류 항목 집약 + 카테고리별 구조 + 19개 백워드 챕터 링크 + 용어 사전 17개 항목
affects: []

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "마스터 색인 패턴: 기존 챕터 흔한-오류--주의 섹션을 백워드 상대 링크로 집약 (전방 참조 금지 규칙은 대상 파일 존재 시 비적용)"
    - "용어 사전 패턴: 핵심 용어 + 정의 표로 부록 마지막에 배치"

key-files:
  created: []
  modified:
    - src/21-troubleshooting/index.md

key-decisions:
  - "06-03: 트러블슈팅 표 자체는 코드블록이 아니므로 # 검증: 주석 예외 처리 — 백워드 링크 heading anchor 패턴(#흔한-오류--주의)으로만 연결"
  - "06-03: 19개 고유 챕터 링크 확보 (요구 최소 15개 초과 달성) — 각 섹션별 출처 링크 + 각 표 내 동일 챕터 반복 링크 포함"
  - "06-03: Task 2 빌드 검증 소스 변경 없음 → no-op 커밋 생략 (03-03/04-03/04-04/04-05/05-01/06-01 동일 패턴)"

patterns-established:
  - "백워드 링크 패턴: 대상 파일이 존재하는 경우 전방 참조 금지 규칙 비적용 — ../NN-name/index.md#흔한-오류--주의 형식"
  - "색인 부록 구조: 상단 callout(이챕터 설명) + 검색 callout(Ctrl+F) + 카테고리별 섹션 + 용어 사전 + 다음 단계"

# Metrics
duration: 2min
completed: 2026-06-11
---

# Phase 06 Plan 03: Ch.21 마스터 트러블슈팅 색인 Summary

**17개 챕터(Ch.1–18, 16 없음) 37개 오류·주의 항목을 6개 카테고리로 집약한 마스터 트러블슈팅 색인 부록, 19개 백워드 챕터 링크 + 용어 사전 17개 항목 포함**

## Performance

- **Duration:** ~2 min
- **Started:** 2026-06-11T01:21:23Z
- **Completed:** 2026-06-11T01:24:17Z
- **Tasks:** 2 (Task 2 no-op — build verification only, no source changes)
- **Files modified:** 1

## Accomplishments

- src/21-troubleshooting/index.md (230줄) — `# placeholder` → 완전한 Ch.21 본문으로 overwrite
- RESEARCH.md 06-03 집약 데이터의 37개 항목 전체 수록 (6개 카테고리 구조)
- 19개 고유 챕터 백워드 상대 링크 (#흔한-오류--주의 앵커 포함) — 요구 최소(15개) 초과
- 용어 사전 17개 항목: SOUL.md/MEMORY.md/USER.md/.hermes.md/스킬/Curator/Tool Gateway/delegate_task/Kanban/Tirith/approvals.mode/hermes doctor/hermes config check/ACP/MCP/s6-overlay/FTS5 trigram/compress.protect_first_n
- hermes doctor=공급망, security audit=미존재, redact_secrets=자격증명만, disabled_toolsets=미존재 — 본문 정정과 일관
- mdbook build exit 0, src/SUMMARY.md 미수정 확인

## Task Commits

Each task was committed atomically:

1. **Task 1: Ch.21 마스터 트러블슈팅 색인 본문 작성** - `0f589df` (feat)
2. **Task 2: 빌드 검증 + SUMMARY.md 미수정 확인** - (no-op — 소스 변경 없음; 커밋 생략 패턴)

**Plan metadata:** (committed below)

## Files Created/Modified

- `src/21-troubleshooting/index.md` — Ch.21 마스터 트러블슈팅 색인 본문 (37개 오류 항목, 6개 카테고리, 19개 백워드 링크, 17개 용어 사전, 230줄)

## Decisions Made

- 트러블슈팅 표 자체는 코드블록이 아니므로 `# 검증: hermes rolling, 2026-06-11` 주석 예외 처리 (실행 명령이 없는 설명 테이블)
- Task 2 빌드 검증 소스 변경 없음 → no-op 커밋 생략 (03-03/04-03/04-04/04-05/05-01/06-01 동일 패턴 재확인)
- 19개 고유 챕터 링크 달성 — 각 섹션 헤더의 출처 링크 + 표 내 항목의 동일 링크 반복으로 요구 최소(15) 초과 달성

## Deviations from Plan

None — plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None — no external service configuration required.

## Next Phase Readiness

Phase 6 (Reference) 전체 완료:
- 06-01 ✓: src/SUMMARY.md 레퍼런스 섹션 + Ch.19 CLI 레퍼런스
- 06-02 ✓: Ch.20 config.yaml 레퍼런스 (Wave 2)
- 06-03 ✓: Ch.21 마스터 트러블슈팅 색인 (Wave 2) — 이 플랜

**튜토리얼 전체(Ch.0–21) 완성.** 남은 작업: Phase 6 complete-phase 아카이브 또는 최종 검토.

---
*Phase: 06-reference*
*Completed: 2026-06-11*
