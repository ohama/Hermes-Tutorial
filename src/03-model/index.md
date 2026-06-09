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
