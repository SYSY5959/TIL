# FINQA: A Dataset of Numerical Reasoning over Financial Data
EMNLP 2021

https://github.com/czyssrs/FinQA

![](<./Images/Pasted image 20250815011604.png>)
금융 전문가가 주어진 보고서(표+본문) 바탕으로 질문(Q), 정답(A), 계산 과정(Calculations), 실행 가능한 프로그램(Program) 생성

# Abstract
- 문제: 방대한 기업 재무제표에서 깊이 있는 질문에 답하려면 복잡한 수치 추론과 다양한 형식(표·텍스트) 이해가 필요하지만, 기존 QA 연구는 일반 도메인 위주로 단순 계산에 그침
- 목표: 금융 전문가 작성 Q&A 데이터셋 기반, 복잡한 금융 질문에 자동으로 답하는 시스템 개발
- 주요 기여:  
  1. 최초의 대규모 금융 QA+수치 추론 데이터셋 **FINQA** 구축
  2. 정답 reasoning 프로그램 annotation 제공 → 해석성 보장  
  3. 다양한 LLM 기반 베이스라인(FinQANet) 실험 수행  
- 결과: 최신 LLM도 전문가 수준에 크게 못 미침 → 향후 연구 필요
- 공개: 데이터와 코드 모두 오픈소스 제공

## 데이터셋 개요
- **도메인:** 금융 (S&P 500 기업 실적 보고서, 1999~2019)
- **규모:** 8,281 QA pairs
- **작성자:** 금융 전문가(회계사·MBA 등) (질문당 $2, 시간당 $35, 총 8주 걸림) 
- **구성:**
    - 질문(Question)
    - 정답(Answer)
    - **정답 도출 프로그램(Reasoning Program, DSL)**
    - supporting facts (표·텍스트 출처)
- **특징:** 표(Table) + 텍스트(Text) **이질 데이터 통합** 필요

## Task 정의
- 입력:
	- 금융 보고서 텍스트 E + 표 T
	- 질문 Q

- 출력: 
	- **실행 가능한 reasoning program** $G={w0​,w1​,...wn​}$
	- 프로그램 실행해서 → 정답 A 도출

- 확률 모델 :
$$P(A∣T,E,Q)=∑P(Gi​∣T,E,Q)$$

- DSL 연산: 
	- 수학 연산 (6개) : `add, subtract, multiply, divide, exp, greater`
	- 테이블 연산 (4개) : `table-sum, table-average, table-max, table-min`
	- 프로그램 구조 : `op1[arg1], op2[arg2], ... , opn[argn]`


- 평가 지표: 
    - **Execution Accuracy**: 프로그램 실행 결과가 정답과 일치하는 비율
    - **Program Accuracy**: 생성된 프로그램이 수학적으로 정답 프로그램과 동등한 비율
    - 금융 도메인에서는 설명 가능성이 매우 중요!!  → 두 지표 모두 중요함

## 데이터 통계
- 데이터 분할: Train 6,251 / Dev 883 / Test 1,147
- 필요 정보 출처:
    - 표만: 62.43%
    - 텍스트만: 23.42%
    - 표+텍스트: 14.15%
- Reasoning step 수:
    - 1 step: 59.10%
    - 2 step: 32.71%
    - 3+ step: 8.19%
- 연산 비중: `divide` (45.29%) > `subtract` (28.20%) > `add` (14.98%) > `multiply` (5.82%)

## 베이스라인

![](<./Images/Pasted image 20250817152436.png>)

- **FinQANet** (retriever + program generator)
    - **Retriever**: BERT 기반, 보고서에서 관련 문장·표 행 추출 (상위 k개)
    - **Program Generator**: LSTM decoder 기반, 실행 가능한 DSL 생성


#### Program Generator
![](<./Images/Pasted image 20250817165929.png>)
 3가지 토큰 소스
1. 입력 facts에서 복사 (숫자/행 이름)
2. DSL 토큰 (연산자, 상수)
3. Step memory : 이전 단계 결과 참조 (#0, #1 …) 

- **Grammar mask** 적용
    - 디코딩 중 문법적으로 불가능한 토큰은 차단 (예: `divide(` 다음엔 반드시 2개의 인수가 와야 함)

- **Step memory 업데이트**
    - 매 단계가 끝날 때(`)`) decoder는 그 단계 결과를 embedding으로 저장
    - 이후 단계에서 `#0`, `#1` 같은 메모리 토큰으로 활용 가능

**자연어 질문 → (Retriever facts) → DSL program** → 실행 결과 : 정답

## 실험
- 비교 대상:
    - TF-IDF + 단일 연산
    - Retriever + Seq2Seq
    - Retriever + NeRd
    - Longformer (end-to-end)

- **최고 성능:** RoBERTa-large 기반 FinQANet → Execution Acc 61.24%

- **인간 성능:**
    - 외부 전문가: Execution Accuracy 91.16% , Program Accuracy 87.49%
    - 비전문가: Execution Accuracy 50.68% , Program Accuracy 48.17%

## 성능 분석 포인트
- 표 전용 문제 > 텍스트 전용 문제 > 표+텍스트 혼합 문제 (난이도 ↑)
- 3단계 이상 reasoning, 단위 변환(상수 사용) 문제에서 큰 성능 저하    
- Retriever 성능이 전체 정확도에 큰 영향 (Top-3 Recall 89.66%)

![](<./Images/Pasted image 20250818013339.png>)