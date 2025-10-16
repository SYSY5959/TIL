# MultiTool-CoT: GPT-3 Can Use Multiple External Tools with Chain of Thought Prompting
ACL 2023 

배경 : LLM 정확한 수치 계산 어려움 & 특정 도메인 지식 부족 → tool 사용
기존에는 하나의 tool 사용 → 이 논문에서는 **여러 tool들을 결합해서 사용**

MultiTool-CoT : 
1. Few-shot CoT prompting : 몇 개의 예제를 GPT-3에게 제공하면서 중간 추론 단계와 함께 도구 호출 방식(tool trigger)을 학습
2. 도구 호출 포맷: `<<도구 이름>>` 형식으로 호출
3. 도구 호출 시 추론 일시 중지, 외부 도구 실행 후 결과를 반영하고 이어서 추론 재개
4. 최종 결과는 마지막 문장에서 추출.
- 예: `2 × 62 = <<Calculator>> ` → 계산기 호출 → 결과 삽입 → 다시 reasoning 반복

![](<./Images/Pasted image 20250820133952.png>)

- Task: NumGLUE Task 2 (화학 지식 기반 수치 추론 문제)
- Tools:
    1. Calculator (계산기)
    2. Chemical reaction predictor (화학 반응식 예측기)
    3. Molar mass list (몰 질량 테이블)
- Results: MultiTool-CoT는 기존 방법들 대비 큰 성능 향상
    - 기존 CoT few-shot 성능: 57.85%
    - MultiTool-CoT (모든 도구 사용): 85.85% (SOTA)

**few-shot 예제 : 5 < 10 < 20 개**
20개 일 때 가장 성능이 높았고, 그 이상은 토큰 길이 제한으로 어려웠음


한계:
1. 단일 벤치마크 : 화학 중심의 수치 추론 task → 복잡한 지식, 다양한 현실적인 문제에서 적용 X
2. few-shot 예제에서 : CoT reasoning 안에 tool trigger를 수작업으로 넣어야함