---
layout: post
title: Fusing Force-Alignment with Vad
categories: 语音识别
description: 
keywords: 语音信号处理
---

Force alignment(FA) 是语音领域中一个常见的问题，输入是音频和对应文本字符，
输出每个字符的起始终止时间戳，相当于将句标签（强标签）转成了帧标签（弱标签）。
FA 在语音领域里的很多场景做为数据前处理的重要一环出现，例如 Durian-based tts 中利用
FA 生成 duration model 训练数据。Hybrid-Asr 中利用 FA 结果训练声学模型等。

Kaldi 中基于 GMM-HMM(包括后来的 Tdnn-HMM 和 CNN-HMM) 的方案一直是 FA 领域里的主流方案，
例如 MFA 等。这些方法可以取得相当不错的效果，但实际实践过程中会发现 FA 的结果可能会在 sil 段
附近，把更多的静音段包括到 sil 附近的发音单元中来，**特别是浊辅音和静音段之间的差别**。

我们使用了一个非常简单的 trick，非常有效的解决了这个问题。这个方法在多个场景中得以验证，
具有相当好的普适性。这个简单的思路就是：使用 vad 的结果来进行融合。

备注: 这里所说 VAD 均指基于能量的 VAD 方法，不是神经网络 VAD。相关介绍可以参考 [Energy Based Vad 原理与应用
](https://liu-feng-deeplearning.github.io/2021/04/19/Energy-Based-Vad%E5%8E%9F%E7%90%86%E4%B8%8E%E5%BA%94%E7%94%A8/)。


思路很简单，(特别是对 tts 场景下这样干净的数据集)即使基于能量的 vad，也能对 sil 段
取得比较准确的预测效果。因此，可以使用 vad 结果对 FA 结果进行修正。

融合方法有很多，一个被我们验证比较好的方案如下：

如果 FA/VAD 都预测某个位置附近有 sil 段（两个结果有交集），
那么我们使用 VAD 的结果。如果 FA 预测出了某个位置的 sil 标签，而 VAD 没有，删除对应的 sil。
其他情况下，**更信任 FA 对 sil 的预测结果**。 

一些融合的 case 如下: 

(s for sil)

```text
fa:  |  a  |  b  |  c  |
vad: ----| s | --------|
->   |  a  |  b  |  c  |

fa:  |  a  | s |  b  |  c  |
vad: -----------------------
->   |  a    |    b  |  c  |

fa:  |  a  | s |  b  |  c  |
vad: ----| s | -------------
->   |  a| s |    b  |  c  |

fa:  |  a  | s |  b  |  c  |
vad: --------------| s |----
->   |  a    |    b  |  c  | 
```

这样做的原因是，(特别是对 tts 场景下这样精细标注的数据集) 对于不短于一定长度的静音段，文本标签中
标出了对应的 sil，Baseline-FA 的结果不会漏掉那些长停顿。
同时，我们对输入文本和算法参数进行了一些处理，保证不会在中文词中间预测出 sil。
以上两点，保证了我们的融合算法要解决的问题只有 1. 词间短停顿的错误预测 2. sil 边界的误差。
因此采用了以上的方案。


该算法的一般流程：

1. get sil duration from force alignment. (fa-sil-dur)
2. if sil not at head or tail of fa-sil-dur, add short sil to make sure fa-sil-dur 
with sil begin and sil end. 保证每个句子的开头和结尾都是 sil，实际上大多数数据本来就是这样。
3. get vad sil durtaion from vad method. (vad-sil-dur)
4. change vad sil to make sure vad-sil-dur with sil begin and sil end. 同step2
5. for sil pairs in fa-sil-dur, find target sil dur in vad-sil-dur. 找到 FA he VAD 结果的交集
6. change fa-sil-dur with target sil dur. 修改边界
 if no target sil dur, remove the sil from fa-sil-dur and using middle of sil as new boundary.
 else using vad-sil-dur instead of fa-sil-dur
 and we make sure every phn be longer than min_phn(0.03s)
 


我们也尝试了其他的一些融合方法，例如融合策略中更相信 VAD 的结果。使用 VAD 结果做为参照，
例如：

```text
proposed:
fa:  |  a  |  b  |  c  |
vad: ----| s | --------|
->   |  a  |  b  |  c  |

another method:
fa:  |  a  |  b  |  c  |
vad: ----| s | --------|
->   |  a| s |b  |  c  | 
```

根据在项目中的经验来看，大多数时候，特别是数据集相对干净的情况下，两种情况差别较小。
其他情况下，proposed method 表现**稍好**。两者都远好于不用 vad 的情况。

