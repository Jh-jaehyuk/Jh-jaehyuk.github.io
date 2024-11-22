---
title: Paper - Flash Attention
date: 2024-11-18 11:33:15 +09:00
categories: [Paper, Flash Attention]
tags: 
    [
        Paper,
        Transformer,
        Computer Vision,
        CV,
        LLM,
        Large Language Model,
        Flash Attention,
    ]
use_math: true
---

# Flash Attention: Fast and Memory-Efficient Exact Attention with IO-Awareness

## Background
Transformer 모델은 자연어 처리와 컴퓨터 비전 분야에서 혁신을 이끌었지만, self-attention 매커니즘에서
시퀀스 길이가 길어질수록 메모리와 연산 비용이 기하급수적으로 증가하는 문제가 있습니다.  
**기존 self-attention 알고리즘은 $O(N^2)$의 시간 복잡도와 메모리 사용량을 요구**하기 때문에, 
긴 시퀀스의 데이터를 처리할 때 bottleneck이 발생합니다.  
이는 GPU 메모리를 과도하게 소모하여 훈련 속도를 저하시킬 뿐 아니라, 
더 큰 모델이나 데이터셋을 사용하는 데 제한을 주게 됩니다.  

이 문제를 해결하기 위해 저자들은 메모리 접근 효율성을 개선하고 훈련 속도를 극대화할 수 있는 새로운 접근법을 제안합니다.  
**Flash Attention** 알고리즘의 주 목표는 **Attention 계산의 정확도는 유지**하면서 **메모리 접근을 최적화**하여
Transformer 모델이 더 효율적으로 동작하도록 하는 것입니다.

GPU는 CPU와 마찬가지로 아래와 같은 메모리 계층을 갖습니다.  
![gpu_hierarchy1](https://github.com/user-attachments/assets/b09b5d6b-2d04-4f66-bb8f-63825b3b536b)  
DRAM은 용량이 가장 크지만 처리 속도가 가장 느리고, SRAM은 용량은 작지만 처리속도가 가장 빠른 것을 확인할 수 있습니다.  
GPU는 병렬 처리 시 데이터를 HBM에서 가져온 후, SRAM에 올려놓고 계산하게 됩니다.  
이후 다른 데이터를 읽어들이면, 기존에 SRAM에 있는 데이터를 HBM으로 내려놓고 
새로운 데이터를 SRAM에 올려서 계산을 진행하게 됩니다.  

---
## Flash Attention
저자들은 Flash attention에 **Tiling**과 **Recomputation**이라는 두 가지 기법을 적용하여
기존 Attention 연산을 가속화하고자 하였습니다.

### 일반적인 Attention의 메모리 사용 문제
**Scaled Dot-Product Attention**의 주요 연산 중 하나는 **Query(Q)** 와 **Key(K)** 의 곱셈으로 
생성되는 Attention Score 행렬입니다.  
* Attention Score 행렬의 크기:  
$[\text{Batch Size, Num Heads, Seq Len, Seq Len}]$  
예를 들어, 입력 시퀀스 길이가 1024라고 하면, $\text{SeqLen}^2 = 1024^2 = 1,048,576$ 크기의 행렬이 생성됩니다.  
이렇게 **시퀀스 길이에 비례하여 메모리 사용량이 제곱으로 증가**하므로, 긴 시퀀스에서는 메모리 부족이 발생할 가능성이 높습니다.  

### Flash Attention의 블록 단위 처리 원리
Flash Attention은 시퀀스를 작은 블록 단위로 나누어 연산을 수행합니다. 예를 들어, 시퀀스의 길이가 1024이고
블록의 크기가 64라면, 전체 Attention 연산은 16개의 작은 블록으로 나뉘게 됩니다.  
* 블록 단위로 Query와 Key를 연산하면, **각 블록에서 생성되는 Attention Score 행렬의 크기는 
$[\text{Batch Size, Num Heads, Block Size, Block Size}]$** 로 줄어듭니다.  
* 이전의 1024$\times$1024 크기의 행렬 대신, 64$\times$64 크기의 행렬만 메모리에 로드하여 계산할 수 있습니다.  
결과적으로, **메모리 사용량은 블록 크기의 제곱에 비례**하며, 전체 연산 과정에서 메모리 사용량이 크게 감소합니다.


### Tiling
기존 self-attention 알고리즘은 쿼리(Q), 키(K), 값(V) 행렬이 큰 크기로 메모리에 적재되며,
이로 인해 메모리 사용량이 $O(N^2)$으로 증가합니다.  
특히 긴 시퀀스 데이터를 다룰 때, **대량의 메모리 접근이 발생하면서 GPU에서 I/O 병목 현상**이 발생합니다.  

**Tiling** 기법은 이를 해결하기 위해 **입력 데이터를 작은 블록(Tile) 단위로 나누어 처리**합니다.  
즉, 전체 시퀀스를 한번에 연산하지 않고, 작은 블록을 순차적으로 GPU의 shared memory에 로드하여 
필요한 연산만 수행한 후 결과를 저장합니다.  
이렇게 하면 메모리 접근이 감소하고 GPU의 SRAM에서 대부분의 연산이 이루어져 효율성이 향상됩니다.  

Flash attention에서 Tiling 기법은 다음과 같이 적용됩니다. 

* **블록(Tile) 단위로 데이터 분할** 
  * 시퀀스 길이 _N_ 을 여러 블록으로 나눕니다.
  * 예를 들어, 길이 1024인 시퀀스를 128블록으로 나눈다면, 한 번에 128개씩 처리하게 됩니다.
  

* **Shared memory 활용** 
  * 각 블록의 쿼리(Q), 키(K), 값(V) 행렬을 GPU의 shared memory로 가져옵니다.
  * 이 메모리는 GPU 내에서 빠르게 접근 가능하기 때문에, HBM에서 직접 접근하는 것보다 훨씬 빠릅니다.


* **블록(Tile) 내에서 연산 수행**
  * 블록 단위로 attention 값을 계산한 후, 필요한 경우 HBM에 다시 씁니다.
  * 이 때, 최소한의 데이터만 메모리에 접근하도록 설계되어 메모리 접근 비용이 크게 절감됩니다.

**그런데 블록 단위로 attention을 계산해도 문제가 발생하지 않을까요?**  
먼저 대답을 하자면, '**그렇다**' 입니다.  
그 이유는 아래와 같습니다.  

기존 softmax 연산은 아래와 같은 과정을 거치게 됩니다.
![softmax1](https://github.com/user-attachments/assets/702ea4a8-5c91-45f9-ba9d-66e78c2bab24)  
vector $x^{(1)}, x^{(2)} \in \mathbb{R}^B$ 일 때, 
vector concatenation $x=[x^{(1)}x^{(2)}]$에 대해 softmax는 다음과 같이 Decompose할 수 있습니다.  
![softmax2](https://github.com/user-attachments/assets/d784851c-5443-4a85-926e-dffac85b380e)  
결과적으로, softmax를 블록 단위로 계산할 수 있음을 알 수 있습니다.  

### Recomputation
저자는 출력 _O_ 와 softmax normalization statistics인 $m(x)$와 $l(x)$를 저장함으로써,
SRAM안에서 Q, K, V의 블록 backward pass시 attention matrix _S_, _P_ 를 
쉽게 recompute 할 수 있도록 하였습니다.  
**이로 인해 FLOPs는 증가하게 되겠지만, HBM에 접근하는 횟수가 줄어들어 전체적인 속도는 향상된다고 합니다.**

![recomputation1](https://github.com/user-attachments/assets/c59f70aa-feea-4cc6-9fd6-8c1e6b21859d)
위의 두 가지 기법을 활용하여 한 번의 HBM Load로 
Matrix multiplication, Masking, Softmax, Dropout, Matrix multiplication 과정을 거친 후,
HBM에 값을 저장할 수 있게 되면서 위 사진처럼 기존보다 빠르게 attention 계산을 할 수 있게 되었습니다.
