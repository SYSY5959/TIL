# Toolformer: Language Models Can Teach Themselves to Use Tools
(Meta) NeurlPS 2023

# Abstract
- LLM은 few-shot / zero-shot 학습에서 뛰어나지만, 기본 계산이나 사실 검색에는 약함
- 이를 해결하기 위해, LLM이 **Tool 사용을 self-supervised 방식으로 학습하는 방법 (Toolformer)** 제안
- Toolformer는 각 API 호출 시기, 인자, 결과 활용법까지 스스로 학습
- 몇 개의 데모 예시만으로 다양한 tool (QA, 계산기, 검색, 번역, 캘린더) 통합
- 결과 : 여러 task에서 zero-shot 성능이 크게 향상, 더 큰 모델과 경쟁 가능. 언어 모델링 능력 손실 없음.

# 1 Introduction

LLM 한계 :
- 최신 정보에 접근 불가
- hallucination
- 저자원 언어 이해 어려움
- 정확한 계산 수행하는 수학적 능력 부족
- 시간의 흐름에 대한 인식 부족
→ 외부 Tool 사용하면, 해결됨

기존 접근법 : Tool 광범위하게 사용 힘듬
1. 대규모 human annotations
2. 특정 task에만 제한된 tool 사용

Toolformer 목표 :
1. self-supervised 방식으로 학습
	-  human annotations 불필요 : *annotation-free*
	- 모델이 유용하다고 느끼는 것이 인간과 다를 수 있음 : *model-centric utility*
2. 일반성 잃지 않고, 언제/어떤 Tool을 사용할지를 스스로 결정할 수 있어야함
	- 특정 task에 묶이지 않고, 포괄적인 Tool 사용 : *task-agnostic, general-purpose tool use*

이 논문의 접근법 :
- 대규모 LMs의 **in-context learning을 이용해 데이터셋을 처음부터 생성**하는 방식에 기반
- API 사용 예시 몇 개만 주면, LLM이 대규모 LM 데이터셋을 잠재적 API 호출로 annotation
- 그 다음, self-supervised loss로 어떤 API 호출이 실제로 모델의 **미래 토큰 예측에 도움이 되는지 판별** *(Perplexity reduction 기준)*

[API 호출 예시]
![](<./Images/Toolformer_Figure.png>)
이런 식으로 연구자가 API 사용 예시 몇 개만 줌 

# 2 Approach

- 목표 : 언어모델 M에 API 호출을 통해 다양한 툴을 사용할 수 있는 능력 부여

- 원본 텍스트 데이터 셋 :  $\mathcal{C} = \{ x^{(1)}, x^{(2)}, \ldots, x^{(|\mathcal{C}|)} \}$
	여기서 $x$ : 토큰 시퀀스
- API 호출: $c = (a_c, i_c)$ *tuple*
	$a_c$ : API 이름, $i_c$ : API 입력

- API 호출 시퀀스 (토큰 시퀀스 형태) → 텍스트 안에서 자연스럽게 삽입
	  $e(c) = \langle \text{API} \rangle \, a_c(i_c) \, \langle / \text{API} \rangle$

- 결과 포함 
	  $e(c, r) = \langle \text{API} \rangle \, a_c(i_c) \, ! \, r \, \langle / \text{API} \rangle$
	ex) `<API> Calculator(2+2) ! 4 </API>`

Given 일반 텍스트 $C=\{x_1​,…,x_{∣C∣} ​\}$ 
API 호출 삽입된 데이터셋 $C^{*}$

API 호출 삽입된 데이터셋 $C^{*}$ 만든는 과정
![](<./Images/Toolformer_Figure-1.png>)
모델이 문장 읽다가, '툴 사용하면 도움이 될 것 같은' 지점에서 API 호출 삽입
→ API 호출과 그 결과가 미래 토큰 예측을 더 잘하면 (=cross entropy loss ↓), 그 호출은 유용하다고 판단, 반대로 도움 안되면 버려짐


#### 1. Sampling API Calls
모델이 문장을 읽다가, “여기서 툴을 쓰면 도움이 될 것 같다” 싶은 지점에서 `[API(...)]` 같은 호출을 삽입

[QA API prompt]
![](<./Images/Toolformer_Figure-2.png>)
각 API 마다, LM이 example $\text{x} = x_1, . . . , x_n$에 API 호출을 추가하도록 유도하는 prompt P(x)를 LM에 넣음

*(GPT)*
→ **prompt는 API 별로 독립적으로 진행. 각 Tool 마다 별도의 prompt가 존재하고, 그 Tool의 예시만 들어감**


각 토큰 위치에서 현재의 입력 $x_{1:i-1}$ 를 바탕으로 모델 M이 위치 $i$에서 `<api>` 호출을 시작할 확률 :
$p_i = p_M\big(\langle \text{API} \rangle \mid P(\text{x}), x_{1:i-1}\big)$
→ API 호출을 수행할 $k$개의 후보 위치를 샘플링

- 모델 M에게 API 사용법이 적힌 소수의 예시로 구성된 프롬프트 P(x) 보여줌
- 이 프롬프트와 원래 텍스트 x 부분 입력 받으면, 각 위치 i에서 $p_{i}$ 계산
- 이 중 확률이 일정 임계값 $\tau_s$ 보다 높은 위치를 샘플링 후보 위치로 고름 (Top $k$개)  → **API call 후보 위치 집합** $I = \{i | p_i > \tau_{s}\}$
- 각 위치 i에서 모델 M에서 $[P(x), x_1, . . . , x_{i-1}, <API>]$부터 시작해서 $\text{</API>}$ 까지 샘플링 → 그 위치 i에서 모델이 생성할 **최대 m개의 API 호출 후보들** $c_i^1, . . . , c_i^m$ : *어떤 API call 넣을지 샘플링*
- 여러 위치에 다양한 API 호출 후보 ${c_{i}}^{j}$ 만들어짐

모델이 “여기서 이런 API 호출을 하면 도움이 될 것 같다”는 후보들을 많이 만들어 냄
아직 호출 결과는 포함하지 않고, 호출 입력(예: 질문 문장)만 만들어 냄



#### 2. Executing API Calls
API 실행하고 결과 얻음. 

- API에 전적으로 의존
- ex) 다른 신경망 호출, 파이썬 스크립트 실행, 검색 수행(retrieval system)
- 각 API 호출 $c_i$에 대한 응답은 단일 시퀀스 $r_i$

#### 3. Filtering API Calls
“API 호출 결과 덕분에 미래 예측이 얼마나 쉬워졌는가?” 를 수치로 판단해서,  
도움되지 않는 호출들은 과감히 제거하고, 도움되는 호출만 남겨 재학습 데이터로 활용함.

- $i$ : API 호출 $c_i$의 위치
- $r_i$ : API 호출 결과
- $z$ : **모델이 미리 받은 prefix (예: API call + 결과)**
- $w_{j-i}$ : 위치별 가중치 (앞부분 토큰보다 *API 근처 토큰에 더 높은 weight*를 줄 수 있음)
- $L_i​(z)$: **모델이 i 이후 토큰들을 예측할 때의 weighted cross-entropy loss**
$$L_i(\mathbf{z}) = - \sum_{j=i}^{n} w_{j-i} \cdot \log p_M(x_j \mid \mathbf{z}, x_{1:j-1})$$
→ API call을 추가하면 모델이 뒤 단어들을 맞추는 데 도움이 되는지를 loss로 측정하는 것

(prefix : 모델에게 토큰 시퀀스 미리 입력으로 주는 것)

[Case 1] API call + 결과 포함 (prefix = e(c,r)) 
```yaml
prefix: Out of 1400 participants, 400 (or <API> Calculator(400/1400) ! 0.29 </API>)
target to predict: 29%) passed the test.
```

[Case 2] API call만 있고 결과 없음 (prefix = e(c))
```yaml
prefix: Out of 1400 participants, 400 (or <API> Calculator(400/1400) </API>)
target to predict: 29%) passed the test.
```
→ 결과값을 예측하기 어려움

[Case 3] 아예 API call 없음 (prefix = ε)
```yaml
prefix: Out of 1400 participants, 400 (or 
target to predict: 29%) passed the test.
```
→ 모델 안 자체 지식만으로 맞춰야함

- $L_i^{+}$ : Case 1 → API 호출과 **그 결과까지** prefix로 줬을 때 loss
$$L_i^{+} = L_i(\mathbf{e}(c_i, r_i))$$

- $L_i^{-}$ : Case 2,3 중에 더 작은 loss
$$L_i^{-} = \min (L_i(\varepsilon), L_i(\mathbf{e}(c_i, \varepsilon)))$$

[필터링 조건]
$$L_i^- - L_i^+ \geq \tau_f$$
**API call + 결과**를 넣은 경우가 “API call 없음(or 결과 없음)”보다 loss를 **최소 $\tau_f$​만큼 줄여줘야** 그 API call을 유용하다고 간주하고 dataset에 포함


이렇게 필터링된 API 호출과 결과를 원본 텍스트에 적절한 위치에 삽입해 새로운 학습 데이터 C* 만듬. → 이 데이터로 언어 모델 다시 finetuning → 모델은 “언제, 어떤 도구를, 어떻게 쓰면 성능이 개선되는가”를 스스로 배우게 되고,  실제 추론(inference) 시에는 필요한 순간에 API를 호출할 수 있게 됨.

#### 4. Model Finetuning

앞에서 필터링하고 남은 API 호출들을 원본 텍스트(입력) 사이사이에 끼워넣음 → $C^*$ 

입력 시퀀스 : $\text{x} = x_1, . . . , x_n$ 에서 위치 i에 대응하는 API 호출 결과 $e(c_i, r_i)$ 있을 때
→ 새로운 시퀀스 : $x^* = x_{1:i−1}, e(c_i, r_i), x_{i:n}$

여러개 API 호출에도 같은 방식으로 적용 가능!

이 새로운 데이터셋 $C^*$ 로 모델 M 언어모델링 목적함수로 파인튜닝
(여기서 달라진 것은 API 호출만. 안에 context 내용은 동일)
→ 자기 피드백만을 바탕으로 어떤 툴을 언제, 어떻게 사용할지 스스로 결정


#### Inference
- 파인튜닝 후 텍스트 생성할 때, 모델 M은 평소처럼 토큰 하나씩 디코딩
- `“→ ”` 토큰 나오면, 디코딩 잠시 중단하고 → 실제 API 호출해서 결과 가져옴
- 그 결과를 문장 안에 삽입하고, `</API>` 토큰까지 붙여서 → 다시 디코딩 이어감
- 학습할 때는 최대 k개 API call이 가능했으나, 추론 시에는 입력에 대해 최대 1회의 API call만 허용
![](<../../../../LLM_research/Fin_Tool/Images/Figure.png>)


[정리]
- **"→ " 토큰 = 툴 결과 요청 신호**
- 이때 inference 루프를 잠시 멈추고 → API 실행 → 결과 + `</API>` 삽입
- 그 다음 다시 디코딩 이어가기


# 3 Tools

- Toolformer는 5가지 Tool (QA, Wikipedia Search, Calculator, Calendar, Machine Translation) 을 대상으로 학습
- Tool 선정 조건 : 
	1. 입력, 출력 모두 텍스트 시퀀스로 표현 가능
	2. 그 툴이 어떻게 쓰이는지에 대한 몇 개의 예시(prompt 안에 포함되는)가 있어야함

[예시]
![](<./Images/Toolformer_Figure-3.png>)


### Question Answering
Atlas : Natural Questions에서 파인튜닝된 검색 augmented LM

### Calculator
사칙연산. 소수점 이하 두자리로 반올림

### Wikipedia Search
Wikipedia에서 짧은 텍스트 snippet 반환하는 검색 엔진
검색을 통해 주제에 대한 보다 포괄적인 정보를 얻을 수 있지만, 관련 부분 스스로 추출해야함.
BM25 검색기 사용

#### QA vs Wiki Search
- QA : 질문 입력하면 바로 정답 반환
- Wiki Search : 검색어 입력하면 원문 텍스트(snippet) 반환. 모델이 그 안에서 필요한 정보 스스로 뽑아야 함

### Machine Translation System
모든 언어 구문 → 영어로 번역
200개 언어 작동, 다국어 기계 번역 모델

- source 언어는 fastText 분류기 사용 → 자동으로 감지. 
- 대상 언어는 항상 영어

### Calendar
쿼리 입력 받지 않고 현재 날짜 반환하는 달력 API
시간에 대한 인식 필요한 예측에 시간적 컨텍스트 제공



# 4 Experiments
## 4.1 Experimental Setup
#### Dataset Generation
- 기본 코퍼스: **CCNet** (Wenzek et al., 2020) 서브셋 사용 → dataset $C$
- 언어모델: **GPT-J (6.7B 파라미터)** (Wang & Komatsuzaki, 2021)
- 효율성을 위해 **heuristic filter**를 적용해 특정 툴에 더 유용할 가능성이 높은 문장만 추림
    - 예: 계산기 API는 숫자가 3개 이상 있는 문장만 고려
- API call 샘플링 → 실행 → 필터링 → 불필요한 예시 제거 → 최종 데이터셋 $C^*$

[Filtering을 위한 Loss]
$$L_i(\mathbf{z}) = - \sum_{j=i}^{n} w_{j-i} \cdot \log p_M(x_j \mid \mathbf{z}, x_{1:j-1})$$
- API call 근처 토큰일수록, $w_{j-i}$ 큰 값 → API 결과가 바로 도움이 되는지 평가


[Weight] API call이 **바로 다음 몇 토큰 예측**에 미치는 영향 평가
$$
w_t = \frac{\tilde{w}_t}{\sum_{s \in N} \tilde{w}_s} \text{ with } \tilde{w}_t = \max(0, 1 - 0.2 \cdot t)
$$
- t가 커질수록, ${\tilde{w}_t}$ 가 1에서 0까지 줄어들고, 5토큰 이후로는 0이 됨. 
- API 호출 직후의 근처 단어들에 더 큰 가중치 부여
→ API 호출이 바로 뒤 단어 예측에 도움이 되도록 유도

#### Model Finetuning
- Batch size = 128
- Learning rate = $1 \times 10^{-5}$ (1e-5) , linear warmup (처음 10%)
- Training steps = 최대 2000 (dev set PPL 기반 체크포인트 선택)
- DeepSpeed ZeRO-3 + 8x A100 GPU (BF16)
- Dataset : CCNet subset → dataset $C$


#### Baseline Models
- **GPT-J**: 원래 GPT-J (6.7B), 파인튜닝 없음
- **GPT-J + CC**: GPT-J를 CCNet subset $C$로 파인튜닝
- **Toolformer**: GPT-J를 증강 코퍼스 $C^*$로 파인튜닝
- **Toolformer (disabled)**: Toolformer인데, inference 때 API 호출 비활성화
    - → API call을 전혀 쓰지 않게 강제
- 비교용 대형모델: **OPT-66B**, **GPT-3 (175B, davinci variant)**

##### What ?! Why?! GPT-J
[What]
GPT-J : 6.7B parameters autoregressive language model
- 오픈소스로 공개된 가장 큰 수준의 GPT류 모델 중 하나
- GPT-3 (175B, OpenAI)보다 작지만, 공개돼서 자유롭게 연구/실험에 활용 가능

[Why]
- 너무 작은 모델(GPT-2급)은 tool use 능력이 emergent하게 나타나지 않음
- 너무 큰 모델(GPT-3급)은 공개가 안 되어 실험 불가
- 6.7B 규모는 연구하기에 충분히 크고, yet computationally feasible
- GPT-J는 공개돼 있어서 fine-tuning, 내부 수정 자유롭게 가능

## 4.2 Downstream Tasks
- **Zero-shot setting**: 태스크별로 in-context 예시를 주지 않고, 단순히 자연어 프롬프트만 제공
- 기존 연구들과 차이점:    
    - 다른 연구들은 태스크별로 “툴을 어떻게 써야 한다”는 예시를 줬지만,
    - Toolformer는 **사용자가 어떻게 툴을 쓸지 지정하지 않아도 스스로 적절한 툴을 호출**
- **Decoding 방식**: greedy decoding 변형
	- Greedy decoding : 확률이 가장 높은 토큰 하나만 선택
    - `<API>` 토큰이 top-10 후보 안에 있으면 툴 호출 허용 (k=10)
    - 입력 하나당 최대 1번만 API call 허용 (무한 루프 방지)


### 4.2.1 LAMA
- 데이터셋: LAMA subsets (SQuAD, Google-RE, T-REx)
- Task: 사실적 지식 채우기 (“Pittsburgh is also known as ___ ” )
- 조건: Toolformer가 Wikipedia Search API를 쓰면 unfair advantage (LAMA 데이터가 위키피디아 기반이니까,,) → 사용 금지
- 결과:
    - GPT-J baseline: 약 17~33점 수준
    - Toolformer: **33.8 (SQuAD), 11.5 (Google-RE), 53.5 (T-REx)**
    - GPT-3 (175B): 26.8 / 7.0 / 39.8

![](<./Images/Toolformer_Figure-4.png>)
- GPT-J 6.7B + Toolformer → **175B GPT-3보다도 성능 우수**
- 모델의 크기가 작아도 외부 Tool을 쓰면 더 성능이 좋음

### 4.2.2 Math Datasets
- 데이터셋: 세 가지 수학 문제 벤치마크 (ASDiv, SVAMP, MAWPS)
- Task: 숫자 연산 문제 풀이
- 결과:
    - GPT-J baseline: ~7–10점
    - Toolformer (disabled): 약 15점 → API 예시 학습만으로도 수학력 조금 향상
    - Toolformer (API 사용): **40.4 (ASDiv), 29.4 (SVAMP), 44.0 (MAWPS)**
    - GPT-3 (175B): 14.0 / 10.0 / 19.8


![](<./Images/Toolformer_Figure-5.png>)
- 계산기 API 덕분에 **GPT-3보다도 훨씬 뛰어난 수학 성능** 확보
- 97.9% 예제에서 계산기 API 호출 → 모델이 계산은 외부 Tool에 맡기는 게 유리하다고 판단
- API를 비활성화한 Toolformer도 성능 향상을 일부 보임

### 4.2.3 Question Answering
- 데이터셋 : WebQS, NQ, TriviaQA
- Task : 일반 질문에 답변
- 설정 : QA 툴 사용 금지 (Atlas 기반 QA 모델과 데이터셋 겹치므로 unfair) → 대신 Wikipedia Search API 사용
- 결과:
    - GPT-J baseline: ~18 / 13 / 44
    - Toolformer: **26.3 (WebQS), 17.7 (NQ), 48.8 (TriviaQA)**
    - GPT-3 (175B): 29.0 / 22.6 / 65.9

![](<./Images/Toolformer_Figure-6.png>)
- GPT-J 기반 모델 대비 명확한 향상, 하지만 여전히 GPT-3보다 낮음
    - 이유: Toolformer는 검색 결과를 refine하거나 여러 결과를 browse 못함 (단일 호출 제한)
	- → 도구의 품질, 상호작용 설계가 전체 성능 좌우


### 4.2.4 Multilingual Question Answering
- 데이터셋: MLQA (질문은 6개 언어: Es, De, Hi, Vi, Zh, Ar, context는 영어)
- 질문을 영어로 이해하고 답변해야 하므로, 
- Tool : 기계 번역(MT) API 사용
- 결과:
    - Toolformer는 스페인어, 베트남어 등에서 baseline GPT-J보다 향상
    - GPT-3와 OPT는 전반적으로 더 안 좋음 (영어로 답 못하는 경우 많음)

![](<./Images/Toolformer_Figure-7.png>)
- 언어별 성능 편차 큼
- MT tool은 대부분 언어에서 60~95% 활용되었지만, Hindi에서는 7.3%만 사용됨
- → 모델이 번역이 필요하다는 것을 잘 감지 못함
- GPT-J > GPT-J + CC : CCNet 파인튜닝으로 일부 언어 성능 저하 (distribution shift 문제)
- 번역 API 사용으로 다국어 QA 향상 가능성 확인, 하지만 **distribution shift** 문제로 성능이 항상 개선되지는 않음
	- distribution shift 문제는 새 데이터로 학습하면서 원래 잘하던 부분(특정 언어 능력)이 망가진 것

### 4.2.5 Temporal Datasets
- 데이터셋:
    - **TEMPLAMA**: Wikidata 기반, 시간에 따라 바뀌는 사실 (“Ronaldo plays for ___ ”)
    - **DATESET**: 새로 제작, 날짜/요일 계산 문제 (“30일 전은 무슨 요일이었나?”)
- Tool : Calendar API
- 결과:
    - Toolformer: TEMPLAMA에서 개선 (16.3 vs baseline ~13), 하지만 Calendar는 거의 사용 안 함 → 주로 QA/검색 툴로 개선
    - DATESET에서는 Calendar API 덕분에 성능 대폭 향상 (27.3 vs baseline ~4–6), API 사용 비율 54.8%

![](<./Images/Toolformer_Figure-8.png>)
**시간 계산이 필요한 경우 Calendar API 활용**이 크게 도움 됨

## 4.3 Language Modeling
- 목적: 툴 학습이 LM 본연의 성능(PPL)에 악영향을 주는지 확인
- 평가 데이터: WikiText, CCNet (훈련에 쓰이지 않은 부분)
- 결과:
  - GPT-J: 9.9 / 10.6
  - GPT-J + CC: 10.3 / 10.5
  - Toolformer (disabled): 10.3 / 10.5
- 결론:
  - Toolformer는 API call 삽입 데이터셋으로 학습해도 **언어모델링 능력 손실 없음**
  - 툴 사용 능력 + 언어 이해 능력 모두 유지

## 4.4 Scaling Laws

GPT-2의 4개 소형 모델(124M, 355M, 775M, 1.6B) VS GPT-J(6.7B) 비교
![](<./Images/Toolformer_Figure-9.png>)

- 소형 모델 (124M, 355M) : tool을 써도 성능 차이가 거의 없음
- 775M 이상부터 tool 사용이 유의미하게 성능 향상, GPT-J (6.7B)에서 가장 뚜렷한 개선
- Toolformer (disabled)도 API 없이 베이스라인보다 개선됨
- 계산력 부족 문제는 scaling으로 해결 X → 외부 tool이 훨씬 효과적

# 5 Analysis
#### Decoding Strategy
- Greedy decoding (k=1) : 확률이 가장 높은 API Call 하나만 선택

k 값에 따른 API call 빈도 & 성능 변화
![](<./Images/Toolformer_Figure-10.png>)
- k = 1 : 일부 case에서만 API call 발생. 선택적인 사용
- k ↑ (3,10) : API call 사용률 급증 (최대 100%)
- 성능은 일정 수준까지 개선되지만, 너무 많이 호출하면 noise 발생 (+ 비용 증가)


#### Data Quality
- **Filtering 기준: API call이 loss를 줄여야 학습 데이터에 포함**
- 대부분 유용한 call만 남지만, 일부 noise도 존재
- But, Noise는 모델이 결과를 그대로 따르지 않고, 판단력을 갖게 만듦

filtering 단계가 있기에, 유용한 API call만 남겨서 Toolformer가 self-supervised로 효율적인 학습 데이터 C* 을 만들 수 있음 !

![](<./Images/Toolformer_Figure-11.png>)


# 6 Related Work


# 7 Limitations
- Tool chaining 불가
	- 한 Tool의 output을 다른 Tool의 input으로 연결 불가

- Interactive tool use 불가
	- 검색 결과가 부적절해도 쿼리를 수정하거나 여러 결과 탐색 불가

- Sample efficiency 낮음
	- C* 만들 때, 수백만 문서 처리해도 실제로 유용한 call은 소수

- Prompt sensitivity
	- Prompt 입력 문장의 wording에 따라 API 호출 여부 달라짐

- API 호출 비용, 효율성 고려 X
	- API 마다 호출 비용이 다르지만 이를 무시 →  불필요한 호출 발생 가능


# 8 Conclusion
- Toolformer 기여:
	- Self-supervised 방식으로 tool 사용법 학습 가능 
	- 작은 모델(6.7B GPT-J)도 tool 사용 시 GPT-3(175B) 능가 (특히 수학, factual QA)

- Future Work
	- Tool Chaining
	- Interactive Tool use : 검색 쿼리 수정, 여러 결과 탐색
	- Sample efficiency 개선 (더 적은 데이터로 효과적 학습)
