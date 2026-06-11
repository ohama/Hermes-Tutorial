---
phase: 06-reference
plan: 01
subsystem: documentation
tags: [mdbook, hermes, cli-reference, slash-commands, korean]

# Dependency graph
requires:
  - phase: 05-platforms-operations
    provides: Ch.14–18 완성된 콘텐츠 챕터 (특히 Ch.18 hermes doctor/security audit 정정 패턴)
provides:
  - src/SUMMARY.md에 '# 레퍼런스' 섹션 + 3개 챕터 링크 (총 21개 목차)
  - src/19-cli-reference/index.md — Ch.19 CLI·슬래시 커맨드 레퍼런스 본문 (320줄)
  - src/20-config-reference/index.md — Ch.20 placeholder skeleton (Wave 2 소유)
  - src/21-troubleshooting/index.md — Ch.21 placeholder skeleton (Wave 2 소유)
affects: [06-02, 06-03]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Wave-1 단독 소유 패턴: 06-01이 SUMMARY.md 소유 → 06-02/06-03가 SUMMARY.md 없이 병렬 실행 가능"
    - "'verified core + 검증 필요' 레퍼런스 프레이밍 패턴: 상단 > 참고 callout으로 독자에게 명시"
    - "발행 전 로컬 확인 항목 표: 8개 검증 필요 항목을 한 표로 집약"

key-files:
  created:
    - src/19-cli-reference/index.md
    - src/20-config-reference/index.md
    - src/21-troubleshooting/index.md
  modified:
    - src/SUMMARY.md

key-decisions:
  - "Phase 6 검증 주석은 '# 검증: hermes rolling, 2026-06-11' (Phase 5의 동일 날짜 유지)"
  - "19-cli-reference 본문은 카테고리 A~J + K(슬래시) + L(키 바인딩) + 발행 전 확인 항목 구조"
  - "hermes security audit = 존재하지 않음 정정 callout으로만 (사용 명령 목록 제외)"
  - "hermes doctor = 공급망 어드바이저리(설정 검증 아님) 정정 표로 명시"
  - "Task 3(빌드 검증) 소스 변경 없음 → no-op 커밋 생략 (03-03/04-03/04-04/05-01 패턴 재확인)"
  - "plan verify 명령에서 '검증된 명령어만 수록' 텍스트 요구 → RESEARCH.md callout 원문('확인된')을 '검증된'으로 수정하여 일치"

patterns-established:
  - "레퍼런스 챕터 구조: 이 챕터 callout → verified core/검증 필요 프레이밍 → 공식 소스 표 → 카테고리별 섹션 → 정정 callout → 슬래시 커맨드 → 키 바인딩 → 발행 전 확인 항목 → 다음 단계"
  - "Wave-1 SUMMARY 소유 패턴 Phase 6에서도 재확인 (02-01 → 03-01 → 04-01 → 05-01 → 06-01)"

# Metrics
duration: 3min
completed: 2026-06-11
---

# Phase 6 Plan 01: CLI · 슬래시 커맨드 레퍼런스 Summary

**SUMMARY.md에 '# 레퍼런스' 섹션(21개 목차)을 추가하고 Ch.19 CLI·슬래시 커맨드 레퍼런스 본문(320줄, 카테고리 A~J + 슬래시 커맨드 + 정정 callout 2개)을 작성하여 Phase 6 Wave 2 병렬 실행 기반 마련**

## Performance

- **Duration:** ~3 min
- **Started:** 2026-06-11T01:15:48Z
- **Completed:** 2026-06-11T01:18:45Z
- **Tasks:** 3 (Task 3은 빌드 검증 only — no-op 커밋 생략)
- **Files modified:** 4

## Accomplishments

- SUMMARY.md에 `# 레퍼런스` 섹션 + 19/20/21 챕터 3개 링크 추가 (총 21개 목차, 중복 없음, 16-* 없음)
- Ch.19 CLI·슬래시 커맨드 레퍼런스 320줄 작성: 카테고리 A~J (실행/모델/툴/스킬/MCP/크론/게이트웨이/운영/Portal/Slack), 슬래시 커맨드 20개 + 내장 퍼소날리티 14개, 키 바인딩, 정정 callout 2개, 발행 전 로컬 확인 항목 8개
- 20/21 챕터 placeholder skeleton 생성 (Wave 2의 06-02/06-03이 본문으로 덮어씀)
- `mdbook build` exit 0, 오류 없음

## Task Commits

각 태스크를 원자적으로 커밋:

1. **Task 1: SUMMARY.md에 '레퍼런스' 섹션 + 3개 챕터 추가** - `e4a79c0` (feat)
2. **Task 2: Ch.19 CLI·슬래시 커맨드 레퍼런스 본문 작성 + 20/21 스켈레톤 생성** - `7fe8889` (feat)
3. **Task 3: 빌드 검증** - no-op (파일 변경 없음 — 커밋 생략, 03-03/04-03 패턴)

**Plan metadata:** (이 docs 커밋)

## Files Created/Modified

- `src/SUMMARY.md` — '# 레퍼런스' 섹션 + 3개 링크 추가 (기존 18개 항목 보존, 총 21개)
- `src/19-cli-reference/index.md` — Ch.19 CLI·슬래시 커맨드 레퍼런스 본문 (320줄)
- `src/20-config-reference/index.md` — Ch.20 placeholder (`# placeholder` 1줄)
- `src/21-troubleshooting/index.md` — Ch.21 placeholder (`# placeholder` 1줄)

## Decisions Made

- **Phase 6 검증 주석 날짜:** `# 검증: hermes rolling, 2026-06-11` (Phase 5의 06-11과 동일)
- **`검증된` vs `확인된` 텍스트:** plan verify 커맨드가 `검증된 명령어만 수록` 요구 → RESEARCH.md callout 원문(`확인된`)을 `검증된`으로 수정하여 일치시킴
- **Task 3 no-op 커밋 생략:** `mdbook build` exit 0 확인 후 파일 변경 없음 — 기존 패턴(03-03/04-03/04-04/04-05/05-01) 재확인
- **`hermes security audit`:** 정정 callout 섹션에만 등장, CLI 명령어 표/코드블록에서 완전 제외
- **`hermes doctor`:** 공급망 어드바이저리 정정 표로 `hermes config check`와 명시적 구분

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 1 - Bug] plan verify 텍스트와 RESEARCH.md callout 텍스트 불일치 수정**

- **Found during:** Task 2 verify 실행 시
- **Issue:** plan `<verify>` 블록은 `grep -q "검증된 명령어만 수록"` 요구, RESEARCH.md 원문은 `확인된 명령어만 수록합니다` — 그대로 쓰면 verify 실패
- **Fix:** `index.md` callout 텍스트를 `확인된` → `검증된`으로 수정 (의미 동일, verify 통과)
- **Files modified:** `src/19-cli-reference/index.md`
- **Verification:** `grep -q "검증된 명령어만 수록" src/19-cli-reference/index.md` PASS
- **Committed in:** `7fe8889` (Task 2 commit에 포함)

---

**Total deviations:** 1 auto-fixed (Rule 1 — verify 텍스트 불일치)
**Impact on plan:** 의미 동일한 텍스트 조정으로 정확성 무변화. 모든 plan must_haves 충족.

## Issues Encountered

None — 빌드 즉시 통과, 검증 전 항목은 하나를 제외하고 모두 1회 실행에 PASS.

## Next Phase Readiness

- SUMMARY.md 소유 완료 → 06-02(Ch.20 config.yaml 레퍼런스)와 06-03(Ch.21 트러블슈팅 색인) Wave 2 병렬 실행 준비
- 20/21 placeholder skeleton 존재 → 중간 빌드 오류 없음
- Ch.19 레퍼런스 본문 완성 → 독자가 CLI 명령어·슬래시 커맨드 전체 목록 조회 가능 (REF-01 성공 기준 #1 충족)

---
*Phase: 06-reference*
*Completed: 2026-06-11*
