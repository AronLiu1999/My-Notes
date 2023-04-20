# 域适应

## 1. 什么是域适应

## 2. 域适应的分类

## 3. 域适应算法

### 3.1 加权算法

![](images/Domain_Adaption/Domain_Adaption1669555458925.png)

### 3.2 [迭代算法](https://proceedings.neurips.cc/paper/2011/file/93fb9d4b16aa750c7475b6d601c35c2c-Paper.pdf)

迭代地标记目标域，通常需要有标签的目标域样本，所以适合有监督和半监督的情况。深度模型在源域数据上进行训练，并对无标签的目标域样本进行标注，然后针对目标域中有标签的样本，重新训练一个新模型。

### 3.3 基于特征的域适应

基于特征的域适应技术旨在通过学习一种用于提取具有领域不变性特征表示的变换方法。通常通过将原始特征映射到新的特征空间来创建新的特征表示，然后在优化过程中最小化新的表示空间中域之间的差距，同时保留原始数据的基础结构。

这种方法又可以分为三类：

a. 基于子空间的域适应：旨在发现源域和目标域的共有的中间表征。

b. 基于特征变换的域适应：通过最小化边缘分布和条件分布之间的差异，同时保留原始数据的潜在结构和特征，将原始特征转换为新的特征表示。

!!! note 条件分布和边缘分布：

c. 基于重构的域适应：旨在使用中间特征表示中的样本重建来减少域分布之间的差异。

### 3.4 [分层贝叶斯模型](https://arxiv.org/pdf/1802.03601.pdf)

使用与domain-specific参数绑定的贝叶斯先验。

## 4. 域适应技术

### 4.1 域不变特征学习

大多数最近的域自适应方法通过创建域不变特征表示来对齐源域和目标域，通常采用特征提取器神经网络的形式。如果无论输入数据来自源域还是目标域，特征都遵循相同的分布，则特征表示是域不变的。假设我们可以使用域不变特征训练分类器在源数据上表现良好。在这种情况下，分类器可以很好地泛化到目标域，因为目标数据的特征与我们训练分类器的特征相匹配。

![](images/Domain_Adaption/Domain_Adaption1669561659392.png)

根据具体的对齐的方式（即图中Alignment Component）可以继续分类：

#### 4.1.1 Divergence-based Domian Adaptation

最小化源域和目标域的分歧评价标准（maximum mean discrepancy，correlation alignment，constructive domain discrepancy，Wasserstein Distance）

[这篇文章](https://openaccess.thecvf.com/content_cvpr_2018/papers/Zhang_Aligning_Infinite-Dimensional_Covariance_CVPR_2018_paper.pdf)通过对齐跨域无限维协方差矩阵来实现域适应的。首先将原始特征映射到再生希尔伯特空间(Reproducing Kernel Hilbert Space，RKHS)，然后在这个空间中使用线性算子将源域数据“移动”到目标域上，从而使转换后的数据和目标数据的RKHS协方差描述接近。

作者通过计算变换后样本与对应的目标域样本的内积，得到了一个新的领域不变的闭合形式的核矩阵，可以用于任何基于核的学习机。

#### 4.1.2 基于重构的域自适应（Reconstruction-based Domain Adaption）

 Alignment Component是一种重构网络，重构网络与特征提取器相反，它获取特征提取器的输出并重新创建特征提取器的输入。

[这篇文章](https://link.springer.com/chapter/10.1007/978-3-319-46493-0_36)提出了深度重建分类网络（Deep Reconstruction Classification Networks，DRCN），这是一个卷积神经网络，它同时学习两个任务：有监督的源域标签预测和无监督的目标数据重构。目标书学得的函数可以在目标域上运作良好

![](images/Domain_Adaption/Domain_Adaption1669564728162.png)

#### 4.1.3 基于对抗的域自适应

![](images/Domain_Adaption/Domain_Adaption1669600554195.png)

构建与domain discriminator相关的对抗性目标，来近似不同域的差异，然后最小化这个目标。

这种情况下的alignment component是一个域分类器（输出是从源域还是目标域生成的特征表示）