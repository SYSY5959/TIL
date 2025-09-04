# Beyond Chain-of-Thought: A Survey of Chain-of-X Paradigms for LLMs

https://arxiv.org/pdf/2404.15676


![](<./Images/Pasted image 20250902174928.png>)


배경 :
[CoT in LLM](<../Method/CoT in LLM.md>)
- **Chain-of-Thought (CoT, Wei et al., NeurIPS 2022)**: 복잡한 문제를 중간 reasoning step으로 나누어 LLM의 성능과 해석 가능성을 크게 향상
- CoT의 성공을 바탕으로 다양한 **Chain-of-X (CoX)** 방법론이 등장. 여기서 X는 단순한 "thought"가 아니라, **intermediates, augmentation, feedback, models** 등 여러 노드로 확장 가능


## Chain-of-X 노드 분류

1. **Chain-of-Intermediates**
    - Problem decomposition (예: CoT, Least-to-Most Prompting)
    - Knowledge composition (예: Chain-of-Knowledge, CueCoT)

2. **Chain-of-Augmentation**
    - Instructions (Chain-of-Task, Chain-of-Instructions)
    - Histories (Chain-of-History, Chain-of-Opinion)
    - Retrievals ([ReAct](<../Method/ReAct.md>), Chain-of-Query, Chain-of-Knowledgea) 
    - **Tools** ( [ChatCoT](<../Method/ChatCoT.md>), [MultiTool-CoT](<../Method/MultiTool-CoT.md>), [CoAbstract](<../Method/CoAbstract.md>)) 

3. **Chain-of-Feedback**
    - Self-feedback (Self-Refine, Chain-of-Verification)
    - External feedback (Chain-of-Hindsight, Chain-of-3DThought)

4. **Chain-of-Models**
    - 여러 LLM 또는 전문가 모델을 연결 (예: Chain-of-Experts, Chain-of-Discussion, ChatEval)

![](<./Images/Pasted image 20250902190122.png>)


## Chain-of-X 적용 과제

- **Multimodal interaction** (Text-Image: Chain-of-Look, Text-Table: Chain-of-Table, Text-Code: Chain-of-Repair 등)
- **Factuality & Safety** (Hallucination 감소: Chain-of-Verification, Alignment: Chain-of-Hindsight)
- **Multi-step reasoning** (법률 추론, 그래프 추론, summarization 등)
- **Instruction following** (Chain-of-Task, LogiCoT, Chain-of-Imagination)
- **LLMs as Agents** (Chain-of-Action, Chain-of-Contacts, Chain-of-Abstraction)
- **Evaluation tools** (BadChain, Chain-of-Utterances, Chain-of-Feedback)