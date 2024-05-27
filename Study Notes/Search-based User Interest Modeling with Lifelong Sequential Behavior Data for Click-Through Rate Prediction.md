**论文地址**：[https://arxiv.org/pdf/2006.05639](https://arxiv.org/pdf/2006.05639)

阿里妈妈广告团队

**问题背景**：
- 当用户行为序列的长度非常长的时候，建模用户兴趣变得非常困难，当前最先进的 MIMN 也只能将用户序列的长度规模扩展到 1000，对更长的序列是无法处理的；
- 23%的淘宝用户在过去5个月的时间里，会有超过1000的点击量。s

**名词解释**：
- SIM：**S**earch-based **I**nterest **M**odel，基于搜索的兴趣模型
- MIMN：**M**ulti-channel user **I**nterest **M**emory **N**etwork，多渠道用户兴趣记忆网络
- SBS: **S**ub user **B**ehavior **S**equence，用户行为子序列
- GSU：**G**eneral **S**earch **U**nit
- ESU：**E**xact **S**earch **U**nit

**论文贡献**：
1. 

**论文方案**：
1. 提出一种全新的建模范式，通过两个级联的搜索单元（GSU和ESU）来提取用户兴趣，处理序列长度 长达54000；
2. GSU从原始的长用户行为序列中，使用候选item作为query来进行检索，得到和候选item相关的用户行为子序列；
3. ESU精确的建模候选item和用户行为子序列的关系；

**策略收益**：
- CTR +7.1%，RPM +4.4%