---
layout: post
title: ddpm 学习笔记之 WaveGrad 解析
categories: 语音信号处理
description: 
keywords: ddpm, 声码器, tts, WaveGrad
---

最近有空在 follow DDPM 方法在 vocoder/TTS/VC 等语音生成问题的应用。
先写下第一篇学习笔记，关于简单介绍 ddpm 及其在声码器中的一个简单应用 (waveGrad)，
借此机会先梳理下整个方法的脉络和要点。

## DDPM 方法简介

ddpm-Denoising Diffusion Probabilistic Model，是进来在生成领域非常火的一类模型。
抛弃严谨的数学推导，
我用自己的理解大概讲一下：传统方案下，需要从无到有/从高斯分布的噪声生成对应的图像，
这个过程非常困难，需要模型从中学到很多东西。扩散模型希望把整个过程拆成若干步，
使用模型每次重建一点点，逐次生成最终的模型。为了能够做到这个事情，需要定义一大堆的中间状态。
考虑生成问题的逆问题，从一张图片生成一个高斯分布，非常简单。每次往图片中随机加一点噪声，
重复足够多的次数，就得到了最终的高斯噪声。记录这个中间过程，想办法逆向完成这个加噪的过程，
即可完成扩散模型。

<div style="text-align: center"><img src="https://github.com/Liu-Feng-deeplearning/Liu-Feng-deeplearning.github.io/blob/master/images/posts/2022/2022-08-26-ddpm-0.png?raw=true" width="600" /></div>


## 声码器

声码器是语音领域里一个经典的生成问题
>对于给定一个mel谱 x，怎样生成对应的信号


y -> x 过程非常简单，但逆问题想做到高精度/效率兼顾却比较难。

另外，声码器和图像生成有一个显著的区别。图像生成中，condition 一般是一个类别标签。
对于同一类别标签，显然有各种风格迥异的图片生成。但对于声码器，mel 是稠密的特征，一旦给定
 ，音频信号大体上确定了。和音频信号信息相比，谱图中只有相位信息以及 fft 高频截断信息被丢掉了。
而高频信息其实人耳听上去的主观感受并不明显。这表明在 cond 条件下，音频生成的丰富度，远低于图像生成问题。
**声码器更近似与一个 1 vs 1 的问题，而不是 1 vs N**。
一些生成问题中常见的 badcase 现象(例如 model-collasp/over-smooth) 在这里表现的并不明显。

因此类似 hifigan 等 gan-based 方法在效果方面表现并不差。大家更关注的还是效果/性能之间的平衡。

## WaveGrad: ddpm for vocoder

<div style="text-align: center"><img src="https://github.com/Liu-Feng-deeplearning/Liu-Feng-deeplearning.github.io/blob/master/images/posts/2022/2022-08-26-ddpm-1.png?raw=true" width="800" /></div>

wavegrad 应该是一个比较早使用 ddpm 的声码器框架，作者开源了其源码，结合论文和代码来做一些解析。

首先来看下算法的基本架构，

<div style="text-align: center"><img src="https://github.com/Liu-Feng-deeplearning/Liu-Feng-deeplearning.github.io/blob/master/images/posts/2022/2022-08-26-ddpm-2.png?raw=true" width="800" /></div>

基本上是一个比较经典的 ddpm 流程，包括 train/sample 两部分

### train

在训练部分，作者设计了一个神经网络模型, nnet_model, 每次从不同scale中，
带噪语音中预测噪声（进而可以得到干净的语音），使得预测噪声和原始生成随机噪声尽可能接近。

```text
noise ~ N(0, 1)
noise_sig = nosie * noise_scale + clean_sig
pred_noise = nnet_model(noise_sig, mel, noise_scale)
Loss = ce(pred_noise, noise)
```

训练过程中，注意对 batch 中每个样本，都要分别产生对应的高斯分布。
以及产生随机的 noise-scale。

### model 设计

模型本身的设计上，没太多好讲的。包括了 Ublock/Film/Dblock 等模块。不过在 Film的时候，
里面加入了 Position encoding，我记得原版是没有的？这里可能要额外注意一下。

### inference/sample

在推理过程中，从随机高斯噪声出发进行采样，迭代求解。注意每次迭代求的时候，
也要和前向过程类似加入随机噪声。具体的公式可以参考原始论文/代码进行推导。

gradWave 的一个优化是对推理步骤，正常来讲 T=1000，这样一个速度是无法接受的。
代码里对此进行了精简，只用 6 次推理。为了保证 T 降低后效果损失较小，作者选取了一个验证集，
在验证集上进行测试，选取最优参数。(这应该也是一个相对比较容易想到和实现的方案了)

### 实验结果

这里就不放论文本身的结果了，但凡能发论文的，大家都会说自己的方法吊打他人。

我自己重新训练和测试了一轮，包括解码时候仔细调了一下参数。我使用了 bzn 的经典数据集 (1w句)，
效果很不错，至少不会比 hifigan 差（6steps）, 不过确实解码速度比较慢，即使只用6steps，
依然会比hifigan 慢一个数量级。

性能依然是 waveGrad 方法的一个瓶颈。


### Ref
- 苏神关于 ddpm 的通俗讲解:盖楼与拆楼 https://spaces.ac.cn/archives/9119
- lilan大神关于 ddpm 更系统的讲解 https://lilianweng.github.io/posts/2021-07-11-diffusion-models/#nice
- WaveGrad https://arxiv.org/pdf/2009.00713.pdf
- WaveGrad 源码 https://github.com/lmnt-com/wavegrad
- ddpm 原始论文 https://arxiv.org/pdf/2006.11239.pdf