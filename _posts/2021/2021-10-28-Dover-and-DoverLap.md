---
layout: post
title: Dover and DoverLap 
categories: 算法与数据结构
description: 
keywords: 算法与数据结构
---

Dover 算法感觉已经成为 Dihard 比赛里的标配算法了，最近在做一个 Speaker Diarization 算法的综述，
顺便就看了一下 Dover 算法相关内容记录如下。

Speaker Diarization 问题就不过多介绍了，本质是对给定序列打标签对问题。

Dover(diarization output voting error reduction) 微软2019年提出的
多个系统结果融合的一种投票方法，但和一般分类问题不同，
需要先把不同系统的结果映射到同一个label空间内，因此，流程上分成mapping 和voting 两个部分。

[Dover 原始论文](https://www.microsoft.com/en-us/research/uploads/prod/2019/09/DOVER__A_Method_for_Combining_Diarization_Outputs__ASRU_2019.pdf)


**label-mapping**

目标是把不同系统的结果对应到同一标签空间。采用的方法是，minimizes the total time duration of speaker mismatches
本质上是一个图论问题，二分图最大权重。可以使用匈牙利算法 https://zhuanlan.zhihu.com/p/96229700

然后采用 pair wise 方式从两个系统扩展到 N 个系统。这里出现了 DOVER 的第一个缺点，就是这个过程和系统合并的顺序有关。
其中第一个系统因为要参与后面每次合并过程，因此最为重要。
论文里也提到了，一个策略是找到和其他系统 DER 最小的系统作为初始情况。还有一种方法是将每个系统各自作为初始系统，各跑一轮再取平均（性能会损失一个数量级）。

**voting**

对齐之后，将整个音频切成若干小段，vote 即可。这里为了防止过多低权重系统对高权重系统的影响，作者提出了 rank-weighting，即对不同排序的系统权重再乘一个系数，从而规避这种影响。

这里又引出了 DOVER 的另一个缺点，就是投票只取权重最高者，导致难以处理 overlap 的重叠。


鉴于这两个问题，DanPovey 他们提出了一个新的方案：DoverLap

### Dover Lap


整体的 pipeline 流程和 Dover 非常像，同样是 mapping 和 voting 两部分。

[DoverLap Paper](https://danielpovey.com/files/2021_slt_doverlap.pdf)

**mapping**

这里改进了之前的 pair-wise 方法。

定义 M(a0, b0) 为 a0 和 b0 之间的重叠段长度，M 越大，说明a0和b0表示同一speaker的可能性越高。

$$a_i$$


构造一个 N 维向量 $C$ as cost-tensor ，以下为叙述简便，假设只有三个系统。我们可以这样定义：

$$C(a_i, b_j, c_k) = -(M(a_i, b_j)+M(a_i, c_k), M(b_j, c_k))$$
 
 (Note: **原文这里有个笔误**)
$C(a_i, b_j, c_k)$ 刻画了 $Ca_i, b_j, c_k$ 属于同一speaker时的可能性。

接下来，问题变成需要寻找一组 $S = {(a_i0, b_j0, c_k0), (a_i1, b_j1, c_k1), ...}$, 
使得 $\Sigma C(S)$ 最小。一个显然的思路是使用贪心算法（原文也是这样做的），按照我自己的理解，
把 algorithm 部分伪代码重写了一下来加深理解。

```math
S = {(a_i0, b_j0, c_k0), ...} // 所有可能的情况
M = {empty}
R = {S}
while R is not empty：
  x = find-min（x in R for C（x）// 排序时候就只排那些合法的x
  M = M + x
  R = remove-union（x） // 包括    
```

  
  
感觉上，原文这里是使用 sorted，可以改成 privority 队列，速度可以更快。但是，实际场景下，
K和N往往都很小，所以这里讨论复杂度意义并不是很大。

**voting** 
nT = int（）
投票过程差别不大，采用 max-n，n通过系数求得来 代替之前的 max 即可

**Rover and Dover**
像微软那篇

**后记**

之前看DiHardIII比赛的工程中，貌似Dover以及成为几乎所有参赛队伍的标配了。看上去无脑堆系统总是可以取得更好的效果。

不过里面还是又很多有意思的算法上的内容，特别是原始版本Dover里的mapping算法，一开始以为贪心很容易，后来发现还是有点复杂的。

毕竟没有实际动手写，纸上谈兵，并不确定这篇文章是否正确。

