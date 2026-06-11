---
phase: 05-platforms-operations
plan: 04
subsystem: content
tags: [hermes, security, approvals, tirith, dotenv, api-keys, supply-chain, cve, mdbook]

# Dependency graph
requires:
  - phase: 05-01
    provides: SUMMARY.md with 18-security entry + src/18-security/index.md skeleton
  - phase: 05-03
    provides: Ch.17 프로덕션 배포 (linked back from Ch.18)
provides:
  - Ch.18 보안 하드닝 body (src/18-security/index.md, 327 lines)
  - approvals.mode (manual/smart/off) documented with production recommendation
  - .env vs config.yaml separation pattern with chmod 600 and Issue #16394 caution
  - Tirith scanning config with accurate redact_secrets scope clarification
  - Operational runbook: hermes doctor (supply-chain CVE) / hermes config check (config) / hermes status (runtime)
  - 10-step production security checklist
  - Three factual corrections: doctor scope, security audit absence, PII scope
affects: [phase-6-reference-appendix]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "No-op commit skipped when full file written in single task (consistent with 03-03/04-03/04-04/04-05)"
    - "Phase 5 검증 주석 패턴: # 검증: hermes rolling, 2026-06-11"
    - "LOW-confidence output → [로컬 실행 후 캡처 필요] placeholder + > 검증 필요 callout"
    - "Phase 6 forward reference = plain prose only (no mdBook relative links to non-existent files)"

key-files:
  created:
    - src/18-security/index.md
  modified: []

key-decisions:
  - "Full file written in Task 1 commit covering both Task 1 + Task 2 content — no-op Task 2 commit skipped (consistent with 03-03/04-03/04-04/04-05 pattern)"
  - "hermes doctor described as supply-chain CVE advisory checker only — config validation is hermes config check"
  - "hermes security audit documented ONLY as non-existent correction callout — never presented as runnable command"
  - "security.redact_secrets scoped precisely to MCP error credential pattern masking — not general PII filter"
  - "approvals.mode production recommendation: manual 유지 (not off)"
  - "All key/token examples use placeholders (<YOUR_ANTHROPIC_API_KEY>) with .env-only-storage + never-commit callout"
  - "Phase 6 reference appendix: plain prose forward reference only — no 06-* mdBook links"

patterns-established:
  - "Factual correction pattern: 흔한 오류/주의 section re-confirms all three corrections at chapter end"
  - "Security audit absence: correction callout in operational runbook section + re-confirmed in 흔한 오류 section"

# Metrics
duration: 2min
completed: 2026-06-11
---

# Phase 5 Plan 04: Ch.18 보안 하드닝 Summary

**Ch.18 보안 하드닝 327줄 — approvals.mode(manual 권장/off 경고), .env 키 관리(chmod 600/#16394), Tirith 스캐닝(redact_secrets PII 범위 정정), hermes doctor(공급망 CVE)/config check(설정)/status(런타임) 운영 런북, 10단계 체크리스트, security audit 부재 정정**

## Performance

- **Duration:** 2 min
- **Started:** 2026-06-11T00:42:56Z
- **Completed:** 2026-06-11T00:45:49Z
- **Tasks:** 2 (written in single file pass; Task 2 no-op commit skipped)
- **Files modified:** 1

## Accomplishments

- Ch.18 본문 전체 작성(327줄) — 연구에서 확인된 세 가지 핵심 정정 사항 정확히 반영
- `hermes doctor` = 공급망 어드바이저리 검사기(Python 패키지 CVE)로 정확히 기술; 설정 검증은 `hermes config check`로 분리
- `hermes security audit` 명령이 존재하지 않음을 정정 callout으로 명시; 어디에도 실행 가능 명령으로 제시하지 않음
- `security.redact_secrets` = MCP 오류 자격증명 패턴 난독화(일반 PII 필터 아님)로 범위 한정
- `approvals.mode: manual` 프로덕션 권장; `off` 경고 포함; YOLO mode 설명
- `.env` 기반 API 키 관리: chmod 600, GitHub Issue #16394 silent skip 버그 주의
- 운영 런북 7단계(doctor→config check→status→gateway status→logs→update→insights)
- 게이트웨이 인증 우선순위 5단계 + 10단계 프로덕션 체크리스트
- Phase 6 레퍼런스 부록 전방 참조 = plain prose only (06-* mdBook 링크 없음)
- `mdbook build` exit 0, SUMMARY.md 미수정 확인

## Task Commits

Each task was committed atomically:

1. **Task 1: 승인 모드 + API 키 관리 + Tirith 작성 (+ Task 2 내용 포함)** - `5eedbbc` (feat)

   *Note: Task 2 content (운영 런북 + 게이트웨이 인증 + 체크리스트 + 마무리) was written in the same
   single file pass as Task 1 — consistent with project pattern (see 02-02, 03-03, 04-03 decisions).
   No-op Task 2 commit skipped.*

**Plan metadata:** (this commit, docs)

## Files Created/Modified

- `src/18-security/index.md` — Ch.18 보안 하드닝 본문 (327줄): approvals.mode + .env 키 관리 + Tirith + 운영 런북 + 10단계 체크리스트 + 3개 정정 callout + 마무리

## Decisions Made

- **Full file written in single pass:** Task 1 commit includes all content. Task 2 no-op commit skipped per 03-03/04-03/04-04/04-05 established pattern.
- **Section ordering:** 개요 → 승인 모드 → API 키 관리 → Tirith → 운영 런북 → 게이트웨이 인증 + 체크리스트 → 흔한 오류 → 마무리 — matches RESEARCH.md structure and plan specification.
- **PII 정정 placement:** Placed as sub-section of Tirith (§ `redact_secrets`) AND re-confirmed in 흔한 오류 section for maximum visibility.
- **security audit callout placement:** Inside 운영 런북 section (## 운영 런북 → ### `hermes security audit`는 존재하지 않음) — matches RESEARCH.md callout position.

## Deviations from Plan

None — plan executed exactly as written. The "single file pass" pattern is not a deviation; it is the established project convention for same-file two-task plans (documented in STATE.md decisions for 02-02).

## Issues Encountered

- **grep for `manual 유지` failed** on first run: heading used `` `manual` 유지 `` (with backticks), which does not match plain-text search. Fixed by ensuring the phrase `manual 유지` appears in prose text. Auto-fixed Rule 1 (minor — verification check compliance).

## User Setup Required

None — no external service configuration required.

## Next Phase Readiness

- Phase 5 Wave-2 is now complete: 05-02 (Ch.15), 05-03 (Ch.17), 05-04 (Ch.18) all done
- `mdbook build` exit 0 with all four chapters (Ch.14–15, Ch.17–18)
- Phase 6 (레퍼런스 부록) can start: Ch.18 마무리 section includes plain-prose forward reference
- No blockers. The three factual corrections for Ch.18 are now accurately documented.

---
*Phase: 05-platforms-operations*
*Completed: 2026-06-11*
