# 스킬 시스템

> **이 챕터를 시작하기 전에**
>
> - [Ch.6 메모리](../06-memory/index.md) — 스킬은 **절차적 메모리(Procedural Memory)**입니다. 메모리 챕터에서 Skills를 Phase 4 범주로 예고했는데, 바로 이 챕터가 그 내용을 이어받습니다.
> - [Ch.7 툴 게이트웨이](../07-tools/index.md) — 에이전트가 툴을 호출하는 루프를 이해하고 있어야 스킬이 어떻게 절차를 자동화하는지 파악할 수 있습니다.

---

## 개요

스킬 시스템은 Hermes의 **재사용 가능한 절차 라이브러리**입니다. 복잡한 다단계 작업을 한 번 성공적으로 수행하면, 그 접근 방법을 스킬로 저장해 두고 다음번에는 슬래시 커맨드 하나로 불러올 수 있습니다.

이 챕터에서는:
1. SKILL.md 형식과 저장 구조를 이해합니다.
2. 직접 SKILL.md를 작성하여 커스텀 스킬을 만듭니다.
3. CLI로 스킬을 관리하고 커뮤니티 스킬을 설치합니다.
4. 에이전트가 스킬을 자동으로 생성·제안하는 개념을 익힙니다.

---

## 개념: 스킬이란 무엇인가

스킬은 **Markdown + YAML frontmatter** 형식의 파일입니다. 워크플로우, 명령어, 분기 조건, 함정, 검증 단계를 텍스트로 담아 에이전트가 작업을 수행할 때 재사용할 수 있도록 합니다.

Hermes는 **agentskills.io 오픈 표준**을 따릅니다. 이 표준 덕분에 하나의 SKILL.md 파일이 25개+ 호환 에이전트 플랫폼 간에 이식됩니다.

### 점진적 공개(Progressive Disclosure)

스킬이 수백 개 있어도 시스템 프롬프트의 토큰 소비는 거의 없습니다. 동작 원리는 다음과 같습니다:

```
시스템 프롬프트 로드 시
  ↓
스킬 인덱스(이름 + 설명만) → Stable Tier, Layer 7에 로드
  ↓
사용자 요청이 스킬 트리거와 매칭될 때
  ↓
해당 스킬 전체 내용을 온디맨드로 컨텍스트에 로드
```

200개 스킬이 설치되어 있어도 활성화되기 전까지는 거의 토큰을 소비하지 않습니다. (스킬 인덱스가 시스템 프롬프트에 표시되는 정확한 형식은 환경에 따라 다를 수 있으므로 단정하지 않습니다.)

---

## 개념: 저장 구조

스킬 파일은 `~/.hermes/skills/` 아래에 카테고리별로 저장됩니다.

```
~/.hermes/skills/
├── <category>/
│   └── <skill-name>/
│       ├── SKILL.md              ← 핵심 파일
│       ├── references/           ← API 문서, 예시 (선택)
│       ├── templates/            ← 설정 템플릿 (선택)
│       ├── scripts/              ← 설정 스크립트 (선택)
│       └── assets/               ← 이미지 등 (선택)
```

외부 디렉터리를 추가로 지정할 수도 있습니다:

```yaml
# 검증: hermes rolling, 2026-06-10
skills:
  external_dirs:
    - "/path/to/additional/skills"
```

---

## 개념: SKILL.md 형식

SKILL.md는 YAML frontmatter로 메타데이터를 선언하고, 본문에 4개의 필수 섹션을 포함합니다.

```yaml
# 검증: hermes rolling, 2026-06-10
---
name: my-skill
description: Brief description of what this skill does
version: 1.0.0
platforms: [macos, linux]        # 선택: OS 제한
metadata:
  hermes:
    tags: [automation, web]
    category: productivity
    config:
      output_format: markdown    # 비밀이 아닌 설정 선언
required_environment_variables:   # 자격증명 프롬프트
  - MY_API_KEY
fallback_for_toolsets: []        # 조건부 활성화
requires_toolsets: [web]         # 필요 툴셋 제약
---

## When to Use

[이 스킬을 언제 사용하는지 설명]

## Procedure

1. [단계 1 — 명령어 또는 액션]
2. [단계 2]
3. [분기: 조건 A면 → 단계 3a, 조건 B면 → 단계 3b]

## Pitfalls

- [알려진 문제 1]
- [알려진 문제 2]

## Verification

[결과가 올바른지 확인하는 방법]
```

**본문 필수 섹션 4개:** When to Use / Procedure / Pitfalls / Verification.

---

## 개념: 스킬 자동 생성과 제안

에이전트가 복잡한 다단계 작업을 성공적으로 완료하면, 그 접근 방법을 **스킬로 저장할지 제안**합니다. 사용자는 대화 중에 수락하거나 거절할 수 있습니다. 수락하면 스킬이 `~/.hermes/skills/<category>/<name>/SKILL.md`에 자동으로 저장됩니다.

이 자동 생성/제안 기능은 Ch.10 학습 루프의 핵심입니다. 큐레이터(Curator)가 비활동 기반으로 스킬 품질을 주기적으로 검토·개선하는 동작은 다음 챕터에서 자세히 다룹니다.

> 검증 필요: 스킬 자동 생성/제안이 발동되는 정확한 조건(툴 호출 횟수 등)과
> 제안 메시지 문구는 환경/버전에 따라 다를 수 있습니다. 로컬 실행 후 확인하십시오.

---

## 개념: 활성화 (슬래시 커맨드)

`~/.hermes/skills/` 에 배치된 스킬은 슬래시 커맨드로 활성화됩니다:

```
/skill-name [task description]
```

플러그인 네임스페이스가 있는 경우:

```
/plugin:skill-name [task description]
```

hermes를 재시작하면 스킬 인덱스가 갱신되어 새 스킬을 사용할 수 있습니다.

---

## 실습: 스킬 목록 확인

현재 설치된 스킬을 확인합니다:

```bash
# 검증: hermes rolling, 2026-06-10
hermes skills list
```

예상 출력:

```
# [로컬 실행 후 캡처 필요 — 설치된 스킬 목록 출력]
```

> 검증 필요: `hermes skills list`의 출력 형식(컬럼 구성, 버전/사용 통계 표시 여부 등)은 버전과 환경에 따라 다릅니다. 로컬에서 실행 후 확인하십시오.

---

## 실습: 직접 SKILL.md 작성하기

다음 명령으로 Python 프로젝트 배포 스킬을 직접 만들어봅니다:

```bash
# 검증: hermes rolling, 2026-06-10
mkdir -p ~/.hermes/skills/productivity/my-deploy-skill
cat > ~/.hermes/skills/productivity/my-deploy-skill/SKILL.md << 'EOF'
---
name: my-deploy-skill
description: 로컬 Python 프로젝트를 서버에 배포하는 절차
version: 1.0.0
metadata:
  hermes:
    tags: [deploy, python]
    category: productivity
---

## When to Use

Python 프로젝트를 원격 서버에 배포할 때 사용합니다.

## Procedure

1. `pytest`로 테스트를 먼저 실행합니다.
2. `git push origin main`으로 코드를 푸시합니다.
3. SSH로 서버에 접속하여 `git pull`을 실행합니다.
4. `systemctl restart myapp`으로 서비스를 재시작합니다.

## Pitfalls

- 테스트가 실패하면 배포를 중단합니다.
- SSH 키가 서버에 등록되어 있어야 합니다.

## Verification

서비스 상태: `systemctl status myapp`으로 active(running) 확인.
EOF

# hermes 재시작 후 /my-deploy-skill 로 활성화 가능
```

스킬 파일을 작성한 후 hermes를 재시작하면 `/my-deploy-skill`로 해당 스킬을 활성화할 수 있습니다.

---

## 실습: 스킬 관리 CLI

### 기본 관리 명령

```bash
# 검증: hermes rolling, 2026-06-10
hermes skills list                    # 설치된 스킬 목록
hermes skills browse                  # 허브 스킬 탐색 (인터랙티브)
hermes skills search <query>          # 이름/태그로 검색
hermes skills inspect <identifier>    # 설치 전 미리보기
hermes skills install <identifier>    # 보안 스캔 후 설치
hermes skills check                   # 업스트림 업데이트 확인
hermes skills update                  # 업데이트된 스킬 재설치
```

세션 내 슬래시 커맨드로도 설치할 수 있습니다:

```
/skills install official/creative/songwriting-and-ai-music
```

### 커뮤니티 스킬 설치

다양한 커뮤니티 스킬 허브(공식 선택 카탈로그, skills.sh, ClawHub, LobeHub 등)에서 스킬을 설치할 수 있습니다:

```bash
# 검증: hermes rolling, 2026-06-10
# 방법 1: 공식 선택 스킬 설치
hermes skills install official/research/arxiv

# 방법 2: URL로 직접 설치
hermes skills install https://sharethis.chat/SKILL.md

# 방법 3: URL에 이름 지정
hermes skills install https://example.com/SKILL.md --name my-skill

# 제거
hermes skills uninstall <skill-name>
```

---

## 심화: 번들 스킬과 skill_manage 툴

### 번들 스킬

Hermes에는 설치 즉시 사용 가능한 번들 스킬이 포함되어 있습니다:

- **번들 스킬:** 약 89개(v0.15.x, 17개 카테고리)
- **선택 카탈로그 스킬:** 약 90개+(별도 카탈로그, 18개 카테고리)

주요 카테고리: GitHub, Research(arXiv), DevOps(Kanban), Software Development(디버깅, TDD), Productivity(Google Workspace, Notion), Note-Taking(Obsidian).

> 참고: 번들 스킬 수는 버전마다 달라집니다.
> v0.10.0에는 118개, v0.15.x 현재에는 약 89개의 번들 스킬이 있습니다.
> `hermes skills list`로 현재 설치된 스킬 수를 항상 확인하십시오.

### skill_manage 툴 (에이전트 내부 사용)

에이전트는 스킬을 생성하거나 수정할 때 내부적으로 `skill_manage` 툴을 사용합니다:

| 액션 | 용도 |
|------|------|
| `create` | 새 스킬 파일 생성 |
| `patch` | 타겟 변경 (토큰 절약, 선호 방식) |
| `edit` | 전체 파일 재작성 |
| `delete` | 스킬 삭제 |
| `write_file` | 지원 파일 작성 |
| `remove_file` | 지원 파일 삭제 |

독자는 `skill_manage`를 직접 호출할 필요가 없습니다. 에이전트에게 자연어로 "이 스킬 개선해줘"라고 요청하면 됩니다. `patch` 액션이 전체 재작성보다 토큰을 절약하므로 에이전트가 선호합니다.

---

## 흔한 오류 / 주의

> 참고: `~/.hermes/skills/`에 스킬 파일을 추가하거나 수정하면
> 현재 실행 중인 hermes 세션에는 즉시 반영되지 않을 수 있습니다.
> hermes를 재시작하면 스킬 인덱스가 갱신됩니다.
> [로컬에서 정확한 동작 확인 필요]

---

> 참고: 번들 스킬 수는 버전마다 달라집니다.
> v0.10.0에는 118개, v0.15.x 현재에는 약 89개의 번들 스킬이 있습니다.
> `hermes skills list`로 현재 설치된 스킬 수를 항상 확인하십시오.

---

> 참고: `hermes skills install`은 설치 전 자동으로 보안 스캔을 수행합니다.
> 스캔 결과에 경고가 있어도 `--force` 플래그로 강제 설치할 수 있지만,
> 위험하지 않은 항목에 대해서만 사용하십시오.

---

## 다음 단계

이제 스킬 시스템의 기초를 이해했습니다. 스킬은 에이전트 학습의 단위이기도 합니다 — 작업을 수행할 때마다 에이전트가 접근 방법을 스킬로 저장·개선하는 Do→Learn→Improve 사이클로 이어집니다.

다음 챕터: [Ch.10 학습 루프](../10-learning-loop/index.md)

이전 챕터: [Ch.8 터미널 백엔드](../08-backends/index.md)
