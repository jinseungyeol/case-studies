# 실무 프로젝트 케이스 스터디

실무에서 설계·구축한 시스템의 **아키텍처, 기술 선택 이유, 문제 해결 과정**을 재구성한 문서 모음입니다.

> ⚠️ **이 저장소는 회사 코드의 사본이 아닙니다.** 실제 코드·회사명·서비스명·내부 주소·실제 스키마·비밀값은 포함하지 않으며, 모든 내용은 일반화된 표현과 의사코드로 재작성했습니다. 코드 원문이 필요한 검증은 면접에서 화면 공유 등으로 별도 진행 가능합니다.

## 프로젝트

| 프로젝트 | 한 줄 요약 | 시작하기 |
|---|---|---|
| [Cafe24 멀티브랜드 모노레포](./cafe24-mall-monorepo/) | 13개 D2C 자사몰 스킨을 한 repo에서 버전 관리·자동배포·AI 처리 — 퍼블리셔 3인 분담을 1인 전담으로 | [README](./cafe24-mall-monorepo/README.md) |
| [커머스 데이터 파이프라인](./commerce-data-pipeline/) | 13개 브랜드 광고비·매출 자동 수집 → Sheets/BigQuery 적재 + 실패 자동진단 봇 | [README](./commerce-data-pipeline/README.md) |
| [Cafe24 WebP 최적화 앱](./cafe24-webp-optimizer/) | 본문 HTML을 건드리지 않는(clean-body) 상품 이미지 WebP 파이프라인 — 평균 84% 용량 절감 | [README](./cafe24-webp-optimizer/README.md) |
| [칼로리바](./caloriebar/) | 프랜차이즈 고객 홈페이지 + 지점 백오피스 (Django) — 홈쇼핑 다채널 신청 시스템 | [README](./caloriebar/README.md) |

각 프로젝트는 같은 구조입니다:

```
README.md               프로젝트 개요 (역할·성과·아키텍처 다이어그램)
docs/
├── architecture.md     시스템 아키텍처
├── tech-stack.md       기술스택과 선택 이유
├── main-features.md    주요 기능 설명
└── troubleshooting.md  기술적 문제 해결 사례
diagrams/               mermaid 다이어그램 소스
```

## 관통하는 설계 원칙

- **staging-first** — 모든 변경은 테스트 환경에 먼저, 운영 반영은 명시적 승인 게이트 뒤에
- **무소음 실패 금지** — 조용히 틀린 값이 들어가는 것이 예외로 죽는 것보다 위험하다. 실패는 반드시 표면화한다
- **오판의 무해화** — 판정 정확도를 높이기보다, 오판해도 피해가 없는 구조(비대칭 기준·기계 가드·구조적 격리)를 먼저 만든다
- **단일 진실(single source of truth)** — 사본은 낡는다. 설정·번호·절차는 한 곳에만 두고 실행 시점에 읽는다

## 연결

- 포트폴리오: https://jinseungyeol.github.io
