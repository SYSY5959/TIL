# A Survey of Large Language Models in Finance (FinLLMs)
Neural computing & applications (Print) 2024

# 1 Introduction

- LLM(ChatGPT 등): Transformer 기반, 대규모 텍스트 학습 → 다양한 NLP 과제 성능 향상
- 금융 특화 LLM(FinLLM) 연구 초기 단계
	→ 대부분 일반 LLM + Prompt Engineering 또는 Instruction Fine-tuning 수준

[이 논문의 기여]
1. 일반 LLM → 금융 LLM 진화사 정리
2. 금융 특화 모델의 학습 방식, 데이터, Fine-tuning 비교
3. 6개 금융 NLP 벤치마크 성능 요약
4. 8개 고급 금융 NLP 과제 제안
5. 기회와 도전과제 

---

# 2 Evolution Trends: from General to Finance

![](<./Images/Pasted image 20250814163208.png>)

## 2.1 General-domain LMs

### GPT Series
- **GPT-1(110M, 2018)**: 최초의 Generative Pre-trained Transformer
- **GPT-2(1.5B, 2019)**: 스케일업 효과 입증, 멀티태스크 가능성 확인
- **GPT-3(175B, 2020)**: in-context learning이라는 emergent property 등장
- **ChatGPT(2022)**: GPT-3 + Codex + InstructGPT(RLHF)
- **GPT-4(추정 1.7T, 2023)**: 법률·의학 시험 통과 수준, 멀티모달 가능

### Open-source LLM
- **BERT(2018)**: 이후 FinBERT 같은 도메인 특화 모델의 기반
- **LLaMA(2023)**: 메타가 7B~65B 모델 공개 → Instruction Fine-tuning, Chain-of-Thought Prompting 등 다양한 파생
- **BLOOM(176B, 2022)**: BigScience 프로젝트, 46개 자연어 + 13개 프로그래밍 언어 지원


## 2.2 Financial-domain LMs

- **FinPLM(Pre-trained Language Model)**: FinBERT-19, FinBERT-20, FinBERT-21, FLANG  
    → 주로 BERT/ELECTRA 기반

- **FinLLM**: FinMA, InvestLM, FinGPT, BloombergGPT  
    → LLaMA, BLOOM 등 대형 모델 기반

[발전 방식]
- Continual Pre-training: 일반 → 금융 데이터 재학습
- Domain-specific Pre-training: 금융 데이터로 처음부터 학습
- Mixed-domain Pre-training: 일반 + 금융 데이터 혼합
- 최신 경향: Instruction Fine-tuning + Prompt Engineering

---

# 3 Techniques: from FinPLMs to FinLLMs

![](<./Images/Pasted image 20250814170510.png>)

| 구분         | PLM (Pre-trained Language Model)              | LLM (Large Language Model)                                         |
| ---------- | --------------------------------------------- | ------------------------------------------------------------------ |
| **정의**     | 대규모 말뭉치로 사전학습된 언어 모델(크기 제한 없음)                | 파라미터 수가 매우 큰 PLM (논문에서 보통 **≥ 7B parameters** 기준)                  |
| **모델 크기**  | 수천만~수억(수백M) 파라미터도 포함                          | 수십억~수천억 파라미터                                                       |
| **학습 데이터** | 대규모지만 LLM보다 작은 경우 많음                          | 초대규모 데이터셋(수백B~수T 토큰)                                               |
| **능력**     | 특정 태스크 성능 높음, 범용성 제한 가능                       | **Emergent abilities**(in-context learning, chain-of-thought 등) 출현 |
| **예시**     | BERT-base(110M), RoBERTa(355M), ELECTRA(110M) | GPT-3(175B), LLaMA-65B, BLOOM(176B), GPT-4(~1.7T)                  |
| **사용 방식**  | 주로 파인튜닝(fine-tuning) 필수                       | 프롬프트만으로 zero/few-shot 수행 가능                                        |

## 3.1 Continual Pre-training
- 기존 PLM → 금융 데이터 추가 학습
- 기존 일반 언어 이해력 유지하면서, 금융 지식 습득
- ex) FinBERT-19 (BERT → 금융 sentiment fine-tuning)

## 3.2 Domain-Specific Pre-training from Scratch
- 금융 데이터로 처음부터 학습 (금융 도메인 최적화)
- ex) FinBERT-20 (금융 커뮤니케이션 코퍼스 4.9B 토큰으로 학습)

## 3.3 Mixed-Domain Pre-training
- 일반 + 금융 데이터 함께 학습 → 일반적 언어 이해력과 금융 지식의 균형
- ex) FinBERT-21 (멀티태스크 학습, 일반 3.3B + 금융 12B 단어)

## 3.4 Mixed-Domain LLM with Prompt Engineering
- 일반+금융 데이터로 LLM 학습 → 프롬프트로 과제 수행 (task 자연어로 설명해서 zero/few-shot 활용)
- ex) BloombergGPT (BLOOM 기반, FinPile 금융 데이터)

## 3.5 Instruction Fine-tuned LLM with Prompt Engineering
- 금융 instruction 데이터로 fine-tuning
- 명시적 지시문(instruction)과 예시를 담은 데이터로 추가 학습해 모델의 제어력·응답 품질 향상
- ex)
  - FinMA (LLaMA 7B·30B, FIT 데이터)
  - InvestLM (LLaMA 65B, CFA·SEC·요약 데이터)
  - FinGPT (여러 오픈소스 LLM, LoRA 적용)

|구분|일반 Fine-tuning|Instruction Tuning|
|---|---|---|
|목적|특정 태스크 성능 극대화|지시문 이해 및 다태스크 대응|
|데이터 형식|입력-출력 쌍|**지시문 + 입력-출력**|
|범용성|한정된 태스크|다수 태스크 zero/few-shot 가능|
|예시|"이 문장을 감정분석하라" 없이 감정 레이블 학습|지시문 포함된 대화형 학습|

#### Instruction Data 생성 방식
1. 전수 수작업 : 도메인 전문가가 직접 **Instruction + Input + Output** 작성
2. 기존 데이터셋 변화 : 존재하는 금융 NLP 데이터셋(SA, QA, NER, Summarization 등)을 가져와서,  
**Instruction 부분만 사람 또는 스크립트로 생성**
3. 모델을 활용한 자동 생성 (Synthetic Generation) : GPT-4, ChatGPT 등 강력한 LLM을 사용해 **기존 데이터 → Instruction 포맷 변환** 자동화


---

# 4 Evaluation: Benchmark Tasks and Datasets

6개 금융 NLP Task

![](<./Images/Pasted image 20250814172005.png>)

## 4.1 Sentiment Analysis (SA)
금융 뉴스·마이크로블로그에서 긍정/부정/중립 감정 분류

데이터셋:
- **FPB**: 4,845개 문장, 전문가가 3가지 감정 레이블 부여
- **FiQA-SA**: 1,173개 포스트, 감정 점수(-1~1) → 분류로 변환

## 4.2 Text Classification (TC)
금융 뉴스 헤드라인 등 문서를 사전 정의된 카테고리로 분류
데이터셋: **Headline**(11,412개 뉴스, 가격 방향 등 9개 라벨)

## 4.3 Named Entity Recognition (NER)
금융 문서에서 PER, ORG, LOC 등 엔티티 추출
데이터셋: **FIN**(8개 금융 계약문서)

## 4.4 Question Answering (QA)
- 금융 문서 기반 질의응답, 특히 수치 추론 필요
- 데이터셋:
    - **FinQA**: 단일 턴 하이브리드 QA(표+텍스트)
    - **ConvFinQA**: 다중 턴 대화형 하이브리드 QA

## 4.5 Stock Movement Prediction (SMP)
- 텍스트+가격 데이터 기반 다음날 주가 방향 예측.
- 데이터셋: **StockNet, CIKM18, BigData22** (이벤트 기반 바이너리 분류)

## 4.6 Text Summarization (Summ)
- 금융 문서 요약(추출식·생성식)
- 데이터셋: **ECTSum** (earnings call transcript + Reuters 요약)

## 4.7 Discussion
- **단순 과제(SA, TC, NER)**: Mixed-domain FinPLM + Fine-tuning → 효율적·성능 우수
- **복잡 과제(QA, SMP, Summ)**: Task-specific SOTA가 여전히 우위
- **GPT-4**: 전반적으로 우수하지만 Summ에서는 SOTA보다 낮음

---

# 5 Advanced Financial NLP Tasks and Datasets
더 복잡하고 실무적으로 중요한 금융 NLP task 8가지

1. Relation Extraction (RE)
   - 엔티티 간 관계 식별 (ex. owned by” 관계)
   - 데이터: FinRED (금융 뉴스·earning call transcript 기반, 29개 관계 태그)

2. Event Detection (ED)
   - 기업 이벤트 식별 및 시장 영향 분석
   - 데이터: EDT (11개 이벤트 유형, 뉴스+시계열)

3. Causality Detection (CD)
   - 금융 텍스트에서 인과관계 문장 식별
   - 데이터: FinCausal20

4. Numerical Reasoning (NR)
   - 숫자·연산자 인식, 계산 수행. 금융 문맥 이해
   - 데이터: FiNER-139, FinQA, ConvFinQA

5. Structure Recognition (SR)
   - 표·셀 구조 및 관계 인식. 셀 간의 관계 파악
   - 데이터: FinTabNet (S&P 500 기업 보고서 PDF에서 표 구조 annotation)

6. Multimodal (MM) Understanding
   - 텍스트+시계열+오디오/비디오 등 다중 모델 데이터 이해
   - 데이터: MAEC (earnings call transcript + 시계열 + 오디오), MONOPOLY (중앙은행 정책 발표 영상 + 텍스트 + 시계열)

7. Machine Translation (MT) in Finance
   - 금융 맥락 이해를 반영한 다국어 번역
   - 데이터: MINDS-14 (14개 언어 은행 업무 대화(텍스트·오디오)) , MultiFin (15개 언어 금융 기사, 6대 주제·23개 하위 주제)

8. Market Forecasting (MF)
   - 가격·변동성·위험 예측 (단순 SMP를 넘어서는 과제)
   - 데이터: StockEmotions, EDT, MAEC, MONOPOLY

# 6 Opportunities and Challenges

### 1. Datasets
- 기회: 
	- 고품질·멀티모달 금융 데이터 확보 → 성능 향상
	- 기존 금융 NLP 데이터셋을 **Instruction Dataset**으로 변환 → 고급 FinLLM 개발 가능
- 한계:
	- 금융 데이터는 공개 범위가 제한적, 라이선스와 보안 이슈 존재
    - 다양한 형식·언어로 균형 있게 수집하기 어려움

### 2. Techniques
- 기회:
	- **Retrieval-Augmented Generation (RAG)** 활용 → 사실성·신뢰성 강화, 내부 민감 데이터 활용 가능
    - PEFT(파라미터 효율적 파인튜닝) 등으로 비용 절감 가능
- 한계: 보안·프라이버시 유지, hallucination 최소화

### 3. Evaluation
- 기회:
	- 금융 전문가 참여 평가로 모델 신뢰성 확보
    - 금융 특화 지표(Sharpe ratio, 백테스트 결과 등) 활용 가능
- 한계: 
	- 현재는 NLP 지표(F1, Accuracy) 중심 → 복잡한 금융 과제 평가에는 부적합
    - 고급 금융 벤치마크(논문 5장 제안 8개 과제) 활용 필요

### 4. Implementation
- 기회: 
	- LLMOps(Continuous Integration/Continuous Delivery) 적용 → 운영 효율화
    - Soft Prompt, LoRA 등 저비용 학습 기법 사용 가능
- 한계: 비용-성능 트레이드오프: 단순 과제는 범용 LLM+프롬프트로 충분, FinLLM 필요성 재검토 필요

### 5. Applications
- 기회: robo-advisor, quantitative trading, low-code 금융 앱, 보고서 생성, 문서 이해 등 실무 적용 확대
- 한계: 산업 장벽, 규제, 데이터 프라이버시, 윤리, 전문가 간 이해 격차