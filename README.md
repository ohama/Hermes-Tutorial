# Hermes Agent 한국어 튜토리얼

[Nous Research Hermes Agent](https://github.com/nousresearch/hermes-agent)를 위한 **개발자 대상 한국어 튜토리얼**입니다. 설치부터 핵심 개념·학습 루프·플랫폼 운영·레퍼런스까지 21개 챕터로 다루며, [mdBook](https://rust-lang.github.io/mdBook/)으로 빌드되어 GitHub Pages에 자동 배포됩니다.

📖 **라이브 사이트: <https://ohama.github.io/Hermes-Tutorial/>**

각 장은 개념 설명과 따라 하는 실습을 함께 제공합니다. 모든 명령어·설정·API는 실제 repo/공식 문서에 근거해 작성했으며, 로컬 실행이 필요한 출력은 발명하지 않고 `[로컬 실행 후 캡처 필요]` 자리표시자로 표기했습니다.

## 목차

**기초**
- 소개: Hermes란 무엇인가
- 설치 (macOS / Linux / WSL2 / Windows)
- 첫 실행
- 모델 설정

**핵심 개념**
- 에이전트 루프 · 컨텍스트 파일 · 메모리 · 툴 게이트웨이 · 터미널 백엔드

**학습과 자동화**
- 스킬 시스템 · 학습 루프 · MCP 연동 · 크론 스케줄러 · 서브에이전트

**플랫폼과 운영**
- 메시징 게이트웨이(Telegram) · 추가 게이트웨이(Discord/Slack 등) · 프로덕션 배포 · 보안 하드닝

**레퍼런스**
- CLI · 슬래시 커맨드 레퍼런스 · `config.yaml` 레퍼런스 · 마스터 트러블슈팅 색인

## 로컬에서 빌드하기

[mdBook](https://rust-lang.github.io/mdBook/) 0.5.3 이상이 필요합니다.

```bash
# mdBook 설치 (pre-built 바이너리 권장; 또는 cargo install mdbook)
mdbook --version

# 로컬 서버로 미리보기 (저장 시 자동 새로고침)
mdbook serve --open

# 정적 사이트 빌드 → book/ 디렉터리 생성
mdbook build
```

## 배포

`main` 브랜치에 push하면 GitHub Actions(`.github/workflows/deploy.yml`)가 mdBook을 빌드해 GitHub Pages에 자동 배포합니다. 별도 작업이 필요 없습니다.

## 프로젝트 구조

```
.
├── book.toml                 # mdBook 설정 (language="ko", site-url)
├── src/                      # 튜토리얼 본문
│   ├── SUMMARY.md            # 목차 (챕터 순서·구조)
│   ├── README.md             # 책 표지/소개
│   └── NN-name/index.md      # 각 챕터
├── .github/workflows/        # GitHub Pages 자동 배포 워크플로우
└── .planning/                # 제작 과정 기록 (GSD 워크플로우 산출물)
```

## 기여

오타·내용 오류·최신화 제안을 환영합니다. 각 페이지 우상단의 편집 링크로 바로 수정 제안을 보내거나 이슈/PR을 열어 주세요.

내용 작성 규칙:
- 모든 본문은 한국어, 코드·명령어는 원문 유지
- 명령어/설정 코드블록은 `# 검증: hermes rolling, YYYY-MM-DD` 주석으로 시작
- 확인되지 않은 명령·출력은 단정하지 말고 `> 검증 필요` 또는 `[로컬 실행 후 캡처 필요]`로 표기

## 라이선스 / 출처

- 튜토리얼 대상: [nousresearch/hermes-agent](https://github.com/nousresearch/hermes-agent) (MIT)
- 공식 문서: <https://hermes-agent.nousresearch.com>

이 튜토리얼은 비공식 학습 자료이며 Nous Research와 직접적인 제휴 관계가 없습니다.
