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
