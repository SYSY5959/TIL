## Equipping Language Models with Tool Use Capability for Tabular Data Analysis in Finance
EACL 2024

# Abstract

- 금융 도메인에서의 LLM은 정밀성 요구로 인해 오류 전파 및 환각 문제가 큼
→ 외부 tool과 결합하여 LLM 추론 상호작용

- LLAMA 2 13B를 기반으로 Supervised Fine-Tuning 수행
	- Tool Router : 이 질문을 모델이 처리할지? 도구가 처리할지?
	- Tool Solver : 실제 답변을 생성하거나, 도구를 사용해서 답변 생성
	→ 둘 다 SFT 방식으로 학습됨

- RAVEN : tool equipped SFT model 
→ 평균 35.2% 성능 향상. GPT 3.5와 경쟁


# 1 Introduction

- LLM에 외부 도구 결합은 정확도·신뢰성 향상을 위한 유망한 접근
- 특히 금융·의료와 같이 정밀성이 중요한 도메인에서 효과 큼
- 기존 접근은 다음 두 가지로 나뉨:
  1. in-context tool prompting (e.g. ReAct, HuggingGPT)
	- Few-shot prompt 기반 tool 사용 유도
	- Tool 사용 예시를 prompt에 넣어서 LLM이 따라하게 함
  2. fine-tuned models with static tool protocols (e.g. Toolformer)
	- static한 tool 사용을 위한 fine-tuning → **모델이 tool 사용 위치를 학습** 
	- **RAVEN 모델**



이 논문에서, 금융 도메인에 tool 기반 추론 적용
	- LLAMA 2 13B 기반, PEFT(LORA) 방식으로 tool use 학습
	- 상업적 모델에 의존하지 않고, 공개 데이터만으로 학습
	- QA 태스크를 instruction 기반으로, 질문마다 동적으로 처리


# 2 RAVEN

- LLAMA 2 12B CHAT 모델 기반
- LoRA 사용해 파인튜닝 (전체 파라미터의 0.2%만 학습)

##### LoRA (Low-Rank Adaptation)
기존 모델 파라미터는 고정하고, 작은 rank의 matrix만 학습하여 계산 효율을 높이는 방식 (PEFT 기법 중 하나)

## 2.1 Fine-Tuning Data
금융, 일반 도메인의 구조적, 비구조적 QA datasets 4가지 mix

### TAT-QA
금융 보고서 기반 QA
수치 추론과 표-텍스트 혼합

### Financial PhraseBank (FPB)
감성 분류용 문장 level 금융 뉴스. 레이블 : 긍정/부정/중립

### Wiki-SQL
자연어 질문, SQL 쿼리 변환 데이터. 표 포함
금융 도메인 데이터 아니지만, 답변 생성하는 스크립트
(모델이 tool 사용하도록 요도하는데 사용)

### OTT-QA
다양한 도메인의 표 + 비구조적 텍스트 기반 multi-hop QA
대부분의 질문은 두가지 형태의 데이터를 모두 포함하는 다단계 추론 필요.


## 2.2 Tools
2가지 Tool : Calculator + Lightweight SQL

#### Calculator
계산기 : python interpreter 기반 → 올바른 산술식 평가

#### SQL Tool
- SQL 엔진 : 관계형 데이터에 대해 SQL 스크립트를 실행할 수 있는 API
- text (JSON) → table 형태 변환
- 데이터 타입도 변환 (문자열 숫자를 정수형으로..)

- API 2가지 input
	1. 구조화된 데이터의 문자열 표현
	2. SQL 스크립트

```plaintext
사용자 질문 → LLM 분석 → SQL 필요 판단
→ SQL 쿼리 생성 → API로 표+쿼리 전송
→ SQL 실행 → 결과 반환 → 사용자에게 응답
```

[Example] 

🧾 입력 표 (텍스트 형태)
```json
{
  "header": ["회사명", "연도", "매출"],
  "rows": [
    ["삼성전자", "2020", "236조"],
    ["LG전자", "2021", "74조"],
    ["카카오", "2021", "7조"],
    ["현대차", "2021", "117조"]
  ]
}
```


❓ 질문 (사용자 입력)
> “2021년에 매출이 가장 높은 회사는?”


🤖 LLM의 판단:
- 표 데이터를 기반으로 조건 필터 (`연도 = 2021`)
- `매출` 기준으로 `내림차순 정렬 + 상위 1개` 필요
→ SQL 도구가 필요하다고 판단함


🧠 LLM이 생성한 SQL
```sql
SELECT [회사명]
FROM data_table
WHERE [연도] = '2021'
ORDER BY CAST(REPLACE([매출], '조', '') AS FLOAT) DESC
LIMIT 1;
```

🛠 SQL 도구 실행 결과
>현대차


🗣️ 최종 출력 (모델 응답)
> “2021년 기준, 매출이 가장 높은 회사는 **현대차**입니다.”


## 2.3 Instruction Tuning
- 모델 학습 : 다양한 지시문 템플릿 통해
- 템플릿 구성 요소 **(1,4번은 필수로 포함)**
	1. **Instruction (지시문)**
	2. Input (텍스트 질문)
	3. Data (표 형태 구조 데이터)
	4. **Derivation or Answer (중간 계산 or 정답)**

- 질문 유형에 따라 프롬프트 구조 다르게 구성
- 산술 계산 → arithmetic 템플릿
- 감성 분류 → classification 템플릿
- 표 질의 → script(SQL) 템플릿
- 단순 추출 → information extraction 템플릿

🤖 Task Router 학습을 위해 질문 → 템플릿 라벨 데이터도 자동 생성함

#### 방식 : Supervised Fine-Tuning (SFT)
**프롬프트 + 모델 파인튜닝(SFT + LoRA)**

RAVEN의 SFT는 **질문 유형에 따라 다르게 설계된 prompt 템플릿에 따라 입력을 구성**하고,  
이를 기반으로 **정답을 생성하는 방식으로 지도 학습**  
**프롬프트 구조를 내재화한 instruction-tuned 모델**

##### 1. Prompt Template

|구성 요소|예시|설명|
|---|---|---|
|`Instruction`|"이 문장의 감정을 분류하세요."|어떤 작업을 할지 설명|
|`Input`|"매출이 35% 증가했다."|대상 문장, 질문 등|
|`Data`|표 또는 수치 정보|구조화된 입력 데이터|
|`Derivation`|"579,800 ÷ 24,500"|중간 계산식이나 정답 생성 절차|
`Instruction` + `Derivation`(or 정답)은 필수, 나머지는 선택적 포함

##### 2. Template Types
**rule-based 방식으로 질문 유형 라벨링**

| 작업 유형                         | 사용 도구  | 템플릿 예시                                                        |
| ----------------------------- | ------ | ------------------------------------------------------------- |
| `Arithmetic`                  | 계산기    | “주당 가치는 얼마인가요?” → 수식 생성 (예: `578806 / 24500`)                 |
| `Script`                      | SQL 엔진 | “2010년에 활동한 팀은?” → SQL 생성 (예: `SELECT ... WHERE Year = 2010`) |
| `Information`<br>`Extraction` | 없음     | “다음 표에서 매출 증가율을 말해줘” → 모델이 직접 추출                              |
| `Classification`              | 없음     | “이 문장의 감성은?” → positive / negative / neutral                  |

##### 3. Example

[Arithmetic]
```yaml
Instruction: "주당 평균 가치는 얼마입니까?"
Input: "Robert Andersen은 24,500주의 주식을 받고 578,806달러를 벌었습니다."
Data: { "Number of Shares": "24500", "Value": "578806" }
Derivation: 578806 / 24500
```

- 이 프롬프트 구조에 대해 정답 학습시킴
- 모델은 이 구조에 익숙해지고, 새로운 질문을 받았을 때도 템플릿 기반으로 잘 대답하게 됨

### Toolformer vs RAVEN

#### 공통점
| 공통점                      | 설명                  |
| ------------------------ | ------------------- |
| 🧩 도구를 활용함               | 계산기, 검색기 등 외부 툴을 연동 |
| 🧠 LLM이 언제 도구를 써야 할지 학습함 | 내부/외부 경로를 선택        |


#### 차이점
| 항목        | Toolformer                                                      | RAVEN                                      |
| --------- | --------------------------------------------------------------- | ------------------------------------------ |
| 데이터 생성 방식 | **self-supervised**: 모델이 스스로 API 호출 위치 예측                       | **rule-based**: 기존 QA 데이터에서 수식/SQL 여부로 라벨링 |
| 학습 목적     | 도구 호출 지점 삽입 학습 (self-learn)                                     | 질문 유형 분류 + 적절한 템플릿 선택                      |
| 학습 데이터 생성 | GPT 기반 rollout + filtering                                      | 사람이 만든 QA 데이터셋 활용, rule 기반 자동 유형 부여        |
| 프롬프트 형태   | 모델이 도구 API 호출을 문장 안에 self-insert<br>(e.g. `<CALC> 2+2 </CALC>`) | 템플릿에 따른 도구 호출 <br>(Router + Solver 분리)     |

## 2.4 Inference

추론 시, 2단계 처리 과정
1. Task Router : 입력된 질문 기반으로 '산술', '분류', 'SQL 스크립트', '정보 추출' 중 가장 적합한 템플릿 선택
2. Task Solver : 선택된 템플릿에 `지시문 + 입력 + 데이터` 전달하면 모델이 최종 출력 생성

```arduino
Task Router → 템플릿 선택 (프롬프트 구조 정함)
→ Task Solver → 해당 프롬프트에 맞춰 모델 실행
→ (프롬프트에 따라 계산기나 SQL 등 도구 연동)
```

- 📑 템플릿별 동작 방식:
  - **Script**: SQL 쿼리 생성 → 경량 DB 엔진에 실행
  - **Arithmetic**: 수식 생성 → 계산기 실행
  - **Information Extraction**: 구조 정보만 활용해 직접 판단
  - **Classification**: 감성/범주 등 직접 예측

**질문 유형에 따라 최적 템플릿을 동적으로 선택하고, 도구를 활용한 정확한 추론 수행**

![](<./Images/스크린샷 2025-05-13 오후 9.51.17.png>)

1. 질문 들어옴
2. Task Router → "이건 `script` 템플릿이네"
3. Task Solver → script 템플릿 구조로 프롬프트 생성
4. → 템플릿에는 “SQL 엔진 호출하기”가 포함돼 있음
5. → SQL 실행 결과를 받고, 최종 출력 완성

# 3 Experiments

비교 : 
- LLAMA2 base
- LLAMA2 + SFT-only : 직접 SFT 파인튜닝만 하고 tool 호출만 안함
- GPT-3.5 (COT, 5-shot, +tools)
- **RAVEN (SFT + Tool)**

## 3.1 Main Results

![](<./Images/Pasted image 20250804005441.png>)

실험 결과:
  - 평균 35.2% 향상 (base 대비)
  - GPT-3.5보다 9.2% 우수

 데이터셋:
  - TAT-QA: 계산기 덕분에 multi-step reasoning 성공
  - **OTT-QA: 도구 없이 reasoning 어려움 → 낮은 성능**
  - WikiSQL: SQL 도구 덕분에 큰 성능 향상
  - PhraseBank: 도구 영향 없음 → SFT-only와 거의 동일

BACKOFF 전략:
  - 도구 실패 시 fallback으로 일반 SFT 응답 생성

## 3.2 Analysis
### Is it better to have a separate model for each task?
TAT-QA 전용 모델 따로 학습했을 때, 57.70% 정확도 → RAVEN보다 2.4% 높음
But, RAVEN처럼 하나의 모델로 다양한 tasl 처리하는 것이 더 실용적

### Why tool augmentation is necessary?

<img src="./Images/Pasted image 20250804021717.png" width="400">


SFT도 모델 정확도에 개선되지만, tool을 사용한 것이 56.7%로 훨씬 더 향상됨.

WIKI-SQL & OTT-QA : 구조화된 형식과 비구조화된 형식 모두 포함하는 데이터의 multi-hop 추론 필요
- WikiSQL: 구조화된 데이터에서 실행될 때 → 답변 생성하는 데이터 추출 스크립트로 주석 처리
	- 스크립트를 SQL 도구 사용해서 정확도 85.52%
- OTT-QA: 중간 파생 단계 없음
	- 도구 사용 어려워서 정확도 낮음 (16.03%)

### What is the impact of question complexity?
산술 연산자 개수 늘어날수록 정확도 감소

<img src="./Images/Pasted image 20250804022940.png" width="400">


# 4 Conclusion

  - LLaMA2 13B의 0.2%만 파인튜닝하여 tool 사용 능력 내장
  - 평균 35.2% 성능 향상 (GPT-3.5 대비 +9.2%)
  - Multi-hop reasoning에 외부 tool이 큰 효과

## Limitations
### Infrastructure Bottleneck
  - 실험은 일반 GPU (RTX 4090) 환경에서 수행
  - 더 큰 모델 (70B)이나 긴 context window 사용 시 성능이 더 향상될 수 있음

### Language model evaluation
- BLEU, Exact Match, BERTScore 모두 수치 중심 QA에선 부정확할 수 있음

| 지표                   | 원래 용도                    | 한계                                                           |
| -------------------- | ------------------------ | ------------------------------------------------------------ |
| **Exact Match (EM)** | 정답과 완전히 같은 문자열일 때만 인정    | `"4,000,000"` vs `"400만"` → 다르게 판단됨                          |
| **BLEU**             | 기계 번역의 단어 <br>n-gram 유사도 | 숫자 표현이나 단위가 달라지면 점수 낮게 나옴                                    |
| **BERTScore**        | 의미적으로 유사한 문장인지 판단        | `"480"` vs `"480 million"` 도 유사하다고 평가할수도 있음 (틀린 정답일수도 있는데,,) |

### GPT-3.5 evaluation

- GPT-3.5 응답 불안정
- 프롬프트 정확하게 따르지 않음
- 확신 없을 때 무응답하거나 회피
- 자동 평가지표(EM)와 맞지 않는 출력 생성
- RAVEN과 공정한 비교 어려움