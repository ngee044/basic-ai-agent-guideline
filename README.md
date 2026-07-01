# basic-ai-agent-guideline

언어별 개발 가이드라인, 코드 규약(CODING_CONVENTION), ISO 문서 규약(ISO_DOCUMENTS_CONVENTION)의 "템플릿" 모음입니다.
실제 Go/Python/C++ 프로젝트 루트에 복사한 뒤, 프로젝트에 맞게 `{{PLACEHOLDER}}`를 채우고 수정해 사용합니다.

## 디렉터리 구조

| 디렉터리 | 설명 |
| --- | --- |
| `global_guideline/` | 모든 Claude Code 세션에 적용되는 사용자 전역 지침(`~/.claude/CLAUDE.md` 사본). |
| `cpp_guideline/` | C/C++ 프로젝트 가이드라인(`CLAUDE.md`, `CODING_CONVENTION.md`, `ISO_DOCUMENTS_CONVENTION.md`). |
| `golang_guideline/` | Go(Gin) REST API 프로젝트 가이드라인(`CLAUDE.md`, `CODING_CONVENTION.md`, `ISO_DOCUMENTS_CONVENTION.md`). |
| `python_guideline/` | Python(FastAPI) REST API 프로젝트 가이드라인(`CLAUDE.md`, `CODING_CONVENTION.md`, `ISO_DOCUMENTS_CONVENTION.md`). |

## 사용법

새 프로젝트에 적용할 때는 대상 언어 디렉터리의 파일을 다음 위치로 복사합니다.

- `CLAUDE.md` → 프로젝트 **루트**(`{{PROJECT_ROOT}}/CLAUDE.md`)
- `CODING_CONVENTION.md` → 프로젝트 **docs/** 폴더(`{{PROJECT_ROOT}}/docs/CODING_CONVENTION.md`)
- `ISO_DOCUMENTS_CONVENTION.md` → 프로젝트 **docs/** 폴더(`{{PROJECT_ROOT}}/docs/ISO_DOCUMENTS_CONVENTION.md`)

복사 후 문서 내 `{{PLACEHOLDER}}`(예: `{{PROJECT_NAME}}`, `{{MODULE_PATH}}`, `{{DB}}`)를 프로젝트 값으로 채워 사용합니다.
`global_guideline/CLAUDE.md`는 프로젝트가 아닌 사용자 홈(`~/.claude/CLAUDE.md`)에 배치하는 전역 지침입니다.

## 언어별 파일 링크

### Go(Gin)

- [golang_guideline/CLAUDE.md](golang_guideline/CLAUDE.md)
- [golang_guideline/CODING_CONVENTION.md](golang_guideline/CODING_CONVENTION.md)
- [golang_guideline/ISO_DOCUMENTS_CONVENTION.md](golang_guideline/ISO_DOCUMENTS_CONVENTION.md)

### Python(FastAPI)

- [python_guideline/CLAUDE.md](python_guideline/CLAUDE.md)
- [python_guideline/CODING_CONVENTION.md](python_guideline/CODING_CONVENTION.md)
- [python_guideline/ISO_DOCUMENTS_CONVENTION.md](python_guideline/ISO_DOCUMENTS_CONVENTION.md)

### C/C++

- [cpp_guideline/CLAUDE.md](cpp_guideline/CLAUDE.md)
- [cpp_guideline/CODING_CONVENTION.md](cpp_guideline/CODING_CONVENTION.md)
- [cpp_guideline/ISO_DOCUMENTS_CONVENTION.md](cpp_guideline/ISO_DOCUMENTS_CONVENTION.md)

### 전역 지침

- [global_guideline/CLAUDE.md](global_guideline/CLAUDE.md)
