**原文地址**：[https://arxiv.org/pdf/2007.16122](https://arxiv.org/pdf/2007.16122)

**名词解释**：
- COLD: **C**omputing power cost-aware **O**nline and **L**ightweight **D**eep pre-ranking system

![[Pasted image 20240513104441.png|600]]

**策略收益**：RPM +6%

**个人理解**：
1. 本文的主要收益来自于两点：
	1. 基线足够低：其中基线的部分更新频率较低，且user 向量并不是在线更新的；
	2. 工程优化：主要还是使用资源换效果。
2. 本文采用的筛选特征的方式也并非SENet的方式，而是将输入层直接输入单层神经网络，学习到每个field的weight，并未做压缩的过程。