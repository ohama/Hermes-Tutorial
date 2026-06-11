# 보안 하드닝

> **이 챕터를 시작하기 전에**
>
> 선행 챕터: [Ch.14 메시징 게이트웨이](../14-gateways/index.md)(게이트웨이 인증·DM 페어링),
> [Ch.17 프로덕션 배포](../17-deploy/index.md)(컨테이너 격리·서비스 등록).
>
> 이 챕터는 튜토리얼 본문(Ch.0–18)의 마지막 단계입니다. 배포된 봇을 프로덕션 환경에서
> 안전하게 운영하는 데 필요한 보안 설정을 다룹니다.

---

## 개요

프로덕션에서 실행되는 봇은 실제 봇 토큰·API 키를 다루고 터미널 명령을 실행합니다.
보안 하드닝은 세 가지 축으로 구성됩니다.

1. **위험 명령 승인** — 승인 모드(`approvals.mode`)로 위험한 명령 실행 전 확인을 강제합니다.
2. **비밀 관리** — API 키·봇 토큰을 `.env`에 격리하고 Git 노출을 방지합니다.
3. **운영 진단 런북** — `hermes doctor`(공급망)·`hermes config check`(설정)·`hermes status`(런타임)로 프로덕션 상태를 지속적으로 점검합니다.

---

## 승인 모드 (approvals)

Hermes는 파일 삭제·스크립트 실행 등 잠재적으로 위험한 명령을 수행하기 전에 사용자 확인을 요청할 수 있습니다.

```yaml
# 검증: hermes rolling, 2026-06-11
approvals:
  mode: manual         # manual (기본, 권장) | smart | off
  timeout: 300         # 승인 대기 시간 (초, 기본값)
  cron_mode: manual    # 헤드리스 cron 실행 시: manual | auto
```

### 모드 비교

| 모드 | 동작 | 권장 환경 |
|------|------|-----------|
| `manual` | 위험한 명령 실행 전 **항상** 사용자 승인 요청 | **프로덕션 기본값** |
| `smart` | 보조 LLM이 위험도 평가 — 저위험 자동 승인, 고위험 자동 거부, 불확실 시 수동 위임 | 반자동화 환경 |
| `off` | 모든 승인 체크 비활성화 (`--yolo`와 동일) | 컨테이너로 격리된 환경에만 한정 |

**Hardline blocklist:** `off` 모드에서도 `rm -rf /`, 포크 폭탄, `mkfs.*` 등의 명령은 항상 차단됩니다.

### 프로덕션 권장: `manual` 유지

> **경고: `approvals.mode: off`는 위험한 명령에 대한 모든 안전 체크를 비활성화합니다.**
>
> 로컬 백엔드에서는 절대 사용하지 마십시오.
>
> Docker/Modal/Daytona 컨테이너가 보안 경계 역할을 할 때만 고려하십시오.
>
> 활성화 시 빨간색 배너 "⚠ YOLO mode"가 표시됩니다.

프로덕션에서는 approvals.mode를 manual 유지하는 것이 기본 원칙입니다. `smart` 모드는 보조 LLM에 의존하므로 오분류 위험이 있습니다.

---

## API 키 관리 (.env)

### .env vs config.yaml 분리 원칙

| 파일 | 저장 내용 | 예시 |
|------|-----------|------|
| `~/.hermes/.env` | **비밀만** | API 키, 봇 토큰, OAuth 자격증명 |
| `~/.hermes/config.yaml` | 비밀 외 설정 | 모델 선택, 압축 설정, 백엔드 타입 |

비밀과 설정을 분리하면 config.yaml을 안전하게 버전 관리에 포함할 수 있습니다.

### 키 설정 방법

```bash
# 검증: hermes rolling, 2026-06-11
# .env 파일 권한 설정 (최초 1회 — 중요)
chmod 600 ~/.hermes/.env

# 올바른 키 설정 방법 (→ .env로 자동 라우팅됨)
hermes config set ANTHROPIC_API_KEY <YOUR_ANTHROPIC_API_KEY>
hermes config set OPENROUTER_API_KEY <YOUR_OPENROUTER_API_KEY>

# 저장된 키 확인
cat ~/.hermes/.env
```

> **주의:** 위 코드 블록의 `<YOUR_ANTHROPIC_API_KEY>` 등은 placeholder입니다.
> 실제 키를 직접 입력하십시오. **실제 키를 스크린샷·이슈·채팅에 절대 노출하지 마십시오.**

### Git 노출 방지

```bash
# 검증: hermes rolling, 2026-06-11
# 프로젝트 .gitignore에 추가
echo ".hermes/" >> .gitignore

# 홈 디렉터리 전역 gitignore (선택)
echo "~/.hermes/.env" >> ~/.gitignore
```

`~/.hermes/.env`는 **절대 Git 저장소에 커밋하지 마십시오.** `.env`가 이미 추적되고 있다면 `git rm --cached ~/.hermes/.env`로 제거하십시오.

### Silent API Key Skip 버그 주의 (GitHub Issue #16394)

> **주의 (GitHub Issue #16394):**
>
> `~/.hermes/.env`에 키 항목이 이미 존재하면 — 값이 잘못되어도 — `hermes setup`이 조용히 건너뜁니다.
> 녹색 체크마크가 표시되어도 키가 유효하지 않을 수 있습니다.
>
> **해결책:**
>
> 1. `~/.hermes/.env`를 직접 열어 잘못된 키 줄을 삭제한 뒤 `hermes setup` 재실행
> 2. 또는 `hermes config set PROVIDER_API_KEY <새 키>`로 덮어쓰기

---

## Tirith 보안 스캐닝

Tirith는 Hermes에 내장된 실시간 명령 스캐너입니다. 명령 실행 전 자동으로 악의적 패턴을 검사합니다.

```yaml
# 검증: hermes rolling, 2026-06-11
security:
  tirith_enabled: true       # 기본값: true
  tirith_timeout: 5          # 스캔 제한 시간 (초)
  tirith_fail_open: true     # 스캐너 미응답 시 실행 허용 (false = 차단)
  redact_secrets: true       # MCP 오류 메시지의 자격증명 패턴 난독화 (기본값: true)
```

Tirith가 감지하는 패턴:

- **호모그래프 URL 스푸핑** — 국제화 도메인을 이용한 피싱 URL (예: `аpple.com` vs `apple.com`)
- **파이프-인터프리터 패턴** — `curl | bash`, `wget | sh` 등 원격 스크립트 직접 실행
- **터미널 주입 공격** — 이스케이프 시퀀스를 이용한 터미널 명령 주입

Tirith는 첫 사용 시 SHA-256 체크섬 검증 후 GitHub에서 자동 설치됩니다.

### `redact_secrets`의 정확한 범위 (PII 정정)

> **참고: Hermes의 "PII 리댁션"은 일반적인 개인정보 필터링이 아닙니다.**
>
> **실제 기능:** MCP 오류 메시지에서 **자격증명 패턴**을 난독화합니다.
>
> - GitHub PAT (`ghp_...`)
> - OpenAI 스타일 키 (`sk-...`)
> - Bearer 토큰, 비밀번호/키 파라미터
>
> 이 기능은 `security.redact_secrets: true`(기본값)로 제어됩니다.
>
> **별도의 "PII 리댁션 설정 키"는 존재하지 않습니다.** 이름, 이메일, 전화번호 등 일반 개인정보를 자동으로 필터링하지 않습니다.

---

## 운영 런북: 진단 명령

### hermes doctor — 공급망 보안 검사

> **중요 수정 사항:** `hermes doctor`는 설정 검증 도구가 **아닙니다**.
>
> `hermes doctor`는 **공급망 어드바이저리 검사기**입니다 — 활성 Python venv에서
> 알려진 취약한 패키지(CVE)를 감지하고 2–4단계 수정 지침을 제공합니다.
>
> **설정 검증은 별도 명령 `hermes config check`를 사용하십시오.**

```bash
# 검증: hermes rolling, 2026-06-11
# 공급망 어드바이저리 확인 (Python 패키지 CVE)
hermes doctor

# 검토된 어드바이저리를 확인 처리 (영구 무시)
hermes doctor --ack <advisory-id>

# 설정 파일 유효성 검사 (설정 검증 — doctor와 별개)
hermes config check

# 전체 설정 확인
hermes config show

# 런타임 상태 (에이전트/인증/플랫폼)
hermes status

# 게이트웨이 서비스 상태
hermes gateway status

# 경고 이상 로그 실시간 확인
hermes logs -f --level WARNING
```

**`hermes doctor` 출력 (어드바이저리 없을 때):**

```
# [로컬 실행 후 캡처 필요 — doctor 정상 실행 시 출력 형식]
```

> 검증 필요: `hermes doctor` 어드바이저리 없을 때의 정확한 출력 형식을 로컬에서 실행하여 확인하십시오.

### 운영 런북 절차

프로덕션 봇의 정기 상태 점검 순서입니다.

```bash
# 검증: hermes rolling, 2026-06-11
# 1. 공급망 보안 확인 (Python 패키지 CVE)
hermes doctor

# 2. 설정 검증 (config.yaml 유효성)
hermes config check

# 3. API 연결 및 모델 상태 확인 (런타임)
hermes status

# 4. 게이트웨이 서비스 상태
hermes gateway status

# 5. 최근 오류 로그 확인
hermes logs --level ERROR --since 1h

# 6. 업데이트 확인
hermes update --check

# 7. 사용량 통계 (7일)
hermes insights --days 7
```

### `hermes security audit`는 존재하지 않음

> **참고: `hermes security audit` 명령은 존재하지 않습니다 (공식 문서 미존재).**
>
> 보안 관련 작업은 다음 명령으로 나뉩니다:
>
> ```
> hermes doctor       → 공급망 어드바이저리(Python 패키지 CVE)
> hermes pairing list → 인증된 사용자 관리
> hermes config check → 설정 검증
> (Tirith 스캔은 명령 실행 시 자동)
> ```
>
> `hermes security audit`를 실행하면 "명령을 찾을 수 없음" 오류가 발생합니다.
> 이 명령 이름을 문서나 스크립트에 사용하지 마십시오.

---

## 게이트웨이 인증 + 프로덕션 체크리스트

### 인증 우선순위

게이트웨이 인증은 아래 순서로 우선순위가 적용됩니다 (낮은 번호가 더 높은 우선순위):

1. **`GATEWAY_ALLOW_ALL_USERS=true` — 절대 금지** (터미널 도구 접근 봇에서 누구나 명령 실행 가능)
2. `GATEWAY_ALLOWED_USERS` — 글로벌 허용 목록
3. 플랫폼별 허용 목록 (`TELEGRAM_ALLOWED_USERS`, `DISCORD_ALLOWED_USERS` 등)
4. DM 페어링 승인 목록 (`hermes pairing approve`)
5. **기본: 전체 거부** — 위 목록에 없는 사용자는 자동 차단

`unauthorized_dm_behavior: pair`로 설정하면 새 사용자가 봇에 DM을 보낼 때 8자리 일회성 페어링 코드를 발급받고, 운영자가 `hermes pairing approve`로 승인할 수 있습니다.

### 게이트웨이 보안 설정

```yaml
# 검증: hermes rolling, 2026-06-11
# config.yaml 보안 관련 설정
unauthorized_dm_behavior: pair    # pair (페어링 코드 발송) | ignore (무시)
security:
  allow_private_urls: false       # RFC 1918 내부 URL 접근 차단
  allow_lazy_installs: false      # 런타임 의존성 자동 설치 차단
```

### 프로덕션 10단계 체크리스트

```
# 검증: 공식 보안 문서 기반, 2026-06-11
□ 1. 명시적 허용 목록 설정 (GATEWAY_ALLOW_ALL_USERS=true 절대 사용 금지)
□ 2. 컨테이너 백엔드 사용 (terminal.backend: docker)
□ 3. CPU/메모리/디스크 제한 설정
□ 4. ~/.hermes/.env 권한 설정 (chmod 600)
□ 5. DM 페어링으로 사용자 인증
□ 6. command_allowlist 정기 감사
□ 7. terminal.cwd 설정으로 민감한 디렉터리 접근 차단
□ 8. 비루트 계정으로 실행
□ 9. ~/.hermes/logs/ 모니터링
□ 10. hermes update 로 최신 상태 유지
```

---

## 흔한 오류 / 주의

이 챕터에서 다룬 세 가지 정정 사항을 핵심만 다시 확인합니다.

> **정정 1 — `hermes doctor`는 설정 검증 도구가 아닙니다.**
>
> `hermes doctor`는 공급망 어드바이저리 검사기(Python 패키지 CVE 탐지)입니다.
> 설정 파일 유효성 검사는 `hermes config check`를 사용하십시오.

> **정정 2 — `hermes security audit`는 존재하지 않습니다.**
>
> 이 명령을 실행하려 하면 오류가 발생합니다. 보안 작업은
> `hermes doctor`(공급망), `hermes pairing`(인증), `hermes config check`(설정)으로 나뉩니다.

> **정정 3 — "PII 리댁션"은 일반 개인정보 필터가 아닙니다.**
>
> `security.redact_secrets: true`는 MCP 오류 메시지에서 API 키·토큰 패턴만 난독화합니다.
> 이름, 이메일, 전화번호 같은 일반 PII는 자동으로 처리되지 않습니다.

> **주의 — `GATEWAY_ALLOW_ALL_USERS=true` 절대 사용 금지**
>
> 터미널 도구에 접근하는 에이전트에서 이 옵션을 활성화하면
> 인터넷의 누구든 봇에게 명령어 실행을 지시할 수 있습니다.
> 반드시 플랫폼별 허용 목록이나 DM 페어링을 사용하십시오.

---

## 마무리

이 챕터로 Hermes Agent 튜토리얼 본문(Ch.0–18)이 마무리됩니다.

지금까지 다룬 내용을 정리하면:

- **Ch.0–3:** 설치, 첫 실행, 에이전트 루프 이해
- **Ch.4–8:** 프로파일, 기억, 도구, 커스텀 스킬
- **Ch.9–13:** 스킬 큐레이션, MCP 통합, 멀티에이전트, 크론, 워크플로우
- **Ch.14–15:** 메시징 게이트웨이 (Telegram, Discord, Slack, WhatsApp, Signal, Email)
- **Ch.17:** 프로덕션 배포 (Docker, VPS/systemd, Modal, Daytona)
- **Ch.18 (이 챕터):** 보안 하드닝 — 승인 모드, API 키 관리, Tirith, 운영 런북

이전 챕터로 돌아가려면 [Ch.17 프로덕션 배포](../17-deploy/index.md)를 참조하십시오.

다음 단계로, CLI 명령어·슬래시 커맨드·설정 키의 전체 목록과 통합 트러블슈팅 색인은 레퍼런스 부록에 정리되어 있습니다.

다음: [Ch.19 CLI·슬래시 커맨드 레퍼런스](../19-cli-reference/index.md)
