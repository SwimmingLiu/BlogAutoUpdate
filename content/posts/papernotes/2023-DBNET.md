---
title: "A Dual-Branch Framework with Prior Knowledge for Precise Segmentation of Lung Nodules in Challenging CT Scans"
date: 2024-03-03T15:37:07+08:00
lastmod: 2024-03-03T15:37:07+08:00
author: ["SwimmingLiu"]

categories:
- 📝 Paper Notes

tags:
- Unet

keywords:
- DBNET

description: "" # 文章描述，与搜索优化相关
summary: "" # 文章简单描述，会展示在主页
weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: ""
math: true
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

## Abstract

> 肺癌是全球最致命的癌症之一，早期诊断对于患者的生存至关重要。**肺结节**是**早期肺癌的主要表现**，通常通过 **CT 扫描**进行评估。如今，计算机辅助诊断系统被广泛用于辅助医生进行疾病诊断。**肺结节的准确分割**受到**内部异质性和外部数据**因素的影响。为了克服**结节的细微、混合、粘附型、良性和不确定类别**的分割挑战，提出了一种新的**混合手动特征网络**，可**增强灵敏度和准确性**。该方法通过**双分支网络框架和多维融合模块**集成**特征信息**。通过使用**多个数据源和不同数据质量**进行训练和验证，我们的方法在 **LUNA16**、**多厚度切片图像数据集 (Multi-thickness Slice Image dataset)**、**LIDC** 和 **UniToChest** 上表现出领先的性能，Dice 相似系数达到 86.89%、75.72%、84.12% 和 80.74分别超过了当前大多数肺结节分割方法。我们的方法进一步提高了肺结节分割任务的**准确性、可靠性和稳定性**，即使是在具有挑战性的 CT 扫描中也是如此。本研究中使用的代码发布在 GitHub 上，可通过以下 URL (https://github.com/BITEWKRER/DBNet) 获取。

## Introduction

> 肺癌是全球癌症相关死亡的主要原因[1]。仅在美国，预计 2023 年将有 127,070 人死于肺癌，占所有癌症死亡的 21% [2]。不幸的是，超过 50% 的肺癌病例发生在发展中国家或不发达国家，与发达国家相比，这些国家的医疗资源有限[3]。
>
> 为了增加生存机会，早期诊断和治疗肺癌仍然至关重要。在中国，研究表明，**小于1厘米的I期肺癌的5年生存率为92%**。然而，**晚期肺癌的5年生存率低得多**，仅为**7.0%**[4]。**利用计算机断层扫描 (CT) 进行肺癌筛查已显示出可大幅降低死亡率的潜力** [5]、[6]。低剂量CT是目前肺癌筛查最常用的方法。此外，移动CT的引入有助于解决欠发达国家和偏远地区缺乏CT扫描仪的问题[6]。由于可能没有明显的症状，检测早期肺癌的存在可能会带来重大挑战。

**这种医学背景数据可以直接借鉴，Chatgpt润色改写就完事儿**

> 在 **CT 图像上识别肺结节提供了疾病的关键指标** [1], [3]。这些结节代表**圆形异常**，其**大小各异**，直径范围为 **3 至 30 毫米** [7]。为了进一步研究肺结节，美国国家癌症研究所组装了“肺部图像数据库联盟和图像数据库资源计划（LIDC）”数据集[8]。
>
> **欠发达地区设备不足、人员不足，导致医生的诊断和治疗时间有限**[9]。在这种情况下，**医生的工作量很大、重复且耗时**[10]、[5]。此外，由于与CT切片相比，**肺部结节性病变**占据相对**较小的面积**，**长时间和密集的CT筛查**可能会导致**漏检小的、细微的或 GGO (肺磨玻璃结节) **[3]，[6]。为了解决这些问题，计算机辅助诊断系统（CAD）出现并得到了快速发展，特别是随着基于深度学习技术的诊断方法的进步。 **CAD系统大大减轻了医生的工作量，最大限度地降低了未发现结节的风险，并提高了肺结节诊断的效率和可靠性**。然而，当前用于肺结节分割的 CAD 系统仍然面临一些挑战。

下面详细阐述了肺结节分割的几个现有挑战，可以从这些挑战入手

> 首先，**放射科医生标记的肺结节**包含**九个诊断特征**[11]，**异质性表型**阻碍了**肺结节分割的发展**。如图1所示，**实心结节（a，b）具有清晰的形状和边界**，而**微妙的GGO结节（e）具有低对比度和模糊的边界**[4]，使得网络很容易将**它们分类为背景区域**。**空洞（g）结节降低了网络分割的敏感性**，并且由于**背景和分割目标之间的极度不平衡，小结节很容易被遗漏**[12]。
>
> 由于周围多余的组织结构，**血管旁或胸膜旁（c、d、f）可能会导致网络分类错误**[13]。此外，**部分实性结节（h）比纯GGO更致密**，产生更**复杂的异质纹理，更容易发展成恶性结节**[14]。

![image-20240303102609243](https://oss.swimmingliu.cn/262f8e71-d931-11ee-b68c-c858c0c1debd)

> 其次，肺结节内部因素造成的分割困难在于**医生注释、层厚、数据来源和数据质量**。**数据质量差**或**不同医生的经验**可能会导致**不同的注释和注释者数量**。由**多名医生注释的病变区域**通常更**可靠**，减少了潜在的临床风险。**在资源有限的地区**，由于 **CT 扫描仪短缺和成像设备陈旧**，**CT 扫描质量差的情况很常见**。**较厚的切片**更有可能产生“**体积平均效应”和伪影**，使医生**难以达成一致的诊断**。即使使用**移动 CT 扫描仪**也可能**无法提供完整的诊断详细信息**。最后，目前**大多数肺结节分割方法**都是基于**2D图像**，但这些方法忽略了**空间关系**，因此提出一种有效的**3D肺结节分割模型**来**捕获肺结节的空间位置**、**纹理和其他详细信息变得越来越重要以避免误诊和漏诊**。

**Challenge**:

1. 异质性: 肺结节的形状多异 （实心结节、磨玻璃结节 (GGO) 、空洞结节、血管和胸膜旁边的结节）

2. 数据集缺陷：数据集质量不好、较厚的切片和医生的不同标注都可能影响最后的分割结果
3. 2D网络缺陷：忽略了CT信息中的**空间关系**

**背景知识补充**：在医学成像，特别是在使用计算机断层扫描（CT）进行诊断时，“体积平均效应”和“伪影”是两个可能影响图像质量和解读准确性的问题。

1. **体积平均效应**：这是一种由于扫描切片厚度较大而引起的现象。在一个较厚的切片中，多个不同密度的结构可能被平均到同一个体素（三维像素）里。这意味着高密度和低密度区域可能会混合在一起，从而降低了图像的对比度和分辨率。这种效应使得较小的结构，如小肺结节，更难以被准确识别和分割，因为它们可能在较厚的切片中“消失”或与周围组织混合。
2. **伪影**：伪影是指在成像过程中由各种原因产生的非实际存在于原始对象中的图像特征。在CT扫描中，伪影可能由患者移动、成像设备的限制、软件处理算法或扫描参数设置不当等因素引起。较厚的切片可能加剧这些伪影，因为每个切片覆盖更大的体积，增加了平均和重建过程中出现错误的机会。

这段话中提到，“较厚的切片更有可能产生‘体积平均效应’和伪影”，意味着在使用较厚切片进行CT扫描时，上述两个问题更容易发生，这会降低图像的质量和可解释性。结果，医生在解读这些图像时可能会遇到困难，因为图像的不清晰和伪影可能导致诊断不一致或误诊。这就强调了使用高分辨率、薄层切片扫描和高质量成像设备的重要性，以提高诊断的准确性和一致性。

> 针对上述肺结节的**异质性特征**，我们提出了一种**新的混合手动特征**，旨在提供**额外的边界**和**病灶信息**，显着**提高肺结节分割的灵敏度，减少漏诊的发生**，降低**像素点的可能性错误分类**。为了保证分割结果的可靠性，我们采用多位医师标注结果的**平均值**作为**GroundTruth**，并对单位医师标注的样本进行二次筛选。为了**确保模型在不同数据源和数据质量上的鲁棒性**，我们在**不同来源、质量和尺度的数据集上训练模型**，并在**不同数据源和切片厚度上进行验证**。为了**有效地整合主、辅分支的特征信息**，我们设计了**多维融合模块，并从空间和通道维度进行学习，以增强网络的表示和泛化能力**。最后，所提出的网络能够进行三维分割，有利于病变信息的全面获取。

下面是 Main Contribution

> （a）提出了一种新的**端到端双分支网络和融合模块**，有效完成**肺结节分割任务，简单易用，泛化性较好能力**。
>
> （b）提出了**混合手动特征**来同时**增强**肺结节**边界信息**和**对比度信息**。
>
> （c）我们的方法显示了分割各种类型的**肺结节的性能改进**，特别是在**细微的、混合纹理的、粘连的、良性的和不确定的结节**中。
>
> (d) 研究并总结了**不同层厚**和**尺寸**下模型的性能变化和模式。

## Related work

> 目前，肺部结节的分割方法有很多，可以分为**基于算法的传统分割方法**和**数据驱动的深度学习分割方法**。然而，**传统的肺结节分割方法存在明显的缺陷**，特别是当**肺结节的边缘变得模糊时**，导致**性能急剧下降**[15]。相比之下，**数据驱动的方法表现出更好的性能**[16]。

数据驱动算法 > 传统分割算法   ==> 调研数据驱动算法的论文，利用数据驱动来创新

> **多分支网络架构**引起了广泛的关注和研究。这些结构擅长集成**不同规模、视角和维度**的不**同特征**，从而提高性能。为了解决肺结节的**异质性**以及结节与其**周围环境的相似性的挑战**，Wang等人[17]引入了**中央聚焦卷积神经网络**。该网络从 **2D 和 3D CT 图像块**中**提取肺结节信息**，并通过**中央池化层保留中央位置**的基本细节。 Chen等人[18]引入了Fast Multi-crop Guided Attention，一种**基于残差的多尺度引导分割网络**来解决这个问题。该方法最初将 **2D 和 3D 图像输入单独的分支网络**，以从**多维相邻轴**捕获上下文特征信息。随后，采用**全局卷积层来感知和融合上下文特征**，同时通过**中央池化层增强对图像块的中心区域的关注**。 Wang等[19]提出了一种**提高不规则肺结节分割效率的方法**，同时保持**简单结节的准确分割**。他们利用**双分支框架来处理 CT 图像和边界梯度信息**，使用**密集注意力模块**来关注**关键结节特征**，并使用**多尺度选择性注意力模块**来连接**不同尺度的特征**。此外，他们引入了**边界上下文增强模块**来**合并和增强边缘相关的体素特征**，从而实现**简单型肺结节的准确分割**。 Xu等人[20]提出了一种用于**分割非典型结节的双编码融合网络**。他们首先使用金字塔上采样方法来**创建平滑的病变图像**，并减少 **CT 图像中高粒度像素的干扰**。然后，他们使用全局和局部分支对上采样的 **CT 图像块**和**原始图像块进行编码**，以**捕获全局和局部信息**。此外，他们采用了金字塔池化模块来增强本地分支的输出特征。最后，他们**融合并解码了来自双分支的特征信息**。 Wang等人[21]提出了一种**混合深度学习模型**（H-DL），用于**分割各种大小和形状的肺结节**。该模型结合了基于VGG19的浅层U-Net网络和基于密集连接的深层U-Net网络，增强了复杂肺结节分割的学习能力。与独立的 UNet 结构模型相比，将这两个模型集成到混合模型 H-DL 中证明了分割结果得到了改善。
>
> **基本块的设计对于肺结节分割**也具有重要意义。研究人员探索了多种设计概念来提高模型性能和效率。 Wang等人[22]设计了一个**基于空洞卷积的深度尺度感知模块**来**聚合上下文**。该模块将**具有不同扩张率（1、2和3）**的并行分支**嵌入到瓶颈层**中，以捕获更丰富的语义信息。 Agnes等人[23]提出了一种**多尺度全卷积3D UNet**模型，其中作者设计了一**个多尺度基本块**，使用 $3×3×3$ 和$5×5×5$ 卷积从各种尺度中提取特征信息。他们使用 **Maxout 激活函数优化多尺度特征信息**，抑制低贡献特征。 Chen等人[15]提出了一种用于分割GGO的注意力级联残差网络。该网络**通过残差结构和扩张的空间金字塔池模块**捕获肺结节的特征信息。在后处理阶段，应用**基于体素的条件随机场**来进一步细化分割结果。 Zhou等[3]介绍了**一种级联的2.5D肺结节检测和分割方法**。在分割网络中，作者将**卷积块注意力模块（CBAM）**[24]合并到编码器中，以增强网络的编码能力。在瓶颈层设计了不同的多尺度卷积扩张率，以实现结节区域的精细分割。
>
> 另一种方法是使用**对抗性生成网络**进行肺结节分割。训练**数据稀缺**和**类别不平衡**一直是影响肺结节分割性能的关键因素。**数据稀缺**会导致**过度拟合或模型收敛失败**，而**类别不平衡**则增加了**目标区域分割**的挑战。为了解决这些问题，Song等人[25]提出了一种基于**生成对抗网络**对**多种类型肺结节进行全自动分割的端到端架构**。该模型包括两个分支。**第一个分支用于潜在的肺结节分割和结节生成**。**第二个分支旨在减少第一个分支产生的潜在假阳性结节**。此外，Tyagi 等人[26]引入了一种**基于条件生成网络**的分割方法。该方法利用**生成器（基于具有空间和通道挤压和激励模块的 UNet）**和**判别器**进行对抗训练来学习训练数据集的样本分布，从而提高分割性能。在Luna16数据集上，其DSC得分为80.74%，灵敏度为85.46%。这两种方法背后的动机是**利用生成对抗网络的特征来学习肺结节内**的**抽象特征**并生成适用于临床环境的训练样本。虽然这种方法在**有足够数量的训练样本**时是可行的，但由于**肺结节数据集中的样本分布不平衡**，它面临着挑战。目前，生成高质量和多样化的数据仍然是一个重大挑战。

## Method

### A. Pre-processing

> 肺结节CT图像预处理步骤如下：
>
> (a) Ground Truth: 同一结节的Mask 根据**质心坐标进行聚类**，并使用**开源Pyldc工具库对不同医生的注释进行平均**，级别设置为0.5[27]。然后获得真实值 (GT)，如图 2 所示。

![image-20240303114739315](https://oss.swimmingliu.cn/265f1704-d931-11ee-8eb8-c858c0c1debd)

> (b) 重新采样：将裁剪后的**肺结节区域**重新采样为 **1 mm**，并将生成的图像块大小调整为 **64×64×64** [28]，[29].
>
> (c) **归一化和混合特征**：**使用Z-score对原始图像块进行标准化**，**将CT图像转换为标准正态分布**，提高网络训练的稳定性。**混合特征通过变换和加权求和**，**同时保留了切归一化后的对比度信息**（Cut-norm，eq（1））和 **Sobel算子的边界信息**，如图3所示。肺HU值的切归一化范围为**[-1000, 400]** [3]，然后**将截断的 CT 图像映射到 [0,1]**，从而**提高 CT 图像中低对比度组织和细节的可视性**，同时**增强网络的灵敏度**。 **Sobel算子是一种简单且稳定的算子，可以有效突出细微、血管旁和胸膜旁结节的边界信息**。另外，**Prewitt算子的成像结果与Sobel算子相似**，因此没有进行进一步的研究。
>
> ![image-20240303114954965](https://oss.swimmingliu.cn/2667ac62-d931-11ee-a124-c858c0c1debd)

![image-20240303142053806](https://oss.swimmingliu.cn/26719fd7-d931-11ee-ab7a-c858c0c1debd)

### B. Overall Design of the Dual-branch network

> 双分支网络 (DBNet) 的架构如图 4 所示。它代表了一种端到端肺结节分割方法，包括信息编码和解码两个阶段。在编码阶段，我们采用**主网络**和**辅助网络**概念。主网络（$En_raw$）以原始CT图像作为输入，而辅助网络（$En_aid$）则利用**混合特征图像**作为输入。主、辅分支的输入图像块大小为 $Image^{C×H×W×D}_{raw}$ ,  $Image^{C×H×W×D}_{help}$ ∈ $R^{1×64×64×64}$，其中 $C$、$H$、$W$、$D$表示输入通道、长度、宽度和深度。编码阶段总共包括**5个编码块**和**4次下采样**操作，不同编码阶段的特征图可以表示为 $f^i_r$ , $f^i_a$ , $i$ ∈ [1, 5]，$r$  和 $a$ 表示主分支和辅助分支，特征图从 $32 × 64 × 64 × 64$ 过渡到 $64 × 32 × 32 × 32$，然后过渡到 $128 × 16 × 16 × 16$、$256 × 8 × 8 × 8$，最后过渡到 $512 × 4 × 4 × 4$

![image-20240303120522585](https://oss.swimmingliu.cn/2693665c-d931-11ee-afc2-c858c0c1debd)

> 在解码阶段，采用**双分支设计**，减少**网络参数和计算量**，同时提供**更多的特征选择和更强的泛化能力**。针对**双分支特征信息的融合设计**，提出了**多维融合模块**，从**多个维度的空间通道**中提取特征信息。然后，注意力模块的**输出特征图**将被**上采样**并与相应的编码阶**段特征图** $ f^i_r $ 和 $f^i_a$  融合，其中 $i$ ∈ [1,4]，以**补偿下采样过程造成的信息损失**。该融合过程重复四次，以逐渐预测肺结节区域。经过 $3×3×3$ 卷积的特征提取和优化后，得到预测的肺结节区域为 $Output^{1×64×64×64}$ ∈ [0,1]。设置**阈值0.5**，如果该值**大于阈值**，则该像素被认为是肺结节的一部分；否则，它被视为背景的一部分。
>
> ![image-20240303143554988](https://oss.swimmingliu.cn/26a13610-d931-11ee-93c0-c858c0c1debd)
>
> 网络的基本单元包括卷积（Conv）、BatchNorm（BN）和Swish激活函数[30]，统称为CBS。 BN **加速网络收敛**并**增强其表达能力**，而 Swish 激活函数可以**减轻与更深网络相关的梯度消失问题**。可训练参数 $beta$ = 1.0 动态调整 Swish 激活函数的形状，以更好地适应肺结节分割任务，如（2）所示，其中 $σ$ 表示 sigmoid 激活函数。将基本块的数量增加到2，并添加残差连接，得到残差基本块RCBS。

![image-20240303143854498](https://oss.swimmingliu.cn/26a92600-d931-11ee-8f0c-c858c0c1debd)

### C. Design of the Multi-Dimensional Fusion Module

> **多维融合模块（MDFM）**由两部分组成：**通道挤压洗牌注意力模块（CESA**）和**轴向多尺度空间注意力模块（AMSA）**，模块设计和内部注意力变化[31]如图5所示，整个过程可以表示为方程3至方程5。
>
> ![image-20240303144204221](https://oss.swimmingliu.cn/26c032c5-d931-11ee-94a2-c858c0c1debd)

#### 1) Channel Extrusion Shuffle Attention Module

> **通道挤压洗牌注意力模块 (CESA)** 的输入来自**双分支编码器**的**级联输出特征图**。在优化通道特征之前，首先使用**自适应均值池化**和**自适应最大池化**将特征图聚合为 $C×1×1×1$，并使用CBS基本块来融合两类特征信息。多尺度设计可以**扩大网络的感受野**，获得**更丰富的局部细节和全局信息**。受这一思想的启发，在通道维度上**对通道特征进行多尺度采样**，在**不改变融合特征图大小的情况下优化特征信息**。通道多尺度涉及**通道维度上的降维和扩展操作**，深度依次为 $C/2$、$C/4$、$C/8$、$C/16$。此外，通过密集连接设计，网络的**表达和泛化能力**得到进一步增强，该过程称为“挤出”。**通道混洗操作**将通道分为**8组**，并通过**混洗通道内的特征信息引入一定程度的随机性和多样性**。然后将**打乱后的特征**添加到**挤压操作后的特征图**上，最后通过点积操作恢复特征图。利用Sigmoid（σ）方法，将**特征图转换为[0,1]之间的概率分布**，并将**注意力权重映射到原始特征图**。

#### 2) Axial Multiscale Spatial Attention Module

> 准确捕获肺结节的**空间位置、纹理和形状信息**对于分割任务至关重要。因此，提出了**轴向多尺度空间注意力模块**。首先，我们使用  $7 × 7 × 7 $ 窗口大小的 **CBS** 块来感知全局上下文信息，同时以 **8、16、32 和 64 的压缩比 (r)** 压缩特征图通道。**压缩后的特征信息**不仅**更加关注空间维度特征**，而且**减少了网络参数和计算量**。然后，我们利用窗口大小 $H×3×3$ 、$3×W×3$ 和 $3×3×D$ 的基本块来感知 **Coronal、Sagittal 和 Axis 方向的特征图**，其中 **H、W 和 D 分别是特征图尺寸**，从而捕获不同平面**（冠状矢状、矢状轴、冠状矢状）**的局部信息以及不同轴向的全局信息。这种设计有助于强调和突出肺结节的关键特征，使模型能够更好地理解和表达多维数据中的特征。**对多维特征信息进行汇总，并利用残差连接补充梯度信息**。最后利用 $7×7×7$ 的卷积核来恢复特征图，完成空间特征信息的提取。

### D. Loss Function

> Dice相似系数[32]（DSC）是一种相似性测量函数，通常用于计算两个样本的相似性。 DSC ∈ [0,1]，**值越小**表明**模型预测结果 **与 **真实标签差距越大**。
>
> ![image-20240303145312767](https://oss.swimmingliu.cn/26c860cf-d931-11ee-ac60-c858c0c1debd)
>
> 其中 $P$ 表示二进制预测结果像素的集合，$G$ 表示二进制真实标签结果的集合。 $| P ∩ G |$ 表示 $P $ 和 $G$ 的交集。

## Experiments

### A. Datasets

> 本文在其实验设置中介绍了四个数据集：
>
> （1）LIDC： 肺部图像数据库联盟和图像数据库资源倡议 (LIDC) 数据集是**全球最大的公开肺癌数据集**。它包括来自多家医院的 1, 018 个研究病例，包含 11 种不同厚度类型的数据，最多有四名医生在 CT 图像中注释病变信息 [8]，**如图 6 所示**。首先，直径为 3mm 或 3mm 的结节由至少两名医生注释的较大的被保留[21]。然后，对由一名医生注释的结节进行二次筛查，以去除不确定或注释错误的样本，总共得到 2, 615 个肺结节样本。

![image-20240303145815570](https://oss.swimmingliu.cn/26dcade4-d931-11ee-948a-c858c0c1debd)

> 2）Luna16： **Luna16 数据集是 LIDC 数据集的子集，旨在解决 LIDC 数据集中 CT 切片厚度变化和数据质量等问题。** Luna16数据集总共包括1186个肺结节样本，这些样本由三名或更多医生注释，直径为3毫米或更大，切片厚度为2.5毫米或更小[33]。
>
> (3) 多厚度切片图像数据集(mThickSImg)： **剔除Luna16肺结节样本后**，共获得1429个多层结节，**主要用于内模型性能测试**，实验数据的样本分布如图7所示。
>
> (4) UniToChest： UniToChest [34] 数据集包括 306,440 个匿名胸部 CT 扫描切片和相应的肺结节分割掩模。这些数据来自623名不同的患者，经过处理和筛选后，总共获得了211张结节大小从3到35mm的多层CT图像用于外部测试集。

![image-20240303150101628](https://oss.swimmingliu.cn/26ead66c-d931-11ee-94d9-c858c0c1debd)

### B. Lung nodule characteristic attributes classification

> LIDC 数据集总共包含由医生注释的 **9 个视觉特征**。在该实验中，去除了**内部结构和钙化特性**，并重新定义了**良性和恶性的分类**[35]，同时还纳入了肺结节的大小[24]。为了保证**多个医师对同一肺结节标注的一致性**，肺结节属性分类采用两种策略：（1）**良恶性肺结节**的定义是**多个医师对结节标注结果的中位数** [35]。 (2)其他特征属性通过投票过程获得。肺结节属性分类如表1所示。

![image-20240303150245612](https://oss.swimmingliu.cn/26f68504-d931-11ee-a565-c858c0c1debd)

### C. Evaluation Metrics

![image-20240303150416975](https://oss.swimmingliu.cn/270ba13a-d931-11ee-87b0-c858c0c1debd)

> 在本文中，我们使用**精度（PRE）、灵敏度（SEN）、骰子相似系数（DSC）和并集平均交集（mIoU）作为评估指标**，如方程（7）至（10）所示。 **True Positive（TP）表示正确分割的焦点区域**，**True Negative（TN）表示正确分割的正常组织区域**，**False Positive（FP）表示正常组织区域被错误地分割为焦点区域**，**False Negative（FN）表示焦点区域被错误地分割为正常组织区域**。

### D. Training Details

> 本实验使用Ubuntu 18.04.3 LTS操作系统作为实验基础平台。 CPU型号为Intel(R)Xeon(R)-Gold 6140，内存大小为187.4G，软件环境为Python 3.8、Conda 10.1、Pytorch 1.8.1在Tesla V100-SXM2-32GB显卡上实验。
>
> 实验遵循所有 **CAD 方法的一致设置**。在训练阶段，**数据集被分为5个子集**，**每个子集循环用作验证集**，**其余4个子集作为训练数据**。应用**数据增强技术，包括水平和垂直翻转、旋转和平移，来增强训练样本并增强模型的鲁棒性**。最终的性能评估基于5倍交叉验证获得的平均结果。 Adam 优化器 [37] 的初始学习率为 3e-4，批量大小为 12，**最多 500 次训练迭代**。使用的损失函数是 Dice 损失。另外，我们使用**提前停止机制来防止过拟合**。在 **50 轮训练中，如果没有达到较小的损失**，模型训练将结束。

## Result

### A. CAD model performance comparison

> 如表2所示，所提出的方法与一些**最近优秀的CAD模型**进行了比较。根据**样本数量和选择策略**，这些方法**分为小样本肺结节分割方法**和**一般样本数量肺结节分割方法**。这些方法的筛选标准列于表中。**在使用小样本量的肺结节分割方法中**，我们提出的模型**在包含 1,186 个结节的 Luna16 数据集上实现了最先进的性能**，通过 **5 folder 交叉验证**实现了最高 DSC 87.68%（最佳）和 86.89%（平均）。就整体 DSC 而言，这比**当前顶级小样本方法 DS-CMSF** 领先 0.93%。这证明了我们的方法在相对较小的训练样本量下的有效性。此外，为了研究大样本量的性能，我们的方法在 **LIDC 数据集中的 2,618 个节点集上进行了训练**。我们取得了最佳 DSC 分数和平均 DSC 分数分别为 85.42% 和 84.12%，显着优于**当前领先的 LNHG** 模型（82.05%）和其他主流算法。
>
> 总体而言，不同数据量下的测试结果表明本文方法具有较强的稳定性和泛化能力。

![image-20240303151144111](https://oss.swimmingliu.cn/27167130-d931-11ee-812d-c858c0c1debd)

### B. Ablation studies

> 在本节中，我们对所提出的 **CAD 模型**的不同部分进行了消融研究，以评估它们对性能的贡献。消融研究中使用的模型是**基于 Luna16 数据集**进行训练的。
>
> 在影响模型性能的因素中，研究人员普遍认为模型规模是关键因素。目前，大多数学者倾向于使用 $64×64×64$ 的 patch大小作为神经网络的输入。按照此设置，模型的初始通道数设置为32，**考虑到输入大小的约束**，模型的**最大下采样次数设置为**4。此时，瓶颈层的输入大小为 $4 × 4 × 4$ ，512 个通道。基于此，我们设计了四种双分支分割模型（称为模型A、B、C和D），深度分别为2、3、4和5层。这些模型均使用CBAM模块作为双分支模型的融合模块。实验结果表明，随着网络深度和通道数的增加，模型性能呈现出不断增加的趋势，DSC从80.61%上升到85.8%，如表3所示。因此，得出结论：在输入尺寸为64×64×64的情况下，本研究模型的最佳深度为5，瓶颈层有512个通道。
>
> 实验中主要探索了**两类辅助特征**。一种是利用**Cut-norm来突出病变的特征信息**，另一种方法**主要利用Sobel梯度算子来增强边界信息**。在最佳模型规模的讨论中，所有模型均使用Z-score标准方法进行训练。训练这**两个手动特征并与模型 D** 进行比较后，发现使用 **Cut-norm 的模型 E 和使用 Sobel 算子的模型 F** 在 DSC 中分别与模型 D 相差 0.61% 和 0.78%。这说明了该辅助分支策略的有效性。此外，进一步验证了Sobel算子在不同数据尺度下的有效性，发现**模型D和模型F**在LIDC数据集上的DSC性能分别达到83.56%和83.82%，相差0.26%。这表明，即使在更大规模的测试集中，Sobel算子仍然有效，因此，暂时将Sobel算子用作辅助特征。

![image-20240303151528629](https://oss.swimmingliu.cn/2722d816-d931-11ee-a180-c858c0c1debd)

> 进一步**基于Sobel算子作为辅助特征**，提出了**通道挤压洗牌注意力模块和轴向多尺度空间注意力模块，即模型G和模型H**，模型性能最优达到86.63%。最后，为了同时保留高对比度病变区域和突出显示的边界细节以方便学习，我们使用**变换操作和加权求和来整合这两个特征**。通过这种策略，本文提出的方法获得了 86.89% DSC 的最佳结果，超过了**单独使用 Sobel 滤波器特征的性能**。直接的性能提升验证了我们辅助功能的有效性。
>
> 虽然**模型 H 和我们的整体性能差距并不显着**，但模型 H **在内部数据集中的 DSC 性能为 63.03%，对于 GGO 类型评估的灵敏度为 62.42%**。然而，**使用混合特征的进一步替代导致灵敏度和 DSC 性能显着提高，分别达到 65.27% 和 65.08%**。观察模型G、H和Ours的收敛速度，随着注意力模块数量的增加和混合特征的替换，模型拟合所需的epoch逐渐增加，但综合考虑，总体费用是值得的。

### C. Comparison experiments of different datasets and segmentation CADs

> 本节将我们的方法与开源医学分割模型 UNet [38]、UNet++ [28]、ReconNet [39]、Unetr [40] 和 Asa [41] 在不同数据集上的测试结果进行比较。**结果如表IV所示**。总体而言，**该方法在 Luna16 数据集上实现了 86.89% 的 DSC**，分别领先 UNet 和 ReconNet 模型 1.38% 和 0.77%。在基于Transformer的方法中，在DSC方面，它以4.01%和2.66%的优势超过了Unetr和ASA。整体性能超过了经典和现有的优秀分割模型。具体而言，该方法的精度为87.13%，灵敏度为87.02%。更高的**灵敏度可以更好地识别结节，防止漏诊**；而更高的精度为医生提供了更可靠的分割区域，减少误诊。这对于肺结节分割任务至关重要。此外，该方法实现了高 mIoU，**表明预测与真实情况之间有更大的重叠**。可以很好地分割大部分肺部病变区域，辅助合理诊断，有利于临床使用。
>
> 随后，所有方法都**在内部 mThickImgs 数据集**和**外部 UniToChest 数据集**上进行了验证。在这两个数据集上，UNet++ 达到了最高的精度，但灵敏度较低。这**表明 UNet++ 在真阳性区域中假阴性的比例较大**，**丢失了更多样本并最终影响了灵敏度指标**。这种**不平衡对于临床目的是有害的**。此外，ReconNet在mThickImgs数据集中具有较高的灵敏度。我们的方法在两个数据集中实现了 75.32% 和 80.64% DSC，分别超过次优方法 0.57% 和 0.59%，实验结果再次证明了我们方法强大的泛化能力。
>
> 总体而言，我们的方法在不同数据集的指标上比其他最先进的3D CAD模型具有明显的优势，特别是在灵敏度和DSC这两个最关键的分割性能指标方面，验证了我们方法的分割能力在多源医学图像中。

![image-20240303152233467](https://oss.swimmingliu.cn/272c056f-d931-11ee-8154-c858c0c1debd)

## Analysis of lung nodule performance

### A. Effect of different layer thicknesses on model performance

> 我们分析了 **mThickSImg 数据集中肺结节的分布**，并检查了**不同 CT 切片厚度和尺寸的结节性能变化**。我们将**数据分为六组**，数据样本分布如表六所示。然后对**不同的CAD模型**进行测试，并将不同模型的平均测试结果作为最终的性能趋势，如表5所示。

![image-20240303152144809](https://oss.swimmingliu.cn/27354151-d931-11ee-b284-c858c0c1debd)

![image-20240303152909321](https://oss.swimmingliu.cn/273f537a-d931-11ee-9f21-c858c0c1debd)

> 我们的方法是基于2.5mm层厚度以下的样本进行训练的，通过观察精度，我们发现在1.5mm和1.5mm ~ 2.5mm的比较中，网络的灵敏度随着层厚度的减小而增加。
>
> 另外，**在观察DSC性能时**，**注意到DSC性能随着层厚度的增加而增加**。然而，**似乎存在随着层厚度减小灵敏度增加而DSC降低的现象**。为什么会出现这种现象呢？**一方面，这种趋势可能会受到数据分布、样本数量和样本标签的影响而有些偏差**。一般来说，**随着 CT 层厚度的增加，图像分辨率会降低，可能导致神经网络难以捕获更精细的特征和结构**。
>
> 因此，**由于较厚的 CT 层中详细信息的丢失**，**网络随着层厚度的增加而表现出较低的灵敏度**。另一方面，**虽然层厚度的减少提高了 CT 图像的层内分辨率**，并能够**更准确地呈现解剖结构和病变**，但同时**也减少了接收到的光子数量**。与较厚的图像相比，光子计数的减少可能会导致噪声增加和对比度降低。这些因素会对肺结节（尤其是 GGO）的分割产生负面影响，可能导致验证集的性能下降。
>
> 我们还发现，在**基于薄层图像进行训练然后在厚层图像上进行验证后**，网络表现出更好的性能。我们相信，更**薄的 CT 图像由于具有更高的分辨率和更详细的信息**，**使网络能够学习更微妙的特征**。这些**细微的特征在2.5mm以上的层厚图像中具有更强的泛化能力**，这是基于厚层数据所不具备的。因此，这种现象可能表明关于层厚度效应的更好的性能。

![image-20240303153007188](https://oss.swimmingliu.cn/274a63f2-d931-11ee-b1c2-c858c0c1debd)

![image-20240303153021615](https://oss.swimmingliu.cn/27589c30-d931-11ee-b0fa-c858c0c1debd)

### B. Heterogeneity analysis of lung nodules

> 本节演示不同分割模型在Luna16数据集和mThickSImg数据集上的测试性能。总的来说，我们的方法在两个数据集上的大多数肺结节分割属性上都取得了领先的结果

#### 1) Subtle feature

> 肺结节的早期诊断和治疗对于疾病的诊断具有重要意义。
>
> 可以看出，在Luna16数据集中，我们的方法和UNet在 **“Extremely Subtle” 属性和“Moderately Subtle”属性的分割上表现更好**，并且**我们的方法仅次于UNet的方法**，达到了70.99%和76.69%。在mThickImgs测试集中，我们的方法**在所有Subtlety特征属性上都达到了最佳性能**，“Extremely Subtle”和“Moderately Subtle”属性分别**超过UNet和ReconNet 2.75％、1.03％和2.33％、0.94％** 。 UNet网络在两个数据集中**表现出不同的性能变化，这并不排除数据分布和数据质量的影响**。由于双分支和混合特征的设计，我们的方法在 mThickImgs 测试集上对“Extremely Subtle”属性和“Moderately Subtle”属性分割有较大的性能提升，这证明了我们的方法具有很强的泛化能力和鲁棒性，并提高了临床使用的可靠性。 Extremely Subtle GGO 的分割结果如图 10-(1) 所示。

![image-20240303152756024](https://oss.swimmingliu.cn/276eb1c2-d931-11ee-8ed7-c858c0c1debd)

#### 2) Texture feature

> 观察两个数据集中“Texture”特征的测试结果可以发现，在Luna16数据集中，我们的方法在“Solid/Mixed”和“NonSolid/Mixed”类别上表现出了很大的进步，分别为2.73%与次优方法相比，分别提高了 1.62% 和 1.62%。在 mThickSImg 数据集中，我们的方法在 GGO 类和“NonSolid/Mixed”类中达到了 65.08% 和 70.2%，分别领先次优 0.35% 和 1.33%。
>
> 虽然对于 GGO 样肺结节，我们的方法没有显着改进，但对于具有混合纹理的结节，我们的方法进一步减轻了分割挑战。固体/混合分割结果如图10-（3至7）所示。

#### 3) Maligancy feature

> **良性和恶性的区分是肺结节分割任务的关键特征**。**良性结节常呈圆形或椭圆形，而恶性结节往往体积较大，并有明显的针状或分叶状特征**。两个数据集的测试结果表明，**不确定属性的分割结果最差，其次是良性结节**。我们的方法改进了这三类属性的分割，**与次优方法、不确定和恶性分割相比**，在 Luna16 数据集中增加了 0.73%（良性）、0.28%（不确定）和 0.87%（恶性）如图10（1、2～7）所示。在**mThickSImg测试集中，该方法的性能提升更为显着，分别领先1%（良性）、0.83%（不确定）和0.61%（恶性）**。提高肺结节良性和不确定性的分割性能对于临床诊断至关重要。采用相同的CAD方法对这两类结节进行分割可以有效观察其发展趋势。与医生手工勾画相比，该方法提高了诊断的准确性和可靠性。

#### 4) Other features

> 除了上述三种常见且具有挑战性的分割特征外，**肺结节的“球形”、“边缘”、“分叶”和“毛刺”特征对临床诊断具有一定的指导意义**。观察“球形”特征，可以发现当前方法在常见的“卵形或圆形”属性上表现出更好的性能。一方面它有更多的训练样本，另一方面也更简单地分割该类的样本。我们的方法在 mThickSImg 数据集中的“线性”和“卵形/线性”病变形状属性中表现出显着优势，导致“线性”和“卵形/线性”属性中的 DSC 次优，分别为 2.74% 和 1.81% 。线性分割结果如图10-(6)所示。还值得注意的是，我们的方法在 Luna16 数据集的 Margin 特征中的五个属性上显示出巨大的改进，特别是“Poorly Defined”属性。
>
> 在 mThickImgs 测试集中，我们的方法在五个属性上分别与次优值相差 0.65%、0.93%、0.7%、0.94% 和 1.02%，这也表明辅助特征可以减轻图像边界的复杂性。病变区域。分叶和毛刺是判断恶性程度的重要指标。这些特征可以帮助医生区分结节边缘的规律性，并有助于对结节进行初步分类和观察。
>
> 可以看出，与次优相比，我们的方法在 mThickImgs 数据集的性能方面表现出了显着的改进。然而，“标记分叶”和“标记毛刺”特征属性的分割仍然具有挑战性。在不同尺寸的结节中，我们的方法在分割3到6mm范围内的实性结节方面也表现出了一定的性能改进。
>
> 与次优方法相比，我们的方法在两个测试集中分别提高了 0.73% 和 1.39%。然而，在“SubSolid”中，大多数方法没有表现出显着差异。

## CONCLUSION

> 我们提出了一种简单有效的双分支网络分割方法。**五折交叉验证的结果表明**，该方法能够有效、稳定地执行分割任务。此外，**我们的方法对大多数肺结节的特征特征和复杂数据样本的分割具有一定的适用性**，特别是对于**细微的、纹理混合的、边界模糊的、良性的和不确定的结节**，可以大大降低临床应用的难度，并提供为医生提供更可靠的参考。
>
> 这种方法也有一些局限性。首先，**虽然混合特征设计保留了边界和对比度信息，但它间接引入了额外的噪声，这可能会干扰网络的特征学习。**其次，**我们的方法在“Subsolid”小结节的分割方面没有表现出显着的性能改进**，未来可**能会尝试更深或更详细的多尺度设计来缓解这个问题**。另外，由于**实验整体采用多位医师标注的平均值作为最终的GT，一定程度上保证了结果的准确性**。但也可能导致最终的GT与实际情况存在差异，导致区域不完整或有刺状等详细信息丢失，如图10-(7)所示，对“Marked Lobulation”的分割提出挑战”和“标记毛刺”属性。未来，可能会开发多置信区域方法来缓解这个问题。

读完的第一感觉： 太长了....