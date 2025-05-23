---
title: "ACC-UNet: A Completely Convolutional UNet model for the 2020s (MICCAI2023)"
date: 2023-11-12T16:32:06+08:00
lastmod: 2023-11-12T16:32:06+08:00
author: ["SwimmingLiu"]

categories:
- 📝 Paper Notes

tags:
- Unet

keywords:
- Unet
- ACC-Unet
- MICCAI2023

description: "" # 文章描述，与搜索优化相关
summary: "" # 文章简单描述，会展示在主页
weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: ""
draft: false # 是否为草稿
comments: true
showToc: true # 显示目录
TocOpen: false # 自动展开目录
autonumbering: true # 目录自动编号
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
searchHidden: false # 该页面可以被搜索到
showbreadcrumbs: true #顶部显示当前路径
mermaid: true
cover:
    image: ""
    caption: ""
    alt: ""
    relative: false
---
# ACC-UNet: A Completely Convolutional UNet model for the 2020s (MICCAI2023)
## 1. Abstract

由于ViT （Vision Transformer）的引入，UNet和Transformer融合已成为大趋势。最近，又有很多研究人员开始重新思考卷积模型，比如将ConvNext嵌入到ResNet，能够达到Swin Transformer的水平。受此启发，作者提出了一个纯粹的卷积UNET模型 （ACC-UNet），并且超越基于Transfomer的模型(如Swin-UNET或UCTransNet)。
作者研究了基于Transfomer的UNET模型优点：长范围依赖关系和跨级别跳过连接。
ACC-UNet结合了**卷积神经网络（ConvNets）的内在归纳偏差**和**Transformer的设计决策**
**卷积神经网络（ConvNets）的内在归纳偏差**：卷积神经网络具有天生的归纳偏差，这意味着它们在处理图像等数据时具有一些固有的假设和特点。例如，卷积神经网络擅长处理局部特征、平移不变性等，这些特点使它们在图像处理任务中表现出色。
**Transformer的设计决策**：Transformer是一种不同的神经网络架构，它采用了一些独特的设计决策，例如自注意力机制和位置编码等。这些设计决策使得Transformer在处理长距离依赖性、全局关系等方面表现出色，适合处理序列数据和具有远程依赖的任务。
ACC-UNet 在 5 个不同的医学图像分割基准上进行了评估，并且始终优于卷积网络、Transfomer及其混合网络。

## 2.Introduction

> 语义分割是计算机辅助医学图像分析的重要组成部分，可识别并突出显示各种诊断任务中感兴趣的区域。然而，由于涉及图像模态和采集以及病理和生物变化的各种因素，这通常变得复杂[18]。深度学习在这一领域的应用无疑在这方面受益匪浅。最值得注意的是，自推出以来，UNet 模型 [19] 在医学图像分割方面表现出了惊人的功效。结果，UNet 及其衍生品已成为事实上的标准[25]。

学习一下这里的背景描述

> 原始的 UNet 模型包含对称的编码器-解码器架构（图 1a）并采用跳跃连接，这为解码器提供了在编码器的池化操作期间可能丢失的空间信息。尽管通过简单串联的信息传播提高了性能，但编码器-解码器特征图之间可能存在语义差距。这导致了第二类 UNet 的发展（图 1b）。 U-Net++ [26] 利用密集连接，而 MultiResUNet [11] 在跳过连接上添加了额外的卷积块作为潜在的补救措施。到目前为止，UNet 的历史上所有创新都是使用 CNN 进行的。然而，2020 年的十年给计算机视觉领域带来了根本性的变化。 CNN 在视觉领域的长期主导地位被视觉转换器打破了 [7]。 Swin Transformers [15] 进一步针对一般视觉应用调整了变压器。因此，UNet 模型开始采用 Transformer [5]。 Swin-Unet [9] 用 Swin Transformer 块取代了卷积块，从而开创了一类新的模型（图 1c）。尽管如此，CNN 在图像分割方面仍然具有各种优点，导致了融合这两者的发展[2]。这种混合类 UNet 模型（图 1d）在编码器-解码器中采用卷积块，并沿跳跃连接使用变换器层。 UCTransNet [22]和MCTrans[24]是此类的两个代表性模型。最后，还尝试开发全变压器 UNet 架构（图 1e），例如，SMESwin Unet [27] 在编码器-解码器块和跳跃连接中都使用变压器。

从 UNet出发，然后逐步介绍他的变体（UNet++等）。随后介绍Transformer和UNet的各种结合体，为后续对比实验做铺垫。最后结合一张发展图，简明扼要描述UNet的创新历程。

> 最近，鉴于 Transformer 带来的进步，研究开始重新发现 CNN 的潜力。这方面的开创性工作是“A ConvNet for the 2020s”[16]，它探讨了 Transformer 引入的各种想法及其在卷积网络中的适用性。通过逐渐融合训练协议和微观-宏观设计选择的思想，这项工作使 ResNet 模型的性能优于 Swin Transformer 模型。
>
> 在本文中，我们在 UNet 模型的背景下提出了同样的问题。我们研究仅基于卷积的 UNet 模型是否可以与基于 Transformer 的 UNet 竞争。在此过程中，我们从 Transformer 架构中获得动力并开发了纯卷积 UNet 模型。我们提出了一种基于补丁的上下文聚合，与基于窗口的自注意力相反。此外，我们通过融合来自多个级别编码器的特征图来创新跳跃连接。对 5 个基准数据集的广泛实验表明，我们提出的修改有可能改进 UNet 模型。

通过介绍CONvNet的重要性，引入本文重点 （纯卷积模块的UNet）。

![image-20231024152252999](https://oss.swimmingliu.cn/nXfsu.png)

## 3. Method

### 3.1 A high-level view of transformers in UNet

> Transformers 显然在两个不同方面改进了 UNet 模型:
>
> 1.**利用自注意力的远程依赖性**: Transformer 可以通过使用（窗口式）自注意力，从更大的上下文视图中计算特征。此外，他们还通过采用反向瓶颈（即增加 MLP 层中的神经元）来提高表达能力。此外，它们包含快捷连接，这有助于学习。
> 2.**通过通道注意力的自适应多级特征组合**: 基于 Transformer 的 UNet 使用通道注意力自适应地融合来自多个编码器级别的特征图。与受当前级别信息限制的简单跳跃连接相比，由于来自不同级别的各种感兴趣区域的组合，这会生成丰富的特征。

主要就两个点：

1. 自主注意力在UNet的编码和解码阶段都能够快速、准确地根据上下文信息提取特征。

2. 自注意力机制可以用来连接encoder阶段不同stage的特征图。

猜想：把新出的自注意力机制加在ACC-Unet是不是也会有提升呢？

### 3.2 Hierarchical Aggregation of Neighborhood Context (HANC)

> 我们首先探索引入远程依赖性并提高卷积块的表达能力的可能性。我们仅使用逐点和深度卷积来降低计算复杂度。为了增加表达能力，我们建议在卷积块中包含反向瓶颈[16]，这可以通过使用逐点卷积将通道数从 cin 增加到 cinv = cin * inv_f ctr 来实现。由于这些额外的通道会增加模型复杂度，因此我们使用3×3深度卷积来补偿。输入特征图 xin ∈ R cin,n,m 被转换为 x1 ∈ R cinv,n,m （图 2b）
>
> ![image-20231024154231200](https://oss.swimmingliu.cn/nXmil.png)
>
> 接下来，我们希望在卷积块中模拟自注意力，其核心是将一个像素与其邻域中的其他像素进行比较[15]。通过将像素值与其邻域的平均值和最大值进行比较可以简化这种比较。因此，我们可以通过附加相邻像素特征的平均值和最大值来提供邻域比较的近似概念。因此，连续逐点卷积可以考虑这些并捕获对比视图。
>
> 由于分层分析对图像有益[23]，我们不是在单个大窗口中计算这种聚合，而是在多个级别中分层计算，例如 2 × 2, 2 ^2 × 2^ 2 , · · · , 2^(k− 1) × 2^(k−1) 个补丁。当 k = 1 时，这将是普通的卷积运算，但是当我们增加 k 的值时，将提供更多的上下文信息，从而绕过对更大卷积核的需要。因此，我们提出的分层邻域上下文聚合通过上下文信息丰富了特征图 x1 ∈ R cinv,n,m 作为 x2 ∈ R cinv*(2k−1),n,m （图 2b），其中 ||对应于沿通道维度的串联。
>
> ![image-20231024154748351](https://oss.swimmingliu.cn/nXWPd.png)
>
> 下面与Transfomer类似，我们在卷积块中包含一个快捷连接，以实现更好的梯度传播。因此，我们执行另一个逐点卷积以减少 cin 的通道数并与输入特征图相加。因此，x2 ∈ R cinv*(2k−1),n,m 变为 x3 ∈ R cin,n,m （图 2b）
>
> ![image-20231024155001509](https://oss.swimmingliu.cn/nXkWK.png)
>
> 最后，我们使用逐点卷积将滤波器的数量更改为cout作为输出
>
> ![image-20231024155250940](https://oss.swimmingliu.cn/nX0g2.png)

"inverted bottlenecks"（反向瓶颈） ：将Bottleneck放在卷积块的开始，而不是结束。它先使用1x1卷积来减少通道数，然后才是较大的卷积层。

![image-20231024153903701](https://oss.swimmingliu.cn/nXBGj.png)

### 3.3 Multi Level Feature Compilation (MLFC)

> 接下来，我们研究多级特征组合的可行性，这是使用基于 Transformer 的 UNet 的另一个优点。
>
> 基于 Transformer 的跳跃连接已经证明了所有编码器级别的有效特征融合以及各个解码器从编译的特征图中进行的适当过滤[24,22,27]。这是通过连接不同级别的投影令牌来执行的[22]。按照这种方法，我们调整从不同编码器级别获得的卷积特征图的大小，以使它们均衡并连接它们。这为我们提供了跨不同语义级别的特征图的概述。我们应用逐点卷积运算来总结这种表示并与相应的编码器特征图合并。整体信息和个体信息的融合通过另一个卷积传递，我们假设它用来自其他级别特征的信息丰富了当前级别特征。
>
> 对于来自 4 个不同级别的特征 x1、x2、x3、x4，特征图可以通过多级信息来丰富（图 2d）
>
> ![image-20231024155452522](https://oss.swimmingliu.cn/nXhpI.png)
>
> 这里， resizei(xj ) 是将 xj 的大小调整为 xi 的大小且 ctot = c1 + c2 + c3 + c4 的操作。此操作针对所有不同级别单独完成。因此，我们提出了另一个名为多级特征编译（MLFC）的新颖块，它聚合来自多个编码器级别的信息并丰富各个编码器特征图。该块如图 2d 所示。

总结： 把各stage的特征联合起来进行逐点卷积，然后各stage得到新的特征信息

![image-20231024155612777](https://oss.swimmingliu.cn/nXs7V.png)

### 3.4 ACC-UNet

> 因此，我们提出全卷积ACC-UNet（图2a）。我们从普通的 UNet 模型开始，并将滤波器的数量减少了一半。然后，我们用我们提出的 HANC 块替换了编码器和解码器中的卷积块。我们考虑 inv_f ctr = 3，而不是第 3 级的最后一个解码器块 (inv_f ctr = 34)，以模拟 Swin Transformer 第 3 阶段的扩展。 k = 3，最多考虑 4 × 4 个补丁，被选择用于除瓶颈级别 (k = 1) 及其旁边的级别 (k = 2) 之外的所有级别。接下来，我们通过使用残差块（图 2c）来修改跳跃连接以减少语义间隙 [11] 并堆叠 3 个 MLFC 块。所有卷积层均经过批量归一化 [12]，由 Leaky-RELU [17] 激活，并通过挤压和激励 [10] 重新校准。
>
> 总而言之，在 UNet 模型中，我们用我们提出的 HANC 块替换了经典的卷积块，该 HANC 块执行近似版本的自注意力，并修改了与 MLFC 块的跳跃连接，MLFC 块考虑来自不同编码器级别的特征图。所提出的模型有 16.77 M 个参数，比普通 UNet 模型大约增加了 2M。

![image-20231024160103607](https://oss.swimmingliu.cn/nXx8J.png)

## 4.Experiments

### 4.1 Datasets

> 为了评估 ACC-UNet，我们在 5 个跨不同任务和模式的公共数据集上进行了实验。我们使用 ISIC-2018 [6,21]（皮肤镜检查，2594 张图像）、BUSI [3]（乳腺超声，使用类似于 [13] 的 437 张良性图像和 210 张恶性图像）、CVC-ClinicDB [4]（结肠镜检查，612 张图像） ）、COVID [1]（肺炎病灶分割，100 张图像）和 GlaS [20]（腺体分割，85 张训练图像和 80 张测试图像）。所有图像和掩模的大小都调整为224×224。对于GlaS数据集，我们将原始测试分割作为测试数据，对于其他数据集，我们随机选择20％的图像作为测试数据。
>
> 剩余的 60% 和 20% 图像用于训练和验证，并使用不同的随机改组重复实验 3 次。

ISIC-2018、BUSI、 CVC-ClinicDB、COVID、GlaS 五个数据集

### 4.2 Comparison Experiments

![image-20231024160330982](https://oss.swimmingliu.cn/nXbAW.png)

### 4.3 Ablation Study

![image-20231024160403489](https://oss.swimmingliu.cn/nXosv.png)

### 4.4  Qualitative Results

![image-20231024160458712](https://oss.swimmingliu.cn/nXSie.png)