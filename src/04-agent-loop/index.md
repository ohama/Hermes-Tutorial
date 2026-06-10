# 에이전트 루프

> **이 챕터를 시작하기 전에**
> - [Ch.2 첫 실행](../02-first-run/index.md)을 완료하고 `hermes` 대화 경험이 있어야 한다.
> - 이 챕터는 **개념 위주**이며 새로운 설치 명령이 없다. AIAgent 내부 동작 원리를 이해하는 것이 목표다.

---

## 개요

Hermes의 모든 진입점(CLI, Gateway, ACP, API)은 하나의 공통 엔진인 **AIAgent**를 공유한다. `run_agent.py`에 구현된 AIAgent는 플랫폼에 무관하게 동작하는 오케스트레이션 엔진으로, 대화 한 턴이 완전히 끝난 후에야 다음 입력을 받는 **동기(synchronous)** 방식으로 작동한다.

이 챕터에서는 다음을 이해한다:

- AIAgent 코어 루프의 공식 사이클 **Assemble → Call → Parse → Execute → Persist → Loop**의 각 단계
- 3티어 프롬프트 시스템이 시스템 프롬프트를 어떻게 조립하는지 (요약)
- 3가지 API 모드와 그 차이
- 툴 디스패치 방식(직접 실행 vs ThreadPoolExecutor)
- 루프 종료 조건과 `max_turns` 설정

---

## 개념: AIAgent 코어와 공식 루프 사이클

### 공식 사이클

공식 개발자 문서에서 AIAgent의 루프 사이클은 다음과 같이 명명된다:

> **Assemble → Call → Parse → Execute → Persist → Loop**

이 프레이밍이 Hermes 공식 문서의 유일한 표현이다. 공식 문서에는 다른 에이전트 루프 표현이 존재하지 않는다.

### 루프 단계 상세 (`run_conversation()` 기준)

아래는 `run_conversation()` 내부의 9단계 흐름이다 (공식 agent-loop 개발자 가이드 기준):

```
1. task_id 생성 (없는 경우)
2. user 메시지를 대화 이력에 추가
3. 시스템 프롬프트 빌드 / 캐시 재사용 (아직 빌드되지 않은 경우)
4. 컨텍스트 50% 초과 시 선제적 압축 실행 (API 호출 전)
5. 대화 이력에서 API 메시지 구성
6. 에페머럴 레이어 주입 (budget warning 등)
7. 캐싱 마커 적용 (Anthropic 전용)
8. 인터럽트 가능 API 호출 (스트리밍)
9. 응답 파싱:
   - tool_calls 있음 → 툴 실행, 결과를 이력에 추가, step 5로 이동
   - 텍스트 응답 → 세션 저장 + 메모리 플러시 → 최종 응답 반환
```

### ASCII 다이어그램

```
사용자 입력
     │
     ▼
task_id 생성
     │
     ▼
시스템 프롬프트 조립 (3티어)
     │
     ├──► 50% 초과? → 컨텍스트 압축 먼저 실행
     │
     ▼
API 메시지 구성 (대화 이력 + 에페머럴 레이어)
     │
     ▼
LLM API 호출 (스트리밍, 인터럽트 가능)
     │
     ├── tool_calls 있음? ──► 툴 디스패치
     │                          ├── 단일: 직접 실행
     │                          └── 복수: ThreadPoolExecutor(≤8)
     │                          │
     │                          └──► 결과를 이력에 추가
     │                                    │
     │                          ◄─────────┘ (루프)
     │
     └── 텍스트 응답 ──► 세션 저장 + 메모리 플러시 → 완료
```

---

## 개념: 3티어 프롬프트 시스템 (요약)

AIAgent의 시스템 프롬프트는 **3티어 캐시드 시스템 프롬프트** 구조로 조립된다. 각 티어는 캐시 안정성과 갱신 주기가 다르다:

| 티어 | 내용 | 캐시 안정성 |
|------|------|------------|
| **Stable** (안정 계층) | 에이전트 정체성(SOUL.md), 툴 가이던스, 스킬 인덱스, 환경 힌트 | 세션 간 높음 — 캐시 prefix로 유지 |
| **Context** (컨텍스트 계층) | 선택적 system_message + 프로젝트 컨텍스트 파일 (단 1개) | 프로젝트 변경 시에만 바뀜 |
| **Volatile** (휘발성 계층) | MEMORY.md 스냅샷 + USER.md 스냅샷 + 타임스탬프 + 세션 메타 | 매 세션마다 변경 가능 |

**최종 조립 순서:** `stable → context → volatile`

10개 레이어의 세부 구성 순서와 컨텍스트 파일 우선순위는 [Ch.5 컨텍스트 파일](../05-context-files/index.md)에서 자세히 다룬다.

---

## 개념: 세 가지 API 모드

AIAgent는 연결된 LLM 공급자에 따라 세 가지 API 모드를 지원한다. 세 모드 모두 내부적으로는 동일한 OpenAI 스타일 메시지 포맷으로 수렴한다:

| 모드 | 엔드포인트 타입 | 클라이언트 라이브러리 |
|------|----------------|----------------------|
| `chat_completions` | OpenAI 호환 (OpenRouter, 커스텀) | `openai.OpenAI` |
| `codex_responses` | OpenAI Codex / Responses API | `openai.OpenAI` (Responses 포맷) |
| `anthropic_messages` | 네이티브 Anthropic Messages API | `anthropic.Anthropic` 어댑터 |

---

## 개념: 툴 디스패치

에이전트가 응답으로 tool_calls를 받으면, 호출 수에 따라 실행 방식이 달라진다:

- **단일 툴 호출:** 직접(동기) 실행
- **복수 툴 호출:** `ThreadPoolExecutor` (최대 8 워커) — 동시 실행
- **인터랙티브 툴 예외:** `clarify`, `todo`, `memory`, `session_search`, `delegate_task`는 상태 유지 또는 사용자 상호작용이 필요하므로 항상 **순차 실행**

각 툴의 실행 흐름은 다음 6단계를 따른다:

1. Tool Registry에서 핸들러 해석
2. `pre_tool_call` 플러그인 훅 실행
3. 위험 여부 확인 → 필요 시 승인 게이트
4. 인자 + task_id로 실행
5. `post_tool_call` 플러그인 훅 실행
6. 툴 결과를 대화 이력에 추가

---

## 실습: 루프를 직접 관찰하기

다음 명령으로 AIAgent 루프가 실제로 툴을 호출하고 결과를 이력에 추가하는 과정을 관찰할 수 있다.

**루프 관찰 (terminal 툴 호출 → 결과 추가 → 텍스트 응답 순서 확인):**

```bash
# 검증: hermes rolling, 2026-06-09
# 에이전트 루프를 직접 관찰하려면:
hermes chat -q "현재 디렉터리의 파일을 나열하고 가장 큰 파일을 알려줘"
```

실행하면 에이전트가 `terminal` 툴을 호출하고, 결과를 이력에 추가한 뒤 텍스트 응답을 반환하는 루프를 확인할 수 있다.

**툴 호출 진행 출력 예시:**

```
# [로컬 실행 후 캡처 필요 — 툴 호출 진행 출력]
```

> **검증 필요:** 이 출력은 `display.tool_progress` 설정과 터미널 스킨에 따라 다르므로 로컬 실행 후 캡처로 교체해야 한다.

**반복 횟수 제한 설정 (기본값 90):**

```bash
# 검증: hermes rolling, 2026-06-09
# 반복 횟수 제한 설정 (기본값 90):
hermes config set agent.max_turns 50
```

---

## 심화: 종료 조건과 콜백/폴백

### 루프 종료 조건

| 조건 | 동작 |
|------|------|
| 텍스트 응답 수신 | `final_response` 반환, 세션 저장 |
| `max_turns` 90회 소진 | 작업 요약 반환 후 중단 (기본값 `agent.max_turns: 90`) |
| 사용자가 `/stop` 전송 | 활성 API 스레드 중단, 대기 중 응답 폐기 |

### 콜백 훅 (developer 레벨)

실행 중 8종의 콜백 훅이 실행된다: `tool_progress_callback`, `thinking_callback`, `reasoning_callback`, `clarify_callback`, `step_callback`, `stream_delta_callback`, `tool_gen_callback`, `status_callback`. 이 훅들은 진입점별로 활용 방식이 다르다(CLI는 디스플레이용, Gateway는 플랫폼 전달용). 개발자 수준 상세 내용이므로 이 튜토리얼 범위를 벗어난다.

### 폴백 동작

429 / 5xx / 401 오류 발생 시, 에이전트는 `fallback_providers` 목록을 순서대로 시도한다. 비전, 압축 등 보조 작업은 독립된 폴백 체인을 가진다.

---

## 흔한 오류 / 주의

> **참고: AIAgent는 동기(synchronous) 엔진입니다.**
> 한 번의 대화 턴이 완전히 완료된 후에야 다음 입력을 받습니다.
> 응답이 느리게 느껴진다면, 도구가 실행 중인 것이며 루프가 작동하고 있는 것입니다.

> **주의: `agent.max_turns` (기본 90)를 초과하면 에이전트가 작업 요약을 반환하고 중단됩니다.**
> 복잡한 장기 작업의 경우 `max_turns`를 늘리거나 서브에이전트(`delegate_task`)를 활용하십시오.

---

## 다음 단계

- 이전: [Ch.3 모델 설정](../03-model/index.md)
- 다음: [Ch.5 컨텍스트 파일](../05-context-files/index.md)
