# 온톨로지 + AI 기반 스마트팩토리 도입 전략서 및 단계별 로드맵

> **문서 목적:** CTO/COO/CEO가 읽고 투자 결정을 내릴 수 있는 수준의 도입 전략서
>
> **작성일:** 2026-03-16
>
> **참조 문서:**
> - pre-research.md (팔란티어 온톨로지 통합 교안)
> - 01-problem-diagnosis.md (현장 문제 진단)
> - 02-ontology-architecture.md (온톨로지 아키텍처 설계)
> - 03-ai-integration-strategy.md (AI 통합 전략)
> - 04-application-scenarios.md (적용 시나리오 검증)

---

## 1. 경영진 요약 (Executive Summary)

### 핵심 메시지: "데이터 중심에서 결정 중심으로의 전환"

우리 공장에는 데이터가 부족하지 않다. ERP, MES, SCADA, QMS, CMMS, WMS, PLM — 7개 이상의 시스템에서 매일 수십만 건의 데이터가 생성된다. **문제는 데이터가 없는 것이 아니라, 데이터 간의 의미와 관계가 없다는 것이다.** 불량이 발생하면 4~5개 시스템에 로그인하여 엑셀로 병합하는 데 2~3일이 걸리고, 그 사이 동일 불량이 계속 생산된다.

**온톨로지 기반 운영형 의사결정 시스템**은 흩어진 데이터를 현실 세계의 객체(설비, Lot, 불량, 레시피)와 관계(이 Lot는 이 설비에서, 이 레시피로, 이 자재를 사용하여 생산됨)로 재구성하고, 그 위에 AI를 올려 **"분석"을 넘어 "실행"까지 자동화**한다.

| 항목 | 내용 |
|------|------|
| **문제** | 데이터 사일로, 품질 추적 불가(원인 파악 2~3일), 의사결정 병목(야간 대응 7시간+) |
| **해결책** | 온톨로지(통합 객체 모델) + AI 에이전트(자동 진단-판단-실행) |
| **기대 효과** | 품질 비용 40~50% 절감, 비계획 다운타임 75% 감소, 의사결정 시간 96% 단축 |
| **총 투자 규모** | Phase 0~3 합산 ₩23~43억원 (18개월) |
| **연간 기대 절감** | ₩108~188억원/년 (매출 1,000억원 기준) |
| **투자 회수 기간** | 6~8개월 (Phase 1 완료 시점) |

> **경영진에게 묻는 질문:**
> "현재 품질 비용이 매출의 15~20%(₩150~200억원/년), 비계획 다운타임이 매출의 11%(₩110억원/년)를 잠식하고 있습니다. 이 비용의 절반만 줄여도 연간 ₩130억원 이상의 효과가 있습니다. ₩23~43억원을 투자하여 이 구조적 문제를 해결할 것인가, 아니면 매년 ₩260억원 이상의 손실을 계속 감수할 것인가?"

---

## 2. 현황 진단 요약

### 2-1. 핵심 발견사항

01-problem-diagnosis.md에서 도출한 **3대 구조적 문제**:

1. **데이터 사일로:** 7개 이상의 이기종 시스템이 각각의 데이터를 소유하며, 시스템 간 의미와 관계가 없음. 같은 설비가 ERP에서는 `EQ-001-A`, MES에서는 `MACHINE_A_LINE1`, CMMS에서는 `ASSET-00142`로 불림.
2. **품질 추적 불가:** 불량 발생 시 Lot→Machine→Tool→Recipe→Material의 추적 경로가 시스템적으로 구축되지 않아, 근본 원인 분석에 6시간~3일 소요.
3. **의사결정 병목:** 이상 발생 후 정보 수집(30분~4시간), 원인 분석(1시간~1일), 조치 결정(30분~4시간), 실행(15분~2시간)까지 합산 최소 2시간~최대 3일 이상 소요.

### 2-2. 현재 비용 규모

| 비용 항목 | 매출 대비 비율 | 연간 금액 (매출 1,000억원 기준) | 출처 |
|-----------|---------------|-------------------------------|------|
| 품질 비용 (COPQ) | 15~20% | ₩150~200억원 | [IISE](https://www.iise.org/details.aspx?id=22118), [Parsec](https://www.parsec-corp.com/blog/cost-of-quality) |
| 비계획 다운타임 | 11% | ₩110억원 | [Siemens, 2024](https://assets.new.siemens.com/siemens/assets/api/uuid:1b43afb5-2d07-47f7-9eb7-893fe7d0bc59/TCOD-2024_original.pdf) |
| 과잉/부족 정비 | 2~3% | ₩20~30억원 | 내부 추정 |
| 비효율적 의사결정(인건비) | 1~2% | ₩10~20억원 | 내부 추정 |
| **합계** | **29~36%** | **₩290~360억원** | |

### 2-3. Do Nothing 시나리오의 리스크

현재 상태를 유지하면 발생하는 리스크:

- **경쟁력 약화:** 글로벌 제조 AI 시장이 연 35.3% 성장 중이며, 77% 이상의 제조사가 이미 AI를 도입함 ([MarketsandMarkets](https://www.marketsandmarkets.com/Market-Reports/artificial-intelligence-manufacturing-market-72679105.html), [Tech-Stack](https://tech-stack.com/blog/ai-adoption-in-manufacturing/)). AI 미도입 시 2~3년 내 원가 경쟁력에서 뒤처질 가능성 높음.
- **인력 리스크:** 숙련 엔지니어의 경험에 의존하는 품질 추적 역량은 인력 퇴직 시 사라짐. 제조업 인력 고령화가 가속되는 상황에서 지식의 시스템화는 생존 과제.
- **규제 리스크:** EU AI Act, 공급망 실사법 등 규제가 강화되면서, 품질 추적의 완전성(Full Traceability)과 AI 의사결정의 감사 가능성(Auditability)이 필수 요건으로 부상.
- **비용 누적:** 매년 ₩290~360억원의 비효율 비용이 축적되며, 시간이 지날수록 레거시 시스템 간 기술 부채(Technical Debt)가 증가하여 추후 전환 비용도 상승.

---

## 3. 전략 비전

### 3-1. 목표 상태 (To-Be): 온톨로지 기반 운영형 의사결정 시스템

```
현재 (As-Is):
  데이터 사일로 → 수동 데이터 수집 → 엑셀 분석 → 경험적 판단 → 수동 실행
  (각 단계에서 시간 손실, 정보 손실, 판단 오류 발생)

목표 (To-Be):
  통합 온톨로지 → 자동 컨텍스트 조합 → AI 분석/판단 → 통제된 Action 실행
  → 결과 반영 → 다음 의사결정에 재사용 (닫힌 루프)
```

**핵심 전환:** BI 질문("불량률이 몇 %인가?")에서 운영 질문("이 불량 Lot를 살리려면 어디를 건드려야 하는가?")으로의 전환.

### 3-2. 04-application-scenarios.md에서 검증된 3개 시나리오의 통합 효과

| 시나리오 | 핵심 가치 | Before → After |
|----------|-----------|----------------|
| **A. 불량 근본 원인 추적** | 품질 추적 시간 99.7% 단축 | 5시간 → 1분, 라인 정지 30분 → 0분 |
| **B. 실시간 공정 이상 대응** | 비계획 다운타임 100% 제거 | 야간 7시간 대응 → 65분, 다운타임 비용 93% 절감 |
| **C. 예지정비 + 생산 일정 연동** | 정비-생산 갈등 해소 | 수동 협의 2시간 → AI 제안 15분, 비계획 정지 리스크 23% → 0% |

세 시나리오는 독립적이지 않다. **동일한 온톨로지 객체(Machine, Lot, Defect 등)를 공유**하여 데이터 선순환을 만든다: 불량 패턴(A) → 이상 감지 정확도 향상(B) → 정비 예측 개선(C) → 정비 후 품질 변화 학습(A).

### 3-3. 이 시스템이 만드는 차별화 역량 3가지

**1. 실시간 품질 추적 역량 (Full Traceability)**
- 불량 발생 → Lot → Machine → Tool → Recipe → Material → Supplier까지 수 초 내 전체 추적
- 고객사 감사(Audit) 대응: IATF 16949, ISO 9001 요구사항을 시스템적으로 충족
- 리콜 범위 최소화: 영향 Lot를 정밀하게 특정하여 과잉 리콜 방지

**2. 24시간 운영 의사결정 역량 (24/7 Decision Intelligence)**
- 야간/주말에도 AI 에이전트가 전문 엔지니어 수준의 분석과 조치 제안을 제공
- 의사결정의 품질이 시간대, 인력 가용성에 좌우되지 않음
- 알람 피로(Alarm Fatigue) 해소: AI가 수백 개 알람을 필터링하여 진짜 위험만 전달

**3. 조직 지식의 시스템화 역량 (Institutional Knowledge Capture)**
- 숙련 엔지니어의 판단 로직이 Function과 Action Type으로 코드화
- 모든 의사결정 이력(누가, 언제, 어떤 맥락에서, 무엇을 결정했는가)이 감사 로그로 축적
- 인력 변동(퇴직, 이동)에도 의사결정 역량이 보존되고 지속적으로 개선

---

## 4. 기술 스택 선택지 비교

### 4-1. Option A: Palantir Foundry 도입

**장점:**
- 검증된 플랫폼: NATO, NHS, 글로벌 제조사 등 대규모 운영 사례 보유
- 빠른 구현: 온톨로지 구축 도구, Workshop(대시보드), AIP(AI 통합)가 원스톱 제공
- 풍부한 사례: 산업별 템플릿과 베스트 프랙티스가 축적
- 비개발자 접근성: 코드 없이 온톨로지 구축 및 대시보드 생성 가능

**단점:**
- 높은 비용: Forrester 분석 기준 연간 라이선스 + 전문 서비스 + 클라우드 비용이 수십억원 규모 ([Palantir TEI Report](https://www.palantir.com/assets/xrfr7uokpv1b/7h0zi3GZrU3L7AM2HO1Q6O/1ad26eaa42ad949f8e3c80ea22f96b7a/The_Total_Economic_Impact_of_Palantir_Foundry.pdf))
- 벤더 종속: OSDK, Pipeline Builder 등이 Palantir 전용. 타 플랫폼 전환 시 상당한 재구축 필요
- 커스터마이징 제약: 플랫폼 프레임워크 내에서만 확장 가능. 독자적 알고리즘/로직 적용에 한계
- 국내 지원: 한국 시장 직접 지원 인프라가 제한적

**적합 대상:** 매출 5,000억원 이상 대기업, 글로벌 운영 체계가 필요한 기업, 빠른 ROI가 필수인 경우

**예상 비용 범위:**

| 항목 | 연간 비용 |
|------|-----------|
| 플랫폼 라이선스 | ₩20~50억원 |
| 구현 컨설팅 (SI) | ₩10~30억원 (초기 1년) |
| 클라우드 인프라 | ₩5~15억원 |
| 내부 인력 (운영) | ₩5~10억원 |
| **초기 1년 합계** | **₩40~105억원** |
| **연간 운영 비용** | **₩25~60억원** |

### 4-2. Option B: 자체 구축 (OSS 기반)

**장점:**
- 벤더 독립: 모든 구성 요소를 오픈소스로 대체 가능, 벤더 종속 없음
- 완전한 커스터마이징: 도메인 특화 알고리즘, 자체 ML 모델 자유 적용
- 장기 비용 절감: 라이선스 비용 없음, 인건비 중심 비용 구조
- 기술 역량 내재화: 팀이 시스템 전체를 이해하고 운영할 수 있음

**단점:**
- 긴 구현 기간: Phase 0~3 완료까지 18~24개월 (Palantir 대비 6~12개월 추가)
- 높은 기술 역량 요구: 온톨로지 엔지니어, LLM 엔지니어 등 희소 인력 필요
- 초기 리스크: 설계 오류 시 전면 재설계 가능성
- 지원 부재: 트러블슈팅을 내부 역량으로 해결해야 함

**적합 대상:** 기술 역량이 있는 중견기업, 장기적 전략 투자가 가능한 기업, 도메인 특화가 핵심인 경우

**03-ai-integration-strategy.md에서 정의한 기술 스택:**

| Palantir 구성 요소 | OSS 대안 |
|-------------------|---------|
| Foundry Ontology | **Neo4j** (Property Graph) + **PostgreSQL** |
| AIP Agent Studio | **LangGraph** (에이전트 오케스트레이션) |
| AIP Logic / Tools | **MCP (Model Context Protocol)** + **FastAPI** |
| Document RAG | **Weaviate** / **Chroma** (벡터 검색) |
| Security / Policy | **OPA (Open Policy Agent)** + **JWT** |
| Dashboard | **Streamlit** (PoC) / **Next.js** (프로덕션) |
| LLM | **Claude API** / **GPT-4** / **Llama 3** (온프레미스) |
| ML 모델 서빙 | **vLLM** + **MLflow** |
| 데이터 파이프라인 | **Apache Kafka** + **dbt** |
| 배포/운영 | **ArgoCD** + **Kubernetes** |

**예상 비용 범위:**

| 항목 | 비용 |
|------|------|
| 인건비 (Phase 0~3, 18개월) | ₩15~25억원 |
| 클라우드 인프라 | ₩3~5억원/년 |
| LLM API 비용 | ₩1~3억원/년 |
| 외부 컨설팅/교육 | ₩2~5억원 (초기) |
| **초기 투자 (18개월)** | **₩21~38억원** |
| **연간 운영 비용** | **₩6~12억원** |

### 4-3. Option C: 하이브리드 (권장)

**접근 방식:**
- **Phase 0~1 (1~6개월):** OSS 기반으로 PoC + 핵심 온톨로지 구축. 최소 비용으로 개념을 검증하고, 내부 역량을 축적
- **Phase 2~3 (6~18개월):** 검증된 PoC를 바탕으로 확장. 이 시점에서 규모와 요구사항에 따라 상용 플랫폼 도입 여부를 재검토

**장점:**
- 리스크 관리: 소규모 투자로 검증 후 확대. "대형 실패" 방지
- 학습 효과: Phase 0~1에서 내부 팀이 온톨로지 개념, AI 통합 패턴을 체득. 이후 상용 플랫폼 도입 시에도 주체적 판단 가능
- 유연성: Phase 2에서 Palantir, Microsoft Fabric, 자체 구축 중 최적 선택지를 데이터에 근거하여 결정
- 비용 효율: Phase 0에 ₩3~5억원으로 PoC 가치 검증 가능

**단점:**
- 전환 비용 가능성: OSS PoC에서 상용 플랫폼으로 전환 시 일부 재구축 필요
- 의사결정 지연: Phase 2 시점에서 "계속 자체 구축 vs. 상용 도입"의 추가 의사결정 필요

**왜 권장하는가:**
1. **불확실성 관리:** 온톨로지 + AI 전략은 아직 초기 단계인 접근법. 대규모 선행 투자보다 점진적 검증이 합리적
2. **내부 역량이 핵심:** 어떤 플랫폼을 선택하든, 온톨로지 설계와 도메인 로직은 내부 팀이 정의해야 함. Phase 0~1에서 이 역량을 먼저 확보
3. **시장 데이터 기반 결정:** Phase 1 완료 후 실제 운영 데이터(성능, 확장성, 운영 비용)가 축적되면, 정량적 근거로 다음 단계를 결정할 수 있음
4. **산업 현실:** 제조 AI 도입 기업의 77%가 이미 착수했으나, 성숙한 배포를 달성한 기업은 1% 미만 ([Netguru, 2026](https://www.netguru.com/blog/ai-adoption-statistics)). 점진적 접근이 성공 확률을 높임

**예상 비용 범위:**

| 항목 | 비용 |
|------|------|
| Phase 0~1 (OSS PoC, 6개월) | ₩11~20억원 |
| Phase 2~3 (확장, 12개월) | ₩12~23억원 |
| **초기 투자 (18개월)** | **₩23~43억원** |
| **연간 운영 비용** | **₩5~12억원** |

### 4-4. 비교 매트릭스

| 기준 | Palantir Foundry | 자체 구축 (OSS) | 하이브리드 (권장) |
|------|:----------------:|:---------------:|:-----------------:|
| **초기 비용** | ₩40~105억 (매우 높음) | ₩21~38억 (중간) | ₩23~43억 (중간) |
| **연간 운영 비용** | ₩25~60억 (높음) | ₩6~12억 (낮음) | ₩5~12억 (낮음) |
| **구현 속도** | 6~12개월 (빠름) | 18~24개월 (느림) | 12~18개월 (중간) |
| **커스터마이징** | 중간 (플랫폼 제약) | 높음 (완전 자유) | 높음 (OSS 기반) |
| **벤더 독립성** | 낮음 (종속) | 높음 (완전 독립) | 높음 (OSS 우선) |
| **기술 역량 요구** | 중간 (교육 필요) | 매우 높음 | 높음 (점진적 축적) |
| **리스크 수준** | 중간 (비용 리스크) | 높음 (기술 리스크) | 낮음 (점진적 검증) |
| **비개발자 접근성** | 높음 | 낮음 (초기) | 중간→높음 (점진) |
| **3년 총 비용 (TCO)** | ₩90~225억 | ₩33~62억 | ₩33~67억 |

---

## 5. 단계별 로드맵

### Phase 0: 기반 구축 (2개월)

**목표:** "이 접근이 우리 현장에서 작동하는가?"를 최소 비용으로 검증

**핵심 활동:**

| 활동 | 상세 | 산출물 |
|------|------|--------|
| 결정 카탈로그 작성 | 현장에서 매일/매주 내려지는 핵심 의사결정 10개 식별. 각 결정에 대해: 누가, 어떤 정보로, 무엇을 결정하고, 결과가 어떻게 반영되는가를 문서화 | 결정 카탈로그 문서 |
| 핵심 Object Type 5개 PoC | Machine, Lot, Defect, WorkOrder, Recipe의 5개 Object Type을 Neo4j에 구현. 실제 데이터 100건 이상 적재 | Object Type 스키마 + 샘플 데이터 |
| 데이터 매핑 설계 | ERP/MES/SCADA 원천 시스템과의 연결 포인트 정의. 마스터 데이터 매핑(설비 코드 통일 등) | 데이터 매핑 문서 |
| 팀 구성 | 온톨로지 엔지니어 1명, 데이터 엔지니어 1명, 도메인 전문가(현장) 1명 | 프로젝트 킥오프 |

**성공 기준:** Competency Question 5개를 시스템으로 답할 수 있는가
1. "Lot L-001이 어떤 설비에서 어떤 레시피로 생산되었는가?"
2. "Machine M-14에 현재 장착된 공구의 마모율은?"
3. "Recipe R-42를 사용한 최근 30일 Lot의 평균 불량률은?"
4. "이번 주 Edge Burr 불량이 집중된 설비는?"
5. "CNC-14의 최근 6개월 정비 이력과 불량률 상관관계는?"

**예상 비용:** ₩3~5억원 (인건비 3명 x 2개월 + 인프라)

---

### Phase 1: 핵심 도메인 구축 (3~6개월, 팀 7명)

**목표:** 시나리오 A(불량 추적)를 완전히 운영 가능한 수준으로 구현하고, Before/After 효과를 측정

**핵심 활동:**

| 활동 | 상세 |
|------|------|
| 전체 Object Type 15개 구축 | Plant, ProductionLine, Machine, Tool, Recipe, Material, Lot, Measurement, Defect, WorkOrder, Operator, Supplier, Shift, MaintenanceRecord, Product (02 설계서 기반) |
| Link Type 14개 구축 | 품질 추적 체인, 정비 체인, 생산 체인의 전체 관계 정의 (02 설계서 기반) |
| Action Type 7개 구현 | EscalateDefect, CreateWorkOrder, QuarantineLot, ChangeRecipe, RequestMaintenance, AssignOperator, UpdateMachineStatus |
| Function 7개 구현 | calculateMachineHealthScore, calculateDefectContribution, assessProcessRisk, predictToolWear, calculateLotSensitivity, evaluateSupplierQuality, estimateDowntimeImpact |
| 데이터 통합 파이프라인 | ERP, MES, SCADA, QMS 4개 시스템의 실시간/배치 데이터 연동 |
| Workshop(대시보드) 구축 | Streamlit 기반 품질 추적 대시보드 + 관계 그래프 시각화 |
| 시나리오 A 완전 구현 | 불량 근본 원인 추적 시나리오 End-to-End 운영 |

**성공 기준:** 시나리오 A의 Before/After 개선 효과 측정
- 원인 추적 시간: 6시간 → 15분 이하
- 영향 범위 자동 파악률: 90% 이상
- 감사 로그 완전성: 100%

**예상 비용:** ₩8~15억원 (인건비 7명 x 4개월 + 인프라 + 데이터 통합)

---

### Phase 2: AI 통합 (6~12개월, 팀 9명)

**목표:** 3개 AI 에이전트를 구현하고, 시나리오 B(이상 대응) + 시나리오 C(예지정비)를 운영

**핵심 활동:**

| 활동 | 상세 |
|------|------|
| LLM 기반 AI 에이전트 3개 | 품질 진단 Agent, 예지정비 Agent, 생산 최적화 Agent (03 설계서 기반) |
| Tool 구조 완전 구현 | Data Tools(8개) + Logic Tools(7개) + Action Tools(7개) → MCP 표준 기반 |
| 시나리오 B 구현 | 실시간 공정 이상 대응: 자동 감지 → 원인 분석 → 조치 옵션 생성 → Action 실행 |
| 시나리오 C 구현 | 예지정비 + 생산 일정 연동: 건강 점수 모니터링 → 정비 시점 최적화 → 대체 설비 자동 탐색 |
| 기존 ML 모델 통합 | 진동 분석, RUL 예측, SPC 이상 탐지 모델을 온톨로지 Function으로 래핑 |
| 보안/거버넌스 모델 적용 | 행 수준/열 수준/Action 권한 매트릭스 운영. OPA 기반 정책 엔진 |
| AI 평가 체계 구축 | LangSmith 기반 에이전트 성능 평가 + 할루시네이션 모니터링 |

**성공 기준:**
- 3개 시나리오 모두 운영
- 의사결정 시간 80% 단축 (주간/야간 동일)
- AI 에이전트 응답 정확도 90% 이상
- 비계획 다운타임 50% 감소

**예상 비용:** ₩8~15억원 (인건비 9명 x 6개월 + LLM API + ML 인프라)

---

### Phase 3: 전사 확장 (12~18개월, 팀 9명)

**목표:** 시스템을 전사 표준으로 확장하고, 닫힌 루프를 완전 운영

**핵심 활동:**

| 활동 | 상세 |
|------|------|
| 추가 도메인 확장 | 에너지 최적화(설비 전력 소비 + 생산 스케줄 연동), 공급망(공급사 품질 연동), 인사(교대/역량 최적화) |
| 닫힌 루프 완전 운영 | Action 실행 → 결과 반영 → 학습 → 다음 판단 개선의 자율 사이클 |
| 외부 시스템 연동 | 고객사(현대모비스 등) 납기 시스템, 공급사 품질 데이터 직접 연동 |
| AI 에이전트 자율성 상향 | Level 1(제안)에서 Level 2~3(승인 후 실행 / 자동 실행)으로 점진적 전환 |
| 다공장 확장 | 수원 1공장 → 2공장 → 해외 법인으로 온톨로지 확장 |
| 프로덕션 UI 전환 | Streamlit → Next.js 기반 프로덕션 대시보드 |

**성공 기준:**
- OEE 85% 이상 (현행 65% 대비 20%p 향상)
- 품질 비용 매출 대비 10% 이하 (현행 15~20% 대비 5~10%p 절감)
- AI 에이전트 일일 사용률 80% 이상 (현장 엔지니어 기준)
- 전사 온톨로지 커버리지 90% 이상

**예상 비용:** ₩4~8억원 (인건비 9명 x 6개월 + 인프라 확장 + UI 개발)

---

### 로드맵 시각화 (ASCII 간트 차트)

```
월      1    2    3    4    5    6    7    8    9   10   11   12   13   14   15   16   17   18
       ├────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┤

Phase 0 ████████
기반구축  결정카탈로그
        Object 5개 PoC
        데이터매핑
        팀 3명

Phase 1      ████████████████████████
핵심도메인    Object 15개 + Link 14개
             Action 7개 + Function 7개
             데이터 파이프라인
             시나리오 A 운영
             팀 7명
                              ▲ 효과 측정 1차

Phase 2                       ████████████████████████████████████████
AI 통합                        AI 에이전트 3개
                               Tool 구조 (Data/Logic/Action)
                               시나리오 B, C 구현
                               ML 모델 통합
                               보안/거버넌스
                               팀 9명
                                                        ▲ 효과 측정 2차

Phase 3                                                  ████████████████████████████████
전사확장                                                   도메인 확장 (에너지/공급망)
                                                          닫힌 루프 완전 운영
                                                          다공장 확장
                                                          프로덕션 UI
                                                          팀 9명
                                                                                  ▲ 최종 효과 측정

투자     ₩3~5억    ₩8~15억             ₩8~15억                     ₩4~8억
(누적)    (₩3~5억)  (₩11~20억)          (₩19~35억)                  (₩23~43억)

BEP                                     ▲ 예상 BEP (6~8개월)
```

---

## 6. ROI 분석

### 6-1. 투자 비용 추정

#### 인건비 (팀 구성 x 기간)

| Phase | 인원 | 기간 | 평균 연봉 기준 | 비용 |
|-------|------|------|---------------|------|
| Phase 0 | 3명 | 2개월 | ₩8,000만원 | ₩0.4억원 |
| Phase 1 | 7명 | 4개월 | ₩8,000만원 | ₩1.9억원 |
| Phase 2 | 9명 | 6개월 | ₩9,000만원 | ₩4.1억원 |
| Phase 3 | 9명 | 6개월 | ₩9,000만원 | ₩4.1억원 |
| **합계** | | **18개월** | | **₩10.5억원** |

참고: 온톨로지 엔지니어/AI 엔지니어의 시장 급여는 미국 기준 $107K~$207K ([ZipRecruiter](https://www.ziprecruiter.com/Salaries/Ontology-Engineer-Salary), [Coursera](https://www.coursera.org/articles/ai-engineer-salary)). 국내 기준 연봉 ₩7,000만~₩1.2억원 수준.

#### 인프라 비용

| 항목 | Phase 0~1 | Phase 2~3 | 연간 운영 |
|------|-----------|-----------|-----------|
| 클라우드 (GCP/AWS) | ₩0.5억원 | ₩1.5억원 | ₩2~3억원 |
| Neo4j Enterprise | ₩0.3억원 | ₩0.5억원 | ₩0.5억원 |
| LLM API (Claude/GPT-4) | ₩0.2억원 | ₩1.0억원 | ₩1~2억원 |
| 벡터 DB (Weaviate) | ₩0.1억원 | ₩0.3억원 | ₩0.3억원 |
| **합계** | **₩1.1억원** | **₩3.3억원** | **₩3.8~5.8억원** |

#### 소프트웨어 라이선스 (Option별)

| 항목 | 하이브리드 (권장) | Palantir | 자체 구축 |
|------|:-----------------:|:--------:|:---------:|
| 플랫폼 라이선스 | ₩0 (OSS) | ₩20~50억원/년 | ₩0 (OSS) |
| Neo4j Enterprise | ₩0.5억원/년 | 포함 | ₩0.5억원/년 |
| LLM API | ₩1~2억원/년 | 포함 | ₩1~2억원/년 |
| 모니터링 (LangSmith) | ₩0.3억원/년 | 포함 | ₩0.3억원/년 |

#### 외부 컨설팅/교육

| 항목 | 비용 | 시기 |
|------|------|------|
| 온톨로지 설계 컨설팅 | ₩1~2억원 | Phase 0~1 |
| LLM/AI 통합 기술 자문 | ₩1~2억원 | Phase 2 |
| 내부 인력 교육 | ₩0.5~1억원 | Phase 0~2 |
| **합계** | **₩2.5~5억원** | |

#### 총 투자 규모 범위 (Phase 0~3)

*참고: 아래 상세 적산은 인건비 고정 + 인프라/컨설팅 변동 기준의 항목별 산출입니다. 총 투자 범위 ₩23~43억원은 Phase별 투자 합산(Phase 0: ₩3~5억 + Phase 1: ₩8~15억 + Phase 2: ₩8~15억 + Phase 3: ₩4~8억)으로, 팀 규모 변동, 인프라 확장, 예비비를 모두 포함한 전체 범위입니다.*

| 항목 | 보수적 | 적극적 |
|------|--------|--------|
| Phase 0 (2개월, 3명) | ₩3억원 | ₩5억원 |
| Phase 1 (4개월, 7명) | ₩8억원 | ₩15억원 |
| Phase 2 (6개월, 9명) | ₩8억원 | ₩15억원 |
| Phase 3 (6개월, 9명) | ₩4억원 | ₩8억원 |
| **합계** | **₩23억원** | **₩43억원** |

### 6-2. 기대 효과 정량화

04-application-scenarios.md의 정량 데이터를 기반으로 산출 (매출 1,000억원 규모 제조사 기준):

#### 품질 비용 절감

| 항목 | 현행 | 목표 | 절감액/년 | 근거 |
|------|------|------|-----------|------|
| COPQ 비율 | 15~20% | 8~10% | **₩50~100억원** | [IISE](https://www.iise.org/details.aspx?id=22118): 세계적 수준 기업 COPQ 5% 이하 |
| 내부 실패 비용 | ₩80억원 | ₩30억원 | ₩50억원 | 시나리오 A: 즉시 격리로 재작업 범위 축소 |
| 외부 실패 비용 | ₩40억원 | ₩15억원 | ₩25억원 | 출하 전 97% 차단 목표 |
| 과잉 검사 비용 | ₩15억원 | ₩5억원 | ₩10억원 | AI 기반 선택적 검사 (검사 범위 60~70% 축소) |

#### 비계획 다운타임 감소

| 항목 | 현행 | 목표 | 절감액/년 | 근거 |
|------|------|------|-----------|------|
| 비계획 다운타임 | 매출 11% | 매출 5% | **₩60억원** | [Siemens, 2024](https://assets.new.siemens.com/siemens/assets/api/uuid:1b43afb5-2d07-47f7-9eb7-893fe7d0bc59/TCOD-2024_original.pdf) |
| 연간 비계획 정지 시간 | 설비당 200시간 | 50시간 | 75% 감소 | 시나리오 B: 비계획 다운타임 100% → 계획 정지 전환 |

#### 의사결정 시간 단축 → 인건비 절감

| 항목 | 현행 | 목표 | 절감액/년 | 근거 |
|------|------|------|-----------|------|
| 불량 추적 인건비 | 4명 x 5시간/건 x 50건/년 | 2명 x 0.25시간/건 | **₩4.8억원** | 시나리오 A: 20인시 → 0.5인시/건 |
| 이상 대응 인건비 | 수면 방해 + 출근 포함 | 현장 인력만 대응 | **₩2.0억원** | 시나리오 B: 에스컬레이션 최소화 |
| 정비 협의 인건비 | 4명 x 2시간 x 24회/년 | AI 자동 제안 | **₩1.0억원** | 시나리오 C: 수동 협의 90% 절감 |

#### 에너지 비용 최적화

| 항목 | 현행 | 목표 | 절감액/년 | 근거 |
|------|------|------|-----------|------|
| 에너지 비용 최적화 | - | 5~10% 절감 | **₩3~5억원** | Phase 3 에너지 도메인 확장 시. SCADA 전력 데이터 + 생산 스케줄 연동 |

#### 합산 연간 효과

```
품질 비용 절감:          ₩60~100억원
다운타임 비용 절감:       ₩30~55억원
인건비 절감:              ₩8~13억원
에너지 비용 최적화:       ₩10~20억원
──────────────────────────────
합계:                   ₩108~188억원/년
```

### 6-3. 투자 회수 기간 (Payback Period)

```
■ Phase별 누적 투자 vs 누적 효과 (기본 시나리오)

                투자(누적)    효과(누적)     순효과
Phase 0 (2개월)    ₩4억         ₩0억        -₩4억
Phase 1 (6개월)    ₩20억        ₩18억       -₩2억    ← 시나리오 A 효과 발생 시작
BEP (6~8개월)      ₩20억        ₩20억+      ₩0억     ← BEP 달성
Phase 2 (12개월)   ₩35억        ₩72억       +₩37억
Phase 3 (18개월)   ₩43억        ₩150억      +₩107억

■ 누적 투자 vs 누적 효과 그래프 (ASCII):

₩(억원)
160│                                                          ╱ 누적 효과
140│                                                        ╱
120│                                                      ╱
100│                                                    ╱
 80│                                                ╱╱
 60│                                           ╱╱
 40│                    ╱╱╱╱╱ 누적 투자   ╱╱╱
 20│              ╱╱╱╱╱╱          ╱╱╱╱╱
  0├─────╱╱╱╱╱╱───────╳──────────────────────────
   0    2    4    6    8   10   12   14   16   18  월
                        ▲
                      BEP (6~8개월)
```

**BEP (Break-Even Point) 예측:** Phase 1 완료 시점(6~8개월) 전후에 누적 투자를 회수. 시나리오 A의 품질 비용 절감만으로도 월 ₩4~8억원의 효과가 발생하기 때문.

### 6-4. 리스크 조정 ROI

| 시나리오 | 성공 확률 | 연간 효과 (기준: ₩108억) | 3년 ROI 계산식 | 3년 ROI |
|----------|-----------|--------------------------|----------------|---------|
| **보수적** | 60% | ₩108억 x 60% = ₩64.8억 | (64.8x3 - 43) / 43 | **약 350%** |
| **기본** | 80% | ₩108억 x 80% = ₩86.4억 | (86.4x3 - 43) / 43 | **약 503%** |
| **낙관적** | 100% | ₩108억 x 100% = ₩108억 | (108x3 - 43) / 43 | **약 654%** |

*3년 ROI 계산: (3년 리스크 조정 누적 효과 - 총 투자) / 총 투자 x 100%*
*보수적 기준으로 연간 절감 하한선(₩108억)과 총 투자 상한선(₩43억)을 사용하여 과대 추정을 방지*
*연간 ROI 참고: 연간 절감(₩108~188억) / 총 투자(₩23~43억) = 251~817% (범위가 넓으므로 3년 누적 ROI를 주요 지표로 사용)*

**산업 벤치마크:**
- 예측 정비 도입 기업 95%가 긍정적 ROI 보고, 27%는 1년 내 투자 회수 ([WorkTrek, 2025](https://worktrek.com/blog/predictive-maintenance-trends/))
- AI 기반 예측 정비 ROI: 12~18개월 내 10:1 ~ 30:1 ([Deloitte](https://www.deloitte.com/us/en/services/consulting/services/predictive-maintenance-and-the-smart-factory.html))
- 글로벌 자동차 제조사: AI 도입 후 다운타임 45% 감소, $1,500만 절감, OEE 12% 향상 ([Bridgera, 2025](https://bridgera.com/predictive-maintenance-in-manufacturing-how-ai-is-transforming-uptime-costs-safety/))

---

## 7. 조직 역량 요건

### 7-1. 필요 인력 구성

| 역할 | 주요 업무 | 필요 스킬 | 시장 현황 |
|------|-----------|-----------|-----------|
| **온톨로지 엔지니어** | Object/Link/Action Type 설계, 온톨로지 생명주기 관리, Competency Question 설계 | 지식 그래프, Property Graph (Neo4j), 도메인 이해, 데이터 모델링 | 글로벌 평균 연봉 $107K ([ZipRecruiter](https://www.ziprecruiter.com/Salaries/Ontology-Engineer-Salary)). 희소 인력. 국내 전문가 매우 부족 → 육성 필수 |
| **데이터 엔지니어** | 데이터 파이프라인 구축, 원천 시스템 연동, 마스터 데이터 관리 | Kafka, dbt, PostgreSQL, ETL/ELT, SQL | 국내 연봉 ₩6,000만~₩1억원. 시장 공급 상대적 양호 |
| **AI/ML 엔지니어** | LLM 에이전트 구현, Tool 구조 설계, ML 모델 통합, 프롬프트 엔지니어링 | LangGraph, MCP, LLM API, Python, ML Ops | 글로벌 평균 연봉 $141K~$206K ([Glassdoor](https://www.glassdoor.com/Salaries/ai-engineer-salary-SRCH_KO0,11.htm), [Coursera](https://www.coursera.org/articles/ai-engineer-salary)). 국내 ₩8,000만~₩1.5억원. 경쟁 치열 |
| **도메인 전문가 (현장)** | 결정 카탈로그 작성, Function 로직 검증, Action 유효성 규칙 정의, AI 응답 품질 평가 | 제조 공정 지식, 품질 관리, 설비 정비 경험 10년+ | 내부 인력 전환. 기존 공정/품질/설비 엔지니어 중 선발 |
| **프로젝트 매니저** | Phase별 일정/범위/리스크 관리, 이해관계자 커뮤니케이션, 성과 측정 | 제조 IT 프로젝트 경험, 애자일/스크럼, 변화 관리 | 내부 선발 또는 외부 채용 |

### 7-2. Phase별 팀 규모

| Phase | 기간 | 인원 | 구성 |
|-------|------|------|------|
| **Phase 0** | 2개월 | 3명 | 온톨로지 엔지니어 1 + 데이터 엔지니어 1 + 도메인 전문가 1 |
| **Phase 1** | 4개월 | 7명 | 온톨로지 엔지니어 2 + 데이터 엔지니어 2 + AI/ML 엔지니어 1 + 도메인 전문가 1 + PM 1 |
| **Phase 2** | 6개월 | 9명 | 온톨로지 엔지니어 2 + 데이터 엔지니어 2 + AI/ML 엔지니어 2 + 도메인 전문가 2 + PM 1 |
| **Phase 3** | 6개월 | 9명 | 온톨로지 엔지니어 2 + 데이터 엔지니어 2 + AI/ML 엔지니어 2 + 도메인 전문가 2 + PM 1 |

### 7-3. 내부 육성 vs 외부 채용 전략

**온톨로지 엔지니어 (가장 희소):**
- 국내 시장에 "온톨로지 엔지니어"라는 직군이 사실상 부재. 채용으로 확보 불가
- **전략:** 기존 데이터 아키텍트/시니어 데이터 엔지니어 중 도메인 이해가 깊은 인력을 선발하여 6~12개월 집중 육성
- 육성 과정: 지식 그래프 기초(2주) → Palantir 온톨로지 패턴 학습(2주) → Neo4j 실습(2주) → 도메인 적용 PoC(Phase 0 전체)
- Phase 0이 곧 육성 기간: 실제 프로젝트를 수행하며 역량을 체득

**AI/ML 엔지니어:**
- 시장 경쟁이 치열하나 확보 가능. LLM 에이전트 경험이 있는 시니어 1명을 외부 채용하고, 나머지는 내부 ML 엔지니어를 전환
- LangGraph, MCP 등 2025~2026년 등장한 기술이므로, 경력보다 학습 능력이 중요

**병행 전략:**
```
단기 (Phase 0~1):  외부 컨설턴트가 설계 리드 + 내부 인력이 함께 수행하며 학습
중기 (Phase 2):    내부 인력이 주도적으로 수행 + 외부 컨설턴트가 기술 자문
장기 (Phase 3):    완전 내부 운영. 외부 의존 제로
```

---

## 8. 거버넌스 모델

### 8-1. 데이터 거버넌스

**온톨로지 변경 관리: 누가 Object Type/Link Type을 추가/수정/폐기할 수 있는가**

```
변경 유형별 승인 권한:

  experimental → active:     도메인 팀 리드 승인
  active → endorsed:         온톨로지 위원회 승인 (전 조직 표준 등재)
  endorsed 수정/폐기:        온톨로지 위원회 + CTO 최종 승인
  Property 추가/변경:        해당 Object Type 소유 팀 리드 승인
  Link Type 추가:            연결되는 양쪽 도메인 팀 합의
```

**온톨로지 위원회(Ontology Committee) 구성:**

| 역할 | 인원 | 참여 빈도 |
|------|------|-----------|
| CTO (또는 CDO) — 위원장 | 1명 | 월 1회 정기, 긴급 시 수시 |
| 온톨로지 엔지니어 — 간사 | 1명 | 상시 |
| 도메인 대표 (생산/품질/설비/물류) | 4명 | 월 1회 정기 |
| IT/데이터 팀 대표 | 1명 | 월 1회 정기 |
| 보안/컴플라이언스 담당 | 1명 | 분기 1회 |

**변경 승인 프로세스:**
1. 변경 요청서 제출 (변경 사유, 영향 범위, 이전 호환성)
2. 영향 분석 자동 수행 (해당 Object/Link를 참조하는 앱, 대시보드, 에이전트 목록)
3. 영향받는 팀 의견 수렴 (5영업일)
4. 위원회 검토/승인
5. 변경 실행 + 영향 소비자 공지

### 8-2. AI 거버넌스

**AI 에이전트 자율성 수준 정의 및 승인 기준:**

| 자율성 수준 | 설명 | 승인 기준 | 초기 적용 |
|------------|------|-----------|-----------|
| Level 0: 정보 제공 | 질문에 답변만, Action 제안 없음 | 기본 | 모든 에이전트 시작점 |
| Level 1: 조치 제안 | Action 제안, 실행은 사용자 판단 | 정확도 80% 이상, 1개월 운영 검증 | Phase 2 초기 |
| Level 2: 승인 후 실행 | 사용자 승인 시 즉시 실행 | 정확도 90% 이상, 3개월 운영 검증 | Phase 2 후반 |
| Level 3: 자동 실행 | 정책 내 자동 실행, 사후 통보 | 6개월 Level 2 무사고 운영, 위원회 승인 | Phase 3 |

**AI 의사결정 감사 체계:**
- 모든 에이전트 응답에 **근거 체인(Evidence Chain)** 첨부: 어떤 Data Tool, Logic Tool, Document를 참조하여 결론을 도출했는지 전 과정 추적 가능
- 주간 AI 품질 리포트: 에이전트별 호출 횟수, 정확도, 거부 사유, 사용자 피드백 자동 집계
- 분기 AI 감사: 무작위 20건 샘플링 → 도메인 전문가가 AI 판단의 타당성 검증

**편향/할루시네이션 모니터링:**
- LangSmith 기반 실시간 모니터링: 에이전트 응답의 일관성, 사실성, 온톨로지 정합성 자동 검사
- 할루시네이션 탐지 규칙: AI 응답에 포함된 Object ID, 수치, 날짜가 온톨로지 데이터와 일치하는지 자동 검증
- 불일치 발견 시: 해당 응답 플래그 + 사용자에게 검증 요청 + 로그 기록

**규제 컴플라이언스:**
- EU AI Act: 본 시스템은 "high-risk AI system" (산업 안전 관련)에 해당할 수 있음. 투명성, 설명 가능성, 인간 감독 요건 충족을 설계에 내장
- 개인정보보호: Operator 객체의 민감 정보(급여, 개인정보)는 별도 Object로 분리, 열 수준 접근 제어 적용

### 8-3. 보안 거버넌스

**02-ontology-architecture.md에서 정의한 접근 권한 매트릭스 운영:**

```
행 수준 (Object-level):
  규칙: Machine.plantId in current_user.authorizedPlantIds
  → 수원 공장 담당자는 수원 공장 데이터만 조회 가능

열 수준 (Property-level):
  규칙: unitCost, maintenanceCost → role in ["finance_manager", "plant_director"]
  → 비용 정보는 재무/관리자만 조회 가능

Action 권한:
  규칙: ChangeRecipe → role in ["process_engineer", "quality_manager"]
  → 레시피 변경은 공정/품질 담당만 제안 가능
```

**정기 접근 권한 검토:**
- 월 1회: 퇴직/이동 인원의 권한 자동 해지 확인
- 분기 1회: 전체 권한 매트릭스 리뷰 (과잉 권한 식별)
- 반기 1회: 침투 테스트 + 접근 로그 이상 패턴 분석

**AI 에이전트 보안:**
- 에이전트는 대리 실행(Proxy Execution) 모델을 따름: 사용자의 권한으로만 데이터 접근 및 Action 실행
- 에이전트에게 별도의 "슈퍼 권한"을 부여하지 않음
- 에이전트 호출 로그: (actor, via_agent, tool, params, timestamp, result_summary) 전 건 기록

---

## 9. 리스크 관리

### 주요 리스크와 대응 방안

| 리스크 | 확률 | 영향 | 대응 방안 |
|--------|:----:|:----:|-----------|
| **기술 리스크** | | | |
| 온톨로지 설계 오류 (Object/Link 구조가 실제 업무와 불일치) | 중 | 높음 | Phase 0에서 Competency Question 기반 검증. 결정 카탈로그에서 역산하여 설계. experimental → active 전환 시 현장 엔지니어 검증 필수 |
| AI 할루시네이션 (잘못된 원인 분석, 부적절한 조치 제안) | 높음 | 중간 | 근거 체인 필수 첨부. Level 1(제안만)에서 시작하여 점진적 자율성 상향. Action 실행 전 정책 게이트 검증. 온톨로지 데이터와 교차 검증 |
| 데이터 통합 실패 (원천 시스템 연동 지연/불가) | 중 | 높음 | Phase 0에서 핵심 4개 시스템(ERP/MES/SCADA/QMS)의 연동 가능성을 선검증. API 미지원 시 DB 직접 연결 또는 파일 기반 연동 대안 확보 |
| LLM 성능 부족 (제조 도메인 특화 질문에 대한 정확도 미달) | 중 | 중간 | 프롬프트 엔지니어링 + Few-shot 예시 + Function-backed context로 보완. 온프레미스 모델(Llama 3) 파인튜닝 옵션 확보 |
| **조직 리스크** | | | |
| 변화 저항 (현장 엔지니어가 AI 시스템을 불신/거부) | 높음 | 높음 | Phase 0부터 현장 도메인 전문가를 팀에 포함. "AI가 대체한다"가 아니라 "AI가 데이터 수집 노동을 대신한다"는 프레이밍. 초기 성공 사례(Quick Win)로 신뢰 구축 |
| 핵심 인력 이탈 (온톨로지/AI 엔지니어 퇴사) | 중 | 높음 | 2인 이상 크로스 트레이닝. 핵심 설계 결정을 문서화(저널/설계서). 경쟁력 있는 보상 패키지. 프로젝트 완료 인센티브 |
| 부서 간 갈등 (데이터 소유권, 온톨로지 정의권 분쟁) | 중 | 중간 | 온톨로지 위원회가 중립적 조율. Golden Record 원칙으로 "하나의 정의, 여러 소비자" 합의. CTO/COO 스폰서십 확보 |
| **사업 리스크** | | | |
| ROI 미달 (기대 효과가 실현되지 않음) | 낮음 | 높음 | Phase별 효과 측정 게이트. Phase 1 완료 시 시나리오 A의 Before/After를 정량적으로 측정하여 Phase 2 진행 여부 결정. Phase 0에 ₩3~5억, Phase 0+1 누적 ₩10~20억 수준에서 검증. 효과 미입증 시 Phase 1 완료 시점에서 중단 가능 |
| 범위 확대 (Scope Creep, "이것도 넣자" 요구 확산) | 높음 | 중간 | Phase별 범위를 명확히 고정. 추가 요구는 "다음 Phase 백로그"로 관리. PM이 범위 변경 영향 분석을 의무적으로 수행 |
| 시장/기술 변화 (더 나은 솔루션 출현) | 낮음 | 낮음 | 하이브리드 접근으로 특정 기술에 고정되지 않음. OSS 기반이므로 구성 요소 교체 가능 (예: LangGraph → 차세대 프레임워크 전환) |

---

## 10. 성공 지표 (KPI)

### Phase별 KPI

| KPI | Phase 0 목표 | Phase 1 목표 | Phase 2 목표 | Phase 3 목표 |
|-----|:----------:|:----------:|:----------:|:----------:|
| **온톨로지 커버리지** (Object Type 구현률) | 5/15 (33%) | 15/15 (100%) | 15/15 + 확장 도메인 | 20+ Object Types |
| **데이터 통합률** (원천 시스템 연결 수) | 2/7 (ERP, MES) | 4/7 (+SCADA, QMS) | 6/7 (+CMMS, WMS) | 7/7 (+PLM) + 외부 |
| **의사결정 시간 단축률** | 측정 기준 수립 | 시나리오 A: 80%↓ | 전체: 80%↓ | 전체: 96%↓ |
| **불량 추적 소요 시간** | 측정 기준 수립 | 6시간 → 15분 | 15분 → 5분 | 1분 이내 |
| **비계획 다운타임 감소율** | 측정 기준 수립 | 20%↓ | 50%↓ | 75%↓ |
| **OEE 개선률** | 현행 측정 (65%) | 70% | 77% | 85%+ |
| **AI 에이전트 사용률** | - | - | 일일 50% (품질팀) | 일일 80% (전 팀) |
| **AI 응답 정확도** | - | - | 85% | 92%+ |
| **사용자 만족도** (1~5점) | - | 3.5 | 4.0 | 4.3+ |
| **감사 로그 완전성** | - | 100% (시나리오 A) | 100% (전 시나리오) | 100% |

### KPI 측정 및 보고 체계

- **주간:** AI 에이전트 사용률, 응답 정확도, 할루시네이션 건수 → 자동 대시보드
- **월간:** 의사결정 시간 단축률, 불량 추적 시간, 다운타임 → 경영진 보고
- **분기:** OEE, 품질 비용(COPQ), ROI 추적 → 투자 위원회 보고
- **Phase 전환 시:** 전체 KPI 리뷰 + 다음 Phase 진행/수정/중단 결정

---

## 참고 자료

**시장 및 투자 데이터:**
- [MarketsandMarkets - AI in Manufacturing Market Size (2025-2032)](https://www.marketsandmarkets.com/Market-Reports/artificial-intelligence-manufacturing-market-72679105.html)
- [Fortune Business Insights - Smart Factory Market (2025-2034)](https://www.fortunebusinessinsights.com/smart-factory-market-112706)
- [Deloitte - 2026 Manufacturing Industry Outlook](https://www.deloitte.com/us/en/insights/industry/manufacturing-industrial-products/manufacturing-industry-outlook.html)
- [IIoT World - 2026 Smart Factory Outlook](https://www.iiot-world.com/smart-manufacturing/2026-smart-factory-outlook-ai-robotics/)
- [한국 스마트 팩토리 시장 규모 보고서 (2025-2033)](https://www.imarcgroup.com/south-korea-smart-factory-market)

**AI 도입 성공률 및 ROI:**
- [Tech-Stack - AI Adoption in Manufacturing: ROI Benchmarks](https://tech-stack.com/blog/ai-adoption-in-manufacturing/)
- [Deloitte - State of AI in the Enterprise 2026](https://www.deloitte.com/global/en/issues/generative-ai/state-of-ai-in-enterprise.html)
- [Netguru - AI Adoption Statistics 2026](https://www.netguru.com/blog/ai-adoption-statistics)
- [NVIDIA - How AI Is Driving Revenue (2026)](https://blogs.nvidia.com/blog/state-of-ai-report-2026/)
- [WorkTrek - Predictive Maintenance Trends](https://worktrek.com/blog/predictive-maintenance-trends/)
- [Deloitte - Predictive Maintenance and the Smart Factory](https://www.deloitte.com/us/en/services/consulting/services/predictive-maintenance-and-the-smart-factory.html)

**비용 및 다운타임 벤치마크:**
- [Siemens - The True Cost of Downtime 2024](https://assets.new.siemens.com/siemens/assets/api/uuid:1b43afb5-2d07-47f7-9eb7-893fe7d0bc59/TCOD-2024_original.pdf)
- [IISE - Measuring the Cost of Quality](https://www.iise.org/details.aspx?id=22118)
- [Parsec - Cost of Quality in Manufacturing](https://www.parsec-corp.com/blog/cost-of-quality)
- [Fabrico - Cost of Poor Quality (COPQ) 2026 Guide](https://www.fabrico.io/blog/cost-of-poor-quality-copq-manufacturing-guide/)
- [Bridgera - AI Predictive Maintenance in Manufacturing](https://bridgera.com/predictive-maintenance-in-manufacturing-how-ai-is-transforming-uptime-costs-safety/)
- [LLumin - Predictive Maintenance: How Factories Slash Downtime by 40%](https://llumin.com/blog/predictive-maintenance-in-2025-how-factories-slash-downtime-by-40/)

**인력 시장 데이터:**
- [ZipRecruiter - Ontology Engineer Salary 2025](https://www.ziprecruiter.com/Salaries/Ontology-Engineer-Salary)
- [Glassdoor - AI Engineer Salary 2026](https://www.glassdoor.com/Salaries/ai-engineer-salary-SRCH_KO0,11.htm)
- [Coursera - AI Engineer Salary Guide 2026](https://www.coursera.org/articles/ai-engineer-salary)

**Palantir 관련:**
- [Palantir - The Total Economic Impact of Foundry (Forrester)](https://www.palantir.com/assets/xrfr7uokpv1b/7h0zi3GZrU3L7AM2HO1Q6O/1ad26eaa42ad949f8e3c80ea22f96b7a/The_Total_Economic_Impact_of_Palantir_Foundry.pdf)
- [Palantir Foundry Plans](https://www.palantir.com/platforms/foundry/plans/)

**기존 리서치 문서:**
- 01-problem-diagnosis.md — 현장 문제 진단 (데이터 사일로, 품질 추적 불가, 의사결정 병목)
- 02-ontology-architecture.md — 온톨로지 아키텍처 (15 Object, 14 Link, 7 Action, 7 Function)
- 03-ai-integration-strategy.md — AI 통합 전략 (3 Agent, Tool 구조, 보안/거버넌스)
- 04-application-scenarios.md — 적용 시나리오 (불량 추적, 이상 대응, 예지정비)
