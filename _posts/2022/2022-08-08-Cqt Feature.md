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


画出同一段音频对应的频谱对比图(红色高跟鞋歌曲的前奏)，可以看出，对于 bgm 场景，
cqt 可以更好的刻画音频中的特征。具体来说，可以更好的将能量强的频谱汇聚在少数几个频带上。
这和我们使用 cqt 的初衷是一样的。

<div style="text-align: center"><img src="https://github.com/Liu-Feng-deeplearning/Liu-Feng-deeplearning.github.io/blob/master/images/posts/2022/2022-0808-mels_Vs_cqt.png?raw=true" width="1000" /></div>


### cqt and pseodo cqt

### other feature related cqt

### cqt Vs Chroma

chroma 类是音乐里面另外常用的一系列特征，Chroma的含义是，对所有频带按照音高进行分类，例如将
cqt 中所有表示 Do 的音进行归并，某种程度上来说，在识别对应的音高，以及并不区分相差整八度的数据。
在 librosa 里面对应求法非常 简单，生成一个转换矩阵，直接将 cqt 特征映射成 chroma_cqt 特征。

类似的，绘出相同音频的 Cqt Vs Chroma 如下: 
<div style="text-align: center"><img src="https://github.com/Liu-Feng-deeplearning/Liu-Feng-deeplearning.github.io/blob/master/images/posts/2022/2022-0808-cqt_Vs_chroma.png?raw=true" width="1000" /></div>

chroma 提取了更为高维的音阶特征。12维分别代表每个音阶的能量分布。

### cens Vs Chroma

另外两个常用的特征是 cens 和 crema，例如 COI 的一共常用的开源数据集 DaTacos 会把这些特征做为开源数据集的附带特征。
我们来看下这样两个特征，其中 cens 是在 chroma_cqt 的基础上，做了如下变换:

```text
1. L-1 normalization of each chroma vector
2. Quantization of amplitude based on "log-like" amplitude thresholds
3. (optional) Smoothing with sliding window. Default window length = 41 frames
4. (optional) Downsampling
```

而 crema 特征，来自于 [论文](https://brianmcfee.net/papers/ismir2017_chord.pdf)，
本质相当于对和弦进行分类，然后提取倒数第二层作为 embedding(类似声纹)，相当于对 和弦进行预测，从而得到对应特征。

<div style="text-align: center"><img src="https://github.com/Liu-Feng-deeplearning/Liu-Feng-deeplearning.github.io/blob/master/images/posts/2022/2022-08-08-cens_Vs_chroma.png?raw=true" width="1000" /></div>
    



  



