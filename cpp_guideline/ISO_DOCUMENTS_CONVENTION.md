# ISO 문서 규약 (C/C++ REST/서버 프로젝트)

| 항목 | 내용 |
|------|------|
| 문서 ID | `{{PROJECT_PREFIX}}-DOCS-001` |
| 대상 | CppToolkit(C++23) 기반 REST/서버 프로젝트의 산출물 문서(SRS·SAD·STP/STR·RTM) |
| 언어 표준 | C++23 (`CMAKE_CXX_STANDARD 23`) + (선택) Go 1.23+ (RestAPI) |
| 포맷터 | `.clang-format` (GNU 기반, Tab 들여쓰기, 170 컬럼) · Go는 `gofmt` |
| 상태 | 활성 (Active) |

> 본 문서는 C/C++ 서버 프로젝트의 ISO 산출물 문서를 **어떤 표준·ID 체계·목차·추적성 형식으로 작성·유지하는가**를 규정하는 템플릿입니다. 실제 적용 사례(RTMC: RestAPI(Go) + MainServer/Consumer/UserClient(C++23) + CppToolkit 서브모듈)에서 검증된 SRS/SAD/STP/RTM 문서 세트를 일반화한 것이며, 상상해서 만든 형식이 아닙니다. `{{PLACEHOLDER}}`는 프로젝트 값으로 치환하고, 규약 자체(표준 매핑·ID 체계·목차·추적성 형식)는 그대로 사용합니다.

---

## 0. 이 템플릿 사용법 (온보딩)

1. `cpp_guideline/`의 `CODING_CONVENTION.md`(스타일·관용구)와 본 `ISO_DOCUMENTS_CONVENTION.md`(문서 규약)를 실제 프로젝트의 `docs/` 폴더로 복사합니다.
2. `{{PLACEHOLDER}}`를 프로젝트 값으로 채웁니다: `{{PROJECT_NAME}}`(예: RealTimeMessageChat), `{{PROJECT_PREFIX}}`(예: RTMC), `{{NAMESPACE}}`(예: `Network`, `Thread`), `{{MODULE_NAME}}`(예: MainServer), 기능 영역 코드 `{{AREA}}`(예: MSG, SRV, CON) 등.
3. 코드 규약(`CODING_CONVENTION.md`)과 본 문서는 짝을 이룹니다. 전자는 *어떻게 코드를 쓰는가*, 본 문서는 *산출물을 어떤 ISO 문서로 기술·추적하는가*를 규정합니다.
4. 전역 지침(`global_guideline/CLAUDE.md`)에 종속됩니다. 충돌 시 전역 지침이 우선합니다.

> **전역 지침 정합**: 한국어·존댓말 작성, 불필요한 서론/요약 생략, 임의 커밋/PR 금지(사용자가 직접 수행 — 본 문서는 커밋/주석의 *형식*만 규정), 모든 분석은 ISO 기반 문서로 작성하여 적합성·추적성 확보, 문서는 프로젝트 `docs/`에 저장, 추적성 우선순위는 0티어, 컴파일 가능한 코드는 반드시 빌드 통과(에러 0).

---

## 1. 목적과 적용 표준

- **목적**: 요구사항(SRS)부터 아키텍처(SAD)·테스트(STP/STR)·소스코드까지의 **양방향 추적성(traceability)** 을 확보하고, 문서와 실제 C++/Go 코드의 정합성을 지속 검증하는 것입니다. 추적성은 최우선(0티어) 관심사입니다.
- 적용 표준(각 표준이 규정하는 바):

| 표준 | 규정 대상 | 본 규약에서의 근거 문서 |
|------|-----------|------------------------|
| **ISO/IEC/IEEE 29148:2018** | 요구공학(requirements engineering): 요구사항 작성·품질 기준·SRS 구조·요구사항 속성(ID·우선순위·검증방법)·추적성 | **SRS**, **RTM** |
| **ISO/IEC/IEEE 42010:2022** | 아키텍처 기술(architecture description): 이해관계자(stakeholder)·관심사(concern)·관점(viewpoint)·뷰(view) 구조 | **SAD** |
| **ISO/IEC/IEEE 29119-3:2021** | 소프트웨어 테스트 문서: 테스트 계획서·테스트 케이스 명세·테스트 결과서 구조 | **STP/STR** |
| **ISO/IEC 25010** | SQuaRE 제품 품질 모델: 8개 품질 특성 정의 | **SRS §4(NFR)**, 품질 속성 매핑(§12) |
| **ISO/IEC/IEEE 12207** | 소프트웨어 생명주기 프로세스: 요구·설계·구현·검증·유지보수 프로세스 프레임(맥락) | 전체 골격(맥락 수준) |
| **ISO/IEC 27001** | 정보보안 경영시스템(ISMS): 보안 통제 항목의 맥락 — 인증·암호화·SQLi 방지 요구사항 근거 | SRS §3.10(Auth&Security), NFR-SEC(맥락) |
| **ISO 9001** | 품질경영시스템(QMS): 문서 관리·리뷰·변경 관리 등 프로세스 품질(맥락) | 문서 이력·상태 관리(맥락) |

---

## 2. 문서 세트 개요

C++ 서버 프로젝트의 필수 산출물은 **SRS / SAD / STP(STR 포함) / RTM** 4종입니다. 각 문서는 하나의 파일에 대응하며 상호 참조로 연결됩니다.

| 문서 | 문서 ID 패턴 | 기준 표준 | 목적 | 파일 위치 |
|------|-------------|-----------|------|-----------|
| SRS (Software Requirements Specification) | `{{PROJECT_PREFIX}}-SRS-001` | ISO/IEC/IEEE 29148:2018 | 기능/비기능 요구사항·제약·외부 인터페이스 정의. 요구사항 ID의 단일 출처 | `docs/SRS.md` |
| SAD (Software Architecture Description) | `{{PROJECT_PREFIX}}-SAD-001` | ISO/IEC/IEEE 42010:2022 | 컴포넌트·프로세스·데이터·배포 뷰로 아키텍처 기술, 요구사항→설계 추적 | `docs/SAD.md` |
| STP/STR (Software Test Plan & Report) | `{{PROJECT_PREFIX}}-STP-001` | ISO/IEC/IEEE 29119-3:2021 | 테스트 전략·스위트·케이스·결과·커버리지 정의, 설계→테스트 추적 | `docs/STP.md` |
| RTM (Requirements Traceability Matrix) | `{{PROJECT_PREFIX}}-RTM-001` | ISO/IEC/IEEE 29148:2018 (Traceability) | SRS→SAD→STP 및 소스코드 간 양방향 추적, 정합성 분석 | `docs/RTM.md` |

- STP와 STR(결과서)는 하나의 문서로 통합 운영합니다. §8과 같이 "테스트 계획(2~3장) + 테스트 결과(4장)"를 한 파일에 둡니다(RTMC의 `STP.md`가 계획과 결과를 함께 담는 형식).
- 문서 ID는 프로젝트가 커져 문서를 분할해도 `-NNN` 일련번호로 확장합니다(예: SRS를 도메인별로 나눌 때 `{{PROJECT_PREFIX}}-SRS-002`).

---

## 3. docs/ 폴더 구조

- 모든 ISO 문서는 프로젝트 루트의 `docs/` 하위에 저장합니다. 전역 지침에 따라 `docs/`는 `.gitignore`에 추가하여 커밋되지 않도록 관리합니다.
- **기본 트리(소규모/단일 저장소 — RTMC와 동일)**: 4개 문서를 `docs/` 바로 아래 평평하게 둡니다.

```text
docs/
├── SRS.md          # {{PROJECT_PREFIX}}-SRS-001  요구사항 명세 (29148)
├── SAD.md          # {{PROJECT_PREFIX}}-SAD-001  아키텍처 기술 (42010)
├── STP.md          # {{PROJECT_PREFIX}}-STP-001  테스트 계획 & 결과 (29119-3)
├── RTM.md          # {{PROJECT_PREFIX}}-RTM-001  추적성 매트릭스 (29148)
├── CODING_CONVENTION.md         # 코드 스타일·관용구 규약 (짝 문서)
└── ISO_DOCUMENTS_CONVENTION.md  # 본 문서
```

- **확장 트리(대규모 — 산출물이 늘어날 때 옵션)**: 문서 종류별 하위 폴더로 분리합니다.

```text
docs/
├── requirements/   # SRS.md (도메인별 분할 시 SRS_<area>.md)
├── architecture/   # SAD.md, adr/ADR-NNN.md (아키텍처 결정 기록)
├── test/           # STP.md, STR.md(결과서 분리 시), test_cases/
└── traceability/   # RTM.md, 정합성 분석 리포트
```

- CppToolkit 소비 프로젝트에서 프레임워크 코드의 검증은 `docs/sample_test_cases.md`(샘플 기반 확인 포인트)와 STP를 함께 사용합니다.
- 어느 트리를 쓰든 문서 간 상호참조 링크는 상대경로로 유지합니다(예: SRS에서 `[SAD.md](SAD.md)`).

---

## 4. 문서 공통 형식

모든 문서는 **최상단 메타데이터 표** + **최하단 변경 이력 표**를 반드시 포함합니다(RTMC 4개 문서가 공통으로 따르는 형식).

### 4.1 최상단 메타데이터 표

문서 제목(H1) 바로 아래에 둡니다.

```markdown
# Software Requirements Specification (SRS)

**{{PROJECT_NAME}} - {{DESCRIPTION}}**

| 항목 | 내용 |
|------|------|
| 문서 ID | {{PROJECT_PREFIX}}-SRS-001 |
| 버전 | 1.0.0 |
| 기준 표준 | ISO/IEC/IEEE 29148:2018 |
| 작성일 | {{YYYY-MM-DD}} |
| 최종 수정일 | {{YYYY-MM-DD}} |
| 상태 | Draft |
```

- 메타 항목: **문서 ID / 버전 / 기준 표준 / 작성일 / 최종 수정일 / 상태**.
- 상태 값: `Draft` → `Active` → (`Deprecated`). 리뷰 단계를 별도로 관리하는 조직은 `Draft` → `Review` → `Active`를 사용합니다.

### 4.2 최하단 변경 이력 표

문서 마지막 절로 둡니다.

```markdown
## 변경 이력

| 버전 | 날짜 | 변경 내용 | 작성자 |
|------|------|-----------|--------|
| 1.0.0 | {{YYYY-MM-DD}} | 초기 작성 | {{AUTHOR}} |
| 1.0.1 | {{YYYY-MM-DD}} | RTM 커버리지 수치 STP와 교차검증, LIM↔CON 교차참조 추가 | {{AUTHOR}} |
```

### 4.3 버전 규칙 (x.y.z)

- `x`(major): 하위호환이 깨지는 구조 변경(예: 요구사항 대량 재편, 컴포넌트 뷰 재설계).
- `y`(minor): 요구사항·컴포넌트·테스트의 추가/보완.
- `z`(patch): 오탈자·서식·수치 보정(예: RTMC STP `1.0.0→1.0.1`의 커버리지 수치 보정).
- SRS의 요구사항이 바뀌면 관련 SAD/STP/RTM도 함께 갱신하고, 각 문서의 버전·최종 수정일·변경 이력을 동시에 올립니다(§13).

---

## 5. 식별자(ID) 체계 [핵심]

추적성의 근간은 일관된 ID입니다. 아래 체계를 전 문서에서 동일하게 사용합니다.

| 대상 | ID 패턴 | 예시 | 정의 위치 |
|------|---------|------|-----------|
| 문서 | `{{PROJECT_PREFIX}}-{SRS\|SAD\|STP\|RTM}-NNN` | `RTMC-SRS-001` | 각 문서 메타 표 |
| 기능 요구사항 | `FR-{AREA}-NN` | `FR-MSG-01`, `FR-SRV-08` | SRS §3 |
| 비기능 요구사항 | `NFR-{ATTR}-NN` | `NFR-PRF-01`, `NFR-SEC-05` | SRS §4 |
| 컴포넌트 | `CMP-{AREA}[-{ROLE}]` | `CMP-API`, `CMP-API-HDL`, `CMP-SRV` | SAD §3 |
| 아키텍처 결정 | `ADR-NN` | `ADR-01` | SAD §2.2 |
| 제약사항 | `CON-NN` | `CON-04` (수평 확장 미지원) | SRS §2.5 |
| 가정/의존성 | `ASM-NN` | `ASM-02` (vcpkg 설치 가정) | SRS §2.6 |
| 아키텍처 제한 | `LIM-NN` | `LIM-01` (단일 서버 인스턴스) | SAD §9 / RTM §4.4 |
| 테스트 스위트 | `TS-NN` | `TS-04` (로컬 스모크) | STP §3 |
| 테스트 케이스 | `TC-{SUITE}-NN` | `TC-SVC-01`, `TC-REPO-03` | STP §3 |
| 통합 테스트 | `IT-NN` | `IT-07` (RabbitMQ publish) | STP §3.5 |

- **기능 영역 코드 `{AREA}`** 는 프로젝트에 맞게 정합니다. RTMC 예시(교체 대상): `MSG`(메시지), `USR`(사용자), `SYS`(시스템), `CON`(Consumer), `SRV`(TCP 서버), `CLT`(클라이언트), `CMN`(공통 모듈), `DB`(데이터베이스), `CFG`(설정), `AUTH`(인증/보안), `DEP`(배포), `MDW`(미들웨어).
- **비기능 속성 코드 `{ATTR}`** 는 25010 기반 고정 집합을 권장합니다: `PRF`(성능/Performance), `REL`(신뢰성/Reliability), `PRT`(이식성/Portability), `MNT`(유지보수성/Maintainability), `SEC`(보안/Security).
- **컴포넌트 역할 코드 `{ROLE}`** 예시: `HDL`(handler), `SVC`(service), `REPO`(repository), `MDW`(middleware), `CFG`(config), `INFRA`(infra client). C++ 툴킷 컴포넌트는 `CMP-TK-{MODULE}` 형태(예: `CMP-TK-NET`, `CMP-TK-THREAD`, `CMP-TK-REDIS`)로 서브모듈을 구분합니다.
- **상태 용어 통일(0티어 원칙)**: 요구사항의 구현/검증 상태는 전 문서에서 **`구현/검증`**(구현되고 테스트로 검증됨), **`구현/미검증`**(구현됐으나 자동 테스트 부재), (필요 시 `미구현`)으로 통일합니다. 상태 용어 불일치는 RTM 정합성 체크(§10)의 실패 항목입니다. 테스트 결과 표기는 `PASS`/`FAIL`/`SKIP`를 사용합니다.

---

## 6. SRS 최소 목차 (ISO/IEC/IEEE 29148)

`docs/SRS.md` — 요구사항의 단일 출처. 아래 목차를 최소로 포함합니다(RTMC SRS 구조 기준).

1. **개요 (Introduction)**
   1.1 목적 — SRS가 SAD/STP/RTM의 기반임을 명시
   1.2 범위 — 시스템이 무엇을 하는가(예: REST → MQ → Consumer → Redis/DB → TCP 브로드캐스트)
   1.3 정의 및 약어 — 표(약어 | 설명): RTMC, TCP, REST, MQ, JWT, AMQP, SRS, SAD, STP, RTM 등
   1.4 참조 문서 — 표(문서 ID | 제목 | 위치): SAD/STP/RTM 링크
2. **전체 시스템 설명 (Overall Description)**
   2.1 제품 관점 — 아키텍처 개관(메시지 흐름 다이어그램)
   2.2 제품 기능 요약 — 개조식 기능 목록
   2.3 사용자 분류 — 표(사용자 | 설명): API Client, TCP Client, Operator, Developer
   2.4 운영 환경 — OS/컴파일러/빌드/패키지/컨테이너/인프라 (예: Linux/macOS, GCC 13+/Clang 16+, C++23, CMake 3.18+, vcpkg, Docker, RabbitMQ/Redis/PostgreSQL)
   2.5 제약사항 — 표(`CON-NN` | 제약사항)
   2.6 가정 및 의존성 — 표(`ASM-NN` | 가정/의존성)
3. **기능 요구사항 (Functional Requirements)** — 기능 영역별로 절을 나누고, 각 절에 표(`FR-{AREA}-NN` | 요구사항 | 우선순위 | 상태). 우선순위는 `필수`/`선택`.
4. **비기능 요구사항 (Non-Functional Requirements)** — 25010 특성별 절: 4.1 성능(`NFR-PRF`), 4.2 신뢰성(`NFR-REL`), 4.3 이식성(`NFR-PRT`), 4.4 유지보수성(`NFR-MNT`), 4.5 보안(`NFR-SEC`). 각 표에 측정 가능한 목표를 둡니다(예: "REST API 응답 시간 < 100ms").
5. **외부 인터페이스 요구사항 (External Interface Requirements)** — 5.1 REST API(요청/응답 JSON 예시), 5.2 TCP 프로토콜(JSON 메시지 형식), 5.3 MQ(큐명·content-type·delivery mode), 5.4 설정 파일(컴포넌트별 JSON 경로).
6. **요구사항 상태 요약** — 6.1 FR 소계 표(카테고리 | 전체 | 구현됨 | 부분 구현 | 미구현), 6.2 NFR 소계 표, 6.3 전체 요약 표(구분 | 전체 | 구현됨 | … | 구현율).
7. **변경 이력** — §4.2 형식.

- 각 요구사항은 **검증 가능**해야 합니다(모호어 금지: "빠르게"(X) → "p95 < 100ms"(O)).
- 요구사항 번호는 삭제·재사용하지 않고 결번으로 남깁니다(추적 안정성).

---

## 7. SAD 최소 목차 (ISO/IEC/IEEE 42010)

`docs/SAD.md` — 42010의 **이해관계자(stakeholder)·관심사(concern)·관점(viewpoint)·뷰(view)** 개념에 따라 아키텍처를 기술합니다. 각 뷰는 하나의 관점에 대응하고, 관점은 이해관계자의 관심사를 다룹니다(예: 운영자→가용성/배포 뷰, 개발자→유지보수성/컴포넌트 뷰, 보안담당→보안/데이터 뷰).

1. **개요 (Introduction)** — 1.1 목적(SRS 요구사항이 어떻게 구현되는지 추적 가능하게 함), 1.2 범위, 1.3 참조 문서(SRS/STP/RTM 관계).
2. **아키텍처 개관 (Architecture Overview)** — 2.1 시스템 컨텍스트(ASCII 다이어그램), 2.2 아키텍처 결정 사항(ADR Summary) 표(`ADR-NN` | 결정 | 근거 | 관련 요구사항).
3. **컴포넌트 뷰 (Component View)** — 3.1 컴포넌트 구조(디렉터리 트리에 `CMP-*` 주석), 3.2 컴포넌트 설명 표(`CMP-*` | 컴포넌트 | 책임 | 관련 요구사항), 3.3 컴포넌트 의존 관계(의존 그래프).
4. **프로세스 뷰 (Process View)** — 4.1 프로세스 모델(프로세스 | 바이너리 | 포트 | 역할), 4.2 스레드 모델(컴포넌트별 스레드 | 소유자 | 역할 | 관련 요구사항 — ThreadPool 우선순위 워커 포함), 4.3 스레드 동기화(mutex/atomic/condition_variable), 4.4 메시지 처리 흐름(End-to-End 다이어그램).
5. **데이터 뷰 (Data View)** — 5.1 데이터/메시지 수명주기(상태 전이 다이어그램), 5.2 스키마(테이블별 컬럼 표), 5.3 인덱스 전략(인덱스 | 대상 컬럼 | 목적).
6. **배포 뷰 (Deployment View)** — 6.1 Docker Compose 서비스(포트/health/의존), 6.2 Dockerfile 멀티스테이지(스테이지 | 베이스 | 용도), 6.3 로컬 빌드 출력(`build/out/`, `build/lib/`).
7. **빌드 구조 (Build Structure)** — 7.1 라이브러리 의존 그래프(실행 파일 → static lib → 외부 라이브러리), 7.2 외부 의존성 표(라이브러리 | 용도 | 컴포넌트). C++는 CppToolkit 규약대로 폴더명/링크 타겟명/네임스페이스 불일치(예: `ThreadPool/`→타겟 `Thread`)를 명시.
8. **설계-요구사항 매핑** — 표(컴포넌트 | 역할 | 구현하는 요구사항). RTM 전방 추적과 일치해야 함.
9. **알려진 아키텍처 제한사항** — 표(`LIM-NN` | 제한사항 | 영향 | 관련 SRS 제약(`CON-NN`) | 상태). LIM↔CON 교차참조 필수.
10. **변경 이력** — §4.2 형식.

- 42010 준수 핵심: 각 뷰가 어떤 이해관계자의 어떤 관심사를 다루는지 명시하고, ADR로 결정 근거를 남깁니다.

---

## 8. STP/STR 최소 목차 (ISO/IEC/IEEE 29119-3)

`docs/STP.md` — 테스트 계획(Test Plan)과 결과(Test Report)를 한 문서에 통합합니다.

1. **개요 (Introduction)** — 1.1 목적(SRS 요구사항·SAD 설계 검증), 1.2 범위(단위/스모크/통합/CI), 1.3 참조 문서.
2. **테스트 전략 (Test Strategy)** — 2.1 테스트 수준 표(수준 | 도구 | 대상 | 환경 | 자동화), 2.2 테스트 환경, 2.3 테스트 실행 명령(§8.1), 2.4 테스트 실행 기준(상황 | 필수 테스트 | 명령).
3. **테스트 스위트 (Test Suites)** — 스위트별 절(`TS-NN`). 각 스위트에 표(`TC-{SUITE}-NN` 또는 `IT-NN` | 테스트 케이스 | 검증 대상 요구사항(`FR-*`/`NFR-*`) | 결과(`PASS`/`FAIL`/`SKIP`)).
4. **테스트 결과 요약 (Test Results Summary)** — 4.1 전체 결과(스위트 수/케이스 수/PASS/FAIL/SKIP/통과율/마지막 실행일), 4.2 스위트별 결과, 4.3 요구사항 커버리지(카테고리 | 요구사항 수 | 유형별 커버 | 총 커버 | 커버율) + 커버리지 상세 근거(RTM 교차검증), 4.4 미커버 요구사항(요구사항 | 사유).
5. **테스트 절차 (Test Procedures)** — 5.1 새 기능 추가 시(요구사항 ID 부여→TC 추가→RTM 매핑→테스트 구현→실행→STP 갱신), 5.2 버그 수정 시(재현 테스트→수정→회귀→갱신), 5.3 결과 갱신 규칙, 5.4 검증 기준(Definition of Done: 빌드 성공, 테스트 통과, 회귀 없음, 문서 갱신).
6. **알려진 테스트 한계** — 표(한계 | 설명 | 대응 방안). 예: C++ 컴포넌트 gtest 부재.
7. **변경 이력** — §4.2 형식.

### 8.1 테스트 실행 명령 예시

**C++ (Google Test + CTest):**

```bash
# 빌드 (CppToolkit 기반: vcpkg 툴체인)
./build.sh
cmake --build build --config Release --parallel

# 단위 테스트 실행
cd build && ctest --output-on-failure

# 특정 테스트만
ctest -R {{TEST_NAME}} --output-on-failure

# 바이너리 산출 확인 (스모크)
ls build/out/{{MODULE_NAME}}
```

**Go (RestAPI가 있는 경우):**

```bash
cd RestAPI
go test ./... -race -cover        # 레이스 검출 + 커버리지
go test -v ./internal/service/... # 특정 패키지만
make check                        # fmt + vet + lint + test (전체 검증)
```

- CppToolkit 소비 프로젝트에서 프레임워크 코드 변경 시, 정식 gtest가 없으면 `docs/sample_test_cases.md`의 샘플 확인 포인트를 테스트 케이스로 사용하고 STP 스위트에 반영합니다.

---

## 9. RTM(추적성) [핵심]

`docs/RTM.md` — 추적성은 0티어 관심사입니다. **SRS(요구사항) → SAD(설계/컴포넌트) → STP(테스트 케이스) → 소스 파일** 을 **양방향**으로 연결합니다.

- 목적: 모든 요구사항이 설계에 반영되고 테스트로 검증됨을 보장하며, 역으로 모든 테스트/컴포넌트가 유효한 요구사항에서 비롯됨을 확인(고아 테스트·근거 없는 컴포넌트 방지)합니다.

```text
SRS (요구사항)  ──→  SAD (설계/컴포넌트)  ──→  STP (테스트 케이스)
     ↑                    ↑                       ↑
     └────────────────────┴───────────────────────┘
                     RTM (본 문서)
```

### 9.1 (a) 전방 추적 매트릭스 (Forward Traceability)

요구사항 ID → 설계 컴포넌트 → 테스트 케이스 → 소스 파일 → 상태. 기능 영역별로 구분자 행을 두고, NFR도 동일 표에 포함합니다.

| 요구사항 ID | 요구사항 설명 | 설계 컴포넌트 | 테스트 케이스 | 소스 파일 | 상태 |
|-------------|-------------|--------------|--------------|-----------|------|
| FR-MSG-01 | REST API 메시지 RabbitMQ 발행 | CMP-API-HDL, CMP-API-INFRA | IT-07, IT-10 | `message_handler.go`, `rabbitmq.go` | 구현/검증 |
| FR-SRV-01 | Boost.Asio TCP 서버 | CMP-SRV, CMP-TK-NET | TC-LOCAL-05, IT-04 | `MainServer.cpp`, `NetworkServer.cpp` | 구현/검증 |
| FR-SRV-08 | 3단계 우선순위 ThreadPool | CMP-SRV, CMP-TK-THREAD | - | `MainServer.cpp`, `ThreadPool.cpp` | 구현/미검증 |
| NFR-SEC-05 | SQL injection 방지 (parameterized query) | CMP-API-REPO, CMP-TK-DB | TC-REPO-* | `user_repository.go`, `PostgresDB.cpp` | 구현/검증 |

### 9.2 (b) 역방향 추적 (Backward Traceability)

- **테스트 → 요구사항**: 표(테스트 스위트 | 테스트 수 | 환경 | 커버하는 요구사항). 각 스위트가 어떤 `FR-*`/`NFR-*`를 커버하는지 나열하여 고아 테스트가 없음을 확인.

| 테스트 스위트 | 테스트 수 | 환경 | 커버하는 요구사항 |
|--------------|----------|------|------------------|
| User Service Tests | 9 | 로컬/CI | FR-USR-01~07 |
| Docker Integration | 12 | Docker | FR-MSG-01~03, FR-SYS-01, FR-CON-01/03, FR-SRV-01/06, FR-CLT-01, FR-DEP-01~05, NFR-REL-01/04 |

- **컴포넌트 → 요구사항**: 표(컴포넌트 | 역할(SAD 참조) | 총 요구사항 수 | 테스트 커버 | 미검증 | 근거). 한 요구사항이 여러 컴포넌트에 매핑될 수 있으므로 컴포넌트별 합계는 SRS 총 수보다 클 수 있음을 명시.

| 컴포넌트 | 역할 (SAD 참조) | 총 요구사항 수 | 테스트 커버 | 미검증 | 근거 |
|----------|----------------|--------------|-----------|--------|------|
| CMP-SRV | TCP 서버 | 8 | 3 | 5 | FR-SRV-01~08(8) |
| CMP-COMMON | 공통 모듈 | 7 | 0 | 7 | FR-CMN-01~07(7) |

- 요구사항 하나가 여러 소스/테스트에 대응하면 필요 시 행을 분할해 정규화합니다(추적 누락 방지).

---

## 10. 정합성 분석

RTM의 한 절(RTMC 기준 §4)로 두고, 문서 간 정합성을 체크리스트로 검증합니다.

### 10.1 정합성 체크 결과

| # | 검증 항목 | 결과 | 비고 |
|---|----------|------|------|
| 1 | 모든 SRS FR이 RTM에 존재하는가? | PASS/FAIL | FR N개 모두 존재 |
| 2 | 모든 SRS NFR이 RTM에 존재하는가? | PASS/FAIL | NFR M개 모두 존재 |
| 3 | SRS ↔ RTM 요구사항 수 일치? | PASS/FAIL | FR N + NFR M = 합계 동일 |
| 4 | 모든 SRS 요구사항이 SAD 컴포넌트에 매핑되는가? | PASS/FAIL | 전체 매핑됨 |
| 5 | 모든 SAD 컴포넌트가 SRS 요구사항에 근거하는가? | PASS/FAIL | 근거 없는 컴포넌트 없음 |
| 6 | SAD 컴포넌트 목록 ↔ SAD 설계-요구사항 매핑 일치? | PASS/FAIL | 컴포넌트 수 일치 |
| 7 | 모든 "구현/검증" 요구사항에 테스트가 있는가? | PASS/FAIL | 미검증 항목 별도 집계(10.2) |
| 8 | 모든 테스트가 요구사항에 매핑되는가? (고아 테스트 없음) | PASS/FAIL | STP 전 TC/IT 매핑됨 |
| 9 | STP ↔ RTM 테스트 ID 일치? | PASS/FAIL | TC/IT ID 동일 |
| 10 | STP 커버리지 수치 ↔ RTM 커버리지 수치 일치? | PASS/FAIL | STP §4.3 ↔ RTM §4.2 동일 |
| 11 | SAD 컴포넌트 ↔ 소스 파일 일치? | PASS/FAIL | 모든 컴포넌트에 대응 소스 존재 |
| 12 | SRS/RTM/STP 상태 용어 통일? | PASS/FAIL | "구현/검증", "구현/미검증" 통일 |
| 13 | SAD LIM ↔ SRS CON 교차참조? | PASS/FAIL | LIM-NN↔CON-NN 매핑 |

### 10.2 커버리지 갭 분석

```text
기능 요구사항 (FR):
  전체:       N개
  테스트 커버: X개 (X/N %)   ← STP 4.3과 일치해야 함
  미검증:     N-X개

비기능 요구사항 (NFR):
  전체:       M개
  테스트 커버: Y개 (Y/M %)

전체 (FR + NFR):
  전체:       N+M개
  테스트 커버: X+Y개
  미검증:     (N+M)-(X+Y)개
```

### 10.3 개선 우선순위 (P0~P3)

미검증 요구사항을 위험도 순으로 분류해 검증 로드맵을 제시합니다(아래 수치·행은 RTMC 사례를 일반화·요약한 예시로, 프로젝트별 미검증 집계에 맞게 교체. RTMC 원본은 RTM §4.3처럼 P1을 "메시지 핸들러"/"미들웨어·보안", P2를 "DB 스키마"/"시스템 엔드포인트"로 세분화합니다).

| 우선순위 | 영역 | 미검증 FR | 미검증 NFR | 합계 | 대응 방안 |
|----------|------|----------|----------|------|----------|
| P0 | C++ 컴포넌트 단위 테스트 | 22 | 3 | 25 | gtest 스위트 추가 |
| P1 | 핸들러/미들웨어 테스트 | 13 | 4 | 17 | httptest 기반 테스트 |
| P2 | DB 스키마/엔드포인트 테스트 | 8 | 2 | 10 | 뷰/함수/인덱스 검증 |
| P3 | 성능/스트레스 테스트 | 0 | 1 | 1 | 동시 접속 벤치마크 |

- P0: 핵심 경로의 검증 공백(회귀 위험 최상). P1: 보안·인증 등 중요 경로. P2: 부가 기능. P3: 개선성 항목.

---

## 11. 코드-문서 연계

목적: 문서 ID를 코드·테스트·커밋에 심어 역방향 추적이 성립하도록 합니다. (임의 커밋 금지 원칙 준수 — 커밋은 사용자가 직접 수행하며, 여기서는 **형식만** 규정합니다.)

- **C++ 주석**(요구사항 참조가 비자명할 때만 — 전역 지침 "주석은 WHY가 비자명할 때만" 준수):

```cpp
// DBWorker::working — 소비된 메시지를 PostgreSQL에 비동기 저장한다.
// Ref: FR-CON-04, FR-CON-05 (암호화 활성 시 AES-256-CBC는 FR-CON-06)
auto DBWorker::working(void) -> std::expected<void, std::string>
{
    // ...
}
```

- **테스트명에 요구사항 의도**를 드러냅니다:

```cpp
// gtest: MainServerConsumer가 FR-CON-02(JSON 검증)를 만족하는지 검증
TEST(MainServerConsumerTest, RejectsMalformedJson_FR_CON_02) { /* ... */ }
```

```go
// TestUserService_Create verifies FR-USR-01.
func TestUserService_Create(t *testing.T) { /* ... */ }
```

- **커밋 메시지 형식**(형식만 규정):

```text
<type>(<scope>): <subject>

<body>

Refs: FR-MSG-01, NFR-SEC-05
```

  - `type`: `feat` / `fix` / `docs` / `refactor` / `test` / `chore`.
  - `Refs:` 트레일러에 관련 요구사항 ID를 나열하여 커밋↔요구사항 추적을 확보합니다.

- **SAD 컴포넌트 ↔ 소스 파일 매핑 유지**: RTM 전방 추적의 "소스 파일" 열과 SAD §3.1 컴포넌트 트리 주석(`# CMP-*`)을 항상 일치시킵니다. 파일 이동/리네임 시 두 문서를 함께 갱신합니다.

---

## 12. 품질 속성 매핑 (ISO/IEC 25010)

8개 품질 특성을 서버/REST 관점 지표로 구체화합니다. 각 항목은 `NFR-{ATTR}-NN`으로 요구사항화하여 SRS §4·RTM에서 추적합니다(아래 값은 RTMC NFR 사례를 일반화한 것으로, 프로젝트 목표에 맞게 교체).

| 특성(25010) | 서버/REST 관점 구체화 | 측정 지표 예 | 대응 NFR |
|-------------|----------------------|-------------|----------|
| 기능 적합성(Functional Suitability) | 명세된 엔드포인트/프로토콜이 계약대로 동작 | 계약 테스트 통과율 100% | (필요 시 NFR-FNC-*) |
| 성능 효율성(Performance Efficiency) | 응답 지연·처리량·폴링 지연 | REST API 응답 p95 < 100ms, RabbitMQ 비동기 처리량 확보 | NFR-PRF-01~04 |
| 호환성(Compatibility) | 인프라 상호운용(RabbitMQ/Redis/PostgreSQL), API 버저닝 | AMQP/Redis 프로토콜 호환, 스키마 breaking change 0 | NFR-PRT-* |
| 사용성(Usability) | API 문서·오류 메시지 명확성 | Swagger 최신화율 100%, 표준 error body 준수 | NFR-MNT-05 |
| 신뢰성(Reliability) | 메시지 손실 방지·장애 복구·데이터 무결성 | RabbitMQ durable queue(손실 0), Graceful shutdown, Redis+DB 이중 저장 | NFR-REL-01~04 |
| 보안(Security) | 인증·인가·전송/저장 보안·주입 방지 | JWT 인증 커버율 100%, AES-256-CBC 암호화, SQLi 방지(parameterized query), 키 로테이션 | NFR-SEC-01~05 |
| 유지보수성(Maintainability) | 모듈성·계층화·테스트성·정적분석 | Handler→Service→Repository 계층화, 콜백 맵 확장, 커버리지 목표, `clang-format`/`go vet` 경고 0 | NFR-MNT-01~04 |
| 이식성(Portability) | 환경 독립성·컨테이너 배포 | Linux(Docker/CI)/macOS(개발) 지원, Docker 무수정 배포, GitHub Actions CI/CD | NFR-PRT-01~03 |

- 측정 방법 예: 성능은 부하테스트로 p95 계측, 커버리지는 `ctest`/`go test -coverprofile`, 정적분석은 `clang-tidy`/`go vet`, 보안은 의존성 취약점 스캔.

---

## 13. 문서 갱신 규칙

전역 지침("feature, bug fix 등 수정 시 ISO 문서 갱신이 필요한 경우 진행")을 반영합니다.

### 13.1 변경 유형별 갱신 대상

| 변경 유형 | 갱신 대상 문서 |
|-----------|---------------|
| 신규 기능(feature) | SRS(요구사항 `FR-*`/`NFR-*` 추가), SAD(컴포넌트·뷰·ADR), STP(TC/IT 추가), RTM(추적 매핑·상태) |
| 버그 수정(bugfix) | STP(재현 테스트 추가·결과 갱신), RTM(테스트 매핑·상태), (요구사항 오류였다면) SRS |
| 리팩터링(refactor) | SAD(컴포넌트 구조/의존 갱신), RTM(소스 파일 열 갱신) |
| 성능/보안 개선 | SRS §4(NFR 목표), SAD(관련 뷰), RTM |
| 아키텍처 제한 해결 | SAD §9(LIM 상태), RTM §4.4, (관련 시) SRS §2.5(CON) |
| API/프로토콜 계약 변경 | SRS §5(외부 인터페이스), SAD, RTM |

### 13.2 정합성 재검증 절차 (RTM §5 형식)

1. SRS 요구사항 수 == RTM 행 수 확인.
2. 각 "구현/검증" 요구사항에 최소 1개 테스트 존재 확인.
3. 각 테스트가 유효한 요구사항 ID를 참조하는지 확인(고아 테스트 없음).
4. STP 커버리지 수치와 RTM 커버리지 수치 교차검증.
5. SAD 컴포넌트 매핑과 RTM 컴포넌트 참조 일치 확인.
6. `LIM-*` 제한사항 목록이 최신인지, LIM↔CON 교차참조가 유효한지 확인.
7. 상태 용어(`구현/검증`, `구현/미검증`)가 전 문서에서 통일되었는지 확인.

### 13.3 변경 이력 기록

- 갱신 시 반드시 대상 문서의 **버전(§4.3)·최종 수정일·변경 이력 표(§4.2)** 를 함께 갱신합니다.
- 문서 변경이 코드 변경과 같은 작업 단위에서 이루어지도록 하여 드리프트를 방지합니다.
- Definition of Done(문서 포함): 컴파일 성공(C++ `./build.sh` 에러 0, Go `go build ./...`), 테스트 통과(회귀 없음), 영향받은 SRS/SAD/STP/RTM 갱신·이력 기록, 상태 용어 통일 확인.

---

## 변경 이력

| 버전 | 날짜 | 변경 내용 | 작성자 |
|------|------|-----------|--------|
| 1.0.0 | {{YYYY-MM-DD}} | 초기 작성 (RTMC SRS/SAD/STP/RTM 문서 세트를 일반화하여 C/C++ 서버 프로젝트용 ISO 문서 규약 템플릿 작성) | {{AUTHOR}} |
