---
phase: 03-core-concepts
plan: 02
subsystem: content
tags: [mdbook, hermes, context-files, soul-md, personality, config, dotenv]

# Dependency graph
requires:
  - phase: 03-01
    provides: SUMMARY.md 핵심 개념 섹션 + 05-context-files/index.md 빈 파일 등록
provides:
  - Ch.5 컨텍스트 파일 본문 (src/05-context-files/index.md, 233줄)
  - 컨텍스트 파일 우선순위 캐스케이드(.hermes.md→AGENTS.md→CLAUDE.md→.cursorrules) 설명
  - SOUL.md 전역 정체성 구분 (레이어 1 vs 레이어 8)
  - /personality 14개 내장 페르소나 표 + 커스텀 config.yaml 예시
  - .env vs config.yaml 분리 + ${VAR} 치환 문법
  - 4개 주의 callout
  - LOW confidence placeholder (배너 컨텍스트 로드 메시지)
affects:
  - 03-03 (Ch.6 메모리 — 다음 단계 링크 수신자)
  - 03-01 (SUMMARY.md 소유 — 수정 없음 확인)

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "LOW confidence placeholder + 검증 필요 callout 패턴 (02-02에서 이어짐)"
    - "# 검증: hermes rolling, YYYY-MM-DD 코드블록 주석 관례 (Phase 2부터 적용)"

key-files:
  created:
    - src/05-context-files/index.md
  modified: []

key-decisions:
  - "SOUL.md 위치: ~/.hermes/SOUL.md — 프로젝트 디렉터리 아님 (RESEARCH.md HIGH confidence 그대로)"
  - "배너 컨텍스트 로드 메시지 LOW → placeholder + 검증 필요 callout (발명 금지 패턴 준수)"
  - "14개 내장 페르소나 정확히 표에 나열 (RESEARCH.md HIGH confidence, 개수 발명 없음)"
  - "src/SUMMARY.md 미수정 확인 — 03-01 owner-plan 패턴 유지"

patterns-established:
  - "Wave 2 owner 패턴: 03-01이 등록한 빈 파일을 overwrite만 함 (write race 없음)"

# Metrics
duration: 2min
completed: 2026-06-10
---

# Phase 3 Plan 02: Ch.5 컨텍스트 파일 Summary

**컨텍스트 파일 우선순위 캐스케이드(.hermes.md→AGENTS.md→CLAUDE.md→.cursorrules, 첫 매치만 로드)·SOUL.md 전역 구분·/personality 14개·${VAR} 치환을 담은 Ch.5 본문 233줄 작성**

## Performance

- **Duration:** ~2 min
- **Started:** 2026-06-10T07:53:12Z
- **Completed:** 2026-06-10T07:55:29Z
- **Tasks:** 2/2 완료
- **Files modified:** 1

## Accomplishments

- `src/05-context-files/index.md` (233줄) 작성 — RESEARCH.md Ch.5 HIGH confidence 사실만 사용
- 컨텍스트 파일 우선순위 캐스케이드 표, SOUL.md 레이어 비교 표, 처리 규칙(20,000자/보안 스캔/frontmatter 제거) 포함
- 실습 섹션: `.hermes.md` 생성 코드블록, 배너 메시지 LOW→placeholder, SOUL.md 확인/편집, `hermes config show`
- /personality 14개 내장 페르소나 완전 표 + 커스텀 `agent.personalities.codereviewer` YAML 예시
- .env vs config.yaml 분리 표, `hermes config set` 자동 라우팅 코드블록, `${GOOGLE_API_KEY}` 치환 예시
- 4개 주의 callout (세션 시작 한 번 로드 / 첫 매치만 / API 키 config.yaml 금지 / ${VAR} 중괄호 필수)
- `mdbook build` exit 0 확인, `git diff --name-only src/SUMMARY.md` = 빈 출력 확인

## Task Commits

1. **Task 1: Ch.5 컨텍스트 파일 본문 작성** - `9b60acf` (feat)
2. **Task 2: 빌드 검증** - 별도 커밋 없음 (build-only verification, 소스 변경 없음)

**Plan metadata:** (아래 docs commit)

## Files Created/Modified

- `src/05-context-files/index.md` (생성, 233줄) — Ch.5 컨텍스트 파일 본문 전체

## Decisions Made

- **배너 메시지 LOW confidence 처리:** 공개 문서에 명시된 배너 텍스트가 없으므로 `# [hermes 시작 배너에서 컨텍스트 파일 로드 메시지 확인 필요 — 로컬 캡처 필요]` placeholder + `> 검증 필요:` callout으로 처리. 발명 금지 패턴 준수.
- **SOUL.md 위치 명시:** RESEARCH.md HIGH confidence — `~/.hermes/SOUL.md`. 프로젝트 디렉터리에 두지 않음을 별도 표로 강조.
- **Wave 2 owner 패턴 유지:** 03-01이 등록한 빈 파일을 overwrite만 하고 src/SUMMARY.md는 미수정. 빌드 후 `git diff --name-only src/SUMMARY.md` 빈 출력으로 확인.

## Deviations from Plan

None — plan executed exactly as written.

## Next Phase Readiness

- Ch.5 본문 완성. Ch.6 메모리(03-03) 링크 수신 가능.
- mdbook build exit 0 유지. src/SUMMARY.md 미변경.
- Wave 2 병렬 실행 구조 정상 동작 확인.
