# ToolQA: A Dataset for LLM Question Answering with External Tools
NeurlPS 2023

https://github.com/night-chen/ToolQA

![](<./Images/Pasted image 20250907170244.png>)
- LLM이 도구를 썼는지, 그냥 기억했는지 구분 불가능
- **Overlap 문제**: LLM의 사전학습 데이터와 평가 데이터가 겹치면, 모델이 실제로 도구를 쓰지 않고도 정답을 맞힐 수 있음
- 따라서, **LLM의 tool-use를 평가하는 공정한 벤치마크**를 만들려면 → 평가 질문은 반드시 **내부 지식만으로는 답을 못하고, 외부 도구를 써야만 답할 수 있도록 설계**해야 함.
- ToolQA의 핵심 동기!!!


# ToolQA
- **LLM의 실제 tool-use 능력을 공정하게 평가**
- 8개 도메인(Flights, Coffee, Yelp, Airbnb, GSM8K, DBLP, SciREX, synthetic Agenda)
- 13개 툴(데이터베이스 연산, 수학 계산기, Python/SQL 인터프리터, 그래프 탐색, 텍스트 검색 등)
- 모든 질문은 반드시 툴을 써야 답할 수 있도록 설계 → LLM의 tool-use 능력을 공정하게 평가

# 데이터셋 제작 방식

![](<./Images/Pasted image 20250907170807.png>)
## 3단계 자동화 파이프라인
### (a) Reference Data 수집
LLM의 사전학습 데이터와 겹치지 않는 외부 corpus 수집 (툴이 필요하도록 설계)

### (b)Human-guided 질문 생성
1. 사람이 질문 템플릿을 만듬
2. LLM이 템플릿에 실제 데이터를 채워 넣어 구체적 질문 자동 생성 (easy/hard 난이도)
3. 사람이 다시 검수해서, LLM 내부 지식만으로는 답 못하고, 반드시 **외부 데이터가 필요한 질문만 선택**

- 난이도 구분
	- **Easy**: 단일 툴로 해결 가능 (예: 특정 항공편의 출발시간)
	- **Hard**: 여러 툴 조합 필요, 연산·비교 포함 (예: 평균값, 최댓값, 조건 필터링)

ex) 템플릿 = *“Did the flight from {Origin} to {Dest} on {Date} get cancelled or diverted?”*
- LLM이 템플릿에 LAX, MDW, 01/09/22 같은 값 채워 넣어서 구체적 질문 생성

### (c) Programmatic Answer 생성
tool chain 실행으로 정답 자동 도출
1. 각 툴에 해당하는 operator 함수 구현 (ex. DB Filter, Graph NodeCheck)
2. 질문 템플릿에 대응하는 tool chain 정의
3. 입력값을 넣어 실제 corpus에서 정답 추출

ex) 질문: _“Did the flight from LAX to MDW on 01/09/22 get canceled or diverted?”_
	- 실행: LoadDB → FilterDB(조건 입력) → GetValue → Finish
	- 정답: DB 결과 기반으로 자동 산출
→ 이렇게 하면 labeling 비용이 줄고, 정답이 항상 **도구 기반 ground truth**로 보장

## 데이터셋 통계
- 8개 도메인총 , **1,530 질문 (Easy 800, Hard 730)**
	- **Temporal**: Flights (항공편), Coffee (커피 시세)
	- **Spatial**: Yelp (지역 비즈니스), Airbnb (숙박)
	- **Mathematical**: GSM8K (수학 문제) – LLM이 틀린 케이스만 사용
	- **Scientific**: SciREX (논문 성능 데이터)
	- **Social**: DBLP (학계 논문-저자 그래프)
	- **Personal**: Synthetic Agenda (가상 일정 데이터, 프라이버시 문제 방지)

- 13개 툴 포함:
	- Text Retrieval (AgendaRetriever, SciREXRetriever)
	- Database (Loader, Filter, GetValue)
	- Math (WolframAlpha Calculator)
	- Graph (Loader, Neighbor, Node, Edge)
	- Code (Python, SQL Interpreter)
	- System (Finish)

![](<./Images/Pasted image 20250907162842.png>)

# Experiment

- **비교 대상**:
    1. **ChatGPT (gpt-3.5-turbo)** → 내부 지식만 사용
    2. **CoT (Chain-of-Thought prompting)** → "Let's think step by step" 추가
    3. **Chameleon** (2023) → 여러 툴을 조합 가능하지만 execution feedback 없음
    4. **ReAct** (2023, GPT-3 & GPT-3.5) → reasoning + action 결합, 툴 실행 피드백 기반 iterative refinement 가능.

- **평가 방법**:
    - ToolQA의 easy/hard 질문에 대해 모델이 최종적으로 낸 답과 ground truth를 **exact match**로 비교.
    - 툴 시연은 **tool-level demonstration** 8개만 제공 (각 툴 최소 1회 포함)


## Results
### (a) Easy Questions
- ChatGPT: 평균 **~5%** (거의 실패 수준)
- CoT: 평균 **~5%**
- Chameleon: 평균 **~10%**
- ReAct (GPT-3): **43.1%** → 가장 높음
- ReAct (GPT-3.5): **36.8%** → GPT-3보다 약간 낮음
    - **특징**: GPT-3은 포맷 잘 따름 → easy에서 강점
    - GPT-3.5는 reasoning·코드 이해가 더 좋아서 hard 질문에서 더 강함

- **Easy 질문은 DB 관련 인자 오류가 큼**

### (b) Hard Questions
- ChatGPT: 평균 **~2%**
- CoT: 평균 **~1.4%**
- Chameleon: 평균 **~1.9%**
- ReAct (GPT-3): **5.1%**
- ReAct (GPT-3.5): **8.2%** → 최고 성능이지만 여전히 매우 낮음
→ Hard 질문(복잡한 툴 조합 필요)에서는 현재 모든 모델이 큰 어려움을 겪음

- **Hard 질문은 코드 실행 오류 + context overflow + hallucination이 주요 문제**

![](<./Images/Pasted image 20250907173338.png>)

## Analysis
1. **Tool-augmented LLM이 확실히 우위**
    - 내부 지식만 쓰는 ChatGPT/CoT는 사실상 무력
    - 툴을 활용할 수 있는 ReAct가 가장 성능 높음

2. **Easy vs Hard 격차 큼**
    - Easy: 최고 43% (ReAct)
    - Hard: 최고 8% (ReAct)
    - → **툴 조합 reasoning**이 현재 LLM의 가장 큰 약점

3. **GPT-3 vs GPT-3.5**
    - GPT-3: easy에 강점 (툴 호출 형식 잘 따라감)
    - GPT-3.5: hard에 강점 (추론·코드 이해 능력으로 새로운 해법 시도 가능)

### 주요 에러 유형 (ReAct GPT-3.5 기준)
- Argument Error : 툴 호출 시 잘못된 인자를 넣는 오류 (가장 많음, easy: DB 관련, hard: 코드 관련)
- Incorrect Data Source : 정답을 위해 참조해야 할 corpus를 잘못 선택
- Innovation ↔ Hallucination (창의적 툴 사용 vs 없는 결과 상상)
- 기타: Infeasible Actions, Long Context, Misunderstanding, Retrieval 실패

![](<./Images/Pasted image 20250907173418.png>)

![](<./Images/Pasted image 20250907174359.png>)

![](<./Images/Pasted image 20250907172432.png>)
(a) Innovation
- **ReAct(GPT-3)** vs **ReAct(GPT-3.5)** 비교:
    - **GPT-3**: context에 있던 DB 조작 패턴만 그대로 따르다가 실패 (too long context 문제)
    - **GPT-3.5**:
        1. DB Filter → GetValue 로 시도했지만 실패
        2. “SQLInterpreter”라는 다른 툴을 스스로 선택해서 `AVG(Close)`를 직접 계산
        3. 결과: 120.9 → **정답 도출 성공**
→  GPT-3.5가 **기존 예시(context)에 없던 툴 조합**을 “창의적으로” 시도해 성공 → 논문에서는 이를 **Innovation**이라고 불러

(b) Hallucination
- **ReAct(GPT-3.5)** 과정:
    1. 날짜 필터를 잘못 줌 → DB에서 데이터를 제대로 못 가져옴
    2. 그럼에도 불구하고, 마치 정상적으로 가져온 것처럼 **존재하지 않는 observation을 만들어냄** (예: 잘못된 Close 값)
    3. 그 결과 계산이 틀려서 23.75라는 오답을 냄
→  여기서 **툴이 반환하지 않은 값**을 모델이 만들어내는 걸 논문은 **Hallucination**이라고 부름
→  즉, 툴 실행 피드백과 실제 DB 값이 불일치하는데, 모델이 상상으로 채워버린 경우