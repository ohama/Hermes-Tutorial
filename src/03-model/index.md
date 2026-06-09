# 모델 설정

> **이 챕터를 시작하기 전에**
> - Ch.1 설치가 완료되어 있어야 합니다 (`which hermes`가 경로를 반환해야 함).
> - 공급자 API 키 또는 Nous Portal 계정이 필요합니다.

## 개요

Hermes는 40개 이상의 모델 공급자를 지원합니다. 이 챕터에서는 `hermes model` 마법사와
`hermes setup --portal`을 사용하여 공급자와 API 키를 설정하고, config.yaml의 모델 섹션을
이해하며, 64K 최소 컨텍스트 요건을 충족하는 모델을 선택하는 방법을 안내합니다.

---

## hermes model 마법사

`hermes model`은 공급자 추가·API 키 설정·OAuth 흐름을 안내하는 대화형 마법사입니다.

```bash
# 검증: hermes rolling, 2026-06-09
hermes model
```

마법사가 실행되면:
- 새 공급자를 추가하거나
- 기존 공급자의 API 키를 업데이트하거나
- OAuth 인증 흐름을 진행할 수 있습니다.

> **중요:** `hermes model`은 **활성 채팅 세션 밖** 터미널에서 실행해야 합니다.
> 채팅 세션 내 `/model` 명령은 이미 설정된 공급자 간 전환만 가능합니다
> (새 공급자 추가 불가 — 주의 5 참조).

---

## 가장 쉬운 시작: hermes setup --portal

신규 사용자에게 권장하는 방법입니다.

```bash
# 검증: hermes rolling, 2026-06-09
hermes setup --portal
```

이 명령 하나로 세 가지가 동시에 처리됩니다:

1. **Nous Portal OAuth 로그인** — 브라우저가 열리며 인증 진행
2. **Nous를 기본 공급자로 설정** — hermes가 Nous Portal 모델을 사용
3. **Tool Gateway 활성화** — 도구 사용 기능 활성화

설정 완료 후 `hermes`를 실행하면 배너에 Nous 모델이 표시됩니다.

> **참고:** Nous Portal Tool Gateway의 무료/유료 경계는 공급자 정책에 따라 변경될 수 있습니다.
> 자세한 요금은 공식 Nous Portal 요금 페이지를 확인하십시오 (공식 문서 참조 권장).

---

## 주요 공급자

### Tier 1 — 상세 안내

| 공급자 | 인증 방식 | 특징 |
|--------|-----------|------|
| Nous Portal | OAuth (`hermes setup --portal`) | 가장 쉬운 시작; 300+ 모델; Tool Gateway 포함 |
| OpenRouter | API 키 (`OPENROUTER_API_KEY`) | 200+ 모델; 단일 키로 다양한 모델 접근 |
| Anthropic | API 키 (`ANTHROPIC_API_KEY`) | Claude 모델; **Claude Pro/Max 구독과 별개** (아래 주의 1 필독) |
| OpenAI | API 키 (`OPENAI_API_KEY` (관례; `hermes model` 마법사로 확인 권장)) | GPT 모델 |

> **OpenAI 환경변수명 주의:** `OPENAI_API_KEY`는 일반 관례이나, hermes 내부에서
> 사용하는 이름은 `hermes model` 마법사 실행 또는 설정 후 `~/.hermes/.env`에서
> 직접 확인할 것을 권장합니다.

### Tier 2 — 간략 언급

Google Gemini (OAuth 또는 `GOOGLE_API_KEY`/`GEMINI_API_KEY`), DeepSeek (`DEEPSEEK_API_KEY`),
NVIDIA NIM (`NVIDIA_API_KEY`), Ollama (로컬, 키 불필요), Custom Endpoint (OpenAI 호환).

40개 이상의 전체 공급자 목록은 공식 문서를 참조하십시오.

---

## API 키 저장 위치

Hermes는 보안을 위해 비밀 정보와 일반 설정을 분리 저장합니다.

| 저장 대상 | 파일 |
|----------|------|
| API 키 (비밀) | `~/.hermes/.env` |
| 비밀 외 설정 | `~/.hermes/config.yaml` |
| OAuth 자격증명 | `~/.hermes/auth.json` |

`hermes config set` 명령을 사용하면 값을 올바른 파일로 자동 라우팅합니다.
**`config.yaml`을 수동 편집하여 API 키를 직접 입력하지 마십시오** (아래 주의 3 참조).

저장 위치 확인:

```bash
# 검증: hermes rolling, 2026-06-09
cat ~/.hermes/.env
```

---

## config.yaml 모델 섹션

`~/.hermes/config.yaml`의 `model` 섹션으로 기본 공급자·모델·컨텍스트 길이·추론 수준을 설정합니다.

```yaml
# 검증: hermes rolling, 2026-06-09
model:
  default: nous/hermes-3-405b   # 또는 provider/model-id 형식
  provider: nous                 # nous|openrouter|anthropic|openai|custom|auto
  context_length: 131072
  reasoning_effort: medium       # none|minimal|low|medium|high|xhigh
```

현재 설정 확인:

```bash
# 검증: hermes rolling, 2026-06-09
hermes config show
```

---

## 64K 최소 컨텍스트 요건

Hermes Agent는 **최소 64,000 토큰**의 컨텍스트 윈도우를 가진 모델이 필요합니다.
이보다 작은 모델은 시작 시 거부됩니다.

**이유:** 매 API 호출마다 약 **13,935 토큰**의 고정 오버헤드가 소비됩니다.

| 오버헤드 항목 | 토큰 수 |
|--------------|---------|
| 도구 정의 | ~8,759 |
| 시스템 프롬프트 | ~5,176 |
| **합계** | **~13,935** |

64K 미만 모델은 실제 대화에 사용할 공간이 거의 없습니다.

**권장 모델:** Claude, GPT, Gemini, Qwen, DeepSeek 등 대부분의 호스티드 모델은 이 요건을 쉽게 충족합니다.

**Ollama 로컬 모델 사용 시:** 컨텍스트 크기를 명시적으로 지정해야 합니다.

```bash
# 검증: ollama (버전 로컬 확인 필요), 2026-06-09
# 예시: 64K 컨텍스트로 실행 (정확한 플래그는 ollama --help로 확인 필요)
ollama run llama3.2 --num-ctx 65536
```

> **검증 필요:** 정확한 플래그는 `ollama --help`로 확인하십시오
> (`--context-size` 또는 `--num-ctx`). 일반적으로 통용되는 형태는 `--num-ctx`이나,
> 로컬에서 실행하여 확인 후 사용하십시오.

---

## 한국어 UI (display.language: ko)

Hermes는 한국어(`ko`)를 포함한 다양한 언어 UI를 **부분적으로** 지원합니다.

**지원 언어 목록:** `en` | `zh` | `zh-hant` | `ja` | `de` | `es` | `fr` | `tr` | `uk` | `af` | `ko` | `it` | `ga` | `pt` | `ru` | `hu`

설정 방법:

```yaml
# 검증: hermes rolling, 2026-06-09
display:
  language: ko    # 승인 프롬프트 등 일부 정적 메시지를 한국어로 번역
                  # 에이전트 응답·로그·도구 출력은 영어 유지
```

> **참고:** 한국어(`ko`) 설정은 CLI 승인 프롬프트와 일부 게이트웨이 슬래시 커맨드 응답
> 등 **소수의 정적 사용자 메시지만** 번역합니다. 에이전트의 실제 응답, 로그, 도구 출력,
> 오류 메시지는 영어로 유지됩니다. 이것은 부분 한국어 지원입니다 — 완전한 한국어 UI가
> 아닙니다.

---

## 흔한 오류 / 주의

### 주의 1 — Claude Pro/Max 구독 ≠ API 접근 (CRITICAL)

> **경고 (Anthropic Claude 공급자 선택 시):**
> `claude.ai`의 Claude Pro 또는 Max **구독**은 Hermes와 같은 서드파티 도구의 API 접근을
> **허용하지 않습니다**. Hermes에서 Claude를 사용하려면 `console.anthropic.com`에서
> 별도 API 계정과 API 키(`sk-ant-...`)를 발급받아야 합니다.
> 대안: OpenRouter를 통해 Claude 모델에 접근할 수도 있습니다.

### 주의 2 — hermes setup Silent API Key Skip (CRITICAL, Issue #16394)

> **주의:** `~/.hermes/.env`에 해당 공급자 키 항목이 이미 존재하면 (값이 잘못되어도)
> `hermes setup`의 API 키 입력 프롬프트가 **조용히 건너뛰어집니다** (GitHub Issue #16394).
> 녹색 체크마크가 표시되어도 실제로는 잘못된 키가 남아 있을 수 있습니다.
>
> 해결 방법:
> - `~/.hermes/.env`를 직접 편집하여 해당 키 줄을 삭제한 후 `hermes setup` 재실행
> - 또는 `hermes config set PROVIDER_API_KEY NEW_KEY`로 직접 덮어쓰기

### 주의 3 — API 키를 config.yaml에 저장 금지

> **경고:** API 키는 반드시 `~/.hermes/.env`에 저장해야 합니다.
> `config.yaml`에 키를 직접 입력하면 git 추적 시 키가 노출될 위험이 있습니다.
> `hermes model` 마법사나 `hermes auth add`를 사용하면 자동으로 `.env`에 저장됩니다.

### 주의 4 — 64K 미만 모델 선택 시

> **주의:** 컨텍스트 윈도우가 64,000 토큰 미만인 모델은 Hermes가 시작 시 거부합니다.
> 무료 또는 소형 모델 선택 시 공급자의 모델 사양에서 context window를 확인하십시오.
> 고정 오버헤드(약 13,935 토큰)가 매 API 호출마다 소비됩니다.

### 주의 5 — /model과 hermes model의 차이

> **참고:** 채팅 세션 내 `/model` 명령은 **이미 설정된 공급자 간 전환만** 가능합니다.
> 새 공급자를 추가하려면 세션을 종료(`Ctrl+D`)하고 터미널에서 `hermes model`을 실행하십시오.

---

## 다음 단계

이 챕터에서 공급자를 연결하고 API 키를 설정했습니다.
[Ch.2 첫 실행](../02-first-run/index.md)으로 돌아가 첫 대화를 완료하거나,
이미 완료했다면 다음 단계로 넘어가십시오.

이제 Hermes를 설치·실행·연결했습니다. 다음 단계에서는 컨텍스트 관리, 메모리, 도구 활용,
게이트웨이(Telegram, Discord, Slack 등), 그리고 서버 배포를 다룹니다. 각 주제는 이후
챕터에서 순서대로 안내합니다.
