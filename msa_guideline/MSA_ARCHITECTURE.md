# MSA 아키텍처 심층 레퍼런스

이 저장소에서 MSA(Microservices Architecture) 원칙의 **단일 진실 원천(SSOT)** 입니다. 언어에 독립적인 아키텍처 규약을 담으며, 언어별 가이드라인(`golang_guideline/`, `python_guideline/`, `cpp_guideline/`)은 심층 원칙을 반복 서술하지 않고 "상세는 `msa_guideline/MSA_ARCHITECTURE.md` 참조"로 위임한 뒤, 각 언어의 구체적 관용구·라이브러리만 담습니다. 실제 시스템에 복사할 때는 `{{PLACEHOLDER}}`를 채워 사용합니다.

짝 문서: `ISO_MSA_CONVENTION.md`(ISO 문서 규약·추적성은 짝 문서가 담당).

## 문서의 목적과 범위

- 목적: 시스템을 여러 서비스로 나눌 때의 **경계·통신·데이터·운영** 관심사에 대한 공통 결정 기준을 제공합니다. 각 영역을 "원칙 → 규칙 → 결정 기준(표) → 안티패턴" 순으로 실무 관점에서 서술합니다.
- 범위: 기존 언어 가이드라인은 "단일 REST API 서버 + 계층형(handler/router → service → repository)"을 전제합니다. 본 문서는 그 전제를 부정하지 않습니다. 계층형은 **각 서비스 내부**의 구조로 그대로 유지하고, 그 위에 서비스 분리·통신·데이터·운영 관심사를 더합니다.
- 표기: 기술 용어·라이브러리명·CLI·코드 식별자는 원문(영어) 그대로 둡니다(예: circuit breaker, saga, outbox, idempotency, OpenTelemetry, gRPC). 코드 예시는 특정 언어에 매이지 않도록 의사코드·다이어그램·표로 표현합니다.

## 채택 원칙: modular-monolith-first [핵심]

- **기본 스탠스**: "모듈러 모놀리스 우선(modular-monolith-first), 정당화될 때 서비스로 추출". 새 시스템은 경계가 명확한 모듈러 모놀리스로 시작하되, 처음부터 MSA로 무중단 추출이 가능하도록 **경계(bounded context)·데이터 소유권·계약(contract)** 을 설계합니다.
- 근거: 분산 시스템은 네트워크 실패·부분 장애·데이터 일관성·운영 복잡도라는 비용을 추가로 부과합니다. 이 비용은 조직·부하·배포 요구가 실제로 그것을 정당화할 때만 지불합니다. 경계가 불명확한 상태에서 성급히 쪼개면 **분산 모놀리스(distributed monolith)** 로 귀결됩니다.
- 모듈러 모놀리스에서 지켜야 할 "추출 준비" 조건:
  - 모듈 간 결합은 **인메모리 계약(interface)** 으로만 하고, 다른 모듈의 내부 구현·테이블에 직접 접근하지 않습니다.
  - 모듈별로 **데이터 소유권**을 분리합니다(같은 DB라도 스키마/테이블 소유를 나누고 cross-module JOIN을 금지). 이렇게 하면 나중에 물리적으로 분리하기 쉽습니다.
  - 도메인 이벤트를 모듈 경계에서 발행/구독하는 형태로 두면, 추출 시 in-process 이벤트 버스를 {{BROKER}} 기반 async로 교체하기 쉽습니다.
- 단지 "코드가 커져서"는 분리 사유가 **아닙니다**. 코드 규모는 모듈 분리·패키지 구조로 먼저 해결합니다.

### 서비스로 추출하는 기준

아래 항목 중 **다수가 참**일 때 해당 경계를 독립 서비스로 추출합니다. 하나만 걸리면 대개 모듈 분리로 충분합니다.

| 기준 | 추출을 정당화하는 신호 | 하나만으로 부족한 이유 |
| --- | --- | --- |
| 독립 배포·릴리스 주기 | 이 경계만 다른 주기로 배포·롤백해야 하고, 모놀리스 배포가 병목이 됨 | 배포 파이프라인 분리(모듈별 아티팩트)로도 일부 완화 가능 |
| 팀 경계·소유권 | 별도 팀이 소유하며 코드 오너십·on-call이 분리됨(Conway) | 코드 오너 파일·모듈 경계로 소유권을 표현할 수 있음 |
| 부하·확장 특성 | 이 경계만 트래픽/자원 특성이 크게 달라 독립 스케일링이 필요 | 단일 프로세스 내 스레드/워커 튜닝으로 대응 가능한 경우 있음 |
| 장애 격리 | 이 경계의 장애가 전체로 번지면 안 되고, 격벽(bulkhead)이 필요 | 프로세스 내 격리(스레드풀·타임아웃)로 부분 격리 가능 |
| 기술 스택 상이 | 언어/런타임/DB가 근본적으로 달라야 함(예: ML 워크로드) | 라이브러리 선택으로 해결되면 분리 불필요 |

- 판단은 되돌릴 수 있게 기록합니다. 추출 결정은 ADR(`docs/adr/`)로 남기고, 되돌릴 조건도 함께 적습니다.

## 1. 서비스 분해(Decomposition)

### 원칙

- **1 서비스 = 1 bounded context**를 기본 단위로 합니다(DDD). 서비스 경계는 조직/도메인 경계와 일치시키고, 데이터베이스 테이블이나 CRUD 단위로 나누지 않습니다.
- 경계는 조직 구조가 아니라 **도메인**을 기준으로 도출합니다. "함께 바뀌는 것은 함께 둔다(high cohesion)"와 "다르게 바뀌는 것은 분리한다(low coupling)"로 판단하고, 동일한 유비쿼터스 언어(ubiquitous language)를 공유하는 개념들을 한 서비스로 묶습니다.

### 규칙

- 경계 도출 절차(권장): (1) 이벤트 스토밍으로 도메인 이벤트·명령·aggregate 식별 → (2) 일관된 ubiquitous language 범위로 context 분할 → (3) context map으로 관계 정리(customer/supplier, conformist, anti-corruption layer) → (4) 트랜잭션 경계가 서비스 경계를 가로지르지 않도록 정렬.
- 서비스 간에 도메인 모델을 공유하지 않습니다. 각 context는 자기 모델을 갖고, 경계에서는 계약(DTO/이벤트 스키마)으로만 교환합니다(anti-corruption layer로 외부 모델을 자기 모델로 번역).
- 하나의 aggregate는 하나의 서비스가 소유합니다. 여러 서비스가 같은 aggregate를 write하면 경계 설계가 잘못된 것입니다.
- 서비스 내부는 기존 언어 가이드라인의 계층형(handler/router → service → repository)을 그대로 사용합니다. 분해는 서비스 **사이**의 관심사입니다.
- 새 엔티티가 생겼다고 새 서비스를 만들지 않습니다. 먼저 기존 context에 속하는지 검토합니다(과분해 방지).

### 결정 기준: 과분해 vs 과결합

| 축 | 과분해(too fine) | 과결합(too coarse) |
| --- | --- | --- |
| 대표 신호 | 하나의 유스케이스가 3개 이상 서비스를 동기 호출해야 완결됨 | 서로 다른 팀·주기의 기능이 한 서비스에 섞여 릴리스가 충돌 |
| 데이터 | 트랜잭션이 서비스 경계를 자주 넘음(분산 트랜잭션 유혹) | 한 서비스가 사실상 무관한 여러 도메인 데이터를 소유 |
| 통신 | 세밀한 호출이 잦음(chatty), 지연 누적 | 내부 결합이 커져 부분 변경이 어려움 |
| 변경 파급 | 사소한 기능 변경에 여러 서비스 동시 배포 필요(lockstep) | 한 부분 변경이 무관한 기능의 재배포·회귀를 유발 |
| 대응 | 서비스 병합 또는 경계 재설정, 동기 호출을 async로 전환 | context 기준으로 분리, 데이터 소유권부터 나눔 |

### 안티패턴

- **엔티티당 서비스(entity service)**: `User`, `Order`, `Item`마다 마이크로서비스를 만들어 유스케이스마다 다수의 동기 호출이 발생. → context 단위로 병합합니다.
- **분산 모놀리스의 씨앗**: 서비스는 나눴는데 함께만 배포·변경되는 상태. → 경계·계약을 재설계하거나 다시 합칩니다.

## 2. 데이터 소유권(Data)

### 원칙

- **database-per-service**: 각 서비스는 자기 데이터만 소유합니다({{DB}}). 다른 서비스의 DB에 **직접 접근하지 않습니다**(공유 DB 금지). 데이터 조회는 오직 그 데이터를 소유한 서비스의 **API 또는 이벤트**로만 합니다.
- 데이터의 단일 소유자(single writer)를 명확히 합니다. 한 데이터를 여러 서비스가 쓰면 소유권이 불명확해지고 일관성이 깨집니다.

### 규칙

- 서비스 간 조인이 필요하면 (a) 소유 서비스의 조회 API를 호출하거나(API composition, 대개 gateway/BFF에서 조합), (b) 이벤트로 필요한 데이터의 **읽기 전용 복제본(read model/materialized view)** 을 자기 DB에 구축합니다(CQRS의 부분 적용, 5장 참조).
- 복제 데이터는 "출처가 아니라 캐시"임을 명확히 합니다. 원본(source of truth)은 항상 소유 서비스에 있고, 복제본은 이벤트로 갱신되며 최종적으로 수렴(eventual consistency)합니다.
- DB 접근 자격증명은 서비스별로 분리하여, 물리적으로 같은 DB 인스턴스를 쓰더라도 스키마/계정 권한으로 소유 경계를 강제합니다(모듈러 모놀리스 → 추출 경로).

### 결정 기준: 다른 서비스 데이터가 필요할 때

| 상황 | 권장 방식 | 비고 |
| --- | --- | --- |
| 최신값이 즉시 필요, 호출 빈도 낮음 | 소유 서비스 조회 API 동기 호출 | timeout·circuit breaker 필수(7장) |
| 조회가 빈번, 약간의 지연 허용 | 이벤트로 read model 복제 후 로컬 조회 | eventual consistency 수용, 재구축 가능하게 설계 |
| 집계/리포팅(여러 서비스 데이터) | 별도 read model 서비스 또는 데이터 파이프라인 | 운영 DB에 cross-service JOIN 금지 |

### 안티패턴

- **공유 데이터베이스(shared database)**: 여러 서비스가 같은 테이블을 읽고 씀. 스키마 변경이 전 서비스로 파급되고 소유권이 사라짐. → database-per-service로 분리, 접근은 API/이벤트로.
- **뒷문 접근(back-door access)**: 다른 서비스의 DB에 직접 SQL. → 반드시 소유 서비스의 계약을 경유합니다.

## 3. 통신(Communication)

### 원칙

- 통신을 **동기(sync: REST/gRPC)** 와 **비동기(async: 메시지/이벤트 via {{BROKER}})** 로 명확히 구분합니다.
- 원칙: 즉시 응답이 필요한 **명령/조회**는 sync, 상태 전파·느슨한 결합·회복탄력성이 중요하면 async(event-driven). 서비스 간 강결합을 줄이기 위해 **가능하면 async를 선호**합니다.
- 동기 호출 체인은 **짧게** 유지합니다. A→B→C→D 형태의 긴 동기 체인은 지연·장애 전파의 원인이며, 하나라도 느려지면 전체가 느려집니다.

### 규칙

- 모든 원격 호출에는 timeout이 있어야 합니다(7장). 타임아웃 없는 동기 호출은 금지합니다.
- 이벤트는 "일어난 사실(past tense)"로 발행합니다(예: `OrderPlaced`). 이벤트에 소비자별 로직을 담지 않고, 소비자가 자기 판단으로 반응합니다. 명령(command)은 sync 또는 지정 큐로, 상태 변화 사실(event)은 async 발행으로 다룹니다.
- 이벤트/메시지 소비자는 idempotent해야 합니다(6장). 브로커는 대개 at-least-once 전달이므로 중복을 전제합니다.
- REST와 gRPC 선택: 외부 공개·브라우저·범용 상호운용은 REST(JSON), 내부 서비스 간 저지연·강타입 계약·스트리밍은 gRPC를 고려합니다.

### 결정 기준: sync vs async

| 축 | sync(REST/gRPC) 유리 | async(event/message) 유리 |
| --- | --- | --- |
| 즉시성(immediacy) | 호출자가 결과를 즉시 받아 다음 단계를 진행해야 함 | 결과를 나중에 반영해도 되고, 완료를 기다릴 필요 없음 |
| 결합도(coupling) | 대상 서비스·응답 스키마를 호출자가 알아야 함(강결합) | 발행자는 소비자를 모름(느슨한 결합, 소비자 추가가 쉬움) |
| 회복탄력성(resilience) | 대상 다운 시 즉시 실패(circuit breaker로 방어) | 브로커가 버퍼링, 소비자 다운 시 큐에 적재 후 복구 처리 |
| 일관성(consistency) | 단일 요청 내 강한 응답 일관성 필요 | 최종 일관성 허용, saga로 장기 트랜잭션 조율 |
| 부하 특성 | 저빈도·요청/응답형 | 스파이크 흡수(back-pressure), 팬아웃/브로드캐스트 |

### 통신 스타일별 장단점

| 스타일 | 장점 | 단점 |
| --- | --- | --- |
| REST(sync) | 단순·범용·디버깅 용이 | 강결합, 체인 지연 누적, 대상 장애에 취약 |
| gRPC(sync) | 강타입 계약, 저지연, 스트리밍 | 스키마/버전 관리 필요, 브라우저 직접 사용 제약 |
| event/message(async) | 느슨한 결합, 확장·회복탄력성, 스파이크 흡수 | 최종 일관성, 순서/중복/추적 난이도, 디버깅 복잡 |

### 이벤트 봉투(envelope) 최소 규격

async 이벤트는 공통 봉투(envelope)로 감싸 발행합니다. 이 최소 규격은 `ISO_MSA_CONVENTION.md`에서 시스템 공유 스키마(`docs/shared/`)로 고정하여 모든 이벤트가 따르도록 관리합니다. `event_id`는 소비자 dedup 키(6장), `trace_id`는 관측성 전파(8장)에 쓰입니다.

```json
{
  "event_id": "unique id (idempotency/dedup 용)",
  "event_type": "domain.entity.action",
  "event_version": "1",
  "occurred_at": "RFC 3339 timestamp",
  "trace_id": "분산 추적/상관관계용 ID (서비스 경계 넘어 전파)",
  "producer": "발행 서비스명",
  "data": { }
}
```

### 안티패턴

- **타임아웃 없는 동기 호출 체인**: 한 서비스의 지연이 전 체인의 스레드/커넥션을 고갈시킴. → 짧은 체인 + timeout + circuit breaker, 가능한 부분은 async로.
- **event를 command처럼 사용**: 이벤트에 "무엇을 하라"를 담아 사실상 원격 호출로 씀. → 이벤트는 사실 통지, 명령이 필요하면 명시적 command/API로.

## 4. API 게이트웨이/BFF

### 원칙

- 외부 클라이언트는 **게이트웨이({{GATEWAY}}) 또는 BFF(Backend for Frontend)** 를 통해서만 진입합니다. 내부 서비스를 외부에 직접 노출하지 않습니다.
- 게이트웨이 경계에서 **인증(authentication)·레이트리밋·라우팅·집계(aggregation)·프로토콜 변환**을 처리하여, 개별 서비스가 횡단 관심사를 중복 구현하지 않게 합니다.

### 규칙

- 인증은 게이트웨이에서(11장), **인가(authorization)** 는 각 서비스에서 수행합니다. 게이트웨이가 검증한 신원을 서비스로 전파합니다(mTLS 또는 검증된 JWT).
- BFF는 클라이언트 유형별(웹/모바일 등) 응답 집계·최적화가 필요할 때 도입합니다. 서비스 간 범용 로직을 BFF에 쌓지 않습니다(BFF가 새로운 모놀리스가 되지 않도록).
- 게이트웨이는 **얇게** 유지합니다. 비즈니스 로직은 서비스에 두고, 게이트웨이에는 라우팅·정책만 둡니다.

### 결정 기준: 게이트웨이 vs BFF vs 직접

| 상황 | 권장 |
| --- | --- |
| 다수 서비스, 단일 외부 진입점·공통 정책 | API gateway |
| 클라이언트 유형별 응답 형태가 크게 다름 | 유형별 BFF |
| 내부 서비스 간 호출 | 게이트웨이 우회, service discovery로 직접(mTLS) |

### 안티패턴

- **게이트웨이에 비즈니스 로직 축적**: 게이트웨이가 도메인 규칙을 알게 되어 배포 병목·강결합 발생. → 정책/라우팅만.
- **서비스 직접 노출**: 내부 서비스를 공개 인터넷에 그대로 노출. → 게이트웨이 뒤로 배치, 내부는 비공개 네트워크.

## 5. 데이터 일관성(Consistency)

### 원칙

- 서비스 경계를 넘는 트랜잭션은 **분산 트랜잭션(2PC)** 대신 **최종 일관성(eventual consistency)** 과 **saga 패턴**으로 처리합니다.
- 신뢰성 있는 이벤트 발행은 **transactional outbox** 패턴으로 dual write 문제를 방지합니다.
- 필요 시 **CQRS**를 부분 적용하여 읽기 모델을 분리합니다.
- 즉시 강한 일관성이 필요한 데이터는 애초에 같은 서비스(같은 트랜잭션 경계) 안에 두는 것이 옳습니다.

### saga: choreography vs orchestration

saga는 여러 로컬 트랜잭션의 시퀀스이며, 각 단계 실패 시 **보상 트랜잭션(compensating transaction)** 으로 이전 단계를 되돌립니다.

| 구분 | choreography(이벤트 기반) | orchestration(중앙 조율자) |
| --- | --- | --- |
| 제어 흐름 | 각 서비스가 이벤트를 구독하고 다음 이벤트 발행(분산) | orchestrator가 각 단계를 명령·응답으로 조율(중앙) |
| 결합도 | 낮음(서비스는 서로 모름) | orchestrator가 흐름을 앎(흐름 결합) |
| 가시성 | 흐름이 코드에 흩어져 추적 어려움 | 흐름이 한 곳에 있어 이해·모니터링 쉬움 |
| 적합 상황 | 단계 적고 단순, 느슨한 결합 우선 | 단계 많고 분기·보상이 복잡, 가시성 중요 |
| 주의 | 순환 이벤트·암묵적 의존 주의 | orchestrator가 단일 장애점/모놀리스화되지 않게 |

- 어느 방식이든 각 단계는 idempotent(6장)해야 하며, 보상 가능하도록 설계합니다(예: 예약→취소, 청구→환불). 되돌릴 수 없는 작업(이메일 발송 등)은 saga 후반부로 미루거나 pending 상태로 처리합니다.

### transactional outbox 흐름

문제: 로컬 DB 커밋과 브로커 발행을 각각 하면(dual write), 한쪽만 성공하는 부분 실패로 데이터와 이벤트가 어긋납니다. 해결: 이벤트를 같은 트랜잭션으로 `outbox` 테이블에 저장하고, 별도 relay가 발행합니다.

```text
[Service 트랜잭션]
  ┌─────────────────────────────────────────────┐
  │ BEGIN TX                                     │
  │   1) 비즈니스 상태 변경  (예: orders INSERT)   │
  │   2) 같은 TX 로 outbox INSERT (이벤트 레코드)  │
  │ COMMIT   ← 원자적: 둘 다 커밋되거나 둘 다 롤백  │
  └─────────────────────────────────────────────┘
                    │
                    ▼  (비동기, 별도 프로세스)
   [Relay / Message Relay]
     - outbox 에서 미발행 레코드 polling (또는 CDC 로 로그 tailing)
     - {{BROKER}} 로 발행 (at-least-once → 소비자는 idempotent)
     - 발행 성공 시 outbox 레코드를 published 로 표시(또는 삭제)
                    │
                    ▼
   [{{BROKER}}] ──▶ 소비자 서비스들 (idempotent 처리, dedup)
```

- 발행은 at-least-once이므로 **소비자 멱등성**이 반드시 필요합니다(6장). outbox 레코드에 고유한 event id를 부여해 소비자 dedup 키로 씁니다.
- polling 방식은 단순하지만 지연·부하가 있고, CDC(change data capture, DB 로그 기반) 방식은 지연이 낮지만 인프라가 필요합니다. 시스템 규모에 맞게 택합니다.
- 반대 방향(수신 측)에는 **inbox 패턴**(처리한 event id를 기록해 중복 소비 차단)을 함께 쓸 수 있습니다.

### CQRS 적용 시점

- 적용: 읽기와 쓰기의 부하/모델이 크게 다르거나, 여러 서비스 데이터를 합친 조회가 빈번할 때. 이벤트로 read model을 별도 저장소에 구축해 조회를 최적화합니다.
- 비적용(기본): 단순 CRUD, 읽기/쓰기 모델이 사실상 동일할 때. CQRS는 복잡도를 늘리므로 필요할 때만 부분 적용합니다.

### 안티패턴

- **dual write without outbox**: DB 커밋 후 별도로 브로커 발행 → 부분 실패로 불일치. → transactional outbox.
- **2PC/분산 락 남용**: 서비스 경계에 강한 분산 트랜잭션을 강제 → 가용성·확장성 저하. → saga + 최종 일관성.
- **보상 없는 saga**: 실패 시 되돌릴 수 없는 상태. → 각 단계에 보상 트랜잭션 정의.

## 6. 멱등성(Idempotency)

### 원칙

- 재시도 대상 API와 메시지/이벤트 소비자는 **idempotent**해야 합니다. 같은 요청/이벤트를 여러 번 처리해도 결과(상태·부수효과)가 한 번 처리한 것과 같아야 합니다.
- 이유: 네트워크 재시도, 브로커의 at-least-once 전달, 클라이언트 중복 제출은 **정상 조건**입니다. 중복은 예외가 아니라 전제입니다.

### 규칙

- 쓰기 API는 **idempotency key**(클라이언트가 생성한 고유 키)를 받아, 같은 키의 재요청은 최초 결과를 그대로 반환합니다. 키-결과 매핑을 저장(TTL 포함)합니다.
- 이벤트 소비자는 **처리 이력(dedup/inbox)** 을 유지합니다: `event_id`를 기록하고, 이미 처리한 id면 스킵합니다. 기록과 상태 변경은 같은 트랜잭션으로 커밋합니다.
- 자연 멱등 연산을 선호합니다: "잔액을 100으로 설정"(멱등)이 "잔액에 10 더하기"(비멱등)보다 안전합니다. 증분 연산에는 반드시 dedup을 붙입니다.

```text
[Idempotency key 처리 의사코드]
  key = request.header["Idempotency-Key"]
  if store.exists(key):
      return store.get_result(key)          # 최초 처리 결과 재반환
  result = do_work(request)                  # 실제 처리
  store.save(key, result, ttl)               # 결과 저장(가능하면 do_work 와 원자적으로)
  return result
```

### 결정 기준: 멱등 보장 위치

| 대상 | 멱등 수단 |
| --- | --- |
| 외부 클라이언트 쓰기 API | client 제공 Idempotency-Key + 결과 저장 |
| 서비스 간 재시도 호출 | 요청에 고유 id, 수신 측 dedup |
| 이벤트/메시지 소비 | `event_id` 기반 inbox/dedup, 처리와 기록을 원자적으로 |

### 안티패턴

- **at-least-once인데 소비자가 비멱등**: 중복 이벤트로 이중 청구·중복 생성. → dedup/inbox 필수.
- **재시도만 켜고 멱등성 미보장**: retry(7장)가 오히려 중복 부작용을 증폭. → 재시도 대상은 반드시 idempotent.

## 7. 회복탄력성(Resilience)

### 원칙

- 모든 원격 호출에 **timeout**을 둡니다. 재시도는 **exponential backoff + jitter**로 제한합니다. 반복 실패에는 **circuit breaker**를, 자원 격리에는 **bulkhead**를 적용합니다. 실패 시 **graceful degradation**을 설계합니다.

### 각 기법의 목적과 파라미터 가이드

| 기법 | 목적 | 기본 파라미터 가이드(출발점, 부하테스트로 조정) |
| --- | --- | --- |
| timeout | 느린 호출이 자원(스레드/커넥션)을 무한 점유하는 것 방지 | 하위 의존성 p99보다 약간 크게. 호출 체인 상위일수록 하위 timeout 합 이상 |
| retry(backoff+jitter) | 일시적(transient) 실패를 회복. 단, 과부하 증폭 방지 | 최대 2~3회, base 100~200ms, factor 2, max 2s, full jitter 적용 |
| circuit breaker | 반복 실패하는 의존성 호출을 차단해 빠르게 실패·회복 유도 | 실패율 임계(예: 50%/최근 N건) 초과 시 open, open 유지 10~30s, half-open 소량 probe |
| bulkhead | 한 의존성의 장애가 전체 자원을 잠식하지 못하게 격리 | 의존성별 동시 호출/커넥션 상한 분리(전용 pool), 초과분은 즉시 거절/대기 상한 |

### 규칙

- **재시도는 idempotent한 대상에만** 적용합니다(6장). 비멱등 연산에 재시도를 걸지 않습니다.
- jitter는 필수입니다. jitter 없는 동시 재시도는 **retry storm**(thundering herd)을 만듭니다. full jitter(0~backoff 사이 무작위)를 기본으로 합니다.
- timeout·재시도·breaker는 **호출 지점마다** 설정하며, 하드코딩하지 않고 설정으로 외부화합니다(10장).
- graceful degradation: 의존성 실패 시 캐시된/부분 응답, 기본값, 기능 축소로 핵심 경로를 살립니다. breaker가 open이면 대체 경로로 우회합니다.
- deadline propagation: 상위에서 정한 전체 deadline을 하위로 전파하여, 남은 시간이 없으면 재시도하지 않습니다.
- **dead letter queue(DLQ)**: 반복 실패한 메시지는 DLQ로 격리하여 소비 루프를 막지 않고 사후 분석·재처리합니다.

### circuit breaker 상태 전이

```text
        실패율 임계 초과
 CLOSED ───────────────▶ OPEN
   ▲                       │  (open 유지 시간 경과)
   │ probe 성공             ▼
   └──────── HALF-OPEN ◀────┘
             │  probe 실패
             └──────────▶ OPEN
  CLOSED     : 정상 통과, 실패 카운트 집계
  OPEN       : 즉시 실패(fail fast), 대상 호출 안 함
  HALF-OPEN  : 소량만 통과시켜 회복 여부 probe
```

### 안티패턴

- **무한 재시도·jitter 없는 재시도**: 장애 대상에 부하를 가중(retry storm). → 횟수 제한 + full jitter + breaker.
- **timeout 없음**: 스레드/커넥션 고갈로 연쇄 장애(cascading failure). → 모든 원격 호출에 timeout.
- **공유 pool**: 한 의존성 지연이 전 호출 자원을 잠식. → bulkhead로 의존성별 격리.

## 8. 관측성(Observability)

### 원칙

- 요청마다 식별자를 부여해 서비스 경계를 넘어 전파하고(**context propagation**), **분산 추적(distributed tracing)·구조적 로깅·메트릭**을 갖춥니다. 분산 시스템에서 관측성은 선택이 아니라 필수입니다.

### correlation ID vs trace ID

| 개념 | 정의 | 용도 |
| --- | --- | --- |
| correlation ID | 하나의 논리적 요청/작업 흐름을 묶는 상위 식별자 | 이벤트/비동기 흐름 포함해 "같은 업무"를 상관 |
| trace ID | 하나의 분산 trace(여러 span)를 식별(OpenTelemetry 표준) | 서비스 간 호출을 트리(span)로 시각화 |
| span ID | trace 내 개별 작업 단위 | 구간별 지연·오류 위치 파악 |

- OpenTelemetry를 권장 표준으로 삼고, trace context를 W3C `traceparent` 헤더로 전파합니다. 비동기 메시지에는 이벤트 봉투(3장)의 `trace_id` 등 메타데이터에 trace context를 실어 전파합니다. trace backend는 {{TRACING_BACKEND}}를 사용합니다.
- correlation ID는 게이트웨이(4장)에서 부여하고, 이후 모든 sync 호출 헤더와 async 메시지 메타데이터로 끝까지 전파합니다.

### RED 메트릭

서비스마다 최소한 RED를 계측합니다.

| 지표 | 의미 | 예시 활용 |
| --- | --- | --- |
| Rate | 초당 요청 수(처리량) | 트래픽 추세, 스케일링 트리거 |
| Errors | 실패 요청 수/비율 | 5xx·타임아웃·breaker open 알람 |
| Duration | 요청 지연 분포(p50/p95/p99) | SLO 위반·지연 회귀 탐지 |

- 리소스 관점(CPU/메모리/큐 길이 등)과 함께 봅니다. saga·async 흐름은 **소비 지연(consumer lag)·처리 실패·재시도 수**를 별도로 계측합니다. SLO/SLI를 정의하고 경보 임계값을 둡니다.

### 구조적 로깅

- 로그는 구조적(JSON 등)으로 출력하고, **모든 로그에 `trace_id`(및 `correlation_id`, `span_id`)를 포함**합니다. 이것이 로그와 trace를 연결하는 핵심입니다.
- 세 신호(logs·metrics·traces)는 correlation/trace ID를 공통 키로 상호 참조가 가능해야 합니다.
- 언어별 구체 로깅 라이브러리·필드 규약은 각 언어 `CODING_CONVENTION.md`를 따릅니다. 민감정보는 로깅하지 않습니다(마스킹).

```text
{ "ts":"...", "level":"error", "service":"{{SERVICE_NAME}}",
  "trace_id":"...", "span_id":"...", "correlation_id":"...",
  "msg":"downstream call failed", "target":"...", "err":"timeout" }
```

### 안티패턴

- **trace_id 없는 로그**: 서비스 경계를 넘는 요청을 재구성 불가. → 모든 로그에 trace_id 포함.
- **경계에서 context 미전파**: trace가 서비스마다 끊김. → sync 헤더·async 메타데이터로 전파.

## 9. 헬스체크/수명주기

### 원칙

- 각 서비스는 **liveness**와 **readiness**를 구분해 노출하고, 종료 시 **graceful shutdown**(진행 중 요청/메시지를 처리한 뒤 종료)을 구현합니다.

### 규칙

| 프로브 | 예시 경로 | 의미 | 실패 시 동작 |
| --- | --- | --- | --- |
| liveness | `/healthz` | 프로세스가 살아있고 데드락이 아님 | 오케스트레이터가 **재시작** |
| readiness | `/readyz` | 트래픽을 받을 준비됨(의존성 연결 OK) | 로드밸런서 로테이션에서 **제외**(재시작 아님) |

- liveness는 가볍게(프로세스 상태) 유지합니다. 하위 의존성까지 확인하면, 의존성 일시 장애로 멀쩡한 프로세스가 재시작되는 연쇄 장애가 생깁니다. 의존성 확인은 readiness에서 합니다.
- graceful shutdown 순서: (1) readiness를 실패로 전환해 새 트래픽 차단 → (2) 진행 중 요청/메시지 처리 완료 대기(상한 timeout) → (3) 브로커 구독 해지·커넥션 정리 → (4) 종료. 언어별 시그널 처리·구현은 각 `CODING_CONVENTION.md` 참조.
- 시작 시 필수 의존성 연결을 확인하고, 실패하면 명확히 실패(fail fast)합니다. 종료 유예 시간(grace period)은 오케스트레이터 강제 종료 시간보다 짧게 정합시킵니다.

### 안티패턴

- **liveness에 의존성 체크 포함**: 의존성 장애가 무관한 서비스 재시작을 유발. → 의존성은 readiness로.
- **즉시 종료(hard kill 방치)**: 진행 중 요청·미커밋 메시지 유실. → graceful shutdown 구현.

## 10. 설정/디스커버리

### 원칙

- **12-factor**에 따라 설정은 환경변수/설정 서버로 외부화합니다. 서비스 위치는 **service discovery/{{REGISTRY}}** 또는 오케스트레이터의 DNS로 해결하고, 주소를 하드코딩하지 않습니다.

### 규칙

- 코드·이미지에 환경별 값(엔드포인트, 자격증명)을 넣지 않습니다. 같은 이미지가 설정만 바꿔 모든 환경에 배포되도록 합니다(build once, deploy anywhere).
- 필수 설정 누락 시 **기동 실패(fail fast)** 로 조기에 드러냅니다.
- 서비스 주소 해결:

| 방식 | 설명 | 적합 |
| --- | --- | --- |
| DNS(오케스트레이터) | `{{SERVICE_NAME}}` 서비스명으로 DNS 해결 | Kubernetes 등 오케스트레이터 기본 |
| service registry({{REGISTRY}}) | 레지스트리에 등록·조회(client/server-side discovery) | 오케스트레이터 밖·동적 토폴로지 |

- secrets는 환경변수 평문이 아니라 시크릿 매니저로 주입합니다(11장). 설정 변경 시 무중단 반영 방식(재시작 vs 핫리로드)을 명시합니다.

### 안티패턴

- **주소 하드코딩**: IP/호스트를 코드·이미지에 고정 → 환경 이동·스케일링 불가. → DNS/registry.
- **환경별 이미지 빌드**: 환경마다 다른 이미지 → 테스트한 것과 배포본이 달라짐. → 설정 외부화, 단일 이미지.

## 11. 보안

### 원칙

- **zero-trust**: 내부망도 신뢰하지 않습니다. 게이트웨이에서 **외부 인증(authentication)**, 각 서비스에서 **인가(authorization)** 를 수행합니다. 서비스 간 통신은 **mTLS** 또는 검증된 **토큰(JWT) 전파**로 인증합니다.

### 규칙

- 게이트웨이(4장)가 외부 요청을 인증하고, 검증된 신원(클레임)을 하위 서비스로 전파합니다. 각 서비스는 전파된 신원으로 **자기 리소스에 대한 인가**를 독립적으로 판단합니다(게이트웨이 통과 = 인가 완료로 가정하지 않음).
- 서비스 간 호출은 전송 구간을 mTLS로 상호 인증·암호화하거나, 서명 검증 가능한 JWT를 전파합니다. 토큰은 만료·audience·scope를 검증합니다.
- secrets(키, DB 비밀번호, 토큰 서명키)는 시크릿 매니저로 주입하고, 코드·이미지·로그·에러 응답에 남기지 않습니다(10장).
- 최소 권한(least privilege): 서비스별 자격증명·네트워크 정책을 자기 소유 리소스로 한정합니다.

### 결정 기준: 서비스 간 인증 방식

| 방식 | 장점 | 고려사항 |
| --- | --- | --- |
| mTLS | 전송 구간 상호 인증·암호화, 애플리케이션 코드 부담 적음 | 인증서 발급·회전(rotation) 인프라 필요 |
| JWT 전파 | 세밀한 신원/클레임 전달, 게이트웨이 인증 재사용 | 서명키 관리·만료·검증 로직 필요, 토큰 크기 |

- 대개 둘을 함께 씁니다: 전송은 mTLS, 신원/권한 정보는 JWT 클레임으로 전파. 보안 요구사항은 ISO/IEC 27001 맥락으로 `ISO_MSA_CONVENTION.md`에 요구사항화하여 추적합니다.

### 안티패턴

- **내부망 암묵 신뢰**: 방화벽 안이니 인증 생략 → 측면 이동(lateral movement)에 취약. → zero-trust, 서비스 간 인증.
- **게이트웨이만 믿고 서비스 인가 생략**: 우회 경로/내부 호출로 무인가 접근. → 서비스에서 인가 재확인.

## 12. 배포/독립성(Deployment)

### 원칙

- 각 서비스는 **독립적으로 빌드·테스트·배포**됩니다(independent deployability). 서비스별 컨테이너 이미지와 파이프라인을 갖고, 오케스트레이션은 {{ORCHESTRATOR}}(예: Docker Compose/Kubernetes)로 합니다.
- 계약(API/이벤트 스키마)은 **버전 관리**하고, **consumer-driven contract test**로 호환성을 지킵니다.

### 규칙

- 한 서비스 배포가 다른 서비스의 동시 배포를 강제하지 않아야 합니다(no lockstep deploy). lockstep이 필요하면 그것은 분산 모놀리스 신호입니다.
- 계약은 명시적 산출물로 관리합니다: 동기 API는 OpenAPI(REST)/스키마 정의(gRPC), 비동기 이벤트는 이벤트 스키마(예: AsyncAPI 또는 JSON Schema)로 기술하고 `ISO_MSA_CONVENTION.md`의 계약/이벤트 카탈로그에 등록합니다.
- 계약 버전은 **semver**를 따릅니다:

| 변경 | semver | 호환성 |
| --- | --- | --- |
| 하위호환 깨짐(필드 제거/의미 변경, 필수 필드 추가) | major | backward-incompatible |
| 하위호환 기능 추가(optional 필드 추가) | minor | backward-compatible |
| 버그·문서 수정 | patch | 호환 |

- **backward compatibility**(새 서버가 옛 클라이언트 지원)와 **forward compatibility**(옛 소비자가 모르는 필드 무시)를 모두 고려합니다. 이벤트 소비자는 모르는 필드를 무시(tolerant reader)하도록 구현합니다.
- breaking change는 **expand/contract(parallel change)** 로 무중단 전개합니다: (1) 새 필드/엔드포인트를 추가(expand, 둘 다 지원, 필요 시 `/v1`·`/v2` 병행) → (2) 소비자 이전 → (3) 옛 것 제거(contract). 필드 즉시 제거·rename을 한 번에 하지 않습니다.
- **consumer-driven contract test(CDC)**: 소비자가 기대하는 계약을 명세로 만들고, 제공자 CI가 그 계약을 깨지 않는지 검증합니다. 제공자 단독 스키마 검사만으로는 실제 소비자 기대를 보장하지 못합니다.
- 배포 전략(rolling/blue-green/canary)은 무중단·빠른 롤백을 기준으로 택합니다. 배포와 릴리스(기능 노출)를 분리하려면 feature flag를 활용합니다.

### 안티패턴

- **lockstep deployment**: 여러 서비스를 항상 함께 배포. → 계약 호환성·expand/contract로 독립 배포.
- **계약 없는 결합**: 스키마 합의·버전·contract test 없이 필드에 의존. → 버전된 계약 + CDC.
- **breaking change 무통보 배포**: 소비자 파괴. → semver + expand/contract + CDC 게이트.

## 레포 전략(monorepo vs polyrepo)

- monorepo와 polyrepo 모두 유효합니다. **권장 기본값**: 초기·소규모는 monorepo(원자적 변경·리팩터링·공통 CI 용이), 팀/소유권이 갈라지면 polyrepo로 분리합니다.
- 어느 쪽이든 **서비스 간 코드는 계약(API/이벤트 스키마)으로만 결합**하고, 다른 서비스의 내부 구현을 직접 import하지 않습니다. monorepo라도 서비스 경계를 코드 의존 규칙으로 강제합니다.

| 축 | monorepo | polyrepo |
| --- | --- | --- |
| 원자적 교차 변경 | 쉬움(한 PR) | 어려움(여러 PR·조율) |
| 소유권 경계 | 도구로 강제 필요(code owners) | 저장소 단위로 자연 분리 |
| CI/빌드 | 영향 범위 한정(selective build) 필요 | 서비스별 단순 |
| 결합 위험 | 내부 직접 import 유혹(규칙으로 차단) | 물리적으로 격리 |

## 안티패턴 요약

| 안티패턴 | 증상 | 대안 |
| --- | --- | --- |
| 분산 모놀리스(distributed monolith) | 서비스는 나눴으나 동기 강결합 + 함께만 배포(lockstep) | 경계·계약 재설계, async 전환, 독립 배포(1·3·12장) |
| 공유 데이터베이스(shared DB) | 여러 서비스가 같은 테이블 읽고 씀, 스키마 변경 파급 | database-per-service, 조회는 API/이벤트(2장) |
| 타임아웃 없는 동기 호출 체인 | 한 서비스 지연이 전 체인 자원 고갈·연쇄 장애 | 짧은 체인 + timeout + circuit breaker + async(3·7장) |
| dual write(outbox 없음) | DB 커밋과 이벤트 발행 부분 실패로 상태·이벤트 불일치 | transactional outbox + 소비자 멱등성(5·6장) |
| 계약 없는 서비스 결합 | 버전·스키마 합의 없이 필드 의존, breaking change로 소비자 파괴 | 버전된 계약(semver) + consumer-driven contract test(12장) |
| 과분해(entity service) | 유스케이스마다 다수 동기 호출, 분산 트랜잭션 유혹 | bounded context 단위로 병합(1장) |
| chatty 통신 | 하나의 요청이 다수의 세밀한 서비스 호출을 유발 | 경계 재설계 또는 API composition/BFF(3·4장) |
| 내부망 암묵 신뢰 | 서비스 간 인증 생략, 측면 이동에 취약 | zero-trust, mTLS/JWT, 서비스 인가(11장) |

## 아키텍처 리뷰 체크리스트

새 서비스 도입·경계 변경·서비스 간 통신 추가 시 아래를 확인합니다.

- [ ] **분해**: 새 서비스는 bounded context 단위인가? "추출 기준" 다수를 충족하는가(단지 코드가 커져서가 아님)? 결정을 ADR로 남겼는가?
- [ ] **데이터**: 서비스가 자기 데이터만 소유하는가(database-per-service)? 다른 서비스 DB에 직접 접근하지 않는가? 복제 데이터의 source of truth가 명확한가?
- [ ] **통신**: sync/async 선택이 즉시성·결합도·회복탄력성·일관성 기준에 부합하는가? 동기 호출 체인이 짧은가? 모든 원격 호출에 timeout이 있는가?
- [ ] **게이트웨이**: 외부 진입이 게이트웨이/BFF를 경유하는가? 내부 서비스가 직접 노출되지 않는가? 게이트웨이에 비즈니스 로직이 없는가?
- [ ] **일관성**: 경계를 넘는 트랜잭션에 2PC 대신 saga를 쓰는가? 각 단계에 보상 트랜잭션이 있는가? 이벤트 발행에 transactional outbox를 쓰는가(dual write 없음)?
- [ ] **멱등성**: 재시도 대상 API·이벤트 소비자가 idempotent한가? idempotency key/inbox(dedup)가 있는가?
- [ ] **회복탄력성**: timeout·retry(backoff+jitter)·circuit breaker·bulkhead가 있는가? 재시도가 idempotent 대상에만 적용되는가? graceful degradation 경로가 있는가?
- [ ] **관측성**: correlation/trace ID가 경계를 넘어 전파되는가? 모든 로그에 trace_id가 포함되는가? RED 메트릭과 (async의 경우) consumer lag을 계측하는가?
- [ ] **수명주기**: liveness/readiness가 분리되어 있는가(liveness에 의존성 체크 없음)? graceful shutdown이 구현되어 있는가?
- [ ] **설정/디스커버리**: 설정이 환경변수/설정 서버로 외부화됐는가? 주소를 하드코딩하지 않고 DNS/{{REGISTRY}}로 해결하는가? secrets를 시크릿 매니저로 주입하는가?
- [ ] **보안**: zero-trust인가? 게이트웨이 인증 + 서비스 인가가 분리됐는가? 서비스 간 통신이 mTLS/검증된 JWT로 인증되는가?
- [ ] **배포/계약**: 서비스가 독립 배포 가능한가(no lockstep)? 계약이 semver로 버전 관리되는가? breaking change를 expand/contract로 전개하는가? consumer-driven contract test가 있는가?
- [ ] **모듈러 모놀리스(해당 시)**: 아직 분리하지 않았다면, 모듈 경계·데이터 소유권·계약이 추후 무중단 추출이 가능하도록 준비돼 있는가?
