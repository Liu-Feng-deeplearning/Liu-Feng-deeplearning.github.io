---
layout: post
title: Constant-Q transform Feature
categories: 音频信号处理
description: 
keywords: 
---

Cqt(Constant-Q transform) 特征是音乐领域里音频处理的常见特征。

在音乐信息抽取(MIR)，
翻唱检测(COI)和音乐歌词识别(LSR)等领域里都很常用。
最近刚好在做这方面内容，特此回去又复习了一下相关领域的知识。

### CQT overview

cqt 特征本质上就是 fft 的变种，相邻两个频谱频带宽度的比值为常数，通常为 2，
因为这一特性刚好和音乐中的音高倍数相同，因此 cqt 常被用在音乐领域。

具体计算方法可以直接参考 [wikipedia](https://en.wikipedia.org/wiki/Constant-Q_transform)

### Cqt Vs Mel

首先同一段音频对应的频谱对比图(红色高跟鞋歌曲的前奏)，可以看出，对于 bgm 场景，
cqt 可以更好的刻画音频中的特征。具体来说，可以更好的将能量强的频谱汇聚在少数几个频带上。
这和我们使用 cqt 的初衷是一样的。

<div style="text-align: center"><img src="https://github.com/Liu-Feng-deeplearning/Liu-Feng-deeplearning.github.io/blob/master/images/posts/2022/2022-0808-mels_Vs_cqt.png?raw=true" width="600" /></div>



### cqt and pseodo cqt

### other feature related cqt 

  



