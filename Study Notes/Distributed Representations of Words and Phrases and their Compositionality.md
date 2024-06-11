**论文地址**：[https://proceedings.neurips.cc/paper/2013/file/9aa42b31882ec039965f3c4923ce901b-Paper.pdf](https://proceedings.neurips.cc/paper/2013/file/9aa42b31882ec039965f3c4923ce901b-Paper.pdf)

本文提出了一些针对 word2vec 的扩展和改进，包括通过负采样替代层次softmax。
此外，还描述了 skip-gram 模型的一个有意思的性质，一些简单的向量运算可以得到很有意义的结果，比如 vec("Russia") + vec("river") 和 vec("Volga River") 很相近，vec("Germany") + vec("capital") 和 vec("Berlin") 比较相近。

>这种组合性表明，通过对词向量表示进行基本的数学运算，可以获得非明显程度的语言理解。

**名词解释**：
- NCE：**N**oise **C**ontrasitive **E**stimation，噪声对比估计

**问题背景**：
- 词表征的一个内在局限性是它们对语序的忽视以及无法表征习语。例如："Canada" 和 "Air" 很难合并为 "Air Canada"，因此使用整个词组的向量表示是更好的；

**Hierarchical Softmax**：
层次softmax使用一棵输出层的二叉树表示，二叉树中的叶子节点表示 $W$ 个 word，因此复杂度是 $\log_{2}(W)$

使用 $n(w,j)$ 表示 从 root 到 $w$ 路径上的第 $j$ 个节点，$L(w)$ 表示路径的长度，这样 $n(w,1) = root$ 并且 $n(w,L(w))=w$. 令 $\text{ch}(n)$ 表示 n 的任意 child，并且 当 x 为 true时候 $[[x]]=1$，否则为 -1. 
>对 $\text{ch}$ 的理解应该是，对应的那个子节点！比如当前应该取 left_child，那么对应 left_child 时为1，如果是right_child 则为0，否则，反之。这样就和 二分类的 正负例计算一样了。

$$
p(w|w_{I})=\prod_{j=1}^{L(w)-1}\sigma\left([[n(w,j+1)=\text{ch}(n(w,j))]]\cdot {v_{n(w,j)}^{'}}^{T}v_{w_{I}}\right)
$$
这里使用二叉霍夫曼树，可以使得性能更好。

>个人理解：二叉树中每个节点都可以当做二分类，整个路径就是一系列的二分类决策过程，根据极大似然估计，将每个节点决策的对应路径概率最大化。
>由于每个节点都需要判断，因此二叉树中每个节点都有一个向量表示，

**Negative Sampling**：
objective:
$$
\log \sigma({v_{w_{O}}^{'}}^{T}v_{w_{I}})+\sum_{i=1}^{k}\text{E}_{w_{i}\sim P_{n}(w)}\left[\log \sigma(-(v_{w_{i}}^{'})^{T}v_{w_{I}})\right]
$$
>NCE (Noice Contrastive Estimation) 假定一个好的模型是可以从噪音中区分出数据的，因此上式中 $w_{i}\sim P_{n}(w)$ 表示的是从 噪音分布中 采样到的噪音!

>根据 "Negative-Sampling Word-Embedding Method" 的介绍，这里实际上的目标是 假设一个分布 $D$，其中 $p(D=1|w,c)$ 的概率表示观测到的数据实际在 分布内的概率，而采样的数据 为 $p(D=0|w,c)$ 的概率。
>因此，不同于 原始的 skip-gram 模型 建模的是 $p(c|w)$，而是建模一个和 $w,c$ 有关的联合分布的量。

- 负采样：负样本被采样的概率是unigram distribution的3/4次方。统计每个单词出现的次数，再对次数求3/4次方，再求和归一化得到概率分布。unigram distribution代表一元分布。

**Subsampling of Frequent Words**：
对于每个 word $w_{i}$，都有一定概率被丢失，丢弃概率为 $\displaystyle P(w_{i})=1-\sqrt{ \frac{t}{f(w_{i})} }$. 其中 $f(w_{i})$ 是单词出现的频率，则达到频率越大，丢失概率越大的效果。
>这个采样方式是启发式的，实际应用可以根据需要制定采样策略！

**策略效果**：
![[Pasted image 20240531163606.png|600]]