[Gorilla](<./Gorilla.md>)
https://github.com/ShishirPatil/gorilla/

## OpenFunctions v2 추가된 기능
### 1. Parallel Functions (병렬 함수 호출)
- **v1 한계**: 한 번에 하나의 함수만 호출 → 복잡한 태스크에서 순차 호출 필요 → 비효율적
- **v2 개선**: 여러 개의 함수를 **동시에 호출** 가능
- 예시 :
    - 사용자 요청: “내 위치 날씨 알려주고, 오늘 일정도 보여줘.”
    - v1: `get_weather` → 응답 → `get_calendar`
    - v2: `get_weather` + `get_calendar` **동시 실행** → 응답 시간 단축, 워크플로우 최적화

### 2. Function Selection & Relevance Detection
- **문제**: LLM이 모든 함수 정의를 다 보면서 불필요하게 긴 프롬프트 → 오버헤드 & hallucination
- **해결**: v2는 **필요한 함수만 필터링해서 선택** → irrelevant function 호출 줄임
- Retrieval 기반 필터링 + relevance score 도입

### 3. Multi-turn & Multi-function Orchestration
- v1은 단순히 “사용자 요청 → 함수 호출” 구조
- v2는 **대화 맥락**을 추적하면서, 여러 함수 호출을 적절히 이어 붙이는 orchestration 지원
- 예시:
    - “내일 서울 날씨 알려줘” → `get_weather(서울, 내일)`
    - 사용자가 “그럼 비 오면 우산 주문해줘” → **이전 함수 호출 결과**를 조건으로 `order_item(우산)` 실행.

### 4. 더 많은 함수 포맷 지원
- v1은 OpenAI function calling과 유사한 포맷 중심
- v2는 JSON Schema 기반 정의, 다양한 parameter 구조, nested argument 지원 → 복잡한 API 호출 가능

### 5. 효율성 / 확장성 강화
- 함수 수가 많아질수록 v1은 “scaling issue” 발생
- v2는 relevance filtering + 병렬 호출 덕분에 **수백~수천 개 함수도 실용적으로 관리 가능**

| 기능       | v1 (OpenFunctions) | v2 (OpenFunctions v2)           |
| -------- | ------------------ | ------------------------------- |
| 함수 호출 방식 | 단일 함수 호출           | **병렬 함수 호출**                    |
| 함수 선택    | 모든 함수 후보 고려        | **Relevance detection 기반 선택**   |
| 맥락 관리    | 단발성 호출             | **Multi-turn orchestration 지원** |
| 함수 정의    | 단순 JSON            | **JSON Schema + nested 구조 지원**  |
| 확장성      | 함수 수 많아지면 비효율      | **대규모 함수 집합 효율적 처리**            |
