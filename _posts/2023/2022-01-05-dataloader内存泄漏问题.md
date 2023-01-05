---
layout: post
title: dataloader内存泄漏问题
categories: 其他
description: 
keywords: 语音信号处理
---

记录最近遇到的一个 PyTorch 训练内存泄漏的问题。查了好几天有点棘手，大费周折。

### 问题现象

正常训练中，同一个 epoch 内，内存会随着 step 缓慢增加，epoch 结束后内存清空。
因此在长时间维度下，内存占用锯齿状，如下图: 

<div style="text-align: center"><img src="https://github.com/Liu-Feng-deeplearning/Liu-Feng-deeplearning.github.io/blob/master/images/posts/2023/2023-01-06-memory.png?raw=true" width="800" /></div>

当数据量足够大时，同一个 epoch 内，内存的最大占用超过机器上限，导致结果 kill 掉。显然这是由于内存泄漏导致的。
虽然 Torch 的内存管理被诟病多年，但在仔细分析之前先别盲目甩锅，探查一下原因对提升相关知识还是很有帮助的。

目前使用 torch 版本为 1.12.0

### 测试方法

Python 不比 C++，很多地方对内存的管理和使用说得不清不楚的。在探究之前先找了找适合查看内存环境的工具。

**方案1-Linux-top**

利用 linux 原生命令直接看内存 RSS 物理占用。粗暴直接，不容易出问题。但查看的是进程整体使用情况，粒度比较粗。

使用如下代码打印内存随时间的变化

```bash
proc=thread_name # or PID
while true;
do
    msg=`ps aux|grep ${proc}|grep -v grep | grep -v collect`
    date_time=`date "+%Y-%m-%d %H:%M:%S"`
    echo ${date_time}" "${msg}
    sleep 3
done
```

**方案2-tracemalloc package**

tracemalloc 是网上很多人推荐的方法，这个包能拿到某个时刻内存使用情况的快照，
然后相邻两次做差得到对应内存增长。但比较遗憾的是，这个包似乎对 pytorch 内部的一些函数不起作用。原因未知。
不过一开始使用这个包得出结论是错误的，导致在错误的方向上走了好久。不推荐。


**方案3-memory_profiler package**

这是 python 下的另一个包，使用方法比较多，一个简单的用法就是将要分析的代码抽象成函数，然后在该函数前加一个名为 profiler 的装饰器，正常运行即可。
运行结束后会给出逐行分析的内存结果。我在测试中会发现，当大批量跑循环的时候，有时对应到具体行可能会有误差，但大体上问题不大。

因此我分析的工具主要选用了方案3和方案1相互验证比对。

### 一些调研+问题分析

作为一个菜鸟，在网上搜了搜网友遇到的类似的问题，然后在我自己的代码里进行比对查找。一些结论:

**几乎绝大部分内存泄漏的问题都和 dataloader 有关**，这里是重点的检查区域。将 num_worker 设置为 0，单线程更容易查找问题。

**python/Numpy 代码本身的问题** 

这部分内容和 torch 无关。主要是 numpy 造成的泄漏。[一个比较好的博客对这个问题的总结。](https://zhuanlan.zhihu.com/p/80689571)
经查，确实有泄漏。主要是 numpy 一些操作返回结果(特别是切片操作)是 view 而非拷贝。这部分占比不低。
一些修改的方案是在某些位置使用 copy 函数进行拷贝，但对于 dataloader 内操作的代码，对性能(耗时)极其敏感，这里可能还要更多的权衡一下。

**torch数据类型导致的问题**

这部分网上很多帖子也有所提到，主要是 list 转 tensor 的时候，如果 list 内元素是原生的 python 类型，没有问题。如果 list 内元素是numpy scalar，有问题。但据说已经在新版本中修复了这个 bug，未验证。
如果是 numpy 转 tensor，没问题。而且最好是在转换的时候，手动指定数据类型。为了防止出问题，代码直接都采用最后一种方案(tensor.from_numpy)

其他的两个问题:

**model.forward函数的检测**

可能一些手写的算子，特别是求 loss 时候如果有大段的自定义代码，可能需要检查看看。

**更新时对梯度的处理**

在该截止梯度的时候要截止。例如 loss 写入的时候要使用 loss.item()

但是后两种情况一般情况下出现几率比较小，即使出现了相对来说占比不太主要。

### 结论

主要还是 review/refactor 整个 dataloader 内部代码，锯齿的情况并未完全消除，但可以大幅度降低。
从而完全不会出现之前说的内存溢出的问题。Torch dataloader 因为运算次数巨大，
所以 __get_item__ 里面每条语句都应该慎之又慎。必要时对 dataloader 进行单元测试。

python 对内存的管理也比较迷，类似 del xxx 之类，只是对 xxx 的引用减1，所以有时del不见得能完全生效。
变量的引用计数器的存在，有时使得代码的内存情况不是特别直观，以及有时一些变量会临时住内存以加速。
和c++相比，python 管理内存有时感到有点无力，也可能 python 设计的初衷就是让程序员**不要**考虑内存之类底层的东西。
如果对内存敏感，就换成别的语言。








# import numpy as np
# import torch
# from memory_profiler import profile
#
# from src.csi.trainer_and_dataset.dataset import AudioFeatDataset, \
#   MPerClassSampler
# from src.csi.v17.model import Model
# from src.utils.utils import load_hparams


# @profile
# def _check_dataset():

https://zhuanlan.zhihu.com/p/80689571


import tracemalloc
不work



python3 -m src.csi.trainer_and_dataset.dataset > memory.log 2>&1