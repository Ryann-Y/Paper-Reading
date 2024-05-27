**论文地址**：[https://dl.acm.org/doi/abs/10.1145/3383313.3412236](https://dl.acm.org/doi/abs/10.1145/3383313.3412236)

**名词解释**：
- CGC: **C**ustomized **G**ate **C**ontrol (CGC)
- PLE: **P**rogressive **L**ayered **E**xtraction, 逐级分层提取
- Negative Transfer: 负迁移问题，推荐系统中的任务通常是低相关甚至是相互冲突的，联合训练可能导致性能下降，称之为负迁移。简单点说负迁移是指多任务学习的效果不如单独训练各个任务的效果，共同训练模型导致任务的效果下降。
- Seesaw Phenomenon: 跷跷板现象，即一个任务的性能往往会通过损害其他任务的性能而得到改善。当任务之间出现比较复杂的关系或者有样本之间的依赖关系时。目前的MTL模型很难同时在所有任务上超过single-task模型。

![[Pasted image 20240514141228.png|800]]

![[Pasted image 20240514143603.png|400]]

**论文贡献**：
1. 指出当前MTL的主流模型主要集中于解决负迁移问题，而工业界多个目标之间也可能存在 “跷跷板” 问题，却常被忽略；
2. MMOE忽略了专家网络之间的区别和互动，无法捕捉复杂的任务相关性，对某些任务引入了有损的噪音（区别），且限制了联合优化的效果（交互）；
3. 提出了一种新的网络结构 PLE，来捕获复杂的任务相关性，显式的分离共享和任务独占的专家网络（区别），并采用多层专家和门控的方式提取出深层知识（交互）。虽然MMOE存在优化到PLE的理论可能，但实际很难收敛到这些极值点；
4. 引入新的MTL的Loss优化方式，对不同task的Loss采取归一化的方式，对task-loss的weight采用动态权重替换静态权重。

>PLE相比CGC不同任务的参数并不是在早期被分离，而是随着层级的加深，逐步分离。
>通过多层专家和门控网络，PLE对每个任务提取和组合更深层次的语义表示，以提高泛化能力。

**Loss Function**：
$$
\displaystyle
L(\theta_{1},\dots,\theta_{K},\theta_{s})=\sum_{k=1}^{K}\omega_{k}L_{k}(\theta_{k},\theta_{s})
$$
$$
L_{k}(\theta_{k},\theta_{s})=\frac{1}{\sum_{i}\delta_{k}^{i}}\sum_{i}\delta_{k}^{i}\textit{loss}_{k}(\hat{y}_{k}^{i}(\theta_{k},\theta_{s}),y_{k}^{i})
$$
$$
\displaystyle \omega_{k}^{(t)}=\omega_{k,0}\times \gamma_{k}^{t}
$$
其中，$\theta_{s}$ 表示共享参数，$K$ 是 tasks 的个数，$L_{k},\omega_{k}$ 和 $\theta_{k}$ 是 task k的 loss，loss权重和对应的参数；$\delta_{k}^{i}\in\{0,1\}$ 表示 sample i是否在任务k的样本空间内；$\omega_{k,0}$ 表示任务 k 的初始 loss weight。

![[Pasted image 20240514144558.png|400]]

**策略收益**：
- video recommendation: vv +2.23%, watch time +1.84%.

| ![[Pasted image 20240514161439.png\|400]] | ![[Pasted image 20240514162427.png\|500]] |
| ----------------------------------------- | ----------------------------------------- |

