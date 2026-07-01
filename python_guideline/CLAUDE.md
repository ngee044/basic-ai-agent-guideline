# CLAUDE.md ({{PROJECT_NAME}})

Python REST API 프로젝트(FastAPI + Pydantic v2) 루트에 놓이는 Claude Code 작업 지침 템플릿입니다. 짧게 스캔 가능하도록 유지하며, 상세 규약은 `docs/` 하위 문서로 위임합니다.

## 전역 지침 상속

- 본 문서는 사용자 전역 지침(`~/.claude/CLAUDE.md`)을 상속하며, 상충 시 프로젝트 루트의 본 문서가 우선합니다.
- 응답·문서는 한국어, 격식(존댓말)으로 작성하고 불필요한 서론/요약은 생략합니다.
- 임의 커밋/PR 생성 금지: 사용자가 직접 수행합니다(단, 사용자 요청에 따른 커밋/PR은 유효).
- 모든 프로젝트 분석은 ISO 기반 문서로 작성하고 적합성·추적성을 확보합니다(추적성은 0 티어).
- 문서는 프로젝트 하위 `docs/` 폴더에 저장하며, `docs/` 경로는 `.gitignore`에 추가합니다.
- 주석은 WHY가 비자명할 때만 답니다.
- 추측 금지: 모르면 검색·확인 후 진행합니다. 지시가 불명확하면 되물어봅니다.
- 타입체크 가능한 언어이므로 완료 전 `mypy .`가 에러 0이어야 합니다.
- 프로젝트 규약(`docs/CODING_CONVENTION.md`)과 외부 라이브러리 문서를 확인하고 준수합니다.

## 이 템플릿 사용법

1. `CODING_CONVENTION.md`와 `ISO_DOCUMENTS_CONVENTION.md`를 프로젝트의 `docs/`로 복사합니다.
2. 본 `CLAUDE.md`를 프로젝트 루트로 복사합니다.
3. 각 문서의 `{{PLACEHOLDER}}`(예: `{{PROJECT_NAME}}`, `{{DESCRIPTION}}`, `{{DB}}`)를 프로젝트 값으로 채웁니다.
4. 채운 뒤 실제 레이아웃·명령·기술 스택과 일치하는지 검증합니다(문서-코드 정합성).

## 프로젝트 개요

- 이름: `{{PROJECT_NAME}}`
- 설명: `{{DESCRIPTION}}`
- 성격: 계층형(router → service → repository) REST API 서버, 인터페이스 기반 의존성 주입.

## 기술 스택

- Python 3.11+ (버전은 `pyproject.toml`의 `requires-python`에 고정).
- FastAPI: 웹 프레임워크(라우팅·`Depends` DI·OpenAPI).
- Pydantic v2: 요청/응답 스키마 검증·직렬화, `pydantic-settings`로 설정 로드.
- uvicorn: ASGI 서버.
- SQLAlchemy 2.0 (async, `AsyncSession`) + `{{DB}}` (async 드라이버), 마이그레이션은 `alembic`.
- uv: 패키지 관리·가상환경·실행(대안 `poetry`). 린트·포맷 `ruff`, 타입체크 `mypy --strict`, 테스트 `pytest`.

## 빌드·실행·테스트

uv 기준이며 각 줄에 poetry 대안을 병기합니다.

```bash
uv sync                                   # 의존성 설치 (poetry: poetry install)
uv run uvicorn app.main:app --reload      # 개발 서버 (poetry: poetry run uvicorn app.main:app --reload)
uv run pytest -q --cov                     # 테스트 + 커버리지 (poetry: poetry run pytest -q --cov)
uv run ruff check .                        # 린트 (poetry: poetry run ruff check .)
uv run ruff format .                       # 포맷 (poetry: poetry run ruff format .)
uv run mypy .                              # 타입 검사 (poetry: poetry run mypy .)
```

## 디렉터리 레이아웃

```text
{{PROJECT_NAME}}/
├── src/
│   └── app/
│       ├── main.py        # FastAPI 앱 생성, lifespan, 라우터 등록
│       ├── api/           # 라우터(엔드포인트) + deps.py(Depends 조립)
│       ├── core/          # 설정·로깅·예외 등 횡단 관심사
│       ├── models/        # SQLAlchemy ORM / 도메인 모델
│       ├── schemas/       # Pydantic 요청/응답 스키마 (DTO)
│       ├── services/      # 비즈니스 로직
│       ├── repositories/  # 데이터 접근 (Protocol + 구현)
│       └── db/            # 세션·엔진·마이그레이션 연동
├── tests/             # unit / integration
└── docs/              # ISO 문서 + 규약 (.gitignore 대상)
```

- 상세 레이아웃(`src` 레이아웃 채택 사유 포함)은 `docs/CODING_CONVENTION.md`를 참조합니다.

## 핵심 규약 요약

상세는 `docs/CODING_CONVENTION.md`를 참조합니다.

- 모든 공개 함수/메서드에 타입 힌트 필수, `mypy --strict` 통과(누락·`Any` 남용 금지).
- 인터페이스는 소비자(service) 측에서 `typing.Protocol`로 좁게 정의하고 구현에 의존하지 않습니다.
- 계층 의존은 단방향(router → service → repository)이며, 결합은 `Depends` 기반 DI로 주입합니다.
- Pydantic 스키마는 요청/응답/도메인(ORM)을 분리하고, ORM 모델을 API 응답으로 직접 노출하지 않습니다.
- `async` 엔드포인트에서 동기 블로킹 I/O(동기 DB 드라이버, `requests`, `time.sleep`) 호출 금지.
- 도메인 예외 → HTTP 변환(`HTTPException`)은 API 계층에서만 수행합니다.

## ISO 문서

- ISO 문서 규약은 `docs/ISO_DOCUMENTS_CONVENTION.md`를 참조합니다.
- feature/bugfix 등 변경 시 관련 요구사항·설계·API·추적성 문서를 갱신합니다.

## 가드레일

- 완료 전 `uv run ruff check .`, `uv run mypy .`, `uv run pytest`가 모두 통과해야 합니다.
- 임의 커밋/PR 생성 금지(사용자가 직접 수행).
- 추측 금지: 불확실하면 검색·확인 후 진행하고, 불명확한 지시는 되물어봅니다.
