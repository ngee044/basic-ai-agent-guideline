# CLAUDE.md ({{PROJECT_NAME}})

CppToolkit(C++23) 기반 C/C++ REST/서버 프로젝트 루트에 놓이는 Claude Code 작업 지침 템플릿입니다. 짧게 스캔 가능하도록 유지하며, 상세 규약은 `docs/` 하위 문서로 위임합니다.

## 전역 지침 상속

- 본 문서는 사용자 전역 지침(`global_guideline/CLAUDE.md`)을 상속하며, 상충 시 프로젝트 루트의 본 문서가 우선합니다.
- 응답·문서는 한국어, 격식(존댓말)으로 작성하고 불필요한 서론/요약은 생략합니다.
- 임의 커밋/PR 생성 금지: 사용자가 직접 수행합니다(단, 사용자 요청에 따른 커밋/PR은 유효).
- 모든 프로젝트 분석은 ISO 기반 문서로 작성하고 적합성·추적성을 확보합니다(추적성은 0 티어).
- 문서는 프로젝트 하위 `docs/` 폴더에 저장하며, `docs/` 경로는 `.gitignore`에 추가합니다.
- 주석은 WHY가 비자명할 때만 답니다.
- 추측 금지: 모르면 검색·확인 후 진행합니다. 지시가 불명확하면 되물어봅니다.
- 컴파일 가능한 언어이므로 완료 전 반드시 `./build.sh`(또는 `cmake --build`) 빌드가 성공(에러 0)해야 합니다.
- 프로젝트 규약(`docs/CODING_CONVENTION.md`)과 외부 라이브러리 문서를 확인하고 준수합니다.

## 이 템플릿 사용법

1. `cpp_guideline/`의 `CODING_CONVENTION.md`(스타일·관용구)와 `ISO_DOCUMENTS_CONVENTION.md`(문서 규약), `.clang-format`을 프로젝트로 복사합니다: 규약 문서 2개는 `docs/`로, `.clang-format`은 프로젝트 루트로 둡니다.
2. 본 `CLAUDE.md`를 프로젝트 루트로 복사합니다.
3. 각 문서의 `{{PLACEHOLDER}}`(예: `{{PROJECT_NAME}}`, `{{PROJECT_PREFIX}}`, `{{DESCRIPTION}}`, `{{MODULE_NAME}}`, `{{NAMESPACE}}`, `{{FRAMEWORKS}}`)를 프로젝트 값으로 채웁니다.
4. 채운 뒤 실제 레이아웃·명령·기술 스택과 일치하는지 검증합니다(문서-코드 정합성).
5. CppToolkit(또는 유사 툴킷)을 서브모듈로 사용하는 경우, 폴더명↔링크 타겟명↔네임스페이스 매핑과 모듈별 API 반환 타입 표를 담은 프로젝트별 `docs/module_usage_guide.md`(`{{PROJECT_PREFIX}}-USAGE-001`) 작성을 권장합니다. flat include·타겟명·반환 타입 불일치가 신규 코드의 가장 흔한 컴파일 실패 원인입니다.

## 프로젝트 개요

- 이름: `{{PROJECT_NAME}}`
- 설명: `{{DESCRIPTION}}`
- 성격: CppToolkit(C++23) 기반 다중 프로세스 서버/클라이언트. (선택) REST API가 있으면 Go(Gin) 또는 C++로 병기.
- 아키텍처 컴포넌트(예시 — 프로젝트에 맞게 교체):
  - `{{MODULE_NAME}}`(예: MainServer): Boost.Asio TCP 서버, 세션 관리·브로드캐스트.
  - `{{MODULE_NAME}}`(예: MainServerConsumer): RabbitMQ 소비, 검증 후 Redis/PostgreSQL 저장.
  - `{{MODULE_NAME}}`(예: UserClient): TCP 클라이언트.
  - `{{MODULE_NAME}}`(예: CommonModule): 공통 메시지 파싱/실행 콜백.
  - (선택) `RestAPI`(예: Go/Gin): HTTP 진입점 → 메시지 큐 발행.
  - `.CppToolkit/` 또는 `.cpp_tool_kit/`: 공유 라이브러리 서브모듈.

## MSA 구성

- 각 프로세스/실행 파일(예: MainServer, MainServerConsumer, UserClient, (선택) RestAPI)을 하나의 **서비스 경계**로 취급합니다. 기존 다중 프로세스 구조가 곧 서비스 분할의 출발점이며, 계층형(handler/router → service → repository) 구조는 각 서비스 **내부** 구조로 그대로 유지합니다.
- 외부 클라이언트 진입은 REST **게이트웨이**(`{{GATEWAY}}` — Go/Gin 또는 C++)로 단일화하고, 내부 서비스를 외부에 직접 노출하지 않습니다.
- 서비스 간 통신은 상태 전파·느슨한 결합이 중요하면 `{{BROKER}}`(RabbitMQ/Kafka) 기반 **비동기**를, 즉시 응답이 필요하면 Boost.Asio/REST 기반 **동기**를 사용합니다(동기 호출 체인은 짧게 유지).
- 기존 규약(오류 `std::expected`/`tuple`, 수명주기 설정→start→사용→stop→reset, Logger 싱글턴, 모듈별 종료 API 차이)을 그대로 재사용하며, 그 위에 계약·멱등성·관측성 전파·회복탄력성·독립 배포를 더합니다.
- 채택 기준(기본 스탠스: modular-monolith-first, 정당화될 때 서비스로 추출)과 패턴 상세는 `msa_guideline/MSA_ARCHITECTURE.md`를 참조합니다.

## 기술 스택

- 언어 표준: C++23 (`CMAKE_CXX_STANDARD 23`).
- 빌드: CMake 3.18+ + Ninja 생성기.
- 패키지: vcpkg(`~/vcpkg/scripts/buildsystems/vcpkg.cmake`, `vcpkg.json` 매니페스트).
- 컴파일러: Clang(macOS) / GCC 13+(Linux/CI).
- 프레임워크·인프라: `{{FRAMEWORKS}}`(예: Boost.Asio, RabbitMQ, Redis, PostgreSQL).
- 공유 라이브러리: CppToolkit 서브모듈(선택) — Utilities/Thread/Network/Database/Redis/RabbitMQ/Kafka 모듈.
- (선택) REST API가 Go면 Go 1.23+ / Gin 병기, 포맷은 `gofmt`.
- 테스트: Google Test(gtest) via `ctest`.

## 빌드·실행·테스트

```bash
git submodule update --init --recursive   # 서브모듈(CppToolkit 등) 초기화

./build.sh                                # 프로젝트 표준 빌드 스크립트

# 또는 수동 빌드
cmake -S . -B build -G Ninja \
  -DCMAKE_TOOLCHAIN_FILE=$HOME/vcpkg/scripts/buildsystems/vcpkg.cmake \
  -DCMAKE_BUILD_TYPE=Release
cmake --build build -j                    # 병렬 빌드 (에러 0 확인)

ctest --output-on-failure                 # 테스트 (build 디렉터리에서, gtest)

clang-format -i <file>                    # 포맷 정렬 (.clang-format 기준)
```

- 모듈 on/off는 configure 시점 옵션으로 제어합니다(예: `-DBUILD_NETWORK_LIB=OFF`, `-DBUILD_SAMPLES=OFF` — 프로젝트 옵션명에 맞게).
- `vcpkg.json`을 변경했다면 의존성 재설치가 필요합니다(예: `FRESH_DEPS=1 ./build.sh` — 프로젝트 스크립트 규약에 맞게).

## 디렉터리 레이아웃

```text
{{PROJECT_ROOT}}/
├── {{MODULE_NAME}}/         # 서비스/실행 파일 (예: MainServer/ — .h/.cpp + main.cpp)
├── {{MODULE_NAME}}/         # 서비스/실행 파일 (예: MainServerConsumer/)
├── CommonModule/            # 공통 모듈 (콜백 타입 정의, 메시지 파싱/실행)
├── .CppToolkit/             # 공유 라이브러리 서브모듈 (경로는 프로젝트마다 상이)
├── RestAPI/                 # (선택) Go/Gin REST API
├── database/                # (선택) 스키마 (schema.sql 등)
├── docker/                  # (선택) docker-compose + 운영 스크립트
├── build/                   # 빌드 산출물 (out/ 실행파일, lib/ 라이브러리) — 생성물
├── .clang-format            # 포맷 설정 (cpp_guideline에서 복사)
├── CMakeLists.txt           # 루트: option() + add_subdirectory()로 모듈 등록
├── vcpkg.json               # vcpkg 매니페스트
└── docs/                    # ISO 문서 + 규약 (.gitignore 대상)
    ├── CODING_CONVENTION.md          # {{PROJECT_PREFIX}}-CONV-001
    ├── ISO_DOCUMENTS_CONVENTION.md   # {{PROJECT_PREFIX}}-DOCS-001
    └── module_usage_guide.md         # {{PROJECT_PREFIX}}-USAGE-001 (툴킷 사용 시 권장)
```

## 핵심 규약 요약

상세는 `docs/CODING_CONVENTION.md`를, 모듈 사용법·링크 규약은 `docs/module_usage_guide.md`를 참조합니다. 아래는 요약입니다.

- **함수 시그니처**: 모든 함수/메서드는 후행 반환 타입(`auto name(params) -> Ret`)을 사용하고, 빈 매개변수는 `()`가 아니라 `(void)`로 명시합니다. getter/setter는 `get_`/`set_` 접두사 없이 같은 이름 오버로드(인자 있으면 setter, `(void) const`면 getter).
- **명명**: 클래스/파일은 PascalCase, 함수/변수는 snake_case, 멤버 변수는 snake_case + `_` 접미사(필수) — getter/setter 오버로드에서 함수명과 멤버명을 구분하는 근거입니다.
- **오류 처리**: 성공/실패만이면 `std::expected<void, std::string>`(성공 `{}`, 실패 `std::unexpected("사유")`), 데이터+실패면 `std::tuple<std::optional<T>, std::optional<std::string>>`. 비동기/네트워크 경로에서는 예외를 던지지 않고 반환값으로 실패를 표현합니다. (애플리케이션 계층은 `std::tuple<bool, std::optional<std::string>>` 관례를 쓰기도 하니 대상 프로젝트 규약을 따릅니다.)
- **헤더/include**: 헤더 최상단은 `#pragma once`, include는 flat(`#include "Logger.h"` — 경로 접두어 금지), 프로젝트 헤더는 큰따옴표·표준/서드파티는 꺾쇠, `SortIncludes: false`이므로 수동 정렬. 헤더에서 `using namespace` 금지.
- **빌드/링크(툴킷 3대 함정)**: 폴더명 ≠ 링크 타겟명 ≠ 네임스페이스(예: `ThreadPool/` → 타겟 `Thread` → ns `Thread`), vcpkg 패키지명 ≠ `find_package` 이름 ≠ 링크 타겟. `target_link_libraries`에는 타겟명을 씁니다. 매핑 표는 `docs/module_usage_guide.md`.
- **수명주기**: 리소스 객체(Logger, ThreadPool, Network*, Redis, RabbitMQ, Kafka)는 명시적 `설정 → start → 사용 → stop → reset` 순서. 종료 API는 모듈마다 다릅니다(ThreadPool은 `wait_stop` 없이 `stop()`이 블로킹, Network/RabbitMQ/Kafka는 `wait_stop()`).
- **포맷**: `.clang-format`(GNU 기반, Tab 들여쓰기 폭 4, `UseTab: Always`, 170 컬럼, Allman 중괄호, `PointerAlignment: Left`)에 위임. 커밋 전 `clang-format -i`.

## ISO 문서

- ISO 문서 규약은 `docs/ISO_DOCUMENTS_CONVENTION.md`를 참조합니다.
- 필수 문서 세트: SRS(`{{PROJECT_PREFIX}}-SRS-001`, 29148), SAD(`{{PROJECT_PREFIX}}-SAD-001`, 42010), STP/STR(`{{PROJECT_PREFIX}}-STP-001`, 29119-3), RTM(`{{PROJECT_PREFIX}}-RTM-001`, 추적성).
- feature 추가·bugfix 시 관련 SRS/SAD/STP/RTM을 함께 갱신하여 요구사항 → 설계 → 테스트 → 소스코드 양방향 추적성과 코드-문서 정합성을 유지합니다.

## 가드레일

- 변경은 반드시 `./build.sh`(또는 `cmake --build build -j`) 빌드가 성공(에러 0)해야 합니다.
- 변경 모듈에 해당하는 `Samples/` 실행 또는 `ctest --output-on-failure`로 검증하고, 커밋 전 `clang-format -i`를 적용합니다.
- 임의 커밋/PR 생성 금지(사용자가 직접 수행).
- 추측 금지: 불확실한 API·반환 타입은 `docs/module_usage_guide.md`·헤더·Samples/·검색으로 확인 후 사용하고, 불명확한 지시는 되물어봅니다.
