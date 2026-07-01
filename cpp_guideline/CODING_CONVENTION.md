# C++ 코딩 규약 (Coding Convention)

| 항목 | 내용 |
|------|------|
| 문서 ID | `{{PROJECT_PREFIX}}-CONV-001` |
| 대상 | {{PROJECT_NAME}} (CppToolkit 기반 C++23 프로젝트: 라이브러리 모듈 + 애플리케이션 서버/클라이언트) |
| 언어 표준 | C++23 (`CMAKE_CXX_STANDARD 23`) |
| 포맷터 | `.clang-format` (GNU 기반, Tab 들여쓰기, 170 컬럼) |
| 상태 | 활성 (Active) |

> 본 문서는 **CppToolkit 및 그 소비 애플리케이션에 실제로 적용된 관례를 추출·일반화한 범용 템플릿**입니다. 새 C++ 프로젝트의 `docs/`로 복사해 `{{PLACEHOLDER}}`(예: `{{PROJECT_NAME}}`, `{{PROJECT_PREFIX}}`, `{{MODULE_NAME}}`, `{{NAMESPACE}}`)를 채워 사용합니다. 규약 자체(C++23, `.clang-format` 설정, 후행 반환 타입, `std::expected`/`tuple` 오류 규약, `#pragma once`, 명명 규칙 등)는 보편 규칙이므로 그대로 유지합니다. 새 코드는 본 문서를 따르고, 기존 코드를 수정할 때는 주변 코드의 스타일에 맞춥니다. 신규 규칙 제안은 PR 논의를 통해 본 문서를 갱신한 뒤 적용합니다.

---

## 1. 적용 범위 (Scope)

- 본 컨벤션은 프로젝트의 모든 C++ 소스(`*.h`, `*.hpp`, `*.cpp`)와 `Samples/`(있는 경우) 예제에 적용됩니다.
- `vcpkg_installed/`, `build/`, 서브모듈(예: `.CppToolkit/`, `.cpp_tool_kit/`) 등 생성물·외부 의존성에는 적용하지 않습니다. 서브모듈은 해당 저장소의 규약을 따릅니다.
- 포맷팅(들여쓰기, 줄바꿈, 중괄호 위치)은 모두 `.clang-format`에 위임합니다. 본 문서는 포맷터가 강제할 수 없는 **명명·구조·관용구**를 규정합니다.
- 짝 문서: 모듈 사용법·CMake 작성·링크 규약(폴더명↔타겟명↔네임스페이스 매핑, vcpkg↔`find_package`↔링크 타겟 매핑, 모듈별 API 반환 타입 표)은 프로젝트별 `module_usage_guide.md`(`{{PROJECT_PREFIX}}-USAGE-001`)에 둡니다. 본 문서(스타일·관용구)와 그 문서(사용법·빌드)는 짝을 이룹니다.

---

## 2. 파일 및 디렉터리 구성 (File / Directory Layout)

- 모듈은 최상위 디렉터리 단위로 분리합니다(예: `{{MODULE_NAME}}/`). 한 모듈이 하나의 CMake 라이브러리 타겟으로 빌드됩니다.
- **한 클래스 = 한 쌍의 파일** (`ClassName.h` + `ClassName.cpp`). 선언과 정의를 분리합니다.
- 파일명은 클래스명과 동일한 **PascalCase**로 합니다: `ThreadPool.h`, `NetworkSession.cpp`, `LogTypes.h`.
- 순수 enum/상수/타입 정의 전용 헤더는 접미사로 의도를 드러냅니다: `LogTypes.h`, `DataModes.h`, `NetworkConstexpr.h`, `Protocol.h`, `JobPriorities.h`.
- 템플릿 전용 정의(선언과 정의를 헤더에서 분리할 수 없는 경우)는 `.hpp` 확장자를 사용합니다(예: `DeliveryResult.hpp`, 애플리케이션의 `ModuleHeader.hpp`).
- 각 모듈 루트에 `CMakeLists.txt`를 두고, 모듈별 사용 설명이 필요하면 `README.md`를 둡니다.

---

## 3. 명명 규칙 (Naming)

| 대상 | 규칙 | 예시 |
|------|------|------|
| 클래스 / 구조체 | PascalCase | `ThreadPool`, `NetworkSession`, `RedisClient` |
| 파일 | PascalCase (클래스명 일치) | `JobPool.cpp` |
| 함수 / 메서드 | snake_case | `push`, `remove_workers`, `thread_title` |
| 지역 변수 / 매개변수 | snake_case | `backup_folder`, `stop_immediately` |
| 멤버 변수 | snake_case + **`_` 접미사** | `thread_title_`, `job_pool_`, `working_` |
| enum class | PascalCase 타입 + PascalCase 값 | `LogTypes::Warning`, `JobPriorities::Top` |
| 네임스페이스 | PascalCase | `{{NAMESPACE}}` (예: `Utilities`, `Thread`, `Network`) |
| 매크로 / 컴파일 플래그 | UPPER_SNAKE_CASE | `USE_ENCRYPT_MODULE`, `WSTRING_IS_U16STRING` |

- 멤버 변수의 `_` 접미사는 **필수**입니다. getter/setter 오버로드(§6)에서 함수명과 멤버명을 구분하는 근거가 됩니다.
- **모듈명(폴더명)과 네임스페이스명이 항상 일치하지는 않습니다.** 대표적으로 `ThreadPool/` 폴더의 네임스페이스는 **`Thread`** 입니다. 신규 파일 작성 시 디렉터리 이름이 아니라 **기존 동일 모듈 파일의 네임스페이스**를 따릅니다. 폴더명↔링크 타겟명↔네임스페이스 매핑 표는 프로젝트별 `module_usage_guide.md`에 둡니다(§15).

---

## 4. 포맷팅 (Formatting) — `.clang-format` 요약

포맷팅은 전적으로 `.clang-format`이 결정합니다. 커밋 전 `clang-format -i <file>`로 정렬합니다. 핵심 설정:

- `BasedOnStyle: GNU`, `IndentWidth: 4`, `TabWidth: 4`, **`UseTab: Always`** (들여쓰기는 탭).
- `ColumnLimit: 170`.
- **Allman 중괄호** (`BreakBeforeBraces: Custom`) — 함수·클래스·구조체·enum·제어문·네임스페이스·람다의 여는 중괄호는 다음 줄에 둡니다.
- `NamespaceIndentation: All` — 네임스페이스 내부 내용도 한 단계 들여씁니다.
- `PointerAlignment: Left` — `int* p` (포인터/참조 기호는 타입에 붙입니다).
- `SortIncludes: false` — include는 **자동 정렬하지 않습니다**(§8 참조, 수동 정렬).
- `BinPackParameters: false` — 인자가 길어지면 한 줄에 하나씩 나눕니다.
- `SpaceBeforeParens: ControlStatements` — `if (`, `for (`처럼 제어문 키워드 뒤에만 공백을 둡니다(함수 호출 `f(`에는 공백 없음).
- `AccessModifierOffset: -4` — `public:`/`private:`는 한 단계 내어씁니다.
- `AlignTrailingComments: Always` — 후행 주석을 정렬합니다.
- `Standard: c++23`.

`.clang-format`은 프로젝트 루트(또는 서브모듈이 제공하는 것)를 그대로 사용합니다. 임의로 설정을 바꾸지 않습니다.

---

## 5. 함수 시그니처 (Function Signatures)

### 5.1 후행 반환 타입 (Trailing Return Type) — 필수

모든 함수/메서드는 **`auto name(params) -> ReturnType`** 형식을 사용합니다. 선행 반환 타입은 사용하지 않습니다.

```cpp
auto push(std::shared_ptr<Job> job) -> std::expected<void, std::string>;
auto thread_title(void) const -> const std::string;
static auto split(const std::string& source, const std::string& token) -> std::vector<std::string>;
```

### 5.2 빈 매개변수 목록은 `(void)`

매개변수가 없으면 `()`가 아니라 **`(void)`** 로 명시합니다. 소멸자도 동일합니다.

```cpp
{{MODULE_NAME}}(const std::string& title = "{{MODULE_NAME}}");
virtual ~{{MODULE_NAME}}(void);
auto start(void) -> std::expected<void, std::string>;
```

### 5.3 매개변수 전달

- 읽기 전용 비-trivial 타입은 `const T&`로 받습니다: `const std::string&`, `const std::vector<uint8_t>&`.
- 소유권을 공유하는 `shared_ptr`는 값으로 받습니다: `std::shared_ptr<Job> job`.
- 기본 인자(default argument)는 **헤더 선언부에만** 둡니다(`.cpp` 정의부에는 다시 적지 않습니다).
- 콜백은 `const std::function<...>&`로 받아 멤버에 저장합니다.

---

## 6. getter / setter 오버로드 패턴

하나의 속성에 대해 **같은 이름의 함수 2개**를 오버로드합니다. 인자가 있으면 setter, `(void) const`이면 getter입니다.

```cpp
auto life_cycle(uint16_t cycle) -> void;        // setter
auto life_cycle(void) const -> uint16_t;        // getter

auto thread_title(const std::string& title) -> void;   // setter
auto thread_title(void) const -> const std::string;    // getter
```

- getter는 가능한 한 `const`로 선언합니다.
- 별도의 `get_`/`set_` 접두사는 쓰지 않습니다.

---

## 7. 오류 처리 (Error Handling)

> 대원칙: **예외를 던지지 않고, 반환값으로 실패를 말합니다.** 비동기·네트워크·백그라운드 스레드 경계에서 예외를 던지면 콜백 스택이 끊기고 리소스가 샙니다. 그래서 실패는 던지지 않고 반환합니다.

### 7.1 표준: `std::expected<void, std::string>`

부수효과만 있고 실패할 수 있는 API는 **`std::expected<void, std::string>`** 를 반환합니다(현행 표준, 라이브러리 계층 전반에서 사용). 성공 시 `{}`, 실패 시 `std::unexpected("사유")`.

```cpp
auto {{MODULE_NAME}}::push(std::shared_ptr<Job> job) -> std::expected<void, std::string>
{
	if (job_pool_ == nullptr)
	{
		return std::unexpected("cannot push a job into null JobPool");
	}
	return job_pool_->push(job);
}

auto {{MODULE_NAME}}::start(void) -> std::expected<void, std::string>
{
	// ...
	auto result = worker->start();
	if (!result)
	{
		return std::unexpected(result.error());   // 오류 전파
	}
	return {};   // 성공
}
```

호출 측 관용구:

```cpp
if (auto result = pool->start(); !result)
{
	Logger::handle().write(LogTypes::Error, result.error());
	return;
}
```

### 7.2 데이터 반환 + 오류: `std::tuple<std::optional<T>, std::optional<std::string>>`

값을 돌려주면서 실패도 표현해야 할 때는 `std::optional<결과>`와 `std::optional<오류문자열>`을 묶은 `tuple`을 씁니다. 성공 시 `{ value, std::nullopt }`, 실패 시 `{ std::nullopt, "사유" }`.

```cpp
auto read_bytes(void) -> std::tuple<std::optional<std::vector<uint8_t>>, std::optional<std::string>>;
auto remove_workers(JobPriorities priority) -> std::tuple<size_t, std::optional<std::string>>;
```

호출 측 관용구:

```cpp
auto [maybe_bytes, error] = file.read_bytes();
if (maybe_bytes.has_value())
{
	// maybe_bytes.value() 사용
}
```

> 신규 코드의 "성공/실패만" 표현에는 `std::expected`를 우선합니다. `tuple` 형태는 결과 데이터를 함께 돌려줘야 할 때로 한정합니다.

### 7.3 비동기 / 네트워크 경로에서 예외 금지

Network·RabbitMQ·Kafka 등 비동기/콜백 경로에서는 예외를 던지지 않습니다. 외부 라이브러리 호출 등 예외가 발생할 수 있는 지점은 `try/catch`로 감싸고 오류를 `expected`/`tuple`로 변환해 반환합니다. 콜백 시그니처 자체가 `std::expected<void, std::string>`을 반환하도록 설계합니다(Job 콜백, 네트워크 콜백, 컨슈머 콜백 등).

### 7.4 방어적 null 검사

`shared_ptr` 멤버를 역참조하기 전에 항상 null을 확인합니다. 치명적이지 않으면 경고 로그를 남기고 안전한 기본값으로 빠져나갑니다.

```cpp
if (job_pool_ == nullptr)
{
	Logger::handle().write(LogTypes::Warning, "cannot get uncompleted jobs by null job_pool");
	return {};
}
```

> **각주(애플리케이션 계층 관례).** 애플리케이션 계층(예: 공용 모듈의 메시지 파싱/실행 콜백)에서는 `std::expected` 도입 이전의 관례로 **`std::tuple<bool, std::optional<std::string>>`** (성공 여부 `bool` + 오류 문자열 `optional`)를 콜백 반환 타입으로 쓰는 경우가 있습니다. 프로젝트의 기존 콜백 시그니처가 이 형태라면 일관성을 위해 그대로 따르되, 신규 라이브러리 계층 API는 `std::expected`를 우선합니다.

---

## 8. 헤더 및 include (Headers / Includes)

- 모든 헤더 최상단은 **`#pragma once`** 로 합니다(include guard 매크로 사용 안 함).
- include는 **수동 정렬**합니다(`SortIncludes: false`). 일반적 순서:
  1. 짝이 되는 헤더 (`.cpp`의 첫 줄은 같은 이름의 `.h`)
  2. 같은 프로젝트의 헤더 — **큰따옴표** `"JobPool.h"`
  3. 표준 라이브러리 — **꺾쇠** `<memory>`, `<expected>`
  4. 서드파티 (Boost 등) — 꺾쇠 `<boost/asio/steady_timer.hpp>`
- **flat include(평평한 포함).** 프로젝트 헤더는 경로 접두어 없이 파일명만 씁니다: `#include "Logger.h"` (O), `#include "Utilities/Logger.h"` (X, 컴파일 에러). 각 모듈이 `target_include_directories(... PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})`로 자기 소스 디렉터리를 공개하기 때문입니다. 이 규약은 CMake 작성 규약(§15)과 짝을 이룹니다.
- 헤더에서는 가능하면 **전방 선언(forward declaration)** 으로 의존성을 줄입니다.

```cpp
namespace {{NAMESPACE}}
{
	class Job;            // 전방 선언
	class JobPool;
	class ThreadWorker;
	class ThreadPool : public std::enable_shared_from_this<ThreadPool>
	{
		/* ... */
	};
} // namespace {{NAMESPACE}}
```

- 관련 멤버를 묶어 가독성을 높일 때 `#pragma region` / `#pragma endregion`을 사용합니다(예: `Logger`의 `#pragma region Handle`).

---

## 9. 네임스페이스 (Namespaces)

- 모든 라이브러리 코드는 모듈 네임스페이스로 감쌉니다: `{{NAMESPACE}}`.
- `NamespaceIndentation: All`에 따라 내부 내용을 한 단계 들여씁니다.
- 네임스페이스 닫는 중괄호 뒤에는 주석으로 이름을 표기합니다: `} // namespace {{NAMESPACE}}`.
- `.cpp`에서는 자주 쓰는 의존 네임스페이스를 파일 상단에서 `using`으로 끌어올 수 있습니다: `using namespace Utilities;`. **헤더에서는 `using namespace`를 금지**합니다.

---

## 10. 메모리 / 수명 관리 (Memory & Lifetime)

- 공유 소유권은 `std::shared_ptr`, 단독 소유권은 `std::unique_ptr`를 씁니다.
- 부모-자식 등 **순환 참조**는 `std::weak_ptr`로 끊습니다.
- 자기 자신을 콜백/하위 객체에 넘겨야 하는 클래스는 `std::enable_shared_from_this<T>`를 상속하고, **`get_ptr()`** 메서드로 `shared_from_this()`를 노출합니다.

```cpp
class {{MODULE_NAME}} : public std::enable_shared_from_this<{{MODULE_NAME}}>
{
public:
	auto get_ptr(void) -> std::shared_ptr<{{MODULE_NAME}}>;   // return shared_from_this();
};
```

- **명시적 수명주기**: 리소스를 쥐는 객체는 **생성 → 설정 → `start()` → 사용 → `stop()` → `reset()`** 순서로 다룹니다. 생성자에서 스레드를 띄우지 않고, 설정을 먼저 다 한 뒤 `start()`로 명시적으로 켭니다. 종료도 명시적으로 `stop()`(필요 시 `wait_stop()`) 후 `reset()`합니다. 소멸자는 안전망일 뿐, 정상 경로에서 종료 순서를 소멸자에 맡기지 않습니다.
- 소멸자에서는 보유 리소스를 정리(`stop()` → `clear()` → `reset()`)하고 디버그 로그를 남깁니다.
- 싱글턴은 **`Logger`만** 허용합니다. `private` 생성자 + `static handle()` + `static destroy()` + `std::unique_ptr` + `std::once_flag` 패턴을 따릅니다. 그 외 컴포넌트는 싱글턴을 만들지 않습니다.

---

## 11. 동시성 (Concurrency)

- 플래그/카운터는 `std::atomic<bool>`, `std::atomic<size_t>` 등을 사용합니다(`working_`, `pause_`, `thread_stop_`).
- 임계 구역은 `std::scoped_lock<std::mutex> lock(mutex_);`로 보호합니다. 잠금 범위는 블록 `{ }`으로 최소화합니다.
- 대기/통지는 `std::condition_variable`을 사용합니다.
- ThreadPool은 우선순위(`Top`/`High`/`Normal`/`Low`/`LongTerm`) 기반 Job 스케줄링을 따릅니다. Job 콜백도 §7의 반환 규약(`std::expected<void, std::string>`)을 동일하게 적용합니다(`return {};` 성공 / `return std::unexpected("...")` 실패).
- 시작/정지는 명시적 `start()` / `stop()`으로 제어합니다. **주의: 종료 API는 모듈마다 다릅니다.** ThreadPool은 **`wait_stop()`가 없고 `stop()` 자체가 블로킹**(워커 조인까지 대기, `stop(true)`는 대기 잡을 버리고 즉시 종료)입니다. 반면 Network/RabbitMQ/Kafka 등은 `wait_stop()`으로 대기 지점을 둡니다. 넘겨짚지 말고 모듈별 사용법(`module_usage_guide.md`)에서 종료 API를 확인합니다.

---

## 12. 문자열 포맷 / 로깅 (Formatting & Logging)

- 문자열 조립은 **`std::format`** 을 사용합니다(`+` 연결, `sprintf` 금지).
- 로그는 프로젝트 공용 전역 싱글턴을 통해 남깁니다: `Logger::handle().write(LogTypes::<레벨>, std::format(...));`. `Logger::handle()`은 **참조**를 반환하며, 설정을 먼저 한 뒤 `start("<파일베이스명>")`을 호출해야 실제로 기록됩니다.
- `LogTypes` 단계(`LogTypes.h`, `enum class : uint8_t`): `None`, `Exception`, `Error`, `Warning`, `Information`, `Debug`, `Sequence`, `Parameter`, `Packet`. 필터는 **`메시지레벨 <= 타겟모드`** — 값이 클수록 더 많이 출력됩니다. 철자 주의: `Warning`/`Information`(WARN/INFO 아님), 최고 심각도는 `Exception`.

| 레벨 | 용도 |
|------|------|
| `Exception` / `Error` | 복구 불가 오류, 예외 |
| `Warning` | 비치명적 이상(예: null 의존성으로 인한 조기 반환) |
| `Information` | 주요 상태 전이 |
| `Debug` | 진단용(예: 객체 소멸) |
| `Sequence` | 호출 흐름 추적 |
| `Parameter` | 설정값/입력 파라미터 기록 |
| `Packet` | 네트워크 패킷 수준 상세 |

---

## 13. 조건부 컴파일 (Conditional Compilation)

- 기능 토글은 CMake 옵션과 매크로로 제어합니다: `USE_*`(예: `USE_ENCRYPT_LIBS`/`USE_ENCRYPT_MODULE`) 및 `BUILD_*`(예: `BUILD_NETWORK_LIB`, `BUILD_REDIS_LIB`, `BUILD_RABBIT_MQ_LIB`, `BUILD_KAFKA_LIB`, `BUILD_SAMPLES`).
- 매크로에 따라 시그니처가 달라지면 `#ifdef`/`#else`/`#endif`로 **양쪽 선언을 모두** 제공합니다(예: 암호화 모듈 사용 시 생성자의 `USE_ENCRYPT_MODULE` 분기).
- 모듈은 `CMakeLists.txt`의 `option()`으로 on/off 가능하게 유지합니다.

---

## 14. 주석 (Comments)

- 주석은 **WHY가 자명하지 않을 때만** 작성합니다. 코드가 스스로 설명하는 내용을 중복 서술하지 않습니다.
- 미완성/보류 작업은 `// TODO:`로 표기하고 무엇이 남았는지 한 줄로 명시합니다.
- 주석 언어는 기존 파일의 관례를 따릅니다(라이브러리 계층은 영어를 기본으로, 프로젝트 문서·설명 주석은 프로젝트 관례).

---

## 15. 빌드 / 모듈 규약 (Build)

- 빌드 시스템은 **CMake + Ninja**, 패키지 관리는 **vcpkg**(`vcpkg.json` 매니페스트)입니다. 툴체인은 `~/vcpkg/scripts/buildsystems/vcpkg.cmake`.
- 새 의존성은 `vcpkg.json`에 추가하고, 새 모듈은 루트 `CMakeLists.txt`에 `option()` + `add_subdirectory()`로 등록합니다.
- 모듈 `CMakeLists.txt`는 **소스를 명시적으로 나열**합니다(`set(HEADER_FILES ...)` / `set(SOURCE_FILES ...)`; `file(GLOB ...)` 금지).
- `target_include_directories(<타겟> PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})`로 flat include(§8)를 성립시킵니다.
- **컴파일 가능한 변경은 반드시 `./build.sh`(또는 `cmake --build`)로 빌드가 에러 0으로 성공**해야 하며, 변경 모듈에 해당하는 `Samples/`를 실행해 검증합니다(`vcpkg.json` 변경 시에는 의존성 재설치 필요, 예: `FRESH_DEPS=1 ./build.sh`).

### 15.1 컴파일이 깨지는 3대 함정 (반드시 인지)

CppToolkit 기반 프로젝트에서 신규/AI 개발자가 거의 항상 처음 막히는 지점입니다. 상세와 매핑 표는 프로젝트별 `module_usage_guide.md`에 둡니다.

1. **include는 평평하다(flat).** 프로젝트 헤더는 파일명만(`#include "Logger.h"`), 경로 접두어(`"Utilities/Logger.h"`)는 금지. 각 모듈이 소스 디렉터리를 `PUBLIC`으로 공개하기 때문입니다.
2. **폴더명 ≠ 링크 타겟명 ≠ 네임스페이스.** `target_link_libraries(...)`에는 폴더명이 아니라 **CMake 타겟명**을 씁니다(대표 함정: `ThreadPool/` 폴더 → 타겟 `Thread` → 네임스페이스 `Thread`). 이미 존재하는 모듈을 소비할 때는 상위 타겟만 링크하면 하위 의존이 `PUBLIC`으로 전파됩니다.
3. **vcpkg 패키지명 ≠ `find_package` 이름 ≠ 링크 타겟.** 새 외부 의존을 붙일 때, `vcpkg.json`에 적는 패키지명으로 `find_package()`를 부르면 실패합니다(예: `librabbitmq` → `find_package(rabbitmq-c)` → `rabbitmq::rabbitmq`). 세 이름의 불일치가 신규 CMake 작성 시 가장 흔한 실패 원인입니다.

> 이 세 매핑 표(폴더↔타겟↔네임스페이스, vcpkg↔`find_package`↔링크 타겟, 모듈별 API 반환 타입)는 프로젝트마다 다르므로 각 프로젝트의 `module_usage_guide.md`(`{{PROJECT_PREFIX}}-USAGE-001`)에 두고, 본 문서는 그 존재와 원칙만 규정합니다. 새 모듈 추가 시 그 문서의 CMake 템플릿과 등록 절차를 그대로 따릅니다.

---

## 부록 A. 신규 클래스 골격 예시 (.h / .cpp 한 쌍)

프로젝트에 맞게 `{{NAMESPACE}}` / 클래스명(`Widget`)을 교체하여 사용합니다.

```cpp
// Widget.h
#pragma once

#include "Dependency.h"

#include <atomic>
#include <expected>
#include <memory>
#include <string>

namespace {{NAMESPACE}}
{
	class Widget : public std::enable_shared_from_this<Widget>
	{
	public:
		Widget(const std::string& name);
		virtual ~Widget(void);

		auto get_ptr(void) -> std::shared_ptr<Widget>;

		auto name(const std::string& value) -> void;   // setter
		auto name(void) const -> std::string;           // getter

		auto start(void) -> std::expected<void, std::string>;

	private:
		std::string name_;
		std::atomic_bool running_;
	};
} // namespace {{NAMESPACE}}
```

```cpp
// Widget.cpp
#include "Widget.h"

#include "Logger.h"

#include <format>

using namespace Utilities;

namespace {{NAMESPACE}}
{
	Widget::Widget(const std::string& name) : name_(name), running_(false) {}

	Widget::~Widget(void) { Logger::handle().write(LogTypes::Debug, std::format("destroyed {}", name_)); }

	auto Widget::get_ptr(void) -> std::shared_ptr<Widget> { return shared_from_this(); }

	auto Widget::name(const std::string& value) -> void { name_ = value; }
	auto Widget::name(void) const -> std::string { return name_; }

	auto Widget::start(void) -> std::expected<void, std::string>
	{
		if (running_.load())
		{
			return std::unexpected("already started");
		}
		running_.store(true);
		return {};
	}
} // namespace {{NAMESPACE}}
```

---

## 부록 B. 리뷰 체크리스트 (PR 전 확인)

커밋·PR 전에 아래 항목을 확인합니다.

- [ ] `./build.sh`(또는 `cmake --build`) 성공, 컴파일/링크 에러 0(`vcpkg.json` 변경 시 의존성 재설치).
- [ ] 변경 모듈의 `Samples/` 실행으로 동작 검증.
- [ ] `clang-format -i` 적용됨(GNU/Tab/170, Allman 중괄호).
- [ ] 명명 규칙 준수: 클래스/파일 PascalCase, 함수/변수 snake_case, 멤버 `_` 접미사.
- [ ] 모든 함수/메서드가 후행 반환 타입(`auto ... -> T`), 빈 매개변수는 `(void)`.
- [ ] getter/setter는 오버로드 패턴(`get_`/`set_` 접두사 없음), getter는 `const`.
- [ ] 오류 규약 준수: 성공/실패만은 `std::expected<void, std::string>`, 데이터+실패는 `tuple<optional<T>, optional<string>>`. 콜백은 `return {};` / `std::unexpected(...)`.
- [ ] `shared_ptr` 역참조 전 null 검사, 순환 참조는 `weak_ptr`로 차단.
- [ ] 비동기/네트워크 경로에 예외 없음(예외 가능 지점은 `try/catch`로 감싸 반환값 변환).
- [ ] 헤더 `#pragma once`, flat include(경로 접두어 없음), include 수동 정렬, 헤더에서 `using namespace` 없음.
- [ ] 리소스 객체는 설정 → `start` → 사용 → `stop` → `reset` 순서(모듈별 종료 API 확인: ThreadPool은 `wait_stop` 없이 `stop()` 블로킹).
- [ ] 로깅은 `Logger` 싱글턴 + `std::format`, 적절한 `LogTypes` 레벨 사용.
- [ ] CMake: 소스 명시 나열(GLOB 금지), 링크는 타겟명(§15.1), 새 외부 의존은 `find_package`/링크 타겟 매핑 일치.
- [ ] 관련 ISO 문서(요구사항·설계·추적성 등) 갱신 필요 시 함께 갱신.
