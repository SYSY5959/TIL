# Evaluating LLMs' Mathematical Reasoning in Financial Document Question Answering
ACL 2024


4가지 금융 QA 데이터셋에서 LLM이 수학적 추론을 얼마나 잘 수행하는지 평가
추론 실패 원인을 유형별로 분석함

[금융 QA 데이터셋]
- FinQA
- TATQA
- ConvFinQA
- Multihiertt

금융 도메인은 손익계산서, 대차대조표와 같은 테이블 기반 문서가 많음
→ 여러 수치를 결합해서 계산해야 하는 다단계 추론이 필요
**⇒ 결론 : 금융 반구조화 문서(table + text)에서의 수학적 추론 능력은 아직 불완전하다!!**

![](<./Images/Pasted image 20250805171822.png>)


모델의 성능 저하 원인을 파악하기 위해!! 
**Metadata Annotation** 5가지
1. 추론 단계 수 : 산술 연산 개수에 따라 난이도 분류
	- **연산 단계 많아질수록 정확도 ↓**
	- **3단계 이상되면 급격히 하락**
2. 질문 범주 : 12개 수학 개념(합, 차, 비율, 평균, 개수,,,)으로 분류
	- 덧셈, 뺄셈 → 대부분의 모델이 잘함
	- **비율, 변화율, 도메인 수식(투자수익률, 매출원가, 감가상각률,,) → 가장 못함**
3. 테이블 길이 : 행 수 많을수록 추론 난이도 ↑
4. 계층적 복잡도 : 셀 정보가 몇 단계 계층 안에 있는지 측정
5. 결측 정보 : 빈 셀 비율 높을수록 해석 오류 가능성 ↑


**EEDP (Elicit → Extract → Decompose → Predict)** : prompting 기법
1. Elicit : LLM이 문제 풀기 위해 도메인 지식을 먼저 스스로 생각하도록 유도
2. Extract : 필요한 숫자, 데이터를 테이블이나 문단에서 추출
3. Decompose : 복잡한 문제를 여러 개의 단순 연산으로 분해
4. Predict : 이런 추론 과정 바탕으로 최종 정답 계산  

#### LLM 수학적 추론 실패 원인
##### 1. Incorrect Extraction
: 필요한 수치 못찾거나 잘못 찾음
- E1 : Missing Evidence 
- E2 : Wrong Evidence 

평가 방법 : LLM 출력에서 언급된 evidence 수작업으로 점검. 사전 annotation된 evidence랑 비교

##### 2. Incorrect Reasoning
: 추출한 정보는 맞지만 어떻게 계산할지 잘못 판단
- R1 : 도메인 지식 부족
- R2 : 질문 잘못 이해

평가 방법 : LLM이 evidence 기반으로 도츌한 수학적 추론 과정과 사전에 제공된 정답 비교

##### 3. Incorrect Calculation 
 : 올바른 정보와 수식을 썼지만, 계산 자체가 틀림
 - C1 : 값 대입 실수
 - C2 : 정밀도 오류 (소수점 문제와 같이 계산 결과 살짝 틀림)

평가 방법 : LLM 출력 중 계산 단계별 결과값을 눈으로 추적하며 비교

[데이터셋 별 에러 유형 비율]
![](<./Images/Pasted image 20250805212254.png>)
- Multihiertt : Extraction 오류가 가장 높은 데이터 셋
	- 계층적 구조 복잡도 높음
	- 테이블 내 정보가 분산되어있음
	- 결측치 문제

- ConvFinQA : Reasoning 오류 비율 가장 높은 데이터셋
	- 대화형 QA → 모델이 이전 턴 질문이랑 혼동하거나, 질문 논리 흐름을 놓침

- TATQA : Calculation 오류 비율이 가장 높은 데이터셋

- FinQA : 도메인 지식 부족 (R1) 오류 비율 가장 높은 데이터셋
	- 도메인 특화 수식(RPI, 투자이익률,,,) 자주 등장
	- 수식 몰라서 틀리는 경우가 많았음
	*→ 금융 도메인에 대한 지식을 Retriever로 가져오면 좋지 않을까..?*

논문에서,,,
> 4. Where do LLMs fail?
>*Calculation errors can be taken care of if a third-party calculation tool (an external agent) is chained to the language model.*

⇒ 금융 문서 QA에서 Mathematical Reasoning을 잘하기 위해서는 외부 tool이 필요함. 


