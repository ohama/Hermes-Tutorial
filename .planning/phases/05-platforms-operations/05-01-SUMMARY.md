---
phase: 05-platforms-operations
plan: 01
subsystem: content
tags: [mdbook, telegram, gateway, hermes, korean, tutorial]

requires:
  - phase: 04-learning-automation
    provides: "Phase 4 Wave-1 owner pattern (SUMMARY.md 단독 소유); Phase 4 챕터 스타일 관례 확립"

provides:
  - "src/SUMMARY.md에 '# 플랫폼과 운영' 섹션 + 4개 챕터 항목(14~18) 추가 — 18개 총 링크, 중복 없음"
  - "src/14-gateways/index.md: Ch.14 Telegram 게이트웨이 본문 221줄 (BotFather + .env + hermes gateway run + 폴링 충돌 함정 + 토큰 보안)"
  - "src/15-more-gateways/index.md, src/17-deploy/index.md, src/18-security/index.md: Wave-2 플랜(05-02~05-04) placeholder 스켈레톤"

affects:
  - 05-02-PLAN (Ch.15 추가 게이트웨이 — 15-more-gateways/index.md 본문 overwrite)
  - 05-03-PLAN (Ch.17 프로덕션 배포 — 17-deploy/index.md 본문 overwrite)
  - 05-04-PLAN (Ch.18 보안 하드닝 — 18-security/index.md 본문 overwrite)

tech-stack:
  added: []
  patterns:
    - "Wave-1 owner plan: SUMMARY.md 단독 소유 패턴 Phase 5에도 적용 (02-01/03-01/04-01 동일)"
    - "Placeholder skeleton: Wave-2 플랜이 덮어쓸 파일은 '# placeholder' 단일 줄로 생성"
    - "검증 주석 관례: 모든 명령/설정 코드블록 첫 줄 '# 검증: hermes rolling, 2026-06-11'"

key-files:
  created:
    - src/14-gateways/index.md
    - src/15-more-gateways/index.md
    - src/17-deploy/index.md
    - src/18-security/index.md
  modified:
    - src/SUMMARY.md

key-decisions:
  - "Ch.14 디렉터리는 14-gateways/ (연구 초안의 14-telegram/ 아님) — PLAN.md planning_context 확정값"
  - "16-* 항목/디렉터리 없음 — Ch.15–16은 15-more-gateways 단일 파일이 커버 (mdBook 중복 빌드 오류 방지)"
  - "hermes gateway vs hermes gateway run 구분 명시 — run은 Docker 내 실행; gateway start는 install 이후 서비스 시작용"
  - "Task 3(빌드 검증) 파일 변경 없음 → no-op 커밋 생략 (03-03/04-03/04-04/04-05 동일 패턴)"
  - "Phase 5 검증 주석은 '# 검증: hermes rolling, 2026-06-11' (Phase 4의 06-10과 구분)"

patterns-established:
  - "토큰 placeholder: .env 예시에 <YOUR_BOT_TOKEN> 사용 — 실제처럼 보이는 토큰 절대 금지"
  - "LOW confidence 출력 → '# [로컬 실행 후 캡처 필요 — ...]' + '> 검증 필요:' callout"
  - "3가지 게이트웨이 실행 방식 구분 표(hermes gateway / hermes gateway run / hermes gateway install + start) — 후속 챕터도 동일 구분 유지"

duration: 2min
completed: 2026-06-11
---

# Phase 5 Plan 01: 플랫폼과 운영 SUMMARY 스켈레톤 + Ch.14 Telegram 게이트웨이 Summary

**BotFather 토큰 → `~/.hermes/.env` → `hermes gateway run` 흐름의 Telegram 게이트웨이 본문(221줄)과 SUMMARY.md 18-챕터 스켈레톤(플랫폼과 운영 4항목) 완성; mdbook build exit 0**

## Performance

- **Duration:** ~2 min
- **Started:** 2026-06-11T00:37:53Z
- **Completed:** 2026-06-11T00:40:02Z
- **Tasks:** 3 (Task 3: build verify, no files modified)
- **Files modified:** 5

## Accomplishments

- `src/SUMMARY.md`에 '# 플랫폼과 운영' 섹션 + 4개 챕터 항목 추가; 총 18개 index.md 링크, 16-* 없음, 중복 없음; Wave-2 병렬 실행 기반 확립
- `src/14-gateways/index.md` 221줄: BotFather 5단계 → .env 토큰 저장 → hermes gateway setup 마법사 → 게이트웨이 시작(포그라운드/Docker/서비스) → 폴링 vs 웹훅 → 4개 주의 callout(폴링 중복/ALLOW_ALL_USERS/토큰 보안/비용) → 다음 챕터 링크
- `src/15-more-gateways/`, `src/17-deploy/`, `src/18-security/` 스켈레톤 생성; `mdbook build` exit 0

## Task Commits

1. **Task 1: SUMMARY.md에 '플랫폼과 운영' 섹션 + 4개 챕터 추가** - `f9e44d2` (feat)
2. **Task 2: Ch.14 Telegram 게이트웨이 본문 + 15/17/18 스켈레톤** - `0021d95` (feat)
3. **Task 3: 빌드 검증** - no commit (파일 변경 없음, no-op 패턴)

**Plan metadata:** (docs commit below)

## Files Created/Modified

- `src/SUMMARY.md` — '# 플랫폼과 운영' 섹션 + 4개 항목 추가 (14-gateways/15-more-gateways/17-deploy/18-security); 18개 총 링크
- `src/14-gateways/index.md` — Ch.14 Telegram 게이트웨이 본문 221줄; BotFather + .env + hermes gateway 실행 방식 + 폴링 충돌 함정 + 토큰 보안
- `src/15-more-gateways/index.md` — placeholder 스켈레톤 (05-02가 본문으로 overwrite)
- `src/17-deploy/index.md` — placeholder 스켈레톤 (05-03이 본문으로 overwrite)
- `src/18-security/index.md` — placeholder 스켈레톤 (05-04가 본문으로 overwrite)

## Decisions Made

- **Ch.14 디렉터리**: `14-gateways/` (PLAN.md 확정값; 연구 초안의 `14-telegram/` 아님)
- **16-* 항목/디렉터리 없음**: Ch.15–16은 `15-more-gateways/index.md` 단일 파일이 커버; 16-* 추가하면 mdBook 0.5.3 빌드 오류
- **`hermes gateway run` 구분 명시**: 포그라운드는 `hermes gateway` 또는 Docker 내 `hermes gateway run`; `hermes gateway start`는 install 이후 서비스 시작 전용임을 본문에서 명확히 구분
- **Phase 5 검증 주석**: `# 검증: hermes rolling, 2026-06-11` (Phase 4의 2026-06-10과 구분)
- **Task 3 no-op 커밋 생략**: 빌드 검증은 파일 변경 없음 — 03-03/04-03/04-04/04-05 동일 패턴

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness

- **05-02 (Ch.15 추가 게이트웨이)**: `src/15-more-gateways/index.md` placeholder 존재; SUMMARY.md 링크 확립; Wave-2 병렬 실행 가능
- **05-03 (Ch.17 프로덕션 배포)**: `src/17-deploy/index.md` placeholder 존재
- **05-04 (Ch.18 보안 하드닝)**: `src/18-security/index.md` placeholder 존재
- **mdbook build exit 0** 확인 완료; 후속 플랜이 스켈레톤을 본문으로 덮어써도 빌드 유지
- **Blocker**: Phase 5 Discord OAuth + Privileged Gateway Intents 흐름 2026-03 이후 변경 가능 — Discord 서브챕터(05-02) 작성 전 검증 권장 (STATE.md 기존 우려사항)

---
*Phase: 05-platforms-operations*
*Completed: 2026-06-11*
