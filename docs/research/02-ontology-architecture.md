# 스마트팩토리 온톨로지 아키텍처 설계서

> **문서 목적:** Palantir Foundry의 온톨로지 패턴을 벤치마크로, 범용 스마트팩토리 도메인에서 데이터 사일로 해소, 품질 추적, 의사결정 속도 향상을 달성하기 위한 온톨로지 아키텍처를 설계한다. 특정 벤더에 종속되지 않는 범용적 설계를 지향하되, Palantir의 실증된 패턴을 충실히 따른다.

---

## 1. 설계 원칙

### 1-1. 결정 중심 설계 (Decision-Centric Design)

온톨로지의 출발점은 "어떤 데이터를 저장할 것인가"가 아니라 \*\*"어떤 결정을 지원할 것인가"\*\*이다.

| 기존 접근 | 본 설계의 접근 |
| --- | --- |
| 데이터 테이블 구조 설계 → 이후 활용 고민 | 의사결정 시나리오 정의 → 필요한 객체/관계 도출 |
| "불량률이 몇 %냐?" (현황 질문) | "이 불량 Lot를 살리려면 어디를 건드려야 하나?" (운영 질문) |
| 쿼리 성능 최적화 | 의사결정 속도 최적화 |

**본 설계가 지원하는 핵심 의사결정:**

1. 불량 발생 시 근본 원인을 어떤 설비/공구/레시피/자재까지 역추적할 것인가
2. 설비 고장 전에 선제적 정비를 언제 실행할 것인가
3. 공정 조건 변경 시 품질 리스크를 어떻게 사전 평가할 것인가
4. 야간조/주간조 전환 시 가장 주의해야 할 설비는 무엇인가
5. 공급사 자재 품질 문제가 최종 제품에 미치는 영향 범위는 어디까지인가

### 1-2. Golden Record 원칙

**"하나의 정의, 여러 소비자 (One Definition, Many Consumers)"**

- 모든 Object Type은 중앙 온톨로지에서 \*\*단일 정의(Single Source of Truth)\*\*를 가진다.
- 생산팀의 "설비"와 정비팀의 "설비"와 품질팀의 "설비"는 동일한 `Machine` Object Type을 참조한다.
- 각 팀은 자신의 뷰(View)와 대시보드에서 필요한 속성만 소비하되, 정의 자체는 공유한다.
- Shared Property를 활용하여 `status`, `location` 등 공통 속성의 의미를 일원화한다.

### 1-3. 생명주기 관리 (Lifecycle Governance)

모든 Object Type, Link Type, Action Type은 다음 생명주기를 따른다:

```
experimental ──→ active ──→ endorsed ──→ deprecated ──→ (삭제)
    │                           │              │
    │  PoC 검증 완료 시         │  표준으로     │  대체 리소스 안내
    │  운영 전환                │  승인          │  마감 기한 명시
    └───────────────────────────┘              │  영향 소비자 공지
                                               └─────────────────
```

| 단계 | 의미 | 접근 범위 |
| --- | --- | --- |
| experimental | 개발/PoC 단계. 스키마 변경 자유 | 개발팀만 |
| active | 운영 중. 스키마 변경 시 영향 분석 필수 | 해당 도메인 팀 |
| endorsed | 조직 표준. 삭제 불가. 변경 시 거버넌스 위원회 승인 | 전 조직 |
| deprecated | 폐기 예고. 대체 리소스 안내 필수. 신규 참조 금지 | 기존 소비자만 (읽기 전용) |

### 1-4. 모든 쓰기는 Action을 통한다

온톨로지 객체의 상태 변경은 반드시 사전 정의된 Action Type을 통해야 한다. 직접 DB UPDATE는 허용하지 않는다. 이를 통해:

- 모든 변경에 유효성 검사(Validation)가 적용된다
- 감사 로그(Audit Log)가 자동 기록된다
- AI 에이전트와 인간이 동일한 변경 경로를 사용한다
- 규제 감사 대응이 구조적으로 보장된다

---

## 2. Object Type 설계

### 2-1. Plant (공장)

> **정의:** 물리적 제조 시설 단위로, 조직의 최상위 공간 경계를 나타낸다.

| Property | 타입 | 설명 | 확장 타입 |
| --- | --- | --- | --- |
| `plantId` | `string` | 공장 고유 식별자 (Primary Key) | \- |
| `name` | `string` | 공장명 (예: "수원 1공장") | \- |
| `location` | `geopoint` | GPS 좌표 | Shared Property |
| `timezone` | `string` | 공장 소재 시간대 (IANA 형식) | \- |
| `status` | `string` | 운영 상태 (`operating` / `shutdown` / `maintenance`) | Shared Property |
| `capacity` | `integer` | 일일 최대 생산 능력 (단위: 제품 수) | \- |
| `certifications` | `string[]` | 보유 인증 목록 (ISO 9001, IATF 16949 등) | \- |

- **Backing Datasource:** ERP 마스터 데이터 (공장 마스터 테이블)
- **Status:** `endorsed` — 조직 핵심 자원, 모든 접근 제어의 최상위 경계

---

### 2-2. ProductionLine (생산 라인)

> **정의:** 공장 내 특정 제품군을 생산하는 논리적/물리적 라인 단위이다.

| Property | 타입 | 설명 | 확장 타입 |
| --- | --- | --- | --- |
| `lineId` | `string` | 라인 고유 식별자 (Primary Key) | \- |
| `plantId` | `string` | 소속 공장 ID (Foreign Key) | \- |
| `name` | `string` | 라인명 (예: "SMT Line A") | \- |
| `productFamily` | `string` | 생산 제품군 | \- |
| `status` | `string` | 가동 상태 (`running` / `idle` / `changeover` / `down`) | Shared Property |
| `taktTime` | `double` | 목표 택트 타임 (초) | \- |
| `currentOEE` | `double` | 현재 OEE (Overall Equipment Effectiveness, 0\~1) | Timeseries |

- **Backing Datasource:** MES 라인 마스터 + 실시간 OEE 집계 파이프라인
- **Status:** `endorsed`

---

### 2-3. Machine (설비)

> **정의:** 생산 공정을 수행하는 개별 물리적 장비로, 센서 데이터와 정비 이력의 귀속 단위이다.

| Property | 타입 | 설명 | 확장 타입 |
| --- | --- | --- | --- |
| `machineId` | `string` | 설비 고유 식별자 (Primary Key) | \- |
| `lineId` | `string` | 소속 라인 ID (Foreign Key) | \- |
| `name` | `string` | 설비명 (예: "CNC-14") | \- |
| `machineType` | `string` | 설비 유형 (`cnc` / `press` / `smt` / `laser` / `assembly`) | \- |
| `status` | `string` | 운영 상태 (`active` / `maintenance` / `fault` / `idle`) | Shared Property |
| `lastMaintenanceDate` | `timestamp` | 최종 정비 일시 | \- |
| `currentSensorTemp` | `double` | 현재 온도 센서값 (실시간) | Timeseries |
| `vibrationLevel` | `double` | 현재 진동 수준 (mm/s) | Timeseries |
| `healthScore` | `double` | 설비 건강 점수 (0~100, calculateMachineHealthScore Function에 의해 주기적 갱신) | Timeseries |

- **Backing Datasource:** IoT 스트림 테이블 + ERP 설비 마스터 (조인)
- **Status:** `endorsed`
- **Interface:** `IAsset` 구현

---

### 2-4. Tool (공구/지그)

> **정의:** 설비에 장착되어 가공을 수행하는 소모성 또는 반소모성 부품이다.

| Property | 타입 | 설명 | 확장 타입 |
| --- | --- | --- | --- |
| `toolId` | `string` | 공구 고유 식별자 (Primary Key) | \- |
| `toolType` | `string` | 공구 유형 (`drill_bit` / `insert` / `jig` / `die`) | \- |
| `status` | `string` | 상태 (`in_use` / `available` / `worn` / `retired`) | Shared Property |
| `usageCount` | `integer` | 누적 사용 횟수 | \- |
| `maxUsageLimit` | `integer` | 최대 허용 사용 횟수 (제조사 권장) | \- |
| `wearPercentage` | `double` | 마모율 (0\~100%) | Timeseries |
| `installedAt` | `timestamp` | 현재 설비 장착 일시 | \- |

- **Backing Datasource:** 공구 관리 시스템 (TMS) + IoT 마모 센서
- **Status:** `active`
- **Interface:** `IAsset` 구현

---

### 2-5. Recipe (레시피/공정 조건)

> **정의:** 특정 제품을 생산하기 위한 공정 파라미터 세트(온도, 압력, 속도, 시간 등)이다.

| Property | 타입 | 설명 | 확장 타입 |
| --- | --- | --- | --- |
| `recipeId` | `string` | 레시피 고유 식별자 (Primary Key) | \- |
| `productId` | `string` | 대상 제품 ID (Foreign Key) | \- |
| `version` | `string` | 레시피 버전 (SemVer 형식) | \- |
| `parameters` | `string` | 공정 파라미터 JSON (온도, 압력, 속도, 시간 등) | \- |
| `tolerances` | `string` | 허용 공차 JSON (각 파라미터별 상한/하한) | \- |
| `status` | `string` | 상태 (`draft` / `approved` / `active` / `deprecated`) | Shared Property |
| `approvedBy` | `string` | 승인자 ID | \- |
| `effectiveFrom` | `timestamp` | 적용 시작 일시 | \- |

- **Backing Datasource:** MES 레시피 관리 모듈 + 공정 엔지니어링 DB
- **Status:** `endorsed`

---

### 2-6. Material (자재/원자재)

> **정의:** 생산에 투입되는 원자재, 부자재, 부품 등 물리적 입력물이다.

| Property | 타입 | 설명 | 확장 타입 |
| --- | --- | --- | --- |
| `materialId` | `string` | 자재 고유 식별자 (Primary Key) | \- |
| `name` | `string` | 자재명 | \- |
| `category` | `string` | 분류 (`raw_material` / `sub_material` / `component`) | \- |
| `specification` | `string` | 스펙/규격 | \- |
| `currentStock` | `double` | 현재 재고량 | \- |
| `unit` | `string` | 단위 (kg, ea, m 등) | \- |
| `trackingId` | `string` | 추적 ID (입고 로트 번호) | Shared Property (ITrackable) |
| `shelfLife` | `integer` | 유효 기간 (일) | \- |

- **Backing Datasource:** ERP 자재 마스터 + WMS(창고 관리 시스템) 재고 테이블
- **Status:** `endorsed`
- **Interface:** `ITrackable` 구현

---

### 2-7. Lot (생산 로트/배치)

> **정의:** 동일 조건(설비, 레시피, 자재)으로 생산된 제품 묶음으로, 품질 추적의 최소 단위이다.

| Property | 타입 | 설명 | 확장 타입 |
| --- | --- | --- | --- |
| `lotId` | `string` | 로트 고유 식별자 (Primary Key) | \- |
| `productId` | `string` | 생산 제품 ID (Foreign Key) | \- |
| `quantity` | `integer` | 생산 수량 | \- |
| `status` | `string` | 상태 (`in_progress` / `completed` / `quarantined` / `released` / `scrapped`) | Shared Property |
| `startTime` | `timestamp` | 생산 시작 일시 | \- |
| `endTime` | `timestamp` | 생산 종료 일시 | \- |
| `defectRate` | `double` | 불량률 (0\~1) | \- |
| `trackingId` | `string` | 추적 ID | Shared Property (ITrackable) |

- **Backing Datasource:** MES 생산 실적 테이블 + 품질 검사 결과 (조인)
- **Status:** `endorsed`
- **Interface:** `ITrackable` 구현

---

### 2-8. Measurement (측정값/센서 데이터)

> **정의:** 설비 센서 또는 품질 검사에서 수집된 개별 측정 레코드이다.

| Property | 타입 | 설명 | 확장 타입 |
| --- | --- | --- | --- |
| `measurementId` | `string` | 측정 고유 식별자 (Primary Key) | \- |
| `machineId` | `string` | 측정 대상 설비 ID (Foreign Key) | \- |
| `lotId` | `string` | 관련 로트 ID (Foreign Key, nullable) | \- |
| `measurementType` | `string` | 측정 유형 (`temperature` / `vibration` / `pressure` / `dimension` / `visual`) | \- |
| `value` | `double` | 측정값 | \- |
| `unit` | `string` | 단위 | \- |
| `timestamp` | `timestamp` | 측정 시각 | \- |
| `isOutOfSpec` | `boolean` | 규격 초과 여부 | \- |

- **Backing Datasource:** IoT 플랫폼 스트림 + 품질 검사 시스템 (SPC)
- **Status:** `active`

---

### 2-9. Defect (불량/결함)

> **정의:** 생산 과정에서 탐지된 개별 불량 건으로, 근본 원인 분석과 시정 조치의 대상이다.

| Property | 타입 | 설명 | 확장 타입 |
| --- | --- | --- | --- |
| `defectId` | `string` | 불량 고유 식별자 (Primary Key) | \- |
| `lotId` | `string` | 발생 로트 ID (Foreign Key) | \- |
| `defectType` | `string` | 불량 유형 (`dimensional` / `surface` / `functional` / `visual`) | \- |
| `severity` | `string` | 심각도 (`minor` / `major` / `critical`) | \- |
| `status` | `string` | 처리 상태 (`open` / `escalated` / `investigating` / `resolved` / `closed`) | Shared Property |
| `detectedAt` | `timestamp` | 탐지 일시 | \- |
| `rootCause` | `string` | 근본 원인 (분석 완료 후 기록) | \- |
| `imageRef` | `string` | 불량 이미지 참조 경로 | Media Reference |

- **Backing Datasource:** 품질 관리 시스템 (QMS) + 비전 검사 시스템
- **Status:** `endorsed`

---

### 2-10. WorkOrder (작업 지시)

> **정의:** 생산 또는 정비를 위해 발행되는 지시서로, 자원 배정과 일정의 기본 단위이다.

| Property | 타입 | 설명 | 확장 타입 |
| --- | --- | --- | --- |
| `workOrderId` | `string` | 작업 지시 고유 식별자 (Primary Key) | \- |
| `orderType` | `string` | 유형 (`production` / `maintenance` / `inspection` / `rework`) | \- |
| `status` | `string` | 상태 (`planned` / `scheduled` / `in_progress` / `completed` / `cancelled`) | Shared Property |
| `priority` | `string` | 우선순위 (`low` / `medium` / `high` / `urgent`) | Shared Property (ISchedulable) |
| `scheduledAt` | `timestamp` | 예정 일시 | Shared Property (ISchedulable) |
| `assignedTo` | `string` | 담당자 ID | Shared Property (ISchedulable) |
| `completedAt` | `timestamp` | 완료 일시 (nullable) | \- |
| `description` | `string` | 작업 내용 설명 | Semantic Search |

- **Backing Datasource:** ERP 작업 지시 테이블 + CMMS(정비 관리 시스템)
- **Status:** `endorsed`
- **Interface:** `ISchedulable` 구현

---

### 2-11. Operator (작업자)

> **정의:** 생산 현장에서 설비를 운전하거나 작업을 수행하는 인적 자원이다.

| Property | 타입 | 설명 | 확장 타입 |
| --- | --- | --- | --- |
| `operatorId` | `string` | 작업자 고유 식별자 (Primary Key) | \- |
| `name` | `string` | 성명 | \- |
| `role` | `string` | 역할 (`operator` / `technician` / `qa_inspector` / `shift_leader`) | \- |
| `certifications` | `string[]` | 보유 자격/인증 목록 | \- |
| `shiftPattern` | `string` | 근무 패턴 (`day` / `evening` / `night` / `rotating`) | \- |
| `plantId` | `string` | 소속 공장 ID (Foreign Key) | \- |

- **Backing Datasource:** HR 시스템 + MES 작업자 마스터
- **Status:** `active`
- **보안 참고:** `salary`, `personalInfo` 등 민감 속성은 본 Object에 포함하지 않고 별도 HR 전용 Object로 분리

---

### 2-12. Supplier (공급사)

> **정의:** 자재 또는 부품을 공급하는 외부 협력사이다.

| Property | 타입 | 설명 | 확장 타입 |
| --- | --- | --- | --- |
| `supplierId` | `string` | 공급사 고유 식별자 (Primary Key) | \- |
| `name` | `string` | 공급사명 | \- |
| `category` | `string` | 분류 (`tier1` / `tier2` / `tier3`) | \- |
| `qualityScore` | `double` | 종합 품질 점수 (0\~100) | Timeseries |
| `leadTime` | `integer` | 평균 리드타임 (일) | \- |
| `contactInfo` | `string` | 담당자 연락처 | \- |
| `certifications` | `string[]` | 보유 인증 (ISO, IATF 등) | \- |

- **Backing Datasource:** SRM(공급사 관계 관리) 시스템 + ERP 구매 모듈
- **Status:** `active`

---

### 2-13. Shift (교대/근무조)

> **정의:** 특정 시간대에 특정 인력이 배정된 근무 단위로, 생산 실적과 품질의 시간 귀속 기준이다.

| Property | 타입 | 설명 | 확장 타입 |
| --- | --- | --- | --- |
| `shiftId` | `string` | 교대 고유 식별자 (Primary Key) | \- |
| `plantId` | `string` | 공장 ID (Foreign Key) | \- |
| `shiftType` | `string` | 근무 유형 (`day` / `evening` / `night`) | \- |
| `date` | `date` | 근무 일자 | \- |
| `startTime` | `timestamp` | 시작 시각 | \- |
| `endTime` | `timestamp` | 종료 시각 | \- |
| `headcount` | `integer` | 투입 인원 수 | \- |

- **Backing Datasource:** MES 근무 스케줄 테이블 + HR 근태 시스템
- **Status:** `active`

---

### 2-14. MaintenanceRecord (정비 이력)

> **정의:** 설비에 대해 수행된 개별 정비 활동 기록이다.

| Property | 타입 | 설명 | 확장 타입 |
| --- | --- | --- | --- |
| `recordId` | `string` | 정비 이력 고유 식별자 (Primary Key) | \- |
| `machineId` | `string` | 대상 설비 ID (Foreign Key) | \- |
| `maintenanceType` | `string` | 유형 (`preventive` / `corrective` / `predictive` / `emergency`) | \- |
| `status` | `string` | 상태 (`scheduled` / `in_progress` / `completed` / `deferred`) | Shared Property (ISchedulable) |
| `scheduledAt` | `timestamp` | 예정 일시 | Shared Property (ISchedulable) |
| `completedAt` | `timestamp` | 완료 일시 (nullable) | \- |
| `findings` | `string` | 점검 소견 | Semantic Search |
| `cost` | `double` | 정비 비용 | \- |

- **Backing Datasource:** CMMS(정비 관리 시스템)
- **Status:** `active`
- **Interface:** `ISchedulable` 구현
- **보안 참고:** `cost` 속성은 열 수준 접근 제어 적용 (재무/관리자만 조회 가능)

---

### 2-15. Product (제품)

> **정의:** 공장에서 생산하는 최종 또는 중간 제품으로, BOM과 레시피의 귀속 대상이다.

| Property | 타입 | 설명 | 확장 타입 |
| --- | --- | --- | --- |
| `productId` | `string` | 제품 고유 식별자 (Primary Key) | \- |
| `name` | `string` | 제품명 | \- |
| `category` | `string` | 제품 분류 | \- |
| `bomVersion` | `string` | BOM(Bill of Materials) 버전 | \- |
| `qualityStandard` | `string` | 적용 품질 기준 (규격 코드) | \- |
| `unitCost` | `double` | 단위 원가 | \- |

- **Backing Datasource:** ERP 품목 마스터 + PLM(제품 수명주기 관리) 시스템
- **Status:** `endorsed`
- **보안 참고:** `unitCost` 속성은 열 수준 접근 제어 적용

---

## 3. Link Type 설계

### 3-1. Plant → contains → ProductionLine

| 항목 | 내용 |
| --- | --- |
| **소스 → 관계 → 타겟** | `Plant` → `contains` → `ProductionLine` |
| **카디널리티** | 1:N (하나의 공장이 여러 라인을 보유) |
| **조인 키** | `ProductionLine.plantId = Plant.plantId` |
| **비즈니스 질문** | "수원 1공장에 현재 가동 중인 라인은 몇 개인가?" |

### 3-2. ProductionLine → operates → Machine

| 항목 | 내용 |
| --- | --- |
| **소스 → 관계 → 타겟** | `ProductionLine` → `operates` → `Machine` |
| **카디널리티** | 1:N (하나의 라인에 여러 설비) |
| **조인 키** | `Machine.lineId = ProductionLine.lineId` |
| **비즈니스 질문** | "SMT Line A의 설비 중 현재 fault 상태인 것은?" |

### 3-3. Machine → mountedWith → Tool

| 항목 | 내용 |
| --- | --- |
| **소스 → 관계 → 타겟** | `Machine` → `mountedWith` → `Tool` |
| **카디널리티** | 1:N (하나의 설비에 여러 공구 장착 가능) |
| **조인 키** | `Tool.currentMachineId = Machine.machineId` (Tool 테이블에 현재 장착 설비 FK) |
| **비즈니스 질문** | "CNC-14에 장착된 공구 중 마모율 80% 이상인 것은?" |

### 3-4. Lot → processedOn → Machine

| 항목 | 내용 |
| --- | --- |
| **소스 → 관계 → 타겟** | `Lot` → `processedOn` → `Machine` |
| **카디널리티** | M:N (하나의 로트가 여러 설비를 거칠 수 있고, 하나의 설비가 여러 로트를 처리) |
| **조인 키** | 브릿지 테이블 `lot_machine_map(lotId, machineId, processOrder, startTime, endTime)` |
| **비즈니스 질문** | "불량 Lot L-2024-0312가 어떤 설비들을 거쳤는가?" — **품질 역추적의 핵심** |

### 3-5. Lot → usedRecipe → Recipe

| 항목 | 내용 |
| --- | --- |
| **소스 → 관계 → 타겟** | `Lot` → `usedRecipe` → `Recipe` |
| **카디널리티** | N:1 (여러 로트가 동일 레시피를 사용) |
| **조인 키** | `Lot.recipeId = Recipe.recipeId` |
| **비즈니스 질문** | "레시피 v2.3을 적용한 모든 로트의 평균 불량률은?" |

### 3-6. Lot → usedMaterial → Material

| 항목 | 내용 |
| --- | --- |
| **소스 → 관계 → 타겟** | `Lot` → `usedMaterial` → `Material` |
| **카디널리티** | M:N (하나의 로트에 여러 자재 투입, 하나의 자재가 여러 로트에 사용) |
| **조인 키** | 브릿지 테이블 `lot_material_map(lotId, materialId, quantity, unit)` |
| **비즈니스 질문** | "자재 M-Al-7075 로트의 불량률이 갑자기 올랐는데, 이 자재를 사용한 다른 Lot는?" |

### 3-7. Defect → observedIn → Lot

| 항목 | 내용 |
| --- | --- |
| **소스 → 관계 → 타겟** | `Defect` → `observedIn` → `Lot` |
| **카디널리티** | N:1 (하나의 로트에서 여러 불량 발생 가능) |
| **조인 키** | `Defect.lotId = Lot.lotId` |
| **비즈니스 질문** | "이번 주 surface 불량이 집중된 로트는 어디인가?" — **불량 역추적 체인의 시작점** |

### 3-8. Machine → hasRecord → MaintenanceRecord

| 항목 | 내용 |
| --- | --- |
| **소스 → 관계 → 타겟** | `Machine` → `hasRecord` → `MaintenanceRecord` |
| **카디널리티** | 1:N (하나의 설비에 여러 정비 이력) |
| **조인 키** | `MaintenanceRecord.machineId = Machine.machineId` |
| **비즈니스 질문** | "CNC-14의 최근 6개월 정비 이력과 고장 패턴은?" — **정비 체인의 핵심** |

### 3-9. WorkOrder → targetsMAchine → Machine

| 항목 | 내용 |
| --- | --- |
| **소스 → 관계 → 타겟** | `WorkOrder` → `targetsMachine` → `Machine` |
| **카디널리티** | N:1 (여러 작업 지시가 하나의 설비를 대상으로 가능) |
| **조인 키** | `WorkOrder.targetMachineId = Machine.machineId` |
| **비즈니스 질문** | "현재 대기 중인 설비 정비 작업 지시가 누적된 설비는?" |

### 3-10. Operator → assignedTo → Shift

| 항목 | 내용 |
| --- | --- |
| **소스 → 관계 → 타겟** | `Operator` → `assignedTo` → `Shift` |
| **카디널리티** | M:N (한 작업자가 여러 교대에, 한 교대에 여러 작업자) |
| **조인 키** | 브릿지 테이블 `operator_shift_map(operatorId, shiftId, role)` |
| **비즈니스 질문** | "오늘 야간조에 배정된 작업자 중 CNC 자격 보유자는 몇 명인가?" |

### 3-11. Supplier → provides → Material

| 항목 | 내용 |
| --- | --- |
| **소스 → 관계 → 타겟** | `Supplier` → `provides` → `Material` |
| **카디널리티** | M:N (하나의 공급사가 여러 자재, 하나의 자재를 여러 공급사가 공급) |
| **조인 키** | 브릿지 테이블 `supplier_material_map(supplierId, materialId, unitPrice, leadTime)` |
| **비즈니스 질문** | "품질 점수가 70 미만인 공급사가 납품하는 자재 목록은?" |

### 3-12. Lot → producedBy → Product

| 항목 | 내용 |
| --- | --- |
| **소스 → 관계 → 타겟** | `Lot` → `producedBy` → `Product` |
| **카디널리티** | N:1 (여러 로트가 하나의 제품을 생산) |
| **조인 키** | `Lot.productId = Product.productId` |
| **비즈니스 질문** | "제품 X의 최근 30일 로트별 불량률 추이는?" |

### 3-13. Recipe → forProduct → Product

| 항목 | 내용 |
| --- | --- |
| **소스 → 관계 → 타겟** | `Recipe` → `forProduct` → `Product` |
| **카디널리티** | N:1 (하나의 제품에 여러 버전의 레시피 존재 가능) |
| **조인 키** | `Recipe.productId = Product.productId` |
| **비즈니스 질문** | "제품 Y의 현재 active 레시피는 어떤 버전인가?" |

### 3-14. Measurement → takenFrom → Machine

| 항목 | 내용 |
| --- | --- |
| **소스 → 관계 → 타겟** | `Measurement` → `takenFrom` → `Machine` |
| **카디널리티** | N:1 (하나의 설비에서 다수 측정 발생) |
| **조인 키** | `Measurement.machineId = Machine.machineId` |
| **비즈니스 질문** | "CNC-14에서 최근 24시간 온도 규격 초과 측정이 몇 건인가?" |

### 핵심 관계 체인 요약

```
■ 품질 추적 체인 (불량 → 원인):
  Defect → [observedIn] → Lot → [processedOn] → Machine → [mountedWith] → Tool
                             └→ [usedRecipe]  → Recipe
                             └→ [usedMaterial] → Material → [provides] ← Supplier

■ 정비 체인 (설비 → 이력 → 작업):
  Machine → [hasRecord] → MaintenanceRecord
  Machine ← [targetsMachine] ← WorkOrder

■ 불량 역추적 체인 (불량 → 설비 → 라인 → 공장):
  Defect → [observedIn] → Lot → [processedOn] → Machine
                                                    → [operates] ← ProductionLine
                                                                      → [contains] ← Plant

■ 인력-생산 체인:
  Operator → [assignedTo] → Shift
  WorkOrder → [assignedTo] → Operator
```

---

## 4. Action Type 설계

### 4-1. EscalateDefect (불량 에스컬레이션)

> **목적:** 탐지된 불량을 상위 심각도로 격상하고, 전담 분석가를 배정하여 조사를 개시한다.

**입력 파라미터:**

| 파라미터 | 타입 | 필수 | 설명 |
| --- | --- | --- | --- |
| `defectId` | `Defect` 객체 참조 | 필수 | 에스컬레이션 대상 불량 |
| `escalationReason` | `string` | 필수 | 에스컬레이션 사유 (최소 20자) |
| `assignedAnalyst` | `Operator` 객체 참조 | 필수 | 배정할 분석 담당자 |
| `severity` | `enum(low, medium, high, critical)` | 필수 | 에스컬레이션 심각도 |

**유효성 검사 규칙:**

1. `defect.status == "open"` — 이미 처리 중이거나 종료된 불량은 에스컬레이션 불가
2. `len(escalationReason) >= 20` — 사유가 충분히 상세해야 함
3. `assignedAnalyst.role in ["qa_inspector", "shift_leader"]` — 분석 역할 보유자만 배정 가능
4. `assignedAnalyst.plantId == defect→lot→machine→line→plant.plantId` — 동일 공장 소속만 배정

**변경 내용:**

- `Defect.status` → `"escalated"`
- `Defect.assignedTo` → `assignedAnalyst.operatorId`
- `Defect.escalatedAt` → `now()`
- `Defect.severity` → 입력된 severity

**부수 효과:**

- 알림 시스템 API 호출 (이메일/메신저로 담당자 및 라인 관리자 통보)
- 감사 로그 자동 기록 (실행자, 시각, 사유, 파라미터 전체)
- `severity == "critical"` 시 JIRA/이슈 트래커 티켓 자동 생성

**실행 모드:** 즉시 실행 (Direct) **승인 워크플로우:** `severity == "critical"` → 품질 관리자(QA Manager) 승인 필요

---

### 4-2. CreateWorkOrder (작업 지시 생성)

> **목적:** 생산, 정비, 검사, 재작업을 위한 작업 지시서를 발행한다.

**입력 파라미터:**

| 파라미터 | 타입 | 필수 | 설명 |
| --- | --- | --- | --- |
| `orderType` | `enum(production, maintenance, inspection, rework)` | 필수 | 작업 유형 |
| `targetMachineId` | `Machine` 객체 참조 | 필수 | 대상 설비 |
| `priority` | `enum(low, medium, high, urgent)` | 필수 | 우선순위 |
| `scheduledAt` | `timestamp` | 필수 | 예정 일시 |
| `assignedTo` | `Operator` 객체 참조 | 선택 | 담당자 (미지정 시 교대 리더가 배정) |
| `description` | `string` | 필수 | 작업 내용 (최소 10자) |

**유효성 검사 규칙:**

1. `targetMachine.status != "retired"` — 퇴역 설비에 작업 지시 불가
2. `scheduledAt > now()` — 과거 일시로 생성 불가
3. `orderType == "maintenance"` 시 동일 설비에 미완료 정비 작업 지시가 없어야 함 (중복 방지)
4. `priority == "urgent"` 시 `scheduledAt`이 24시간 이내여야 함

**변경 내용:**

- 새 `WorkOrder` 객체 생성 (`status = "planned"`)
- `assignedTo` 지정 시 해당 `Operator`에 작업 연결

**부수 효과:**

- 담당자에게 알림 발송
- `priority == "urgent"` 시 교대 리더에게 즉시 푸시 알림
- 감사 로그 기록

**실행 모드:** 즉시 실행 (Direct) **승인 워크플로우:** `orderType == "maintenance" && priority == "urgent"` → 생산 관리자 승인 필요 (생산 일정 영향 검토)

---

### 4-3. QuarantineLot (Lot 격리)

> **목적:** 품질 이상이 의심되는 Lot를 격리하여 출하를 차단하고, 추가 검사를 지시한다.

**입력 파라미터:**

| 파라미터 | 타입 | 필수 | 설명 |
| --- | --- | --- | --- |
| `lotId` | `Lot` 객체 참조 | 필수 | 격리 대상 로트 |
| `quarantineReason` | `string` | 필수 | 격리 사유 (최소 15자) |
| `relatedDefectIds` | `Defect[]` 객체 참조 | 선택 | 관련 불량 건 목록 |
| `inspectionRequired` | `boolean` | 필수 | 추가 검사 필요 여부 |

**유효성 검사 규칙:**

1. `lot.status in ["in_progress", "completed"]` — 이미 격리/폐기된 로트는 재격리 불가
2. `lot.quantity > 0` — 빈 로트는 격리 대상 아님
3. 격리 실행자는 `qa_inspector` 또는 `shift_leader` 역할이어야 함

**변경 내용:**

- `Lot.status` → `"quarantined"`
- `Lot.quarantinedAt` → `now()`
- `Lot.quarantineReason` → 입력된 사유

**부수 효과:**

- WMS(창고 관리 시스템)에 출하 차단 API 호출
- `inspectionRequired == true` 시 검사 WorkOrder 자동 생성 (CreateWorkOrder 체인 호출)
- 품질팀 및 물류팀에 동시 알림
- 감사 로그 기록

**실행 모드:** 즉시 실행 (Direct) **승인 워크플로우:** 없음 (품질 안전은 즉시 조치 원칙)

---

### 4-4. ChangeRecipe (레시피/공정 조건 변경)

> **목적:** 공정 파라미터(온도, 압력, 속도 등)를 변경하고, 변경 전후 품질 영향을 추적한다.

**입력 파라미터:**

| 파라미터 | 타입 | 필수 | 설명 |
| --- | --- | --- | --- |
| `recipeId` | `Recipe` 객체 참조 | 필수 | 변경 대상 레시피 |
| `newParameters` | `string (JSON)` | 필수 | 변경할 파라미터 세트 |
| `changeReason` | `string` | 필수 | 변경 사유 (최소 30자) |
| `newVersion` | `string` | 필수 | 새 버전 번호 (SemVer) |
| `effectiveFrom` | `timestamp` | 필수 | 적용 시작 일시 |

**유효성 검사 규칙:**

1. `recipe.status == "active"` — active 상태의 레시피만 변경 가능
2. `newVersion > recipe.version` — 버전은 반드시 상위여야 함
3. `newParameters`의 각 값이 해당 파라미터의 물리적 허용 범위 내 — `assessProcessRisk()` Function 호출로 검증
4. `effectiveFrom >= now() + 2h` — 최소 2시간 전 사전 등록 (현장 준비 시간 확보)
5. 현재 해당 레시피를 사용 중인 `in_progress` 로트가 없어야 함

**변경 내용:**

- 기존 `Recipe.status` → `"deprecated"`
- 새 `Recipe` 객체 생성 (신규 버전, `status = "approved"`)
- `effectiveFrom` 도달 시 자동으로 `status → "active"`

**부수 효과:**

- 공정 엔지니어 전원에게 변경 공지
- 영향받는 ProductionLine의 교대 리더에게 알림
- 변경 전후 비교 보고서 자동 생성 예약
- 감사 로그 기록 (변경 전/후 파라미터 전체 diff 포함)

**실행 모드:** 검토 후 실행 (Staged) **승인 워크플로우:** 공정 엔지니어 제안 → 품질 관리자 검토 → 생산 관리자 최종 승인 (3단계)

---

### 4-5. RequestMaintenance (정비 요청)

> **목적:** 설비 이상 징후 또는 예방 정비 일정에 따라 정비를 요청하고 기록한다.

**입력 파라미터:**

| 파라미터 | 타입 | 필수 | 설명 |
| --- | --- | --- | --- |
| `machineId` | `Machine` 객체 참조 | 필수 | 정비 대상 설비 |
| `maintenanceType` | `enum(preventive, corrective, predictive, emergency)` | 필수 | 정비 유형 |
| `symptomDescription` | `string` | 필수 | 증상/요청 사유 (최소 15자) |
| `requestedBy` | `Operator` 객체 참조 | 필수 | 요청자 |
| `urgency` | `enum(normal, high, emergency)` | 필수 | 긴급도 |

**유효성 검사 규칙:**

1. `machine.status != "retired"` — 퇴역 설비 정비 요청 불가
2. `maintenanceType == "emergency"` 시 `urgency`는 반드시 `"emergency"` — 일관성 강제
3. 동일 설비에 미완료 `emergency` 정비 요청이 이미 있으면 중복 생성 차단
4. `requestedBy`가 해당 설비의 공장 소속이어야 함

**변경 내용:**

- 새 `MaintenanceRecord` 객체 생성 (`status = "scheduled"`)
- `urgency == "emergency"` 시 `Machine.status` → `"maintenance"` 즉시 전환
- 정비 WorkOrder 자동 생성 (CreateWorkOrder 체인 호출)

**부수 효과:**

- CMMS에 정비 요청 API 호출
- `urgency == "emergency"` 시 해당 라인 관리자 및 정비팀 리더에게 즉시 알림
- 감사 로그 기록

**실행 모드:** `emergency` → 즉시 실행 / 그 외 → 자동화 실행 (정비 스케줄러가 일정 배정) **승인 워크플로우:** `emergency` 정비로 인해 라인 중단이 필요한 경우 → 생산 관리자 승인 필요

---

### 4-6. AssignOperator (작업자 배정)

> **목적:** 특정 작업 지시 또는 설비에 작업자를 배정하고, 자격 요건을 자동 검증한다.

**입력 파라미터:**

| 파라미터 | 타입 | 필수 | 설명 |
| --- | --- | --- | --- |
| `workOrderId` | `WorkOrder` 객체 참조 | 필수 | 배정 대상 작업 지시 |
| `operatorId` | `Operator` 객체 참조 | 필수 | 배정할 작업자 |
| `role` | `string` | 선택 | 작업 역할 (미지정 시 기본 역할) |

**유효성 검사 규칙:**

1. `workOrder.status in ["planned", "scheduled"]` — 이미 진행 중이거나 완료된 작업에는 배정 변경 불가
2. `operator`가 해당 시간대의 Shift에 배정되어 있어야 함
3. `workOrder.orderType == "maintenance"` 시 `operator.certifications`에 해당 설비 유형 자격이 포함되어야 함
4. `operator`의 동시간대 배정 작업 수가 최대 허용치(configurable, 기본 2)를 초과하지 않아야 함

**변경 내용:**

- `WorkOrder.assignedTo` → `operatorId`
- `WorkOrder.status` → `"scheduled"` (planned 상태였다면)

**부수 효과:**

- 배정된 작업자에게 모바일 알림
- 교대 리더의 인력 배치 현황 대시보드 자동 갱신
- 감사 로그 기록

**실행 모드:** 즉시 실행 (Direct) **승인 워크플로우:** 없음 (교대 리더 권한으로 즉시 배정)

---

### 4-7. UpdateMachineStatus (설비 상태 변경)

> **목적:** 설비의 운영 상태를 공식적으로 전환하고, 상태 변경에 따른 후속 조치를 트리거한다.

**입력 파라미터:**

| 파라미터 | 타입 | 필수 | 설명 |
| --- | --- | --- | --- |
| `machineId` | `Machine` 객체 참조 | 필수 | 대상 설비 |
| `newStatus` | `enum(active, maintenance, fault, idle)` | 필수 | 새 상태 |
| `reason` | `string` | 필수 | 상태 변경 사유 (최소 10자) |
| `estimatedResolution` | `timestamp` | 선택 | 정상화 예상 시각 (fault/maintenance 시) |

**유효성 검사 규칙:**

1. `newStatus != machine.currentStatus` — 동일 상태로 변경 불가
2. 상태 전이 규칙 준수:
   - `active` → `maintenance`, `fault`, `idle` 가능
   - `maintenance` → `active`, `fault` 가능
   - `fault` → `maintenance` 가능 (`fault` → `active` 직접 전이 불가, 반드시 정비를 거쳐야 함)
   - `idle` → `active` 가능
3. `newStatus == "active"` 전환 시 미완료 `emergency` MaintenanceRecord가 없어야 함

**변경 내용:**

- `Machine.status` → `newStatus`
- `Machine.statusChangedAt` → `now()`

**부수 효과:**

- `newStatus == "fault"` 시:
  - 해당 설비의 진행 중인 모든 Lot에 경고 플래그 설정
  - 라인 관리자 및 정비팀에 즉시 알림
  - 자동 RequestMaintenance(emergency) 체인 실행 제안
- `newStatus == "active"` 시:
  - 대기 중인 WorkOrder 실행 가능 상태로 전환
- 감사 로그 기록 (이전 상태 → 새 상태, 사유)

**실행 모드:** 즉시 실행 (Direct) **승인 워크플로우:** `fault → maintenance → active` 복구 경로 시 정비 완료 확인자(정비 리더)의 서명 필요

---

## 5. Function 설계

### 5-1. calculateMachineHealthScore — 설비 건강 점수

> **목적:** 설비의 센서 데이터, 정비 이력, 불량률, 공구 상태를 종합하여 0\~100 건강 점수를 산출한다.

**입력/출력 시그니처:**

```
Input:  machine: Machine
Output: { score: number (0~100), factors: FactorBreakdown[], riskLevel: string }
```

**의사코드:**

```
function calculateMachineHealthScore(machine):
    // 1. 정비 주기 점수 (30% 가중치)
    daysSinceLastMaintenance = daysDiff(machine.lastMaintenanceDate, now())
    expectedInterval = getMaintenanceCycle(machine.machineType)
    maintenanceScore = max(0, 100 - (daysSinceLastMaintenance / expectedInterval) * 100)

    // 2. 센서 이상 점수 (30% 가중치)
    recentMeasurements = machine → [takenFrom] ← Measurement
                          .filter(last24Hours)
    outOfSpecRate = count(m.isOutOfSpec == true) / total
    sensorScore = (1 - outOfSpecRate) * 100

    // 3. 불량률 점수 (20% 가중치)
    recentLots = machine → [processedOn] ← Lot.filter(last7Days)
    avgDefectRate = average(lot.defectRate for lot in recentLots)
    defectScore = (1 - avgDefectRate) * 100

    // 4. 공구 마모 점수 (20% 가중치)
    tools = machine → [mountedWith] → Tool
    maxWear = max(tool.wearPercentage for tool in tools)
    toolScore = (1 - maxWear / 100) * 100

    // 종합
    finalScore = maintenanceScore * 0.3
               + sensorScore * 0.3
               + defectScore * 0.2
               + toolScore * 0.2

    riskLevel = "critical" if finalScore < 40
                "warning"  if finalScore < 70
                "normal"   otherwise

    return { score: finalScore, factors: [...], riskLevel }
```

**활용처:**

- Workshop 대시보드: 설비 건강 히트맵 표시
- AIP Logic: "가장 위험한 설비 5대"질문 응답 시 호출
- Action 유효성 검사: 건강 점수 40 미만 설비에 신규 Lot 배정 경고
- 자동화: 점수 40 미만 시 자동 RequestMaintenance 트리거

---

### 5-2. calculateDefectContribution — 불량 기여 요인 분석

> **목적:** 특정 불량에 대해 설비, 공구, 레시피, 자재, 작업자 각각의 기여도를 백분율로 산출한다.

**입력/출력 시그니처:**

```
Input:  defect: Defect
Output: { contributions: { factor: string, percentage: number, evidence: string }[] }
```

**의사코드:**

```
function calculateDefectContribution(defect):
    lot = defect → [observedIn] → Lot
    machine = lot → [processedOn] → Machine
    tools = machine → [mountedWith] → Tool
    recipe = lot → [usedRecipe] → Recipe
    materials = lot → [usedMaterial] → Material[]

    // 설비 기여도: 동일 설비의 최근 유사 불량 빈도
    machineDefectHistory = machine의 최근 30일 동일 유형 불량 건수
    machineContrib = normalize(machineDefectHistory)

    // 공구 기여도: 마모율과 불량 상관관계
    toolWearCorrelation = correlate(tools.wearPercentage, defectRate)
    toolContrib = normalize(toolWearCorrelation)

    // 레시피 기여도: 동일 레시피 다른 설비에서의 불량률 비교
    recipeBaseline = recipe를 사용한 다른 설비의 평균 불량률
    recipeContrib = normalize(deviation from baseline)

    // 자재 기여도: 동일 자재 배치의 다른 로트 불량률
    materialBaseline = 동일 자재를 사용한 다른 로트 불량률
    materialContrib = normalize(deviation from baseline)

    // 작업자 기여도: 동일 작업자의 최근 불량 패턴
    operatorHistory = 해당 Shift 작업자의 동일 유형 불량 빈도
    operatorContrib = normalize(operatorHistory)

    return sorted contributions by percentage descending
```

**활용처:**

- Defect 상세 화면: 근본 원인 분석(RCA) 지원
- AIP Logic: "이 불량의 원인이 뭐야?" 질문 응답
- EscalateDefect Action: 에스컬레이션 시 자동 기여도 분석 첨부

---

### 5-3. assessProcessRisk — 공정 리스크 평가

> **목적:** 레시피 변경, 자재 교체 등 공정 조건 변경 시 품질에 미칠 리스크를 사전 평가한다.

**입력/출력 시그니처:**

```
Input:  recipe: Recipe, proposedChanges: ParameterChangeSet
Output: { riskScore: number (0~100), riskFactors: RiskFactor[], recommendation: string }
```

**의사코드:**

```
function assessProcessRisk(recipe, proposedChanges):
    // 1. 파라미터 민감도 분석
    for each param in proposedChanges:
        historicalVariance = 과거 해당 파라미터 변경 시 불량률 변동 분석
        sensitivity = calculateSensitivity(param, historicalVariance)

    // 2. 유사 변경 이력 조회
    similarChanges = Recipe 변경 이력 중 유사 파라미터 변경 건 검색
    historicalOutcome = 유사 변경 후 불량률 변화 통계

    // 3. 현재 설비 상태 반영
    affectedMachines = recipe를 사용하는 Machine 목록
    machineHealthScores = [calculateMachineHealthScore(m) for m in affectedMachines]
    equipmentRisk = 건강 점수 낮은 설비 비율

    // 4. 자재 호환성 검증
    currentMaterials = recipe와 연결된 Material 목록
    materialCompatibility = 신규 파라미터와 자재 스펙 호환성 체크

    riskScore = weighted_sum(sensitivity, historicalOutcome, equipmentRisk, materialCompatibility)

    recommendation = "승인 권장" if riskScore < 30
                     "시범 생산 후 판단" if riskScore < 60
                     "변경 보류 권장" if riskScore >= 60

    return { riskScore, riskFactors, recommendation }
```

**활용처:**

- ChangeRecipe Action: 유효성 검사 단계에서 자동 호출
- Workshop: 레시피 변경 시뮬레이션 화면
- AIP Logic: "이 공정 조건 바꾸면 어떻게 되나?" 질문 응답

---

### 5-4. predictToolWear — 공구 마모 예측

> **목적:** 현재 마모율과 사용 패턴을 기반으로 공구 잔여 수명을 예측한다.

**입력/출력 시그니처:**

```
Input:  tool: Tool
Output: { remainingLife: number (%), estimatedReplacementDate: timestamp, confidence: number }
```

**의사코드:**

```
function predictToolWear(tool):
    // 1. 마모 이력 추세선 계산
    wearHistory = tool.wearPercentage.history(last30Days)
    wearTrend = linearRegression(wearHistory)  // 일일 마모율

    // 2. 현재 가공 조건 반영 (레시피 난이도)
    machine = tool이 장착된 Machine
    currentRecipe = machine에서 사용 중인 Recipe
    difficultyFactor = recipeDifficulty(currentRecipe)  // 경질재 가공 등

    // 3. 잔여 수명 계산
    adjustedWearRate = wearTrend.slope * difficultyFactor
    remainingLife = (100 - tool.wearPercentage) / adjustedWearRate  // 일 단위

    estimatedReplacementDate = now() + remainingLife days
    confidence = rSquared(wearHistory)  // 추세선 신뢰도

    return { remainingLife, estimatedReplacementDate, confidence }
```

**활용처:**

- calculateMachineHealthScore 내부에서 호출 (공구 마모 점수 계산)
- Workshop: 공구 교체 일정 캘린더
- 자동화: 잔여 수명 3일 미만 시 자동 알림

---

### 5-5. calculateLotSensitivity — Lot 민감도 계산

> **목적:** 특정 Lot의 공정 조건 민감도를 평가하여, 조건 변동에 의한 품질 영향 가능성을 산출한다.

**입력/출력 시그니처:**

```
Input:  lot: Lot
Output: { sensitivityScore: number (0~100), criticalParams: string[], safeMargin: number }
```

**의사코드:**

```
function calculateLotSensitivity(lot):
    recipe = lot → [usedRecipe] → Recipe
    materials = lot → [usedMaterial] → Material[]
    machine = lot → [processedOn] → Machine

    // 1. 레시피 공차 여유도
    for each param in recipe.parameters:
        actualValue = machine의 실시간 측정값
        tolerance = recipe.tolerances[param]
        margin = (tolerance.max - actualValue) / (tolerance.max - tolerance.min)
        // margin이 작을수록 민감

    // 2. 자재 품질 변동성
    materialVariance = 각 자재의 최근 입고 검사 편차

    // 3. 설비 안정성
    machineStability = 최근 1시간 센서 값의 표준편차

    sensitivityScore = weighted_sum(1 - avgMargin, materialVariance, machineStability)
    criticalParams = margin이 20% 미만인 파라미터 목록
    safeMargin = min(all margins)

    return { sensitivityScore, criticalParams, safeMargin }
```

**활용처:**

- AIP Logic: "현재 진행 중인 Lot 중 가장 주의가 필요한 것은?" 질문 응답
- QuarantineLot Action: 격리 우선순위 판단 보조
- Workshop: 실시간 Lot 민감도 모니터링 패널

---

### 5-6. evaluateSupplierQuality — 공급사 품질 평가

> **목적:** 공급사의 납품 자재 품질, 납기 준수, 불량 기여도를 종합 평가한다.

**입력/출력 시그니처:**

```
Input:  supplier: Supplier, evaluationPeriod: DateRange
Output: { qualityScore: number (0~100), dimensions: DimensionScore[], trend: string }
```

**의사코드:**

```
function evaluateSupplierQuality(supplier, period):
    materials = supplier → [provides] → Material[]

    // 1. 입고 검사 합격률 (40%)
    incomingInspections = materials의 입고 검사 결과 (period 내)
    passRate = 합격 건수 / 전체 건수

    // 2. 공정 불량 기여율 (30%)
    lots = materials → [usedMaterial] ← Lot (period 내)
    defects = lots → [observedIn] ← Defect
    materialRelatedDefects = defects 중 자재 기여도가 30% 이상인 건
    defectContribRate = materialRelatedDefects / total defects

    // 3. 납기 준수율 (20%)
    deliveries = 해당 공급사의 납품 이력 (period 내)
    onTimeRate = 납기 내 도착 건수 / 전체 납품 건수

    // 4. 이슈 대응 속도 (10%)
    issues = 공급사 관련 이슈 이력
    avgResolutionTime = 평균 이슈 해결 소요 시간

    qualityScore = passRate * 40 + (1 - defectContribRate) * 30
                 + onTimeRate * 20 + responseScore * 10

    trend = compare(current period score, previous period score)

    return { qualityScore, dimensions, trend }
```

**활용처:**

- Supplier 상세 화면: 품질 트렌드 차트
- AIP Logic: "품질이 가장 안 좋은 공급사 3곳과 이유는?" 질문 응답
- 구매팀: 공급사 선정/갱신 의사결정 지원

---

### 5-7. estimateDowntimeImpact — 다운타임 영향도 산출

> **목적:** 특정 설비가 다운될 경우 생산 일정, 납기, 비용에 미치는 영향을 예측한다.

**입력/출력 시그니처:**

```
Input:  machine: Machine, estimatedDowntime: Duration
Output: {
    productionLoss: number (단위: 제품 수),
    affectedWorkOrders: WorkOrder[],
    costImpact: number (원),
    deliveryDelayRisk: { orderId: string, delayDays: number }[]
}
```

**의사코드:**

```
function estimateDowntimeImpact(machine, estimatedDowntime):
    line = machine → [operates] ← ProductionLine

    // 1. 생산 손실 계산
    taktTime = line.taktTime
    productionLoss = estimatedDowntime / taktTime  // 예상 미생산 수량

    // 2. 영향받는 작업 지시
    affectedWOs = machine ← [targetsMachine] ← WorkOrder
                   .filter(status in ["scheduled", "in_progress"])
                   .filter(scheduledAt within downtime period)

    // 3. 비용 영향
    product = line에서 생산 중인 Product
    directCost = productionLoss * product.unitCost
    overtimeCost = 대체 라인 가동 시 추가 인건비 추정
    costImpact = directCost + overtimeCost

    // 4. 납기 지연 리스크
    for each wo in affectedWOs:
        if wo has linked customer order:
            originalDelivery = customer order due date
            adjustedDelivery = recalculate with downtime
            delayDays = adjustedDelivery - originalDelivery

    return { productionLoss, affectedWorkOrders, costImpact, deliveryDelayRisk }
```

**활용처:**

- RequestMaintenance Action: 정비 긴급도 판단 시 다운타임 비용 참고
- AIP Logic: "이 설비 지금 세우면 영향이 어떻게 되나?" 질문 응답
- 생산 관리자: 정비 일정 최적화 의사결정

---

## 6. Interface 설계

### 6-1. IAsset (자산 인터페이스)

> **목적:** Machine, Tool 등 물리적 자산의 공통 형태를 정의하여, 자산 관리 화면과 쿼리에서 다형적 접근을 가능하게 한다.

```
interface IAsset {
    assetId: string          // 자산 고유 식별자
    status: string           // 운영 상태
    location: geopoint       // 현재 위치
    lastInspectedAt: timestamp  // 최종 점검 일시
}

구현하는 Object Type:
  - Machine implements IAsset
      machineId → assetId, status → status, (line의 위치) → location, lastMaintenanceDate → lastInspectedAt
  - Tool implements IAsset
      toolId → assetId, status → status, (장착 설비의 위치) → location, installedAt → lastInspectedAt
```

**활용 예시:**

- "현재 위치에서 반경 500m 내 점검 필요한 자산 전체" — 단일 IAsset 쿼리로 Machine과 Tool을 동시에 검색
- 자산 총 현황 대시보드에서 Machine/Tool을 통합 표시

---

### 6-2. ITrackable (추적 인터페이스)

> **목적:** Lot, Material 등 추적이 필요한 객체의 공통 형태를 정의하여, 이력 추적과 위치 조회를 통합한다.

```
interface ITrackable {
    trackingId: string        // 추적 식별자
    currentLocation: string   // 현재 위치 (공정 단계 또는 창고 위치)
    history: Event[]          // 이동/상태 변경 이력
}

구현하는 Object Type:
  - Lot implements ITrackable
      lotId → trackingId, 현재 공정 단계 → currentLocation, 상태 변경 로그 → history
  - Material implements ITrackable
      trackingId → trackingId, 현재 창고 위치 → currentLocation, 입출고 이력 → history
```

**활용 예시:**

- "이 추적 ID의 현재 위치와 이동 경로는?" — ITrackable 인터페이스로 Lot과 Material을 동일하게 추적
- 리콜 상황 시 영향 범위 파악: ITrackable 기반 역추적

---

### 6-3. ISchedulable (일정 관리 인터페이스)

> **목적:** WorkOrder, MaintenanceRecord 등 일정 관리가 필요한 객체의 공통 형태를 정의한다.

```
interface ISchedulable {
    scheduledAt: timestamp    // 예정 일시
    assignedTo: string        // 담당자 ID
    priority: string          // 우선순위 (low / medium / high / urgent)
    status: string            // 상태 (scheduled / in_progress / completed / deferred)
}

구현하는 Object Type:
  - WorkOrder implements ISchedulable
  - MaintenanceRecord implements ISchedulable
```

**활용 예시:**

- "내일까지 예정된 모든 일정(작업 지시 + 정비)을 우선순위 순으로 보여줘" — ISchedulable 단일 쿼리
- 교대 리더의 통합 일정 대시보드: WorkOrder와 MaintenanceRecord를 하나의 타임라인에 표시

---

### 6-4. Interface 설계 원칙

| 원칙 | 설명 |
| --- | --- |
| 최소 공통 속성 | Interface는 모든 구현 Object가 실제로 가지는 속성만 포함 |
| 비즈니스 의미 기반 | 기술적 편의가 아닌 업무적 공통성이 있을 때만 Interface 생성 |
| 독립적 구현 가능 | Interface 속성은 각 Object의 기존 속성으로 매핑 가능해야 함 |
| 다중 구현 허용 | 하나의 Object가 여러 Interface를 구현 가능 (예: Tool은 IAsset이자 미래에 ITrackable도 가능) |

---

## 7. 보안 모델 설계

### 7-1. 행 수준 접근 제어 (Row-Level Security)

**원칙:** 사용자는 자신이 권한을 가진 공장(Plant)의 데이터만 조회/변경할 수 있다.

```
Granular Policy: "담당 공장 데이터 접근"

규칙:
  object.plantId in current_user.authorizedPlantIds

적용 경로:
  - Plant 직접 조회 시: Plant.plantId 필터
  - ProductionLine 조회 시: ProductionLine.plantId 필터
  - Machine 조회 시: Machine → ProductionLine → Plant.plantId 경로로 필터
  - Lot 조회 시: Lot → Machine → ProductionLine → Plant.plantId 경로로 필터
  - 이하 모든 Object에 동일 원칙 적용

예시:
  수원 공장 담당자 → authorizedPlantIds: ["PLT-SUWON"]
    → 수원 공장의 설비, 로트, 불량, 정비 이력만 조회 가능
  글로벌 품질 관리자 → authorizedPlantIds: ["PLT-SUWON", "PLT-HWASEONG", "PLT-VIETNAM"]
    → 전체 공장 데이터 조회 가능
```

**중요:** 이 정책은 Workshop 대시보드, OSDK 기반 앱, AIP 에이전트 등 모든 접근 채널에 동일하게 적용된다. AI 에이전트도 사용자의 권한 범위 내에서만 데이터를 조회한다.

### 7-2. 열 수준 접근 제어 (Column-Level Security)

**원칙:** 민감한 속성은 해당 역할의 사용자만 조회할 수 있다.

| 제한 속성 | 소속 Object | 접근 가능 역할 | 제한 이유 |
| --- | --- | --- | --- |
| `MaintenanceRecord.cost` | MaintenanceRecord | 재무팀, 공장장 | 정비 비용 정보 |
| `Product.unitCost` | Product | 재무팀, 생산 관리자 | 원가 정보 |
| `Supplier.contractTerms` (확장 시) | Supplier | 구매팀, 법무팀 | 계약 조건 |
| `Operator.personalInfo` (확장 시) | Operator | HR팀 | 개인정보 |

```
Column Policy 예시:
  정책명: "원가 정보 접근 제한"
  대상 속성: Product.unitCost, MaintenanceRecord.cost
  규칙: current_user.role in ["finance", "plant_manager", "executive"]
  비권한 사용자: 해당 속성이 null로 표시됨 (존재 자체는 알 수 있으나 값 비공개)
```

### 7-3. Action 권한 매트릭스

각 역할이 실행할 수 있는 Action을 명시적으로 정의한다.

| Action \\ 역할 | Operator | Shift Leader | QA Inspector | Process Engineer | Production Manager | Plant Manager |
| --- | --- | --- | --- | --- | --- | --- |
| EscalateDefect | \- | 실행 | 실행 | 실행 | 실행+승인 | 실행+승인 |
| CreateWorkOrder | \- | 실행 | 실행(inspection만) | 실행 | 실행 | 실행 |
| QuarantineLot | \- | 실행 | 실행 | \- | 실행 | 실행 |
| ChangeRecipe | \- | \- | \- | 제안 | 검토 | 최종 승인 |
| RequestMaintenance | 실행(normal) | 실행 | \- | 실행 | 실행 | 실행 |
| AssignOperator | \- | 실행 | \- | \- | 실행 | 실행 |
| UpdateMachineStatus | 실행(idle↔active) | 실행 | \- | 실행 | 실행 | 실행 |

**범례:** `-` 권한 없음 / `실행` 직접 실행 가능 / `제안` Staged 모드로 제안만 가능 / `검토` 제안 검토 가능 / `승인` 최종 승인 가능 / `실행+승인` 실행 및 다른 사용자의 승인 요청 처리 가능

### 7-4. AI 에이전트 보안 원칙

```
1. 동일 정책 적용: AIP 에이전트가 온톨로지에 접근할 때
   인간 사용자와 동일한 보안 정책(행/열/Action)이 적용된다.

2. 대리 실행 원칙: 에이전트는 요청한 사용자의 권한 범위 내에서만 동작한다.
   Operator가 에이전트에게 질문하면, 에이전트도 Operator 권한으로만 데이터 조회.

3. Action 실행 제한: 에이전트가 Action을 실행할 때도
   해당 사용자의 Action 권한 매트릭스가 적용된다.
   승인이 필요한 Action은 에이전트가 "제안"만 하고, 인간이 승인한다.

4. 감사 추적: 에이전트의 모든 조회/실행은 별도 태그로 감사 로그에 기록된다.
   (actor: "user:OPR-001", via: "agent:quality-assistant")
```

---

## 8. 전체 온톨로지 그래프 시각화

```
                                    ┌──────────┐
                                    │  Plant   │
                                    │ (공장)   │
                                    └────┬─────┘
                                         │ contains (1:N)
                                         ▼
                              ┌──────────────────┐
                              │ ProductionLine   │
                              │ (생산 라인)       │
                              └────────┬─────────┘
                                       │ operates (1:N)
                                       ▼
┌──────────┐  provides   ┌──────────┐     mountedWith    ┌──────────┐
│ Supplier │────(M:N)───→│ Material │     (1:N)          │   Tool   │
│ (공급사) │             │ (자재)   │         ┌──────────→│ (공구)   │
└──────────┘             └────┬─────┘         │           └──────────┘
                              │               │
                              │ usedMaterial   │
                              │ (M:N)         │
                              ▼               │
┌──────────┐  forProduct ┌────┴─────┐  processedOn  ┌──────────────┐
│ Product  │←──(N:1)─────│ Recipe   │    (M:N)      │   Machine    │
│ (제품)   │             │ (레시피) │         ┌─────→│   (설비)     │
└────┬─────┘             └────┬─────┘         │      └──┬───┬───┬──┘
     │                        │               │         │   │   │
     │ producedBy             │ usedRecipe    │         │   │   │
     │ (N:1)                  │ (N:1)         │         │   │   │
     ▼                        ▼               │         │   │   │
┌──────────┐                                  │         │   │   │
│   Lot    │◄─────────────────────────────────┘         │   │   │
│ (로트)   │                                            │   │   │
└──┬───────┘                                            │   │   │
   │                                                    │   │   │
   │ observedIn (N:1)              hasRecord (1:N) ─────┘   │   │
   ▼                               ▼                        │   │
┌──────────┐              ┌──────────────────┐              │   │
│  Defect  │              │MaintenanceRecord │              │   │
│ (불량)   │              │ (정비 이력)       │              │   │
└──────────┘              └──────────────────┘              │   │
                                                            │   │
                          targetsMachine (N:1) ─────────────┘   │
                          ▼                                     │
                   ┌──────────────┐              takenFrom (N:1)│
                   │  WorkOrder   │                             │
                   │ (작업 지시)   │                             │
                   └──────────────┘                             │
                                                                ▼
                   ┌──────────────┐                    ┌──────────────┐
                   │  Operator    │                    │ Measurement  │
                   │ (작업자)     │                    │ (측정값)      │
                   └──────┬───────┘                    └──────────────┘
                          │
                          │ assignedTo (M:N)
                          ▼
                   ┌──────────────┐
                   │    Shift     │
                   │ (교대/근무조) │
                   └──────────────┘


═══════════════════════════════════════════════════════════════
                    핵심 추적 경로 (Critical Trace Paths)
═══════════════════════════════════════════════════════════════

■ 품질 역추적 (Root Cause Analysis):
  Defect → Lot → Machine → Tool
               → Recipe
               → Material → Supplier

■ 정비 영향 분석 (Maintenance Impact):
  Machine → MaintenanceRecord
         → WorkOrder
         → ProductionLine → Plant

■ 생산 계보 (Production Genealogy):
  Product → Recipe → Lot → Machine → ProductionLine → Plant
                        → Material → Supplier
```

---

> **설계 요약:** 본 온톨로지는 15개 Object Type, 14개 Link Type, 7개 Action Type, 7개 Function, 3개 Interface로 구성된다. Palantir의 "결정 중심 설계, Golden Record, Action 기반 거버넌스" 원칙을 따르되, 특정 플랫폼에 종속되지 않는 범용 스마트팩토리 표준 아키텍처를 지향한다. 이 구조 위에 LLM 기반 AIP 에이전트를 연결하면, "현황 조회"를 넘어 "운영 실행"까지 자동화하는 닫힌 루프(Closed-Loop) 의사결정 시스템이 완성된다.