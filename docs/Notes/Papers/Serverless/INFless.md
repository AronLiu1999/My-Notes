# [INFless: a native serverless system for low-latency, high-throughput inference](https://dl.acm.org/doi/abs/10.1145/3503222.3507709?casa_token=jqqJo6-xObQAAAAA:73rB8Ciy_dxlfxm-T4TFFz1W0xnTws2-Di9Xe5bCLNGgU_OIxx9CWWem7xyihMD68NWBY67B6Lw)

ASPLOS ’22

Authors: Yanan Yang, Laiping Zhao, Yiming Li, Huanyu Zhang, Jie Li, Mingyang Zhao, Xingzhen Chen, Keqiu Li

Affiliation: College of Intelligence & Computing (CIC), Tianjin University, Tianjin Key Lab. of Advanced Networking (中国天津大学智能与计算学院和天津市先进网络关键技术实验室)

Keywords: Serverless Computing, Machine Learning, Inference System

## Abstract

This paper proposes a new serverless platform called INFless that is specifically designed for machine learning inference services. INFless provides a unified resource abstraction between CPU and accelerators, built-in batching, and non-uniform scaling mechanisms to achieve high throughput and low latency.

## Introduction

Advantages of Serverless Computing: resource management-free, auto-scaling and cost efficiency.

Disadvantages of current commercial serverless platforms:

1. do not allow users to specify latency requirements in their Service Level Objectives (SLOs), thus cannot satisfy diverse latnecy requirements.
2. Large inference models heavily rely on accelerators (such as GPUs, FPGAs NPUs), but current serverless platforms do not support accelerators.

On-Top-of-Platform (OTP) design

- Building another new buffer layer on top of the commercial serverless platform
- Do improve the cost-effiency and throughput
- But **also introduce additional latency for request scheduling**, and **the end-to-end latency guarantee is still not addressed**, and they are **oblivious to the underlying resource allocations and have limited ability to optimize system throughput**

![](images/INFless/INFless1681309191684.png)

Native Serverless Inference System:

- integrate the inference features inside the platform design (accelerator support, batching, ...)
- Challenges that need to be addressed:
  - Low Latency: guarantee the end-to-end latency
  - High Throughput: choose optimal hardware and batch configurations to maximize throughput
  - Low Overhead: practically deployable, efficient and easy to use

INFless:

- Co-design batch management and heterogeneous resource allocation and propose *the non-uniform scaling* policy to maximize resource efficiency.
- a light-weight combined operator profiling method to infer an appropriate configuration for latency requirement
- a Long-Short Term Histogram policy to reduce the cold start rate
- implement INFless on OpenFaaS

## Background and Motivation

Advantages of serving inference on serverless systems:

1. inference services can be deployed as stateless functions
2. without participating in instance management
3. their auto-scaling ability can deal with bursty workloads
4. pay-per-use billing model can reduce the cost

Limitations of Existing Serverless Platforms:

1. High Latency: ❌support accelerators
2. Cannot provide batched low-latency inference services for small size models
3. Over-provisioning -> poor resource utilization
4. one-to-one mapping between requests and instances -> poor resource utilization
5. OTP batching ❌ codesign batch config, instance scheduling and resource allocation
   1. batching is maintained in anther server
   2. delay and time consumed in severless platform are not transparent
   3. Uniform scaling policy in severless platform make it unable to adapt to different workloads -> suboptimal scheduling

Desirable severless inference system:

1. Supporting hybird CPU or accelerators
2. Resource-efficient scheduling
3. Built-in batching

## Methodology and System Design

![](images/INFless/INFless1681391181274.png)

### Batching

Each instance has its own batch queue, the batchsize and resource quata of each instance can differ

Existed instances' requests per second (RPS) has a lower bound and a upper bound, $R_{min}=\sum_{i \in[1, \ldots, n]} r_{low}^i$ and $R_{max}=\sum_{i \in[1, \ldots, n]} r_{up}^i$, respectively.

1. $R > R_{max}$: launch new instances for left $R - R_{max}$ RPS.
2. $\alpha R_{min}+(1-\alpha) R_{max} \leq R \leq R_{max}$: each instance proportionally share the workload, i.e., $r_i=r_{u p}^i-\frac{R_{max }-R}{R_{max }-R_{min }} \times\left(r_{u p}^i-r_{l o w}^i\right)$
3. $R < \alpha R_{min}+(1-\alpha) R_{max}$: extra instances will be released

### Operator Profiling

![](images/INFless/INFless1681394662788.png)

A model's profile is estimated by its ***dominated operators***

$o_i = \left\langle p_i, b_i, c_i, g_i, t_i\right\rangle$

![](images/INFless/INFless1681395953710.png)

### Scheduling

NP-hard problem --> Greedy Algorithm

![](images/INFless/INFless1681397662606.png)

Inorder to **maximize the throughput** while reducing **resourse fragmentation**, INFless develop a resource efficiency metric: $e_{i j}=\frac{R P S / \text { resource }}{\text { fragmentation }}=\frac{r_{u p} /\left(\beta c_i+g_i\right)}{1-\left(\beta c_i+g_i\right) /\left(\beta C_j+G_j\right)}$

### Coping with Cold Start

Hybird histogram policy(HHP): ***too conservative***

So a new method named Long-Short Term Histogram (LSTH) is proposed: weighted sum of head and tail of the long-term and short-term histograms

## Evaluation

### Evaluation Setup

- Experimental Setup

  - ***Local cluster analysis*** and ***Large-scale simulation***

    > Our experiment combines scale-up simulations with experiments on a local testbed cluster.

- Workloads

  - Vision and NLP models
  - Three different types of production traces: ***sporadic***, ***periodic*** and ***bursty***

![production traces](images/INFless/INFless1681831609751.png)

- Comparision

![Comparision Systems](images/INFless/INFless1681976938171.png)

### Result

- High Throughput

![Normalized Throughput](images/INFless/INFless1681977543213.png)
![Throughput and Component Analysis](images/INFless/INFless1681977500014.png)

- Component analysis
  > Every component contributes to throuput improvement(batching -> most important)

- INFless tends to choose flexible configurations on batchsize and resource quota

![](images/INFless/INFless1681996287152.png)

- INFless reduces resource provisioning significantly

![](images/INFless/INFless1681996267395.png)

- INFless guarantees SLO

![](images/INFless/INFless1681996791621.png)

- Reduce Cold start rate by 20%

![](images/INFless/INFless1681997237777.png)

- INFless scales well in large-scale evaluations

![](images/INFless/INFless1681997267555.png)

- Reduces resource fragments significantly
- Cost Efficency