---
title: Paper - LLMLingua Series
date: 2024-12-10 13:03:50 +09:00
categories: [Paper, LLMLingua]
tags: 
    [
        Paper,
        LLM,
        Large Language Model,
        LLMLingua,
        LongLLMLingua,
        LLMLingua-2,
        Prompt Compression
    ]
use_math: true
---

# Paper - LLMLingua Series (Prompt Compression)
최근 RAG, Chain-of-Thought, In-Context Learning과 같은 프롬프팅 기법이
LLM의 응답 성능을 높이기 위해 많이 사용되고 있습니다.
하지만 위와 같은 기법들은 프롬프트의 길이를 늘리는 경향이 있습니다.  
프롬프트의 길이가 길면 어떠한 문제가 발생하게 될까요?  
* 모델의 최대 입력 길이를 초과할 수 있음
* 중요한 정보가 Context 중간에서 잊혀지는 '*Lost In The Middle*' 같은 현상 발생
* 모델의 응답 속도 저하(Latency 증가)
  
위와 같은 문제점들을 해결하기 위해 제안된 것이 *Prompt Compression*.  
즉, **프롬프트 압축** 기법입니다.

---

## LLMLingua (2023)
> phi-2나 llama-7b와 같은 작은 언어모델을 이용하여 프롬프트를 압축하는 방법

![LLMLingua-2048x1146](https://github.com/user-attachments/assets/9ae11d07-ea53-47b4-a3f4-c9ffc7fbeed4)

### Budget Controller
> 프롬프트를 원하는 압축률만큼 압축하면서도 의미를 유지하기 위해 필요한 토큰 예산을 할당

* 작은 모델은 프롬프트의 각 예시의 perplexity를 추정하고, 더 많은 perplexity(더 유익한)를 가진 내용을 포함하도록 우선 순위를 매김  
* 남은 토큰 예산은 의미를 유지하기 위해 프롬프트의 서로 다른 부분 간, 동적으로 조절 및 할당됨
  
### Token-level Iterative Algorithm
> 세분화된 수준에서 프롬프트 압축을 개선함으로써 토큰 드롭아웃과 같이 더 간단한 압축 방법의 비효율성을 해결

* 프롬프트는 Segment로 나눠지며, 작은 모델은 각 Segment에서 토큰들의 perplexity를 계산
* 토큰들은 perplexity 분포에서 파생된 동적인 임계값을 기반으로 선택적으로 압축됨
* 알고리즘은 Segment들을 반복해서 동작하며, 압축된 프롬프트가 원본 프롬프트의 의미를 유지하는 것을 보장하도록 기여함

:computer: **예시 코드**  

```python
from llmlingua import PromptCompressor

llm_lingua = PromptCompressor()
compressed_prompt = llm_lingua.compress_prompt(prompt, instruction="", question="", target_token=200)

# Or use the phi-2 model,
llm_lingua = PromptCompressor("microsoft/phi-2")

# Or use the quantation model, like TheBloke/Llama-2-7b-Chat-GPTQ, only need <8G memory.
# Before that, you need to pip install optimum auto-gptq
llm_lingua = PromptCompressor("TheBloke/Llama-2-7b-Chat-GPTQ", model_config={"revision": "main"})
```

---

## LongLLMLingua(2024)
> LongLLMLingua는 LLMLingua의 기본 구조를 확장하며, LLM이 긴 문맥을 처리하는 데
겪는 문제를 해결하기 위해 다양한 개선점을 도입

![longllmlingua](https://github.com/user-attachments/assets/f037f76b-6afe-4642-8c3f-f2bbf56398c2)

### Improvements: Compare to LLMLingua
1. 압축 기술 향상
   - Contrastive Perplexity
     - LLMLingua에서 사용된 Perplexity 기반 접근법보다 더 효과적으로 중요한 토큰을 식별함
   - Question-Aware Fine-Grained Compression
     - LLMLingua의 Token-level Iterative Algorithm을 기반으로 개선됨
     - 압축 과정에서 질문의 중요도를 반영하여 가장 관련 높은 토큰이 유지되도록 보장

2. 문서 재배치
   - 문서를 중요도 점수에 따라 재배치하여 중요한 정보가 입력 초기에 배치되도록 설계
   - LLM이 정보의 위치에 민감하다는 특성을 활용(Lost In The Middle 논문에서 제안)

3. 동적 예산 할당
   - LLMLingua의 균등 예산 적용 방식과 달리, 문서의 중요도에 따라 압축 예산을 동적으로 할당
   - 선형 스케줄러를 사용하여 더 정교하게 제어 가능

4. 부분 문자열 복원
   - 압축 과정에서 발생할 수 있는 정보 손실을 줄이기 위해, 부분 문자열 복원(Subsequence Recovery) 메커니즘 도입
   - 날짜나 이름과 같은 핵심 정보를 정확히 복원하여 작업의 신뢰성을 높임

5. 성능 및 효율성
   - 특정 벤치마크에서 최대 17.1% 성능 향상 달성
   - 입력 토큰 수를 4배 감소시키는 데 성공
   - 긴 문맥(Long Context)에서 최대 3.8배의 지연 시간 단축을 보여줌

|    특징     |  LLMLingua | LongLLMLingua |  
|:---------:|:----------:|:-------------:|  
| 압축 측정 방식  | Perplexity | Contrastive Perplexity |  
| 예산 할당 방식  | Uniform budget | Dynamic budget |  
|   문서 처리   | Static compression | Condition on importance score |  
| 핵심 정보 무결성 | 기본적인 토큰 유지 | 부분 문자열 복원 도입 |
  
:computer: **예시 코드**  

```python
from llmlingua import PromptCompressor

llm_lingua = PromptCompressor()
compressed_prompt = llm_lingua.compress_prompt(
    prompt_list,
    question=question,
    rate=0.55,
    # Set the special parameter for LongLLMLingua
    condition_in_question="after_condition",
    reorder_context="sort",
    dynamic_context_compression_ratio=0.3, # or 0.4
    condition_compare=True,
    context_budget="+100",
    rank_method="longllmlingua",
)
```

---
  
## LLMLingua-2 (2024)
> LLMLingua-2는 이전의 두가지 기법에서 제시된 프롬프트 압축 기술을 크게 개선
> 데이터 증류와 Token-level Classification를 기반으로 하는 효율적인 프롬프트 압축 메커니즘 제공

![LLMLingua-2-2048x1157](https://github.com/user-attachments/assets/f526e259-b07b-4295-baa6-cf625e227e06)

### Improvements: Compare to LLMlingua & LongLLMLingua
1. 데이터 증류(Data Distillation) 기반 접근법
   - GPT-4로부터 데이터를 증류하여 학습
   - 주어진 텍스트를 최대한 간결하게 압축하면서도 정보를 충실히 유지하도록 설계
   - 기존 고정된 압축 비율 접근법과 달리, 문맥에 따라 적응적인 압축 비율을 채택하여 긴 텍스트를 더 효과적으로 처리 가능

2. Token-level Classification
   - BERT와 유사한 인코더를 사용하여 각 토큰을 유지할지 삭제할지 분류함
   - LongLLMLingua의 청크 단위(chunk-wise) 처리에서 더 나아가 문맥 정보를 세밀하게 반영하며 정보 손실 최소화
   - LLMLingua 시리즈 중 가장 높은 정확도와 효율성을 달성하게 해주는 핵심 요소

3. 성능 개선
   - 속도와 효율성 측면에서 기존 두가지 기법보다 3배에서 6배 더 빠르며, 사용되는 토큰 수를 줄여 운영 비용 절감
   - 복잡한 장문을 처리하는데 있어 LongLLMLingua보다 더 적은 정보 손실로 더 높은 성능을 보여줌

4. 도메인 확장성
   - 도메인 비특화(task-agnostic) 환경에서도 강력한 성능 발휘
   - 특히, 비정형 데이터(out-of-domain data) 처리에서도 높은 효과를 보임
   - 기존 LLMLingua 기법이 도메인 내 정보 밀도와 장르에 따라 성능이 달라지는 단점을 보완함

:computer: **예시 코드**  

```python
from llmlingua import PromptCompressor

llm_lingua = PromptCompressor(
    model_name="microsoft/llmlingua-2-xlm-roberta-large-meetingbank",
    use_llmlingua2=True, # Whether to use llmlingua-2
)
compressed_prompt = llm_lingua.compress_prompt(prompt, rate=0.33, force_tokens = ['\n', '?'])

## Or use LLMLingua-2-small model
llm_lingua = PromptCompressor(
    model_name="microsoft/llmlingua-2-bert-base-multilingual-cased-meetingbank",
    use_llmlingua2=True, # Whether to use llmlingua-2
)
```

---

:link: **참고 자료**  

**Github**
* [microsoft/LLMLingua](https://github.com/microsoft/LLMLingua)  

**Paper**
* [LLMLingua: Compressing Prompts for Accelerated Inference of Large Language Models](https://arxiv.org/abs/2310.05736)
* [LongLLMLingua: Accelerating and Enhancing LLMs in Long Context Scenarios via Prompt Compression](https://arxiv.org/abs/2310.06839)
* [LLMLingua-2: Data Distillation for Efficient and Faithful Task-Agnostic Prompt Compression](https://arxiv.org/abs/2403.12968)