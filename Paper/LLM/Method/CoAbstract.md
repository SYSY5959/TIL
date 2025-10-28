# Efficient Tool Use with Chain-of-Abstraction Reasoning

COLING 2024

# Abstract

문제 의식 :
- Toolformer 같은 기존 tool-augmented LLM은 multi-step reasoning에서 **연결된 툴 호출의 비효율**과 **응답 대기 시간 문제**로 성능 저하

**Chain-of-Abstraction (CoA)** 제안
- LLM이 먼저 **abstract placeholder으로 reasoning chain 생성**
- 이후 **외부 도메인 tool (계산기, Wiki 검색)로 placeholder 채워 넣기**
→ 일반적 reasoning 전략 학습 + 지식 변화에 robust
→ tool 호출과 LLM 디코딩을 병렬 처리 가능 → 시간 지연 최소화

실험 결과:
- 도메인 : 수학적 reasoning, Wikipedia QA
- 성능 : 
	- in-distribution + out-of-distribution 모두 향상
	- QA 정확도 평균 **+6%** 개선
	- 추론 속도 **1.4×** 빨라짐

의의 :
LLM이 multi-step reasoning에서 더 정확하고 더 빠르게 도구 사용할 수 있는 프레임워크 제안


# 1 Introduction

Toolformer와 같은 기존 접근 한계: 
multi-step reasoning에서
- tool 호출 간 연계 부족 → planning 실패
- tool 호출 대기 시간 → 추론 속도 저하
→ 복잡한 reasoning 문제에서 성능 떨어짐

CoA 제안 :
- 추상 placeholder 기반 reasoning chain 생성 → 이후 한 번에 tool 호출하여 구체화
- 일반적 reasoning 전략 학습 + 병렬 처리로 효율성 개선

[CoA 추론 과정]
![](<./Images/Pasted image 20250903152235.png>)


# 2 Related Work

### Tool-Augmented LLMs

#### In-context 기반 접근법 : 
- 프롬프트에 툴 사용 예시를 넣어 LLM이 도구를 활용하도록 유도
- 한계 : 프롬프트 엔지니어링에 크게 의존, 모델의 instruction-following 능력에 제한됨

#### Fine-tuning 기반 접근법 :
- [@ Toolformer](<../../../LLM_research/Fin_Tool/@ Toolformer.md>) (Schick, 2023), **TALM** (Parisi, 2022), **ToolkenGPT** (Hao, 2023b)
- LLM을 툴 사용에 맞게 직접 학습시켜 더 robust한 성능
- 한계: 추론 중 API 호출을 **순차적으로(interleaved)** 수행해야 해서, latency와 호출 횟수만큼 속도가 느려짐

#### Multi-step Reasoning 
- **ReAct** (Yao, 2023b) & **FireAct** (Chen, 2023)
	- LLM이 thought → action → observation 루프를 반복.
    - 장점: 복잡한 reasoning 가능.
    - 한계: reasoning 루프가 길어져 **속도 저하**, 여전히 sequential API 호출 구조

#### Program-based Reasoning
- **PoT, DECLARATIVE, PAL**
	- LLM이 reasoning을 프로그램(코드) 형태로 생성 → 코드 실행기와 상호작용
    - 장점: 계산 정확성 확보
    - 한계: 폐쇄형 코드 모델(Codex)에 의존, 수학/절차적 문제에만 적합

### Tool Usage Planning
- **HuggingGPT, Chameleon, OpenAGI, MetaTool** → 다중 도메인 태스크에서 **tool 사용 순서 계획**
- **LATM, ML-BENCH, Gorilla** → 여러 API를 조합해 **프로그램·스크립트 생성**
- **ToolChain*** → 트리 탐색 기반으로 툴 사용 계획을 최적화, 절차적 작업에 강점

- **CoA의 차별점**: 특정 도구 조합/절차적 실행이 아니라, **chain-of-thought reasoning 자체를 추상화하여 계획**


# 3 Method

### Chain of Abstraction (CoA) Reasoning
- **일반 추론(Reasoning strategy)** 과 **툴로 얻는 값** 을 분리
	→ 추론과 tool 호출을 병렬화(pipelining) → inference 속도 향상
- LLM은 **추상 placeholder (y1, y2 …)** 로 reasoning chain을 생성
- 외부 tool(계산기, Wiki 검색 등)이 placeholder를 실제 값으로 채워 넣음 → 최종 답변 생성

CoT가 아닌, 추론의 abstract chain 생성
→ 모델이 값 계산/사실 회상 대신 **추론 전략 자체를 학습**할 수 있음
→ domain shift 에 잘 대응

### Fine-tuning Data Construction

1. 기존 QA 데이터셋에서 Gold Answer 수집

2. LLaMa-70B에게 Gold Answer → **Abstract chain으로 재작성** 시킴
    - 연산 결과(숫자, 엔티티)는 placeholder(y1, y2 …)로 교체
    - 같은 결과는 항상 동일한 placeholder로 표시 → step 간 연결 보장

3. **도메인 툴로 검증**
    - 계산기/검색기로 placeholder를 실제 값으로 대체
    - 올바른 답이 나오면 **학습 데이터로 채택**
    - 오류가 있으면 폐기


[CoA 학습 데이터 생성 과정]
![](<./Images/Pasted image 20250903153832.png>)
- Question : 원래 데이터셋에 있는 질문
- Gold Answer : 원래 데이터셋에 포함된 정답
- Abstract Reasoning Chain : LLM이 gold answer를 **추상 placeholder(y1, y2, …)** 로 다시 쓴 CoA reasoning chain → *CoA 학습 데이터*
- Reified Chain : 도메인 tool이 placeholder를 실제 값으로 채워 넣은 것 → *검증 도구*


# 4 Experimental Settings

## 4.1 Mathematical Reasoning
- **Task**: 수학 문제를 단계적 산술 전개로 풀어내는 reasoning
- **데이터**:
    - Fine-tuning: GSM8K(7,473문제), ASDiv(691문제, 단일 단계 문제 보완)
    - Evaluation: GSM8K, ASDiv (in-distribution), SVAMP, MAWPS (out-of-distribution)
- **도구**: Equation Solver (SymPy) → CoA trace의 식들을 연립방정식으로 풀어 placeholder 채움
- **검증 결과**: 생성된 CoA trace 중 약 76.6%만 유효 판정 → 나머지는 폐기

## 4.2 Wikipedia QA
- **Task**: Wiki 지식을 multi-hop으로 연결해 답을 찾는 reasoning
- **데이터**:
    - Fine-tuning: HotpotQA (113K 예시 중 Bridge QA 72,991, Comparison QA 17,456)
    - 검증 통과 후 최종 사용: 8,956 (Bridge), 5,405 (Comparison)
    - Evaluation: HotpotQA dev set + WebQuestions, NaturalQuestions, TriviaQA (out-of-distribution)
- **도구**:
    - WikiSearch (BM25 + SBERT re-rank)
    - NER (SpaCy → 6개 general type으로 단순화)
	    - SpaCy : Python 기반의 유명한 NLP 라이브러리
		- NER : 문장에서 인물(Person), 조직(ORG), 장소(LOC), 날짜(DATE) 같은 **고유명사(Entity)를 인식하고 라벨링하는 작업**
- **검증 통과율**: 약 15.9%만 학습 데이터로 사용 가능 (수학보다 훨씬 낮음)

![](<./Images/Pasted image 20250904123036.png>)

## 4.3 Baselines
- **CoT-FSP**: Few-shot prompting with chain-of-thought
- **CoT-FT**: Fine-tuning with chain-of-thought traces
- **Toolformer**: Schick et al., 2023 (원래는 CCNet 데이터로 FT)
- **Toolformer-Math / Toolformer-Wiki**: in-domain CoT data로 다시 FT한 버전
- **FireAct**: HotpotQA ReAct trajectories 기반 FT

# 5 Results and Analysis

## 5.1 Mathematical Reasoning

![](<./Images/Pasted image 20250904125156.png>)

- **메인 결과**: CoA가 모든 baseline보다 consistently 우수
    - GSM8K, ASDiv (in-distribution) → 정확도 +2~5%p 향상
    - SVAMP, MAWPS (out-of-distribution) → 분포 바뀌어도 robust, 특히 7B 모델에서 CoT-FT보다 훨씬 좋음
- **Toolformer 대비**:
    - Toolformer는 sequential 호출이라 accuracy & speed에서 CoA에 밀림.
    - CoA는 Toolformer-Math보다도 consistently 우수.
- **Ablation**: CoA(no Tool) = 툴 안 쓰고 모델이 직접 계산 → 정확도 떨어짐. 툴 활용이 필수임을 확인.
- **Long chain 효과**: reasoning step이 길수록 CoA의 이점이 커짐.
- **Human eval**: Arithmetic error 0%, Reasoning error ~8%p 감소

## 5.2 Wikipedia QA

![](<./Images/Pasted image 20250904125908.png>)

- **메인 결과**: HotpotQA Bridge QA(난이도 높은 multi-hop)에서 CoA가 가장 크게 개선
    - Bridge QA: FireAct, Toolformer보다 높음
    - Comparison QA: 성능은 비슷하지만 속도는 CoA가 더 효율적
- **Generalization**:
    - WebQ, NQ, TriviaQA에서도 좋은 성능. 특히 NQ, TriviaQA에서 baseline보다 우수
- **효율성**:
    - 평균 응답 시간 Toolformer, FireAct보다 짧음
    - 이유: CoA는 긴 위키 문서 생성을 피하고, 추상 chain만 디코딩 → 속도 절약


# 6 Conclusion

- **기여**:
    - LLM의 일반 추론과 도메인 특화 지식을 분리하는 **Chain-of-Abstraction (CoA)** 방법 제안
    - 추상적 reasoning chain을 계획 → 외부 툴로 구체화 → 최종 답변

- **효과**:
    - **정확성↑**: 수학 reasoning(+7.5%p), Wiki QA(+4.5%p)
    - **효율성↑**: 추론 속도 수학 1.47×, Wiki QA 1.33× 빠름
    - **robustness↑**: in-domain + out-of-domain 모두 향상, 복잡한 multi-step 문제에서 특히 강함
    - **human eval**: reasoning 오류 ~8% 감소

- in-domain : 파인튜닝에서 사용된 데이터셋과 같은 분포의 데이터 (**훈련과 같은 종류의 문제**)
→ in-domain 성능은 모델이 학습한 패턴을 얼마나 잘 재현하는지 보여줌.

- out-of-domain : 학습에 포함되지 않은, **다른 분포의 데이터셋**. 모델이 **처음 보는 스타일/도메인**의 문제
→ out-of-domain 성능은 모델이 일반화 능력을 얼마나 갖췄는지를 보여줌.


- **의의**:
    - CoA는 단순한 “툴 호출”을 넘어서, **추상적 계획 → 병렬화된 효율적 툴 사용 → robust reasoning**이라는 새로운 패러다임을 제시
    - 다양한 reasoning 도메인으로 확장 가능성이 큼

# 7 Limitations
1. **도메인 범위 제한**
    - 실험은 수학 reasoning, Wikipedia QA 두 가지 도메인에 집중.
    - 실제 세계의 다양한 reasoning 시나리오를 모두 포괄하지 못함.

2. **언어 범위 제한**
    - 실험은 영어 데이터셋 기반
    - 다국어 환경에서의 일반화는 확인되지 않음

3. **학습 비용**
    - 논문에서는 full fine-tuning으로 CoA를 학습 → 계산 자원 많이 필요
    - 더 효율적인 학습법(e.g., LoRA, parameter-efficient tuning)은 향후 과제



