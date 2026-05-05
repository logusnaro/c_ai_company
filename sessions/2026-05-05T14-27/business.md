# 💰 Business — 행동 변화 유도(Action-Result)를 위한 핵심 데이터 모델의 필수 필드와 수익화(Tier 2) 연계 로직에 필요한 데이터 요구사항을 정의하세요.

💰 Business: 작업 시작합니다.

## 행동 변화 유도(Action-Result) 기반 핵심 데이터 모델 및 수익화 연계 요구사항 정의

이 정의는 LogUs의 핵심 가치인 'AI 인사이트 기반 행동 변화 유도'를 측정하고, 이를 기반으로 수익화 티어(Tier 2)를 설계하기 위한 데이터 구조의 기반을 제공합니다.

### 1. 핵심 데이터 모델 필수 필드 정의 (Action-Result 중심)

핵심 데이터 모델은 사용자의 **행동(Action)**, 그 행동이 발생한 **맥락(Context)**, 그리고 그 행동의 **결과(Result)**, 그리고 AI가 도출한 **인사이트(Insight)**를 연결하는 데 초점을 맞춥니다.

| 엔티티 | 필수 필드 | 설명 및 행동 변화 연관성 | 수익화 연계 목표 |
| :--- | :--- | :--- | :--- |
| **User** | `user_id`, `profile_data`, `goals_set` | 사용자 식별 및 목표 설정 정보. (수익화 대상 정의) | 구독 레벨 결정 |
| **Action Log** | `action_id`, `user_id`, `timestamp`, `action_type` (예: 수면 기록, 습관 시작), `raw_data` | 사용자가 수행한 모든 기록 및 행동의 원천 데이터. (데이터 입력 마찰 최소화) | 데이터 볼륨 측정 |
| **Context** | `context_id`, `action_id`, `location`, `mood_state`, `external_factors` | 행동이 발생한 환경 및 심리적 맥락 데이터. (행동의 원인 분석) | 심층 분석 기능 (Tier 2) |
| **Result** | `result_id`, `action_id`, `timestamp`, `outcome_metric` (측정 가능한 결과), `self_reflection` | 행동의 직접적인 결과 및 사용자의 주관적 피드백. (결과 측정) | 성과 보고 및 코칭 기능 (Tier 2) |
| **Insight** | `insight_id`, `user_id`, `action_id`, `result_id`, `ai_analysis_type` (예: 상관관계, 패턴), `recommendation` | AI가 Action, Context, Result를 분석하여 도출한 행동 변화에 대한 구체적인 제안. (핵심 가치) | 프리미엄 코칭 제공 (Tier 2) |
| **Subscription** | `sub_id`, `user_id`, `plan_type` (Free, Basic, Pro), `status`, `access_level` | 사용자의 현재 구독 상태 및 접근 권한. | 수익화 관리의 최종 지표 |

### 2. 수익화(Tier 2) 연계를 위한 데이터 요구사항

수익화 티어(Tier 2)는 단순히 데이터의 양이 아니라, **데이터를 통해 사용자가 얻는 '행동 변화에 대한 통제력'과 'AI의 예측적 가치'**에 기반해야 합니다.

#### A. 핵심 데이터 요구사항 (Monetization Trigger)

1.  **행동-결과 연결 강도 (Action-Result Link Strength):**
    *   **요구사항:** `Action Log`와 `Result` 간의 인과관계(Causality)를 AI가 얼마나 명확하게 추론했는지 측정하는 지표가 필요합니다.
    *   **연계:** 이 강도가 높을수록 사용자는 AI의 추천을 더 신뢰하고, 고급 코칭으로의 전환을 고려하게 됩니다.

2.  **맥락 기반 예측 정확도 (Contextual Prediction Accuracy):**
    *   **요구사항:** `Context` 데이터(감정, 환경)를 활용하여 특정 `Action`의 `Result`를 예측하는 AI 모델의 정확도를 측정해야 합니다.
    *   **연계:** 예측 정확도가 높을수록 '예방적 행동 변화' 코칭(Pro Tier)의 가치가 높아집니다.

3.  **반복 학습 및 피드백 루프 데이터 (Feedback Loop Data):**
    *   **요구사항:** 사용자가 `Insight`에 대해 수용하거나 거부하는 피드백(`self_reflection`) 데이터를 체계적으로 저장해야 합니다.
    *   **연계:** 이 피드백을 통해 AI는 개인화된 코칭 전략을 더욱 정교화하며, 이는 구독 서비스의 핵심 차별점이 됩니다.

#### B. 수익화 티어별 데이터 접근 차별화 로직

| 구독 티어 | 접근 가능한 핵심 데이터 | 제공되는 핵심 가치 (Monetization Hook) |
| :--- | :--- | :--- |
| **Free (기본)** | `Action Log`, 기본 `Result` 요약 | 기록 및 추적 (Tracking) |
| **Basic (중급)** | 모든 기본 데이터 + `Context` 데이터 | 패턴 발견 (Pattern Recognition) |
| **Pro (고급/Tier 2)** | 모든 데이터 + **`Insight` (AI 분석) 및 `Recommendation`** | **행동 변화 유도 및 예측적 코칭 (Actionable Coaching)** |

### 📝 다음 단계:
데이터 모델 설계(Schema)를 구체화하고, 각 필드에 대한 초기 데이터 타입 및 관계를 정의하는 작업에 착수합니다.
