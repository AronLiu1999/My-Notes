# On-Device Learning for Model Personalization with Large-Scale Cloud-Coordinated Domain Adaption

SJTU & Alibaba, KDD '22

## Abstract

提出问题：cloud-based learning 是工业和学术界的主流。云端数据和用户本地数据的分布有差异，导致全局模型对用户本地数据进行推理时得到的结果不是最优。

> the model trained over the global data on the cloud may be **non-optimal** to do inference over each user’s local data.

但是根据本地数据直接在终端设备上训练会导致严重的过拟合。为了解决纯云学习和纯终端学习的矛盾，作者提出了MPDA——一种新的端云协作学习框架。

主要思想：

> retrieve some similar data from the cloud's global pool, which functions as large-scale source domains, to augment the user's local data as the target domain

## 1. Introduction

不同的用户通常有不同的行为模式，他们的数据分布往往彼此不同。全局数据分布作为所有用户数据分布的混合，可能会偏离某个用户的数据分布。这种分布差异进一步意味着，在云上对全球数据进行训练的模型可能不是对每个用户的本地数据进行推理的最佳选择。所以作者认为每个用户应该持有一个个性化模型。

!!! question 那么这个个性化模型应该存入云端还是设备上呢？

模型数量与用户数是相同数量级，所以
> it is practically **infeasible** for the cloud to store a large scale of models, update them periodically, and provide real-time inference services for each user

但是

> training only over the local data will suffer from serious overfitting, since the local data on each device is scarce, which is different from the sufficiency of training data on the cloud

所以采用端云协同学习的方式，利用云存储的数据减轻边缘设备上学习时的过拟合，利用无处不在的移动设备的可扩展性来维护本地的个性化模型并对其进行实时更新和使用。

on-device <u>**M**</u>odel <u>**P**</u>ersonalization with large-scale cloud-coordinated <u>**D**</u>omain <u>**A**</u>daption

general idea: retrieve some outside samples from the global pool on the cloud for each user, which are similar to the user’s local data distribution and can augment the user’s local dataset

![](images/On-Device%20Learning%20for%20Model%20Personalization%20with%20Large-Scale%20Cloud-Coordinated%20Domain%20Adaption/On-Device%20Learning%20for%20Model%20Personalization%20with%20Large-Scale%20Cloud-Coordinated%20Domain%20Adaption1668927821081.png)

MPDA解决的是一个domain adaption problem, where the number of domains is large, while the size of each domain is small：

> from the perspective of each user, any other user’s data on the cloud can be viewed as a candidate source domain, while the user’s own data on the local device can be regarded as the target domain

在云上暴力搜索在target domain上表现最好的source domain是不可行的（时间复杂度与source domain个数呈指数增加），所以作者采用了如下流程将时间复杂度降低到与匹配到的少量用户数呈线性：

1. coarse-grained user matching on the cloud side
2. apply on-device incremental training over the samples of the match users
3. keep only the best model according to the evaluation performance over local samples

实验表明，MPDA可以减少分布差异并且减小过拟合的风险

为了支持MPDA，作者搭建了 device-tunnel-cloud pipeline：device-side主要包括on-device training and inference, model version control；cloud-side包括sample preparation and similarity-based user matching；up tunnel用于用户上传本地数据；down tunnel用于云端将匹配到的用户数据分发给各个设备。

## 2. Related Work

### 2.1 Device-Cloud Collaborative Learning

Federated Learning，Google提出最流行的端云协作框架，多个移动设备用户协同训练全局模型，无需将本地数据上传到云服务器，从而可以保护数据隐私。早期的工作集中在训练一个全局模型，最近的工作已经转向用于模型个性化的联合学习。

除了将整个学习任务卸载到设备端之外，还有工作将深度网络拆分成两部分。

MPDA与federated learning不同，MPDA保存用户的数据继而为用户提供更好的服务，而联邦学习是不保存用户数据的。MPDA也与现有的端云协同框架不同，MPDA侧重于domain adaption范式下的模型个性化。

### 2.2 Recommender System

推荐系统的基本任务是预测每个用户候选item的CTR（Click-Through-Rate），经典模型：LR（Logistic Regression），Wide & Deep，DeepFM，PNN，DIN（Deep Interest Network）

当前推荐系统的主流是将数据输入全局推荐模型进行推理。

### 2.3 Domain Adaptation

现有的multiple-source domain主要集中在少量大规模数据集之间的adaption，但是本文将每个用户的数据视作一个domain，所以domain的数量很大，但是每个domain的size很小，现有方法不适用于这种情况

## 3. Problem Formulation

描述端云协同模型个性化的目标，作者将之描述为一个domain adaption问题

### 3.1 Device-Cloud Collaborative Learning for Model Personalization: Our Consideration

![](images/On-Device%20Learning%20for%20Model%20Personalization%20with%20Large-Scale%20Cloud-Coordinated%20Domain%20Adaption/On-Device%20Learning%20for%20Model%20Personalization%20with%20Large-Scale%20Cloud-Coordinated%20Domain%20Adaption1669036020193.png)

共有$(n+1)$个用户，分别用$\{0,1,2,...,n\}$表示，$\mathcal{D}_i$表示第$i$个用户的真实数据分布，$\hat{\mathcal{D}_i}$表示第$i$个用户本地数据的empirial分布，$m_i$表示第$i$个用户的本地数据，云端收集所有用户的数据并维护一个全局数据池。$m \triangleq \sum_{i=0}^n m_i$表示云上全局数据大小，$\hat{\mathcal{U}} \triangleq \sum_{i=0}^n \frac{m_i}{m} \hat{\mathcal{D}}_i$表示全局数据的经验分布，$\mathcal{U}$表示$\hat{\mathcal{U}}$对应的真实分布。所以针对用户$i$的优化目标：

$$
\min _{h \in \mathcal{H}} L_{\mathcal{D}_i}(h) \triangleq \mathbb{E}_{z \sim \mathcal{D}_i} l(h, z)
$$

其中$\mathcal{H}$代表所有可能模型的集合，$l\left(h,z\right)$表示模型$h$在样本$z$上的非负损失（有上界），同时引入$h_{\mathcal{D}_i} \triangleq \arg \min _{h \in \mathcal{H}} L_{\mathcal{D}_i}(h)$来表示用户$i$的最优模型。

### 3.2 Cloud-Based Learning: The Mainstream

> 基于云的模型$h_{\hat{\mathcal{U}}}$和最佳模型$h_{\mathcal{D}_i}$的误差上界以不低于$1-\delta$的概率满足：
> $$ L_{\mathcal{D}_i}\left(h_{\hat{\mathcal{U}}}\right)-L_{\mathcal{D}_i}\left(h_{\mathcal{D}_i}\right) \leq 2 B \sqrt{\frac{4 d \log \frac{e m}{d}+\log \frac{1}{\delta}}{m}}+2 \operatorname{disc}_{\mathcal{H}}\left(\mathcal{U}, \mathcal{D}_i\right) $$
> 其中，$e$是自然常数，$d$是损失函数集合$\mathcal{L} \triangleq\{z \mapsto l(h, z) \mid h \in \mathcal{H}\}$的pseudo dimention，$\operatorname{disc}_{\mathcal{H}}\left(\mathcal{U}, \mathcal{D}_i\right)\triangleq \max _{h \in \mathcal{H}} |L_ \mathcal{U} \left( h \right) - L_{\mathcal{D}_i} \left( h \right)|$表示$\mathcal{U}$和$\mathcal{D}_i$的标签差异

由以上定理知，云上的全局数据$m$很大，第一项泛化误差较小；但是第二项的分布差异会影响模型的性能。

### 3.3 On-Device Training: A Trivial Method

> 在用户$i$本地数据上优化的模型$h_{\hat{\mathcal{D}_i}}$的泛化误差的上界以不低于以不低于$1-\delta$的概率满足：
> $$
L_{\mathcal{D}_i}\left(h_{\hat{\mathcal{D}}_i}\right)-L_{\mathcal{D}_i}\left(h_{\mathcal{D}_i}\right) \leq 2 B \sqrt{\frac{4 d \log \frac{e m_i}{d}+\log \frac{1}{\delta}}{m_i}}
$$

从以上定理得知，本地数据$m_i$的大小较小，所以完全在本地设备上训练会产生较高的泛化误差（但是会避免之前云上学习的分布差异）

## 4. Design and Analysis of MPDA

### 4.1 Fundamental Algorithm Design

![](images/On-Device%20Learning%20for%20Model%20Personalization%20with%20Large-Scale%20Cloud-Coordinated%20Domain%20Adaption/On-Device%20Learning%20for%20Model%20Personalization%20with%20Large-Scale%20Cloud-Coordinated%20Domain%20Adaption1669039583013.png)

首先，每个用户$i$从云端检索所有其他$n$个用户的数据；然后对于$2^n-1$个可能的混合经验分布（记为$\hat S$），用户$i$为每个混合分布优化模型，得到候选模型$\mathcal{H} _{\hat{\mathcal{S}}}$；最终，用户$i$在本地数据$\hat{\mathcal{D}_i}$上评估每个候选模型，然后选择具有最小损失的模型$h_{\hat{\mathcal{D}}_{\mathcal{A}}}$

### 4.2 Error and Complexity Analyses

为了模型在不同的target domain上的损失小，需要：

1. 从经验分布$\hat{\mathcal{D}}$到真实分布$\mathcal{D}$的泛化性能要好
2. 源域$\mathcal{D}$和目标域$\mathcal{D}_i$的分布差异要小

所以每个用户需要选择数据分布和本地数据分布相似的用户去最小化分布差异，同时，用户要尽可能多的引入外部用户去提升模型的泛化性能。但是以上的过程都是基于本地数据的经验分布$\hat{\mathcal{D}}_i$，而目标是针对$\mathcal{D}_i$，所以本身存在一个泛化误差：

> 算法1得到的模型$h_{\hat{\mathcal{D}}_{\mathcal{A}}}$的误差以不低于$1-\delta$的概率满足：
> $$
L_{\mathcal{D}_i}\left(h_{\hat{\mathcal{D}}_{\mathcal{A}}}\right)-L_{\mathcal{D}_i}\left(h_{\mathcal{D}_i}\right) \leq 2 B \sqrt{\frac{n \log 2+\log \frac{2}{\delta}}{2 m_i}}+\min _{\hat{\mathcal{D}} \in \hat{\mathcal{S}}}\left(L_{\mathcal{D}}\left(h_{\hat{\mathcal{D}}}\right)-L_{\mathcal{D}}\left(h_{\mathcal{D}}\right)+2 \operatorname{disc}\left(\mathcal{D}, \mathcal{D}_i\right)\right)
$$

第一项是根据本地经验分布选择外部数据的泛化误差，误差量级为$\mathcal{O}\left(n/m_i \right)$，第二项是在source domain上训练模型的泛化误差和分布差异所能达到的最小值（此项验证了算法1可以同时最小化泛化误差和分布差异）

本地数据大小范围：本地数据比候选用户数小得多的话，选择外部数据的泛化误差依然很大；如果本地数据比伪维度$d$大得多的话，直接在本地数据上进行on-device训练得到的模型泛化误差足够小，没必要使用MPDA来扩充数据集。

### 4.3 Efficient Approximation

![](images/On-Device%20Learning%20for%20Model%20Personalization%20with%20Large-Scale%20Cloud-Coordinated%20Domain%20Adaption/On-Device%20Learning%20for%20Model%20Personalization%20with%20Large-Scale%20Cloud-Coordinated%20Domain%20Adaption1669045956287.png)

云端增加粗粒度用户匹配，设备端使用增量训练来细粒度选择用户

## 5. Evaluation

离线测试数据集：MovieLens 20M Dataset，Amazon Electronics Dataset

模型：LR, Wide & Deep, DeepFM, PNN, DIN

评价标准：AUC

![](images/On-Device%20Learning%20for%20Model%20Personalization%20with%20Large-Scale%20Cloud-Coordinated%20Domain%20Adaption/On-Device%20Learning%20for%20Model%20Personalization%20with%20Large-Scale%20Cloud-Coordinated%20Domain%20Adaption1669248869131.png)

![](images/On-Device%20Learning%20for%20Model%20Personalization%20with%20Large-Scale%20Cloud-Coordinated%20Domain%20Adaption/On-Device%20Learning%20for%20Model%20Personalization%20with%20Large-Scale%20Cloud-Coordinated%20Domain%20Adaption1669248999045.png)

![](images/On-Device%20Learning%20for%20Model%20Personalization%20with%20Large-Scale%20Cloud-Coordinated%20Domain%20Adaption/On-Device%20Learning%20for%20Model%20Personalization%20with%20Large-Scale%20Cloud-Coordinated%20Domain%20Adaption1669249009595.png)

在线测试：淘宝

评价：clk-click, exp-exposure, PV-page view, UV-Unique Visitor

1. $\frac{\text{clk PV}}{\text{exp PV}}$，即CTR
2. $\frac{\text{clk PV}}{\text{exp UV}}$，表示每个用户的平均点击次数，可以用来衡量用户活跃度
3. $\frac{\text{clk UV}}{\text{exp UV}}$，表示至少点击区域图标一次的用户比例

![](images/On-Device%20Learning%20for%20Model%20Personalization%20with%20Large-Scale%20Cloud-Coordinated%20Domain%20Adaption/On-Device%20Learning%20for%20Model%20Personalization%20with%20Large-Scale%20Cloud-Coordinated%20Domain%20Adaption1669249450297.png)