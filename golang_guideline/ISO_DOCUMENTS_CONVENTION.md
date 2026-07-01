# ISO 문서 규약 (Go REST API 프로젝트)

Go 기반 REST API 프로젝트의 산출물(요구사항·아키텍처·설계·테스트·추적성 등)을 ISO/IEC/IEEE 표준에 맞춰 작성·관리하기 위한 규약입니다. ISO 문서는 언어 독립적이되, 본 규약의 예시는 Go(Gin 기반 REST API) 프로젝트 기준으로 구체화합니다.

## 목적과 적용 표준

- 목적: 요구사항부터 코드·테스트·API까지의 **양방향 추적성(traceability)** 을 확보하고, 문서와 실제 코드의 정합성을 지속적으로 검증하는 것입니다. 추적성은 최우선 관심사입니다.
- 적용 표준(각 표준이 규정하는 바):
  - **ISO/IEC/IEEE 12207** — 소프트웨어 생명주기 프로세스. 요구사항·설계·구현·검증·유지보수 등 소프트웨어 단위의 프로세스 프레임을 규정합니다. 본 규약의 기본 골격입니다.
  - **ISO/IEC/IEEE 15288** — 시스템 생명주기 프로세스. 소프트웨어가 상위 시스템(인프라·외부 연동 포함)의 일부일 때 적용합니다. 단일 API 서비스라면 참조 수준으로만 둡니다.
  - **ISO/IEC/IEEE 29148** — 요구공학(requirements engineering). 요구사항의 작성·품질 기준·SRS 구조·요구사항 속성(ID·우선순위·검증방법)을 규정합니다. SRS의 근거입니다.
  - **ISO/IEC/IEEE 42010** — 아키텍처 기술(architecture description). 이해관계자(stakeholder)·관심사(concern)·관점(viewpoint)·뷰(view)의 구조로 아키텍처를 기술하도록 규정합니다. 아키텍처 기술서의 근거입니다.
  - **ISO/IEC 25010** — SQuaRE 제품 품질 모델. 8개 품질 특성(제품 품질)을 정의합니다. 비기능 요구사항과 품질 목표의 근거입니다.
  - **ISO/IEC 27001** — 정보보안 경영시스템(ISMS). 정보보안 통제 항목의 맥락을 제공합니다. 인증·인가·시크릿 관리·감사로그 요구사항의 근거로 참조합니다(맥락 수준).
  - **ISO 9001** — 품질경영시스템(QMS). 문서 관리·리뷰·변경 관리 등 프로세스 품질의 맥락을 제공합니다(맥락 수준).

## docs/ 폴더 구조

- 모든 ISO 문서는 프로젝트 루트의 `docs/` 하위에 저장합니다. 전역 지침에 따라 `docs/`는 `.gitignore`에 추가하여 커밋되지 않도록 관리합니다.
- 표준 트리:

```text
docs/
├── requirements/    # 요구사항 명세(SRS). 기능/비기능/제약, 요구사항 ID 관리
├── architecture/    # 아키텍처 기술서(42010): 이해관계자·관심사·관점·뷰
├── design/          # 상세 설계서: 패키지/계층 설계, 데이터 모델, 시퀀스
├── api/             # API 명세: OpenAPI(openapi.yaml/json), 엔드포인트 계약
├── test/            # 테스트 계획서/결과서: 전략, 케이스, 실행 결과, 커버리지
├── traceability/    # 추적성 매트릭스(요구사항↔설계↔코드↔테스트↔API)
├── quality/         # 품질 모델(25010) 매핑, 품질 목표·측정 지표
├── security/        # 보안 요구사항·통제(27001 맥락), 위협 모델, 시크릿 정책
├── operations/      # 운영/배포: 런북, 헬스체크, 모니터링, 롤백 절차
└── adr/             # Architecture Decision Records: 결정 단위 기록(ADR-001...)
```

- 폴더별 역할:
  - `requirements/` — 무엇을 만들 것인가. 요구사항 ID의 단일 출처(single source of truth).
  - `architecture/` — 왜 이 구조인가. 관점·뷰 기반 상위 구조와 근거.
  - `design/` — 어떻게 구현하는가. Go 패키지/계층(handler→service→repository)과 인터페이스 경계.
  - `api/` — 외부 계약. Gin 라우트와 OpenAPI 명세를 일치시킵니다.
  - `test/` — 검증 방법·증거. `go test` 결과와 커버리지 리포트.
  - `traceability/` — 위 산출물 간 연결. 최우선 관리 대상.
  - `quality/` — 25010 특성별 목표·지표.
  - `security/` — 보안 요구사항·통제·위협.
  - `operations/` — 배포·운영 절차.
  - `adr/` — 되돌리기 어려운 결정의 근거 기록.

## 필수 문서와 최소 목차

문서마다 상단 메타(다음 절 참조)를 반드시 포함합니다. 아래는 각 문서의 최소 목차입니다.

### SRS — 소프트웨어 요구사항 명세 (ISO/IEC/IEEE 29148) · `docs/requirements/SRS.md`

1. 문서 개요(목적·범위·용어·참조)
2. 시스템 개요(대상 도메인, 이해관계자, 가정·제약)
3. 기능 요구사항 — 요구사항 ID·설명·입력/출력·우선순위·수용 기준
4. 비기능 요구사항 — 25010 특성과 연결(성능·신뢰성·보안 등), 측정 가능한 목표
5. 인터페이스 요구사항 — 외부 API·DB·메시지 큐 연동
6. 제약사항 — 기술 스택({{LANGUAGE_VERSION}}, {{FRAMEWORK}}), 규제·보안 정책
7. 요구사항 목록 부록 — ID/상태 표

- 요구사항 ID 예: `REQ-F-001`(기능), `REQ-NF-001`(비기능). 각 요구사항은 검증 가능해야 합니다("빠르게" 금지, "p95 응답시간 ≤ 200ms" 사용).

### 아키텍처 기술서 (ISO/IEC/IEEE 42010) · `docs/architecture/ARCHITECTURE.md`

1. 이해관계자(stakeholder)와 관심사(concern) — 예: 운영자(가용성), 보안담당(인증), 개발자(유지보수성)
2. 아키텍처 관점(viewpoint) 정의 — 논리/프로세스/배포/데이터 관점
3. 뷰(view) — 각 관점에 대응하는 구체 기술
   - 논리 뷰: 계층 구조(handler/router → service → repository)와 인터페이스 경계
   - 프로세스 뷰: 요청 처리 흐름, 동시성(goroutine) 모델
   - 배포 뷰: 컨테이너·인프라 토폴로지
   - 데이터 뷰: {{DB}} 스키마 개요, 캐시/큐
4. 아키텍처 결정 근거 — ADR 참조
5. 품질 속성 시나리오 — 25010 특성별 대응(품질 모델 문서와 연결)

### 상세 설계서 · `docs/design/DESIGN.md`

1. 패키지 구조 — 예: `internal/handler`, `internal/service`, `internal/repository`, `internal/domain`
2. 인터페이스 정의 — service·repository의 Go interface 경계와 책임
3. 데이터 모델 — 도메인 구조체, DTO, DB 매핑
4. 주요 시퀀스 — 대표 유스케이스별 handler→service→repository 흐름
5. 오류 처리 전략 — 오류 래핑(`fmt.Errorf("...: %w", err)`), HTTP 상태 매핑
6. 횡단 관심사 — 로깅(`log/slog`), 검증(`go-playground/validator`), 미들웨어

### API 명세 (OpenAPI) · `docs/api/openapi.yaml`

1. API 개요(base path, 버전, 인증 방식)
2. 엔드포인트 — METHOD/path/요청/응답/상태코드
3. 스키마(request/response DTO)
4. 오류 응답 규격(공통 error body)
5. 인증·인가·레이트리밋 정책

- Go에서는 핸들러에 `swaggo/swag` 주석을 붙여 `swag init`으로 OpenAPI 문서를 생성하는 방식을 권장합니다. 생성 산출물(`docs/`의 `swagger.yaml`/`swagger.json`)을 `docs/api/`와 정합시킵니다. 코드 주석이 API 명세의 원천이 되도록 하여 문서-코드 드리프트를 줄입니다.

### 테스트 계획서/결과서 · `docs/test/TEST_PLAN.md`, `docs/test/TEST_RESULT.md`

- 계획서: 1) 테스트 범위·전략(단위/통합/E2E) 2) 대상 요구사항 ID 목록 3) 테스트 케이스(케이스 ID·대상 REQ·절차·기대결과) 4) 환경·데이터 5) 통과 기준(커버리지 목표 등)
- 결과서: 1) 실행 일시·환경 2) 결과 요약(통과/실패/스킵) 3) 커버리지 리포트 4) 실패 항목·조치 5) 잔여 리스크

### 추적성 매트릭스 · `docs/traceability/TRACEABILITY.md`

- 다음 절의 5단계 양방향 추적을 표로 관리합니다.

### 품질 모델 (ISO/IEC 25010) · `docs/quality/QUALITY_MODEL.md`

1. 8개 특성별 프로젝트 적용 정의
2. 특성 → 비기능 요구사항(REQ-NF-xxx) 매핑
3. 측정 지표·목표값·측정 방법

## 문서 식별·버전·이력

- 문서 ID 체계(접두어-일련번호): `SRS-001`, `ARC-001`, `DSN-001`, `API-001`, `TST-001`, `TRC-001`, `QLT-001`, `SEC-001`, `OPS-001`, `ADR-001`.
- 버전 규칙: 문서 버전은 `x.y`(major.minor)를 기본으로 하며, API 명세처럼 외부 계약을 갖는 문서는 semver(`x.y.z`)를 사용할 수 있습니다. major는 하위호환이 깨지는 변경, minor는 내용 추가/보완, patch는 오탈자·서식 수정입니다.
- 상태 값: `Draft` → `Review` → `Approved` → `Deprecated`.
- 문서 상단 메타 표(모든 문서 필수):

| 항목 | 값 |
| --- | --- |
| 문서 ID | SRS-001 |
| 버전 | 1.2 |
| 작성일 | {{YYYY-MM-DD}} |
| 작성자 | {{AUTHOR}} |
| 검토자 | {{REVIEWER}} |
| 상태 | Approved |

- 변경 이력 표(문서 하단 필수):

| 버전 | 일자 | 작성자 | 변경 내용 | 관련 이슈/요구사항 |
| --- | --- | --- | --- | --- |
| 1.0 | {{YYYY-MM-DD}} | {{AUTHOR}} | 최초 작성 | REQ-F-001 |
| 1.1 | {{YYYY-MM-DD}} | {{AUTHOR}} | 인증 요구사항 추가 | REQ-NF-003 |

## 추적성(traceability) [핵심]

- 추적성은 이 규약의 최우선(0티어) 관심사입니다. **요구사항 ID → 아키텍처/설계 → 코드 → 테스트 → API 엔드포인트** 의 5단계를 **양방향** 으로 연결합니다.
  - 순방향: 요구사항이 어느 코드/테스트/API로 구현·검증되는가.
  - 역방향: 특정 코드/엔드포인트가 어떤 요구사항에서 왔는가(고아 코드·요구사항 없는 기능 방지).
- 각 단계의 Go 프로젝트 표기 규칙:
  - **요구사항**: `REQ-F-001` / `REQ-NF-001`
  - **아키텍처/설계**: `ARC-001` / `DSN-001`(문서 내 절 번호까지 지정 권장)
  - **코드**: Go 패키지·타입·함수 경로. 예: `internal/service/user.(*UserService).Create`
  - **테스트**: `go test` 함수명. 예: `TestUserService_Create`(파일 경로 병기)
  - **API**: Gin 라우트를 `METHOD /path` 형식으로. 예: `POST /api/v1/users`
- 추적성 매트릭스 표 예시:

| 요구사항 ID | 설계 | 코드 위치 | 테스트 | API | 상태 |
| --- | --- | --- | --- | --- | --- |
| REQ-F-001 | DSN-001 §3.2 | `internal/service/user.(*UserService).Create` | `TestUserService_Create` | `POST /api/v1/users` | 구현완료 |
| REQ-F-002 | DSN-001 §3.3 | `internal/service/user.(*UserService).GetByID` | `TestUserService_GetByID` | `GET /api/v1/users/:id` | 구현완료 |
| REQ-NF-003 | ARC-001 §5.1 | `internal/middleware/auth.RequireJWT` | `TestRequireJWT_RejectsExpired` | `(all) /api/v1/*` | 검증중 |

- 상태 값 예: `미착수` / `설계완료` / `구현중` / `구현완료` / `검증중` / `완료`.
- 요구사항 하나가 여러 코드/테스트/API에 대응할 수 있으므로 필요 시 행을 분할하여 1:1로 정규화합니다(추적 누락 방지).

## 코드-문서 연계

- 목적: 문서의 ID를 코드·테스트·커밋에 심어 역방향 추적이 자동으로 성립하도록 합니다.
- 코드 주석(요구사항 참조가 비자명할 때만 — 전역 지침의 "주석은 WHY가 비자명할 때만" 준수):

```go
// Create 는 신규 사용자를 생성합니다.
// Ref: REQ-F-001 (이메일 중복 시 409 반환은 REQ-NF-002 제약)
func (s *UserService) Create(ctx context.Context, in CreateUserInput) (*domain.User, error) {
    // ...
}
```

- 테스트 함수명에 요구사항 의도를 드러내고, 필요 시 주석으로 ID를 병기합니다:

```go
// TestUserService_Create verifies REQ-F-001.
func TestUserService_Create(t *testing.T) { /* ... */ }
```

- 커밋 메시지 **형식**(임의 커밋 금지 원칙 준수 — 커밋은 사용자가 직접 수행하며, 여기서는 형식만 규정합니다):

```text
<type>(<scope>): <subject>

<body>

Refs: REQ-F-001, REQ-NF-002
```

  - `type`: `feat` / `fix` / `docs` / `refactor` / `test` / `chore`.
  - `Refs:` 트레일러에 관련 요구사항 ID를 나열하여 커밋↔요구사항 추적을 확보합니다.
- API 명세 연계: `swaggo/swag` 주석의 `@Summary`/`@Router`에 대응하는 요구사항 ID를 설명에 포함해 API↔요구사항을 연결합니다.

## 품질 속성 매핑(ISO/IEC 25010)

REST API 관점으로 8개 특성을 구체화하고 측정 지표 예를 제시합니다. 각 항목은 `REQ-NF-xxx`로 요구사항화하여 추적합니다.

| 특성(25010) | REST API 관점 구체화 | 측정 지표 예 |
| --- | --- | --- |
| 기능 적합성(Functional Suitability) | 명세된 엔드포인트가 계약대로 동작 | OpenAPI 계약 테스트 통과율 100% |
| 성능 효율성(Performance Efficiency) | 응답 지연·처리량·자원 사용 | 응답시간 p95 ≤ 200ms, 처리량 ≥ {{RPS}} rps |
| 호환성(Compatibility) | API 버저닝, 하위호환, 외부 연동 | 버전 간 계약 호환성 검사 통과, 스키마 breaking change 0 |
| 사용성(Usability) | API 문서·오류 메시지의 명확성 | OpenAPI 최신화율 100%, 표준 error body 준수율 100% |
| 신뢰성(Reliability) | 가용성·장애 복구·오류율 | 가용성 ≥ 99.9%, 5xx 오류율 ≤ 0.1% |
| 보안(Security) | 인증·인가·전송보안·감사 | 인증 필수 엔드포인트 커버율 100%, 알려진 취약점(고위험) 0 |
| 유지보수성(Maintainability) | 모듈성·테스트성·정적분석 | 테스트 커버리지 ≥ {{COVERAGE}}%, `go vet`/linter 경고 0 |
| 이식성(Portability) | 환경 독립성·배포 용이성 | 컨테이너 이미지로 무수정 배포, 설정은 환경변수화 100% |

- 측정 방법 예: 성능은 부하테스트 도구로 p95 계측, 커버리지는 `go test -coverprofile`, 정적분석은 `go vet` 및 linter, 보안은 `govulncheck`.

## 검증·적합성

- 문서 리뷰 절차:
  1. 작성자가 `Draft`로 작성 후 셀프 체크(메타·이력·요구사항 ID 부여 여부).
  2. 검토자가 `Review` 단계에서 정합성·완전성·추적성 확인.
  3. 지적사항 반영 후 `Approved`로 승격, 버전·이력 갱신.
- 코드-문서 정합성 확인 항목:
  - 모든 기능 요구사항이 코드·테스트·API로 추적되는가(고아 요구사항 없음).
  - 모든 API 엔드포인트가 요구사항으로 역추적되는가(고아 엔드포인트 없음).
  - `docs/api/openapi.yaml`(또는 `swag` 생성물)이 실제 Gin 라우트와 일치하는가.
  - 테스트 함수가 대응 요구사항을 실제로 검증하는가.
- **Definition of Done**(문서 갱신 포함):
  - [ ] 관련 요구사항 ID 정의·추적성 매트릭스 갱신 완료.
  - [ ] 코드가 컴파일되고 `go build ./...` 성공, `go vet ./...` 경고 0.
  - [ ] `go test ./...` 통과, 커버리지 목표 충족.
  - [ ] API 변경 시 OpenAPI(`swag init`) 재생성·정합.
  - [ ] 영향받은 ISO 문서(SRS/설계/추적성/품질) 갱신·이력 기록.
  - [ ] 리뷰 승인(`Approved`).

## 문서 갱신 규칙

- 전역 지침("feature, bug fix 등 수정 시 ISO 문서 갱신이 필요한 경우 진행")을 반영합니다. 코드 변경 유형별 갱신 대상 판단 기준:

| 변경 유형 | 갱신 대상 문서 |
| --- | --- |
| 신규 기능(feature) | SRS(요구사항 추가), 설계서, API 명세, 추적성, 테스트 계획, (필요 시) 아키텍처·ADR |
| 버그 수정(bugfix) | 추적성(테스트 추가), 테스트 결과서, (요구사항 오류였다면) SRS |
| 리팩터링(refactor) | 설계서, 추적성(코드 위치 갱신) |
| 성능/보안 개선 | 품질 모델(지표), 비기능 요구사항, 보안 문서 |
| API 계약 변경 | API 명세(버전↑), SRS(인터페이스), 추적성, 호환성 검토 |

- 갱신 시 반드시: 대상 문서의 버전·`작성일`·변경 이력 표를 갱신하고, 관련 요구사항 ID를 이력의 "관련 이슈/요구사항" 칸에 기록합니다.
- 문서 변경이 코드 변경과 같은 작업 단위에서 이루어지도록 하여 드리프트를 방지합니다.

## 도구·자동화

- **OpenAPI 생성**: `swaggo/swag`로 핸들러 주석에서 명세를 생성합니다.

```bash
go install github.com/swaggo/swag/cmd/swag@latest
swag init -g cmd/{{APP}}/main.go -o docs/api
```

- **커버리지 리포트**: 표준 도구 체인으로 생성·확인합니다.

```bash
go test ./... -coverprofile=docs/test/coverage.out
go tool cover -func=docs/test/coverage.out   # 함수별/총계
go tool cover -html=docs/test/coverage.out -o docs/test/coverage.html
```

- **정적분석·취약점 점검**:

```bash
go vet ./...
go run golang.org/x/vuln/cmd/govulncheck@latest ./...
```

- **CI에서의 문서/추적성 점검(권장)**:
  - PR에서 `go build ./...`, `go test ./...`, `go vet ./...`를 필수 게이트로 실행.
  - `swag init` 재생성 후 `git diff --exit-code docs/api`로 OpenAPI 최신화 여부 검사(드리프트 시 실패).
  - 추적성 매트릭스의 참조(요구사항 ID·코드 경로·테스트명·라우트)가 실제로 존재하는지 링크 체크 스크립트로 확인.
  - 커버리지 임계값 미달 시 실패 처리({{COVERAGE}}% 기준).
- 자동화의 목표는 "문서가 코드와 함께 갱신되지 않으면 CI가 실패하도록" 만들어 추적성과 정합성을 강제하는 것입니다.
