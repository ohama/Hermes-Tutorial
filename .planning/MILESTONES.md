# Project Milestones: Hermes Agent 한국어 튜토리얼

## v1 MVP (Shipped: 2026-06-11)

**Delivered:** Nous Research Hermes Agent의 개발자 대상 한국어 mdBook 튜토리얼 — 설치부터 핵심 개념·학습 루프·플랫폼 운영·레퍼런스까지 21개 챕터, GitHub Pages 자동 배포.

**Phases completed:** 1-6 (24 plans)

**Key accomplishments:**

- mdBook 0.5.3 + GitHub Actions(`deploy-pages@v5`) 자동 배포 파이프라인 구축 — push 시 라이브 반영 (https://ohama.github.io/Hermes-Tutorial/)
- 21개 한국어 챕터 작성: 기초(설치·첫실행·모델) → 핵심개념(에이전트루프·컨텍스트·메모리·툴·백엔드) → 학습/자동화(스킬·학습루프·MCP·크론·서브에이전트) → 플랫폼/운영(게이트웨이·배포·보안) → 레퍼런스(CLI·config·트러블슈팅)
- "공식 소스 기반 정확성" 원칙 관철 — 매 페이즈 실제 repo/문서 리서치로 다수의 잘못된 가정을 작성 전에 정정(에이전트 루프 표현, 메모리 경로, 큐레이터 트리거, `hermes cron` vs `gateway`, MCP/ACP 분리, `hermes doctor`=공급망 CVE 어드바이저리, `hermes security audit` 미존재, PII=`redact_secrets` 한정, Discord 2-Intents)
- 검증 불가 항목은 발명 대신 `[로컬 실행 후 캡처 필요]` placeholder + `> 검증 필요` 콜아웃으로 처리; 모든 토큰은 placeholder + `.env` 보안 콜아웃
- 마일스톤 감사에서 교차 페이즈 정확성 모순(Ch.1의 `hermes doctor` 표현)과 4개 내비게이션 갭을 발견·수정 후 재배포

**Stats:**

- 21 챕터, ~5,610 lines Korean markdown
- 6 phases, 24 plans, 82 commits
- 7 days (2026-06-05 → 2026-06-11)

**Git range:** `feat(01-01)` → `fix(audit)` (tag: milestone-v1)

**What's next:** (미정 — `/gsd:new-milestone`에서 정의)

---
