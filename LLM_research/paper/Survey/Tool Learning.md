# Tool Learning with Large Language Models: A Survey
Frontiers Comput. Sci.2024

![](<./Images/Pasted image 20250809193532.png>)


![](<./Images/Pasted image 20250809193654.png>)


---

## Why Tool Learning?

LLM에 툴을 결합하는 장점:

1. **Knowledge Acquisition** : 검색 엔진, DB, 날씨·지도 API로 실시간·정형 정보 습득
2. **Expertise Enhancement** : 수학 툴, Python interpreter, 화학·의학·경제 전용 툴로 전문성 강화
3. **Automation & Efficiency** : 일정관리, 이메일 필터링, 온라인 쇼핑, 데이터 분석 자동화하고 효율성 높임
4. **Interaction Enhancement** : 다중 모달 입력(음성·이미지), 번역기, NLP 툴로 상호작용 품질 향상
5. **Interpretability & Trust** : 도구 사용 과정을 투명하게 제시해 신뢰성 향상
6. **Robustness & Adaptability** : 다양한 입력 변형에 강건성 확보

---

## How Tool Learning?

![](<./Images/Pasted image 20250809220557.png>)

### 왼쪽: 4단계 Tool 활용 워크플로우

1. **Stage 1: Task Planning**
    - 입력: 사용자의 질문(Question).
    - 작업: LLM이 의도 파악 후 질문을 하위 질문으로 분해
    - 결과적으로 여러 개의 Sub Question이 만들어져 이후 단계로 전달

2. **Stage 2: Tool Selection**
    - **Retriever-based Selection**: 검색기(Retriever)를 사용해 관련성 높은 툴 상위 K개를 선별
    - **LLM-based Selection**: LLM이 직접 제공된 후보 툴 중 적합한 툴 선택
    - 목표는 각 Sub Question에 맞는 최적의 툴을 찾는 것

3. **Stage 3: Tool Calling
    - LLM이 각 Sub Question에 맞춰 파라미터 추출
    - 추출한 파라미터와 함께 Tool Server(API)에 GET/POST 요청을 보내서 Output 추출

4. **Stage 4: Response Generation**
    - **Direct Insertion Methods**: 툴 호출 결과를 그대로 문장에 삽입
        - ex. "It is (get_weather(Beijing)→rainy)."
    - **Information Integration Methods**: 툴 결과를 LLM 입력으로 넣어 문맥에 맞는 자연스러운 응답 생성.
        - ex. "It is rainy."


##### Tool Learning with LLMs – 4 Stages & Methodology Overview

| Stage                            | 목적                           | 하위 분류                       | 설명                                 | 주요 예시                                                                                                                            |
| -------------------------------- | ---------------------------- | --------------------------- | ---------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| **Stage 1: Task Planning**       | 복잡한 질의를 하위 질문으로 분해, 실행 순서 정의 | **Tuning-free**             | LLM 프롬프트 기반, 추가 학습 없음              | CoT, ReACT, ART, RestGPT, HuggingGPT, ToolChain*, PLUTO, ATC, Sum2Act, BTP                                                       |
|                                  |                              | **Tuning-based**            | 툴 계획 데이터셋으로 fine-tuning            | Toolformer, TaskMatrix.AI, Toolink, TPTU-v2, α-UMi, COA, DEER, SOAY, TP-LLaMA, APIGen                                            |
| **Stage 2: Tool Selection**      | 하위 질문에 적합한 툴 선택              | **Retriever-based**         | IR 기법으로 상위 K개 후보 검색 (Sparse/Dense) | BM25, Sentence-BERT, CRAFT, ProTIP, COLT, ToolRerank                                                                             |
|                                  |                              | **LLM-based**               | LLM이 직접 후보 툴 중 선택                  | (Tuning-free) CoT, ReACT, DFSDT, ChatCoT, ToolNet, AnyTool / (Tuning-based) ToolBench, TRICE, ToolLLaMA, Confucius, ToolVerifier |
| **Stage 3: Tool Calling**        | 툴 파라미터 추출·포맷팅 후 API 호출       | **Tuning-free**             | 프롬프트·규칙 기반 추출                      | Reverse Chain, EasyTool, Tool Doc Compression, ConAgents                                                                         |
|                                  |                              | **Tuning-based**            | 호출 데이터로 fine-tuning                | GPT4Tools, ToolkenGPT, Themis, STE, ToolVerifier, TRICE                                                                          |
| **Stage 4: Response Generation** | 툴 결과와 내부 지식 결합, 최종 답변 생성     | **Direct Insertion**        | 툴 결과를 그대로 삽입                       | TALM, Toolformer, ToolkenGPT                                                                                                     |
|                                  |                              | **Information Integration** | 툴 결과를 요약·압축·해석해 응답 생성              | RestGPT, ToolLLaMA, ReCOMP, ConAgents                                                                                            |

### 오른쪽: 두 가지 Tool Learning 패러다임

1. **Tool Learning with One-step Task Solving**
    - 4단계를 한 번에 순차적으로 진행
    - 초기 계획 수립 후 중간 피드백 없이 끝까지 수행

2. **Tool Learning with Iterative Task Solving**
    - 각 단계를 반복하며 피드백 반영
    - 예: Stage 3에서 받은 툴 출력이 기대와 다르면 다시 Stage 1~2로 돌아가 계획 수정

#### Tool Learning with One-step Task Solving

1. Stage 1 (Task Planning)에서 LLM이 전체 문제 해결 계획과 모든 하위 작업을 한 번에 설계
2. Stage 2 → Stage 3 → Stage 4를 순차적으로 진행하면서 **중간에 계획을 수정하지 않음**
3. 툴 호출이 잘못되거나 정보가 누락되어도 계획을 다시 세우지 않고 그대로 진행

장점:
- 단순 구조 → 구현이 쉽고 빠름.
- 반복 호출이 없어서 **지연(latency)** 이 적음
- 툴 사용 비용 절감

단점:
- 초기에 세운 계획이 잘못되면 결과 품질 하락
- 툴 호출 실패나 잘못된 데이터에도 대응이 어려움

적합한 경우
- 문제의 구조가 명확하고 오류 가능성이 낮을 때.
- 실시간성이 중요한 서비스(예: 금융 시세 질의)에서 지연을 최소화해야 할 때.

#### Tool Learning with Iterative Task Solving

1. Stage 1에서 부분 계획만 세우고 바로 Stage 2~3에서 툴을 호출
2. 툴 결과를 보고 계획을 수정하거나 새로운 하위 작업을 생성
3. 이 과정을 **필요할 때까지 반복**하면서 점점 정밀한 답변으로 발전
4. 마지막에 Stage 4에서 최종 응답 생성

장점:
- 툴 출력이 부정확하거나 부족하면 계획을 수정 가능.
- 복잡한 문제나 다단계 의존성이 있는 작업에서 정확도 향상.
- 실패 회복력(recovery)이 높음

단점:
- 여러 번 툴 호출 → 지연 시간 증가 및 비용 상승
- 시스템 설계와 컨텍스트 관리가 더 복잡

적합한 경우
- 다단계 의존성이 있는 문제 (예: 먼저 환율을 찾고, 그 값을 이용해 금 시세 계산)
- 정보가 불확실하거나 툴 응답이 변동성이 큰 경우
- 고정된 계획으로는 성공률이 낮은 복잡한 질의

---
### 1. Task Planning
사용자의 복잡한 질의 이해 → 해결 가능한 하위 질문과 실행 순서로 분해
복잡한 문제는 툴 호출 필요하므로, 이 단계에서 전체 전략 세움

주요 작업 :
- **Intent Understanding** – 질문의 의도 파악
- **Task Decomposition** – 문제를 최소한의 하위 질문들로 분해
- **Dependency Mapping** – 하위 작업 간 의존 관계 및 순서 결정

|구분|장점|단점|예시|
|---|---|---|---|
|**Tuning-free**|모든 LLM 사용 가능, 빠른 적용, 학습비용 없음|품질·일관성 제한, 프롬프트 설계 난이도|CoT, ReACT, HuggingGPT, RestGPT|
|**Tuning-based**|높은 정확도·일관성, 도메인 최적화 가능|오픈소스 LLM 한정, 학습·데이터 비용 큼|Toolformer, TaskMatrix.AI, TP-LLAMA|

#### Tuning-free Method
LLM의 **기존 능력**(few-shot, zero-shot prompting)을 활용해 계획을 세우는 방식

- 추가 학습 필요 없고 모든 LLM에 적용 가능
- 주로 **prompt engineering**과 **multi-step reasoning** 기법 사용
- 구현 비용 낮음, 하지만 계획 품질이 일관적이지 않을 수 있음

[예시]
- **CoT (Chain-of-Thought)** [Wei et al., 2022] → "Let's think step-by-step" 같은 지시어로 단계적 사고 유도
- **ReACT** [Yao et al., 2022] → Reasoning과 Action을 번갈아 수행
- **ART** [Paranjape et al., 2023] → 과거 작업 라이브러리에서 예시 검색 후 계획 생성
- **RestGPT** → Coarse-to-Fine 방식으로 점진적 계획 refinement
- **HuggingGPT** → 명세 기반 프롬프트와 예시 기반 파싱 결합
- **ToolChain*** → 전체 API 호출 공간을 의사결정 트리로 구성
- **PLUTO** → 반복적으로 가설 생성 → 클러스터 분석 → 하위 질문 개선
- **ATC** → 블랙박스 툴 탐색 및 학습을 통한 계획 생성
- **Sum2Act** → 진행 상황 요약 후 다음 액션 결정
- **BTP** → 비용 제약 하에서 최적의 툴 사용 계획 수립

#### Tuning-based Method
Task Planning 성능을 높이기 위해 **도구 사용 예시 데이터셋**으로 LLM을 추가 학습하는 방식

- 계획 정확도와 일관성이 높음
- 특정 도메인/툴셋에 최적화 가능
- 단점: 오픈소스 LLM에만 가능, 데이터 준비와 학습 비용 큼

[예시]
- **Toolformer** [Schick et al., 2024] → API 호출 예시를 스스로 생성해 fine-tuning
- **TaskMatrix.AI** → RLHF로 툴 사용 계획 학습
- **Toolink** → 목표 작업을 도구 모음(toolkit)으로 분해 후 Chain-of-Solving 방식 적용
- **TPTU-v2** → 도메인 특화 API 호출 계획을 학습
- **α-UMi** → 2단계 학습: 일반 계획 → 세부 계획 fine-tuning
- **COA** → 추상적 reasoning 체인을 먼저 생성 후, 도메인 툴로 세부화
- **DEER** → 다중 의사결정 브랜치를 포함한 계획 예시 생성·학습
- **SOAY** → API 호출 계획 → 실행 코드 생성의 2단계 파이프라인 학습
- **TP-LLAMA** → 실패 시도까지 학습 데이터로 활용하여 정책 최적화(DPO)
- **APIGen** → 대규모·다양한 함수 호출 데이터셋 자동 생성

---

### 2. Tool Selection
Task Planning 단계에서 나온 하위 질문(Sub-questions)을 해결하기 위해 적합한 도구(API, 플러그인 등)를 선택하는 과정
실제 환경에서는 수백~수천 개의 툴이 존재하므로 효율적 검색과 선택 전략이 필요

|구분|장점|단점|주요 적용|
|---|---|---|---|
|**Retriever-based**|대규모 툴셋에서 빠른 후보 검색, LLM 입력 부담 완화|후보에 적절한 툴이 없으면 실패|대규모 API 환경|
|**LLM-based**|논리적/복합 선택 가능, 순차 호출 계획 가능|후보 수 많으면 context 길이 초과|소규모 후보 세트, 고정밀 선택|

#### Retriever-based Tool Selection
- **정보 검색(IR) 기법**을 사용해, 전체 툴셋에서 **상위 K개의 후보 툴**을 먼저 찾는 방식
- LLM은 여기서 바로 툴을 고르는 게 아니라, **검색 결과**만 받아 이후 선택에 활용

- 장점:
    - 대규모 툴셋(수천 개 이상)에서도 빠르게 후보를 줄일 수 있음
    - LLM 입력 길이 제한(context length) 문제 완화

- 단점:
    - 검색기의 품질이 낮으면 적절한 툴이 후보에 포함되지 않을 수 있음
    - 의미적 관련성 외에 툴 기능 완전성까지 고려하기 어려움

방법론 분류 : 
1. **Term-based (Sparse Retrieval)**
    - TF-IDF, BM25 같은 키워드 기반 매칭
    - ex) *Gorilla 모델이 BM25+GPT-Index를 사용*

2. **Semantic-based (Dense Retrieval)**
    - Sentence-BERT, ANCE, Contriever 등 임베딩 기반 유사도 검색
    - ex)
        - **CRAFT**: LLM이 가짜 툴 설명을 만들어 검색
        - **ProTIP**: 작업 분해 기반 검색
        - **COLT**: 그래프 신경망(GNN)으로 툴 완전성 고려
        - **ToolRerank**: 계층 구조와 미사용 툴까지 고려해 재랭킹

#### LLM-based Tool Selection
- **LLM이 직접 툴 설명·파라미터 목록과 질의**를 함께 보고, 적합한 툴을 **이해·추론하여 선택**하는 방식
- Retriever-based Selection 이후 후보 툴이 적을 때 주로 사용

- 장점:
    - 툴 간 의존 관계나 순서를 고려한 복합 선택 가능
    - 시맨틱(semantic)뿐 아니라 논리적(reasoning) 연결까지 고려

- 단점:
    - 후보 툴 수가 많으면 context length 문제 발생
    - 프롬프트 설계·튜닝 필요

방법론 분류 :
1. **Tuning-free**
    - 프롬프트 기반 in-context learning
    - ex)
        - **CoT**: 단계적 추론으로 툴 선택
        - **ReACT**: Reasoning과 Action을 번갈아 수행
        - **DFSDT**: 깊이 우선 탐색으로 오류 전파 방지
        - **ChatCoT**: 대화 기반 다단계 툴 선택
        - **ToolNet**: 툴 그래프 탐색
        - **AnyTool**: 계층적 구조 기반 효율적 선택

2. **Tuning-based**
    - 툴 선택 데이터로 LLM fine-tuning
    - ex)
        - **ToolBench**: 시연 예시+시스템 프롬프트 조합 학습
        - **TRICE**: 행동 복제(behavior cloning) + 실행 피드백 강화학습
        - **ToolLLaMA**: DFSDT 생성 데이터로 LLaMA 학습
        - **Confucius**: 툴 난이도에 따라 점진적 학습(curriculum learning)
        - **ToolVerifier**: 후보 툴 간 대조 질문을 통해 자기검증(self-verification)

---
### 3. Tool Calling
선택된 툴을 실제로 호출하기 위해 필요한 **입력 파라미터**를 사용자 질문과 툴 설명에서 추출하고, 정해진 포맷에 맞게 API 요청을 생성하는 단계
ex) 사용자 질문: “현재 뉴욕의 기온을 알려줘”
- 툴 설명: `get_weather(city: STRING, unit: STRING)`
- 추출 파라미터: `{city: "New York", unit: "Celsius"}`

💡 에러 처리의 중요성
- **형식 오류**: JSON 형식 불일치, 잘못된 데이터 타입
- **값 오류**: 범위를 벗어난 입력값
- **서버 오류**: API 서버 응답 실패
- 해결책: 오류 메시지를 기반으로 호출 내용을 재생성하는 **Self-repair loop** 설계

|구분|장점|단점|예시|
|---|---|---|---|
|**Tuning-free**|모든 LLM 적용 가능, 데이터 불필요, 빠른 적용|복잡한 파라미터 구조에 취약|Reverse Chain, EasyTool, ConAgents|
|**Tuning-based**|높은 정확도·포맷 준수, 도메인 최적화|학습 비용·데이터 준비 부담, 새 툴 일반화 한계|GPT4Tools, ToolkenGPT, STE|

#### Tuning-free Methods
- 파라미터 추출과 포맷팅을 추가 학습 없이 수행
- 주로 **프롬프트 설계**와 **예시(few-shot) 제공** 또는 **규칙 기반 처리**를 사용

- 장점:
    - 모든 LLM에 적용 가능
    - 빠른 적용, 데이터 준비 불필요

- 단점:
    - 복잡한 파라미터 구조나 새 툴 형식에서 실패 가능성 높음

[예시]
- **Reverse Chain** [Zhang et al., 2023]
    - 먼저 최종 툴을 결정한 뒤 필요한 파라미터를 채움
    - 부족한 파라미터는 추가 툴 호출로 채움
- **EasyTool** [Yuan et al., 2024]
    - 툴 설명을 간결하게 재작성하고 사용 가이드를 포함시켜 이해도 향상
- **Tool Documentation Compression** [Xu et al., 2024b]
    - 툴 문서를 요약·압축하여 핵심 파라미터 정보만 제공
- **ConAgents** [Shi et al., 2024]
    - 여러 에이전트를 협력시켜, **전담 실행 에이전트**가 파라미터 추출·호출 담당


#### Tuning-based Methods
**툴 호출 데이터셋**으로 LLM을 **fine-tuning**하여 파라미터 추출과 호출 능력을 강화

- 장점:
    - 특정 툴셋에 최적화 가능
    - 파라미터 정확도 및 포맷 준수율 상승

- 단점:
    - 학습 데이터 준비 비용 큼
    - 새로운 툴에는 성능 저하(훈련 데이터 의존성)

[예시]
- **GPT4Tools** [Yang et al., 2024]
    - ChatGPT가 생성한 툴 사용 데이터로 LoRA 파인튜닝
- **ToolkenGPT** [Hao et al., 2024]
    - 특수 토큰(“toolkens”)을 예측하면 툴 호출 모드로 전환하여 파라미터 생성
- **Themis** [Li et al., 2023]
    - 툴 사용 + 추론 과정을 통합하여 동적으로 파라미터 설정
- **STE** [Wang et al., 2024]
    - 생물학적 학습 메커니즘(시도-오류, 상상, 기억)을 모방해 파라미터 정확도 향상
- **ToolVerifier** [Mekala et al., 2024]
    - 생성된 파라미터를 자기 검증하여 오류 줄임
- **TRICE** [Qiao et al., 2024]
    - 행동 복제 후 툴 실행 피드백을 통한 강화학습

---

### 4. Response Generation
- **툴 호출 결과**와 LLM의 **내부 지식**을 결합해 사용자의 질문에 최종 답변을 생성
- 툴 출력은 텍스트, 숫자, 코드, 이미지, 비디오 등 다양한 형태일 수 있으므로, 이를 사용자 친화적이고 완결성 있는 응답으로 변환하는 것이 핵심

| 구분                          | 장점               | 단점               | 적합한 경우        | 예시               |
| --------------------------- | ---------------- | ---------------- | ------------- | ---------------- |
| **Direct Insertion**        | 구현 단순, 빠른 응답     | 복잡·다중 모달 출력에 부적합 | 짧고 단순한 툴 결과   | TALM, Toolformer |
| **Information Integration** | 복잡/다중 툴 결과 종합 가능 | 컨텍스트 길이·전처리 부담   | 복합 정보 통합 필요 시 | RestGPT, ReCOMP  |

#### Direct Insertion Methods
- 툴 호출 결과를 **그대로**, **간단한 치환 방식**으로 최종 응답에 삽입하는 방법
- ex)
    - 원문: `"It is Weather()."`
    - 치환 후: `"It is rainy."`

- 장점:
    - 구현이 단순하고 빠름
    - 툴 출력이 짧고 명확할 때 적합

- 단점:
    - 툴 출력이 길거나 복잡하면 읽기 어려움
    - 다양한 모달리티(이미지, 코드 등) 처리에 한계

[예시]
- **TALM** [Parisi et al., 2022]
- **Toolformer** [Schick et al., 2024]
- **ToolkenGPT** [Hao et al., 2024]

#### Information Integration Methods
툴 결과를 **LLM 입력 context**에 포함시켜, LLM이 해당 결과를 해석·요약·통합한 후 응답을 생성하는 방식

- 장점:
    - 툴 출력이 길거나 구조화된 경우에도 자연스러운 답변 생성 가능
    - 여러 툴의 결과를 결합해 종합적인 응답 생성

- 단점:
    - 컨텍스트 길이 제한 문제 발생 가능
    - 툴 출력 전처리·요약 과정 필요

- 해결 방법:
    - **결과 요약**: RestGPT의 Schema-based 요약
    - **결과 압축**: ReCOMP의 정보 압축 기법
    - **출력 자르기**: ToolLLaMA의 Truncation
    - **다이나믹 파싱**: ConAgents의 Schema-free Extraction

 [예시]
- **RestGPT** [Song et al., 2023] – 미리 정의된 Schema로 툴 결과 간소화
- **ToolLLaMA** [Qin et al., 2024] – 출력 길이 초과 시 잘라서 사용
- **ReCOMP** [Xu et al., 2024] – 긴 텍스트에서 핵심 정보만 압축
- **ConAgents** [Shi et al., 2024] – 동적 추출 함수를 생성해 툴 출력 파싱

---
## Benchmark

![](<./Images/Pasted image 20250810164920.png>)

- 평가하는 tool learning의 단계 (① task planning, ② tool selection, ③ tool calling, ④ response generation)

---
## Toolkit
LLM이 **외부 도구를 사용하도록 연결·관리하는 개발 프레임워크나 라이브러리**
툴 학습 실험이나 실제 서비스 구현 시, **LLM ↔ 툴/API ↔ 사용자**를 중간에서 이어주는 **인프라/플랫폼**

💡 Toolkits의 역할
1. **개발 편의성**: LLM과 API 연결을 쉽게 해줌 (프롬프트·파라미터 처리 자동화)
2. **표준화**: 툴 설명·호출 포맷을 통일
3. **확장성**: 새로운 툴이나 데이터 소스 추가 용이
4. **실험 지원**: 다양한 툴 조합·워크플로우 실험 가능

| Toolkit       | 설명                                 | 주요 특징                                         | 깃허브                                        |
| ------------- | ---------------------------------- | --------------------------------------------- | ------------------------------------------ |
| **LangChain** | LLM을 API, DB, 다른 시스템과 연결해 워크플로우 구축 | 체인(Chain)·에이전트(Agent)·메모리 등 모듈 제공, 플러그인 확장 쉬움 | `github.com/langchain-ai/langchain`        |
| **Auto-GPT**  | LLM이 스스로 목표를 세우고, 하위 작업을 만들고 실행    | 인터넷 검색, 파일 관리, API 호출을 자율적으로 수행               | `github.com/Significant-Gravitas/Auto-GPT` |
| **BabyAGI**   | 경량 자율 에이전트 프레임워크                   | 모듈형 구조, 외부 API와 쉽게 통합                         | `github.com/yoheinakajima/babyagi`         |
| **BMTools**   | 다양한 툴을 LLM에 연결하고 공유하는 커뮤니티 플랫폼     | Python 함수 기반 플러그인, ChatGPT 플러그인 호환            | `github.com/openbmb/BMTools`               |
| **WebCPM**    | 중국어 롱폼 질의응답 특화, 웹 검색 시뮬레이션         | 검색→정보추출→응답의 멀티스텝 프로세스 구현                      | `github.com/WebCPM/WebCPM`                 |

##### 금융 도메인 에서 비교
| Toolkit       | 주요 기능                                             | 장점                                    | 단점                      | 금융 도메인 적용 시 강점                                 |
| ------------- | ------------------------------------------------- | ------------------------------------- | ----------------------- | ---------------------------------------------- |
| **LangChain** | LLM 체인(Chain) 구성, 에이전트 실행, 메모리 관리, 다양한 API 연결     | 모듈화·확장성 우수, 대규모 커뮤니티, 금융 API 연동 예시 많음 | 초기 설정 복잡, 최적화 필요        | **다중 데이터 소스 결합**에 강함 (시세 API + 뉴스 API + 분석 모듈) |
| **Auto-GPT**  | 자율적 목표 수립·하위 작업 생성·실행                             | 완전 자율형 실행 가능, 반복 작업 자동화               | 불필요한 API 호출 많음, 느릴 수 있음 | **시장 모니터링** 자동화, 목표에 맞춰 API 반복 호출              |
| **BabyAGI**   | 경량 자율 에이전트, 모듈 교체 쉬움                              | 단순·빠른 프로토타이핑                          | 복잡한 워크플로우에 한계           | **금융 리서치 봇** 같은 단일 목적 에이전트 설계에 적합              |
| **BMTools**   | 다양한 커뮤니티 제작 툴 연결, Python 기반 플러그인, ChatGPT 플러그인 호환 | 툴 제작·공유 편리, 표준 포맷                     | 플러그인 품질 편차              | **다양한 외부 금융 API**(주가, 환율, 경제지표) 쉽게 통합          |
| **WebCPM**    | 웹 검색 기반 롱폼 QA, 중국어 특화                             | 검색→추출→응답 구조화, 금융 보고서 분석에 유리           | 중국어 중심, 영문 자료는 추가 튜닝 필요 | **금융 보고서/뉴스 분석**에 적합, 웹 크롤링 기반 최신 데이터 반영       |

##### 💡 금융 도메인에서의 선택 가이드
- **복잡한 멀티소스 분석** → **LangChain**  
    예: 주가 API + 금융 뉴스 + ESG 데이터 종합 분석

- **자율형 모니터링·자동 보고서 생성** → **Auto-GPT**  
    예: 매일 아침 전일 종가, 환율, 주요 뉴스 요약 보고

- **단일 목적·경량 봇** → **BabyAGI**  
    예: 채권 금리 데이터만 모아주는 봇

- **다양한 금융 API 통합** → **BMTools**  
    예: 거래소 API + 블룸버그 + 환율 API를 하나의 인터페이스로

- **금융 뉴스/보고서 검색·요약** → **WebCPM**  
    예: 중국 증시 분석, 해외 금융 보고서 자동 요약

---
## Evaluation

📍 Stage별 평가 방법

| Stage                            | 평가 목적                        | 주요 지표                             | 설명                                       |
| -------------------------------- | ---------------------------- | --------------------------------- | ---------------------------------------- |
| **Stage 1: Task Planning**       | 질의가 툴 사용이 필요한지 여부, 계획의 품질 평가 | **Tool Usage Awareness Accuracy** | 해당 질의가 툴 필요 여부를 정확히 판단하는 비율              |
|                                  |                              | **Pass Rate**                     | 계획이 실제로 문제를 해결하는 비율 (자동 평가 또는 인공지능 평가)   |
|                                  |                              | **Accuracy**                      | 생성된 계획이 정답 계획과 얼마나 일치하는지                 |
| **Stage 2: Tool Selection**      | 적합한 툴을 선택하는 능력 평가            | **Recall@K**                      | 상위 K개 선택 툴 중 정답 툴 비율                     |
|                                  |                              | **NDCG@K**                        | 선택 툴의 순위 품질 (관련 툴이 상위에 올수록 점수 높음)        |
|                                  |                              | **COMP@K**                        | 상위 K개 툴이 정답 툴 세트를 완전히 포함하는지 여부           |
| **Stage 3: Tool Calling**        | 툴 파라미터 추출·포맷 정확도             | **Format Consistency**            | 툴 문서 규격에 맞게 파라미터를 추출했는지                  |
|                                  |                              | **Parameter Coverage**            | 필수 파라미터를 모두 포함했는지                        |
|                                  |                              | **Parameter Validity**            | 값이 유효 범위에 맞는지                            |
| **Stage 4: Response Generation** | 최종 응답 품질                     | **BLEU**                          | 생성 텍스트의 n-gram 일치율                       |
|                                  |                              | **ROUGE-L**                       | 참조 응답과의 longest common subsequence 비율    |
|                                  |                              | **Exact Match (EM)**              | 생성 응답이 정답과 완전히 동일한 비율                    |
|                                  |                              | **F1**                            | Precision과 Recall의 조화 평균 (QA 평가에서 주로 사용) |

### Stage 1: Task Planning
 **Tool Usage Awareness Accuracy**
  - 툴 사용이 필요한 질의를 정확히 판별하는 비율
$$\text{Accuracy} = \frac{\text{\# Correct Judgments}}{\text{\# Total Judgments}}
  $$
  
**Pass Rate**
  - 계획이 실제로 문제를 해결하는 비율 (자동/휴먼 평가)
   $$
  \text{Pass Rate} = \frac{\text{\# Successful Executions}}{\text{\# Total Executions}}
  $$

**Plan Accuracy**
  - 생성된 계획과 정답 계획의 일치도

### Stage 2: Tool Selection
**Recall@K**  
  상위 K개의 선택 툴 중 정답 툴 비율  
  $$
  \text{Recall@K} = \frac{1}{|Q|} \sum_{q=1}^{|Q|} \frac{|T_q^K \cap T_q^*|}{|T_q^*|}
  $$
  - $Q$: 질의 집합  
  - $T_q^*$: 질의 $q$의 정답 툴 집합  
  - $T_q^K$: 질의 $q$에서 상위 K개 선택 툴 집합

**NDCG@K**  
  관련 툴의 순위 품질 평가  
  $$
  \text{DCG}_q@K = \sum_{i=1}^K \frac{2^{g_i} - 1}{\log_2(i+1)}
  $$
  $$
  \text{NDCG@K} = \frac{1}{|Q|} \sum_{q=1}^{|Q|} \frac{\text{DCG}_q@K}{\text{IDCG}_q@K}
  $$
  - $g_i$: $i$번째 툴의 graded relevance score  
  - IDCG: 이상적인 DCG 값


**COMP@K**  
  상위 K개의 툴이 정답 툴 세트를 완전히 포함하는지 여부  
  $$
  \text{COMP@K} = \frac{1}{|Q|} \sum_{q=1}^{|Q|} I(\Phi_q \subseteq \Psi_q^K)
  $$
  - $\Phi_q$: 질의 $q$의 정답 툴 집합  
  - $\Psi_q^K$: 질의 $q$에서 상위 K개 선택 툴 집합  
  - $I(\cdot)$: indicator function

### Stage 3: Tool Calling
- **Format Consistency**: 툴 문서 포맷과 일치 여부  
- **Parameter Coverage**: 필수 파라미터 포함 여부  
- **Parameter Validity**: 값이 유효 범위에 속하는지 여부

### Stage 4: Response Generation
- **BLEU** (n-gram precision 기반 기계 번역/생성 평가)  
- **ROUGE-L** (참조 응답과의 longest common subsequence 비율)  
- **Exact Match (EM)**  
  $$
  \text{EM} = \frac{\text{\# Exact Matches}}{\text{\# Total Cases}}
  $$
- **F1 Score**  
  $$
  \text{F1} = \frac{2 \cdot \text{Precision} \cdot \text{Recall}}{\text{Precision} + \text{Recall}}
  $$

---


## Challenges & Future Directions

1. **High Latency in Tool Learning**
    - 다단계 툴 호출로 인한 응답 지연 문제.
    - 필요 없는 툴 호출 최소화, 툴 자체의 경량화 필요.

2. **Rigorous and Comprehensive Evaluation**
    - 현재는 지표와 벤치마크가 통일되어 있지 않음.
    - Stage별 기여도를 분리 평가할 수 있는 **표준 평가 프레임워크** 필요.

3. **Comprehensive and Accessible Tools**
    - 기존 툴셋의 접근성·포괄성이 부족.
    - LLM을 활용한 **자동 툴 생성**과 다분야 커버리지가 중요.

4. **Safe and Robust Tool Learning**
    - 실제 환경의 노이즈·보안 위협에 취약.
    - 안전 검증 및 공격 대응 기법 필수.

5. **Unified Tool Learning Framework**
    - 연구들이 Stage별로 단편적으로 진행됨.
    - **통합형 프레임워크**로 표준화 필요.

6. **Real-world Benchmark for Tool Learning**
    - 실제 사용자 질의 기반의 벤치마크 부족.
    - LLM-툴 시스템 실사용 데이터를 반영한 벤치마크 구축 필요.

7. **Tool Learning with Multi-Modal**
    - 대부분의 툴 학습 연구가 텍스트 기반.
    - 이미지, 음성, 영상 등 **멀티모달 툴 사용 연구** 확장 필요.