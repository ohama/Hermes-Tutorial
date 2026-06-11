---
phase: 06-reference
verified: 2026-06-11T10:30:00Z
status: passed
score: 11/11 must-haves verified
gaps: []
---

# Phase 6: 레퍼런스 Verification Report

**Phase Goal:** 독자가 CLI·슬래시 커맨드·설정 레퍼런스와 통합 트러블슈팅 색인을 언제든 조회할 수 있다.
**Verified:** 2026-06-11T10:30:00Z
**Status:** PASSED
**Re-verification:** No — initial verification

---

## Goal Achievement

### Observable Truths

| # | Truth | Status | Evidence |
|---|-------|--------|----------|
| 1 | SUMMARY.md에 '# 레퍼런스' 섹션과 3개 항목이 추가되고 총 21개 index.md 링크, 중복 없음, 16-* 없음 | VERIFIED | `grep -c "index.md" SUMMARY.md` = 21; no duplicates; no 16-* entries |
| 2 | Ch.19 CLI·슬래시 커맨드 레퍼런스가 mdBook에서 렌더링된다 | VERIFIED | book/19-cli-reference/index.html exists (43,690 bytes); mdbook build exit 0 |
| 3 | Ch.19가 'verified core + 검증 필요' 프레이밍 callout을 포함한다 | VERIFIED | Line 10: "검증된 명령어만 수록합니다"; 8+ "검증 필요" callouts throughout |
| 4 | Ch.19가 'hermes security audit는 존재하지 않음'을 정정 callout으로만 명시한다 | VERIFIED | Lines 237-246: correction section only; not present in any command list |
| 5 | Ch.19가 'hermes doctor = 공급망 어드바이저리, 설정 검증 = hermes config check'를 정확히 구분한다 | VERIFIED | Line 154: "공급망 어드바이저리 검사 (Python CVE — NOT 설정 검증)"; line 233: "설정 검증 = `hermes config check`" |
| 6 | Ch.20 config.yaml 레퍼런스가 mdBook에서 렌더링된다 | VERIFIED | book/20-config-reference/index.html exists (56,005 bytes); mdbook build exit 0 |
| 7 | approvals.timeout이 60과 300 양쪽을 제시하고 단정 없이 hermes config show 헤지가 달린다 | VERIFIED | Line 246: "03-RESEARCH: 60, 05-RESEARCH: 300"; line 261-263: "검증 필요" callout with "hermes config show" |
| 8 | agent.disabled_toolsets가 '존재하지 않음'으로만 등장하고 사용 가능한 키로 제시되지 않는다 | VERIFIED | Line 119: "존재하지 않습니다"; line 500: 미존재 키 표에서만 |
| 9 | security.redact_secrets가 자격증명 패턴 난독화로 기술되며 일반 PII 아님이 명시된다 | VERIFIED | Lines 293-300: "일반 PII 필터가 아닙니다"; "자격증명 패턴(ghp_.../sk-.../Bearer)" |
| 10 | Ch.21 마스터 트러블슈팅 색인이 mdBook에서 렌더링되며 Ctrl+F 검색 안내, 카테고리별 구조, 용어 사전을 포함한다 | VERIFIED | Line 9: Ctrl+F callout; 6 category sections; glossary at line 196; 19 unique chapter backward links |
| 11 | 모든 실행 명령/설정 코드블록이 '# 검증: hermes rolling, 2026-06-11' 주석으로 시작한다 | VERIFIED | Ch.19: 11 code blocks all start with comment; Ch.20: 17 code blocks all start with comment; Ch.21: 0 code blocks (tables/prose only — exempt per plan) |

**Score:** 11/11 truths verified

---

## Required Artifacts

| Artifact | Expected | Lines | Status | Details |
|----------|----------|-------|--------|---------|
| `src/SUMMARY.md` | 21 chapter links, no duplicates, no 16-*, '# 레퍼런스' section | 50 | VERIFIED | Exactly 21 index.md links; 3-entry 레퍼런스 section added; 0 duplicates; 0 16-* entries |
| `src/19-cli-reference/index.md` | Ch.19 CLI reference, min 180 lines | 320 | VERIFIED | 320 lines, fully substantive; categories A–J, slash commands, key bindings, 2 correction sections |
| `src/20-config-reference/index.md` | Ch.20 config reference, min 180 lines | 568 | VERIFIED | 568 lines, fully substantive; all 14 key domains, approvals conflict hedge, non-existent keys table |
| `src/21-troubleshooting/index.md` | Ch.21 troubleshooting index, min 180 lines | 230 | VERIFIED | 230 lines, fully substantive; 6 category sections, 19 unique backward links, glossary |

---

## Key Link Verification

| From | To | Via | Status | Details |
|------|----|-----|--------|---------|
| SUMMARY.md | 19-cli-reference/index.md, 20-config-reference/index.md, 21-troubleshooting/index.md | mdBook 레퍼런스 섹션 | WIRED | All 3 paths present once each; mdbook build exit 0 |
| Ch.19 | ../20-config-reference/index.md | "다음 단계" + top callout | WIRED | Line 6 (top callout) + line 320 (다음 단계) |
| Ch.20 | ../21-troubleshooting/index.md | "다음 단계" + top callout | WIRED | Line 7 (top callout) + line 568 (다음 단계) |
| Ch.21 | ../NN-name/index.md#흔한-오류--주의 (Ch.1–18) | 카테고리별 출처 링크 | WIRED | 17 unique source chapter backward links; all target files confirmed to exist |
| Ch.21 | ../19-cli-reference/index.md, ../20-config-reference/index.md | "다음 단계" + top callout | WIRED | Lines 7 + 229-230 |

---

## Accuracy / Documentation Checks

### Verification comment coverage
- Ch.19: 11 bash code blocks — all start with `# 검증: hermes rolling, 2026-06-11`. No JSON blocks present. Tables exempt.
- Ch.20: 17 YAML code blocks — all start with `# 검증: hermes rolling, 2026-06-11`. No JSON blocks. `.env` environment variable section is a markdown table (exempt).
- Ch.21: 0 code blocks (entire chapter is tables and prose per the plan's exemption rule).

### hermes security audit (must be ABSENT as usable command)
- Ch.19: appears only in correction section (lines 237, 239, 246). Not in any command category table or code block.
- Ch.20: not mentioned.
- Ch.21: appears only in correction context (lines 185, 190) as "존재하지 않는 명령어".
- PASS: Never presented as a usable command anywhere.

### agent.disabled_toolsets (must be ABSENT as real key)
- Ch.19: line 277 — explicitly states "존재하지 않으므로 사용하지 마십시오".
- Ch.20: line 119 correction callout + line 500 non-existent keys table only. Not present in any valid key section.
- Ch.21: line 87 — only in troubleshooting table as "존재하지 않는 키".
- PASS: Never presented as a usable config key.

### hermes doctor framing
- Ch.19: lines 154-155, 214-233 — "공급망 어드바이저리 검사기", "설정 검증 = hermes config check" explicitly.
- Ch.21: line 189, 213 — "공급망 검사기 (설정 검증이 아님)".
- PASS: Correct framing throughout.

### security.redact_secrets scope
- Ch.20: lines 277, 291-300 — scoped to "MCP 오류 메시지 자격증명 패턴" (ghp_/sk-/Bearer); explicitly "일반 PII 필터가 아닙니다".
- PASS: Correct scope.

### approvals.timeout conflict (REAL DATA CONFLICT — both values required)
- Ch.20 line 246: `timeout: 300 # 승인 대기 시간 (초) — 03-RESEARCH: 60, 05-RESEARCH: 300`
- "60" appears at line 246 (in comment) and line 558 (검증 필요 table header).
- "300" appears at line 246 (as YAML value with conflict annotation).
- Line 261-263: explicit "검증 필요" callout saying neither value should be asserted, with "hermes config show" instruction.
- PASS: Both values present; no single default asserted.

### SUMMARY.md 21-chapter link integrity
- Total `index.md` links: exactly 21.
- Duplicate paths: none.
- 16-* entries: none.
- Existing 18 entries preserved intact (00-intro through 18-security).
- 3 new entries added: 19-cli-reference, 20-config-reference, 21-troubleshooting.

### Ch.21 backward links to existing chapters
- 17 source chapter backward links (Ch.1–18, no Ch.16, with `#흔한-오류--주의` anchor).
- All 17 target files confirmed to exist on disk.
- 2 additional same-section links to Ch.19 and Ch.20 (correct — those files exist).
- No links to non-existent files (no ../16-* anywhere).

### mdbook build
- Exit code: 0.
- Output: no "error" or "unknown" strings; clean INFO messages only.
- All 3 chapter HTML pages rendered: 19/index.html (43 KB), 20/index.html (56 KB), 21/index.html (45 KB).

---

## Anti-Patterns Found

| File | Pattern | Severity | Impact |
|------|---------|----------|--------|
| None | — | — | — |

No placeholder content, TODO/FIXME comments, empty implementations, or stub patterns found in any of the 3 chapter files.

---

## Requirements Coverage

| Requirement | Status | Notes |
|-------------|--------|-------|
| REF-01: CLI·슬래시 커맨드 레퍼런스 조회 (Ch.19) | SATISFIED | 320-line Ch.19 with categories A–J, slash commands, key bindings |
| REF-01: config.yaml 레퍼런스 조회 (Ch.20) | SATISFIED | 568-line Ch.20 with 14 key domains, accuracy corrections |
| REF-01: 마스터 트러블슈팅 색인 조회 (Ch.21) | SATISFIED | 230-line Ch.21 with 6 categories, 17-chapter backward links, glossary |

---

## Human Verification

No human verification required. All checks are structural/content-based and verified programmatically.

---

_Verified: 2026-06-11T10:30:00Z_
_Verifier: Claude (gsd-verifier)_
