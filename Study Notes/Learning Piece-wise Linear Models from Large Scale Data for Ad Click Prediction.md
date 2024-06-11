**论文地址**：[https://arxiv.org/pdf/1704.05194](https://arxiv.org/pdf/1704.05194)

**名词解释**：
- LS-PLM：**L**arge **S**cale **P**iece-wise **L**inear **M**odel，大规模分段线性模型

**问题背景**：
- CTR 预估问题是典型的 非线性稀疏优化问题，为了帮助LR模型捕获非线性关系，需要大量的特征工程，非常耗时耗力；
- 像2014年Facebook提出的GBDT+LR，其中决策树扮演了非线性特征转换的角色，但决策树对于高维稀疏问题并不合适；
- 像FM模型可以捕获2维交叉特征，但无法适用于更多广泛的非线性模式。

**论文思想**：
- 采用分治策略：
	- 首先，将特征空间划分为多个局部区域；
	- 其次，在每个区域训练一个线性模型；
	- 最后，输出预估结果的线性加权。
- 模型优势：非线性、扩展性、稀疏性

模型分类超平面示意图：
![[Pasted image 20240529191523.png]]

**模型结构**：
![[Pasted image 20240529192737.png|400]]

**论文方案**：
1. 采用分段线性模型解决非线性问题，$\displaystyle p(y=1\mid x)=g\left( \sum_{j=1}^{m}\sigma(u_{j}^{T}x)\eta(w_{j}^{T}x) \right)$.
2. 其中 $\sigma(u_{j}^{T}x)$ 将特征空间划分为 m 个不同的区域，$\eta(w_{j}^{T}x)$ 给出每个区域中的线性预估结果。

>实验中，$\sigma(x)$ 采用的是 softmax，而 $\eta(x)$ 采用 sigmoid 函数，$g(x)=x$.
>实际的公式为：$\displaystyle p(y=1|x)=\sum_{i=1}^{m} \frac{\exp(u_{i}^{T}x)}{\sum_{j=1}^{m}\exp(u_{j}^{T}x)}\cdot \frac{1}{1+\exp(-w_{i}^{T}x)}$.

>数值最优化方法部分跳过！

**Loss Function**：
$$
\displaystyle 
\begin{align}
&\arg\min_{\theta}f(\theta)=loss(\theta)+\lambda \Vert\theta\Vert_{2,1}+\beta\Vert\theta\Vert_{1} \\
&\Vert\theta\Vert_{2,1}=\sum_{i=1}^{d}\sqrt{ \sum_{j=1}^{2m}\theta_{ij}^{2} } \\
&\Vert\theta\Vert_{1}=\sum_{ij}\left| \theta_{ij} \right|
\end{align}
$$

**实验设置**：
- m = 12，划分区域的个数
- 实验增加了 $L_{1}$ 和 $L_{2}$ 两个范数，证明都加的效果更好，最后，$\beta=1, \lambda=1$。

**策略收益**：
- AUC +1.44%