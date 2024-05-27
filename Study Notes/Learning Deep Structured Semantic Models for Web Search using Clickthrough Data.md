**背景**
- 基于关键词的匹配方式由于存在 **不同词汇的概念表达及语言风格的问题** 无法满足 matching 的需要。

**名词解释**
- DSSM: **D**eep **S**tructured **S**emantic **M**odels，深度结构化语义模型
- LSA: **L**atent **S**emantic **A**nalysis，潜在语义分析
- PLSA: **P**robabilistic **LSA**
- LDA: **L**atent **D**irichlet **A**llocation，潜在狄利克雷分配

**相关工作**
- 潜在语义模型：类似LSA/PLSA/LDA，通过将发生在相同上下文中的不同 terms 映射到 相同的语义聚类中，但通常是采用无监督的方式进行训练，和最终 召回任务的评估指标 松耦合。


**模型结构**

![[Pasted image 20240523104145.png|600]]

**Word Hashing**
- 由于输入词表的巨大，甚至可能是无限的，因此文中采用了 n-gram vector 的方式来表达每个单词，以 good 为例，可以表示为 4个 trigrams: \#go, goo, ood, od\#。
  问题：可能存在较小概率的冲突，词表越大，冲突概率越小
  优势：
  1. 词表越大，表示的压缩比例越大，比如 50w的词表，可以将 50w 维的稀疏向量 表示为 30621 维的 稠密向量。
  2. 能够解决 out-of-vocabulary 问题，当出现词表中未出现过的单词，可以使用 n-gram 很好的学习未登录词
  3. 可以学习到词语的组成形态，将类似形态的单词映射到相近的空间中。

$$
\displaystyle P(D\mid Q)=\frac{\exp(\gamma R(Q,D))}{\sum_{D'\in \text{D}}\exp(\gamma R(Q,D'))}
$$
其中，$\text{D}$ 表示候选文档的集合，理论上，应该包含所有可能的文档。

这里，我们为每个正例，选择**4**个负例以供学习。

**策略收益**：
- NDCG@1 +2.5-4.3%
