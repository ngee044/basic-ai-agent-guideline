# CLAUDE.md — {{PROJECT_NAME}}

이 파일은 Claude Code가 이 프로젝트에서 작업할 때 따르는 지침입니다. Go(Gin) 기반 REST API 프로젝트의 루트에 두고, 아래 `{{PLACEHOLDER}}`를 프로젝트 실제 값으로 채워 사용합니다.

## 전역 지침 상속

- 이 프로젝트는 `global_guideline/CLAUDE.md`(사용자 전역 지침)를 상속합니다. 아래 규칙은 전역 지침에 종속되며, 충돌 시 전역 지침이 우선합니다.
- 한국어·존댓말로 응답하고, 불필요한 서론/요약은 생략합니다.
- 임의로 커밋·PR을 생성하지 않습니다. 커밋·PR은 사용자가 직접 수행하며, 사용자가 명시적으로 요청한 경우에만 대행합니다.
- 모든 분석·설계는 ISO 기반 문서로 작성하고 적합성·추적성을 확보합니다. 추적성의 우선순위는 0티어입니다.
- 문서는 프로젝트 하위 `docs/` 폴더에 저장하고, `docs/` 경로는 `.gitignore`에 추가하여 커밋되지 않도록 관리합니다.
- 추측하지 않습니다. 모르면 검색·확인 후 답변하고, 지시가 불명확하면 되물어봅니다.
- 컴파일/타입체크가 가능한 언어이므로, 작업 완료 전 반드시 빌드·검사를 통과(에러 0)시킵니다.
- 프로젝트 규약(`docs/CODING_CONVENTION.md`)과 외부 라이브러리 문서를 확인하고 준수합니다.

## 이 템플릿 사용법

1. `golang_guideline/CODING_CONVENTION.md`와 `golang_guideline/ISO_DOCUMENTS_CONVENTION.md`를 실제 프로젝트의 `docs/` 폴더로 복사합니다.
2. 이 `CLAUDE.md`를 실제 프로젝트 루트로 복사합니다.
3. 파일 내 `{{PLACEHOLDER}}`(예: `{{PROJECT_NAME}}`, `{{MODULE_PATH}}`, `{{APP}}`, `{{DB}}`, `{{CACHE}}`)를 프로젝트 값으로 채웁니다.
4. 기술 스택·명령·레이아웃이 프로젝트 실제와 다르면 해당 섹션을 조정합니다.

## 프로젝트 개요

- 이름: {{PROJECT_NAME}}
- 설명: {{DESCRIPTION}}
- Go module path: `{{MODULE_PATH}}`
- 주 진입점: `cmd/{{APP}}`

## 기술 스택

- 언어: Go 1.22+
- 웹 프레임워크: Gin (`github.com/gin-gonic/gin`)
- 요청 검증: go-playground/validator (Gin 기본 바인딩 검증기)
- 로깅: 표준 `log/slog` (구조적 로깅)
- 데이터베이스: {{DB}}
- 캐시: {{CACHE}}

## 빌드·실행·테스트

```bash
go build ./...                    # 전체 빌드 (에러 0 확인)
go run ./cmd/{{APP}}              # 로컬 실행
go test ./... -race -cover        # 테스트 (레이스 검출 + 커버리지)
go vet ./...                      # 정적 분석
gofmt -l .                        # 미포맷 파일 확인 (출력 없어야 정상)
golangci-lint run                 # 린터 (설정: .golangci.yml)
```

## 디렉터리 레이아웃

```
{{PROJECT_ROOT}}/
├── cmd/{{APP}}/       # 실행 바이너리 진입점(main). 얇게 유지: 설정 로드·의존성 조립·서버 기동
├── internal/          # 외부에 공개하지 않는 애플리케이션 코드
│   ├── handler/       # HTTP 핸들러·라우터 (Gin). 요청 파싱·검증·응답 매핑
│   ├── service/       # 도메인/비즈니스 로직. 인터페이스로 repository 소비
│   └── repository/    # DB·외부 시스템 접근 구현
├── pkg/               # 외부 프로젝트에서 재사용 가능한 공개 패키지(신중히 사용)
├── api/               # OpenAPI 스펙 등 API 계약 (예: api/openapi.yaml)
├── configs/           # 설정 파일 템플릿·예시
└── docs/              # ISO 문서 + 규약 (CODING_CONVENTION.md, ISO_DOCUMENTS_CONVENTION.md)
```

## MSA 구성 [선택]

이 템플릿의 계층형 구조(`handler → service → repository`)는 그대로 유지하되, **각 서비스 "내부"의 구조**로 봅니다. 서비스 분리(bounded context)·서비스 간 통신·데이터 소유권·운영(관측성·회복탄력성) 등 서비스 경계 관심사의 심층 원칙은 `msa_guideline/MSA_ARCHITECTURE.md`를 따릅니다. 기본 스탠스는 "modular-monolith-first, 정당화될 때 서비스로 추출"입니다.

Go 관점의 요점:

- 멀티 서비스 레이아웃: 서비스마다 `cmd/{{SERVICE_NAME}}/main.go` 진입점을 두고(monorepo 기준), 공유 코드는 `internal/`(또는 별도 모듈)에 둡니다. 다른 서비스의 `internal/` 패키지를 직접 import하지 않고 계약(API/이벤트 스키마)으로만 결합합니다.
- 서비스 간 통신: 즉시 응답이 필요하면 `net/http` 클라이언트 또는 gRPC로 동기 호출하고(모든 호출에 `context.WithTimeout` 필수), 상태 전파·느슨한 결합이 중요하면 {{BROKER}} 기반 비동기(event-driven)를 선호합니다.
- trace 전파: `context.Context`를 서비스 경계까지 전파하고, correlation ID/trace ID를 요청 헤더로 넘겨 분산 추적을 잇습니다(상세는 `CODING_CONVENTION.md`의 "MSA 확장" 절).
- 헬스체크·수명주기: 각 서비스는 `/healthz`(liveness)·`/readyz`(readiness)를 노출하고, 종료 시 graceful shutdown(기존 Gin 규약 재사용)으로 진행 중 요청/메시지를 마친 뒤 내려갑니다.

monorepo 멀티 서비스 트리 예시(단일 서비스 레이아웃 위에 서비스 경계를 얹은 형태):

```text
{{SYSTEM_NAME}}/
├── cmd/
│   └── {{SERVICE_NAME}}/       # 서비스별 진입점(main.go). {{SERVICE_LIST}}의 각 서비스마다 하나
├── internal/
│   ├── {{SERVICE_NAME}}/       # 서비스별 내부: handler/service/repository (서비스마다)
│   └── platform/              # 공유 인프라(로깅·trace·설정 등, 도메인 로직 아님)
├── api/                       # 서비스별 계약: OpenAPI(.yaml)·gRPC(.proto)
├── deploy/                    # 서비스별 컨테이너·오케스트레이션({{ORCHESTRATOR}})
└── docs/                      # 시스템/서비스 ISO 문서
```

## 핵심 규약 요약

상세 규약은 `docs/CODING_CONVENTION.md`를 참조합니다. 아래는 요약입니다.

- 인터페이스는 소비하는 쪽(service·handler)에서 정의하고 작게 유지합니다. 구현체가 아니라 소비자가 필요로 하는 최소 메서드만 담습니다.
- 에러는 `fmt.Errorf("...: %w", err)`로 래핑하여 원인을 보존하고, 경계에서 `errors.Is`/`errors.As`로 판별합니다.
- 계층은 `handler → service → repository` 단방향으로 흐르며, 상위 계층은 하위 계층을 인터페이스로 주입받습니다(인터페이스 기반 DI).
- 요청/응답 DTO는 도메인 모델과 분리하고, 요청 DTO에는 `binding` 태그로 검증 규칙을 명시합니다.
- 공개 API 경계로 넘어가는 모든 함수는 첫 인자로 `context.Context`를 받습니다.
- 포맷은 `gofmt`, 로깅은 `log/slog`를 사용합니다.

## ISO 문서

- ISO 문서 규약은 `docs/ISO_DOCUMENTS_CONVENTION.md`를 참조합니다.
- 필수 문서: SRS(`docs/requirements/SRS.md`), 아키텍처(`docs/architecture/ARCHITECTURE.md`), 상세 설계(`docs/design/DESIGN.md`), API 명세(`docs/api/openapi.yaml`), 테스트 계획/결과(`docs/test/`), 추적성 매트릭스(`docs/traceability/TRACEABILITY.md`).
- feature 추가·bugfix 시 관련 ISO 문서(요구사항·설계·API·추적성)를 함께 갱신하여 코드-문서 정합성을 유지합니다.

## 가드레일

- 작업 완료 전 다음을 모두 통과시킵니다: `gofmt -l .`(출력 없음), `go vet ./...`, `golangci-lint run`, `go test ./... -race`.
- 임의로 커밋·PR을 생성하지 않습니다(사용자가 직접 수행).
- 추측하지 않습니다. 불확실한 API·문법은 검색·확인 후 사용하고, 지시가 불명확하면 되물어봅니다.
