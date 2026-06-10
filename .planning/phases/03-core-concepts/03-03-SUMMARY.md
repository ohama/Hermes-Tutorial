---
phase: 03-core-concepts
plan: "03"
subsystem: content
tags: [hermes, memory, FTS5, SQLite, compression, session_search, mdbook, korean]

# Dependency graph
requires:
  - phase: 03-01
    provides: SUMMARY.md with 06-memory/index.md entry + skeleton placeholder

provides:
  - "Ch.6 메모리 본문 (4계층 아키텍처 + 정정된 파일 경로 + session_search 툴 + FTS5 CJK + 압축 비용 설정)"

affects:
  - 03-04 (Ch.7 툴 게이트웨이 — session_search는 memory 툴셋에도 포함)
  - 03-05 (Ch.8 백엔드 — 압축 비용 맥락 재활용 가능)

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "LOW confidence 코드블록에 검증 필요 callout 동반"
    - "잘못된 경로를 텍스트 내에서 참조해야 할 때 그레이 리터럴 대신 설명으로 회피"

key-files:
  created:
    - src/06-memory/index.md
  modified: []

key-decisions:
  - "grep 검증 조건 `! grep -q '~/.hermes/MEMORY.md'` 충족을 위해 잘못된 경로 예시를 리터럴 대신 설명으로 표기 ('memories/ 하위 디렉터리를 생략한 경로')"
  - "'4,500x' 벤치마크 미인용 — RESEARCH.md Open Question #7에 따라 '~20ms FTS5 쿼리'만 사용"
  - "session_search를 슬래시 커맨드가 아닌 에이전트 내부 툴로 일관되게 표기"
  - "Task 2(빌드 검증)는 파일 변경 없음 — mdbook build exit 0 확인 후 별도 커밋 생략 (no-op commit 방지)"

patterns-established:
  - "메모리 관련 오류 중 가장 흔한 혼란(스냅샷 동결)을 개념 섹션과 흔한 오류 섹션 양쪽에서 반복 강조"
  - "한국어 독자 전용 callout 패턴: CJK trigram 테이블 설명"

# Metrics
duration: 2min
completed: 2026-06-10
---

# Phase 3 Plan 03: Ch.6 메모리 Summary

**4계층 메모리 아키텍처, 정정된 파일 경로(~/.hermes/memories/), session_search 내부 툴, FTS5 trigram CJK 지원, 압축 비용 관리 설정을 담은 Ch.6 한국어 본문 277줄 작성 및 mdbook build 통과**

## Performance

- **Duration:** 2min
- **Started:** 2026-06-10T07:53:05Z
- **Completed:** 2026-06-10T07:55:50Z
- **Tasks:** 2 (Task 1: 본문 작성, Task 2: 빌드 검증)
- **Files modified:** 1

## Accomplishments

- `src/06-memory/index.md` 277줄 한국어 본문 작성 — RESEARCH.md Ch.6 섹션의 HIGH confidence 사실만 사용
- 정정된 파일 경로 (`~/.hermes/memories/MEMORY.md`, `~/.hermes/memories/USER.md`) 정확히 반영
- `session_search`를 에이전트 내부 툴로 명확히 설명 (슬래시 커맨드 아님)
- `messages_fts_trigram` 테이블 + 한국어(CJK) 독자 전용 callout 포함
- 압축 비용 설정 (`threshold`, `target_ratio`, `protect_last_n`, 보조 모델) + ~1,300토큰 오버헤드 개념
- 4개 흔한 오류/주의 callout (스냅샷 동결, 게이트웨이 비용 급증, session_search 오해, 보조 모델 컨텍스트 윈도우)
- 모든 코드블록에 `# 검증: hermes rolling, 2026-06-09` 주석 적용
- mdbook build exit 0, SUMMARY.md 변경 없음 확인

## Task Commits

Each task was committed atomically:

1. **Task 1: Ch.6 메모리 본문 작성** - `1600815` (feat)
2. **Task 2: 빌드 검증** - 파일 변경 없음, 별도 커밋 없음 (exit 0 확인)

**Plan metadata:** (docs commit follows)

## Files Created/Modified

- `src/06-memory/index.md` — Ch.6 메모리 본문 (277줄): 4계층 아키텍처, 파일 경로, memory 툴, 메모리 플러시 타이밍, session_search + FTS5, 메모리 설정 + 압축, 실습 CLI, 4개 주의 callout, 다음 단계 링크

## Decisions Made

- `grep -q "~/.hermes/MEMORY.md"` 검증 조건 충족을 위해 잘못된 경로 예시를 리터럴 코드가 아닌 자연어 설명("memories/ 하위 디렉터리를 생략한 경로")으로 표기
- "4,500x" 수치 미인용 — RESEARCH.md Open Question #7에 따라 "~20ms FTS5 쿼리"만 사용
- Task 2는 파일 변경 없는 순수 검증 태스크 — no-op 커밋 생략, mdbook build exit 0만 확인

## Deviations from Plan

None — plan executed exactly as written. 모든 12개 grep 검증 조건 PASS, mdbook build exit 0, SUMMARY.md 미수정 확인.

## Issues Encountered

None.

## User Setup Required

None — no external service configuration required.

## Next Phase Readiness

- Ch.6 본문 완료 — Phase 3 Wave 2 중 03-02(Ch.5), 03-03(Ch.6), 03-04(Ch.7), 03-05(Ch.8) 모두 실행 가능
- mdbook build 통과 — 빌드 파이프라인 정상

---
*Phase: 03-core-concepts*
*Completed: 2026-06-10*
