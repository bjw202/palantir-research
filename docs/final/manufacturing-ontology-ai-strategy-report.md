# 제조 온톨로지 + AI 전략 통합 보고서

## Manufacturing Ontology-AI Strategy — Integrated Report

> **발행일:** 2026-03-16
> **발행 주체:** Manufacturing Sector AI-Driven Research Center
> **보고서 유형:** 경영 전략 + 기술 아키텍처 통합본
> **대상 독자:** CTO/COO/CEO + 기술팀 리더 + 현장 엔지니어

---

## 제0장. 경영진 요약 (Executive Summary)

### 한 줄 요약

> **흩어진 데이터를 "현실 세계의 객체와 관계"로 재구성하고, 그 위에 AI를 올려 "분석"을 넘어 "실행"까지 자동화하는 온톨로지 기반 운영형 의사결정 시스템을 구축한다.**

### 핵심 수치

| 항목 | 수치 |
|------|------|
| **현재 연간 비효율 비용** | ₩290~360억원/년 (매출 1,000억원 기준, 매출의 29~36%) |
| **총 투자 규모** | ₩23~43억원 (18개월, 하이브리드 옵션) |
| **연간 기대 절감** | ₩108~188억원/년 |
| **투자 회수 기간** | 6~8개월 (Phase 1 완료 시점) |
| **3년 ROI (리스크 조정, 보수적 60%)** | 약 350% |
| **3년 ROI (기본 80%)** | 약 503% |

### 3대 Pain Point → 해결 메커니즘 → 기대 효과

| Pain Point | 온톨로지 해결 메커니즘 | 기대 효과 |
|------------|----------------------|-----------|
| 데이터 사일로 (7개 시스템 단절) | 통합 객체 세계 — 같은 설비를 하나의 Object로 정의 | 시스템 간 데이터 교차 분석 시간 90% 단축 |
| 품질 추적 불가 (원인 파악 2~3일) | 관계 그래프 탐색 — Lot→Machine→Tool→Recipe→Material 체인 | 원인 추적 5시간 → 1분 (99.7% 단축) |
| 의사결정 병목 (야간 대응 7시간+) | AI + Action 자동화 — 감지→분석→제안→실행 닫힌 루프 | 대응 시간 96% 단축, 비계획 다운타임 75% 감소 |

### 전략 권장: 하이브리드 접근

Phase 0~1은 OSS 기반으로 PoC + 핵심 온톨로지 구축, Phase 2~3에서 필요 시 상용 플랫폼 도입을 검토한다. Phase 0에 ₩3~5억원, Phase 0+1 누적 ₩10~20억 수준에서 효과를 검증한 후 확대 투자를 결정할 수 있어 리스크가 관리 가능하다. 효과 미입증 시 Phase 1 완료 시점에서 중단 가능.

---

## 제1장. 현황 진단 — 왜 지금 바꿔야 하는가

> 상세: [01-problem-diagnosis.md](../research/01-problem-diagnosis.md)

### 1.1 스마트팩토리의 구조적 문제

제조 현장은 단일 시스템이 아니다. ERP, MES, SCADA, QMS, WMS, CMMS, PLM — 10~20년에 걸쳐 도입된 이기종 시스템들이 **각자의 데이터를 각자의 스키마로, 각자의 프로토콜로** 소유하고 있다.

```
문제의 본질:
  같은 설비가 ERP에서는 "EQ-001-A", MES에서는 "MACHINE_A_LINE1", CMMS에서는 "ASSET-00142"
  → 세 개의 이름, 하나의 설비, 연결 불가
```

### 1.2 품질 추적의 현실

불량이 발생하면 엔지니어가 4~5개 시스템에 각각 로그인하여 엑셀로 데이터를 병합한다.

- **이상적 추적 경로:** Lot → Machine → Tool → Recipe → Material → Supplier
- **현실:** 각 연결 지점이 서로 다른 시스템에 있어 수작업 매핑 필요
- **소요 시간:** 최소 6시간 ~ 최대 3일
- **그 사이:** 동일 불량이 계속 생산됨

**품질 비용(COPQ):** 제조 기업 매출의 **15~20%** (연 매출 1,000억원 기준 ₩150~200억원) — IISE, Parsec

### 1.3 의사결정 병목

```
이상 감지 → 인지(5분~2시간) → 정보 수집(30분~4시간) → 원인 분석(1시간~1일)
→ 조치 결정(30분~4시간) → 실행(15분~2시간)
합산: 최소 2시간 ~ 최대 3일+
```

야간/주말에는 전문 엔지니어 부재로 의사결정 공백이 발생한다. **비계획 다운타임 비용:** 매출의 **11%**, 전 세계 1.4조 달러 규모 — Siemens 2024

### 1.4 기존 접근이 실패하는 이유

| 접근 | 하는 일 | 못 하는 일 |
|------|---------|-----------|
| BI 대시보드 | "불량률 3.2%" 시각화 | "왜?"에 답하지 못함 |
| ML 모델 | "고장 확률 73%" 예측 | 실행으로 연결 안 됨 (성공률 11%) |
| 데이터 레이크 | 데이터 저장 | 의미와 관계 없음 (80% 실패) |
| RPA | 반복 작업 자동화 | 판단 불가 |

> **핵심:** 네 가지를 모두 도입해도 "데이터를 이해하고, 관계를 추론하고, 맥락에 맞는 판단을 내리고, 실행하는" 통합 흐름은 만들어지지 않는다.

### 1.5 Do Nothing 시나리오

- 매년 ₩290~360억원의 비효율 비용 누적
- 글로벌 제조 AI 시장 연 35.3% 성장, 77% 기업이 이미 AI 도입 — 2~3년 내 원가 경쟁력 상실
- 숙련 엔지니어 퇴직 시 품질 추적 역량 소멸
- EU AI Act, 공급망 실사법 등 규제 대응 불가

---

## 제2장. 전략 비전 — 온톨로지 기반 운영형 의사결정 시스템

### 2.1 팔란티어 벤치마크에서 추출한 핵심 통찰

Palantir는 수십 개 고객사 배포 경험에서 문제의 프레임을 바꿨다.

```
기존 프레임:  "데이터를 어떻게 저장/이동/분석할 것인가?"
새 프레임:    "어떤 결정을 어떻게 지원할 것인가?"
```

이 관점 전환에서 나온 설계가 **온톨로지** — 데이터를 현실 세계의 객체(Object)와 관계(Link)로 재구성하고, 그 위에 판단(Function)과 실행(Action)과 보안(Security)을 올리는 구조.

### 2.2 목표 아키텍처

```
┌─────────────────────────────────────────────────────────────────┐
│          사용자 인터페이스 (자연어 채팅 / 대시보드 / 모바일)         │
├─────────────────────────────────────────────────────────────────┤
│          AI 중간 계층 (AIP Logic 패턴)                             │
│   ┌────────────┐  ┌────────────┐  ┌────────────┐               │
│   │ 품질 진단   │  │ 예지정비    │  │ 생산 최적화  │  ← AI 에이전트 │
│   │ Agent      │  │ Agent      │  │ Agent      │               │
│   └─────┬──────┘  └─────┬──────┘  └─────┬──────┘               │
│         └───────────────┼───────────────┘                       │
│    Data Tools ──── Logic Tools ──── Action Tools                │
│                   정책 게이트 (보안 + 권한 + 감사)                  │
├─────────────────────────────────────────────────────────────────┤
│          온톨로지 계층                                             │
│   15 Object Types / 14 Link Types / 7 Action Types / 7 Functions│
├─────────────────────────────────────────────────────────────────┤
│          데이터 통합 계층                                          │
│   ERP + MES + SCADA + QMS + WMS + CMMS + PLM + IoT Stream      │
└─────────────────────────────────────────────────────────────────┘
```

### 2.3 핵심 원칙

1. **결정 중심 설계:** "어떤 데이터를 저장할까"가 아니라 "어떤 결정을 지원할까"에서 출발
2. **Golden Record:** 하나의 정의, 여러 소비자 — 모든 팀이 동일한 Object Type을 참조
3. **모든 쓰기는 Action을 통과:** 유효성 검사 → 권한 확인 → 실행 → 감사 로그
4. **AI에게 초능력을 주지 않는다:** LLM은 Tool 사용을 "요청"만 하고, 실행은 사용자 권한 범위 내에서 수행

---

## 제3장. 온톨로지 아키텍처 설계

> 상세: [02-ontology-architecture.md](../research/02-ontology-architecture.md)

### 3.1 Object Type (15개) — 세계를 객체로 정의

| Object Type | 정의 | 핵심 Property | 원천 시스템 |
|-------------|------|--------------|-----------|
| **Plant** | 공장/사업장 | plantId, name, location, timezone | ERP |
| **ProductionLine** | 생산 라인 | lineId, name, status, capacity | MES |
| **Machine** | 설비/장비 | machineId, status, currentVibration(시계열), healthScore | MES+SCADA+CMMS |
| **Tool** | 공구/지그 | toolId, cumulativeUsageHours, wearLevel | TMS/CMMS |
| **Recipe** | 공정 조건 세트 | recipeId, version, feedRate, spinSpeed, temperature | PLM+MES |
| **Material** | 원자재/부자재 | materialId, grade, supplierLotId, expirationDate | WMS |
| **Lot** | 생산 단위 | lotId, quantity, startTime, endTime, status | MES |
| **Measurement** | 센서/검사 측정값 | measurementId, value, timestamp, sensorType | SCADA+QMS |
| **Defect** | 불량/결함 | defectId, type, severity, detectedAt, rootCause | QMS |
| **WorkOrder** | 작업 지시 | workOrderId, type, priority, status, scheduledAt | MES+CMMS |
| **Operator** | 작업자 | operatorId, name, certifications, shiftId | MES+HR |
| **Supplier** | 공급사 | supplierId, name, qualityScore, leadTime | ERP+QMS |
| **Shift** | 교대/근무조 | shiftId, type, startTime, endTime | MES |
| **MaintenanceRecord** | 정비 이력 | recordId, type, duration, partsUsed, technicianId | CMMS |
| **Product** | 최종 제품 | productId, name, bomVersion, customer | ERP+PLM |

### 3.2 Link Type (14개) — 핵심 관계 체인

**품질 역추적 체인:**
```
Defect →[observed_in]→ Lot →[processed_on]→ Machine →[mounted_with]→ Tool
                         │                      └──→[ran_recipe]→ Recipe
                         └──[used_material]→ Material →[supplied_by]→ Supplier
```

**정비 영향 체인:**
```
Machine →[has_maintenance]→ MaintenanceRecord →[triggered_by]→ WorkOrder
```

**생산 계보 체인:**
```
Lot →[operated_by]→ Operator →[belongs_to]→ Shift
Lot →[fulfills]→ WorkOrder →[produces]→ Product
```

### 3.3 Action Type (7개) — 통제된 변경

| Action Type | 목적 | 실행 모드 | 승인 조건 |
|------------|------|----------|----------|
| **EscalateDefect** | 불량 에스컬레이션 | 즉시/검토 | severity=critical → 관리자 승인 |
| **CreateWorkOrder** | 작업지시 생성 | 즉시 | 긴급 등급 시 자동 생성 가능 |
| **QuarantineLot** | Lot 격리 | 검토 후 | 품질팀 승인 필수 |
| **ChangeRecipe** | 레시피 변경 | 검토 후 | 공정팀장 + 품질팀 합의 필수 |
| **RequestMaintenance** | 정비 요청 | 즉시/자동 | 건강 점수 임계치 이하 시 자동 |
| **AssignOperator** | 작업자 배정 | 즉시 | 자격 인증 자동 검증 |
| **UpdateMachineStatus** | 설비 상태 변경 | 즉시 | 권한 있는 역할만 실행 |

### 3.4 Function (7개) — 도메인 판단 로직

| Function | 목적 | 주요 입력 | 출력 |
|----------|------|----------|------|
| calculateMachineHealthScore | 설비 건강 점수 | 센서, 정비 이력, 불량률 | 0~100 점수 |
| calculateDefectContribution | 불량 기여 요인 | Defect 관련 전체 객체 | 요인별 기여도 % |
| assessProcessRisk | 공정 리스크 평가 | Recipe, Machine 상태 | 리스크 등급 |
| predictToolWear | 공구 마모 예측 | 사용 시간, 가공 조건 | 잔여 수명 시간 |
| calculateLotSensitivity | Lot 민감도 계산 | Recipe, Material 특성 | 민감도 점수 |
| evaluateSupplierQuality | 공급사 품질 평가 | 입고 검사, 불량 이력 | 품질 등급 |
| estimateDowntimeImpact | 다운타임 영향도 | WorkOrder, 생산 계획 | 비용·납기 영향 |

### 3.5 보안 모델

- **행 수준:** 공장별 접근 제어 (Plant.plantId 기반)
- **열 수준:** 비용/급여 정보 접근 제한 (HR/재무 역할만)
- **Action 권한:** 역할별 실행 가능 Action 매트릭스
- **AI 동일 적용:** AI 에이전트도 인간과 동일한 보안 정책 적용 → 무단 접근 구조적 불가

---

## 제4장. AI 통합 전략

> 상세: [03-ai-integration-strategy.md](../research/03-ai-integration-strategy.md)

### 4.1 Tool 구조 (AIP Logic 패턴)

LLM이 온톨로지와 상호작용하는 도구는 3종으로 분류된다.

| Tool 유형 | 역할 | 예시 | 수량 |
|-----------|------|------|------|
| **Data Tools** (읽기) | 객체 검색, 관계 탐색, 집계 | searchMachines, traceDefectCause, getDefectRateByMachine | 16종 |
| **Logic Tools** (계산) | Function 실행, 판단 | calculateMachineHealthScore, assessProcessRisk | 7종 |
| **Action Tools** (실행) | 상태 변경, 워크플로 | escalateDefect, quarantineLot, createWorkOrder | 7종 |

**안전장치:** LLM은 Tool 사용을 "요청"만 한다. 정책 게이트(Policy Gate)가 보안 검증 + 권한 확인 + 감사 로그를 수행한 후 실행한다.

### 4.2 제조 현장용 AI 에이전트 3개

**품질 진단 에이전트 (Quality Diagnosis Agent)**
```
역할: 불량 근본 원인 추적 + 영향 범위 산정 + 조치 제안
대화 예시:
  사용자: "오늘 CNC-14에서 Edge Burr 불량이 급증했어. 원인이 뭘까?"
  에이전트: [Defect 조회] → [Lot-Machine-Tool-Recipe-Material 관계 탐색]
           → [calculateDefectContribution 실행] → [SOP 문서 참조]
           → "1순위 원인: 3/14 레시피 변경(피드 속도 +12%), 기여도 62%
              2순위: Tool T-33 마모(누적 847h/권장 1000h), 기여도 28%
              권장 Action: [레시피 롤백] + [Tool 교체 스케줄링]"
```

**예지정비 에이전트 (Predictive Maintenance Agent)**
```
역할: 설비 건강 모니터링 + 정비 시점 예측 + 생산 일정 영향 분석
대화 예시:
  사용자: "이번 주 정비가 필요한 설비 알려줘"
  에이전트: [calculateMachineHealthScore 전체 실행] → [predictToolWear]
           → [estimateDowntimeImpact로 생산 영향 분석]
           → "CNC-14 건강 점수 62/100, 72시간 내 정비 권장
              최적 시점: 목요일 교대 시간(06:00~08:00), 생산 영향 최소
              대체 설비: CNC-22 (동일 공정 가능, 현재 유휴)"
```

**생산 최적화 에이전트 (Production Optimization Agent)**
```
역할: 생산 일정 최적화 + 설비 할당 + 에너지 효율 + 병목 분석
대화 예시:
  사용자: "현대모비스 긴급 주문이 D+3인데, 현재 생산 계획으로 맞출 수 있어?"
  에이전트: [WorkOrder 조회] → [Machine 가용성 확인] → [estimateDowntimeImpact]
           → "현재 계획으로는 4시간 초과. 제안: CNC-22를 09:00부터 병렬 투입 시
              D+2.5일에 완료 가능. [AssignOperator] + [CreateWorkOrder] 실행할까요?"
```

### 4.3 기존 ML 모델 통합

현장에 이미 도입된 ML 모델(예측정비, 품질 예측 등)을 온톨로지 Function으로 래핑하여 통합한다.

```
기존: ML 모델이 예측값을 대시보드에 표시 → 사람이 보고 판단 (Last Mile 공백)
통합 후: ML 예측값 → Function 결과 → AI 에이전트가 해석 → Action 제안 → 실행
```

### 4.4 Graph RAG vs. 우리 전략

| 특성 | 일반 Graph RAG | 우리 전략 (운영형 온톨로지) |
|------|---------------|-------------------------|
| 목적 | 지식 검색 | **운영 실행** |
| 쓰기 능력 | 없음 | Action을 통한 상태 변경 |
| 거버넌스 | 없음 | 권한·감사·유효성 검사 내장 |
| 비즈니스 로직 | 없음 | Function으로 온톨로지에 내장 |

> **우리는 Graph RAG을 하위 구성요소로 포함하되, 그 위에 Function + Action + Security를 올린다.**

---

## 제5장. 적용 시나리오 — Before/After 완전 비교

> 상세: [04-application-scenarios.md](../research/04-application-scenarios.md)

### 시나리오 A: 불량 근본 원인 추적

**상황:** CNC-14에서 Edge Burr 불량률 30% 발생, 긴급 납기 물량

| 항목 | Before | After |
|------|--------|-------|
| 원인 파악 시간 | 5시간 (4개 시스템 수동 교차 분석) | **1분** (관계 그래프 자동 탐색) |
| 추가 불량 생산 | 8개 Lot (분석 중 계속 가동) | **0개** (즉시 원인 특정 + 조치) |
| 라인 정지 | 30분 (원인 불명으로 전체 정지) | **0분** (원인 특정 후 정밀 대응) |
| 감사 추적 | 수동 보고서 | 자동 감사 로그 |

**동작 흐름:**
```
불량 보고 → 시스템이 Lot→Machine→Tool→Recipe→Material 자동 탐색
→ calculateDefectContribution 실행: "피드 속도 변경 62%, 공구 마모 28%"
→ AI 제안: [레시피 롤백] + [Lot 격리] + [Tool 교체 스케줄링]
→ 엔지니어 승인 → Action 실행 → 결과 반영 (닫힌 루프)
```

### 시나리오 B: 실시간 공정 이상 대응 (야간)

**상황:** 야간 02:35, 설비 진동 이상, 전문 엔지니어 부재

| 항목 | Before | After |
|------|--------|-------|
| 대응 시간 | 7시간 (야간 공백 + 주간 분석) | **65분** (AI 자동 분석 + 원격 승인) |
| 비계획 다운타임 | 5시간 40분 | **0분** |
| 비용 | $124,000 | **$8,500** (93% 절감) |

**핵심:** 야간에도 AI 에이전트가 전문 엔지니어 수준의 분석을 제공하고, 리스크 수준별 조치 옵션을 제시한다.

### 시나리오 C: 예지정비 + 생산 일정 연동

**상황:** 설비 건강 점수 하락 + 긴급 고객 주문 (정비 vs 생산 갈등)

| 항목 | Before | After |
|------|--------|-------|
| 정비 판단 근거 | 경험 + 정비 주기표 | 건강 점수 + ML 예측 + 생산 영향 분석 |
| 비계획 고장 | "미루다가 고장" 패턴 빈발 | 상태 기반 정비로 예방 |
| 정비 비용 | 시간 기반 과잉/부족 정비 | **67% 절감** |
| OEE | 65% | **80%+ 목표** |

### 통합 시너지

세 시나리오는 동일한 온톨로지 객체를 공유하여 **데이터 선순환**을 만든다:
```
불량 패턴 분석(A) → 이상 감지 정확도 향상(B) → 정비 예측 개선(C)
→ 정비 후 품질 변화 학습(A) → ... (닫힌 루프)
```

**종합 ROI:** 매출 1,000억원 기준 연간 **₩108~188억원** 절감

---

## 제6장. 도입 전략 및 로드맵

> 상세: [05-strategy-roadmap.md](../research/05-strategy-roadmap.md)

### 6.1 기술 스택 권장: 하이브리드 접근

| 기준 | Palantir Foundry | OSS 자체 구축 | **하이브리드 (권장)** |
|------|-----------------|-------------|-------------------|
| 초기 비용 | ₩40~105억 | ₩21~38억 | **₩23~43억** |
| 구현 속도 | 빠름 (6~9개월) | 느림 (12~18개월) | **중간 (12~15개월)** |
| 벤더 독립성 | 낮음 | 높음 | **높음** |
| 커스터마이징 | 제한적 | 완전 | **완전** |
| 리스크 | 벤더 종속 | 기술 역량 의존 | **Phase 0에서 검증 후 확대** |

**하이브리드 권장 이유:**
1. Phase 0에 ₩3~5억, Phase 0+1 누적 ₩10~20억 수준에서 효과 검증 가능 (효과 미입증 시 Phase 1 완료 시점에서 중단 가능)
2. 온톨로지 설계 역량을 내부에 축적
3. Phase 2~3에서 필요 시 상용 플랫폼 전환 옵션 보유
4. 벤더 종속 없이 장기 전략적 유연성 확보

**OSS 기술 스택:**
- 에이전트: LangGraph + MCP (Model Context Protocol)
- 그래프 DB: Neo4j
- API: FastAPI + Pydantic (Action Pattern)
- 벡터 DB: Weaviate / Chroma
- LLM: Claude / GPT-4 / 온프레미스 Llama
- 모니터링: LangSmith

### 6.2 단계별 로드맵

```
Month:  1   2   3   4   5   6   7   8   9  10  11  12  13  14  15  16  17  18
        ├───┤
        Phase 0: 기반 구축 (₩3~5억, 3명)
            ├───────────────────┤
            Phase 1: 핵심 도메인 (₩8~15억, 7명)
                                ├───────────────────────────┤
                                Phase 2: AI 통합 (₩8~15억, 9명)
                                                            ├───────────────┤
                                                            Phase 3: 전사 확장 (₩4~8억, 9명)
```

**Phase 0 — 기반 구축 (2개월) | ₩3~5억**
- 결정 카탈로그: 현장 핵심 의사결정 10개 식별
- PoC: Machine, Lot, Defect, WorkOrder, Recipe 5개 Object Type
- 데이터 매핑: ERP/MES/SCADA 연결 설계
- 팀: 온톨로지 엔지니어 1 + 데이터 엔지니어 1 + 도메인 전문가 1
- **성공 기준:** Competency Question 5개를 시스템으로 답할 수 있는가

**Phase 1 — 핵심 도메인 구축 (4개월) | ₩8~15억**
- Object Type 15개 + Link Type 14개 전체 구축
- Action Type 7개 + Function 7개 구현
- 데이터 통합 파이프라인
- 시나리오 A (불량 추적) 완전 구현
- 팀: 7명
- **성공 기준:** 불량 추적 시간 Before/After 측정

**Phase 2 — AI 통합 (6개월) | ₩8~15억**
- AI 에이전트 3개 구현 (품질 진단 / 예지정비 / 생산 최적화)
- Tool 구조 (Data 16 + Logic 7 + Action 7) 완전 구현
- 시나리오 B + C 구현
- ML 모델 통합, 보안/거버넌스 적용
- 팀: 9명
- **성공 기준:** 3개 시나리오 운영, 의사결정 시간 80% 단축

**Phase 3 — 전사 확장 (6개월) | ₩4~8억**
- 추가 도메인 확장 (에너지, 공급망)
- 외부 시스템(고객사, 공급사) 연동
- AI 에이전트 자율성 점진적 상향
- 팀: 9명
- **성공 기준:** OEE 85%+, 품질 비용 매출 대비 10% 이하

### 6.3 ROI 분석

**투자 대비 효과:**

| 항목 | 연간 기대 효과 |
|------|-------------|
| 품질 비용 절감 (COPQ 40~50% 감소) | ₩60~100억원 |
| 비계획 다운타임 감소 (75% 감소) | ₩30~55억원 |
| 의사결정 시간 단축 → 인건비 절감 | ₩8~13억원 |
| 에너지 비용 최적화 | ₩10~20억원 |
| **합계** | **₩108~188억원/년** |

**투자 회수:**
```
누적 투자 vs 누적 효과 (₩억원)

       투자    효과    순효과
Phase 0 (M2)    4      0      -4
Phase 1 (M6)   20     18     -2   ← 시나리오 A 효과 발생 시작
BEP (M6~8)     20     20+     0   ← BEP 달성
Phase 2 (M12)  35     72     +37
Phase 3 (M18)  43    150    +107
```

**리스크 조정 3년 ROI:**

*기준: 연간 절감 하한선(₩108억)과 총 투자 상한선(₩43억)을 사용하여 과대 추정 방지*

| 시나리오 | 성공률 | 연간 효과 | 3년 ROI 계산 | 3년 ROI |
|----------|--------|----------|-------------|---------|
| 보수적 | 60% | ₩108억 x 60% = ₩64.8억 | (64.8x3 - 43) / 43 | **약 350%** |
| 기본 | 80% | ₩108억 x 80% = ₩86.4억 | (86.4x3 - 43) / 43 | **약 503%** |
| 낙관적 | 100% | ₩108억 x 100% = ₩108억 | (108x3 - 43) / 43 | **약 654%** |

*연간 ROI 참고: 연간 절감(₩108~188억) / 총 투자(₩23~43억) = 251~817% (범위가 넓으므로 3년 누적 ROI를 주요 지표로 사용)*

---

## 제7장. 조직·거버넌스·리스크

### 7.1 필요 인력

| 역할 | Phase 0 | Phase 1 | Phase 2 | Phase 3 |
|------|---------|---------|---------|---------|
| 온톨로지 엔지니어 | 1 | 2 | 2 | 2 |
| 데이터 엔지니어 | 1 | 2 | 2 | 2 |
| AI/ML 엔지니어 | 0 | 1 | 2 | 2 |
| 도메인 전문가 (현장) | 1 | 1 | 2 | 2 |
| PM | 0 | 1 | 1 | 1 |
| **소계** | **3** | **7** | **9** | **9** |

### 7.2 거버넌스

**온톨로지 거버넌스:**
- 온톨로지 위원회(월 1회): Object Type/Link Type/Action Type의 추가·수정·폐기 심의
- 생명주기 관리: experimental → active → endorsed → deprecated

**AI 거버넌스:**
- 자율성 4단계: 정보 제공 → 추천 제안 → 인간 승인 실행 → 자율 실행
- 모든 AI 의사결정에 근거 체인(Evidence Chain) 필수
- 분기별 AI 감사 리뷰

**보안 거버넌스:**
- AI 에이전트에 인간과 동일한 보안 정책 적용
- 행·열·Action 수준의 접근 제어

### 7.3 주요 리스크

| 리스크 | 확률 | 영향 | 대응 |
|--------|------|------|------|
| 온톨로지 설계 오류 | 중 | 높 | Phase 0 PoC로 검증, 생명주기 관리 |
| AI 할루시네이션 | 중 | 중 | Action 실행은 인간 승인 필수, 근거 체인 |
| 현장 변화 저항 | 높 | 높 | 도메인 전문가 참여, 점진적 도입, 작은 성공 사례 |
| 인력 확보 어려움 | 중 | 중 | 내부 육성(6~12개월) + 외부 컨설턴트 병행 |
| ROI 미달 | 낮 | 높 | Phase 0에 ₩3~5억, Phase 0+1 누적 ₩10~20억 수준에서 검증. 효과 미입증 시 Phase 1 완료 시점에서 중단 가능 |

---

## 제8장. 성공 지표 (KPI)

| KPI | Phase 0 | Phase 1 | Phase 2 | Phase 3 |
|-----|---------|---------|---------|---------|
| 온톨로지 커버리지 | 7 Object Types | 15 Types | 15 + AI Tools | 20+ Types |
| 데이터 통합률 | 3 시스템 | 5 시스템 | 7 시스템 | 7+ 외부 |
| 불량 추적 시간 | 베이스라인 측정 | 2시간 이하 | 10분 이하 | 1분 이하 |
| 의사결정 시간 단축 | - | 50% | 80% | 96% |
| 비계획 다운타임 | 베이스라인 | -30% | -60% | -75% |
| OEE | 베이스라인 | +5%p | +10%p | +15%p (85%+) |
| 품질 비용 (COPQ/매출) | 15~20% | 13~17% | 10~13% | 10% 이하 |
| AI 에이전트 사용률 | - | - | 일 50회+ | 일 200회+ |

---

## 부록. 리서치 산출물 목록

| 문서 | 파일 | 핵심 내용 |
|------|------|----------|
| 현장 문제 진단 | [01-problem-diagnosis.md](../research/01-problem-diagnosis.md) | 3대 Pain Point 구조 분석, 산업 벤치마크 데이터, 기존 접근 실패 원인 |
| 온톨로지 아키텍처 | [02-ontology-architecture.md](../research/02-ontology-architecture.md) | 15 Object Types, 14 Link Types, 7 Action Types, 7 Functions, 보안 모델, ASCII 그래프 |
| AI 통합 전략 | [03-ai-integration-strategy.md](../research/03-ai-integration-strategy.md) | 3층 아키텍처, Tool 30종, AI 에이전트 3개, ML 통합, Graph RAG 비교 |
| 적용 시나리오 | [04-application-scenarios.md](../research/04-application-scenarios.md) | 불량 추적/이상 대응/예지정비 3개 완전 시나리오, Before/After, ROI |
| 도입 전략 로드맵 | [05-strategy-roadmap.md](../research/05-strategy-roadmap.md) | 기술 스택 비교, 4단계 로드맵, ROI 분석, 조직/거버넌스/리스크 |

---

## 핵심 정리 5문장

> 1. **문제는 데이터가 없는 것이 아니라, 데이터 간의 의미와 관계와 행동이 없는 것이다.** 7개 시스템에 데이터는 넘치지만, 그것들이 연결되지 않아 매년 매출의 29~36%가 비효율로 손실된다.
>
> 2. **온톨로지는 SQL JOIN을 "업무 관계"로, raw table을 "현실 객체"로 바꾼다.** 이 변환이 일어나는 순간, BI 질문("불량률 몇 %?")이 운영 질문("이 불량을 살리려면 어디를 건드려야 하나?")으로 바뀐다.
>
> 3. **Function은 분석 코드가 아니라 도메인 판단 로직이고, Action은 수동 조치가 아니라 통제된 운영 개입이다.** 둘이 합쳐져야 "실행 가능한 온톨로지"가 된다.
>
> 4. **AI는 이 구조 위에서 자연어 인터페이스이자 24시간 운영 판단 엔진이다.** 하지만 실행은 반드시 Tool과 권한 체계를 통해 통제된다 — AI에게 초능력을 주지 않는다.
>
> 5. **하이브리드 접근으로 Phase 0에 ₩3~5억에서 검증하고, 효과가 입증되면 확대한다.** 18개월간 ₩23~43억을 투자하여 연간 ₩108~188억원의 비효율을 제거한다. 투자 회수 6~8개월.

---

*본 보고서는 Manufacturing Sector AI-Driven Research Center에서 6개 전문 에이전트 팀의 병렬 리서치를 통해 작성되었습니다.*

*참고 자료 상세 목록은 각 리서치 문서(01~05)의 참고 자료 섹션을 참조하십시오.*
