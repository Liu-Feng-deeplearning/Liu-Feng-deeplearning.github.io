---
layout: post
title: Energy Based Vad 原理与应用
categories: 语音信号处理
description:  
keywords: 语音信号处理， VAD
---

简单的 vad 算法，有时可以却起到意想不到的效果。


VAD-Voice Active Detection. 几乎所有的语音任务中，都可以见到VAD的身影。
在编解码问题中，vad可应用于低码率编码静音段数据减少网络数据传输。
在ASR中，vad可以用来定位音频起始时间，大幅度提升系统rtf。
在语音增强中，vad用来获得谱减法需要的时间背景噪声信号。
TTS/VC中，vad同样在数据处理中发挥了重要作用。

DNN-base or 更复杂的vad算法大行其道，数据驱动下，vad可以做更多的事情，
例如区分人声与非人声（SAD/Speech Active Detection）等。
数据驱动方案固然可以得到更好的精度，但也要付出性能和延迟的代价，以及面临目录领域和数据领域匹配的问题。

这篇post会更多讲一下基于信号处理中VAD算法的实现和应用。
例如现代asr系统中，一个合理的选择就是前端的轻量级vad+后端足够鲁棒的asr模型，而不是一个较重的前端降噪系统。


### python implement


完整项目和更多细节可以参考[我的github项目](https://github.com/Liu-Feng-deeplearning/Energy_Based_VAD)

核心算法非常简单，通过计算每个窗口内能量并合阈值进行判断。代码如下：

```text
mse = librosa.feature.rms(y, frame_length=self._frame_length,
                              hop_length=self._hop_length) ** 2
signal_db = librosa.power_to_db(mse.squeeze(), ref=self._ref, top_db=None)

start, end = None, None
sil_endpoints = []
for x in range(-1, len(signal_db)):
  if x not in sil_index and (x + 1) in sil_index:
    start = int(librosa.frames_to_samples(x + 1, self._hop_length))
  if x in sil_index and x + 1 not in sil_index:
    end = int(librosa.frames_to_samples(x, self._hop_length))
    sil_endpoints.append((start, end))
sil_endpoints = [(_s, _e) for _s, _e in sil_endpoints if _e - _s > 1]
```

与此同时，对核心代码进行了上层封装，使之可以支持采样点和mel谱两种输入，同时实现了离线和流式两种封装接口以方便使用。
根据以往的项目经验，提供一些常用的经验参数可供使用。

### offlineVAD and onlineVAD

- offlineVAD 使用单条音频的最大能量作为基准值，onlineVAD 人为给定全局能量基准值。
- 可以通过tools.feature_and_signal.tool_vad中的方法获得全局能量基准值。
- 一般来说，offlineVAD 更为精确，但两者差别不大。

### vad for tts(tacotron)

基于 attention 的 tts 声学模型(e.g. Tacotron)，如果训练数据中，
句首和句末包括了过长的静音，可能会存在对齐无法收敛和解码停不下来的情况。
因此，librosa.trim_silence 是很多实现中多会采用的 trick。
如果是自己采集的数据，句中的过长静音也同样是需要考虑的问题。

### vad for vocoder

- 截取训练数据中过多的静音段，避免标签的不平衡（静音段占比过大），提高模型鲁棒性。
特别是针对 wrnn 等自回归类声码器（wrnn）。
- vad获得背景杂音，使用谱减法对音频进行降噪。（和解码中先生成静音bias，在用实际生成语音减掉bias类似。）
- 推理过程中使用基于频谱的vad算法，提高推理速度。特别是在流式场景下，省掉首片解码中的静音段可以大幅降低开销。
- 在自回归类声码器中(wrnn)，根据vad结果，重置gru/lstm的memory，缓解解码阶段超长音频带来的影响。


### vad for vc

与asr类似，使用 vad 对输入音频进行预处理。vad切分，可以避免过长的音频，减轻转换模型的负担。
同时，使用 vad 过滤掉静音段（以及含有微小噪声的非语音段），可以达到降噪并提高 rtf 的效果。一举两得。


