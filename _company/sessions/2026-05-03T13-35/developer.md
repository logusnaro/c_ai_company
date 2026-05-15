# 💻 Developer — 요구된 기능(음성 메모, 수유 데이터, 기저귀 데이터 세분화, 수면 분리 관리)을 지원하기 위한 데이터 모델(DB 스키마)의 확장안을 설계하고, 특히 음성 입력 시스템의 기술적 구현 가능성을 초기 검토하세요.

💻 Developer: 작업 시작합니다.

요청하신 기능(음성 메모, 데이터 세분화, AI 기능)을 지원하기 위한 데이터 모델 확장안과 음성 입력 시스템의 기술적 구현 가능성을 설계합니다.

---

## 1. 데이터 모델 확장안 (DB 스키마 설계)

LogUs의 핵심 가치인 '행동 변화 유도'를 지원하기 위해, 단순 기록을 넘어 **상황(Context)**과 **행동(Action)**을 기록할 수 있도록 엔티티를 확장하고 관계를 설정합니다.

### A. 핵심 엔티티 정의

| 엔티티 (Table) | 주요 역할 | 핵심 필드 (Key Fields) | 비고 |
| :--- | :--- | :--- | :--- |
| **User** | 사용자 계정 관리 | `user_id`, `name`, `settings` | 관리자(Admin) 접근 권한 관리 |
| **BabyProfile** | 아기 기본 정보 | `baby_id`, `user_id`, `date_of_birth`, `gender` | |
| **FeedingLog** | 수유 기록 | `feed_id`, `baby_id`, `timestamp`, `type` (모유/분유), `duration_min`, `input_method` (수동/음성) | 직전 데이터 자동 입력 로직 지원 |
| **DiaperLog** | 기저귀 기록 | `diaper_id`, `baby_id`, `timestamp`, `wet_status` (젖음 여부), `stool_status` (대변 포함/미포함/대변만), `notes` | 세분화된 상태 기록 |
| **SleepLog** | 수면 기록 | `sleep_id`, `baby_id`, `start_time`, `end_time`, `duration_hr`, `type` (낮잠/밤잠) | 낮/밤 분리 관리 |
| **VoiceMemo** | 음성 메모 기록 | `memo_id`, `baby_id`, `timestamp`, `audio_file_url`, `transcription_text`, `context_tags` | 음성 입력 시스템의 결과 저장 |
| **AIInsight** | AI 분석 결과 | `insight_id`, `baby_id`, `analysis_date`, `suggested_action`, `confidence_score`, `source_data_ids` | 행동 변화 제안의 결과 저장 |
| **UserQuery** | 사용자 질문/상담 기록 | `query_id`, `user_id`, `timestamp`, `prompt`, `ai_response`, `status` (Pending/Answered) | AI 상담 기능의 이력 관리 |

### B. 데이터 관계 및 확장 상세

1.  **FeedingLog 확장 (직전 데이터 자동 입력 지원):**
    *   `FeedingLog` 테이블은 `timestamp`를 기준으로 **직전 기록**을 쉽게 조회할 수 있도록 인덱싱되어야 합니다.
    *   **자동 입력 로직:** 새로운 기록 시, 시스템은 `FeedingLog`에서 해당 `baby_id`의 가장 최근 기록을 찾아 `duration_min` 또는 `type` 필드를 기본값으로 제안하여 사용자에게 확인을 요청하는 로직을 추가합니다. (이는 백엔드 로직에서 구현되어야 합니다.)

2.  **DiaperLog 세분화:**
    *   `stool_status`는 Enum 또는 정수 코드를 사용하여 명확하게 정의합니다.
        *   `1`: 대변 포함 (Full)
        *   `2`: 대변 미포함 (No Poop)
        *   `3`: 대변만 (Poop Only)
    *   이를 통해 AI가 배변 패턴의 미묘한 변화까지 분석할 수 있는 기반을 마련합니다.

3.  **SleepLog 분리 관리:**
    *   `SleepLog`는 `type` 필드를 통해 `DAY` 또는 `NIGHT`로 명확히 구분하며, 각 시간대별 수면 패턴 분석을 위한 별도의 지표(예: 평균 수면 깊이 추정)를 계산할 수 있도록 설계합니다.

4.  **VoiceMemo 통합:**
    *   `VoiceMemo` 테이블은 음성 파일 자체를 저장하기보다는, **STT 처리 결과(Transcription)**와 **메모의 맥락(Context Tags)**을 저장하는 데 중점을 둡니다. 실제 음성 파일은 별도의 스토리지(S3 등)에 보관하고 URL만 참조합니다.

---

## 2. 음성 입력 시스템 기술적 구현 가능성 검토

음성 입력(STT) 시스템은 **고성능의 파이프라인**을 통해 데이터의 정확성과 사용 흐름의 자연스러움을 동시에 확보해야 합니다.

### A. 기술 구현 흐름 (Pipeline Flow)

1.  **음성 입력 (Input):** 사용자 음성 녹음 및 전송.
2.  **STT 처리 (Transcription):** 음성 파일을 텍스트로 변환 (예: Whisper API, Google Speech-to-Text 등 활용).
3.  **LLM Contextualization (맥락화):** 변환된 텍스트를 기반으로, 해당 기록이 어떤 데이터(수유, 기저귀 등)에 연결되어야 하는지 **메타데이터를 추출**하는 단계. (예: "오늘 수유 기록"이라는 맥락을 추출)
4.  **Data Mapping & Validation (매핑 및 검증):** 추출된 텍스트와 시스템의 데이터 스키마(`FeedingLog`, `DiaperLog` 등)를 매핑하여, **구조화된 JSON 데이터**로 변환.
5.  **Storage (저장):** 구조화된 JSON 데이터를 DB에 저장하고, 원본 텍스트와 음성 파일 URL을 `VoiceMemo` 테이블에 저장.

### B. 기술적 검토 결과 및 제언

| 영역 | 검토 내용 | 기술적 제언 |
| :--- | :--- | :--- |
| **정확도 (Accuracy)** | 육아 용어의 특성상 STT 오류가 발생하기 쉬움. | **도메인 특화 모델(Fine-tuning):** 일반적인 STT 모델 대신, 육아 관련 용어에 특화된 어휘 모델을 사용하여 오인식률을 최소화해야 함. |
| **사용 흐름 (UX)** | 기록 과정에서 사용자가 텍스트 입력보다 음성 입력이 더 자연스러워야 함. | **Context-Aware Prompting:** 입력 직전에 AI가 "지금 수유 기록을 하실 건가요?"와 같은 질문을 통해 음성 입력의
