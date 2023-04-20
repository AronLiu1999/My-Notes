# Device-cloud Collaborative Recommendation via Meta Controller

单位：上海交通大学、Alibaba

期刊/会议：KDD

时间：2022

## Abstract

推荐系统的端云协作范式往往是顺序机制，即在基于云的推荐器之上构建了在终端设备上的推荐器。但是这样的设计是不灵活的，当用户兴趣急剧变化的时候，设备上的模型受限于缓存而卡住，而云端模型在没有新的刷新反馈下是不会响应的。为此，作者提出一种元控制器来动态管理设备和云上recommender的协作，并针对数据集缺失问题，从因果角度引入了一种新的高效的样本构建方式。

## 1. Introduction

会话推荐：在一个会话中对前K个候选项进行排名，可以提高效率并减少云服务器的计算负担。

当session长度变为1的时候就变成了路径3，3的优点是实时性好，可以更加频繁地与用户交互，但是会过多消耗云端算力。

边缘计算的发展推动了模型部署在移动端设备。设备端推荐系统与基于云端的推荐结合，如路径2所示，云端先筛选出一些候选递交给本地模型，然后本地模型再根据实时的用户行为做出相应的排名。但是由于移动端存储的限制，本地缓存的候选项不能满足用户兴趣急剧变化。

![](images/Device-cloud%20Collaborative%20Recommendation%20via%20Meta%20Controller/Device-cloud%20Collaborative%20Recommendation%20via%20Meta%20Controller1669202606352.png)

三种方法各有优劣，可以互补：用户兴趣几乎不变，只需要plain session recommendation而无需唤醒on-device recommender进行排名；用户兴趣轻微变化，调用on-device model根据实时特征对缓存中对候选项重新排名；当用户兴趣剧烈变化，需要从云上的大规模item pool中选出更多的候选项。

## 2. Related Works

### 2.1 Cloud-Based Recommendation

### 2.2 On-Device Recommendation

### 2.3 Causal Inference

## 3. The Proposed Approach

给定一组固定长度的历史点击$\mathbf{H}_{\text {cloud }, i}$，即第$i$个用户在当前会话之前的交互，还有所有的side feature（如统计特征）$\mathbf{~S}_{\text {cloud },i}$，基于云的推荐是建立当前用户对第$j$个候选$\mathbf{X}_{j, i}$感兴趣的概率的模型：
$$
f_{\text {cloud }}\left(\mathbf{X}_{j, i} \mid \mathbf{H}_{\text {cloud }, i}, \mathbf{~S}_{\text {cloud }, i}\right)
$$

对于设备上的推荐，模型还有一些其他特征可以利用，比如当前会话中的实时交互和其他的细粒度特征，分别用$\mathbf{H}_{\text {device }, i}$和$\mathbf{~S}_{\text {device }, i}$表示（$\mathbf{H}_{\text {device }, i}=\mathbf{H}_{\text {cloud }, i} \cup \mathbf{H}_{\mathbf{add}, i}$）

$$
f_{\text {device }}\left(\mathbf{X}_{j, i} \mid \mathbf{H}_{\text {device }, i}, \mathbf{~S}_{\text {device }, i}\right)
$$

对于刷新推荐，它与基于云的推荐共享模型，但是调用时输入特征不同，这种机制的增益来自于特征而不是模型：

$$
f_{\text {cloud }}\left(\mathbf{X}_{j, i} \mid \mathbf{H}_{\text {device }, i}, \mathbf{~S}_{\text {cloud }, i}\right)
$$

### 3.1 Meta Controller

本文的目标是构建一个元控制器来管理上述三种机制的协作，$f_{\text {meta }}(\cdot)$表示元控制器模型，输入与$f_{\text {device }}(\cdot)$相同的特征，并且也是部署在设备上，假设$t = 0, 1, 2$分别表示调用图1中的三种推荐机制，所以本文希望元控制器模型可以做出选择以最小化下面这个问题：

$$
\min _{f_{\text {meta }}} \mathbb{E}_{(\mathbf{X}, \mathbf{Y}) \sim P_{\text {meta }}(\mathbf{X}, \mathbf{Y})}\left[\ell\left(\mathbf{Y}, \sum_{t=0}^2 f_{\text {meta }}(\mathbf{X}) f_t(\mathbf{X})\right)\right]
$$

!!! danger 数据集缺失，无法获取服从$P_{\text {meta }}(\mathbf{X}, \mathbf{Y})$的数据

### 3.2 用因果推理构建样本

收集三种不同场景的训练样本：

$$
\mathcal{D}_t= \begin{cases}\left\{\left(\mathbf{H}_{\text {cloud }}, \mathbf{O}(t)\right)\right\}_{\text {cloud }}, & t=0 \\ \left\{\left(\mathbf{H}_{\text {device }}, \mathbf{O}(t)\right)\right\}_{\text {device }}, & t=1 \\ \left\{\left(\mathbf{H}_{\text {device }}, \mathbf{O}(t)\right)\right\}_{\text {re-fresh }}, & t=2\end{cases}
$$

$\mathbf{O}(\cdot)$表示在$t$下用户是否点击接下来的$L$个物品$\left\{\left(x_l, y_l\right)\right\}_{l \in(1,2, \ldots, L)}$

有了这三个异构样本集，可以利用counterfactual inference的思想去比较这几种推荐机制的因果效应，然后去构建元控制器的训练样本。

$$
\text { CATE: }\left\{\begin{array}{l}
\tau_{\text {cloud }}(\mathbf{H})=0(\text { session recommendation as the base) } \\
\tau_{\text {device }}(\mathbf{H})=\mathbb{E}[\mathbf{O}(1) \mid \mathbf{H}]-\mathbb{E}[\mathbf{O}(0) \mid \mathbf{H}] \\
\tau_{\text {re-fresh }}(\mathbf{H})=\mathbb{E}[\mathbf{O}(2) \mid \mathbf{H}]-\mathbb{E}[\mathbf{O}(0) \mid \mathbf{H}]
\end{array}\right.
$$

将基于云的会话推荐作为对照组，可以通过T-learner或X-learner来估计剩余两种机制的CATE

![](images/Device-cloud%20Collaborative%20Recommendation%20via%20Meta%20Controller/Device-cloud%20Collaborative%20Recommendation%20via%20Meta%20Controller1669208715694.png)

base network是为对照组的数据$\mathcal{D}_0$建立baseline，形式上：

$$
g_{\text {cloud }}(\cdot)=g_{\text {base }}(\cdot) \circ \phi_{\text {seq }}(\cdot)
$$

$\phi_{\text {seq }}(\cdot)$是历史点击交互的编码器

$$
g_{\text {device }}(\cdot)=g_{\text {uplift1 }}(\cdot) \circ \phi_{\text {seq }}(\cdot)
$$

$$
g_{\text {re-fresh }}(\cdot)=g_{\text {uplift2 }}(\cdot) \circ \phi_{\text {seq }}(\cdot)
$$

所以可以通过解决如下的优化问题来学习这三个模型

$$
\left\{\begin{array}{l}
\min \frac{1}{\left|\mathcal{D}_0\right|} \sum_{(h, o) \in \mathcal{D}_0} \ell_b\left(o, g_{\text {cloud }}(h)\right) \\
\min \frac{1}{\left|\mathcal{D}_1\right|} \sum_{(h, o) \in \mathcal{D}_1} \ell_t^1\left(o, g_{\text {device }}(h)\right) \\
\min \frac{1}{\left|\mathcal{D}_2\right|} \sum_{(h, o) \in \mathcal{D}_2} \ell_t^2\left(o, g_{\text {re-fresh }}(h)\right)
\end{array}\right.
$$

得到这三个模型后，可以通过比较每个样本在三种机制下的CATE来获得一个新的counterfactual training dataset

$$
\mathcal{D}_{\text {meta }}=\left\{(h, c) \mid c=\operatorname{IND}\left(\tau_{\text {cloud }}(h), \tau_{\text {device }}(h), \tau_{\text {re-fresh }}(h)\right),\right. \\
\left.\forall(h, o) \in \mathcal{D}_0 \cup \mathcal{D}_1 \cup \mathcal{D}_2\right\}
$$

### 3.3 训练元控制器

单标签多分类：

$$
\min _{f_{\text {meta }}} \frac{1}{\left|\mathcal{D}_{\text {meta }}\right|} \sum_{(h, c) \in \mathcal{D}_{\text {meta }}} \ell\left(c, f_{\text {meta }}(h)\right)
$$

标签平滑化方法，给定基于云的刷新的最大调用率$\epsilon$，

$$
\min _{f_{\text {meta }}} \frac{1}{\left|\mathcal{D}_{\text {meta }}\right|} \sum_{(h, c) \in \mathcal{D}_{\text {meta }}} \ell\left(\hat{c}, f_{\text {meta }}(h)\right),
$$
where $\hat{c}= \begin{cases}(1-\lambda) c+\lambda c^{-}, & c=[0,0,1]^{\top} \& \text { over budget } \epsilon \\ c, & \text { Default }\end{cases}$

使用$\hat{c}$来代替$c$用来训练模型，当预测batch的比率超过$\epsilon$时，$\hat{c}$就被平滑化，$c^-$对应$f_{\text {meta }}(h)$预测的第二大one-hot向量。训练至调用率小于$\epsilon$，得到最终的元控制器

![](images/Device-cloud%20Collaborative%20Recommendation%20via%20Meta%20Controller/Device-cloud%20Collaborative%20Recommendation%20via%20Meta%20Controller1669211393317.png)

## 4. Experiments

!!! question RQ1：与之前的三种机制相比，所提出的方法是否可以帮助训练元控制器通过端云协作实现更好的性能？（Meta Controller有效性）

!!! question RQ2：提出的方法是否能在标签平滑化的方法下满足实际的资源约束？（标签平滑化的有效性）

!!! question RQ3：三种推荐机制退化成任意两种会怎样？

### 4.1 实验设置

#### 4.1.1 数据集

两个公共数据集（Movienlens和Amazon）+ Alipay

![](images/Device-cloud%20Collaborative%20Recommendation%20via%20Meta%20Controller/Device-cloud%20Collaborative%20Recommendation%20via%20Meta%20Controller1669199592015.png)

每个数据集包含五个训练集（训练云推荐器、本地推荐器的两个外加三个训练meta-controller），每个数据集的测试集分为两组，一组用于测试CTR，一组用于测试meta-controller的CATE

#### 4.1.2 Baseline

使用DIN作为backbone，三种机制相对应的模型——CRec、CRRec、ORec，本文的方法MCRec。此外，还有个随机混合三种机制的baseline——RMixRec。

#### 4.1.3 Evauation Protocols

数据集构建阶段的评价使用AUUC(Area Under Uplift Curve)和QINI

$$
\begin{aligned}
&\mathrm{AUUC}=\frac{1}{2 N} \sum_{n=1}^N[p(n) \operatorname{Diff}(n)-p(n-1) \operatorname{Diff}(n-1)] \\
&\mathrm{QINI}=\mathrm{AUUC}-\mathrm{AUUC}_{\text {random }}
\end{aligned}
$$

$\mathcal{U}$表示用户集，$\mathbb{I}(\cdot)$是指示函数，$R_{u , g_u}$是模型为用户$u$生成的对于$g_u$的排名，$f$是待评估的模型，$D^{(u)}_T , D^{(u)}_F$分别为测试数据中的正负样本集合。

推荐性能的评价使用HitRate，AUC，NDCG(Normalized Discounted Cumulative Gain)：

$$
\begin{aligned}
&\text { HitRate@K }=\frac{1}{|\mathcal{U}|} \sum_{u \in \mathcal{U}} \mathbb{I}\left(R_{u, g_u} \leq K\right) \\
&\text { NDCG@K }=\sum_{u \in \mathcal{U}} \frac{1}{|\mathcal{U}|} \frac{2^{\mathbb{I}\left(R_{u, g_u} \leq K\right)}-1}{\log _2\left(\mathbb{I}\left(R_{u, g_u} \leq K\right)+1\right)}, \\
&\mathrm{AUC}=\sum_{u \in \mathcal{U}} \frac{1}{|\mathcal{U}|} \frac{\sum_{x_0 \in D_T^{(u)}} \sum_{x_1 \in D_F^{(u)}} \mathbb{I}\left[f\left(x_1\right)<f\left(x_0\right)\right]}{\left|D_T^{(u)}\right|\left|D_F^{(u)}\right|}
\end{aligned}
$$

### 4.2 实验结果和分析

#### RQ1:

![](images/Device-cloud%20Collaborative%20Recommendation%20via%20Meta%20Controller/Device-cloud%20Collaborative%20Recommendation%20via%20Meta%20Controller1669214369864.png)

图三表明，设备端推荐和基于云的刷新都在普通会话推荐的基础上实现了正增益，说明实时反馈对于提升推荐性能很重要。最左边的图表明设备上的推荐和云刷新的优劣是不确定的

![](images/Device-cloud%20Collaborative%20Recommendation%20via%20Meta%20Controller/Device-cloud%20Collaborative%20Recommendation%20via%20Meta%20Controller1669215118589.png)

![](images/Device-cloud%20Collaborative%20Recommendation%20via%20Meta%20Controller/Device-cloud%20Collaborative%20Recommendation%20via%20Meta%20Controller1669214914469.png)

表三中ORec和CRRec的AUUC和QINI是正的，表明了这两种treatment增加了会话推荐的长期增益。元控制器的目标就是区分它们的不同优点并调用合适的推荐器。

表二中CTR，CRec往往比ORec和CRRec差，而MCRec通过三种方式协作取得了较大的性能提升，而RMixRec效果提升不明显。

#### RQ2:

![](images/Device-cloud%20Collaborative%20Recommendation%20via%20Meta%20Controller/Device-cloud%20Collaborative%20Recommendation%20via%20Meta%20Controller1669215555728.png)

表二显示CTR与$\epsilon$正相关

#### RQ3:

![](images/Device-cloud%20Collaborative%20Recommendation%20via%20Meta%20Controller/Device-cloud%20Collaborative%20Recommendation%20via%20Meta%20Controller1669216393516.png)

设备上推荐器和基于云的刷新的组合实现了最佳性能，再次证明实时特征对于提高最终模型的性能很重要

还研究了超参数$\lambda$在不同约束$\epsilon$下控制调用率的效果，较大的$\lambda$带来了更多的对于云刷新的惩罚

![](images/Device-cloud%20Collaborative%20Recommendation%20via%20Meta%20Controller/Device-cloud%20Collaborative%20Recommendation%20via%20Meta%20Controller1669216713877.png)

![](images/Device-cloud%20Collaborative%20Recommendation%20via%20Meta%20Controller/Device-cloud%20Collaborative%20Recommendation%20via%20Meta%20Controller1669217040229.png)