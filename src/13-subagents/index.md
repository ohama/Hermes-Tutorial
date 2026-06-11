# 서브에이전트

> **이 챕터를 시작하기 전에**
> - [Ch.9 스킬 시스템](../09-skills/index.md) — 스킬과 프로시저 메모리 개념
> - [Ch.12 크론 스케줄러](../12-cron/index.md) — 자동화 개념과 격리 세션 이해

## 개요

이 챕터는 **두 개의 별개지만 관련된** 기능을 다룬다.

1. **`delegate_task` — 서브에이전트 위임:** 병렬 추론 작업을 위한 블로킹 서브에이전트 호출. 독립 LLM 세션을 동시 최대 3개(기본값, 구성 가능) 병렬 실행한다.
2. **Kanban — 내구적 다중 에이전트 시스템:** 장기·재개 가능·human-in-the-loop 작업을 위한 SQLite 기반 작업 큐. 단순 옵션 툴셋이 아니라 고유의 데이터베이스·CLI·브라우저 대시보드·디스패처를 갖춘 완전한 협업 시스템이다.

독자는 자연어로 위임을 요청하고 Hermes가 `delegate_task`를 내부적으로 호출한다. Kanban은 CLI와 브라우저 대시보드(`hermes dashboard`)로 작업 흐름을 관찰하고 관리한다.

이 챕터를 마치면 독자는 병렬 추론 작업을 서브에이전트에 위임하고, Kanban 보드로 장기 다중 에이전트 작업을 추적·재개·인간 승인 처리할 수 있다 (성공 기준 #5, LEARN-04).

---

## 개념: `delegate_task` — 서브에이전트 위임

### 작동 원리

`delegate_task`는 에이전트가 **내부적으로 호출하는 툴**이다. 독자는 에이전트에게 자연어로 위임을 요청하면 되고, 에이전트가 판단하여 `delegate_task`를 호출한다.

```python
# 에이전트 내부 사용 — 독자는 자연어로 위임 요청
# 이 코드블록은 설명용이며 독자가 직접 실행하지 않는다
delegate_task(
    goal="목표 설명",
    context="서브에이전트가 알아야 할 모든 컨텍스트",
    toolsets=["web", "file"],
    max_iterations=30,          # 선택: 최대 반복 횟수
    model="anthropic/claude-...",   # 선택: 모델 오버라이드
    provider="anthropic"        # 선택: 프로바이더 오버라이드
)
```

**병렬 배치 모드** — 여러 서브태스크를 한 번에 위임:

```python
# 에이전트 내부 사용 — 독자는 자연어로 위임 요청
delegate_task(tasks=[
    {
        "goal": "WebAssembly 서버사이드 현황 조사",
        "context": "runtimes, WASI, cloud/edge 사례 중점",
        "toolsets": ["web"]
    },
    {
        "goal": "RISC-V 서버 칩 채택 현황 조사",
        "context": "서버 칩, 클라우드 프로바이더 채택 현황",
        "toolsets": ["web"]
    },
    {
        "goal": "실용적인 양자 컴퓨팅 응용 사례 조사",
        "context": "에러 교정, 실제 사용 사례, 주요 기업",
        "toolsets": ["web"]
    }
])
```

---

## 개념: 컨텍스트 격리와 동시성

### 컨텍스트 격리 모델

서브에이전트는 **완전히 새로운 대화**로 시작한다. 부모 에이전트의 대화 이력을 전혀 알지 못한다.

따라서:
- 부모는 `goal`과 `context` 파라미터에 서브에이전트가 필요한 **모든 정보를 명시적으로** 전달해야 한다.
- "이전에 말한 내용" 또는 "현재 프로젝트 상황" 같은 참조는 서브에이전트에게 통하지 않는다.
- 자식의 **최종 요약만** 부모에게 반환된다. 중간 추론 과정은 반환되지 않는다.
- 결과는 완료 순서와 무관하게 **입력 순서**로 정렬되어 반환된다.

### 동시성 제어

- 기본 최대 동시 서브에이전트: **3개** (구성 가능, 상한 없음)
- `ThreadPoolExecutor`로 병렬 실행
- 부모는 모든 자식이 완료될 때까지 블록된다
- 새 메시지 전송 또는 `/stop` 명령으로 활성 자식을 취소할 수 있다

### Leaf 서브에이전트 차단 툴

기본(leaf) 서브에이전트에서 다음 툴은 차단된다:

| 차단 툴 | 이유 |
|---------|------|
| `delegation` | 재귀적 위임 방지 |
| `clarify` | 서브에이전트는 사용자와 상호작용 불가 |
| `memory` | 메모리 격리 유지 |
| `code_execution` | 격리된 실행 환경 보호 |
| `send_message` | 플랫폼 메시지 발송 제한 |

### Orchestrator 패턴 설정

중첩 위임이 필요한 경우 `config.yaml`에서 depth를 늘릴 수 있다. `role="orchestrator"` 서브에이전트에게만 추가 위임 능력이 유지된다.

```yaml
# 검증: hermes rolling, 2026-06-10
delegation:
  max_spawn_depth: 2          # 위임 중첩 허용 레벨 (기본값 1)
  max_concurrent_children: 30 # 호출당 최대 병렬 워커 수
```

기본값 `max_spawn_depth: 1`은 평면 구조(1단계 서브에이전트만)를 의미한다.

---

## 개념: 언제 위임하고 언제 하지 말까

### 사용하기 좋은 경우

- **추론 집약적 서브태스크:** 디버깅, 코드 리뷰, 연구 합성처럼 독립적인 깊은 추론이 필요한 작업
- **컨텍스트가 넘칠 작업:** 중간 데이터가 많아 하나의 컨텍스트로 담기 어려운 작업
- **독립 병렬 워크스트림:** 서로 의존하지 않고 병렬로 완료 가능한 작업들

### 사용하지 말아야 하는 경우

- **단일 툴 호출** → 직접 툴 사용이 더 빠르고 저렴하다
- **기계적 다단계 작업** → `execute_code`가 적합하다
- **사용자 상호작용이 필요한 작업** → 서브에이전트는 `clarify` 툴을 사용할 수 없다
- **내구적 장기 작업** → Kanban 또는 `cronjob`을 사용하라

---

## 실습: 병렬 연구 위임하기

에이전트에게 다음과 같이 요청해 보자:

> "WebAssembly 서버사이드, RISC-V 서버 칩, 실용 양자 컴퓨팅 세 주제를 병렬로 조사해서 각각 요약해줘."

에이전트는 세 개의 서브에이전트를 생성해 동시에 실행하고, 입력 순서대로 요약 결과를 반환한다.

**예상 동작 순서:**

1. 에이전트가 세 개의 `delegate_task`를 `tasks=[...]` 배치로 호출
2. 세 서브에이전트가 `ThreadPoolExecutor`를 통해 병렬 실행됨
3. 모든 서브에이전트 완료 후 결과가 입력 순서(WebAssembly → RISC-V → 양자 컴퓨팅)로 반환됨
4. 에이전트가 세 요약을 통합해 최종 응답 제공

```
# [로컬 실행 후 캡처 필요 — 위임 시작/완료 메시지 및 각 서브에이전트 요약 출력]
```

> 검증 필요: 에이전트가 서브에이전트 위임을 시작할 때 표시하는 정확한 UI 메시지와 진행 표시를 로컬 실행 후 캡처하십시오.

---

## 개념: Kanban 다중 에이전트 시스템

### Kanban이란

Kanban은 단순히 "kanban 툴셋을 켜는 것"이 아니다. 고유의 SQLite 데이터베이스, CLI, 브라우저 대시보드, 디스패처를 갖춘 **내구적 다중 에이전트 협업 시스템**이다.

**데이터베이스:**
- 기본 보드: `~/.hermes/kanban.db`
- 명명된 보드: `~/.hermes/kanban/boards/<slug>/kanban.db`

**테이블 구조:**

| 테이블 | 역할 |
|--------|------|
| `tasks` | 태스크 본체 (제목, 상태, 담당자 등) |
| `task_links` | 태스크 간 의존 관계 |
| `task_comments` | 코멘트 및 노트 |
| `task_runs` | 에이전트 실행 이력 |
| `task_events` | Append-only 감사 로그 (모든 상태 변경 기록) |

### 태스크 상태 머신

```
triage → todo → ready → running → done
                                ↘ blocked
                                ↘ archived
```

태스크는 `triage`에서 시작하여 준비 상태(`ready`)가 되면 디스패처가 에이전트에 할당하고 `running`으로 전환한다. 인간 입력이 필요하면 `blocked`로 표시하고, 확인 후 `unblock`으로 재개한다.

### 초기화 및 대시보드

```bash
# 검증: hermes rolling, 2026-06-10
hermes kanban init           # 선택적 — 첫 kanban 명령 시 자동 초기화
hermes dashboard             # 브라우저 대시보드 열기 (http://127.0.0.1:9119)
```

### 핵심 CLI 명령

```bash
# 검증: hermes rolling, 2026-06-10
hermes kanban create "태스크 제목" --assignee profile --tenant project
hermes kanban show <id>            # 태스크 상세
hermes kanban complete <id>        # 완료 처리
hermes kanban block <id>           # 차단 표시 (human-in-the-loop)
hermes kanban unblock <id>         # 차단 해제
hermes kanban decompose <id>       # 서브태스크로 분해
hermes kanban runs <id>            # 실행 이력
hermes kanban watch --kinds completed,gave_up,timed_out  # 이벤트 모니터링
```

### 디스패치 (다중 프로파일 워커 실행)

```bash
# 검증: hermes rolling, 2026-06-10
hermes gateway start    # 임베디드 dispatcher가 모든 프로파일 태스크를 처리
```

### Kanban 툴셋 활성화

Kanban 툴셋은 일반 채팅 세션에서 **기본적으로 비활성화**되어 있다. 두 가지 방법으로 활성화한다.

**방법 1 — Orchestrator 프로파일 명시 활성화:**

```yaml
# 검증: hermes rolling, 2026-06-10
# config.yaml (orchestrator 프로파일)
toolsets:
  - hermes-cli
  - kanban            # 명시적으로 추가 (기본 비활성)
```

**방법 2 — 디스패처 자동 활성화:**

`hermes gateway start`로 디스패처를 실행하면, 디스패처가 워커 에이전트를 생성할 때 `HERMES_KANBAN_TASK` 환경 변수를 통해 Kanban 툴셋을 자동으로 활성화한다. 일반 채팅 세션에는 적용되지 않는다.

---

## 개념: Kanban vs `delegate_task` — 선택 기준

두 시스템은 대체 관계가 아니라 **상호 보완 관계**다. 작업 성격에 따라 적합한 도구가 다르다.

| 항목 | Kanban | `delegate_task` |
|------|--------|-----------------|
| 실행 모델 | fire-and-forget 작업 큐 | 블로킹 함수 호출 |
| 부모 동작 | 태스크 생성 후 즉시 반환 | 자식 완료까지 블록 |
| 지속성 | SQLite 내구적 행 | 컨텍스트 압축 시 소실 |
| 인간 참여 | 지원 (comment/block/unblock) | 미지원 |
| 재개 가능성 | block → unblock → retry | 없음 — 실패는 최종 |
| 용도 | 장기, 복잡, human-in-the-loop 작업 | 짧은 병렬 추론 작업 |

**요약:**
- 짧고 독립적인 병렬 추론 → **`delegate_task`**
- 장기, 재개 가능, 인간 참여가 필요한 작업 → **Kanban**

---

## 심화: 다중 에이전트 협업 패턴

Kanban은 다양한 다중 에이전트 협업 패턴을 지원한다.

- **Fan-out:** 동일 유형의 병렬 형제 태스크를 여러 에이전트에 분산
- **Pipeline (scout → editor → writer):** 순차 역할 체인 — 정보 수집 에이전트가 완료하면 편집 에이전트가 이어받음
- **Voting/quorum:** 다수의 에이전트가 독립 평가하고 집계자가 최종 결과를 합산
- **Long-running journals:** 세션 간 메모리를 축적하는 영속적 어시스턴트
- **Human-in-the-loop:** 워커가 `kanban_block`으로 사용자 입력을 기다리고, 확인 후 `unblock`으로 재개

**워커 에이전트 툴:**

워커 에이전트(디스패처가 생성)는 `kanban` 툴셋의 도구(`kanban_show`, `kanban_list`, `kanban_complete`, `kanban_block`, `kanban_heartbeat`, `kanban_comment`, `kanban_create`, `kanban_link`, `kanban_unblock`)를 사용하여 Kanban 보드와 상호작용한다. 독자는 이 툴을 직접 호출하지 않고 CLI와 대시보드로 결과를 관찰한다.

---

## 흔한 오류 / 주의

> **주의 1 — 서브에이전트 컨텍스트 블리드 없음**
>
> 서브에이전트는 부모의 대화 이력을 전혀 알지 못합니다.
> `goal`과 `context` 파라미터에 필요한 모든 정보를 명시적으로 전달해야 합니다.
> "알고 있는 내용"이나 "현재 프로젝트" 같은 참조 없이 완전히 자기 완결적인 컨텍스트를 작성하십시오.

> **주의 2 — delegate_task 비용**
>
> 각 서브에이전트는 독립적인 LLM 세션입니다.
> 3개 병렬 서브에이전트 = 최소 3× 기본 오버헤드(~14K 토큰/세션).
> 단순 반복 작업이나 단일 툴 호출에는 서브에이전트 대신 직접 툴을 사용하십시오.

> **주의 3 — max_spawn_depth 기본값 1**
>
> 기본 `max_spawn_depth: 1`에서 서브에이전트는 추가 서브에이전트를 생성할 수 없습니다.
> 중첩 위임이 필요한 경우 `delegation.max_spawn_depth: 2` 이상으로 설정하십시오.
> 단, 깊은 중첩은 지수적 비용 증가를 초래합니다.

> **주의 4 — Kanban과 delegate_task는 보완 관계**
>
> Kanban과 `delegate_task`는 서로를 대체하지 않습니다.
> 짧은 병렬 추론 작업에는 `delegate_task`를, 장기·재개 가능·인간 참여 작업에는 Kanban을 사용하십시오.
> 하나의 워크플로우 안에서 함께 사용할 수 있습니다.

> **주의 5 — Kanban 툴셋 기본 비활성**
>
> Kanban 툴셋은 일반 채팅 세션에서 기본적으로 비활성화되어 있습니다.
> `hermes gateway start`로 디스패처를 실행하면 워커 에이전트에 자동 활성화되고,
> 또는 `config.yaml` orchestrator 프로파일에 `toolsets: [..., kanban]`을 명시하여 활성화하십시오.
> `hermes dashboard`로 브라우저 대시보드를 열어 진행 상황을 시각적으로 확인할 수 있습니다.

---

## 다음 단계

이전 챕터: [Ch.12 크론 스케줄러](../12-cron/index.md) — 격리 에이전트 세션과 자동화 스케줄링.

이 챕터에서 다룬 `hermes gateway start`는 Kanban 디스패처를 실행하는 명령이기도 하지만, Hermes의 게이트웨이 시스템 전체의 입구이기도 하다. 다음 단계에서는 이 게이트웨이를 통해 Discord·Telegram 같은 플랫폼에 에이전트를 배포하고, 원격 백엔드와 연결하며, 멀티플랫폼 운영을 구성하는 방법을 다룬다. `hermes gateway start`로 Kanban 디스패처를 띄우는 것이 게이트웨이 운영의 첫 걸음이다.

다음: [Ch.14 메시징 게이트웨이](../14-gateways/index.md)
