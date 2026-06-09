---
phase: 02-foundations
plan: "04"
subsystem: content
tags: [hermes, mdbook, model-setup, providers, api-keys, korean]

# Dependency graph
requires:
  - phase: 02-01
    provides: SUMMARY.md 스켈레톤 + 03-model/index.md placeholder (빈 파일)
provides:
  - Ch.3 "모델 설정" 완전한 본문 (src/03-model/index.md)
  - hermes model 마법사 + hermes setup --portal 설명
  - Tier 1/2 공급자 표 + 인증 방식
  - API 키 저장 위치 (.env / config.yaml / auth.json 분리)
  - config.yaml 모델 섹션 예시
  - 64K 최소 컨텍스트 요건 + ~13,935 토큰 고정 오버헤드
  - display.language ko 부분 한국어 지원 정확한 설명
  - 5개 주의 callout (Claude Pro/Max, #16394, .env, 64K, /model)
  - 다음 단계 평문 산문 (Phase 3+ 링크 없음)
affects:
  - 02-02 (Ch.1 설치), 02-03 (Ch.2 첫 실행) — 이미 완료; 이 플랜의 Ch.3이 그 뒤를 잇는다
  - Phase 3+ 챕터 (컨텍스트·메모리·도구·게이트웨이·배포) — 아직 생성 전

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "RESEARCH.md 사실만 사용, 불확실 항목(OpenAI env var, Ollama 플래그)은 명시적 불확실성 표기"
    - "# 검증: <component> <version>, YYYY-MM-DD 주석 모든 코드블록에 적용"
    - "Phase 3+ 안내는 평문 산문만 — 존재하지 않는 파일 링크 금지 (Pitfall B-11)"

key-files:
  created: []
  modified:
    - src/03-model/index.md

key-decisions:
  - "OpenAI 환경변수명 OPENAI_API_KEY는 관례이며 MEDIUM confidence — 마법사로 확인 권장으로 표기"
  - "Ollama 컨텍스트 플래그는 --num-ctx (LOW confidence) — ollama --help로 확인 권장 callout 추가"
  - "Nous Portal 가격 정보는 LOW confidence — 공식 요금 페이지 확인 권장으로만 언급"
  - "display.language: ko = 부분 지원 (정적 메시지만); '미지원' 또는 '완전 지원' 표현 금지"
  - "Phase 3+ 다음 단계 안내는 mdBook 상대 링크 없이 평문 산문으로만 작성"

patterns-established:
  - "5개 주의 callout: blockquote admonition 형식 (> **경고/주의/참고:** ...)"
  - "불확실 항목: 표에 괄호로 불확실성 표기 + 별도 참고 callout"

# Metrics
duration: 2min
completed: 2026-06-09
---

# Phase 02 Plan 04: Ch.3 모델 설정 Summary

**hermes model 마법사·setup --portal·공급자 표·API 키 저장 분리·64K 오버헤드·ko 부분 지원·5개 CRITICAL 주의 callout을 포함한 Ch.3 완전 본문 작성 및 mdbook build 검증 완료.**

## Performance

- **Duration:** 2min
- **Started:** 2026-06-09T08:34:28Z
- **Completed:** 2026-06-09T08:36:35Z
- **Tasks:** 4 completed
- **Files modified:** 1 (src/03-model/index.md)

## Accomplishments

- `src/03-model/index.md` 완전 본문 작성: 공급자 설정 마법사, 공급자 표, API 키 저장 위치, config.yaml 모델 섹션, 64K 요건, ko 부분 지원, 5개 주의 callout, 다음 단계
- 모든 명령/config 코드블록에 `# 검증: hermes rolling, 2026-06-09` 주석 적용; 불확실 항목 명시적 표기
- `mdbook build` exit 0 — `book/03-model/index.html` 정상 렌더링 확인

## Task Commits

Each task was committed atomically:

1. **Task 1: Ch.3 본문 — 공급자 설정 마법사 + 공급자 표 + 키 저장 위치** - `d3525e5` (feat)
2. **Task 2: Ch.3 본문 — config.yaml 모델 섹션 + 64K 요건 + display.language ko** - `60ed177` (feat)
3. **Task 3: Ch.3 본문 — 5개 주의 callout + 다음 단계** - `2051612` (feat)
4. **Task 4: 빌드 검증** - (no commit needed — no files changed; build passed)

**Plan metadata:** (docs commit follows)

## Files Created/Modified

- `src/03-model/index.md` — Ch.3 "모델 설정" 완전 본문 (마법사·공급자·키·config·64K·ko·주의·다음 단계)

## Decisions Made

- **OpenAI env var:** `OPENAI_API_KEY` (관례, MEDIUM confidence) — 표에 불확실성 표기 + 마법사로 확인 권장 callout 추가
- **Ollama 플래그:** `--num-ctx` (LOW confidence) — 검증 필요 callout로 처리; `--context-size` 가능성도 언급
- **Nous Portal 가격:** LOW confidence — 단정 없이 "공식 요금 페이지 확인" 권장으로만 언급
- **display.language ko:** RESEARCH.md 정정 반영 — "부분 한국어 지원" (정적 메시지만); "미지원" 표현 사용 금지
- **Phase 3+ 다음 단계:** 평문 산문만 — `../04-`, `../05-` 등 존재하지 않는 파일 링크 전혀 없음 (Pitfall B-11 준수)

## Deviations from Plan

None — plan executed exactly as specified.
