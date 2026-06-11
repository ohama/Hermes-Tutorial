---
phase: 05-platforms-operations
plan: 02
subsystem: content
tags: [mdbook, discord, slack, whatsapp, signal, email, gateway, hermes, korean, tutorial]

requires:
  - phase: 05-01
    provides: "src/15-more-gateways/index.md placeholder 스켈레톤; SUMMARY.md 링크(15-more-gateways) 확립; 검증 주석 관례 '# 검증: hermes rolling, 2026-06-11'"

provides:
  - "src/15-more-gateways/index.md: Ch.15 추가 게이트웨이 본문 338줄 (Discord 2-Intents + Slack Socket Mode + WhatsApp/Signal/Email 개요 + DM 페어링 + 함정 callout)"
  - "Discord: Server Members Intent + Message Content Intent 필수 2개 정확 기술; Presence Intent 불필요 명시; permission 274878286912; DISCORD_BOT_TOKEN placeholder"
  - "Slack: Socket Mode 필수(WebSocket, NAT 뒤 동작); SLACK_BOT_TOKEN(xoxb) + SLACK_APP_TOKEN(xapp) placeholder; OAuth Scopes 목록"
  - "WhatsApp/Signal/Email: MEDIUM/HIGH 신뢰도 명시 개요; .env 변수명 정확 기술; 비공식 프로토콜 주의 callout"
  - "공통: hermes gateway setup + hermes gateway + hermes pairing list/approve/revoke"
  - "흔한 오류: Discord Message Content Intent 비활성화 #1 무반응 원인 callout; 재시작 필수 절차 명시"
  - "다음 단계: 17-deploy/index.md + 14-gateways/index.md 인라인 링크"

affects:
  - 05-03-PLAN (Ch.17 프로덕션 배포 — 17-deploy/index.md 본문 overwrite)
  - 05-04-PLAN (Ch.18 보안 하드닝 — 18-security/index.md 본문 overwrite)

tech-stack:
  added: []
  patterns:
    - "MEDIUM 신뢰도 섹션: 개요 수준 기술 + 신뢰도 명시 callout (WhatsApp/Signal 패턴)"
    - "Message Content Intent 경고: 가장 중요한 Discord 함정을 챕터 상단 미리읽기 callout + 하단 전용 오류 섹션 두 곳에 배치"
    - "토큰 placeholder 패턴: <YOUR_DISCORD_BOT_TOKEN>, xoxb-<YOUR_BOT_TOKEN>, xapp-<YOUR_APP_TOKEN> — 실제처럼 보이는 토큰 절대 금지 (05-01 패턴 유지)"
    - "검증 필요 callout: gateway setup 마법사 UI, slack manifest JSON 내용 등 LOW/MEDIUM 출력"

key-files:
  created: []
  modified:
    - src/15-more-gateways/index.md

key-decisions:
  - "Discord Message Content Intent를 챕터 상단 미리읽기 callout과 하단 전용 오류 섹션 두 곳에 배치 — #1 무반응 원인이므로 중복 배치로 강조"
  - "WhatsApp/Signal MEDIUM 신뢰도를 섹션 헤더 직후 blockquote로 명시 — 개요 수준이며 깊은 보장 없음을 독자에게 선제적으로 전달"
  - "hermes gateway setup 마법사 UI를 검증 필요 callout으로 처리 — Open Question #2 미해결 (05-RESEARCH.md 기재)"
  - "Slack manifest JSON 내용을 검증 필요 callout으로 처리 — Open Question #8 미해결"
  - "Task 2 빌드 검증은 파일 변경 없음 → no-op 커밋 생략 (03-03/04-03/04-04/04-05/05-01 동일 패턴)"

patterns-established:
  - "두 섹션 배치 패턴: 중요 경고는 챕터 상단(미리읽기)과 하단(전용 오류 섹션) 두 곳 배치"
  - "신뢰도 명시 패턴: MEDIUM confidence 섹션은 헤더 바로 아래 blockquote로 신뢰도 및 개요 수준임을 명시"

duration: 2min
completed: 2026-06-11
---

# Phase 5 Plan 02: Ch.15 추가 게이트웨이 Summary

**Discord 2-Intents(Server Members + Message Content; Presence 불필요) + Slack Socket Mode + WhatsApp/Signal/Email 개요 + DM 페어링 + #1 무반응 함정 callout을 담은 Ch.15 본문 338줄 완성; mdbook build exit 0**

## Performance

- **Duration:** ~2 min
- **Started:** 2026-06-11T00:42:46Z
- **Completed:** 2026-06-11T00:45:18Z
- **Tasks:** 2
- **Files modified:** 1

## Accomplishments

- `src/15-more-gateways/index.md` 338줄 작성: Discord 앱 생성 6단계 + Privileged Gateway Intents 필수 2개(Server Members + Message Content, Presence 불필요) + 봇 초대 OAuth(permission 274878286912) + Slack Socket Mode + xoxb/xapp placeholder + WhatsApp/Signal/Email 개요 + hermes pairing + Message Content Intent #1 무반응 함정 callout
- 모든 토큰 예시 placeholder(`<YOUR_DISCORD_BOT_TOKEN>`, `xoxb-<YOUR_BOT_TOKEN>`, `xapp-<YOUR_APP_TOKEN>`) — 실제처럼 보이는 토큰 없음
- SUMMARY.md 미수정 확인; `mdbook build` exit 0 확인; 17-deploy/14-gateways 인라인 링크 포함

## Task Commits

1. **Task 1: Ch.15 Discord + Slack 본문 작성** - `5ef8ecd` (feat)
2. **Task 2: WhatsApp/Signal/Email 개요 + 공통 설정 + 함정 + 다음 단계** - `be19fa6` (feat)

**Plan metadata:** (docs commit below)

## Files Created/Modified

- `src/15-more-gateways/index.md` — Ch.15 추가 게이트웨이 본문 338줄: Discord 2-Intents + Slack Socket Mode + WhatsApp/Signal/Email 개요 + DM 페어링 + 함정 callout + 다음 단계 링크

## Decisions Made

- **Discord Message Content Intent 이중 배치:** 챕터 상단 미리읽기 callout과 하단 전용 오류 섹션 두 곳에 배치 — #1 무반응 원인이므로 중복 배치로 강조
- **MEDIUM 신뢰도 명시 패턴:** WhatsApp/Signal 섹션 헤더 직후 blockquote로 "신뢰도: MEDIUM, 개요 수준" 명시 — 독자 선제 전달
- **gateway setup 마법사 → 검증 필요 callout:** Open Question #2 미해결, 정확한 UI 텍스트 발명 금지
- **Slack manifest JSON → 검증 필요 callout:** Open Question #8 미해결
- **Task 2 빌드 검증 no-op 생략:** 파일 변경 없음 — 03-03/04-03/04-04/04-05/05-01 동일 패턴 계속 적용

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness

- **05-03 (Ch.17 프로덕션 배포)**: `src/17-deploy/index.md` placeholder 존재; SUMMARY.md 링크 확립; Wave-2 병렬 실행 가능
- **05-04 (Ch.18 보안 하드닝)**: `src/18-security/index.md` placeholder 존재
- **mdbook build exit 0** 확인 완료; 후속 플랜이 스켈레톤을 본문으로 덮어써도 빌드 유지
- **Discord Open Flag:** Discord Developer Portal UI 변경 가능성 (STATE.md 기존 우려사항) — 현재 기술 내용은 RESEARCH.md HIGH confidence이나 실제 포털 확인 권장

---
*Phase: 05-platforms-operations*
*Completed: 2026-06-11*
