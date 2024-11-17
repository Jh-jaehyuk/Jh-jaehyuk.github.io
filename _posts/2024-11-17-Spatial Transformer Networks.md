---
title: Paper - Spatial Transformer Networks
date: 2024-11-17 17:30:30 +09:00
categories: [Paper, Spatial Transformer Networks]
tags: 
    [
        Paper,
        Transformer,
        Computer Vision,
        DeepMind,
    ]
use_math: true
---

# Spatial Transformer Networks(STN)

## What is STN?
아래 사진은 무작위로 위치 이동, 스케일링, 회전, 왜곡된 MNIST 데이터셋을 입력 값으로 하여 STN을 적용한 것입니다.   
![example image1](https://github.com/user-attachments/assets/3cec2b78-7562-4e98-87d8-fee4783638f1) 
**작거나 찌그러진 특징**을 검출해내어 **정상 데이터 형태로 변형**하는 방법  

### 그렇다면 위와 같은 변형 방법이 왜 필요할까요?  
저자들은 **CNN**의 Maxpooling이 입력 데이터에 대해 *Spatially invariant* 하지 못한 부분을 약간은 해소해주지만,
2x2 연산정도로는 데이터의 *Spatially variability* 에 대응하기 어렵다고 언급합니다.  
**spatially variability:* translation(위치 이동), scale(확대/축소), rotation(회전) 등의 공간적 변화

### STN의 개념
다시 간단하게 말하자면, **STN이란 기존 인공 신경망 구조에 추가되어 아래와 같이 공간적 변형을 동적으로 제공**해주는 것입니다.  
![example image2](https://github.com/user-attachments/assets/1956fe01-5d53-4e7a-b8c2-2f3bb769c5e9)  
STN은 기존 이미지에 현재 가장 연관 있는 부분만 추출해내거나, 
뒤에 오는 인공 신경망 계층의 추론을 돕기 위해 기존 이미지를 일반적인 형태로 변환하는 용도로 사용할 수 있습니다.  
위 사진은 MNIST 데이터셋에 spatial transform을 적용한 것을 입력 데이터로 하여 STN에 입력하는 과정을 보여줍니다.  
STN은 (a)와 같은 사진에서 (b)와 같이 영역을 특징해내어 (c)와 같은 출력을 만들어냅니다.  
(d)는 (c)에서 만들어낸 출력을 기반으로 Fully-connected layer가 예측한 결과입니다.  

Spatial Transformer의 동작은 입력 데이터의 형태에 따라 매번 달라지며, 
별도의 supervise 없이 **모델의 학습 과정에서 Backpropagation을 통해 같이 학습됩니다.**

---
## STN Architecture
![architecture1](https://github.com/user-attachments/assets/8f23b174-4649-43e8-b4e4-0f0a2cf71f0b)  
STN은 위와 같이 크게 세 가지 구조로 이루어져 있습니다.

### Localisation net
**Localisation net**에서는 입력 Feature map _U_ 에 적용할 변형 파라미터인 $\theta$를 추정합니다.  
여기서 입력 Feature map _U_ 는 가로 _W_, 세로 _H_, 채널 _C_ 의 크기를 갖습니다.  
Localisation net은 Fully-connected network 또는 Convolution network 둘 다 가능하지만,
마지막에 변형 파라미터 $\theta$를 추정할 수 있는 regression layer를 반드시 포함해야 한다고 합니다.  
*$\theta$를 affine transformation으로 표현하고자 한다면, 
affine transformation은 6차원 벡터로 표현할 수 있으므로 $\theta$는 6차원의 벡터가 될 것

### Grid generator
**Grid generator**는 위에서 추정된 파라미터 $\theta$에 따라, 
입력 Feature map _U_ 에서 sampling 할 위치를 정하는 $\tau_{\theta}(G)$ 계산합니다.  
출력 _V_ 의 sampling grid _G_ 는 계산된 $\tau_{\theta}$를 거쳐 $\tau_{\theta}(G)$로 mapping됩니다.  
이 과정을 사진으로 보면 아래와 같습니다.  
![grid_generator1](https://github.com/user-attachments/assets/a10a2835-d4a5-40b0-9c4c-ac5932e94fad)  
만약 $\tau_{\theta}$가 affine transformation이라면 아래와 같이 나타낼 수 있습니다.
![grid_generator2](https://github.com/user-attachments/assets/4654c94f-36fe-460d-92f1-16652eb09293)  
$\tau_{\theta}$는 affine transformation 이외에도 각 파라미터에 대해 미분가능하다면 어떤 것이든 사용 가능합니다.

### Sampler
**Sampler**는 입력 Feature map _U_ 에 $\tau_{\theta}(G)$를 적용하여
변환된 출력 Feature map _V_ 를 만들어냅니다.  
출력 _V_ 에서 특정 픽셀 값을 어디서 가져와야할지에 대한 정보는 sampling grid $\tau_{\theta}(G)$가 가지고 있습니다.  
하지만 그 위치가 정확하게 정수 값이 아닌 경우도 존재하기 때문에, 그런 경우 **주변 값들에 대한 보간법**을 적용하여
해당 픽셀 값을 구하게 됩니다.  
그 경우는 아래와 같은 식을 계산하는 과정을 거치게 됩니다.
![sampler1](https://github.com/user-attachments/assets/1f1018d1-6826-436b-97c9-e1eb675bf457)  
다소 복잡한 위의 식을 일반화된 sample kernel _k_ 가 아닌 **bilinear interpolation**을 적용하면, 
아래와 같이 조금 단순해진 식을 얻을 수 있게 됩니다.  
![sampler2](https://github.com/user-attachments/assets/a9d71166-41f0-4e21-826a-fa32600568ba)  

전체 네트워크에서 loss 값을 backpropagation을 통해 계산하기 위해선 위의 식이 _U_ 와 _G_ 에 대해 미분가능함을 확인해야 합니다.  
결과 확인을 위해, 위의 식을 _U_ 와 _G_ 에 대해 각각 미분해보면 아래와 같은 결과를 얻을 수 있게 됩니다.
![sampler3](https://github.com/user-attachments/assets/7a7001e3-999d-4201-acaf-30a06719255f)  
![sampler4](https://github.com/user-attachments/assets/21f30953-09c2-4eb6-bc69-237412901752)   
아래와 같이 sampling function이 _모든 구간에 대해 미분가능하지않더라도_, 
**구간별로 나누어 subgradient를 구하면** backpropagation를 계산할 수 있습니다.  

---
## Spatial Transformer Networks
결론적으로, 위와 같은 Spatial Transformer 구조를 기존 CNN에 끼워넣은 것을 **Spatial Transformer Networks**라고 합니다.  
Spatial transformer 구조는 이론상 CNN의 어느 위치에나, 몇개든 끼워넣을 수 있습니다.  
**일반적으로 CNN의 바로 앞에 배치**하지만, network 내부의 깊은 layer에 배치해 좀 더 추상화된 정보에 적용을 한다거나 
여러 개를 병렬로 배치해서 한 image에 속한 여러 부분을 각각 tracking하는 용도로 사용할 수도 있다고 합니다.

