---
phase: 06-reference
plan: 02
subsystem: docs
tags: [mdbook, config.yaml, hermes, reference, korean]

# Dependency graph
requires:
  - phase: 06-01
    provides: src/20-config-reference/index.md placeholder skeleton + SUMMARY.md 레퍼런스 섹션
provides:
  - Ch.20 config.yaml 레퍼런스 본문 (568줄) — display/model/agent/memory/compression/toolsets/approvals/security/terminal/delegation/mcp_servers/skills/cron/curator 영역별 키 용도·기본값·허용값
  - approvals.timeout/cron_mode 데이터 충돌 헤지 (양쪽 값 + hermes config show 로컬 확인)
  - agent.disabled_toolsets 미존재 키 표 (대안: toolsets 배열/hermes tools UI)
  - security.redact_secrets 정확한 범위 (MCP 오류 자격증명 패턴만, 일반 PII 아님)
affects: [06-03, phase-completion]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "approvals 데이터 충돌 → 양쪽 값 제시 + hermes config show 헤지 패턴"
    - "미존재 키 표 (혼동 방지) 패턴 — 06-01 Ch.19에서 도입된 정정 callout과 동일 계열"
    - "# 검증: hermes rolling, YYYY-MM-DD 코드블록 주석 관례 (Phase 5에서 계속)"

key-files:
  created: []
  modified:
    - src/20-config-reference/index.md

key-decisions:
  - "approvals.timeout 기본값(60 vs 300)과 cron_mode 값(deny|approve vs manual|auto)을 어느 한쪽으로도 단정하지 않고 양쪽 제시 + hermes config show 로컬 확인 헤지 적용"
  - "agent.disabled_toolsets는 미존재 키 표에서만 언급 — 사용 가능한 키 목록에 절대 포함 안 함 (toolsets 배열/hermes tools UI가 대안)"
  - "security.redact_secrets = MCP 오류 메시지의 ghp_/sk-/Bearer 자격증명 패턴 난독화만 (일반 PII 필터 아님, 별도 PII 리댁션 키 없음)"
  - "Task 2 빌드 검증은 소스 변경 없으므로 no-op 커밋 생략 (03-03/04-03 패턴 재적용)"

patterns-established:
  - "데이터 충돌 헤지 패턴: 두 연구 파일 간 불일치 시 양쪽 값 YAML 인라인 주석 + > 검증 필요 callout + hermes config show 안내"

# Metrics
duration: 5min
completed: 2026-06-11
---

# Phase 06 Plan 02: Ch.20 config.yaml 레퍼런스 Summary

**config.yaml 14개 영역 키 레퍼런스 568줄 — approvals 데이터 충돌 헤지, disabled_toolsets/redact_secrets 정정, 모든 코드블록 검증 주석 포함**

## Performance

- **Duration:** 5 min
- **Started:** 2026-06-11T01:21:08Z
- **Completed:** 2026-06-11T01:26:32Z
- **Tasks:** 2
- **Files modified:** 1

## Accomplishments

- src/20-config-reference/index.md 플레이스홀더를 568줄 본문으로 overwrite — display/model/agent/memory/compression/toolsets/approvals/security/terminal/delegation/mcp_servers/skills/cron/curator 14개 영역 모두 수록
- approvals.timeout(60 vs 300)·cron_mode(deny|approve vs manual|auto) 실제 데이터 충돌을 YAML 인라인 주석 + `> 검증 필요` callout으로 헤지 (어느 한쪽도 단정 없음)
- agent.disabled_toolsets 미존재 키 표에서만 언급; security.redact_secrets 정확한 범위(MCP 오류 자격증명 패턴만) 기술; mdbook build exit 0; SUMMARY.md 미수정 확인

## Task Commits

Each task was committed atomically:

1. **Task 1: Ch.20 config.yaml 레퍼런스 본문 작성** - `a2ecafc` (feat)
2. **Task 2: 빌드 검증 + SUMMARY.md 미수정 확인** - no-op 커밋 생략 (소스 변경 없음, 03-03 패턴)

**Plan metadata:** (this commit — docs(06-02))

## Files Created/Modified

- `src/20-config-reference/index.md` — Ch.20 config.yaml 키 레퍼런스 본문 568줄 (display/model/agent/memory/compression/toolsets/approvals/security/terminal/delegation/mcp_servers/skills/cron/curator + .env 환경변수 표 + 미존재 키 표 + 발행 전 확인 항목)

## Decisions Made

- approvals.timeout 기본값(60 vs 300)과 cron_mode 값(deny|approve vs manual|auto)을 어느 한쪽으로도 단정하지 않고 YAML 인라인 주석 + `> 검증 필요` callout으로 양쪽 제시 — 두 연구 파일 간 실제 불일치이므로 로컬 확인(`hermes config show`) 안내
- agent.disabled_toolsets는 미존재 키 표에서만 언급; 사용 가능한 키 목록에 포함하지 않음 (RESEARCH.md 03 확인)
- security.redact_secrets = MCP 오류 메시지의 자격증명 패턴(ghp_/sk-/Bearer) 난독화만; 일반 PII 필터 아님; 별도 PII 리댁션 키 없음 (05-RESEARCH.md 확인)
- Task 2 빌드 검증에서 소스 변경 없으므로 no-op 커밋 생략 (03-03/04-03/04-04/04-05/05-01/06-01 패턴 재적용)

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness

- Ch.20 config.yaml 레퍼런스 완료 — Phase 6 남은 작업: 06-03 (Ch.21 마스터 트러블슈팅 색인)
- mdbook build exit 0 확인됨
- src/SUMMARY.md 미수정 확인 (06-01 단독 소유 패턴 유지)

---
*Phase: 06-reference*
*Completed: 2026-06-11*
