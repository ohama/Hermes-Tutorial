---
phase: 04-learning-automation
plan: "03"
subsystem: content
tags: [mdbook, mcp, hermes, bidirectional, mcp-client, mcp-server, acp]

# Dependency graph
requires:
  - phase: 04-01
    provides: SUMMARY.md Wave-1 owner; src/11-mcp/index.md skeleton placeholder created
provides:
  - Ch.11 MCP 연동 본문 (src/11-mcp/index.md) — 양방향 MCP(클라이언트+서버), ACP 구분, include 우선순위, 보안
affects:
  - 04-04-PLAN (Ch.12 크론 스케줄러 — 이전 챕터 링크가 Ch.11을 참조)
  - 04-05-PLAN (Ch.13 서브에이전트 — 동일 wave)

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "검증 안 된 내용은 placeholder + 검증 필요 callout으로 처리 (발명 금지 패턴)"
    - "Phase 4 코드블록 주석 관례 # 검증: hermes rolling, 2026-06-10"
    - "JSON 블록은 // 검증: ... 형식 사용"

key-files:
  created: []
  modified:
    - "src/11-mcp/index.md — Ch.11 MCP 연동 본문 274줄 (placeholder 1줄 → 274줄)"

key-decisions:
  - "hermes mcp serve를 '~10개 플랫폼 메시징 툴을 노출하는 stdio MCP 서버'로 정확히 기술 — 전체 에이전트 릴레이 단언 없음"
  - "ACP vs MCP 별개 통합 구분 표 포함 — ACP는 간단 언급, 상세는 본 튜토리얼 범위 밖 처리"
  - "tools.include 우선순위 명시: include + exclude 동시 설정 시 include 우선, exclude 무시"
  - "에디터(Cursor/VS Code/Zed) MCP 설정 파일 경로 미검증 → placeholder 코드블록 + 검증 필요 callout"
  - "정확한 10개 툴 이름 미검증 → 검증 필요 callout으로 처리"
  - "Task 2(빌드 검증)는 파일 변경 없음 — no-op 커밋 생략, mdbook build exit 0만 확인"

patterns-established:
  - "Phase 4 Wave-2 패턴: Wave-1이 만든 placeholder를 overwrite하여 본문 작성"
  - "ACP/MCP 혼동 방지 패턴: 챕터 상단에 ACP vs MCP 비교 표 배치"

# Metrics
duration: 2min
completed: 2026-06-10
---

# Phase 4 Plan 03: Ch.11 MCP 연동 Summary

**양방향 MCP 연동 챕터 — Hermes를 MCP 클라이언트(외부 서버 연결)와 MCP 서버(hermes mcp serve로 ~10개 메시징 툴 노출)로 모두 다루며, ACP 별개 통합 구분 표·include 우선순위·보안 callout 포함**

## Performance

- **Duration:** ~2 min
- **Started:** 2026-06-10T08:42:41Z
- **Completed:** 2026-06-10T08:44:28Z
- **Tasks:** 2 (Task 1: 본문 작성, Task 2: 빌드 검증)
- **Files modified:** 1

## Accomplishments

- `src/11-mcp/index.md` 274줄 본문 작성 (placeholder 1줄 → 274줄)
- 양방향 MCP 정확하게 기술: (a) Hermes를 클라이언트로 외부 MCP 서버 연결, (b) `hermes mcp serve`로 MCP 서버 노출
- ACP vs MCP 비교 표로 혼동 방지; `hermes mcp serve`를 전체 에이전트 릴레이로 단언하지 않음
- `tools.include` 우선순위 규칙 명시, 화이트리스트 보안 권장
- 미검증 내용(정확한 툴 이름, 에디터 설정 경로) → placeholder + 검증 필요 callout
- `mdbook build` exit 0 확인; `src/SUMMARY.md` 미변경

## Task Commits

1. **Task 1: Ch.11 MCP 연동 본문 작성** - `de0808e` (feat)
2. **Task 2: 빌드 검증** - no-op (파일 변경 없음, 커밋 생략)

**Plan metadata:** (docs commit — see below)

## Files Created/Modified

- `src/11-mcp/index.md` — Ch.11 MCP 연동 본문 274줄: 개요/ACP 구분/모드 1(클라이언트)/실습/모드 2(서버)/심화 config 레퍼런스/흔한 오류/다음 단계

## Decisions Made

- `hermes mcp serve` = ~10개 플랫폼 메시징 툴(Telegram, Discord, Slack 등) stdio 서버; 전체 에이전트 릴레이 단언 없음 — RESEARCH.md HIGH confidence 준수
- ACP(`hermes acp`) = 에디터 네이티브 임베딩 (별개 통합); 이 챕터는 MCP만 다루며 ACP는 간단 언급
- 에디터(Cursor/VS Code/Zed) 설정 경로 미검증 → `# [로컬 실행 후 캡처 필요]` placeholder + `> 검증 필요` callout
- Task 2는 빌드만 확인(파일 변경 없음) — no-op 커밋 생략 (03-03과 동일 패턴)
- JSON 코드블록은 `// 검증: hermes rolling, 2026-06-10` 형식 사용(셸 주석 `#`와 구분)

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness

- Ch.11 본문 완성; Ch.12(크론 스케줄러) 04-04-PLAN이 이전 챕터 링크 `../11-mcp/index.md` 사용 가능
- Wave-2 병렬 플랜(04-02, 04-04, 04-05) 모두 독립적으로 진행 가능

---
*Phase: 04-learning-automation*
*Completed: 2026-06-10*
