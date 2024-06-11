**论文地址**：[https://arxiv.org/pdf/2202.06472](https://arxiv.org/pdf/2202.06472)
**源码地址**：[https://github.com/ychen216/DEFUSE.git.](https://github.com/ychen216/DEFUSE.git.)
**参考**：
- 官方：[https://zhuanlan.zhihu.com/p/506476146](https://zhuanlan.zhihu.com/p/506476146)

**名词解释**：
- DEFUSE: **DE**layed **F**eedback modeling with **U**nbia**S**ed **E**stimation，具有无偏估计的延迟反馈建模

**问题背景**：
- 延迟反馈问题对于在线广告的 CVR 预估至关重要。之前的建模方法主要有以下几种：
	- 使用观察窗口来平衡等待准确标签和消费新鲜反馈之间的权衡。
	- 为了基于新观测到的但带有假阴性的有偏分布数据来估计CVR，重要性抽样被广泛使用来减小分布偏差。
- 以上方法虽然有效，但我们认为以前的方法在重要性加权时错误地**将假负样本视为真负**，并且没有充分利用观察到的正样本，导致性能不佳。

![[Pasted image 20240603225244.png|400]]

**业界方案**：

假设：
$$
\begin{align}
\mathcal{L}&=E_{(\text{x},y)\sim p(\text{x},y)}\ell(y,f_{\theta}(\text{x})) \\
&=E_{(\text{x},y)\sim q(\text{x},y)}w(\text{x},y)\ell(y,f_{\theta}(\text{x}))
\end{align}
$$
其中，$p(\text{x},y)$ 和 $q(\text{x},y)$ 分别表示 真实分布和观测（包括回补）的分布，$w(\text{x},y)$ 表示似然比。
>假设 $p(\text{x})\approx q(\text{x})$ ！！！

$$
\begin{align}
\mathcal{L}&=E_{(\text{x},y)\sim q(\text{x},y)}w(\text{x},y)\ell(y,f_{\theta}(\text{x})) \\
&=\int q(\text{x}) \, dx \int q(y|\text{x}) \frac{p(\text{x},y)}{q(\text{x},y)}\ell(\text{x},y;f_{\theta}(\text{x})) \, dy \\
&=\int q(\text{x}) \frac{p(\text{x})}{q(\text{x})} \, dx \int q(y|\text{x}) \frac{p(y|\text{x})}{q(y|\text{x})} \ell(\text{x},y;f_{\theta}(\text{x})) \, dy \\
&\approx \int q(\text{x}) \, dx \int q(y|\text{x}) \frac{p(y|\text{x})}{q(y|\text{x})}\ell(\text{x},y;f_{\theta}(\text{x})) \, dy \\
&\approx \sum_{(\text{x}_{i},y_{i})\in D}\left[y_{i} \frac{p(y_{i}=1|\text{x}_{i})}{q(y_{i}=1|\text{x}_{i})}\log f_{\theta}(\text{x}_{i})+(1-y_{i})\frac{p(y_{i}=0|\text{x}_{i})}{q(y_{i}=0|\text{x}_{i})}\log(1-f_{\theta}(\text{x}_{i})) \right]
\end{align}
$$

- FNC/FNW 将所有样本作为负例训练，当转化反馈接收时，重新下发所有正例；$\displaystyle q_{fnw}(y=0|\text{x})=\frac{1}{1+p(y=1|\text{x})}$.
- ES-DFM 只重复之前被错误标记为假阴性的延迟正例样本；$\displaystyle q_{esdfm}(y=0|\text{x})=\frac{p(y=0|\text{x})+f_{dp}(\text{x})}{1+f_{dp}(\text{x})}$.
- DEFER 在完成标签归属后重用所有具有实际标签的样本，以保持特征分布相等，并利用真实负样本。$\displaystyle q_{defer}(y=0|\text{x})=p(y=0|\text{x})+\frac{1}{2}f_{dp}(\text{x})$.

![[Pasted image 20240604165358.png|400]]

局限性：假负例应该表示为 $\displaystyle \frac{p(d>w_{o},y=1|\text{x})}{q(y=0|\text{x})}$，而在上述推导中被当成了真负例！！！

**解题思想**：
1. 在应用重要度采样之前在观测到的负样本中，推断其假负例的概率；
2. 为了充分利用观测数据分布中的直接正例，通过双分布建模框架来对无偏直接正例和有偏延迟转化联合建模。

>DEFUSE 目的是在更细的粒度上分别校正 immediate positive、fake negative、real negative 和 delayed positive 样本的重要权值

**模型BI-DEFUSE结构**：
![[Pasted image 20240603225442.png|400]]

**论文方案**：
延迟反馈的无偏估计：$\displaystyle \mathcal{L}_{ub}=\int q(\text{x}) \, dx \int q(v|x) \frac{p(\text{x})}{q(\text{x})} \frac{p(y(v,d)|\text{x})}{q(v|\text{x})}\ell(\text{x},y(v,d);f_{\theta}(\text{x})) \, dv$，其中 $\displaystyle y=y(v,d)=\begin{cases} 1, v=1 \\0, v=0,d=+\infty\\1,v=0,d>w_{o}\end{cases}$，$v$ 是观测 label.
importance weight of DEFUSE：通过将样本划分为四个部分，可以将 $\mathcal{L}_{ub}$ 进行改写：$\displaystyle \mathcal{L}_{ub}=\int q(x) \left[\sum_{v_{i}}q(v_{i}|x)w_{i}(\text{x},y(v_{i},d))\ell(\text{x},y(v_{i},d);f_{\theta}(\text{x}))\right] \, dx$，这里 $\displaystyle w_{i}=\frac{p(x,y(v_{i},d))}{q(x,v_{i})},i\in\{IP,FN,RN,DP\}$，这里 $\mathcal{L}_{ub}$ 里的 $w_{i}(\text{x},y(v_{i},d))$ 应该表示的是输入为 $\text{x}$ 和 $y(v_{i},d)$ 的函数。
四类样本中，IP、DP均可直接观测到，并根据 $d$ 与$w_{o}$ 的关系可相互区分，只有RN与FN无法通过观测数据直接区分。为了解决这一问题从而求解，我们引入潜变量 $z$ 来刻画一个观测到的负样本是否为假负样本FN，从而区分RN与FN，因此通过推导可知，
$$
\begin{align}
&\underset{\theta}\min \mathcal{L}_{ub} \\
\iff &\underset{\theta}\min \int q(\text{x})\bigg[v(w_{DP}\log f_{\theta}(\text{x})+\mathbb{I}_{IP}(w_{IP}-w_{DP})\log f_{\theta}(\text{x}))+(1-v)(w_{FN}\log f_{\theta}(\text{x})z+w_{RN}\log(1-f_{\theta}(\text{x}))(1-z))\bigg] \, dx  \\
s.t. \\
&w_{IP}(\text{x})=w_{RN}(\text{x})=1+f_{dp}(\text{x}) \\
&w_{DP}(\text{x})+w_{FN}(\text{x})=1+f_{dp}(\text{x})
\end{align}
$$
其中，$w_{IP}(\text{x})$，$w_{DP}(\text{x})$，$w_{FN}(\text{x})$，$w_{RN}(\text{x})$ 即四类样本对应的 importance weight，$\mathbb{I}_{IP}$ 为样本是否为观测窗口内成交的正样本(IP)的示性函数。考虑到DP可观测，我们将关于 $w_{DP}(\text{x})$ 和 $w_{FN}(\text{x})$ 的约束简化为 $w_{DP}(\text{x})=1$ 与$w_{FN}(\text{x})=f_{dp}(x)$，同时引入辅助模型 $f_{dp}(x)$ 通过建模一个点击最终为窗口外延迟成交的概率来刻画 $w_{i}$。

![[Pasted image 20240605103344.png|600]]

![[Pasted image 20240605103423.png|600]]

**评估指标**：
- AUC
- PR-AUC
- NLL
- RI-AUC: $\displaystyle \text{RI-AUC}_{\text{DEFUSE}}=\frac{\text{AUC}_{\text{DEFUSE}}-\text{AUC}_{\text{Pre-trained}}}{\text{AUC}_{\text{Oracle}}-\text{AUC}_{\text{Pre-trained}}}\times 100\%$.
	- 其中 $\text{Oracle}$ 是基于 ground-truth 训练，被认为是延迟反馈建模的上界

**策略收益**：
- CVR +2.28%

> 重点：论文中指出，单独建模两个模型，效果其实更好，这也说明了两个模型分布的差异巨大

![[Pasted image 20240604193358.png|400]]

