---
phase: 02-foundations
plan: 01
subsystem: content
tags: [mdbook, korean, hermes, tutorial, ch0, summary-md]

# Dependency graph
requires:
  - phase: 01-scaffold-ci
    provides: mdBook 0.5.3 scaffolding, book.toml, src/SUMMARY.md skeleton, CI deploy pipeline
provides:
  - SUMMARY.md with 4-chapter 기초 skeleton (00-intro/01-install/02-first-run/03-model)
  - Ch.0 소개 본문 (학습 루프 Do→Learn→Improve, 멀티모델 no-lock-in, 튜토리얼 맵, 한국어 UI callout)
  - Temporary placeholder index.md for 01-install/02-first-run/03-model (Wave-2 overwrites)
affects:
  - 02-02-PLAN (Ch.1 설치 — overwrites src/01-install/index.md)
  - 02-03-PLAN (Ch.2 첫 실행 — overwrites src/02-first-run/index.md)
  - 02-04-PLAN (Ch.3 모델 설정 — overwrites src/03-model/index.md)
  - All Phase 3+ content phases depend on this SUMMARY.md structure

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Ch.0 오리엔테이션 전용 패턴: 설치·명령 코드블록 없음, 개념 설명만"
    - "Wave-1 SUMMARY.md 단독 소유 패턴: 이후 챕터 플랜이 SUMMARY.md 쓰기 충돌 없이 병렬 실행 가능"
    - "mdBook auto-placeholder 패턴: SUMMARY.md에 링크된 미작성 파일은 mdBook 0.5.3가 자동 생성"

key-files:
  created:
    - src/01-install/index.md (placeholder — Wave-2에서 덮어씀)
    - src/02-first-run/index.md (placeholder — Wave-2에서 덮어씀)
    - src/03-model/index.md (placeholder — Wave-2에서 덮어씀)
  modified:
    - src/SUMMARY.md (단일 항목 → 4-챕터 스켈레톤)
    - src/00-intro/index.md (placeholder → Ch.0 소개 본문)

key-decisions:
  - "SUMMARY.md 단일 항목을 in-place 교체 (추가 금지): 중복 경로는 하드 빌드 오류 (Pitfall B-11)"
  - "Ch.0는 오리엔테이션 전용: 설치 명령·CLI 상세 없음, 명령 코드블록 없음"
  - "한국어 UI는 '부분 지원'으로 정확히 표현: 승인 프롬프트 등 정적 메시지만 번역, 에이전트 응답은 영어"
  - "mdBook auto-placeholder 파일을 별도 커밋에 포함: Wave-2 플랜이 덮어쓸 임시 파일임을 커밋 메시지에 명시"

patterns-established:
  - "Wave-1 owns SUMMARY.md: 후속 Wave-2 플랜(02-02/03/04)이 src/SUMMARY.md를 건드리지 않아도 됨"
  - "HIGH-confidence-only content: RESEARCH.md LOW confidence 항목은 챕터에 포함 금지"

# Metrics
duration: 2min
completed: 2026-06-09
---

# Phase 2 Plan 01: Ch.0 소개 본문 + SUMMARY.md 스켈레톤 Summary

**SUMMARY.md 4-챕터 스켈레톤 확정 + Ch.0 소개(학습 루프 Do→Learn→Improve, 멀티모델 40+ 공급자, 튜토리얼 맵) 작성, mdbook build exit 0 확인**

## Performance

- **Duration:** ~2 min
- **Started:** 2026-06-09T08:29:47Z
- **Completed:** 2026-06-09T08:31:25Z
- **Tasks:** 3
- **Files modified:** 5 (SUMMARY.md, 00-intro/index.md, 3x placeholder)

## Accomplishments

- SUMMARY.md 기초 섹션: 단일 항목 `[소개 및 배경]` → 4-챕터 스켈레톤(00-intro/01-install/02-first-run/03-model), 중복 경로 없음
- Ch.0 소개 본문: 학습 루프(Do→Learn→Improve), 멀티모델·노 락인(40+ 공급자), 튜토리얼 맵, 한국어 UI 부분지원 callout 포함, placeholder 제거
- mdBook 0.5.3 build exit 0, 경고/오류 없음 — Wave-2 플랜 병렬 실행 기반 완성

## Task Commits

1. **Task 1: SUMMARY.md 4-챕터 스켈레톤 확정** - `667bbb3` (feat)
2. **Task 2: Ch.0 소개 본문 작성** - `02cc5e6` (feat)
3. **Task 3: 빌드 검증** - `72f873d` (chore)

## Files Created/Modified

- `src/SUMMARY.md` — 단일 항목 → 4-챕터 스켈레톤으로 in-place 교체
- `src/00-intro/index.md` — 관례 데모 placeholder → Ch.0 소개 본문으로 overwrite
- `src/01-install/index.md` — mdBook auto-placeholder (Wave-2 02-02 플랜이 덮어씀)
- `src/02-first-run/index.md` — mdBook auto-placeholder (Wave-2 02-03 플랜이 덮어씀)
- `src/03-model/index.md` — mdBook auto-placeholder (Wave-2 02-04 플랜이 덮어씀)

## Decisions Made

- **SUMMARY.md in-place 교체:** 기존 `[소개 및 배경](00-intro/index.md)` 항목을 추가가 아닌 교체. 중복 경로는 `mdbook build` 하드 실패 (Pitfall B-11) — 이 결정은 RESEARCH.md와 플랜 모두 명시한 CRITICAL 제약이다.
- **한국어 UI 표현 정확화:** STATE.md 기존 결정("한국어 UI 미지원")을 RESEARCH.md 최신 조사로 업데이트 — `ko` 지원 존재하나 범위 협소(정적 메시지만). Ch.0 callout이 이를 정확히 반영.
- **명령 코드블록 없음:** Ch.0는 오리엔테이션 전용이므로 `# 검증:` 주석 불필요. RESEARCH.md 권고 준수.
- **mdBook auto-placeholder 커밋:** mdBook 0.5.3가 누락 링크 파일을 자동 생성 — 별도 커밋(chore)으로 Wave-2의 임시 파일임을 명시.

## Deviations from Plan

None — plan executed exactly as written. mdBook 0.5.3 auto-generated the 3 placeholder files (MEDIUM-confidence behavior mentioned in plan Task 3), confirming the fallback was not needed.

## Issues Encountered

None.

## User Setup Required

None — no external service configuration required.

## Next Phase Readiness

- SUMMARY.md 스켈레톤 완료: Wave-2 플랜(02-02/03/04)이 SUMMARY.md 쓰기 충돌 없이 병렬 실행 가능
- 01-install/02-first-run/03-model placeholder 존재: 각 Wave-2 플랜이 실제 본문으로 덮어쓰면 됨
- mdbook build 통과 상태 유지: Wave-2 각 플랜도 build 검증 태스크 포함해야 함
- Phase 2 성공 기준 #1 달성: Ch.0 소개 챕터가 학습 루프·멀티모델·튜토리얼 맵을 포함하며 빌드됨

---
*Phase: 02-foundations*
*Completed: 2026-06-09*
