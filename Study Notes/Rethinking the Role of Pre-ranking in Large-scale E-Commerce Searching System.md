
**论文地址**：[https://arxiv.org/pdf/2305.13647](https://arxiv.org/pdf/2305.13647)
**参考链接**：
- [https://zhuanlan.zhihu.com/p/641425090](https://zhuanlan.zhihu.com/p/641425090)
- [https://zhuanlan.zhihu.com/p/587353144](https://zhuanlan.zhihu.com/p/587353144)

**场景**：淘宝搜索粗排

>**Highlight**
>**粗排的上限不是精排，粗排有自己的最优解，盲目模仿精排严重降低了粗排天花板**
>粗排输出是一个set，对于粗排的公平评价是需要找到一个评价指标评测这个set的整体质量，全域hitrate是一个目前我们能找到的比较合理的指标
>粗排与精排既存在竞争又存在配合

**名词解释**：
- ASMOL：All-Scenario based Multi-Objective Learning framework，基于多目标任务的全场景学习框架
- ASH/ASPH：All-Scenario Hitrate，全域命中率
- ISPH: In-Scenario Purchase Hitrate，搜索场景内的购买命中率

**问题描述**：
- 对于粗排模型，AUC的评价指标往往和在线实验的结果不一致，尽管盲目的让粗排模型像精排一样，由于粗排存在很严重的 SSB 问题，一些优化也仅仅可以在短期内取得收益，而会导致严重的马太效应并且从系统的长期视角是不利的。

**论文思路**：
1. 粗排模型不赢盲目的模仿精排模型，其主要任务是返回一个合适的 **无序集合** 而非item的有序列表；
2. ASH更适合来评估粗排模型的效果

粗排系统的目标：
- High quality set: 通过解决粗排候选的 SSB 问题来输出高质量的候选集
- High Quality rank: 和精排保证输出一致性

**论文贡献**：
- 为粗排提出了一种新的被称为全域命中率 ASH 的评估指标，其在离线实验和在线A/B的大量试验中被证明是有效且一致的；
- 提出了一种全新的基于多目标的全场景学习框架，可以很好的提升 ASH，甚至在返回数千item时，效果比精排模型还好；

**论文方案**：
**评估指标：ASH**
- 文中提出，Hitrate@k之所以在召回中可以采用，在粗排不行，是由于召回层面是多路召回，而粗排阶段返回的结果一定是粗排结果选出来的，因此 $\displaystyle hitrate\text{@}k=\frac{\sum_{i=1}^{k}1(p_{i}\in T)}{\left|T\right|}\equiv 1$。
- 为了解决粗排层面hitrate恒为1的问题，论文引入了非搜索场景的正例数据，为了解决这些正例无query的情况，论文做了匹配，匹配到一个query，组成正例

**模型结构**：
![[Pasted image 20240517161713.png|800]]

**样本构造**：
1. Exposures 曝光样本，最终曝光出来的样本；
2. Ranking Candidates (RC)：排序候选样本，通过pre-ranking粗选，提交给精排排序，但未成功曝光的样本；
3. Pre-Ranking Candidates (PRC)：由召回提供给粗排进行打分但最终未提交精排的结果。

一次query 的 RC是10条，PRC 是40条

**模型目标**：
1. All-Scenario Purchase Label (ASPL)：全域购买标记，只要用户最终购买了商品，标记就是1，否则是0
2. All-Scenario Click Label (ASCL)：只要ASPL为1，则ASCL为1，全场景点击标记；
3. Adaptive Exposure Label (AEL)：是否在当前请求下曝光。当ASCL为1时，AEL强制为1.

![[Pasted image 20240517161735.png|1000]]

![[Pasted image 20240517180834.png|400]]

**Loss Function**：
$$
L=L_{rank}+L_{distill}
$$
其中采用一个全新的 list-wise loss，期望学习到重要性的序：purchased items > clicked but non-purchased items > non-click exposures > RC and PRC

$$
L_{rank}=\alpha_{ex}L_{exposure}+\alpha_{cl}L_{click}+\alpha_{pur}L_{purchase}
$$
List-wise ranking loss:
$$\displaystyle L_{purchase}=\sum_{i \in D}-\log \frac{\exp z_{i}}{\sum_{j\in S}\exp z_{j}}$$
其中 $z$ 是 logit，$S$ 是全训练样本集，$D$ 是正例样本集

![[Pasted image 20240517175316.png|300]]

![[Pasted image 20240517175535.png|300]]

**策略收益**：
- GMV+1.2% on Taobao Search

![[Pasted image 20240517172454.png|400]]

**思考**：
1. 为什么还要强调粗排的两个目标？既然确定粗排要返回一个集合，那么为什么还要保留rank能力？
2. Hitrate@k 的解释太牵强了。。即便对于多路召回，$\left|T\right|$ 是曝光后的全部点击样本，如何证明在反事实空间中，召回的未曝光样本不好呢？而ASH的指标需要强依赖搜索推荐系统，用全局数据来衡量单场景的效果。原文的说法是：we introduce more positives from other scenarios in Taobao. 这又带来了搜索和推荐有无query的差异问题。
3. 整体的方案像是技术的堆叠，新意不够