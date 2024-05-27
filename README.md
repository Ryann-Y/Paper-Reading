# 推荐系统相关论文汇总

## 介绍
1. 截至2024-04-02，本仓库收集汇总了推荐系统领域相关论文共 **805** 篇，涉及：**召回**，**粗排**，**精排**，**重排**，**多任务**，**多场景**，**多模态**，**冷启动**，**校准**，**纠偏**，**多样性**，**反馈延迟**，**蒸馏**，**对比学习**，**因果推断**，**Look-Alike**，**Learning-to-Rank**，**强化学习** 等领域，本仓库会跟踪业界进展，持续更新。
2. 因文件名特殊字符的限制，故论文title中所有的`:`都改为了` -`，检索时请注意。
3. 文件名前缀中带有`[]`的，表明本人已经通读过，第一个`[]`中为论文年份，第二个`[]`中为发表机构或公司(可选)，第三个`[]`中为论文提出的model或method的简称(可选)。
4. 在某些一级分类下面，还有若干二级分类；一篇论文可能应该涉及多个二级分类(例如用对比学习的方法做召回)，最终我会将论文放在较主要的那一类下；分类也会随时调整优化。    
5. 关于排序算法的一些实现，请见另一个repo: https://github.com/tangxyw/RecAlgorithm    


| Title                                                                                                                                                                                                                                | Year | Publisher |  Author   | Tags         | Alias |  Score   | IsRead |                                                                  Notes                                                                  | Remarks            |
| :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :--: | :-------: | :-------: | :----------- | :---: | :------: | :----: | :-------------------------------------------------------------------------------------------------------------------------------------: | ------------------ |
| [Batch Normalization: Accelerating Deep Network Training by Reducing Internal Covariate Shift]([2015][BN]%20Batch%20Normalization%20-%20Accelerating%20Deep%20Network%20Training%20by%20Reducing%20Internal%20Covariate%20Shift.pdf) | 2015 |   PMLR    |  Google   |              |  #BN  | ⭐️⭐⭐️⭐⭐️ |   N    |                                                                                                                                         |                    |
| [A Brief History of Recommender Systems]([2022]%20A%20Brief%20History%20of%20Recommender%20Systems.pdf)                                                                                                                              | 2022 |    KDD    | ByteDance |              |       |   ⭐️⭐⭐   |   Y    |                                 [论文解读](Study%20Notes/A%20Brief%20History%20of%20Recommender%20Systems)                                  | 推荐系统技术的总结          |
| [Towards Understanding the Overfitting Phenomenon of Deep Click-Through Rate Prediction Models]([2022]%20Towards%20Understanding%20the%20Overfitting%20Phenomenon%20of%20Deep%20Click-Through%20Rate%20Prediction%20Models.pdf)      | 2022 |   CIKM    |  Alibaba  | #Overfitting |       |   ⭐⭐⭐⭐   |   Y    | [论文解读](Study%20Notes/Towards%20Understanding%20the%20Overfitting%20Phenomenon%20of%20Deep%20Click-Through%20Rate%20Prediction%20Models) | 引发对Abacus两阶段训练的思考？ |
|                                                                                                                                                                                                                                      |      |           |           |              |       |          |        |                                                                                                                                         |                    |

