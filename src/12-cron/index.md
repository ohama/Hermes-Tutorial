# 크론 스케줄러

> **이 챕터를 시작하기 전에**
> - [Ch.9 스킬 시스템](../09-skills/index.md) — 스킬 주입 이해 (크론 작업에 스킬을 주입하려면 반드시 선행)
> - [Ch.7 툴 게이트웨이](../07-tools/index.md) — 게이트웨이 개념 이해 (결과 배달/스케줄 실행에 필요)
>
> 게이트웨이(`hermes gateway`)는 스케줄 실행과 결과 배달을 담당하는 서비스입니다. 크론 작업 **등록** 자체는 게이트웨이 실행 여부와 독립적으로 가능합니다.

## 개요

크론 스케줄러를 사용하면 **무인 반복 작업**, **지연 실행**, **특정 시각 작업**을 등록하고 자동으로 실행할 수 있습니다. 예를 들어 매일 오전 9시에 뉴스를 요약해 Telegram으로 받거나, 2시간마다 코드 저장소를 감시하거나, 특정 날짜에 보고서를 자동 생성할 수 있습니다.

각 작업은 **완전히 격리된 새로운 에이전트 세션**에서 실행됩니다. 결과는 Telegram, Discord 등 다양한 플랫폼으로 자동 배달됩니다. 토큰 소비 없이 셸 스크립트만 실행하는 `--no-agent` 모드도 지원합니다.

## 개념: 명령은 hermes cron 입니다

> **정확성 핵심:** 크론 작업을 등록·관리하는 CLI는 **`hermes cron`** 입니다. `hermes gateway`가 아닙니다.

`hermes gateway`는 60초마다 tick하며 스케줄을 실행하고 결과를 배달하는 **서비스**입니다. 크론 작업을 등록·수정·삭제하는 명령이 아닙니다.

### 아키텍처 개요

```
hermes cron CLI       →  ~/.hermes/cron/<job_id>.json 에 작업 저장
hermes gateway (서비스) →  60초마다 next_run_at 확인 → 격리 AIAgent 실행 → 결과 배달
                           파일 잠금으로 이중 실행 방지
```

| 요소 | 역할 |
|------|------|
| `hermes cron` | 작업 등록·관리 CLI |
| `gateway/run.py` (내부) | 스케줄러 프로세스 — cron 스케줄 실행 |
| `hermes gateway` | 게이트웨이 서비스 시작 명령 (포그라운드) |
| `hermes gateway install` | launchd/systemd에 서비스 등록 |
| `~/.hermes/cron/` | 작업 JSON 저장 위치 |
| `~/.hermes/cron/output/<job_id>/<timestamp>.md` | 실행 결과 저장 |

`hermes cron` CLI는 게이트웨이 실행 여부와 **무관하게** 작업을 등록하고 조회할 수 있습니다. 실제 **스케줄 실행**은 게이트웨이가 필요합니다.

## 개념: cron CLI

### 작업 관리 명령

```bash
# 검증: hermes rolling, 2026-06-10
# 작업 관리
hermes cron list                              # 모든 작업 보기
hermes cron create "schedule" "prompt"        # 작업 추가
hermes cron edit <job_id> --schedule "..."    # 스케줄 수정
hermes cron pause <job_id_or_name>            # 일시 중지
hermes cron resume <job_id_or_name>           # 재개
hermes cron run <job_id_or_name>              # 즉시 수동 트리거
hermes cron remove <job_id_or_name>           # 삭제
hermes cron status                            # 전체 상태 확인
hermes cron tick                              # 수동 tick (디버그용)
```

### 게이트웨이 서비스 명령 (스케줄 실행·결과 배달에 필요)

아래는 `hermes cron` 명령이 아니라 **게이트웨이 서비스** 명령입니다. 스케줄 자동 실행과 결과 배달에 필요합니다.

```bash
# 검증: hermes rolling, 2026-06-10
hermes gateway                       # 포그라운드 실행 (테스트용)
hermes gateway install               # launchd/systemd 사용자 서비스 등록
sudo hermes gateway install --system # 시스템 서비스로 등록
```

## 개념: 스케줄 형식

| 형식 | 예시 | 동작 |
|------|------|------|
| 상대 지연 | `30m`, `2h`, `1d` | 1회 실행 |
| 인터벌 | `every 30m`, `every 2h` | 영구 반복 |
| Cron 표현식 | `0 9 * * *` (매일 오전 9시) | 영구 반복 |
| ISO 타임스탬프 | `2026-03-15T09:00:00` | 1회 지정 시각 |

## 실습: 예약 작업 등록

### 기본 예시

```bash
# 검증: hermes rolling, 2026-06-10
# 매일 오전 9시 요약 보고서 — Telegram으로 배달
hermes cron create "0 9 * * *" \
  "오늘의 주요 뉴스를 검색하고 한국어로 요약해서 알려줘" \
  --deliver telegram

# 2시간마다 블로그 모니터링 — 스킬 주입, 이름 지정
hermes cron create "every 2h" \
  "새 블로그 포스트 확인 및 요약" \
  --skill blogwatcher \
  --deliver telegram \
  --name "blog-monitor"

# 특정 작업 디렉터리 지정 — PR 감사 보고서 (절대 경로 필수)
hermes cron create "every 1d at 09:00" \
  "PR 감사 보고서 작성" \
  --workdir /home/me/projects/acme \
  --deliver "discord:#engineering"
```

### 등록 후 확인

```bash
# 검증: hermes rolling, 2026-06-10
hermes cron list
```

## 실습: 즉시 실행과 상태 확인

```bash
# 검증: hermes rolling, 2026-06-10
# 수동으로 즉시 실행 (스케줄 대기 없이 트리거)
hermes cron run blog-monitor

# 전체 상태 확인
hermes cron status
```

> **검증 필요:** `hermes cron run <name>`이 gateway 프로세스 없이 독립적으로 실행 가능한지
> 로컬에서 확인하십시오(gateway 중지 상태로 테스트).

## 개념: 스킬 주입과 다중 플랫폼 배달

### 스킬 주입

스킬은 프롬프트 실행 **전에** 격리된 새 세션에 주입됩니다. 스킬이 없으면 격리 세션은 해당 워크플로우를 알지 못합니다. [Ch.9 스킬 시스템](../09-skills/index.md)을 먼저 학습하는 이유가 여기 있습니다.

```bash
# 검증: hermes rolling, 2026-06-10
# 단일 스킬 주입
hermes cron create "every 1h" "피드 요약" --skill blogwatcher

# 복수 스킬 주입 — 순서대로 로드됨
hermes cron create "every 1h" "통합 브리핑" \
  --skill blogwatcher --skill maps
```

### 다중 플랫폼 배달

응답은 `--deliver` 지정 대상으로 자동 배달됩니다. 프롬프트에 `send_message`를 호출할 필요가 없습니다.

| 배달 대상 | 문법 |
|----------|------|
| 기본 (origin) | `"origin"` |
| 로컬 파일 | `"local"` → `~/.hermes/cron/output/` |
| Telegram | `"telegram"` |
| Discord 채널 | `"discord:#engineering"` |
| 복수 대상 | `"telegram,discord"` |
| 모든 설정된 플랫폼 | `"all"` |
| 복합 | `"origin,all"` |

## 심화: no-agent 모드와 config

### no-agent 모드 — LLM 없이 셸 스크립트 실행

LLM 추론이 필요 없는 감시·점검 작업은 `--no-agent --script` 조합을 사용합니다. 토큰을 소비하지 않습니다.

```bash
# 검증: hermes rolling, 2026-06-10
hermes cron create "every 5m" \
  --no-agent \
  --script memory-watchdog.sh \
  --deliver telegram \
  --name "memory-watchdog"
```

스크립트 동작 규칙:

| 결과 | 동작 |
|------|------|
| stdout 비어 있음 | 조용히 종료 (알림 없음) |
| 비 0 exit 코드 | 에러 알림 배달 |
| 마지막 줄 `{"wakeAgent": false}` | 조용히 종료 |

### config.yaml cron 설정

```yaml
# 검증: hermes rolling, 2026-06-10
cron:
  wrap_response: false          # 래퍼 헤더/푸터 억제
  script_timeout_seconds: 300   # 사전 실행 스크립트 시간 제한 (초)
```

### 작업 체이닝 — context_from

에이전트 내부에서 `cronjob` 툴의 `context_from` 파라미터를 사용하면 이전 작업의 출력을 다음 작업의 컨텍스트로 전달할 수 있습니다. 이 파이프라인은 에이전트 내부에서 구성하며, 자연어로 요청하면 에이전트가 `cronjob` 툴을 적절히 호출합니다.

## 흔한 오류 / 주의

### 주의 1: 크론 세션은 메모리가 없습니다

> **경고:** cron 작업은 **완전히 새로운 에이전트 세션**에서 실행됩니다.
> 현재 대화 내용, 메모리, 사용자 프로파일을 알지 못합니다.
> 필요한 모든 컨텍스트(URL, 설정, 기호)를 프롬프트에 명시적으로 포함하십시오.
>
> **나쁜 예:** `"요약해줘"` (무엇을 요약해야 할지 모름)
>
> **좋은 예:** `"https://example.com/feed 에서 새 포스트를 확인하고 한국어로 요약해줘"` (URL 명시)

### 주의 2: workdir 미설정 시 파일 경로 예측 불가 + 순차 실행 사이드이펙트

> **주의 (A-20):** `--workdir`를 지정하지 않으면 작업이 Hermes 설치 디렉터리에서 실행됩니다.
> 상대 경로로 파일을 생성하거나 읽는 작업에서 예상치 못한 경로 문제가 발생합니다.
> 파일 I/O가 있는 모든 cron 작업에는 반드시 `--workdir /절대/경로`를 지정하십시오.
>
> **추가 주의 — 순차 실행:** `--workdir`를 지정한 작업은 **순차 실행**됩니다 (병렬 실행 불가).
> workdir 전환이 프로세스 전역 상태를 변경하기 때문입니다.
> workdir 없는 작업들은 여전히 병렬 실행됩니다.

### 주의 3: macOS launchd PATH 문제 (A-15)

> **주의 (macOS):** `hermes gateway install`로 launchd 서비스를 설치한 후
> Homebrew나 nvm으로 새 도구를 설치하면, launchd의 PATH에 새 도구가 포함되지 않습니다.
> 새 도구 설치 후 `hermes gateway install`을 재실행하여 PATH를 갱신하십시오.

### 주의 4: gateway 없이 스케줄 자동 실행 불가

> **참고:** `hermes cron create`로 작업을 등록할 수 있지만,
> 실제 **스케줄 자동 실행**은 gateway 프로세스가 필요합니다.
> `hermes gateway` 또는 `hermes gateway install`로 게이트웨이를 먼저 시작하십시오.
>
> `hermes cron run <name>`으로 즉시 수동 실행은 gateway 없이도 가능한 것으로 설명되어 있으나,
> gateway 독립 실행 여부는 아직 로컬에서 확인되지 않았습니다.

> **검증 필요:** `hermes cron run <name>`이 gateway 프로세스 없이 독립적으로 실행 가능한지 로컬에서 확인하십시오.

## 다음 단계

이전 챕터: [Ch.11 MCP 연동](../11-mcp/index.md)

다음 챕터: [Ch.13 서브에이전트](../13-subagents/index.md)
