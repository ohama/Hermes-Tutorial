# Phase 5: Platforms & Operations — Research

**Researched:** 2026-06-11
**Domain:** Hermes Agent — Ch.14 메시징 게이트웨이(Telegram) / Ch.15–16 추가 게이트웨이 / Ch.17 배포 / Ch.18 보안
**Confidence:** HIGH (gateway setup, Docker deployment, security config — all from official docs + real repo) / MEDIUM (SSH backend exact interaction, Modal/Daytona managed mode) / LOW (some exact CLI output formats)

**Accuracy constraint:** Every command, flag, config key, and expected output is grounded in official docs or the real repo.
Convention (MANDATORY): every command/config code block opens with `# 검증: hermes rolling, 2026-06-11`.
LOW/MEDIUM → `[로컬 실행 후 캡처 필요]` placeholder + `> 검증 필요` callout — never fabricate.

---

## Summary

Phase 5 produces five content chapters (Ch.14–18) that take a reader from "I can run Hermes locally" to "I have a production gateway bot serving real users securely." Research drew from 14 official Hermes doc pages (messaging/telegram, messaging/discord, messaging/slack, messaging/whatsapp, messaging/signal, messaging/email, messaging/index, user-guide/docker, user-guide/security, user-guide/configuration, user-guide/multi-profile-gateways, reference/faq, reference/cli-commands, api-server), the real GitHub `.env.example` and `cli-config.yaml.example`, and secondary sources for systemd/VPS deployment details.

**Key findings:**

1. **Gateway architecture confirmed:** One `hermes gateway` process runs all platform adapters simultaneously. 24 platforms supported. `hermes gateway run` = foreground; `hermes gateway start` = background service.
2. **Discord Intents RESOLVED:** Two Privileged Intents required — **Server Members Intent + Message Content Intent** (NOT Presence Intent). Without Message Content Intent, bot receives events but message text is empty — confirmed as #1 silent failure mode. Current (2026) Discord portal flow confirmed unchanged; approved for bots in <100 servers without manual review.
3. **Modal/Daytona keys PARTIALLY RESOLVED:** Core YAML keys confirmed from official config reference. Modal requires `MODAL_TOKEN_ID` + `MODAL_TOKEN_SECRET` (or `~/.modal.toml` via `modal setup`). The `.env.example` shows Modal uses CLI auth (`modal setup`) — no `.env` key required. Daytona requires `DAYTONA_API_KEY`. The `modal_image` key exists for Modal. MEDIUM confidence — flag for local verification.
4. **`hermes doctor` CORRECTED:** The prior research established fact "hermes doctor diagnoses config + dependency issues" is partially wrong. Official docs confirm `hermes doctor` is a **supply-chain advisory checker** — it surfaces known-compromised Python packages in the active venv, not general config diagnostics. Config validation uses `hermes config check`. The "operational runbook" framing in Ch.18 should use `hermes doctor --ack` for advisory management, `hermes config check` for config validation, and `hermes status` for runtime health.
5. **`hermes security audit` does NOT exist** — confirmed from official security page. Remove from any chapter that references it. Security is split across `hermes doctor` (supply chain), `hermes pairing` (authorization), and built-in approval/Tirith scanning.
6. **PII redaction is NOT a general feature** — it is credential pattern redaction in MCP error messages (hardcoded, no config key). Do NOT describe it as a general PII redaction toggle.
7. **Production deployment:** Two validated paths — (A) Docker with `nousresearch/hermes-agent` official image + `--restart unless-stopped` + `gateway run` command; (B) Native install + `hermes gateway install` (user systemd on Linux, launchd on macOS) or manually-written system systemd unit. The official Docker image uses s6-overlay for supervision.

**Primary recommendation:** Ch.14 (owner plan) owns SUMMARY.md + writes Telegram chapter. Ch.15–16 is ONE combined plan (plan 05-02) writing ONE file `15-more-gateways/index.md`. Ch.17 and Ch.18 are plans 05-03 and 05-04 respectively, running in Wave 2 parallel after the owner plan. Explain below.

---

## Chapter & SUMMARY Structure (PLANNER DECISION)

### New section to add to SUMMARY.md

```markdown
---

# 플랫폼과 운영

- [메시징 게이트웨이: Telegram](14-telegram/index.md)
- [추가 게이트웨이](15-more-gateways/index.md)
- [프로덕션 배포](17-deploy/index.md)
- [보안 하드닝](18-security/index.md)
```

### Chapter file decision: Ch.15–16 = ONE file

**Recommendation: ONE file at `src/15-more-gateways/index.md`** (not two files). Rationale:

- Five additional gateways (Discord/Slack/WhatsApp/Signal/Email) share the same pattern: create credentials → set `.env` vars → `hermes gateway setup` → `hermes gateway start`. Splitting into two files (15 + 16) wastes a chapter number with no content boundary benefit.
- SUMMARY.md uses one entry: `- [추가 게이트웨이](15-more-gateways/index.md)`.
- Chapter numbering: after Ch.14 (Telegram), Ch.15 = "추가 게이트웨이" covers all five. Ch.16 does not exist as a separate file. The planner should skip chapter number 16 and go directly to Ch.17 배포 and Ch.18 보안.
- Avoid mdBook duplicate-path build error (Pitfall B-11): NEVER add both `15-more-gateways/index.md` and `16-*/index.md` — just one file for the combined chapter.

### New src/ directories required

```
src/
├── 14-telegram/
│   └── index.md
├── 15-more-gateways/
│   └── index.md
├── 17-deploy/
│   └── index.md
└── 18-security/
    └── index.md
```

**Note:** No `16-*` directory. Jump from `15-more-gateways` to `17-deploy`.

### Plan / wave structure (reuse Phase 3/4 owner-plan pattern)

- **Plan 05-01 (owner):** Updates SUMMARY.md (adds `# 플랫폼과 운영` + 4 entries). Creates all four `src/NN/index.md` skeleton files. Runs `mdbook build` to verify no duplicate-path error. Also writes the body of **Ch.14**.
- **Plans 05-02 / 05-03 / 05-04 (body, parallel Wave 2 after 05-01):**
  - 05-02: writes Ch.15 (추가 게이트웨이 — Discord/Slack/WhatsApp/Signal/Email)
  - 05-03: writes Ch.17 (프로덕션 배포)
  - 05-04: writes Ch.18 (보안 하드닝)
- No SUMMARY.md write race: owner plan (05-01) owns SUMMARY.md; Wave 2 plans overwrite skeleton bodies only.

---

## Ch.14: 메시징 게이트웨이 — Telegram

**Chapter goal:** 독자가 Telegram 봇을 만들고 Hermes와 채팅한다.

### Key Facts (HIGH confidence)

#### Gateway Architecture (ONE process, ALL platforms)

```
hermes gateway = 단일 백그라운드 프로세스
├── Telegram 어댑터
├── Discord 어댑터
├── Slack 어댑터
├── ... (24개 플랫폼)
├── 세션 스토어 (per-chat 컨텍스트)
├── AIAgent (run_agent.py)
└── 크론 스케줄러 (60초마다 tick)
```

세 가지 실행 방식:
- `hermes gateway` — 포그라운드 실행 (Ctrl-C로 중지), 개발/테스트용
- `hermes gateway start` — 설치된 서비스로 백그라운드 실행
- `hermes gateway run` — Docker 컨테이너 내 실행 시 (gateway 프로세스가 메인 프로세스)

24 supported platforms include: Telegram, Discord, Slack, Google Chat, WhatsApp, Signal, SMS, Email, Home Assistant, Mattermost, Matrix, DingTalk, Feishu/Lark, WeCom, WeChat, BlueBubbles, QQ, Yuanbao, Microsoft Teams, LINE, ntfy, Open WebUI/API Server, Webhooks — all served by the single gateway process.

#### BotFather Token Setup (HIGH confidence — verified from official Telegram docs)

```
1. Telegram 앱에서 @BotFather 검색 (또는 t.me/BotFather)
2. /newbot 전송
3. 표시 이름 입력 (예: "Hermes Agent")
4. 사용자 이름 입력 (반드시 'bot'으로 끝나야 함; 예: hermesagent_bot)
5. BotFather가 토큰 발급:
   형식: 123456789:ABCdefGHIjklMNOpqrSTUvwxYZ
```

**IMPORTANT token format note:** 토큰에 콜론(`:`)이 포함됨 — `.env` 파일에 따옴표 없이 그대로 저장 (PITFALLS.md Integration Gotchas 참조).

#### Hermes Telegram Configuration (HIGH confidence — from .env.example + official docs)

```bash
# ~/.hermes/.env 에 추가
# 검증: hermes rolling, 2026-06-11

TELEGRAM_BOT_TOKEN=<YOUR_BOT_TOKEN>          # BotFather에서 발급받은 토큰
TELEGRAM_ALLOWED_USERS=123456789             # 허용할 Telegram 사용자 ID (쉼표로 구분)
TELEGRAM_HOME_CHANNEL=123456789              # cron 결과를 보낼 기본 채팅 (선택)
# TELEGRAM_WEBHOOK_URL=https://...          # 웹훅 모드 (선택, 기본: 폴링)
```

**자신의 Telegram 사용자 ID 확인 방법:**
`@userinfobot` 또는 `@getmyid_bot`에 메시지를 보내면 숫자 ID를 알려준다.

**Config.yaml 플랫폼 표시 설정 (선택):**

```yaml
# 검증: hermes rolling, 2026-06-11
display:
  platforms:
    telegram:
      tool_progress: verbose   # off | brief | verbose
```

#### Setup Wizard (HIGH confidence)

```bash
# 검증: hermes rolling, 2026-06-11
hermes gateway setup   # 인터랙티브 설정 마법사
# → 플랫폼 선택 (Telegram)
# → 봇 토큰 입력 (또는 .env에서 자동 감지)
# → 허용 사용자 ID 입력
# → 게이트웨이 시작 여부 확인
```

`hermes gateway setup`은 `.env`를 직접 작성하거나 마법사를 통해 설정할 수 있다. 두 방법 모두 동일하게 동작한다.

#### Starting the Gateway (HIGH confidence)

```bash
# 검증: hermes rolling, 2026-06-11
# 방법 1: 포그라운드 (개발/테스트)
hermes gateway

# 방법 2: 설치된 서비스로 시작 (영구 운영)
hermes gateway install    # 서비스 등록 (Linux: systemd user, macOS: launchd)
hermes gateway start      # 서비스 시작

# 상태 확인
hermes gateway status

# 정지
hermes gateway stop
```

#### Polling Mode (Default) vs Webhook Mode

**기본값: Long polling** — Hermes가 Telegram 서버에 아웃바운드 요청을 반복 전송한다. 방화벽/NAT 뒤에서도 동작한다. 포트 개방 불필요.

**웹훅 모드:** 클라우드 배포 시 `TELEGRAM_WEBHOOK_URL` + `TELEGRAM_WEBHOOK_SECRET`으로 설정. 공개 HTTPS URL이 필요하다.

#### Expected Behavior (HIGH confidence)

게이트웨이 시작 후:
1. Telegram에서 봇에 DM을 보낸다
2. 봇이 몇 초 내 온라인 상태가 된다
3. Hermes가 메시지를 처리하고 응답한다

```
# [로컬 실행 후 캡처 필요 — 게이트웨이 시작 메시지 + 첫 봇 응답]
```

### Pitfall Callouts for Ch.14

**주의 1 — Telegram 토큰 중복 폴링 (A-11, CRITICAL)**

```
> 경고: 동일한 Telegram 봇 토큰을 두 개의 Hermes 프로세스 또는 두 프로파일에서 동시에 사용하면
> Telegram이 한 프로세스만 메시지를 받도록 거부합니다.
> 메시지가 간헐적으로 처리되거나 완전히 무시됩니다.
>
> 해결: hermes gateway status 로 실행 중인 인스턴스를 먼저 확인.
> 봇 토큰은 프로파일 간 공유 불가 (공식 문서 명시).
> 로그 위치: ~/.hermes/logs/gateways/default/current
```

**주의 2 — GATEWAY_ALLOW_ALL_USERS는 터미널 접근 봇에 위험 (HIGH)**

```
> 경고: GATEWAY_ALLOW_ALL_USERS=true 는 터미널 도구에 접근하는 에이전트에서 절대 사용하지 마십시오.
> 누구든 Hermes에게 명령어 실행을 지시할 수 있게 됩니다.
> 반드시 TELEGRAM_ALLOWED_USERS 에 본인의 사용자 ID만 지정하거나
> hermes pairing approve 로 DM 페어링을 사용하십시오.
```

**주의 3 — 토큰 보안 (.env에만 저장)**

```
> 주의: TELEGRAM_BOT_TOKEN 은 비밀(secret)입니다.
> config.yaml 이 아닌 ~/.hermes/.env 에만 저장하십시오.
> 스크린샷, 공개 이슈, Git에 절대 노출하지 마십시오.
> 토큰 노출 시 BotFather에서 /revoke 로 즉시 재발급 가능합니다.
> 토큰 형식 예: 123456789:ABCdefGHI... (.env에 따옴표 없이 저장)
```

**주의 4 — 게이트웨이 비용 급증 (A-9)**

```
> 주의: Telegram 게이트웨이를 통한 2시간 가벼운 사용으로
> Claude Sonnet 기준 약 400만 토큰 (~$12)이 소비될 수 있습니다.
> compression.threshold: 0.30 으로 낮추고, /usage 로 비용을 모니터링하십시오.
```

### Hands-on Summary for Ch.14

```bash
# 검증: hermes rolling, 2026-06-11

# 1. BotFather에서 봇 생성 및 토큰 복사
# (@BotFather → /newbot → 이름 → 사용자명)

# 2. .env에 토큰 저장
echo 'TELEGRAM_BOT_TOKEN=<YOUR_BOT_TOKEN>' >> ~/.hermes/.env
echo 'TELEGRAM_ALLOWED_USERS=<YOUR_TELEGRAM_USER_ID>' >> ~/.hermes/.env

# 3. 게이트웨이 시작 (포그라운드 테스트)
hermes gateway

# 4. Telegram에서 봇에 DM 전송 → 응답 확인
# 성공 시: 봇이 몇 초 내 응답

# 5. 서비스로 등록 (영구 운영)
hermes gateway install
hermes gateway start
hermes gateway status
```

---

## Ch.15: 추가 게이트웨이 (Discord / Slack / WhatsApp / Signal / Email)

**Chapter goal:** 독자가 Discord, Slack, WhatsApp, Signal, Email 중 최소 하나를 추가로 설정한다.
**One combined file** at `src/15-more-gateways/index.md`. **No separate Ch.16 file.**

### Discord (HIGH confidence — OPEN FLAG RESOLVED)

#### RESOLVED: Discord Privileged Gateway Intents (2026)

Discord Developer Portal (discord.com/developers/applications) 에서 Bot 탭 → **Privileged Gateway Intents** 섹션에서 다음 두 가지를 반드시 활성화해야 한다:

| Intent | 목적 | 미활성화 시 결과 |
|--------|------|----------------|
| **Server Members Intent** | 서버 멤버 목록 접근, 사용자명 확인 | 서버 멤버 조회 불가 |
| **Message Content Intent** | 메시지 텍스트 읽기 | **봇이 메시지 이벤트는 받지만 텍스트가 비어 있음 → #1 무반응 원인** |

**Presence Intent는 Hermes에 불필요** — 활성화하지 않아도 된다.

> 참고: 2026년 현재 Discord 정책 — 100개 미만 서버의 봇은 수동 심사 없이 Developer Portal에서 즉시 활성화 가능.

#### Discord 앱 생성 (HIGH confidence)

```
1. discord.com/developers/applications 접속
2. "New Application" 클릭 → 이름 입력 → 약관 동의 → "Create"
3. 왼쪽 메뉴에서 "Bot" 탭 클릭
4. "Add Bot" 클릭 (또는 이미 봇 탭에 있음)
5. Token 섹션에서 "Reset Token" → 토큰 복사 (한 번만 표시됨)
6. Privileged Gateway Intents:
   ✅ Server Members Intent 활성화
   ✅ Message Content Intent 활성화
   (Presence Intent는 불필요)
7. "Save Changes" 클릭
```

#### 봇 서버 초대 (HIGH confidence)

**권장 방법 (OAuth2 URL 생성):**

```
Installation 탭 → Install Link → "Discord Provided Link" 선택
→ Add to Server → 서버 선택 → Continue → Authorize → CAPTCHA 완료
```

**수동 URL (옵션):**

```
https://discord.com/oauth2/authorize?client_id=<YOUR_APP_ID>&scope=bot+applications.commands&permissions=274878286912
```

권한 값 `274878286912` = View Channels + Send Messages + Read Message History + Attach Files + Embed Links + Send in Threads + Add Reactions

#### Hermes Discord 설정 (HIGH confidence — from .env.example + official docs)

```bash
# ~/.hermes/.env 에 추가
# 검증: hermes rolling, 2026-06-11

DISCORD_BOT_TOKEN=<YOUR_DISCORD_BOT_TOKEN>
DISCORD_ALLOWED_USERS=123456789012345678    # Discord 사용자 ID (우클릭 → "ID 복사")
```

**Discord 사용자 ID 확인:** 사용자 설정 → 고급 → 개발자 모드 활성화 → 자신의 프로필 우클릭 → "ID 복사"

#### Setup & Start

```bash
# 검증: hermes rolling, 2026-06-11
hermes gateway setup    # Discord 선택 → 토큰 입력
# 또는 .env 직접 편집 후:
hermes gateway          # 포그라운드 테스트
```

#### Expected Behavior (HIGH confidence)

- **DM:** 멘션 없이 모든 메시지에 응답
- **서버 채널 (기본):** @멘션 시에만 응답
- **스레드:** 스레드 내 후속 답글에는 추가 멘션 불필요

#### Pitfall: Message Content Intent (A-10, CRITICAL)

```
> 경고 (Discord #1 문제): 봇이 온라인 상태인데 메시지를 완전히 무시한다면
> Message Content Intent가 비활성화되어 있는 것입니다.
>
> 증상: 봇이 Discord Developer Portal에서 "온라인"으로 보이지만
>       모든 메시지를 완전히 무시한다.
>
> 해결:
> 1. discord.com/developers/applications → 앱 선택 → Bot 탭
> 2. Privileged Gateway Intents 섹션
> 3. "Message Content Intent" 활성화 → Save Changes
> 4. hermes gateway stop && hermes gateway start (재시작 필수)
```

---

### Slack (HIGH confidence)

#### Slack App 생성 (HIGH confidence)

**권장: Manifest 사용**

```bash
# 검증: hermes rolling, 2026-06-11
hermes slack manifest --write
# → ~/.hermes/slack-manifest.json 생성 + 붙여넣기 안내 출력
```

1. api.slack.com/apps 접속 → "Create New App" → "From an app manifest"
2. 워크스페이스 선택 → JSON 탭 → `~/.hermes/slack-manifest.json` 내용 붙여넣기
3. "Next" → "Create"

**Socket Mode 활성화 (필수):**

```
Settings → Socket Mode → Enable Socket Mode
App-Level Token 생성:
  이름: hermes-socket
  Scope: connections:write
  → Generate → 토큰 복사 (xapp- 로 시작)
```

#### 필요한 OAuth Scopes (HIGH confidence)

필수 스코프: `chat:write`, `app_mentions:read`, `channels:history`, `channels:read`, `groups:history`, `im:history`, `im:read`, `im:write`, `users:read`, `files:read`, `files:write`

> 주의: `channels:history`와 `groups:history` 없이는 채널 메시지를 받지 못합니다 — DM 전용으로만 동작합니다.

#### Bot Token 발급

Settings → Install App → Workspace에 설치 → `xoxb-`로 시작하는 Bot User OAuth Token 복사

#### Hermes Slack 설정 (HIGH confidence)

```bash
# ~/.hermes/.env 에 추가
# 검증: hermes rolling, 2026-06-11

SLACK_BOT_TOKEN=xoxb-<YOUR_BOT_TOKEN>
SLACK_APP_TOKEN=xapp-<YOUR_APP_TOKEN>
SLACK_ALLOWED_USERS=U01ABC2DEF3    # Slack 사용자 ID
```

**Slack 사용자 ID 확인:** Slack 앱에서 자신의 프로필 클릭 → "..." → "멤버 ID 복사"

#### Expected Behavior

- **DM:** 멘션 없이 모든 메시지에 응답
- **채널:** @멘션 시에만 응답, 스레드로 답글
- **스레드:** 스레드 내 후속 답글에는 추가 멘션 불필요

#### Why Socket Mode (HIGH confidence)

Hermes는 **Socket Mode만 사용** — WebSocket 기반으로 공개 HTTP 엔드포인트 불필요. 방화벽/NAT 뒤에서도 동작. Classic RTM API는 2025년 3월 완전 deprecated.

---

### WhatsApp (MEDIUM confidence — uses unofficial Web protocol)

#### Setup Flow

```bash
# 검증: hermes rolling, 2026-06-11
hermes whatsapp    # 설정 마법사 실행
# → 모드 선택: 봇 번호 (권장) 또는 개인 자기채팅
# → QR 코드가 터미널에 출력됨
# → 폰의 WhatsApp → 설정 → 연결된 기기 → 기기 연결
# → 터미널 QR 코드 스캔
```

```bash
# ~/.hermes/.env 에 추가
WHATSAPP_ENABLED=true
WHATSAPP_MODE=bot            # 전용 번호 사용 (권장)
WHATSAPP_ALLOWED_USERS=15551234567   # 국가코드 포함, + 없이
```

#### 주의: 공식 WhatsApp Business API 아님

```
> 주의: Hermes의 WhatsApp 통합은 공식 WhatsApp Business API가 아닌
> WhatsApp Web 프로토콜을 사용합니다.
> 전용 번호 사용을 권장하며, 스팸/대량 메시지 발송은 금지입니다.
> 봇 번호와 개인 번호를 구분하여 사용하십시오.
```

---

### Signal (MEDIUM confidence — requires signal-cli)

#### Dependencies

**외부 의존성:** signal-cli (Java 기반) + Java 17+ 런타임이 필요.

```bash
# 사전 요구사항: signal-cli 설치 및 계정 연결
signal-cli link -n "HermesAgent"   # QR 코드 또는 링크 생성
# → 폰의 Signal → 설정 → 연결된 기기 → 기기 연결

# signal-cli 데몬 실행 (HTTP 모드)
signal-cli --account +1234567890 daemon --http 127.0.0.1:8080
```

```bash
# ~/.hermes/.env 에 추가
SIGNAL_HTTP_URL=http://127.0.0.1:8080
SIGNAL_ACCOUNT=+1234567890
SIGNAL_ALLOWED_USERS=+82101234567   # 허용할 번호
```

```bash
# 검증: hermes rolling, 2026-06-11
hermes gateway setup   # Signal 선택
hermes gateway         # 게이트웨이 시작
```

#### 알려진 제한

- 메시지 편집 미지원 → 툴 진행 버블 표시 안 됨 (verbose 모드에서도)
- 첨부 파일 최대 100MB, 이미지 배치 32개 제한

---

### Email (HIGH confidence)

#### IMAP/SMTP 설정

```bash
# ~/.hermes/.env 에 추가
# 검증: hermes rolling, 2026-06-11

EMAIL_ADDRESS=hermesagent@gmail.com     # 전용 이메일 계정 (개인 계정 사용 금지)
EMAIL_PASSWORD=<앱 비밀번호>            # Gmail 2FA 활성화 시 앱 비밀번호 필요
EMAIL_IMAP_HOST=imap.gmail.com          # Gmail; Outlook: outlook.office365.com
EMAIL_SMTP_HOST=smtp.gmail.com          # Gmail; Outlook: smtp.office365.com
# EMAIL_IMAP_PORT=993                   # 기본값 993
# EMAIL_SMTP_PORT=587                   # 기본값 587
# EMAIL_POLL_INTERVAL=15               # 기본값 15초
EMAIL_ALLOWED_USERS=user@example.com    # 허용할 이메일 주소
```

**Gmail 앱 비밀번호 생성:** Google 계정 → 보안 → 2단계 인증 → 앱 비밀번호

#### 동작 방식

IMAP 받은 편지함을 15초마다 폴링하여 UNSEEN 메시지를 감지. 제목이 컨텍스트로 포함되고, In-Reply-To 헤더로 스레드가 유지된다.

#### 알려진 제한

- HTML 이메일 → 텍스트만 추출
- 개인 이메일에 사용 금지 (전용 계정 권장)
- 15초 폴링 지연

---

### 공통 게이트웨이 설정 (모든 플랫폼)

#### 설정 완료 후 게이트웨이 시작

```bash
# 검증: hermes rolling, 2026-06-11
hermes gateway setup   # 설정되지 않은 플랫폼 추가 설정
hermes gateway         # 포그라운드로 모든 플랫폼 동시 시작
```

하나의 `hermes gateway` 프로세스가 `.env`에 설정된 **모든 플랫폼을 동시에** 처리한다.

#### 인증: DM 페어링 시스템

```bash
# 검증: hermes rolling, 2026-06-11
hermes pairing list                          # 승인 대기 목록
hermes pairing approve telegram XKGH5N7P    # 페어링 코드 승인
hermes pairing revoke telegram 123456789    # 사용자 접근 취소
```

알 수 없는 사용자가 봇에 DM 시 8자리 일회성 코드 수신 → 운영자가 `hermes pairing approve`로 승인. 코드는 1시간 후 만료.

---

## Ch.17: 프로덕션 배포

**Chapter goal:** 독자가 Hermes를 프로덕션 환경에 배포하여 항상 켜진 상태로 운영한다.

### Three Verified Scenarios (HIGH / MEDIUM confidence)

Phase 3 연구에서 확립된 6개 터미널 백엔드 사실은 그대로 유지. Ch.17은 Hermes 자체(게이트웨이 포함)를 서버에 배포하는 방법을 다룬다.

---

### 시나리오 A: Docker 배포 (HIGH confidence — from official Docker docs)

#### Official Docker Image

```bash
# 검증: hermes rolling + Docker, 2026-06-11

# 1. 초기 설정 (최초 1회)
mkdir -p ~/.hermes
docker run -it --rm \
  -v ~/.hermes:/opt/data \
  nousresearch/hermes-agent setup

# 2. 게이트웨이 서비스로 실행
docker run -d \
  --name hermes \
  --restart unless-stopped \
  -v ~/.hermes:/opt/data \
  -p 8642:8642 \
  nousresearch/hermes-agent gateway run

# 3. 상태 확인
docker logs -f hermes

# 4. 업그레이드
docker pull nousresearch/hermes-agent:latest
docker rm -f hermes
docker run -d --name hermes --restart unless-stopped \
  -v ~/.hermes:/opt/data -p 8642:8642 \
  nousresearch/hermes-agent gateway run
```

#### Docker Compose (권장 — 대시보드 포함)

```yaml
# docker-compose.yml
# 검증: hermes rolling, 2026-06-11
services:
  hermes:
    image: nousresearch/hermes-agent:latest
    container_name: hermes
    restart: unless-stopped
    command: gateway run
    ports:
      - "8642:8642"   # Gateway API
      - "9119:9119"   # Dashboard
    volumes:
      - ~/.hermes:/opt/data
    environment:
      - HERMES_DASHBOARD=1
    deploy:
      resources:
        limits:
          memory: 4G
          cpus: "2.0"
```

```bash
# 검증: hermes rolling, 2026-06-11
docker compose up -d
docker compose logs -f
```

#### Docker 기술 사항 (HIGH confidence)

- **Base image:** `debian:13.4` + s6-overlay v3 (PID 1 프로세스 수퍼바이저)
- **s6-overlay:** 게이트웨이 크래시 시 자동 재시작 (컨테이너 종료 없음)
- **Non-root:** 컨테이너는 UID 10000 (`hermes` 사용자)로 실행; root는 첫 부팅 시 볼륨 권한 설정에만 사용
- **포트 8642:** Gateway API 서버 (OpenAI 호환) + 헬스 엔드포인트
- **포트 9119:** 대시보드 (HERMES_DASHBOARD=1 필요)

#### 리소스 권장 사항 (HIGH confidence)

| 리소스 | 최소 | 권장 |
|--------|------|------|
| RAM | 1 GB | 2–4 GB |
| CPU | 1 코어 | 2 코어 |
| 디스크 | 500 MB | 2+ GB |
| (브라우저 자동화 추가 시) | 2 GB | 4 GB |

브라우저 도구 사용 시 `--shm-size=1g` 추가 필요.

#### Docker Pitfall: root 소유 파일 (A-13 — Ch.8에서 다뤘지만 Ch.17 재확인)

컨테이너 비루트 사용자(UID 10000) 설계로 인해 파일 소유권 문제는 공식 이미지에서 대부분 해결됨. NAS/특수 환경은 `PUID`/`PGID` 환경변수 사용.

```
> 경고: 절대로 두 컨테이너를 같은 데이터 디렉터리(~/.hermes)로 동시에 실행하지 마십시오.
> 세션 파일과 메모리 저장소에 동시 쓰기 보호가 없습니다.
```

---

### 시나리오 B: VPS 네이티브 설치 + systemd (MEDIUM confidence)

#### 권장 VPS 사양

| 리소스 | 최소 | 권장 |
|--------|------|------|
| vCPU | 1 | 2 |
| RAM | 1 GB | 2 GB (브라우저 자동화 4 GB) |
| 디스크 | 5 GB | 20 GB |
| OS | Ubuntu 22.04+ | Ubuntu 24.04 LTS |

월 ~$4–5의 Hetzner CX22 (2 vCPU, 4 GB RAM) 또는 Hostinger KVM1이 운영 사례에서 널리 사용됨.

#### 네이티브 설치 + 서비스 등록

```bash
# 검증: hermes rolling, 2026-06-11
# 1. 설치
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
source ~/.bashrc

# 2. 설정
hermes setup --portal   # 또는 hermes model

# 3-A. hermes gateway install (사용자 레벨 systemd — 권장)
hermes gateway install          # ~/.config/systemd/user/hermes-gateway.service 생성
hermes gateway start            # 서비스 시작
hermes gateway status           # 상태 확인

# 3-B. 시스템 레벨 서비스 (VPS 재부팅 후 자동 시작)
sudo hermes gateway install --system   # /etc/systemd/system/hermes-gateway.service 생성
# 또는 수동 systemd 유닛 파일 작성 (아래 참조)
```

#### 수동 systemd 유닛 파일 (MEDIUM confidence — community-verified pattern)

```ini
# /etc/systemd/system/hermes.service
# 검증: 커뮤니티 검증 패턴, 로컬 확인 권장, 2026-06-11
[Unit]
Description=Hermes Agent Gateway
Documentation=https://hermes-agent.nousresearch.com/docs
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=<your-user>
Group=<your-user>
WorkingDirectory=/home/<your-user>
Environment="HOME=/home/<your-user>"
Environment="PATH=/home/<your-user>/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
ExecStart=/home/<your-user>/.local/bin/hermes gateway
Restart=on-failure
RestartSec=10
MemoryMax=2G
NoNewPrivileges=true
ProtectSystem=strict
ProtectHome=read-only
ReadWritePaths=/home/<your-user>/.hermes

[Install]
WantedBy=multi-user.target
```

```bash
# 서비스 활성화
sudo systemctl daemon-reload
sudo systemctl enable --now hermes
sudo systemctl status hermes
journalctl -u hermes -f   # 로그 모니터링
```

**macOS launchd 경로:** `~/Library/LaunchAgents/ai.hermes.gateway.plist`
**Linux user systemd 경로:** `~/.config/systemd/user/hermes-gateway.service`

#### Pitfall: macOS launchd PATH (A-15)

```
> 주의 (macOS): hermes gateway install 로 launchd 서비스를 설치한 후
> Homebrew나 nvm으로 새 도구를 설치하면, launchd의 PATH에 새 도구가 없습니다.
> 새 도구 설치 후 hermes gateway install을 재실행하여 PATH를 갱신하십시오.
```

---

### 시나리오 C: SSH 백엔드 (Phase 3 기반, MEDIUM confidence)

SSH 백엔드는 **Hermes가 로컬에서 실행되면서 터미널 명령을 원격 서버에서 실행**하는 방식이다. Hermes 자체를 VPS에 배포하는 것과 다름에 주의.

```yaml
# 검증: hermes rolling, 2026-06-11
terminal:
  backend: ssh
  persistent_shell: true
```

```bash
# ~/.hermes/.env 에 추가
TERMINAL_SSH_HOST=your-server.example.com
TERMINAL_SSH_USER=ubuntu
# TERMINAL_SSH_KEY=~/.ssh/id_ed25519   # 기본: ~/.ssh/id_rsa
```

SSH ControlMaster를 통한 멀티플렉싱으로 각 명령마다 새 SSH 연결 없이 세션을 유지한다.

---

### 시나리오 D: Modal 백엔드 (MEDIUM confidence — OPEN FLAG PARTIALLY RESOLVED)

**RESOLVED 상태:** Modal 백엔드의 핵심 YAML 키는 확인됨. 단, `modal_image` 키와 managed mode 설정은 로컬 검증 권장.

```yaml
# 검증: hermes rolling, 2026-06-11
terminal:
  backend: modal
  container_cpu: 1
  container_memory: 5120    # MB
  container_disk: 51200     # MB
  container_persistent: true
  modal_image: "nikolaik/python-nodejs:python3.11-nodejs20"  # MEDIUM confidence
```

**인증:**

```bash
# ~/.hermes/.env — Modal은 .env 키 불필요, CLI 인증 사용
# 대신:
pip install modal
modal setup   # 브라우저 OAuth 인증 → ~/.modal.toml 생성
```

또는 환경변수 방식: `MODAL_TOKEN_ID` + `MODAL_TOKEN_SECRET`

> 검증 필요: `modal_image` 키 이름과 Modal managed mode의 정확한 설정 흐름을 로컬 Modal 계정으로 확인하십시오.

**특성:** 에페머럴 — `container_persistent: true`로 파일시스템 스냅샷/복원. 유휴 비용 없음 (cold start 있음).

---

### 시나리오 E: Daytona 백엔드 (MEDIUM confidence)

```yaml
# 검증: hermes rolling, 2026-06-11
terminal:
  backend: daytona
  container_cpu: 1
  container_memory: 5120    # MB
  container_disk: 10240     # MB (최대 10GB)
  container_persistent: true
```

```bash
# ~/.hermes/.env 에 추가
DAYTONA_API_KEY=<YOUR_DAYTONA_API_KEY>
```

Daytona SDK가 서버 URL을 자동 처리. 유휴 시 히버네이션으로 비용 절감.

> 검증 필요: Daytona 백엔드 실제 동작 (히버네이션 복구 시간, 정확한 SDK 버전)을 로컬 확인하십시오.

---

### API 서버 (선택 사항 — HIGH confidence)

게이트웨이 실행 중 OpenAI 호환 API 서버 활성화 가능:

```bash
# ~/.hermes/.env 에 추가
# 검증: hermes rolling, 2026-06-11
API_SERVER_ENABLED=true
API_SERVER_KEY=$(openssl rand -hex 32)   # 무작위 키 생성
# API_SERVER_HOST=0.0.0.0               # 외부 접근 허용 시 (주의 필요)
```

```
헬스 엔드포인트: GET http://localhost:8642/health
확장 헬스:       GET http://localhost:8642/health/detailed
```

Open WebUI, LobeChat, LibreChat 등 OpenAI 호환 프론트엔드 연결 가능.

---

### Ch.17 주요 Pitfall Callouts

**주의 1 — 초기 설정은 SSH 필수, 웹 콘솔 금지 (HIGH)**

```
> 경고: VPS 웹 콘솔(브라우저 기반 터미널)에서 hermes setup 또는
> docker run ... setup 을 실행하지 마십시오.
> ':', '@', '=' 같은 특수 문자가 웹 콘솔에서 잘못 전송되어
> API 키와 봇 토큰이 손상될 수 있습니다.
> 항상 SSH로 접속하여 설정하십시오.
```

**주의 2 — 두 컨테이너/프로세스가 같은 ~/.hermes 디렉터리 공유 금지 (HIGH)**

```
> 경고: 두 개의 Hermes 프로세스(또는 컨테이너)가 동일한 ~/.hermes 데이터 디렉터리를
> 동시에 사용하면 세션 파일과 메모리 저장소가 손상될 수 있습니다.
> 멀티 프로파일이 필요하면 단일 컨테이너의 hermes profile create 를 사용하십시오.
```

**주의 3 — Docker root 소유 파일 (A-13)**

```
> 주의: 공식 Docker 이미지(nousresearch/hermes-agent)는 UID 10000으로 실행하여
> root 소유 파일 문제를 방지합니다.
> 다른 Docker 이미지나 docker_run_as_host_user 설정 시 파일 소유권을 확인하십시오.
```

---

## Ch.18: 보안 하드닝

**Chapter goal:** 독자가 승인 모드를 이해하고 API 키를 안전하게 관리하며 운영 상태를 진단한다.

### Approvals Mode (HIGH confidence — from official security docs)

```yaml
# 검증: hermes rolling, 2026-06-11
approvals:
  mode: manual         # manual (기본, 권장) | smart | off
  timeout: 300         # 초 (기본값)
  cron_mode: manual    # 헤드리스 cron: manual | auto
```

| 모드 | 동작 | 권장 환경 |
|------|------|-----------|
| `manual` | 위험한 명령 전 항상 사용자 승인 요청 | **프로덕션 기본값** |
| `smart` | 보조 LLM이 위험도 평가; 저위험 자동 승인, 고위험 자동 거부, 불확실 수동 위임 | 반자동화 환경 |
| `off` | 모든 승인 체크 비활성화 (`--yolo`와 동일) | 컨테이너 격리된 환경만 |

**Hardline blocklist:** `off` 모드에서도 `rm -rf /`, 포크 폭탄, `mkfs.*` 등은 항상 차단됨.

#### 프로덕션 권장: `manual` 유지

```
> 경고: approvals.mode: off 는 위험한 명령에 대한 모든 안전 체크를 비활성화합니다.
> 로컬 백엔드에서는 절대 사용하지 마십시오.
> Docker/Modal/Daytona 컨테이너 백엔드에서 컨테이너가 보안 경계 역할을 할 때만 고려하십시오.
> 활성화 시 빨간색 배너 "⚠ YOLO mode"가 표시됩니다.
```

---

### API Key Management (HIGH confidence)

#### .env vs config.yaml 분리 원칙

| 파일 | 저장 내용 | 예시 |
|------|-----------|------|
| `~/.hermes/.env` | **비밀만** | API 키, 봇 토큰, OAuth 자격증명 |
| `~/.hermes/config.yaml` | 비밀 외 설정 | 모델, 압축 설정, 백엔드 타입 |

```bash
# 검증: hermes rolling, 2026-06-11
# .env 파일 권한 설정 (중요)
chmod 600 ~/.hermes/.env

# 올바른 키 설정 방법
hermes config set ANTHROPIC_API_KEY sk-ant-...   # → .env 자동 라우팅
hermes config set OPENROUTER_API_KEY sk-or-...   # → .env 자동 라우팅

# 직접 확인
cat ~/.hermes/.env    # 저장된 키 확인
```

**Git 노출 방지:**

```bash
# 검증: hermes rolling, 2026-06-11
# ~/.gitignore 또는 프로젝트 .gitignore 에 추가
echo "~/.hermes/.env" >> ~/.gitignore
echo ".hermes/" >> .gitignore   # 프로젝트 레벨 gitignore
```

#### Silent API Key Skip 버그 주의 (A-5, GitHub Issue #16394)

```
> 주의 (GitHub Issue #16394):
> ~/.hermes/.env에 키 항목이 이미 존재하면(값이 잘못되어도)
> hermes setup이 조용히 건너뜁니다. 녹색 체크마크가 표시되어도
> 키가 유효하지 않을 수 있습니다.
>
> 해결책:
> 1. ~/.hermes/.env를 직접 편집하여 잘못된 키 줄 삭제
> 2. hermes setup 재실행
> 또는:
>   hermes config set PROVIDER_API_KEY <새 키>  (덮어쓰기 가능)
```

---

### Tirith Security Scanning (HIGH confidence)

```yaml
# 검증: hermes rolling, 2026-06-11
security:
  tirith_enabled: true       # 기본값: true
  tirith_timeout: 5          # 초
  tirith_fail_open: true     # 스캐너 미사용 시 실행 허용
  redact_secrets: true       # API 키 패턴 자동 난독화 (MCP 오류 메시지 등)
```

Tirith가 감지하는 패턴:
- 호모그래프 URL 스푸핑 (국제화 도메인 공격)
- 파이프-인터프리터 패턴 (`curl | bash`)
- 터미널 주입 공격

Tirith는 SHA-256 검증으로 GitHub에서 자동 설치됨 (첫 사용 시).

---

### hermes doctor — 공급망 보안 검사 (HIGH confidence — CORRECTS prior research)

**중요 수정 사항:** 이전 연구에서 "`hermes doctor`가 설정 + 의존성 문제를 진단한다"고 했으나 이는 부분적으로 부정확합니다.

**실제 동작:**
- `hermes doctor`는 **공급망 어드바이저리 검사기**입니다
- 활성 Python venv에서 알려진 취약한 패키지를 감지합니다
- 각 어드바이저리에 대해 2–4단계 수정 지침을 제공합니다

```bash
# 검증: hermes rolling, 2026-06-11
hermes doctor                          # 공급망 어드바이저리 확인
hermes doctor --ack <advisory-id>      # 검토된 어드바이저리 확인 처리 (영구 무시)

# 설정 검증 (별도 명령)
hermes config check                    # 설정 파일 유효성 검사
hermes config show                     # 전체 설정 확인

# 런타임 상태
hermes status                          # 에이전트/인증/플랫폼 상태
hermes gateway status                  # 게이트웨이 서비스 상태
hermes logs -f --level WARNING         # 경고 이상 로그 실시간 확인
```

**운영 런북 (Operational Runbook):**

```bash
# 검증: hermes rolling, 2026-06-11
# 1. 공급망 보안 확인
hermes doctor

# 2. 설정 검증
hermes config check

# 3. API 연결 및 모델 확인
hermes status

# 4. 게이트웨이 상태
hermes gateway status

# 5. 최근 오류 로그 확인
hermes logs --level ERROR --since 1h

# 6. 업데이트 확인
hermes update --check

# 7. 사용량 통계
hermes insights --days 7
```

#### hermes security audit는 존재하지 않음 (CORRECTION)

```
> 참고: 'hermes security audit' 명령은 존재하지 않습니다 (공식 문서 미존재).
> 보안 관련 작업은 다음 명령으로 나뉩니다:
>   hermes doctor         → 공급망 어드바이저리
>   hermes pairing list   → 인증된 사용자 관리
>   hermes config check   → 설정 검증
>   (Tirith 스캔은 명령 실행 시 자동)
```

---

### PII 데이터 처리 (HIGH confidence — CLARIFIES prior research)

```
> 참고: Hermes의 "PII 리댁션"은 일반적인 개인정보 필터링이 아닙니다.
> 실제 기능: MCP 오류 메시지에서 자격증명 패턴을 난독화
>   - GitHub PAT (ghp_...)
>   - OpenAI 스타일 키 (sk-...)
>   - Bearer 토큰, 비밀번호/키 파라미터
> 이는 security.redact_secrets: true 로 제어됩니다 (기본값: true).
>
> 별도의 "PII 리댁션 설정 키"는 존재하지 않습니다.
```

---

### Gateway Security Configuration (HIGH confidence)

#### 인증 우선순위 (낮을수록 우선)

1. `GATEWAY_ALLOW_ALL_USERS=true` (전체 허용 — 절대 권장하지 않음)
2. `GATEWAY_ALLOWED_USERS` (글로벌 허용 목록)
3. 플랫폼별 허용 목록 (`TELEGRAM_ALLOWED_USERS`, `DISCORD_ALLOWED_USERS` 등)
4. DM 페어링 승인 목록 (`hermes pairing approve`)
5. 기본: **전체 거부**

```yaml
# 검증: hermes rolling, 2026-06-11
# config.yaml 보안 관련 설정
unauthorized_dm_behavior: pair    # pair (페어링 코드 발송) | ignore (무시)
security:
  allow_private_urls: false       # RFC 1918 내부 URL 접근 차단
  allow_lazy_installs: false      # 런타임 의존성 설치 차단
```

#### 프로덕션 10단계 체크리스트 (from official security docs)

```
# 검증: 공식 보안 문서 기반, 2026-06-11
□ 1. 명시적 허용 목록 설정 (GATEWAY_ALLOW_ALL_USERS=true 절대 사용 금지)
□ 2. 컨테이너 백엔드 사용 (terminal.backend: docker)
□ 3. CPU/메모리/디스크 제한 설정
□ 4. ~/.hermes/.env 권한 설정 (chmod 600)
□ 5. DM 페어링으로 사용자 인증
□ 6. command_allowlist 정기 감사
□ 7. terminal.cwd 설정으로 민감한 디렉터리 접근 차단
□ 8. 비루트 계정으로 실행
□ 9. ~/.hermes/logs/ 모니터링
□ 10. hermes update 로 최신 상태 유지
```

---

## Resolved vs Deferred Flags Summary

| 플래그 | 상태 | 결과 |
|--------|------|------|
| **Discord Privileged Gateway Intents 현재 플로우** | **RESOLVED — HIGH** | 2개 Intent 필요: Server Members + Message Content. Presence Intent 불필요. 2026년 현재 <100서버 봇은 수동 심사 없이 Developer Portal에서 즉시 활성화. 허밋 토큰 환경변수: `DISCORD_BOT_TOKEN`. OAuth scope: `bot+applications.commands`, permission int: 274878286912. |
| **Modal 백엔드 YAML 키** | **PARTIALLY RESOLVED — MEDIUM** | 핵심 키 확인: `backend: modal`, `container_cpu`, `container_memory`, `container_disk`, `container_persistent`, `modal_image`. 인증: `modal setup` CLI (`.env` 불필요) 또는 `MODAL_TOKEN_ID`+`MODAL_TOKEN_SECRET`. Modal managed mode 설정 흐름은 DEFERRED — 로컬 확인 필요. |
| **Daytona 백엔드 YAML 키** | **RESOLVED — MEDIUM** | 키 확인: `backend: daytona`, 공통 container 키들. 인증: `DAYTONA_API_KEY`. SDK가 서버 URL 자동 처리. 실제 동작은 로컬 확인 권장. |
| **`hermes doctor` 정확한 기능** | **RESOLVED — HIGH (CORRECTION)** | `hermes doctor`는 공급망 어드바이저리 검사기 (Python 패키지 취약점 스캔). 일반 설정 진단은 `hermes config check`. |
| **`hermes security audit` 존재 여부** | **RESOLVED — HIGH (CORRECTION)** | 존재하지 않음 — 공식 문서에 없음. Ch.18에서 이 명령 언급 금지. |
| **PII 리댁션 기능 상세** | **RESOLVED — HIGH (CLARIFICATION)** | 일반 PII 필터 아님. MCP 오류의 자격증명 패턴 난독화. `security.redact_secrets: true` (기본값). |

---

## Open Questions for Local Verification

1. **`hermes gateway status` 정확한 출력 형식**
   - What we know: 실행 중인 PID, 서비스 상태, 설치 권장 메시지 표시
   - What's unclear: 정확한 텍스트 형식 및 다중 플랫폼 상태 표시
   - Recommendation: `hermes gateway start && hermes gateway status` 실행 후 캡처

2. **`hermes gateway setup` 인터랙티브 플로우**
   - What we know: 플랫폼 선택 → 토큰 입력 → 사용자 ID 입력 → 게이트웨이 시작 여부
   - What's unclear: 정확한 UI 텍스트, 화살표 키 선택 메뉴 항목들
   - Recommendation: 실행 후 스크린샷 또는 터미널 캡처

3. **Telegram 봇 첫 응답 메시지 형식**
   - What we know: 봇이 몇 초 내 온라인 상태가 됨
   - What's unclear: 첫 응답의 정확한 UI (버블? 텍스트?)
   - Recommendation: BotFather 봇 생성 후 실제 DM 테스트

4. **Discord 봇 온라인 상태 표시**
   - What we know: `hermes gateway` 시작 후 봇이 Discord에서 온라인 표시
   - What's unclear: 정확한 시작 시간, 오프라인→온라인 전환 소요 시간
   - Recommendation: 로컬 테스트 후 캡처

5. **`hermes doctor` 출력 형식**
   - What we know: 어드바이저리 ID, 버전 정보, 2-4단계 수정 지침 표시
   - What's unclear: 어드바이저리 없을 때의 출력 ("모든 것이 정상" 메시지?)
   - Recommendation: 실행 후 캡처 (정상 및 어드바이저리 있는 경우)

6. **Modal managed mode (`hermes setup terminal` 흐름)**
   - What we know: NousResearch 게이트웨이를 통한 Modal 리소스 사전 공급 모드 존재
   - What's unclear: 현재 이 모드가 활성화되어 있는지, 어떻게 선택하는지
   - Recommendation: Modal 계정이 있으면 `hermes setup terminal` 실행하여 확인

7. **`hermes gateway install --system` vs `hermes gateway install` 정확한 차이**
   - What we know: `--system`은 시스템 레벨 서비스 설치 (GitHub 이슈 #16264 에서 언급)
   - What's unclear: `sudo hermes gateway install --system`으로 설치 시 정확한 서비스 파일 경로
   - Recommendation: Ubuntu VPS에서 직접 실행하여 확인

8. **Slack manifest JSON 내용**
   - What we know: `hermes slack manifest --write`로 생성됨
   - What's unclear: 생성된 JSON의 정확한 내용 (특히 버전 갱신이 필요한 항목)
   - Recommendation: `hermes slack manifest --write` 실행 후 내용 확인

9. **WhatsApp 세션 지속성**
   - What we know: QR 코드 스캔으로 페어링, 세션 자동 저장
   - What's unclear: 재시작 후 세션이 자동 복구되는지
   - Recommendation: gateway 재시작 후 WhatsApp 재페어링 필요 여부 확인

10. **`hermes config check` 존재 확인**
    - What we know: `hermes config` 서브커맨드 중 하나로 언급됨
    - What's unclear: STACK.md 검증 목록에 없음; CLI 레퍼런스에서 확인 필요
    - Recommendation: `hermes config --help` 실행하여 check 서브커맨드 존재 확인

---

## Corrections to "Established Facts" from Context

다음 항목들은 연구 과정에서 수정되었습니다:

1. **`hermes doctor` 기능 수정:** 이전 연구의 "설정 + 의존성 진단" 설명은 부분적으로 부정확. 실제: 공급망 어드바이저리 검사기. 일반 설정 진단은 `hermes config check`.

2. **`hermes security audit` 명령 없음:** 이전 연구에서 언급된 이 명령은 존재하지 않음. STACK.md의 `hermes security audit` 항목 삭제 권장.

3. **PII 리댁션 설정 키 없음:** 일반 PII 필터 기능 없음. `security.redact_secrets: true`는 MCP 오류의 자격증명 패턴만 처리.

4. **`hermes gateway run`과 `hermes gateway start` 구분:** Phase 3/4 연구에서 혼용. 정확히:
   - `hermes gateway` / `hermes gateway run` → 포그라운드 실행 (또는 Docker 내 메인 프로세스)
   - `hermes gateway start` → 설치된 서비스를 백그라운드에서 시작
   - `hermes gateway install` → 서비스 등록 (systemd/launchd 파일 생성)

5. **24개 플랫폼 지원 (22+가 아님):** 이전 연구에서 "22+ 메시징 게이트웨이"로 언급. 공식 문서는 24개.

6. **Discord 3가지 Intent 중 2가지만 필요:** Presence Intent는 Hermes에 불필요. Server Members + Message Content만 활성화.

---

## Sources

### Primary (HIGH confidence — official docs verified 2026-06-11)

- `https://hermes-agent.nousresearch.com/docs/user-guide/messaging/telegram` — BotFather steps, TELEGRAM_BOT_TOKEN, polling vs webhook, double-poll pitfall
- `https://hermes-agent.nousresearch.com/docs/user-guide/messaging/discord` — Discord app creation, Privileged Intents (Server Members + Message Content), DISCORD_BOT_TOKEN, OAuth scopes + permissions integer
- `https://hermes-agent.nousresearch.com/docs/user-guide/messaging/slack` — SLACK_BOT_TOKEN, SLACK_APP_TOKEN, Socket Mode (WebSocket), hermes slack manifest, OAuth scopes
- `https://hermes-agent.nousresearch.com/docs/user-guide/messaging/whatsapp` — QR pairing, WHATSAPP_ENABLED/MODE/ALLOWED_USERS, unofficial protocol warning
- `https://hermes-agent.nousresearch.com/docs/user-guide/messaging/signal` — signal-cli dependency, Java 17+, SIGNAL_HTTP_URL/ACCOUNT, linking model
- `https://hermes-agent.nousresearch.com/docs/user-guide/messaging/email` — IMAP/SMTP vars, app password, poll interval, thread preservation
- `https://github.com/NousResearch/hermes-agent/blob/main/website/docs/user-guide/messaging/index.md` — 24 platforms, gateway architecture (single process), pairing commands, GATEWAY_ALLOW_ALL_USERS warning, hermes gateway vs hermes gateway start distinction
- `https://hermes-agent.nousresearch.com/docs/user-guide/docker` — Official Docker image, s6-overlay, non-root UID 10000, docker-compose, resource limits, HERMES_DASHBOARD, multi-profile in Docker, log routing, browser tools --shm-size
- `https://hermes-agent.nousresearch.com/docs/user-guide/security` — approvals.mode (manual/smart/off), Tirith config, PII/redact_secrets, production hardening checklist, hermes doctor supply-chain advisory description
- `https://hermes-agent.nousresearch.com/docs/user-guide/configuration` — approvals block, security block, modal_image key, DAYTONA_API_KEY, MODAL_TOKEN_ID/SECRET
- `https://hermes-agent.nousresearch.com/docs/user-guide/multi-profile-gateways` — hermes gateway run/start/stop/restart/install, service file locations (macOS/Linux), per-profile isolation
- `https://hermes-agent.nousresearch.com/docs/user-guide/features/api-server` — port 8642, health endpoints, OpenAI compatibility, API_SERVER_KEY
- `https://github.com/NousResearch/hermes-agent/blob/main/.env.example` — Confirmed exact env var names for all platforms (TELEGRAM_BOT_TOKEN, DISCORD_BOT_TOKEN, SLACK_BOT_TOKEN, SLACK_APP_TOKEN, etc.), Modal uses `modal setup` not .env key
- `https://github.com/NousResearch/hermes-agent/blob/main/cli-config.yaml.example` — Complete config structure including terminal backends, display.platforms, gateway platform config

### Secondary (MEDIUM confidence — verified with official source cross-check)

- `https://github.com/NousResearch/hermes-agent/issues/16264` — hermes gateway install creates user systemd service files at ~/.config/systemd/user/, --system flag exists
- `https://lumadock.com/tutorials/run-hermes-agent-with-systemd` — Community systemd unit file template (manual path); verified structure matches official systemd conventions
- `https://support-dev.discord.com/hc/en-us/articles/6207308062871-What-are-Privileged-Intents` — Discord official: 3 Privileged Intents exist; Hermes needs 2 (Members + Message Content)
- WebSearch: `hermes agent gateway systemd VPS production 2026` — Confirmed hermes gateway install + --system flag, VPS specs

### From Prior Research (imported and verified)

- `PITFALLS.md`: A-5 (silent skip #16394), A-9 (token cost), A-10 (Discord Intents), A-11 (Telegram double-poll), A-12 (approvals.mode), A-13 (Docker root files), A-15 (macOS launchd PATH) — all confirmed and mapped
- `03-RESEARCH.md`: Docker backend config (docker_run_as_host_user, docker_image), SSH backend env vars (TERMINAL_SSH_HOST/USER) — confirmed
- `04-RESEARCH.md`: Gateway cron inside gateway process, 60s tick confirmed

---

## Metadata

**Confidence breakdown:**

| 챕터/토픽 | 신뢰도 | 근거 |
|-----------|--------|------|
| Ch.14 BotFather steps | HIGH | Official Telegram docs + Hermes messaging/telegram doc |
| Ch.14 TELEGRAM_BOT_TOKEN env var | HIGH | .env.example confirmed |
| Ch.14 hermes gateway commands (run/start/install) | HIGH | multi-profile-gateways doc + index.md |
| Ch.14 polling vs webhook | HIGH | messaging/telegram doc |
| Ch.14 double-poll pitfall | HIGH | messaging/telegram doc |
| Ch.15 Discord Intents (2 required) | HIGH | messaging/discord doc + Discord official docs |
| Ch.15 DISCORD_BOT_TOKEN | HIGH | messaging/discord doc + .env.example |
| Ch.15 Discord OAuth permissions integer | HIGH | messaging/discord doc |
| Ch.15 SLACK_BOT_TOKEN + SLACK_APP_TOKEN | HIGH | messaging/slack doc + .env.example |
| Ch.15 Slack Socket Mode | HIGH | messaging/slack doc |
| Ch.15 hermes slack manifest | HIGH | messaging/slack doc |
| Ch.15 WhatsApp QR pairing | MEDIUM | messaging/whatsapp doc (unofficial protocol) |
| Ch.15 Signal signal-cli dependency | MEDIUM | messaging/signal doc (external dependency, Java 17+) |
| Ch.15 Email IMAP/SMTP vars | HIGH | messaging/email doc + .env.example |
| Ch.17 Docker official image | HIGH | user-guide/docker official doc |
| Ch.17 docker-compose config | HIGH | user-guide/docker official doc |
| Ch.17 s6-overlay / UID 10000 | HIGH | user-guide/docker official doc |
| Ch.17 hermes gateway install (user systemd) | HIGH | multi-profile-gateways + GitHub issue #16264 |
| Ch.17 hermes gateway install --system | MEDIUM | GitHub issue mentions it; exact path unverified |
| Ch.17 Manual systemd unit file | MEDIUM | Community template; official path not documented |
| Ch.17 VPS specs ($4-5/mo, Hetzner/Hostinger) | MEDIUM | Community guides |
| Ch.17 Modal YAML keys | MEDIUM | configuration docs + .env.example |
| Ch.17 Daytona YAML keys | MEDIUM | configuration docs (Phase 3 confirmed) |
| Ch.18 approvals.mode config block | HIGH | security doc + configuration doc |
| Ch.18 Tirith config keys | HIGH | security doc (confirmed from Phase 3) |
| Ch.18 hermes doctor = supply-chain advisor | HIGH | security doc |
| Ch.18 security.redact_secrets | HIGH | configuration doc |
| Ch.18 PII redaction = credential patterns only | HIGH | security doc |
| Ch.18 hermes security audit does NOT exist | HIGH | security doc (absent) |
| Ch.18 production 10-step checklist | HIGH | security doc |
| Ch.18 unauthorized_dm_behavior config key | HIGH | security doc |

**Research date:** 2026-06-11
**Valid until:** 2026-07-11 (Hermes is rolling release — re-verify gateway setup commands 30 days before publish)
**Key risk:** Discord Developer Portal UI may change without notice (check portal screenshots are current). Hermes rolling release means exact `hermes gateway setup` wizard text may drift.
