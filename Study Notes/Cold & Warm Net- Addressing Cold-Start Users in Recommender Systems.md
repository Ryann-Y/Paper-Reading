
**论文地址**：[https://arxiv.org/pdf/2309.15646](https://arxiv.org/pdf/2309.15646)

来自腾讯QQ短视频推荐团队

**名词解释**：
- DKD: Dynamic knowledge distillation, 动态知识蒸馏，当 $l_{cold} \gt l_{warm}$ 时，即冷启动专家网络效果比热启动专家网络差时，我们需要从热启专家网络蒸馏知识，反之则不需要。

**网络结构**：
![[Pasted image 20240515152653.png|600]]

**论文贡献**：
1. 双专家网络：通过对冷启动和热启动专家网络的划分，cold&warm网络可以动态地生成用户在冷启和热启阶段不同的兴趣表征，网络可以通过门控机制根据用户的不同类型和阶段自动合并两个专家的结果，通过冷启和预热专家解决样本之间差异的问题；
2. 灵活的教师选择器：使用教师选择器通过动态知识蒸馏的方式训练冷启和热启专家网络。选择器根据预测精度为冷启动专家选择合适的教师，避免了冷启动专家的欠拟合，同时防止训练后两个专家的同化，从而能够从冷启用户那里学习到更充分的信息；
3. 偏差网络：使用偏差网络 (bias net) 对用户的行为偏差进行显式建模。论文通过利用互信息选择和用户行为高度相关的特征用于建模偏差网络。然后结合原始网络的相似度得分和偏差网络的偏置得分，充分考虑了隐藏在用户行为背后的信息。

**用户的三种特征**：
1. $U_{a}$：基于用户画像信息的特征，这也是新用户能拿到的信息；
2. $U_{b}$：基于用户行为提炼出来的特征，这是只有老用户，也就是 warm users 才有的特征；
3. $U_{c}$：是 user group 信息。先用一个 pre-trained model 把一些 active users 压缩成 embedding，再使用 k-means 聚类成 m 个 user group。$U_{c}$ 就是由这 m 个 user group 中心组成的 m 个 embedding。

**对应的Embedding表示**：
1. $\overrightarrow{e}_{up}\in R^{1\times d}$，user profile embedding
2. $\overrightarrow{e}_{ua}\in R^{1\times d}$，user action embedding
3. $E_{ug}\in R^{m\times d}$，user group embedding

**顶层交互**：
cold&warm net产出 user embedding $\overrightarrow{e}_{u}$ 以及 $\overrightarrow{e}_{i}$，然后进行相似度计算：
$$
\displaystyle
y_{sim\_score}=\frac{\overrightarrow{e_{u}}\cdot \overrightarrow{e_{i}}}{\Vert\overrightarrow{e_{u}}\Vert\Vert\overrightarrow{e_{i}}\Vert}
$$
$$
y=sigmoid(y_{sim\_score}+y_{bias\_score})
$$

**bias net**：
其中 通过互信息，筛选 $\beta$ 个相关特征，作为bias net的输入，记为 ${\chi}_{b}$，实验中 $\beta$ 取值为10。

**用户cold&warm嵌入层**：
![[Pasted image 20240515163342.png|600]]
采用注意力机制从先验用户群向量中获取信息来辅助建模冷启动用户：$\displaystyle \overrightarrow{e}^{a}_{cold}=softmax\left( \frac{\overrightarrow{e}_{up}E_{ug}^{T}}{\sqrt{ d }} \right)E_{ug}$，cold-start expert的输出 $\overrightarrow{e}_{cold}=mlp(\overrightarrow{e}_{up};\overrightarrow{e}_{cold}^{a})$.
同理，warm-start net 先将 $\chi_{up}$ 和 $\chi_{ua}$ 作为输入，获取 embedding $\overrightarrow{e}_{ut}\in R^{1 \times d}$，然后使用注意力机制得到：$\displaystyle \overrightarrow{e}^{a}_{warm}=softmax\left( \frac{\overrightarrow{e}_{ut}E_{ug}^{T}}{\sqrt{ d }} \right)E_{ug}$，warm-start expert的输出 $\overrightarrow{e}_{warm}=mlp(\overrightarrow{e}_{ut};\overrightarrow{e}_{warm}^{a})$.

采用门控机制，融合cold和warm net的输出，$\overrightarrow{e}_{u}=w_{cold}\cdot\overrightarrow{e}_{cold}+w_{warm}\cdot\overrightarrow{e}_{warm}$，其中 $w_{cold},w_{warm}=f_{weight}(\chi_{us})$. $\chi_{us}$ 表示状态信息，包括 登录状态和活跃度等信息。

**Algorithm: Dynamic knowledge distillation**
![[Pasted image 20240516105951.png|500]]
**注**：图中的符号写反了!!!

**Loss Function**：$L_{o}=L+\alpha \cdot L_{d}$，其中实验的 $\alpha$ 取值为 5e-2.

**策略收益**：
1. 冷启动用户：APT(app dwell time) +3.27%, URR(User retention rate) +1.01%, VPI(Video play integrity) +23.34%, VSR(video skip rate) -14.30%