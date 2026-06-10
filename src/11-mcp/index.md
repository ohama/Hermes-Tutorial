# MCP 연동

> **이 챕터를 시작하기 전에**
> - [Ch.7 툴 게이트웨이](../07-tools/index.md)를 완료했는지 확인하세요 — 에이전트의 툴 표면 개념이 이 챕터의 기반입니다.
> - [Ch.9 스킬 시스템](../09-skills/index.md)을 완료했는지 확인하세요 — 절차적 메모리와 툴 흐름을 이해한 상태에서 MCP를 읽으면 훨씬 자연스럽습니다.

## 개요

MCP(Model Context Protocol)는 Hermes의 **양방향** 툴 확장 프로토콜입니다. 두 가지 방향으로 동작합니다.

- **(a) Hermes를 MCP 클라이언트로:** 외부 MCP 서버(GitHub, 데이터베이스, 파일시스템, 원격 API 등)를 Hermes에 연결해 에이전트의 툴 표면을 동적으로 확장합니다.
- **(b) Hermes를 MCP 서버로:** `hermes mcp serve` 명령으로 Hermes를 stdio MCP 서버로 노출해, Claude Desktop·Cursor·VS Code·Zed 같은 MCP 클라이언트에서 Hermes의 플랫폼 메시징 툴을 사용할 수 있게 합니다.

이 챕터는 MCP만 다룹니다. ACP(Agent Client Protocol)는 MCP와 **별개의 통합**이므로 아래 "개념: ACP와 MCP는 별개입니다" 절에서 간단히 구분합니다.

---

## 개념: ACP와 MCP는 별개입니다

ACP와 MCP는 이름이 비슷해 자주 혼동되지만, 완전히 다른 통합입니다.

| 항목 | ACP (Agent Client Protocol) | MCP (Model Context Protocol) |
|------|------------------------------|-------------------------------|
| **목적** | 에디터 임베딩 — Hermes를 에디터 네이티브 코딩 에이전트로 실행 | 외부 툴 서버 연결 + Hermes를 MCP 서버로 노출 |
| **방향** | 에디터 → Hermes | 양방향 |
| **지원 에디터** | VS Code, Zed, JetBrains | Claude Desktop, Cursor, VS Code (MCP 클라이언트), Zed |
| **핵심 명령어** | `hermes acp` / `hermes-acp` | `hermes mcp serve` (서버) / config.yaml `mcp_servers` (클라이언트) |
| **추가 설치** | `pip install -e '.[acp]'` | 기본 설치에 포함 |
| **프로토콜** | stdio/JSON-RPC, ACP 전용 렌더링 | stdio 또는 HTTP |

**이 챕터는 MCP만 다룹니다.** ACP를 사용해 Hermes를 에디터 네이티브 에이전트로 실행하는 방법(`hermes acp`)은 본 튜토리얼의 범위 밖이므로, 공식 문서를 직접 참고하십시오.

---

## 개념: 모드 1 — Hermes를 MCP 클라이언트로 (외부 서버 연결)

외부 MCP 서버를 Hermes에 연결하는 방법은 두 가지입니다: **명령 방식(CLI)** 과 **config.yaml 방식**.

### 명령 방식 (CLI)

```bash
# 검증: hermes rolling, 2026-06-10
hermes mcp                              # 인터랙티브 MCP 카탈로그 피커
hermes mcp catalog                      # 카탈로그 텍스트 목록
hermes mcp install <name>               # 큐레이션 카탈로그에서 설치
hermes mcp add <name> --preset <type>   # 사전 설정된 전송 방식으로 추가
hermes mcp configure <name>             # 설치 후 툴 선택 업데이트
hermes mcp login <server>              # OAuth 원격 서버 인증 완료
```

### config.yaml 방식 — stdio 서버 (로컬 프로세스)

```yaml
# 검증: hermes rolling, 2026-06-10
mcp_servers:
  filesystem:
    command: "npx"
    args: ["-y", "@modelcontextprotocol/server-filesystem", "/home/user/my-project"]
    env:
      SOME_VAR: "value"
    tools:
      include: [read_file, list_directory]    # 허용 툴 화이트리스트
      exclude: [write_file]                   # 제외 (include가 있으면 무시됨)
    timeout: 30                               # 툴 실행 제한 시간 (초)
    supports_parallel_tool_calls: true        # 병렬 실행 허용
```

### config.yaml 방식 — HTTP 서버 (원격)

```yaml
# 검증: hermes rolling, 2026-06-10
mcp_servers:
  remote_api:
    url: "https://mcp.example.com/mcp"
    headers:
      Authorization: "Bearer ${MY_API_TOKEN}"   # .env에서 참조
    ssl_verify: true
```

### config.yaml 방식 — OAuth 인증 HTTP 서버

```yaml
# 검증: hermes rolling, 2026-06-10
mcp_servers:
  linear:
    url: "https://mcp.linear.app/mcp"
    auth: oauth
    # hermes mcp login linear 로 PKCE OAuth 완료
    # 토큰 캐시: ~/.hermes/mcp-tokens/linear.json
```

Hermes는 OAuth PKCE 흐름(디스커버리, 동적 클라이언트 등록, 토큰 교환·갱신·캐시)을 자동으로 처리합니다. 토큰은 `~/.hermes/mcp-tokens/<server>.json`에 캐시됩니다.

### 툴 네이밍 규칙

MCP 서버에서 가져온 툴은 `mcp_<서버명>_<툴명>` 형식으로 등록됩니다. 서버명이나 툴명에 하이픈이나 점이 있으면 언더스코어로 변환됩니다.

예: `filesystem` 서버의 `read-file` 툴 → `mcp_filesystem_read_file`

### 재시작 없이 MCP 리로드

```
/reload-mcp
```

config.yaml의 `mcp_servers` 변경 사항을 hermes 재시작 없이 즉시 반영합니다.

---

## 실습: 파일시스템 MCP 서버 연결

파일시스템 MCP 서버(`@modelcontextprotocol/server-filesystem`)를 Hermes에 연결하는 전체 흐름입니다.

**1단계: config.yaml에 mcp_servers 항목 추가**

```yaml
# 검증: hermes rolling, 2026-06-10
mcp_servers:
  filesystem:
    command: "npx"
    args: ["-y", "@modelcontextprotocol/server-filesystem", "/home/user/my-project"]
    tools:
      include: [read_file, list_directory, search_files]
    timeout: 30
    supports_parallel_tool_calls: true
```

**2단계: MCP 리로드 (hermes 재시작 불필요)**

```
/reload-mcp
```

**3단계: 에이전트가 MCP 툴을 사용하도록 요청**

hermes 세션에서 다음과 같이 요청하면 에이전트가 `mcp_filesystem_read_file`, `mcp_filesystem_list_directory` 등의 툴을 자동으로 사용합니다.

```
내 프로젝트 디렉터리(/home/user/my-project)의 구조를 분석해줘
```

> 검증 필요: 실제 툴 이름과 동작은 `@modelcontextprotocol/server-filesystem` 버전에 따라 다를 수 있습니다. 로컬에서 `/reload-mcp` 후 에이전트에게 "사용 가능한 MCP 툴을 나열해줘"라고 요청해 확인하십시오.

---

## 개념: 모드 2 — Hermes를 MCP 서버로 노출 (hermes mcp serve)

`hermes mcp serve`를 실행하면 Hermes가 **stdio MCP 서버**로 기동됩니다. Claude Desktop·Cursor·VS Code·Zed 같은 MCP 클라이언트가 이 서버에 접속해 Hermes의 플랫폼 메시징 툴을 사용할 수 있습니다.

```bash
# 검증: hermes rolling, 2026-06-10
hermes mcp serve              # stdio MCP 서버 실행
hermes mcp serve --verbose    # 디버그 로깅과 함께 실행
```

> **중요:** `hermes mcp serve`는 Hermes 에이전트 전체를 MCP로 노출하는 **전체 에이전트 릴레이**가 아닙니다. 현재 구현은 **~10개 플랫폼 메시징 툴**(Telegram, Discord, Slack 등의 메시지 관리 도구)을 노출하며, Hermes의 세션 스토어에서 대화 데이터를 읽습니다.

> 검증 필요: `hermes mcp serve`가 노출하는 정확한 툴 이름은
> `hermes mcp serve --verbose`로 실행하고 MCP 클라이언트를 연결해 로컬에서 확인하십시오.

### Claude Desktop 연동 (HIGH confidence)

Claude Desktop의 MCP 서버 설정 파일(`~/.claude/claude_desktop_config.json`)에 다음을 추가합니다.

```json
// ~/.claude/claude_desktop_config.json
// 검증: hermes rolling, 2026-06-10
{
  "mcpServers": {
    "hermes": {
      "command": "hermes",
      "args": ["mcp", "serve"]
    }
  }
}
```

설정 후 Claude Desktop을 재시작하면 Hermes의 메시징 툴이 Claude Desktop에 표시됩니다.

### Cursor / VS Code / Zed 연동

핵심 명령은 동일하게 `hermes mcp serve`입니다. 그러나 각 에디터의 MCP 클라이언트 설정 파일 경로와 UI는 미검증입니다.

```bash
# [로컬 실행 후 캡처 필요 — Cursor/VS Code/Zed MCP 설정 경로]
# hermes mcp serve 를 stdio 서버 명령으로 등록하는 방식은 동일
```

> 검증 필요: Cursor/VS Code/Zed의 정확한 MCP 클라이언트 설정 파일 경로와 UI는
> 로컬에서 확인하십시오. 핵심 명령은 동일하게 `hermes mcp serve` 입니다.

---

## 심화: 전체 config 레퍼런스와 include 우선순위

### 전체 mcp_servers 설정 키

```yaml
# 검증: hermes rolling, 2026-06-10
mcp_servers:
  my-server:
    # 연결 (stdio 또는 HTTP 중 하나)
    command: "..."            # stdio 전용
    args: [...]               # stdio 전용
    env: {}                   # stdio 전용
    url: "https://..."        # HTTP 전용
    headers: {}               # HTTP 전용

    # 보안
    ssl_verify: true
    client_cert: "/path/cert.pem"
    client_key: "/path/key.pem"
    auth: oauth               # OAuth 2.1 PKCE

    # 런타임 제어
    enabled: true             # false면 이 서버 건너뜀
    timeout: 30               # 툴 실행 시간 제한 (초)
    connect_timeout: 10       # 초기 연결 시간 제한 (초)

    # 툴 필터링
    supports_parallel_tool_calls: false
    tools:
      include: [tool1, tool2] # 화이트리스트 (설정 시 exclude 무시)
      exclude: [tool3]        # 블랙리스트
      resources: false        # 리소스 유틸리티 툴 비활성화
      prompts: false          # 프롬프트 유틸리티 툴 비활성화
```

### include 우선순위 규칙

`tools.include`와 `tools.exclude`를 **동시에** 설정하면 **`include`가 우선**합니다. `exclude` 목록은 무시됩니다.

실무적 권장: `include`만 단독 사용하는 **화이트리스트 방식**이 보안상 권장됩니다. MCP 서버가 새 툴을 추가하더라도 화이트리스트 밖의 툴은 에이전트에 노출되지 않습니다.

---

## 흔한 오류 / 주의

### 주의 1: hermes mcp serve는 전체 에이전트 릴레이가 아님

> **주의:** `hermes mcp serve`는 Hermes 에이전트 전체를 MCP로 노출하는 것이 아닙니다.
> 현재 구현은 플랫폼 메시징 툴(Telegram, Discord 등) ~10개를 stdio MCP 서버로 노출합니다.
> `[정확한 노출 툴 수는 로컬 실행으로 확인 필요]`
> Hermes를 에디터에서 완전히 실행하려면 ACP(`hermes acp`)를 사용하십시오.

### 주의 2: ACP와 MCP는 별개

> **참고:** ACP(Agent Client Protocol)와 MCP(Model Context Protocol)는 별도 통합입니다.
>
> - **ACP:** `hermes acp` — Hermes를 VS Code/Zed/JetBrains의 에디터 네이티브 에이전트로 실행
> - **MCP:** `hermes mcp serve` / `mcp_servers` config — 외부 툴 서버를 Hermes에 연결하거나 Hermes를 MCP 서버로 노출
>
> 혼동하지 마십시오. 두 기능의 설치 방법과 동작 방식이 다릅니다.

### 주의 3: include 우선순위

> **참고:** `mcp_servers` 설정에서 `tools.include`와 `tools.exclude`를 동시에 설정하면
> `include`가 우선합니다. `exclude`는 무시됩니다.
> 화이트리스트 방식(`include`만 사용)이 보안상 권장됩니다.

### 주의 4: MCP 보안

> **경고:** MCP 서버 연결은 에이전트의 툴 표면을 확장합니다.
> `tools.include` 화이트리스트를 즉시 설정하여 필요한 툴만 허용하십시오.
> 신뢰할 수 없는 MCP 서버는 연결하지 마십시오.
> OAuth 원격 서버의 토큰은 `~/.hermes/mcp-tokens/`에 캐시됩니다.

---

## 다음 단계

이전 챕터: [Ch.10 학습 루프](../10-learning-loop/index.md)

다음 챕터: [Ch.12 크론 스케줄러](../12-cron/index.md)
