---
phase: 05-platforms-operations
plan: 03
subsystem: content
tags: [hermes, docker, systemd, ssh, modal, daytona, deployment, gateway, vps, s6-overlay]

# Dependency graph
requires:
  - phase: 05-01
    provides: SUMMARY.md 스켈레톤 + src/17-deploy/index.md placeholder
provides:
  - src/17-deploy/index.md 본문 (353줄): Docker 배포 + VPS systemd + SSH/Modal/Daytona 백엔드 + API 서버 + 함정 3개 + 챕터 내비게이션
affects:
  - 05-04 (Ch.18 보안 하드닝 — 이전 챕터 링크가 Ch.17 배포를 가리킴)

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "# 검증: hermes rolling, 2026-06-11 — 모든 코드블록 첫 줄 필수 주석"
    - "> 검증 필요 callout — MEDIUM/DEFERRED 항목 (--system 경로, modal_image, Modal managed mode)"
    - "placeholder 값 패턴 — <YOUR_DAYTONA_API_KEY> 등 실제처럼 보이는 값 금지"

key-files:
  created:
    - src/17-deploy/index.md
  modified: []

key-decisions:
  - "Task 1+2를 단일 파일 작성으로 처리 — 동일 파일 순차 편집이므로 완전한 파일 한 번에 작성 + atomic commit (02-02 패턴 재적용)"
  - "hermes gateway run vs hermes gateway start 구분 명시 — run은 Docker 내 실행; start는 install 이후 서비스 시작"
  - "sudo hermes gateway install --system 경로 MEDIUM → 검증 필요 callout으로 처리 (Ubuntu VPS 직접 확인 안내)"
  - "Modal managed mode 설정 흐름 DEFERRED → 검증 필요 callout (modal_image 키도 MEDIUM 헤지)"
  - "API 서버 섹션: API_SERVER_KEY 생성 명령을 환경변수 값으로 직접 기술 (openssl rand -hex 32 설명)"

patterns-established:
  - "커뮤니티 검증 패턴 코드블록: # 검증: 커뮤니티 검증 패턴, 로컬 확인 권장, 2026-06-11 (수동 systemd 유닛 파일)"
  - "시나리오 단독 완결성: 독자가 A/B/C/D/E 중 하나만 읽어도 배포 가능한 구조"

# Metrics
duration: 2min
completed: 2026-06-11
---

# Phase 5 Plan 03: 프로덕션 배포 Summary

**Ch.17 프로덕션 배포 본문 353줄 — Docker(nousresearch/hermes-agent + s6-overlay + UID 10000) / VPS systemd(gateway install + --system 헤지) / SSH·Modal·Daytona 백엔드 / API 서버 / 함정 3개 / 챕터 내비게이션**

## Performance

- **Duration:** ~2 min
- **Started:** 2026-06-11T00:42:47Z
- **Completed:** 2026-06-11T00:45:08Z
- **Tasks:** 2 (1 atomic commit — 단일 파일 전체 작성)
- **Files modified:** 1

## Accomplishments

- Docker 배포 시나리오: `nousresearch/hermes-agent` 공식 이미지, `gateway run`, `--restart unless-stopped`, 포트 8642/9119, UID 10000, s6-overlay v3, Docker Compose + 리소스 제한
- VPS systemd 시나리오: `hermes gateway install`(사용자 레벨) + `--system`(시스템 레벨, 경로 헤지), 수동 커뮤니티 유닛 파일, macOS launchd PATH 주의 callout, journalctl 로그 모니터링
- SSH/Modal/Daytona 백엔드 시나리오: 핵심 YAML 키 + 인증 방법 + MEDIUM 헤지 callout
- API 서버 선택 섹션: `API_SERVER_ENABLED`, `/health` 엔드포인트, OpenAI 호환 프론트엔드 연결 안내
- 함정 3개: 웹 콘솔 초기 설정 금지 / 공유 `~/.hermes` 금지 / `docker_run_as_host_user` root 소유 파일
- 챕터 내비게이션: `15-more-gateways/index.md` ← 이전 / `18-security/index.md` → 다음

## Task Commits

1. **Task 1+2: Ch.17 전체 본문 (Docker + VPS systemd + SSH/Modal/Daytona + API 서버 + 함정 + 내비게이션)** - `f698366` (feat)

**Plan metadata:** (docs commit below)

## Files Created/Modified

- `src/17-deploy/index.md` — Ch.17 프로덕션 배포 본문 (353줄, placeholder 대체)

## Decisions Made

- Task 1+2를 단일 파일 작성으로 처리 — 동일 파일 순차 편집이므로 완전한 파일 한 번에 작성 + atomic commit (02-02, 03-03 패턴 재적용)
- `hermes gateway run` vs `hermes gateway start` 구분: `gateway run`은 Docker 내 메인 프로세스 실행, `gateway start`는 install 이후 백그라운드 서비스 시작 (05-01 결정 재확인)
- `sudo hermes gateway install --system` 경로 MEDIUM → `> 검증 필요` callout으로 처리 (Ubuntu VPS 직접 확인 안내)
- Modal managed mode 설정 흐름 DEFERRED, `modal_image` 키 MEDIUM → `> 검증 필요` callout
- API 서버 코드블록에서 `API_SERVER_KEY` 생성 명령을 설명 텍스트로 표기 (`<openssl rand -hex 32 로 생성한 무작위 키>`)

## Deviations from Plan

None — plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None — no external service configuration required.

## Next Phase Readiness

- Ch.17 배포 챕터 완성 — `src/17-deploy/index.md` 353줄, mdbook build exit 0 확인
- Ch.18 보안 하드닝(05-04) 실행 가능 — 이전 챕터 링크(`../15-more-gateways/index.md`) 및 다음 챕터 링크(`../18-security/index.md`) 정확히 설정됨
- 05-02(Ch.15 추가 게이트웨이)와 병렬 실행 완료 기준 충족

---
*Phase: 05-platforms-operations*
*Completed: 2026-06-11*
