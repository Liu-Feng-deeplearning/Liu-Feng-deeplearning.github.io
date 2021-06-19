---
layout: post
title: Kaldi笔记-Tdnnf模型
categories: 语音识别 Kaldi
description:  
keywords: 语音识别， Kaldi
---


最近在某个任务上，尝试对seq进行帧级别的分类，回去仔细研究了一下kaldi中的tdnnf结构，
感觉模型设计非常精巧，对后续学习和帮助很有帮助。


kaldi中tdnnf模型在pytorch上的实现，可以参考我之前的项目
 [kaldi_asr_factorized_tdnn](https://github.com/Liu-Feng-deeplearning/Kaldi_ASR_Factorized_Tdnn)

---
## Factorized Tdnn Layer

Factorized Tdnn 本身的介绍和特点已经很多人有写过对应的解析了。
例如比较好的一篇：[tdnnf介绍](https://qingen.github.io/ai/2020/04/25/tdnnf.html)，也可以阅读原始论文
[paper](http://www.danielpovey.com/files/2018_interspeech_tdnnf.pdf)

几个核心的要点以及对应实现也都有了，但我这次又重新进去研究了一下细节，
发现还是有一些比较有意思的地方。

#### skip-connection
类似resnet，用来实现足够深的网络。

论文里提到了几种不同的 merge option，
包括 small-to-big, big-to-big, big-to-small等，但是在kaldi的源代码中，
论文中说small-to-big效果最好，但好像kaldi源码采用的是big-to-big，不确定这里差异点在哪里。

另外，skip-connect有两种做法，resnet原文是简单的sum，另外一些变种实现中是使用了concat，
两种方案到底有多少差异。

kaldi里面提到了另外一个有趣的点是，connection时候，可以选取之前临近但不连续的若干层，可能会有更好的效果。
实际尝试了一下，确实如此（果然是大神）。这可能是和保持信息传递时的多样性有关吧。


#### dropout
这里提出一种新型的dropout方式，感觉和之前的spec-aug很像。大概也是一种抑制过拟合的有效方法。
同时，可变系数的想法也值得尝试。

这部分代码在 https://github.com/Liu-Feng-deeplearning/Kaldi_ASR_Factorized_Tdnn/blob/main/tdnnf_model.py
中也有实现，之后有机会进行一下验证。


#### 3-stage splicing

比传统tdnn的2-stage结构有更宽的感受野，更少的参数和更快的计算效率。

#### factorize for prefinal layer

factorize 全连接层，同样有效果。同时，Dan的论文里重点提到了怎样在神经网络训练中，使用迭代方法获得正交矩阵也是很有趣。
数学推导浅显易懂，实现起来也比较清晰。实际操作中，会发现全连接层的正交化比卷积层要慢很多（大概是因为矩阵维度显著提高了）。

---
## Kaldi的阅读笔记

首先贴一下kaldi实现tdnnf的经典结构。

#### kaldi/egs/swbd/s5c/local/chain/tuning/run_tdnn_7p
基本上就是经典的 tdnnf 1280->256->1280 的结构，基本和论文一致。
可以重点关注一下skip-connection的设计。

```text
  input dim=100 name=ivector
  input dim=40 name=input

  # please note that it is important to have input layer with the name=input
  # as the layer immediately preceding the fixed-affine-layer to enable
  # the use of short notation for the descriptor
  fixed-affine-layer name=lda input=Append(-1,0,1,ReplaceIndex(ivector, t, 0)) affine-transform-file=$dir/configs/lda.mat

  # the first splicing is moved before the lda layer, so no splicing here
  relu-batchnorm-dropout-layer name=tdnn1 $opts dim=1280
  linear-component name=tdnn2l0 dim=256 $linear_opts input=Append(-1,0)
  linear-component name=tdnn2l dim=256 $linear_opts input=Append(-1,0)
  relu-batchnorm-dropout-layer name=tdnn2 $opts input=Append(0,1) dim=1280
  linear-component name=tdnn3l dim=256 $linear_opts input=Append(-1,0)
  relu-batchnorm-dropout-layer name=tdnn3 $opts dim=1280 input=Append(0,1)
  linear-component name=tdnn4l0 dim=256 $linear_opts input=Append(-1,0)
  linear-component name=tdnn4l dim=256 $linear_opts input=Append(0,1)
  relu-batchnorm-dropout-layer name=tdnn4 $opts input=Append(0,1) dim=1280
  linear-component name=tdnn5l dim=256 $linear_opts
  relu-batchnorm-dropout-layer name=tdnn5 $opts dim=1280 input=Append(0, tdnn3l)
  linear-component name=tdnn6l0 dim=256 $linear_opts input=Append(-3,0)
  linear-component name=tdnn6l dim=256 $linear_opts input=Append(-3,0)
  relu-batchnorm-dropout-layer name=tdnn6 $opts input=Append(0,3) dim=1536
  linear-component name=tdnn7l0 dim=256 $linear_opts input=Append(-3,0)
  linear-component name=tdnn7l dim=256 $linear_opts input=Append(0,3)
  relu-batchnorm-dropout-layer name=tdnn7 $opts input=Append(0,3,tdnn6l,tdnn4l,tdnn2l) dim=1280
  linear-component name=tdnn8l0 dim=256 $linear_opts input=Append(-3,0)
  linear-component name=tdnn8l dim=256 $linear_opts input=Append(0,3)
  relu-batchnorm-dropout-layer name=tdnn8 $opts input=Append(0,3) dim=1536
  linear-component name=tdnn9l0 dim=256 $linear_opts input=Append(-3,0)
  linear-component name=tdnn9l dim=256 $linear_opts input=Append(-3,0)
  relu-batchnorm-dropout-layer name=tdnn9 $opts input=Append(0,3,tdnn8l,tdnn6l,tdnn5l) dim=1280
  linear-component name=tdnn10l0 dim=256 $linear_opts input=Append(-3,0)
  linear-component name=tdnn10l dim=256 $linear_opts input=Append(0,3)
  relu-batchnorm-dropout-layer name=tdnn10 $opts input=Append(0,3) dim=1536
  linear-component name=tdnn11l0 dim=256 $linear_opts input=Append(-3,0)
  linear-component name=tdnn11l dim=256 $linear_opts input=Append(-3,0)
  relu-batchnorm-dropout-layer name=tdnn11 $opts input=Append(0,3,tdnn10l,tdnn9l,tdnn7l) dim=1280
  linear-component name=prefinal-l dim=256 $linear_opts

  relu-batchnorm-layer name=prefinal-chain input=prefinal-l $opts dim=1536
  linear-component name=prefinal-chain-l dim=256 $linear_opts
  batchnorm-component name=prefinal-chain-batchnorm
  output-layer name=output include-log-softmax=false dim=$num_targets $output_opts

  relu-batchnorm-layer name=prefinal-xent input=prefinal-l $opts dim=1536
  linear-component name=prefinal-xent-l dim=256 $linear_opts
  batchnorm-component name=prefinal-xent-batchnorm
  output-layer name=output-xent dim=$num_targets learning-rate-factor=$learning_rate_factor $output_opts
```

---

#### kaldi/egs/swbd/s5c/local/chain/tuning/run_tdnn_7q

7q和7p相比，使用了更简洁清晰的结构，skip方式也更统一。两者效果大体相当，但少数据量时，可能后者更好。

```text
  input dim=100 name=ivector
  input dim=40 name=input

  # please note that it is important to have input layer with the name=input
  # as the layer immediately preceding the fixed-affine-layer to enable
  # the use of short notation for the descriptor
  fixed-affine-layer name=lda input=Append(-1,0,1,ReplaceIndex(ivector, t, 0)) affine-transform-file=$dir/configs/lda.mat

  # the first splicing is moved before the lda layer, so no splicing here
  relu-batchnorm-dropout-layer name=tdnn1 $affine_opts dim=1536
  tdnnf-layer name=tdnnf2 $tdnnf_opts dim=1536 bottleneck-dim=160 time-stride=1
  tdnnf-layer name=tdnnf3 $tdnnf_opts dim=1536 bottleneck-dim=160 time-stride=1
  tdnnf-layer name=tdnnf4 $tdnnf_opts dim=1536 bottleneck-dim=160 time-stride=1
  tdnnf-layer name=tdnnf5 $tdnnf_opts dim=1536 bottleneck-dim=160 time-stride=0
  tdnnf-layer name=tdnnf6 $tdnnf_opts dim=1536 bottleneck-dim=160 time-stride=3
  tdnnf-layer name=tdnnf7 $tdnnf_opts dim=1536 bottleneck-dim=160 time-stride=3
  tdnnf-layer name=tdnnf8 $tdnnf_opts dim=1536 bottleneck-dim=160 time-stride=3
  tdnnf-layer name=tdnnf9 $tdnnf_opts dim=1536 bottleneck-dim=160 time-stride=3
  tdnnf-layer name=tdnnf10 $tdnnf_opts dim=1536 bottleneck-dim=160 time-stride=3
  tdnnf-layer name=tdnnf11 $tdnnf_opts dim=1536 bottleneck-dim=160 time-stride=3
  tdnnf-layer name=tdnnf12 $tdnnf_opts dim=1536 bottleneck-dim=160 time-stride=3
  tdnnf-layer name=tdnnf13 $tdnnf_opts dim=1536 bottleneck-dim=160 time-stride=3
  tdnnf-layer name=tdnnf14 $tdnnf_opts dim=1536 bottleneck-dim=160 time-stride=3
  tdnnf-layer name=tdnnf15 $tdnnf_opts dim=1536 bottleneck-dim=160 time-stride=3
  linear-component name=prefinal-l dim=256 $linear_opts

  prefinal-layer name=prefinal-chain input=prefinal-l $prefinal_opts big-dim=1536 small-dim=256
  output-layer name=output include-log-softmax=false dim=$num_targets $output_opts

  prefinal-layer name=prefinal-xent input=prefinal-l $prefinal_opts big-dim=1536 small-dim=256
  output-layer name=output-xent dim=$num_targets learning-rate-factor=$learning_rate_factor $output_opts
```


同时，kaldi源代码中，给出了一些关于 tdnnf-layer 的细节说明。
注意看这里对bypass的处理。

 
```text
# This class is intended to implement an extension of the factorized TDNN
# (TDNN-F) that supports resnet-type 'bypass' connections.  It is for lines like
# the following:
#
# tdnnf-layer name=tdnnf2 dim=1024 bottleneck-dim=128 dropout-proportion=0.0 time-stride=3
#
# The line above would be roughly equivalent to the following four lines (except
# for different naming, and the use of TdnnComponent, for efficiency, in place
# of AffineComponent).  Assume that the previous layer (the default input) was tdnnf1:
#
#  linear-component name=tdnnf2.linear dim=128 orthonormal-constraint=-1.0 input=Append(Offset(-3, tdnnf1), tdnnf1)
#  relu-batchnorm-dropout-layer name=tdnnf2.affine dim=1024 dropout-proportion=0.0 \
#    dropout-per-dim-continuous=true input=Append(0,3)
#  no-op-component name=tdnnf2 input=Sum(Scale(0.66,tdnnf1), tdnn2.affine)
#
#  Documentation of some of the important options:
#
#   - dropout-proportion
# This gets passed through to the dropout component.  If you don't set
# 'dropout-proportion', no dropout component will be included; it would be like
# using a relu-batchnorm-layer in place of a relu-batchnorm-dropout-layer.  You
# should only set 'dropout-proportion' if you intend to use dropout (it would
# usually be combined with the --dropout-schedule option to train.py).  If you
# use the --dropout-schedule option, the value doesn't really matter since it
# will be changed during training, and 0 is recommended.
#
#  - time-stride
# Controls the time offsets in the splicing, e.g. if you set time-stride to
# 1 instead of the 3 in the example, the time-offsets would be -1 and 1 instead
# of 1 and 3.
# If you set time-stride=0, as a special case no splicing over time will be
# performed (so no Append() expressions) and the second linear component (named
# tdnnf2l in the example) would be omitted, since it would add no modeling
# power.
# You can set time-stride to a negative number which will negate all the
# time indexes; it might potentially be useful to alternate negative and positive
# time-stride if you wanted to force the overall network to have symmetric
# context, since with positive time stride, this layer has more negative
# than positive time context (i.e. more left than right).
#
#  - bypass-scale
#
# A scale on the previous layer's output, used in bypass (resnet-type)
# connections.  Should not exceed 1.0.  The default is 0.66.  If you set it to
# zero, the layer will lack the bypass (but we don't recommend this).  won't use
# a bypass connection at all, so it would be like conventional TDNN-F Note: the
# layer outputs are added together after the batchnorm so the model cannot
# control their relative magnitudes and this does actually affect what it can
# model.  When we experimented with having this scale trainable it did not seem
# to give an advantage.
#
#  - l2-regularize
# This is passed through to the linear and affine components.  You'll normally
# want this to be set to a nonzero value, e.g. 0.004.
``` 

---

最后感慨，DanPovey 真大神，kaldi 源码常读常新，受益匪浅。

