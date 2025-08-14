# Chain-of-Thought Prompting Elicits Reasoning in Large Language Models

NeurIPS 2022 (Google Research, Brain Team)

## 전체 내용 정리

**chain-of-thought prompting** : 문제 해결 과정의 중간 추론 단계를 자연어로 생성하게 하는 기법

**fine-tuning 없이!!**
 **중간 사고 과정을 포함한 prompt (〈input, chain of thought, output〉)가 LLM의 reasoning 성능을 향상시킴**

![[Pasted image 20250722013329.png]]

나머지 task : 수작업 된 8개의 예제 공통으로 사용
AQuA (객관식) : 별도의 5개 예제 사용
![[Pasted image 20250722021249.png]]

결과 : **LaMDA < GPT-3 < PaLM** 성능 Good
→ 모델 크기 커질수록 CoT prompting이 추론 능력 향상시킴


# Abstract

chain of thought : 복잡한 추론을 할 때 거치는 중간 단계의 일련의 사고 과정
이 사고 과정을 생성하게 되면, LLM의 복잡한 추론 능력이 크게 향상 됨.


![[Pasted image 20250722013321.png]]

# 1 Introduction

**chain-of-thought prompting** : 문제 해결 과정의 중간 추론 단계를 자연어로 생성하게 하는 기법

두가지 아이디어 결합
1. 수학 문제에서 자연어 근거를 생성하여 해답에 이르게 하는 방식 (과거 연구)
2. few-shot prompting : 사전 학습된 거대 언어 모델에게 적은 수의 예시(input-output쌍)를 보여주어 새로운 작업을 수행하게 하는 것

이 논문에서 :
문제 입력, 추론 과정(CoT), 출력이 들아간 세 부분으로 구성된 예시를 few-shot prompting에 사용해 추론 능력을 끌어내는 방법 제안
→ arithmetic(산수), commonsense(상식), symbolic reasoning(기호 추론) 벤치마크에서 표준 prompting 보다 월등히 좋은 성능


# 2 Chain-of-Thought Prompting

인간이 생각하는 것처럼 문제를 중단 단계로 나누어 해결 과정을 자연어로 표현하는 방법 
**입력 - CoT - 출력** 쌍 예제 

**fine-tuning 없이!!**
 **중간 사고 과정을 포함한 prompt (〈input, chain of thought, output〉)가 LLM의 reasoning 성능을 향상시킴**

CoT prompting 장점 :
- 문제를 단계별로 처리하여 난이도 높은 문제에 더 많은 계산 자원 투입 가능
- 모델 내부의 추론 경로를 보여주어서, 잘못된 부분을 디버깅할 기회 제공 
- math word problems, commonsense reasoning, symbolic manipulation에 적용 가능
- few-shot prompting의 예시에 CoT 시퀀스 예시를 포함함으로써, 별도의 모델 학습 없이 바로 활용 가능

# 3 Arithmetic Reasoning

사용된 벤치마크 :
- GSM8K (Grade School Math 8K)
- SVAMP
- ASDiv (Algebraic and Diverse)
- AQuA (Algebra Question Answering)  → 객관식 문제 : 4개의 별도 예시
- MAWPS (Math Word Problem Repository)
→ AQuA를 제외한 나머지에는 8개의 수작업 CoT 예시가 공통적으로 적용됨
→ few-shot prompting에 사용

8개의 예제를 선택한 구체적인 기준에 대한 설명 X 

![[Pasted image 20250722021237.png]]

### LM
- GPT-3 : 최대 175B 파라미터
- LaMDA : 최대 137B 파라미터
- PaLM : 최대 540B 파라미터
- UL2
- Codex

파라미터 크기 : 
**LaMDA < GPT-3 < PaLM**

GPT-3: 350M , 1.3B , 6.7B , 175B 파라미터로 구성됨. 최대 175B
LaMDA: 422M, 2B, 8B, 68B, 137B 파라미터로 구성됨. 최대 137B 
PaLM: 8B, 62B, 540B 파라미터로 구성됨. 최대 540B →  GPT-3, LaMDA보다 훨씬 큰 모델

## Results
- CoT는 충분히 큰 모델 크기에서만 효과 
- 작은 규모에서는 성능 개선에 도움 X, 오히려 논리적이지 않은 중간 단계 출력 생성해냄
- 약 1000억 파라미터 이상 모델 부터 성능 향상이 두드러짐. 특히 복잡한 문제에서 큰 성능 개선됨

![[Pasted image 20250720192517.png]]

**LaMDA < GPT-3 < PaLM** 으로 갈수록 모델 파라미터 커짐
→ 모델 크기 커질수록 CoT prompting이 추론 능력 향상시킴

빨간선 : 기존에 파인튜닝된 지도학습 모델의 최고 성능 (SOTA)
**여기서 제안한 모델 : 파인튜닝 없이 단순히 CoT 예시만을 제시해 성능 크게 개선함!!**

## Ablation Study

![[Pasted image 20250720193959.png]]

### Equation only
수학 방정식만 생성하여 답변하는 방식
→ GSM8K에서는 큰 효과 없음

- GSM8K : 복잡한 다단계 산술 문제 해결 능력을 평가하는 데 적합 → 성능 향상 X
- 단일단계나 2단계 문제에서는 성능 향상됨.

### Variable compute only
CoT를 쓰면 성능이 좋아지는 이유가, 
중간 생각을 많이 쓰면서 계산량이 많아져서 그런거 아닌가? 
(문제 풀면서 더 많이 생각하고 시간도 더 쓰니까)

그래서 중간 생각 없이 똑같은 계산량을 만들기 위해, 문제 풀 때 필요한 숫자 개수만큼 점(...) 찍음
→ 그러면 점이 많아지니까 계산도 많이 하는 것처럼 보이도록.
⇒ 성능 향상 X


### Chain of thought after answer
답변 후에 CoT 생성
→ 기본 prompt와 유사한 성능
⇒ 답변 전에 CoT (단계별 추론)을 하는 것이 중요


# 4 Commonsense Reasoning

![[Pasted image 20250720205241.png]]


# 5 Symbolic Reasoning

![[Pasted image 20250720205746.png]]

# 6 Discussion

- 산술 추론에서 CoT를 사용하면 성능이 크게 향상되고, 다양한 주석자, 예제, 모델에 대해 robust
- but, 인간과 같은 실제 이성적 사고인지 여부는 아직 미해결 문제로 남아있음.
- CoT를 제공하는 데 드는 데이터 주석 비용이 적은 few-shot setting에서는 실용적이지만, fine-tuning에는 비용이 크다는 한계점 존재
- 잘못된 추론 경로가 생성될 수 있어 정확한 사실성 보장은 어려우며, 앞으로 개선 과제가 있음
- 대형 모델에서만 나타나는 emergent ability이므로 현실적 적용을 위한 작은 모델에서의 유도 방법 연구가 필요함

## 한계점

### 1. **Chain of Thought = 진짜 추론(reasoning)이 아님**

- 모델이 자연어로 중간 reasoning을 생성한다고 해서,
    > “정말로 논리적으로 사고하는가?”  
    > 라는 질문에는 아직 답할 수 없음.
- 단지 **추론을 흉내내는(imitating)** 것일 수 있음.

> ❝ Although CoT emulates human reasoning, it remains unclear whether LLMs are _actually_ reasoning. ❞

### 2. **CoT 생성은 수작업이 필요함 (Scalability 이슈)**

- Few-shot setting에서는 예시가 몇 개라서 직접 작성 가능하지만,  
    **fine-tuning이나 대규모 데이터 생성에는 많은 코스트**가 듦
- 자동 생성 or synthetic data로 해결 가능성은 있음 (향후 과제)

### 3. **CoT가 있어도 항상 정답은 아님**

- 중간 reasoning이 **잘못된 논리로도 정답을 맞출 수 있음** (또는 그 반대).
- 즉, 모델의 **output이 맞았다고 reasoning이 맞았다는 보장은 없음**.
    

> 논문에서는 이걸 보여주기 위해 오답 사례 50개에 대한 수동 분석도 했어:
> - 약 46%는 사소한 실수 (계산 실수 등)
> - 나머지 54%는 논리 자체가 틀림

### 4. **모델이 충분히 커야만 효과 있음**

- CoT prompting은 **100B+ 모델 규모**에서만 성능이 크게 향상됨
- 작은 모델에서는 CoT가 있어도 **오히려 성능이 낮아질 수도 있음**

### 5. **Inference 시간/비용 증가**

- 중간 reasoning까지 출력해야 하므로, **출력 길이가 길어지고 연산량도 늘어남**
- 특히 OOD 문제나 multi-hop 추론일수록 더 심해짐
