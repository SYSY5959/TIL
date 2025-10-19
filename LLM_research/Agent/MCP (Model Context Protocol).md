: Agent가 다른 프로그램(Tool)과 연동하기 위한 protocol

https://modelcontextprotocol.io/docs/getting-started/intro

![AI디자이너를 위한 MCP 이해하기](https://img1.daumcdn.net/thumb/R1280x0.fwebp/?fname=http://t1.daumcdn.net/brunch/service/user/3XFk/image/KWzF1l2E6sLRIM_-atRcpxq3JzY)


https://techblog.woowahan.com/22342/

- Model : AI 모델에게
- Context : 프로그램의 사용법을 알려주는
- Protocol : 규약

like USB-C : 주변 장치와 악세사리에 장치를 연결하는 표준화된 방법

## MCP Server
- LLM이 활용할 수 있도록 **실제 기능, 데이터를 제공함**
- 웹 서비스나 특정 기능을 외부 AI가 사용할 수 있게 MCP protocol에 맞추어 서버 구현
- Notion, Figma


## MCP Client
- LLM이 실제로 실행되는 측. AI가 다양한 도구나 서비스를 사용할 수 있도록 **요청을 보내는 쪽**
- OpenAI ChatGPT와 Google Gemini는 웹과 데스크톱 UI에서는 MCP 연결을 지원 X , SDK에서는 지원
- [OpenAI Platform](https://platform.openai.com/docs/guides/tools-remote-mcp) / [Google AI For Developer](https://ai.google.dev/gemini-api/docs/function-calling?hl=ko&amp;example=meeting#model_context_protocol_mcp)

![](<./Images/Pasted image 20250910133058.png>)

https://bcho.tistory.com/1470
## Tool
- LLM이 호출할 수 있는 외부 도구
- 주로 API로 통신할 수 있음

## Agent
- LLM(brain 역할)을 이용해서, 
- LLM으로부터 질문에 대한 답변 가지고 오거나, 
- 필요한 Tool 호출해서 그 결과를 LLM에 전달

- Agent는 어떤 Tool을 사용할지 결정해야하고, history도 기억해야 되고, 등등
- Agent 구현에 필요한 요소가 많아서, 처음부터 만들기 어렵고 Agent Framework를 사용
-  Langchain, CrewAI, LIamaIndex 등 많은 오픈소스/상용 Framework 존재

## Tool SDK
- 이런 Agent Framework는 Tool과 연동할 수 있는 각자 Tool SDK 제공
- **Agent Framework마다 Tool 별로 SDP 만듬**
	- Framework 제작사에서 모든 Tool 회사와 컨택해서, 스펙 정의/개발 등등 다 해야함.
	- Tool (서비스 제공) 업체는 Framework 회사들과 개별 컨택해서 연동해야함.
	- → 너무 번거롭고 불편함!!
	- ⇒ 그래서 나온 게 **MCP**


![](<./Images/Pasted image 20250910132851.png>)

![](<./Images/Pasted image 20250910142944.png>)

# MCP
- Tool별 SDK 만들지마!! protocol 스펙만 맞춰!!!

![](<./Images/Pasted image 20250910143324.png>)
- MCP는 코드나 SDK가 아니고, 정확하게는 protocol
- JSON RPC 사용. SSE (Server Sent Events) : JSON RPC + HTTP

[List Tool]
![](<./Images/Pasted image 20250910143744.png>)
- LLM application 내의 agent가 Weather service에게 list tools 요청 
- → Weather service가 get_weather라는 tool을 제공, city를 인자로 받아서, 해당 city의 날씨 정보를 리턴할 수 있다고 agent에게 알려줌


[Call Tool]
![](<./Images/Pasted image 20250910144058.png>)
- 그러면 이제 어떤 걸 호출할 수 있는지, 방법까지 알게 됨!
- 이제 "New York" 날씨 정보 얻기 위해, Tool 호출

## MCP SDK

- 개발자 입장에서 이 protocol을 어떻게 구현해야할까?

![](<./Images/Pasted image 20250910181635.png>)

- MCP를 사용할 수 있는 SDK 만듬 → MCP sever SDK / MCP Client SDK
- MCP Client SDK : Agent Framework가 MCP Server를 쉽게 호출할 수 있는 SDK 제공 (보통 Agent Framework에 포함되어 있음)
- MCP Server : MCP 요청을 받는 서버를 만드는 SDK
- Server와 Client의 SDK 제공

## MCP가 왜 중요한가??
1. LLM과 도구의 표준화된 연결 : 
2. 사전 구축된 다양한 context 도구 연결 : 이미 구축되어 있는 MCP 서버들의 기능을 활용하여 LLM이 외부 시스템, context에 빠르고 쉽게 연동할 수 있음
3. 유연하고 확장 가능한 시스템 : 호스트-클라이언트-서버로 구성된 3계층 아키텍쳐. 각 요소에 대한 유지보수가 따로 가능함

- 서로 협의 없이 스펙만 맞추면 Agent와 Tool 연동 가능
- 많은 tool이 제공되고, 통합도 쉬워짐


# Agent 확장을 위한 MCP 서버

사전 구축된 MCP 서버 Agent 연결하여 사용 가능

http://anthropic.com/news/model-context-protocol

https://docs.anthropic.com/en/docs/mcp

## Agent에 툴 연동해서 성능 개선
MCP 서버를 모아두고 있는 마켓 플레이스 → 사용할 서버 가지고 오기
https://smithery.ai/

https://www.youtube.com/watch?v=IOfn0MGzYd4


