# Roadmap: Hermes Agent 한국어 튜토리얼

## Overview

이 로드맵은 Nous Research Hermes Agent 한국어 튜토리얼을 mdBook으로 제작해 GitHub Pages에 자동 배포하는 전 과정을 다룬다. 인프라 스캐폴드와 CI/CD 파이프라인을 먼저 구축해 모든 콘텐츠 페이즈가 실제 동작하는 사이트에 쌓이도록 하고, 이후 설치·기초부터 핵심 개념·학습 루프·플랫폼 운영까지 의존성 순서대로 19개 챕터를 완성한 뒤 레퍼런스 부록으로 마무리한다.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [x] **Phase 1: Scaffold & CI** - mdBook 프로젝트 구조와 GitHub Pages 자동 배포 파이프라인 구축
- [x] **Phase 2: Foundations** - 소개·설치·첫 실행·모델 설정 챕터 (Ch.0–3)
- [x] **Phase 3: Core Concepts** - 에이전트 루프·컨텍스트·메모리·툴 게이트웨이 챕터 (Ch.4–8)
- [x] **Phase 4: Learning & Automation** - 스킬·학습 루프·MCP·크론·서브에이전트 챕터 (Ch.9–13)
- [ ] **Phase 5: Platforms & Operations** - 메시징 게이트웨이·배포·보안 챕터 (Ch.14–18)
- [ ] **Phase 6: Reference** - CLI·슬래시 커맨드·설정·트러블슈팅 레퍼런스 부록

## Phase Details

### Phase 1: Scaffold & CI
**Goal**: 빌드 가능한 mdBook 레포와 GitHub Pages 자동 배포 파이프라인이 존재해 모든 콘텐츠 챕터가 실제 퍼블릭 URL에서 렌더링된다
**Depends on**: Nothing (first phase)
**Requirements**: INFRA-01, INFRA-02, INFRA-03, INFRA-04
**Success Criteria** (what must be TRUE):
  1. `mdbook build`가 로컬에서 오류 없이 실행되고 `book/` 디렉터리가 생성된다
  2. `git push`를 하면 GitHub Actions 워크플로우가 트리거되어 GitHub Pages에 자동 배포된다
  3. 배포된 사이트가 `https://<owner>.github.io/<repo>/`에서 한국어 SUMMARY.md 기반 목차와 함께 정상 접근된다
  4. 모든 코드블록에 버전·검토일 주석 관례(`# 검증: hermes vX.Y, YYYY-MM-DD`)가 적용된 예시 챕터가 존재한다
**Plans**: 3 plans

Plans:
- [x] 01-01-PLAN.md — mdBook 0.5.3 스캐폴드: book.toml(`language="ko"`, site-url), 한국어 SUMMARY.md, `# 검증:` 주석 예시 챕터, 로컬 빌드 검증
- [x] 01-02-PLAN.md — GitHub Actions 워크플로우: pre-built mdBook 0.5.3 바이너리, configure/upload/deploy-pages 흐름, Pages 권한
- [x] 01-03-PLAN.md — 최종 검증 + 배포: 로컬 빌드 재확인(자동) + GitHub 리모트·Pages 소스·푸시·라이브 사이트 확인(수동 체크포인트) → https://ohama.github.io/Hermes-Tutorial/

---

### Phase 2: Foundations
**Goal**: 독자가 튜토리얼을 따라 Hermes를 설치·실행하고 모델을 연결할 수 있다 (Ch.0–3)
**Depends on**: Phase 1
**Requirements**: FOUND-01, FOUND-02, FOUND-03, FOUND-04
**Success Criteria** (what must be TRUE):
  1. Ch.0 소개 챕터가 렌더링되고 Hermes의 핵심 철학(학습 루프, 자가 개선)과 튜토리얼 사용법이 설명된다
  2. Ch.1 설치 챕터를 따라 macOS/Linux/WSL2/Windows에서 Hermes를 설치하고 `hermes doctor`로 검증할 수 있다 (PATH 재로드 단계 포함)
  3. Ch.2 첫 실행 챕터를 따라 `hermes`를 실행해 첫 대화를 완료하고 주요 CLI 명령어(`hermes --help` 등)를 확인할 수 있다
  4. Ch.3 모델 설정 챕터를 따라 `hermes model`로 제공자와 API 키를 설정하고 64K 최소 컨텍스트 요건을 충족하는 모델을 선택할 수 있다
**Plans**: 4 plans (2 waves — 02-01이 SUMMARY.md 스켈레톤 확정[Wave 1], 02-02/03/04 챕터 본문 병렬[Wave 2])

Plans:
- [x] 02-01-PLAN.md — Ch.0 소개 + SUMMARY.md 4-챕터 스켈레톤 (학습 루프·멀티모델 철학, 튜토리얼 맵)
- [x] 02-02-PLAN.md — Ch.1 설치 (OS별 curl|bash/ps1, PATH 재로드, hermes doctor, sudo/Windows/Termux 주의)
- [x] 02-03-PLAN.md — Ch.2 첫 실행 (hermes/--tui 실행, 첫 대화, CLI 명령어 일람[hermes --help], 슬래시 커맨드, 출력 placeholder)
- [x] 02-04-PLAN.md — Ch.3 모델 설정 (hermes model/setup --portal, 공급자·키 .env, 64K 요건, ko 부분 지원, 5개 주의)

---

### Phase 3: Core Concepts
**Goal**: 독자가 에이전트 루프·컨텍스트 파일·메모리·툴 게이트웨이의 작동 원리를 이해하고 직접 설정할 수 있다 (Ch.4–8)
**Depends on**: Phase 2
**Requirements**: CORE-01, CORE-02, CORE-03, CORE-04
**Success Criteria** (what must be TRUE):
  1. Ch.4 에이전트 루프 챕터를 읽고 AIAgent 코어 루프와 3티어 프롬프트 시스템의 동작 원리를 설명할 수 있다
  2. Ch.5 컨텍스트 파일 챕터를 따라 `.hermes.md`/`AGENTS.md`/`SOUL.md` 계층과 `/personality` 슬래시 커맨드를 설정할 수 있다
  3. Ch.6 메모리 챕터를 따라 `MEMORY.md`/`USER.md`, SQLite FTS5 대화 검색, 압축 설정을 이해하고 활용할 수 있다
  4. Ch.7 툴 게이트웨이 챕터를 따라 `hermes tools`로 검색/이미지/TTS/브라우저 툴을 설정하고 승인 모드를 이해할 수 있다
**Plans**: 5 plans

Plans:
- [x] 03-01: Ch.4 에이전트 루프 — AIAgent 코어, `run_agent.py` 흐름, 프롬프트 3티어 구조
- [x] 03-02: Ch.5 컨텍스트 파일 — `.hermes.md`/`AGENTS.md`/`SOUL.md` 계층, 퍼스널리티 설정, `.env` vs `config.yaml` 분리
- [x] 03-03: Ch.6 메모리 — `MEMORY.md`/`USER.md`, SQLite FTS5 세션 검색, compression 설정, 비용 관리
- [x] 03-04: Ch.7 툴 게이트웨이 — `hermes tools`, 내장 툴 목록, 승인 모드, Nous Portal 유료 기능 경고
- [x] 03-05: Ch.8 터미널 백엔드 — local/Docker/SSH/Modal/Daytona 옵션 개요 (Docker 실습 포함)

---

### Phase 4: Learning & Automation
**Goal**: 독자가 스킬 시스템과 학습 루프를 이해하고 MCP 연동·크론 스케줄러·서브에이전트를 직접 구성할 수 있다 (Ch.9–13)
**Depends on**: Phase 3
**Requirements**: LEARN-01, LEARN-02, LEARN-03, LEARN-04
**Success Criteria** (what must be TRUE):
  1. Ch.9 스킬 챕터를 따라 `SKILL.md` 형식으로 직접 스킬을 작성하고 스킬 자동 생성/제안 트리거 개념을 이해할 수 있다 (정확한 트리거 조건은 로컬 검증 대상 — 사실 단언 금지)
  2. Ch.10 학습 루프 챕터를 따라 Do→Learn→Improve 사이클과 스킬 자가 개선 흐름을 실습할 수 있다
  3. Ch.11 MCP 연동 챕터를 따라 외부 MCP 서버 연결과 `hermes mcp serve` 양방향 설정을 완료할 수 있다
  4. Ch.12 크론 챕터를 따라 `hermes cron`으로 무인 예약 작업을 등록하고 실행을 확인할 수 있다 (`hermes gateway`는 스케줄 실행·결과 배달 서비스로만 사용)
  5. Ch.13 서브에이전트 챕터를 따라 병렬 작업 위임을 설정하고 Kanban 보드 흐름을 이해할 수 있다
**Plans**: 5 plans (Wave 1: 04-01 / Wave 2: 04-02~04-05 병렬)

Plans:
- [x] 04-01: Ch.9 스킬 시스템 + SUMMARY.md 소유 — SKILL.md 형식, agentskills.io 표준, 직접 작성 실습, 번들 ~89개, 커뮤니티 허브
- [x] 04-02: Ch.10 학습 루프 — Do→Learn→Improve 사이클, 스킬 자가 개선, 플래그십 기능 실습
- [x] 04-03: Ch.11 MCP 연동 — 양방향(클라이언트 연결 + `hermes mcp serve` 메시징 툴 노출), ACP는 별개 통합
- [x] 04-04: Ch.12 크론 스케줄러 — `hermes cron` 무인 예약, 스킬 주입, 멀티플랫폼 배달, workdir 순차 실행 함정
- [x] 04-05: Ch.13 서브에이전트 — delegate_task 병렬 위임(최대 3 동시), Kanban 내구 시스템(dashboard/dispatcher), 선택 기준

---

### Phase 5: Platforms & Operations
**Goal**: 독자가 메시징 게이트웨이 봇을 설정하고 프로덕션 배포와 보안 하드닝을 완료할 수 있다 (Ch.14–18)
**Depends on**: Phase 4
**Requirements**: PLAT-01, PLAT-02, PLAT-03
**Success Criteria** (what must be TRUE):
  1. Ch.14 메시징 게이트웨이 챕터를 따라 Telegram 봇을 설정하고 Hermes와 대화할 수 있다 (Discord Message Content Intent 함정 포함)
  2. Ch.15–16 추가 게이트웨이 챕터를 따라 Discord/Slack/WhatsApp/Signal/Email 중 하나를 추가로 설정할 수 있다
  3. Ch.17 배포 챕터를 따라 적어도 하나의 시나리오($5 VPS, Docker, SSH, Modal, Daytona)로 Hermes를 프로덕션 배포할 수 있다
  4. Ch.18 보안 챕터를 따라 승인 모드, `.env` 기반 API 키 관리, `hermes doctor` 운영 런북을 적용할 수 있다
**Plans**: TBD

Plans:
- [ ] 05-01: Ch.14 Telegram 게이트웨이 — 봇 토큰 설정, `hermes gateway`, 폴링 중복 함정
- [ ] 05-02: Ch.15–16 추가 게이트웨이 — Discord (Intents 함정), Slack, WhatsApp/Signal/Email
- [ ] 05-03: Ch.17 배포 전략 — VPS/Docker/SSH/Modal/Daytona 시나리오별 단계, Docker root 파일 함정
- [ ] 05-04: Ch.18 보안 하드닝 — 승인 모드 설정, `.env` 키 관리, PII 리댁션, `hermes doctor` 런북

---

### Phase 6: Reference
**Goal**: 독자가 CLI·슬래시 커맨드·설정 레퍼런스와 통합 트러블슈팅 색인을 언제든 조회할 수 있다
**Depends on**: Phase 5 (모든 챕터 확정 후 집약)
**Requirements**: REF-01
**Success Criteria** (what must be TRUE):
  1. 레퍼런스 부록 페이지가 배포 사이트에서 렌더링되고 CLI 명령어·슬래시 커맨드 전체 목록이 조회된다
  2. `config.yaml` 주석 달린 레퍼런스가 존재하고 각 키의 기본값과 허용값이 명시된다
  3. 마스터 트러블슈팅 색인이 모든 챕터의 "흔한 오류" 섹션을 집약해 오류 메시지로 검색 가능하다
**Plans**: TBD

Plans:
- [ ] 06-01: CLI·슬래시 커맨드 레퍼런스 — 전체 명령어 목록, 옵션, 예시 (공식 docs 기반)
- [ ] 06-02: `config.yaml` 주석 레퍼런스 — 전체 키, 기본값, 허용값, `.env` 분리 원칙
- [ ] 06-03: 마스터 트러블슈팅 색인 — 챕터별 "흔한 오류" 집약, 용어 사전

---

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3 → 4 → 5 → 6

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Scaffold & CI | 3/3 | ✓ Complete | 2026-06-09 |
| 2. Foundations | 4/4 | ✓ Complete | 2026-06-09 |
| 3. Core Concepts | 5/5 | ✓ Complete | 2026-06-10 |
| 4. Learning & Automation | 5/5 | ✓ Complete | 2026-06-10 |
| 5. Platforms & Operations | 0/4 | Not started | - |
| 6. Reference | 0/3 | Not started | - |

---
*Roadmap created: 2026-06-05*
*Last updated: 2026-06-10 after Phase 4 completion (Ch.0–13 live at https://ohama.github.io/Hermes-Tutorial/)*
