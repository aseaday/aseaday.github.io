---
title: "CUDA HyerIntro"
classes: wide
date: 2023-03-20
categories:
  - blog
tags:
  - developer
---
## 硬件

### CUDA

CUDA 是 Compute Unified Device Architecture (统一计算设备架构)的缩写。根据上下文，"CUDA"可以指代多个不同的概念:一个高级设备架构、一个基于该设计的并行编程模型，或是一个扩展了 C 等高级语言以添加该编程模型的软件平台。

CUDA 的愿景在 [Lindholm 等人 2008 年的白皮书](https://www.cs.cmu.edu/afs/cs/academic/class/15869-f11/www/readings/lindholm08_tesla.pdf)中有详细阐述。我强烈推荐这篇论文，它是 NVIDIA 文档中许多说法、图表，甚至特定措辞的原始来源。


在这里，主要关注 CUDA 的设备架构部分。"CUDA"的核心特征是相对于之前的 GPU 架构而言的简单性。

在 GeForce 8800 和由此衍生的 Tesla 数据中心 GPU 之前，NVIDIA 的 GPU 采用了复杂的管线着色器架构，将软件着色器阶段映射到异构的专用硬件单元上。这种架构对软件和硬件双方都带来了挑战:它要求软件工程师将程序映射到固定管线上，并迫使硬件工程师去猜测管线各步骤之间的负载比例。

![固定管线设备架构(G71)示意图。注意用于处理片段和顶点着色的独立处理器组](/images/light-fixed-pipeline-g71.svg "固定管线设备架构(G71)示意图。注意用于处理片段和顶点着色的独立处理器组")

具有统一架构的 GPU 设备要简单得多：硬件单元完全统一，每个单元都能执行广泛的计算任务。这些单元被称为流式多处理器（Streaming Multiprocessors，简称 SMs），其主要子组件是 CUDA 核心（CUDA Cores）和（在最新的 GPU 中）张量核心（Tensor Cores）。


![统一计算设备架构(G80)的示意图。没有不同类型的处理器, 所有有意义的计算都发生在图中央完全相同的流式多处理器中，这些处理器接收顶点、几何和像素线程的指令](/images/light-cuda-g80.svg "统一计算设备架构(G80)的示意图。没有不同类型的处理器, 所有有意义的计算都发生在图中央完全相同的流式多处理器中，这些处理器接收顶点、几何和像素线程的指令")

关于 CUDA 硬件架构的历史和设计，Fabien Sanglard 的[这篇博文](https://fabiensanglard.net/cuda/)提供了一个易于理解的入门介绍。该博文引用了一些高质量的参考资料，如 NVIDIA 的 Fermi 计算架构白皮书。 另外值得一提的事，Lindholm 等人在 2008 年发表的介绍[Tesla 架构的白皮书](https://images.nvidia.com/content/pdf/tesla/whitepaper/pascal-architecture-whitepaper.pdf)。NVIDIA 的 Tesla P100 白皮书虽然不那么学术化，但记录了一些对当今大规模神经网络工作负载至关重要的特性，如 NVLink 和片上高带宽内存。

### SM

当我们编程 GPU 时,我们会产生一系列指令供其流式多处理器执行。

![展示了 H100 GPU 中流式多处理器的内部架构。GPU 核心以绿色显示,其他计算单元为褐红色,调度单元为橙色,内存为蓝色。此图改编自 NVIDIA 的 H100 白皮书。](https://object.hejubian.me/2024/12/120ec823078b7d755de28be1d6178b94.svg "展示了 H100 GPU 中流式多处理器的内部架构。GPU 核心以绿色显示,其他计算单元为褐红色,调度单元为橙色,内存为蓝色。此图改编自 NVIDIA 的 H100 白皮书。")


NVIDIA GPU 的流式多处理器(SM)大致类似于 CPU 的核心。也就是说,SM 既执行计算,又在寄存器中存储可用于计算的状态,并配有相关的缓存。与 CPU 核心相比,GPU 的 SM 是相对简单且性能较弱的处理器。SM 中的执行在指令内部是流水线化的(如同 1990 年代以来的几乎所有 CPU),但没有推测执行或指令指针预测功能(这与当今所有高性能 CPU 不同)。

然而,GPU 的 SM 可以并行执行更多线程。

作为对比:AMD EPYC 9965 CPU 最大功耗为 500W,有 192 个核心,每个核心同时最多可以执行两个线程的指令,总共可并行 384 个线程,每个线程大约消耗 1.25W 功率。

H100 SXM GPU 最大功耗为 700W,有 132 个 SM,每个 SM 有 4 个线程束调度器,每个调度器每个时钟周期可以并行地向 32 个线程(即一个线程束)发出指令,总共可以并行超过 16,000 个线程,每个线程仅消耗约 0.05W。需要注意的是,这是真正的并行:16,000 个线程中的每一个都可以在每个时钟周期取得进展。

GPU 的 SM 还支持大量并发线程 -- 即那些指令交错执行的线程。
H100 上的单个 SM 可以并发执行多达 2048 个线程,这些线程被分成 64 个线程组,每组 32 个线程。有了 132 个 SM,总共可以支持超过 250,000 个并发线程。

CPU 也可以并发运行许多线程。但在 GPU 上,线程束之间的切换发生在单个时钟周期内(比 CPU 上的上下文切换快 1000 多倍),这同样得益于 SM 的线程束调度器。大量可用的线程束和快速的线程束切换有助于隐藏内存读取、线程同步或其他耗时指令造成的延迟,确保计算资源(特别是 CUDA 核心和张量核心)得到充分利用。

这种延迟隐藏是 GPU 优势的关键。CPU 试图通过维护大型的硬件管理缓存和复杂的指令预测来对终端用户和程序员隐藏延迟。这些额外的硬件限制了 CPU 可以分配给计算的硅面积、功率和散热预算的比例。

![GPU 相比 CPU 将更多的面积用于计算(绿色),而用于控制和缓存(橙色和蓝色)的面积较少。](https://object.hejubian.me/2024/12/0b7afbf46e508a19c25e53bc8de5737c.svg "GPU 相比 CPU 将更多的面积用于计算(绿色),而用于控制和缓存(橙色和蓝色)的面积较少。")

对于神经网络推理或顺序数据库扫描等程序或功能,程序员相对容易表达缓存的行为(例如,存储每个输入矩阵的一个块,并将其保持在缓存中足够长的时间以计算相关输出),最终结果是获得了更高的吞吐量。

### Core

GPU 核心类型包括 CUDA core 和 Tensor core。

虽然 GPU 核心在执行实际计算这一点上与 CPU 核心类似，但这种类比可能会产生误导。实际上，流式多处理器（SM）才更接近于 CPU 核心的等价物。

### Warp Scheduler

流式多处理器（SM）的线程束调度器负责决定执行哪组线程。

这些线程组被称为[线程束](#warps), 每个时钟周期切换一次, 大约需要一个纳秒。

CPU 线程上下文切换需要保存一个线程的上下文并恢复另一个线程的上下文, 需要数百到数千个时钟周期(比一个纳秒更接近一个微秒), 因为需要保存和恢复线程的上下文。此外, CPU 上的上下文切换会导致局部性降低, 进一步降低性能, 增加缓存未命中率。

因为每个线程都有自己的私有寄存器, 从 SM 的 [Register File](#register-file) 中分配, GPU 上的线程束切换不需要任何数据移动来保存或恢复上下文。GPU 上的线程束切换对缓存命中率影响较小, 因为 L1 缓存可以完全由程序员管理, 并且在线程束之间共享(见[协作线程数组](#cooperative-thread-array))。 

### CUDA Core & Tensor Core
| 特性 | CUDA Core | Tensor Core |
| --- | --- | --- |
| 主要用途 | 通用计算，单精度和双精度浮点运算 | 矩阵运算，深度学习加速 |
| 指令类型 | 单一标量运算 (如加、乘) | 矩阵乘加运算 (如 D = A×B + C) |
| 数据操作粒度 | 单个数值运算 | 整个矩阵块运算 |
| 每SM数量 | 128个 (以H100为例) | 4个 (以H100为例) |
| 精度支持 | FP32, FP64, INT32 | FP32, FP16, BF16, INT8, FP8 |
| 计算效率 | 标量运算高效 | 矩阵运算高效 (比CUDA Core快8-16倍) |
| 应用场景 | 通用并行计算、图形渲染 | 深度学习训练和推理、大规模矩阵计算 |

#### Tensor Core

Tensor Core 用来矩阵运算。

例如，**mma** PTX 指令可以计算矩阵运算 $D = A \times B + C$，其中 $A、B、C、D$ 都是矩阵。在单次指令获取中处理更多数据可以显著降低功耗。

Tensor Core 比 CUDA Core 更大且数量更少。以 H100 SXM5 为例，每个流式多处理器只有 4 个 Tensor Core，相比之下 CUDA Core 有数百个。

Tensor Core 首次在 V100 GPU 中引入，这代表了 NVIDIA GPU 在大型神经网络工作负载适用性方面的重大改进。

#### CUDA Core

CUDA Core 进行标量运算。

与 CPU 核心不同，发送给 CUDA 核心的指令通常不是独立调度的。相反，一组核心会由线程束调度器同时接收相同的指令，但这些指令会作用于不同的寄存器。通常情况下，这些组的大小为 32（即一个线程束的大小），但在现代 GPU 中，组的大小可以小到只有一个线程，不过这会影响性能。

CUDA Core 有一点特殊，在不同的流式多处理器架构中，CUDA core 可以由不同的单元组成 -- 32 位整数和 32 位和 64 位浮点运算单元的不同组合。

例如，H100 白皮书指出，H100 GPU 的流式多处理器（SM）每个有 128 个“FP32 CUDA 核心”，这准确地计数了 32 位浮点运算单元的数量，但却是 32 位整数或 64 位浮点运算单元数量的两倍。为了估算性能，最好直接查看给定操作的硬件单元数量。

### Register File

寄存器文件是通常位于芯片上的存储位置，提供对最近读取或写入主内存（RAM）的数据的快速访问。此外，L1 在活动数据量超过 SM 寄存器文件的存储能力时充当溢出区域，这种情况称为“寄存器溢出”。在 L1 中，缓存行和溢出的寄存器被组织成银行，就像在寄存器文件中一样。

## 设备软件

### Warps

线程束（Warp）是一组被一起调度并并行执行的线程。一个线程束中的所有线程都被调度到单个流式多处理器（SM）上。一个 SM 通常会同时执行多个线程束，至少会执行来自同一个协作线程数组（也称为线程块）的所有线程束。

线程束是 GPU 上的典型执行单位。在正常执行过程中，一个线程束中的所有线程并行执行相同的指令 —— 这就是所谓的"单指令多线程"（SIMT）模型。线程束的大小在技术上是一个与机器相关的常量，但在实践中是 32。

当一个线程束被发出指令时，结果通常不会在单个时钟周期内可用，因此依赖的指令无法立即发出。这种情况在访问全局内存（通常需要访问芯片外部）时最为明显，某些算术指令也是如此（具体指令的每时钟周期结果，请参见 CUDA C++ 编程指南中的"性能指南"表）。

当多个线程束被调度到单个 SM 上时，线程束调度器不会等待一个线程束返回结果，而是会选择另一个线程束执行。这种"延迟隐藏"机制使 GPU 能够实现高吞吐量，并确保执行期间所有核心都有可用的工作。因此，最大化每个 SM 上调度的线程束数量通常是有益的，这样可以确保 SM 始终有准备就绪的线程束可以运行。

线程束实际上不是 CUDA 编程模型的线程组层次结构的一部分。相反，它们是该模型在 NVIDIA GPU 上实现的实现细节。从这个角度来看，它们有点类似于 CPU 中的缓存行：这是一个你无法直接控制且不需要考虑程序正确性的硬件特性，但对于实现最大性能来说却很重要。

根据 Lindholm 等人 2008 年的说法，线程束（Warp）的命名参考了编织（Weaving）这一"最早的并行线程技术"。在其他 GPU 编程模型中，与线程束等价的概念包括 WebGPU 中的子组（subgroups）、DirectX 中的波（waves）和 Metal 中的 SIMD 组（simdgroups）。





