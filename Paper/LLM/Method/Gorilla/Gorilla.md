# Gorilla: Large Language Model Connected with Massive APIs
NeurlPS 2023, UC Berkeley & Microsoft Research

https://gorilla.cs.berkeley.edu

# Abstract
- LLM은 수학적 추론, 프로그램 생성 등 다양한 작업에서 우수하지만 **API 호출 능력**은 미흡
- GPT-4조차 **정확한 입력 인자 생성 실패**, **잘못된 API 사용(hallucination)** 문제
- **Gorilla**: LLaMA 기반 파인튜닝 모델, API 호출 작성에서 GPT-4 성능 능가
- 문서 검색기(document retriever) 결합 → **테스트 시 문서 변경에도 적응** 가능
- Hallucination 문제를 크게 완화
- 평가용 데이터셋 **APIBench** 구축 (HuggingFace, TorchHub, TensorHub API 포함)
- Gorilla + Retrieval → 도구 사용 정확도 향상, 문서 업데이트 대응, 출력 신뢰성↑
- 코드/모델/데이터/데모 공개: [https://gorilla.cs.berkeley.edu](https://gorilla.cs.berkeley.edu)

# 기존 연구 문제점
1. 최근 연구들은 LLM + Tool Usage (검색 기능, 계산기, 번역기, Python 실행기) 
	- ex) Toolformer, WebGPT
	- Toolformer는 LLM이 스스로 툴을 학습해서 사용
	- → 대부분의 연구가 한정된 소수의 툴에 집중

2. Hallucination 문제
	- GPT-4, Claude 같은 SOTA LLM도 존재하지 않는 API 만들어내거나, 잘못된 인자 생성

3. 업데이트 불가
	- 기존 LLM은 파라미터 안에만 지식 저장 → API 문서가 변경되면, 다시 학습해야 함
	- 현실에서 API 문서 자주 바뀜. 

![](<./Images/Gorilla_Figure1.png>)

# Gorilla 제안

**Gorilla : LLM이 수많은 API(Tool)를 정확히 선택하고 호출하도록 학습하는 모델**
사용자가 원하는 작업을 자연어로 설명하면, 그에 맞는 API를 직접 선택하고, 정확한 호출 코드까지 생성함

- LLaMA-7B
- 대규모 instruction–API 호출 쌍 데이터로 supervised fine-tuning
- **문서 검색(retriever)을 통해 최신 API 문서를 참조할 수 있도록 학습**

같은 데이터셋을 두 포맷으로 준비해서 모델 두 개를 따로 파인튜닝. (사용자가 상황에 맞게 선택 가능)
- **Zero-shot 모델**
	- 입력: 사용자 자연어 요청만
	- 출력: Gorilla가 학습된 패턴에 따라 바로 API 호출 생성
	- → 빠르고 단순.
- **Retriever-aware 모델**
	- 입력: 사용자 요청 + **retriever(BM25, GPT-Index 등)가 가져온 최신 API 문서 JSON**
	- 출력: Gorilla가 문서를 참고하여 최신 API 호출 생성
	- → 문서 변경(버전 업데이트, 레지스트리 이동 등)에 적응 가능

![](<./Images/Gorilla_Figure2.png>)
## Gorilla Training Pipeline
### 0. 데이터 수집 (APIBench 구축)
3가지 공개 모델 허브에서 API 수집
- TorchHub → 94개 API (전체)
- TensorFlow Hub v2 → 626개 API (전체)
- HuggingFace Hub → 925개 API (도메인별 Top 20)
→ **총 1645개 API 정리**
→ 각 API를 JSON으로 표준화
`{domain, framework, functionality, api_name, api_call, arguments, env_reqs, example_code, performance, description}`

### 1. Instruction 데이터 생성
- Self-Instruct 방식 : 
	- GPT-4에게 API 문서 + 3개의 in-context 예시 주고,
	- "이 API를 활용할 수 있는 현실적인 시나리오(Instruction) 만들어줘" 시킴
	- instruction에는 API 이름이나 힌트 넣지 말라고 제한
→ API 하나당 **10개의 {instruction, API} pair** 생성 → 총 **16,450 pairs**

### 2. 학습 데이터 포멧팅
- 각 pair를 **대화 형식 (user ↔ agent)** 으로 변환:
→ 실제 사용자 프롬프트에 반응하는 능력 강화
```pgsql
User: I want to remove the background from an image.
Assistant: torch.hub.load('pytorch/vision', 'fcn_resnet50', pretrained=True)
```

- **Retriever-aware 버전**은 프롬프트에 문서를 추가:
```pgsql
User: I want to remove the background from an image.
Use this API documentation for reference: <retrieved_API_doc_JSON>
Assistant: torch.hub.load('pytorch/vision', 'fcn_resnet50', pretrained=True)
```

### 3. 모델 학습 (Fine-tuning)
- 베이스 모델: LLaMA-7B
- **Instruction fine-tuning** 수행:
	- Loss: next-token prediction (일반적인 LM loss)
	- Optimizer: AdamW
	- Hyperparameters: learning rate 2e-5, batch size 64, epochs 5, warmup ratio 0.03, weight decay 0, max sequence length 2048
- 환경 : 8 × A100 (40GB) GPU

#### 2가지 버전
- **Without Retriever**: APIBench pairs만 사용 → zero-shot에서 강력
- **With Retriever**: 학습할 때 문서 JSON까지 같이 넣음 → test-time에서 문서 업데이트에 적응 가능

### 4. 검증 / 평가
- **AST Sub-tree Matching**으로 API 호출 정확성 측정:
    - 올바른 API 콜 여부
    - 잘못된 인자 사용 (error)
    - 존재하지 않는 API 호출 (hallucination)

- **비교 모델**: GPT-3.5, GPT-4, Claude, LLaMA-7B baseline
→  Gorilla는 zero-shot에서도 GPT-4보다 API 정확도가 20% 이상 높았음

#### AST Sub-tree Matching
- **AST(Abstract Syntax Tree)**: 파이썬 같은 **프로그래밍 언어 코드를 트리 구조로 표현**한 것
- **Sub-tree Matching**:
    - Gorilla가 생성한 API 호출을 AST로 변환
    - APIBench에 있는 정답 API 호출들을 AST로 변환
    - 두 트리를 비교해서, **생성된 호출이 정답 호출의 서브트리(sub-tree)** 인지 확인

→ 모델이 출력한 API 호출이 정답 데이터셋의 API 호출과 기능적으로 같은지 확인 가능
→ 생성된 API 호출이 데이터셋 어떤 AST와도 매칭되지 않으면 → 존재하지 않는 API (hallucination) 검출 가능

![](<./Images/Gorilla_Figure3.png>)

## Inference 단계
1. 사용자 입력 (자연어 프롬프트)
2. (옵션) Retriever 단계 (2가지 모드로 작동) → 사용자가 결정
	- **Zero-shot 모드**: 그냥 사용자 프롬프트만 보고 API 호출 생성
	- **With Retrieval 모드**: BM25나 GPT-Index 같은 검색기가 가장 관련 있는 **최신 API 문서**를 가져와 프롬프트 뒤에 붙여줌 → Gorilla는 문서 변화에 실시간 적응 가능
```arduino
User: I want to remove the background from an image.
Use this API documentation for reference: <retrieved_API_doc_JSON>
  ```

3. Gorilla 모델 (Retriever-aware fine-tuned LLaMA-7B)
	- 입력받은 프롬프트(사용자 요청 + 문서 JSON) 해석하고
	- 훈련 때 배운 instruction ↔ API 호출 매핑을 활용해, **올바른 API 호출 코드**를 한 줄로 출력
4. API call
	- 예시 상황별 출력


![](<./Images/Gorilla_Figure4.png>)
- 사용자의 요청이 같아도, **retriever가 가져온 최신 문서에 따라 Gorilla 출력 달라짐**
- 다른 시점 / 다른 문서 상태:
    1. 문서에 `fcn_resnet50`이 있을 때 → ResNet50 API 호출 출력
    2. 문서가 업데이트되어 `fcn_resnet101`로 교체되었을 때 → ResNet101 API 호출 출력
    3. 문서가 NVIDIA 레지스트리로 옮겨졌을 때 → NVIDIA/DeepLearningExamples 레지스트리 API 호출 출력

---

# 결론

## 실험 결과
비교 대상: GPT-3.5, GPT-4, Claude, LLaMA

- **Zero-shot Gorilla** → GPT-4보다 최대 20% 이상 높은 정확도
- **Hallucination(존재하지 않는 API 호출)** 비율이 GPT-4보다 현저히 낮음
- HuggingFace, TorchHub, TensorHub **세 데이터셋 모두**에서 강력한 성능

## Gorilla 차별점
- 대규모 API 호출 학습
- **Retriever-aware Fine-tuning** : 학습 단계부터 “문서 JSON + instruction” 같이 학습
- **AST Sub-tree Matching 평가** : API 호출 정확성 자체를 정밀하게 평가
- Hallucination 최소화
- 단순 API 호출이 아니라, 조건(reasoning)을 반영해 올바른 API 선택

## 한계점

- 도메인 편향 (ML API에만 집중)
- Retriever 품질 의존성 : retriever 성능이 bottleneck
- API 실행 결과 보장 X : 생성된 호출 코드가 실제 환경에서 돌아가는지 확인 X
- 범용 대화 능력보다는 API 호출에 특화 



---


사용자 요청 (ex. 영상에서 자동차를 감지하고 싶어) 이 들어오면, 
Gorilla는,,,
1. 어떤 작업인지 파악
2. 어떤 API가 적합한지 선택
3. API 문서 읽기
4. 코드로 정확하게 호출해줌 (ex. `torch.hub.load('datvuthanh/hybridnets', 'hybridnets', pretrained=True)`)

![](<./Images/Gorilla_Figure2.png>)

→ Gorilla는 GPT-4, Claude, GPT-3.5, LLaMA 대비
	더 높은 정확도 & 더 낮은 hallucination 비율
