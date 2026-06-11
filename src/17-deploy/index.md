# 프로덕션 배포

> **이 챕터를 시작하기 전에**
>
> - [Ch.14 메시징 게이트웨이](../14-gateways/index.md) — 게이트웨이 실행 방식(`hermes gateway`, `hermes gateway start`, `hermes gateway run`)을 먼저 이해하십시오.
> - [Ch.8 터미널 백엔드](../08-backends/index.md) — Docker/SSH/Modal/Daytona 백엔드 개념을 다룹니다.
>
> 이 챕터는 Hermes **게이트웨이 자체**를 서버에 항상 켜진 상태로 배포하는 방법을 다룹니다. 로컬 에이전트를 실행하면서 터미널 명령을 원격 서버에서 실행하는 것(백엔드 연결)과는 다릅니다.

---

## 개요

항상 켜진 Hermes 게이트웨이를 배포하는 두 가지 검증된 경로가 있습니다:

| 경로 | 방법 | 적합한 상황 |
|------|------|------------|
| **(A) Docker** | 공식 이미지 `nousresearch/hermes-agent` + `gateway run` | 서버에 Docker가 있을 때 (권장) |
| **(B) 네이티브 + systemd/launchd** | `hermes gateway install` 서비스 등록 | Docker 없는 VPS, macOS 서버 |

추가적으로, Hermes를 로컬에서 실행하면서 터미널 명령을 원격에서 실행하는 세 가지 백엔드 시나리오도 다룹니다:

- **(C) SSH 백엔드** — 원격 서버 터미널 연결
- **(D) Modal 백엔드** — 에페머럴 클라우드 컨테이너
- **(E) Daytona 백엔드** — 히버네이션 지원 개발 환경

**아래 시나리오 중 하나만 따라 해도 충분합니다.** 필요에 맞는 시나리오를 선택하십시오.

---

## 시나리오 A: Docker 배포

공식 이미지 `nousresearch/hermes-agent`를 사용합니다. **이 방법이 가장 권장됩니다.**

### 기본 Docker 실행

```bash
# 검증: hermes rolling + Docker, 2026-06-11

# 1. 초기 설정 (최초 1회 — SSH로 접속하여 실행, 웹 콘솔 금지)
mkdir -p ~/.hermes
docker run -it --rm \
  -v ~/.hermes:/opt/data \
  nousresearch/hermes-agent setup

# 2. 게이트웨이 서비스로 실행
docker run -d \
  --name hermes \
  --restart unless-stopped \
  -v ~/.hermes:/opt/data \
  -p 8642:8642 \
  nousresearch/hermes-agent gateway run

# 3. 로그 확인
docker logs -f hermes

# 4. 업그레이드
docker pull nousresearch/hermes-agent:latest
docker rm -f hermes
docker run -d --name hermes --restart unless-stopped \
  -v ~/.hermes:/opt/data -p 8642:8642 \
  nousresearch/hermes-agent gateway run
```

> **참고:** `gateway run`은 Docker 컨테이너 내부에서 게이트웨이를 메인 프로세스로 실행할 때 사용합니다. 로컬 서비스로 시작할 때 쓰는 `gateway start`와 다릅니다.

### Docker Compose (대시보드 포함 — 권장)

```yaml
# docker-compose.yml
# 검증: hermes rolling, 2026-06-11
services:
  hermes:
    image: nousresearch/hermes-agent:latest
    container_name: hermes
    restart: unless-stopped
    command: gateway run
    ports:
      - "8642:8642"   # Gateway API + 헬스 엔드포인트
      - "9119:9119"   # 대시보드 (HERMES_DASHBOARD=1 필요)
    volumes:
      - ~/.hermes:/opt/data
    environment:
      - HERMES_DASHBOARD=1
    deploy:
      resources:
        limits:
          memory: 4G
          cpus: "2.0"
```

```bash
# 검증: hermes rolling, 2026-06-11
docker compose up -d
docker compose logs -f
```

### Docker 기술 사항

| 항목 | 내용 |
|------|------|
| Base image | `debian:13.4` |
| 프로세스 수퍼바이저 | s6-overlay v3 (PID 1) — 게이트웨이 크래시 시 컨테이너 종료 없이 자동 재시작 |
| 실행 사용자 | UID 10000 (`hermes` 사용자) — 비루트 실행 |
| 포트 8642 | Gateway API 서버 (OpenAI 호환) + `/health`, `/health/detailed` 엔드포인트 |
| 포트 9119 | 대시보드 (`HERMES_DASHBOARD=1` 환경변수 필요) |

#### 리소스 권장 사항

| 리소스 | 최소 | 권장 |
|--------|------|------|
| RAM | 1 GB | 2–4 GB |
| CPU | 1 코어 | 2 코어 |
| 디스크 | 500 MB | 2+ GB |
| 브라우저 자동화 추가 시 | 2 GB | 4 GB (`--shm-size=1g` 추가) |

---

## 시나리오 B: VPS 네이티브 설치 + systemd

Docker 없이 VPS에 직접 설치하는 방법입니다.

### 권장 VPS 사양

| 리소스 | 최소 | 권장 |
|--------|------|------|
| vCPU | 1 | 2 |
| RAM | 1 GB | 2 GB (브라우저 자동화 시 4 GB) |
| 디스크 | 5 GB | 20 GB |
| OS | Ubuntu 22.04+ | Ubuntu 24.04 LTS |

월 약 $4–5의 Hetzner CX22 (2 vCPU, 4 GB RAM) 또는 Hostinger KVM1이 커뮤니티 운영 사례에서 널리 사용됩니다.

### 설치 + 서비스 등록

```bash
# 검증: hermes rolling, 2026-06-11

# 1. 설치 (SSH로 접속하여 실행, 웹 콘솔 금지)
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
source ~/.bashrc

# 2. 설정
hermes setup --portal   # 또는 hermes model

# 3-A. 사용자 레벨 서비스 등록 (권장)
hermes gateway install          # 사용자 systemd/launchd 서비스 파일 생성
hermes gateway start            # 서비스 시작 (백그라운드)
hermes gateway status           # 상태 확인

# 3-B. 시스템 레벨 서비스 (재부팅 후 자동 시작)
sudo hermes gateway install --system   # 시스템 레벨 서비스 설치
```

> **검증 필요:** `sudo hermes gateway install --system`이 생성하는 정확한 서비스 파일
> 경로(`/etc/systemd/system/...`)는 환경/버전에 따라 다를 수 있습니다. Ubuntu VPS에서
> 직접 실행해 확인하십시오.

**서비스 파일 위치 (참고):**
- **macOS launchd:** `~/Library/LaunchAgents/ai.hermes.gateway.plist`
- **Linux user systemd:** `~/.config/systemd/user/hermes-gateway.service`

### 수동 systemd 유닛 (커뮤니티 패턴)

`hermes gateway install`을 사용할 수 없는 경우 수동으로 systemd 유닛 파일을 작성할 수 있습니다.

```ini
# /etc/systemd/system/hermes.service
# 검증: 커뮤니티 검증 패턴, 로컬 확인 권장, 2026-06-11
[Unit]
Description=Hermes Agent Gateway
Documentation=https://hermes-agent.nousresearch.com/docs
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=<your-user>
Group=<your-user>
WorkingDirectory=/home/<your-user>
Environment="HOME=/home/<your-user>"
Environment="PATH=/home/<your-user>/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
ExecStart=/home/<your-user>/.local/bin/hermes gateway
Restart=on-failure
RestartSec=10
MemoryMax=2G
NoNewPrivileges=true
ProtectSystem=strict
ProtectHome=read-only
ReadWritePaths=/home/<your-user>/.hermes

[Install]
WantedBy=multi-user.target
```

```bash
# 서비스 활성화
# 검증: 커뮤니티 검증 패턴, 로컬 확인 권장, 2026-06-11
sudo systemctl daemon-reload
sudo systemctl enable --now hermes
sudo systemctl status hermes
journalctl -u hermes -f   # 로그 실시간 모니터링
```

> **주의 (macOS):** `hermes gateway install`로 launchd 서비스를 설치한 후 Homebrew나 nvm으로
> 새 도구를 설치하면, launchd의 PATH에 새 도구가 반영되지 않습니다.
> 새 도구 설치 후 `hermes gateway install`을 재실행하여 PATH를 갱신하십시오.

---

## 시나리오 C: SSH 백엔드

SSH 백엔드는 **Hermes가 로컬에서 실행되면서 터미널 명령을 원격 서버에서 실행**하는 방식입니다. Hermes 자체를 VPS에 배포하는 시나리오 A/B와 다릅니다.

```yaml
# 검증: hermes rolling, 2026-06-11
terminal:
  backend: ssh
  persistent_shell: true
```

```bash
# ~/.hermes/.env 에 추가
# 검증: hermes rolling, 2026-06-11
TERMINAL_SSH_HOST=your-server.example.com
TERMINAL_SSH_USER=ubuntu
# TERMINAL_SSH_KEY=~/.ssh/id_ed25519   # 기본: ~/.ssh/id_rsa
```

SSH ControlMaster 멀티플렉싱을 통해 각 명령마다 새 SSH 연결 없이 세션을 유지합니다.

백엔드 상세 설정은 [Ch.8 터미널 백엔드](../08-backends/index.md)를 참조하십시오.

---

## 시나리오 D: Modal 백엔드

> **MEDIUM confidence** — 핵심 YAML 키는 확인되었으나, 전체 설정 흐름은 로컬 검증 권장.

Modal 백엔드는 에페머럴 클라우드 컨테이너에서 터미널 명령을 실행합니다. `container_persistent: true`로 파일시스템 스냅샷/복원이 가능합니다.

```yaml
# 검증: hermes rolling, 2026-06-11
terminal:
  backend: modal
  container_cpu: 1
  container_memory: 5120      # MB
  container_disk: 51200       # MB
  container_persistent: true
  modal_image: "nikolaik/python-nodejs:python3.11-nodejs20"   # MEDIUM confidence
```

**인증:**

```bash
# 검증: hermes rolling, 2026-06-11
pip install modal
modal setup   # 브라우저 OAuth 인증 → ~/.modal.toml 생성
```

또는 환경변수 방식:

```bash
# ~/.hermes/.env 에 추가
# 검증: hermes rolling, 2026-06-11
MODAL_TOKEN_ID=<YOUR_MODAL_TOKEN_ID>
MODAL_TOKEN_SECRET=<YOUR_MODAL_TOKEN_SECRET>
```

> **검증 필요:** `modal_image` 키 이름과 Modal managed mode의 정확한 설정 흐름을
> 로컬 Modal 계정으로 확인하십시오.

**특성:** 에페머럴 — 유휴 비용 없음. cold start 있음. `container_persistent: true`로 파일시스템 스냅샷/복원.

---

## 시나리오 E: Daytona 백엔드

> **MEDIUM confidence** — 핵심 YAML 키는 확인되었으나, 실제 동작은 로컬 검증 권장.

```yaml
# 검증: hermes rolling, 2026-06-11
terminal:
  backend: daytona
  container_cpu: 1
  container_memory: 5120      # MB
  container_disk: 10240       # MB (최대 10 GB)
  container_persistent: true
```

```bash
# ~/.hermes/.env 에 추가
# 검증: hermes rolling, 2026-06-11
DAYTONA_API_KEY=<YOUR_DAYTONA_API_KEY>
```

Daytona SDK가 서버 URL을 자동 처리합니다. 유휴 시 히버네이션으로 비용이 절감됩니다.

> **검증 필요:** Daytona 백엔드 실제 동작 (히버네이션 복구 시간, 정확한 SDK 버전)을 로컬 확인하십시오.

---

## (선택) API 서버

게이트웨이가 실행 중일 때 OpenAI 호환 API 서버를 활성화할 수 있습니다. Open WebUI, LobeChat, LibreChat 등 OpenAI 호환 프론트엔드를 연결할 수 있습니다.

```bash
# ~/.hermes/.env 에 추가
# 검증: hermes rolling, 2026-06-11
API_SERVER_ENABLED=true
API_SERVER_KEY=<openssl rand -hex 32 로 생성한 무작위 키>
# API_SERVER_HOST=0.0.0.0   # 외부 접근 허용 시 (주의: 방화벽 설정 필요)
```

**헬스 엔드포인트:**

```
GET http://localhost:8642/health
GET http://localhost:8642/health/detailed
```

---

## 흔한 오류 / 주의

> **경고: VPS 웹 콘솔에서 초기 설정 금지**
>
> VPS 웹 콘솔(브라우저 기반 터미널)에서 `hermes setup` 또는 `docker run ... setup`을 실행하지 마십시오.
> `:`, `@`, `=` 같은 특수 문자가 웹 콘솔에서 잘못 전송되어 API 키와 봇 토큰이 손상될 수 있습니다.
> **항상 SSH로 접속하여 설정하십시오.**

---

> **경고: 두 컨테이너/프로세스가 같은 `~/.hermes` 디렉터리를 동시에 사용하지 마십시오**
>
> 두 개의 Hermes 프로세스(또는 컨테이너)가 동일한 `~/.hermes` 데이터 디렉터리를 동시에 사용하면
> 세션 파일과 메모리 저장소가 손상될 수 있습니다.
> 멀티 프로파일이 필요하면 단일 컨테이너의 `hermes profile create`를 사용하십시오.

---

> **주의: Docker root 소유 파일**
>
> 공식 Docker 이미지(`nousresearch/hermes-agent`)는 UID 10000으로 실행하여 root 소유 파일 문제를 방지합니다.
> 다른 Docker 이미지나 `docker_run_as_host_user` 설정 시 파일 소유권을 확인하십시오.

---

## 다음 단계

이전: [Ch.15 추가 게이트웨이](../15-more-gateways/index.md)

다음: [Ch.18 보안 하드닝](../18-security/index.md) — 게이트웨이를 프로덕션에 배포했다면 승인 모드, API 키 관리, 공급망 보안 검사로 이어집니다.
