NeurlPS 2023
# Tree of Thoughts: Deliberate Problem Solving with Large Language Models


- 여러개의 가능한 사고 경로 (thoughts)를 트리 구조로 검색
- **prompting-based reasoning framework** → *No training, only prompting*

|구성 요소|프롬프트로 하는 일|역할|
|---|---|---|
|**1. Thought Generation Prompt**|“이 문제를 해결하기 위해 다음 단계로 무엇을 시도할 수 있을까?”|후보 thought 생성|
|**2. Thought Evaluation Prompt**|“이 단계는 문제 해결에 얼마나 유망한가? (sure/maybe/impossible)”|각 후보 평가|
|**3. Voting Prompt (optional)**|“다음 후보 중 어느 것이 가장 좋다고 생각하나요?”|후보 비교·투표|
|**4. Search Control Logic**|(코드 레벨에서 BFS/DFS 적용)|탐색 제어|
→ 이렇게 4가지 prompting으로 **생각하기, 평가하기, 비교하기**를 반복하며 탐색 수행

# ToT

![](<./Images/ToT_Figure.png>)

>**ToT idea:**  
>기존 LLM은 “왼쪽에서 오른쪽으로 한 줄로 생각하는(CoT)” 반면,  
>ToT는 여러 가지 **“생각(thought)” 경로를 트리(tree) 형태로 탐색**하여  
>**lookahead**, **backtracking**, **self-evaluation**을 수행함으로써  
>더 의도적(deliberate)이고 계획적인(reasoned) 문제 해결을 수행하도록 함


## 1. Thought Decomposition
- 문제를 중간 단계(thoughts)로 쪼갬
- 각 thought는 문장, 수식, 단락 단위의 텍스트로 구성
- example:
	- Game of 24 → 한 단계는 “중간 수식”
	- Creative Writing → 한 단계는 “글쓰기 계획 문단”
	- Crossword → 한 단계는 “단어 하나 채우기”

> thought가 너무 작으면 평가가 어렵고, 너무 크면 다양성 줄기 때문에, 
> *적당히 짧지만 의미 있는 단위* 여야 함.
> %% 이걸 어떻게 기준을 정하고 정의하지..? 논문에서는 예시처럼 "문제 특성에 맞게 prompt engineer가 적절히 설정해야한다고 씀. %%

## 2. Thought Generator
*"다음에 시도할 thought 후보들을 제안해줘"*
- $G(p_{θ},s,k)$ : Thought Generator
- 현재 상태 $s = [x,z_{1}, ..., z_{i}]$ (입력 + 지금까지의 thoughts)로부터, **다음 thought 후보 $z_{i+1}$들을 k개 생성**하는 함수

[2가지 방법]
1.  i.i.d. sampling
	- $z^{(j)} ∼ p^{CoT}_{θ}(z_{i+1}|s) = p^{CoT}_{θ}(z_{1...i}) \space\space (j=1...k)$
	- 각 thought를 CoT 스타일로 독립적으로 샘플링
	- 창의적인 문제(Creative Writing)에 적합
2. sequential proposal prompting
	- $[z^{(1)},···,z^{(k)}]∼p^{propose}_{θ} (z^{1···k}_{i+1} |s)$
	- 하나의 prompt에서 연속적으로 여러 thought 제안
	- 제약된 문제(Game of 24, Crossword)에 적합

## 3. State Evaluator
*"이 thought가 문제 해결에 얼마나 유망한 것 같아?"*
- $V(p_{θ}​,S)$ : State Evaluator
- 여러 후보 상태 S를 평가하여, **어떤 thought가 좋은 방향인지 점수화**하는 휴리스틱 평가 함수
- → 평가는 완벽할 필요가 없고, *대략적으로 좋은 방향을 가리키기만 해도 충분*
> 절대적인 평가 기준이 있는게 아니라, “지금까지의 추론 문맥에서 이 thought가 ‘좋은 방향처럼 보이는가?’”를 LLM 스스로 언어적으로 reasoning 해서 판단 → self-evaluation reasoning

[2가지 방법]
1. Value estimation (단일 평가)
	- each state independently: $V(p_{θ},S)(s)∼ p^{value}_{θ}(v|s)\space\space ∀s∈S$
	- 각 상태에 대해 1-10 점수 or 'sure/likely/impossible' 평가 부여
	- 이런 평가는 완벽하지 않아도 됨. 대략적인 방향을 위한 휴리스틱 
2. Voting (비교 평가)
	- $V(p_{θ},S)(s) = 1[s = s^∗]$  , good state $s^{*} ∼ p^{vote}_{θ}(s^{*}|S)$
	- 여러 후보 상태를 나열하고 LLM이 직접 투표
	- "어떤 thought가 가장 유망한가?" 판단
	- Creative Writing 등 “정량 평가 어려운 과제”에서 사용

## 4. Search Algorithm
- 생성된 thought 트리 안을 탐색하여 최적의 해를 찾음
- **thought 탐색하면서 트리를 동적으로 생성하고 확장함!!**

### BFS
- 매 단계마다 가장 유망한 b개의 상태를 유지
- **얕은 트리(깊이 ≤3)** 의 문제에 적합 (Game of 24, Creative Writing)
- → 안정적 성능. but, 계산량 많음(모든 분기 탐색)

![|500](<./Images/ToT_Figure_1.png>)
1. 초기 상태 S₀ ← {x}
2. 각 step t에서:
    - 이전 단계의 상태들($S_{t-1}$)로부터 후보 thoughts $S'_t = \{ [s, z] | s ∈ S_{t-1}, z ∈ G(p_θ, s, k) \}$ 생성
    - 각 상태 평가 $V_t = V(p_θ, S'_t)$
    - 상위 b개의 상태 선택만 선택해서, $S_t = \arg\max_{|S|=b} Σ V_t(s)$로 저장
3. 이 과정을 depth T까지 반복하면서 트리 점점 확장
4. 마지막 상태에서 결과 생성

[Game of 24]
![](<./Images/ToT_Figure_3.png>)

[Creative Writing]
![](<./Images/ToT_Figure_4.png>)

### DFS
- 한 경로를 끝까지 탐색하다가 “막히면(backtrack)” 부모 노드로 돌아감
- **깊은 트리(≥10단계)** 문제에 적합 (Crosswords)
- → 효율적(유망 경로에 집중), but 휴리스틱이 부정확하면 좋은 경로 놓칠 수 있음

![|500](<./Images/ToT_Figure_2.png>)
1.  현재 상태 s 에서 thought 후보 k개 생성
2. 각각을 즉시 평가(V) :
	- $V(p_θ, s) > v_{th}$ ​이면 다음 단계 탐색
	- 아니라면 가지치기(pruning) 후 backtrack
3. 유망한 thought만 바로 다음 단계로 넘어감 (recursion)
4. 깊이가 한계 T를 넘으면 결과 기록 → backtrack
5. 최종적으로 가장 깊은(완전한) 상태 반환

[Mini Crosswords]
![](<./Images/ToT_Figure_5.png>)


# ToT vs CoT
ToT는 기존의 CoT prompting을 확장해서, LLM이 단순히 한 줄로 reasoning 하는 게 아니라,
여러 사고 경로(thought paths)를 **트리 형태로 탐색**하면서 더 deliberate(신중)하게 사고하도록 하는 방법

|항목|CoT (Chain-of-Thought)|ToT (Tree-of-Thought)|
|---|---|---|
|구조|단일 reasoning chain|다중 reasoning tree|
|탐색 방식|greedy decoding|DFS / BFS search|
|평가 기준|없음 (직관적 생성)|LM heuristic value|
|장점|간단하고 빠름|deliberation 가능|
|한계|error propagation|높은 compute cost|


# 한계점

1. 계산 비용 
	- 한 단계마다 여러 thought 생성 + 평가 → 한 문제 풀 때 LLM을 수십~수백번 호출
	- BFS : 병렬 분기 때문에 폭발적인 토큰 소비
	- DFS : 깊은 탐색 때문에 시간이 많이 걸림
2. 평가 신뢰성 문제
	- LLM이 스스로 thought 평가 (self-evaluation) → 일관성 부족할 수 있음
	- 창의적, 모호한 문제에서는 좋은 thought을 잘못 평가해서, 유망한 경로로 잘라버리는 경우 있음
3. 탐색 Depth & Latency Limits
	- 트리 탐색 깊이 T가 커질수록 계산량 $O(b^T)$ 로 증가
	- 깊은 reasoning / multi-step 계획을 실시간으로 처리하기 어려움
	- → 얕은 문제에서는 강력하지만, long-horizon reasoning에서는 비현실적
4. No Training / Static LM
5. Generalization & Adaptation
	- 문제 도메인 별로 thought의 granularity, 평가 기준, 탐색 폭(b)을 수동으로 설계해야 함
	- → “자동으로 최적의 트리 구조를 찾는 기능”은 없음