# 실무 프로젝트 케이스 스터디

실무에서 설계·구축한 시스템의 **아키텍처, 기술 선택 이유, 문제 해결 과정**을 재구성한 문서 모음입니다.

> ⚠️ **이 저장소는 회사 코드의 사본이 아닙니다.** 실제 코드·회사명·서비스명·내부 주소·실제 스키마·비밀값은 포함하지 않으며, 모든 내용은 일반화된 표현과 의사코드로 재작성했습니다. 코드 원문이 필요한 검증은 면접에서 화면 공유 등으로 별도 진행 가능합니다.

## 프로젝트

| 프로젝트 | 한 줄 요약 | 시작하기 |
|---|---|---|
| [Cafe24 멀티브랜드 모노레포](./cafe24-mall-monorepo/) | 13개 D2C 자사몰 스킨을 한 repo에서 버전 관리·자동배포 — Slack 요청 접수→AI 처리→운영 반영까지 자동화, 퍼블리셔 3인 분담을 1인 전담으로 | [README](./cafe24-mall-monorepo/README.md) |
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

특정 프로젝트 하나의 원칙이 아니라, 네 프로젝트에 반복해서 나타나는 공통 원칙입니다. 원칙마다 실제로 구현된 프로젝트와 그 형태를 함께 적었습니다.

- **staging-first** — 모든 변경은 테스트 환경에 먼저, 운영 반영은 명시적 승인 게이트 뒤에
  - [모노레포](./cafe24-mall-monorepo/) : push는 staging 스킨까지만 자동배포, 운영 반영은 명시적 promote + 운영 파일 hash 대조 게이트 뒤에
  - [칼로리바](./caloriebar/) : 로컬 개발은 환경 변수 하나로 빈 SQLite 구동 — 운영 DB 오염 원천 차단
- **무소음 실패 금지** — 조용히 틀린 값이 들어가는 것이 예외로 죽는 것보다 위험하다. 실패는 반드시 표면화한다
  - [데이터 파이프라인](./commerce-data-pipeline/) : 빈 표·스테일 값 등 "성공처럼 보이는 실패"를 다층 검증으로 차단, 실행 실패·적재값 이상은 Slack DM으로 표면화
  - [WebP 앱](./cafe24-webp-optimizer/) : 본문↔장부 드리프트를 매일 스캔으로, 스니펫 소실을 라이브 워치독으로 감지
- **오판의 무해화** — 판정 정확도를 높이기보다, 오판해도 피해가 없는 구조(비대칭 기준·기계 가드·구조적 격리)를 먼저 만든다
  - [모노레포](./cafe24-mall-monorepo/)의 Slack 에이전트 : 애매하면 수동 전환(비대칭 기준), diff 상한 초과 시 AI 판단과 무관하게 강제 수동(기계 가드), 오판해도 staging까지만 도달(구조적 격리)
  - [WebP 앱](./cafe24-webp-optimizer/) : 본문 write 경로 자체가 없는 clean-body 구조 — 시스템이 어떻게 오동작해도 본문 HTML은 불가침
  - [데이터 파이프라인](./commerce-data-pipeline/)의 자동진단 봇 : 격리 환경에서 조사만 하고 수정안은 DM 제안으로 — 오진해도 파이프라인·데이터를 직접 건드리지 않음
- **단일 진실(single source of truth)** — 사본은 낡는다. 설정·번호·절차는 한 곳에만 두고 실행 시점에 읽는다
  - [모노레포](./cafe24-mall-monorepo/) : 스킨 번호·recipe·Figma 매핑은 브랜드별 `mapping.yaml` 한 곳, 유효한 절차·확정값은 CURRENT.md 한 곳 — 문서에 사본 복제 금지
  - [WebP 앱](./cafe24-webp-optimizer/) : 서빙 여부는 장부 DB 상태 하나로 판정 — 롤백·재최적화가 DB 상태 변경만으로 발효
  - [칼로리바](./caloriebar/) : 신청 가능 차수는 허용 리스트 한 줄로만 제어

## 연결

- 포트폴리오: https://jinseungyeol.github.io
