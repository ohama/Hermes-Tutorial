# 마스터 트러블슈팅 색인

> **이 챕터를 시작하기 전에**
>
> 이 페이지는 튜토리얼 본문(Ch.1–18)의 모든 "흔한 오류 / 주의" 항목을 한곳에 모은 색인 부록입니다.
> 각 항목에서 출처 챕터로 바로 이동해 자세한 설명을 확인할 수 있습니다.
> 관련 레퍼런스: [Ch.19 CLI 레퍼런스](../19-cli-reference/index.md) · [Ch.20 config.yaml 레퍼런스](../20-config-reference/index.md)

> **빠른 검색:** 오류 메시지나 증상의 핵심 단어로 Ctrl+F 검색하거나,
> 아래 카테고리별로 찾아보십시오. 각 항목의 [Ch.N] 링크로 출처 챕터의
> "흔한 오류 / 주의" 섹션으로 바로 이동할 수 있습니다.

---

## 설치 / 첫 실행 오류 (Ch.1–3)

출처: [Ch.1 설치](../01-install/index.md#흔한-오류--주의) · [Ch.2 첫 실행](../02-first-run/index.md#흔한-오류--주의) · [Ch.3 모델 설정](../03-model/index.md#흔한-오류--주의)

### Ch.1 설치 오류

| # | 오류/증상 | 원인 | 해결책 |
|---|-----------|------|--------|
| 1 | `hermes: command not found` | PATH 미반영 | `source ~/.bashrc` 또는 `~/.zshrc` 실행; `which hermes`로 확인 |
| 2 | sudo 설치로 인한 권한 문제 | `sudo curl|bash` 사용 | `sudo rm /usr/local/bin/hermes` 후 표준 방법으로 재설치 |
| 3 | Windows 네이티브 제한 | PowerShell 설치 | POSIX PTY 기능·브라우저 대시보드 제한됨; WSL2 권장 |
| 4 | Termux 설치 실패 | 음성 의존성 오류 | 공식 문서 Termux 별도 안내 참조 |

### Ch.2 첫 실행 오류

| # | 오류/증상 | 원인 | 해결책 |
|---|-----------|------|--------|
| 1 | `"No provider configured"` 오류 | 공급자 미설정 | `Ctrl+D`로 종료 후 Ch.3 모델 설정 완료 |

### Ch.3 모델 설정 오류

| # | 오류/증상 | 원인 | 해결책 |
|---|-----------|------|--------|
| 1 | Claude 사용 불가 (구독 있음) | Claude Pro/Max 구독 ≠ API 접근 | console.anthropic.com에서 별도 API 키 발급; 또는 OpenRouter로 접근 |
| 2 | 녹색 체크마크 후에도 인증 실패 | Silent API Key Skip (Issue #16394) | `~/.hermes/.env`에서 잘못된 키 줄 직접 삭제 후 재설정 |
| 3 | API 키 git 노출 위험 | config.yaml에 키 저장 | API 키는 반드시 `.env`에 저장; `hermes model` 마법사 사용 |
| 4 | 모델 시작 거부 | 64K 미만 컨텍스트 윈도우 | 64,000 토큰 이상 컨텍스트 윈도우 모델 선택 |
| 5 | 채팅 중 공급자 추가 불가 | `/model`은 전환만 지원 | 세션 종료(`Ctrl+D`) 후 `hermes model` 실행 |

---

## 에이전트 동작 이해 (Ch.4–6)

출처: [Ch.4 에이전트 루프](../04-agent-loop/index.md#흔한-오류--주의) · [Ch.5 컨텍스트 파일](../05-context-files/index.md#흔한-오류--주의) · [Ch.6 메모리](../06-memory/index.md#흔한-오류--주의)

### Ch.4 에이전트 루프 오류

| # | 오류/증상 | 원인 | 해결책 |
|---|-----------|------|--------|
| 1 | 응답이 오래 걸리는 것처럼 느껴짐 | 동기 엔진 특성 | 도구 실행 중임; 정상 동작 |
| 2 | 에이전트가 작업 요약 후 중단 | `agent.max_turns` (기본 90) 초과 | `max_turns` 늘리기 또는 `delegate_task` 서브에이전트 활용 |

### Ch.5 컨텍스트 파일 오류

| # | 오류/증상 | 원인 | 해결책 |
|---|-----------|------|--------|
| 1 | `.hermes.md` 수정이 현재 세션에 반영 안 됨 | 세션 시작 시 한 번만 로드 | `hermes` 재시작 후 반영됨 |
| 2 | `.hermes.md`와 `AGENTS.md` 중 하나만 로드됨 | 우선순위: 첫 매치만 로드 | 두 파일 내용을 `.hermes.md`로 병합 |
| 3 | API 키 git 노출 | config.yaml에 저장 | `.env` 파일에 저장 (참조만 `${VAR}`) |
| 4 | `${VAR_NAME}` 치환 안 됨 | `$VAR_NAME`(중괄호 없음) 사용 | `${VAR_NAME}` 형식 사용 필수 |

### Ch.6 메모리 오류

| # | 오류/증상 | 원인 | 해결책 |
|---|-----------|------|--------|
| 1 | 세션 중 저장한 메모리가 현재 대화에 반영 안 됨 | 세션 시작 시 스냅샷 동결 | 의도된 동작; 다음 세션부터 반영 |
| 2 | 게이트웨이 2시간 사용으로 ~$12 비용 발생 | 기본 압축 임계값 50% | `compression.threshold: 0.30` 설정; `/compress` 수동 압축; 저비용 보조 모델 지정; `/usage` 모니터링 |
| 3 | `/session_search` 커맨드 없음 | 에이전트 내부 툴, 슬래시 커맨드 아님 | 자연어로 질문: "지난달에 배포에 대해 얘기한 내용 기억해?" |
| 4 | 압축 중 내용이 조용히 잘림 | 보조 모델의 작은 컨텍스트 윈도우 | 보조 모델도 큰 컨텍스트 윈도우 선택 (예: Gemini Flash 1M) |

---

## 도구 · 백엔드 · MCP (Ch.7–8, Ch.11)

출처: [Ch.7 툴 게이트웨이](../07-tools/index.md#흔한-오류--주의) · [Ch.8 터미널 백엔드](../08-backends/index.md#흔한-오류--주의) · [Ch.11 MCP 연동](../11-mcp/index.md#흔한-오류--주의)

### Ch.7 툴 게이트웨이 오류

| # | 오류/증상 | 원인 | 해결책 |
|---|-----------|------|--------|
| 1 | `approvals.mode: off`로 인한 위험 | YOLO 모드 활성화 | 로컬 백엔드에서 절대 사용 금지; 컨테이너 백엔드에서만 고려 |
| 2 | Tool Gateway 도구 호출 시 `subscription required` | 무료 계정 한도 초과 | `hermes portal info`로 구독 상태 확인; 유료 구독 또는 직접 API 키 사용 |
| 3 | `agent.disabled_toolsets` 설정이 동작하지 않음 | 존재하지 않는 키 | `toolsets:` 배열 또는 `hermes tools` UI 사용 |

### Ch.8 터미널 백엔드 오류

| # | 오류/증상 | 원인 | 해결책 |
|---|-----------|------|--------|
| 1 | `"Cannot connect to Docker daemon"` | Docker 데몬 미실행 | `docker info`로 확인 후 Docker 시작 |
| 2 | 컨테이너 내 생성 파일이 root 소유 | `docker_run_as_host_user: false` (기본값) | `docker_run_as_host_user: true` 설정 (단, apt install 권한 오류 발생 가능) |
| 3 | Modal 세션 간 파일 소실 | 에페머럴 특성 | `container_persistent: true` 설정 |
| 4 | `approvals.mode: off`가 로컬에서 위험 | 로컬 백엔드 사용 | 컨테이너 백엔드 사용 시에만 `off` 고려 |

### Ch.11 MCP 연동 오류

| # | 오류/증상 | 원인 | 해결책 |
|---|-----------|------|--------|
| 1 | `hermes mcp serve`가 전체 에이전트를 노출하지 않음 | 10개 메시징 툴만 노출 | 에디터 임베딩은 `hermes acp` 사용 |
| 2 | ACP와 MCP 혼동 | 별도 통합 | ACP = 에디터 네이티브 에이전트; MCP = 툴 서버 연결 |
| 3 | `tools.include`와 `tools.exclude` 동시 설정 시 `exclude` 무시 | `include` 우선순위 | `include`만 사용하는 화이트리스트 방식 권장 |
| 4 | 신뢰할 수 없는 MCP 서버 연결 후 보안 위협 | 툴 표면 확장 | `tools.include` 화이트리스트로 필요한 툴만 허용 |

---

## 스킬 · 학습 루프 · 자동화 (Ch.9–10, Ch.12)

출처: [Ch.9 스킬 시스템](../09-skills/index.md#흔한-오류--주의) · [Ch.10 학습 루프](../10-learning-loop/index.md#흔한-오류--주의) · [Ch.12 크론 스케줄러](../12-cron/index.md#흔한-오류--주의)

### Ch.9 스킬 시스템 오류

| # | 오류/증상 | 원인 | 해결책 |
|---|-----------|------|--------|
| 1 | 스킬 파일 추가 후 현재 세션에 반영 안 됨 | 스킬 인덱스 세션 시작 시 로드 | `hermes` 재시작 |
| 2 | 번들 스킬 수가 예상과 다름 | 버전마다 다름 (v0.10.0: 118개, v0.15.x: ~89개) | `hermes skills list`로 현재 설치 수 확인 |
| 3 | 스킬 설치 보안 경고 | 보안 스캔 감지 | `--force` 플래그로 강제 설치 가능하나 비위험 항목에만 사용 |

### Ch.10 학습 루프 오류

| # | 오류/증상 | 원인 | 해결책 |
|---|-----------|------|--------|
| 1 | 스킬 저장 제안을 놓침 | 에이전트 메시지 미확인 | 작업 완료 후 에이전트 메시지 주의 깊게 확인 |
| 2 | Curator가 즉시 실행되지 않음 | 비활동 기반 트리거 (2h 경과 + 7일 간격) | `hermes curator run`으로 즉시 수동 실행 |
| 3 | Honcho 설정 후 연결 실패 | 외부 계정 필요 | honcho.dev에서 계정 생성 + `HONCHO_API_KEY` 설정 |

### Ch.12 크론 스케줄러 오류

| # | 오류/증상 | 원인 | 해결책 |
|---|-----------|------|--------|
| 1 | 크론 작업이 컨텍스트를 모름 | 새로운 격리 세션으로 실행 | 프롬프트에 모든 컨텍스트 명시 (URL, 설정, 기호) |
| 2 | 크론 작업의 파일 경로가 예상과 다름 | `--workdir` 미지정 시 설치 디렉터리 사용 | 파일 I/O 작업은 `--workdir /절대/경로` 필수 지정; workdir 지정 시 순차 실행됨 |
| 3 | 새 Homebrew/nvm 도구가 launchd에서 인식 안 됨 (macOS) | launchd PATH 고정 | 새 도구 설치 후 `hermes gateway install` 재실행 |
| 4 | 스케줄 자동 실행 안 됨 | gateway 프로세스 미실행 | `hermes gateway` 또는 `hermes gateway install && hermes gateway start` |

---

## 서브에이전트 · Kanban (Ch.13)

출처: [Ch.13 서브에이전트](../13-subagents/index.md#흔한-오류--주의)

| # | 오류/증상 | 원인 | 해결책 |
|---|-----------|------|--------|
| 1 | 서브에이전트가 부모 대화 내용을 모름 | 완전 격리 새 세션 | `goal`과 `context`에 모든 필요 정보 명시 |
| 2 | 서브에이전트 비용 급증 | 각 서브에이전트 = 독립 LLM 세션 (~14K 토큰 오버헤드) | 단순 작업은 직접 툴 사용; 서브에이전트는 추론 집약적 작업에만 |
| 3 | 서브에이전트가 추가 서브에이전트 생성 불가 | `max_spawn_depth: 1` (기본) | `delegation.max_spawn_depth: 2` 이상으로 설정 (지수적 비용 주의) |
| 4 | Kanban과 `delegate_task` 혼동 | 서로 다른 용도 | 짧은 추론 → `delegate_task`; 장기·재개·인간참여 → Kanban |
| 5 | 일반 채팅에서 Kanban 툴 없음 | 기본 비활성 | `hermes gateway start`(디스패처) 또는 `config.yaml`에 `toolsets: [..., kanban]` 명시 |

---

## 게이트웨이 · 배포 · 보안 (Ch.14–15, Ch.17–18)

출처: [Ch.14 Telegram 게이트웨이](../14-gateways/index.md#흔한-오류--주의) · [Ch.15 추가 게이트웨이](../15-more-gateways/index.md#흔한-오류--주의) · [Ch.17 프로덕션 배포](../17-deploy/index.md#흔한-오류--주의) · [Ch.18 보안 하드닝](../18-security/index.md#흔한-오류--주의)

### Ch.14 Telegram 게이트웨이 오류

| # | 오류/증상 | 원인 | 해결책 |
|---|-----------|------|--------|
| 1 | 메시지가 간헐적으로 처리되거나 무시됨 | 동일 토큰 중복 폴링 (두 프로세스) | `hermes gateway status`로 인스턴스 확인; 토큰은 프로파일 간 공유 불가 |
| 2 | 봇이 모든 사용자 명령 실행 | `GATEWAY_ALLOW_ALL_USERS=true` | 플랫폼별 허용 목록 또는 `hermes pairing approve` 사용 |
| 3 | Telegram 토큰 노출 | config.yaml에 저장 | `.env`에만 저장; 노출 시 BotFather `/revoke`로 재발급 |
| 4 | 2시간 사용으로 ~$12 비용 | 기본 압축 임계값 | `compression.threshold: 0.30` + `/usage` 모니터링 |

### Ch.15 추가 게이트웨이 오류

| # | 오류/증상 | 원인 | 해결책 |
|---|-----------|------|--------|
| 1 | Discord 봇이 온라인인데 메시지를 완전히 무시 | **Message Content Intent 비활성** (최빈 원인) | Discord Developer Portal → Bot → Privileged Gateway Intents → Message Content Intent 활성화 → Save → 게이트웨이 재시작 |
| 2 | Slack 봇이 채널 메시지를 못 받음 | `channels:history`/`groups:history` 스코프 누락 | Slack App 설정에서 두 스코프 추가 후 앱 재설치 |
| 3 | WhatsApp 세션 단절/기능 제한 | 비공식 Web 프로토콜 | WhatsApp 정책 변경에 따른 것; 전용 번호 사용 권장 |

### Ch.17 프로덕션 배포 오류

| # | 오류/증상 | 원인 | 해결책 |
|---|-----------|------|--------|
| 1 | API 키·토큰이 손상됨 (VPS 초기 설정) | 웹 콘솔 특수 문자 오전송 | 반드시 SSH로 접속하여 설정 |
| 2 | 세션 파일·메모리 저장소 손상 | 두 프로세스가 동일 `~/.hermes` 공유 | 단일 컨테이너의 `hermes profile create` 사용 |
| 3 | Docker 컨테이너 파일 소유권 문제 | root 소유 파일 | 공식 이미지는 UID 10000 사용; `docker_run_as_host_user` 확인 |

### Ch.18 보안 하드닝 오류

> **주의:** `hermes doctor`와 `hermes security audit`에 대한 흔한 오해가 있습니다. 아래를 확인하십시오.

| # | 오류/증상 | 원인 | 해결책 |
|---|-----------|------|--------|
| 1 | `hermes doctor`로 설정 오류를 진단하려 함 | `hermes doctor` = **공급망 검사기** (설정 검증 아님) | 설정 검증은 `hermes config check` 사용 |
| 2 | `hermes security audit` 실행 오류 | **존재하지 않는 명령어** | `hermes doctor`(공급망), `hermes pairing`(인증), `hermes config check`(설정) 사용 |
| 3 | "PII 리댁션"으로 이름·이메일 필터 기대 | 일반 PII 필터 없음 | `security.redact_secrets`는 MCP 오류의 API 키 패턴만 처리 |
| 4 | 봇이 모든 인터넷 사용자 명령 실행 | `GATEWAY_ALLOW_ALL_USERS=true` | 플랫폼별 허용 목록 또는 DM 페어링 사용 |

---

## 용어 사전 (Glossary)

핵심 용어 빠른 참조. 상세 설명은 각 챕터 본문에서 확인하십시오.

| 용어 | 정의 |
|------|------|
| SOUL.md | 전역 에이전트 정체성 파일 (`~/.hermes/SOUL.md`); 프롬프트 레이어 1 |
| MEMORY.md | 에이전트 메모 파일 (`~/.hermes/memories/MEMORY.md`); 세션 시작 시 동결 로드 |
| USER.md | 사용자 프로파일 (`~/.hermes/memories/USER.md`); 세션 시작 시 동결 로드 |
| .hermes.md | 프로젝트 컨텍스트 파일; git root까지 상위 탐색; 세션 시작 시 한 번만 로드 |
| 스킬 (Skill) | 재사용 가능한 절차를 기술한 Markdown 파일; `~/.hermes/skills/` |
| Curator | 백그라운드 스킬 품질 검토 프로세스; 비활동 기반 트리거 (기본 7일 간격 + 2h 경과) |
| Tool Gateway | Nous Portal을 통해 라우팅되는 도구 서비스 (웹검색, 이미지, TTS, 브라우저) |
| delegate_task | 서브에이전트 스폰 도구; 격리 자식 에이전트로 병렬 추론 |
| Kanban | SQLite 기반 다중 에이전트 태스크 큐 시스템 (`~/.hermes/kanban.db`); 기본 비활성 |
| Tirith | 명령 실행 전 보안 패턴 스캔 (항상 활성); 구성 가능 |
| approvals.mode | 위험 명령 승인 정책: `manual` (기본) \| `smart` \| `off` |
| hermes doctor | 공급망 어드바이저리 검사기 (Python CVE 탐지); **설정 검증이 아님** |
| hermes config check | 설정 파일 유효성 검사 (`hermes doctor`와 별도) |
| ACP | Agent Client Protocol; Hermes를 에디터 네이티브 에이전트로 실행 |
| MCP | Model Context Protocol; 외부 툴 서버 연결 + Hermes MCP 서버 노출 |
| s6-overlay | Docker 이미지 내 프로세스 수퍼바이저 (PID 1); 크래시 시 자동 재시작 |
| FTS5 trigram | SQLite FTS5 한국어(CJK) 전문 검색 테이블 (`messages_fts_trigram`) |
| compress.protect_first_n | 항상 보존하는 첫 N개 메시지; 하드코딩 3 (설정 불가) |

---

## 다음 단계

이 페이지는 **Hermes Agent 튜토리얼 (Ch.0–18 본문 + Ch.19–21 레퍼런스 부록)의 마지막 페이지**입니다.

관련 레퍼런스 부록:

- [Ch.19 CLI · 슬래시 커맨드 레퍼런스](../19-cli-reference/index.md) — 모든 `hermes` 서브커맨드와 세션 내 슬래시 커맨드 목록
- [Ch.20 config.yaml 레퍼런스](../20-config-reference/index.md) — `~/.hermes/config.yaml` 키 전체 레퍼런스
