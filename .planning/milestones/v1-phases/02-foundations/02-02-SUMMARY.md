---
phase: 02-foundations
plan: "02"
subsystem: content
tags: [hermes, install, mdbook, korean, PATH, cli]

# Dependency graph
requires:
  - phase: 02-01
    provides: SUMMARY.md 4-챕터 스켈레톤 + 01-install/index.md 링크 등록 + mdbook build 검증

provides:
  - "src/01-install/index.md — Ch.1 설치 본문 (OS별 설치 + PATH 재로드 + doctor + 주의 callout)"

affects:
  - 02-03 (Ch.2 첫 실행 — 이전 챕터 링크가 01-install을 가리킴)
  - 02-04 (Ch.3 모델 설정)

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "# 검증: hermes rolling (설치 시점 최신), YYYY-MM-DD 주석 — 모든 코드블록 첫 줄"
    - "LOW/MEDIUM confidence 출력은 [로컬 실행 후 캡처 필요] placeholder + > 검증 필요 callout"

key-files:
  created:
    - src/01-install/index.md
  modified: []

key-decisions:
  - "Tasks 1+2를 단일 파일 작성으로 처리 — 두 태스크 모두 동일 파일을 쓰므로 완전한 파일을 한 번에 작성하고 atomic commit"
  - "hermes doctor 출력은 RESEARCH LOW confidence → placeholder + 검증 필요 callout (발명 금지)"
  - "hermes version 예시 출력(0.8.0)은 MEDIUM confidence → '예시' 명시, 현재 버전으로 단정하지 않음"
  - "Windows 네이티브 제한 callout + Termux 별도 경로 callout 모두 포함 (RESEARCH Pitfall A-3, A-17)"

patterns-established:
  - "Confidence 기반 placeholder 패턴: MEDIUM/LOW 출력 → placeholder + > 검증 필요 admonition block"
  - "4개 주의 callout 구조: PATH 미반영(CRITICAL) + sudo 금지(CRITICAL) + Windows 제한 + Termux 별도 경로"

# Metrics
duration: 2min
completed: 2026-06-09
---

# Phase 2 Plan 02: Ch.1 설치 Summary

**macOS/Linux/WSL2 curl|bash + Windows PowerShell iex/irm 설치 명령, PATH 재로드 (#1 첫 실행 실패), hermes doctor/version 검증 (placeholder), 4개 주의 callout 포함한 Ch.1 설치 본문 작성**

## Performance

- **Duration:** ~2 min
- **Started:** 2026-06-09T08:34:03Z
- **Completed:** 2026-06-09T08:35:35Z
- **Tasks:** 3 (Task 1+2 단일 파일 작성, Task 3 빌드 검증)
- **Files modified:** 1

## Accomplishments

- src/01-install/index.md 전체 본문 작성 (macOS/Linux/WSL2 curl|bash + Windows ps1 설치 명령)
- PATH 재로드 섹션 (source ~/.bashrc/~/.zshrc, export PATH, which hermes) — 설치 직후 배치
- hermes doctor/version 섹션 — 불확실 출력은 placeholder + `> 검증 필요` callout으로 처리
- 4개 주의 callout: PATH 미반영(CRITICAL), sudo 금지(CRITICAL), Windows 네이티브 제한, Termux 별도 경로
- mdbook build exit 0, book/01-install/index.html 생성 확인

## Task Commits

1. **Task 1+2: Ch.1 설치 본문 전체 작성** - `e229cfd` (feat)
2. **Task 3: mdbook build 검증** — build exit 0, no new files to commit (verification only)

**Plan metadata:** (docs commit — see below)

## Files Created/Modified

- `src/01-install/index.md` — Ch.1 설치 본문: 사전 요구사항 표, macOS/Linux/WSL2 설치, Windows 설치, PATH 재로드, 설치 위치 표, hermes doctor/version 검증, 4개 주의 callout, 이전/다음 챕터 링크

## Decisions Made

- Tasks 1+2를 단일 atomic write로 처리 — 두 태스크가 동일 파일을 순차적으로 수정하므로 완전한 파일을 한 번에 작성하고 하나의 커밋으로 처리
- `hermes doctor` 출력은 RESEARCH에서 LOW confidence (출력 형식 미검증) → placeholder block + `> 검증 필요` callout 사용, 발명 절대 금지
- `hermes version` 예시(`0.8.0 (2026.4.8) [af4abd2f]`)는 RESEARCH MEDIUM confidence → "예시"로 명시하고 현재 버전으로 단정하지 않음
- Termux 명령 플래그는 LOW confidence (--skip-voice 등 미검증) → 명령 없이 "공식 문서 별도 안내" callout만 포함

## Deviations from Plan

None — plan executed exactly as written. Tasks 1+2 content written together in single file write (both target same file); all verification conditions passed.

## Issues Encountered

None.

## User Setup Required

None — no external service configuration required.

## Next Phase Readiness

- Ch.1 설치 본문 완료 → Wave-2 나머지 플랜(02-03 Ch.2, 02-04 Ch.3)이 이전/다음 챕터 링크로 연결 가능
- hermes doctor 출력, hermes version 정확한 버전 문자열은 로컬 설치 후 캡처 필요 (placeholder 교체 시점에 처리)
- Windows + Termux callout은 RESEARCH 기반 MEDIUM confidence — 로컬 검증 후 정확한 내용으로 업데이트 가능

---
*Phase: 02-foundations*
*Completed: 2026-06-09*
