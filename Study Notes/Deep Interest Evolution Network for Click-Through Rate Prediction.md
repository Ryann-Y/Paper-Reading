**论文地址**：[https://aaai.org/ojs/index.php/AAAI/article/view/4545/4423](https://aaai.org/ojs/index.php/AAAI/article/view/4545/4423)
**源码地址**：[https://github.com/mouna99/dien](https://github.com/mouna99/dien)

**名词解释**：
- DIEN: **D**eep **I**nterest **E**volution **N**etwork，深度兴趣进化网络
- AUGRU: GRU with **A**ttentional **U**pdate gate，具有注意力更新门的GRU

**问题背景**：
- 考虑到外部环境的变化和内部认知，用户兴趣是随着时间动态进化的，然而目前的 CTR 建模方法都直接讲用户行为的表征当做兴趣，而兴趣很难被显式行为所反映，缺乏会背后潜在兴趣的建模。

**论文贡献**：
1. 关注电商领域的兴趣进化现象，提出新的网络结构 DIEN 来建模兴趣进化过程，提升CTR预估的精准度；
2. 和直接采用行为作为兴趣表征不同，我们设计了用户提取层，针对隐层对兴趣表征弱的问题，引入辅助loss，来监督隐藏状态的学习，使得隐层对表征潜在兴趣更具表现力；
3. 新颖的设计了兴趣进化网络，使用带有注意力更新门的GRU来加强相关兴趣对目标item的影响，克服了兴趣漂移的问题。

**模型结构**：
![[Pasted image 20240527113621.png|800]]

**论文方案**：
1. DIEN 由两个关键模块所组成，兴趣提取层 用于从显式用户行为中提取潜在的 **即时兴趣**，另一个用于建模 **兴趣进化** 过程。
2. 兴趣提取层（Interest Extractor Layer）
	1. 采用GRU来建模兴趣之间的依赖；
	2. Auxiliary Loss: 遵循兴趣导致直接行为的原则，在兴趣提取层，使用下一行为 来监督当前隐层的学习（这类隐层我们称为 **兴趣状态**），有助于捕获更多的语义用于兴趣表示，并推动GRU的隐藏状态来有效地表示兴趣；
3. 兴趣漂移现象：用户兴趣在邻近的访问中可能是非常不同的，用户的行为可能依赖用户很久之前的行为。
4. 兴趣进化层（Interest Evolution Layer）
	1. 基于从兴趣提取层获得的兴趣序列，设计了具有注意力更新门的GRU称为 AUGRU。通过兴趣状态和 目标item 的相关性计算，在兴趣进化过程中，AUGRU加强相关兴趣的影响，同时减弱由兴趣漂移带来的非相关兴趣的影响。通过注意力机制的引入，AUGRU可以演化出不同目标item下特定的兴趣进化过程。


**兴趣提取层**：
$$
\begin{align}
&\text{u}_{t}=\sigma(W^{u}\text{i}_{t}+U^{u}\text{h}_{t-1}+\text{b}^{u}) \\
&\text{r}_{t}=\sigma(W^{r}\text{i}_{t}+U^{r}\text{h}_{t-1}+\text{b}^{r}) \\
&\text{\~h}_{t}=\tanh(W^{h}\text{i}_{t}+\text{r}_{t}\circ U^{h}\text{h}_{t-1}+\text{b}^{h}) \\
&\text{h}_{t}=(\text{1}-\text{u}_{t})\circ \text{h}_{t-1}+\text{u}_{t}\circ \text{\~h}_{t}
\end{align}
$$

**Auxiliary Loss**：
利用行为 $\text{b}_{t+1}$ 来监督兴趣状态 $\text{h}_{t}$ 的学习，负样本通过采样获得。假设有 $N$ 对 行为 embedding 序列 $\{\text{e}_{b}^{i}, \hat{\text{e}}_{b}^{i}\} \in D_{B}, i \in 1,2,\dots,N$ 其中 $\text{e}_{b}^{i}$ 表示点击行为序列，$\hat{\text{e}}_{b}^{i}$ 表示负采样序列，$T$ 表示序列行为的数量，$\text{e}_{b}^{i}[t]$ 代表第 t 次 用户点击的 item embedding 向量，$\hat{\text{e}}_{b}^{i}[t]$ 代表从除了 用户 $i$ 在第 $t$ 步点击 item 以外的 item构成的集合中 采样到的 embedding
$$
\displaystyle L_{aux}=-\frac{1}{N}\left( \sum_{i=1}^{N} \sum_{t}\log \sigma(\text{h}_{t},\text{e}_{b}^{i}[t+1]) + \log(1-\sigma(\text{h}_{t},\hat{\text{e}}_{b}^{i}[t+1])) \right), \sigma(\text{x}_{1},\text{x}_{2})=\frac{1}{1+\exp(-[\text{x}_{1},\text{x}_{2}])}
$$

优势：
1. 从兴趣学习的视角看，辅助 loss 的引入可以帮助 GRU 隐状态更好的学习兴趣表达；
2. 从GRU优化的角度看，辅助 loss 降低了 GRU 在建模长行为序列时 反向传播的难度；
3. 最后但是最重要的一点，辅助 loss 提供了更多的语义信息，从而可以学习到更好的语义信息表达。

**兴趣进化层**：
- 由于外部环境和内部认知的联合影响，以 衣服的兴趣为例，随着大众流行趋势和用户品味的改变，用户对衣服的偏好进化了。

兴趣进化的优势：
1. 兴趣进化模块可以为最终兴趣的表示提供更多的相关历史信息；
2. 遵循兴趣进化的趋势来预测 目标 item 的 CTR 会得到更好的效果。

兴趣进化的特点：
1. 由于兴趣的多样性，兴趣可以随波逐流。兴趣漂移对行为的影响是用户可能在一段时间内对某种书籍感兴趣，而在另一段时间内对服装有需求。
2. 虽然兴趣之间可能会相互影响，但每一种兴趣都有自己的演变过程，例如书籍和服装的演变过程几乎是独立的。我们只关注与目标项目相关的演进过程。

兴趣进化过程的 attention 机制：$\displaystyle a_{t} = \frac{\exp(\text{h}_{t}W\text{e}_{a})}{\sum_{j=1}^{T}\exp(\text{h}_{j}W\text{e}_{a})}$，其中 $W \in R^{n_{H}\times n_{A}}$，$n_{H}$ 是隐层的维度，$n_{A}$ 是目标广告向量的维度。
- **AIGRU**：GRU with attentional input
	- AIGRU 使用 注意力分 来影响 兴趣进化网络的输入 $\text{i}_{t}^{'}=\text{h}_{t}*a_{t}$ 
	- 但 AIGRU 并不是很奏效，原因是即使 得到0输入 还是会影响 GRU 的隐层，不相关的兴趣还是会影响进化过程的学习。
- **AGRU**：Attention based GRU
	- 类似16年在问答领域提出的方法一样，在兴趣进化过程中提取相关兴趣 $\text{h}_{t}^{'}=(1-a_{t})*\text{h}_{t-1}^{'}+a_{t}*\text{\~h}_{t}^{'}$.
	- 虽然 AGRU 可以直接使用注意力分数来控制隐藏状态的更新，但它使用标量(注意力分数at)来代替向量，忽略了不同维度之间的重要性差异。
- **AUGRU**：GRU with attentional update gate
	- 将注意力机制和 GRU 无缝衔接：$\text{\~{u}}_{t}^{'}=a_{t}*\text{u}_{t}^{'}$，$\text{h}_{t}^{'}=(\text{1}-\text{\~{u}}_{t}^{'})\circ \text{h}_{t-1}^{'}+\text{\~{u}}_{t}^{'}\circ \text{\~h}_{t}^{'}$.

**Loss Function**:
$$
L=L_{target} + \alpha * L_{aux}
$$

**策略收益**：
- 相比MLP，ctr +20.7%, ecpm +17.1%, cpc -3.0%
![[Pasted image 20240527155944.png]]

公共数据集上的消融实验
![[Pasted image 20240527160220.png|]]