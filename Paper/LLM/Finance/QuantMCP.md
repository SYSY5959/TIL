# QuantMCP: Grounding Large Language Models in Verifiable Financial Reality
arXiv 2025

https://arxiv.org/abs/2506.06622

# Abstract
## QuantMCP
LLM이 실시간, 검증 가능한 외부 금융데이터(API)를 호출해 근거 있는 구조화된 데이터를 받아서, 그 위에 정확한 분석을 수행하도록 만드는 프레임워크

### Why QuantMCP ?
1. LLM의 금융 적용 한계 
	- Hallucination : 금융에서는 작은 오류도 치명적
	- 실시간 데이터 접근 불가 : 금융 데이터는 초 단위로 변하는데, LLM은 학습 시점 이후 데이터는 알 수 없음 → 최신성(realtime)과 정확성(verifiability) 확보 X

2. 금융 데이터 환경의 복잡성
	- 데이터 제공자가 다양함 (Wind, Bloomberg, yfinance, Tushare 등)
	- 각 API마다 호출 방식·데이터 구조가 다름
	- 사용자나 LLM이 직접 다루기 어려움 → **표준화 필요** **: QuantMCP**

- **해결책**: QuantMCP (MCP 활용해 LLM ↔ 금융 API 연결)
⇒  LLM은 자연어로 질문만 해도 → MCP 서버가 API 호출 → 실제 데이터(JSON) 반환

## MCP (Model Context Protocol)
[MCP (Model Context Protocol)](<../../../LLM_research/Agent/MCP (Model Context Protocol).md>)

LLM이 외부 리소스(API, 데이터베이스, 툴, 플러그인 등)에 **표준화된 방식으로 접근**할 수 있도록 설계된 **protocol**
= LLM이 다양한 API 호출할 수 있게 해주는 공통 언어와 규칙 세트
= API 함수들을 표준화된 "**Tool 객체**"로 정의

- **Tool discovery**: 
	- MCP 서버는 자신이 제공할 수 있는 기능 **(API 호출)을 manifest(메타데이터 설명서) 형식**으로 LLM에게 알림
	- **이름(name), 설명(description), 입력 파라미터(parameters), 출력(output)**
	- ex) “`get_stock_price` 툴 → 종목코드, 기간, 지표 입력 필요”
- **표준화된 호출**: 
	- LLM은 MCP가 제공하는 일관된 인터페이스를 통해 API를 호출
	- API마다 구조가 달라도, MCP가 이를 통일된 포멧(JSON,,)으로 변환
- **Context management**: 
	- LLM이 대화 기록·사용자 상태를 툴에 전달 가능 → 더 정교한 동작
	- ex) 이전에 조회한 종목을 자동으로 이어받아 분석 가능
- **보안성**: 
	- API key, 인증정보는 MCP 서버가 관리
	- LLM이나 사용자에게 직접 노출되지 않음 → 금융 데이터 환경에서 중요


- QuantMCP는 **MCP를 금융 도메인에 적용** → LLM이 금융 분석을 신뢰성 있게 수행할 수 있도록 확장
- 다양한 금융 데이터 API(Wind, yfinance, Tushare, Alpha Vantage 등)를 **MCP 기반으로 묶음**
- 보안 관리(예: API Key 서버에서만 저장) 강화
- 실시간 데이터 grounding을 통해 LLM 환각 방지
- 금융 분석용 end-to-end workflow (데이터 조회 → 분석 → 시각화) 제시


# Framework

![](<../Method/Images/Pasted image 20250905145713.png>)
사용자 질의 → LLM 해석 → MCP 서버가 금융 API 호출 → JSON 데이터 반환 → LLM 분석/보고

1. 사용자 → **자연어 질의** 입력
2. **LLM 처리 (NLI Layer)** → 질의 의도 파악, 종목코드·기간 등 entity 추출
3. **MCP Server** 
	- LLM이 MCP Server에 "이런 tool을 실행해야겠다"고 요청
	- MCP Server는 **Tool manifest**(설명서)를 통해 어떤 파라미터가 필요한지 확인
	- LLM이 추출한 값을 Tool에 전달
4. **금융 데이터 API** : MCP Server가 실제 API(Wind, yfinance, Tushare 등)를 호출
5. 결과(JSON 등 structured format) → LLM에게 반환
6. **LLM** → 데이터 분석·계산·시각화 수행 → 사용자에게 결과 전달

## Natural Language Interface (NLI Layer)
- LLM (GPT-4, Gemini 2.5, DeepSeek-V3)이 사용자 요청 해석
- LLM이 자체 지식으로 답하지 않고, 필요한 정보를 tool 호출로 계획

Task :
- **Intent recognition** (무엇을 알고 싶은가?)
- **Entity extraction** (종목코드, 기간, 지표 등)
- **Query disambiguation** (애매한 요청을 명확히 변환)

## MCP Server for Financial Tools
- 다양한 API를 tool로 감싼 후, manifest로 LLM에 노출
- 툴 설명, 파라미터, 출력 형식(JSON)까지 표준화
→ LLM과 금융 데이터 API 간의 **중재자 + 보안 gateway** 
(API Key는 서버에서만 관리 → LLM은 접근 불가)

## Data Retrieval and Structuring
- MCP Server가 API 호출 → 결과를 JSON 등 구조화된 포맷으로 변환
- LLM은 이 구조화된 “grounded data” 위에서만 분석 → hallucination 최소화
(LLM 자체 지식이 아닌, 외부 API에서 가져온 진짜 데이터 기반으로 분석하기 때문)


# Implementation

- LLM: DeepSeek-V3-0324
- MCP Server: Cherry Studio
- API: WindPy (전문 금융 데이터)
- 보안: API Key 서버에서만 관리

예시 :
- CATL(300750.SZ) 2024 Q1 평균 주가·turnover 계산
- 2025년 5월 데이터 시각화 (closing price, P/B ratio 등)

[QuantMCP의 Tool 입력 스키마]
![](<../Method/Images/Pasted image 20250905161438.png>)

[DeepSeek 대화창에서 tool 호출됨]
![](<../Method/Images/Pasted image 20250905161601.png>)

[Tool의 JSON 포멧]
![](<../Method/Images/Pasted image 20250905161854.png>)

[LLM이 생성한 시각화 그래프]
![](<../Method/Images/Pasted image 20250905162501.png>)


# Discussion
- **성과**: 환각 해결, MCP 통한 표준화, 자연어 인터페이스 제공
- **활용**: KPI 계산, anomaly 탐지, comparative analysis, 자동 리포트 생성
- **영향**: 애널리스트 효율화, 새로운 통찰, 금융 리터러시 확산
- **의의**: 금융 LLM → “대화형 도구”에서 “데이터 기반 분석 어시스턴트”로 진화

# Conclusion 
- **QuantMCP**는 금융 도메인에서 LLM이 가진 핵심 문제(환각, 최신성 부족)를 해결하기 위한 **프레임워크** 제안
- **MCP 기반 구조**로, 다양한 금융 데이터 API를 안전하게 연결하고 LLM이 자연어로 질의·분석 가능
- 이를 통해 LLM은 단순 Q&A 도구가 아니라 **데이터 기반 금융 분석 어시스턴트**로 확장됨
- 금융 업계에서 **신뢰성 있는 AI-driven 분석 시스템** 구축의 초석이 될 수 있음.

# Limitations
- **데이터 의존성**:
    - 외부 API 품질·가용성에 의존.
    - API가 없거나 불완전하면 한계 발생.

- **보안 & 확장성**:
    - MCP 서버가 API Key 관리 등 보안의 단일 지점(Single Point of Failure).
    - 대규모 사용자·다양한 API 환경에서 확장성 문제 발생 가능.

- **LLM 자체 한계**:
    - 데이터 grounding 후에도 고차원 reasoning, 도메인 특화 추론에서는 여전히 제약 존재.
    - 잘못된 분석이나 해석 오류 가능성은 완전히 배제할 수 없음.

