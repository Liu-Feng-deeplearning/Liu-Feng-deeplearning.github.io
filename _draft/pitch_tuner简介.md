
Pitch Tuner

基于 Psola 采样点级别的 Pitch 调整算法。

Pitch/F0 目前已经成为了语音中非常常用的特征，特别是针对声调语言（中日韩越粤等语言）。
中文中， pitch 的高低不仅影响汉字的音调，同时也是反应语言情绪表达的主要特征。无论是 TTS 
还是 vc 任务，均有一类基于 pitch 的方案活跃在 paper和工程中。

这篇文章会重点介绍一种基于采样点级别的pitch调整算法。

#### 基频提取器
什么是基频/f0/pitch

几个常用的提取基频库。具体原理可以参考对应文档。为了调用方便，针对前三种方法，我写成了对应python的封装。
- SPKT
- PyWorld
- Praat
- Kaldi

实际使用中，发现理想状态下，各个库效果差不多，但带底噪情况下，Praat和Spkt貌似有更好的鲁棒性。
在性能方面，pyworld明显速度快于其他算法。


#### Pitch Tuner 

何谓tuner？
常见的Pitch修改器一般都是seq级别的，例如sox/ffmpeg/Sptk。
某些特定的场景下，我们需要一个采样点级别的pitch修改器，可以更灵活的对音频中某一段的基频进行修改。

本质上来说，基于Psola或Wsola的算法，实现采样点级别的pitch修改并无额外难度，但对应好用的算法接口却并不多。
Praat是其中一个，它提供了比较灵活的pitch-tuner接口。可以实现下述类似的功能。

![](../images/posts/2021/test_pitch_by_point.png)


- - - 

https://github.com/Yablon/auorange

https://zhuanlan.zhihu.com/p/100287790