# Project State

## Project Reference

See: .planning/PROJECT.md (updated 2026-06-05)

**Core value:** 개발자가 따라 하면 Hermes Agent를 설치·실행하고, 스킬·게이트웨이·배포까지 실제로 동작시킬 수 있다. 모든 명령어·API는 공식 소스 근거 필수.
**Current focus:** Phase 4 — Learning & Automation (content chapters Ch.9–13)

## Current Position

Phase: 4 of 6 (Learning & Automation) — **In progress**
Plan: 5 of 5 in current phase — **04-05 DONE** (Wave 2: Ch.13 서브에이전트 본문 303줄; Phase 4 마지막 챕터)
Status: 04-05 complete — src/13-subagents/index.md 303줄(delegate_task 격리/동시성/leaf 제약 + Kanban 내구 SQLite 시스템 + 선택 기준 비교표); mdbook build exit 0; SUMMARY.md 미수정
Last activity: 2026-06-10 — Completed 04-05-PLAN.md (Ch.13 서브에이전트 본문 작성)

Progress: [███████████████░] ~72% (14/~18 plans estimated)

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
- 03-03: grep 검증 조건  충족을 위해 잘못된 경로 예시를 리터럴 대신 설명("memories/ 하위 디렉터리를 생략한 경로")으로 표기
- 03-03: "4,500x" 벤치마크 미인용 — RESEARCH.md Open Question #7에 따라 "~20ms FTS5 쿼리"만 사용
- 03-03: Task 2(빌드 검증)는 파일 변경 없음 — no-op 커밋 생략, mdbook build exit 0만 확인
- 03-04: agent.disabled_toolsets 키는 존재하지 않음 — toolsets 배열/hermes tools UI/session /tools disable 만 유효 (주의 callout에서만 언급)
- 03-04: Nous Tool Gateway 무료 풀 정확 수치 단정 안 함 — Open Question #2, 검증 필요 callout + Nous Portal 가격 페이지 확인 안내
- 03-04: kanban 툴셋 opt-in 명시 — all/* 와일드카드에도 미포함
- 03-04: use_gateway: true는 직접 API 키보다 우선함 명시
- 03-05: Docker 하드닝 플래그 목록은 설명 텍스트로 표현 — 독자 설정 불필요이므로 # 검증: 주석 예외 처리
- 03-05: hermes setup terminal 마법사 흐름 MEDIUM → 검증 필요 callout (Open Question #3)
- 03-05: Phase 5 전방 참조 = plain prose only (상대 링크 금지 — 파일 미존재)
- 03-05: 원격 백엔드 섹션 상단에 전체 섹션 MEDIUM 신뢰도 경고 callout 배치
- 04-01: SUMMARY.md Wave-1 단독 소유 패턴 계속 적용 — 후속 Wave-2 플랜(04-02~05)이 src/SUMMARY.md를 건드리지 않아도 됨, 중복 경로 빌드 오류 방지
- 04-01: 번들 스킬 수를 '약 89개(v0.15.x)'로 기술 — 118(v0.10.0 과거값)·19,932 커뮤니티 집계 수치를 사실로 단언하지 않음
- 04-01: 스킬 자동 생성 정확한 조건은 '> 검증 필요' 헤지로 처리 — '5회 툴 호출마다' 단언 금지
- 04-01: hermes skills list 출력은 [로컬 실행 후 캡처 필요] placeholder + 검증 필요 callout
- 04-01: Phase 4 검증 주석은 '# 검증: hermes rolling, 2026-06-10' (Phase 3의 06-09와 구분)
- 04-04: hermes cron run 독립 실행 여부 MEDIUM → 검증 필요 callout 두 곳 적용 (Open Question #4 미해결)
- 04-04: workdir 지정 시 순차 실행 사이드이펙트(프로세스 전역 상태)를 주의 callout으로 명시 (RESEARCH.md HIGH)
- 04-04: Task 2(빌드 검증) 소스 변경 없음 → no-op 커밋 생략 (03-03 패턴 재적용)
- 04-03: hermes mcp serve = ~10개 플랫폼 메시징 툴 stdio 서버; 전체 에이전트 릴레이 단언 없음
- 04-03: ACP(hermes acp) vs MCP 별개 통합 구분 — 이 튜토리얼은 MCP만, ACP는 간단 언급
- 04-03: tools.include + tools.exclude 동시 설정 시 include 우선(exclude 무시); 화이트리스트 권장
- 04-03: 에디터(Cursor/VS Code/Zed) MCP 설정 경로 미검증 → placeholder + 검증 필요 callout
- 04-03: Task 2(빌드 검증)는 파일 변경 없음 — no-op 커밋 생략 패턴 계속 적용
- 04-05: delegate_task Python 시그니처 코드블록은 실행 명령이 아닌 설명용 — 주석으로 명시, '# 검증:' 주석 예외 처리
- 04-05: Kanban을 단순 opt-in toolset이 아닌 SQLite(kanban.db)+CLI+hermes dashboard+hermes gateway start dispatcher 내구 시스템으로 기술
- 04-05: Phase 5 전방 참조 = plain prose only (14+ 챕터 파일 링크 없음) — 03-05 패턴 재확인
- 04-05: Task 2(빌드 검증) 파일 변경 없음 → no-op 커밋 생략 패턴 계속 적용 (03-03/04-03/04-04 동일)

### Pending Todos

None yet.

### Blockers/Concerns

- **[Node.js 20 deprecation — LOW, 2026-06-16]** GitHub Actions annotations warn that checkout@v4, configure-pages@v5, and upload-artifact@v4 use Node.js 20 (deprecated); GitHub forces Node 24 runtime from 2026-06-16. Non-blocking — actions continue to work post-migration. Future maintenance: update to action versions natively targeting Node 24 when available. Does NOT block Phase 2.
- Phase 4: Honcho integration 세부 사항 (외부 계정 필요 여부, 정확한 config 키) MEDIUM confidence — Ch.10 작성 전 별도 검증 필요
- Phase 5: Discord OAuth + Privileged Gateway Intents 흐름 2026-03 이후 변경 가능 — Discord 서브챕터 작성 전 검증 필요
- Phase 5: Modal/Daytona 백엔드 YAML 키 공개 문서 미확인 — serverless 배포 섹션 작성 전 검증 필요

## Session Continuity

Last session: 2026-06-10 (Plan 04-05 executed; Ch.13 서브에이전트 본문 303줄; delegate_task+Kanban 내구 시스템+비교표; mdbook build exit 0; SUMMARY.md 미수정)
Stopped at: Plan 04-05 complete — Phase 4 Wave-2 마지막 챕터(Ch.13 서브에이전트) 완성; 04-02(Ch.10 학습 루프) 미완료 시 해당 플랜 후 Phase 4 complete 처리 가능
Resume file: None
Next workflow trigger: 04-02(Ch.10 학습 루프) — 아직 미완료라면 실행; 이후 `/gsd:complete-phase 04` → Phase 5(배포/게이트웨이) 시작
