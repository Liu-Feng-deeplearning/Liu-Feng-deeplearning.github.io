# Hpc 学习笔记



CS267 笔记
https://sites.google.com/lbl.gov/cs267-spr2021

## Lec1～4

**Lec1**

Overiew

**Lec2**

- 寄存器简介，cache-hit and cache-miss
- 不同级别的cache速度差异
- SMP/DMP/SIMD 简介（貌似也是贯穿整个课程的）
- FMA 介绍（一次乘法+一次加分）为什么要单独提出这个概念？？ 

Example: gemm/gemv
- 提出概念 计算次数 and 访问内存次数
- 重点关系 matrix storage
- blocking or tiling for resgister 矩阵分片，用以降低读取数据的消耗

Example: conv
- 卷积核读到临时变量中，避免preload消耗. 如下，使用临时变量会更快。
```
res += filter[0] * signal[0] // using filter value at slow memory

float f0 = filter[0]
res += f0 * signal[0] // using temp value at fast memory(register)
```


**Lec3** 

- recurrent multiply method
- strassen  method (算法层面的优化)

roofline model介绍: 
- 计算效果既和算力大小有关，也和访存带宽有关。
- 神经网络模型推理速度，既和参数量大小有关，也和flops有关。两者共同决定了瓶颈。

<div style="text-align: center"><img src="https://github.com/Liu-Feng-deeplearning/Liu-Feng-deeplearning.github.io/blob/master/images/posts/2021/2021-08-21-hpc-roofline.png?raw=true" width="500" /></div>





Lec4

pthread 及各种锁，mutex的用法
https://zhuanlan.zhihu.com/p/112297714

OpenMp 简介 and usage

SMP-share memory program 用法

并行不起作用的根源：
fase sharing:
L1 cache line 有大小（64bytes）。在同一个cache line的不同变量不是独立的，系统对整个cacheline有标签。
所以必行计算时，如果不同的变量位于同一个cache line内，会发生数据竞争。
一个帮助理解的文章：https://zhuanlan.zhihu.com/p/85984250

OpenMp 需要在实践中练习。
一些例子对比openmp中各种trick的用法。


--------------------------------------------------------------------------------
Lec5

一些例子。。。
Lec6
n-body 问题

PDE问题

Jim Demmel 好像更偏传统工程学

--------------------------------------------------------------------------------
Lec7

Cpu和gpu架构上的区别
gpu的基本简介