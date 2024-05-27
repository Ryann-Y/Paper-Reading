**名词解释**：
- BST: user **B**ehavior **S**equence **T**ransformer

**模型结构**：
![[Pasted image 20240524173826.png|800]]

>**注意**：论文中将 target item 也作为序列中的信息引入到 transformer中，同时通过 positional feature 来区分。

**positional encoding**:
$$
pos(v_{i})=t(v_{t})-t(v_{i})
$$
代表推荐时间差，论文尝试了这种方法比 sin 和 cos 效果好

**策略收益**：
![[Pasted image 20240524185558.png|400]]