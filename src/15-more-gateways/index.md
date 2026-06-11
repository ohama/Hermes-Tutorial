# 추가 게이트웨이

> **이 챕터를 시작하기 전에:** [Ch.14 메시징 게이트웨이: Telegram](../14-gateways/index.md)에서 게이트웨이 아키텍처, 실행 방식(포그라운드/서비스/Docker), DM 페어링을 먼저 학습하십시오.
>
> 단일 `hermes gateway` 프로세스가 `.env`에 설정된 **모든 플랫폼을 동시에** 처리합니다. Telegram을 이미 설정했다면 이 챕터에서 추가 플랫폼을 설정해도 기존 Telegram 연결은 그대로 유지됩니다.

## 개요

이 챕터에서는 Telegram에 이어 다섯 가지 추가 게이트웨이(Discord / Slack / WhatsApp / Signal / Email)를 설정하는 방법을 다룹니다. 모든 플랫폼이 동일한 패턴을 따릅니다.

```
자격증명 생성 → .env 변수 설정 → hermes gateway setup (또는 .env 직접 편집) → hermes gateway (포그라운드 테스트) → 서비스로 등록
```

**최소 하나만 설정해도 성공입니다.** 사용하는 플랫폼에 해당하는 섹션으로 바로 이동하십시오.

---

## Discord

### Privileged Gateway Intents (필수 2개)

Discord Developer Portal (discord.com/developers/applications)의 Bot 탭에서 **Privileged Gateway Intents** 섹션에 있는 다음 두 가지를 반드시 활성화해야 합니다.

| Intent | 목적 | 미활성화 시 결과 |
|--------|------|----------------|
| **Server Members Intent** | 서버 멤버 목록 접근, 사용자명 확인 | 서버 멤버 조회 불가 |
| **Message Content Intent** | 메시지 텍스트 읽기 | **봇이 메시지 이벤트는 받지만 텍스트가 비어 있음 → #1 무반응 원인** |

> **중요:** **Presence Intent는 Hermes에 불필요**합니다. 활성화하지 않아도 됩니다.

2026년 현재 Discord 정책: 100개 미만 서버의 봇은 수동 심사 없이 Developer Portal에서 즉시 활성화 가능합니다.

---

> **주의 (Discord #1 문제 — 미리 읽어두십시오):**
> Message Content Intent가 비활성화되어 있으면 봇이 온라인 상태로 보이지만 모든 메시지를 완전히 무시합니다.
> 이는 **#1 무반응 원인**입니다. 자세한 해결 방법은 이 챕터 하단 [흔한 오류 / 주의](#흔한-오류--주의) 섹션을 참조하십시오.

---

### Discord 앱 생성

```
# 검증: hermes rolling, 2026-06-11
1. discord.com/developers/applications 접속
2. "New Application" 클릭 → 이름 입력 → 약관 동의 → "Create"
3. 왼쪽 메뉴에서 "Bot" 탭 클릭
4. Token 섹션에서 "Reset Token" → 토큰 복사 (한 번만 표시됨 — 즉시 저장)
5. Privileged Gateway Intents 섹션:
   ✅ Server Members Intent 활성화
   ✅ Message Content Intent 활성화
   (Presence Intent는 불필요 — 활성화 안 해도 됩니다)
6. "Save Changes" 클릭
```

> **토큰 보안:** Reset Token 화면에서 발급된 토큰은 한 번만 표시됩니다. 반드시 즉시 복사하여 `~/.hermes/.env`에 저장하십시오. 이 화면을 벗어나면 다시 볼 수 없으며 재발급이 필요합니다.

### 봇 서버 초대

**권장 방법 — Installation 탭 사용:**

```
# 검증: hermes rolling, 2026-06-11
Installation 탭 → Install Link → "Discord Provided Link" 선택
→ Add to Server → 서버 선택 → Continue → Authorize → CAPTCHA 완료
```

**수동 OAuth URL (옵션):**

```
https://discord.com/oauth2/authorize?client_id=<YOUR_APP_ID>&scope=bot+applications.commands&permissions=274878286912
```

권한 값 `274878286912` = View Channels + Send Messages + Read Message History + Attach Files + Embed Links + Send in Threads + Add Reactions

`<YOUR_APP_ID>`는 Discord Developer Portal → 앱 선택 → General Information에서 확인할 수 있습니다.

### Hermes Discord 설정

```bash
# 검증: hermes rolling, 2026-06-11
# ~/.hermes/.env 에 추가

DISCORD_BOT_TOKEN=<YOUR_DISCORD_BOT_TOKEN>
DISCORD_ALLOWED_USERS=123456789012345678    # Discord 사용자 ID (아래 확인 방법 참조)
```

**Discord 사용자 ID 확인 방법:** Discord 설정 → 고급 → 개발자 모드 활성화 → 자신의 프로필 우클릭 → "ID 복사"

설정 후:

```bash
# 검증: hermes rolling, 2026-06-11
hermes gateway setup    # Discord 선택 → 토큰 입력
# 또는 .env 직접 편집 후 바로 게이트웨이 시작:
hermes gateway          # 포그라운드 테스트
```

> 검증 필요: `hermes gateway setup` 마법사의 정확한 UI 텍스트와 메뉴 항목은 버전에 따라 다를 수 있습니다. 로컬 실행 후 확인하십시오.

### 기대 동작

- **DM:** 멘션 없이 모든 메시지에 응답
- **서버 채널 (기본):** @멘션 시에만 응답
- **스레드:** 스레드 내 후속 답글에는 추가 멘션 불필요

---

## Slack

### Slack App 생성 (Manifest 권장)

```bash
# 검증: hermes rolling, 2026-06-11
hermes slack manifest --write
# → ~/.hermes/slack-manifest.json 생성 + 붙여넣기 안내 출력
```

> 검증 필요: `hermes slack manifest --write`가 생성하는 JSON의 정확한 내용은
> 버전에 따라 다를 수 있습니다. 로컬 실행 후 확인하십시오.

생성된 파일을 Slack에 적용하는 방법:

```
# 검증: hermes rolling, 2026-06-11
1. api.slack.com/apps 접속 → "Create New App" 클릭
2. "From an app manifest" 선택 → 워크스페이스 선택
3. JSON 탭 선택 → ~/.hermes/slack-manifest.json 내용 붙여넣기
4. "Next" → "Create"
```

### Socket Mode 활성화 (필수)

Hermes는 **Socket Mode만 사용**합니다. WebSocket 기반으로 공개 HTTP 엔드포인트가 불필요하므로 방화벽/NAT 뒤에서도 정상 동작합니다. Classic RTM API는 2025년 3월 완전 deprecated되었습니다.

```
# 검증: hermes rolling, 2026-06-11
Settings → Socket Mode → "Enable Socket Mode" 활성화
App-Level Token 생성:
  이름: hermes-socket
  Scope: connections:write
  → "Generate" → 토큰 복사 (xapp- 로 시작)
```

### OAuth Scopes

Slack 앱이 필요한 Bot Token Scopes:

`chat:write`, `app_mentions:read`, `channels:history`, `channels:read`, `groups:history`, `im:history`, `im:read`, `im:write`, `users:read`, `files:read`, `files:write`

> **주의:** `channels:history`와 `groups:history` 없이는 채널 메시지를 받지 못합니다. 이 스코프가 없으면 DM 전용으로만 동작합니다.

### Bot Token 발급 + Hermes 설정

Slack 앱 페이지에서: Settings → **Install App** → Workspace에 설치 → `xoxb-`로 시작하는 **Bot User OAuth Token** 복사

```bash
# 검증: hermes rolling, 2026-06-11
# ~/.hermes/.env 에 추가

SLACK_BOT_TOKEN=xoxb-<YOUR_BOT_TOKEN>
SLACK_APP_TOKEN=xapp-<YOUR_APP_TOKEN>
SLACK_ALLOWED_USERS=U01ABC2DEF3    # Slack 사용자 ID (아래 확인 방법 참조)
```

**Slack 사용자 ID 확인 방법:** Slack 앱에서 자신의 프로필 클릭 → "..." 메뉴 → "멤버 ID 복사"

### 기대 동작

- **DM:** 멘션 없이 모든 메시지에 응답
- **채널:** @멘션 시에만 응답, 스레드로 답글
- **스레드:** 스레드 내 후속 답글에는 추가 멘션 불필요

---

## WhatsApp (개요)

> **신뢰도:** MEDIUM — Hermes의 WhatsApp 통합은 공식 WhatsApp Business API가 아닌 WhatsApp Web 프로토콜을 사용합니다. 이 섹션은 개요 수준으로 다루며, 깊은 동작 보장은 제공하지 않습니다.

**설정 흐름:**

```bash
# 검증: hermes rolling, 2026-06-11
hermes whatsapp    # 설정 마법사 실행
# → 모드 선택: 봇 번호 (권장) 또는 개인 자기채팅
# → QR 코드가 터미널에 출력됨

# 폰의 WhatsApp: 설정 → 연결된 기기 → 기기 연결
# → 터미널 QR 코드 스캔
```

```bash
# 검증: hermes rolling, 2026-06-11
# ~/.hermes/.env 에 추가

WHATSAPP_ENABLED=true
WHATSAPP_MODE=bot                      # 전용 번호 사용 (권장)
WHATSAPP_ALLOWED_USERS=15551234567     # 국가코드 포함, + 없이
```

> **주의: 공식 WhatsApp Business API 아님**
>
> Hermes의 WhatsApp 통합은 공식 WhatsApp Business API가 아닌 WhatsApp Web 프로토콜을 사용합니다.
> 전용 번호 사용을 권장하며, 스팸/대량 메시지 발송은 금지입니다.
> 봇 번호와 개인 번호를 구분하여 사용하십시오.

---

## Signal (개요)

> **신뢰도:** MEDIUM — Signal 통합은 외부 의존성(signal-cli + Java 17+)에 의존합니다. 이 섹션은 개요 수준으로 다루며, 깊은 동작 보장은 제공하지 않습니다.

**사전 요구사항:** signal-cli (Java 기반) + Java 17+ 런타임이 필요합니다.

```bash
# 검증: hermes rolling, 2026-06-11
# 1. signal-cli로 계정 연결
signal-cli link -n "HermesAgent"
# → QR 코드 또는 연결 링크 생성
# → 폰의 Signal → 설정 → 연결된 기기 → 기기 연결 → 스캔

# 2. signal-cli 데몬을 HTTP 모드로 실행
signal-cli --account +1234567890 daemon --http 127.0.0.1:8080
```

```bash
# 검증: hermes rolling, 2026-06-11
# ~/.hermes/.env 에 추가

SIGNAL_HTTP_URL=http://127.0.0.1:8080
SIGNAL_ACCOUNT=+1234567890
SIGNAL_ALLOWED_USERS=+82101234567    # 허용할 번호 (+ 포함)
```

```bash
# 검증: hermes rolling, 2026-06-11
hermes gateway setup    # Signal 선택
hermes gateway          # 포그라운드 테스트
```

**알려진 제한:**

- 메시지 편집 미지원 → 툴 진행 버블 표시 안 됨 (verbose 모드에서도)
- 첨부 파일 최대 100MB, 이미지 배치 32개 제한

---

## Email (개요)

**설정 방법:**

```bash
# 검증: hermes rolling, 2026-06-11
# ~/.hermes/.env 에 추가

EMAIL_ADDRESS=hermesagent@gmail.com     # 전용 이메일 계정 (개인 계정 사용 금지)
EMAIL_PASSWORD=<앱_비밀번호>            # Gmail 2FA 활성화 시 앱 비밀번호 필요
EMAIL_IMAP_HOST=imap.gmail.com          # Gmail; Outlook: outlook.office365.com
EMAIL_SMTP_HOST=smtp.gmail.com          # Gmail; Outlook: smtp.office365.com
# EMAIL_IMAP_PORT=993                   # 기본값 993
# EMAIL_SMTP_PORT=587                   # 기본값 587
# EMAIL_POLL_INTERVAL=15               # 기본값 15초
EMAIL_ALLOWED_USERS=user@example.com    # 허용할 이메일 주소
```

**Gmail 앱 비밀번호 생성:** Google 계정 → 보안 → 2단계 인증 → 앱 비밀번호

**동작 방식:** IMAP 받은 편지함을 15초마다 폴링하여 UNSEEN 메시지를 감지합니다. 제목이 컨텍스트로 포함되고, In-Reply-To 헤더로 스레드가 유지됩니다.

**알려진 제한:**

- HTML 이메일 → 텍스트만 추출
- 개인 이메일에 사용 금지 (전용 계정 권장)
- 15초 폴링 지연

---

## 공통: 게이트웨이 시작과 DM 페어링

### 게이트웨이 시작

플랫폼 설정이 완료되면 게이트웨이를 시작합니다.

```bash
# 검증: hermes rolling, 2026-06-11
hermes gateway setup   # 설정되지 않은 플랫폼 추가 설정 (선택)
hermes gateway         # 포그라운드로 모든 플랫폼 동시 시작 (Ctrl-C로 중지)
```

> 검증 필요: `hermes gateway setup` 마법사의 정확한 UI 텍스트와 메뉴 항목은 버전에 따라 다를 수 있습니다. 로컬 실행 후 확인하십시오.

하나의 `hermes gateway` 프로세스가 `.env`에 설정된 **모든 플랫폼을 동시에** 처리합니다. Discord, Slack, Telegram이 모두 설정되어 있다면 단일 명령으로 세 플랫폼 모두 활성화됩니다.

영구 운영을 위해 서비스로 등록하는 방법은 [Ch.14 메시징 게이트웨이: Telegram](../14-gateways/index.md)의 "서비스로 등록" 섹션을 참조하십시오.

### DM 페어링

`DISCORD_ALLOWED_USERS` 등에 등록되지 않은 알 수 없는 사용자가 봇에 DM을 보내면, 봇은 8자리 일회성 코드를 발급합니다. 운영자가 이 코드를 승인하면 해당 사용자는 이후 인증 없이 사용할 수 있습니다. 코드는 1시간 후 만료됩니다.

```bash
# 검증: hermes rolling, 2026-06-11
hermes pairing list                             # 승인 대기 목록 확인
hermes pairing approve telegram XKGH5N7P       # 페어링 코드 승인
hermes pairing revoke telegram 123456789       # 사용자 접근 취소
```

---

## 흔한 오류 / 주의

> **경고 (Discord #1 문제): 봇이 온라인인데 메시지를 완전히 무시한다면**
> **Message Content Intent가 비활성화된 것입니다.**
>
> **증상:** Discord Developer Portal에서 봇이 "온라인"으로 보이지만 모든 메시지를 완전히 무시합니다.
>
> **해결:**
> 1. discord.com/developers/applications → 앱 선택 → Bot 탭
> 2. Privileged Gateway Intents 섹션
> 3. "Message Content Intent" 활성화 → Save Changes
> 4. `hermes gateway stop && hermes gateway start` (재시작 필수)

---

> **주의 (Slack):** `channels:history`와 `groups:history` OAuth Scope가 누락되면 채널 메시지를 받지 못하고 DM 전용으로만 동작합니다. Slack App 설정에서 두 스코프를 확인하고 앱을 재설치하십시오.

---

> **주의 (WhatsApp):** Hermes의 WhatsApp 통합은 WhatsApp Web 비공식 프로토콜을 사용합니다. WhatsApp 정책 변경에 따라 세션이 끊기거나 기능이 제한될 수 있습니다. 전용 번호 사용을 권장합니다.

---

## 다음 단계

추가 게이트웨이를 설정했다면, 이제 Hermes를 프로덕션 환경에 배포할 차례입니다.

- 다음: [Ch.17 프로덕션 배포](../17-deploy/index.md) — Docker 배포, VPS 네이티브 설치, 서비스 등록
- 이전: [Ch.14 메시징 게이트웨이: Telegram](../14-gateways/index.md) — 게이트웨이 아키텍처, 실행 방식, DM 페어링
