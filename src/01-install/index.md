# 설치

> **이 챕터를 시작하기 전에**
> - Git이 설치되어 있어야 합니다.
> - 인터넷 연결이 필요합니다.

## 개요

이 챕터에서는 macOS, Linux, WSL2, Windows 각 플랫폼에 Hermes를 설치하고, PATH를 반영한 뒤 `hermes version`으로 설치를 확인합니다. PATH 재로드는 첫 실행 실패의 가장 흔한 원인이므로 설치 직후 반드시 수행하십시오. (참고: `hermes doctor`는 흔히 "설치 검증" 도구로 오해되지만 실제로는 공급망 CVE 검사기입니다 — 정정 상세는 [Ch.18 보안](../18-security/index.md)·[Ch.19 CLI 레퍼런스](../19-cli-reference/index.md) 참고. 설정 검증은 `hermes config check`를 사용합니다.)

---

## 사전 요구사항

설치 스크립트를 실행하기 전에 아래 요구사항을 확인하십시오.

| 요구사항 | 플랫폼 | 비고 |
|----------|--------|------|
| Git | macOS / Linux / WSL2 / Termux | 유일한 수동 설치 요구사항 |
| PowerShell | Windows | 기본 포함 |
| (자동 설치) Python 3.11 | 모두 | 직접 설치 금지 — 인스톨러가 uv로 관리 |
| (자동 설치) Node.js v22 | 모두 | 직접 설치 금지 |
| (자동 설치) ripgrep, ffmpeg | 모두 | 직접 설치 금지 |

Python, Node.js, ripgrep, ffmpeg는 **직접 설치하지 마십시오.** Hermes 인스톨러가 의존성 버전을 직접 관리합니다.

---

## 설치 명령 (macOS / Linux / WSL2)

```bash
# 검증: hermes rolling (설치 시점 최신), 2026-06-09
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

스크립트는 코드를 `~/.hermes/hermes-agent/`에, 실행 파일을 `~/.local/bin/hermes`에 설치합니다.

> 참고: macOS/Windows용 Hermes Desktop GUI 인스톨러도 있습니다 (hermes-agent.nousresearch.com/desktop). 이 튜토리얼은 개발자 중심으로 CLI 설치(curl/ps1)를 기본 경로로 사용합니다.

---

## 설치 명령 (Windows 네이티브 PowerShell)

PowerShell을 관리자 권한 **없이** 실행하십시오.

```powershell
# 검증: hermes rolling (설치 시점 최신), 2026-06-09
iex (irm https://hermes-agent.nousresearch.com/install.ps1)
```

> 참고 (Windows 사용자): PowerShell 설치 방법은 CLI와 게이트웨이를 지원하지만,
> 브라우저 대시보드 채팅 창과 일부 POSIX PTY 기능이 제한됩니다.
> 모든 기능을 사용하려면 WSL2 환경을 권장합니다.

> 참고 (Android/Termux): 표준 curl|bash 설치가 음성 의존성 오류로 실패할 수 있습니다. Termux 설치는 공식 문서의 별도 안내를 참조하십시오.

---

## PATH 재로드 (가장 흔한 첫 실행 실패)

설치가 완료되어도 현재 셸에 PATH가 반영되지 않으면 `hermes: command not found` 오류가 납니다. 설치 직후 반드시 아래 명령을 실행하십시오.

```bash
# 검증: bash/zsh shell reload, 2026-06-09
# bash 사용자:
source ~/.bashrc

# zsh 사용자 (macOS 기본):
source ~/.zshrc
```

셸 종류를 모르거나 위 명령이 효과 없을 경우, 아래 명령으로 현재 세션에 PATH를 직접 추가하십시오.

```bash
# 검증: manual PATH export, 2026-06-09
export PATH="$HOME/.local/bin:$PATH"
```

이 `export`는 **현재 세션에만** 유효합니다. 영구 적용은 `~/.bashrc` 또는 `~/.zshrc` 파일에 해당 줄을 추가해야 합니다.

PATH 반영 여부를 확인하십시오.

```bash
# 검증: hermes rolling, 2026-06-09
which hermes
```

예상 출력:
- macOS: `/Users/<username>/.local/bin/hermes`
- Linux / WSL2: `/home/<username>/.local/bin/hermes`

출력이 비어 있으면 PATH가 아직 반영되지 않은 것입니다. 새 터미널 탭을 열거나 위의 `source` 명령을 다시 실행하십시오.

---

## 설치 위치

| 설치 방식 | 코드 위치 | 바이너리 | 데이터 |
|-----------|-----------|---------|--------|
| 사용자 설치 (표준) | `~/.hermes/hermes-agent/` | `~/.local/bin/hermes` | `~/.hermes/` |
| root 설치 (비권장) | `/usr/local/lib/hermes-agent/` | `/usr/local/bin/hermes` | `/root/.hermes/` |

표준 사용자 설치를 권장합니다. root 설치는 설정 파일이 root 소유가 되어 이후 관리가 복잡해집니다.

---

## 설치 확인과 진단 도구

설치가 완료되면 먼저 `hermes version`으로 바이너리가 PATH에서 실행되는지 확인합니다.

```bash
# 검증: hermes rolling, 2026-06-09
hermes version
```

설정 파일(`config.yaml`)의 유효성을 점검하려면 `hermes config check`를 사용합니다.

```bash
# 검증: hermes rolling, 2026-06-09
hermes config check
```

> **정정 — `hermes doctor`는 설치/설정 검증 도구가 아닙니다.** `hermes doctor`는 활성 Python 환경에서 알려진 취약 패키지(CVE)를 검사하는 **공급망 어드바이저리** 도구입니다. 설정 검증은 위의 `hermes config check`를, 설치 확인은 `hermes version`을 사용하십시오. 자세한 내용은 [Ch.18 보안](../18-security/index.md)과 [Ch.19 CLI 레퍼런스](../19-cli-reference/index.md)를 참고하세요.

```bash
# 검증: hermes rolling, 2026-06-09
hermes doctor   # 공급망 CVE 검사 (설치/설정 검증 아님)
```

```text
# [로컬 실행 후 캡처 필요 — 출력은 환경에 따라 다름]
[hermes doctor 실제 출력 — 로컬 실행 후 캡처 교체 필요 (검증 미완)]
```

> 검증 필요: 위 출력은 자리표시자입니다. 실제 `hermes doctor` 출력은 로컬 실행 후 캡처로 교체해야 합니다.

---

## 버전 확인: hermes version

```bash
# 검증: hermes rolling, 2026-06-09
hermes version
```

출력 예시: `version: 0.8.0 (2026.4.8) [af4abd2f]`

Hermes는 롤링 릴리스이므로 실행 시점에 따라 버전 문자열이 다를 수 있습니다. 위 예시는 참고용이며 현재 버전을 단정하지 않습니다.

---

## 흔한 오류 / 주의

**주의 1 — PATH 미반영 (CRITICAL)**

> 주의: "hermes: command not found" 오류가 나타나면 PATH가 아직 반영되지 않은 것입니다.
> `source ~/.bashrc` (또는 `~/.zshrc`)를 실행하거나 새 터미널 탭을 여십시오.
> `which hermes`로 인식 여부를 확인하세요.

**주의 2 — sudo 사용 금지 (CRITICAL)**

> 경고: 설치 스크립트를 `sudo`로 실행하지 마십시오.
> sudo 설치 시 바이너리가 `/usr/local/bin`에 들어가 설정 파일이 root 소유가 됩니다.
> 이미 sudo로 설치했다면: `sudo rm /usr/local/bin/hermes` 후 표준 방법으로 재설치하십시오.

**주의 3 — Windows 네이티브 제한**

> 참고 (Windows 사용자): PowerShell 설치 방법은 CLI와 게이트웨이를 지원하지만,
> 브라우저 대시보드 채팅 창과 일부 POSIX PTY 기능이 제한됩니다.
> 모든 기능을 사용하려면 WSL2 환경을 권장합니다.

**주의 4 — Termux 별도 경로 (MEDIUM)**

> 참고 (Android/Termux): 표준 curl|bash 설치가 음성 의존성 오류로 실패할 수 있습니다.
> Termux 설치는 공식 문서의 별도 안내를 참조하십시오.

---

## 다음 단계

이전 챕터: [Ch.0 소개](../00-intro/index.md)

다음 챕터: [Ch.2 첫 실행](../02-first-run/index.md)
