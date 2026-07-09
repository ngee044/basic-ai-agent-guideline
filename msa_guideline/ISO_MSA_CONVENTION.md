# ISO 문서 규약 (MSA — 다수 서비스 시스템)

여러 서비스로 이뤄진 시스템({{SYSTEM_NAME}})의 산출물(요구사항·아키텍처·설계·계약·테스트·추적성 등)을 ISO/IEC/IEEE 표준에 맞춰 작성·관리하기 위한 규약입니다. 언어별 ISO 문서 규약(`golang_guideline/ISO_DOCUMENTS_CONVENTION.md` 등)이 "단일 서비스 내부"의 문서 골격을 규정한다면, 본 규약은 그 위에 **시스템 레벨**(서비스들의 집합)과 **서비스 간 계약·추적성** 관심사를 더합니다. 아키텍처 심층 원칙은 반복 서술하지 않고 상세는 `msa_guideline/MSA_ARCHITECTURE.md`를 참조합니다.

> 본 규약은 템플릿입니다. `{{PLACEHOLDER}}`(예: `{{SYSTEM_NAME}}`, `{{SERVICE_NAME}}`, `{{SERVICE_LIST}}`, `{{TEAM}}`(소유팀), `{{BROKER}}`, `{{GATEWAY}}`, `{{REGISTRY}}`, `{{TRACING_BACKEND}}`, `{{ORCHESTRATOR}}`, `{{DB}}`)를 프로젝트 값으로 치환하고, 규약 자체(표준 매핑·ID 체계·목차·추적성 형식)는 그대로 사용합니다.

> **전역 지침 정합**: 한국어·존댓말 작성, 불필요한 서론/요약 생략, 임의 커밋/PR 금지(사용자가 직접 수행 — 본 문서는 커밋/주석의 *형식*만 규정), 모든 분석은 ISO 기반 문서로 작성하여 적합성·추적성 확보, 문서는 프로젝트 `docs/`에 저장하고 `docs/` 경로는 `.gitignore`에 추가, 추적성 우선순위는 0티어, 컴파일/타입체크 가능한 언어는 반드시 빌드·검사 통과(에러 0).

## 목적과 적용 표준

- 목적: 시스템 전체 요구사항부터 **서비스 → (컴포넌트) → API/이벤트 계약 → 테스트**까지의 **양방향 추적성(traceability)** 을 서비스 경계를 넘어 확보하고, 문서와 실제 배포된 서비스 토폴로지의 정합성을 지속 검증하는 것입니다. 추적성은 최우선(0티어) 관심사입니다.
- MSA에서는 서비스들의 집합이 하나의 **시스템**을 이루므로, 소프트웨어 레벨 프로세스(12207)만으로는 부족하며 **시스템 레벨 프로세스(15288)** 가 나란히 요구됩니다. 따라서 본 규약은 15288을 12207과 함께 1티어 표준으로 격상합니다.
- 적용 표준(각 표준이 규정하는 바):

| 표준 | 규정 대상 | 본 규약에서의 근거 문서 | 티어 |
| --- | --- | --- | --- |
| **ISO/IEC/IEEE 15288** | 시스템 생명주기 프로세스. 서비스들의 집합인 "시스템" 레벨의 요구·아키텍처·통합·검증·운영 프로세스 프레임 | System StRS/SRS, System SAD, 서비스 카탈로그, 시스템 통합/E2E 테스트 | **1티어** |
| **ISO/IEC/IEEE 12207** | 소프트웨어 생명주기 프로세스. 개별 서비스(소프트웨어 단위)의 요구·설계·구현·검증·유지보수 프로세스 프레임 | 서비스별 SRS/설계/테스트 | **1티어** |
| **ISO/IEC/IEEE 29148** | 요구공학. 요구사항 작성·품질 기준·SRS/StRS 구조·요구사항 속성(ID·우선순위·검증방법) | System StRS/SRS, 서비스별 SRS | 근거 |
| **ISO/IEC/IEEE 42010** | 아키텍처 기술. 이해관계자·관심사·관점·뷰 구조. 본 규약은 이를 **C4 model**(Context/Container/Component/Code)로 실무화 | System SAD, 서비스별 설계서 | 근거 |
| **ISO/IEC 25010** | SQuaRE 제품 품질 모델. 8개 품질 특성. 본 규약은 분산 관점(신뢰성·성능 지연 예산·상호운용성·서비스 간 보안)으로 보강 | System 품질 모델, 서비스별 NFR | 근거 |
| **ISO/IEC/IEEE 42030** | 아키텍처 평가(evaluation). 서비스 분리/병합 등 구조 결정의 평가 근거(맥락) | ADR, 서비스 분리 결정 기록 | 맥락 |
| **ISO/IEC 27001** | 정보보안 경영시스템(ISMS). 서비스 간 인증(mTLS/JWT 전파)·시크릿 관리·감사로그 요구사항의 근거(맥락) | System 보안 문서, 서비스별 보안 요구사항 | 맥락 |
| **ISO 9001** | 품질경영시스템(QMS). 문서 관리·리뷰·변경 관리 등 프로세스 품질(맥락) | 문서 이력·상태 관리 | 맥락 |

## docs/ 폴더 구조

- 모든 ISO 문서는 프로젝트 루트의 `docs/` 하위에 저장합니다. 전역 지침에 따라 `docs/`는 `.gitignore`에 추가하여 커밋되지 않도록 관리합니다.
- MSA 문서는 **시스템 레벨(system/)** 과 **서비스 레벨(services/<service>/)** 로 이원화합니다. 시스템 레벨은 서비스들의 집합을 하나의 시스템으로 기술하고, 서비스 레벨은 각 서비스 내부를 언어별 규약대로 기술합니다.

```text
docs/
├── system/                         # 시스템 레벨 문서(15288): 서비스들의 집합을 하나의 시스템으로 기술
│   ├── requirements/               # System StRS(이해관계자 요구)·System SRS(시스템 기능/비기능)
│   ├── architecture/               # System SAD(42010/C4): Context·Container 뷰, 브로커/게이트웨이 토폴로지
│   ├── service-catalog/            # 서비스 카탈로그(SVC-CAT): 시스템 추적성의 허브
│   ├── contracts/                  # 시스템 차원 계약 개요: 동기 API 목록·이벤트 카탈로그·스키마 호환성 정책
│   ├── traceability/               # 분산 추적성 매트릭스(요구사항↔서비스↔계약↔테스트)
│   ├── quality/                    # System 품질 모델(25010 분산 관점), 지연 예산·SLO
│   ├── security/                   # 서비스 간 인증(mTLS/JWT)·시크릿·zero-trust 정책, 위협 모델
│   ├── operations/                 # 배포 토폴로지({{ORCHESTRATOR}}), 런북, 관측성(대시보드/추적), 롤백
│   └── adr/                        # 시스템 차원 결정(ADR-SYS-001...): 서비스 분리/병합, 통신 방식 선택
├── services/                       # 서비스 레벨 문서: 서비스별 1 디렉터리(= 1 bounded context)
│   ├── {{SERVICE_NAME}}/
│   │   ├── requirements/           # 서비스 SRS(29148)
│   │   ├── design/                 # 서비스 상세 설계(계층: handler→service→repository)
│   │   ├── contracts/              # 이 서비스가 소유·발행하는 계약
│   │   │   ├── openapi.yaml      #   REST 계약(OpenAPI)
│   │   │   ├── proto/            #   gRPC 계약(.proto)
│   │   │   └── asyncapi.yaml     #   비동기 이벤트 계약(AsyncAPI) / 발행·구독 이벤트
│   │   ├── test/                   # 서비스 단위·계약·컴포넌트 테스트 계획/결과
│   │   └── adr/                    # 서비스 로컬 결정(ADR-{{SERVICE_NAME}}-001...)
│   └── .../                        # {{SERVICE_LIST}}의 각 서비스
└── shared/                         # (선택) 서비스 간 공유 계약 스키마·공통 이벤트 봉투(envelope) 규격
```

- 폴더별 역할 요약:
  - `system/requirements/` — 시스템이 무엇을 해야 하는가(이해관계자 요구 StRS + 시스템 SRS). 시스템 요구사항 ID의 단일 출처.
  - `system/architecture/` — 왜 이 서비스 구성인가. C4 Context/Container 뷰와 토폴로지.
  - `system/service-catalog/` — 어떤 서비스가 무엇을 소유하는가. **시스템 추적성의 허브**(아래 별도 절).
  - `system/contracts/` — 서비스 간 계약의 시스템 차원 목록·호환성 정책(개별 계약 원본은 각 서비스의 `contracts/`에 둠).
  - `services/<service>/` — 각 서비스 내부. 계층형 설계·계약·테스트는 해당 언어 ISO 규약을 그대로 따릅니다.
  - `shared/` — 이벤트 봉투(envelope) 등 다수 서비스가 참조하는 공유 스키마(구현 코드가 아닌 계약만 공유).

## 필수 문서와 최소 목차

문서마다 상단 메타(뒤의 "문서 식별·버전·이력" 절 참조)를 반드시 포함합니다. 아래는 각 문서의 최소 목차입니다. 서비스 레벨 문서(SRS/설계/테스트)의 세부 목차는 언어별 ISO 규약을 따르므로 여기서는 시스템 레벨과 MSA 특화 문서 위주로 규정합니다.

### System StRS/SRS — 시스템 요구사항 명세 (ISO/IEC/IEEE 15288·29148) · `docs/system/requirements/`

1. 문서 개요(목적·범위·용어·참조)
2. 시스템 개요 — 시스템 경계(system boundary), 외부 액터·외부 시스템, 이해관계자
3. 시스템 기능 요구사항 — 요구사항 ID·설명·**관여 서비스**·우선순위·수용 기준. 서비스 경계를 넘는 흐름(예: saga)은 관여 서비스를 모두 명시
4. 시스템 비기능 요구사항 — 25010 분산 관점(신뢰성·지연 예산·상호운용성·서비스 간 보안), 측정 가능한 SLO
5. 시스템 인터페이스 — 외부 진입점({{GATEWAY}}), 외부 연동, 브로커({{BROKER}})
6. 제약사항 — 오케스트레이션({{ORCHESTRATOR}}), 규제·보안 정책
7. 시스템 요구사항 목록 부록 — ID/상태 표

- 시스템 요구사항 ID 예: `SYS-F-001`(시스템 기능), `SYS-NF-001`(시스템 비기능). 서비스 로컬 요구사항은 서비스별 SRS에서 `REQ-F-001` 등으로 관리하고, 시스템 요구사항과 매핑합니다.

### System SAD — 시스템 아키텍처 기술서 (ISO/IEC/IEEE 42010, C4 model) · `docs/system/architecture/SYSTEM_SAD.md`

1. 이해관계자와 관심사 — 예: 운영자(가용성·관측성), 보안담당(서비스 간 인증), 팀 리드(독립 배포·소유권)
2. 아키텍처 관점(viewpoint)과 C4 뷰의 매핑
3. C4 뷰(view):
   - **Context 뷰(C1)**: 시스템({{SYSTEM_NAME}})과 외부 액터·외부 시스템의 관계
   - **Container 뷰(C2)**: 서비스({{SERVICE_LIST}})·데이터스토어({{DB}})·브로커({{BROKER}})·게이트웨이({{GATEWAY}})의 토폴로지와 동기/비동기 통신 경로 표기
   - **Component 뷰(C3)**: 특정 서비스 내부의 주요 컴포넌트(필요한 서비스만)
   - **Code 뷰(C4)**: 필요 시에만. 통상 코드로 대체
4. **배포 뷰**: {{ORCHESTRATOR}} 상의 배포 단위·복제·네트워크 경계
5. 아키텍처 결정 근거 — ADR 참조(특히 서비스 분해·통신 방식·데이터 소유권 결정)
6. 품질 속성 시나리오 — 25010 분산 관점 대응(품질 모델 문서와 연결)

- C4 다이어그램은 텍스트 기반(예: Structurizr DSL, Mermaid `C4Context`/`C4Container`)으로 작성해 버전관리·diff가 가능하도록 합니다.

### 서비스 카탈로그 (Service Catalog) · `docs/system/service-catalog/SERVICE_CATALOG.md` [핵심]

- 서비스 카탈로그는 **시스템 추적성의 허브**입니다. 모든 서비스의 소유권·책임·계약·의존을 한 표로 모아, 요구사항↔서비스↔계약↔배포를 잇는 기준점으로 삼습니다.
- 필수 컬럼:

| 서비스 | 소유팀 | 책임(bounded context) | 노출 API | 발행 이벤트 | 구독 이벤트 | 의존 서비스 | 데이터 저장소 | 소스 저장소 | 상태 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| {{SERVICE_NAME}} | {{TEAM}} | 주문 관리 | `POST /orders`, `GET /orders/{id}` | `OrderCreated`, `OrderCancelled` | `PaymentApproved` | payment-svc, inventory-svc | orders-db({{DB}}) | services/order-svc | Active |

- "데이터 저장소" 컬럼은 서비스가 소유하는 DB/영속 저장소를, "소스 저장소" 컬럼은 monorepo/polyrepo 어느 전략이든 서비스 소스 위치(repo 또는 경로)를 가리켜 코드와 문서를 연결합니다.
- "상태" 값: `Planned` / `Active` / `Deprecated` / `Retired`.
- 데이터 소유권 원칙(database-per-service)에 따라 한 데이터 저장소는 하나의 서비스만 소유합니다. 두 서비스가 같은 저장소를 가리키면 규약 위반(공유 DB 안티패턴)으로 리뷰에서 차단합니다.

### 계약(Contract) 문서 [핵심 · 1급 산출물]

- 계약은 서비스 간 결합의 유일한 접점이므로 코드와 동급의 1급 산출물로 버전 관리합니다. 계약 원본은 각 서비스의 `docs/services/<service>/contracts/`에 두고, 시스템 차원 목록은 `docs/system/contracts/`에서 집계합니다.

| 통신 유형 | 계약 형식 | 원본 위치 | 호환성 도구(예) |
| --- | --- | --- | --- |
| 동기 REST | OpenAPI(`openapi.yaml`) | `services/<svc>/contracts/openapi.yaml` | OpenAPI diff, consumer-driven contract test |
| 동기 gRPC | protobuf(`.proto`) | `services/<svc>/contracts/proto/` | `buf lint` / `buf breaking` |
| 비동기 이벤트 | AsyncAPI(`asyncapi.yaml`) 또는 스키마 레지스트리 등록 스키마 | `services/<svc>/contracts/asyncapi.yaml` | AsyncAPI validate, schema registry 호환성 모드 |

- **계약 버전·호환성 정책**(문서화 필수):
  - 계약 버전은 semver(`x.y.z`)를 사용합니다. major=하위호환 파괴, minor=하위호환 추가, patch=문서/설명 수정.
  - 원칙적으로 **backward-compatible 진화**만 허용합니다(필드 추가는 optional, 필드 삭제·의미 변경·필수화는 major이며 소비자 마이그레이션 계획 필수).
  - breaking change가 불가피하면 새 버전을 병행 제공(예: `/v2`, `OrderCreated.v2`)하고, 구버전 폐기 일정(deprecation window)을 카탈로그·계약 문서에 기록합니다.
  - 이벤트는 최소 봉투(envelope) 규격(예: `event_id`, `event_type`, `event_version`, `occurred_at`, `trace_id`)을 `docs/shared/`에 고정하고 모든 이벤트가 이를 따르도록 합니다.

### 서비스 레벨 SRS/설계/테스트 · `docs/services/<service>/`

- 각 서비스는 언어별 ISO 규약(`golang_guideline` / `python_guideline` / `cpp_guideline`의 `ISO_DOCUMENTS_CONVENTION.md`)에 따라 SRS·설계서·테스트 문서를 작성합니다. 계층형(handler/router → service → repository) 구조는 서비스 내부 설계로 그대로 유지됩니다.
- 서비스 SRS는 상단 메타에 소유 시스템 요구사항 ID(`SYS-F-xxx`)와의 매핑을 포함합니다.

### 시스템 통합/E2E 테스트 계획서·결과서 · `docs/system/test/`

- 계획서: 1) 테스트 범위(계약 테스트/서비스 간 통합/시스템 E2E) 2) 대상 시스템 요구사항 ID(`SYS-*`) 및 관여 서비스 3) 계약 테스트 케이스(consumer/provider 쌍) 4) saga·이벤트 흐름 시나리오 5) 환경(브로커·게이트웨이 포함) 6) 통과 기준
- 결과서: 1) 실행 일시·환경(배포 토폴로지 버전) 2) 계약/통합/E2E 결과 요약 3) 계약 호환성 검사 결과 4) 실패 항목·조치 5) 잔여 리스크

## 문서 식별·버전·이력

- 문서 ID 체계(접두어-일련번호):
  - 시스템 레벨: `SYS-SRS-001`(시스템 요구사항), `SYS-SAD-001`(시스템 아키텍처), `SVC-CAT-001`(서비스 카탈로그), `SYS-TRC-001`(시스템 추적성), `SYS-QLT-001`(시스템 품질), `SYS-SEC-001`(시스템 보안), `SYS-OPS-001`(운영), `ADR-SYS-001`(시스템 결정).
  - 서비스 레벨: 서비스 접두어를 부여합니다. 예 `ORDER-SRS-001`, `ORDER-DSN-001`, `ORDER-TST-001`, `ADR-ORDER-001`.
  - 계약: `CTR-<SVC>-API-001`(OpenAPI/proto), `CTR-<SVC>-EVT-001`(이벤트/AsyncAPI).
- 버전 규칙: 문서 버전은 `x.y`(major.minor)를 기본으로 하며, 외부 계약을 갖는 문서(OpenAPI/proto/AsyncAPI)는 semver(`x.y.z`)를 사용합니다. major는 하위호환 파괴, minor는 추가/보완, patch는 오탈자·서식 수정입니다.
- 상태 값: `Draft` → `Review` → `Approved` → `Deprecated`.
- 문서 상단 메타 표(모든 문서 필수):

| 항목 | 값 |
| --- | --- |
| 문서 ID | SVC-CAT-001 |
| 버전 | 1.3 |
| 작성일 | {{YYYY-MM-DD}} |
| 작성자 | {{AUTHOR}} |
| 검토자 | {{REVIEWER}} |
| 상태 | Approved |

- 변경 이력 표(문서 하단 필수):

| 버전 | 일자 | 작성자 | 변경 내용 | 관련 이슈/요구사항 |
| --- | --- | --- | --- | --- |
| 1.0 | {{YYYY-MM-DD}} | {{AUTHOR}} | 최초 작성 | SYS-F-001 |
| 1.1 | {{YYYY-MM-DD}} | {{AUTHOR}} | payment-svc 추가 | SYS-F-004 |

## 분산 추적성(traceability) [핵심]

- 추적성은 이 규약의 최우선(0티어) 관심사입니다. MSA에서는 추적 사슬이 서비스 경계를 넘으므로 단계를 다음 5단계로 확장합니다:

  **시스템 요구사항 → 서비스 → (컴포넌트) → API/이벤트 계약 → 테스트(계약/통합/E2E 포함)**

  - 순방향: 시스템 요구사항이 어느 서비스·계약·테스트로 구현·검증되는가.
  - 역방향: 특정 계약/엔드포인트/이벤트가 어떤 시스템 요구사항에서 왔는가(고아 계약·요구사항 없는 이벤트 방지).
- 각 단계의 표기 규칙:
  - **시스템 요구사항**: `SYS-F-001` / `SYS-NF-001`
  - **서비스**: 서비스 카탈로그의 서비스 식별자(예: `order-svc`). 카탈로그가 이 단계의 단일 출처.
  - **컴포넌트**(선택): 서비스 내부 설계 ID(예: `ORDER-DSN-001 §3.2`) 또는 코드 경로
  - **계약**: 동기는 `METHOD /path`(REST) 또는 `service.Method`(gRPC), 비동기는 `이벤트명@버전`(예: `OrderCreated@1.0`)
  - **테스트**: 계약 테스트(consumer/provider), 통합, E2E 케이스 ID/함수명
- **서비스 경계를 넘는 요구사항**(예: saga)은 한 요구사항이 여러 서비스·여러 계약에 매핑됩니다. 이때 행을 서비스 단위로 분할하여 1:1로 정규화하고, 같은 `SYS-F` ID로 묶습니다. 예시(주문 결제 saga):

| 시스템 요구사항 | 서비스 | 컴포넌트/설계 | 계약(API/이벤트) | 테스트 | 상태 |
| --- | --- | --- | --- | --- | --- |
| SYS-F-010 (주문 결제 saga) | order-svc | ORDER-DSN-001 §4.1 | `POST /orders`, 발행 `OrderCreated@1.0` | `PactProvider_Order`, `E2E_OrderCheckout` | 구현완료 |
| SYS-F-010 (주문 결제 saga) | payment-svc | PAY-DSN-001 §3.3 | 구독 `OrderCreated@1.0`, 발행 `PaymentApproved@1.0` | `PactConsumer_Payment_OrderCreated` | 검증중 |
| SYS-F-010 (주문 결제 saga) | inventory-svc | INV-DSN-001 §2.5 | 구독 `PaymentApproved@1.0`, 발행 `StockReserved@1.0` | `E2E_OrderCheckout` | 구현중 |

- 상태 값 예: `미착수` / `설계완료` / `구현중` / `구현완료` / `검증중` / `완료`.
- saga 흐름은 이벤트 순서(choreography)나 오케스트레이터 단계(orchestration)를 추적성 표와 별도의 흐름 다이어그램/시퀀스로도 문서화하여, 이벤트 사슬이 끊긴 지점을 시각적으로 검출할 수 있게 합니다.

## 코드-문서 연계

- 목적: 문서의 ID를 코드·계약·테스트·커밋에 심어 역방향 추적이 자동으로 성립하도록 합니다.
- 계약↔요구사항 연계: OpenAPI 오퍼레이션의 `description`, proto 서비스/메서드 주석, AsyncAPI 메시지의 설명에 대응 시스템 요구사항 ID(`SYS-F-xxx`)를 포함합니다.
- 이벤트 봉투에 `trace_id`를 실어 관측성(분산 추적)과 문서 추적성을 잇습니다(요청 단위 상관관계는 correlation ID/trace ID로 서비스 경계를 넘어 전파 — 상세는 `msa_guideline/MSA_ARCHITECTURE.md` 관측성 절).
- 서비스별 코드 주석·테스트 명명·커밋 형식은 해당 언어 ISO 규약을 따릅니다.
- 커밋 메시지 **형식**(임의 커밋 금지 원칙 준수 — 커밋은 사용자가 직접 수행하며, 여기서는 형식만 규정합니다):

```text
<type>(<service>): <subject>

<body>

Refs: SYS-F-010, ORDER-DSN-001
```

  - `type`: `feat` / `fix` / `docs` / `refactor` / `test` / `chore` / `contract`(계약 변경 시).
  - `scope`에는 영향 서비스명을 적어 커밋↔서비스↔요구사항 추적을 확보합니다.

## 품질 속성 매핑(ISO/IEC 25010 — 분산 관점 보강)

8개 특성을 MSA(분산 시스템) 관점으로 구체화하고 측정 지표 예를 제시합니다. 각 항목은 `SYS-NF-xxx`로 요구사항화하여 추적합니다. 단일 서비스 관점 지표는 서비스별 품질 문서에서 별도로 관리합니다.

| 특성(25010) | MSA(분산) 관점 구체화 | 측정 지표 예 |
| --- | --- | --- |
| 기능 적합성(Functional Suitability) | 서비스가 계약(OpenAPI/proto/AsyncAPI)대로 동작 | consumer-driven contract test 통과율 100% |
| 성능 효율성(Performance Efficiency) | 종단 지연 + **per-hop latency budget**(호출 체인 각 구간 지연 예산 배분) | E2E p95 ≤ 500ms, 서비스 간 1-hop p95 ≤ 50ms, 체인 hop 수 ≤ 3 |
| 호환성(Compatibility) | 서비스 간 **interoperability**·계약 호환(버전 진화) | breaking change 0(`buf breaking` 통과), 구/신 계약 병행 소비 성공 |
| 사용성(Usability) | API/이벤트 문서 최신성·오류 규격 일관성 | OpenAPI/AsyncAPI 최신화율 100%, 공통 error/envelope 준수율 100% |
| 신뢰성(Reliability) | **fault tolerance**(부분 장애 격리)·**recoverability**(재처리·복구) | 의존 서비스 장애 시 graceful degradation 동작 100%, at-least-once 재처리 시 중복 부작용 0(멱등) |
| 보안(Security) | **서비스 간 인증**(mTLS/JWT 전파, zero-trust)·게이트웨이 인증 | 서비스 간 호출 mTLS/토큰 검증 커버율 100%, 게이트웨이 외 직접 노출 서비스 0 |
| 유지보수성(Maintainability) | 서비스 독립 배포성·계약 기반 결합도 | 독립 배포 가능 서비스 비율 100%, 서비스 간 내부 구현 직접 import 0 |
| 이식성(Portability) | 컨테이너·설정 외부화·오케스트레이터 이식 | 서비스별 컨테이너 이미지로 무수정 배포, 설정 환경변수화 100% |

- 측정 방법 예: 종단·per-hop 지연과 hop 수는 분산 추적({{TRACING_BACKEND}}, OpenTelemetry) 스팬으로 계측, RED(Rate/Errors/Duration) 메트릭으로 서비스별 상태 관측, 계약 호환성은 `buf breaking`·OpenAPI diff·contract test, 서비스 간 인증 커버리지는 게이트웨이/서비스 메시 정책으로 확인합니다.

## 검증·적합성

- 문서 리뷰 절차:
  1. 작성자가 `Draft`로 작성 후 셀프 체크(메타·이력·요구사항/서비스 ID 부여 여부, 서비스 카탈로그 갱신 여부).
  2. 검토자가 `Review` 단계에서 정합성·완전성·추적성 확인(특히 계약 호환성, 데이터 소유권 위반 여부).
  3. 지적사항 반영 후 `Approved`로 승격, 버전·이력 갱신.
- 시스템-문서 정합성 확인 항목:
  - 모든 시스템 기능 요구사항이 서비스·계약·테스트로 추적되는가(고아 요구사항 없음).
  - 모든 노출 API/발행 이벤트가 시스템 요구사항으로 역추적되는가(고아 계약·이벤트 없음).
  - 서비스 카탈로그의 각 항목(노출 API·발행/구독 이벤트·의존·데이터 저장소)이 실제 계약·배포와 일치하는가.
  - **데이터 소유권 위반 없음**: 한 데이터 저장소를 두 서비스가 소유하지 않는가(공유 DB 금지).
  - 계약(OpenAPI/proto/AsyncAPI)이 실제 서비스 구현과 일치하는가(드리프트 없음).
  - saga/이벤트 흐름 문서의 발행-구독 쌍이 실제 계약에서 짝이 맞는가(끊긴 이벤트 없음).
- **Definition of Done**(문서 갱신 포함):
  - [ ] 관련 시스템/서비스 요구사항 ID 정의·분산 추적성 매트릭스 갱신 완료.
  - [ ] 서비스 카탈로그 갱신(신규/변경 서비스·계약·의존 반영).
  - [ ] 계약 스냅샷(OpenAPI/proto/AsyncAPI) 최신화, 계약 호환성 검사 통과(breaking change 0 또는 마이그레이션 계획 문서화).
  - [ ] 영향 서비스의 언어별 빌드·테스트 통과(에러 0).
  - [ ] 계약/통합/E2E 테스트 통과.
  - [ ] 영향받은 ISO 문서(System SRS/SAD/카탈로그/추적성/품질/보안) 갱신·이력 기록.
  - [ ] 리뷰 승인(`Approved`).

## 문서 갱신 규칙

- 전역 지침("feature, bug fix 등 수정 시 ISO 문서 갱신이 필요한 경우 진행")을 반영합니다. MSA 특유의 변경 유형별 갱신 대상:

| 변경 유형 | 갱신 대상 문서 |
| --- | --- |
| **신규 서비스 추가** | 서비스 카탈로그(행 추가), System SAD(Container 뷰), System SRS(관여 서비스), 배포 토폴로지(운영), 신규 서비스의 서비스 레벨 문서 일체, 분산 추적성 |
| **서비스 분리/병합** | 서비스 카탈로그(책임·소유권·데이터 저장소 재배치), System SAD, ADR(분리/병합 결정 근거), 데이터 마이그레이션(운영), 분산 추적성, 영향 계약 |
| **동기 API 계약 변경** | 해당 서비스 OpenAPI/proto(버전↑), 서비스 카탈로그(노출 API), System contracts 호환성 정책, 소비 서비스 영향 분석, contract test, 분산 추적성 |
| **이벤트 계약 변경** | 해당 서비스 AsyncAPI/스키마 레지스트리(버전↑), 서비스 카탈로그(발행/구독 이벤트), 이벤트 봉투 규격(`shared/`), 구독 서비스 영향 분석, 분산 추적성 |
| **saga 흐름 변경** | System SRS(saga 요구사항), saga 흐름 다이어그램, 관여 서비스들의 설계·계약, 분산 추적성(관여 서비스 행 전체), 통합/E2E 테스트 |
| **배포 토폴로지 변경** | System SAD(배포 뷰), 운영 문서({{ORCHESTRATOR}} 매니페스트·복제·네트워크 경계), 서비스 카탈로그(저장소·상태) |
| 성능/보안 개선 | System 품질 모델(지연 예산·SLO), 시스템 비기능 요구사항, System 보안 문서(서비스 간 인증) |

- 갱신 시 반드시: 대상 문서의 버전·`작성일`·변경 이력 표를 갱신하고, 관련 요구사항 ID를 이력의 "관련 이슈/요구사항" 칸에 기록합니다. 계약 변경은 **소비 서비스 영향 분석**을 함께 남깁니다.
- 문서 변경이 코드·계약 변경과 같은 작업 단위에서 이루어지도록 하여 드리프트를 방지합니다.

## 도구·자동화

- **계약 스냅샷 드리프트 검사**(CI 필수 게이트): 서비스 구현에서 계약을 생성/추출한 뒤, 커밋된 계약 스냅샷과 차이가 있으면 실패시킵니다. 계약이 코드와 함께 갱신되지 않으면 CI가 깨지도록 강제하는 것이 목적입니다.

```bash
# gRPC(proto) — buf로 lint 및 하위호환 파괴 검사
buf lint
buf breaking --against '.git#branch=main'    # main 대비 breaking change 검출

# 비동기 이벤트(AsyncAPI) — 스키마 유효성 검사
asyncapi validate docs/services/{{SERVICE_NAME}}/contracts/asyncapi.yaml
```

  - REST(OpenAPI)는 서비스가 사용하는 언어의 생성 도구로 스냅샷을 재생성한 뒤 `git diff --exit-code`로 드리프트를 검사합니다(생성 방식은 언어별 규약 참조 — 예: Go는 `swaggo/swag`).
- **계약 테스트(consumer-driven contract)**: 소비자-제공자 계약을 자동 검증합니다(예: Pact). consumer 측에서 기대(expectation)를 정의하고 provider 측에서 실제 응답이 이를 만족하는지 검증하여, 계약 위반을 배포 전에 차단합니다.
- **서비스 카탈로그↔실제 배포 정합성 검사**: 카탈로그에 기재된 서비스·의존·데이터 저장소가 실제 배포 매니페스트({{ORCHESTRATOR}})와 일치하는지 CI에서 대조합니다. 불일치(카탈로그에 없는 서비스가 배포됨, 데이터 저장소 중복 소유 등)는 실패 처리합니다.
- **CI에서의 문서/추적성 점검(권장)**:
  - 변경된 서비스별로 빌드·테스트를 필수 게이트로 실행(언어별 규약의 명령 사용).
  - 위 계약 드리프트·계약 테스트·카탈로그 정합성 검사를 PR 게이트로 실행.
  - 분산 추적성 매트릭스의 참조(시스템 요구사항 ID·서비스 식별자·계약명·테스트명)가 서비스 카탈로그·실제 계약에 존재하는지 링크 체크 스크립트로 확인.
- 자동화의 목표는 "계약·카탈로그·추적성 문서가 코드와 함께 갱신되지 않으면 CI가 실패하도록" 만들어, 서비스 경계를 넘는 추적성과 정합성을 강제하는 것입니다.
