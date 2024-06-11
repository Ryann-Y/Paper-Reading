**论文地址**：[https://arxiv.org/pdf/2108.04468](https://arxiv.org/pdf/2108.04468)

**名词解释**：
- ETA: End-to-end Target Attention，端到端目标注意力；
- LSH: Locality-sensitive hashing，局部敏感 哈希。

**问题背景**：
- 引入长序列的用户行为建模非常必要，在电商业务中，过去5个月点击行为超过1000的用户占比高达23%
- 对于长序列建模通常采用 两阶段的方案 (SIM)，第一个阶段从长序列中筛选出 top-k 个相似的 item，第二个阶段再采用传统的注意力机制进行建模。但是，这种方式使得 retrieval 阶段和 主 CTR 任务之间存在 信息差 **Information gap**。
	- 首先，第一阶段采用的类目和CTR模型的预估目标不同，造成信息gap；
	- 其次，由于现在很多CTR模型采用online learning的训练方式，而这种预训练生成embedding构建倒排索引的方式，显然会造成 embedding 过时，造成信息gap；

**论文贡献**：
- 提出一种 End-to-end 的 Target Attention CTR预估模型 ETA，首次采用 端到端 的方式进行 long-term 用户行为序列建模；
- 检索的复杂度 从 $O(L*B*d)$ 到 $O(L*B)$，其中 $L$ 是序列的长度，$B$ 是候选 item 的数量，$d$ 是 embedding 的维度。复杂度的降低使我们可以在线实时计算所有 item的 hamming distance 来替代原有的第一阶段的辅助任务。

>向量的内积被 simHash 签名后的向量 的 hamming distance 所替代！！

**局部敏感哈希**：
![[Pasted image 20240529154035.png|400]]

>实际上就是给定了一个 哈希矩阵，每列代表一个 哈希函数，通过矩阵乘法，得到 embedding 在每个哈希函数下生成的值；
>每个哈希函数的值通过 sign函数得到最后的 哈希编码。

如下是 simHash 的图示解释
![[Pasted image 20240529154053.png|600]]

hamming distance 的计算：
>use 𝑙𝑜𝑔(𝑚)-bits integer to represent the signature vector because each element in 𝒔𝒊𝒈𝑘 is either 1 or 0.
>This can greatly reduce the cost of memory and can speed up the calculation of hamming distance.

**模型结构**：
![[Pasted image 20240529154143.png|800]]

**策略收益**：
- GMV +3.1% compared to SIM

**线上性能**：
![[Pasted image 20240529182101.png]]