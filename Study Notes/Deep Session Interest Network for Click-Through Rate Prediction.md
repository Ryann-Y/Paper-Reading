**论文地址**：[https://arxiv.org/pdf/1905.06482](https://arxiv.org/pdf/1905.06482)
**源码地址**：[https://github.com/shenweichen/DSIN](https://github.com/shenweichen/DSIN)

**名词解释**：
- DSIN：**D**eep **S**ession **I**nterest **N**etwork，深度会话兴趣网络

**问题背景**：
- 当前的主流模型主要通过从用户的行为序列中挖掘用户的动态且进化的兴趣，但都忽略了用户行为序列**内在的结构**（序列是由session构成）；
- 用户session内部的行为都是高度**同质**的，而跨session的行为则是**异构**的。

DSIN 被提出来提取用户 session 粒度的兴趣，然后捕获 session 兴趣的序列关系！

**论文贡献**：
- 提出用户行为在 session 内部高度同质，跨 session 异构的现象，并提出全新的 可以高效建模用户多 session 兴趣的 CTR 预估模型 DSIN；
- 设计了带有 bias 编码的自注意力机制来提取每个 session 准确的兴趣表示，然后采用 BI-LSTM 来捕获历史 session 中的序列关系，最后利用局部激活单元来聚合不同 session 兴趣对目标 item 的影响；
- 两组分别在广告和商品推荐的数据上进行的实验证明了提出方案的优越性。

**模型结构**：
![[Pasted image 20240529102219.png]]

**论文方案**：
- 首先，采用带有bias编码的自注意力机制 self-attention with bias encoding来提取 用户在每个session 中的兴趣；
- 然后，应用 BI-LSTM来建模用户兴趣的进化 和 跨session 的互动；
- 最后，采用局部激活单元自适应学习不同会话兴趣对目标项目的影响。

模型分为四层：
1. Session Division Layer：会话切分层，将用户兴趣切分成间隔30min的 sessions；
2. Session Interest Extractor Layer：会话兴趣提取层，使用self-attention 提取用户的会话兴趣；
3. Session Interest Interacting Layer：会话兴趣交互层，使用 BI-LSTM 捕获会话兴趣间的序列关系；
4. Session Interest Activating Layer：会话兴趣激活层，使用 Local Attention Unit 激活兴趣。

会话兴趣提取层：
- session内部的兴趣是高度相关的，此外会话内部的随意行为也会使会话的兴趣偏离原始的表达；
- 因此，捕捉行为之中的内在关系，降低不相关行为的影响，在每个 session 中采用 多头自注意力机制；
- **positional encoding** $\text{BE}\in R^{K\times T\times d_{model}}$
	- $\text{BE}_{(k,t,c)}=\text{w}_{k}^{K}+\text{w}_{t}^{T}+\text{w}_{c}^{C}$；
	- $\text{w}^{K}\in R^{K}$ 是 session 的 bias vector，$k$ 是 session 的下标；
	- $\text{w}^{T}\in R^{T}$ 是 位置 的 bias vector，$t$ 是 session 内 行为的下标；
	- $\text{w}^{C}\in R^{d_{model}}$ 是 行为 embedding 中的 unit position

会话兴趣交互层：
- 使用 BI-LSTM 建模 session 兴趣之间的序列关系

会画兴趣激活层：
- $\displaystyle a_{k}^{I}=\frac{\exp(\text{I}_{k}\text{W}^{I}\text{X}^{I})}{\sum_{k}^{K}\exp(\text{I}_{k}\text{W}^{I}\text{X}^{I})}$，$\displaystyle \text{U}^{I}=\sum_{k}^{K}a_{k}^{I}\text{I}_{k}$，这里仅对 Q 进行了线性变换。

>Similarly, the adap- tive representation of session interests mixed with contextual information w.r.t. the target item is calculated as follows.
- $\displaystyle a_{k}^{H}=\frac{\exp(\text{H}_{k}\text{W}^{H}\text{X}^{I})}{\sum_{k}^{K}\exp(\text{H}_{k}\text{W}^{H}\text{X}^{I})}$，$\displaystyle \text{U}^{H}=\sum_{k}^{K}a_{k}^{H}\text{H}_{k}$.

**可视化**
![[Pasted image 20240529115600.png]]

**策略收益**：
![[Pasted image 20240529115812.png|400]]

**思考**：
>1. 文中提到的 Multi-head self-attention 实际上使用的正是 transformer 来提取序列关系，得到 session 的兴趣表达
>2. 在建模 session 兴趣之间的 序列关系时，不同于 DIEN 使用的 GRU，这里使用 BI-LSTM
>3. 在激活层，并不像 DIEN 使用 AUGRU 来影响更新门，而是使用 常规的 attention 机制，但是将 session 兴趣 和 提取后的兴趣均 激活，送入 MLP