# Phase 6: Reference — Research

**Researched:** 2026-06-11
**Domain:** Hermes Agent — 레퍼런스 부록 (CLI·슬래시 커맨드 / config.yaml / 트러블슈팅 색인)
**Confidence:** HIGH (commands/keys assembled from prior-phase verified research) / MEDIUM (a small number of subcommands confirmed only indirectly) / LOW (any entry explicitly marked `> 검증 필요`)

**Accuracy constraint — CORE PRINCIPLE:** A reference page that lists commands or config keys MUST be correct. A wrong entry in a reference page is worse than a missing entry. This research presents a **verified core + 검증 필요 remainder** split, not a fabricated-exhaustive list. The planner and executor should present this framing explicitly in the published pages.

Convention (MANDATORY): every command/config code block opens with:
```
# 검증: hermes rolling, 2026-06-11
```

---

## Summary

Phase 6 adds a new "# 레퍼런스" section with three appendix chapters (19-cli-reference, 20-config-reference, 21-troubleshooting). The material is assembled **entirely from prior-phase verified research** (02-RESEARCH.md through 05-RESEARCH.md) plus a cross-check against the official CLI reference and configuration docs. No new speculative material is introduced here.

**What the planner gets from this document:**

1. **06-01 (19-cli-reference):** The full verifiable CLI command list with categories, plus the slash-command list — assembled from HIGH/MEDIUM confidence prior research. Commands not confirmable are individually marked `> 검증 필요`.
2. **06-02 (20-config-reference):** The config.yaml key reference grouped by area, with purpose, defaults, and allowed values — assembled from prior research. Defaults not confirmed are marked `> 검증 필요`.
3. **06-03 (21-troubleshooting):** The master troubleshooting index assembled by grepping the 17 existing chapter "흔한 오류 / 주의" sections (now fully captured below), plus a short glossary. Structure recommendation: error-message-searchable, linked back to source chapters.
4. **SUMMARY/file structure:** Exact `src/` folder names, SUMMARY.md entries, and plan/wave recommendation.
5. **Meta-accuracy risk note** at end of every section.

**Primary recommendation:** The reference pages must be accurate, not exhaustive. Scope each reference as "verified core" and label anything unverifiable as `> 검증 필요`. State that framing on the page itself (e.g., a `> 참고` callout at the top of each reference page).

---

## 06-01: CLI 커맨드 · 슬래시 커맨드 레퍼런스 (Chapter 19)

### Meta-accuracy risk statement (include on the page)

```
> 참고: 이 레퍼런스는 공식 문서와 튜토리얼 검증 과정에서 확인된 명령어만 수록합니다.
> Hermes는 롤링 릴리스 방식으로 배포되므로 명령어가 변경될 수 있습니다.
> `hermes --help` 및 각 서브커맨드의 `--help` 플래그로 현재 전체 목록을 확인하십시오.
> 아래 `> 검증 필요` 표시 항목은 로컬에서 추가 확인이 필요합니다.
```

### 공식 참조 소스

| 소스 | 용도 |
|------|------|
| `hermes --help` | 최신 전체 명령어 트리 (항상 실행 시점 정확) |
| `https://hermes-agent.nousresearch.com/docs/reference/cli-commands` | 공식 CLI 레퍼런스 (HIGH confidence) |

---

### A. 최상위 실행 명령어 (HIGH confidence — 02/03-RESEARCH.md)

```bash
# 검증: hermes rolling, 2026-06-11
hermes                    # 클래식 CLI 모드로 시작 (prompt_toolkit TUI)
hermes --tui              # 모던 TUI 모드로 시작 (모달 오버레이, 마우스 선택)
hermes --continue         # 가장 최근 세션 재개
hermes chat -q "질문"     # 비대화형 단일 질의 (파이프라인에 적합)
hermes --help             # 도움말 출력 (하위 명령어 목록 포함)
```

**주의:** `hermes help`(플래그 없음)는 최상위 커맨드로 문서화되지 않음. 반드시 `hermes --help` 또는 `hermes -h` 사용.

---

### B. 모델·설정 명령어 (HIGH confidence — 02/03/05-RESEARCH.md)

```bash
# 검증: hermes rolling, 2026-06-11
hermes model              # 공급자·모델 설정 마법사 (세션 외부에서 실행)
hermes setup              # 전체 설정 마법사
hermes setup --portal     # Nous Portal OAuth 원클릭 설정 (공급자 + Tool Gateway)
hermes setup terminal     # 터미널 백엔드 설정 마법사
hermes config show        # 현재 설정 전체 출력
hermes config set <key> <value>   # 설정 키 직접 변경 (.env 자동 라우팅 포함)
hermes config check       # 설정 파일 유효성 검사 (≠ hermes doctor)
hermes version            # 버전 정보 출력 (형식: X.Y.Z (YYYY.M.D) [git-hash])
```

> **검증 필요:** `hermes config check`의 정확한 동작은 `hermes config --help`로 확인 요망.

---

### C. 도구 명령어 (HIGH confidence — 03-RESEARCH.md)

```bash
# 검증: hermes rolling, 2026-06-11
hermes tools              # 플랫폼별 인터랙티브 툴 설정 UI (curses 기반)
hermes tools --summary    # 현재 활성화된 툴 요약 출력 후 종료
```

---

### D. 스킬 명령어 (HIGH confidence — 04-RESEARCH.md)

```bash
# 검증: hermes rolling, 2026-06-11
hermes skills list                    # 설치된 스킬 목록
hermes skills browse                  # 허브 스킬 탐색 (인터랙티브)
hermes skills search <query>          # 이름·태그로 검색
hermes skills inspect <identifier>    # 설치 전 미리보기
hermes skills install <identifier>    # 보안 스캔 후 설치
hermes skills install <url>           # URL로 직접 설치
hermes skills install <url> --name <name>   # URL 설치 + 이름 지정
hermes skills check                   # 업스트림 업데이트 확인
hermes skills update                  # 업데이트된 스킬 재설치
hermes skills uninstall <skill-name>  # 스킬 제거
```

---

### E. MCP 명령어 (HIGH confidence — 04-RESEARCH.md)

```bash
# 검증: hermes rolling, 2026-06-11
hermes mcp                          # 인터랙티브 MCP 카탈로그 피커
hermes mcp catalog                  # 카탈로그 텍스트 목록
hermes mcp install <name>           # 큐레이션 카탈로그에서 설치
hermes mcp add <name> --preset <type>  # 사전 설정된 전송 방식으로 추가
hermes mcp configure <name>         # 설치 후 툴 선택 업데이트
hermes mcp login <server>           # OAuth 원격 서버 인증
hermes mcp serve                    # Hermes를 stdio MCP 서버로 실행 (10개 메시징 툴 노출)
hermes mcp serve --verbose          # 디버그 로깅
```

> **검증 필요:** `hermes mcp serve`가 노출하는 정확한 툴 이름 목록은 로컬 실행 후 `--verbose`로 확인 필요.

---

### F. 크론 스케줄러 명령어 (HIGH confidence — 04-RESEARCH.md)

```bash
# 검증: hermes rolling, 2026-06-11
hermes cron list                              # 모든 작업 보기
hermes cron create "schedule" "prompt"        # 작업 추가 (예: "0 9 * * *", "every 2h")
hermes cron edit <job_id> --schedule "..."    # 스케줄 수정
hermes cron pause <job_id_or_name>            # 일시 중지
hermes cron resume <job_id_or_name>           # 재개
hermes cron run <job_id_or_name>              # 즉시 수동 트리거
hermes cron remove <job_id_or_name>           # 삭제
hermes cron status                            # 전체 상태
hermes cron tick                              # 수동 tick (디버그용)
```

**스케줄 형식 요약:**

| 형식 | 예시 | 동작 |
|------|------|------|
| 상대 지연 | `30m`, `2h`, `1d` | 1회 실행 |
| 인터벌 | `every 30m`, `every 2h` | 영구 반복 |
| Cron 표현식 | `0 9 * * *` | 영구 반복 |
| ISO 타임스탬프 | `2026-03-15T09:00:00` | 1회 지정 시간 |

---

### G. 게이트웨이 명령어 (HIGH confidence — 04/05-RESEARCH.md)

```bash
# 검증: hermes rolling, 2026-06-11
hermes gateway            # 포그라운드 실행 (Ctrl-C로 중지, 개발/테스트용)
hermes gateway run        # Docker 컨테이너 내 실행 시 메인 프로세스로 사용
hermes gateway install    # 사용자 서비스로 등록 (Linux: ~/.config/systemd/user/, macOS: launchd)
sudo hermes gateway install --system  # 시스템 서비스로 설치 (/etc/systemd/system/)
hermes gateway start      # 등록된 서비스 시작
hermes gateway stop       # 서비스 중지
hermes gateway restart    # 서비스 재시작
hermes gateway status     # 서비스 상태 확인
hermes gateway setup      # 인터랙티브 플랫폼 설정 마법사
```

> **검증 필요:** `hermes gateway install --system`의 정확한 서비스 파일 경로는 Ubuntu VPS 로컬 확인 권장 (GitHub Issue #16264에서 언급).

---

### H. 운영 명령어 (HIGH confidence — 03/04/05-RESEARCH.md)

```bash
# 검증: hermes rolling, 2026-06-11
hermes doctor                          # 공급망 어드바이저리 검사 (Python CVE — NOT 설정 검증)
hermes doctor --ack <advisory-id>      # 어드바이저리 확인 처리 (영구 무시)
hermes status                          # 에이전트/인증/플랫폼 상태 요약
hermes version                         # 버전 정보 출력
hermes sessions list                   # 세션 목록
hermes sessions browse                 # 인터랙티브 세션 탐색기
hermes sessions stats                  # state.db 통계
hermes sessions prune                  # 오래된 세션 삭제
hermes memory setup                    # 외부 메모리 프로바이더 설정 (Honcho, Mem0 등)
hermes memory status                   # 현재 메모리 프로바이더 확인
hermes memory off                      # 외부 프로바이더 비활성화
hermes curator status                  # 큐레이터 마지막 실행·스킬 카운트 확인
hermes curator run                     # 큐레이터 즉시 실행
hermes curator run --background        # 백그라운드 실행
hermes curator run --dry-run           # 변경 없이 미리보기
hermes curator rollback                # 마지막 실행 전 상태로 복원
hermes dashboard                       # 브라우저 대시보드 열기 (http://127.0.0.1:9119)
hermes pairing list                    # 승인 대기 목록
hermes pairing approve <platform> <code>  # 페어링 코드 승인
hermes pairing revoke <platform> <user>   # 사용자 접근 취소
hermes portal info                     # Nous Portal 로그인 상태 + 구독 + Tool Gateway 요약
hermes portal tools                    # Tool Gateway 카탈로그
hermes kanban init                     # Kanban 초기화 (첫 kanban 명령 시 자동)
hermes kanban create "제목" --assignee <profile>  # 태스크 생성
hermes kanban show <id>                # 태스크 상세
hermes kanban complete <id>            # 완료 처리
hermes kanban block <id>               # 차단 표시
hermes kanban unblock <id>             # 차단 해제
hermes kanban decompose <id>           # 서브태스크로 분해
hermes kanban runs <id>                # 실행 이력
hermes kanban watch --kinds completed,gave_up,timed_out  # 이벤트 모니터링
```

> **검증 필요:** `hermes logs` 서브커맨드 플래그 (`--level WARNING`, `--since 1h`)는 05-RESEARCH.md에서 운영 런북에 포함되었으나 공식 CLI 레퍼런스에서 직접 확인 필요. 로컬에서 `hermes logs --help`로 확인하십시오.

> **중요 수정 사항 (05-RESEARCH.md):** `hermes security audit`은 **존재하지 않습니다.** 보안 관련 작업은 다음으로 나뉩니다:
> - `hermes doctor` → 공급망 어드바이저리
> - `hermes pairing` → 인증된 사용자 관리
> - `hermes config check` → 설정 검증
> - Tirith 스캔은 모든 명령 실행 시 자동으로 수행됨

---

### I. Nous Portal / 인증 명령어

```bash
# 검증: hermes rolling, 2026-06-11
hermes setup --portal     # OAuth 로그인 + Nous 공급자 + Tool Gateway (원클릭)
hermes portal info        # 로그인 상태 + 구독 확인
hermes portal tools       # Tool Gateway 카탈로그
```

> **검증 필요:** `hermes auth add` (03-RESEARCH.md에서 모델 설정 관련으로 언급됨)가 별도 최상위 명령어로 존재하는지, 아니면 `hermes model` 마법사 내 서브명령인지 확인 필요.

---

### J. Slack 특수 명령어 (HIGH confidence — 05-RESEARCH.md)

```bash
# 검증: hermes rolling, 2026-06-11
hermes slack manifest --write   # ~/.hermes/slack-manifest.json 생성
hermes whatsapp                 # WhatsApp 설정 마법사 (QR 코드 출력)
```

---

### K. 세션 내 슬래시 커맨드 (HIGH confidence — 02/03/04-RESEARCH.md)

슬래시 커맨드는 대소문자 비구분 (`/HELP` = `/help`).

| 커맨드 | 기능 | 출처 |
|--------|------|------|
| `/help` | 사용 가능한 커맨드 목록 | HIGH — 02-RESEARCH.md |
| `/model` | 현재 모델 표시 / 이미 설정된 공급자 간 전환 | HIGH — 02-RESEARCH.md |
| `/personality [name]` | 세션 성격 변경 (14개 내장 + 커스텀) | HIGH — 03-RESEARCH.md |
| `/tools` | 현재 세션 활성 툴 목록 | HIGH — 03-RESEARCH.md |
| `/tools list` | 현재 세션 활성 툴 목록 | HIGH — 03-RESEARCH.md |
| `/tools disable <toolset>` | 툴셋 비활성화 | HIGH — 03-RESEARCH.md |
| `/tools enable <toolset>` | 툴셋 활성화 | HIGH — 03-RESEARCH.md |
| `/skills browse` | 스킬 허브 탐색 | HIGH — 04-RESEARCH.md |
| `/skills install <identifier>` | 스킬 설치 | HIGH — 04-RESEARCH.md |
| `/compress` | 수동 컨텍스트 압축 | HIGH — 03-RESEARCH.md |
| `/usage` | 현재 세션 토큰 사용량 표시 | HIGH — 03-RESEARCH.md |
| `/rollback` | 마지막 체크포인트로 되돌리기 | HIGH — 02-RESEARCH.md |
| `/voice on` | 음성 모드 활성화 | HIGH — 02-RESEARCH.md |
| `/reasoning high` | 추론 수준 조정 (none/minimal/low/medium/high/xhigh) | HIGH — 02-RESEARCH.md |
| `/title` | 세션 이름 지정 | HIGH — 02-RESEARCH.md |
| `/status` | 세션 정보 요약 | HIGH — 02-RESEARCH.md |
| `/stop` | 활성 에이전트 스레드 중단 | HIGH — 03-RESEARCH.md |
| `/reload-mcp` | MCP 서버 재로드 (재시작 없이) | HIGH — 04-RESEARCH.md |
| `/<skill-name> [task]` | 스킬 직접 실행 | HIGH — 04-RESEARCH.md |
| `/<plugin>:<skill-name>` | 플러그인 네임스페이스 스킬 실행 | HIGH — 04-RESEARCH.md |

**내장 퍼소날리티 목록 (14개 — HIGH confidence, 03-RESEARCH.md):**
`helpful` / `concise` / `technical` / `creative` / `teacher` / `kawaii` / `catgirl` / `pirate` / `shakespeare` / `surfer` / `noir` / `uwu` / `philosopher` / `hype`

---

### L. 키 바인딩 (HIGH confidence — 02-RESEARCH.md)

| 키 | 동작 |
|----|------|
| `Enter` | 메시지 전송 |
| `Alt+Enter` / `Ctrl+J` / `Shift+Enter` | 줄바꿈 (다음 줄) |
| `Ctrl+G` | 외부 에디터에서 입력 편집 |
| `Ctrl+C` | 에이전트 중단 (두 번 연속 → 강제 종료) |
| `Ctrl+D` | 종료 |

---

### 06-01 Verified vs 검증 필요 요약

| 항목 | 신뢰도 | 비고 |
|------|--------|------|
| 최상위 실행 명령어 (A) | HIGH | 공식 CLI 레퍼런스 + 02-RESEARCH.md |
| 모델·설정 명령어 (B) | HIGH, `config check` MEDIUM | 공식 docs |
| 도구 명령어 (C) | HIGH | |
| 스킬 명령어 (D) | HIGH | 04-RESEARCH.md |
| MCP 명령어 (E) | HIGH (`mcp serve` 노출 툴 MEDIUM) | 04-RESEARCH.md |
| 크론 명령어 (F) | HIGH | 04-RESEARCH.md |
| 게이트웨이 명령어 (G) | HIGH (`--system` flag MEDIUM) | 05-RESEARCH.md |
| 운영 명령어 (H) | HIGH, `hermes logs` flags MEDIUM | 03/04/05-RESEARCH.md |
| 슬래시 커맨드 (K) | HIGH | 02/03/04-RESEARCH.md |

---

## 06-02: config.yaml 레퍼런스 (Chapter 20)

### Meta-accuracy risk statement (include on the page)

```
> 참고: 이 레퍼런스는 공식 문서와 튜토리얼 검증 과정에서 확인된 키만 수록합니다.
> Hermes는 롤링 릴리스 방식으로 배포되므로 키 이름과 기본값이 변경될 수 있습니다.
> `hermes config show`로 현재 실제 설정값을 항상 확인하십시오.
> `> 검증 필요` 표시 항목은 로컬에서 추가 확인이 필요합니다.
```

### .env vs config.yaml 분리 원칙 (HIGH confidence — 03-RESEARCH.md)

```
비밀 (API 키, 봇 토큰)  → ~/.hermes/.env
비밀 외 설정            → ~/.hermes/config.yaml
OAuth 자격증명          → ~/.hermes/auth.json
```

**해결 우선순위 (높을수록 우선):**
1. CLI 인수 (`--model`, `--backend`)
2. `~/.hermes/config.yaml`
3. `~/.hermes/.env`
4. 내장 기본값

**환경변수 치환:** config.yaml 내에서 `${VAR_NAME}` 문법 사용 (중괄호 필수; `$VAR_NAME`은 치환 안 됨).

**공식 예시 파일:** `https://github.com/NousResearch/hermes-agent/blob/main/cli-config.yaml.example`

---

### A. display (표시 설정) — HIGH confidence

```yaml
# 검증: hermes rolling, 2026-06-11
display:
  language: ko            # 지원 값: en|zh|zh-hant|ja|de|es|fr|tr|uk|af|ko|it|ga|pt|ru|hu
                          # 주의: 승인 프롬프트 등 일부 정적 메시지만 번역; 에이전트 응답은 영어 유지
  platforms:
    telegram:
      tool_progress: verbose   # off | brief | verbose
    discord:
      tool_progress: verbose   # off | brief | verbose
```

---

### B. model (모델 설정) — HIGH confidence

```yaml
# 검증: hermes rolling, 2026-06-11
model:
  default: nous/hermes-3-405b   # provider/model-id 형식
  provider: nous                 # nous|openrouter|anthropic|openai|custom|auto
  context_length: 131072         # 최소 64000 필수
  reasoning_effort: medium       # none|minimal|low|medium|high|xhigh
```

---

### C. agent (에이전트 동작) — HIGH confidence

```yaml
# 검증: hermes rolling, 2026-06-11
agent:
  max_turns: 90               # 루프 최대 반복 횟수 (기본값: 90)
  personalities:              # 커스텀 퍼소날리티 정의
    codereviewer: >
      You are a meticulous code reviewer. ...
```

> **검증 필요:** `agent.disabled_toolsets`는 **존재하지 않습니다** (03-RESEARCH.md 확인). 툴셋 비활성화는 `toolsets:` 배열 또는 `hermes tools` UI로 설정.

---

### D. memory (메모리 설정) — HIGH confidence

```yaml
# 검증: hermes rolling, 2026-06-11
memory:
  memory_enabled: true          # MEMORY.md 활성화 (기본: true)
  user_profile_enabled: true    # USER.md 활성화 (기본: true)
  memory_char_limit: 2200       # MEMORY.md 최대 문자 수 (~800 토큰)
  user_char_limit: 1375         # USER.md 최대 문자 수 (~500 토큰)
  provider: honcho              # 외부 메모리 프로바이더 (선택; 기본: 내장)
```

**메모리 파일 경로:**
- `~/.hermes/memories/MEMORY.md` — 에이전트 메모 (2,200자 한도)
- `~/.hermes/memories/USER.md` — 사용자 프로파일 (1,375자 한도)
- `~/.hermes/SOUL.md` — 에이전트 정체성 (전역, 프로젝트와 무관)

---

### E. compression (컨텍스트 압축) — HIGH confidence

```yaml
# 검증: hermes rolling, 2026-06-11
compression:
  enabled: true
  threshold: 0.50       # 컨텍스트의 50% 사용 시 압축 트리거 (기본값)
  target_ratio: 0.20    # 압축 후 최근 20% 유지 (기본값)
  protect_last_n: 20    # 최근 20개 메시지 항상 보존 (기본값, 변경 가능)
  # protect_first_n: 3  ← 하드코딩됨 — 설정 불가
```

**보조 압축 모델:**

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

---

### F. toolsets / custom_toolsets (툴셋 설정) — HIGH confidence

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

> **검증 필요:** 기본 CLI 플랫폼의 `toolsets:` 키의 정확한 기본값 이름(`hermes-cli`인지 다른 이름인지)은 `hermes config show`로 확인 필요.

---

### G. approvals (승인 모드) — HIGH confidence

```yaml
# 검증: hermes rolling, 2026-06-11
approvals:
  mode: manual             # manual (기본, 가장 안전) | smart | off
  timeout: 300             # 승인 대기 시간 (초) — 03-RESEARCH.md: 60, 05-RESEARCH.md: 300
  cron_mode: manual        # 헤드리스 cron: manual | auto (03-RESEARCH.md: deny|approve)
  destructive_slash_confirm: true  # /clear, /new, /reset 전 확인
```

| 모드 | 동작 |
|------|------|
| `manual` | 위험한 명령 전 항상 사용자 승인 요청 (기본, 프로덕션 권장) |
| `smart` | 보조 LLM이 위험도 평가; 저위험 자동 승인, 고위험 자동 거부 |
| `off` | 모든 승인 체크 비활성화 (`HERMES_YOLO_MODE=true`와 동일) |

> **검증 필요:** `timeout` 기본값이 60초인지 300초인지, `cron_mode` 값이 `deny|approve`인지 `manual|auto`인지 `hermes config show`로 확인 필요. 두 연구 파일에서 값이 다르게 기록됨 — 로컬 확인 우선.

---

### H. security (보안 설정) — HIGH confidence

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

**주의:** `security.redact_secrets`는 일반 PII 필터가 아님. MCP 오류 메시지에서 `ghp_...`, `sk-...`, Bearer 토큰 패턴만 난독화.

**Hardline blocklist:** `off` 모드에서도 `rm -rf /`, 포크 폭탄, `mkfs.*` on mounted root 등은 항상 차단됨.

---

### I. terminal (터미널 백엔드) — HIGH/MEDIUM confidence

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

**환경변수 (`.env`에 저장):**

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

### J. delegation (서브에이전트) — HIGH confidence

```yaml
# 검증: hermes rolling, 2026-06-11
delegation:
  max_spawn_depth: 1           # 위임 중첩 허용 레벨 (기본: 1 = 평면)
  max_concurrent_children: 3  # 동시 서브에이전트 수 (기본: 3)
```

---

### K. MCP 서버 설정 — HIGH confidence

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

---

### L. Skills 설정 — HIGH confidence

```yaml
# 검증: hermes rolling, 2026-06-11
skills:
  external_dirs:
    - "/path/to/additional/skills"
```

---

### M. Cron 설정 — HIGH confidence

```yaml
# 검증: hermes rolling, 2026-06-11
cron:
  wrap_response: false              # 래퍼 헤더/푸터 억제
  script_timeout_seconds: 300       # 사전 실행 스크립트 시간 제한
```

---

### N. Curator 설정 — HIGH confidence

```yaml
# 검증: hermes rolling, 2026-06-11
curator:
  enabled: true
  interval_hours: 168        # 7일 (기본값)
  stale_after_days: 30       # 30일 미사용 → stale
  archive_after_days: 90     # 90일 미사용 → archived
```

---

### O. Gateway 보안 관련 환경변수 — HIGH confidence

| 변수명 | 설명 |
|--------|------|
| `GATEWAY_ALLOW_ALL_USERS=true` | 전체 허용 — **절대 사용 금지** |
| `GATEWAY_ALLOWED_USERS` | 글로벌 허용 목록 |

**플랫폼별 환경변수 (`.env`):**

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

### P. config.yaml 미존재 키 (중요 — 혼동 방지)

| 키 | 상태 |
|----|------|
| `agent.disabled_toolsets` | **존재하지 않음** — 조용히 무시됨 (03-RESEARCH.md) |
| `memory.provider` + `honcho` 세부 키들 | `memory.provider: honcho` 존재; Honcho 세부 설정은 `~/.honcho/config.json` |
| `security.pii_redact` 또는 유사 키 | **존재하지 않음** — PII 필터는 별개 기능 없음 (05-RESEARCH.md) |

---

### 06-02 Verified vs 검증 필요 요약

| 키 그룹 | 신뢰도 | 비고 |
|---------|--------|------|
| display | HIGH | |
| model | HIGH | |
| agent.max_turns, agent.personalities | HIGH | `agent.disabled_toolsets` 없음 확인 |
| memory | HIGH | 파일 경로 `/memories/` 서브디렉터리 확인 |
| compression | HIGH | `protect_first_n` 하드코딩 확인 |
| toolsets, custom_toolsets | HIGH, 기본값 이름 MEDIUM | |
| approvals | HIGH, timeout/cron_mode 값 MEDIUM | 두 연구에서 불일치 → 로컬 확인 |
| security | HIGH | `redact_secrets` = credential only 확인 |
| terminal (docker 키) | HIGH | |
| terminal (modal_image) | MEDIUM | 로컬 확인 권장 |
| terminal (ssh 키) | HIGH | |
| delegation | HIGH | |
| mcp_servers | HIGH | |
| skills.external_dirs | HIGH | |
| cron, curator | HIGH | |
| .env 변수 목록 | HIGH | .env.example로 확인 |

---

## 06-03: 마스터 트러블슈팅 색인 (Chapter 21)

### 집약 방법론

이 색인은 `src/*/index.md`의 모든 "흔한 오류 / 주의" 섹션을 집약한 것이다. 플래너와 실행자는 아래 집약된 데이터를 그대로 사용하면 된다 — 이미 모든 챕터 파일에서 추출 완료.

**색인 구조 권장:** 오류 메시지 또는 증상으로 검색 가능하게 설계. 각 항목에 출처 챕터 링크 포함.

---

### 집약된 오류·주의 항목 (원문 그대로)

아래는 실제 챕터 파일에서 추출한 항목들이다. Ch.0은 "흔한 오류" 섹션 없음 (첫 명령어가 없는 소개 챕터).

---

#### Ch.1 설치 (src/01-install/index.md)

| # | 오류/증상 | 원인 | 해결책 |
|---|---------|------|--------|
| 1 | `hermes: command not found` | PATH 미반영 | `source ~/.bashrc` 또는 `~/.zshrc` 실행; `which hermes`로 확인 |
| 2 | sudo 설치로 인한 권한 문제 | `sudo curl|bash` 사용 | `sudo rm /usr/local/bin/hermes` 후 표준 방법으로 재설치 |
| 3 | Windows 네이티브 제한 | PowerShell 설치 | POSIX PTY 기능·브라우저 대시보드 제한됨; WSL2 권장 |
| 4 | Termux 설치 실패 | 음성 의존성 오류 | 공식 문서 Termux 별도 안내 참조 |

---

#### Ch.2 첫 실행 (src/02-first-run/index.md)

| # | 오류/증상 | 원인 | 해결책 |
|---|---------|------|--------|
| 1 | `"No provider configured"` 오류 | 공급자 미설정 | `Ctrl+D`로 종료 후 Ch.3 모델 설정 완료 |

---

#### Ch.3 모델 설정 (src/03-model/index.md)

| # | 오류/증상 | 원인 | 해결책 |
|---|---------|------|--------|
| 1 | Claude 사용 불가 (구독 있음) | Claude Pro/Max 구독 ≠ API 접근 | console.anthropic.com에서 별도 API 키 발급; 또는 OpenRouter로 접근 |
| 2 | 녹색 체크마크 후에도 인증 실패 | Silent API Key Skip (Issue #16394) | `~/.hermes/.env`에서 잘못된 키 줄 직접 삭제 후 재설정 |
| 3 | API 키 git 노출 위험 | config.yaml에 키 저장 | API 키는 반드시 `.env`에 저장; `hermes model` 마법사 사용 |
| 4 | 모델 시작 거부 | 64K 미만 컨텍스트 윈도우 | 64,000 토큰 이상 컨텍스트 윈도우 모델 선택 |
| 5 | 채팅 중 공급자 추가 불가 | `/model`은 전환만 지원 | 세션 종료(`Ctrl+D`) 후 `hermes model` 실행 |

---

#### Ch.4 에이전트 루프 (src/04-agent-loop/index.md)

| # | 오류/증상 | 원인 | 해결책 |
|---|---------|------|--------|
| 1 | 응답이 오래 걸리는 것처럼 느껴짐 | 동기 엔진 특성 | 도구 실행 중임; 정상 동작 |
| 2 | 에이전트가 작업 요약 후 중단 | `agent.max_turns` (기본 90) 초과 | `max_turns` 늘리기 또는 `delegate_task` 서브에이전트 활용 |

---

#### Ch.5 컨텍스트 파일 (src/05-context-files/index.md)

| # | 오류/증상 | 원인 | 해결책 |
|---|---------|------|--------|
| 1 | `.hermes.md` 수정이 현재 세션에 반영 안 됨 | 세션 시작 시 한 번만 로드 | `hermes` 재시작 후 반영됨 |
| 2 | `.hermes.md`와 `AGENTS.md` 중 하나만 로드됨 | 우선순위: 첫 매치만 로드 | 두 파일 내용을 `.hermes.md`로 병합 |
| 3 | API 키 git 노출 | config.yaml에 저장 | `.env` 파일에 저장 (참조만 `${VAR}`) |
| 4 | `${VAR_NAME}` 치환 안 됨 | `$VAR_NAME`(중괄호 없음) 사용 | `${VAR_NAME}` 형식 사용 필수 |

---

#### Ch.6 메모리 (src/06-memory/index.md)

| # | 오류/증상 | 원인 | 해결책 |
|---|---------|------|--------|
| 1 | 세션 중 저장한 메모리가 현재 대화에 반영 안 됨 | 세션 시작 시 스냅샷 동결 | 의도된 동작; 다음 세션부터 반영 |
| 2 | 게이트웨이 2시간 사용으로 ~$12 비용 발생 | 기본 압축 임계값 50% | `compression.threshold: 0.30` 설정; `/compress` 수동 압축; 저비용 보조 모델 지정; `/usage` 모니터링 |
| 3 | `/session_search` 커맨드 없음 | 에이전트 내부 툴, 슬래시 커맨드 아님 | 자연어로 질문: "지난달에 배포에 대해 얘기한 내용 기억해?" |
| 4 | 압축 중 내용이 조용히 잘림 | 보조 모델의 작은 컨텍스트 윈도우 | 보조 모델도 큰 컨텍스트 윈도우 선택 (예: Gemini Flash 1M) |

---

#### Ch.7 툴 게이트웨이 (src/07-tools/index.md)

| # | 오류/증상 | 원인 | 해결책 |
|---|---------|------|--------|
| 1 | `approvals.mode: off`로 인한 위험 | YOLO 모드 활성화 | 로컬 백엔드에서 절대 사용 금지; 컨테이너 백엔드에서만 고려 |
| 2 | Tool Gateway 도구 호출 시 `subscription required` | 무료 계정 한도 초과 | `hermes portal info`로 구독 상태 확인; 유료 구독 또는 직접 API 키 사용 |
| 3 | `agent.disabled_toolsets` 설정이 동작하지 않음 | 존재하지 않는 키 | `toolsets:` 배열 또는 `hermes tools` UI 사용 |

---

#### Ch.8 터미널 백엔드 (src/08-backends/index.md)

| # | 오류/증상 | 원인 | 해결책 |
|---|---------|------|--------|
| 1 | `"Cannot connect to Docker daemon"` | Docker 데몬 미실행 | `docker info`로 확인 후 Docker 시작 |
| 2 | 컨테이너 내 생성 파일이 root 소유 | `docker_run_as_host_user: false` (기본값) | `docker_run_as_host_user: true` 설정 (단, apt install 권한 오류 발생 가능) |
| 3 | Modal 세션 간 파일 소실 | 에페머럴 특성 | `container_persistent: true` 설정 |
| 4 | `approvals.mode: off`가 로컬에서 위험 | 로컬 백엔드 사용 | 컨테이너 백엔드 사용 시에만 `off` 고려 |

---

#### Ch.9 스킬 시스템 (src/09-skills/index.md)

| # | 오류/증상 | 원인 | 해결책 |
|---|---------|------|--------|
| 1 | 스킬 파일 추가 후 현재 세션에 반영 안 됨 | 스킬 인덱스 세션 시작 시 로드 | `hermes` 재시작 |
| 2 | 번들 스킬 수가 예상과 다름 | 버전마다 다름 (v0.10.0: 118개, v0.15.x: ~89개) | `hermes skills list`로 현재 설치 수 확인 |
| 3 | 스킬 설치 보안 경고 | 보안 스캔 감지 | `--force` 플래그로 강제 설치 가능하나 비위험 항목에만 사용 |

---

#### Ch.10 학습 루프 (src/10-learning-loop/index.md)

| # | 오류/증상 | 원인 | 해결책 |
|---|---------|------|--------|
| 1 | 스킬 저장 제안을 놓침 | 에이전트 메시지 미확인 | 작업 완료 후 에이전트 메시지 주의 깊게 확인 |
| 2 | Curator가 즉시 실행되지 않음 | 비활동 기반 트리거 (2h 경과 + 7일 간격) | `hermes curator run`으로 즉시 수동 실행 |
| 3 | Honcho 설정 후 연결 실패 | 외부 계정 필요 | honcho.dev에서 계정 생성 + `HONCHO_API_KEY` 설정 |

---

#### Ch.11 MCP 연동 (src/11-mcp/index.md)

| # | 오류/증상 | 원인 | 해결책 |
|---|---------|------|--------|
| 1 | `hermes mcp serve`가 전체 에이전트를 노출하지 않음 | 10개 메시징 툴만 노출 | 에디터 임베딩은 `hermes acp` 사용 |
| 2 | ACP와 MCP 혼동 | 별도 통합 | ACP = 에디터 네이티브 에이전트; MCP = 툴 서버 연결 |
| 3 | `tools.include`와 `tools.exclude` 동시 설정 시 `exclude` 무시 | `include` 우선순위 | `include`만 사용하는 화이트리스트 방식 권장 |
| 4 | 신뢰할 수 없는 MCP 서버 연결 후 보안 위협 | 툴 표면 확장 | `tools.include` 화이트리스트로 필요한 툴만 허용 |

---

#### Ch.12 크론 스케줄러 (src/12-cron/index.md)

| # | 오류/증상 | 원인 | 해결책 |
|---|---------|------|--------|
| 1 | 크론 작업이 컨텍스트를 모름 | 새로운 격리 세션으로 실행 | 프롬프트에 모든 컨텍스트 명시 (URL, 설정, 기호) |
| 2 | 크론 작업의 파일 경로가 예상과 다름 | `--workdir` 미지정 시 설치 디렉터리 사용 | 파일 I/O 작업은 `--workdir /절대/경로` 필수 지정; workdir 지정 시 순차 실행됨 |
| 3 | 새 Homebrew/nvm 도구가 launchd에서 인식 안 됨 (macOS) | launchd PATH 고정 | 새 도구 설치 후 `hermes gateway install` 재실행 |
| 4 | 스케줄 자동 실행 안 됨 | gateway 프로세스 미실행 | `hermes gateway` 또는 `hermes gateway install && hermes gateway start` |

---

#### Ch.13 서브에이전트 (src/13-subagents/index.md)

| # | 오류/증상 | 원인 | 해결책 |
|---|---------|------|--------|
| 1 | 서브에이전트가 부모 대화 내용을 모름 | 완전 격리 새 세션 | `goal`과 `context`에 모든 필요 정보 명시 |
| 2 | 서브에이전트 비용 급증 | 각 서브에이전트 = 독립 LLM 세션 (~14K 토큰 오버헤드) | 단순 작업은 직접 툴 사용; 서브에이전트는 추론 집약적 작업에만 |
| 3 | 서브에이전트가 추가 서브에이전트 생성 불가 | `max_spawn_depth: 1` (기본) | `delegation.max_spawn_depth: 2` 이상으로 설정 (지수적 비용 주의) |
| 4 | Kanban과 `delegate_task` 혼동 | 서로 다른 용도 | 짧은 추론 → `delegate_task`; 장기·재개·인간참여 → Kanban |
| 5 | 일반 채팅에서 Kanban 툴 없음 | 기본 비활성 | `hermes gateway start`(디스패처) 또는 `config.yaml`에 `toolsets: [..., kanban]` 명시 |

---

#### Ch.14 메시징 게이트웨이: Telegram (src/14-gateways/index.md)

| # | 오류/증상 | 원인 | 해결책 |
|---|---------|------|--------|
| 1 | 메시지가 간헐적으로 처리되거나 무시됨 | 동일 토큰 중복 폴링 (두 프로세스) | `hermes gateway status`로 인스턴스 확인; 토큰은 프로파일 간 공유 불가 |
| 2 | 봇이 모든 사용자 명령 실행 | `GATEWAY_ALLOW_ALL_USERS=true` | 플랫폼별 허용 목록 또는 `hermes pairing approve` 사용 |
| 3 | Telegram 토큰 노출 | config.yaml에 저장 | `.env`에만 저장; 노출 시 BotFather `/revoke`로 재발급 |
| 4 | 2시간 사용으로 ~$12 비용 | 기본 압축 임계값 | `compression.threshold: 0.30` + `/usage` 모니터링 |

---

#### Ch.15 추가 게이트웨이 (src/15-more-gateways/index.md)

| # | 오류/증상 | 원인 | 해결책 |
|---|---------|------|--------|
| 1 | Discord 봇이 온라인인데 메시지를 완전히 무시 | **Message Content Intent 비활성** (최빈 원인) | Discord Developer Portal → Bot → Privileged Gateway Intents → Message Content Intent 활성화 → Save → 게이트웨이 재시작 |
| 2 | Slack 봇이 채널 메시지를 못 받음 | `channels:history`/`groups:history` 스코프 누락 | Slack App 설정에서 두 스코프 추가 후 앱 재설치 |
| 3 | WhatsApp 세션 단절/기능 제한 | 비공식 Web 프로토콜 | WhatsApp 정책 변경에 따른 것; 전용 번호 사용 권장 |

---

#### Ch.17 프로덕션 배포 (src/17-deploy/index.md)

| # | 오류/증상 | 원인 | 해결책 |
|---|---------|------|--------|
| 1 | API 키·토큰이 손상됨 (VPS 초기 설정) | 웹 콘솔 특수 문자 오전송 | 반드시 SSH로 접속하여 설정 |
| 2 | 세션 파일·메모리 저장소 손상 | 두 프로세스가 동일 `~/.hermes` 공유 | 단일 컨테이너의 `hermes profile create` 사용 |
| 3 | Docker 컨테이너 파일 소유권 문제 | root 소유 파일 | 공식 이미지는 UID 10000 사용; `docker_run_as_host_user` 확인 |

---

#### Ch.18 보안 하드닝 (src/18-security/index.md)

| # | 오류/증상 | 원인 | 해결책 |
|---|---------|------|--------|
| 1 | `hermes doctor`로 설정 오류를 진단하려 함 | `hermes doctor` = 공급망 검사기 (설정 검증 아님) | 설정 검증은 `hermes config check` 사용 |
| 2 | `hermes security audit` 실행 오류 | 존재하지 않는 명령어 | `hermes doctor`(공급망), `hermes pairing`(인증), `hermes config check`(설정) 사용 |
| 3 | "PII 리댁션"으로 이름·이메일 필터 기대 | 일반 PII 필터 없음 | `security.redact_secrets`는 MCP 오류의 API 키 패턴만 처리 |
| 4 | 봇이 모든 인터넷 사용자 명령 실행 | `GATEWAY_ALLOW_ALL_USERS=true` | 플랫폼별 허용 목록 또는 DM 페어링 사용 |

---

### 색인 구성 권장 사항

#### 권장 섹션 구조

```markdown
# Ch.21 마스터 트러블슈팅 색인

> 빠른 검색: 오류 메시지나 증상의 핵심 단어로 Ctrl+F 검색하거나,
> 아래 카테고리별로 찾아보십시오.

## 설치 / 첫 실행 오류 (Ch.1–3)
## 에이전트 동작 이해 (Ch.4–6)
## 도구 · 백엔드 · MCP (Ch.7–8, Ch.11)
## 스킬 · 학습 루프 · 자동화 (Ch.9–10, Ch.12)
## 서브에이전트 · Kanban (Ch.13)
## 게이트웨이 · 배포 · 보안 (Ch.14–15, Ch.17–18)
## 용어 사전 (Glossary)
```

#### 각 항목 형식

```markdown
### "hermes: command not found"

**증상:** `hermes` 명령 실행 시 "command not found" 오류
**원인:** PATH에 Hermes 바이너리 경로 미등록
**해결:** `source ~/.bashrc` (또는 `~/.zshrc`) 실행; 또는 `export PATH="$HOME/.local/bin:$PATH"`
**상세:** [Ch.1 설치](../01-install/index.md#흔한-오류--주의)
```

#### 용어 사전 (Glossary) 권장 항목

| 용어 | 정의 |
|------|------|
| SOUL.md | 전역 에이전트 정체성 파일 (`~/.hermes/SOUL.md`); 프롬프트 레이어 1 |
| MEMORY.md | 에이전트 메모 파일 (`~/.hermes/memories/MEMORY.md`); 세션 시작 시 동결 로드 |
| USER.md | 사용자 프로파일 (`~/.hermes/memories/USER.md`); 세션 시작 시 동결 로드 |
| .hermes.md | 프로젝트 컨텍스트 파일; git root까지 상위 탐색 |
| 스킬 (Skill) | 재사용 가능한 절차를 기술한 Markdown 파일; `~/.hermes/skills/` |
| Curator | 백그라운드 스킬 품질 검토 프로세스; 비활동 기반 트리거 (기본 7일) |
| Tool Gateway | Nous Portal을 통해 라우팅되는 도구 서비스 (웹검색, 이미지, TTS, 브라우저) |
| delegate_task | 서브에이전트 스폰 도구; 격리 자식 에이전트로 병렬 추론 |
| Kanban | SQLite 기반 다중 에이전트 태스크 큐 시스템 (`~/.hermes/kanban.db`) |
| Tirith | 명령 실행 전 보안 패턴 스캔 (항상 활성); 구성 가능 |
| approvals.mode | 위험 명령 승인 정책: `manual` \| `smart` \| `off` |
| hermes doctor | 공급망 어드바이저리 검사기 (Python CVE 탐지); 설정 검증 아님 |
| hermes config check | 설정 파일 유효성 검사 (`hermes doctor`와 별도) |
| ACP | Agent Client Protocol; Hermes를 에디터 네이티브 에이전트로 실행 |
| MCP | Model Context Protocol; 외부 툴 서버 연결 + Hermes MCP 서버 노출 |
| s6-overlay | Docker 이미지 내 프로세스 수퍼바이저 (PID 1); 크래시 시 자동 재시작 |
| FTS5 trigram | SQLite FTS5 한국어(CJK) 전문 검색 테이블 (`messages_fts_trigram`) |
| compress.protect_first_n | 항상 보존하는 첫 N개 메시지; 하드코딩 3 (설정 불가) |

---

## SUMMARY.md / src/ 폴더 구조 권장 사항

### 권장 SUMMARY.md 추가 내용

현재 SUMMARY.md의 끝에 다음을 추가:

```markdown
---

# 레퍼런스

- [CLI · 슬래시 커맨드 레퍼런스](19-cli-reference/index.md)
- [config.yaml 레퍼런스](20-config-reference/index.md)
- [마스터 트러블슈팅 색인](21-troubleshooting/index.md)
```

**섹션 이름:** `# 레퍼런스` 권장 (짧고 명확; 대안: `# 부록` 가능하나 `레퍼런스`가 검색성 우수)

### 권장 src/ 폴더 이름

```
src/
├── 19-cli-reference/
│   └── index.md
├── 20-config-reference/
│   └── index.md
└── 21-troubleshooting/
    └── index.md
```

**번호 체계:** Ch.19/20/21으로 기존 챕터 번호(`00-18`, 건너뜀 16) 이어서 진행. `appendix-*` 접두사 대신 숫자 연속 유지 권장 — 기존 패턴 일관성 + mdBook SUMMARY.md에서 경로 예측 가능.

### 계획(Plan) / Wave 구조 권장

- **Plan 06-01 (owner):** SUMMARY.md에 `# 레퍼런스` 섹션 + 3개 항목 추가. 세 개의 `src/NN/index.md` 스켈레톤 파일 생성. `mdbook build`로 중복 경로 오류 없음 확인. **Ch.19 (CLI 레퍼런스) 본문 작성.**
- **Plan 06-02 (body, Wave 2):** Ch.20 (config.yaml 레퍼런스) 본문 작성. Plan 06-01에 `depend_on`.
- **Plan 06-03 (body, Wave 2):** Ch.21 (트러블슈팅 색인) 본문 작성. Plan 06-01에 `depend_on`. (Wave 2에서 06-02와 병렬 실행 가능)

**Wave 2 병렬 가능 이유:** Plan 06-01이 SUMMARY.md와 세 스켈레톤 파일을 이미 소유한 후, Plans 06-02/06-03은 각자 다른 파일(20-config-reference vs 21-troubleshooting)을 쓰므로 쓰기 경합 없음.

**Pitfall B-11 방지:** SUMMARY.md에 이미 존재하는 경로를 추가하지 않도록 주의. 세 개의 새 경로(`19-cli-reference`, `20-config-reference`, `21-troubleshooting`)는 모두 신규 경로.

---

## 메타-정확성 리스크 및 플래너 지침

### 핵심 원칙

레퍼런스 페이지는 "전체 목록"을 지향하지 않는다. 다음 두 가지만 약속한다:

1. **Verified Core:** 2–5개 선행 연구 파일에서 HIGH confidence로 확인된 명령어·키만 수록.
2. **검증 필요:** 직접 확인하지 못한 항목은 개별적으로 `> 검증 필요` 표시.

이 프레임을 각 레퍼런스 페이지 상단의 `> 참고` callout으로 독자에게 명시해야 한다.

### 발행 전 필수 로컬 확인 항목

| 항목 | 명령어 |
|------|--------|
| 전체 최신 명령어 트리 | `hermes --help` |
| 설정 기본값 | `hermes config show` |
| `config check` 존재 여부 | `hermes config --help` |
| `approvals.timeout` 기본값 | `hermes config show \| grep timeout` |
| `approvals.cron_mode` 값 | `hermes config show \| grep cron_mode` |
| `hermes logs` 플래그 | `hermes logs --help` |
| `hermes mcp serve` 노출 툴 | `hermes mcp serve --verbose` (MCP 클라이언트 연결 후 툴 목록 확인) |
| `hermes gateway install --system` 경로 | Ubuntu VPS에서 `sudo hermes gateway install --system` 후 경로 확인 |
| 기본 `toolsets:` 키 이름 | `hermes config show \| grep toolsets` |
| `modal_image` 키 존재 | Modal 계정으로 `hermes setup terminal` 실행 |

### 절대로 추가하지 말 것

- `hermes security audit` — 존재하지 않음 (05-RESEARCH.md 확인)
- `agent.disabled_toolsets` — 존재하지 않음 (03-RESEARCH.md 확인)
- "PII 리댁션 설정 키" — 존재하지 않음 (05-RESEARCH.md 확인)
- 실제 스킬 이름/커뮤니티 스킬 수 — 롤링 릴리스로 지속 변경

---

## Sources

### PRIMARY (HIGH confidence — 이전 연구 파일에서 검증됨)

- `02-RESEARCH.md` — Ch.0–3: 설치 명령, 슬래시 커맨드, 기본 설정 키 (display, model), CLI 레퍼런스 URL 확인
- `03-RESEARCH.md` — Ch.4–8: 에이전트 루프, 컨텍스트 파일, 메모리 키, 압축 키, 툴 명령어, 백엔드 YAML 키 (Docker HIGH, Modal MEDIUM)
- `04-RESEARCH.md` — Ch.9–13: 스킬 CLI, MCP 명령어, 크론 CLI, 서브에이전트 설정, Kanban CLI, Curator CLI
- `05-RESEARCH.md` — Ch.14–18: 게이트웨이 명령어 (run/start/install 구분), .env 변수 전체 목록, 보안 설정, hermes doctor 정확한 기능, security audit 미존재 확인

### SECONDARY (MEDIUM confidence — 이전 연구에서 부분 확인)

- `https://hermes-agent.nousresearch.com/docs/reference/cli-commands` — 공식 CLI 레퍼런스 (이전 연구에서 참조됨)
- `https://github.com/NousResearch/hermes-agent/blob/main/cli-config.yaml.example` — 공식 설정 예시 파일
- `https://github.com/NousResearch/hermes-agent/blob/main/.env.example` — 공식 .env 예시 파일

### 실제 챕터 파일 (직접 추출)

- `src/01-install/index.md` through `src/18-security/index.md` — "흔한 오류 / 주의" 섹션 직접 추출 (로컬 파일, 100% 신뢰도)

---

## Metadata

**Confidence breakdown:**

| 영역 | 신뢰도 | 근거 |
|------|--------|------|
| CLI 명령어 (core — A~J 카테고리) | HIGH | 4개 선행 연구 파일 + 공식 docs |
| 슬래시 커맨드 (K 카테고리) | HIGH | 02/03/04-RESEARCH.md |
| Config.yaml 키 (대부분) | HIGH | 03/04/05-RESEARCH.md + 공식 설정 docs |
| `approvals.timeout` 기본값 | MEDIUM | 두 연구 파일에서 불일치 (60 vs 300) |
| `hermes logs` 플래그 | MEDIUM | 런북에서만 언급, CLI 레퍼런스 직접 확인 필요 |
| `modal_image` 키 이름 | MEDIUM | 설정 docs에서 확인, 실제 동작 로컬 확인 필요 |
| `hermes gateway install --system` 경로 | MEDIUM | GitHub Issue에서 언급, 정확한 경로 로컬 확인 필요 |
| 트러블슈팅 색인 항목 | HIGH | 실제 챕터 파일에서 직접 추출 (100% 신뢰도) |
| 용어 사전 | HIGH | 선행 연구 파일에서 확인된 정의 |
| SUMMARY.md / 파일 구조 | HIGH | 기존 패턴 분석 기반 |

**Research date:** 2026-06-11
**Valid until:** 2026-07-11 (Hermes 롤링 릴리스 — 발행 30일 전 `hermes --help`와 `hermes config show`로 재검증 필수)
**Key risk:** 레퍼런스 페이지는 일반 챕터보다 정확성이 더 중요합니다. `> 검증 필요` 항목은 발행 전 반드시 로컬에서 확인하거나, 확인 불가 시 해당 항목을 과감히 제거하십시오. 잘못된 명령어·키를 포함하는 것이 빠뜨리는 것보다 나쁩니다.
