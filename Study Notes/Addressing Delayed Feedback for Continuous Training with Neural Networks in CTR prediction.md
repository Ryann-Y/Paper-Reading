**论文地址**：[https://arxiv.org/pdf/1907.06558](https://arxiv.org/pdf/1907.06558)

本文介绍了 Twitter 关于在线广告 模型实时训练场景下的 延迟反馈建模 经验和方案。文章中对比了两种不同的建模方案（LR / WDL）和5中不同的损失函数（log loss / FN weighted loss / FN calibration / positive unlabeled loss / delayed feedback loss）。

>文中强调了实时训练的重要性，为了将 online learning 应用在 CVR 预测中，将 $w_{o}$ 设置为0，即所有样本均先以 负例形式参与训练，待正例回收之后再进回补训练。
>原文描述是：samples are labeled as negatives and ingested to the training pipeline, and then duplicated with a positive label as soon as a user engagement takes place.

![[Pasted image 20240606140401.png|400]]

**名词解释**：
- RCE: **R**elative **C**ross **E**ntropy，相对交叉熵。
- RPMq: Revenue per thousand requests，每千次请求的收益.
- IS: **I**mportance **S**ampling，重要性采样
- FNW: FN weighted
- FNC: FN calibration

**论文方案**

对于真实数据分布 $p$，其交叉熵损失 为 $\displaystyle \mathfrak{L}(\theta)=-\mathbb{E}_{p}[\log f_{\theta}(\text{y}|\text{x})]=-\sum_{x,y}p(\text{x},\text{y})\log f_{\theta}(\text{y}|\text{x})$. 我们可以采用如下公式来对其进行无偏估计：$\displaystyle \mathbb{E}_{p}[\log f_{\theta}(\text{y}|\text{x})]=\mathbb{E}_{b}\left[ \frac{p(\text{x,y})}{b(\text{x,y})}\log f_{\theta}(\text{y|x}) \right]$，其中 $b$ 表示观测分布.

其中的权重 $\displaystyle w(\text{x,y}=\frac{p(\text{x,y})}{b(\text{x,y})}$ 是重要性权重，用来纠正在不同分布上带来的平均偏差

> This method of using samples from one distribution to estimate the expectation with respect to another distribution is called importance sampling.

**Loss Functions**:

**Delayed feedback loss**：
$$
\begin{align}
\arg&\underset{\theta,\text{w}_{d}}\min\mathcal{L}_{DF}(\theta,\text{w}_{d})+\alpha(\Vert\theta\Vert_{2}^{2}+\Vert\text{w}_{d}\Vert_{2}^{2}) \\
\mathcal{L}_{DF}(\theta,\text{w}_{d})=&-\sum_{\text{x},y=1}\log f_{\theta}(\text{x})+\log \lambda(\text{x})-\lambda(\text{x})d \\
&-\sum_{\text{x},y=0}\log[1-f_{\theta}(\text{x})+f_{\theta}(\text{x})\exp(-\lambda(\text{x})e)]
\end{align}
$$
The loss function can be computed in a more numerically stable way as follows:
$$
\begin{align}
\mathcal{L}_{DF}(\theta,\text{w}_{d})=-\sum_{\text{x},y}&\log f_{\theta}(\text{x})-\sum_{\text{x},y=1}\text{w}_{d}\cdot \text{x}-\lambda(\text{x})d \\
&-\sum_{\text{x},y=0}\log[\exp(-f_{\theta}(\text{x}))+\exp(-\lambda(\text{x})e)]
\end{align}
$$

>个人感觉，这个公式给错了，实际应该是：

$$
\begin{align}
\mathcal{L}_{DF}(\theta,\text{w}_{d})=-\sum_{\text{x},y}&\log f_{\theta}(\text{x})-\sum_{\text{x},y=1}\text{w}_{d}\cdot \text{x}-\lambda(\text{x})d \\
&-\sum_{\text{x},y=0}\log[\exp(-\theta\text{x})+\exp(-\lambda(\text{x})e)]
\end{align}
$$
>之所以说更稳定，是因为化简了计算，将不必要的部分sigmoid的计算省掉，差别在 $y=0$ 样本时，$f_{\theta}(\text{x})$ 换成了 $\theta \text{x}$.

**Positive-unlabeled loss**：
$$
\mathcal{L}_{PU}(\theta)=-\sum_{\text{x},y=1}[\log f_{\theta}(\text{x})-\log(1-f_{\theta}(\text{x}))]-\sum_{\text{x},y=0}\log(1-f_{\theta}(\text{}x))
$$

该式可能也存在问题，应该是：
$$
\mathcal{L}_{PU}(\theta)=-\sum_{\text{x},y=1}[\log f_{\theta}(\text{x})-\log(1-f_{\theta}(\text{x}))]-\sum_{\text{x},y}\log(1-f_{\theta}(\text{}x))
$$

>即 将原本负样本 的 $\displaystyle \sum_{\text{x},y=0}$ 改为 $\displaystyle \sum_{\text{x},y}$.

看下原文对 PU-Loss 的介绍：$\hat{R}_{PN}(f_{\theta})=p(y=1)\mathbb{E}_{p(x|y=1)}[l(f_{\theta}(x))]+p(y=0)\mathbb{E}_{p(x|y=0)}[l(1-f_{\theta}(x))]$
由于 $p(x)=p(y=1)p(x|y=1)+p(y=0)p(x|y=0)$，则 $p(y=0)\mathbb{E}_{p(x|y=0)}[l(1-f_{\theta}(x))]=\mathbb{E}_{p(x)}[l(1-f_{\theta}(x))]-p(y=1)\mathbb{E}_{p(x|y=1)}[l(1-f_{\theta}(x))]$.
则 $\hat{R}_{PU}(f_{\theta})=p(y=1)\mathbb{E}_{p(x|y=1)}[l(f_{\theta}(x))]-p(y=1)\mathbb{E}_{p(x|y=1)}[l(1-f_{\theta}(x))]+\mathbb{E}_{p(x)}[l(1-f_{\theta}(x))]$.

附加原文的评论：
>Empirically this could be perceived as applying the traditional log loss for both positive and negative samples. In addition, a step in the opposite direction of negative gradients is made whenever a positive example is observed. This assumption is reasonable given the fact that for every positive sample there have been parameter updates based on gradients from a fake negative sample.

**FNW: Fake Negative Weighted**

<font color=orange>假设</font>：$b(x|y=0)=p(x)$ 和 $b(x|y=1)=p(x|y=1)$，

根据贝叶斯公式可知：$\displaystyle b(y=1|x)=\frac{b(y=1)b(x|y=1)}{b(y=1)b(x|y=1)+b(y=0)b(x|y=0)}$.
设 $\displaystyle w(x):=\frac{1}{1+p(y=1|x)}$，$\displaystyle b(y=0)=\frac{1}{1+p(y=1)}=w(x)$，且 $b(y=1)=1-b(y=0)=w(x)p(y=1)$。
可得：$\displaystyle b(y=1|x)=\frac{w(x)p(y=1)p(x|y=1)}{w(x)p(y=1)p(x|y=1)+w(x)p(x)}=\frac{p(y=1)p(x|y=1)}{p(y=1)p(x|y=1)+p(x)}=\frac{p(y=1|x)p(x)}{p(y=1|x)p(x)+p(x)}=\frac{p(y=1|x)}{1+p(y=1|x)}$.

相应的，$\displaystyle b(y=0|x)=1-b(y=1|x)=\frac{1}{1+p(y=1|x)}$.

这样，整体的 Loss function：
$$
\begin{align}
\mathcal{L}_{IS}(\theta)&=-\sum_{\text{x,y}}p(\text{y}=1|\text{x})\log f_{\theta}(\text{x})+p(\text{y}=0|\text{x})\log f_{\theta}(\text{y}=0|\text{x})=-\sum_{\text{x,y}}b(\text{y}=1|\text{x}) \frac{p(\text{y}=1|\text{x})}{b(\text{y}=1|\text{x})}\log f_{\theta}(\text{x})+b(\text{y}=0|\text{x}) \frac{p(\text{y}=0|\text{x})}{b(\text{y}=0|\text{x})}\log f_{\theta}(\text{y}=0|\text{x}) \\
&=-\sum_{\text{x,y}}b(\text{y}=1|\text{x})(1+p(\text{y}=1|\text{x}))\log f_{\theta}(\text{x})+b(\text{y}=0|\text{x})p(\text{y}=0|\text{x})(1+p(\text{y}=1|\text{x}))\log f_{\theta}(\text{y}=0|\text{x})
\end{align}
$$

因此，对于 $b$ 分布中的正样本，加权为 $1+p(y=1|x)$，对于负样本，加权为 $(1-p(y=1|x))\cdot(1+p(y=1|x))$.

>其实这里不用推导，直接根据整体比例的变化，便可求出，方法类似 FNC的推导，就可以得到加权的权重。。！！！

**FNC: Fake Negative Calibration**
在训练得到 $b(y=1|x)$ 之后，对其进行校准，$\displaystyle p(y=1|x)=\frac{b(y=1|x)}{1-b(y=1|x)}$.

证明：
令正例数 为 PN, 负例个数为 NN，$PN+NN$ 是真实样本总数，则 $\displaystyle b(y=1|x)=\frac{PN}{2PN+NN}$，而 $\displaystyle p(y=1|x)=\frac{PN}{PN+NN}=\frac{PN}{2PN+NN}/\frac{PN+NN}{2PN+NN}=\frac{b(y=1|x)}{1-b(y=1|x)}$.

**策略收益**：
- RCE +3%，RPMq +55%，CTR+23.01%