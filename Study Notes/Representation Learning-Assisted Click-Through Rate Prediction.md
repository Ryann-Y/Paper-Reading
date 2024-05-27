
**论文地址**：[https://arxiv.org/pdf/1906.04365](https://arxiv.org/pdf/1906.04365)

**名词解释**：
- DeepMCP: **Deep** **M**atching **C**orrelation **P**rediction Model, 主要由其三个组成部分的名称首字母构成；

**论文背景**：

**论文贡献**：
- 学习到的特征表示具有良好的预估效果和表达能力；

**论文思想**：
- 由于特征 embedding 随机初始化，当考虑预测任务时，用户的特征表示可能存在较大差异，这是由于模型并没有建模特征之间的关系，如果我们进一步考虑 user-ad 和 ad-ad 之间的关系进一步学习其表示，将有助于 pCTR的准确性。

**网络结构**：
![[Pasted image 20240516180429.png|800]]

DeepMCP由三部分组成：
- matching subnet: model user-ad 关系
- correlation subnet: model ad-ad 关系
- prediction subnet: model feature-CTR 关系

特别说一下 correlation subnet:
参考 skip-gram的方式，来学习ad表示，给定用户对广告的点击序列 $\{a_{1},a_{2},\dots,a_{L}\}$，最大化平均对数似然函数 $\displaystyle ll=\frac{1}{L}\sum_{i=1}^{L}\sum_{-C\leq j\leq C}^{1\leq i+j\leq L,j\neq 0}\log p(a_{i+j}\mid a_{i})$，其中 $L$ 表示序列的长度，$C$ 代表上下文窗口size。

**loss function**：
$$
loss=loss_{p}+\alpha loss_{m}+\beta loss_{c}
$$
其中 $loss_{p}$ 是 predict subnet 的 loss，$loss_{m}$ 是 matching subnet 的 loss，$loss_c$ 是 correlation subnet 的 loss

$$
\displaystyle
loss_{m}=-\frac{1}{\left|Y\right|}\sum_{y\in Y}\left[y(u,a)\log s(v_{u},v_{a})+(1-y(u,a))\log(1-s(v_{u},v_{a}))\right]
$$
$$
\displaystyle
loss_{c}=\frac{1}{L}\sum_{i=1}^{L}\sum_{-C\leq j\leq C}^{1\leq i+j\leq L,j\neq 0}\left[-\log\left[\sigma(h_{a_{i+j}}^{T}h_{a_{i}})\right]-\sum_{q=1}^{Q}\log\left[\sigma(-h_{a_{q}}^{T}h_{a_{i}})\right]\right]
$$


**策略收益**：
![[Pasted image 20240516193829.png|400]]