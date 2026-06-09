# 소개 및 배경

Hermes Agent 한국어 튜토리얼에 오신 것을 환영합니다.

이 튜토리얼은 Nous Research의 Hermes Agent를 한국어로 소개합니다. 모든 명령어와 API 예시는 공식 소스에 근거하며, 각 코드블록은 검증 버전과 검토일을 주석으로 표기합니다.

## 코드블록 검증 주석 관례

이 튜토리얼의 모든 코드블록은 첫 줄에 다음 형식의 주석을 답니다:

```text
# 검증: <컴포넌트> <버전>, YYYY-MM-DD
```

예를 들어 Hermes 설치 명령은 다음과 같이 표기합니다:

```bash
# 검증: hermes rolling (설치 시점 최신), 2026-06-09
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

설치 후 버전을 확인합니다:

```bash
# 검증: hermes rolling, 2026-06-09
hermes version
```

인프라 도구 명령도 동일한 관례를 따릅니다:

```bash
# 검증: mdBook 0.5.3, 2026-06-09
mdbook build
```
