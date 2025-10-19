# ReAct: Synergizing Reasoning and Acting in Language Models
ICLR 2023


**ReAct = Reasoning + Acting** prompting 기법

prompt 예시 : 
**Reasoning** (CoT - 내부 지식 활용) → **Action** (tool 호출) → **Observation** (tool 실행 결과) → 다시 reasoning → ... → Final Answer / Task Complete 

![](<./Images/Pasted image 20250721013611.png>)

- Task 1 : HotpotQA (지식 기반 질의응답)
	- CoT-only : hallucination
	- Act-only : 'Apple Remote'만 검색하다가 질문 형식에 맞지 않는 대답함(실패)
	- ReAct : 몇 번의 search(act)와 obs로 올바른 정답 도출
- Task 2 : LFWorld (가상 환경에서의 상호작용)
	- 상황 판단 + 행동 선택

prompt 예시 몇 개만 제공해줘도, 위와 같은 방식으로 reasoning과 acting을 번갈아 수행
(이 논문에서는 1~6개 prompt 예시 제공)

한계 :
- in-context prompting : tool 사용 방식을 prompt로 다 보여줘야함 → 긴 trajectory(답 찾기까지의 과정) 예시 있으면 context 길이 초과 → 복잡한 행동, 추론을 다 학습시키는 것 어려움 
- 예시 프롬프트 : thought + action(tool) + observation 사람이 직접 만들어야함

→ reasoning의 성능을 높이기 위해, **외부 tool을 잘 활용하도록 유도하는 prompting 기법**
**나의 연구 주제(tool 사용 능력 자체를 학습)와 결이 다름**
- ReAct : prompt에 제시된 tool을 바탕으로 정답을 맞추겠지만, 
	나의 연구에서는 모델이 스스로 판단해서 tool 선택
