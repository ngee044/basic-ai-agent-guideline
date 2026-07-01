# ISO 문서 규약 (Python REST API 프로젝트)

FastAPI + Pydantic v2 기반 Python REST API 프로젝트에서 ISO/IEC/IEEE 표준을 근거로 요구사항·아키텍처·설계·테스트·추적성 문서를 일관되게 작성·유지하기 위한 규약입니다. ISO 문서는 언어 독립적이되, 본 문서의 예시는 Python 프로젝트 기준으로 구체화합니다.

## 목적과 적용 표준

- 프로젝트 산출물의 **적합성(conformance)** 과 **추적성(traceability)** 을 확보하기 위해 아래 표준을 근거로 문서를 작성합니다. 추적성은 최우선(0 티어) 관심사입니다.
- 표준은 형식주의가 아니라 "요구사항이 어떻게 코드·테스트·API로 실현되는가"를 증명하는 수단으로 사용합니다. 프로젝트 규모에 맞게 취사선택하되, 채택하지 않은 표준은 그 사유를 남깁니다.

| 표준 | 규정 범위 (1~2줄) |
| --- | --- |
| ISO/IEC/IEEE 12207 | 소프트웨어 생명주기 프로세스. 요구사항·설계·구현·검증·운영·유지보수 등 개발 활동의 프로세스 골격을 정의합니다. |
| ISO/IEC/IEEE 15288 | 시스템 생명주기 프로세스. 소프트웨어가 상위 시스템(인프라·외부 연동 포함)의 일부일 때 시스템 관점 프로세스를 규정합니다. 필요 시 적용합니다. |
| ISO/IEC/IEEE 29148 | 요구공학(Requirements Engineering). 요구사항의 도출·명세·검증 방법과 SRS/StRS 구조·품질 기준(명확성·검증가능성 등)을 규정합니다. |
| ISO/IEC/IEEE 42010 | 아키텍처 기술(Architecture Description). 이해관계자(stakeholder)·관심사(concern)·관점(viewpoint)·뷰(view)로 아키텍처를 기술하는 틀을 규정합니다. |
| ISO/IEC 25010 (SQuaRE) | 제품 품질 모델. 소프트웨어 제품 품질을 8개 특성으로 정의하여 비기능 요구사항·측정 지표의 근거를 제공합니다. |
| ISO/IEC 27001 | 정보보안 경영시스템(ISMS). 정보 자산의 기밀성·무결성·가용성 통제 요구사항을 규정합니다(보안 요구사항의 맥락 표준). |
| ISO 9001 | 품질경영시스템(QMS). 문서화·리뷰·개선 등 조직 차원의 품질 프로세스를 규정합니다(문서 관리 절차의 맥락 표준). |

## docs/ 폴더 구조

- 모든 ISO 문서는 프로젝트 루트 하위 `docs/`에 저장합니다.
- 전역 지침에 따라 `docs/` 경로는 `.gitignore`에 추가하여 문서가 커밋되지 않도록 관리합니다(사내 규정에 따라 별도 문서 저장소·리포지토리로 분리 운용 가능).
- 폴더는 표준 활동 단위로 구분하여 산출물의 소속과 추적 경로를 명확히 합니다.

```text
docs/
├── requirements/    # 요구사항: SRS(기능/비기능/제약), 유즈케이스, 요구사항 ID 대장 (29148)
├── architecture/    # 아키텍처 기술서: 이해관계자·관심사·관점·뷰, 결정 근거 (42010)
├── design/          # 상세 설계: 모듈/계층(router → service → repository), 데이터 모델, 시퀀스
├── api/             # API 명세: OpenAPI 스냅샷(openapi.json/yaml), 엔드포인트 명세, 오류 규격
├── test/            # 테스트 계획서/결과서: 테스트 전략, 케이스, 커버리지 리포트
├── traceability/    # 추적성 매트릭스: 요구사항 ↔ 설계 ↔ 코드 ↔ 테스트 ↔ API
├── quality/         # 품질 모델(25010): 품질 속성 목표·측정 지표·측정 결과
├── security/        # 보안 문서: 위협 모델, 인증/인가 설계, 27001 통제 매핑
├── operations/      # 운영: 배포·구성(uvicorn/ASGI), 모니터링, 구조적 로깅(logging/structlog), 롤백 절차
└── adr/             # Architecture Decision Records: 결정 사항과 배경·대안·결과
```

- 폴더별 역할
  - `requirements/`: 무엇을 만들 것인가. 요구사항 ID의 원천이며 모든 추적의 출발점입니다.
  - `architecture/`: 큰 그림. 계층형 아키텍처(router → service → repository)와 외부 시스템 경계를 기술합니다.
  - `design/`: 아키텍처를 모듈/함수·클래스 수준으로 구체화합니다. Pydantic 모델·`Protocol`/`ABC` 인터페이스 설계를 포함합니다.
  - `api/`: 대외 계약(contract). FastAPI가 생성하는 OpenAPI를 정본으로 삼습니다.
  - `test/`: 검증 활동의 계획과 증거.
  - `traceability/`: 표준 준수의 핵심 산출물. 각 요구사항이 끝까지 실현되었음을 증명합니다.
  - `quality/`: 비기능 목표(성능·신뢰성 등)와 측정치.
  - `security/`: 보안 요구·설계·통제 매핑.
  - `operations/`: 운영·배포·관측성.
  - `adr/`: 되돌아볼 수 있는 의사결정 기록.

## 필수 문서와 최소 목차

- 아래 문서는 REST API 프로젝트에서 필수로 유지합니다. 각 문서는 상단 메타 블록(문서 식별 섹션 참조)을 포함합니다.

### SRS — 소프트웨어 요구사항 명세서 (29148)

- 위치: `docs/requirements/SRS-001-*.md`
- 요구사항은 고유 ID를 부여하고, 검증 가능(verifiable)·모호하지 않게 기술합니다.
- 최소 목차
  1. 소개(목적, 범위, 정의·약어, 참조)
  2. 전체 설명(제품 개요, 사용자·이해관계자, 제약)
  3. 기능 요구사항(`FR-xxx`): 엔드포인트/유즈케이스 단위
  4. 비기능 요구사항(`NFR-xxx`): 25010 품질 속성과 연계(성능·보안·신뢰성 등)
  5. 인터페이스 요구사항(외부 API, DB `{{DB}}`, 메시지 브로커 등)
  6. 제약(Python 3.11+, 규제, 라이선스)
  7. 요구사항 ID 대장(추적 대상 목록)

### 아키텍처 기술서 (42010)

- 위치: `docs/architecture/ARC-001-*.md`
- 최소 목차
  1. 이해관계자(stakeholder)와 관심사(concern) 목록
  2. 관점(viewpoint) 정의: 각 관점이 어떤 관심사를 다루는지
  3. 뷰(view): 관점별 실제 기술
     - 논리 뷰: 계층(router → service → repository), 인터페이스(`Protocol`/`ABC`)
     - 프로세스/런타임 뷰: 요청 흐름, ASGI(uvicorn) 워커, 비동기 처리
     - 배포 뷰: 컨테이너·환경 구성
     - 데이터 뷰: 엔티티·Pydantic 스키마·영속 모델
  4. 아키텍처 결정(주요 결정은 `adr/`로 연결)
  5. 품질 속성 시나리오(25010 매핑)

### 상세 설계서

- 위치: `docs/design/DSN-001-*.md`
- 최소 목차
  1. 모듈/패키지 구조(`{{MODULE_PATH}}` 기준 트리)
  2. 계층별 책임과 의존 방향(상위→하위 단방향, 인터페이스로 역전)
  3. 데이터 모델(Pydantic v2 모델, 영속 모델, 매핑)
  4. 인터페이스 계약(`typing.Protocol`/`abc.ABC` 시그니처)
  5. 오류 처리 규격(예외 계층, HTTP 상태 매핑)
  6. 주요 시퀀스(요청→검증→서비스→리포지토리→응답)

### API 명세 (OpenAPI)

- FastAPI가 자동 생성하는 OpenAPI 문서를 **정본**으로 사용합니다. 런타임에 `/openapi.json`(및 `/docs`, `/redoc`)로 제공됩니다.
- 릴리스 시점의 스냅샷을 `docs/api/openapi.json`(또는 `.yaml`)으로 고정 보관하여 계약 변경 이력을 추적합니다.

```python
# FastAPI 앱에서 OpenAPI 스냅샷을 파일로 추출하는 스크립트 예시
import json
from pathlib import Path

from {{MODULE_PATH}}.main import app  # FastAPI 인스턴스

Path("docs/api/openapi.json").write_text(
    json.dumps(app.openapi(), ensure_ascii=False, indent=2),
    encoding="utf-8",
)
```

- 별도 명세서(`docs/api/API-001-*.md`) 최소 목차
  1. 공통 규약(base path, 버전 전략, 인증 헤더, 페이지네이션)
  2. 엔드포인트 목록(`METHOD /path` → 요구사항 ID)
  3. 요청/응답 스키마(Pydantic 모델 참조)
  4. 오류 응답 규격(코드, 메시지, 예시)

### 테스트 계획서 / 결과서

- 위치: `docs/test/TST-001-plan-*.md`, `docs/test/TST-001-report-*.md`
- 계획서 최소 목차: 범위·전략(단위/통합/계약), 환경, 테스트 케이스(요구사항 ID 매핑), 합격 기준, 커버리지 목표
- 결과서 최소 목차: 실행 요약(pass/fail), 커버리지 실측치, 미해결 결함, 요구사항별 검증 상태

### 추적성 매트릭스

- 위치: `docs/traceability/TRC-001-*.md`
- 최소 목차: 매핑 표(다음 섹션 참조), 미커버 요구사항 목록, 고아(orphan) 코드/테스트 목록

### 품질 모델 (25010)

- 위치: `docs/quality/QLT-001-*.md`
- 최소 목차: 8개 품질 특성별 목표·측정 지표·측정 방법·현재 값(품질 속성 매핑 섹션 참조)

## 문서 식별·버전·이력

- **문서 ID 체계**: `{유형}-{일련번호}` 형식. 예: `SRS-001`, `ARC-001`, `DSN-001`, `API-001`, `TST-001`, `TRC-001`, `QLT-001`, `ADR-001`.
- **요구사항 ID 체계**: 기능은 `FR-001`, 비기능은 `NFR-001` 형식. 이 ID가 추적의 기준 키가 됩니다.
- **버전**: 문서는 `x.y`(major.minor)를 기본으로 하며, API 계약처럼 외부 호환성이 중요한 산출물은 semver(`x.y.z`)를 사용할 수 있습니다. major는 구조·의미 변경, minor는 내용 보완입니다.
- **상단 메타 블록**: 모든 문서 최상단에 아래 표를 둡니다.

| 항목 | 값 |
| --- | --- |
| 문서 ID | SRS-001 |
| 버전 | 1.2 |
| 작성일 | 2026-01-01 |
| 작성자 | {{AUTHOR}} |
| 검토자 | {{REVIEWER}} |
| 상태 | Draft / Reviewed / Approved / Deprecated |

- **변경 이력 표**: 문서 하단에 유지합니다.

| 버전 | 일자 | 작성자 | 변경 내용 | 관련 ID |
| --- | --- | --- | --- | --- |
| 1.0 | 2026-01-01 | {{AUTHOR}} | 최초 작성 | FR-001~FR-010 |
| 1.1 | 2026-02-15 | {{AUTHOR}} | 인증 요구사항 추가 | FR-011 |

## 추적성(traceability) [핵심]

- 추적성은 본 규약의 0 티어 관심사입니다. 모든 요구사항은 아래 5단계로 **양방향(forward/backward)** 연결되어야 하며, 어느 단계에서도 끊기면 결함으로 간주합니다.

```text
요구사항(FR/NFR) ⇄ 아키텍처/설계(ARC/DSN) ⇄ 코드(모듈·함수·클래스)
                                          ⇄ 테스트(pytest 노드 ID) ⇄ API(FastAPI 라우트)
```

- 단계별 식별 규칙(Python 기준)
  - **요구사항**: `FR-xxx` / `NFR-xxx`
  - **설계**: 아키텍처/설계 문서 ID + 섹션(예: `DSN-001 §4.2`)
  - **코드**: 모듈 경로 + 심볼. 예: `{{MODULE_PATH}}/services/order_service.py::OrderService.create`
  - **테스트**: pytest 노드 ID. 예: `tests/services/test_order_service.py::test_create_order_success`
  - **API**: FastAPI 라우트를 `METHOD /path`로 표기. 예: `POST /orders`

- 추적성 매트릭스 표 예시

| 요구사항 ID | 설계 | 코드 위치 | 테스트 | API | 상태 |
| --- | --- | --- | --- | --- | --- |
| FR-001 | DSN-001 §4.1 | `{{MODULE_PATH}}/services/order_service.py::OrderService.create` | `tests/services/test_order_service.py::test_create_order_success` | `POST /orders` | Verified |
| FR-002 | DSN-001 §4.2 | `{{MODULE_PATH}}/services/order_service.py::OrderService.get` | `tests/api/test_orders.py::test_get_order_404` | `GET /orders/{id}` | Verified |
| NFR-001 | ARC-001 §5 | `{{MODULE_PATH}}/main.py::app` (미들웨어) | `tests/perf/test_latency.py::test_p95_under_200ms` | 전역 | In Progress |

- **미커버(orphan) 점검**: 매트릭스에 나타나지 않는 코드/테스트(고아)와 코드가 없는 요구사항을 정기적으로 식별합니다. 자동화 섹션 참조.

## 코드-문서 연계

- 코드·테스트·커밋이 요구사항 ID를 참조하도록 하여 추적성을 코드 레벨에서 유지합니다. 주석은 WHY가 비자명할 때만 달되, 추적 ID는 예외적으로 명시합니다.

- **코드 주석**: 요구사항과의 연결이 코드만으로 자명하지 않을 때 ID를 남깁니다.

```python
# GOOD: 비자명한 제약의 근거를 ID로 남김 (FR-011: 인증 실패 시 계정 잠금)
if attempts >= MAX_ATTEMPTS:  # FR-011
    raise AccountLockedError(user_id)

# BAD: 자명한 동작에 불필요한 주석 + ID 남용
count += 1  # count를 1 증가 (FR-001)
```

- **테스트명**: 테스트 함수명 또는 마커로 요구사항 ID를 드러냅니다. pytest 마커를 쓰면 필터링·집계가 쉽습니다.

```python
import pytest

@pytest.mark.requirement("FR-001")  # pyproject.toml의 [tool.pytest.ini_options] markers에 등록
def test_create_order_success() -> None:
    ...
```

- **독스트링 참조**: 서비스/인터페이스 구현부 독스트링에 관련 요구사항을 명시합니다.

```python
class OrderService:
    def create(self, cmd: CreateOrderCommand) -> Order:
        """주문을 생성합니다. 관련 요구사항: FR-001, FR-003."""
        ...
```

- **커밋 메시지 형식**: 전역 지침상 임의 커밋은 금지되며 커밋은 사용자가 직접 수행합니다. 본 규약은 **형식만** 규정합니다.

```text
<type>(<scope>): <제목>

<본문>

Refs: FR-001, FR-003
```

  - `type`: feat, fix, docs, refactor, test 등. `Refs:` 트레일러에 요구사항 ID를 나열하여 커밋↔요구사항 추적을 가능하게 합니다.

## 품질 속성 매핑 (25010)

- ISO/IEC 25010의 8개 제품 품질 특성을 REST API 관점으로 구체화하고, 검증 가능한 측정 지표를 부여합니다. NFR ID와 연결합니다.

| 품질 특성 | REST API 관점 구체화 | 측정 지표 예 |
| --- | --- | --- |
| 기능 적합성(Functional Suitability) | 명세된 엔드포인트가 계약(OpenAPI)대로 정확히 동작 | 계약 테스트 통과율 100%, 요구사항 커버리지 |
| 성능 효율성(Performance Efficiency) | 응답 지연·처리량·자원 사용 | 응답시간 p95 < 200ms, throughput(RPS), CPU/메모리 사용률 |
| 호환성(Compatibility) | API 버전 간 호환, 외부 시스템 연동 | 하위 호환 위반 0건, 스키마 breaking change 탐지 |
| 사용성(Usability) | API 사용 편의(문서·오류 메시지 명확성) | OpenAPI 문서 완전성, 오류 응답 규격 준수율 |
| 신뢰성(Reliability) | 가용성·장애 복원력 | 가용성 99.9%, 오류율 < 0.1%, MTTR |
| 보안(Security) | 인증·인가·데이터 보호(27001 연계) | 인증 우회 0건, 취약점 스캔 High 0건, 민감정보 로깅 0건 |
| 유지보수성(Maintainability) | 모듈성·테스트 용이성·분석성 | 테스트 커버리지 ≥ 80%, mypy `--strict` 통과, ruff 위반 0 |
| 이식성(Portability) | 환경 이식(컨테이너·설정 외부화) | 환경변수 기반 구성률, 컨테이너 이미지 정상 기동 |

- 각 지표는 `docs/quality/QLT-001`에 목표값·측정 방법·현재값을 기록하고, 성능·신뢰성 지표는 `docs/test/`의 결과서와 연결합니다.

## 검증·적합성

- **문서 리뷰 절차**
  1. 작성자가 문서 상태를 `Draft`로 등록하고 변경 이력에 기록합니다.
  2. 검토자가 29148/42010/25010 기준으로 명확성·검증가능성·완전성·추적성을 검토합니다.
  3. 지적 사항 반영 후 상태를 `Reviewed` → 승인 시 `Approved`로 전환합니다.
- **코드-문서 정합성 확인**
  - 추적성 매트릭스의 코드 위치·테스트 노드 ID·API 라우트가 실제 저장소와 일치하는지 확인합니다(경로/심볼 존재, 테스트 수집 가능, 라우트 등록 여부).
  - OpenAPI 스냅샷(`docs/api/openapi.json`)이 현재 앱의 `app.openapi()` 출력과 일치하는지 확인합니다.
- **Definition of Done (문서 갱신 포함)**
  - 관련 SRS/설계/추적성 매트릭스가 갱신되었다.
  - 요구사항 ID별로 코드·테스트·API가 연결되어 매트릭스 상태가 `Verified`이다.
  - `ruff` 위반 0, `mypy --strict` 통과, `pytest` 전부 통과, 커버리지 목표 충족.
  - OpenAPI 스냅샷과 API 명세가 최신이다.
  - 신규/변경 요구사항의 품질 지표(25010)가 측정·기록되었다.

## 문서 갱신 규칙

- feature/bugfix 등 코드 변경 시 어떤 문서를 갱신할지 아래 기준으로 판단하고, 갱신 시 반드시 변경 이력에 기록합니다(전역 지침: "수정 시 ISO 문서 갱신").

| 변경 유형 | 갱신 대상 문서 |
| --- | --- |
| 신규 기능(엔드포인트·유즈케이스 추가) | SRS(FR 추가), 설계서, API 명세/OpenAPI 스냅샷, 테스트, 추적성 매트릭스 |
| 비기능 목표 변경(성능·보안 등) | SRS(NFR), 품질 모델(QLT), 관련 테스트·측정 결과 |
| 계약 변경(요청/응답 스키마·상태코드) | API 명세, OpenAPI 스냅샷, 계약 테스트, 추적성 매트릭스 |
| 아키텍처/주요 결정 변경 | 아키텍처 기술서, ADR, 설계서 |
| bugfix(계약·요구사항 불변) | 테스트(회귀 케이스) 추가, 필요 시 설계서 주석; 요구사항 미변경 시 SRS 갱신 불필요 |

- 판단 원칙: "요구사항 또는 대외 계약이 바뀌면 상위 문서(SRS/API)까지, 내부 구현만 바뀌면 설계/테스트까지" 갱신합니다.

## 도구·자동화

- **OpenAPI 생성**: FastAPI의 `app.openapi()`로 스냅샷을 추출하는 스크립트를 리포지토리에 두고, CI에서 갱신 여부를 검증합니다.

```bash
# 스냅샷 생성 (uv 기준; 대안 poetry run)
uv run python -m scripts.dump_openapi
# CI: 스냅샷이 최신인지 확인 (변경분이 있으면 실패시켜 갱신 누락 방지)
uv run python -m scripts.dump_openapi && git diff --exit-code docs/api/openapi.json
```

- **커버리지 리포트**: `pytest-cov`로 측정하여 품질 지표를 산출합니다.

```bash
uv run pytest --cov={{MODULE_PATH}} --cov-report=term-missing --cov-fail-under=80
```

- **정적 검사**: `ruff`(린트·포맷), `mypy --strict`(타입체크)를 CI 게이트로 둡니다.

```bash
uv run ruff check .
uv run ruff format --check .
uv run mypy --strict {{MODULE_PATH}}
```

- **추적성 점검(CI)**
  - 추적성 매트릭스의 코드 위치·테스트 노드 ID가 실존하는지 검사합니다(예: `pytest --collect-only -q`로 테스트 노드 존재 확인, 경로 존재 확인 스크립트).
  - 문서 내 상대 링크의 깨짐 여부를 검사합니다(링크 체크 스크립트/도구).
  - 요구사항 ID 대장과 매트릭스를 대조하여 미커버 요구사항·고아 코드/테스트를 리포트합니다.
- **원칙**: 자동화는 "문서가 코드와 어긋나면 CI가 실패"하도록 구성하여 정합성을 지속적으로 강제합니다. 검증 스크립트 작성 시 토큰·분량 제한을 두지 않습니다.
