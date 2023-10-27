---
layout: minimal
title: CNN Network Slimming
nav_exclude: true
parent: Paper Review
---

## Learning Efficient Convolutional Networks through Network Slimming

_2022.10.25_  
 <br>
I actually tried this out, and got similar results!
**Keywords: #Pruning #Quantization #Batch Normalization**

---

**0. Abstract**

- Goal

  1. Reduce the model size
  2. Decrease the run-time memory footprint
  3. Lower the number of computing operation without compromising accuracy

- Distinction
  - Directly applies to modern CNN architectures
  - Minimum overhead to the training process
  - Requires no special software/hardware accelerators for the resulting models

**1.Introduction**

- CNN constraints

  1. Model size: trainable parameters, along with network structure information need to be stored on disk and loaded into memory during inference time (ex) typical CNN trained on ImageNet ⇒ 300MB

  2. Run-time memory: During inference time, the intermediate activations/responses of CNNs could even take more memory space than storing the model parameters, even with batch size 1.

  → 가중치를 저장하는 것보다 forward pass에서 하나의 데이터를 연산하는 과정에서 발생하는 ‘중간값’ 자체의 용량이 더 클 수 있다.

  3. Number of computing operations: Convolution operations are computationally intensive → 컨볼루션 연산 자체가 빡세다

- Main approach
  - Imposes L1 regularization on the scaling factors in BN layers → Push the values of BN scaling factor towards zero → enables us to identify insignificant channels
  - each scaling factor corresponds to a specific convolutional channel

**3. Network Slimming**

> Advantages of Channnel-level Sparsity

- Sparsity - can be realized at different levels (ex) **weight level, kernel level, channel level, layer level** - Weight-level (fine-grained level): highest flexibility and generality, higher compression rate → but requires special software/hardware accelerators to do fast inference - Layer-level (coarsest level): does not require special packages to harvest the inference speedup → but less flexible, only effective in depth is sufficiently large (>50 layers) - **Channel-level**: nice tradeoff between flexibility and ease of implementation → for FC, treat each neuron as a channel
  > Challenges for channel-level sparsity
- channel-level sparsity requires pruning all the incoming and outgoing connections associated with a channel
  → the method of directly pruning weights on a pre-trained model is ineffective
  → because it is unlikely that all the weights at the input/output end of a channel happen to have near zero values
- Channel-level sparsity: 채널과 연결되어 있는 모든 연결들을 pruning 해야 한다. → 따라서, pre-trained model에서 pruning 시도하는 것은 의미가 없다. → channel input (이전 kernel weights), output (다음 kernel weights)이 전부 다 ‘unimportant’(near zero values) 할 리가 없기 때문이다.

> Solution for challenge: Scaling Factors and Sparsity-induced Penalty

- **Introduce a scaling factor $\gamma$ for each channel**, which is multiplied to the output of that channel
- Then, jointly train the network weights and these scaling factors, with **sparsity regularization imposed on the latter**
- $g()$ is a sparsity induced penalty on the scaling factors; **we choose L1-norm $g(s) = |s|$ , to achieve sparsity**
- **The scaling factors act as the agents for channel selection.**
  > Leveraging the Scaling Factors in BN Layers
- BN layer normalizes the internal activaions using mini-batch statistics
- γ and β are trainable affine transformation parameters (scale and shift)
- It is common practice to insert a BN layer after a convolutional layer, with channel-wise scaling/shifting parameters. Therefore, we can directly leverage the γ parameters in BN layers as the scaling factors we need for network slimming.
  ⇒ **It has the great advantage of introducing no overhead to the network**
- 위는 일반적인 BN 식이다. 즉, 새로운 scaling factor $\gamma$ 를 소개할 필요 없이, BN에 사용되는 scaling factor을 그대로 사용하면 된다. → 추가적인 overhead가 발생하지 않는다.

- 정당성

  1. BN 없이 scaling factor $\gamma$만 사용할 경우: the value of the scaling factors are not meaningful for evaluating the importance of a channel, because both convolution layers and scaling layers are linear transformations. → weights 값 키우고, scaling factor 줄이면, weights 값 줄이고, scaling factor 키우는 거랑 같은 효과를 낼 수 있음. 중요성의 정도 파악 안 됨.

  2. scaling layer before BN: BN의 normalization 과정으로 인해 scaling 효과 무의미해짐

  3. scaling layer after BN: scaling 두 번 되는 꼴, BN 내에서 scaling, scaling factor 두 번 → redundancy
     **⇒ scaling factor로 BN의 scaling을 활용하겠다!**

> Channel Pruning and Fine-tuning

- After training under **channel-level sparsity-induced regularization**, we obtain a model in which many scaling factors are near zero
- Then, we prune channels with near-zero scaling factors
  → (ex) prune 70% channels with lower scaling factors by choosing the percentile threshold as 70%

**5. Analysis**

- Two crucial hyper-parameters in network slimming
  1. pruned percentage $t$
  2. regularization term $\lambda$

> **Effect of Pruned Percentage**

- Too high?
  → may not be able to recover the accuracy by fine-tuning
- Too low?
  → resource saving can be very limited

> **Channel Sparsity Regularization**

- Purpose of the L1 sparsity term ⇒ to force many of the scaling factors to be near zero
- High? the scaling factors are more and more concentrated near zero
- Low? No sparsity regularization, the distribution is relatively flat
- **This process can be seen as a feature selection happening in intermediate layers of deep networks, where only channels with non-negligible scaling factors are chosen.**
