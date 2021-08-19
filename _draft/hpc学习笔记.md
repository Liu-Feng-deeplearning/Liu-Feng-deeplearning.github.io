# Hpc 学习笔记



--------------------------------------------------------------------------------
CS267 笔记
https://sites.google.com/lbl.gov/cs267-spr2021

Lec1
Overiew

Lec2


Lec2
寄存器
cache-最快 cache hit/miss
不同级别的cache
SIMD
FMA 

Example: gemm
- matrix stage
- blocking or tiling for resgister // 降低读取数据的消耗

Example: conv
- 卷积核读到临时变量中，避免preload消耗
filter[0] saved at slow memory
but 
float f0 = filter[0] // saved at fast memory(register)

--------------------------------------------------------------------------------
Lec3 
recurrent maltiply
strassen  method

roofline model: 
计算效果既和算力大小有关，也和访存带宽有关。
神经网络模型推理速度，既和参数量大小有关，也和flops有关。两者共同决定了瓶颈。




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