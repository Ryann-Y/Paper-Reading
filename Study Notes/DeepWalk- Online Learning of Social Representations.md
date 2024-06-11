**论文地址**：[https://arxiv.org/pdf/1403.6652](https://arxiv.org/pdf/1403.6652)

>学习网络中节点的潜在表示，这些表示可以将社交关系编码到一个连续向量空间中。该方法在一些社交网络的多标签网络分类任务中取得了非常好的效果。

**论文贡献**：
- 我们引入深度学习作为分析图形的工具，以构建适合统计建模的鲁棒表示。DeepWalk在短的随机游走中学习出现的结构规则。作为一种无监督学习方法，该方法可以独立于标签的分布来学习捕获图结构的特征。
- 我们广泛地评估了我们在几个社交网络上的多标签分类任务上的表示。在标签稀疏性的情况下，我们显示出显著提高的分类性能，在我们考虑的最稀疏的问题上，得到了Micro F1 5%-10%的改进。在某些情况下，即使训练数据少了60%，DeepWalk的表现也能胜过竞争对手。

**Random Walk**：
![[Pasted image 20240603170940.png|400]]

**策略收益**：
- 在稀疏标签的数据中，F1 相比基线提升 10%