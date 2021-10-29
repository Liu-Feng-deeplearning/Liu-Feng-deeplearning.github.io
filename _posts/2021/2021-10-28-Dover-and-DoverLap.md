---
layout: post
title: Dover and DoverLap 
categories: 算法与数据结构
description: 
keywords: 算法与数据结构
---

最近花了不少精力在总结整理 speaker diarization 的相关内容，发现了一类专门适合 diarization
的融合算法，特此记录整理。

Speaker Diarization 具体问题就不过多介绍了，本质是一个 seq to seq 的问题。
因此融合算法要比一般的分类问题更复杂一点。

Dover(diarization output voting error reduction) 算法微软2019年最初提出的一种专门解决
此类问题的方法。大体思路是先把不同系统的结果映射到同一个 label 空间内，然后在每个时间段内做投票。

因此，Dover 算法在流程上分成 mapping 和 voting 两个部分。本位先从此算法展开。

### Dover

[Dover 原始论文](https://www.microsoft.com/en-us/research/uploads/prod/2019/09/DOVER__A_Method_for_Combining_Diarization_Outputs__ASRU_2019.pdf) 

**label-mapping**

目标是把不同系统的结果对应到同一标签空间，使得不同系统之间 speaker mismatch 最小。

首先考虑只有两个系统的情况。开始我以为一个简单的贪心就可以搞定，后来发现好像不行。😭
这个问题可以抽象成图论中的二分图最大权重问题，进而可以采用匈牙利算法及其扩展方法解决。

这里略过具体的算法细节。一些相关介绍和实现:
- [匈牙利算法](https://zhuanlan.zhihu.com/p/96229700)
- [二分图最大权重问题的实现](https://www.cxyzjd.com/article/xiaoli_nu/74011927)

两个系统的 map 解决后，采用 pairwise 方式扩展到 N 个系统。
这里出现了 DOVER 的第一个缺点，就是这个扩展过程和系统合并的顺序有关。
第一个出现系统要参与后面每次扩展过程，因此最为重要。
论文里也提到了，一个策略是找到和其他系统 DER 最小的系统作为初始情况。还有一种方法是将每个系统各自作为初始系统，各跑一轮再取平均（性能会损失一个数量级）。

一个简单的示意图如下，图片来自原始论文：
<div style="text-align: center"><img src="https://github.com/Liu-Feng-deeplearning/Liu-Feng-deeplearning.github.io/blob/master/images/posts/2021/2021-10-28-Dover-and-DoverLap.png?raw=true" width="360" /></div>

**voting**

对齐之后，将整个音频切成若干小段，vote 即可。这里为了防止过多低权重系统对高权重系统的影响，作者提出了 rank-weighting，即对不同排序的系统权重再乘一个系数，从而规避这种影响。

这里又引出了 DOVER 的另一个缺点，就是投票只取权重最高者，导致难以处理 overlap 的重叠。

鉴于这两个问题，Shinji 和 DanPovey 他们提出了一个新的方案：DoverLap

### Dover Lap

[DoverLap 原始论文](https://danielpovey.com/files/2021_slt_doverlap.pdf)

整体的 pipeline 流程和 Dover 非常像，同样是 mapping 和 voting 两部分。


**mapping**

这里改进了之前的 pair-wise 方法。

定义 $M(a_0, b_0)$ 为 $a_0$ 和 $b_0$ 之间的重叠段长度，M 越大，说明 $a_0$ 和 $b_0$ 表示同一speaker的可能性越高。

构造一个 N 维向量 $C$ as cost-tensor ，以下为叙述简便，假设只有三个系统。我们可以这样定义：

$$C(a_i, b_j, c_k) = -(M(a_i, b_j)+M(a_i, c_k), M(b_j, c_k))$$
 
 (Note: **原文这里有个笔误**)
$C(a_i, b_j, c_k)$ 刻画了 $Ca_i, b_j, c_k$ 属于同一speaker时的可能性。

接下来，问题变成需要寻找一组 $S = \{(a_{i0}, b_{j0}, c_{k0}), (a_{i1}, b_{j1}, c_{k1}), ...\}$, 
使得 $\Sigma C(S)$ 最小。一个显然的思路是使用贪心算法（原文也是这样做的），按照我自己的理解，
把 preduso-algorithm 部分重写了一下来加深理解。

```text
S = {(a_i0, b_j0, c_k0), ...}  // S 中包括了所有可能的情况
M = {empty}
R = {S}
while R is not empty：
  x = find-min(C(x) for x in R) // 排序时候就只排那些合法的 x
  M = M + x  
  R = remove(R, x) // 这里把 R 中所有和 x 有冲突的都移除    
```
算法的复杂度核心在于第五行的排序。原文这里使用的是 sort，显然如果改成 priority 队列，
速度可以更快一点。但是，在实际场景下，融合系统数 N 和说话人个数往往都很小，常数项开销占比不可忽略，
所以这里讨论复杂度意义并不是很大。

**voting** 

投票过程和 Dover 差别不大。对每个时间段，通过 rank-weight 求得 $n_{spk}$ 代表同一时刻
允许的最多说话人数量即可。

$$ n_{spk} = \Sigma n_k * w_k $$


### Rover and Dover

在 Dover 原文中提出了关于 Rover(Recognize output voting error reduction) 和 Dover 方法作为 dual question 的观点，感觉还是有点意思。
Rover 中，多个识别结果在空间标签上已经统一（对应同一份发音词典），但需要在时间维度做alignment。
Dover 中，多个识别结果在时间维度上已经同一，但需要在空间标签维度上做 mapping。两者在问题结构上具有高度相似性。

（虽然 duality 好像对解决问题并没有什么帮助？）

### 后记

在 NCMMSC2021-DiHardIII 的报告会上，发现Dover几乎成为所有参赛队伍的标配了。
据参赛的选手讲，无脑融合总是可以取得比单系统更好的效果（这一点其实我有点存疑）。

不过，仔细研究一下，发现融合算法中还是有很多有意思的内容，学习一下还是受益匪浅。
特别是原始版本 Dover 里的 mapping 算法，一开始以为贪心很容易，后来发现实现有点复杂的。

毕竟没有实际动手实践，纸上谈兵，并不确定这篇文章是否正确。
