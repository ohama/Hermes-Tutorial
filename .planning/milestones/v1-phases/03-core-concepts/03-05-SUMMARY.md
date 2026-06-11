---
phase: 03-core-concepts
plan: "05"
subsystem: content
tags: [mdbook, hermes, terminal-backend, docker, ssh, modal, daytona, singularity, korean]

# Dependency graph
requires:
  - phase: 03-01
    provides: SUMMARY.md 핵심 개념 섹션 + 08-backends/index.md 빈 플레이스홀더
provides:
  - Ch.8 터미널 백엔드 본문 (287줄) — 6개 백엔드 개요·전환 3방법·Docker 하드닝/핸즈온·원격 백엔드 개요·4개 주의 callout
affects:
  - Phase 5 (배포) — Docker/SSH/Modal/Daytona/Singularity 심화 섹션이 이 챕터를 선행 요건으로 참조

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Phase 5 전방 참조 = plain prose only (mdBook 상대 링크 금지 — 파일 미존재)"
    - "MEDIUM/LOW confidence 항목 = 검증 필요 callout + 로컬 실행 후 캡처 필요 placeholder"
    - "모든 명령/설정 코드블록 첫 줄 # 검증: ... 주석 (단, 설명용 플래그 목록은 예외)"

key-files:
  created:
    - src/08-backends/index.md
  modified: []

key-decisions:
  - "Docker 보안 하드닝 플래그 목록은 명령이 아닌 설명 텍스트로 표현 — # 검증: 주석 예외 처리"
  - "hermes setup terminal 마법사 흐름은 MEDIUM → 검증 필요 callout으로 처리 (Open Question #3)"
  - "Modal managed mode 미문서화 → 검증 필요 callout으로 처리 (Open Question #5)"
  - "Phase 5 링크 금지 — 파일 미존재로 dead link 위험; plain prose로만 전방 참조"

patterns-established:
  - "원격 백엔드 섹션 상단에 전체 섹션에 대한 MEDIUM 신뢰도 경고 callout 배치"
  - "로컬 실행 불가 출력은 코드블록 안에 # [... 로컬 실행 후 캡처 필요] placeholder + 별도 검증 필요 callout"

# Metrics
duration: 1min
completed: "2026-06-10"
---

# Phase 3 Plan 05: Ch.8 터미널 백엔드 Summary

**6개 백엔드 개요표(local/docker/ssh/singularity/modal/daytona) + 전환 3방법 + Docker 자동 하드닝 + 최소 핸즈온 + MEDIUM/LOW callout을 포함한 287줄 한국어 챕터**

## Performance

- **Duration:** ~1 min
- **Started:** 2026-06-10T07:53:35Z
- **Completed:** 2026-06-10T07:54:30Z
- **Tasks:** 2
- **Files modified:** 1

## Accomplishments

- RESEARCH.md의 Ch.8 섹션을 근거로 `src/08-backends/index.md` 완전 본문 작성 (287줄)
- 6개 백엔드 개요표, 전환 3방법, Docker 전체 설정, Docker 핸즈온, 원격 백엔드 4개 개요 포함
- MEDIUM/LOW confidence 항목 전체를 `> 검증 필요:` callout + placeholder로 처리
- Phase 5 전방 참조는 plain prose만 사용 (죽은 mdBook 링크 없음)
- `mdbook build` exit 0, `src/SUMMARY.md` 변경 없음

## Task Commits

1. **Task 1: Ch.8 터미널 백엔드 본문 작성** - `6f74498` (feat)
2. **Task 2: 빌드 검증** — 별도 커밋 불필요 (파일 미수정; mdbook build exit 0 확인만)

**Plan metadata:** (이 커밋)

## Files Created/Modified

- `src/08-backends/index.md` — Ch.8 터미널 백엔드 본문 (287줄); 6개 백엔드 개요·전환·Docker 하드닝/핸즈온·원격 백엔드·4개 주의 callout

## Decisions Made

- **Docker 보안 하드닝 플래그 목록 표현:** 명령이 아닌 설명 텍스트로 표현하여 `# 검증:` 주석 예외 처리. 독자가 직접 설정할 내용이 아니므로 적절.
- **hermes setup terminal MEDIUM 처리:** 마법사 정확한 흐름 미검증 → `> 검증 필요:` callout으로 표시 (Open Question #3).
- **Modal managed mode 미문서화:** 공개 문서에 충분히 기술되지 않아 단정 없이 callout만 (Open Question #5).
- **Phase 5 링크 금지:** Phase 5 파일들이 미존재 → 상대 링크 생성 시 dead link 위험; plain prose 전방 참조만 사용.
- **원격 백엔드 섹션 상단 전체 MEDIUM callout:** 섹션 전체에 대한 신뢰도 경고를 상단에 배치하고 각 백엔드별 추가 callout 병행.

## Deviations from Plan

None — plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None — no external service configuration required. Docker 백엔드 사용 시 독자가 Docker 데몬을 실행해야 하지만 이는 Hermes 설정이 아닌 시스템 요구사항이며 챕터 본문에 안내됨.

## Next Phase Readiness

- Phase 3 Wave 2 완료 (03-05). Wave 2의 나머지 플랜(03-02 ~ 03-04)이 완료되면 Phase 3 전체 완성.
- Phase 5(배포 단계)에서 SSH/Modal/Daytona 심화 내용 작성 시 이 챕터의 개요 섹션을 선행 참조할 것.
- Modal/Daytona/Singularity YAML 키는 MEDIUM 신뢰도 — Phase 5 작성 전 로컬 검증 필요 (Open Question #8 유지).

---
*Phase: 03-core-concepts*
*Completed: 2026-06-10*
