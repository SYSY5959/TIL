ICLR 2023
# ToolLLM: Facilitating Large Language Models to Master 16000+ Real-world APIs

# Abstract
- ToolBench 데이터셋: ChatGPT로 자동 생성
	1. RapidAPI Hub에서 16,464개 API 수집 (49개 카테고리)
	2. ChatGPT로 다양한 instruction 생성 → single/multi-tool 포함
	3. ChatGPT로 API Call Chain annotation
- **DFS 기반 Decision Tree : 여러 reasoning 경로 탐색해 추론 능력 강화**
- ToolEval : 툴 사용 능력 자동 평가
- **Neural API Retriever : 각 instruction에 맞는 API 추천**

→ ToolLLaMA는 **복잡한 명령 실행 + 모르는 API 일반화** 에 강함
→ ChatGPT와 비슷한 성능
→ APIBench 에서도 zero-shot 일반화 성능 입증


# 기존 연구 문제점

1. 오픈 소스 LLM의 한계 : 대부분 언어 중심 task에만 초점을 맞춘 연구들, **외부 툴 (API) 활용 능력이 부족함.**
2. 폐쇠형 LLM(ChatGPT, Claude, Bard) 이미 **tool-use capability** 잘 보여줌
→ “언어만 잘하는 오픈소스 모델”과 “도구까지 잘 쓰는 폐쇄형 모델” 사이에 큰 격차 존재

![](<./Images/ToolLLM_Table.png>)

# ToolLLM 제안

## ToolBench
**(Tool(API), Instruction, Solution Path)** 구성
- Tool(API) : RapidAPI에서 가져온 API와 그 메타데이터
- Instruction : ChatGPT가 seed를 바탕으로 생성한 자연어 요청
- Solution Path: Instruction을 해결하기 위한 API 호출 순서 (chain of calls)

[Example]
- Tool(API): Weather API (`/getWeather(city)`)
- Instruction: “서울의 현재 날씨를 알려줘.”
- Solution Path:
    1. 호출: `GET /getWeather?city=Seoul`
    2. 응답: `{ "temp": 25°C, "condition": "sunny" }`
    3. 최종 답변: “현재 서울은 맑고 기온은 25도입니다.”

### API Collection
- 출처 : RapidAPI (real-world APIs marketplace) - https://rapidapi.com/hub
- **16,464개 RESTful API (3,451 high-quality Tools)**, 49개 카테고리 (금융, 스포츠, 날씨 등등)
- 실제로 사용 가능한 API pool 구축 → ToolBench

- 하나의 Tool(서비스)이 여러개 API endpoint로 구성될 수 있음. 
- ex) "Weather Tool" → `/getCurrentWeather`, `/getForecast`, `/getAirQuality`

- 각 툴마다 크롤링 : **툴 정보 / 설명, 호스트 URL (API 서버 주소), 툴에 속한 모든 API 목록**
- 각 **API의 메타데이터 기록** : 
	- API 이름, 설명, 
	- HTTP method (GET, POST 등)
	- 필수 파라미터, 선택적 파라미터
	- 요청 body 구조
	- 실행 가능한 코드 스니펫 (ex. Python request 예시), 예시 API 호출 응답
	→ 이렇게 디테일한 메타데이터가 있으면, zero-shot(처음보는 API)에서도 설명, 파라미터 구조만 보고 합리적인 호출 추론 가능

![](<./Images/ToolLLM_Figure-1.png>)

### Instruction Generation
ChatGPT를 이용해 instruction 자동 생성

- $\{ \text{API}_1 , \dots , \text{API}_N \} \in S_{\text{API}}$ 
- $\{ \text{seed}_1 , \dots , \text{seed}_3 \} \in S_{\text{seed}}$ : Seed 프롬프트 3개 (Instruction generation을 위한 초기 예시 문장) 
- $\text{Inst}_i$ : 생성된 instruction 
- $\mathbb{S}^{\text{rel}}_i$​: 그 instruction에 관련 있는 API들의 subset

$$
\textbf{ChatGPT}\left( \left( \left\{ \left[ \mathbb{S}^{\text{rel}}_1 , \text{Inst}_1 \right] , \dots , \left[ \mathbb{S}^{\text{rel}}_{N'} , \text{Inst}_{N'} \right] \right\} \middle| \text{API}_1 , \dots , \text{API}_N , \text{seed}_1 , \dots , \text{seed}_3 \right) \right)
$$
 - 입력: (API 집합 + 3개 seed instruction)
 - 출력: (instruction + 관련 API subset) N개쌍

ex)
- Instruction: “서울의 현재 날씨를 알려주고, 결과를 영어로 번역해줘.”
- 관련 API 집합: {Weather API, Translation API}

### Solution Path Annotation
**Solution Path : API Call chain** = Instruction → (API1 호출 → API2 호출 …) → 답변
- ChatGPT로 API Call chain 생성
- 주어진 Instruction $Inst_∗$에 대해, ChatGPT에게 'valid action sequence $\{a_1, \dots, a_N\}$를 찾아라' 라고 프롬프트 함. 

**Multi-step Decision-making → Multi-round Conversation** 
- 여러 단계 reasoning 필요하니까 → multi-round 대화 형식
$$ChatGPT(a_t​∣{a_1​,r_1​,…,a_{t−1}​,r_{t−1}​},Inst_∗)$$
→ 이전까지의 action/response + Instruction 바탕으로 다음 action 선택  (a: API 호출, r: 실제 API 응답)

- 여기서 Action $a_t$ 는 3가지 반드시 포함
	1. Thought : 모델이 왜 이 API 쓰려고 하는지 reasoning
	2. API Name
	3. Parameters

++ ) **ChatGPT의 Function Call 활용**
- 각 API를 special function으로 정의하고, 그 API documentation(설명, 파라미터 등)을 ChatGPT의 함수 필드에 입력 
- *OpenAI에서 추가한 기능으로, 미리 정의한 함수 호출하고 출력을 구조화 할 수 있음*
- *함수 정의에는 name, description, schema를 JSON Schema 형식으로 넣을 수 있음*
→ ChatGPT가 이 API는 어떻게 호출하는지 이해하고 적절한 action 생성할 수 있음

**Depth First Search-based Decision Tree (DFSDT)**

![](<./Images/ToolLLM_Figure-2.png>)
- 기존 LLM reasoning (CoT, ReAct)에서는 단일 reasoning path
→ 복잡한 multi-tool instruction에서는 한 번 잘못된 API를 선택하면 전체 경로가 실패
→ 여러 가능한 경로 탐색하는 DFS 기반 Decision Tree

- 루트에서 시작해 가능한 Action들을 DFS로 탐색 
- → 각 Action에서 “thought + API + parameters” 생성
- → Action 실행 (API call 결과 얻음) 
- → 성공 : 다음 Action /  실패 : 백트래킹
- 종료 조건 : "Finish with Final Answer"(성공) or "Finish by Giving Up" (모든 경로 실패)

```css

                 Start (Instruction)
                        |
         --------------------------------
        |                                |
   API: getWeather(Seoul)          API: translate("??", en)
        |                                |
   Response: 날씨=맑음,25도        실패 (입력 없음) → Backtrack
        |
   API: translate("맑음,25도", en)
        |
   Response: "Clear, 25°C"
        |
   Finish with Final Answer

```



## ToolLLaMA Framework
LLaMA 기반으로, ToolBench 데이터셋과 DFS 알고리즘, Neural API Retriever 이용해 API 사용 능력을 학습시킨 오픈소스 Tool 사용 특화 LLM

![](<./Images/ToolLLM_Figure.png>)
### Training
- 데이터 : ToolBench (16,464개 API, Instruction, Solution Path)
- LLaMA에 SFT(Supervised Fine-Tuning) → Instruction → 올바른 API Call chain 생성하도록 학습
- 학습 과정에서 DFS 기반 DT 탐색 → 더 robust한 reasoning 학습
- **+ Neural API Retriever** : ToolBench에 포함된 API 중 Instruction에 맞는 후보 빠르게 찾을 수 있도록!

### Inference
1. 사용자 자연어 입력 instruction 
2. Neural API Retriever가 관련 API 후보 제시
3. DFS Decision Tree로 여러 action sequence 탐색
4. 실제 API 호출 실행
5. API 결과 통합해서 최종 답변 생성

### ToolEval
- 기존의 LLM 평가는 보통 언어 출력 품질 위주
-  Tool-use capability는 **API를 올바르게 호출하고, 실행 가능한 결과를 만들어 내는 능력** 이 중요함
- → **ToolEval : API 호출 과정을 자동으로 검증하는 시스템**

#### Pass Rate
모델이 생성한 **Solution Path(API 호출 chain)** 이 실제 실행 가능한지, 그리고 Instruction을 충족했는지를 비율로 측정

$$
\text{Pass Rate} \;=\; \frac{\# \; \text{of instructions successfully solved}}{\text{Total \# of instructions}}
$$
#### Win Rate
동일한 Instruction에 대해 **두 모델의 출력을 비교**했을 때, 어느 모델이 더 나은 해답을 냈는지 비율로 계산

$$
\text{Win Rate}(M_1) \;=\; \frac{\# \; \text{of instructions where } M_1 \; \text{is judged better}}{\text{Total comparisons}}
$$

# 실험

**ToolLLaMA (LLaMA 기반 fine-tuned)**
- 비교 모델 : Text-Davinci-003, Claude-2, ChatGPT
- 데이터셋 : 
	- ToolBench
	- APIBench : distribution 밖의 unseen API로 일반화 능력 측정 ([Gorilla](<../Gorilla/Gorilla.md>)의 APIBench)

![](<./Images/ToolLLM_Table2.png>)

![](<./Images/ToolLLM_Table3.png>)
## 주요 결과
1. ToolBench에서 성능: Davinci-003, Claude-2 능가. ChatGPT와 거의 비슷한 수준
2. Generalization (APIBench에서 성능): 보지 못한 API에서도 zero-shot으로 일반화 가능
3. Multi-tool 호출 시나리오도 잘 처리. DFS 기반 탐색 → 여러 경로 고려해서 성공률 상승

## Ablation Study
- **DFS 기반 Decision Tree** 유무 비교:
    - 단순 CoT 방식보다 성능 확실히 개선

- **Neural API Retriever** 유무 비교:
    - Retriever 없으면 관련 API를 못 찾고 성능 급락

→ 두 구성 요소 모두 필수적이다!

# 한계점
- 실제 운영 환경에서는 (ex. 실시간 API 호출, 네트워크 지연, 인증 키 필요, rate limit 문제 등)에서는 **latency, 비용, 안전성** 문제가 훨씬 커질 수 있음
- ToolBench : ChatGPT가 만든 instruction + solution path에서 ChatGPT의 편향 그대로 들어감
- DFSDT 탐색 과정에서 실패한 경로는 학습 신호로 활용되지 않음
	- 최근 후속 연구(_TP-LLaMA, Advancing Tool-Augmented LLMs, 2024_)는 실패 trajectory도 학습에 활용하면 성능이 더 좋아진다고 보고함
- API missuse(잘못된 사용), 보안 위험, 개인정보 침해 가능성 같은 **safety 문제**를 거의 다루지 않음
- API 후보 pool이 16k개라서, Neural Retriever + DFSDT 탐색 자체가 계산 비용이 큼
- 실제 응용에서는 여러 턴 대화 속에서 API 사용을 누적 관리해야 하는데, 이런 long-term interaction 시나리오는 다루지 않음. 

