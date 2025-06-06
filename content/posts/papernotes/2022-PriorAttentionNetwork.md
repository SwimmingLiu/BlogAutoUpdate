---
title: "Prior Attention Network: 用于医学图像中多病灶分割的预先注意网络"
date: 2023-11-28T14:55:47+08:00
lastmod: 2023-11-28T14:55:47+08:00
author: ["SwimmingLiu"]

categories:
- 📝 Paper Notes

tags:
- Unet
- DeepSupervision
- IntermediateSupervision

keywords:
- PANet
- DS
- IS

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
# Prior Attention Network: 用于医学图像中多病灶分割的预先注意网络

## Abstract 

> 医学图像中邻近组织的多种类型病变的准确分割在临床实践中具有重要意义。基于从**粗到精策略的卷积神经网络（CNN）**已广泛应用于该领域。然而，由于**组织的大小、对比度和高类间相似性的不确定性**，多病灶分割仍然具有挑战性。此外，**普遍采用的级联策略**对**硬件要求较高**，限制了临床部署的潜力。为了解决上述问题，我们提出了一种新颖的先验注意网络（PANet），它**遵循从粗到细的策略**来在医学图像中执行多病灶分割。所提出的网络通过在网络中**插入与病变相关的空间注意机制**，在单个网络中实现了**两个步骤的分割**。此外，我们还提出了**中间监督策略**，用于**生成与病变相关的注意力**来获取**感兴趣区域（ROI）**，这加速了收敛并明显提高了分割性能。我们在两个应用中研究了所提出的分割框架：**肺部 CT 切片**中多发性**肺部感染的 2D 分割**和**脑 MRI 中多发性病变的 3D 分割**。实验结果表明，与级联网络相比，在 2D 和 3D 分割任务中，我们提出的网络以更少的计算成本实现了更好的性能。所提出的网络可以被视为 2D 和 3D 任务中多病灶分割的通用解决方案。源代码可在 https://github.com/hsiangyuzhao/PANet 获取

问题导向：

①**组织的大小、对比度和高类间相似性的不确定性**

②多类别病灶分割

③**普遍采用的级联策略**对**硬件要求较高**

## Introduction

> **医学图像分割对于疾病的准确筛查和患者的预后具有重要意义。基于病灶分割的病灶评估提供了疾病进展的信息，帮助医生提高临床诊断和治疗的质量。然而，手动病变分割相当主观且费力，这限制了其潜在的临床应用。近年来，随着人工智能的快速发展，基于深度学习的算法得到了广泛的应用，并在医学图像分割方面取得了最先进的性能[1]**。卷积神经网络（CNN）由于其高分割质量而在医学图像中的病变分割中很受欢迎。此类算法通常具有**深度编码器**，可从输入图像中自动提取特征，并通过以下操作**生成密集预测**。例如，Long等人[2]提出了一种用于图像语义分割的全卷积网络，该网络颇具影响力，并启发了后来的医学分割中的端到端框架。 Ronneberger等人[3]提出了一种用于医学图像分割的U形网络（U-Net），该网络在医学分割的许多领域都显示出了可喜的结果，并已成为许多医学分割任务的虚拟基准。

这一段都可以当成经典医学图像分割的背景引入

> 然而，尽管医学分割取得了这些突破，但目前的**医学分割方法主要集中在病灶的二元分割上**，即**区分病灶（前景）和其他一切（背景）**。尽管二元分割确实有助于**隔离某些感兴趣区域**并允许对**医学图像进行精确分析**，但在某些需要**对病变进行多类分割的场景中，二元分割还不够**。与二元分割相比，由于**组织的类间相似性，这种情况要困难得多**，因为**不同类型的病变在纹理、大小和形状上可能相似**。具有从粗到细策略的级联网络已广泛应用于此类场景，例如**肝脏和病变的分割、脑肿瘤分割**[4]、[5][6]、[7]。
>
> 此类网络通常由两个独立的网络组成，其中第一个**网络执行粗分割，第二个网络基于从第一个网络分割的 ROI 细化分割**。然而，尽管**级联网络**已广泛应用于医学图像的**多病灶分割**，但级联策略也有其缺点。由于**级联网络由两个独立的网络组成**，**参数量和显存占用通常是单个网络的两倍**，这对硬件要求较高，限制了其在临床使用的潜力。更重要的是，由于级联网络中的两个网络通常是独立的，因此**级联网络的训练过程有时比单个网络更困难**，这可能导致欠拟合。

级联网络：参数量大、容易欠拟合。

> 在本文中，我们提出了一种名为先验注意网络（PANet）的新型网络结构，用于在医学图像中执行**多病灶分割**。所提出的网络由**一个**用于**特征提取的编码器**和**两个分别生成病变区域注意力和最终预测的解码器组成**。该网络与**注意力机制**结合在一起。为了**减少参数大小和硬件**占用，我们使用**网络编码器的深层、语义丰富的特征**来**生成病变区域的空间注意力**。
>
> 然后，**编码器生成的特征**表示通过**空间注意力**进行细化，并将其发送到解码器以进行**最终的多类预测**。为了**提高分割性能并加速收敛**，我们还在网络结构中引入了**中间监督和深度监督**。通过这些改进，与传统的级联网络相比，所提出的网络以**显着降低的参数大小和计算成本**实现了有竞争力的结果。

利用网络编码器的深层、特征信息来生成空间注意力（WTF ???）

中间监督、深度监督 （不错不错， 好多一区和顶会的文章都用深度监督）

> 这项工作的贡献体现在三个方面。**首先**，我们提出了一种新颖的网络架构，通过将传统级联网络中的两个分割步骤结合在**单个网络**中，遵循 2D 和 3D 医学图像中多病灶分割的**从粗到细的策略**。与级联网络相比，所提出的架构以更少的额外计算成本实现了有竞争力的分割性能，更容易训练和部署到生产环境。**其次**，我们提出了一种**监督空间注意力机制**，将**病变区域的注意力与网络提取的特征相结合**，将多病变分割分解为**两个更容易的阶段**，并且与当前**基于注意力的方法相比具有更好的可解释性**。**第三**，所提出的网络已在两个实际应用中得到验证，**包括肺部 CT 切片中的 COVID-19 病变的 2D 分割和多模态 MRI 中的脑肿瘤的 3D 分割**。所提出的网络在 2D 和 3D 任务中都优于前沿方法，并且在参数和计算成本方面比当前网络更高效。

一个网络、监督空间注意力机制、参数和计算成本方面比当前网络更高效。

## Related Work

> 1）图像分割的网络结构：用于图像分割的典型卷积神经网络通常由一个卷积特征提取器组成，其拓扑类似于常见的分类网络，自动从输入图像中提取特征，并进行基于卷积的操作以生成最终的密集预测。在自然图像分割领域，FCN [2]、DeepLab [8]、PSPNet [9] 和 SegNet [10] 因其性能和效率而颇受欢迎。对于医学分割，U-Net [3] 在许多任务中相当流行，并且已被修改为许多改进版本，例如 Attention U-Net [11]、U-Net++ [12]、V-Net [13] 和H-DenseUNet [14]在某些领域获得更好的性能。
>
> 2）级联网络在医学分割中的应用：**级联网络**已广泛应用于**正常组织和病变的分割以及不同类型病变的分割**，包括**肝脏病变、脑肿瘤、硬化病变和前列腺癌**的分割[4] ，[15][5]，[16]。例如，Awad 等人[17]提出了一个名为 **CU-Net** 的级联框架，用于在 **CT 扫描中对肝脏和病变进行自动分割**。他们还提供了可以指导临床治疗的有用信息和解释。 Xi等人[18]提出了一种**级联U-ResNets**，它遵循一种新颖的**垂直级联策略**，并在他们的工作中评估了**不同类型的损失函数**。除了肝脏病灶分割之外，级联策略在 BraTS 挑战中也很流行。例如，BraTS 2019挑战赛的Top-2解决方案[6]、[7]都是具有不同级联策略的级联网络。
>
> 3）神经网络中的注意力：注意力机制受到人类感知和视觉认知的启发，并已普遍应用于计算机视觉任务中[19]，[20][11]，[21]。计算机视觉任务中的**注意力机制**是在**神经网络提取的特征表示上生成空间或通道权重图**。例如，Woo等人[20]开发了一个卷积块注意力模块（CBAM）来引入一种融合注意力机制，其中包括**通道注意力和空间注意力**。这种注意力模块可以插入到**常用的分类或分割网络中**。Oktay等人[11]在Attention U-Net中提出了一种新颖的注意力门，用于细化网络编码器提取的特征表示，**以促进网络专注于ROI**。最近，首先在自然语言处理任务中提出的变压器[22]已被引入到医学分割任务中。例如，Wang 等人 [23] 提出了一种 TransBTS，用于从多模态脑 MRI 中执行脑肿瘤分割。
>
> 综上所述，注意力机制已被广泛用于突出ROI并抑制不相关信息，但目前基于注意力的方法的研究并没有对注意力如何产生以及网络为何关注某些区域提供清晰的解释，这使得限制了注意力机制的可解释性。

![image-20231127123123123](https://oss.swimmingliu.cn/p3bh0.png)

## Method

在本节中，我们将详细介绍所提出的先验注意网络架构。在第一部分中，我们将概述所提议的网络。然后，我们相应地提供有关具有**中间监督、参数化跳跃连接和具有深度监督的多类解码器**的所提出的**注意引导解码器的详细信息**。

### 3.1 . Overview of Network Architecture

> 基本上，我们提出的网络是基于 U-Net [3] 架构进行修改的，该架构具有 U 形拓扑以及编码器和解码器之间的跳跃连接。在提出的**先验注意网络**中，一种新颖的**注意引导解码器模块**被集成到网络的**跳跃连接**中，以通过**空间注意来细化特征表示**。网络中还引入了一种新颖的**参数化跳跃连接**，以指导网络学习**普通特征图和精炼特征图之间**的**比率**。注意力引导解码器从编码器获取丰富的语义特征并**生成空间注意力图**来指导接下来的**多类分割**。为了产生与投资回报率相关的注意力，框架中使用了**中间监督策略**。然后将**细化的特征图发送到多类解码器**以进行最终的密集预测。
>
> 多类解码器采用**深度监督策略**以获得**更好的收敛性并提高分割性能**。这种网络拓扑**通过注意力引导解码器生成的注意力图**在单个网络中实现了**传统级联网络**的两个步骤。组网方案如图2所示。

![image-20231127114826756](https://oss.swimmingliu.cn/p3oUC.png)

### 3.2 Attention Guiding Decoder

> 在典型的级联网络中，分割的第一步是**执行粗分割**并找到输入图像中的 **ROI**。在提出的先验注意网络中，我们提出**了一个注意引导解码器**来执行该过程。所提出的**注意力引导解码器**被集成到网络中**以生成与 ROI 相关的注意力图**，然后利用**这些图来细化特征**表示并提高多类分割性能。
>
> 1）模块拓扑：所提出的**注意力引导解码器的基本拓扑**基于FCN [2]中提出的特征融合。从**网络解码器最深三层提取的特征表示**被馈送到**该模块**。由于特征图的空间大小不同，因此**首先执行线性插值以对特征图进行上采样**。然后对**特征进行压缩**，**抑制通道维度中的不相关信息**，**降低计算成本**。然后将压缩后的特征分别在**通道维度上连接起来以进行特征融合**。(torch.cat) 最后，**融合三个特征图以获得最终的预测**。
>
> 为了简单起见，我们使用 2D 分割来说明注意力图的计算。我们使用 Xi ∈ R Ci × Hi × Wi,i ∈ (3, 4, 5) 表示从网络编码器提取的特征图，其中 X5 表示最深的特征。特征压缩和融合计算如下：
>
> ![image-20231127115911999](https://oss.swimmingliu.cn/p38uW.png)
>
> 其中Z5 ∈ R C4 × H4 × W4表示X5和X4的融合特征，Z4 ∈ R C3 × H3 × W3表示X4和X3的融合特征，Wc5 ∈RC5×C4和Wc4 ∈RC4×C3表示相应的压缩卷积， W4 ∈ RC4×C4 表示融合 X5 和 X4 的融合卷积，⊕ 表示特征串联。
>
> 注意力引导解码器的输出计算如下：
>
> ![image-20231127120221195](https://oss.swimmingliu.cn/p3dMJ.png)
>
> 其中 W3 ∈ R C3×C3 表示融合 X4 和 X3 的融合卷积，Wout ∈ R C3 × 1 表示输出卷积，σ 分别表示 Sigmoid 激活。

![image-20231127120200365](https://oss.swimmingliu.cn/p3Ddv.png)

> 2）中间监督：计算机视觉中的传统注意力机制自动生成注意力图，但**注意力生成的过程通常是人类无法解释的**，并且**网络关注的区域可能与人类关注的区域不同**。
>
> **这种差距会限制注意力机制的性能和可解释性，有时还会导致网络容量的恶化**。为了解决这些问题，我们在网络中引入了**中间监督策略**。在遵循**从粗到细的方式的多病灶分割任务**中，我们首先生成一个二元Ground Truth，其中前景表示所有类型的病灶，背景表示其他一切。在具有 C 种病变的多病变分割任务中，我们使用 Gi,i ∈ (1,...,C) 表示第 i 类病变的二元基本事实，其中前景表示特定病变背景代表其他一切。二进制真实值 Gb 计算如下：
>
> ![image-20231127120613763](https://oss.swimmingliu.cn/p35GV.png)
>
> 然后利用二元损失函数来计算**二元真实值** yb 和**注意力引导解码器生成的注意力图** Y 之间的**二元损失 l**：
>
> ![image-20231127120701851](https://oss.swimmingliu.cn/p34eI.png)
>
> 其中 Lb 表示二元损失函数。然后利用计算出的损失 l 来监督注意力引导解码器的参数更新。

这个地方的中间监督主要是 要让中间的注意力机制起作用，不能随便生成。

> 通过引入中间监督，生成的注意力由输入图像的二元真实值进行监督。这样，**网络被迫学习多病灶分割任务的分解**，即**首先提取病灶区域**，然后**对病灶区域进行细粒度分类**。这种分解降低了**多病灶分割的难度**，并且与当前在“黑匣子”中生成注意力的基于注意力的医学分割方法相比，具有更好的可解释性。

### 3.3 Parameterized Skip Connections

> 跳跃连接已广泛应用于流行的卷积网络中，包括U-Net [3]、ResNet [24]等。受[11]的启发，我们建议将**注意力图**集成到**连接网络编码器和多网络**的跳跃连接中，形成多级解码器。在跳跃连接中，我们还引入了额外的残差路径来**恢复普通特征图**并进一步**提高分割性能**。与传统的残差路径相比，**残差路径的幅值因子**αi,i ∈ (1, 2,…, 5)被设置为网络的可学习参数，并在反向传播过程中更新。我们相信这样的设置可以为网络增加额外的非线性能力并增强跳过连接的有效性。
>
> 我们使用 Fi,i ∈ (1, 2,..., 5) 表示来自网络编码器的**普通特征图**，Y 表示**注意力图**，精炼后的特征图 Fri,i ∈ (1, 2,. .., 5) 多类解码器接收的信息计算如下：
>
> ![image-20231127121513541](https://oss.swimmingliu.cn/p3j6j.png)
>
> 然后将**细化的特征图**发送到**多类解码器**以进行最终的多类预测。

### 3.4 Multi-Class Decoder With Deep Supervision

> U形分割网络中的解码器用于接收编码器发送的特征图，随着解码器中的**特征通道数量的减少**和**空间分辨率的增加**，**分割性能逐步细化**。然而，随着网络变深，最深的解码器块变得难以训练，这可能会限制最终的分割性能。**深度监督策略已经被提出来训练深度卷积网络**[25]、[26]。在所提出的先验注意网络中，辅助预测是从不同级别的解码器块中提取的，并使用相同的基本事实进行监督。我们使用 Pi,i ∈ (1, 2, 3) 表示来自多类解码器的辅助预测，Pm 表示最终的多类预测，g 表示真实值，Lm 表示多类损失函数。最终的多类损失计算如下：
>
> ![image-20231127121708354](https://oss.swimmingliu.cn/p3iP2.png)
>
> 所提出的多类解码器的解码器块也与当前网络设置具有共同的设置，即卷积层、归一化层和非线性激活单元的堆栈。

总结一下：

① 注意力机制 + 中间监督：最后三层特征融合 + 这一部分做深度监督（原来这样也叫创新）

② 跳跃连接的部分加了一个α因子 （感觉像权重一样的东西  ）

③ 多阶段的深度监督 （这个就算一个trick吧，大家都在用）， 不过这里变成了多类别

## Experiments

### 4.1 Strong Baselines and Evaluation Metrics

> 为了研究网络架构的性能差异，我们将所提出的先验注意网络与医学分割中最流行的方法进行了比较，包括 U-Net [3]、Attention U-Net [11] 和级联 U-Net，在两个 2D 中和 3D 分割任务。值得注意的是，与**他们论文中提出的原始版本相比**，基**线方法根据网络拓扑方面的某些任务进行了修改和优化**，以获**得性能提升**。我们将残差连接[24]、批量归一化[30]和来自 ImageNet 的预训练编码器引入到 2D COVIDlesion 分割任务的基线方法中，并且我们还将残差连接、实例归一化和 PReLU 激活引入到 3D 脑肿瘤分割任务中。除了网络拓扑之外，基线方法与所提出的先验注意网络共享相同的数据增强和训练配置。
>
> 对于 COVID-19 病变的 2D 分割，我们使用 Dice 指数、精度分数和召回分数来评估所提出的网络的性能。 Dice指数是一种用来衡量两个样本相似度的统计量，已广泛用于分割算法的评估。精确率衡量的是实际正确的阳性识别的比例，召回率衡量的是算法对阳性样本的敏感度。对于 BraTS 2020 挑战赛的 3D 分割，在在线门户上进行评估，并根据 Dice 指数和 95% Hausdorff 距离（HD）对算法进行排名。
>
> 我们使用 G 表示ground truth，P 表示密集预测，TP 表示正确预测的正样本，FP 表示错误预测的正样本，TN 表示正确预测的副样本，FN 表示错误预测的副样本。这些指标的计算方式如下：
>
> ![image-20231127185553944](https://oss.swimmingliu.cn/p3Jhd.png)

这个HD（Hausdorff ）也是一种评估分割结果的方式 （alright  又多了一种指标）

### 4.2 肺部 CT 切片中的 COVID-19 病灶的 2D 多病灶分割

> 1）数据：由于可用的开源COVID-19 CT分割数据集通常很小，因此利用**两个独立的公开可用数据集**，即**COVID-19 CT分割数据集[27]和CC-CCII数据集[28]**来验证所提出的方法二维分割任务中的方法。第一个数据集**包含来自 40 多名患者的 100 个轴向 CT 切片**，这些切片已重新缩放**至 512 × 512 像素并进行灰度化**。所有切片均由放射科医生**用不同的标签**进行分割，以识别**不同类型的肺部感染**。第二个数据集由 **150 名 COVID-19 患者的 750 张 CT 切片**组成，这些切片被手动分割为**背景、肺野、毛玻璃混浊和实变**。由于并**非所有 750 个切片都包含病变**，我们最终使用了 **150 名患者的 549 个带注释的切片**。对于这两个数据集，利用 **5 倍交叉验证**来评估所提出模型的性能。
>
> 折叠之间的数据根据患者进行分割，以避免潜在的数据泄漏。最终的标签和分割图包含 3 个类别，包括背景、**毛玻璃不透明度** (GGO) 和**合并** (CON.)。
>
> 2）实现细节：a）**模型设置和损失函数**：对于预训练网络编码器，我们采用来自ImageNet的预训练ResNeXt-50（32 × 4d）[31]作为基线方法和所提出的**先验注意网络**的编码器。对于解码器中的上采样，**采用双线性插值**，**比例因子为2**。对于中间监督和级联U-Net第一阶段的二元损失函数，我们采用**Dice Loss [13]和Focal的线性组合损失**[32]作为损失函数。对于最终输出的多类损失函数，我们采用**Focal Tversky Loss** [33]作为损失函数。
>
> b) 训练细节：我们的模型是在 Ubuntu 16.04 服务器上使用 PyTorch 1.7.1 框架实现的。我们使用 NVIDIA RTX 2080 Ti GPU 来加速我们的训练过程。在我们的训练过程中使用**Albumentations** [34] 进行数据增强，以**减少过度拟合**并**提高泛化能力**。首先，将所有**输入图像重新缩放为 560 × 560**，然后进行**随机亮度和对比度偏移以及随机仿射变换**。然后将图像随机裁剪为 512 × 512，然后进行随机弹性变换，最后输入网络。**该模型由 Adam 优化器优化**，β1 = 0.9、β2 = 0.999、γ = 1e − 8。L2 正则化也用于减少过度拟合。我们将模型权重衰减设置为 1e − 5。初始学习率设置为 1e −4 并降低，然后采用余弦退火策略。批量大小设置为 4，模型训练 40 轮。该模型使用 5 倍交叉验证进行评估。
>
> 3）定量结果：我们在两个数据集上的实验中不同模型的详细比较分别如**表一和表二**所示。如图所示，我们提出的网络在毛玻璃不透明度和固结的 Dice 分数方面优于 U-Net、Attention U-Net。所提出的 **PANet 以更少的参数和计算成本**实现了与级联 U-Net 竞争的结果。由于这些模型在模型主干和训练策略上是相同的，很明显，所提出的**注意力引导解码器、中间监督和深度监督**的组合对分割性能有很大贡献。**注意力引导解码器的利用**有助于模型更准确地检测感染组织并生成与感染相关的注意力图，从而有利于解码器中的多类分割。
>
> 此外，中间监督和深度监督的引入促进了网络的收敛，这也有助于提高性能。

![image-20231127190349627](https://oss.swimmingliu.cn/p3zUK.png)

好好好，这哥们儿，睁着眼睛说瞎话是吧（这Unet明明比你低啊，精度也没差多少啊）

> 4）定性结果：不同模型在 2D COVID-19 切片上的视觉比较如图 4 所示。由于模型在 Dice 分数方面非常接近，因此乍一看这些模型的表现相似。但与 U-Net 和 Attention U-Net 相比，所提出的 PANet 在实变和微小病变的分割上表现更好。
>
> 与 U-Net 和 Attention U-Net 相比，PANet 产生更准确的分割掩模，并且与 Cascaded U-Net 相比，所提出的网络以更少的计算成本实现了有竞争力的结果。

额 只要定量结果上去了，好像定性结果都是挑好的说吧？

![image-20231128140016412](https://oss.swimmingliu.cn/p31Fl.png)

![image-20231128140032492](https://oss.swimmingliu.cn/p3gdu.png)

> 5) ) 消融实验：进行了几次消融实验来评估我们模型中组件的性能，如表 III 所示。
>
> a）具有深度监督的多类解码器的有效性：为了探索深度监督策略的贡献，我们建立了两个实验：No.1（U-Net）和No.2（U-Net + DS）。表三的结果表明，深度监督在一定程度上对绩效有所贡献。
>
> b）注意力引导解码器的有效性：我们通过构建实验 3（U-Net + AGD w/o IS）来研究所提出的网络中所提出的注意力引导解码器的有效性。如表III所示，与实验1相比，注意力引导解码器的引入提供了显着的性能提升。这表明注意力引导解码器在所提出的网络中提供了有效的注意力图，从而指导解码器中的多类分割。
>
> c）参数化跳跃连接的有效性：为了探索所提出的参数化跳跃连接的有效性，我们建立了两个实验4（U-Net + AGD*）和5（U-Net + AGD）。引入参数化跳跃连接后，分割性能得到了提高，几乎没有额外的参数或计算成本。
>
> d）中间监督的有效性：为了研究所提出的 PANet 中中间监督策略的有效性，我们比较了第 3 号（U-Net + AGD w/o IS）和第 5 号（U-Net + AGD）。如表 III 所示，与第 3 种相比，具有中间监督的网络获得了额外的改进。此外，在比较第 6 种（U-Net + DS + AGD w/o IS）和第 7 种（PANet）时也可以观察到改进。 ）尽管改进相对较小。可以看出，深度监管的引入也对绩效产生了提升，因此中级监管对绩效的提升并不像以前那么显着。

![image-20231128140745555](https://oss.swimmingliu.cn/p3eST.png)

### 4.3 3D Multi-Lesion Segmentation of Brain Tumor From Multi-Modality Brain MRIs

> 1）数据：我们使用来自 BraTS 2020 挑战赛的开源多模态 MRI 数据集 [29]、[36] [37]。训练集由 369 个多对比 MRI 扫描组成，其中每个扫描包含四种模式，即原生 T1 加权、对比后 T1 加权 (T1Gd)、T2 加权 (T2) 和 T2 流体衰减反转恢复 (FLAIR) ）。
>
> 每次扫描都有相应的 4 类标签：背景（标签 0）、GD 增强肿瘤（ET，标签 4）、瘤周水肿（ED，标签 2）以及坏死和非增强肿瘤核心（NET/ NCR，标签 1)。验证集由 125 个多重对比 MRI 扫描组成，其模式与训练集和隐藏的基本事实相同。所有 MRI 扫描均去除颅骨，与相同的大脑模板 (SRI24) 对齐，并插值至 1mm3 分辨率。验证阶段通过在线门户进行，算法根据 3 个重叠肿瘤区域的性能进行排名，即增强肿瘤 (ET)、肿瘤核心 (ET + NET/NCR) 和整个肿瘤 (ET) + NET/NCR + ED）
>
> 2）实现细节：a）模型设置和损失函数：对于3D分割，由于缺乏开源预训练编码器，所有模型都是从头开始训练的。下采样通过跨步 **3 × 3 × 3 填充卷积**执行，**上采样通过三线性插值**实现。为了进一步提高在BraTS数据集上的性能，我们采用基于区域的训练策略（直接在重叠区域而不是独立标签上优化）并增强肿瘤抑制（如果增强肿瘤的预测体积为，则用坏死替换预测的增强肿瘤）小于某个阈值）在训练过程中。对于损失函数，我们采用 Dice Loss [13] 和 Cross Entropy Loss 的线性组合作为网络中**二分类和多分类**阶段的损失函数。
>
> b) 训练细节：我们的模型是在 Ubuntu 服务器上使用 PyTorch 1.7.1 框架实现的。由于3D分割的训练，尤其是级联U-Net对显存的要求较高，因此我们使用NVIDIA RTX 2080 Ti GPU和NVIDIA RTX 3090 GPU分别训练单个模型和级联模型。
>
> 由于训练过程的显存占用大于11Gb，我们采用PyTorch框架提供的原生混合精度训练程序来节省显存使用并加速训练过程。人工智能医学开放网络（MONAI）项目[38]和TorchIO[39]分别用于训练和推理阶段的数据加载过程。数据增强是在训练过程中通过 MONAI 项目进行的。
>
> 首先，**分别使用 z 分数标准化对所有模态进行标准化。然后通过随机翻转、随机强度偏移、随机强度缩放和弹性变换来增强图像。**最后，我们将图像块随机裁剪为 **128 × 128 × 128** 并将其输入网络。对于推理，我**们还采用基于补丁的推理管道来生成 BraTS 2020 验证集的预测**。面片大小设置为 128 × 128 × 128，**面片之间的重叠设置为 75%**。重叠区域中的预测是重叠块的平均值。这种重叠配置可以被视为自集成并产生更好的分割性能。对于模型评估，我们进行了 2 个单独的评估程序来比较分割模型的性能。首先，**我们对 BraTS 2020 训练集进行 5 倍交叉验证**，以比较**离散区域（增强肿瘤、瘤周水肿和非增强肿瘤核心）上的分割性能**。然后，我们对 BraTS 2020 验证集进行评估，以比较重叠区域（增强肿瘤、肿瘤核心和整个肿瘤）的分割性能，其中我们可以将所提出的方法与最先进的方法进行比较BraTS 挑战。
>
> 3）定量结果：BraTS 2020 训练集的交叉验证性能已在表 IV 中报告。
>
> 所提出的 PANet 在 **Dice 分数和 Hausdorff 距离方面**优于所有其他网络。此外，我们还在 BraTS 2020 验证集上对训练后的模型进行了验证。通过这种方式，我们还将所提出的网络与模态配对学习（BraTS 2020 中的 Top-2 解决方案）[35] 和研究中的 Transformer TransBTS [23] 进行了比较，除了基于 U 的常用网络之外-网。
>
> 性能列于表五中。我们提出的 PANet 在 BraTS 2020 验证集上优于 U-Net、Attention U-Net、级联 U-Net 和 TransBTS。此外，所提出的 PANet 在 BraTS 2020 上实现了与 Top2 解决方案类似的 Dice 分数，并且具有更好的 Hausdorff 距离。另外，值得注意的是，与 U-Net 相比，所提出的 PANet 仅增加了 3.9% 的额外 GFlops，并且以更少的计算成本实现了比级联 U-Net 更好的性能。这些结果表明，所提出的先验注意网络在 BraTS 2020 数据集上的分割性能和计算效率之间取得了复杂的平衡。

![image-20231128143139994](https://oss.swimmingliu.cn/p3cMp.png)

> 4）定性结果：我们可视化具有**不同分割难度**的 3 幅 MRI 图像，以展示不同模型的性能。对于最简单的情况（BraTS20_Validation_077），所有模型都能够分割病变，而所提出的 PANet 获得最高的**分割 Dice 分数**。对于有一定难度的病例（BraTS20_Validation_028），Attention U-Net和级联U-Net在**水肿分割方面都产生了严重的误报**，导致整个**肿瘤的Dice评分较低**。对于最难的情况（BraTS20_Validation_076），由于**增强肿瘤的假阳性分割**，U-Net、Attention U-Net 和级联 U-Net 的性能并不乐观，而所提出的 PANet 产生了最好的分割性能。所提出的 PANet 的成功归功于具有中间监督的注意力引导解码器，特征图通过空间注意力图进行细化，这可以防止潜在的误报预测。   

关于医学影像中的轴位面（横断面）、冠状面、矢状面的解释

1.冠状面 （Coronal），又称额状面。即从左右方向，沿人体的长轴将人体纵切为前、后两部分的切面。这种提法只是为了在临床中将器官位置描述的更具体，英文名称是：Coronal section；

2.矢状面 (Sagittal)就是把人体分成左右两面的解剖面，于这个面平行的也是矢状面。出于这个位置的叫矢状位。矢状位的英文名称是：Median sagittal section；

3.水平位 (Axial)又称横断位，即左右、前后构成的面为水平位，英文名称是:Transverse section。

![image-20231128143857189](https://oss.swimmingliu.cn/p376m.png)

![image-20231128143306784](https://oss.swimmingliu.cn/p3Q9R.png)

> 5）消融分析：在 BraTS 2020 验证数据集上也进行了与 2D 分割类似的消融实验，以评估我们模型中呈现的组件的有效性，如表六所示。
>
> a）具有深度监督的多类解码器的有效性：我们通过向网络解码器引入深度监督来构建实验2（U-Net + DS）。与基线（No.1）相比，实验No.2在增强肿瘤、肿瘤核心和整个肿瘤的分割性能方面提供了一定程度的性能提升。此外，当引入注意力引导解码器时，可以观察到性能的提高（第3和第6）。
>
> b) 注意力引导解码器的有效性：我们构建实验 3（U-Net + AGD w/o IS）来研究所提出的网络中注意力引导解码器的有效性。与基线相比，注意力引导解码器在增强肿瘤和肿瘤核心的分割性能方面产生了显着的改善。这表明注意力引导解码器对从编码器提取的特征图产生有效的注意力，从而更容易区分病变。
>
> c）参数化跳跃连接的有效性：我们建立了两个实验No.4（U-Net + AGD*）和No.5（UNet + AGD）来探索所提出的参数化跳跃连接的有效性。引入参数化跳跃连接后，在增强肿瘤和肿瘤核心方面分割性能有所提高，但整个肿瘤的分割性能略有下降。
>
> d）中间监督的有效性：我们通过比较表六中的第3号和第5号来调查中间监督策略的有效性。在No.5（U-Net + AGD）中，通过引入中间监督，增强肿瘤和肿瘤核心的Dice得分显着提高。但是当引入深度监督时，中间监督的性能提升并不像以前那么显着（第6和第7），这与2D分割情况类似。

![image-20231128143636223](https://oss.swimmingliu.cn/p3OcN.png)

## DISCUSSION AND CONCLUSION

> **多病灶分割**在临床场景中具有重要意义，因为**某种疾病**可能同时发生**多种类型的感染，不同感染阶段的患者可能会出现不同类型的病灶**。例如，**磨玻璃样混浊**（GGO）和**实变**（CON.）是COVID-19患者典型的肺部病变，前者**通常发生在早期患者**，而后者的增加**可能表明病情恶化**。胶质瘤可分为**低级别胶质瘤**（LGG）和**胶质母细胞瘤**，即**高级别胶质瘤**（GBM/HGG），并且在发生HGG的患者中更有可能发现**强化肿瘤**。因此，多病灶分割在患者的筛查和预后方面具有巨大的潜力。
>
> 多病灶分割问题可以分解为**粗分割**和**细分割**，粗分割是对病灶进行粗**略分割**，而细分割则**基于前一分割**，以产生最终的**分割图**。级联网络广泛用于多病变分割任务，因为这些算法背后的逻辑非常自然。然而，级联网络在潜在的临床部署中**受到限制**，因为它们**缺乏灵活性**并且对**计算资源的要求很高**。与现有的级联网络相比，我们开发了一种先验注意网络，它将粗分割和细分割集成到一个网络中。我们提出的网络架构的优点是**分割性能**和**计算效率**的平衡。通过将分割的两个步骤结合在一个网络中，所提出的**先验注意网络**能够实现多病灶分割的**端到端训练**，在训练和推理方面都具有更大的灵活性，并且在临床部署中具有更大的潜力。此外，我们设法保持所提出的先验注意网络的性能，在分割性能和运行效率之间实现复杂的平衡。
>
> 与级联 U-Net 相比，我们提出的先验注意力网络在 2D 和 3D 任务中都实现了更好的性能和效率，如图 6 所示。
>
> 总之，我们提出了一种新颖的分割网络，即先验注意网络，用于医学图像中的多病灶分割。受流行的从粗到细策略的启发，我们通过**空间注意机制**将级联网络的两个步骤聚合成一个网络。此外，我们引入了一种新颖的**中间监督机制**来指导与**病变相关的注意图**的生成，这可以指导解码器中的后续多类分割。所提出的网络在 2D 和 3D 医学图像（包括 CT 扫描和多模态 MRI）上进行评估。对于 2D 分割，与级联 U-Net 相比，所提出的先验注意力网络以更少的计算成本获得了有竞争力的结果。对于 3D 脑肿瘤分割，所提出的先验注意网络在 BraTS 2020 验证数据集上产生了最先进的性能，并且优于基于 U-Net 的其他基线方法。实验结果表明，该方法在医学影像中的许多多病灶分割任务中具有巨大的应用潜力，与二元分割相比可以提供更多信息，并有助于医生未来的临床诊断。

![image-20231128144442585](https://oss.swimmingliu.cn/p3uCt.png)