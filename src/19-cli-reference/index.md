# CLI · 슬래시 커맨드 레퍼런스

> **이 챕터를 시작하기 전에**
>
> 이 페이지는 튜토리얼 본문(Ch.0–18)에서 다룬 명령어를 한곳에 모은 부록입니다.
> 관련 레퍼런스: [Ch.20 config.yaml 레퍼런스](../20-config-reference/index.md) · [Ch.21 마스터 트러블슈팅 색인](../21-troubleshooting/index.md)

---

> 참고: 이 레퍼런스는 공식 문서와 튜토리얼 검증 과정에서 검증된 명령어만 수록합니다.
> Hermes는 롤링 릴리스 방식으로 배포되므로 명령어가 변경될 수 있습니다.
> `hermes --help` 및 각 서브커맨드의 `--help` 플래그로 현재 전체 목록을 확인하십시오.
> 아래 `> 검증 필요` 표시 항목은 로컬에서 추가 확인이 필요합니다.

### 공식 참조 소스

| 소스 | 용도 |
|------|------|
| `hermes --help` | 최신 전체 명령어 트리 (항상 실행 시점 정확) |
| `https://hermes-agent.nousresearch.com/docs/reference/cli-commands` | 공식 CLI 레퍼런스 (HIGH confidence) |

---

## A. 최상위 실행 명령어

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

## B. 모델 · 설정 명령어

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

**중요:** `hermes config check`는 `hermes doctor`와 다른 명령어입니다. 설정 파일 유효성 검사에는 반드시 `hermes config check`를 사용하십시오 ([정정 섹션](#정정-hermes-doctor-vs-hermes-config-check) 참조).

---

## C. 도구 명령어

```bash
# 검증: hermes rolling, 2026-06-11
hermes tools              # 플랫폼별 인터랙티브 툴 설정 UI (curses 기반)
hermes tools --summary    # 현재 활성화된 툴 요약 출력 후 종료
```

---

## D. 스킬 명령어

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

## E. MCP 명령어

```bash
# 검증: hermes rolling, 2026-06-11
hermes mcp                          # 인터랙티브 MCP 카탈로그 피커
hermes mcp catalog                  # 카탈로그 텍스트 목록
hermes mcp install <name>           # 큐레이션 카탈로그에서 설치
hermes mcp add <name> --preset <type>  # 사전 설정된 전송 방식으로 추가
hermes mcp configure <name>         # 설치 후 툴 선택 업데이트
hermes mcp login <server>           # OAuth 원격 서버 인증
hermes mcp serve                    # Hermes를 stdio MCP 서버로 실행 (약 10개 메시징 툴 노출)
hermes mcp serve --verbose          # 디버그 로깅
```

> **검증 필요:** `hermes mcp serve`가 노출하는 정확한 툴 이름 목록은 로컬 실행 후 `--verbose`로 확인 필요.

---

## F. 크론 스케줄러 명령어

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

## G. 게이트웨이 명령어

```bash
# 검증: hermes rolling, 2026-06-11
hermes gateway            # 포그라운드 실행 (Ctrl-C로 중지, 개발/테스트용)
hermes gateway run        # Docker 컨테이너 내 실행 시 메인 프로세스로 사용
hermes gateway install    # 사용자 서비스로 등록 (Linux: ~/.config/systemd/user/, macOS: launchd)
sudo hermes gateway install --system  # 시스템 서비스로 설치 (/etc/systemd/system/ 또는 유사 경로)
hermes gateway start      # 등록된 서비스 시작
hermes gateway stop       # 서비스 중지
hermes gateway restart    # 서비스 재시작
hermes gateway status     # 서비스 상태 확인
hermes gateway setup      # 인터랙티브 플랫폼 설정 마법사
```

> **검증 필요:** `hermes gateway install --system`의 정확한 서비스 파일 경로는 Ubuntu VPS 로컬 확인 권장 (GitHub Issue #16264에서 언급됨).

---

## H. 운영 명령어

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

> **검증 필요:** `hermes logs` 서브커맨드 플래그 (`--level WARNING`, `--since 1h`)는 운영 런북에 포함되었으나 공식 CLI 레퍼런스에서 직접 확인 필요. 로컬에서 `hermes logs --help`로 확인하십시오.

---

## I. Nous Portal / 인증 명령어

```bash
# 검증: hermes rolling, 2026-06-11
hermes setup --portal     # OAuth 로그인 + Nous 공급자 + Tool Gateway (원클릭)
hermes portal info        # 로그인 상태 + 구독 확인
hermes portal tools       # Tool Gateway 카탈로그
```

> **검증 필요:** `hermes auth add`가 별도 최상위 명령어로 존재하는지, 아니면 `hermes model` 마법사 내 서브명령인지 확인 필요.

---

## J. Slack/WhatsApp 특수 명령어

```bash
# 검증: hermes rolling, 2026-06-11
hermes slack manifest --write   # ~/.hermes/slack-manifest.json 생성
hermes whatsapp                 # WhatsApp 설정 마법사 (QR 코드 출력)
```

---

## 정정: hermes doctor vs hermes config check

> **중요 수정 사항:** 이 두 명령어는 자주 혼동되지만 전혀 다른 기능을 수행합니다.

| 명령어 | 실제 기능 | 오해 |
|--------|-----------|------|
| `hermes doctor` | 공급망 어드바이저리 검사 — Python 패키지 CVE 탐지 | ~~설정 검증~~ (아님) |
| `hermes config check` | 설정 파일(`config.yaml`) 유효성 검사 | — |

```bash
# 검증: hermes rolling, 2026-06-11
# 설정 오류 진단 시 올바른 명령어:
hermes config check

# 공급망(Python CVE) 어드바이저리 확인:
hermes doctor
hermes doctor --ack <advisory-id>   # 특정 어드바이저리 영구 무시
```

`hermes doctor`를 설정 검증 도구로 오해하는 경우가 많습니다. Ch.18에서 확립된 원칙: **설정 검증 = `hermes config check`**, **공급망 검사 = `hermes doctor`**.

---

## 정정: hermes security audit는 존재하지 않음

> 참고: `hermes security audit` 명령은 존재하지 않습니다.
> 보안 관련 작업은 다음으로 나뉩니다:
> - `hermes doctor` → 공급망 어드바이저리
> - `hermes pairing` → 인증된 사용자 관리
> - `hermes config check` → 설정 검증
> - Tirith 스캔은 모든 명령 실행 시 자동으로 수행됨

이 명령어를 사용 가능한 명령 목록에서 찾을 수 없는 것은 정상입니다. `hermes security audit`는 처음부터 존재하지 않는 명령어입니다 (05-RESEARCH.md 확인).

---

## 세션 내 슬래시 커맨드

슬래시 커맨드는 대소문자 비구분 (`/HELP` = `/help`).

| 커맨드 | 기능 |
|--------|------|
| `/help` | 사용 가능한 커맨드 목록 |
| `/model` | 현재 모델 표시 / 이미 설정된 공급자 간 전환 |
| `/personality [name]` | 세션 성격 변경 (14개 내장 + 커스텀) |
| `/tools` | 현재 세션 활성 툴 목록 |
| `/tools list` | 현재 세션 활성 툴 목록 |
| `/tools disable <toolset>` | 툴셋 비활성화 |
| `/tools enable <toolset>` | 툴셋 활성화 |
| `/skills browse` | 스킬 허브 탐색 |
| `/skills install <identifier>` | 스킬 설치 |
| `/compress` | 수동 컨텍스트 압축 |
| `/usage` | 현재 세션 토큰 사용량 표시 |
| `/rollback` | 마지막 체크포인트로 되돌리기 |
| `/voice on` | 음성 모드 활성화 |
| `/reasoning high` | 추론 수준 조정 (none/minimal/low/medium/high/xhigh) |
| `/title` | 세션 이름 지정 |
| `/status` | 세션 정보 요약 |
| `/stop` | 활성 에이전트 스레드 중단 |
| `/reload-mcp` | MCP 서버 재로드 (재시작 없이) |
| `/<skill-name> [task]` | 스킬 직접 실행 |
| `/<plugin>:<skill-name>` | 플러그인 네임스페이스 스킬 실행 |

**툴셋 비활성화:** `/tools disable <toolset>` 및 `/tools enable <toolset>`이 툴셋 제어의 유효한 경로입니다. `config.yaml`의 `agent.disabled_toolsets` 키는 존재하지 않으므로 사용하지 마십시오 — `toolsets:` 배열 또는 `hermes tools` UI를 사용하십시오.

**내장 퍼소날리티 목록 (14개):**

`helpful` / `concise` / `technical` / `creative` / `teacher` / `kawaii` / `catgirl` / `pirate` / `shakespeare` / `surfer` / `noir` / `uwu` / `philosopher` / `hype`

커스텀 퍼소날리티는 `config.yaml`의 `agent.personalities` 키로 정의합니다 ([Ch.20 config.yaml 레퍼런스](../20-config-reference/index.md) 참조).

---

## 키 바인딩

| 키 | 동작 |
|----|------|
| `Enter` | 메시지 전송 |
| `Alt+Enter` / `Ctrl+J` / `Shift+Enter` | 줄바꿈 (다음 줄) |
| `Ctrl+G` | 외부 에디터에서 입력 편집 |
| `Ctrl+C` | 에이전트 중단 (두 번 연속 → 강제 종료) |
| `Ctrl+D` | 종료 |

---

## 발행 전 로컬 확인 항목

이 레퍼런스에 `> 검증 필요` 표시가 붙은 항목들은 아래 방법으로 직접 확인하십시오.

| 확인 항목 | 명령어 | 상태 |
|-----------|--------|------|
| 전체 최신 명령어 트리 | `hermes --help` | > 검증 필요 |
| `config check` 정확한 동작 | `hermes config --help` | > 검증 필요 |
| `approvals.timeout` 기본값 | `hermes config show \| grep timeout` | > 검증 필요 |
| `hermes logs` 서브커맨드 플래그 | `hermes logs --help` | > 검증 필요 |
| `hermes mcp serve` 노출 툴 목록 | `hermes mcp serve --verbose` (MCP 클라이언트 연결 후) | > 검증 필요 |
| `hermes gateway install --system` 경로 | Ubuntu VPS에서 직접 실행 후 경로 확인 | > 검증 필요 |
| `hermes auth add` 존재 여부 | `hermes --help \| grep auth` | > 검증 필요 |
| 기본 `toolsets:` 키 이름 | `hermes config show \| grep toolsets` | > 검증 필요 |

---

## 다음 단계

이전 챕터: [Ch.18 보안 하드닝](../18-security/index.md)

다음: [Ch.20 config.yaml 레퍼런스](../20-config-reference/index.md) — Hermes의 모든 설정 키를 카테고리별로 정리한 레퍼런스입니다.
