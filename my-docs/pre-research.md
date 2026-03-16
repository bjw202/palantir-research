# Palantir 온톨로지 통합 교안

## 엔터프라이즈 AI의 운영 원리를 온톨로지로 이해하다

> **문서 목적:** 이 교안은 두 가지 질문에 동시에 답한다.
>
> - *"팔란티어 온톨로지는 개념적으로 어떻게 설계되었는가?"* (아키텍처·구성 요소·OWL 비교)
> - *"팔란티어 온톨로지는 실제 운영에서 어떻게 동작하는가?"* (6단계 운영 루프·산업 사례·LLM 통합)\
>   \*\***전제 지식:** 온톨로지 기초(클래스·속성·추론·OWL)를 이해한 학습자\
>   \*\***예상 학습 시간:** 4\~6시간 (실습 포함)

---

## 목차

 1. [탄생 배경과 핵심 철학](https://claude.ai/chat/f2002f5e-8ed0-4e19-9625-df6603ca7d14#1-%ED%83%84%EC%83%9D-%EB%B0%B0%EA%B2%BD%EA%B3%BC-%ED%95%B5%EC%8B%AC-%EC%B2%A0%ED%95%99)
 2. [한 줄 정의와 디지털 트윈 개념](https://claude.ai/chat/f2002f5e-8ed0-4e19-9625-df6603ca7d14#2-%ED%95%9C-%EC%A4%84-%EC%A0%95%EC%9D%98%EC%99%80-%EB%94%94%EC%A7%80%ED%84%B8-%ED%8A%B8%EC%9C%88)
 3. [6단계 운영 메커니즘 — How It Works](https://claude.ai/chat/f2002f5e-8ed0-4e19-9625-df6603ca7d14#3-6%EB%8B%A8%EA%B3%84-%EC%9A%B4%EC%98%81-%EB%A9%94%EC%BB%A4%EB%8B%88%EC%A6%98)
 4. [핵심 구성 요소 상세 분석](https://claude.ai/chat/f2002f5e-8ed0-4e19-9625-df6603ca7d14#4-%ED%95%B5%EC%8B%AC-%EA%B5%AC%EC%84%B1-%EC%9A%94%EC%86%8C)
 5. [학문적 온톨로지 vs. Palantir — 개념 재매핑](https://claude.ai/chat/f2002f5e-8ed0-4e19-9625-df6603ca7d14#5-%EA%B0%9C%EB%85%90-%EC%9E%AC%EB%A7%A4%ED%95%91)
 6. [전체 아키텍처 — Foundry + AIP 스택](https://claude.ai/chat/f2002f5e-8ed0-4e19-9625-df6603ca7d14#6-%EC%A0%84%EC%B2%B4-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98)
 7. [보안 및 거버넌스 모델](https://claude.ai/chat/f2002f5e-8ed0-4e19-9625-df6603ca7d14#7-%EB%B3%B4%EC%95%88-%EB%B0%8F-%EA%B1%B0%EB%B2%84%EB%84%8C%EC%8A%A4)
 8. [산업별 실전 설계 패턴](https://claude.ai/chat/f2002f5e-8ed0-4e19-9625-df6603ca7d14#8-%EC%82%B0%EC%97%85%EB%B3%84-%EC%8B%A4%EC%A0%84-%EC%84%A4%EA%B3%84-%ED%8C%A8%ED%84%B4)
 9. [AIP — 온톨로지 위에 올라탄 AI](https://claude.ai/chat/f2002f5e-8ed0-4e19-9625-df6603ca7d14#9-aip%EC%99%80-%EC%98%A8%ED%86%A8%EB%A1%9C%EC%A7%80-%ED%86%B5%ED%95%A9)
10. [제조 현장 완전 시나리오: LLM + 온톨로지 동작 흐름](https://claude.ai/chat/f2002f5e-8ed0-4e19-9625-df6603ca7d14#10-%EC%A0%9C%EC%A1%B0-%ED%98%84%EC%9E%A5-%EC%99%84%EC%A0%84-%EC%8B%9C%EB%82%98%EB%A6%AC%EC%98%A4)
11. [Graph RAG vs. Palantir — 무엇이 다른가](https://claude.ai/chat/f2002f5e-8ed0-4e19-9625-df6603ca7d14#11-graph-rag-vs-palantir)
12. [한계, 비판, 대안 비교](https://claude.ai/chat/f2002f5e-8ed0-4e19-9625-df6603ca7d14#12-%ED%95%9C%EA%B3%84%EC%99%80-%EB%8C%80%EC%95%88)
13. [설계 원칙 추출 — 내 시스템에 적용하기](https://claude.ai/chat/f2002f5e-8ed0-4e19-9625-df6603ca7d14#13-%EC%84%A4%EA%B3%84-%EC%9B%90%EC%B9%99-%EC%A0%81%EC%9A%A9)
14. [핵심 질문 및 심화 과제](https://claude.ai/chat/f2002f5e-8ed0-4e19-9625-df6603ca7d14#14-%ED%95%B5%EC%8B%AC-%EC%A7%88%EB%AC%B8-%EB%B0%8F-%EC%8B%AC%ED%99%94-%EA%B3%BC%EC%A0%9C)

---

## 1. 탄생 배경과 핵심 철학

### 1-1. 문제 진단: 왜 기존 데이터 아키텍처가 실패하는가

Palantir가 온톨로지를 플랫폼의 핵심으로 채택한 것은 이론적 선택이 아니라 수십 개 고객사 배포 경험에서 나온 결론이다. Gotham(국방·정보 분석)과 Foundry(민간 기업 운영)를 반복 배포하면서 동일한 문제 패턴을 마주쳤다.

**문제 1: 데이터 사일로와 의미의 파편화**

```
영업팀의 "고객"  ≠  물류팀의 "고객"  ≠  재무팀의 "고객"
같은 단어, 다른 스키마, 다른 시스템 → 통합 불가
```

**문제 2: 분석은 되지만 실행이 안 된다**

대시보드는 현황을 보여주지만 "그래서 무엇을 해야 하는가"로 연결되지 않는다. 분석 결과를 실제 운영 시스템(ERP, CRM, SCADA)에 반영하려면 수동 작업이 필요하다.

**문제 3: AI가 맥락을 모른다**

LLM이 아무리 강력해도, 기업의 "Customer"가 무엇인지, "공정 이상"이 어떻게 정의되는지 모른다. 데이터 레이크에 쌓인 테이블과 컬럼은 기계가 읽을 수 있지만 **이해**할 수 없다.

### 1-2. 핵심 철학: 데이터 중심 → 결정 중심

Palantir는 조직이 존재하는 이유를 "결정을 내리는 것"으로 정의한다. 이 관점에서 데이터 시스템의 목적도 재정의된다.

| 기존 관점 | Palantir의 관점 |
| --- | --- |
| 데이터를 어떻게 저장할 것인가 | 어떤 결정을 지원할 것인가 |
| 쿼리 성능 최적화 | 의사결정 속도 최적화 |
| 스키마 설계 | 결정 시나리오 설계 |
| 데이터 정합성 | 결정의 일관성과 감사 가능성 |

**결정 데이터(Decision Data)의 개념:** Palantir의 온톨로지는 전통적 데이터(구조화·비정형·스트리밍)와 함께 **결정이 내려지는 과정에서 생성되는 데이터**도 포함한다. 어떤 맥락에서, 어떤 옵션들을 검토했고, 무엇을 선택했고, 그 결과는 무엇이었는지가 모두 온톨로지에 저장된다. 이것이 AI가 단순 데이터 조회를 넘어 **의사결정 패턴을 학습**할 수 있게 하는 기반이다.

---

## 2. 한 줄 정의와 디지털 트윈

### 2-1. 가장 짧은 정의

> **조직의 흩어진 데이터 자산을 현실 세계의 객체(Object)와 관계(Link)로 재구성하고,\
> 그 위에 로직(Function), 행동(Action), 권한(Security)을 얹어서\
> 사람과 소프트웨어와 AI가 같은 운영 세계를 공유하게 만드는 시스템**

Palantir 공식 문서는 온톨로지를 단순한 semantic layer가 아니라 조직의 **operational layer**로 설명하며, 이 구조가 data·logic·action·security의 4중 통합으로 의사결정을 모델링한다고 밝힌다.

### 2-2. 디지털 트윈으로서의 온톨로지

Palantir는 온톨로지를 조직의 **디지털 트윈(Digital Twin)** 으로 정의한다.

```
실세계 조직                       온톨로지 표현
─────────────────────────────────────────────
공장, 설비, 제품         →    Object Type
"이 설비가 저 공정에 사용됨"  →    Link Type
"가동 중단 시 에스컬레이션"  →    Action Type
센서 측정값, 직원 이름    →    Property
"최근 3일 평균 불량률 계산"  →    Function
```

이 트윈이 실시간으로 동기화된 상태를 유지할 때, 인간과 AI 에이전트가 공유하는 **"현실의 실행 가능한 표현"** 이 된다.

---

## 3. 6단계 운영 메커니즘

*이 섹션이 "How?"의 핵심이다. 온톨로지가 실제 조직에서 어떻게 동작하는지를 단계별로 추적한다.*

### 단계 1: 원천 데이터 수집

조직에는 이미 데이터가 여러 시스템에 흩어져 있다. 이 단계는 아직 "여러 데이터가 있음" 상태다. 즉 **현실이 아니라 시스템별 파편**이다.

| 도메인 | 원천 데이터 예시 |
| --- | --- |
| 제조 | ERP(주문/BOM/원가), MES(생산 이력), 설비 로그(알람/센서), 품질 시스템, 공급망, 정비 시스템 |
| 의료/공공 | 병원 운영 시스템, 대기자 명단, 인력 배치, 재정/예산, 기관별 행정 데이터 |
| 군사/국방 | ISR/정찰 데이터, 위치/트랙 데이터, 임무 계획, 보급/정비 자산, 보고서/통신 기록 |

### 단계 2: 현실 객체(Object Type)로 재정의

여기서 온톨로지가 등장한다. 팔란티어는 데이터를 먼저 "테이블"이 아니라 **객체 타입(Object Type)** 으로 재해석한다. 중요한 것은 이것이 단순 클래스 이름이 아니라는 점이다. **각 객체는 실제 데이터 소스에 연결(Backing Datasource)** 된다.

```
원래 상태                      Ontology 적용 후
─────────────────────────────────────────────
machine_log.csv       →    Machine 객체
erp_orders            →    Order 객체
mes_events            →    Production Run 객체
qa_results            →    Quality Incident 객체
```

### 단계 3: 객체 간 관계(Link Type) 정의

객체만 있으면 아직 "목록"이다. 운영은 관계에서 나온다.

```
SQL join  = 기술적 연결
Link Type = 업무 의미를 가진 연결
```

제조업 예시:

- `Lot → processed_on → Machine`
- `Tool → mounted_on → Machine`
- `Defect → observed_in → Lot`

이 순간, 데이터 조인이 **"업무 관계 탐색"** 으로 바뀐다.

### 단계 4: 비즈니스 로직(Function) 작성

여기서부터 팔란티어가 단순 그래프 플랫폼과 달라진다. Function은 **도메인 판단 로직**이다. 분석 코드가 아니다.

```
calculate_risk(machine)
→ 진동, 온도, 최근 불량률, 공구 수명, 작업 부하를 종합해서
→ 해당 machine이 다음 24시간 내 문제를 일으킬 가능성 반환
```

### 단계 5: 행동(Action)으로 실제 운영 변경

온톨로지는 읽기 전용 모델이 아니다. **쓰기(write-back) 가능한 운영 모델**이다.

```
Object   = 무엇을 다룰 것인가
Function = 어떻게 판단할 것인가
Action   = 그래서 무엇을 실행할 것인가
```

예시: 작업 지시 생성, 레시피 변경 승인, Lot 격리, 정비 요청 생성, 경보 발행

### 단계 6: 보안(Security)과 감사(Audit) 적용

현업에서 가장 중요한 것은 "누가 무엇을 볼 수 있고, 누가 무엇을 바꿀 수 있는가"다. 같은 `Patient`, `Asset`, `Mission` 객체를 보더라도:

- 어떤 사용자는 전체를 본다
- 어떤 사용자는 일부 속성만 본다
- 어떤 사용자는 집계값만 본다
- 어떤 사용자는 Action 실행 권한이 있다

> 온톨로지가 실제 조직 운영체계가 되려면 **추론 능력보다 거버넌스가 더 중요할 때가 많다.**

### 전체 운영 루프 요약

```
원천 데이터 수집
→ 현실 객체(Object) 정의
→ 객체 간 관계(Link) 정의
→ 비즈니스 로직(Function) 작성
→ 사용자가 화면에서 객체 단위로 조회
→ 시스템이 추천/판단 제공 (Function)
→ Action 실행으로 실제 상태 변경
→ 변경 결과가 다시 객체 상태에 반영
→ 다음 의사결정에 재사용 (닫힌 루프)
```

> **팔란티어는 데이터를 객체 세계로 재구성하고, 그 객체 세계에서 판단하고,\
> 그 결과를 다시 객체 세계에 반영하는 닫힌 루프(closed-loop)를 만든다.**

---

## 4. 핵심 구성 요소

### 4-1. Object Type — 세계를 개체로 정의하다

**정의:** 실세계의 개체(entity) 또는 이벤트(event)를 표현하는 스키마.\
OWL 클래스와 개념적으로 동일하지만, **실제 데이터소스와 직접 연결**된다는 점이 결정적 차이다.

```
Object Type: Machine
├── Properties
│   ├── machineId: string (primary key)
│   ├── name: string
│   ├── status: string ("active" | "maintenance" | "fault")
│   ├── lastMaintenanceDate: date
│   └── currentSensorTemp: double (실시간 갱신)
├── Backing Datasource: IoT 스트림 테이블 + ERP 설비 테이블 (조인)
├── Status: endorsed (핵심 자원, 삭제 불가)
└── Security: 설비 담당자만 status 편집 가능
```

**Object Type 생명주기:**

```
experimental → active → endorsed
                            ↓
                        deprecated → (삭제 예정 공지 + 대체 리소스 안내)
```

### 4-2. Property — 개체의 특성을 정의하다

**기본 타입:** `string`, `integer`, `double`, `boolean`, `date`, `timestamp`, `geopoint`

**OWL에 없는 확장 타입:**

```
미디어 참조 (Media Reference):
  - 이미지, 영상, 오디오, 문서를 Object에 직접 연결
  - 비전-언어 모델(VLM)과 연계하여 자동 분석 가능

시맨틱 검색 인덱스 (Semantic Search):
  - string 속성에 벡터 임베딩 인덱스 추가
  - "이와 유사한 불량 패턴을 가진 설비 찾기" 같은 의미 유사도 검색 가능
  - OWL SPARQL의 정확 매칭과 달리 의미적 근사 검색 지원

시계열 (Timeseries):
  - property.latestValue, property.history() 지원
  - 센서 데이터처럼 지속적으로 변하는 값을 효율적으로 처리
```

**Shared Property — 중앙화된 속성 관리:**

```
여러 Object Type이 "location" 속성을 공통으로 사용한다면?

일반 방식:  Machine.location, Vehicle.location, Employee.location 각각 정의
            → 정의가 달라질 수 있음, 유지보수 비용 3배

Shared Property 방식:
  shared_properties/
    └── location: geopoint  ← 한 번 정의
  Machine.location → shared:location 참조
  Vehicle.location → shared:location 참조
  → 의미가 보장되고, 메타데이터 변경이 일괄 적용됨
```

### 4-3. Link Type — 개체 사이의 관계를 정의하다

**정의:** 두 Object Type 사이의 관계 스키마.\
OWL의 Object Property에 대응하지만, 실제로는 **데이터셋 조인**으로 구현된다.

**구현 메커니즘 — OWL Triple vs. Palantir Join:**

```
OWL/RDF:
  <machine:001> <ontology:assignedTo> <line:A>  ← 트리플로 저장

Palantir:
  machine_table:  | machineId | lineId | name |
  line_table:     | lineId    | name   |      |
  
  Link Type은 machine_table.lineId = line_table.lineId 조인 정의
  → 기존 DB 구조를 그대로 유지하면서 관계를 온톨로지에 표현
  → 데이터 마이그레이션 없이 온톨로지 도입 가능
```

**Link를 통한 그래프 탐색 패턴:**

```
질문: "현재 고장난 설비가 담당하는 공정의 예약된 WorkOrder는?"

탐색 경로:
Machine (status="fault")
  → [assignedTo] → ProductionLine
  → [scheduledFor] → WorkOrder (status="scheduled")

이 탐색이 OSDK나 Workshop에서 자동 수행됨
```

### 4-4. Action Type — 변경을 통제하다 ← Palantir의 핵심 독창성

Action Type은 OWL에 존재하지 않는 개념이다. 이것이 Palantir 온톨로지를 단순 데이터 모델과 구분하는 핵심이다.

**원칙: "모든 쓰기(Write)는 Action을 통해야 한다. 직접 수정은 불가능하다."**

```
Action Type: EscalateDefect (불량 에스컬레이션)

입력 파라미터:
  - defectId: Defect 객체 (필수)
  - escalationReason: string (필수, 최소 20자)
  - assignedAnalyst: Employee 객체 (필수)
  - severity: enum("low", "medium", "high", "critical")

유효성 검사:
  - defect.status == "open" (이미 처리된 불량은 에스컬레이션 불가)
  - assignedAnalyst.role in ["QA_Lead", "Process_Engineer"]

변경 내용:
  - defect.status → "escalated"
  - defect.assignedTo → assignedAnalyst
  - defect.escalatedAt → now()

부수 효과:
  - 외부 알림 시스템 API 호출
  - JIRA 티켓 자동 생성
  - 감사 로그 자동 기록 (누가, 언제, 무슨 이유로)

승인 워크플로우:
  - severity == "critical"인 경우 → 상위 관리자 승인 필요
```

**Action Type의 세 가지 실행 모드:**

```
1. 즉시 실행 (Direct):     파라미터 입력 → 유효성 검사 → 즉시 적용
2. 검토 후 실행 (Staged):  LLM이 제안 → 사람이 검토 → 승인 시 적용
3. 자동화 실행 (Auto):     이벤트 트리거 → 조건 검사 → 자동 실행
```

**Action Type의 실질적 가치:**

```
Before: 분석가 A가 직접 DB UPDATE → 유효성 검사 없음, 감사 불가
After:  모든 변경 → Action Type → 검증 → 적용 → 자동 감사
        → 변경 이력 완전 보존, 규제 감사 대응
        → AI 에이전트도 인간과 동일 경로로 변경
```

### 4-5. Function — 비즈니스 로직을 코드로 정의하다

**정의:** Object와 ObjectSet을 입력받아 처리 결과를 반환하는 TypeScript/Python 코드.

```typescript
// 설비 건강 점수 계산 Function 예시
function calculateMachineHealthScore(machine: Machine): number {
  const daysSinceLastMaintenance = 
    daysDiff(machine.lastMaintenanceDate, now());
  const defectRate = 
    machine.getLinkedDefects()
           .filter(d => d.detectedAt > thirtyDaysAgo())
           .length / 30;
  const tempScore = 
    machine.currentSensorTemp < machine.normalTempMax ? 1.0 : 0.5;
  
  return (100 - daysSinceLastMaintenance * 0.5) * (1 - defectRate) * tempScore;
}
```

**Function 활용처:**

- Action Type의 유효성 검사 로직으로 호출
- Workshop(대시보드) 애플리케이션에서 파생 지표로 표시
- AIP Logic에서 LLM이 호출할 수 있는 도구(tool)로 등록
- OSDK를 통해 외부 애플리케이션에서 호출

### 4-6. Interface — 다형성으로 확장성 확보

**정의:** 여러 Object Type이 공통으로 가져야 할 "형태(shape)"를 정의하는 추상 타입.

```
문제: 제조 도메인에서 Machine, Vehicle, ProductionTool이
      모두 "자산(Asset)"으로 처리될 필요가 있다.

Interface 정의:
  interface IAsset {
    assetId: string
    status: string
    location: geopoint
    lastInspectedAt: timestamp
  }

구현:
  Machine implements IAsset
  Vehicle implements IAsset
  ProductionTool implements IAsset

→ "현재 위치에서 반경 500m 내 유지보수 필요한 자산 전체"
  쿼리가 단일 Interface 쿼리로 처리 가능
```

---

## 5. 개념 재매핑

### 5-1. OWL/RDF ↔ Palantir 전체 매핑

| OWL/RDF 개념 | Palantir 대응 개념 | 핵심 차이 |
| --- | --- | --- |
| `owl:Class` | Object Type | 추상 정의 vs. **실 데이터소스 바인딩** |
| `owl:Individual` | Object (인스턴스) | 수동 선언 vs. **파이프라인 자동 생성** |
| `owl:ObjectProperty` | Link Type | RDF Triple vs. **데이터셋 조인** |
| `owl:DatatypeProperty` | Property | XSD 타입 vs. **미디어·시맨틱 등 확장 타입** |
| `owl:Axiom` | Action Type + Function | 논리 공리 vs. **비즈니스 로직 + 트랜잭션** |
| `rdfs:subClassOf` | Interface (다형성) | 계층 상속 vs. **형태 기반 다형성** |
| Description Logic Reasoner | AIP (LLM 기반) | 형식 논리 추론 vs. **의미 기반 추론** |
| SPARQL | OSDK / 자연어 질의 | 기술 언어 vs. **개발자+비개발자 모두 접근** |
| Open World Assumption | Closed-action model | "모를 수 있다" vs. **"모르면 쓸 수 없다"** |

### 5-2. 가장 중요한 차이 — OWA vs. 거버넌스 모델

학문적 온톨로지는 **Open World Assumption(OWA)** 을 따른다. 기록되지 않은 것은 알 수 없다. 이것이 추론의 유연성을 만들지만, 운영 시스템에서는 문제가 된다.

Palantir는 OWA 대신 **"쓰기는 반드시 Action을 통한다"** 는 원칙을 채택했다. 이는 CWA(Closed World Assumption)에 가깝지만, 데이터의 진실성보다 **변경의 통제**에 초점이 있다.

```
OWL 세계:   누구나 트리플을 추가할 수 있다. 모순은 추론기가 사후 탐지.
Palantir:   변경은 반드시 사전 정의된 Action을 통해야 한다.
            Action에는 유효성 검사, 승인, 감사 추적이 내장된다.
```

### 5-3. 표현력 vs. 운영 실용성 트레이드오프

```
표현력 (무엇을 표현할 수 있는가)
높음 │ OWL Full   ← 추론 불가
     │ OWL DL    ← 완전한 논리 추론
     │ OWL Lite
낮음 │ Palantir Ontology ← 논리 추론 없음, 하지만 실시간 운영 가능

운영 실용성 (실제 조직에서 쓸 수 있는가)
낮음 │ OWL Full / DL
높음 │ Palantir Ontology ← 비개발자도 구축, 실시간 동기화, Action 거버넌스
```

Palantir는 논리적 완전성을 포기하는 대신 **운영 현장 채택 가능성**을 극대화했다. 이것은 결함이 아니라 의도적 설계 선택이다.

---

## 6. 전체 아키텍처

### 6-1. Foundry + AIP + Apollo 3층 구조

```
┌──────────────────────────────────────────────────────────┐
│  APOLLO — 자율 배포 관제탑                                 │
│  온톨로지 변경 사항을 엣지/클라우드/온프레미스에 자동 배포    │
└───────────────────────────┬──────────────────────────────┘
                            │
┌───────────────────────────┴──────────────────────────────┐
│  AIP — AI 운영 플랫폼                                      │
│  ┌─────────────┐ ┌─────────────┐ ┌──────────────────────┐ │
│  │ AIP Logic   │ │ Agent Studio│ │ AIP Evals            │ │
│  │ (no-code LLM│ │ (AI 에이전트│ │ (성능 평가)          │ │
│  └─────────────┘ └─────────────┘ └──────────────────────┘ │
│          ↕ 모두 온톨로지를 통해 읽고 쓴다                   │
└───────────────────────────┬──────────────────────────────┘
                            │
┌───────────────────────────┴──────────────────────────────┐
│  FOUNDRY — 데이터 운영 플랫폼                               │
│                                                           │
│  ┌───────────────────────────────────────────────────┐   │
│  │           ONTOLOGY LAYER (핵심)                    │   │
│  │  Object Types + Link Types + Action Types         │   │
│  │  Functions + Interfaces + Security Policies       │   │
│  │              ↑ Golden Record                       │   │
│  └───────────────────────────────────────────────────┘   │
│                                                           │
│  ┌───────────────────────────────────────────────────┐   │
│  │         DATA INTEGRATION LAYER                    │   │
│  │  Pipeline Builder (ETL) + Object Storage          │   │
│  │  Raw Datasets → Cleaned → Mapped to Object Types  │   │
│  └───────────────────────────────────────────────────┘   │
│                                                           │
│  ┌───────────────────────────────────────────────────┐   │
│  │              RAW DATA SOURCES                     │   │
│  │  DB · API · IoT Stream · Files · ML Models        │   │
│  └───────────────────────────────────────────────────┘   │
│                                                           │
│  APPLICATION LAYER (온톨로지 소비)                         │
│  Workshop(대시보드) · Quiver(분석) · Object Explorer      │
│  OSDK 기반 커스텀 앱 · AIP 에이전트                        │
└───────────────────────────────────────────────────────────┘
```

### 6-2. 읽기/쓰기 경로 상세

**읽기 경로 (Read Path):**

```
1. 사용자/에이전트 요청 (자연어 or OSDK 코드)
2. 온톨로지 레이어가 Object Type 스키마 확인
3. 보안 정책 검사
4. Object Storage에서 실제 데이터 반환
5. Link Type이 있다면 관련 Object까지 그래프 탐색
6. Function이 있다면 파생값 계산 후 포함
```

**쓰기 경로 (Write Path):**

```
1. 사용자/에이전트 Action 실행 요청 (파라미터 포함)
2. Action Type 스키마 조회
3. 파라미터 유효성 검사 (실패 시 즉시 거부 + 이유 반환)
4. 권한 검사
5. 승인 워크플로우 필요 시 → 대기 상태 전환
6. 승인 완료 또는 즉시 실행 → Object Storage 업데이트
7. 부수 효과 실행 (외부 시스템 API 호출)
8. 감사 로그 자동 기록
9. 영향받는 모든 애플리케이션에 변경 전파
```

---

## 7. 보안 및 거버넌스

### 7-1. 보안의 두 계층

**계층 1: 스키마 보안** — "이 Object Type 정의를 누가 볼 수 있고 수정할 수 있는가"

**계층 2: 데이터 보안** — "이 Object의 어떤 속성을 누가 볼 수 있는가"

```
행 수준 (Object-level):
  예: 각 병원 직원은 자신이 담당하는 환자 데이터만 볼 수 있다

열 수준 (Property-level):
  예: 일반 직원은 Employee.salary를 볼 수 없다. HR만 가능.

셀 수준 (Cell-level):
  행 + 열 조합으로 특정 셀 접근 제어
```

### 7-2. Granular Policy — 동적 접근 제어

```
Granular Policy 예시 (제조 도메인):
  정책명: "담당 공장 설비 접근"
  규칙: Machine.factoryId in current_user.authorizedFactoryIds

  → 수원 공장 담당자는 수원 공장 설비 데이터만 볼 수 있다
  → 글로벌 관리자는 모든 공장 설비를 볼 수 있다
  → 이 정책은 Workshop, OSDK, AIP 에이전트 모두에 동일하게 적용된다
```

### 7-3. 보안이 AI 에이전트에도 동일하게 적용된다

```
중요: AIP 에이전트가 온톨로지에 접근할 때도
      인간 사용자와 동일한 보안 정책이 적용된다.

→ 에이전트가 "다른 부서 데이터를 몰래 훔쳐보는" 일이 구조적으로 불가능하다.
→ LLM에게 "권한 없는 초능력"을 주지 않는다.
→ AI 거버넌스 컴플라이언스 달성을 시스템 수준에서 보장한다.
```

---

## 8. 산업별 실전 설계 패턴

### 8-1. 제조업 (스마트 팩토리)

**Object Types:** Plant, Process, Machine, Tool, Recipe, Lot, Measurement, Defect, Operator, Supplier

**Link Types:**

- `Lot → processed_on → Machine`
- `Tool → mounted_on → Machine`
- `Defect → observed_in → Lot`
- `Recipe → applied_to → Process`
- `Supplier → provides → Material`

**Functions:**

- 특정 Lot 불량의 원인 기여도 계산
- 설비 이상 조짐 점수 계산 (진동/온도/공구 수명 종합)
- 공정 조건 변경 시 품질 리스크 예측

**Actions:**

- 작업 지시 생성
- 레시피 변경 승인
- Lot 격리
- 정비 요청 생성
- 공급사 이슈 에스컬레이션

> **현업의 변화:** 질문이 "이번 주 불량률 몇 %냐?"에서\
> \*\***"이 불량 Lot를 살리려면 어디를 건드려야 하나?"** 로 바뀐다.

### 8-2. 의료/공공 (NHS 연방 데이터 플랫폼 사례)

NHS England는 Federated Data Platform(Palantir 기반)을 통해 NHS 전반의 정보를 연결해 더 나은 환자 케어와 정보 기반 의사결정을 지원하고 있다. 수술 일정 조정에는 staff availability, hospital beds, medication, equipment, waiting list 정보가 맞물리는데, FDP는 이 조각들을 한데 모아 더 빠르고 조정된 운영을 가능하게 했다. 2025년 9월 말 기준 NHS England의 theatre 관련 모듈 사용 trust들에서 누적 추가 시술 환자 수는 99,690명에 달했다.

**Object Types:** Patient Pathway, Procedure, Theatre Session, Clinician, Bed, Waitlist Entry, Hospital Site

**Link Types:**

- `Patient Pathway → requires → Procedure`
- `Procedure → scheduled_in → Theatre Session`
- `Theatre Session → staffed_by → Clinician`

**Functions:**

- 대기자 우선순위 재평가
- 사용되지 않는 Theatre Slot 탐지
- 당일 취소분 대체 추천

**Actions:**

- 환자 예약 확정
- 대기자 우선순위 변경
- 수술 일정 재배치 제안

### 8-3. 군사/국방 (NATO 사례)

NATO는 2025년 4월 Palantir Maven Smart System NATO를 Allied Command Operations에 도입했다. 미 육군도 2025년 7월 Palantir와의 enterprise agreement를 통해 데이터 통합, 분석, AI 도구를 제공해 readiness와 operational efficiency를 높이겠다고 밝혔다.

**Object Types:** Unit, Platform, Target, Sensor, Route, Area of Interest, Supply Node, Mission

**Link Types:**

- `Sensor → detects → Target`
- `Unit → assigned_to → Mission`
- `Mission → depends_on → Supply Node`

**Functions:**

- 위협 우선순위 점수 계산
- 보급 지속 가능 시간 계산
- 경로 리스크 평가

**Actions:**

- 관심 구역 변경, 자산 배치 변경, 경보 발행, 임무 계획 수정

> 군사에서 중요한 것은 **데이터를 전장 객체로 번역하는 일**이다.\
> 지휘관이 "상황판"이 아니라 **"운영 가능 세계"** 를 보게 만드는 것이다.

---

## 9. AIP와 온톨로지 통합

### 9-1. LLM이 없을 때의 한계

온톨로지만 있어도 객체 중심 운영은 가능하다. 하지만 여전히 다음에 취약하다.

- 복잡한 질의를 자연어로 하기 어렵다
- 문서/비정형 정보까지 함께 묻기 어렵다
- 다단계 판단 흐름을 직접 설계해야 한다
- 운영 로직을 사람이 일일이 따라가야 한다

**결론:** 온톨로지는 구조를 주지만, **상호작용성과 유연한 해석 능력**은 약하다.

### 9-2. LLM이 온톨로지와 결합하면

```
LLM 없을 때: 사람이 직접 화면을 넘기며 → 객체 찾기 → 관계 탐색 → 계산 → 판단 → 조치

LLM 결합 후: 사용자가 자연어로 질문
  "지금 가장 위험한 설비 5개와 이유를 보여줘"

  LLM이:
    → 질문 해석
    → 적절한 객체 쿼리 선택
    → 관계 탐색
    → Function 실행
    → 결과 요약
    → Action 제안 또는 실행
```

### 9-3. AIP Logic의 핵심 메커니즘

AIP Logic의 tool은 세 가지 범주를 가진다.

- **data tools**: Ontology 객체 조회, 검색
- **logic tools**: Function 실행, 계산
- **action tools**: Action 실행, 상태 변경

**결정적으로 중요한 원칙:**

> **LLM은 tool에 직접 접근하지 않는다.\
> LLM은 tool 사용을 "요청"하고, 실제 실행은 AIP Logic이 사용자의 권한 범위 내에서 수행한다.**

이 구조가 만드는 세 가지 효과:

1. **LLM에게 권한 없는 초능력을 주지 않는다** — 미리 허용된 query, function, action만 호출 가능
2. **자연어와 운영 시스템 사이에 제어층이 생긴다** — 질문 → 해석 → tool 요청 → 정책 검증 → 실행
3. **설명 가능성·감사 가능성이 높아진다** — 무슨 query, function, action을 했는지 추적 가능

### 9-4. Agent Studio의 컨텍스트 구조

Agent Studio에서 LLM이 받는 retrieval context는 매 사용자 메시지마다 세 가지를 포함한다.

```
1. Ontology context    → 구조화된 객체/관계 정보
2. Document context    → 보고서, 메모, SOP 같은 비정형 정보
3. Function-backed context → 사용자 정의 검색, 혼합 검색, 특수 계산 결과
```

**팔란티어식 LLM 통합 = Vector RAG + Graph RAG + Tool Calling + Operational Actions를 한 판에 묶는 구조**

### 9-5. 온톨로지와 LLM의 시너지

|  | 온톨로지가 LLM에 주는 것 | LLM이 온톨로지에 주는 것 |
| --- | --- | --- |
| 구조 | 객체 중심 스키마, 관계 탐색 경로 | 자연어 인터페이스 |
| 의미 | 도메인 의미, 보안 경계 | 문서+구조정보 통합 해석 |
| 실행 | 실행 가능한 Action 표면 | 다단계 판단 흐름 자동화 |
| 설명 | 감사 로그, 추적 가능성 | 설명 생성, 근거 제시 |

> **온톨로지는 LLM에게 세계의 구조를 주고,\
> LLM은 온톨로지에게 유연한 해석과 대화형 판단 인터페이스를 준다.**

---

## 10. 제조 현장 완전 시나리오

*LLM + 온톨로지가 실제로 돌아가는 모습을 처음부터 끝까지 추적한다.*

**사용자 질문:**

> "오늘 야간조에서 품질 리스크가 가장 높은 설비 3대와, 당장 해야 할 조치를 알려줘."

**Step 1. 질문 해석 (LLM)**

```
추출 정보:
  - 시간 범위: 오늘 야간조
  - 대상 객체: 설비(Machine)
  - 판단 목표: 품질 리스크 순위
  - 출력 목표: 조치(Action recommendation)
```

**Step 2. Ontology Query (data tools)**

```
허용된 object query tool로 다음을 조회:
  - 야간조에 가동 예정인 Machine 목록
  - 각 Machine의 최근 Lot 불량률
  - 센서 이상값 (진동, 온도)
  - Tool wear 상태
  - 최근 maintenance history
  - 현재 Recipe / Product mix

LLM은 raw SQL을 직접 쓰는 대신
허용된 object query 도구를 통해 객체와 Link 중심으로 접근한다.
```

**Step 3. Function 실행 (logic tools)**

```
machine_quality_risk_score(machine_id)
tool_failure_likelihood(tool_id)
lot_sensitivity(recipe_id, material_id)

각 함수는 Ontology 객체와 Link를 따라가며 계산한다.
```

**Step 4. 문서 컨텍스트 결합**

```
최근 엔지니어 메모, 작업 지침서, 특이 알람 설명 문서 → Document context로 추가
→ 단순 수치뿐 아니라 "왜 이 기계가 위험한가"를 문장으로 설명 가능
```

**Step 5. LLM 종합 판단**

```
구조화 결과 + 문서 컨텍스트 통합:

1위: Machine M-14
  이유: 최근 48시간 vibration drift 증가, Tool T-33 wear 임계치 근접,
        해당 레시피의 공차 민감도 높음

2위: Machine M-08
  이유: 동일 Lot family에서 edge burr 불량 재발

3위: Machine M-21
  이유: 작업자 변경 직후 setup offset 변동 이력
```

**Step 6. Action 제안 및 실행 (action tools)**

```
LLM이 제안하는 Action:
  - M-14 사전 점검 작업지시 생성
  - T-33 공구 교체 승인 요청
  - M-08에 대해 첫 3개 Lot 100% 검사
  - M-21 setup verification checklist 강제

→ 자동 실행 또는 사용자 확인 후 실행 (설정에 따라)
```

**Step 7. 실행 결과가 다시 Ontology에 반영**

```
작업지시 생성, 점검 상태 변경, 승인 여부, 결과 측정값이
다시 객체 속성/링크에 반영된다.
→ 다음 질문 때는 바뀐 세계 상태가 반영된다.
```

> **일반 챗봇은 답하고 끝난다.\
> 운영형 Ontology + LLM 시스템은 답한 뒤 세계를 바꾼다.**

---

## 11. Graph RAG vs. Palantir

*팔란티어를 단지 "Graph RAG 회사"로 보면 절반만 보는 것이다.*

**일반 Graph RAG:**

```
질문 → 엔티티 추출 → 그래프 검색 → 관련 문맥 생성 → LLM 응답 생성
```

**팔란티어식 운영형 Ontology + LLM:**

```
질문 → 객체 검색 → 관계 탐색 → 함수 실행 → 답변 → 액션 제안/실행 → 상태 업데이트 → 감사 로그
```

| 특성 | Graph RAG | Palantir 온톨로지 + LLM |
| --- | --- | --- |
| 목적 | 지식 검색 및 응답 생성 | 운영 실행 |
| 쓰기 능력 | 없음 | Action을 통한 상태 변경 |
| 거버넌스 | 없음 | 권한·감사·유효성 검사 내장 |
| 실시간 데이터 | 제한적 | Object Storage 실시간 동기화 |
| 비즈니스 로직 | 없음 | Function으로 온톨로지에 내장 |

> 팔란티어의 본질은 **"지식 검색 시스템"이 아니라 "운영 실행 시스템"** 에 더 가깝다.

---

## 12. 한계와 대안

### 12-1. 벤더 종속성과 비용

```
구조적 제약:
  - Foundry 플랫폼 전용 아키텍처
  - OSDK는 Palantir 특화
  - 다른 시스템과의 연동에 커스텀 통합 필요

비용 현실:
  - 플랫폼 라이선스: 수억~수십억 원 규모
  - 구현 컨설팅: 대형 SI 필요
  - 내부 온톨로지 엔지니어 육성: 6~12개월
```

### 12-2. 경쟁 솔루션 비교

| 특성 | Palantir Foundry | Microsoft Fabric | Databricks Unity Catalog | 자체 구축 (OSS) |
| --- | --- | --- | --- | --- |
| 온톨로지 지원 | 핵심 기능 | 제한적 | 메타데이터 수준 | 설계에 따라 다름 |
| AI 통합 | AIP (심층) | Copilot (중간) | AI Gateway (중간) | 완전 커스텀 |
| 거버넌스 | Action 기반 강력 | 중간 | 중간 | 설계에 따라 |
| 보안 | 셀 수준 | 테이블 수준 | 테이블 수준 | 설계에 따라 |
| 비용 | 매우 높음 | 중간 | 중간 | 낮음 (인건비 제외) |
| 벤더 독립성 | 낮음 | 낮음 | 중간 | 높음 |
| 비개발자 접근성 | 높음 | 높음 | 낮음 | 낮음 |

**중소 조직 대안:**

- Neo4j: 그래프 DB, Property Graph 방식
- Apache Jena + Protégé: OWL 기반, 오픈소스
- 자체 구축: **Python FastAPI + LangGraph + Neo4j**

### 12-3. 학문적 온톨로지(OWL/RDF)가 유리한 경우

```
1. 논리적 일관성이 생사와 연결될 때
   예: 의약품 상호작용, 항공기 시스템 인증
   → OWL DL + Reasoner로 모순 자동 탐지

2. 다른 조직의 온톨로지와 직접 연결해야 할 때
   예: 학술 데이터 공유, 정부 데이터 연계
   → RDF Linked Data, OWL/SPARQL 표준

3. 복잡한 서브클래스 추론이 필요할 때
   예: Gene Ontology, SNOMED CT
   → Description Logic Reasoner
```

---

## 13. 설계 원칙 적용

*Palantir 사례에서 플랫폼과 무관하게 추출할 수 있는 설계 원칙들.*

### 원칙 1: 데이터가 아닌 결정을 설계하라

```
나쁜 출발점: "우리가 가진 데이터를 온톨로지로 만들자"
좋은 출발점: "어떤 결정을 언제 누가 내려야 하는가?
             → 그것을 지원하는 온톨로지를 만들자"
```

### 원칙 2: 쓰기는 반드시 통제 지점을 통과하게 하라 (Action Pattern)

```python
# FastAPI로 Palantir Action Pattern 구현 예시
from pydantic import BaseModel, validator
from fastapi import FastAPI, Depends, HTTPException
from typing import Literal
from datetime import datetime

app = FastAPI()

class EscalateDefectAction(BaseModel):
    defect_id: str
    escalation_reason: str
    assigned_analyst_id: str
    severity: Literal["low", "medium", "high", "critical"]

    @validator("escalation_reason")
    def reason_must_be_detailed(cls, v):
        if len(v) < 20:
            raise ValueError("에스컬레이션 이유는 최소 20자 이상이어야 합니다")
        return v

@app.post("/actions/escalate-defect")
async def escalate_defect(
    action: EscalateDefectAction,
    current_user: User = Depends(get_current_user),
    db: Session = Depends(get_db)
):
    # 1. 비즈니스 유효성 검사
    defect = db.query(Defect).get(action.defect_id)
    if defect.status != "open":
        raise HTTPException(400, "이미 처리된 불량입니다")

    # 2. 상태 변경
    defect.status = "escalated"
    defect.assigned_to = action.assigned_analyst_id
    defect.escalated_at = datetime.now()

    # 3. 감사 로그 자동 기록
    audit_log = AuditLog(
        action_type="EscalateDefect",
        actor_id=current_user.id,
        target_id=action.defect_id,
        timestamp=datetime.now(),
        parameters=action.dict()
    )
    db.add(audit_log)
    db.commit()

    return {"success": True, "action_id": audit_log.id}
```

### 원칙 3: Golden Record — 하나의 정의, 여러 소비자

```
여러 팀이 각자 "설비" 개념을 정의하지 않도록:
  중앙 온톨로지에 Machine 정의 → 모든 팀이 참조

Palantir로 구현:
  endorsed Object Type + 모든 앱이 동일 온톨로지 소비

자체 구축 (OWL):
  단일 온톨로지 파일 + owl:imports로 팀별 확장
```

### 원칙 4: 보안은 데이터 레벨에서, 애플리케이션 레벨이 아니라

```
나쁜 패턴:
  앱 A에서는 admin만 볼 수 있게 UI를 막음
  앱 B에서는 필터를 빠뜨려서 누구나 볼 수 있음

좋은 패턴:
  데이터 레이어에서 접근 제어 → 어떤 앱을 통해도 동일하게 적용
  자체 구축: Row-level security + JWT claim 기반 필터
```

### 원칙 5: 온톨로지 생명주기 관리

```
experimental (개발) → active (운영) → deprecated (폐기 예고) → 삭제

폐기 시 반드시:
  - 이유 문서화
  - 마감 기한 명시
  - 대체 리소스 안내
  - 영향받는 소비자(앱, 팀)에 사전 공지
```

### 자신의 도메인에 적용하는 실습 프레임워크

```
Step 1: 결정 카탈로그 작성 (1일)
  당신의 도메인에서 매일/매주 내려지는 중요 결정 10개 나열
  각 결정에 대해:
    - 누가 내리는가
    - 어떤 정보가 필요한가
    - 결정 후 무엇이 변경되는가

Step 2: 온톨로지 스케치 (2일)
  결정 카탈로그를 바탕으로:
  - 핵심 Object Type (5~10개)
  - 핵심 Link Type (5~7개)
  - 핵심 Action Type (3~5개)

Step 3: 가장 중요한 결정 1개 선택하여 PoC 구현
  - Python FastAPI + Pydantic으로 Action 패턴 구현
  - Competency Question 5개를 API로 답하기

Step 4: 반복 확장
  PoC 검증 후 범위 확장
```

---

## 14. 핵심 질문 및 심화 과제

### 이해도 확인 질문

**개념 이해:**

1. Palantir 온톨로지에서 Action Type이 OWL 공리(Axiom)와 개념적으로 유사하지만 근본적으로 다른 이유는 무엇인가?
2. "결정 데이터(Decision Data)"가 단순 트랜잭션 데이터와 다른 점은 무엇이며, AI 학습에 어떤 이점을 주는가?
3. LLM이 AIP Logic에서 tool을 "직접 실행"하지 않고 "요청"만 하는 구조가 운영 안전성에 미치는 영향을 설명하라.
4. Palantir 온톨로지가 "단순 Graph RAG 이상"인 이유를 구조적으로 설명하라.

**비교 분석:** 5. OWL DL Reasoner와 AIP(LLM 기반)가 각각 "추론"을 어떻게 다르게 수행하는가? 각각이 더 적합한 상황은? 6. NHS FDP 사례에서 온톨로지가 없었다면 동일한 운영 최적화가 가능했을까? 어떤 한계가 있었을까? 7. 팔란티어의 보안 모델이 "애플리케이션 수준 접근 제어"보다 우월한 이유를 두 가지 제시하라.

**적용 설계:** 8. 레이저 공정 도메인에서 Palantir 방식으로 온톨로지를 설계할 때:

- Object Type 7개를 정의하고 각각의 핵심 Property 3개를 나열하라
- Link Type 5개를 정의하고 카디널리티를 명시하라
- Action Type 3개를 정의하고 각각의 유효성 검사 규칙을 작성하라
- AI 에이전트가 "현재 공정 품질이 저하된 원인"을 진단하는 탐색 경로를 그려라

### 심화 과제

**과제 1 — Python Action 패턴 구현 (2\~3일)**

FastAPI + Pydantic으로 Palantir Action Type의 핵심 개념을 OSS 스택으로 재현한다. 위 13장의 코드 예시를 출발점으로, 다음을 추가 구현하라:

- 유효성 검사 실패 시 구체적 오류 메시지 반환
- 감사 로그 DB 저장 (SQLite 또는 PostgreSQL)
- 승인 워크플로우 (critical severity 시 2단계 승인)
- 비동기 부수 효과 (이메일 또는 슬랙 알림)

**과제 2 — 도메인 온톨로지 OWL vs. Palantir 패턴 비교 (3\~5일)**

자신의 업무 도메인에서 동일한 Competency Question 5개를 두 가지 방식으로 구현하고 비교 보고서를 작성하라.

- OWL 버전: Protégé로 온톨로지 설계 → SPARQL로 CQ 답하기 → HermiT 추론기 일관성 검사
- Palantir 패턴 버전: Pydantic 모델로 Object Type, SQLAlchemy로 Link Type, FastAPI로 Action Type
- 비교 항목: 코드 양, 추론 능력 차이, 거버넌스 구현 용이성, 확장성

**과제 3 — Mini-AIP 아키텍처 설계 (5\~7일)**

Python + LangGraph + FastAPI로 Palantir AIP의 핵심 개념 구현:

```
구현 대상:
1. OntologyManager: Object Type 스키마 등록 및 관리
2. ActionExecutor: Action Type 정의 + 유효성 검사 + 감사 로그
3. OntologyAgent: LangGraph 기반 에이전트
   - 온톨로지 쿼리 도구
   - Action 실행 도구 (human-in-the-loop 옵션)
4. SimpleOSDK: Python 클라이언트 자동 생성 (Pydantic 모델 기반)

평가 기준:
- 에이전트가 자연어로 "고장 설비 리스트"를 조회할 수 있는가
- Action 실행 시 감사 로그가 자동 기록되는가
- 권한 없는 Action 시도 시 거부되는가
- 실행 결과가 다음 쿼리에 반영되는가 (닫힌 루프)
```

---

## 핵심 정리 5문장

> 1. **팔란티어 Ontology의 본질은 데이터 모델링이 아니라 운영 모델링이다.** 객체와 관계만 정의하는 것이 아니라, 그 위에 Function과 Action과 Security를 얹는다.
>
> 2. **온톨로지는 SQL join을 업무 관계로 바꾸고, raw table을 현실 객체로 바꾼다.** 이 변환이 일어나는 순간 BI 질문("불량률이 몇 %냐")이 운영 질문("이 불량을 살리려면 어디를 건드려야 하나")으로 바뀐다.
>
> 3. **Function은 분석 코드가 아니라 도메인 판단 로직이고, Action은 edit가 아니라 운영 개입 수단이다.** 두 개념이 합쳐져야 진정한 "실행 가능한 온톨로지"가 된다.
>
> 4. **LLM은 이 구조 위에서 자연어 인터페이스이자 문서와 객체 세계를 연결하는 조정자다.** 하지만 실제 조회와 실행은 tool과 권한 체계를 통해 통제된다.
>
> 5. **팔란티어의 강점은 "똑똑한 AI" 자체보다, 구조화된 운영 세계 위에 AI를 안전하게 붙이는 방식에 있다.** 그래서 "Graph RAG 회사"가 아니라 "온톨로지 기반의 운영형 의사결정 시스템 회사"다.

---

## 참고 자료

**Palantir 공식 문서**

- Ontology Overview: <https://www.palantir.com/docs/foundry/ontology/overview>
- Object Types Overview: <https://palantir.com/docs/foundry/object-link-types/object-types-overview/>
- Functions Overview: <https://palantir.com/docs/foundry/functions/overview/>
- Action Types Overview: <https://palantir.com/docs/foundry/action-types/overview/>
- AIP Architecture: <https://www.palantir.com/docs/foundry/architecture-center/aip-architecture>
- Agent Studio Tools: <https://palantir.com/docs/foundry/agent-studio/tools/>
- Ontology SDK: <https://www.palantir.com/docs/foundry/ontology-sdk/overview>
- AI Ethics & Governance: <https://palantir.com/docs/foundry/aip/ethics-governance/>

**실사례 자료**

- NHS England Federated Data Platform: <https://www.england.nhs.uk/digitaltechnology/nhs-federated-data-platform/>
- NATO Maven Smart System: <https://shape.nato.int/news-releases/nato-acquires-aienabled-warfighting-system->

**OWL/RDF 비교를 위한 배경 자료**

- Allemang & Hendler, *Semantic Web for the Working Ontologist*, 3rd Ed.
- Noy & McGuinness, *Ontology Development 101*, Stanford
- W3C OWL 2 Primer: <https://www.w3.org/TR/owl2-primer/>

**자체 구현을 위한 OSS 자료**

- Apache Jena (Java RDF/OWL): <https://jena.apache.org>
- RDFLib (Python): <https://rdflib.readthedocs.io>
- Neo4j Property Graph: <https://neo4j.com/docs>
- LangGraph (에이전트 오케스트레이션): <https://langchain-ai.github.io/langgraph>