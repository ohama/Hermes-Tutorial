# 툴 게이트웨이

> **이 챕터를 시작하기 전에**
> - Ch.4 에이전트 루프 개념을 이해하고 있어야 합니다 (Assemble→Call→Parse→Execute→Persist→Loop 사이클, 툴 디스패치).
> - `hermes` CLI가 설치되어 있어야 합니다.

## 개요

Hermes는 **약 71개의 내장 툴**을 **30개 이상의 툴셋**으로 묶어 제공합니다. 이 챕터에서는 다음을 다룹니다.

- **툴 레지스트리와 툴셋:** 내장 툴 카탈로그 구조와 카테고리
- **`hermes tools`:** 툴을 탐색하고 활성/비활성화하는 인터랙티브 UI
- **Nous Tool Gateway:** 웹검색·이미지 생성·TTS·브라우저를 구독 서비스로 라우팅하는 유료 게이트웨이
- **`use_gateway` 설정:** 툴별 게이트웨이/직접 키 전환
- **승인 모드와 보안:** 위험한 명령을 제어하는 `manual / smart / off` 모드와 Tirith 스캔

---

## 개념: 툴 레지스트리와 툴셋

### ~71개 툴, 30+ 툴셋

Hermes 툴 레지스트리에는 약 71개의 내장 툴이 있습니다. 실제로 등록되는 툴의 수는 설치된 능력(Python 패키지, 서비스 자격증명)과 현재 활성 플랫폼(CLI, Telegram, Discord 등)에 따라 달라집니다.

**카테고리별 주요 툴셋:**

| 카테고리 | 툴셋 | 주요 툴 |
|----------|------|---------|
| 파일 & 터미널 | `terminal`, `file` | `terminal`, `process`, `read_file`, `write_file`, `patch`, `search_files` |
| 웹 | `web`, `browser` | `web_search`, `web_extract`, `browser_navigate`, `browser_click`, `browser_snapshot`, `browser_vision` |
| AI | `vision`, `image_gen`, `tts`, `moa` | `vision_analyze`, `image_generate`, `text_to_speech`, `mixture_of_agents` |
| 에이전트 | `delegation`, `clarify`, `todo`, `kanban` | `delegate_task`, `clarify`, `todo`, `kanban_show` |
| 메모리 | `memory`, `session_search`, `skills` | `memory`, `session_search`, `skill_manage`, `skill_view` |
| 자동화 | `cronjob`, `messaging` | `cronjob`, `send_message` |
| 특수 | `homeassistant`, `computer_use`, `spotify` | `ha_call_service`, `computer_use`, `spotify_playback` |
| 플랫폼 | `discord`, `feishu_doc`, `feishu_drive`, `x_search`, `yuanbao` | `discord`, `feishu_doc_read`, `x_search` |

> **참고: `kanban` 툴셋은 opt-in 전용입니다.**
> `hermes chat --toolsets all` 또는 `--toolsets *` 와일드카드를 사용해도 `kanban` 툴셋은 포함되지 않습니다.
> kanban 기능을 사용하려면 `--toolsets kanban` 또는 config에서 명시적으로 지정해야 합니다.

---

## 실습: `hermes tools`로 도구 탐색·설정

### 인터랙티브 UI와 요약 출력

```bash
# 검증: hermes rolling, 2026-06-09
hermes tools               # 플랫폼별 인터랙티브 툴 설정 UI (curses 기반)
hermes tools --summary     # 현재 활성화된 툴 요약 출력 후 종료
```

`hermes tools`는 curses 기반 인터랙티브 UI를 열어 툴셋별 활성/비활성 상태를 확인하고 변경할 수 있게 합니다. `--summary` 플래그는 UI 없이 현재 활성 툴 목록을 출력하고 즉시 종료합니다.

```
# [로컬 실행 후 캡처 필요 — hermes tools --summary 출력은 환경에 따라 다름]
```

> 검증 필요: 위 출력 예시는 자리표시자입니다. `hermes tools --summary`의 실제 출력은 로컬 실행 후 캡처로 교체하십시오.

### 세션 내 툴 명령

실행 중인 hermes 세션에서 슬래시 커맨드로 툴셋을 제어할 수 있습니다.

```
/tools list                 # 현재 세션 활성 툴 목록
/tools disable browser      # 브라우저 툴셋 비활성화 (현재 세션)
/tools enable homeassistant # Home Assistant 툴셋 활성화 (현재 세션)
```

세션 내 변경은 현재 세션에만 적용됩니다. 영구 변경은 아래의 CLI 플래그 또는 config를 사용하십시오.

### 세션별 툴셋 선택 (CLI 플래그)

```bash
# 검증: hermes rolling, 2026-06-09
hermes chat --toolsets web,file,terminal   # 특정 툴셋만 활성화
hermes chat --toolsets debugging           # composite toolset (file + terminal + web)
hermes chat --toolsets all                 # 모든 툴셋 활성화 (kanban 제외)
```

### 커스텀 툴셋 정의 (config.yaml)

반복적으로 사용하는 툴셋 조합을 이름을 붙여 저장할 수 있습니다.

```yaml
# 검증: hermes rolling, 2026-06-09
custom_toolsets:
  data-science:
    - file
    - terminal
    - code_execution
    - web
    - vision
```

정의 후 `hermes chat --toolsets data-science`로 호출합니다.

### 툴셋 비활성화 (config.yaml — `toolsets` 배열)

특정 플랫폼의 기본 툴셋을 제한하려면 `toolsets` 배열을 직접 지정합니다.

```yaml
# 검증: hermes rolling, 2026-06-09
# CLI 기본 플랫폼의 툴셋을 제한하려면 toolsets 배열을 직접 지정:
toolsets:
  - hermes-cli   # 기본 전체 툴셋 대신 필요한 것만 나열
```

> **주의: `agent.disabled_toolsets` 키는 존재하지 않습니다.**
> 툴셋 비활성화 방법은 위의 `toolsets` 배열 지정, `hermes tools` 인터랙티브 UI, 또는 `/tools disable` 세션 커맨드입니다.
> 존재하지 않는 config 키는 조용히 무시되며 아무 효과도 없습니다.

---

## 개념: 툴 동시성

Ch.4 에이전트 루프와 일관되게, 여러 툴 호출이 한 번에 요청될 때 Hermes는 **ThreadPoolExecutor(최대 8 워커)**를 사용해 병렬로 실행합니다. 경로 스코프 동시성은 안전합니다 — 서로 다른 파일 경로에 대한 동시 호출은 안전하고, 같은 경로에 대한 호출은 레지스트리가 순서를 관리합니다.

단, **인터랙티브 툴은 항상 순차 실행**됩니다: `clarify`, `todo`, `memory`, `session_search`, `delegate_task`. 이 툴들은 사용자 인터랙션이나 상태 관리가 필요하기 때문입니다.

---

## 개념: Nous Tool Gateway — 유료 기능

Nous Tool Gateway는 웹검색, 이미지 생성, TTS, 브라우저 자동화를 구독 서비스로 라우팅합니다. 각 서비스별 개별 API 키 없이 이 기능들을 사용할 수 있습니다.

### 무료 vs. 유료 경계

| 기능 | 무료 계정 | 유료 구독 |
|------|-----------|-----------|
| 추론 (LLM 대화, 300+ 모델) | 가능 | 가능 |
| Tool Gateway (웹검색, 이미지, TTS, 브라우저) | **무료 툴 풀** (자동 부여, 한도 있음) | **전체 무제한** |
| 한도 초과 시 | 툴 오류 (`subscription required`) | 해당 없음 |

일부 계정은 첫 사용 시 자동으로 제한된 무료 툴 풀이 부여됩니다.

> **검증 필요: 무료 툴 풀의 정확한 할당량은 공식 문서에 명시되어 있지 않습니다.**
> 현재 정책은 [Nous Portal 가격 페이지](https://hermes-agent.nousresearch.com)에서 확인하십시오.

### Nous Portal 연결

```bash
# 검증: hermes rolling, 2026-06-09
hermes setup --portal      # OAuth 로그인 + Nous 공급자 설정 + Tool Gateway 활성화
hermes portal info         # 로그인 상태 + 구독 + 게이트웨이 라우팅 요약
hermes portal tools        # Tool Gateway 카탈로그 (툴별 라우팅 현황)
```

---

## 개념: 게이트웨이 기능과 `use_gateway` 설정

### per-tool `use_gateway` 설정

각 툴을 Nous Gateway를 통해 라우팅할지, 직접 API 키로 실행할지 개별적으로 지정할 수 있습니다.

```yaml
# 검증: hermes rolling, 2026-06-09
web:
  backend: firecrawl
  use_gateway: true    # true = Nous Gateway 라우팅 / false = 직접 API 키 사용
```

`use_gateway: true`는 직접 API 키 설정보다 우선합니다 — 두 설정이 모두 있을 때 `use_gateway: true`가 이깁니다.

### 웹검색

게이트웨이를 통한 웹검색은 Firecrawl 백엔드를 사용합니다 (에이전트 등급, 게이트웨이 경유 시 레이트 리밋 없음).

### 이미지 생성 (FAL.ai 9개 모델)

`image_generate` 툴은 FAL.ai를 통해 다음 9개 모델을 지원합니다.

| 모델 ID | 이름 |
|---------|------|
| `fal-ai/flux-2/klein/9b` | FLUX 2 Klein 9B **(기본값)** |
| `fal-ai/flux-2-pro` | FLUX 2 Pro |
| `fal-ai/z-image/turbo` | Z-Image Turbo |
| `fal-ai/nano-banana-pro` | Nano Banana Pro (Gemini 3 Pro) |
| `fal-ai/gpt-image-1.5` | GPT Image 1.5 |
| `fal-ai/gpt-image-2` | GPT Image 2 |
| `fal-ai/ideogram/v3` | Ideogram V3 |
| `fal-ai/recraft/v4/pro/text-to-image` | Recraft V4 Pro |
| `fal-ai/qwen-image` | Qwen Image |

### TTS (Text-to-Speech)

`text_to_speech` 툴은 OpenAI TTS voices를 사용합니다. Telegram 음성 메모, 오디오 파이프라인 등에 활용됩니다.

### 브라우저 자동화

`browser` 툴셋은 Browser Use를 통한 헤드리스 Chromium 세션을 제공합니다.

기본 프리미티브:
- `browser_navigate` — URL 이동
- `browser_click` — 요소 클릭
- `browser_type` — 텍스트 입력
- `browser_vision` — 현재 화면 스냅샷 + 비전 분석

게이트웨이 사용 시 **Browserbase 계정은 필요하지 않습니다.**

---

## 개념: 승인 모드와 보안

### 승인 모드 설정

```yaml
# 검증: hermes rolling, 2026-06-09
approvals:
  mode: manual         # manual (기본) | smart | off
  timeout: 60          # 승인 대기 시간 (초)
  cron_mode: deny      # 헤드리스 cron: deny | approve
  destructive_slash_confirm: true  # /clear, /new, /reset 전 확인
```

### 모드별 동작

| 모드 | 동작 |
|------|------|
| `manual` | 위험한 명령 전 항상 사용자 승인 요청 (기본값, 가장 안전) |
| `smart` | 보조 LLM이 위험도를 평가하여 저위험은 자동 승인, 고위험은 자동 거부, 불확실한 경우 수동 위임 |
| `off` | 모든 승인 체크 비활성화 (`HERMES_YOLO_MODE=true`와 동일) |

### Hardline 차단 목록

`off` 모드에서도 다음 명령들은 **어떤 설정에서도 항상 실행이 거부**됩니다.

- `rm -rf /` 및 유사 재귀 루트 삭제
- 포크 폭탄 (`:(){ :|:& };:` 류)
- 마운트된 루트 파티션에 대한 `mkfs.*`

### Tirith 보안 스캔

Tirith는 항상 켜져 있는 보안 스캐너입니다.

```yaml
# 검증: hermes rolling, 2026-06-09
security:
  tirith_enabled: true       # 기본값: true
  tirith_timeout: 5          # 타임아웃 (초)
  tirith_fail_open: true     # 스캐너 실패 시 실행 허용 (fail-open)
```

**Tirith가 탐지하는 패턴:**
- Homograph URL 스푸핑 (유니코드 혼동 문자로 만든 가짜 도메인)
- `curl | bash` 파이프 — 파이프-투-인터프리터 패턴
- 터미널 인젝션 공격 (이스케이프 시퀀스를 이용한 명령 주입)

---

## 흔한 오류 / 주의

**주의 1 — `approvals.mode: off` 위험성 (A-12, CRITICAL)**

> 경고: `approvals.mode: off` (또는 `HERMES_YOLO_MODE=true`)는
> 위험한 명령에 대한 모든 안전 프롬프트를 비활성화합니다.
> **로컬 백엔드에서 절대 사용하지 마십시오.**
> 컨테이너 백엔드(Docker/Modal/Daytona)에서 컨테이너가 보안 경계 역할을 할 때만 합리적입니다.
> 활성화 시 빨간색 배너 **"⚠ YOLO mode"** 가 표시됩니다.

**주의 2 — Tool Gateway 무료/유료 경계 (A-7)**

> 주의: Nous Portal 무료 계정은 추론(LLM 대화)에는 사용 가능하지만,
> Tool Gateway(웹검색, 이미지 생성, TTS, 브라우저)는 유료 구독 또는
> 무료 툴 풀 한도 내에서만 사용 가능합니다.
> `hermes portal info`로 현재 구독 상태를 확인하십시오.
> 구독 없이 도구를 호출하면 "subscription required" 오류가 발생합니다.

**주의 3 — `agent.disabled_toolsets` 설정 키 없음 (MEDIUM — 흔한 오해)**

> 참고: 툴셋 비활성화는 `config.yaml`의 `agent.disabled_toolsets` 키가 아닌
> `toolsets` 배열 또는 `hermes tools` 인터랙티브 UI를 통해 설정합니다.
> `agent.disabled_toolsets`는 존재하지 않는 키이며, 작성해도 조용히 무시됩니다.

---

## 다음 단계

이전 챕터: [Ch.6 메모리](../06-memory/index.md)

다음 챕터: [Ch.8 터미널 백엔드](../08-backends/index.md)
