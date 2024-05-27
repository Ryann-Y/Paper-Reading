**原文地址**：[https://arxiv.org/pdf/2006.11011](https://arxiv.org/pdf/2006.11011)

![[Pasted image 20240506130212.png|800]]

**论文贡献**：
1. 首次从 user 角度，显示拆分 用户兴趣 和 从众性 的embedding 表征，对热门 item 的 用户 conformity 行为 进行 “纠偏”；
2. 采用 结构因果模型（SCM）的 **对撞结构** 来刻画 推荐系统的用户交互行为，考虑 interest 和 conformity 对 click 的因果效应；
3. 构造 causal-specific 不等式 训练数据，分别对 interest 和 conformity 的 embedding 进行 pairwise 学习，来解决 ground-truth 缺乏的问题；
4. 采用 multi-task 和 curriculum 的训练方式来平衡 兴趣和从众性 (对 $\alpha$ 和 $m_{up}$ 及 $m_{down}$ 采用 decay, 论文中是 0.9)。

**Loss Function**：
- $L=L_{click}^{O_{1}+O_{2}}+\alpha(L_{interest}^{O_{2}}+L_{conformity}^{O_{1}+O_{2}})+\beta L_{discrepancy}$，$\alpha\;0.1,\beta\;0.01$

