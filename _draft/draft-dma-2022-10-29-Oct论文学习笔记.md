---
layout: post
title: 2022Oct论文笔记
categories: 读书笔记
description: 
keywords: 读书笔记
---
2022年10月 论文/读书笔记

这周去旁听了 ICASSP-2022 深圳会场的 oral，其中感觉收获很大的是听了微软谭老师关于 
TTS-Survey 和 Neural-Speech 的介绍。 一些关于TTS 的

## TTS 

#### A: [Learning Latent Rep- resentations for Style Control and Transfer in End-to-end Speech Synthesis](https://arxiv.org/pdf/1812.04342.pdf)

微软使用 vae 对 tts 做风格建模的文章，结构上和之前 g 的 gst 有点像，从 target audio 中提取固定长度 vec 作为 latent variables **z**, 
然后推理的时候，从 ref audio 中提取。

为了防止 KL collapse，采取了kl-loss 系数从小到大的策略，防止开始阶段，之间把 z 采成正态分布从而啥都没学到。
除此，一些模型结构和参数也可以参考借鉴。vae 的一个核心 encoder，就是把可变长度的向量/矩阵，
转成一个固定长度的向量。

基本这个结构就是和我自己在 vc 中使用的相同。

## CSI

A: [WideResNet with Joint Representation Learning and Data Augmentation for
Cover Song Identification](https://www.isca-speech.org/archive/pdfs/interspeech_2022/hu22f_interspeech.pdf)

腾讯音乐 Lyra lab 关于 csi 方向的工作。

几个核心的思路和创新点:
1. 使用了分类+表征学习结合的思路（似乎已经不是很新了？）
2. 使用了 prototypical network/loss 
3. SpecAugment （也是 asr 中常用的方法）

prototypical network/loss 虽然看上去比较高级，但似乎感觉不是这里的主要问题？
wide-resnet 模型本身的结构感觉也不会是解决问题的关键因素。

另外就是关于实验部分做得比较详细: 
1. *shs100k级别的数据量下，数据量的扩充可以显著提升指标*。这看上去也很正常。
2. 字节的bytecover 效果复现不出来，这个和我的结论也是基本吻合的。
（之前复现 bytecover，需要很多 trick 才能得出和原论文一样的效果，但其实是可复现的）
3. 一些新的baseline结果，数据很漂亮刷新了 sota，而且提升的幅度不小。但是因为引入了 shs600k 这个大数据集，看上去和之前其他论文中的比较不太"公平"
4. 对比了 PicklyNet，之前没太关注这个方法，查缺补漏

整体来看，论文还不错，虽然立意和创新度不太够，但完整实验还是有很多收获在其中。

---

B: [TONET: TONE-OCTAVE NETWORK FOR SINGING MELODY EXTRACTION FROM 
POLYPHONIC MUSIC](https://arxiv.org/pdf/2202.00951.pdf)

预测 tone/octave 的方法。

1. 重排 cfp 特征。
2. Encoder-Docoder 的backone 架构。 
3. 后处理的融合算法

看上去模型结构比较平常，基本都是其他领域内也很常见的结构。
比较有意思的点是关于 cfp rearrange 部分，将不同 octave 里面相同音节合并（重排）。
这里操作和 chroma 特征有点像，但 chroma 是做了能量的合并，这里仅仅是重排。
从信息角度上并没有任何损失。

看上去，cqt 似乎也可以使用类似操作，是否会对 csi 有所帮助？？

音乐门外汉的粗浅理解，chroma/cfp-rearragne 都是统一处理，将不同频率的 do/mi 归类到一个类别上，
认为他们之间有某种联系，从而抽取更好的表征。例如同一首歌，所有音符上升八度之后，依然是同一首歌。
