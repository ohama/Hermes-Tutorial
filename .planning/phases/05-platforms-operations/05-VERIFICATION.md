---
phase: 05-platforms-operations
verified: 2026-06-11T00:00:00Z
status: passed
score: 18/18 must-haves verified
re_verification: false
---

# Phase 5: Platforms & Operations Verification Report

**Phase Goal:** 독자가 메시징 게이트웨이 봇을 설정하고 프로덕션 배포와 보안 하드닝을 완료할 수 있다 (Ch.14–18).
**Verified:** 2026-06-11
**Status:** PASSED
**Re-verification:** No — initial verification

---

## Goal Achievement

### Observable Truths

| # | Truth | Status | Evidence |
|---|-------|--------|----------|
| 1 | SUMMARY.md에 "# 플랫폼과 운영" 섹션 + 4개 항목 추가, 기존 14개 항목 보존 | VERIFIED | SUMMARY.md 36행에 섹션 존재, 총 18개 index.md 링크 |
| 2 | SUMMARY.md 총 18개 링크, 중복 경로 없음, 16-* 항목 없음 | VERIFIED | `grep -c "index.md"` = 18, uniq -d 결과 빈값, no 16-* |
| 3 | Ch.14 Telegram 게이트웨이 본문 — BotFather→.env→hermes gateway run→대화 흐름 (PLAT-01 #1) | VERIFIED | 221행, BotFather 7회, TELEGRAM_BOT_TOKEN 3회, hermes gateway run 3회 |
| 4 | Ch.14가 24개 플랫폼으로 정확히 기술 (22+ 아님) | VERIFIED | "24개 플랫폼 어댑터" 11행, 22+는 전혀 없음 |
| 5 | Ch.14 hermes gateway / hermes gateway run = 포그라운드; gateway start = 서비스용 구분 명확 | VERIFIED | 47행 callout: "install 없이 start를 실행하면 오류가 발생합니다" |
| 6 | Ch.14 폴링 중복 함정 경고 | VERIFIED | 188행: "두 개의 Hermes 프로세스 또는 두 프로파일에서 동시에 사용하면" |
| 7 | Ch.14 토큰 예시 placeholder 전용, .env 저장·Git 미커밋 callout | VERIFIED | `<YOUR_BOT_TOKEN>` 78행; Git 노출 금지 67행·207행 |
| 8 | Ch.15 Discord 2 Privileged Intents = Server Members + Message Content (NOT Presence) | VERIFIED | 27-30행 표: Server Members, Message Content, "Presence Intent는 불필요" |
| 9 | Ch.15 Message Content Intent 미활성화 → 텍스트 비어 #1 무반응 원인 함정 callout | VERIFIED | 28행 "→ #1 무반응 원인", 313-321행 경고 callout |
| 10 | Ch.15 Discord DISCORD_BOT_TOKEN, Slack xoxb/xapp Socket Mode 정확 | VERIFIED | DISCORD_BOT_TOKEN 85행; Socket Mode 133행; xoxb/xapp 162-163행 |
| 11 | Ch.15 WhatsApp/Signal/Email 개요 수준 + MEDIUM 신뢰 명시 | VERIFIED | signal-cli MEDIUM 212행; WhatsApp/Email 개요 존재 |
| 12 | Ch.17 Docker 배포 — nousresearch/hermes-agent, gateway run, restart unless-stopped, 8642/9119, UID 10000, s6-overlay | VERIFIED | 모두 확인: 18행·44행·49행·51행·80행·103-104행 |
| 13 | Ch.17 VPS systemd — hermes gateway install/start, --system 헤지, journalctl | VERIFIED | 147행·155행(헤지)·202행 |
| 14 | Ch.17 Modal/Daytona 핵심 키 + managed mode 헤지 | VERIFIED | modal_image 250행, 검증 필요 270행; DAYTONA_API_KEY 294행 |
| 15 | Ch.17 두 컨테이너 ~/.hermes 공유 금지 함정 + docker_run_as_host_user 함정 | VERIFIED | 345행 docker_run_as_host_user; 웹 콘솔 금지 40행 |
| 16 | Ch.18 approvals.mode manual 권장, off 위험 경고, .env 키 관리, 운영 런북 (PLAT-03 #4) | VERIFIED | 31행 manual, 46행 off 경고, 56행 "manual 유지", 76행 chmod 600 |
| 17 | Ch.18 정정 3개 — doctor=공급망CVE(설정검증 아님), security audit 부재, PII=redact_secrets 범위 한정 | VERIFIED | 157-162행(doctor 정정), 224-226행(audit 부재), 139-149행(PII 정정) |
| 18 | Ch.18 Phase 6 전방 참조 plain prose, 06-* mdBook 링크 없음 | VERIFIED | `grep -E '\]\(\.\./06-'` = 결과 없음 |

**Score:** 18/18 truths verified

---

## Required Artifacts

| Artifact | Min Lines | Actual Lines | Status | Key Contents |
|----------|-----------|--------------|--------|--------------|
| `src/SUMMARY.md` | — | 41 | VERIFIED | 18 index.md links, 4 sections, no duplicates, no 16-* |
| `src/14-gateways/index.md` | 120 | 221 | VERIFIED | TELEGRAM_BOT_TOKEN, hermes gateway run, 24 platforms, BotFather, polling pitfall |
| `src/15-more-gateways/index.md` | 140 | 338 | VERIFIED | Message Content Intent, Server Members Intent, Presence NOT needed, xoxb/xapp, Socket Mode, signal-cli, WhatsApp, Email |
| `src/17-deploy/index.md` | 140 | 353 | VERIFIED | nousresearch/hermes-agent, s6-overlay, UID 10000, 8642/9119, install/start/--system, SSH/Modal/Daytona backends |
| `src/18-security/index.md` | 150 | 327 | VERIFIED | approvals.mode, chmod 600, #16394, redact_secrets, doctor=supply chain, security audit absent, no 06-* links |

---

## Key Link Verification

| From | To | Via | Status |
|------|----|-----|--------|
| SUMMARY.md | 14-gateways/index.md | 플랫폼과 운영 섹션 | WIRED |
| SUMMARY.md | 15-more-gateways/index.md | 플랫폼과 운영 섹션 | WIRED |
| SUMMARY.md | 17-deploy/index.md | 플랫폼과 운영 섹션 | WIRED |
| SUMMARY.md | 18-security/index.md | 플랫폼과 운영 섹션 | WIRED |
| Ch.14 | Ch.15 | ../15-more-gateways/index.md 다음 단계 링크 221행 | WIRED |
| Ch.15 | Ch.14 / Ch.17 | 이전/다음 챕터 링크 337-338행 | WIRED |
| Ch.17 | Ch.15 / Ch.18 | 이전/다음 챕터 링크 351·353행 | WIRED |
| Ch.18 | Ch.17 | 이전 챕터 링크 6행·325행 | WIRED |

---

## Requirements Coverage

| Requirement | Success Criteria | Status | Evidence |
|-------------|-----------------|--------|----------|
| PLAT-01 | Ch.14 Telegram 봇 설정 + 대화 (Discord Intent 함정) | SATISFIED | Ch.14 221행 본문 완성; Ch.15 Discord 2-Intents 함정 정확 |
| PLAT-01 | Ch.15–16: Discord/Slack/WhatsApp/Signal/Email 추가 설정 | SATISFIED | Ch.15 338행, 5개 플랫폼 모두 커버 |
| PLAT-02 | Ch.17: 하나의 시나리오로 프로덕션 배포 | SATISFIED | Docker(시나리오A) + VPS(B) + SSH(C) + Modal(D) + Daytona(E) |
| PLAT-03 | Ch.18: 승인 모드 + .env 키 관리 + 운영 런북 | SATISFIED | approvals.mode manual, chmod 600, doctor/config check/status 런북 |

---

## Documentation Accuracy Checks

### 검증 Comment Convention

| Chapter | Opening Fences | 검증 Comments | Assessment |
|---------|---------------|---------------|------------|
| Ch.14 | 8 | 8 (line 1) | PASS — every block opens with # 검증: |
| Ch.15 | 12 | 16 | PASS — all 12 code blocks covered; extra 검증 in prose callouts |
| Ch.17 | 14 | 14 | PASS* — 7 blocks use filename comment on line 1, 검증 on line 2; all blocks have 검증 within first 2 lines |
| Ch.18 | 7 | 8 | PASS — all blocks covered |

*Ch.17 note: 7 code blocks place a filename label (`# docker-compose.yml`, `# ~/.hermes/.env 에 추가`, etc.) on line 1 and `# 검증:` on line 2. The 검증 tag is present in every block. This is a minor stylistic variation but does not violate the intent.

### Corrected Fact Honoring (Ch.18)

| Fact | Required | Status |
|------|----------|--------|
| `hermes doctor` = supply-chain/CVE advisory, NOT config validation | 157행: "공급망 어드바이저리 검사기", "설정 검증 도구가 아닙니다" | PASS |
| config validation = `hermes config check` (separate) | 162행: "설정 검증은 별도 명령 hermes config check" | PASS |
| `hermes security audit` absent as usable command | 224-237행: 존재하지 않음 정정 callout; no runnable code line | PASS |
| PII redaction scoped to `security.redact_secrets` only | 139-149행 PII 정정 callout; 294행 흔한오류 재확인 | PASS |

### Discord Intents (Ch.15)

| Fact | Required | Status |
|------|----------|--------|
| TWO Privileged Intents = Server Members + Message Content | 27행 표에 정확히 기술 | PASS |
| Presence Intent NOT needed | 30행: "Presence Intent는 Hermes에 불필요" | PASS |
| Without Message Content Intent → empty message text → #1 무반응 | 28행 "→ #1 무반응 원인", 313행 경고 callout | PASS |

### Gateway Command Framing (Ch.14, Ch.15, Ch.17)

- `hermes gateway` = foreground: confirmed in Ch.14 (43-47행), Ch.15 (개요), Ch.17 (19행 table)
- `hermes gateway run` = Docker main process: confirmed Ch.14 (44행), Ch.17 (52행)
- `hermes gateway start` = service after install only: Ch.14 (47행 callout explicitly warns), Ch.17 (147-152행)
- No chapter uses `gateway start` as foreground start command.

### Platform Count

- Ch.14 line 11: "24개 플랫폼 어댑터" — correct.
- "22+" does not appear anywhere in any chapter file.

### Token Security

| File | Real Telegram token (digits:alnum) | Real sk-ant- key | Real sk-or- key |
|------|-------------------------------------|------------------|-----------------|
| Ch.14 | None | None | None |
| Ch.15 | None | None | None |
| Ch.17 | None | None | None |
| Ch.18 | None | None | None |

All token examples use explicit placeholders: `<YOUR_BOT_TOKEN>`, `<YOUR_DISCORD_BOT_TOKEN>`, `xoxb-<YOUR_BOT_TOKEN>`, `xapp-<YOUR_APP_TOKEN>`, `<YOUR_ANTHROPIC_API_KEY>`, `<YOUR_DAYTONA_API_KEY>`.

### LOW/MEDIUM Items — Placeholder Routing

| Item | Chapter | Status |
|------|---------|--------|
| gateway setup 마법사 UI | Ch.14 | `[로컬 실행 후 캡처 필요]` + `> 검증 필요` callout |
| 게이트웨이 시작 메시지 출력 | Ch.14 | `[로컬 실행 후 캡처 필요]` + `> 검증 필요` callout |
| Slack manifest 내용 | Ch.15 | `> 검증 필요` callout |
| hermes gateway install --system 경로 | Ch.17 | `> 검증 필요` callout (155행) |
| Modal managed mode 흐름 | Ch.17 | `> 검증 필요` callout (270행) |
| doctor 출력 형식 | Ch.18 | `[로컬 실행 후 캡처 필요]` + `> 검증 필요` callout (191-194행) |

### Phase 6 Forward References (Ch.18)

- `grep -E '\]\(\.\./06-'` returns no matches.
- Closing prose uses plain text: "CLI 명령어·슬래시 커맨드·설정 키의 전체 목록은 이후 레퍼런스 부록에서 정리됩니다." (no dead link).

---

## Build Verification

```
mdbook build
INFO Book building has started
INFO Running the html backend
INFO HTML book written to /Users/ohama/projs/hermes-tutorial/book
EXIT_CODE: 0
```

- Exit code: 0
- Errors in output: none
- Warnings/unknowns in output: none
- Built HTML pages: book/14-gateways/index.html, book/15-more-gateways/index.html, book/17-deploy/index.html, book/18-security/index.html — all exist.

---

## Anti-Patterns Found

| File | Pattern | Severity | Assessment |
|------|---------|----------|------------|
| None | — | — | No placeholder stubs, no TODO/FIXME, no empty implementations found in any chapter |

Ch.15/17/18: original `# placeholder` skeleton content fully overwritten with real content (verified by `! grep -qx "# placeholder"` passing for each).

---

## Human Verification Required

The following items are marked `[로컬 실행 후 캡처 필요]` in the chapters and require local execution to confirm exact output format:

### 1. hermes gateway setup 마법사 UI (Ch.14, Ch.15)

**Test:** Run `hermes gateway setup` on a machine with Hermes installed
**Expected:** Interactive menu-driven wizard appears for configuring gateway platforms
**Why human:** Exact menu text and selection options depend on version/environment; not verifiable from source

### 2. 게이트웨이 시작 메시지 출력 (Ch.14)

**Test:** Run `hermes gateway` on a machine with TELEGRAM_BOT_TOKEN configured
**Expected:** Gateway start log messages appear; bot responds to Telegram DM within seconds
**Why human:** Exact log format depends on runtime environment

### 3. hermes doctor 정상 실행 출력 (Ch.18)

**Test:** Run `hermes doctor` in an environment with no known CVE advisories
**Expected:** Clean output confirming no supply-chain advisories
**Why human:** Exact output format not documented; marked with 검증 필요 callout

These are informational gaps only — the chapter prose accurately describes the expected behavior and all three are marked with `> 검증 필요` callouts per the plan requirements.

---

## Summary

Phase 5 achieves its goal. All four chapters are substantive, accurately reflect the corrected facts from the research phase, contain no real tokens, and build cleanly with mdbook. The three key accuracy corrections (hermes doctor scope, hermes security audit absence, PII redact_secrets scope) are all present and correctly framed. Discord's two required Privileged Intents (Server Members + Message Content, not Presence) are precisely documented with the #1 무반응 pitfall callout. All LOW/MEDIUM confidence items are routed to `[로컬 실행 후 캡처 필요]` placeholders rather than fabricated. SUMMARY.md has exactly 18 chapter links, no duplicates, no 16-* entry.

---

_Verified: 2026-06-11_
_Verifier: Claude (gsd-verifier)_
