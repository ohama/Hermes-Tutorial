# 터미널 백엔드

> **이 챕터를 시작하기 전에**
> - [Ch.7 툴 게이트웨이](../07-tools/index.md)를 완료했는지 확인하십시오.
> - 특히 승인 모드(`approvals.mode`)를 이해하고 있어야 합니다. 컨테이너 백엔드는 보안 경계 역할을 하기 때문에 승인 모드 설정에 직접 영향을 줍니다.
> - 이 챕터는 **개요** 수준입니다. 실제 VPS·서버리스 배포, 항상 켜진 에이전트 구성은 이후 배포 단계(Phase 5)에서 자세히 다룹니다.

## 개요

터미널 백엔드는 에이전트의 `terminal`·`file` 툴이 **실제로 실행되는 환경**을 결정합니다. 기본값인 `local` 백엔드는 호스트 시스템에서 직접 명령을 실행하지만, `docker` 백엔드로 전환하면 모든 명령이 격리된 컨테이너 안에서 실행됩니다.

이 챕터에서는:
1. 6개 백엔드의 용도와 특성을 이해하고
2. Docker 백엔드로 전환해 로컬 샌드박싱을 직접 실습합니다.

원격·클라우드 백엔드(SSH/Modal/Daytona/Singularity)는 이 챕터에서 개요 수준으로 소개합니다. 터미널 백엔드를 활용한 실제 배포(VPS/서버리스 상세, 항상 켜진 에이전트)는 이후 배포 단계에서 자세히 다룹니다.

---

## 개념: 6개 백엔드

Hermes는 6개의 터미널 백엔드를 제공합니다. 각 백엔드는 격리 수준·파일 지속성·환경 요구사항이 다릅니다.

| 백엔드 | 격리 수준 | 파일 지속성 | 주요 용도 | 환경 요구사항 |
|--------|-----------|-------------|-----------|---------------|
| `local` | 없음 (호스트 직접) | 호스트 파일시스템 | 개발·테스트 기본값 | 없음 |
| `docker` | 강함 (cap-drop ALL, pids-limit 256) | 컨테이너 `/opt/data` | 프로덕션, 보안 중요 환경 | Docker 데몬 실행 중 |
| `ssh` | 네트워크 경계 | 원격 서버 파일시스템 | 항상 켜진 VPS, 원격 실행 | SSH 접근 가능한 서버 |
| `singularity` | 네임스페이스 격리 | 오버레이 파일시스템 | HPC 클러스터 | `apptainer` 또는 `singularity` in `$PATH` |
| `modal` | 클라우드 샌드박스 | 에페머럴 (세션 간 지속 없음) | 서버리스, 비용 최적화 | `MODAL_TOKEN_ID` 또는 `~/.modal.toml` |
| `daytona` | 관리형 워크스페이스 | 영구 (최대 10GB 디스크) | 클라우드 개발 환경 | `DAYTONA_API_KEY` |

**Modal 특이 사항:** 에페머럴 방식으로 동작합니다. 세션 간 파일이 자동으로 유지되지 않으며, 서버리스 특성상 유휴 비용이 없는 대신 cold start가 발생합니다.

**Daytona 특이 사항:** 유휴 시 히버네이션으로 전환되어 세션 간 상태를 유지하면서 비용을 최소화합니다.

---

## 개념: 백엔드 선택·전환

백엔드를 전환하는 방법은 세 가지입니다.

### 방법 1: config.yaml (영구적)

```yaml
# 검증: hermes rolling, 2026-06-09
terminal:
  backend: docker    # local | docker | ssh | singularity | modal | daytona
```

`~/.hermes/config.yaml`에서 직접 편집합니다.

### 방법 2: hermes config set (CLI에서 즉시)

```bash
# 검증: hermes rolling, 2026-06-09
hermes config set terminal.backend docker
hermes config set terminal.backend local   # 되돌리기
```

명령 실행 즉시 `~/.hermes/config.yaml`에 저장됩니다.

### 방법 3: hermes setup terminal (인터랙티브 마법사)

```bash
# 검증: hermes rolling, 2026-06-09
hermes setup terminal
```

> **검증 필요:** `hermes setup terminal` 마법사의 정확한 상호작용 흐름(표시되는 질문, 선택지)은 백엔드별로 다르며 로컬 실행 후 확인이 필요합니다 (Open Question #3).

### Docker 데몬 상태 확인

Docker 백엔드 사용 전 데몬이 실행 중인지 확인하십시오:

```bash
# 검증: 로컬 Docker 설치, 2026-06-09
docker info
```

---

## 개념: Docker 백엔드 보안 하드닝

Docker 백엔드는 다음 보안 플래그를 **자동으로** 적용합니다. 독자가 직접 설정할 필요 없음 — Hermes가 컨테이너 시작 시 모두 자동 적용합니다:

```
--cap-drop ALL
--cap-add DAC_OVERRIDE
--cap-add CHOWN
--security-opt no-new-privileges
--pids-limit 256
--tmpfs /tmp:rw,nosuid,size=512m
--tmpfs /var/tmp:rw,noexec,nosuid
```

이 하드닝 덕분에 `approvals.mode: off` 설정도 Docker 백엔드에서는 합리적인 선택이 될 수 있습니다(컨테이너가 보안 경계를 제공하기 때문). 하지만 `local` 백엔드에서는 절대 `off`로 설정하지 마십시오.

### Docker 백엔드 전체 config.yaml 설정

```yaml
# 검증: hermes rolling, 2026-06-09
terminal:
  backend: docker
  docker_image: "nikolaik/python-nodejs:python3.11-nodejs20"  # 기본값
  docker_mount_cwd_to_workspace: false
  docker_run_as_host_user: false    # true: 컨테이너가 호스트 UID로 실행 (파일 소유권 보존)
  docker_forward_env:               # 컨테이너에 전달할 호스트 환경변수
    - "GITHUB_TOKEN"
  docker_env:                       # 컨테이너 전용 환경변수
    DEBUG: "1"
  docker_volumes:                   # 추가 볼륨 마운트
    - "/host/path:/container/path"
  docker_extra_args:                # 추가 docker run 플래그
    - "--gpus=all"
  container_cpu: 1                  # CPU 코어
  container_memory: 5120            # MB (기본 5GB)
  container_disk: 51200             # MB (기본 50GB)
  container_persistent: true        # 세션 간 컨테이너 유지
  docker_persist_across_processes: true
  docker_orphan_reaper: true
```

---

## 실습: Docker 백엔드로 전환하기

아래 순서를 따라 Docker 백엔드로 전환하고 컨테이너 내 실행을 확인해보십시오.

```bash
# 검증: hermes rolling + Docker 설치, 2026-06-09
# 1. Docker 백엔드로 전환
hermes config set terminal.backend docker

# 2. hermes 실행 — 첫 툴 실행 시 컨테이너 자동 시작
hermes

# 3. 에이전트에게 파이썬 버전 확인 요청 (컨테이너 내 실행됨)
# 채팅에서: "파이썬 버전을 확인해줘"
# 예상 동작: 에이전트가 terminal 툴로 docker exec 실행
#            출력: Python 3.11.x [GCC ... ] on linux

# 4. 로컬로 되돌리기
hermes config set terminal.backend local
```

**첫 실행 시 컨테이너 초기화 출력 (LOW confidence):**

```
# [첫 Docker 백엔드 실행 시 컨테이너 빌드/풀 메시지 — 로컬 실행 후 캡처 필요]
```

> **검증 필요:** 첫 Docker 백엔드 실행 시 이미지 풀·컨테이너 시작 메시지의 정확한 형식은 로컬 실행 후 캡처가 필요합니다 (Open Question #4).

---

## 개념: 원격·클라우드 백엔드 (개요)

> **검증 필요:** Modal/Daytona/Singularity의 정확한 YAML 키와 설정 흐름은 MEDIUM 신뢰도이며 게재 전 로컬 검증이 필요합니다 (Open Question #8). 아래 설정 예시는 공식 문서와 DeepWiki 분석을 교차 검증한 결과이나, rolling release에 의해 키 이름이 변경될 수 있습니다.

### SSH 백엔드

원격 서버에서 직접 명령을 실행합니다. 항상 켜진 VPS 환경에 적합합니다.

```bash
# 검증: hermes rolling, 2026-06-09
# SSH 백엔드 설정 (환경변수 방식)
# ~/.hermes/.env에 추가:
TERMINAL_SSH_HOST=your-server.example.com
TERMINAL_SSH_USER=ubuntu
# 선택사항:
# TERMINAL_SSH_PORT=22
# TERMINAL_SSH_KEY=~/.ssh/id_ed25519
```

```yaml
# 검증: hermes rolling, 2026-06-09
terminal:
  backend: ssh
  persistent_shell: true   # SSH ControlMaster 멀티플렉싱 (기본 true)
```

SSH 백엔드는 SSH ControlMaster를 통한 `tar` 파이프로 `~/.hermes` 디렉터리를 원격 호스트와 동기화하며, 세션 종료 시 수정된 파일을 `~/.hermes/cache/remote-syncs/`에 저장합니다.

### Modal 백엔드

서버리스 클라우드 샌드박스입니다. 유휴 비용 없음(cold start 있음).

```yaml
# 검증: hermes rolling, 2026-06-09
terminal:
  backend: modal
  container_cpu: 1
  container_memory: 5120    # MB
  container_disk: 51200     # MB
  container_persistent: true
```

**필요 자격증명:** `MODAL_TOKEN_ID` 환경변수 또는 `~/.modal.toml` (`modal auth login`으로 생성).

> **검증 필요:** NousResearch가 Modal 리소스를 사전 공급하는 "Managed mode"의 정확한 설정 흐름은 공개 문서에 충분히 기술되지 않아 로컬 검증이 필요합니다 (Open Question #5).

### Daytona 백엔드

관리형 클라우드 개발 환경입니다. 히버네이션으로 세션 간 상태 유지, 최대 10GB 디스크.

```yaml
# 검증: hermes rolling, 2026-06-09
terminal:
  backend: daytona
  container_cpu: 1
  container_memory: 5120    # MB
  container_disk: 10240     # MB (최대 10GB)
  container_persistent: true
```

**필요 자격증명:** `DAYTONA_API_KEY` 환경변수. Daytona SDK가 서버 URL을 자동으로 처리합니다.

### Singularity/Apptainer 백엔드

HPC 클러스터 전용입니다. 루트 권한 없이 컨테이너를 실행할 수 있어 공유 슈퍼컴퓨터 환경에 적합합니다.

```yaml
# 검증: hermes rolling, 2026-06-09
terminal:
  backend: singularity
```

**필요 환경:** `apptainer` 또는 `singularity` 명령이 `$PATH`에 있어야 합니다.

**스크래치 디렉터리 해결 순서:**
1. `TERMINAL_SCRATCH_DIR` 환경변수
2. `TERMINAL_SANDBOX_DIR/singularity`
3. `/scratch/$USER/hermes-agent` (HPC 관례)
4. `~/.hermes/sandboxes/singularity`

---

## 흔한 오류 / 주의

> **주의: Docker 데몬 미실행 (A-13 관련)**
>
> Docker 백엔드를 사용하려면 Docker 데몬이 실행 중이어야 합니다.
> `docker info` 명령으로 데몬 상태를 먼저 확인하십시오.
> 데몬 미실행 시 터미널 툴 호출마다 "Cannot connect to Docker daemon" 오류가 발생합니다.

---

> **주의: docker_run_as_host_user와 파일 소유권 (A-13, 중요)**
>
> `docker_run_as_host_user: false` (기본값)로 Docker 백엔드를 사용하면
> 컨테이너 내에서 생성된 파일이 root 소유가 될 수 있습니다.
> 호스트에서 일반 사용자로 이 파일을 편집·삭제하려면 `sudo`가 필요합니다.
>
> **해결책:**
> ```yaml
> docker_run_as_host_user: true  # 컨테이너가 호스트 UID로 실행
> ```
> 단, 이 설정을 활성화하면 컨테이너 내 패키지 설치(`apt install`)가
> 권한 오류로 실패할 수 있습니다.

---

> **참고: Modal 에페머럴 특성 (LOW — 부분적으로 불확실)**
>
> Modal 백엔드는 기본적으로 에페머럴입니다.
> 세션이 종료되면 컨테이너 내 파일이 사라질 수 있습니다 (스냅샷 미설정 시).
> 세션 간 파일을 유지하려면 `container_persistent: true`를 설정하십시오.
> [Modal 스냅샷 설정의 정확한 동작은 로컬 검증 필요]

---

> **참고: approvals.mode: off는 컨테이너 백엔드에서만 합리적 (A-12)**
>
> Docker/Modal/Daytona 백엔드를 사용할 경우 컨테이너 자체가 보안 경계 역할을 합니다.
> 이 환경에서는 `approvals.mode: off`가 합리적일 수 있습니다.
> 그러나 **`local` 백엔드에서는 절대 `off`로 설정하지 마십시오.**

---

## 다음 단계

이전: [Ch.7 툴 게이트웨이](../07-tools/index.md)

이로써 Hermes의 핵심 개념 5가지(에이전트 루프·컨텍스트 파일·메모리·툴 게이트웨이·터미널 백엔드)를 모두 살펴봤습니다.

터미널 백엔드를 활용한 실제 배포(VPS/서버리스 상세, 항상 켜진 에이전트)는 이후 배포 단계에서 자세히 다룹니다.
