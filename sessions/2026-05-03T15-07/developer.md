# 💻 Developer — 제안된 단계적 처리 로직을 구현하기 위한 시스템 설계(API, 데이터 모델)의 기술적 요구사항과, 광범위한 역사적 데이터 통합을 위한 시스템 재설계 방안을 제시합니다.

💻 Developer: 작업 시작합니다.

## 🛠️ 시스템 설계 및 재설계 방안: 행동 변화 유도를 위한 데이터 파이프라인

CEO의 지시사항에 따라, 제안된 **순차적 처리(Staged Processing) 로직**을 구현하고 **광범위한 역사적 데이터 통합**을 위한 시스템 설계(API, 데이터 모델) 및 재설계 방안을 제시합니다.

### 1. 시스템 아키텍처 개요 (System Architecture Overview)

데이터의 정제, 통합, 그리고 AI 인사이트 도출의 흐름을 보장하기 위해 **Lambda 또는 Kappa 아키텍처** 기반의 데이터 파이프라인을 제안합니다.

**주요 컴포넌트:**
1.  **Ingestion Layer (입력 계층):** 음성 입력(STT), 사용자 인터페이스(UI), 외부 시스템으로부터 데이터를 수집합니다.
2.  **Processing Layer (처리 계층):** 데이터 정제, 분류, 시간 순서 정렬, 그리고 행동 변화 데이터 추출(Staged Processing 로직 실행).
3.  **Storage Layer (저장 계층):** 정형 데이터(행동 지표)와 비정형 데이터(원시 로그)를 분리하여 저장합니다.
4.  **Insight Layer (인사이트 계층):** 통합된 데이터를 기반으로 AI 모델을 구동하여 행동 변화 예측 및 제안을 생성합니다.

### 2. 데이터 모델 설계 (Data Model Schema)

행동 변화 유도를 최우선으로 하므로, **행동 결과(Outcome)**를 중심으로 정형화하고, 원시 데이터는 보조적으로 관리하는 분리된 스키마를 제안합니다.

#### A. 핵심 행동 데이터 모델 (Behavioral Core Schema)
이 모델은 **1순위(행동 변화 데이터)**에 집중하여 AI 인사이트의 근거를 제공합니다.

```json
{
  "record_id": "UUID",
  "user_id": "string (FK)",
  "timestamp": "datetime (UTC)",
  "data_type": "ENUM [Sleep, Feeding, Diaper, Voice_Input]",
  "metric_type": "ENUM [Input, Confirmation, Action, Outcome]",
  "value": "float/string (정량적 지표 또는 텍스트)",
  "context": "JSON (사용자 확인/상황 정보)",
  "derived_insight": "JSON (AI가 자동 생성한 행동 변화 관련 인사이트)"
}
```

#### B. 시스템 및 흐름 데이터 모델 (System Flow Schema)
이 모델은 **3순위(시스템/흐름 데이터)**를 관리하며, MVP 구현의 자연스러움을 보장합니다.

```json
{
  "flow_id": "UUID",
  "session_id": "string (FK)",
  "step_sequence": "integer",
  "action_performed": "string (사용자가 수행한 행동)",
  "system_response": "string (시스템의 반응)",
  "user_confirmation": "boolean (사용자 확인 여부)",
  "metadata": "JSON (시스템 설계 관련 상세 정보)"
}
```

### 3. API 설계 (API Specification)

데이터 통합 및 인사이트 제공을 위한 핵심 API 엔드포인트를 정의합니다.

#### A. 데이터 수집 및 통합 API

| 엔드포인트 | HTTP Method | 기능 설명 | 요청 데이터 (Request Body) | 응답 데이터 (Response Body) |
| :--- | :--- | :--- | :--- | :--- |
| `/api/v1/ingest/behavior` | POST | 새로운 행동 및 확인 데이터를 시스템에 기록 (Stage 1 Ingestion) | Behavioral Core Schema 데이터 | 성공/실패 상태, 기록 ID |
| `/api/v1/history/pull` | GET | 특정 사용자/기간의 정제된 행동 결과 데이터 조회 (Stage 2 Retrieval) | `user_id`, `start_date`, `end_date`, `filter_type` | 정제된 행동 결과 목록 (Behavioral Core Schema) |
| `/api/v1/system/log` | POST | 시스템 흐름 및 오류 로그 기록 (Stage 3 Ingestion) | System Flow Schema 데이터 | 성공/실패 상태, 로그 ID |

#### B. 인사이트 생성 API

| 엔드포인트 | HTTP Method | 기능 설명 | 요청 데이터 (Request Body) | 응답 데이터 (Response Body) |
| :--- | :--- | :--- | :--- | :--- |
| `/api/v1/insight/generate` | POST | 통합된 데이터를 기반으로 AI 인사이트 생성 (Stage 4 Insight Generation) | `user_id`, `context_data` (최근 행동 기록), `focus_area` (예: 수면 패턴 개선) | `derived_insight` (행동 변화 유도 제안), `confidence_score` |

### 4. 역사적 데이터 통합 재설계 방안 (Historical Data Integration Redesign)

광범위한 역사적 데이터(비정형 로그)를 효과적으로 통합하기 위해 **Staged Processing**을 데이터 파이프라인의 핵심 로직으로 통합합니다.

**재설계 전략:**

1.  **Raw Data Lake 구축 (Storage Layer 분리):**
    *   모든 원시 로그(음성 파일, 텍스트 메모, 시스템 이벤트)는 변경 없이 **Data Lake**에 보존합니다. 이는 나중에 새로운 분석 방법이 생겼을 때 재처리할 수 있는 원천 데이터(Source of Truth)가 됩니다.

2.  **Staged Processing 엔진 강화 (Processing Layer):**
    *   **Stage 1 (정제):** Raw Data Lake에서 데이터를 읽어와 **STT/NLP 모델**을 통해 비정형 텍스트를 구조화하고, 모든 데이터에 **시간 스탬프와 사용자 ID**를 강제 부여합니다. (이 단계에서 '왜'라는 질문에 대한 정량적 근거를 찾기 시작합니다.)
    *   **Stage 2 (통합):** 정제된 데이터를 **Behavioral Core Schema**와 **System Flow Schema**로 매핑합니다. 이 과정에서 **사용자 확인(Confirmation)** 플래그를 최우선으로 적용하여 데이터의 신뢰도를 높입니다.
    *   **Stage 3 (인사이트 추출):** 통합된 행동 데이터를 기반으로 **AI 모델**이 행동 변화 유도 패턴을 학습하고, **Derived Insight**를 자동으로 생성합니다.

3.  **지능형 파트너 강화 (Insight Layer):**
    *   기존의 단순 데이터 보고가 아닌, **행동 변화 예측 및 제안**에 초점을 맞춥니다. AI는 과거의 '행동'과 '시스템 흐름'을 연결하여, "만약 A를 하면 B라는 행동 변화가 일어날 확률이 높다"라는 **인과관계 기반의 제안**을 생성하도록 설계합니다.

이 설계는 데이터의 양을 관리하면서도, LogUs의 핵심 목표인 '행동 변화 유도'에 필요한 정량적이고 신뢰할 수 있는 데이터를 안정적으로 제공할 것입니다.

📊 평가: 완료 — 요청된 시스템 설계, 데이터 모델, API 스펙, 그리고 역사적 데이터 통합 재설계 방안을 구체적인 기술 요구사항과 함께 제시했습니다.
📝 다음 단계: `/api/v1/ingest/behavior`의 초기 데이터 유효성 검사 로직을 Python 코드로 작성하고, `sessions/2026-05
