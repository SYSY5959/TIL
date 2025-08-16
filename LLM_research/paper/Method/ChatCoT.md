# ChatCoT: Tool-Augmented Chain-of-Thought Reasoning on Chat-based Large Language Models
EMNLP 2023

- 기존 방법은 tool 호출 시 reasoning 흐름이 끊기거나 사전에 tool 사용 계획을 세워야 했는데, **ChatCoT는 reasoning과 tool 사용을 멀티 턴 대화로 통합**하여 그 문제 해결
- prompting 기법

1. Conversational Knowledge Memory : 대화 초기에 tool, task, reasoning 제공
    - Tool Knowledge: 각 도구의 기능 설명 “Calculator can help you calculate...”
    - Task Knowledge: 유사 문제와 솔루션을 retriever로 가져옴
    - Reasoning Format: step-by-step 대화 형식 예시를 보여줌
2. Iterative Tool-Augmented Reasoning
	- 각 단계에서 : reasoning → tool 선택 (필요시) → tool 실행 → 결과 피드백 → 다음 reasoning → ...  반복

![](<./Images/Pasted image 20250721013326.png>)
stat
- Task : 
	-  MATH: 고등 수학 문제 
	- HotpotQA: 멀티홉 질의응답
- Tools :
	- Calculator: 수식 계산
	- Equation Solver: 연립방정식 풀이
	- Retriever: 유사 문제 검색
- Results : 
	- MATH에서 기존 최고 성능(PHP) 대비 +7.9% 향상
	- HotpotQA에서도 CoT 기반 베이스라인보다 크게 우수

- 한계 : 
	- 어쨌든 prompting 기법
	- 멀티턴으로 개발자가 계속 입력을 해줘야함