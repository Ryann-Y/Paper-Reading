**论文地址**：[https://wnzhang.net/share/rtb-papers/delayed-feedback.pdf](https://wnzhang.net/share/rtb-papers/delayed-feedback.pdf)

**参考**：
- https://zhuanlan.zhihu.com/p/350653347

发表在14年的kdd，来自criteo公司，做retargeting的国际化公司。

**Highlight**

>1. 用指数分布对转化的延迟（到来）进行建模，观察到的数据，是伯努利分布和指数分布的联合分布。
>2. 对建模后的推导，采取了分类讨论的方法，对正例和unlabeld分开讨论和建模，简化了问题。
>3. <font color=orange>缺点</font>：由于延迟成交的样本只能通过联合建模来推断而无法再成交时作为确定性的正反馈，因此提升效果有限！！！
>4. 重点在于，极大似然估计的地方，即 “问题建模” 中的两个概率计算。

**问题背景**：
CVR预估问题普遍存在延迟反馈的问题，如下是转化数据回收基于时间的累计分布，可以看到 以30天回收转化为全集来看，首日仅可回收不到50%的转化数据
![[Pasted image 20240605152504.png|400]]

此外，针对一些时效性要求高，例如新 campaign占比较高 的场景，如何权衡时效性和数据label的准确就成了延迟反馈建模需要解决的重点。

**问题定义**：

设定如下符号表示：
- $X$ 表示特征集合；
- $Y\in\{0,1\}$ 表示转化事件是否已发生，即观测结果；
- $C\in\{0,1\}$ 表示用户最终是否会转化，即需要预测的最终结果；
- $D$ 是点击和转化事件之间的时间，当 $C=0$ 时，含义未定义；
- $E$ 表示点击之后的间隔时间

**问题建模**

用两个参数模型来拟合数据：
1. 转化概率的模型 $\text{Pr}(C|X)$，其表示为 $\text{Pr}(C=1|X=\text{x})=p(\text{x})$ with $\displaystyle p(\text{x})=\frac{1}{1+\exp(-\text{w}_{c}\cdot \text{x})}$；
2. 转化延迟模型 $\text{Pr}(D|X,C=1)$，其表示为 $\text{Pr}(D=d\mid X=\text{x},C=1)=\lambda(\text{x})\exp(-\lambda(\text{x})d)$，为了保证 $\lambda(\text{x})>0$，我们参数化 $\lambda(\text{x})=\exp(\text{w}_{d}\cdot \text{x})$.

针对观测到的<font color=orange>正例</font>有如下表示：
$$
\begin{align}
\text{Pr}(&Y=1,D=d_{i}\mid X=\text{x}_{i},E=e_{i}) \\
&=\text{Pr}(C=1,D=d_{i}\mid X=\text{x}_{i},E=e_{i}) \\
&=\text{Pr}(C=1,D=d_{i}\mid X=\text{x}_{i}) \\
&=\text{Pr}(D=d_{i}\mid X=\text{x}_{i},C=1)\text{Pr}(C=1\mid X=\text{x}_{i}) \\
&=\lambda(\text{x}_{i})\exp(-\lambda(\text{x}_{i})d_{i})p(\text{x}_{i})
\end{align}
$$
这里 由于是观测到的正例，所以 $e_{i}$ 一定大于 $d_{i}$，这里为了便于理解，可以直接看第三行，应该更好理解，即 观测到的正例，已经是 发生在 $d_{i}$ 时刻 $C=1$ 的真实正例。

针对观测到的<font color=orange>负例</font>有如下表示：
$$
\text{Pr}(Y=0\mid X=\text{x}_{i},E=e_{i})=\text{Pr}(Y=0\mid C=0,X=\text{x}_{i},E=e_{i})P(C=0\mid X=\text{x}_{i})+\text{Pr}(Y=0\mid C=1,X=\text{x}_{i},E=e_{i})P(C=1\mid X=\text{x}_{i})
$$
更进一步，
$$
\begin{align}
\text{Pr}(Y=0&\mid C=1,X=\text{x}_{i},E=e_{i}) \\
&=\text{Pr}(D>E\mid C=1,X=\text{x}_{i},E=e_{i}) \\
&=\int_{e_{i}}^{\infty}\lambda(\text{x})\exp(-\lambda(\text{x})t) \, dt \\
&=\exp(-\lambda(\text{x})e_{i}) 
\end{align}
$$
由于 $\text{Pr}(Y=0\mid C=0,X=\text{x}_{i},E=e_{i})=1$，则可得到 $\text{Pr}(Y=0\mid X=\text{x}_{i},E=e_{i})=1-p(\text{x}_{i})+p(\text{x}_{i})\exp(-\lambda(\text{x}_{i})e_{i})$.

**EM 优化算法**：
$C$ 作为 EM 算法中的一个隐变量，由于无法对延迟正样本做明确的监督，这也限制了最终算法的效果！！！

对于 给定的 样本点 $(\text{x}_{i},y_{i},e_{i})$，令 $\text{Pr}(C=1\mid X=\text{x}_{i},Y=y_{i}):=w_{i}$，
当 $y_{i}=1$ 时，$w_{i}=1$；
当 $y_{i}=0$ 时，$w_{i}=\text{Pr}(C=1\mid Y=0,X=\text{x}_{i},E=e_{i})=\text{Pr}(Y=0\mid C=1,X=\text{x}_{i},E=e_{i})\text{Pr}(C=1\mid X=\text{x}_{i})=\exp(-\lambda(\text{x}_{i})e_{i})p(\text{x}_{i})$.

> 上式的结果：一个是条件概率，一个是联合分布，其实不相等，即 $\text{Pr}(C=1\mid Y=0)\neq \text{Pr}(Y=0,C=1)$. 但当 y=0 确认发生时，可 认为相等，其实看的还是条件概率。

计算 对数似然函数的 期望得到：
$$
\begin{align}
&\sum_{i,y_{i}=1}\log \text{Pr}(Y=1,D=d_{i}\mid X=\text{x}_{i},E=e_{i})+\sum_{i,y_{i}=0}(1-w_{i})\log \text{Pr}(Y=0,C=0\mid X=\text{x}_{i},E=e_{i})+w_{i}\log \text{Pr}(Y=0,C=1\mid X=\text{x}_{i},E=e_{i}) \\
&=\sum_{i}w_{i}\log p(\text{x}_{i})+(1-w_{i})\log(1-p(\text{x}_{i}))+\sum_{i}\log(\lambda(\text{x}_{i}))y_{i}-\lambda(\text{x}_{i})t_{i}w_{i}
\end{align}
$$
这里 $t_{i}:=\begin{cases} e_{i},&\text{if}\, y_{i}=0 \\ d_{i} &\text{if}\, y_{i} = 1 \end{cases}$.

>这个函数有个有<font color=orange>意思的性质</font>：
>1. $p$ 和 $\lambda$ 可独立优化，并且这两个都是凸函数
>2. 可解释性很强

**联合优化**：
$$
\arg \underset{\text{w}_{c},\text{w}_{d}}\min L(\text{w}_{c},\text{w}_{d})+\frac{\mu}{2}(\Vert\text{w}_{c}\Vert_{2}^{2}+\Vert\text{w}_{d}\Vert_{2}^{2})
$$
$$
\begin{align}
L(\text{w}_{c},\text{w}_{d})&=-\sum_{i,y_{i}=1}\log p(\text{x}_{i})+\log \lambda(\text{x}_{i})-\lambda(\text{x}_{i})d_{i} \\
&=-\sum_{i,y_{i}=0}\log[1-p(\text{x}_{i})+p(\text{x}_{i})\exp(-\lambda(x_{i})e_{i})]
\end{align}
$$

这是根据 在数据 D 上观测到的真实 label Y 计算的极大似然估计。