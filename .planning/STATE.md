# Project State

## Project Reference

See: .planning/PROJECT.md (updated 2026-06-05)

**Core value:** 개발자가 따라 하면 Hermes Agent를 설치·실행하고, 스킬·게이트웨이·배포까지 실제로 동작시킬 수 있다. 모든 명령어·API는 공식 소스 근거 필수.
**Current focus:** Phase 3 — Core Concepts (content chapters Ch.4–8)

## Current Position

Phase: 3 of 6 (Core Concepts) — **In progress**
Plan: 2 of 5 in current phase — **03-02 DONE** (Wave 2: Ch.5 컨텍스트 파일 complete)
Status: 03-02 complete — Ch.5 컨텍스트 파일 본문 233줄 작성; 우선순위 캐스케이드·SOUL.md·/personality 14개·${VAR} 치환; mdbook build exit 0; SUMMARY.md 미수정
Last activity: 2026-06-10 — Completed 03-02-PLAN.md (Ch.5 컨텍스트 파일 — .hermes.md→AGENTS.md→CLAUDE.md→.cursorrules 캐스케이드 + SOUL.md 전역 구분 + /personality 14개 + .env vs config.yaml 분리 + mdbook build exit 0)

Progress: [██████████░] ~55% (10/~18 plans estimated)

## Performance Metrics

**Velocity:**
- Total plans completed: 9
- Average duration: ~5 min
- Total execution time: ~41 min

**By Phase:**

| Phase | Plans | Total | Avg/Plan |
|-------|-------|-------|----------|
| 01-scaffold-ci | 3/3 (DONE) | ~33 min | ~11 min |
| 02-foundations | 4/4 (DONE) | ~6 min | ~2 min |
| 03-core-concepts | 2/5 (in progress) | ~5 min | ~2.5 min |

**Recent Trend:**
- Last 5 plans: 02-03 (~2 min), 02-04 (~2 min), 03-01 (3 min), 03-02 (~2 min)
- Trend: Content plans very fast — pure writing from RESEARCH, no human-action needed

*Updated after each plan completion*

## Accumulated Context

### Decisions

Decisions are logged in PROJECT.md Key Decisions table.
Recent decisions affecting current work:

- Init: mdBook 0.5.3 + `actions/deploy-pages@v5` 공식 스타터 워크플로우 사용 (pre-built 바이너리, cargo install 금지)
- Init: 한국어 UI (`display.language: ko`) 초기 기록은 "미지원" — 02-01 RESEARCH.md에서 정정됨 (아래 참조)
- 02-01: 한국어 UI **부분 지원** 정정 — `ko`는 지원 목록에 있으나 범위 협소(승인 프롬프트 등 정적 메시지만 번역; 에이전트 응답·로그·도구 출력은 영어 유지). Ch.3에서 정확히 설명할 것.
- 02-01: SUMMARY.md Wave-1 단독 소유 패턴 확립 — 후속 Wave-2 플랜(02-02/03/04)이 src/SUMMARY.md를 건드리지 않아도 됨, 중복 경로 빌드 오류 방지
- 02-01: mdBook 0.5.3 auto-placeholder 패턴 확인 — SUMMARY.md 링크 미작성 파일은 build 시 자동 생성됨 (exit 0)
- Init: 코드블록 버전·검토일 주석 관례 (`# 검증: hermes vX.Y, YYYY-MM-DD`) Phase 1부터 적용
- 01-01: mdBook 0.5.3 was already on PATH (pre-built binary); installation step skipped per plan's skip-if-present instruction
- 01-01: `<owner>` placeholders left in git-repository-url/edit-url-template — plan 01-03 reconciles once remote exists
- 01-01: book.toml must NOT include multilingual/curly-quotes/copy-fonts/google-analytics/smart-punctuation/hash-files (removed in 0.5.0, cause hard errors)
- 01-02: deploy.yml uses pre-built binary install (not cargo install); pins deploy-pages@v5; permissions pages:write + id-token:write required by deploy-pages
- 01-02: YAML validation via PyYAML (python3 yaml.safe_load) — confirmed available on system
- 01-03: owner=ohama, repo=Hermes-Tutorial; remote=https://github.com/ohama/Hermes-Tutorial; live URL=https://ohama.github.io/Hermes-Tutorial/
- 01-03: book.toml site-url="/Hermes-Tutorial/", git-repository-url/edit-url-template reconciled to ohama/Hermes-Tutorial (commit cc2701b)
- 01-03: GitHub Pages source must be enabled (build_type=workflow via gh api) BEFORE first push — ordering non-negotiable (Pitfall 3)
- 01-03: Workflow run 27192158593 green (build 7s + deploy 10s); live site HTTP 200, Korean TOC renders
- 02-02: hermes doctor 출력 LOW confidence → placeholder + 검증 필요 callout; 발명 금지 패턴 확립
- 02-02: hermes version 예시(0.8.0) MEDIUM confidence → "예시" 명시, 현재 버전으로 단정하지 않음
- 02-02: Tasks 1+2를 단일 파일 작성으로 처리 — 동일 파일 순차 편집이므로 완전한 파일 한 번에 작성 + atomic commit
- 02-04: OpenAI env var `OPENAI_API_KEY` MEDIUM confidence — 표에 불확실성 표기, 마법사로 확인 권장
- 02-04: Ollama 컨텍스트 플래그 `--num-ctx` LOW confidence — 검증 필요 callout로 처리
- 02-04: Nous Portal 가격 LOW confidence — 공식 요금 페이지 확인 권장으로만 언급, 단정 없음
- 02-04: display.language ko = 부분 지원 확정 기술 (정정 반영); '미지원' 표현 완전 제거
- 02-04: Phase 3+ 다음 단계 = 평문 산문만 (Pitfall B-11 준수; 존재하지 않는 파일 링크 없음)
- 03-01: perceive→decide→act 프레이밍 완전 배제 — 공식 사이클 Assemble→Call→Parse→Execute→Persist→Loop만 사용 (검증 항목으로 강제)
- 03-01: SUMMARY.md owner-plan 패턴 재확인 — Wave 2 플랜(03-02~05)이 SUMMARY.md 쓰기 충돌 없이 병렬 실행 가능
- 03-01: mdBook auto-created 05~08 files는 커밋 제외 — Wave 2 소유 파일이므로 Wave 2 플랜이 덮어씀
- 03-02: 배너 컨텍스트 로드 메시지 LOW → placeholder + 검증 필요 callout (발명 금지 패턴 준수)
- 03-02: SOUL.md 위치 ~/ .hermes/SOUL.md HIGH confidence — 프로젝트 디렉터리 아님을 표로 강조
- 03-02: 14개 내장 페르소나 정확히 표 나열 (RESEARCH.md HIGH confidence, 개수 발명 없음)

### Pending Todos

None yet.

### Blockers/Concerns

- **[Node.js 20 deprecation — LOW, 2026-06-16]** GitHub Actions annotations warn that checkout@v4, configure-pages@v5, and upload-artifact@v4 use Node.js 20 (deprecated); GitHub forces Node 24 runtime from 2026-06-16. Non-blocking — actions continue to work post-migration. Future maintenance: update to action versions natively targeting Node 24 when available. Does NOT block Phase 2.
- Phase 4: Honcho integration 세부 사항 (외부 계정 필요 여부, 정확한 config 키) MEDIUM confidence — Ch.10 작성 전 별도 검증 필요
- Phase 5: Discord OAuth + Privileged Gateway Intents 흐름 2026-03 이후 변경 가능 — Discord 서브챕터 작성 전 검증 필요
- Phase 5: Modal/Daytona 백엔드 YAML 키 공개 문서 미확인 — serverless 배포 섹션 작성 전 검증 필요

## Session Continuity

Last session: 2026-06-10 (Plan 03-02 executed; Ch.5 컨텍스트 파일 본문 233줄 작성; build exit 0; SUMMARY.md 미수정)
Stopped at: Plan 03-02 complete — Phase 3 Wave 2 Ch.5 done; Ch.5 컨텍스트 파일 완성 (우선순위 캐스케이드 + SOUL.md + /personality 14개 + .env/config.yaml 분리); Wave 2 나머지 03-03~05 계속 가능
Resume file: None
Next workflow trigger: Wave 2 계속 — `/gsd:execute-plan 03-03` (Ch.6 메모리), `03-04` (Ch.7 툴 게이트웨이), `03-05` (Ch.8 터미널 백엔드)
