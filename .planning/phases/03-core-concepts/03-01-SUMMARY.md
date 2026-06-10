---
phase: 03-core-concepts
plan: 01
subsystem: docs
tags: [mdbook, hermes, agent-loop, prompt-system, korean-tutorial]

# Dependency graph
requires:
  - phase: 02-foundations
    provides: "기초 4개 챕터 + SUMMARY.md 기초 섹션 (이 플랜이 핵심 개념 섹션을 추가)"
provides:
  - "src/SUMMARY.md 핵심 개념 섹션 (5개 챕터 링크: 04~08)"
  - "src/04-agent-loop/index.md — Ch.4 에이전트 루프 본문"
  - "Wave 2 병렬 실행 기반 (03-02~03-05가 SUMMARY.md 쓰기 충돌 없이 병렬 실행 가능)"
affects:
  - 03-02-Ch.5-context-files
  - 03-03-Ch.6-memory
  - 03-04-Ch.7-tools
  - 03-05-Ch.8-backends

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "SUMMARY.md 섹션 구분자(---) + 헤딩으로 mdBook 챕터 그룹화"
    - "# 검증: hermes rolling, YYYY-MM-DD 주석을 모든 실행 명령 블록 첫 줄에 추가"
    - "LOW confidence 출력은 # [로컬 실행 후 캡처 필요 — ...] + > 검증 필요 callout으로 표시"

key-files:
  created:
    - src/04-agent-loop/index.md
  modified:
    - src/SUMMARY.md

key-decisions:
  - "SUMMARY.md 단독 소유 플랜으로 Wave 2 병렬 실행 경쟁 조건 제거 (owner-plan 패턴)"
  - "perceive→decide→act 프레이밍 완전 배제 — 공식 사이클 Assemble→Call→Parse→Execute→Persist→Loop만 사용"
  - "mdBook auto-created 05~08 placeholder files는 커밋하지 않음 (Wave 2 플랜이 덮어씀)"

patterns-established:
  - "Owner-plan 패턴: SUMMARY.md를 한 플랜이 단독 소유하여 후속 병렬 플랜의 쓰기 충돌 방지"
  - "RESEARCH.md HIGH confidence 사실만 본문에 사용; LOW/MEDIUM 출력은 placeholder"
  - "공식 루프 사이클 표현 고정: Assemble→Call→Parse→Execute→Persist→Loop"

# Metrics
duration: 3min
completed: 2026-06-10
---

# Phase 03 Plan 01: 에이전트 루프 + SUMMARY.md 핵심 개념 스켈레톤 Summary

**SUMMARY.md에 # 핵심 개념 섹션(5개 챕터 링크)을 추가하고, Ch.4 에이전트 루프 본문을 공식 사이클(Assemble→Call→Parse→Execute→Persist→Loop)·3티어 프롬프트·ThreadPoolExecutor 툴 디스패치·max_turns 90 종료 조건으로 작성**

## Performance

- **Duration:** 3 min
- **Started:** 2026-06-10T07:48:10Z
- **Completed:** 2026-06-10T07:50:44Z
- **Tasks:** 3
- **Files modified:** 2

## Accomplishments

- `src/SUMMARY.md`에 `# 핵심 개념` 섹션과 5개 챕터 링크(04~08) 추가 — 기존 기초 4개 항목 보존, 중복 경로 없음
- `src/04-agent-loop/index.md` 196줄 작성 — 공식 루프 사이클 9단계, ASCII 다이어그램, 3티어 요약, API 모드 표, ThreadPoolExecutor 툴 디스패치, max_turns 종료 조건
- `mdbook build` exit 0 확인 — 05~08 파일은 mdBook이 자동 생성(Wave 2 플랜이 덮어씀)

## Task Commits

각 태스크는 개별 커밋으로 저장됨:

1. **Task 1: SUMMARY.md에 핵심 개념 섹션 + 5개 챕터 추가** - `5add9b6` (feat)
2. **Task 2: Ch.4 에이전트 루프 본문 작성** - `69c6884` (feat)
3. **Task 3: 빌드 검증** — 별도 커밋 없음 (src 변경 없음, mdBook이 auto-create한 파일은 Wave 2 소유)

**Plan metadata:** (이 커밋 후 생성됨)

## Files Created/Modified

- `src/SUMMARY.md` — 기초 4개 + 핵심 개념 5개 = 9개 챕터 링크 (중복 경로 없음)
- `src/04-agent-loop/index.md` — Ch.4 에이전트 루프 본문 (196줄)

## Decisions Made

- **perceive→decide→act 배제:** 공식 문서에 없는 프레이밍이므로 완전 배제; `# 검증:` 검증 검사(`! grep -qi "perceive"`)를 통과하도록 해당 단어를 언급하지 않고 공식 표현만 사용
- **mdBook auto-created 05~08 files:** `src/05-context-files/`, `src/06-memory/`, `src/07-tools/`, `src/08-backends/`는 mdBook이 자동 생성한 미커밋 파일 — Wave 2 플랜(03-02~03-05)이 덮어쓸 것이므로 커밋하지 않음
- **날짜 주석 `2026-06-09` 유지:** RESEARCH.md와 PLAN.md 모두 `# 검증: hermes rolling, 2026-06-09`를 명시하므로 그대로 사용

## Deviations from Plan

None — plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness

- SUMMARY.md 스켈레톤 완성 — Wave 2 플랜(03-02~03-05)이 병렬 실행 가능
- Ch.4 에이전트 루프 본문 완성 (196줄, min_lines 80 초과)
- mdBook build 오류 없음 (exit 0)
- 다음: Wave 2 플랜 병렬 실행 (03-02 Ch.5 컨텍스트 파일 / 03-03 Ch.6 메모리 / 03-04 Ch.7 툴 게이트웨이 / 03-05 Ch.8 터미널 백엔드)

---
*Phase: 03-core-concepts*
*Completed: 2026-06-10*
