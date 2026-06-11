# Project State

## Project Reference

See: .planning/PROJECT.md (updated 2026-06-11)

**Core value:** 개발자가 따라 하면 Hermes Agent를 설치·실행하고, 스킬·게이트웨이·배포까지 실제로 동작시킬 수 있다. 모든 명령어·API는 공식 소스 근거 필수.
**Current focus:** v1 SHIPPED — 다음 마일스톤 계획 대기 (`/gsd:new-milestone`)

## Current Position

Milestone: v1 MVP — **COMPLETE & SHIPPED (2026-06-11)**
Phase: — (all 6 phases archived)
Status: Ready to plan next milestone
Last activity: 2026-06-11 — v1 milestone archived & tagged (milestone-v1)

Progress: [██████████] 100% — 6/6 phases, 24/24 plans, 20/20 requirements

## Milestone Record

**v1 MVP:** 21-chapter Korean mdBook tutorial for Hermes Agent, live at https://ohama.github.io/Hermes-Tutorial/.
- Archived: `.planning/milestones/v1-ROADMAP.md`, `v1-REQUIREMENTS.md`, `v1-MILESTONE-AUDIT.md`, `v1-phases/`
- Stats: 6 phases, 24 plans, ~5,610 lines, 82 commits, 7 days
- Full record: `.planning/MILESTONES.md`

## Accumulated Context

### Key facts carried forward

- Repo: `ohama/Hermes-Tutorial`; remote origin set; push to main auto-deploys via GitHub Actions (`deploy-pages@v5`). Live: https://ohama.github.io/Hermes-Tutorial/
- mdBook 0.5.3; `book.toml` site-url=`/Hermes-Tutorial/`, language=`ko`. SUMMARY.md = 21 chapter links across 5 sections.
- Accuracy corrections established in v1 (apply consistently in v2): `hermes doctor`=공급망 CVE 어드바이저리(설정 검증은 `hermes config check`); `hermes security audit` 미존재; `agent.disabled_toolsets` 미존재; PII=`security.redact_secrets` 한정; cron CLI=`hermes cron`(not gateway); MCP/ACP 별개; `display.language: ko` 부분 지원.
- Pattern: SUMMARY.md owner-plan (Wave 1 owns SUMMARY + writes 1 chapter; Wave 2 body-only parallel). Verify greps must include file-path args.

### Blockers/Concerns (carried to next milestone)

- **[Node.js 20 deprecation — LOW, 2026-06-16]** GitHub forces Node 24 on Actions from 2026-06-16; pinned actions (checkout@v4/configure-pages@v5/upload-artifact@v4) keep working. Update action versions when convenient.
- **[Local-capture placeholders — tech debt]** Several outputs marked `[로컬 실행 후 캡처 필요]` need a live Hermes run to replace with real captures (hermes doctor, gateway setup wizard, welcome banner, dashboard, cron run independence, mcp serve tool list). All deliberately hedged, not fabricated.

## Session Continuity

Last session: 2026-06-11 — v1 milestone complete, archived, tagged (milestone-v1), audit fixes deployed.
Resume file: None
Next workflow trigger: `/gsd:new-milestone` (questioning → research → requirements → roadmap for v2)
