**原文地址**：[https://arxiv.org/pdf/2209.06053](https://arxiv.org/pdf/2209.06053)
**论文解读**：[学习笔记](obsidian://open?vault=Ryann&file=07.%20Recommender%20System%2F07.%20Overfitting%2FOne-epoch%20Overfitting)

![[Pasted image 20240510144555.png|500]]

**论文贡献**：
1. 验证发现 **模型结构**，具有快速收敛率的 **优化算法** 和 **特征稀疏** 性 与 one-epoch现象密切相关；
2. 提出解释One-Epoch现象的 **假设**：训练集和测试集的 概率分布 $\mathcal{D}(\textit{EMB}(x), y)$ 不同，而MLP层可以在第二个epoch开始时迅速收敛于经验分布 $\mathcal{D}(\textit{EMB}(x_{trained}), y)$，导致了one-epoch现象。
   论文同时针对 发生与未发生 one-epoch 的情况和不同粒度的特征进行分析，验证了 $\mathcal{D}(\textit{EMB}(x_{trained}), y)$ 和 $\mathcal{D}(\textit{EMB}(x_{untrained}), y)$ 的差异。对比的指标是 $\mathcal{A}(\mathcal{D(\text{+},\text{-})})=\mathcal{A}(\mathcal{D}(\textit{EMB}(x), y=0),\mathcal{D}(\textit{EMB}(x), y=1))$

**收获**：
1. Baidu 的 Abacus 训练两阶段，通过 join 阶段预训练 embedding 参数，再通过 update 阶段 做整体的训练，是否从某种程度上 相较于 端到端训练 缓解了 embedding 欠拟合的问题？