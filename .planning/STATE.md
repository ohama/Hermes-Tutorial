# Project State

## Project Reference

See: .planning/PROJECT.md (updated 2026-06-05)

**Core value:** 개발자가 따라 하면 Hermes Agent를 설치·실행하고, 스킬·게이트웨이·배포까지 실제로 동작시킬 수 있다. 모든 명령어·API는 공식 소스 근거 필수.
**Current focus:** Phase 2 — Foundations (content chapters Ch.0–3)

## Current Position

Phase: 2 of 6 (Foundations) — **Phase 2 COMPLETE**
Plan: 4 of 4 in current phase — **ALL DONE** (02-01/02-02/02-03/02-04 complete)
Status: Phase 2 complete — Ch.0~3 all written; mdbook build passing
Last activity: 2026-06-09 — Completed 02-04-PLAN.md (Ch.3 모델 설정 — hermes model 마법사 + 공급자 표 + API 키 저장 분리 + config.yaml 모델 섹션 + 64K 요건 + ko 부분 지원 + 5개 주의 callout + mdbook build exit 0)

Progress: [████████░░] ~44% (8/~18 plans estimated)

## Performance Metrics

**Velocity:**
- Total plans completed: 8
- Average duration: ~5 min
- Total execution time: ~39 min

**By Phase:**

| Phase | Plans | Total | Avg/Plan |
|-------|-------|-------|----------|
| 01-scaffold-ci | 3/3 (DONE) | ~33 min | ~11 min |
| 02-foundations | 4/4 (DONE) | ~6 min | ~2 min |

**Recent Trend:**
- Last 5 plans: 01-01 (2 min), 01-02 (1 min), 01-03 (~30 min incl. human-action), 02-01 (~2 min), 02-02 (~2 min), 02-03 (~2 min)
- Trend: Wave-2 plans (02-02/03) very fast — pure content writing from RESEARCH, no human-action needed

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

### Pending Todos

None yet.

### Blockers/Concerns

- **[Node.js 20 deprecation — LOW, 2026-06-16]** GitHub Actions annotations warn that checkout@v4, configure-pages@v5, and upload-artifact@v4 use Node.js 20 (deprecated); GitHub forces Node 24 runtime from 2026-06-16. Non-blocking — actions continue to work post-migration. Future maintenance: update to action versions natively targeting Node 24 when available. Does NOT block Phase 2.
- Phase 4: Honcho integration 세부 사항 (외부 계정 필요 여부, 정확한 config 키) MEDIUM confidence — Ch.10 작성 전 별도 검증 필요
- Phase 5: Discord OAuth + Privileged Gateway Intents 흐름 2026-03 이후 변경 가능 — Discord 서브챕터 작성 전 검증 필요
- Phase 5: Modal/Daytona 백엔드 YAML 키 공개 문서 미확인 — serverless 배포 섹션 작성 전 검증 필요

## Session Continuity

Last session: 2026-06-09 (Plan 02-04 executed; Ch.3 모델 설정 완성; build exit 0; Phase 2 ALL DONE)
Stopped at: Plan 02-04 complete — Phase 2 complete (4/4 plans done); Ch.0~3 all written; mdbook build passing
Resume file: None
Next workflow trigger: `/gsd:plan-phase 03` — Phase 3 (컨텍스트·메모리) 또는 `/gsd:complete-phase 02` to archive Phase 2
