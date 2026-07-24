# 주요 기능

## 1. push 자동배포 (브랜드별 ×13)

`brands/{brand}/skin/**`이 바뀌면 해당 브랜드의 배포 워크플로만 트리거되어 staging 스킨에 올라간다.

- **incremental** — 직전 커밋과의 diff만 전송, 파일 몇 개 변경은 수 초 안에 완료
- **경로 필터** — 다른 브랜드·설계 문서 변경은 배포를 트리거하지 않음 (브랜드 단위 격리)
- 전체 업로드는 자동 전환되지 않고 명시적 `mode=full` 입력을 요구

## 2. staging-first + 명시적 promote

운영 반영은 사람이 promote 워크플로를 직접 실행할 때만 일어난다. 실행 시:

1. 성능 스니펫 게이트 — 온보딩된 브랜드의 스니펫 누락 검사
2. **M-smart 검증** — 영향 파일을 운영에서 내려받아 hash 대조. admin에서 직접 수정된 파일이 있으면 배포를 중단해 덮어쓰기 사고를 막음
3. incremental 업로드 후 `prod-{brand}-{date}-{sha}` 태그 — 다음 promote의 diff 기준점

## 3. 자연어 요청의 AI 처리 (Claude Code 스킬 ×13 + recipe)

반복되는 자연어 요청("메인 배너 링크 바꿔줘")을 AI가 처리하는 체계.

- **스킬** — 브랜드별로 처리 절차를 문서로 정의: staging-first 규칙, 검증 URL 패턴, recipe 참조·학습 규칙
- **recipe** — 처리한 요청을 `자연어 키워드 ↔ 파일:라인` 매핑으로 축적. 다음 같은 요청은 탐색 없이 정확한 위치로 직행
- **code/admin 구분** — recipe에 `source: code|admin` 필드를 둬서, admin 설정 영역 요청은 코드를 건드리지 않고 "관리자 페이지 경로 안내"로 처리

```yaml
# recipe 개념 예시 (실제 값 아님)
change_recipes:
  - natural_language: ["오늘출발 숨겨줘", "당일배송 표기 제외"]
    source: code
    file: skin/js/product-badges.js
    method: "숨김 대상 상품번호 배열에 추가"
  - natural_language: ["대표 전화번호 변경"]
    source: admin
    guide: "관리자 → 상점 정보 → 연락처에서 수정 (코드 수정 아님)"
```

## 4. Slack 접수 에이전트 파이프라인

타팀의 수정 요청이 "DM → 담당자 복붙 → 처리 → 회신"으로 흐르던 중간다리를 제거하려는 자동화. 접수 폼 → AI 헤드리스 처리 → staging 반영 → 결과 회신 → 요청자 확정 시 자동 운영 배포까지의 흐름을 설계하고 MVP 엔진 코드를 작성했다(설계·엔진 완성 / Slack 연동 활성화 대기). 상세는 [slack-agent-pipeline.md](./slack-agent-pipeline.md).

## 5. 디자인 시스템 동기화 (h2f 파이프라인)

- 자체 도구 **h2f**가 라이브 페이지를 노드 단위로 직렬화해 Figma 플러그인으로 충실히 재구성 — 레퍼런스를 "실물 화면" 기준으로 확보
- 브랜드마다 Foundation 토큰 → Atoms → Organisms → 참조 페이지 swap 순서로 구축해, Figma 컴포넌트를 고치면 참조 화면이 함께 바뀌는 단일 소스를 만듦

## 6. 성능 스니펫 단일 소스 주입

[WebP 최적화 앱](../../cafe24-webp-optimizer/)의 head 스니펫을 템플릿 하나에서 온보딩된 브랜드 레이아웃에 스크립트로 일괄 주입. promote 게이트가 누락을 차단해 "스킨 리뉴얼로 스니펫이 조용히 사라지는" 사고를 막는다.

## 7. Cafe24 전용 가드 스크립트

| 가드 | 막는 문제 |
|---|---|
| 대소문자 충돌 검사 | default 스킨이 같은 파일을 두 케이스로 배포하는 phantom 파일 (macOS에선 보이지도 않음) |
| drift 검사 | mapping.yaml과 실제 코드의 어긋남 |
| dead partial 검사 | 어디서도 참조되지 않는 조각 파일 |
| 스니펫 검사 | 성능 스니펫 누락 |
