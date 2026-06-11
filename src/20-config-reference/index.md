# config.yaml 레퍼런스

> 이 챕터를 시작하기 전에
>
> 이 페이지는 `~/.hermes/config.yaml`의 키를 한곳에 모은 부록입니다.
> 각 명령어의 `--help` 출력, 설정 명령어에 대한 설명은 [Ch.19 CLI 레퍼런스](../19-cli-reference/index.md)를 참고하십시오.
> 설정 문제 해결은 [Ch.21 마스터 트러블슈팅 색인](../21-troubleshooting/index.md)을 참고하십시오.

---

> 참고: 이 레퍼런스는 공식 문서와 튜토리얼 검증 과정에서 확인된 키만 수록합니다.
> Hermes는 롤링 릴리스 방식으로 배포되므로 키 이름과 기본값이 변경될 수 있습니다.
> `hermes config show`로 현재 실제 설정값을 항상 확인하십시오.
> `> 검증 필요` 표시 항목은 로컬에서 추가 확인이 필요합니다.

---

## .env vs config.yaml 분리 원칙

Hermes 설정은 **비밀과 일반 설정을 명확히 분리**합니다:

```
비밀 (API 키, 봇 토큰)  → ~/.hermes/.env
비밀 외 설정            → ~/.hermes/config.yaml
OAuth 자격증명          → ~/.hermes/auth.json
```

**해결 우선순위 (높을수록 우선):**

| 순위 | 소스 | 예시 |
|------|------|------|
| 1 | CLI 인수 | `--model`, `--backend` |
| 2 | `~/.hermes/config.yaml` | 비밀 외 모든 설정 |
| 3 | `~/.hermes/.env` | API 키, 봇 토큰 |
| 4 | 내장 기본값 | — |

**환경변수 치환:** config.yaml 내에서 `${VAR_NAME}` 문법을 사용합니다. 중괄호가 필수이며, `$VAR_NAME`(중괄호 없음)은 치환되지 않습니다.

```yaml
# 검증: hermes rolling, 2026-06-11
auxiliary:
  vision:
    api_key: ${GOOGLE_API_KEY}   # 중괄호 필수; $GOOGLE_API_KEY는 치환 안 됨
```

**공식 예시 파일:** `https://github.com/NousResearch/hermes-agent/blob/main/cli-config.yaml.example`

---

## display

표시 언어 및 플랫폼별 진행 상황 출력 방식을 설정합니다.

```yaml
# 검증: hermes rolling, 2026-06-11
display:
  language: ko            # 지원 값: en|zh|zh-hant|ja|de|es|fr|tr|uk|af|ko|it|ga|pt|ru|hu
                          # 주의: 승인 프롬프트 등 일부 정적 메시지만 번역
                          # 에이전트 응답·로그·도구 출력은 영어 유지
  platforms:
    telegram:
      tool_progress: verbose   # off | brief | verbose
    discord:
      tool_progress: verbose   # off | brief | verbose
```

| 키 | 용도 | 기본값 | 허용값 |
|----|------|--------|--------|
| `display.language` | UI 표시 언어 | `en` | en, zh, zh-hant, ja, de, es, fr, tr, uk, af, ko, it, ga, pt, ru, hu |
| `display.platforms.<플랫폼>.tool_progress` | 도구 실행 진행 표시 수준 | `verbose` | off, brief, verbose |

> **주의:** `ko` 설정은 승인 프롬프트 등 일부 정적 메시지만 한국어로 표시합니다. 에이전트 응답·로그·도구 출력은 영어로 유지됩니다.

---

## model

기본 모델과 공급자를 설정합니다.

```yaml
# 검증: hermes rolling, 2026-06-11
model:
  default: nous/hermes-3-405b   # provider/model-id 형식
  provider: nous                 # nous|openrouter|anthropic|openai|custom|auto
  context_length: 131072         # 최소 64000 필수
  reasoning_effort: medium       # none|minimal|low|medium|high|xhigh
```

| 키 | 용도 | 기본값 | 허용값 |
|----|------|--------|--------|
| `model.default` | 기본 모델 (provider/model-id 형식) | `nous/hermes-3-405b` | 각 공급자가 지원하는 모델 ID |
| `model.provider` | 모델 공급자 | `nous` | nous, openrouter, anthropic, openai, custom, auto |
| `model.context_length` | 컨텍스트 윈도우 크기 | — | 최소 64000 필수 |
| `model.reasoning_effort` | 추론 수준 | `medium` | none, minimal, low, medium, high, xhigh |

> **주의:** `context_length`가 64,000 미만이면 에이전트가 시작을 거부합니다.

---

## agent

에이전트 루프 동작과 커스텀 퍼소날리티를 설정합니다.

```yaml
# 검증: hermes rolling, 2026-06-11
agent:
  max_turns: 90               # 루프 최대 반복 횟수 (기본값: 90)
  personalities:              # 커스텀 퍼소날리티 정의
    codereviewer: >
      You are a meticulous code reviewer. Focus on correctness,
      security, and idiomatic patterns. Be concise and direct.
```

| 키 | 용도 | 기본값 | 허용값 |
|----|------|--------|--------|
| `agent.max_turns` | 에이전트 루프 최대 반복 횟수 | `90` | 양의 정수 |
| `agent.personalities.<이름>` | 커스텀 퍼소날리티 프롬프트 정의 | — | 임의의 프롬프트 문자열 |

> **중요:** `agent.disabled_toolsets` 키는 **존재하지 않습니다.** 이 키를 config.yaml에 추가해도 조용히 무시됩니다. 툴셋 비활성화는 `toolsets:` 배열 또는 `hermes tools` UI로 설정하십시오. 아래 [미존재 키 표](#미존재-키-혼동-방지)를 참고하십시오.

---

## memory

에이전트 메모리 시스템을 설정합니다.

```yaml
# 검증: hermes rolling, 2026-06-11
memory:
  memory_enabled: true          # MEMORY.md 활성화 (기본: true)
  user_profile_enabled: true    # USER.md 활성화 (기본: true)
  memory_char_limit: 2200       # MEMORY.md 최대 문자 수 (~800 토큰)
  user_char_limit: 1375         # USER.md 최대 문자 수 (~500 토큰)
  provider: honcho              # 외부 메모리 프로바이더 (선택; 기본: 내장)
```

| 키 | 용도 | 기본값 | 허용값 |
|----|------|--------|--------|
| `memory.memory_enabled` | MEMORY.md 파일 활성화 | `true` | true, false |
| `memory.user_profile_enabled` | USER.md 파일 활성화 | `true` | true, false |
| `memory.memory_char_limit` | MEMORY.md 최대 문자 수 | `2200` | 양의 정수 |
| `memory.user_char_limit` | USER.md 최대 문자 수 | `1375` | 양의 정수 |
| `memory.provider` | 외부 메모리 프로바이더 | 내장 | honcho, (기타 프로바이더) |

**메모리 파일 경로:**

| 파일 | 경로 | 한도 |
|------|------|------|
| 에이전트 메모 | `~/.hermes/memories/MEMORY.md` | 2,200자 (~800 토큰) |
| 사용자 프로파일 | `~/.hermes/memories/USER.md` | 1,375자 (~500 토큰) |
| 에이전트 정체성 | `~/.hermes/SOUL.md` | 전역 (프로젝트와 무관) |

> **참고:** `memory.provider: honcho`는 존재하지만, Honcho 세부 설정은 `~/.honcho/config.json`에서 관리됩니다. Honcho 계정은 honcho.dev에서 별도 생성이 필요합니다.

---

## compression

컨텍스트 압축 동작을 설정합니다. 긴 세션에서 토큰 비용을 관리하는 핵심 설정입니다.

```yaml
# 검증: hermes rolling, 2026-06-11
compression:
  enabled: true
  threshold: 0.50       # 컨텍스트의 50% 사용 시 압축 트리거 (기본값)
  target_ratio: 0.20    # 압축 후 최근 20% 유지 (기본값)
  protect_last_n: 20    # 최근 20개 메시지 항상 보존 (기본값, 변경 가능)
  # protect_first_n: 3  ← 하드코딩됨 — 설정 불가
```

| 키 | 용도 | 기본값 | 허용값 |
|----|------|--------|--------|
| `compression.enabled` | 자동 압축 활성화 | `true` | true, false |
| `compression.threshold` | 압축 트리거 임계값 (컨텍스트 사용 비율) | `0.50` | 0.0–1.0 |
| `compression.target_ratio` | 압축 후 유지할 최근 컨텍스트 비율 | `0.20` | 0.0–1.0 |
| `compression.protect_last_n` | 압축에서 제외할 최근 메시지 수 | `20` | 양의 정수 |
| `compression.protect_first_n` | **설정 불가** — 하드코딩 3 | `3` | — |

**보조 압축 모델 설정:**

```yaml
# 검증: hermes rolling, 2026-06-11
auxiliary:
  compression:
    model: ""         # 빈 칸 = 메인 모델 사용 (저비용 모델 지정 가능)
    provider: "auto"
    base_url: null
  vision:
    api_key: ${GOOGLE_API_KEY}   # 환경변수 치환 시 중괄호 필수
```

> **팁:** 비용 절감을 위해 `compression.threshold: 0.30`으로 낮추거나, 보조 압축 모델로 저비용 모델(예: Gemini Flash)을 지정할 수 있습니다.

---

## toolsets / custom_toolsets

에이전트가 사용할 툴셋을 설정합니다.

```yaml
# 검증: hermes rolling, 2026-06-11
# 특정 플랫폼의 툴셋 제한 (전체 대신 필요한 것만)
toolsets:
  - hermes-cli        # 기본 전체 툴셋 이름 (검증 필요: 정확한 이름)
  - kanban            # 명시적 활성화 필요 (기본 비활성)

# 커스텀 툴셋 조합 정의
custom_toolsets:
  data-science:
    - file
    - terminal
    - code_execution
    - web
    - vision
```

**Tool Gateway per-tool 설정:**

```yaml
# 검증: hermes rolling, 2026-06-11
web:
  backend: firecrawl
  use_gateway: true   # true = Nous Gateway 라우팅 / false = 직접 API 키 사용
```

| 키 | 용도 | 기본값 | 허용값 |
|----|------|--------|--------|
| `toolsets` | 활성화할 툴셋 목록 | — | 툴셋 이름 배열 |
| `custom_toolsets.<이름>` | 커스텀 툴셋 정의 | — | 기존 툴셋 이름 배열 |
| `web.use_gateway` | Tool Gateway 라우팅 여부 | `false` | true, false |

> **검증 필요:** 기본 CLI 플랫폼의 `toolsets:` 키의 정확한 기본값 이름(`hermes-cli`인지 다른 이름인지)은 `hermes config show`로 확인 필요.

> **참고:** `kanban` 툴셋은 기본 비활성이며, `all/*` 와일드카드에도 포함되지 않습니다. 사용하려면 반드시 명시적으로 활성화해야 합니다.

---

## approvals

도구 실행 승인 정책을 설정합니다.

```yaml
# 검증: hermes rolling, 2026-06-11
approvals:
  mode: manual             # manual (기본, 가장 안전) | smart | off
  timeout: 300             # 승인 대기 시간 (초) — 03-RESEARCH: 60, 05-RESEARCH: 300
  cron_mode: manual        # 헤드리스 cron: manual | auto (deny|approve로 기록된 연구도 있음)
  destructive_slash_confirm: true  # /clear, /new, /reset 전 확인
```

**승인 모드 설명:**

| 모드 | 동작 |
|------|------|
| `manual` | 위험한 명령 전 항상 사용자 승인 요청 (기본, 프로덕션 권장) |
| `smart` | 보조 LLM이 위험도 평가; 저위험 자동 승인, 고위험 자동 거부 |
| `off` | 모든 승인 체크 비활성화 (`HERMES_YOLO_MODE=true`와 동일) |

> **프로덕션 권장:** `approvals.mode: manual`을 유지하십시오. `mode: off`는 로컬 백엔드에서 절대 사용하지 마십시오 — YOLO 모드 빨간 배너가 표시됩니다.

> **검증 필요:** `timeout` 기본값이 60초인지 300초인지, `cron_mode` 값이 `deny|approve`인지
> `manual|auto`인지 두 연구 파일에서 다르게 기록되었습니다. 어느 한쪽도 단정하지 마십시오.
> `hermes config show`로 로컬에서 실제 값을 확인하십시오.

---

## security

보안 스캔과 MCP 오류 메시지 자격증명 난독화를 설정합니다.

```yaml
# 검증: hermes rolling, 2026-06-11
security:
  tirith_enabled: true       # Tirith 보안 스캔 (기본: true)
  tirith_timeout: 5          # 타임아웃 (초)
  tirith_fail_open: true     # 스캐너 실패 시 실행 허용 (기본: true)
  redact_secrets: true       # MCP 오류 메시지에서 자격증명 패턴 난독화 (기본: true)
  allow_private_urls: false  # RFC 1918 내부 URL 접근 차단 (기본: false)
  allow_lazy_installs: false # 런타임 의존성 설치 차단 (기본: false)
```

| 키 | 용도 | 기본값 | 허용값 |
|----|------|--------|--------|
| `security.tirith_enabled` | Tirith 보안 스캔 활성화 | `true` | true, false |
| `security.tirith_timeout` | Tirith 스캔 타임아웃 (초) | `5` | 양의 정수 |
| `security.tirith_fail_open` | 스캐너 실패 시 실행 허용 | `true` | true, false |
| `security.redact_secrets` | MCP 오류 메시지 자격증명 패턴 난독화 | `true` | true, false |
| `security.allow_private_urls` | RFC 1918 내부 URL 접근 허용 | `false` | true, false |
| `security.allow_lazy_installs` | 런타임 의존성 자동 설치 허용 | `false` | true, false |

### security.redact_secrets의 정확한 범위

`security.redact_secrets`는 **일반 PII 필터가 아닙니다.**

이 설정이 하는 일: MCP 오류 메시지에서 다음 자격증명 패턴을 감지하여 난독화합니다:
- `ghp_...` (GitHub Personal Access Token)
- `sk-...` (OpenAI/Anthropic API 키)
- Bearer 토큰

이 설정이 하지 않는 일: 이름, 이메일 주소, 전화번호 등 **일반 PII를 필터링하지 않습니다.** 별도의 "PII 리댁션 설정 키"는 존재하지 않습니다.

> **Hardline blocklist:** `approvals.mode: off`(YOLO 모드)에서도 `rm -rf /`, 포크 폭탄, 마운트된 루트 파티션에 대한 `mkfs.*` 등은 항상 차단됩니다.

---

## terminal

에이전트가 명령을 실행하는 터미널 백엔드를 설정합니다.

```yaml
# 검증: hermes rolling, 2026-06-11
terminal:
  backend: local             # local | docker | ssh | singularity | modal | daytona

  # Docker 백엔드 설정
  docker_image: "nikolaik/python-nodejs:python3.11-nodejs20"  # 기본값
  docker_mount_cwd_to_workspace: false
  docker_run_as_host_user: false    # true: 컨테이너가 호스트 UID로 실행
  docker_forward_env:               # 컨테이너에 전달할 호스트 환경변수
    - "GITHUB_TOKEN"
  docker_env: {}                    # 컨테이너 전용 환경변수
  docker_volumes: []                # 추가 볼륨 마운트
  docker_extra_args: []             # 추가 docker run 플래그
  container_cpu: 1                  # CPU 코어 (모든 컨테이너 백엔드)
  container_memory: 5120            # MB (기본: 5GB)
  container_disk: 51200             # MB (기본: 50GB)
  container_persistent: true        # 세션 간 컨테이너 유지
  docker_persist_across_processes: true
  docker_orphan_reaper: true

  # SSH 백엔드 설정
  persistent_shell: true            # SSH ControlMaster 멀티플렉싱

  # Modal 백엔드 설정 (MEDIUM — 키 이름 로컬 확인 권장)
  modal_image: "nikolaik/python-nodejs:python3.11-nodejs20"  # MEDIUM confidence

  # Daytona 백엔드 설정
  # container_disk: 10240            # Daytona 최대 10GB
```

| 키 | 용도 | 기본값 | 허용값 |
|----|------|--------|--------|
| `terminal.backend` | 터미널 백엔드 | `local` | local, docker, ssh, singularity, modal, daytona |
| `terminal.docker_image` | Docker 이미지 | `nikolaik/python-nodejs:python3.11-nodejs20` | 유효한 Docker 이미지 |
| `terminal.docker_run_as_host_user` | 호스트 UID로 실행 | `false` | true, false |
| `terminal.container_memory` | 컨테이너 메모리 (MB) | `5120` | 양의 정수 |
| `terminal.container_disk` | 컨테이너 디스크 (MB) | `51200` | 양의 정수 (Daytona 최대 10240) |
| `terminal.container_persistent` | 세션 간 컨테이너 유지 | `true` | true, false |
| `terminal.persistent_shell` | SSH ControlMaster 멀티플렉싱 | `true` | true, false |

> **검증 필요:** `terminal.modal_image` 키 이름과 기본값은 MEDIUM confidence입니다. Modal 계정으로 `hermes setup terminal`을 실행하여 실제 키 이름을 확인하십시오.

**SSH/원격 백엔드 환경변수 (`.env`에 저장):**

| 변수명 | 용도 | 백엔드 |
|--------|------|--------|
| `TERMINAL_SSH_HOST` | SSH 서버 주소 | ssh |
| `TERMINAL_SSH_USER` | SSH 사용자명 | ssh |
| `TERMINAL_SSH_PORT` | SSH 포트 (선택, 기본 22) | ssh |
| `TERMINAL_SSH_KEY` | SSH 키 경로 (선택) | ssh |
| `MODAL_TOKEN_ID` | Modal 인증 ID | modal |
| `MODAL_TOKEN_SECRET` | Modal 인증 시크릿 | modal |
| `DAYTONA_API_KEY` | Daytona API 키 | daytona |

---

## delegation

서브에이전트 위임 동작을 설정합니다.

```yaml
# 검증: hermes rolling, 2026-06-11
delegation:
  max_spawn_depth: 1           # 위임 중첩 허용 레벨 (기본: 1 = 평면)
  max_concurrent_children: 3  # 동시 서브에이전트 수 (기본: 3)
```

| 키 | 용도 | 기본값 | 허용값 |
|----|------|--------|--------|
| `delegation.max_spawn_depth` | 서브에이전트 중첩 깊이 | `1` | 양의 정수 |
| `delegation.max_concurrent_children` | 동시 서브에이전트 수 | `3` | 양의 정수 |

> **주의:** `max_spawn_depth`를 2 이상으로 올리면 서브에이전트가 추가 서브에이전트를 생성할 수 있습니다. 지수적 비용 증가에 주의하십시오.

---

## mcp_servers

외부 MCP(Model Context Protocol) 서버를 설정합니다.

```yaml
# 검증: hermes rolling, 2026-06-11
mcp_servers:
  <server-name>:
    # 연결 (stdio 또는 HTTP 중 하나)
    command: "npx"             # stdio 전용
    args: ["-y", "@mcp/..."]   # stdio 전용
    env: {}                    # stdio 전용
    url: "https://..."         # HTTP 전용
    headers: {}                # HTTP 전용

    # 보안
    ssl_verify: true
    client_cert: "/path/cert.pem"
    client_key: "/path/key.pem"
    auth: oauth                # OAuth 2.1 PKCE

    # 런타임 제어
    enabled: true
    timeout: 30
    connect_timeout: 10

    # 툴 필터링 (include가 exclude보다 우선)
    supports_parallel_tool_calls: false
    tools:
      include: [tool1, tool2]
      exclude: [tool3]
      resources: false
      prompts: false
```

| 키 | 용도 | 기본값 |
|----|------|--------|
| `command` + `args` + `env` | stdio 전송 방식 연결 | — |
| `url` + `headers` | HTTP 전송 방식 연결 | — |
| `ssl_verify` | SSL 인증서 검증 | `true` |
| `auth: oauth` | OAuth 2.1 PKCE 인증 | — |
| `enabled` | 서버 활성화 여부 | `true` |
| `timeout` | 요청 타임아웃 (초) | `30` |
| `tools.include` | 허용할 툴 목록 (화이트리스트) | — |
| `tools.exclude` | 차단할 툴 목록 | — |

> **중요:** `tools.include`와 `tools.exclude`를 동시에 설정하면 `include`가 우선합니다(`exclude`는 무시됩니다). 보안상 화이트리스트(`include`만 사용) 방식을 권장합니다.

---

## skills

외부 스킬 디렉터리를 추가합니다.

```yaml
# 검증: hermes rolling, 2026-06-11
skills:
  external_dirs:
    - "/path/to/additional/skills"
```

| 키 | 용도 | 기본값 |
|----|------|--------|
| `skills.external_dirs` | 추가 스킬 검색 디렉터리 목록 | — |

---

## cron / curator

크론 스케줄러와 스킬 큐레이터 동작을 설정합니다.

```yaml
# 검증: hermes rolling, 2026-06-11
cron:
  wrap_response: false              # 래퍼 헤더/푸터 억제
  script_timeout_seconds: 300       # 사전 실행 스크립트 시간 제한
```

```yaml
# 검증: hermes rolling, 2026-06-11
curator:
  enabled: true
  interval_hours: 168        # 7일 (기본값)
  stale_after_days: 30       # 30일 미사용 → stale
  archive_after_days: 90     # 90일 미사용 → archived
```

**cron 설정:**

| 키 | 용도 | 기본값 |
|----|------|--------|
| `cron.wrap_response` | 크론 출력 래퍼 헤더/푸터 포함 여부 | `false` |
| `cron.script_timeout_seconds` | 사전 실행 스크립트 타임아웃 (초) | `300` |

**curator 설정:**

| 키 | 용도 | 기본값 |
|----|------|--------|
| `curator.enabled` | 큐레이터 활성화 | `true` |
| `curator.interval_hours` | 큐레이터 실행 간격 (시간) | `168` (7일) |
| `curator.stale_after_days` | stale 판정 기준 (일) | `30` |
| `curator.archive_after_days` | archive 판정 기준 (일) | `90` |

> **참고:** 큐레이터는 비활동 기반으로 트리거됩니다 (2시간 유휴 + 7일 간격 모두 충족 시). `hermes curator run`으로 즉시 수동 실행할 수 있습니다.

---

## 미존재 키 (혼동 방지)

다음 키는 **config.yaml에 존재하지 않습니다.** 추가해도 조용히 무시되거나 오류가 발생할 수 있습니다.

| 키 | 상태 | 대안 |
|----|------|------|
| `agent.disabled_toolsets` | **존재하지 않음** — 조용히 무시됨 | `toolsets:` 배열 또는 `hermes tools` UI |
| `security.pii_redact` (또는 유사 키) | **존재하지 않음** — PII 필터는 별개 기능 없음 | — |
| `memory.provider` + Honcho 세부 키 | `memory.provider: honcho`는 존재; Honcho 세부 설정은 `~/.honcho/config.json` | — |

---

## .env 환경변수

비밀 값(API 키, 봇 토큰)은 `~/.hermes/.env`에 저장합니다. config.yaml에 직접 저장하지 마십시오.

**게이트웨이 보안:**

| 변수명 | 설명 |
|--------|------|
| `GATEWAY_ALLOW_ALL_USERS=true` | 전체 허용 — **절대 사용 금지** |
| `GATEWAY_ALLOWED_USERS` | 글로벌 허용 사용자 목록 |

**플랫폼별 환경변수:**

| 변수명 | 플랫폼 |
|--------|--------|
| `TELEGRAM_BOT_TOKEN` | Telegram |
| `TELEGRAM_ALLOWED_USERS` | Telegram |
| `TELEGRAM_HOME_CHANNEL` | Telegram |
| `TELEGRAM_WEBHOOK_URL` | Telegram (웹훅 모드) |
| `DISCORD_BOT_TOKEN` | Discord |
| `DISCORD_ALLOWED_USERS` | Discord |
| `SLACK_BOT_TOKEN` | Slack (xoxb-) |
| `SLACK_APP_TOKEN` | Slack Socket Mode (xapp-) |
| `SLACK_ALLOWED_USERS` | Slack |
| `WHATSAPP_ENABLED` | WhatsApp |
| `WHATSAPP_MODE` | WhatsApp |
| `WHATSAPP_ALLOWED_USERS` | WhatsApp |
| `SIGNAL_HTTP_URL` | Signal |
| `SIGNAL_ACCOUNT` | Signal |
| `SIGNAL_ALLOWED_USERS` | Signal |
| `EMAIL_ADDRESS` | Email |
| `EMAIL_PASSWORD` | Email |
| `EMAIL_IMAP_HOST` | Email |
| `EMAIL_SMTP_HOST` | Email |
| `EMAIL_ALLOWED_USERS` | Email |
| `API_SERVER_ENABLED` | API 서버 |
| `API_SERVER_KEY` | API 서버 인증 키 |
| `OPENROUTER_API_KEY` | OpenRouter |
| `ANTHROPIC_API_KEY` | Anthropic |
| `HONCHO_API_KEY` | Honcho |

**공식 .env 예시 파일:** `https://github.com/NousResearch/hermes-agent/blob/main/.env.example`

---

## 발행 전 로컬 확인 항목

이 레퍼런스의 다음 항목들은 발행 전 로컬에서 실제 값을 확인해야 합니다:

| 확인 항목 | 명령어 | 상태 |
|----------|--------|------|
| 설정 기본값 전체 | `hermes config show` | 검증 필요 |
| `approvals.timeout` 기본값 (60초 vs 300초) | `hermes config show \| grep timeout` | 검증 필요 |
| `approvals.cron_mode` 값 (`deny\|approve` vs `manual\|auto`) | `hermes config show \| grep cron_mode` | 검증 필요 |
| 기본 `toolsets:` 키 이름 | `hermes config show \| grep toolsets` | 검증 필요 |
| `terminal.modal_image` 키 존재 | `hermes setup terminal` (Modal 계정으로) | 검증 필요 |

---

## 다음 단계

- 이전: [Ch.19 CLI 레퍼런스](../19-cli-reference/index.md) — 전체 CLI 명령어·슬래시 커맨드 목록
- 다음: [Ch.21 마스터 트러블슈팅 색인](../21-troubleshooting/index.md) — 오류 증상별 해결 색인
