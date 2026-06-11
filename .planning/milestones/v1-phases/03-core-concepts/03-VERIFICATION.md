---
phase: 03-core-concepts
verified: 2026-06-10T08:02:14Z
status: passed
score: 13/13 must-haves verified
re_verification: false
---

# Phase 3: 핵심 개념 Verification Report

**Phase Goal:** 독자가 에이전트 루프·컨텍스트 파일·메모리·툴 게이트웨이의 작동 원리를 이해하고 직접 설정할 수 있다 (Ch.4–8).
**Verified:** 2026-06-10T08:02:14Z
**Status:** passed
**Re-verification:** No — initial verification

---

## Goal Achievement

### Observable Truths

| # | Truth | Status | Evidence |
|---|-------|--------|----------|
| 1 | SUMMARY.md에 "# 핵심 개념" 섹션과 5개 챕터 항목(04–08)이 추가되며 기존 기초 4개 항목은 보존된다 | VERIFIED | `grep -c "index.md" src/SUMMARY.md` → 9; "# 핵심 개념" present; 04–08 all listed |
| 2 | SUMMARY.md에 중복 경로가 없다 | VERIFIED | `grep -o "(.*index.md)" src/SUMMARY.md \| sort \| uniq -d` → empty |
| 3 | Ch.4 에이전트 루프가 공식 사이클 Assemble→Call→Parse→Execute→Persist→Loop를 설명하고 "perceive→decide→act" 프레이밍이 없다 | VERIFIED | grep match for cycle phrase at lines 15, 29; grep for "perceive" returns nothing |
| 4 | Ch.4가 3티어 프롬프트 시스템(Stable/Context/Volatile)을 언급하고 Ch.5로 연결한다 | VERIFIED | 3티어 표 at lines 87–94; Ch.5 link at lines 95, 196 |
| 5 | Ch.4가 동기 엔진·툴 디스패치(ThreadPoolExecutor ≤8)·max_turns 90 종료 조건을 설명한다 | VERIFIED | ThreadPoolExecutor at line 116; max_turns at lines 19, 157, 169, 188 |
| 6 | Ch.5가 컨텍스트 파일 우선순위 캐스케이드(.hermes.md→AGENTS.md→CLAUDE.md→.cursorrules)를 설명하고 SOUL.md를 구분한다 | VERIFIED | Cascade table at lines 29–34; SOUL.md section lines 44–57 |
| 7 | Ch.5가 /personality 14개 내장 + config.yaml 커스텀을 다루고 .env vs config.yaml 분리를 설명한다 | VERIFIED | All 14 personas at lines 134–147; custom personalities config block; .env/.yaml separation section |
| 8 | Ch.6가 MEMORY.md/USER.md 경로를 ~/.hermes/memories/ 아래로 정확히 명시하고 잘못된 ~/.hermes/MEMORY.md 경로가 없다 | VERIFIED | `~/.hermes/memories/MEMORY.md` and `~/.hermes/memories/USER.md` present; `~/.hermes/MEMORY.md` absent |
| 9 | Ch.6가 session_search를 내부 에이전트 툴(슬래시 커맨드 아님)로 설명하고 messages_fts_trigram CJK callout이 존재한다 | VERIFIED | Line 104: "에이전트 내부 툴"; lines 127–129: trigram/CJK callout; lines 259: "/session_search 같은 슬래시 커맨드는 없습니다" |
| 10 | Ch.6가 압축 설정(threshold 0.50, target_ratio, protect_last_n)과 ~1,300토큰 메모리 오버헤드를 다루고 4,500x 수치를 인용하지 않는다 | VERIFIED | All three config keys present; "~1,300토큰" at line 164; "4,500"/"4500" absent |
| 11 | Ch.7가 hermes tools 검색/이미지/TTS/브라우저 게이트웨이 + 승인 모드를 다루고 agent.disabled_toolsets를 동작 키로 문서화하지 않는다 | VERIFIED | hermes tools, use_gateway, approvals, Tirith all present; disabled_toolsets appears only in warning callouts (lines 111, 272–276) |
| 12 | Ch.7가 Nous Portal 무료 할당량 정확 수치를 단정하지 않는다 | VERIFIED | Line 139: "무료 툴 풀의 정확한 할당량은 공식 문서에 명시되어 있지 않습니다" with verification callout |
| 13 | Ch.8가 6개 백엔드 개요, Docker 핸즈온, Modal/Daytona/Singularity 검증 필요 callout, Phase 5 plain prose 전방 참조(dead link 없음)를 포함한다 | VERIFIED | All 6 backends in table; Docker hands-on section; "검증 필요" callout at line 159 for modal/daytona/singularity; "Phase 5" prose at lines 6, 16, 287; no `](../09` or `](../10` links |

**Score:** 13/13 truths verified

---

### Required Artifacts

| Artifact | Min Lines | Actual | Status | Contains Check |
|----------|-----------|--------|--------|---------------|
| `src/SUMMARY.md` | — | 23 lines | VERIFIED | "# 핵심 개념", all 5 chapter paths 04–08 |
| `src/04-agent-loop/index.md` | 80 | 196 | VERIFIED | "Assemble", ThreadPoolExecutor, max_turns, 3티어 |
| `src/05-context-files/index.md` | 90 | 233 | VERIFIED | .hermes.md, AGENTS.md, CLAUDE.md, .cursorrules, SOUL.md, 14 personas, ${GOOGLE_API_KEY} |
| `src/06-memory/index.md` | 90 | 277 | VERIFIED | ~/.hermes/memories/MEMORY.md, messages_fts_trigram, threshold, protect_last_n |
| `src/07-tools/index.md` | 90 | 284 | VERIFIED | hermes tools, use_gateway, approvals, Tirith |
| `src/08-backends/index.md` | 90 | 287 | VERIFIED | docker, singularity, modal, daytona, ssh, cap-drop, docker_run_as_host_user |

---

### Key Link Verification

| From | To | Via | Status | Details |
|------|----|-----|--------|---------|
| `src/SUMMARY.md` | `src/04-agent-loop/index.md` | mdBook 핵심 개념 섹션 | WIRED | Exact path present once |
| `src/SUMMARY.md` | `src/05-context-files/index.md` | mdBook 핵심 개념 섹션 | WIRED | Exact path present once |
| `src/SUMMARY.md` | `src/06-memory/index.md` | mdBook 핵심 개념 섹션 | WIRED | Exact path present once |
| `src/SUMMARY.md` | `src/07-tools/index.md` | mdBook 핵심 개념 섹션 | WIRED | Exact path present once |
| `src/SUMMARY.md` | `src/08-backends/index.md` | mdBook 핵심 개념 섹션 | WIRED | Exact path present once |
| `src/04-agent-loop/index.md` | `src/05-context-files/index.md` | 다음 단계 링크 + 본문 인라인 링크 | WIRED | Lines 95, 196 |
| `src/05-context-files/index.md` | `src/06-memory/index.md` | 다음 단계 링크 | WIRED | Line 233 |
| `src/06-memory/index.md` | `src/07-tools/index.md` | 다음 단계 링크 | WIRED | Line 277 |
| `src/07-tools/index.md` | `src/08-backends/index.md` | 다음 단계 링크 | WIRED | Line 284 |
| `src/08-backends/index.md` | Phase 5 (배포) | plain prose only (no mdBook link) | WIRED | Lines 6, 16, 287; no `](../09` or `](../10` links |

---

### Requirements Coverage

| Requirement | Status | Notes |
|-------------|--------|-------|
| CORE-01: Ch.4 에이전트 루프 + 3티어 프롬프트 시스템 | SATISFIED | Assemble→…→Loop cycle, 3-tier table, tool dispatch, max_turns all present |
| CORE-02: Ch.5 컨텍스트 파일 계층 + /personality | SATISFIED | Cascade table (.hermes.md→.cursorrules), SOUL.md distinction, 14 personas, custom config, .env/.yaml split |
| CORE-03: Ch.6 MEMORY.md/USER.md + FTS5 + 압축 | SATISFIED | Correct paths (~/.hermes/memories/), trigram CJK callout, compression config, 1,300-token note |
| CORE-04: Ch.7 hermes tools + 게이트웨이 + 승인 모드 | SATISFIED | hermes tools, use_gateway, gateway features, approval modes, Tirith scan |
| PLAT-02 배경: Ch.8 백엔드 개요 + Docker 실습 | SATISFIED | 6-backend table, 3 switching methods, Docker hardening, hands-on, remote backend overviews |

---

### Anti-Patterns Found

None blocking goal achievement.

| File | Pattern | Severity | Notes |
|------|---------|----------|-------|
| `src/07-tools/index.md` | "kategorie" (line 11) | Info | Appears to be a typo/German word for "카테고리" in an overview sentence; cosmetic, does not affect accuracy or function |

---

### Accuracy Checks (Critical — Project Core Value)

| Check | Result |
|-------|--------|
| Loop cycle is "Assemble→Call→Parse→Execute→Persist→Loop" | PASS |
| "perceive→decide→act" framing absent | PASS |
| Memory paths are `~/.hermes/memories/MEMORY.md` and `~/.hermes/memories/USER.md` | PASS |
| Wrong path `~/.hermes/MEMORY.md` absent | PASS |
| `session_search` described as internal agent tool (not slash command) | PASS |
| `agent.disabled_toolsets` mentioned ONLY as nonexistent (in warning callouts) | PASS |
| Nous Portal free-quota number NOT asserted | PASS |
| Modal/Daytona/Singularity flagged with "검증 필요" | PASS |
| "4,500x" benchmark NOT cited | PASS |
| FTS5 `messages_fts_trigram` CJK callout present in Ch.6 | PASS |
| No fabricated outputs — LOW/MEDIUM items use `[로컬 실행 후 캡처 필요]` placeholders | PASS |
| All command/config code blocks open with `# 검증: <component>, 2026-06-09` | PASS (34 occurrences across 5 files) |

---

### Build Verification

```
mdbook build
→ INFO Book building has started
→ INFO Running the html backend
→ INFO HTML book written to /Users/ohama/projs/hermes-tutorial/book
Exit code: 0
No "error" in stdout/stderr
```

All 5 chapter HTML pages rendered:
- `book/04-agent-loop/index.html` — EXISTS
- `book/05-context-files/index.html` — EXISTS
- `book/06-memory/index.html` — EXISTS
- `book/07-tools/index.html` — EXISTS
- `book/08-backends/index.html` — EXISTS

---

### Human Verification Required

The following items cannot be verified programmatically and require a human to run locally:

#### 1. 컨텍스트 파일 로드 배너 메시지

**Test:** `cd ~/my-project && cat > .hermes.md << 'EOF' ... EOF && hermes`
**Expected:** hermes가 시작 시 컨텍스트 파일 로드를 알리는 배너 또는 메시지를 표시하는지 확인
**Why human:** 배너 메시지 텍스트가 공식 문서화되어 있지 않으며 버전에 따라 다를 수 있음. 현재 챕터는 placeholder를 사용하고 있으나, 실제 출력으로 교체해야 할 시점에 사람이 확인 필요.

#### 2. Docker 첫 실행 출력

**Test:** `hermes config set terminal.backend docker && hermes` (Docker 데몬이 실행 중인 환경에서)
**Expected:** 컨테이너 이미지 풀/빌드 메시지의 정확한 형식 확인
**Why human:** 첫 Docker 백엔드 실행 시 출력은 이미지 캐시 상태, Docker 버전, 네트워크 상태에 따라 다름. 현재 placeholder 사용 중.

#### 3. hermes setup terminal 마법사 흐름

**Test:** `hermes setup terminal` 실행
**Expected:** 백엔드 선택 프롬프트와 설정 절차 확인
**Why human:** 마법사 대화 흐름이 백엔드별로 다르며 rolling release에 따라 변경될 수 있음.

#### 4. hermes tools --summary 출력 형식

**Test:** `hermes tools --summary` 실행
**Expected:** 현재 활성 툴 목록 출력 형식 확인
**Why human:** 출력 형식은 설치 능력과 활성 플랫폼에 따라 환경마다 다름.

---

## Summary

Phase 3 goal is fully achieved. All 13 must-haves are verified against the actual source files.

**Accuracy highlights:** All critical accuracy requirements from the project spec pass — the official loop cycle name is used correctly, memory file paths are correct (`~/.hermes/memories/` subdirectory, not the wrong `~/.hermes/` directly), session_search is correctly framed as an internal tool, agent.disabled_toolsets is documented as nonexistent, no Nous Portal free quota number is asserted, no 4,500x benchmark is cited, and all Modal/Daytona/Singularity content carries explicit "검증 필요" callouts.

**Build:** `mdbook build` exits 0 with no errors. All 5 chapter HTML pages render.

**Navigation chain:** Complete and correct — Ch.3→Ch.4→Ch.5→Ch.6→Ch.7→Ch.8, with Ch.8 using plain prose for the Phase 5 forward reference (no dead links).

**Minor cosmetic note:** "kategorie" at src/07-tools/index.md line 11 appears to be a typo (German/mixed word for "카테고리"). No impact on correctness or build.

---

_Verified: 2026-06-10T08:02:14Z_
_Verifier: Claude (gsd-verifier)_
