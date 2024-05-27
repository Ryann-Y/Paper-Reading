**论文地址**：[https://dl.acm.org/doi/pdf/10.1145/3209978.3210104](https://dl.acm.org/doi/pdf/10.1145/3209978.3210104)

**名词解释**：
- ESMM: **E**ntire **S**pace **M**ulti-task **M**odel
- SSB: sample selection bias, 样本选择偏差, CVR模型在点击样本训练，但在全域空间中进行预测。
- DS: data sparsity, 数据稀疏, CVR模型的训练样本远少于CTR模型

**论文方案**：
- 有效利用用户行为序列，将CVR预估问题转化为条件概率求解问题 $p(y=1,z=1|x)=p(y=1|x)\times p(z=1|y=1,x)$；
- 引入两个辅助任务 CTR 和 CTCVR，采用在全域空间隐式建模 CVR 的方式来替代 点击数据下的直接建模，缓解了 SSB 样本选择偏差问题；
- CTR和CVR tower底层参数共享，进而通过特征表示的迁移策略，来解决 DS 数据稀疏问题；
- 此外，论文采用 乘积 的方式 而非 相除 的方式来建模 任务之间的关系，避免 p(CTCVR) 和 p(CTR) 相除 可能带来的浮点数不稳定的问题，并且确保了 p(CVR) 在 $[0,1]$ 范围之内。

**Loss Function**：
$$
\displaystyle L(\theta_{cvr},\theta_{ctr})=\sum_{i=1}^{N}l(y_{i},f(x_{i};\theta_{ctr})) + \sum_{i=1}^{N}l(y_{i}\&z_{i},f(x_{i};\theta_{ctr})\times f(x_{i};\theta_{cvr}))
$$

![[Pasted image 20240515103041.png|400]]

**策略收益**：
- AUC gain of 2.18% on CVR task and 2.32% on CTCVR task.
