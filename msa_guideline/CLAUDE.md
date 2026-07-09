# CLAUDE.md — {{SYSTEM_NAME}} (MSA)

다수 서비스로 구성된 MSA 시스템(또는 그 안의 개별 서비스) 루트에 놓이는 Claude Code 작업 지침 템플릿입니다. 언어 독립적이며, 각 서비스의 언어별 규약(`golang_guideline/`, `python_guideline/`, `cpp_guideline/`)과 **함께** 사용합니다. 심층 아키텍처 규칙은 `MSA_ARCHITECTURE.md`, ISO 문서 규약은 `ISO_MSA_CONVENTION.md`로 위임합니다. 아래 `{{PLACEHOLDER}}`를 실제 값으로 채워 사용합니다.

## 전역 지침 상속

- 본 문서는 `global_guideline/CLAUDE.md`(사용자 전역 지침)를 상속하며, 상충 시 시스템/서비스 루트의 본 문서가 우선합니다.
- 한국어·존댓말로 응답·작성하고, 불필요한 서론/요약은 생략합니다.
- 임의로 커밋·PR을 생성하지 않습니다. 커밋·PR은 사용자가 직접 수행하며, 사용자가 명시적으로 요청한 경우에만 대행합니다.
- 모든 분석·설계는 ISO 기반 문서로 작성하고 적합성·추적성을 확보합니다. 추적성의 우선순위는 0티어입니다.
- 문서는 프로젝트 하위 `docs/` 폴더에 저장하고, `docs/` 경로는 `.gitignore`에 추가하여 커밋되지 않도록 관리합니다.
- 추측하지 않습니다. 모르면 검색·확인 후 답변하고, 지시가 불명확하면 되물어봅니다.
- 컴파일/타입체크가 가능한 언어로 작성된 서비스는, 작업 완료 전 반드시 해당 언어 guideline의 빌드·검사를 통과(에러 0)시킵니다.

## 이 템플릿 사용법

`msa_guideline`의 3파일은 다음 위치에 둡니다.

1. `MSA_ARCHITECTURE.md`(아키텍처 심층)와 `ISO_MSA_CONVENTION.md`(ISO 문서 규약)는 **시스템 저장소**의 `docs/` 또는 `docs/architecture/`로 복사합니다.
2. 본 `CLAUDE.md`는 **시스템 루트**(monorepo) 또는 **각 서비스 루트**(polyrepo)로 복사합니다.
3. 각 서비스는 자신의 언어에 해당하는 guideline(`golang_guideline/` 등)의 `CLAUDE.md`·`CODING_CONVENTION.md`·`ISO_DOCUMENTS_CONVENTION.md`를 **함께** 서비스 `docs/`로 복사합니다. 본 문서는 서비스 간 관심사(경계·통신·데이터·운영)를, 언어 guideline은 서비스 내부 구현 규약을 담당합니다.
4. 파일 내 `{{PLACEHOLDER}}`(예: `{{SYSTEM_NAME}}`, `{{SERVICE_NAME}}`, `{{SERVICE_LIST}}`, `{{GATEWAY}}`, `{{BROKER}}`, `{{REGISTRY}}`, `{{TRACING_BACKEND}}`, `{{ORCHESTRATOR}}`, `{{DB}}`)를 프로젝트 값으로 채웁니다.
5. 서비스 구성·통신 방식·인프라가 실제와 다르면 해당 섹션을 조정합니다.

## 시스템 개요

- 시스템 이름: {{SYSTEM_NAME}}
- 서비스 목록: {{SERVICE_LIST}}
- API gateway / BFF: {{GATEWAY}}
- 메시지 브로커: {{BROKER}}
- 서비스별 DB: 각 서비스는 자신의 {{DB}}를 소유(database-per-service)
- service registry / discovery: {{REGISTRY}}
- 오케스트레이션: {{ORCHESTRATOR}}
- distributed tracing backend: {{TRACING_BACKEND}}

외부 클라이언트는 gateway를 통해서만 진입하고, 서비스는 동기(REST/gRPC) 또는 비동기(broker 이벤트)로 통신하며, 각 서비스는 자기 DB만 소유합니다.

```text
                          ┌───────────────────────────────┐
        Client ─────────▶ │  {{GATEWAY}} (인증·라우팅·집계)  │
                          └───────────────┬───────────────┘
                                          │  sync: REST/gRPC
                    ┌─────────────────────┼─────────────────────┐
                    ▼                     ▼                     ▼
             ┌────────────┐        ┌────────────┐        ┌────────────┐
             │  Service A │        │  Service B │        │  Service C │
             └─────┬──────┘        └─────┬──────┘        └─────┬──────┘
                   │  owns               │  owns               │  owns
                   ▼                     ▼                     ▼
                [DB A]                [DB B]                [DB C]
                   │                     │                     │
                   └───────── async: publish/subscribe ────────┘
                                          │
                                 ┌────────▼────────┐
                                 │   {{BROKER}}    │  (이벤트·명령 전파)
                                 └─────────────────┘
```

## 레포 전략

- monorepo(단일 저장소에 다수 서비스)와 polyrepo(서비스별 저장소) 둘 다 유효합니다.
- 권장 기본값: 초기·소규모는 **monorepo**(원자적 변경·리팩터링 용이)로 시작하고, 팀 경계와 소유권이 갈라지면 **polyrepo**로 분리합니다.
- 어느 쪽이든 서비스 간 코드는 계약(API/이벤트 스키마)으로만 결합하고, 다른 서비스의 내부 구현을 직접 import하지 않습니다.

## 아키텍처 채택 원칙

- 기본 스탠스는 **modular-monolith-first**입니다. 새 시스템은 경계가 명확한 모듈러 모놀리스로 시작하되, 처음부터 무중단 추출이 가능하도록 bounded context·데이터 소유권·계약(contract)을 설계합니다.
- 단지 "코드가 커져서"는 서비스 분리 사유가 아닙니다. 아래 기준 중 **다수가 참일 때** 서비스로 추출합니다.
  1. 독립 배포·릴리스 주기가 실제로 필요합니다.
  2. 팀 경계와 소유권이 분리됩니다.
  3. 부하/확장 특성이 크게 다릅니다.
  4. 장애 격리가 필요합니다.
  5. 기술 스택이 달라야 합니다.
- 상세 판단 기준·분해 방법(DDD bounded context)은 `MSA_ARCHITECTURE.md`를 참조합니다.

## 서비스 공통 규칙 요약

상세는 `MSA_ARCHITECTURE.md`를 참조합니다. 아래는 최소 요약입니다.

- **database-per-service**: 서비스는 자기 데이터만 소유하고, 다른 서비스 DB에 직접 접근하지 않습니다(공유 DB 금지). 조회는 API/이벤트로만.
- **계약 버전 관리**: API/이벤트 스키마는 버전 관리하고, consumer-driven contract test로 호환성을 지킵니다.
- **correlation/trace ID 전파**: 요청마다 correlation ID/trace ID를 부여하고 서비스 경계를 넘어 전파(context propagation)합니다.
- **health/readiness**: 각 서비스는 liveness(`/healthz`)·readiness(`/readyz`)를 노출하고 종료 시 graceful shutdown을 구현합니다.
- **모든 원격 호출에 timeout**: 재시도는 exponential backoff + jitter로 제한하고, 반복 실패에는 circuit breaker를 적용합니다.
- **idempotent 소비자**: 메시지/이벤트 소비자와 재시도 대상 API는 idempotent해야 합니다(idempotency key 또는 dedup).
- **독립 배포**: 각 서비스는 독립적으로 빌드·테스트·배포되며, 서비스별 컨테이너 이미지와 파이프라인을 갖습니다.

## ISO 문서

- ISO 문서 규약은 `ISO_MSA_CONVENTION.md`를 참조합니다.
- MSA는 **시스템 레벨** 문서 세트(시스템 SRS·전체 아키텍처·서비스 카탈로그·계약/이벤트 카탈로그·시스템 추적성)와 **서비스 레벨** 문서 세트(서비스별 SRS·설계·API·테스트·추적성)를 함께 유지합니다.
- 서비스 추가·경계 변경·계약 변경 시 시스템 레벨과 서비스 레벨 문서를 함께 갱신하여 코드-문서 정합성을 유지합니다. 상세 구성·경로는 `ISO_MSA_CONVENTION.md`를 참조합니다.

## 가드레일

- 임의로 커밋·PR을 생성하지 않습니다(사용자가 직접 수행).
- 추측하지 않습니다. 불확실한 API·문법·인프라 설정은 검색·확인 후 사용하고, 지시가 불명확하면 되물어봅니다.
- 각 서비스는 자기 언어 guideline의 빌드/검사 게이트(예: Go `go vet`·`golangci-lint`·`go test -race`, Python `ruff`·`mypy`·`pytest`)를 모두 통과해야 합니다.
- 계약(API/이벤트 스키마) 변경 시 consumer-driven contract test를 갱신·통과시키고, 관련 ISO 문서(계약 카탈로그·추적성)를 함께 갱신합니다.
