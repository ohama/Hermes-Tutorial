# Hermes Agent 한국어 튜토리얼

## What This Is

Nous Research의 오픈소스 AI 에이전트 프레임워크 [Hermes Agent](https://github.com/nousresearch/hermes-agent)를 위한 **개발자 대상 한국어 튜토리얼**이다. mdBook으로 작성되어 GitHub Pages로 자동 배포되는, 설치부터 고급 기능·배포까지 다루는 전체 종합 가이드다. 각 장은 개념 설명과 따라하는 실습을 함께 제공한다.

## Core Value

개발자가 Hermes Agent를 처음부터 설치·실행하고, 스킬·게이트웨이·배포까지 실제로 동작시킬 수 있게 만드는 것. 모든 명령어·API는 실제 repo/공식 문서에 근거해 정확해야 한다.

## Requirements

### Validated

<!-- Shipped and confirmed in v1. -->

- ✓ 개발자가 따라 하면 Hermes를 설치하고 첫 대화를 실행할 수 있다 — v1 (Ch.1–2)
- ✓ 모델 선택·전환(OpenRouter, Nous Portal 등)을 설명하고 실습할 수 있다 — v1 (Ch.3)
- ✓ 스킬(Skills) 시스템 — 자동 생성·자가 개선·agentskills.io 표준 — 을 다룬다 — v1 (Ch.9–10)
- ✓ 컨텍스트 파일, 툴 게이트웨이, 압축(compression), 퍼스널리티, 승인 모드를 다룬다 — v1 (Ch.5–7)
- ✓ 메시징 게이트웨이(Telegram/Discord/Slack/WhatsApp/Signal/Email) 설정을 다룬다 — v1 (Ch.14–15)
- ✓ cron 스케줄러와 서브에이전트(병렬 작업)를 다룬다 — v1 (Ch.12–13)
- ✓ 배포(local/Docker/SSH/Singularity/Modal/Daytona)와 서버리스 운영을 다룬다 — v1 (Ch.8, Ch.17)
- ✓ 각 장이 개념 설명 + 따라하는 실습(혼합형) 구조를 따른다 — v1
- ✓ mdBook으로 빌드되어 GitHub Pages로 자동 배포된다 — v1 (Phase 1)
- ✓ MCP 양방향 연동, 보안 하드닝, CLI/config/트러블슈팅 레퍼런스 — v1 (Ch.11, Ch.18, Ch.19–21)

### Active

<!-- Next milestone scope — defined via /gsd:new-milestone -->

(None yet — v1 shipped; define next milestone with /gsd:new-milestone)

### Out of Scope

- Hermes Agent 자체 소스코드 수정/기여 가이드 — 사용 튜토리얼이지 컨트리뷰션 가이드가 아님
- 영어 버전 — 한국어 단일 언어로 시작
- 동영상/스크린캐스트 — 텍스트 기반 mdBook에 집중
- OpenClaw(전신) 마이그레이션 상세 — 신규 사용자 대상이므로 우선순위 낮음

## Current State

**v1 SHIPPED (2026-06-11):** 21개 챕터 한국어 mdBook 튜토리얼이 https://ohama.github.io/Hermes-Tutorial/ 에 라이브. 레포 `ohama/Hermes-Tutorial`, push to main 시 GitHub Actions가 자동 빌드·배포. ~5,610 lines Korean markdown across 6 phases / 24 plans.

**v2 후보(아직 미확정):** 영어 버전 동시 제공(LOC-01), Honcho 심화(ADV-01), API 서버/ACP·플러그인·음성 전사 심화(ADV-02~04), KakaoTalk 등 한국 특화 게이트웨이(LOC-02). 모두 v1에서 v2로 연기됨.

**알려진 잔여 tech debt:** (1) 로컬 Hermes 실행이 필요한 출력 캡처 placeholder들(`hermes doctor`/`gateway setup`/배너/`dashboard` 등) — 실제 캡처로 교체 필요. (2) 페이즈별 `# 검증:` 주석 날짜 차이(06-09/06-10/06-11, cosmetic). (3) Node.js 20 Actions deprecation(2026-06-16부터 Node 24 강제, 동작에는 영향 없음).

## Context

- **원본:** https://github.com/nousresearch/hermes-agent (MIT, Python 84% / TypeScript 12%)
- **공식 문서/설치:** hermes-agent.nousresearch.com (install.sh / install.ps1)
- **주요 개념:** Skills(절차적 기억, 자가 개선), Context Files, Tool Gateway, Compression, Personalities, Approval Mode, 학습 루프(Honcho dialectic 기반 user modeling)
- **CLI 진입점 예:** `hermes`, `hermes model`, `hermes tools`, `hermes gateway`, `hermes setup`
- **정확성 원칙:** 명령어·API·옵션은 추측하지 않고 실제 repo/공식 문서를 조사해 반영한다(research 단계에서 검증).

## Constraints

- **Tooling**: mdBook — 빌드 가능한 책 구조(SUMMARY.md 기반 챕터)로 구성
- **Language**: 한국어 — 모든 본문 한국어, 코드/명령어는 원문 유지
- **Deployment**: GitHub Pages — CI로 자동 빌드·배포
- **Accuracy**: 모든 기술적 내용은 공식 소스 근거 필수(추측 금지)
- **Audience**: 개발자 — Python/CLI에 익숙하다고 가정, AI 에이전트 개념은 설명

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| mdBook 형식 | 다장 구성·GitHub Pages 배포·코드 친화적 | ✓ Good — 0.5.3로 21챕터 안정 빌드·배포 |
| 한국어 단일 언어 | 대상 독자가 한국어 개발자 | ✓ Good — 영어판은 v2(LOC-01)로 연기 |
| 전체 종합 범위 | 핵심부터 배포까지 한 권으로 완결 | ✓ Good — Ch.0–21 완결 |
| 혼합형(개념+실습) | 개발자가 이해하고 바로 따라 할 수 있게 | ✓ Good |
| 공식 소스 기반 정확성 | 명령어·API 오류 방지, 신뢰성 확보 | ✓ Good — 리서치가 다수 오류를 작성 전에 정정, 감사가 교차페이즈 모순까지 포착 |
| GitHub Pages 자동 배포 | 공개 사이트로 손쉽게 운영 | ✓ Good — push 시 자동 배포 동작 |
| SUMMARY.md owner-plan 패턴 | Wave 2 병렬 챕터 작성 시 SUMMARY 쓰기 충돌 방지 | ✓ Good — 5개 페이즈에서 무충돌 병렬 실행 |
| 검증 불가 출력 = placeholder | 발명 금지(정확성 원칙) | ✓ Good — 단, 로컬 캡처 교체는 잔여 tech debt |

---
*Last updated: 2026-06-11 after v1 milestone*
