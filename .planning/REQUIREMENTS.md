# Requirements: Hermes Agent 한국어 튜토리얼

**Defined:** 2026-06-05
**Core Value:** 개발자가 따라 하면 Hermes Agent를 설치·실행하고, 스킬·게이트웨이·배포까지 실제로 동작시킬 수 있다. 모든 명령어·API는 실제 repo/공식 문서에 근거해 정확해야 한다.

## v1 Requirements

초기 릴리스 범위. 각 요구사항은 로드맵 페이즈에 매핑된다. (모든 본문은 한국어, 코드·명령어는 원문 유지. Hermes UI는 영어 — `display.language`에 `ko` 미지원이므로 명시.)

### Infrastructure (인프라·결과물 토대)

- [x] **INFRA-01**: mdBook 0.5.3 프로젝트 구조(`book.toml`, `src/`, `SUMMARY.md`)가 빌드 가능하게 구성된다
- [x] **INFRA-02**: `book.toml`에 `language = "ko"`와 GitHub Pages용 `site-url = "/<repo>/"`가 설정된다
- [x] **INFRA-03**: GitHub Actions로 mdBook을 빌드하고 `actions/deploy-pages@v5`로 GitHub Pages에 자동 배포한다
- [x] **INFRA-04**: 모든 코드블록은 버전·검토일 주석 관례를 따른다(정확성 추적)

### Foundations (기초)

- [ ] **FOUND-01**: 독자는 Hermes가 무엇이며 학습 루프 철학과 이 튜토리얼 사용법을 이해한다
- [ ] **FOUND-02**: 독자는 OS별(macOS/Linux/WSL2/Windows)로 Hermes를 설치할 수 있다(git 전제조건, PATH 갱신 함정 포함)
- [ ] **FOUND-03**: 독자는 `hermes`를 실행해 첫 대화를 하고 주요 명령어를 파악한다
- [ ] **FOUND-04**: 독자는 `hermes model`로 모델 제공자(OpenRouter/Nous Portal/OpenAI 등)와 API 키를 설정한다(64K 최소 컨텍스트·키 무음 스킵 주의)

### Core Concepts (핵심 개념)

- [ ] **CORE-01**: 독자는 AIAgent 코어 루프 동작 원리와 프롬프트 시스템을 이해한다
- [ ] **CORE-02**: 독자는 컨텍스트 파일 계층(`.hermes.md`/`AGENTS.md`/`SOUL.md`)과 퍼스널리티(`/personality`)를 설정한다
- [ ] **CORE-03**: 독자는 메모리 시스템(`MEMORY.md`/`USER.md`, SQLite FTS5 대화 검색, 압축)을 이해하고 활용한다
- [ ] **CORE-04**: 독자는 내장 툴과 툴 게이트웨이(`hermes tools`, 검색/이미지/TTS/브라우저)를 설정하고 승인 모드를 이해한다

### Learning & Automation (학습·자동화)

- [ ] **LEARN-01**: 독자는 스킬 시스템(SKILL.md, agentskills.io 표준, 자동 생성·자가 개선)을 이해하고 직접 스킬을 만든다
- [ ] **LEARN-02**: 독자는 MCP 연동을 양방향으로 설정한다(외부 MCP 서버 연결 + `hermes mcp serve`)
- [ ] **LEARN-03**: 독자는 cron 스케줄러로 무인 예약 작업을 설정한다
- [ ] **LEARN-04**: 독자는 서브에이전트 스폰으로 병렬 작업을 위임한다

### Platform & Operations (플랫폼·운영)

- [ ] **PLAT-01**: 독자는 메시징 게이트웨이(Telegram/Discord/Slack 등) 봇을 설정한다(토큰, Discord Message Content Intent 함정 포함)
- [ ] **PLAT-02**: 독자는 터미널 백엔드(local/Docker/SSH/Modal/Daytona)와 서버리스 배포를 구성한다
- [ ] **PLAT-03**: 독자는 보안을 하드닝한다(승인 모드, `.env` 기반 API 키 관리, 프로덕션 주의점)
- [ ] **REF-01**: 독자는 CLI/슬래시/설정 레퍼런스와 통합 트러블슈팅 색인을 참조할 수 있다

## v2 Requirements

향후 릴리스로 연기. 추적하되 현재 로드맵에는 미포함.

### Advanced

- **ADV-01**: Honcho dialectic 사용자 모델링 심화(외부 API 계정 필요 — MEDIUM confidence)
- **ADV-02**: API 서버 / ACP 모드 심화
- **ADV-03**: 플러그인 시스템 심화
- **ADV-04**: 음성 메모 전사 파이프라인 상세
- **ADV-05**: 배치 처리 / RL trajectory export

### Localization

- **LOC-01**: 영어 버전 동시 제공
- **LOC-02**: KakaoTalk 등 한국 특화 게이트웨이(현재 공식 문서 미확인)

## Out of Scope

명시적으로 제외. 스코프 크리프 방지.

| Feature | Reason |
|---------|--------|
| Hermes Agent 소스 수정/기여 가이드 | 사용 튜토리얼이지 컨트리뷰션 가이드가 아님 |
| 동영상/스크린캐스트 | 텍스트 기반 mdBook에 집중, 터미널 스크린샷은 stale 위험 |
| OpenClaw(전신) 마이그레이션 상세 | 신규 사용자 대상이라 우선순위 낮음 |
| 한국어 Hermes UI 설정 | `display.language`에 `ko` 미지원 — 영어 UI 사용을 명시 |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| INFRA-01 | Phase 1 — Scaffold & CI | Complete |
| INFRA-02 | Phase 1 — Scaffold & CI | Complete |
| INFRA-03 | Phase 1 — Scaffold & CI | Complete |
| INFRA-04 | Phase 1 — Scaffold & CI | Complete |
| FOUND-01 | Phase 2 — Foundations | Pending |
| FOUND-02 | Phase 2 — Foundations | Pending |
| FOUND-03 | Phase 2 — Foundations | Pending |
| FOUND-04 | Phase 2 — Foundations | Pending |
| CORE-01 | Phase 3 — Core Concepts | Pending |
| CORE-02 | Phase 3 — Core Concepts | Pending |
| CORE-03 | Phase 3 — Core Concepts | Pending |
| CORE-04 | Phase 3 — Core Concepts | Pending |
| LEARN-01 | Phase 4 — Learning & Automation | Pending |
| LEARN-02 | Phase 4 — Learning & Automation | Pending |
| LEARN-03 | Phase 4 — Learning & Automation | Pending |
| LEARN-04 | Phase 4 — Learning & Automation | Pending |
| PLAT-01 | Phase 5 — Platforms & Operations | Pending |
| PLAT-02 | Phase 5 — Platforms & Operations | Pending |
| PLAT-03 | Phase 5 — Platforms & Operations | Pending |
| REF-01 | Phase 6 — Reference | Pending |

**Coverage:**
- v1 requirements: 20 total
- Mapped to phases: 20
- Unmapped: 0

---
*Requirements defined: 2026-06-05*
*Last updated: 2026-06-05 after roadmap creation — all 20 v1 requirements mapped*
