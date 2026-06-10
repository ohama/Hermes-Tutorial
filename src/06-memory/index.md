# 메모리

> **이 챕터를 시작하기 전에**
> - [Ch.4 에이전트 루프](../04-agent-loop/index.md)를 완료했는지 확인하세요 — 루프 끝에서 메모리가 플러시되는 방식이 이 챕터와 직접 연결됩니다.
> - [Ch.5 컨텍스트 파일](../05-context-files/index.md)의 SOUL.md 개념을 숙지하세요 — SOUL.md는 메모리 파일과 다른 위치에 있습니다.

## 개요

Hermes의 메모리 시스템은 4계층으로 구성되어 있어, 단순한 대화 기록 이상의 기능을 제공합니다. 이 챕터에서는 독자가 실제로 관리하게 되는 핵심 레이어인 **코어 메모리 파일**, **SQLite FTS5 세션 이력 검색**, **압축 비용 관리**를 다룹니다.

이 챕터를 마치면 다음을 이해할 수 있습니다:
- `~/.hermes/memories/MEMORY.md`와 `~/.hermes/memories/USER.md`의 역할과 관리 방법
- `session_search` 툴이 어떻게 과거 대화를 검색하는지 (슬래시 커맨드와의 차이)
- 압축 설정으로 API 비용을 효율적으로 관리하는 방법

---

## 개념: 4계층 메모리 아키텍처

Hermes의 메모리는 네 개의 독립적인 레이어로 나뉩니다.

| 레이어 | 저장 위치 | 용량 | 로딩 방식 |
|--------|-----------|------|-----------|
| **Persistent Core Memory** | `~/.hermes/memories/MEMORY.md` + `USER.md` | MEMORY: ~2,200자(~800토큰), USER: ~1,375자(~500토큰) → 합계 ~1,300토큰 | 세션 시작 시 항상 동결 스냅샷으로 로드 |
| **Session History (FTS5)** | `~/.hermes/state.db` (SQLite) | 무제한 | `session_search` 툴로 온디맨드 쿼리 |
| **External Provider** | Honcho/Mem0/Hindsight 등 (선택 사항) | 제공자마다 다름 | Phase 4 범주 — 이 챕터에서는 이름만 언급 |
| **Procedural (Skills)** | `~/.hermes/skills/*.md` | 매칭 시 주입 | Phase 4 범주 |

이 챕터에서는 처음 두 레이어(Persistent Core Memory와 Session History)를 집중적으로 다룹니다. External Provider와 Procedural(Skills) 레이어는 Phase 4에서 자세히 다룹니다.

---

## 개념: 코어 메모리 파일 경로

Hermes의 코어 메모리 파일은 `~/.hermes/memories/` 하위 디렉터리에 위치합니다.

```
~/.hermes/
├── SOUL.md                    # 에이전트 정체성 (memories/ 아래가 아님 — Ch.5 참조)
├── memories/
│   ├── MEMORY.md              # 에이전트 메모 (2,200자 한도, ~800토큰)
│   └── USER.md                # 사용자 프로파일 (1,375자 한도, ~500토큰)
├── state.db                   # SQLite FTS5 세션 이력
└── skills/                    # 절차적 메모리 (Phase 4)
```

> **중요:** `MEMORY.md`와 `USER.md`는 반드시 `~/.hermes/memories/` 아래에 있습니다. `memories/` 하위 디렉터리를 생략한 경로는 잘못된 경로입니다 — 공식 메모리 문서가 명시적으로 정정한 사항입니다.

**각 파일의 역할:**

| 파일 | 용도 | 용량 한도 |
|------|------|-----------|
| `MEMORY.md` | 에이전트가 기억해야 할 내용: 사용자 선호도, 프로젝트 규약, 완료된 작업, 배운 해결책 | ~2,200자 (~800토큰) |
| `USER.md` | 사용자 프로파일: 직업, 선호 언어, 작업 스타일 등 | ~1,375자 (~500토큰) |

SOUL.md는 `~/.hermes/SOUL.md`에 위치하며 `memories/` 하위가 **아닙니다** — 에이전트의 정체성을 정의하는 다른 역할입니다.

---

## 개념: 내장 memory 툴

에이전트는 내부적으로 `memory` 툴을 사용해 MEMORY.md와 USER.md를 관리합니다. 대화 중 "이걸 기억해줘" 또는 "내 선호 설정을 저장해줘"와 같이 자연어로 요청하면 에이전트가 자율적으로 이 툴을 호출합니다.

**memory 툴의 세 가지 동작:**

| 동작 | 용도 |
|------|------|
| `add` | 새 메모리 항목 추가 |
| `replace` | 기존 항목 수정 (old_text 부분 문자열 매칭) |
| `remove` | 항목 삭제 (부분 문자열로 식별) |

파일 내 항목 구분자는 `§` (section sign, U+00A7)를 사용합니다.

**저장 대상 vs. 건너뛸 내용:**

| 저장 대상 | 건너뛸 내용 |
|-----------|-------------|
| 사용자 선호도 | 사소한 사실 |
| 환경 설정 | 코드 덤프 |
| 프로젝트 규약 | 세션 특화 내용 |
| 배운 해결책 / 우회법 | SOUL.md에 이미 있는 내용 |
| 완료된 작업 | 일시적 컨텍스트 |

**용량 관리:** 용량 한도에 도달하면 memory 툴이 오류를 반환합니다. 약 80% 차면 통합을 권장하며, 독자가 직접 파일을 편집할 수도 있습니다.

**보안:** 메모리 항목은 프롬프트 인젝션 패턴과 자격증명 유출 패턴에 대해 자동으로 보안 스캔됩니다.

---

## 개념: 메모리 플러시 타이밍

메모리 변경(add/replace/remove)은 세션 중 **즉시 디스크에 저장**됩니다. 하지만 변경 사항이 시스템 프롬프트에 반영되는 것은 **다음 세션부터**입니다.

이는 메모리 스냅샷이 **세션 시작 시 동결**되기 때문입니다 — 세션 내에서 에이전트가 MEMORY.md를 업데이트해도 현재 대화의 시스템 프롬프트에는 반영되지 않습니다.

> **주의 (가장 흔한 혼란):** 세션 중 에이전트가 MEMORY.md를 업데이트해도 현재 대화에는 반영되지 않습니다.
> 메모리 변경 사항은 다음 `hermes` 실행 시부터 시스템 프롬프트에 포함됩니다.
> 이것은 의도된 동작입니다 — 세션 내 컨텍스트 일관성을 보장합니다.

---

## 개념: 세션 검색 (session_search 툴)

`session_search`는 **에이전트 내부 툴**입니다 — 슬래시 커맨드(`/session_search`)가 아닙니다. 에이전트가 문맥상 과거 대화 검색이 필요하다고 판단할 때 자율적으로 호출합니다.

**트리거 방법:** 에이전트에게 자연어로 질문하면 됩니다:
- "지난주에 배포에 대해 얘기한 내용이 뭐였지?"
- "우리가 Python 프로젝트 설정에 대해 논의한 것 기억해?"
- "저번에 해결한 CORS 문제 방법이 뭐였더라?"

에이전트가 `session_search`를 자율적으로 호출해 `~/.hermes/state.db`를 SQLite FTS5로 쿼리합니다.

**session_search의 특징:**
- `~/.hermes/state.db`를 SQLite FTS5로 쿼리
- 세 가지 호출 패턴: discovery(세션 목록), scroll(세션 내 이동), browse(세션 간 검색)
- **raw 메시지 반환** — LLM 요약 없음, 절단 없음
- ~20ms FTS5 쿼리, ~1ms scroll (매우 빠름, LLM 비용 없음)
- 별도 설정 불필요, 항상 사용 가능

### FTS5 테이블 구성

`state.db`는 두 개의 FTS5 테이블을 포함합니다:

| 테이블 | 목적 |
|--------|------|
| `messages_fts` | 표준 FTS5 전문 검색 |
| `messages_fts_trigram` | 트라이그램 토크나이저 — **한국어(CJK)** 및 부분 문자열 검색 |

> **한국어 독자 참고:** `messages_fts_trigram` 테이블은 v10+에서 CJK/부분 문자열 검색을 위해 추가되었습니다. 이 덕분에 대화 이력의 한국어 콘텐츠가 완전히 검색 가능합니다 — 단어 경계에 의존하지 않는 트라이그램 방식으로 한국어 검색이 정확하게 동작합니다.

### 직접 DB 쿼리 (고급)

```bash
# 검증: SQLite 로컬 실행 필요, 2026-06-09
# 예시: 특정 키워드가 포함된 과거 대화 검색
sqlite3 ~/.hermes/state.db \
  "SELECT created_at, role, snippet(messages_fts, 2, '**', '**', '...', 20) 
   FROM messages_fts 
   WHERE messages_fts MATCH 'deploy'
   ORDER BY created_at DESC LIMIT 10"
```

> 검증 필요: 스키마는 공식 session-storage 문서 기준이나 로컬 state.db에서 실행 확인 후 게재 권장

---

## 개념: 메모리 설정과 압축 비용

### 메모리 설정

```yaml
# 검증: hermes rolling, 2026-06-09
memory:
  memory_enabled: true
  user_profile_enabled: true
  memory_char_limit: 2200     # MEMORY.md 최대 문자 수 (~800토큰)
  user_char_limit: 1375       # USER.md 최대 문자 수 (~500토큰)
```

### 압축과 메모리 오버헤드

압축은 메모리 파일(MEMORY.md, USER.md)을 직접 변경하지 않습니다. 하지만 **~1,300토큰의 메모리 오버헤드는 매 API 호출마다 포함**됩니다.

압축이 중간 대화 턴을 제거해도 메모리 스냅샷은 시스템 프롬프트에 계속 유지됩니다. 따라서 아무리 압축을 해도 API 호출당 비용이 **~1,300토큰 아래로 내려가지 않습니다**.

### 압축 비용 관리 설정

```yaml
# 검증: hermes rolling, 2026-06-09
compression:
  enabled: true
  threshold: 0.50      # 컨텍스트의 50% 사용 시 압축 트리거
  target_ratio: 0.20   # 압축 후 최근 20% 유지
  protect_last_n: 20   # 최근 20개 메시지 항상 보존 (기본값, 변경 가능)
  # protect_first_n: 3  ← 하드코딩됨, 설정 불가
```

게이트웨이 세션에서는 85% 사용 시 2차 압축기가 추가로 트리거됩니다 — 장시간 게이트웨이 세션의 API 오류를 방지하는 안전망입니다.

### 보조 압축 모델 설정

```yaml
# 검증: hermes rolling, 2026-06-09
auxiliary:
  compression:
    model: ""        # 빈 칸 = 메인 모델 사용 (저비용 모델 지정 가능)
    provider: "auto"
    base_url: null
```

---

## 실습: 메모리 확인·관리

### 코어 메모리 파일 직접 보기

```bash
# 검증: hermes rolling, 2026-06-09
# MEMORY.md 내용 확인
cat ~/.hermes/memories/MEMORY.md

# USER.md 내용 확인
cat ~/.hermes/memories/USER.md
```

이 두 명령은 항상 작동합니다 — hermes가 실행 중이 아니어도 파일을 직접 읽을 수 있습니다.

### 외부 메모리 프로바이더 관리

```bash
# 검증: hermes rolling, 2026-06-09
hermes memory setup      # 외부 메모리 프로바이더 설정 (Honcho, Mem0 등)
hermes memory status     # 현재 메모리 프로바이더 확인
hermes memory off        # 외부 프로바이더 비활성화 (내장 메모리만 사용)
```

### 세션 관리

```bash
# 검증: hermes rolling, 2026-06-09
hermes sessions list     # 최근 세션 목록
hermes sessions browse   # 인터랙티브 세션 탐색기
hermes sessions stats    # state.db 통계
hermes sessions prune    # 오래된 세션 삭제
```

### 세션 내 슬래시 커맨드

| 슬래시 커맨드 | 기능 |
|--------------|------|
| `/compress` | 수동으로 컨텍스트 압축 실행 |
| `/usage` | 현재 세션 토큰 사용량 표시 |

---

## 흔한 오류 / 주의

### 주의 1 — 메모리 스냅샷은 세션 시작 시 동결됨 (가장 흔한 혼란)

> 주의: 세션 중 에이전트가 MEMORY.md를 업데이트해도 현재 대화에는 반영되지 않습니다.
> 메모리 변경 사항은 다음 `hermes` 실행 시부터 시스템 프롬프트에 포함됩니다.
> 이것은 의도된 동작입니다 — 세션 내 컨텍스트 일관성을 보장합니다.

### 주의 2 — 게이트웨이 토큰 비용 급증 (A-9)

> 경고: Telegram/Discord 게이트웨이를 통한 2시간 사용으로
> Claude Sonnet 기준 약 400만 토큰(~$12)이 소비될 수 있습니다 (GitHub Issue 보고).
> 기본 압축 임계값(50%)이 이 비용을 유발합니다.
>
> 비용 절감 방법:
> 1. `compression.threshold: 0.30`으로 낮추기 (더 빨리 압축 트리거)
> 2. `/compress`로 수동 압축
> 3. 저비용 보조 모델 설정: `auxiliary.compression.model: "google/gemini-flash-1.5"`
> 4. `/usage`로 현재 토큰 사용량 모니터링

### 주의 3 — session_search는 슬래시 커맨드가 아님

> 참고: `session_search`는 에이전트가 자동으로 호출하는 내부 툴입니다.
> `/session_search` 같은 슬래시 커맨드는 없습니다.
> 과거 대화를 검색하려면 에이전트에게 자연어로 물어보세요:
> "지난달에 우리가 배포에 대해 얘기한 내용 기억해?"

### 주의 4 — 보조 모델 컨텍스트 윈도우 (A-14)

> 주의: `auxiliary.compression.model`에 메인 모델보다 작은
> 컨텍스트 윈도우의 모델을 지정하면 요약 중 중요 내용이
> 오류 없이 조용히 잘릴 수 있습니다.
> 저비용 요약 모델도 반드시 큰 컨텍스트 윈도우 모델을 선택하십시오
> (예: Gemini Flash — 1M 컨텍스트).

---

## 다음 단계

이전: [Ch.5 컨텍스트 파일](../05-context-files/index.md)

다음: [Ch.7 툴 게이트웨이](../07-tools/index.md)
