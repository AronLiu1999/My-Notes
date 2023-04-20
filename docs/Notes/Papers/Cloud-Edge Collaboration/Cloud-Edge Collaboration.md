# AppealNet: An Efficient and Highly-Accurate Edge/Cloud Collaborative Architecture for DNN Inference

DAC 2021（CCF-A）, CUHK and SJTU

![](images/Cloud-Edge%20Collaboration/Cloud-Edge%20Collaboration1668152210457.png)

![](images/Cloud-Edge%20Collaboration/Cloud-Edge%20Collaboration1668153268950.png)

![](images/Cloud-Edge%20Collaboration/Cloud-Edge%20Collaboration1668476384446.png)

predictor输出Feature Extractor提取特征的“自信度”，由此来决定是否传输数据至云端模型。

AppealNet不止可以节省计算量（与云端模型相比），还可以减少开销（添加的predictor head与the gap/score margin相比），甚至可以取得比云端模型更高的准确率（云端模型参数是不参与训练的，作者解释说是因为云端模型预测损失被纳入训练损失之中导致的）

思考：

1. 切分网络时仅仅考虑特征数最少的层，换层会不会取得更好的效果？
2. 云端大模型接受的输入还是raw data，是否可以接受小模型中间层的输出以减小开销？
3. 小模型的选择还是利用现成的高效模型，根据硬件限制手动选择，能否根据特定任务对模型进行结构进行优化或是采用NAS等自动化方法构建小模型？
4. overall开销不明确，仅给出了相对值而没有给出绝对值，传输开销和云端模型推理开销的大小？

---

# Auto-Split: A General Framework of Collaborative Edge-Cloud AI

单位：华为

期刊/会议：SIGKDD（ccf-a）

时间：2021

## Abstract

It is a big challenge to design industry products to support sophisticated deep model deployment and conduct model inference in an efficient manner so that the model accuracy remains high and the end-to-end latency is kept low.

This paper describes the techniques and engineering practice behind Auto-Split, an edge-cloud collaborative prototype of Huawei Cloud.

## Introduction

The recent exciting advances in AI are heavily driven by two spurs, **large scale deep learning models** and **huge amounts of data**.

the gap between huge amounts of data and large deep learning models：大规模深度模型通常托管在具有强大计算能力的云服务器中。同时，数据往往分布在边缘端（edge）。

由于算力需求和模型体积，完全在edge上运行模型是不可能的。但是对于实际的行业应用而言，把edge上获取的数据上传服务器也不可取，会造成传输成本高、端到端延迟高和隐私问题。

端云协同的分类：

- Edge-only：应用模型压缩技术，造成严重的准确度下降
- Cloud-only
- Distributed：execute the model partially on the edge and partially on the cloud
  - Cloud-only approach: conducts inference on the cloud
  - cascaded edge-cloud inference approach: divides a task into multiple sub-tasks, deploys some sub-tasks on the edge and transmits the output of those tasks to the cloud where the other tasks are run.
  - multi-exit solution: deploys a lightweight model on the edge, which processes the simpler cases, and transmits the more difficult cases to a larger model in the cloud servers

目前在实际应用中的有Edge-only和Distributed

![](2022-11-09-09-59-56.png)

端云协同方式利用了深度网络某些中间层的数据体积要远远小于原始的输入数据，因此萌生了将DNN graph分成两部分——edge DNN and Cloud DNN，从而减少传输成本和端到端延迟。

端云协同的方式是通用的，不像之前提到的cascaded edge-cloud inference approach和multi-exit solution。

Auto-SPLIT partitions the DNN into an edge DNN to be executed on the edge device and a cloud DNN for the cloud device.Auto-Split also applies post-training quantization on the edge DNN and assigns bit-widths to the edge DNN layers.

Auto-Split considers the following constraints:

- edge device constrains: uch as on-chip and off-chip memory, number and size of NN accelerator engines, device bandwidth, and bit-width support
- network constraints: such as uplink bandwidth based on the network type (e.g., BLE, 3G, 5G, or WiFi)
- cloud device constraints: uch as memory, bandwidth and compute capability
- required accuracy threshold provided by the usr

![](2022-11-09-09-59-25.png)

## Related Works

聚焦分布式Inference，假设模型已经训练完成

### Model Compression (On-Device Inference)

量化：

- Quantization-aware training
- post-training quantization

Mixed precision post-training quantization

### Edge Cloud Partitioning

A light weight model is stored on the edge device and returns the result as long as the minimum accuracy threshold is met. Otherwise, intermediate features are transmitted to the cloud to execute a larger model.

现在的渐进推理（progressive inference）大多依赖人工设计，并且只适用于特定的应用，会存在延迟不确定的情况（根据输入数据的不同会路由到不同的exit），而且需要retraining并且重新设计DNN去实现多个exit points

![](2022-11-09-09-58-59.png)

### Related Services and Products In Industry

## Problem Formulation

![](2022-11-09-14-24-54.png)

## Auto-Split solution

### Potential Split Identification

![](2022-11-09-14-26-00.png)

Preprocess the DNN graph:

   1. conduct graph optimizations such as batch-norm folding and activation fusion on the original graph to obtain an optimized graph
   2. create a weighted DAG where nodes are layers and weights of edges are the lowest transmission costs
   3. the weighted DAG is sorted in topological order and a new transmission DAG is created

Thus, the list of potential splitting points is given by
![](2022-11-09-14-29-31.png)

### Bit-Width Assignment

先算
![](2022-11-09-14-33-16.png)

然后选择满足精度下降阈值A的方案，但是这个方案的搜索空间随之前的可能方案n的个数呈指数增加，为了减小搜索空间，将问题7解耦为两个问题：

![](2022-11-09-14-36-09.png)

并使用拉格朗日方法解决这两个问题

![](2022-11-09-14-42-11.png)

![](2022-11-09-14-48-06.png)

### Post-Soluton Engineering Steps

## Experiments

硬件
![](2022-11-09-14-55-54.png)

---

# On-Device Learning for Model Personalization with Large-Scale Cloud-Coordinated Domain Adaption

SIGKDD 2022, SJTU & Alibaba

基于全部数据在云端训练一个大模型对于每个用户不是最优解，每个用户应该拥有属于自己的个性化模型。但是每个用户的模型在云端进行训练和存储是不现实的，所以需要将模型训练和存储下放到每个用户的移动设备之中。但是这样又存在过拟合的问题，因为本地数据稀缺。

MPDA: on-device **M**odel **P**ersonlization with large-scale cloud-coordinated **D**omain **A**daptation

![](images/Cloud-Edge%20Collaboration/Cloud-Edge%20Collaboration1668497731952.png)

主要思想是 针对每个用户从全局数据中取一些outside samples，这些samples的分布要与用户本地数据分布相似，由此来扩充用户本地数据集。

实际上是解决一个domain adaption problem： 为了提高效率和可扩展性，首先在云端进行粗粒度的用户匹配，然后将匹配到的用户samples用于终端的增量训练，然后将模型用于评估本地样本，只保留性能最好的模型。

---

# CiNet: Redesigning Deep Neural Networks for Efficient Mobile-Cloud Collaborative Inference

SDM 2021(ccf-b)，Worcester Polytechnic Institute

![](images/Cloud-Edge%20Collaboration/Cloud-Edge%20Collaboration1668558510961.png)

![](images/Cloud-Edge%20Collaboration/Cloud-Edge%20Collaboration1668510932145.png)

两个子模型，终端提取区域，并将区域上传至云端处理

依然是卷积, model-lever collaboration

---

# CNNPC: End-Edge-Cloud Collaborative CNN Inference With Joint Model Partition and Compression

TPDS 2022（ccf-a）, XJTU

![](images/Cloud-Edge%20Collaboration/Cloud-Edge%20Collaboration1668562540656.png)

时延与计算量的trade-off

![](images/Cloud-Edge%20Collaboration/Cloud-Edge%20Collaboration1668562570608.png)

work flow:

![](images/Cloud-Edge%20Collaboration/Cloud-Edge%20Collaboration1668563738093.png)

首先进行system profiling 记录每个CNN层在各个设备上的计算时间和每个CNN层输出在End-Edge和Edge-Cloud之间传输所需时间，多次测试，取平均值。然后基于这些记录和传输过程中的模型压缩方法，CNNPC搜索最佳模型优化策略（给定推理准确率或时延的要求），最后将生成的子模型分别部署在不同的设备上。

压缩方法：8位量化的identical channel pruning

---

# Complexity-aware Adaptive Training and Inference for Edge-Cloud Distributed AI Systems

ICDCS(ccf-b) 2021, Purdue University

![](images/Cloud-Edge%20Collaboration/Cloud-Edge%20Collaboration1668567145469.png)

main block在云端训练，然后下载到边缘设备之中。在边缘端训练时main block不变，改变adaptive block 和 Extension Block的权重。

![](images/Cloud-Edge%20Collaboration/Cloud-Edge%20Collaboration1668567469236.png)

![](images/Cloud-Edge%20Collaboration/Cloud-Edge%20Collaboration1668609512181.png)

![](images/Cloud-Edge%20Collaboration/Cloud-Edge%20Collaboration1668647733501.png)

---

# HierTrain: Fast Hierarchical Edge AI Learning With Hybrid Parallelism in Mobile-Edge-Cloud Computing

IEEE Open Journal of the Communications Society 2020, 中山大学

![](images/Cloud-Edge%20Collaboration/Cloud-Edge%20Collaboration1668648925794.png)


a不能充分利用各个设备的计算资源，b通信延迟会导致训练缓慢。作者观察到大多数参数都是来自最后的全连接层，所以考虑将最后的全连接层放入一个worker，前面的卷积层分配给其他worker，这样只需要交换少量数据（局部梯度和局部模型参数）。

作者提出了HierTrain框架，用于选定DNN的最佳分区点，框架包括三个阶段：分析、优化和分层训练：
  1. 分析：分析不同设备中不同的层的平均执行时间、分析每一层输出的大小
  2. 优化：选择最佳的DNN模型划分点，并确定不同设备上的训练样本数（样本数由模型层的执行时间、层输出大小、带宽联合确定）
  3. 分层训练：根据调度策略分发样本、然后以分层的方式协同训练


# A Novel Architecture Slimming Method for Network Pruning and Knowledge Distillation

单位：浙江大学

期刊/会议：--

主要思想：利用PCA实现卷机过滤器剪枝

## 提出问题

剪枝大多依赖expert-designed handcraft rules，需要人的大量干预；知识蒸馏需要考虑学生和教师的容量差距。

为了解决这个问题，一些自动剪枝方法和学生网络架构搜索策略被提出。但是基于搜索的自动方法往往消耗算力巨大。

为了无需进行多轮搜索、并且保证得到的模型容量符合要求，作者提出了一种自动layer-wise的架构优化算法3AS，**主要针对CNN**。

## 方法

针对卷积层，提出一种最小化filters数量同时保持最大的信息容量的优化算法。受信息论与概率论启发，最大容量=参数最diverse=filter variance最大。

对于某一层，将卷积的每个filter squeeze成向量，将filter向量作为行组成矩阵，矩阵的列可以看作是多维空间中的点，每一个维度对应一个filter，此时这些点可以看作是多维高斯分布，此分布的协方差矩阵表示了这些点之间分布的离散程度，即信息容量

做正交线性变换后，信息容量不变；所以要通过线性变换找到并删去方差最小的维度，这样信息容量损失最小，作者采用了PCA降维的方式实现这一过程。

具体方法：逐层对协方差矩阵进行SVD求得特征值，逐层计算累计贡献率（前i个特征值之和占所有特征值和的比率）。

## 实验

实验设置：

- 数据集：MNIST、CIFAR10、ImageNet
- 模型：VGGNet、ResNet

对比对象：
- 剪枝标准：l1-norm, l2- norm, Geometric Median(GM), Batch Normalization(BN)
- 技术：剪枝和知识蒸馏


## 缺点

1. 仅适用于卷积层


