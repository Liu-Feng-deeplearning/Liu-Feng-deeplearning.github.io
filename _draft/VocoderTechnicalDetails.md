# Technical Details in Vocoder

记录声码器中使用的一些技巧（偏工程，非模型算法的改进）

## tts-VAD

在解码时，VAD对mel谱进行判断，如果为False，直接返回静音段（跳过实际的解码部分）

ttsVAD算法本质上是基于能量的计算方法，结构和实现原理类似librosa.trim_silence方法，但基于tts的使用场景和数据（往往更干净），进行了一定程度的优化和接口完善。

### why we need vad

- 避免生成结果中静音段的微小杂音。
- 提高推理速度，mel谱中往往包含相当比例的静音段。特别是在流式场景下，省掉首片解码中的静音段可以大幅降低开销。
- 模型不需要学习静音段的预测，提高模型鲁棒性，同时解决训练数据中静音段占比过大的缺陷。
    eg：zjl数据，直接训练(wrnn)600kstep发散。去掉静音后训练，不会出现发散。
- 在自回归类声码器中(wrnn)，缓解解码阶段超长音频带来的badcase。

Note: 本项目中所有profile结果，均未使用vad，即表示实际使用中的最坏情况。

### usage

#### 特征提取(训练)
- 对音频去静音预处理后再提特征

eg: vad前后音频对比
![](pic/vad.png)

#### 推理
- 对mel谱进行判断，只将非静音部分送入声码器 

### offlineVAD and onlineVAD
- offlineVAD使用单条音频的最大能量作为基准值，onlineVAD人为给定全局能量基准值。
- 可以通过tools.feature_and_signal.tool_vad中的方法获得全局能量基准值。
- 一般来说，offlineVAD更为精确，但两者差别不大。

 