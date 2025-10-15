# Fin-R1: A Large Language Model for Financial Reasoning through Reinforcement Learning
arXiv 2025

Github : https://github.com/SUFE-AIFLM-Lab/Fin-R1

# Abstract
- 7B 파라미터 금융 reasoning 특화 LLM
- 데이터셋: Fin-R1-Data (60,091 CoT, reasoning/non-reasoning 시나리오) 직접 만듬
- 학습 방식: **SFT + RL (GRPO) 2단계 프레임워크**
- 성과:
	- ConvFinQA 85.0, FinQA 76.0 → SOTA
	- 평균 75.2, 전체 2위 (DeepSeek-R1-Distill-Llama-70B보다 우수)


# Why Fin-R1?
기존 **general-purpose reasoning LLM**은 금융 도메인 적용 시
1. fragmented financial data : 서로 다른 출처/형식의 금융 지식이 흩어져 있어서, 통합/전처리가 어렵고 누락/중복이 발생하는 문제
2. reasoning black-box : 규제, 감사 요건이 있는 금융에서는 추론 과정의 해석 가능성 X
3. weak business generalization : 새로운 금융 시나리오에 안정적으로 적용되기 힘듬
→ 실제 금융업계에서 활용하기 어려움

# Fin-R1 Data
LLM을 금융 추론용으로 파인튜닝하기 위해 만든 SFT용 데이터셋

- **60,091 CoT 예제** (reasoning + non-reasoning, 영어·중국어 혼합)
- 출처: FinQA, ConvFinQA, Finance-Instruct-500K, FinCorpus 등 + 자체 제작 **금융대학원 입시 문제(FinPEE)**
- **생성 과정**:
    - **DeepSeek-R1**로 CoT 생성 →
    - **LLM-as-Judge (Qwen2.5-72B)** 로 품질 필터링 →
    - 사람 검증으로 고품질 데이터 구축
        →  금융 reasoning에 최적화된 **고품질 CoT 데이터셋**

## Processing Pipeline

![](<../Method/Images/Pasted image 20250908181759.png>)
- Stage 1 (데이터 구축) : 
	- 입력 데이터 → DeepSeek-R1으로 CoT 생성 → Qwen2.5-72B-Instruct로 Answer Check + Reasoning Selection → Fin-R1 Data 생성
- Stage 2 (모델 훈련) : 
	- SFT : Qwen2.5-7B-Instruct 같은 모델을 Fin-R1-Data로 미세조정
	- RL : GRPO로 정책 최적화, RL 보상 평가에 Qwen2.5-Max 같은 judge를 사용


### (1) Data Distillation (증류)
- 수집한 금융 데이터에서 **DeepSeek-R1**을 활용해 CoT reasoning 생성
- 출력 형식을 통일:
    - 수학 문제 → 답을 `\boxed{}` 로 묶음
    - 출력 시작 시 `\n` 강제 

### (2)Answer Check 
- **DeepSeek-R1이 낸 답 ≠ 정답** → 버림
- **객관식/수치 문제**: 정확 매칭
- **주관식/텍스트 문제**: LLM-as-Judge로 의미적 일치 여부 평가
- Judge 모델 비교: **Qwen2.5-72B-Instruct** (정확도 99.6%) > GPT-4o
    → 따라서 Qwen2.5-72B-Instruct 선택

### (3) Reasoning Selection
- reasoning 과정의 **품질 평가**
- 평가도 LLM-as-Judge(Qwen2.5-72B-Instruct)로 수행
- **낮은 품질의 reasoning trajectory** → 제거

- 평가 기준 7가지:
    1. Internal consistency (내적 일관성)
    2. Term overlap rate (용어 일치율)
    3. Number of reasoning steps (추론 단계 수)
    4. Logical coherence (논리적 일관성)
    5. Content diversity (내용 다양성)
    6. Task-domain relevance (금융 도메인 적합성)
    7. Alignment with instructions (지시문 적합성)

![](<../Method/Images/Pasted image 20250908225958.png>)

## Data Statistics

![](<../Method/Images/Pasted image 20250908182946.png>)

![](<../Method/Images/Pasted image 20250908225800.png>)

# Framework

![](<../Method/Images/Pasted image 20250908181429.png>)
생성(DeepSeek-R1) → 판정(LLM-as-Judge) → 추론품질 필터링 → SFT로 형식화된 CoT 학습 → GRPO로 형식·정확성 보상하며 정책 최적화 → 금융 응용(코드·전문 Q&A·업무 자동화)

SFT에 사용되는 데이터셋 (필터링해서 생존한 애들)
 합계 약 77.9% , 금융 전문지식 약 21.9% , 금융 코드 약 0.2%

## Stage 1 : Supervised Fine-Tuning (SFT)
Goal : 일반 LLM(Qwen2.5-7B-Instruct)을 금융 reasoning 패턴에 맞게 적응


- 데이터 : **Fin-R1-Data** (고품질 CoT 포함)
	 `<think> reasoning trace </think>` + `<answer> final answer </answer>`

- 학습 포맷 : v = (x, c, y*)    *→ 모델이 구조화된 reasoning + 답 동시에 학습*
	- 입력: 질문 (x)
	- 출력: reasoning trace (c) + 정답(y*)

→ 금융 reasoning 패턴/구조 학습

## Stage 2 : Reinforcement Learning (RL, GRPO)
Goal : SFT로 학습된 모델을 형식 + 정답 정확성 강화, 안정적 reasoning 확보

- **알고리즘** : **Group Relative Policy Optimization (GRPO)**
	- PPO(Policy Optimization)의 변형
	- 여러 candidate 출력을 생성 → 그룹 내 평균 대비 상대적 보상으로 업데이트
	- 장점: 안정적이고 효율적인 policy 개선

- 학습 포맷 : v = (x, y*)
	- 입력: 질문 (x)
	- 출력: 답 (y*) (reasoning trace 없음)
	→ **Format reward** + **Accuracy reward** 로 업데이트

- 여기에서 KL divergence : 새 policy와 기준 policy의 분포 차이를 재는 값
- GRPO에서 이 항을 넣어서, 모델이 너무 급격하게 바뀌지 않고 안정적으로 학습되도록 제어

[정리]
- **Reward**: “정답 맞추고 포맷 지켜라”
- **KL penalty**: “하지만 너무 성급하게 확 바꾸지 말고, 조금씩 개선해라”

![](<../Method/Images/Pasted image 20250908230106.png>)

#### GRPO (Group Relative Policy Optimization)

- PPO (Proximal Policy Optimization) : 
→ 하나의 답변을 보고, 그 답이 잘했는지 (보상) 기반으로 policy 업데이트 

- GRPO (Group Relative Policy Optimization) : 
→ PPO를 확장해서, 한 질문에 대해 여러 답변을 동시에 평가하고, 그룹 평균보다 잘한 답변을 더 강화하는 알고리즘
*(이 그룹 안에서 더 잘한 답변이 무엇인가..? 기준)*

[Example]
“2023년에 회사 주가는 $50, 2024년에 $75라면, 성장률은 얼마인가?”

1) 모델 출력 후보
- 후보 A : 
```css
<think> (75 - 50) / 50 = 0.5 = 50% </think>
<answer> 50% </answer>
```

- 후보 B :
```css
<think> (75 ÷ 50) - 1 = 0.5 </think>
<answer> 50% </answer>
```

- 후보 C :
```css
<think> 75 - 50 = 25 </think>
<answer> 25 </answer>
```

- 후보 D :
```css
<answer> 1.5 </answer>
```

2) Reward 할당
- **Format reward**: 태그 사용 여부
- **Accuracy reward**: 정답(50%)과 일치 여부

| 후보  | Format | Accuracy | 총 보상 |
| --- | ------ | -------- | ---- |
| A   | 1      | 1        | 2    |
| B   | 1      | 1        | 2    |
| C   | 1      | 0        | 1    |
| D   | 0      | 0        | 0    |

3) 그룹 상대 평가 
- 평균 보상 = (2+2+1+0) / 4 = 1.25
- A, B > 평균 → 강화 : 답변 내는 확률 더 높아짐
- C, D < 평균 → 약화
