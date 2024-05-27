论文链接：[[2019][SENet] Squeeze-and-Excitation Networks](obsidian://open?vault=Ryann&file=10.%20Paper%20Reading%2FCV%2F%5B2019%5D%5BSENet%5D%20Squeeze-and-Excitation%20Networks.pdf)
代码实现：[https://github.com/hujie-frank/SENet](https://github.com/hujie-frank/SENet)


**SENet** 由 **SE block** (Squeeze-and-Excitation Blocks) 组成，主要是用来描述图像处理中 **多通道依赖关系** 的，本质是一个轻量级的 **门控机制**，可以理解为是对图像多通道数据进行 **attention** 处理，原文的描述是 **perform dynamic channel-wise feature recalibration**。

>理解：SENet是一种应用于图像处理的新型网络结构，基于CNN结构，通过对 **特征通道间的相关性进行建模，对重要特征进行强化来提升模型准确率**，本质上是针对CNN中间层卷积核特征的Attention操作。获得2017 ILSVR竞赛的冠军。

注：**个人理解**，$F_{tr}$ 常见为 卷积操作 (Convolution operation)，而传统的卷积操作之后在输入给上层表示 默认各通道之间的重要性是一样的，即权重都为1.
SENet 的创新点即通过全局信息为每个渠道 生成了一个 **attention weight** 实现了不同渠道的重要性区分，这也是推荐系统中使用这个权重作为特征重要性的原因。

![[SEBlock.png]]


**Squeeze**：挤压（压缩）采用全局平均池化，将 H * W 维度的通道数据压缩为1维，其中 $z\in R^{C}$ 是对 $U$ 进行 Squeeze，即 $F_{sq}(\cdot)$ 之后得到的，$z_{c}$ 即代表第 $c$ 个渠道的描述符，$z$ 可以理解为多个渠道的 **局部描述符** 的集合，通过它即可表达整个图像。

![[Pasted image 20240402181450.png]]

**Excitation**：激发，将 Squeeze 生成的 $C$ 维渠道描述符，通过 2层 MLP 进行 **非线性变换**，得到综合全局信息的 **attention weight**，第一层使用 Relu激活，第二层使用 Sigmoid.

![[Pasted image 20240402182400.png]]

这里，$\delta$ 表示 $ReLU$ 激活函数，$W_{1}\in R^{\frac{C}{r}\times C}$，$W_{2}\in R^{C\times \frac{C}{r}}$。$r$ 作为 reduction ratio.

**简图**：
![[Pasted image 20240402183049.png]]

**Discussion**：
1. **为什么采用 2 层MLP**：1层MLP非线性能力不足，需要2层MLP来综合全局信息，增加非线性能力；
2. **为什么使用sigmoid**：第2层输出的权重是 **attention weight**，使值在 0-1 之间更符合直觉；
3. **为什么使用reduction ratio**：猜测渠道见的关系不复杂，使用 reduction layer即可，论文也对比了不同 reduction ratio的结果，4和16并无太大差异。