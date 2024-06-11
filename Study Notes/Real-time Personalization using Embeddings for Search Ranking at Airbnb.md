**论文地址**：[https://dl.acm.org/doi/10.1145/3219819.3219885](https://dl.acm.org/doi/10.1145/3219819.3219885)
**论文解读**：
- https://zhuanlan.zhihu.com/p/55149901
- https://zhuanlan.zhihu.com/p/57313656

本文是KDD‘18 Best Paper

**问题背景**：
- Airbnb 作为一家 房屋短期租赁平台，具有其鲜明的特点：
	- 作为一个双端市场，需要同时根据 房主和租客的喜好来进行优化；
	- 很少有用户会同时消费相同的 item 两次 并且 在特定时期，一个房屋只能供一个顾客使用。
- 搜索排序和相似推荐作为两个 Airbnb 转化最大的渠道，占有平台 99% 的转化；
- Airbnb上的订单数据非常稀疏，一般平均每人每年旅游1-2次，因此订单数据存在较大的稀疏性，为长期兴趣建模带来了困难。

**论文贡献**：
- 构建了一个real time的ranking model，为了捕获用户short term以及long term的兴趣，先对user和listing进行了embedding，进而利用embedding的结果构建出诸多feature，作为ranking model的输入。
- 针对 short term 的 Embedding生成：
	- 引入了适用于集合搜索的适应性训练方法（对应搜索的房源基本处于**相同区域**的特性）；
	- 采用转化作为**全局上下文**；
	- 根据 搜索多数处于相同区域的特性，为 Listing 生成冷启动 Embedding。
- 针对 long term 的 Embedding 生成：
	- 针对 User Type 和 Listing Type 生成 Embedding；
	- 引入rejection作为显式负例进行Embedding的生成。

**论文方案**：

**Listing Embedding**：
通过 click session 数据生成 listing 的 embedding，生成这个embedding的目的是为了进行listing的相似推荐，以及对用户进行session内的实时个性化推荐（生成相应的相似度特征）。

问题定义：假设有一个 从 $N$ 个用户处采集到的 $\text{S}$ 大小的 click session 集合 $S$，这里 $s=(l_{1},\dots,l_{M})\in S$ 是用户的点击行为序列。

>在原始word2vec embedding的基础上，针对其业务特点，Airbnb的工程师希望能够把booking的信息引入embedding。
>这样直观上可以使Airbnb的搜索列表和similar item列表中更倾向于推荐之前booking成功session中的listing。从这个motivation出发，Airbnb把click session分成两类，最终产生booking行为的叫booked session，没有的称做exploratory session。

![[Pasted image 20240528140703.png|400]]

样本构造：
1. 从线上登录用户的所有搜索行为中创建了 800 million 的 click session 数据，根据 user id 聚合之后 按照 listing id时间 对点击进行排序；
2. 对数据根据 30分钟 不活跃规则 切分成 click session；
3. 删除意外和短点行为（低于30s停留的点击都被定义为短点），丢弃点击行为只有1个的session；
4. 根据离线评估结果，针对上述生成的 click session 中的  booked sessions 进行 **5x** 的过采样。

针对 **exploratory session**：采用 skip-gram 对 item 进行编码（使用负采样），优化目标为：
$$
\displaystyle \underset{\theta}{\arg\max}\sum_{(l,c)\in D_{p}}\log \frac{1}{1+e^{-v_{c}^{'}v_{l}}}+\sum_{(l,c)\in D_{n}}\log \frac{1}{1+e^{v_{c}^{'}v_{l}}}
$$
其中，$D_{p}$ 是正例 集合，$D_{n}$ 是负采样得到的负例集合。

针对 **booked session**：通过将最终 booked item 作为全局上下文，更改之后的优化目标为：
$$
\displaystyle \underset{\theta}{\arg\max}\sum_{(l,c)\in D_{p}}\log \frac{1}{1+e^{-v_{c}^{'}v_{l}}}+\sum_{(l,c)\in D_{n}}\log \frac{1}{1+e^{v_{c}^{'}v_{l}}}+\log \frac{1}{1+e^{-v_{l_{b}}^{'}v_{l}}}
$$
其中，$v_{l_{b}}$ 代表 booked 全局 session。

**Adapting Training for Congregated Search**：由于用户的搜索行为主要集中在相同的地区，因此可以引入同地域的负样本作为补充
$$
\displaystyle \underset{\theta}{\arg\max}\sum_{(l,c)\in D_{p}}\log \frac{1}{1+e^{-v_{c}^{'}v_{l}}}+\sum_{(l,c)\in D_{n}}\log \frac{1}{1+e^{v_{c}^{'}v_{l}}}+\log \frac{1}{1+e^{-v_{l_{b}}^{'}v_{l}}}+\sum_{(l,m_{n})\in D_{m_{n}}}\log \frac{1}{1+e^{v_{m_{n}}^{'}v_{l}}}
$$
其中 $D_{m_{n}}$ 是在相同 market 采样得到的负例。

>实验发现，每天重新训练得到的 embedding 比 增量训练产生的 embedding 效果要好。这样训练后，即便向量存在一定的改变和差异，但由于我们通常是使用 cosine 相似度 而不使用向量本身，因此系统不会有什么影响。

实验设置：
- dim = 32
- window size m = 5
- iteration = 10

离线评估：
- 通过计算cosine相似度来进行排序，查看 booked listing的排序位置。

**User-type & Listing-type Embedding**：

通过 booking session 生成 user-type 和 listing-type 的embedding，捕捉不同 user-type 的 long term 偏好。
由于booking signal过于稀疏，Airbnb对同属性的 user 和 listing 进行了聚合，形成了user-type和 listing-type 这两个embedding的对象。

![[Pasted image 20240528213349.png|400]]

将 $(u_{type},l_{type})$ 的元组 一起 生成元组序列 $S_{b}$，其中 $s_{b}=(u_{type_{1}},l_{type_{1}},\dots,u_{type_{M}},l_{type_{M}})\in S_{b}$. 优化目标为：
针对中心 item 为 $user\_{}type(u_t)$：
$$
\displaystyle \underset{\theta}{\arg\max}\sum_{(u_{t},c)\in D_{book}} \log \frac{1}{1+e^{-v_{c}^{'}v_{u_{t}}}}+\sum_{(u_{t},c)\in D_{neg}} \log \frac{1}{1+e^{v_{c}^{'}v_{u_{t}}}}
$$
针对中心 item 为 $listing\_type(l_{t})$：
$$
\displaystyle \underset{\theta}{\arg\max}\sum_{(l_{t},c)\in D_{book}} \log \frac{1}{1+e^{-v_{c}^{'}v_{l_{t}}}}+\sum_{(l_{t},c)\in D_{neg}} \log \frac{1}{1+e^{v_{c}^{'}v_{l_{t}}}}
$$

**Explicit Negatives for Rejections**.
$$
\displaystyle \underset{\theta}{\arg\max}\sum_{(u_{t},c)\in D_{book}} \log \frac{1}{1+e^{-v_{c}^{'}v_{u_{t}}}}+\sum_{(u_{t},c)\in D_{neg}} \log \frac{1}{1+e^{v_{c}^{'}v_{u_{t}}}}+\sum_{(u_{t},l_{t})\in D_{reject}}\log \frac{1}{1+e^{v_{l_{t}}^{'}v_{u_{t}}}}
$$
$$
\displaystyle \underset{\theta}{\arg\max}\sum_{(l_{t},c)\in D_{book}} \log \frac{1}{1+e^{-v_{c}^{'}v_{l_{t}}}}+\sum_{(l_{t},c)\in D_{neg}} \log \frac{1}{1+e^{v_{c}^{'}v_{l_{t}}}}+\sum_{(l_{t},u_{t})\in D_{reject}}\log \frac{1}{1+e^{v_{u_{t}}^{'}v_{l_{t}}}}
$$

**线上应用**：
- 相似推荐 直接使用 listing 的 Embedding 做 cosine 相似度  选择 Top 的结果
- search ranking，构造多种不同的feature，用于 pairwise 的 ranking model 中

![[Pasted image 20240528184156.png|400]]

![[Pasted image 20240528184500.png|400]]

$\displaystyle \text{EmbClickSim}(l_{i}, H_{c})=\underset{ m \in M }{\max} \cos \left( v_{l_{i}}, \sum_{l_{h}\in m,l_{h} \in H_{c}} v_{l_{h}} \right)$ 其中 $M$ 是 用户点击过的 markets 的集合。

$\text{EmbLastLongClickSim}(l_{i}, H_{l_{c}})=\cos (v_{l_{i}},v_{l_{last}})$.

**策略收益**：
- Similar Listing Recommendation：CTR +21%，满意度（在该场景完成booking的占比）+4.9%
