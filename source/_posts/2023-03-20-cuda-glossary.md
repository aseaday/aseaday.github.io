---
title: "CUDA 超链接词汇表"
classes: wide
date: 2022-06-18
categories:
  - blog
tags:
  - developer
---

## 硬件

## CUDA

CUDA 是 Compute Unified Device Architecture (统一计算设备架构)的缩写。根据上下文，"CUDA"可以指代多个不同的概念:一个高级设备架构、一个基于该设计的并行编程模型，或是一个扩展了 C 等高级语言以添加该编程模型的软件平台。

CUDA 的愿景在 [Lindholm 等人 2008 年的白皮书](https://www.cs.cmu.edu/afs/cs/academic/class/15869-f11/www/readings/lindholm08_tesla.pdf)中有详细阐述。我强烈推荐这篇论文，它是 NVIDIA 文档中许多说法、图表，甚至特定措辞的原始来源。


在这里，主要关注 CUDA 的设备架构部分。"CUDA"的核心特征是相对于之前的 GPU 架构而言的简单性。

在 GeForce 8800 和由此衍生的 Tesla 数据中心 GPU 之前，NVIDIA 的 GPU 采用了复杂的管线着色器架构，将软件着色器阶段映射到异构的专用硬件单元上。这种架构对软件和硬件双方都带来了挑战:它要求软件工程师将程序映射到固定管线上，并迫使硬件工程师去猜测管线各步骤之间的负载��例。

![固定管线设备架构(G71)示意图。注意用于处理片段和顶点着色的独立处理器组](/images/light-fixed-pipeline-g71.svg)

具有统一架构的 GPU 设备要简单得多：硬件单元完全统一，每个单元都能执行广泛的计算任务。这些单元被称为流式多处理器（Streaming Multiprocessors，简称 SMs），其主要子组件是 CUDA 核心（CUDA Cores）和（在最新的 GPU 中）张量核心（Tensor Cores）。


![统一计算设备架构(G80)的示意图。没有不同类型的处理器, 所有有意义的计算都发生在图中央完全相同的流式多处理器中，这些处理器接收顶点、几何和像素线程的指令](/light-cuda-g80.svg)

关于 CUDA 硬件架构的历史和设计，Fabien Sanglard 的[这篇博文](https://fabiensanglard.net/cuda/)提供了一个易于理解的入门介绍。该博文引用了一些高质量的参考资料，如 NVIDIA 的 Fermi 计算架构白皮书。 另外值得一提的事，Lindholm 等人在 2008 年发表的介绍[Tesla 架构的白皮书](https://images.nvidia.com/content/pdf/tesla/whitepaper/pascal-architecture-whitepaper.pdf)。NVIDIA 的 Tesla P100 白皮书虽然不那么学术化，但记录了一些对当今大规模神经网络工作负载至关重要的特性，如 NVLink 和片上高带宽内存。