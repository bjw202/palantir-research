# 스마트팩토리 AI 통합 전략 설계서

> **문서 목적:** 02-ontology-architecture.md에서 설계한 온톨로지 위에 AI/LLM을 결합하여,
> "현황 조회"를 넘어 "운영 실행"까지 자동화하는 닫힌 루프(Closed-Loop) 의사결정 시스템의 전략을 설계한다.
> Palantir AIP의 패턴을 충실히 벤치마크하되, 벤더 비종속적(Vendor-Agnostic) 관점에서 설계한다.
>
> **작성일:** 2026-03-16
>
> **참조:**
> - pre-research.md (팔란티어 온톨로지 통합 교안, 특히 섹션 9-10)
> - 01-problem-diagnosis.md (현장 문제 진단 결과)
> - 02-ontology-architecture.md (온톨로지 아키텍처 설계 결과)

---

## 1. AI 통합 아키텍처 개요

### 1-1. 3층 구조: 온톨로지 → AI 중간 계층 → 사용자 인터페이스

본 아키텍처는 팔란티어의 Foundry + AIP 스택을 벤치마크하되, OSS 기반으로 재설계한 3층 구조를 따른다.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    사용자 인터페이스 계층 (UI Layer)                       │
│                                                                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌────────────┐  │
│  │ 자연어 채팅   │  │ 대시보드      │  │ 모바일 알림   │  │ OSDK 패턴  │  │
│  │ (Web/Mobile)  │  │ (Workshop)   │  │ (Push/SMS)   │  │ 외부 앱 API│  │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └─────┬──────┘  │
│         │                 │                 │                │          │
├─────────┴─────────────────┴─────────────────┴────────────────┴──────────┤
│                    AI 중간 계층 (AIP Logic Layer)                         │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                   에이전트 오케스트레이터                         │    │
│  │  ┌──────────────┐ ┌──────────────┐ ┌──────────────────────┐    │    │
│  │  │ 품질 진단     │ │ 예지정비      │ │ 생산 최적화           │    │    │
│  │  │ Agent        │ │ Agent        │ │ Agent                │    │    │
│  │  └──────┬───────┘ └──────┬───────┘ └──────────┬───────────┘    │    │
│  │         └────────────────┴────────────────────┘                │    │
│  │                          │                                     │    │
│  │         ┌────────────────┼────────────────┐                   │    │
│  │         ▼                ▼                ▼                   │    │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐            │    │
│  │  │ Data Tools  │ │ Logic Tools │ │ Action Tools│            │    │
│  │  │ (읽기 도구)  │ │ (계산 도구)  │ │ (실행 도구)  │            │    │
│  │  └──────┬──────┘ └──────┬──────┘ └──────┬──────┘            │    │
│  └─────────┼───────────────┼───────────────┼────────────────────┘    │
│            │               │               │                         │
│     ┌──────┴───────────────┴───────────────┴──────┐                  │
│     │         정책 게이트 (Policy Gate)              │                  │
│     │    보안 검증 + 권한 확인 + 감사 로그           │                  │
│     └──────────────────────┬──────────────────────┘                  │
│                            │                                         │
├────────────────────────────┼─────────────────────────────────────────┤
│                    온톨로지 계층 (Ontology Layer)                       │
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────────┐  │
│  │               온톨로지 (Object + Link + Action + Function)       │  │
│  │  15개 Object Type / 14개 Link Type / 7개 Action Type / 7개 Fn   │  │
│  └─────────────────────────┬───────────────────────────────────────┘  │
│                            │                                          │
│  ┌─────────────────────────┴───────────────────────────────────────┐  │
│  │               데이터 통합 (Data Integration)                      │  │
│  │  ERP + MES + SCADA + QMS + WMS + CMMS + PLM + IoT Stream       │  │
│  └─────────────────────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────────────────────┘
```

### 1-2. 각 계층의 역할과 데이터 흐름

| 계층 | 역할 | 주요 구성 요소 |
|------|------|---------------|
| **온톨로지 계층** | 현실 세계의 디지털 트윈. 모든 데이터의 의미, 관계, 변경 규칙을 정의 | Object Type, Link Type, Action Type, Function, Security Policy |
| **AI 중간 계층** | LLM이 온톨로지를 "이해"하고 활용하는 연결부. Tool 호출, 에이전트 로직, 정책 검증 수행 | Agent Orchestrator, Data/Logic/Action Tools, Policy Gate, Context Manager |
| **사용자 인터페이스** | 인간과 AI 에이전트 간 상호작용 표면 | 자연어 채팅, 대시보드, 모바일 알림, API |

**데이터 흐름 (읽기 경로):**
```
사용자 질문 → LLM 해석 → Data Tool 선택 → 정책 게이트(권한 확인)
→ 온톨로지 쿼리 → Object/Link 탐색 → Function 실행 → 결과 반환
→ LLM 요약/해석 → 사용자에게 응답
```

**데이터 흐름 (쓰기 경로):**
```
LLM Action 제안 → 정책 게이트(권한 + 유효성 검사) → 승인 필요 여부 판단
→ [즉시 실행] 또는 [사용자 검토 요청] → Action 실행
→ Object 상태 변경 → 감사 로그 기록 → 부수 효과(알림, 외부 API)
→ 변경된 상태가 다음 쿼리에 반영 (닫힌 루프)
```

### 1-3. 핵심 원칙

> **"LLM은 도구를 직접 실행하지 않고, 요청만 한다. 실행은 온톨로지 계층에서 권한 범위 내에서 수행한다."**

이 원칙이 만드는 세 가지 구조적 효과:

1. **권한 없는 초능력 방지** -- LLM에게 미리 허용된 Tool(query, function, action)만 호출 가능하다. 임의의 SQL, 파일 접근, 시스템 명령은 구조적으로 불가능하다.
2. **자연어와 운영 시스템 사이의 제어층** -- 질문 해석 → Tool 요청 → 정책 검증 → 실행의 4단계를 반드시 거친다. LLM의 할루시네이션이 실제 운영에 영향을 주는 것을 차단한다.
3. **완전한 설명 가능성과 감사 가능성** -- 어떤 query, function, action을 요청했고, 어떤 결과가 반환되었는지 전 과정이 추적 가능하다.

---

## 2. Tool 구조 설계 (AIP Logic 패턴)

02-ontology-architecture.md에서 정의한 15개 Object Type, 14개 Link Type, 7개 Function, 7개 Action Type을 기반으로, LLM이 호출할 수 있는 Tool을 세 범주로 설계한다.

### 2-1. Data Tools (읽기 도구)

Data Tools는 온톨로지의 Object와 Link를 조회하는 읽기 전용 도구다. LLM이 "무엇을 알아야 하는가"를 판단한 후, 적절한 Data Tool을 선택하여 호출한다.

#### 객체 검색 도구 (Object Query Tools)

| Tool 이름 | 입력 파라미터 | 출력 형식 | 설명 |
|-----------|-------------|----------|------|
| `searchMachines` | `filters: { status?, lineId?, machineType?, healthScoreRange? }` | `Machine[]` | 조건에 맞는 설비 목록 반환 |
| `getMachineDetail` | `machineId: string` | `Machine (full properties + linked summary)` | 특정 설비의 전체 속성 + 연결된 Tool, Lot, MaintenanceRecord 요약 |
| `searchLots` | `filters: { status?, productId?, dateRange?, defectRateRange? }` | `Lot[]` | 조건에 맞는 Lot 목록 반환 |
| `getLotDetail` | `lotId: string` | `Lot (full properties + genealogy)` | 특정 Lot의 전체 속성 + 공정 계보(Machine, Recipe, Material, Operator) |
| `searchDefects` | `filters: { status?, defectType?, severity?, dateRange?, lotId? }` | `Defect[]` | 조건에 맞는 불량 목록 반환 |
| `searchWorkOrders` | `filters: { status?, orderType?, priority?, assignedTo?, dateRange? }` | `WorkOrder[]` | 조건에 맞는 작업 지시 목록 반환 |
| `getShiftInfo` | `filters: { plantId, date?, shiftType? }` | `Shift (with operators)` | 교대 정보 + 배정된 작업자 목록 |
| `searchSuppliers` | `filters: { qualityScoreRange?, category? }` | `Supplier[]` | 조건에 맞는 공급사 목록 반환 |

#### 관계 탐색 도구 (Link Traversal Tools)

| Tool 이름 | 입력 파라미터 | 출력 형식 | 설명 |
|-----------|-------------|----------|------|
| `traceDefectCause` | `defectId: string` | `{ lot, machine, tools[], recipe, materials[], operator, shift }` | 불량의 전체 원인 추적 체인을 한 번에 반환 (품질 역추적 체인) |
| `findRelatedLots` | `condition: { machineId? \| recipeId? \| materialId? \| operatorId? }, dateRange` | `Lot[]` | 특정 조건(동일 설비/레시피/자재/작업자)을 공유하는 Lot 검색 |
| `getMachineLineage` | `machineId: string` | `{ line, plant, tools[], recentLots[], maintenanceRecords[], workOrders[] }` | 설비의 전체 관계 맥락 반환 |
| `getSupplierImpact` | `supplierId: string, dateRange` | `{ materials[], lots[], defects[], defectRate }` | 공급사가 영향을 미친 전체 생산 체인 추적 |

#### 집계/통계 도구 (Aggregation Tools)

| Tool 이름 | 입력 파라미터 | 출력 형식 | 설명 |
|-----------|-------------|----------|------|
| `getDefectRateByMachine` | `dateRange, groupBy?: "day" \| "week"` | `{ machineId, defectRate, trend }[]` | 설비별 불량률 집계 + 추세 |
| `getShiftPerformance` | `plantId, dateRange` | `{ shiftType, avgDefectRate, avgOEE, incidentCount }[]` | 교대별 성과 비교 |
| `getProductionSummary` | `lineId?, dateRange` | `{ totalLots, completedLots, avgDefectRate, totalDefects, OEE }` | 생산 라인별 실적 요약 |
| `getMaintenanceSummary` | `machineId?, dateRange` | `{ totalRecords, byType, avgResolutionTime, pendingCount }` | 정비 이력 요약 통계 |
| `getSupplierRanking` | `evaluationPeriod` | `{ supplierId, qualityScore, trend }[]` | 공급사 품질 순위 |

#### 보안 필터 적용 방식

모든 Data Tool은 호출 시 다음 보안 필터를 자동 적용한다:

```
1. 행 수준 필터: current_user.authorizedPlantIds 기반 데이터 범위 제한
   → 수원 공장 담당자가 호출하면 수원 공장 데이터만 반환

2. 열 수준 필터: current_user.role 기반 민감 속성 마스킹
   → 재무 정보(unitCost, maintenanceCost)는 권한 있는 역할만 조회

3. 감사 로그: 모든 Data Tool 호출은 자동 기록
   → (actor: "user:OPR-001", via: "agent:quality-diagnosis", tool: "traceDefectCause", params: {...})
```

### 2-2. Logic Tools (계산 도구)

02-ontology-architecture.md에서 정의한 7개 Function을 LLM이 호출 가능한 Tool로 래핑한다. Logic Tool은 읽기 + 계산을 수행하되, 온톨로지 상태를 변경하지 않는다.

| Tool 이름 | 래핑 대상 Function | 호출 맥락 | 입력 요약 |
|-----------|-------------------|----------|----------|
| `evaluateMachineHealth` | `calculateMachineHealthScore` | "위험한 설비 알려줘", "M-14 상태 어때?" | `machineId: string` |
| `analyzeDefectContribution` | `calculateDefectContribution` | "이 불량 원인이 뭐야?", "왜 이렇게 불량이 많지?" | `defectId: string` |
| `assessRecipeChangeRisk` | `assessProcessRisk` | "이 레시피 바꾸면 괜찮을까?", "공정 조건 변경 리스크는?" | `recipeId: string, proposedChanges: JSON` |
| `predictToolRemainingLife` | `predictToolWear` | "이 공구 언제 교체해야 해?", "T-33 수명 얼마나 남았어?" | `toolId: string` |
| `evaluateLotSensitivity` | `calculateLotSensitivity` | "지금 진행 중인 Lot 중 위험한 거 있어?" | `lotId: string` |
| `evaluateSupplierPerformance` | `evaluateSupplierQuality` | "공급사 품질 어때?", "S-07 최근 성적은?" | `supplierId: string, period: DateRange` |
| `estimateDowntimeImpact` | `estimateDowntimeImpact` | "이 설비 세우면 영향이 어때?", "정비하면 생산 차질 얼마나?" | `machineId: string, estimatedDowntime: Duration` |

#### Function 체이닝 패턴

단일 Function으로 답할 수 없는 복합 질문은 여러 Function을 순차/병렬로 조합한다. LLM이 이 체이닝을 자율적으로 설계한다.

**패턴 1: 순차 체이닝 (Sequential Chaining)**
```
질문: "M-14 설비를 지금 정비하면, 가장 큰 피해를 보는 주문이 뭐야?"

Step 1: evaluateMachineHealth(M-14)     → 현재 건강 점수 확인
Step 2: estimateDowntimeImpact(M-14, 4h) → 다운타임 영향도 산출
Step 3: searchWorkOrders(machineId=M-14, status=scheduled) → 영향받는 작업 지시 조회
→ LLM이 결과를 종합하여 "가장 납기가 촉박한 주문"을 식별
```

**패턴 2: 병렬 체이닝 (Parallel Chaining)**
```
질문: "오늘 야간조에서 가장 주의해야 할 설비 3대는?"

Parallel:
  - searchMachines(status=active, shift=night) → 야간 가동 설비 목록
  - getShiftInfo(plantId, date=today, shiftType=night) → 야간조 인력 현황

For each machine (Parallel):
  - evaluateMachineHealth(machineId)

→ LLM이 건강 점수 기준 하위 3대 + 야간조 인력 대비 리스크 종합 판단
```

**패턴 3: 조건부 체이닝 (Conditional Chaining)**
```
질문: "불량 D-2024-0891의 원인을 찾고, 같은 원인의 다른 불량이 있는지 확인해줘"

Step 1: analyzeDefectContribution(D-2024-0891) → 기여 요인 분석
Step 2: (기여도 1위가 Tool이면)
         → findRelatedLots(toolId=T-33, dateRange=last7Days)
         → searchDefects(lotIds=[...])
        (기여도 1위가 Recipe면)
         → findRelatedLots(recipeId=R-42, dateRange=last7Days)
         → searchDefects(lotIds=[...])
→ LLM이 패턴 일치 여부 판단, 영향 범위 산정
```

### 2-3. Action Tools (실행 도구)

02-ontology-architecture.md에서 정의한 7개 Action Type을 LLM이 **제안/요청**할 수 있는 Tool로 래핑한다.

| Tool 이름 | 래핑 대상 Action | 실행 모드 | 설명 |
|-----------|-----------------|----------|------|
| `proposeEscalateDefect` | `EscalateDefect` | 즉시 실행 / critical 시 승인 필요 | 불량 에스컬레이션 제안 |
| `proposeCreateWorkOrder` | `CreateWorkOrder` | 즉시 실행 / urgent maintenance 시 승인 | 작업 지시 생성 제안 |
| `proposeQuarantineLot` | `QuarantineLot` | 즉시 실행 (품질 안전 즉시 조치) | Lot 격리 제안 |
| `proposeChangeRecipe` | `ChangeRecipe` | 검토 후 실행 (3단계 승인) | 레시피 변경 제안 |
| `proposeRequestMaintenance` | `RequestMaintenance` | emergency 즉시 / 그 외 자동화 | 정비 요청 제안 |
| `proposeAssignOperator` | `AssignOperator` | 즉시 실행 | 작업자 배정 제안 |
| `proposeUpdateMachineStatus` | `UpdateMachineStatus` | 즉시 실행 / 복구 시 서명 필요 | 설비 상태 변경 제안 |

#### 실행 모드별 동작

```
┌────────────────────────────────────────────────────────────────────┐
│ 실행 모드 1: 즉시 실행 (Direct)                                     │
│                                                                    │
│ LLM "proposeQuarantineLot" 호출                                     │
│  → 정책 게이트: 파라미터 유효성 검사 + 사용자 권한 확인               │
│  → 통과: 즉시 실행 → Object 상태 변경 → 감사 로그 → 부수 효과       │
│  → 실패: 거부 사유 반환 → LLM이 사용자에게 설명                      │
│                                                                    │
│ 예시: QuarantineLot, AssignOperator, UpdateMachineStatus            │
├────────────────────────────────────────────────────────────────────┤
│ 실행 모드 2: 사용자 검토 후 실행 (Staged)                            │
│                                                                    │
│ LLM "proposeChangeRecipe" 호출                                      │
│  → 정책 게이트: 파라미터 유효성 검사                                 │
│  → 통과: 사용자에게 검토 요청 UI 표시                                │
│    ┌─────────────────────────────────────────────────────┐        │
│    │ AI 제안: 레시피 R-42를 v2.3 → v2.4로 변경             │        │
│    │ 변경 내용: 피드 속도 15% 감소                          │        │
│    │ 리스크 평가: 23점 (저위험)                              │        │
│    │ 근거: M-14 진동 이상과 피드 속도 상관관계 분석            │        │
│    │                                                       │        │
│    │  [승인]  [수정 후 승인]  [거부]                          │        │
│    └─────────────────────────────────────────────────────┘        │
│  → 승인 시: 승인 워크플로 진행 (공정 → 품질 → 생산 3단계)           │
│                                                                    │
│ 예시: ChangeRecipe, EscalateDefect(critical)                        │
├────────────────────────────────────────────────────────────────────┤
│ 실행 모드 3: 자동 실행 (Auto)                                       │
│                                                                    │
│ 이벤트 트리거 → 조건 검사 → 자동 실행                                │
│  예: 설비 건강 점수 40 미만 → 자동 RequestMaintenance(predictive)    │
│  예: Tool 마모율 90% 이상 → 자동 CreateWorkOrder(maintenance)       │
│                                                                    │
│ 자동 실행도 반드시 감사 로그 기록 + 사후 알림                        │
└────────────────────────────────────────────────────────────────────┘
```

#### 안전장치 설계

```
핵심 원칙: LLM은 Action을 "제안(propose)"만 한다. 실행 여부는 정책이 결정한다.

안전장치 1: 파라미터 유효성 검사
  → Action Type에 정의된 모든 유효성 규칙을 정책 게이트가 검증
  → LLM이 비현실적 파라미터를 제안해도 자동 차단

안전장치 2: 권한 매트릭스 적용
  → 현재 사용자의 역할에 따라 실행 가능한 Action이 제한됨
  → Operator가 에이전트를 통해 ChangeRecipe를 요청해도 "제안" 권한 없으면 거부

안전장치 3: 승인 워크플로
  → critical/emergency 조건에서는 반드시 상위 관리자 승인 필요
  → AI가 아무리 확신해도 승인 단계를 건너뛸 수 없음

안전장치 4: 속도 제한 (Rate Limiting)
  → 단일 세션에서 Action 실행 횟수 제한 (configurable)
  → 비정상적 대량 Action 요청 자동 차단

안전장치 5: 롤백 가능성
  → 모든 Action의 이전 상태가 감사 로그에 기록
  → 오류 발견 시 역Action 실행 가능 (예: 격리 해제)
```

---

## 3. 제조 현장용 AI 에이전트 설계

Palantir Agent Studio 패턴을 적용한 3개 전문 에이전트를 설계한다. 각 에이전트는 특정 도메인에 특화된 Tool 세트, 컨텍스트, 시스템 프롬프트를 가진다.

### 3-1. 품질 진단 에이전트 (Quality Diagnosis Agent)

#### 역할
불량 근본 원인 추적 + 영향 범위 산정 + 시정/예방 조치 제안

#### 사용하는 Tools

| 범주 | Tools |
|------|-------|
| **Data Tools** | `searchDefects`, `getLotDetail`, `getMachineDetail`, `traceDefectCause`, `findRelatedLots`, `getDefectRateByMachine`, `getSupplierImpact` |
| **Logic Tools** | `analyzeDefectContribution`, `evaluateMachineHealth`, `evaluateLotSensitivity`, `evaluateSupplierPerformance` |
| **Action Tools** | `proposeEscalateDefect`, `proposeQuarantineLot`, `proposeCreateWorkOrder(inspection/rework)` |

#### 컨텍스트 구조

```
Ontology Context (구조화):
  - 대상 Defect 객체 + 연결된 Lot, Machine, Tool, Recipe, Material, Operator 전체 그래프
  - 동일 설비/레시피/자재의 최근 30일 불량 패턴

Document Context (비정형):
  - SOP (Standard Operating Procedure): 해당 공정의 표준 작업 절차서
  - 작업 지침서: 설비별 셋업 가이드, 공차 관리 기준
  - 엔지니어 메모: 최근 공정 변경 사유, 특이사항 기록
  - CAPA 이력: 과거 유사 불량의 시정/예방 조치 기록

Function-backed Context (계산 결과):
  - calculateDefectContribution 결과: 기여 요인별 백분율
  - calculateMachineHealthScore 결과: 관련 설비의 건강 상태
  - 최근 센서 데이터 요약: 불량 발생 시점 전후 30분 센서 트렌드
```

#### 대화 시나리오 예시

```
사용자 (품질 엔지니어):
  "오늘 아침에 라인 A에서 surface 불량이 급증했어. 원인 좀 찾아줘."

에이전트 내부 동작:
  1. searchDefects(defectType="surface", dateRange=today, lineId="LINE-A")
     → 12건 탐지, Lot 3개에 집중 (L-0317, L-0318, L-0319)
  2. Parallel: traceDefectCause(각 Lot의 defectId) × 12건
     → 공통점 발견: 모두 Machine M-14, Recipe R-42 v2.3
  3. analyzeDefectContribution(대표 defectId)
     → 기여도: Recipe 변경 45%, Tool 마모 32%, 자재 변동 15%, 기타 8%
  4. Document context 검색: "R-42 v2.3 변경 이력"
     → 어제 피드 속도 15% 증가 변경 확인

에이전트 응답:
  "오늘 라인 A에서 surface 불량 12건이 발생했습니다. 3개 Lot(L-0317~0319)에 집중되어 있습니다.

  근본 원인 분석 결과:
  1순위 (45%): 어제 적용된 Recipe R-42 v2.3의 피드 속도 15% 증가
    - 동일 레시피를 사용한 타 설비(M-08, M-21)에서는 불량 미발생
    - M-14의 현재 Tool T-33 마모율(82%)과 증가된 피드 속도의 복합 작용으로 추정
  2순위 (32%): Tool T-33 마모 진행 (현재 82%, 교체 권장 시점 85%)
  3순위 (15%): 자재 Lot M-Al-0892의 입고 검사 경계값 (공차 상한 근접)

  권장 조치:
  [즉시] M-14의 레시피를 R-42 v2.2로 롤백 → [레시피 롤백 실행]
  [오늘 중] Tool T-33 교체 예약 → [정비 요청 생성]
  [모니터링] L-0317~0319 추가 검사 지시 → [검사 작업 지시 생성]

  동의하시면 각 조치를 실행하겠습니다."

사용자:
  "레시피 롤백이랑 공구 교체 진행해줘. Lot 검사도."

에이전트:
  → proposeChangeRecipe(R-42, rollback to v2.2) → 승인 워크플로 시작
  → proposeRequestMaintenance(M-14, "Tool T-33 교체", urgency=high)
  → proposeCreateWorkOrder(type=inspection, targetMachine=M-14,
      description="L-0317~0319 surface 불량 관련 추가 전수검사")

  "3건의 조치를 요청했습니다.
   - 레시피 롤백: 공정 엔지니어 → 품질 관리자 → 생산 관리자 승인 대기 중
   - Tool T-33 교체: 정비팀에 요청 완료 (작업 지시 WO-2024-1847)
   - 추가 검사: 작업 지시 WO-2024-1848 생성 완료, QA팀에 알림 발송됨"
```

### 3-2. 예지정비 에이전트 (Predictive Maintenance Agent)

#### 역할
설비 건강 모니터링 + 정비 시점 예측 + 생산 일정 영향 분석 + 정비 스케줄 제안

#### 사용하는 Tools

| 범주 | Tools |
|------|-------|
| **Data Tools** | `searchMachines`, `getMachineDetail`, `getMachineLineage`, `searchWorkOrders`, `getMaintenanceSummary`, `getShiftInfo` |
| **Logic Tools** | `evaluateMachineHealth`, `predictToolRemainingLife`, `estimateDowntimeImpact`, `evaluateLotSensitivity` |
| **Action Tools** | `proposeRequestMaintenance`, `proposeCreateWorkOrder(maintenance)`, `proposeUpdateMachineStatus`, `proposeAssignOperator` |

#### 기존 ML 모델과의 통합 방법

```
┌──────────────────────────────────────────────────────────────────┐
│           예지정비 에이전트의 ML 모델 통합 구조                     │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ LLM (예지정비 에이전트)                                    │    │
│  │   "M-14의 잔여 수명을 예측해줘"                            │    │
│  └──────────────────────┬──────────────────────────────────┘    │
│                         │ Tool 호출                              │
│                         ▼                                        │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ Logic Tool: evaluateMachineHealth                        │    │
│  │   → calculateMachineHealthScore Function 실행            │    │
│  │   → 내부에서 ML 모델 API 호출:                            │    │
│  │     ┌───────────────────┐  ┌───────────────────┐        │    │
│  │     │ 진동 분석 ML 모델  │  │ RUL 예측 ML 모델   │        │    │
│  │     │ (vibration_model) │  │ (rul_model)        │        │    │
│  │     │ Input: 시계열센서  │  │ Input: 멀티센서    │        │    │
│  │     │ Output: 이상 확률  │  │ Output: 잔여 수명  │        │    │
│  │     └───────────────────┘  └───────────────────┘        │    │
│  │   → ML 예측값이 Function 결과에 포함되어 반환              │    │
│  └─────────────────────────────────────────────────────────┘    │
│                         │                                        │
│                         ▼                                        │
│  LLM이 ML 예측값 + 온톨로지 컨텍스트(정비 이력, 생산 일정)를     │
│  종합하여 자연어로 해석하고 조치를 제안                           │
└──────────────────────────────────────────────────────────────────┘
```

핵심: LLM은 ML 모델을 직접 호출하지 않는다. ML 모델은 온톨로지 Function 내부에 래핑되어 있으며, LLM은 Function의 결과만 받는다. 이렇게 하면 ML 모델의 입력 전처리, 스케일링, 버전 관리가 Function 계층에서 통제된다.

#### 대화 시나리오 예시

```
사용자 (설비 관리자):
  "다음 주 정비 스케줄을 잡아야 하는데, 어떤 설비를 우선 정비해야 할까?"

에이전트 내부 동작:
  1. searchMachines(plantId="PLT-SUWON", status=active)
     → 가동 중 설비 42대 조회
  2. Parallel: evaluateMachineHealth(각 machineId) × 42
     → 건강 점수 산출 + 리스크 레벨 분류
  3. 하위 10대에 대해: predictToolRemainingLife(장착 Tool)
  4. 하위 5대에 대해: estimateDowntimeImpact(4h 기준)
  5. getShiftInfo(PLT-SUWON, next week) → 다음 주 정비 인력 가용 확인

에이전트 응답:
  "다음 주 정비 우선순위 분석 결과입니다.

  긴급 정비 필요 (건강 점수 40 미만):
  1. M-14 (점수: 34) — Tool T-33 잔여 수명 2일, 진동 상승 추세
     → 다운타임 영향: 4시간 정비 시 생산 손실 약 320개, 납기 지연 리스크 1건
     → 권장: 월요일 야간 정비 (생산 영향 최소화)
  2. M-22 (점수: 38) — 베어링 진동 패턴 이상, RUL 예측 5일
     → 다운타임 영향: 3시간 정비 시 생산 손실 약 180개
     → 권장: 화요일 교대 전환 시간 활용

  예방 정비 권장 (건강 점수 40~60):
  3. M-08 (점수: 52) — 캘리브레이션 주기 도래
  4. M-31 (점수: 55) — 쿨런트 교체 시기
  5. M-17 (점수: 58) — 필터 교체 예정

  다음 주 정비 인력: 주간 2명, 야간 1명 가용
  → M-14(월 야간) + M-22(화 주간)을 우선 배정하면 인력 제약 내에서 최적입니다.

  정비 작업 지시를 생성할까요?"
```

### 3-3. 생산 최적화 에이전트 (Production Optimization Agent)

#### 역할
생산 일정 최적화 + 설비 할당 + 에너지 효율 + 병목 분석

#### 사용하는 Tools

| 범주 | Tools |
|------|-------|
| **Data Tools** | `searchMachines`, `searchLots`, `searchWorkOrders`, `getProductionSummary`, `getShiftPerformance`, `getShiftInfo`, `searchSuppliers` |
| **Logic Tools** | `evaluateMachineHealth`, `estimateDowntimeImpact`, `evaluateLotSensitivity`, `assessRecipeChangeRisk` |
| **Action Tools** | `proposeCreateWorkOrder(production)`, `proposeAssignOperator`, `proposeUpdateMachineStatus` |

#### 대화 시나리오 예시

```
사용자 (생산 관리자):
  "이번 주 금요일까지 제품 X를 500개 추가 납품해야 해. 가능한지 확인하고 최적 스케줄 짜줘."

에이전트 내부 동작:
  1. searchMachines(productFamily="X", status=active) → 가용 설비 파악
  2. getProductionSummary(product="X", dateRange=thisWeek) → 현재 생산 속도 확인
  3. Parallel: evaluateMachineHealth(각 설비) → 가용 설비의 안정성 평가
  4. getShiftInfo(remainingDays) → 남은 인력 가용도 확인
  5. estimateDowntimeImpact(각 설비, 예상 추가 가동 시간)

에이전트 응답:
  "500개 추가 생산 가능성 분석 결과입니다.

  현재 상황:
  - 제품 X 일일 생산 능력: 320개/일 (라인 A + B)
  - 이번 주 잔여 생산일: 3일 (수~금)
  - 계획 생산량: 기존 주문 750개 + 추가 500개 = 1,250개
  - 필요 일일 생산량: 417개/일

  최적 스케줄:
  Option 1 (권장): 라인 C 추가 가동
  - 라인 C의 M-31, M-32를 제품 X 생산으로 전환 (현재 idle)
  - 추가 생산 능력: +150개/일 → 총 470개/일
  - 금요일 17:00까지 1,410개 생산 가능 (여유 160개)
  - 리스크: M-31 건강 점수 55 (주의 필요), 교대 인력 1명 추가 배정 필요

  Option 2: 야간 연장 근무
  - 라인 A, B 야간조 2시간 연장
  - 추가 생산: +80개/일 → 총 400개/일
  - 금요일까지 1,200개 (50개 부족 가능)
  - 리스크: 야간 연장 시 피로도 증가, 품질 리스크 상승

  Option 1을 진행할 경우:
  - 라인 C 전환 작업 지시 생성
  - M-31 사전 점검 정비 요청
  - 야간조 추가 인력 1명 배정

  진행할까요?"
```

---

## 4. 기존 ML 모델 통합 방안

### 4-1. 스마트팩토리에서 일반적으로 사용하는 ML 모델 유형

| 분류 | 모델 유형 | 입력 데이터 | 출력 | 현재 한계 |
|------|----------|-----------|------|----------|
| **예측정비** | 진동 분석 (FFT + CNN/LSTM) | 가속도 센서 시계열 | 이상 확률, 주파수 스펙트럼 이상 | "이상 확률 73%"만 출력, 조치 연결 없음 |
| **예측정비** | RUL 예측 (Remaining Useful Life) | 멀티센서 시계열 (온도, 진동, 전류) | 잔여 수명 (시간/일) | 예측 불확실성 전달 어려움, 정비 일정 연동 없음 |
| **품질 예측** | SPC 이상 탐지 (Isolation Forest, Autoencoder) | 공정 파라미터 시계열 | 이상 점수, 기여 변수 | 정적 임계치 기반, 컨텍스트(레시피 변경 등) 미반영 |
| **품질 예측** | 불량 분류 (Vision CNN/ViT) | 검사 이미지 | 불량 유형, 위치, 심각도 | 분류만 하고 원인 추론 없음 |
| **수요/스케줄링** | 수요 예측 (Prophet, Transformer) | 과거 주문/판매 시계열 | 향후 N일 수요 | 설비 상태, 자재 가용성과 비연동 |
| **수요/스케줄링** | 생산 스케줄링 (OR/CP Solver, RL) | 주문, 설비, 인력 제약 | 최적 생산 일정 | 실시간 변동(고장, 긴급 주문) 반영 어려움 |

### 4-2. ML 모델을 온톨로지 Function으로 래핑하는 방법

```python
# 예시: 진동 분석 ML 모델을 온톨로지 Function으로 래핑

class VibrationAnalysisFunction:
    """
    온톨로지 Function: analyzeVibrationPattern
    - 온톨로지의 Machine 객체와 Measurement 시계열을 입력으로 받음
    - 내부에서 ML 모델을 호출하되, 결과를 온톨로지 의미 체계로 변환하여 반환
    """

    def __init__(self, ml_model_endpoint: str):
        self.model = MLModelClient(ml_model_endpoint)

    def execute(self, machine: Machine) -> VibrationAnalysisResult:
        # 1. 온톨로지에서 데이터 수집 (Object + Link 탐색)
        measurements = machine.getLinkedMeasurements(
            type="vibration", period="last_24h"
        )
        maintenance_history = machine.getLinkedMaintenanceRecords(
            period="last_6months"
        )

        # 2. ML 모델 입력 전처리 (Function 내부에서 통제)
        features = self._preprocess(measurements, maintenance_history)

        # 3. ML 모델 추론
        prediction = self.model.predict(features)

        # 4. ML 결과를 온톨로지 의미 체계로 변환
        return VibrationAnalysisResult(
            anomaly_probability=prediction.score,
            dominant_frequency=prediction.peak_freq,
            likely_cause=self._map_to_ontology_cause(prediction),
            # ML 모델이 "bearing_wear" 출력 → 온톨로지의 MaintenanceRecord 유형으로 매핑
            recommended_action="predictive_maintenance" if prediction.score > 0.7 else "monitor",
            confidence=prediction.confidence,
            model_version=self.model.version,
            # 설명 가능성: 어떤 주파수 대역이 이상인지 기록
            explanation=prediction.feature_importance
        )
```

### 4-3. ML 예측 결과를 Object Property로 반영하는 방법

```
ML 모델 출력 → 온톨로지 Object Property 매핑:

진동 분석 모델:
  prediction.anomaly_score  → Machine.vibrationAnomalyScore (Timeseries)
  prediction.rul_estimate   → Machine.estimatedRUL (double, 일 단위)

품질 예측 모델:
  prediction.defect_prob    → Lot.predictedDefectRate (double)
  prediction.risk_factors   → Lot.qualityRiskFactors (string JSON)

비전 검사 모델:
  prediction.defect_type    → Defect.defectType (string, 자동 분류)
  prediction.confidence     → Defect.classificationConfidence (double)
  prediction.image_ref      → Defect.imageRef (Media Reference)

이 매핑은 데이터 파이프라인(배치/스트리밍)을 통해 자동 갱신된다.
→ LLM은 ML 모델을 직접 호출할 필요 없이,
   Object Property에서 최신 예측 결과를 바로 읽을 수 있다.
```

### 4-4. LLM이 ML 모델 결과를 해석하고 Action과 연결하는 패턴

```
ML 예측: "M-14 RUL = 72시간 (신뢰도 85%)"
  ↓
LLM 해석 (온톨로지 컨텍스트 결합):
  - M-14의 현재 생산 스케줄: 긴급 주문 2건 (48시간 내 완료 필요)
  - 대체 설비: M-22 (건강 점수 78, 가용)
  - 정비 인력: 내일 주간 2명 가용
  - 부품 재고: 베어링 교체 부품 재고 있음
  ↓
LLM 판단:
  "RUL 72시간이면 긴급 주문 완료(48시간) 후 정비 가능.
   단, 신뢰도 85%이므로 20% 안전 마진 적용 시 실제 여유는 57.6시간.
   긴급 주문 완료 직후(48시간 후) 정비 시작이 최적."
  ↓
LLM Action 제안:
  1. proposeCreateWorkOrder(maintenance, M-14, scheduledAt=48h later, priority=high)
  2. proposeAssignOperator(정비 WO, 내일 주간 정비 기사)
  3. 모니터링 알림 설정: M-14 RUL이 48시간 미만으로 떨어지면 즉시 에스컬레이션
```

---

## 5. 컨텍스트 관리 전략

LLM의 매 쿼리에 제공되는 3종 컨텍스트를 체계적으로 관리한다. 이것이 팔란티어식 LLM 통합의 핵심이다: **Vector RAG + Graph RAG + Tool Calling + Operational Actions를 한 판에 묶는 구조**.

### 5-1. Ontology Context (구조화된 맥락)

#### 정의
온톨로지의 Object, Link, Property를 LLM에 전달하는 구조화된 정보. "이 설비가 어떤 라인에 속하고, 어떤 공구가 장착되어 있고, 최근 불량률이 얼마인가" 같은 사실적 정보.

#### 동적 선택 전략

```
사용자 질문 해석 → 관련 Object 식별 → Link 기반 확장 → 깊이 제한 적용

예시: "M-14 설비 상태가 어때?"

1단계 (직접 관련): Machine M-14의 전체 Property
2단계 (1-hop 확장):
  - M-14 → [mountedWith] → Tool[] (장착된 공구)
  - M-14 → [hasRecord] → MaintenanceRecord[] (최근 5건)
  - M-14 ← [processedOn] ← Lot[] (최근 3일 Lot)
  - M-14 ← [targetsMachine] ← WorkOrder[] (대기 중 작업 지시)
3단계 (2-hop, 조건부):
  - Lot → [observedIn] ← Defect[] (최근 불량, 불량률 높을 때만)
  - Tool → wearPercentage (마모율 70% 이상일 때만)

깊이 제한: 기본 2-hop, 복잡한 분석 시 3-hop까지 확장
크기 제한: 각 hop에서 최대 20개 Object (최신순/관련도순 정렬)
```

#### 컨텍스트 직렬화 형식

```json
{
  "ontology_context": {
    "focus_object": {
      "type": "Machine",
      "id": "M-14",
      "properties": {
        "status": "active",
        "vibrationLevel": 4.2,
        "lastMaintenanceDate": "2026-03-10T08:00:00Z",
        "healthScore": 34
      }
    },
    "linked_objects": [
      {
        "relation": "mountedWith",
        "objects": [
          {"type": "Tool", "id": "T-33", "wearPercentage": 82, "remainingLife": "2d"}
        ]
      },
      {
        "relation": "processedOn (recent 3 days)",
        "objects": [
          {"type": "Lot", "id": "L-0317", "defectRate": 0.08, "status": "completed"},
          {"type": "Lot", "id": "L-0318", "defectRate": 0.12, "status": "completed"}
        ]
      }
    ]
  }
}
```

### 5-2. Document Context (비정형 맥락)

#### 정의
SOP, 작업 지침서, 엔지니어 메모, 정비 매뉴얼, CAPA 보고서 등 비정형 문서에서 추출한 관련 정보.

#### 검색 전략: 벡터 임베딩 + 온톨로지 필터링 조합

```
전통적 RAG:
  사용자 질문 → 벡터 유사도 검색 → 상위 k개 문서 반환
  → 문제: "M-14 진동 이상"을 검색하면 M-08의 진동 매뉴얼도 반환될 수 있음

온톨로지 강화 RAG (본 설계):
  사용자 질문 → LLM이 관련 Object 식별 (M-14, Machine 타입)
  → 1차 필터: 문서에 태그된 Object Type/ID로 범위 축소
     (M-14 관련 문서, Machine 유형 일반 문서)
  → 2차 검색: 축소된 범위 내에서 벡터 유사도 검색
  → 결과: 높은 정밀도의 관련 문서만 반환

문서 태깅 구조:
  {
    "document_id": "SOP-CNC-VIBRATION-001",
    "title": "CNC 설비 진동 이상 시 대응 절차",
    "tags": {
      "object_types": ["Machine"],
      "machine_types": ["cnc"],
      "topics": ["vibration", "maintenance", "troubleshooting"]
    },
    "chunks": [
      {
        "content": "진동이 3.5mm/s 이상일 때 즉시 피드 속도를 30% 감소...",
        "embedding": [0.12, -0.34, ...],  // 벡터 임베딩
        "section": "4.2 긴급 대응"
      }
    ]
  }
```

#### 문서 유형별 활용

| 문서 유형 | 연결되는 에이전트 | 활용 방식 |
|----------|----------------|----------|
| SOP (표준 작업 절차) | 품질 진단, 예지정비 | "표준 절차에 따르면..." 근거 제시 |
| 작업 지침서 | 품질 진단 | 셋업 가이드 참조, 공차 기준 확인 |
| 정비 매뉴얼 | 예지정비 | 정비 절차, 필요 부품, 소요 시간 참조 |
| CAPA 보고서 | 품질 진단 | 과거 유사 사례의 시정/예방 조치 참조 |
| 엔지니어 메모 | 전체 | 최근 변경 사항, 특이사항, 경험적 지식 |
| 설비 스펙 시트 | 예지정비, 생산 최적화 | 설비 성능 한계, 운영 조건 범위 |

### 5-3. Function-backed Context (계산 기반 맥락)

#### 정의
사용자 정의 검색, 혼합 검색, 특수 계산의 결과로 생성되는 동적 컨텍스트. 정적으로 저장된 데이터가 아니라, 쿼리 시점에 Function이 실행되어 생성된다.

#### 유형별 예시

**실시간 센서 데이터 요약:**
```
Function: summarizeSensorTrend(machineId, sensorType, period)
출력:
  {
    "sensor": "vibration",
    "period": "last_24h",
    "summary": {
      "mean": 3.8, "max": 4.2, "min": 2.1, "std": 0.6,
      "trend": "increasing", "changePoint": "2026-03-15T22:30:00Z",
      "anomalyPeriods": [
        {"start": "2026-03-16T02:00", "end": "2026-03-16T03:30", "maxValue": 4.2}
      ]
    }
  }
→ LLM에게 "지난 24시간 진동 추세: 상승 중, 어젯밤 22:30부터 변곡점, 새벽 2~3시 최대 4.2mm/s"
```

**유사 패턴 검색 (혼합 검색):**
```
Function: findSimilarIncidents(defectPattern, dateRange)
  - 벡터 유사도: 불량 이미지/센서 패턴 유사도
  - 온톨로지 필터: 동일 설비 유형, 동일 제품군
  - 결과: 과거 유사 사례 목록 + 각 사례의 근본 원인 + 해결 조치
→ LLM에게 "과거 유사 사례 3건: (1) 2025-11 M-08 Tool 마모 → 교체로 해결, (2) ..."
```

**교차 분석 계산:**
```
Function: correlateDefectWithConditions(defectId)
  - 불량 발생 시점의 모든 공정 조건 (레시피 파라미터 실측값)
  - 동일 레시피의 정상 Lot 공정 조건 통계
  - 편차 분석: 어떤 파라미터가 얼마나 벗어났는가
→ LLM에게 "불량 발생 시 피드 속도가 정상 대비 +18% 편차, 온도는 정상 범위"
```

---

## 6. Graph RAG vs. 운영형 온톨로지 -- 우리 전략의 위치

### 6-1. 일반 Graph RAG의 동작 원리

```
일반 Graph RAG 파이프라인:
  문서/데이터 → 엔티티 추출 → 관계 추출 → 지식 그래프 구축
  사용자 질문 → 엔티티 인식 → 그래프 검색 → 관련 서브그래프 추출
  → LLM 컨텍스트에 주입 → 응답 생성

핵심 목적: 지식 검색의 정확도와 맥락성을 높이는 것
제한: 읽기 전용. 실세계를 변경하는 능력이 없다.
```

### 6-2. 우리 전략: 운영 실행 최적화

```
운영형 온톨로지 + AI 파이프라인:
  원천 시스템 → 객체/관계 정의(온톨로지) → 실시간 동기화
  사용자 질문 → LLM 해석 → Data Tool(읽기) + Logic Tool(계산) + Document 검색
  → 종합 판단 → Action Tool(쓰기) 제안 → 정책 검증 → 실행
  → 상태 변경 → 감사 로그 → 다음 쿼리에 반영 (닫힌 루프)

핵심 목적: 의사결정과 운영 실행을 자동화하는 것
Graph RAG의 기능(지식 검색)을 하위 구성요소로 포함한다.
```

### 6-3. 비교 표

| 특성 | 일반 Graph RAG | 우리 전략 (운영형 온톨로지 + AI) |
|------|-------------|-------------------------------|
| **목적** | 지식 검색 + 응답 생성 | 운영 의사결정 + 실행 자동화 |
| **읽기** | 그래프 검색 → LLM 컨텍스트 | Data Tools + Ontology Context |
| **쓰기 능력** | 없음 | Action Type을 통한 통제된 상태 변경 |
| **비즈니스 로직** | 없음 (LLM이 추론) | Function으로 온톨로지에 내장 |
| **거버넌스** | 없음 또는 최소 | 권한 + 감사 + 유효성 검사 + 승인 워크플로 |
| **실시간성** | 지식 그래프 갱신 주기에 의존 | Object Property 실시간 동기화 |
| **데이터 원천** | 문서/텍스트 중심 | 운영 시스템(ERP, MES, SCADA) + 문서 |
| **감사 추적** | 없음 | 모든 조회/실행의 완전한 감사 로그 |
| **보안 모델** | 문서 수준 접근 제어 | Object/Property/Action 수준 세분화 |
| **닫힌 루프** | 미지원 | Action 실행 → 상태 반영 → 다음 판단에 재사용 |

### 6-4. 우리의 전략적 포지션

> **"우리는 Graph RAG을 하위 구성요소로 포함하되, 그 위에 Function + Action + Security를 올린다."**

```
┌──────────────────────────────────────────────────┐
│        운영형 온톨로지 + AI 시스템 (우리 전략)      │
│                                                    │
│  ┌──────────────────────────────────────────┐     │
│  │  Action Layer: 통제된 상태 변경 + 감사      │     │
│  └──────────────────────┬───────────────────┘     │
│                         │                          │
│  ┌──────────────────────┴───────────────────┐     │
│  │  Function Layer: 도메인 판단 로직           │     │
│  └──────────────────────┬───────────────────┘     │
│                         │                          │
│  ┌──────────────────────┴───────────────────┐     │
│  │  Security Layer: 행/열/Action 접근 제어     │     │
│  └──────────────────────┬───────────────────┘     │
│                         │                          │
│  ┌──────────────────────┴───────────────────┐     │
│  │  Knowledge Layer: Graph RAG 포함            │     │
│  │  (Ontology Context + Document Context       │     │
│  │   + Vector Search + Graph Traversal)        │     │
│  └─────────────────────────────────────────────┘     │
└──────────────────────────────────────────────────────┘
```

Graph RAG는 우리 시스템의 "읽기 최적화" 구성요소다. 하지만 제조 현장의 가치는 "읽는 것"이 아니라 "실행하는 것"에서 나온다. 01-problem-diagnosis.md에서 진단한 의사결정 병목(2시간~3일)을 해결하려면, 읽기를 넘어 판단(Function)과 실행(Action)까지 닫힌 루프로 연결해야 한다.

---

## 7. 보안과 AI 거버넌스

### 7-1. AI 에이전트에 적용되는 보안 정책

핵심 원칙: **AI 에이전트는 인간 사용자와 동일한 보안 정책을 적용받는다.** 에이전트에게 별도의 "슈퍼 권한"을 부여하지 않는다.

```
대리 실행(Proxy Execution) 모델:

  사용자 (Operator, OPR-Kim)
    → 에이전트에게 질문
    → 에이전트는 OPR-Kim의 권한으로 Tool 호출
    → OPR-Kim이 볼 수 없는 데이터는 에이전트도 볼 수 없음
    → OPR-Kim이 실행할 수 없는 Action은 에이전트도 실행 불가

  감사 로그 기록:
    {
      "actor": "user:OPR-Kim",
      "via": "agent:quality-diagnosis",
      "tool": "traceDefectCause",
      "params": {"defectId": "D-2024-0891"},
      "timestamp": "2026-03-16T14:42:00Z",
      "result_summary": "5 objects returned",
      "data_access_scope": "PLT-SUWON only"
    }
```

### 7-2. LLM이 접근할 수 있는 도구/데이터 범위 제한

```
Tool 허용 목록 (Allowlist) 방식:

  품질 진단 에이전트:
    allowed_tools: [
      "searchDefects", "getLotDetail", "traceDefectCause",
      "analyzeDefectContribution", "evaluateMachineHealth",
      "proposeEscalateDefect", "proposeQuarantineLot",
      "proposeCreateWorkOrder"  // inspection, rework만
    ]
    forbidden_tools: [
      "proposeChangeRecipe",  // 품질 에이전트는 레시피 변경 제안 불가
      "*Supplier*"  // 공급사 직접 조회 불가 (품질 영향만 간접 확인)
    ]

  → 에이전트가 허용 목록 외 Tool을 호출하려 하면 정책 게이트가 차단
  → LLM의 프롬프트 인젝션 공격으로도 허용 범위를 넘길 수 없음 (코드 레벨 차단)
```

### 7-3. AI 의사결정의 설명 가능성 (Explainability)

모든 AI 에이전트 응답에는 "근거 체인(Evidence Chain)"이 첨부된다.

```
근거 체인 구조:

  에이전트 응답: "M-14를 오늘 야간에 정비할 것을 권장합니다."

  근거 체인:
  1. [Data] searchMachines(status=active) → M-14 건강 점수 34점 (하위 3%)
  2. [Logic] evaluateMachineHealth(M-14) →
     - 정비 주기 초과: 12일 (기준 10일) → 주기 점수 60
     - 센서 이상: 진동 4.2mm/s (기준 3.5) → 센서 점수 35
     - 불량률: 최근 7일 8.2% (기준 3%) → 불량 점수 45
     - 공구 마모: T-33 82% (기준 85%) → 공구 점수 18
  3. [Logic] estimateDowntimeImpact(M-14, 4h) →
     - 야간 정비 시 생산 손실: 80개 (주간 대비 60% 감소)
  4. [Data] getShiftInfo(tonight) → 야간 정비 기사 1명 가용
  5. [Document] "CNC 진동 이상 대응 SOP" 섹션 4.2:
     "진동 4.0mm/s 이상 지속 시 24시간 내 정비 권장"

  결론 도출 과정:
  - 건강 점수 34 < 임계값 40 → 긴급 정비 대상
  - 야간 정비 시 생산 영향 최소화 (낮 생산 우선)
  - 인력 가용 확인됨 → 실행 가능
  - SOP 근거 확보됨 → 표준 절차 준수
```

### 7-4. AI 에이전트의 자율성 수준 설정

| 자율성 수준 | 설명 | 적용 상황 | 예시 |
|------------|------|----------|------|
| **Level 0: 정보 제공** | 질문에 답하되 Action 제안 없음 | 일반 조회, 현황 파악 | "현재 라인 A의 OEE는 82%입니다" |
| **Level 1: 조치 제안** | Action을 제안하되 실행하지 않음 | 대부분의 운영 상황 | "레시피 롤백을 권장합니다. [승인] 버튼을 눌러주세요" |
| **Level 2: 승인 후 실행** | 사용자 승인 시 즉시 실행 (Human-in-the-loop) | 표준적 정비, 검사 | "정비 작업 지시를 생성했습니다. 확인해주세요" |
| **Level 3: 자동 실행 + 사후 통보** | 정책 내에서 자동 실행, 사후 알림 | 저위험 반복 작업 | "Tool T-33 마모율 90% 도달. 교체 작업 지시 자동 생성됨" |
| **Level 4: 완전 자율** | 판단~실행~모니터링 전 과정 자동 | 현 단계에서는 미적용 | 향후 충분한 신뢰도 확보 후 검토 |

```
자율성 수준 설정 매트릭스:

                        위험도 낮음          위험도 높음
                    ┌─────────────────┬─────────────────┐
  빈도 높음 (일상)   │  Level 3         │  Level 2         │
  (센서 모니터링,    │  자동 실행        │  승인 후 실행     │
   일상 보고)       │  + 사후 통보      │                  │
                    ├─────────────────┼─────────────────┤
  빈도 낮음 (비정상) │  Level 2         │  Level 1         │
  (불량 급증,       │  승인 후 실행     │  조치 제안만      │
   설비 고장)       │                  │  (인간 판단 필수) │
                    └─────────────────┴─────────────────┘

예외: 안전 관련 Action (Lot 격리, Emergency 정비 요청)은
      위험도와 무관하게 Level 2 이상 (즉시 실행 허용)
      → 품질 안전은 지연보다 과잉 대응이 나음
```

---

## 8. 기술 스택 옵션

자체 구축 시 고려할 OSS 기술 스택을 Palantir의 각 구성 요소에 대응시켜 정리한다.

### 8-1. 기술 스택 매핑

| Palantir 구성 요소 | 역할 | OSS 대안 | 비고 |
|-------------------|------|---------|------|
| **Foundry Ontology** | Object/Link/Property 관리 | **Neo4j** (Property Graph) + **PostgreSQL** (관계형 백업) | Neo4j가 그래프 탐색에 최적, PostgreSQL이 트랜잭션/집계에 적합. 하이브리드 구성 권장 |
| **Foundry Pipeline** | 데이터 통합/ETL | **Apache Kafka** (스트리밍) + **dbt** (배치 변환) | 실시간 센서 데이터는 Kafka, 배치 집계는 dbt |
| **AIP Logic (LLM)** | LLM 추론 엔진 | **Claude** / **GPT-4** / **Llama 3** | 클라우드: Claude API / GPT-4 API. 온프레미스: Llama 3 (vLLM 서빙) |
| **AIP Agent Studio** | 에이전트 프레임워크 | **LangGraph** (권장) / CrewAI | LangGraph: 그래프 기반 상태 관리, human-in-the-loop 네이티브 지원, 프로덕션 검증됨 |
| **AIP Tools** | Tool 정의/실행 | **MCP (Model Context Protocol)** + **FastAPI** | MCP: 2026년 에이전트 Tool 표준. FastAPI: Tool 서버 구현 |
| **Document RAG** | 비정형 문서 검색 | **Pinecone** / **Weaviate** / **Chroma** | 프로덕션: Pinecone/Weaviate. PoC: Chroma (로컬, 무료) |
| **Ontology Security** | 행/열/Action 접근 제어 | **OPA (Open Policy Agent)** + **JWT** | OPA: 정책 엔진. JWT claim 기반 사용자 권한 전달 |
| **OSDK** | 외부 앱 API | **FastAPI** + **Pydantic** 모델 자동 생성 | Pydantic 모델이 Object Type 스키마와 1:1 매핑 |
| **Workshop** | 대시보드/UI | **Streamlit** (PoC) / **Next.js** (프로덕션) | Streamlit: 빠른 프로토타이핑. Next.js: 프로덕션 대시보드 |
| **AIP Evals** | AI 성능 평가 | **LangSmith** / **Weights & Biases** | LangSmith: LangGraph와 네이티브 통합. W&B: ML 모델 실험 관리 |
| **Apollo** | 배포 관제 | **ArgoCD** + **Kubernetes** | GitOps 기반 배포 자동화 |

### 8-2. 권장 아키텍처 구성도

```
┌─────────────────────────────────────────────────────────────────┐
│                        클라이언트 계층                            │
│  Next.js 대시보드 / 모바일 앱 / Streamlit PoC                    │
└───────────────────────────┬─────────────────────────────────────┘
                            │ REST/WebSocket
┌───────────────────────────┴─────────────────────────────────────┐
│                        API Gateway (FastAPI)                     │
│  인증(JWT) + 라우팅 + Rate Limiting                              │
└───────────┬──────────────┬──────────────┬───────────────────────┘
            │              │              │
┌───────────┴──┐  ┌────────┴───────┐  ┌──┴──────────────────┐
│ Agent Service│  │ Tool Service   │  │ Auth Service        │
│ (LangGraph)  │  │ (FastAPI+MCP)  │  │ (OPA + JWT)         │
│              │  │                │  │                     │
│ 3개 전문     │  │ Data Tools     │  │ Policy Engine       │
│ 에이전트     │  │ Logic Tools    │  │ 권한 매트릭스       │
│              │  │ Action Tools   │  │ 감사 로그 수집      │
└───────┬──────┘  └──────┬─────────┘  └─────────────────────┘
        │                │
        │     ┌──────────┴──────────────────────┐
        │     │          Ontology Service        │
        │     │  ┌──────────┐  ┌──────────────┐ │
        │     │  │ Neo4j    │  │ PostgreSQL   │ │
        │     │  │ (Graph)  │  │ (Relational) │ │
        │     │  └──────────┘  └──────────────┘ │
        │     └─────────────────────────────────┘
        │
┌───────┴──────────────────────────────────────────┐
│              Context Service                      │
│  ┌──────────────┐  ┌──────────────┐              │
│  │ Vector DB    │  │ Document     │              │
│  │(Weaviate/    │  │ Store        │              │
│  │ Pinecone)    │  │ (S3/MinIO)   │              │
│  └──────────────┘  └──────────────┘              │
└──────────────────────────────────────────────────┘
        │
┌───────┴──────────────────────────────────────────┐
│           Data Integration (Kafka + dbt)          │
│  ERP ─┐                                          │
│  MES ─┤  Kafka (실시간)                           │
│  SCADA┤  ──────────→ Ontology Service             │
│  QMS ─┤                                          │
│  CMMS─┘  dbt (배치)                               │
│          ──────────→ PostgreSQL + Neo4j            │
└──────────────────────────────────────────────────┘
```

### 8-3. MCP (Model Context Protocol) 적용 전략

2025년 Anthropic이 발표하고 2026년 현재 에이전트 Tool 표준으로 자리잡은 MCP를 본 시스템의 Tool 통신 프로토콜로 채택한다.

```
MCP 적용 구조:

  LangGraph Agent (MCP Client)
    ↕ JSON-RPC over HTTP/SSE
  MCP Server: Ontology Tools
    ├── Data Tools (searchMachines, traceDefectCause, ...)
    ├── Logic Tools (evaluateMachineHealth, ...)
    └── Action Tools (proposeEscalateDefect, ...)

MCP 도입의 이점:
1. Tool 표준화: 에이전트 프레임워크 교체 시에도 Tool 서버 재사용 가능
2. 동적 Tool 발견: 에이전트가 사용 가능한 Tool을 런타임에 자동 탐색
3. 보안 격리: MCP Server가 독립 프로세스로 실행, Tool 실행 환경 격리
4. 확장성: 새 Tool 추가 시 MCP Server에만 등록하면 모든 에이전트가 즉시 사용
```

### 8-4. 단계별 도입 로드맵

```
Phase 1 (PoC, 4~6주):
  - LLM: Claude API (클라우드)
  - Agent: LangGraph (단일 에이전트, 품질 진단)
  - DB: PostgreSQL + Chroma (벡터)
  - UI: Streamlit
  - 목표: "불량 원인 추적" 시나리오 1개 동작 검증

Phase 2 (MVP, 8~12주):
  - 3개 전문 에이전트 구현
  - Neo4j 도입 (그래프 탐색 최적화)
  - MCP 기반 Tool 서버 분리
  - OPA 기반 권한 관리
  - Action 실행 + 감사 로그
  - 목표: 현장 파일럿 (1개 라인)

Phase 3 (Scale, 12~20주):
  - Weaviate/Pinecone (프로덕션 벡터 DB)
  - Kafka 기반 실시간 데이터 파이프라인
  - ML 모델 통합 (진동 분석, RUL 예측)
  - Next.js 프로덕션 대시보드
  - 목표: 전체 공장 확대

Phase 4 (Optimize, 지속):
  - LangSmith 기반 에이전트 성능 모니터링
  - 에이전트 자율성 수준 점진적 상향
  - 온프레미스 LLM 검토 (보안 요건에 따라)
  - 멀티 공장 확장
```

---

## 참고 자료

**팔란티어 공식 문서**
- [AIP Architecture](https://www.palantir.com/docs/foundry/architecture-center/aip-architecture)
- [Agent Studio Tools](https://palantir.com/docs/foundry/agent-studio/tools/)
- [AI Ethics & Governance](https://palantir.com/docs/foundry/aip/ethics-governance/)

**에이전트 아키텍처 트렌드 (2025-2026)**
- [5 Key Trends Shaping Agentic Development in 2026 - The New Stack](https://thenewstack.io/5-key-trends-shaping-agentic-development-in-2026/)
- [7 Agentic AI Trends to Watch in 2026 - MachineLearningMastery.com](https://machinelearningmastery.com/7-agentic-ai-trends-to-watch-in-2026/)
- [State of AI Agents - LangChain](https://www.langchain.com/state-of-agent-engineering)
- [Agentic AI Architectures, Taxonomies - arXiv](https://arxiv.org/html/2601.12560v1)

**MCP (Model Context Protocol)**
- [The 2026 MCP Roadmap](http://blog.modelcontextprotocol.io/posts/2026-mcp-roadmap/)
- [Introducing the Model Context Protocol - Anthropic](https://www.anthropic.com/news/model-context-protocol)
- [MCP Architecture, Components & Workflow - Kubiya](https://www.kubiya.ai/blog/model-context-protocol-mcp-architecture-components-and-workflow)

**에이전트 프레임워크 비교**
- [The 2026 AI Agent Framework Decision Guide: LangGraph vs CrewAI vs Pydantic AI](https://dev.to/linou518/the-2026-ai-agent-framework-decision-guide-langgraph-vs-crewai-vs-pydantic-ai-b2h)
- [LangGraph vs CrewAI: Multi-Agent Performance and Cost in Production 2026](https://markaicode.com/vs/langgraph-vs-crewai-multi-agent-production/)

**Graph RAG 및 지식 그래프**
- [Knowledge Graphs vs RAG: When to Use Each for AI in 2026 - Atlan](https://atlan.com/know/knowledge-graphs-vs-rag-for-ai/)
- [GraphRAG & Knowledge Graphs: Making Your Data AI-Ready for 2026 - Fluree](https://flur.ee/fluree-blog/graphrag-knowledge-graphs-making-your-data-ai-ready-for-2026/)
- [6 Data Predictions for 2026 - VentureBeat](https://venturebeat.com/data/six-data-shifts-that-will-shape-enterprise-ai-in-2026)

**제조 AI 관련**
- [Siemens - The True Cost of an Hour's Downtime (2024)](https://blog.siemens.com/2024/07/the-true-cost-of-an-hours-downtime-an-industry-analysis/)
- [Unlocking the Potential of Predictive Maintenance (Springer, 2024)](https://link.springer.com/article/10.1007/s41471-024-00204-3)
