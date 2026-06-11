# 메시징 게이트웨이: Telegram

> **이 챕터를 시작하기 전에**
> - [Ch.7 툴 게이트웨이](../07-tools/index.md) — 게이트웨이 개념 이해 (Hermes가 외부 서비스에 연결되는 방식)
> - [Ch.12 크론 스케줄러](../12-cron/index.md) — 게이트웨이가 스케줄 실행과 결과 배달을 담당함을 이미 다뤘습니다
>
> `hermes gateway`는 Telegram을 포함한 모든 메시징 플랫폼 어댑터를 운영하는 단일 서비스입니다.

## 개요

지금까지 터미널에서 Hermes를 사용했습니다. 이 챕터에서는 **Telegram 봇**으로 노출하는 방법을 다룹니다. 단일 `hermes gateway` 프로세스 하나가 24개 플랫폼 어댑터를 동시에 처리하므로, Telegram을 시작으로 나중에 Discord나 Slack도 추가할 수 있습니다.

이 챕터를 완료하면 Telegram에서 봇에게 DM을 보내 Hermes와 대화할 수 있습니다.

## 개념: 게이트웨이 아키텍처

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

하나의 게이트웨이 프로세스가 `.env`에 설정된 **모든 플랫폼을 동시에** 처리합니다. Telegram을 활성화해도 다른 플랫폼은 비활성화되지 않으며, 이후 Discord나 Slack을 추가해도 게이트웨이를 재설치할 필요가 없습니다.

24개 지원 플랫폼: Telegram, Discord, Slack, Google Chat, WhatsApp, Signal, SMS, Email, Home Assistant, Mattermost, Matrix, DingTalk, Feishu/Lark, WeCom, WeChat, BlueBubbles, QQ, Yuanbao, Microsoft Teams, LINE, ntfy, Open WebUI/API Server, Webhooks 등

## 개념: 게이트웨이 실행 방식 3가지

```bash
# 검증: hermes rolling, 2026-06-11
# 방법 1: 포그라운드 실행 — 개발·테스트용, Ctrl-C로 중지
hermes gateway

# 방법 2: Docker 컨테이너 내 실행 — gateway가 메인 프로세스
hermes gateway run

# 방법 3: 설치된 서비스를 백그라운드로 시작 — 영구 운영용
hermes gateway install    # 먼저 서비스 등록 (Linux: systemd user, macOS: launchd)
hermes gateway start      # 등록된 서비스 시작
```

> **중요:** 처음 게이트웨이를 띄울 때는 `hermes gateway`(터미널 포그라운드) 또는 `hermes gateway run`(Docker 메인 프로세스)을 사용합니다. `hermes gateway start`는 `hermes gateway install`로 서비스를 먼저 등록한 이후에만 사용하는 명령입니다. install 없이 start를 실행하면 오류가 발생합니다.

## 실습: BotFather로 봇 만들기

Telegram 봇은 @BotFather를 통해 생성합니다. 아래 5단계를 순서대로 진행하십시오.

1. Telegram 앱에서 **@BotFather** 를 검색하거나 `t.me/BotFather`로 접속합니다.
2. `/newbot` 명령을 전송합니다.
3. **표시 이름**을 입력합니다. (예: `Hermes Agent`) — 이름에 "bot"이 없어도 됩니다.
4. **사용자 이름**을 입력합니다. — 반드시 `bot`으로 끝나야 합니다. (예: `hermesagent_bot`)
5. BotFather가 **봇 토큰**을 발급합니다. 형식:

```
123456789:ABCdefGHIjklMNOpqrSTUvwxYZ
```

토큰에 **콜론(`:`)이 포함**됩니다. `.env` 파일에 따옴표 없이 그대로 저장해야 합니다 (따옴표를 추가하면 인식 오류가 발생합니다).

> **주의: TELEGRAM_BOT_TOKEN은 비밀(secret)입니다.**
> `config.yaml`이 아닌 `~/.hermes/.env`에만 저장하십시오.
> 스크린샷, 공개 이슈, Git에 절대 노출하지 마십시오.
> 토큰 노출 시 BotFather에서 `/revoke`로 즉시 재발급 가능합니다.

## 실습: .env에 토큰 저장

**자신의 Telegram 사용자 ID 확인 방법:** `@userinfobot` 또는 `@getmyid_bot`에 메시지를 보내면 숫자 ID를 알려줍니다.

```bash
# 검증: hermes rolling, 2026-06-11
# ~/.hermes/.env 에 추가 (파일이 없으면 새로 생성)

TELEGRAM_BOT_TOKEN=<YOUR_BOT_TOKEN>           # BotFather에서 발급받은 토큰 (따옴표 없이)
TELEGRAM_ALLOWED_USERS=<YOUR_TELEGRAM_USER_ID>  # 허용할 Telegram 사용자 ID (쉼표로 구분)
TELEGRAM_HOME_CHANNEL=<YOUR_TELEGRAM_USER_ID>   # cron 결과를 보낼 기본 채팅 (선택)
# TELEGRAM_WEBHOOK_URL=https://...             # 웹훅 모드 (선택, 기본: 폴링)
```

`TELEGRAM_ALLOWED_USERS`를 반드시 설정하십시오. 설정하지 않으면 누구나 봇에게 메시지를 보낼 수 있습니다. 자신의 ID만 허용하는 것이 가장 안전합니다.

선택 사항으로 `config.yaml`에서 플랫폼 표시 설정을 조정할 수 있습니다.

```yaml
# 검증: hermes rolling, 2026-06-11
display:
  platforms:
    telegram:
      tool_progress: verbose   # off | brief | verbose
```

## 실습: 게이트웨이 설정 마법사

`.env`를 직접 편집하는 대신 인터랙티브 마법사를 사용할 수 있습니다.

```bash
# 검증: hermes rolling, 2026-06-11
hermes gateway setup   # 인터랙티브 설정 마법사
# → 플랫폼 선택 (Telegram)
# → 봇 토큰 입력 (또는 .env에서 자동 감지)
# → 허용 사용자 ID 입력
# → 게이트웨이 시작 여부 확인
```

> 검증 필요: `hermes gateway setup` 마법사의 정확한 화면 텍스트와 선택 메뉴 항목은
> 버전/환경에 따라 다를 수 있습니다. 로컬 실행 후 확인하십시오.

`.env`를 직접 편집해도 마법사와 동일하게 동작합니다. 두 방법 중 편한 것을 선택하십시오.

## 실습: 게이트웨이 시작과 봇 대화

### 포그라운드 실행 (개발·테스트)

```bash
# 검증: hermes rolling, 2026-06-11
hermes gateway
# Ctrl-C로 중지
```

게이트웨이가 시작되면 Telegram에서 봇에게 DM을 보내십시오. 몇 초 내로 봇이 온라인 상태가 되고 Hermes가 응답합니다.

```
# [로컬 실행 후 캡처 필요 — 게이트웨이 시작 메시지 + 첫 봇 응답]
```

> 검증 필요: 게이트웨이 시작 로그와 첫 봇 응답의 정확한 출력 형식은 환경에 따라 다를 수 있습니다.
> 로컬에서 실행 후 확인하십시오.

### Docker 내 실행

Docker를 사용한다면 `hermes gateway run`이 컨테이너의 메인 프로세스로 실행됩니다 (자세한 내용은 Ch.17 프로덕션 배포).

```bash
# 검증: hermes rolling, 2026-06-11
docker run -d \
  --name hermes \
  --restart unless-stopped \
  -v ~/.hermes:/opt/data \
  nousresearch/hermes-agent gateway run
```

### 영구 운영 (서비스로 등록)

```bash
# 검증: hermes rolling, 2026-06-11
# 서비스 등록 (Linux: ~/.config/systemd/user/, macOS: ~/Library/LaunchAgents/)
hermes gateway install

# 서비스 시작
hermes gateway start

# 상태 확인
hermes gateway status

# 서비스 중지
hermes gateway stop
```

`hermes gateway install`을 실행하면 시스템 재부팅 이후에도 게이트웨이가 자동으로 시작됩니다.

## 개념: 폴링 vs 웹훅

**기본값: Long polling** — Hermes가 Telegram 서버에 아웃바운드 요청을 반복 전송합니다.

- 방화벽/NAT 뒤에서도 동작합니다.
- 포트 개방이 필요하지 않습니다.
- 가정용 인터넷 환경에서 바로 사용할 수 있습니다.

**웹훅 모드:** 클라우드 서버에 배포할 경우 사용할 수 있습니다.

```bash
# 검증: hermes rolling, 2026-06-11
# ~/.hermes/.env 에 추가 (웹훅 모드 활성화)
TELEGRAM_WEBHOOK_URL=https://your-domain.example.com/webhook
TELEGRAM_WEBHOOK_SECRET=<YOUR_WEBHOOK_SECRET>
```

웹훅 모드는 공개 HTTPS URL이 필요합니다. 로컬 개발에서는 Long polling이 더 간편합니다.

## 흔한 오류 / 주의

### 주의 1 — Telegram 토큰 중복 폴링 (A-11, CRITICAL)

> **경고:** 동일한 Telegram 봇 토큰을 두 개의 Hermes 프로세스 또는 두 프로파일에서 동시에 사용하면
> Telegram이 한 프로세스만 메시지를 받도록 거부합니다.
> 메시지가 간헐적으로 처리되거나 완전히 무시됩니다.
>
> **해결:** `hermes gateway status`로 실행 중인 인스턴스를 먼저 확인하십시오.
> 봇 토큰은 프로파일 간 공유 불가합니다 (공식 문서 명시).
> 게이트웨이 로그 위치: `~/.hermes/logs/gateways/default/current`

### 주의 2 — GATEWAY_ALLOW_ALL_USERS는 위험 (HIGH)

> **경고:** `GATEWAY_ALLOW_ALL_USERS=true`는 터미널 도구에 접근하는 에이전트에서 절대 사용하지 마십시오.
> 누구든 Hermes에게 명령어 실행을 지시할 수 있게 됩니다.
> 반드시 `TELEGRAM_ALLOWED_USERS`에 본인의 사용자 ID만 지정하거나
> `hermes pairing approve`로 DM 페어링을 사용하십시오.

### 주의 3 — 토큰 보안 (.env에만 저장)

> **주의:** `TELEGRAM_BOT_TOKEN`은 비밀(secret)입니다.
> `config.yaml`이 아닌 `~/.hermes/.env`에만 저장하십시오.
> 스크린샷, 공개 이슈, Git에 절대 노출하지 마십시오.
> 토큰 노출 시 BotFather에서 `/revoke`로 즉시 재발급 가능합니다.
> 토큰 형식 예: `123456789:ABCdefGHI...` (`.env`에 따옴표 없이 저장)

### 주의 4 — 게이트웨이 비용 급증 (A-9)

> **주의:** Telegram 게이트웨이를 통한 2시간 가벼운 사용으로
> Claude Sonnet 기준 약 400만 토큰(~$12)이 소비될 수 있습니다.
> `compression.threshold: 0.30`으로 낮추고, `/usage`로 비용을 모니터링하십시오.

## 다음 단계

이전: [Ch.13 서브에이전트](../13-subagents/index.md)

다음: [Ch.15 추가 게이트웨이](../15-more-gateways/index.md) — Discord, Slack, WhatsApp, Signal, Email 등 5개 플랫폼을 추가로 설정합니다.
