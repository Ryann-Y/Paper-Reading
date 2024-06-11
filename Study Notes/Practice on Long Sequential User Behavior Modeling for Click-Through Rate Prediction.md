**论文地址**：[https://arxiv.org/pdf/1905.09248](https://arxiv.org/pdf/1905.09248)
**源码地址**：[https://github.com/UIC-Paper/MIMN](https://github.com/UIC-Paper/MIMN)

来自阿里妈妈展示广告团队

**名词解释**：
- MIMN：**M**ulti-channel user **I**nterest **M**emory **N**etwork，多通道用户兴趣记忆网络。受到 NTM的启发，改进原始的 NTM架构通过 memory utilization regularization 和 memory induction unit。
- UIC：**U**ser **I**nterest **C**enter，用户兴趣中心，latency free，和主模型解耦，用于提供用户长期兴趣；
- NTM：Neual Turing Machine，神经图灵机
- MUR：**M**emory **U**tilization **R**egularization
- MIU：**M**emory **I**nduction **U**nit

**论文方案**：
- 从系统服务角度看：
	- 将最消耗资源的用户兴趣模块从整体模型中解耦出来，设计了一个名为 UIC 的 用户兴趣中心模块。使用流式数据更新，而不依赖在线流量，因此是 latency free的；
- 从机器学习算法角度看：
	- 提出了一个新的 基于记忆的模型结构，命名为 MIMN 来从用户长序列行为数据中捕获用户兴趣。
- 注意：本方案中，用户的行为数据不会存储，只会更新 UIC 中的兴趣表示。

**系统架构**：
![[Pasted image 20240530165741.png|1000]]

**模型结构**：
![[Pasted image 20240530170000.png|1000]]

**Memory Network**：
- 记忆网络被提出 使用外部记忆组件来提取知识，这个想法被广泛的应用于 NLP中，例如问答系统。



**MUR: Memory Utilization Regularization**：
由于每个用户的兴趣强度不均匀，并且内存的初始化是随机的，因此在基本NTM模型中，存储的利用是不平衡的。这个问题会损害记忆的学习，使其不足以利用有限的内存存储。

**MIU: Memory Induction Unit**：
具有记忆感应单元的MIMN通过从基本NTM诱导记忆，能够捕获高阶信息并带来更多改进。

![[Pasted image 20240530183519.png|600]]

**策略收益**：
- 相比 DIEN auc +1%，CTR +7.5%，RPM +6%.









