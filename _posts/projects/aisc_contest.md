---
layout: default
title: AI Semiconductor Design Contest
parent: Projects
nav_order: 0
nav_exclude: true
---

## AI Semiconductor Design Contest

_2022.10.26_  
 <br>
First project that got me into Deep Learning compression and hardware implementation.  
**Keywords: #Pruning #Quantization #tinyyolov3 #SAM**

---

### Our Proposal

> Channel-level pruning using SAM(Sam and Max)

**0. Abstract**  
본 아이디어는 기존의 Channel-Level Pruning 기법을 변형하여, Sum-and-Max (SAM) 기법으로 기존 Batch Normalization의 스케일링 요소에 L1 규제를 가한다. 이러한 sparsity training 단계 이후에 스케일링 요소 중 0에 가까운 값을 가진 channel 전체에 대한 pruning을 진행한다. 최종 목표는 CNN을 최적화하여 모델 크기를 줄이고 실행 시 메모리 사용량을 줄이면서 정확도를 유지하는 것이다. 더 나아가, 연산량을 최소화하여 inference 속도를 높여 모델의 효율성을 향상시키고 리소스 사용을 최적화하는 방법을 제안하고자 한다. 이 방법은 현대적인 CNN 아키텍처에 직접 적용할 수 있으며, Channel-level Pruning을 이용해 추가적인 overhead나 소프트웨어/하드웨어 가속기 없이 모델 경량화 효과를 기대할 수 있다.

**1. Introduction**  
고효율 학습 및 추론을 위한 IoT 기반 ISP에서 사용될 딥러닝 모델은 Convolutional Neural Networks (CNN) 모델이다. CNN은 다양한 컴퓨터 비전 작업에 효과적으로 사용되고 있지만, **1) 모델의 크기, 2) 실행 시 메모리 사용량, 3) 컨볼루션 연산량**으로 인해 사용에 많은 제약을 받는다. ImageNet을 학습시킨 일반적인 CNN 모델의 크기는 보통 300MB인데, 이는 일반적인 임베디드 시스템 위에 올리는데 한계가 있다. 제한된 하드웨어 리소스만 있는 환경에서 CNN 모델을 사용하기 위한 다양한 연구가 선행되었고, 가장 대표적인 예시로 Pruning과 Quantization이 있다. 본 아이디어는 그중 Pruning, 특히 Channel-Level Pruning 기법을 모델에 적용하여 IoT 기반 ISP에 사용될 수 있는 크기로 모델 사이즈를 줄이는 방법을 소개하고자 한다.

**2. Channel-level Pruning**  
Pruning은 sparsity level에 따라 크게 weight, kernel, channel, layer level로 나눌 수 있다. 일반적으로 weight level pruning을 많이 진행하는데, CNN 모델 같은 경우 레이어의 개수가 많은 모델의 경우, 채널 전체를 pruning해도 정확도 손실이 크지 않기 때문에 채널 단위로 pruning하는 연구들이 선행되었다. 그중 본 아이디어가 사용할 기법은 Channel-level pruning의 일종인 ‘Network Slimming’이다.

Network Slimming은 Batch Normalization(BN) 레이어의 스케일링 요소에 L1 규제를 적용하는 방식을 제안한다. 각 스케일링 요소는 특정한 컨볼루션 채널에 곱해지며, 이를 활용하여 중요한 채널을 식별하고, 중요하지 않은 채널을 pruning 대상으로 지정된다. L1 규제는 스케일링 요소 값들을 sparse하게 만드는 특징이 있기 때문에 중요하지 않은 채널들을 쉽게 식별 및 pruning 가능하게 만든다. 다음은 Network Slimming 기법의 특징을 요약한 것이다.

- **Channel-level sparsity의 이점**: Pruning은 sparsity level에 따라 크게 weight, kernel, channel, layer level로 나눌 수 있다. 가중치 단위의 sparsity는 generalization 측면에서 유리하지만, 빠른 추론 속도를 위해서 소프트웨어 및 하드웨어 가속기가 필수적이다. 반면, 레이어 단위의 sparsity는 별도의 가속기나 소프트웨어 패키지가 필요하지는 않지만, 모델의 깊이가 50 레이어 이상일 때만 정확도 손실이 없다. 따라서, 그 중간 단계인 채널 단위의 sparsity를 소개함으로써 별도의 가속기 없이, 유연하게 모델의 크기를 조절한다.
- **Channel-level sparsity의 어려움:** 채널 단위의 pruning을 하기 위해서는 pruning을 할 채널의 모든 입력과 출력값을 같이 제거해야 한다. 하지만, pre-trained model(학습하지 않은 모델)에서 채널 전후로 모두 0값을 가진 채널은 많지 않고, 있다고 해도 학습 이전이기 때문에 중요하지 않은 채널이라는 보장이 없다. **따라서, pruning 이전의 ‘sparsity training’ 즉, pruning 할 대상 채널을 찾는 과정이 필수적이다.**
- **BN 레이어의 스케일링 요소 활용** : 모든 CNN 모델은 컨볼루션 연산 이후 Batch Normalization 과정을 통해 각 레이어에서 입력 분포가 변하는 문제를 완하한다. 이때, 정규화 이후 scale-and-shift를 이용해 모델의 유연성을 높이는데, 학습 가능한 스케일링 요소에 L1 규제를 가해 학습시키면 각 채널이 결과에 얼만큼 기여하는지, 그 정도를 추정할 수 있다. 즉, BN 레이어의 스케일링 요소를 직접 활용해 중요한 채널을 선택할 수 있는 것이다.
- **Pruning 채널 선정:** Sparsity training 이후에 0에 가까운 스케일링 요소를 가진 채널을 pruning 해준다. 예를 들어, pruning 파라미터를 70%로 잡는다면, 70% 퍼센타일에 해당하는 스케일링 값을 threshold로 선정하여 그 이하의 스케일링 값에 해당하는 모든 채널을 pruning한다. 즉, BN 레이어의 스케일링 요소를 직접 활용하여 중요한 채널을 선택하게 되는 것이다.

**3. Why use SAM(Sum and Max)?**  
Network Slimming 기법은 본 학습 이전에 ‘sparsity training’이라는 추가적인 과정을 통해 pruning 할 대상 채널을 찾아야만 pruning을 수행할 수 있다. 하지만 CNN 모델의 크기가 커질수록, sparsity training을 함께 했을 때, 총 학습 시간은 기존 학습 시간에 1.5배-2배 정도의 시간으로 늘어난다. Pruning을 하기 위한 사전 과정이 너무 길어지는 것을 방지하고자, 연산 속도가 더 빠른 SAM(Sum and Max) 기법을 제안한다.

SAM 기법은 일반적인 CNN 모델에서 사용되는 MAC (Multiply and Accumulate) 연산과 비슷한 효과를 내지만, 연산 속도와 연산량을 크게 줄일 수 있다. 일반적으로 가중치를 곱한 입력값은 결과값을 추론하는데 얼만큼 영향을 주는지에 비례한다. 이러한 관점에서 봤을 때, 가중치는 굳이 곱해질 필요 없이 더해지는 것만으로 비슷한 효과를 낼 수 있다. 다음, 가중치와 곱해진 입력값을 모두 더하고, 뒤에 활성화 함수를 통과시켜 일정한 역치를 넘겨야지만 값이 유효한 것 또한 max function으로 대체될 수 있다. 가중치를 더한 입력값들 중 가장 큰 값을 선택하는 것으로 결과값에 가장 큰 영향을 주는 입력값을 찾을 수 있다. 따라서, 뉴런 한 개에 대한 hypothesis는 다음과 같이 재정의될 수 있다.

$$
o_j = \max_j(x_i+W_{i,j}^+)+ \min_j(x_i+W_{i,j}^-)
$$

max뿐만 아니라 min값을 더해주는 이유는 음의 방향을 가진 데이터를 완전히 무시하는 방향으로 학습하는 것을 방지하기 위해서이다.

이러한 SAM 기법을 Network Slimming의 sparsity training 단계에 도입하면 pruning하기 위한 사전단계에 소요되는 시간을 크게 줄일 수 있을 것으로 예상된다. 또한, 곱하기 연산을 아예 사용하지 않고 더하기와 max/min 연산만 사용하기 때문에 연산량을 크게 줄일 수 있다.

**4. Experiment**

- oxford hand dataset을 이용해 yolov3-tiny를 sparse training, 일반 학습, pruning 해보고 channel layer pruning의 효과를 검증해보았다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/8d9c865b-00cd-4baf-b964-f3479804c52d/b17c2d23-26c3-4837-b882-52e18f78d009/Untitled.png)

→ normal weights로 detect 했을 때 총 inference time = 0.462s, pruned weights (sparse training)으로 detect 했을 때 총 inference time = 0.447s 로 더 빠르다는 것을 확인할 수 있다.

- normal training의 총 학습 시간 = 4.531 hours (273 epoch), sparse training의 총 학습 시간 = 약 2 hours (100 epoch, 사진 없음 ), sparse training의 연산 속도가 매우 느리기 때문에 SAM 기법 도입을 제안한다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/8d9c865b-00cd-4baf-b964-f3479804c52d/099fd27f-a63f-4993-a3e7-543129047708/Untitled.png)

- oxford hand dataset을 이용해 yolov3-tiny의 레이어들을 quantization 레이어(4-bit)로 바꾸어 학습시키고 quantization의 효과를 검증해보았다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/8d9c865b-00cd-4baf-b964-f3479804c52d/e8b39af8-44b9-481b-b7bd-43c0e50900d1/Untitled.png)

→ yolov3 config file에서 일반 [conv] 레이어를 [quantize_convolutional] 레이어로 바꾼 모습

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/8d9c865b-00cd-4baf-b964-f3479804c52d/ccb3d695-b57d-4af9-b91d-b9422a1b0798/Untitled.png)

→ normal training mAP=0.64, quantization training mAP = 0.63로 정확도 손실이 거의 없다는 것을 확인할 수 있다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/8d9c865b-00cd-4baf-b964-f3479804c52d/ff1ef6d1-e66d-4354-bc93-5daa787bf2ca/Untitled.png)

→ 4 bit quantization weights로 test.jpg에 detection task를 수행할 때, 의도한 대로 손이 인식되는 것을 확인할 수 있다.

**5. Conclusion**  
본 아이디어는 CNN 모델의 크기를 줄이는 방법으로 Channel-level pruning을 소개하고, pruning을 준비하는 사전 단계인 sparsity training의 연산 속도와 연산량을 줄이기 위해 SAM 기법을 적용하는 것을 제안한다. 이 방법은 최신 CNN 아키텍처에 쉽게 적용 가능하며 별도의 소프트웨어/하드웨어 리소스 없이 효율적인 CNN 모델을 얻을 수 있습니다. 더 나아가, 고효율 학습 및 추론을 위한 IoT 기반 ISP에서 사용될 딥러닝 모델에 이러한 기법을 적용한다면, 모델의 크기를 줄여 inference time을 크게 개선할 수 있을 것으로 기대된다.

### Paper Review

- [Attention Is All you Need](https://www.notion.so/Attention-Is-All-you-Need-5860311e7cc84e94aaed795be16913d0?pvs=21)
- [An Image is Worth 16x16 Words](https://www.notion.so/An-Image-is-Worth-16x16-Words-1fbdbc3f05ab4259b1300fb88dc431c2?pvs=21)
- [Deep Compression: Pruning, Quantization, Huffman Coding](https://www.notion.so/Deep-Compression-Pruning-Quantization-Huffman-Coding-adf02ffc41704fe297e06f53fc80b61f?pvs=21)
- [CNN Network Slimming ](https://www.notion.so/CNN-Network-Slimming-3c72dd1d7aec40ab93582467cf3b2749?pvs=21)
- [Sum and Max for Low Power Classification](https://www.notion.so/Sum-and-Max-for-Low-Power-Classification-483342df38b643a3a7b20c37000e9691?pvs=21)
