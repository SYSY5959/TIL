# A Multimodal Foundation Agent for Financial Trading: Tool-Augmented, Diversified, and Generalist
KDD 2024

https://arxiv.org/pdf/2402.18485v1

# Abstract 
- 금융 트레이딩은 뉴스, 가격, 차트 등 **멀티모달 데이터**를 필요로 함
- 기존 딥러닝, RL 기법의 한계 : 멀티모달 처리 능력 부족, 일반화 부족

**FinAgent 제안** : 
- Market Intelligence Module: 텍스트·수치·시각 데이터 통합 분석
- Dual-level Reflection: low/high reflection + memory retrieval
- Tool-Augmented Decision-making: 전통적 전략 + 전문가 인사이트
- Action reasoning 제공 → 신뢰성 향상

실험 성과 :
- 6개 데이터셋 (주식 + Crypto)에서 9개 SOTA 대비 평균 +36% 수익
- TSLA 데이터셋: **92.27% 수익률 (84.39% 개선)**

의의 : 금융 트레이딩을 위한 최초의 **멀티모달 Foundation Agent**

# 1 Introduction
금융 트레이딩 기존 접근의 한계
- Rule-based: 시장 변화에 적응 불가 → 성과 저하
- RL-based: 데이터 요구량 큼, explainability 부족, 노이즈 민감
- 멀티모달 정보(뉴스, 가격, 차트 등) 통합 실패

최근 발전 
- LLM 에이전트: 메모리·계획 모듈로 동적 환경 적응
- 멀티모달 LLM (예: GPT-4V): 텍스트+시각 데이터 처리 가능
- Tool-augmented 모델 (Toolformer 등): 외부 툴 활용 능력

도전 과제 : 
1. **Multimodal data processing 부족**  → Market Intelligence Module
2. **목표 지향적 정보 검색 부족**  → Dual-level Reflection
3. **급변하는 시장 적응력 부족**  → Diversified Memory Retrieval
4. **도메인 지식(전략·전문가 조언) 통합 어려움**  → Expert Knowledge + Reasoning 제공
5. **Explainable reasoning 부족**  → Expert Knowledge + Reasoning 제공

FinAgent 제안 :
- Multimodal LLM + 툴 증강형 금융 트레이딩 에이전트

주요 기여 (4가지) :
1. 멀티모달 데이터(가격, 차트, 뉴스, 전문가 분석)에서 핵심 인사이트 추출
2. 트레이딩 요약 + 검색 전용 쿼리 필드 제공 (다양한 retrieval type 지원)
3. Dual-level Reflection (저수준: 가격 움직임 분석, 고수준: 과거 결정 평가)
4. 전문가 지침 + 기술 지표 기반 전략을 통합한 Tool-Augmented Decision-making

성과 :
- 6개 금융 데이터셋 (주식 + Crypto)
- 9개 SOTA 대비 평균 +36% 수익 개선
- TSLA 데이터셋에서 **92.27% ARR (84.39% 개선)**
- 최초의 **멀티모달 Foundation Agent for Finance**


# 2 Related Work

![](<../Method/Images/Pasted image 20250901153432.png>)
### 1. AI 기반 금융 트레이딩

- **Rule-based**: 초기 시스템은 기술적 지표(RSI, MACD 등)를 규칙으로 사용하는 단순한 형태였으나, 시장 변화에 적응하기 어려움
- **Machine Learning**: RNN, CNN, GCN 등이 사용되어 주가 예측 및 이벤트 분석을 시도. 하지만 대부분 **단일 모달 데이터(가격 시계열)**에 치중
- **Reinforcement Learning**: 포트폴리오 관리, 트레이딩 정책 학습에 활용되었으나 데이터 요구량, 설명력 부족, 노이즈 민감성 문제 존재


### 2. 금융 특화 LLM

- **BloombergGPT (2023)**: 363B 토큰의 금융 데이터로 학습된 대규모 언어모델. 금융 QA, sentiment 분석, 뉴스 요약 등에 활용
- **FinGPT (2023)**: 오픈소스 금융 LLM, 투자 리서치와 금융 QA 태스크 지원
- **한계**: 대부분 텍스트 중심이며, 실제 트레이딩의 **순차적 의사결정**이나 **멀티모달 데이터 처리**는 다루지 못함


### 3. LLM 기반 에이전트

- **AutoGPT, BabyAGI, LangChain 에이전트**: LLM에 memory, planning, tool 사용을 접목하여 복잡한 태스크 해결 가능
- **FinMEM (2023)**: 금융 트레이딩 특화 LLM agent. 인간과 유사한 memory 구조를 가지지만, 멀티모달 데이터나 도메인 툴 활용은 제한적
- **Toolformer (2023)**: LLM이 스스로 외부 API/툴 사용법을 학습할 수 있음을 증명


### 4. 기존 연구의 한계

- 금융 도메인에서 **멀티모달 (가격+뉴스+차트)** 통합 처리 거의 없음
- **툴 증강 (전문가 전략, 기술 지표)** 을 트레이딩에 본격적으로 활용한 사례 부족
- 대부분 QA, 요약 같은 **언어 태스크**에 머물러 있고, 실제 **트레이딩 실행(의사결정)** 으로 연결되지 않음

⇒ 이 논문은 **멀티모달 LLM + 툴 증강 + 트레이딩 전용 reflection & memory 구조**라는 점에서 기존 연구와 차별성을 주장

# 3 Problem Formulation

## 3.1 Financial Trading as MDP

## 3.2 Problem Formulation


# 4 FinAgent Framework

![](<../Method/Images/Pasted image 20250901153449.png>)
분석 → 검색 → 반성 → 의사결정 → 실행 → 반성 : 순환 알고리즘

## 4.1 Market Intelligence Module
- 입력: **수치 데이터(가격, 지표), 텍스트 데이터(뉴스, 리포트), 시각 데이터(캔들 차트)**
- 역할: 서로 다른 모달 데이터를 통합 분서 → 시장 상황 요약 생성
	- ex) “TSLA, 신형 배터리 발표 → 긍정적 신호. 최근 7일 상승세 유지.”
- 출력 : 시장 상황 요약 + 핵심 인사이트
- 장점: 단일 모달에 의존하지 않고 다양한 신호를 융합 → **Ch1 해결 (멀티모달 처리 부족)**

## 4.2 Memory Module : Diversified Memory Retrieval
- LLM이 검색용 쿼리 생성
	- ex) “유사한 배터리 관련 뉴스 이후의 주가 반응 찾아줘”
- Retriever가 메모리 DB에서 과거 사례 반환
	- ex) “2023년 신차 발표 당시: 매수 후 10일간 +15% 상승”
- 출력 : 관련된 과거 사례와 성과

- 거래(task)와 검색(retrieval)을 **분리**해서 처리
- 검색 요청 시, 쿼리 유형별로 다른 retrieval 방식 적용 (예: 가격 패턴, 뉴스, 과거 거래 기록 등)
- 장점: 노이즈 감소 + 목표 지향적 검색 가능 → **Ch2, Ch3 해결**

### MI Memory (Market Intelligence Memory)
- Market Intelligence 모듈이 다루는 기억 저장소 
- 가격 시계열, 뉴스, 차트 같은 **멀티모달 시장 정보** 기록
→ 현재 상황 분석 시 과거 유사한 **시장 패턴**을 불러오기

### LLR Memory (Low-Level Reflection Memory)
- Low-Level Reflection 모듈에서 생성된 단기 분석/반성 기록
- **단기 가격 움직임, 기술적 지표 반응, 미시적 시장 패턴**
→ 단기 전략 적응력 강화 (ex: 모멘텀, 변동성 분석)

### HLR Memory (High-Level Reflection Memory)
- High-Level Reflection 모듈에서 생성된 고수준 교훈/피드백 저장소
- 과거 **트레이딩 의사결정 + 결과 + 교훈**
→ 장기적 학습, 자기 개선 (self-improvement)


## 4.3 Reflection Module
자기 행동의 결과까지 포함해서 학습하는 자기 피드백 메커니즘

- **Low-level Reflection**: 가격 움직임과 단기적 패턴 분석 → 단기 인사이트 제공
- **High-level Reflection**: 과거 트레이딩 의사결정을 돌아보고 개선점을 학습
- 출력 : 현재 의사결정에 대한 보조적 통찰
- 효과: 자기 피드백(self-reflection) 구조로 시장 적응력 강화 → **Ch2, Ch3 해결 (검색/적응성 문제)**

## 4.4 Tool-Augmented Decision-making Module
- 입력 : Market Intelligence + Retrieval + Reflection 결과
- 툴 호출 : 
	- 기술적 지표 툴 (MACD, RSI, Mean Reversion 등) 실행
	- 전문가 전략 (Trend Following, Momentum, Risk Management) 고려
- LLM이 reasoning 포함 최종 의사결정 도출
	- ex) “뉴스 긍정 + 기술적 상승 신호 + 과거 사례 수익 → 매수 진입”
- 출력: **행동(action) + reasoning(이유)**
	- ex)  “뉴스는 긍정적이고, 기술적 신호도 상승을 지지. 과거 유사 사례에서도 매수 후 수익 발생. 따라서 매수 진입 추천.”
	- → 최종 행동 : "Buy TSLA"

→ 최종 행동 → 거래 실행 → 거래 결과(Profit / Loss) 메모리에 기록 → Reflection 모듈이 결과를 평가하고 학습

- **전문가 지침(expert guidance)** 과 **기술적 지표 기반 전략(MACD, RSI, Mean Reversion 등)** 을 결합
- 최종 행동(매수/매도/유지)에 대해 reasoning을 함께 제공 → explainability 확보
- → **Ch4, Ch5 해결 (도메인 지식 통합 + 추론 제공)**

[FinAgent의 모듈별 워크플로우]
![](<../Method/Images/Pasted image 20250901181208.png>)
각 단계별로 **입력(prompt)과 출력(response)** 결과 보여줌
**MI → 과거검색 → LLR → HLR → 최종의사결정** 순차적 진행

# 5 Experiment Setup

- 데이터셋 : 6개 금융 시계열 데이터
	- 5개 주식: AAPL, AMZN, GOOGL, MSFT, TSLA
    - 1개 암호화폐: ETH-USD

- 기간: 약 5~10년치 가격/뉴스/차트 데이터 사용

- 비교 대상 (Baselines, 총 9개)
    - Rule-based: Buy & Hold, Moving Average (MA), RSI
    - Machine Learning: XGBoost, LSTM, GCN
    - Reinforcement Learning: DQN, PPO
    - LLM Agent: FinMEM

- 평가지표 (Metrics): 6개 금융 성과 지표
    - ARR (Annualized Rate of Return)
    - Sharpe Ratio
    - Sortino Ratio
    - Max Drawdown
    - Win Rate
    - Volatility

# 6 Experiment Results

- **전체 성능**: FinAgent가 **6개 지표 전반에서 평균 +36% 개선**
- **주식 vs 암호화폐**:
    - 주식 데이터셋에서 특히 강력한 성과
    - TSLA에서 **92.27% 수익률 (84.39% 개선)** → 최고 성능
    - ETH-USD(암호화폐)에서는 상대적으로 성능 저하 → 시장 잡음이 커서 explainability와 tool 기반 reasoning의 효과가 제한적

# 7 Ablation Studies

- **Reflection 제거** 시 적응력 급감 (시장 변화에 둔감해짐)
- **툴 증강 제거** 시 성능도 하락 (특히 변동성 관리 지표)
- **멀티모달 데이터 중 일부 제외** (차트나 뉴스 제거) → 성능 눈에 띄게 악화
- 즉, FinAgent는 **모듈들이 상호작용할 때 최적 성능** 발휘

# 8 Conclusion And Future work

- FinAgent는 금융 도메인 최초의 **멀티모달 + 툴 증강 + 메모리/리플렉션 기반 Foundation Agent**

- **장점**:
    - 다양한 모달 데이터 통합 가능 (텍스트+시각+수치)
    - Reflection을 통한 **자기개선(self-improvement)** 능력
    - 툴 증강으로 **도메인 지식(전문가 전략, 기술적 지표) 통합**
    - reasoning 기반 의사결정으로 **explainability** 확보

- **한계**:
    - 암호화폐처럼 **잡음이 큰 시장**에서는 안정적이지 않음
    - **실제 트레이딩 환경(latency, 거래비용, 슬리피지)**은 고려하지 않음
    - 프롬프트 체인 기반이라 inference 비용이 큼 (GPT-4/4V 호출 비용)

- **향후 연구 방향**:
    - 포트폴리오 관리, 자산 배분, 리스크 관리 태스크로 확장
    - 멀티모달 LLM을 금융 도메인에 특화된 방식으로 파인튜닝 가능성 제안
    - 효율적이고 저비용 inference 구조 설계 필요