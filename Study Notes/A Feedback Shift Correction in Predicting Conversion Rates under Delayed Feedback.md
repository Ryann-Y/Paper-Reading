**论文地址**：[https://arxiv.org/pdf/2002.02068](https://arxiv.org/pdf/2002.02068)

**名词解释**：
- FSIW: **F**eedback **S**hift **I**mportance **W**eight

**问题定义**：
$$
G\equiv\mathbb{E}_{(x,c)\sim(X,C)}\left[L(x,c;\hat{f}(x,\theta))\right].
$$
使用 FSIW 之后的 加权损失是：
$$
\hat{G}_{IW}^{(n)}\equiv \frac{1}{n}\sum_{i=1}^{n} \frac{P(C=y_{i}|X=x_{i})}{P(Y=y_{i}|X=x_{i})}L\left(x_{i},y_{i};\hat{f}(x_{i},\theta)\right)
$$
上述两个式子等价。

基于延迟反馈的基本定义，有如下公式：
$$
\begin{align}
&P(Y=1|X=x)=P(C=1|X=x)P(S=1|C=1,X=x) \\
&P(Y=0|X=x)=P(C=0|X=x)+P(S=0,C=1|X=x)
\end{align}
$$
其中 $S$ 表示是否标记正确，可推导出：
$$
\begin{align}
&\frac{P(C=1|X=x)}{P(Y=1|X=x)}=\frac{1}{P(S=1|C=1,X=x)} \\
&\frac{P(C=0|X=x)}{P(Y=0|X=x)}=1-\frac{P(S=0,C=1|X=x)}{P(Y=0|X=x)}
\end{align}
$$