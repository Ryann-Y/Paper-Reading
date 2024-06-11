**论文地址**：[https://arxiv.org/pdf/2006.05639](https://arxiv.org/pdf/2006.05639)
**参考**：
- https://zhuanlan.zhihu.com/p/370951728
- https://zhuanlan.zhihu.com/p/148416488

阿里妈妈广告团队

**问题背景**：
- 当用户行为序列的长度非常长的时候，建模用户兴趣变得非常困难，当前最先进的 MIMN 也只能将用户序列的长度规模扩展到 1000，对更长的序列是无法处理的；
- 当用户序列长度达到10000甚至更多的时候，MIMN将用户历史行为编码到一个固定尺寸的记忆矩阵会导致记忆单元中存在大量的噪声，从而无法准确的捕捉用户的兴趣；
- 23%的淘宝用户在过去5个月的时间里，会有超过1000的点击量。

**名词解释**：
- SIM：**S**earch-based **I**nterest **M**odel，基于搜索的兴趣模型
- MIMN：**M**ulti-channel user **I**nterest **M**emory **N**etwork，多渠道用户兴趣记忆网络
- SBS: **S**ub user **B**ehavior **S**equence，用户行为子序列，长度可以在几百的量级
- GSU：**G**eneral **S**earch **U**nit
- ESU：**E**xact **S**earch **U**nit
- RTP：**R**eal-**T**ime **P**rediction
- UBT：**U**ser-**B**ehavior **T**ree

**论文贡献**：
1. 本文提出了一种 建模长序列用户行为数据 的 新范式 SIM，两阶段级联的搜索机制使得 SIM 可以在建模 生命周期级别序列行为数据上具有更好的扩展性和准确率；
2. 提供了相关的实践经验。自从 2019 年期，SIM 给 阿里巴巴带来 7.1% 的 CTR 和 4.4% 的 RPM 提升；
3. 把长序列的建模长度扩大到了 54000，54倍于 MIMN。

>受到 DIN 的思想，仅捕获和特定候选 item 相关的用户兴趣！

**模型结构**：
![[Pasted image 20240527183208.png|800]]

**论文方案**：
1. 提出一种全新的建模范式，通过两个级联的搜索单元（GSU和ESU）来提取用户兴趣，处理序列长度 长达54000；
2. GSU从原始的长用户行为序列中，使用候选item作为query来进行检索，得到和候选item相关的用户行为子序列；
3. ESU精确的建模候选item和用户行为子序列的关系；

**General Search Unit**
对于用户行为序列 $\text{B}=[\text{b}_{1};\text{b}_{2};\cdots;\text{b}_{T}]$，GSU 会对于每个行为 $\text{b}_{i}$ 基于候选 item 计算相关打分 $r_{i}$，找出 Top-K 个和候选结果相关的 用户行为子序列 $\text{B}^{*}$。

相关性打分 $r_{i}$：
$$
r_{i}=\begin{cases}
Sign(C_{i}=C_{a})&hard-search \\
(W_{b}\text{e}_{i})\odot(W_{a}\text{e}_{a})^{T}&soft-search
\end{cases}
$$


hard-search:
- Hard Search只会选择和 候选 item 相同类目的行为 $b_i$，然后组成行为子集，这种方法简单且高效。

soft-search:
- 采用辅助 CTR 任务的方式来生成 embedding 向量，其中，建模的序列采用 用户长期行为序列，当然如果用户行为大到一定规模，是没有办法直接将序列全部纳入到模型中训练的，这种情况下，可以采用同分布采样的方式采样子集进行训练。

>$W_{b}$ 和 $W_{a}$ 可以理解为 attention 中变换的参数矩阵，$W_{b}e_{i}$ 得到了 key，$W_{a}e_{a}$ 得到了 Query，然后再做相似度计算，而不是直接向量内积
>在soft search 模型中直接使用从短期用户兴趣建模中学到的参数可能会误导长期用户兴趣建模！！！

**Exact Search Unit**

考虑到这些子集跨越了相当长时间，对用户行为的贡献是不同的，因此这里引入了时间间隔信息 $\text{D}=[\Delta_{1};\Delta_{2};\dots;\Delta_{K}]$. $\text{B}^{*}$ 和 $\text{D}$ 被分别编码成 $\text{E}^{*}=[\text{e}_{1}^{*};\text{e}_{2}^{*};\dots;\text{e}_{K}^{*}]$ 和 $\text{E}_{t}=[\text{e}_{1}^{t};\text{e}_{2}^{t};\dots;\text{e}_{K}^{t}]$. 这两个表示被 concat 到一起来表示最终的用户行为序列表征 $\text{z}_{j}=\text{concat}(e_{j}^{*},e_{j}^{t})$. 采用多头注意力机制来捕获用户的多样性兴趣：
$$
\begin{align}
&\text{att}_{score}^{i}=\text{Softmax}(W_{bi}\text{z}_{b}\odot W_{ai}\text{e}_{a}) \\
&\text{head}_{i}=\text{att}_{score}^{i}\text{z}_{b}
\end{align}
$$

>这里 $W_{bi}$ 和 $W_{ai}$ 是第 i 个参数权重，即attention时采用的K的矩阵和Q的矩阵，最后的 用户长期兴趣 表示为 $U_{lt}=\text{concat}(\text{head}_{1};\dots;\text{head}_{q})$.

**Loss Function**：
$$
Loss = \alpha Loss_{GSU}+\beta Loss_{ESU}
$$
论文中，$\alpha$ 和 $\beta$ 都设置为1，当 GSU 使用 hard-search 的时候，$\alpha$ 为0.

>这里其实实验了添加timeinfo的 SIM，有一定提升。

**策略收益**：
- CTR +7.1%，RPM +4.4%