# Wasserstein距离

## 简介

令$X \sim P, Y \sim Q$，并且概率密度分别为$p$和$q$，假设$X,Y \in \mathbb{R}^d$。有很多方法可以定义$P$和$Q$之间的距离：

$$
\text { 总变差(Total Variation) : } \sup _A|P(A)-Q(A)|=\frac{1}{2} \int|p-q|
$$

$$
\text { 海林格距离(Hellinger Distance) } : \sqrt{\int(\sqrt{p}-\sqrt{q})^2} 
$$

$$
L_2: \int(p-q)^2
$$

$$
\chi^2 : \int \frac{(p-q)^2}{q}
$$

这些距离的缺陷：

1. 不适用于一离散分布和一连续分布之间的距离比较
2. 以上这些距离都忽略了分布空间潜在的几何特征
![](images/Wasserstein距离/Wasserstein距离1669553411407.png)
3. 当对不同的对象（如分布或者图片）取平均时，理想的结果是得到一个相似的对象。
![](images/Wasserstein距离/Wasserstein距离1669553722915.png)
4. 当计算两个分布的一般距离时，通常只得到一个数字，而不能获知为什么这些分布是不同的。使用Wasserstein距离时可以得到
5. 当需要建立两个分布$P_0$和$P_1$的中间插值分布$P_t$时，理想的$P_t$是要保留原始分布的基本结构
![](images/Wasserstein距离/Wasserstein距离1669554170626.png)
6. 有一些距离对分布的小扰动是非常敏感的，但是Wasserstein距离对于小扰动是不敏感的。

## 最优运输问题

如果$T: \mathbb{R}_d \rightarrow \mathbb{R}_d$，则$T(X)$的分布被叫做the push-forward of $P$，用$T_\# P$表示：
$$
T_{\#} P(A)=P\left(\{x: T(x) \in A)=P\left(T^{-1}(A)\right)\right.
$$
