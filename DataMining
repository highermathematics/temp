# 关联规则挖掘详细知识点整理
## 一、核心荣誉案例
- 经典案例：啤酒和尿布

## 二、基本概念
### （一）基础定义
1. 项（Item）：单个的事物个体，所有项的集合记为 \( I = \{i_1, i_2…, i_m\} \)，其中 \( |I|=m \) 为项的总数。
2. 项集（Item Set）/模式（Pattern）：项的集合，包含 \( k \) 个项的项集称为 \( k \)-项集（\( k \)-ItemSet），若事务 \( T \) 包含项集 \( A \)，当且仅当 \( A \subseteq T \)。
3. 数据集（Data Set）/数据库（Data Base）：与任务相关的数据库事务（记录）的集合，记为 \( D = \{T_1, T_2 …, T_n\} \)，其中 \( T_i (i\in[1, n]) \subseteq I \)，每个事务有唯一标识符（TID），\( |D|=n \) 为数据集中事务的总数。
4. 支持度（support）：项集的出现频率/比例，即包含该项目的事务数量。
5. 置信度/可信度（confidence）：在数据集 \( D \) 中包含项集 \( A \) 的事务中，同时包含项集 \( B \) 的条件概率 \( P(B|A) \)。

### （二）关键概念拓展
1. 频繁项集（Frequent ItemSet）/模式（Pattern）：若项集的支持度大于或等于最小支持度（minimum support），则称该项集为频繁项集（频繁模式集），频繁 \( k \)-项集的集合记为 \( L_k \)。
2. 关联规则（Association Rules）：形如 \( A\Rightarrow B \) 的蕴涵式，其中 \( A\subset I \)，\( B\subset I \)，且 \( A\cap B=\emptyset \)。规则 \( A\Rightarrow B \) 在事务集 \( D \) 中成立时，具有：
    - 支持度 \( s = support(A\Rightarrow B) = P(AB) = support(AB) \)  （公式1）
    - 置信度 \( c = confidence(A\Rightarrow B) = P(B|A) = \frac{P(AB)}{P(A)} = \frac{support(AB)}{support(A)} \)  （公式2）
3. 强规则：同时满足最小支持度（min_sup）和最小置信度（min_conf）的关联规则。
4. 关联规则挖掘两步法：
    - 第一步：找出所有频繁模式集（核心难点）；
    - 第二步：由频繁模式集产生强规则（相对容易）。

### （三）示例说明
已知条件：最小支持度 50%，最小可信度 50%
以规则 \( A \Rightarrow C \) 为例：
- 支持度 \( support = support(\{A\cup C\}) = 50\% \)
- 置信度 \( confidence = \frac{support(\{A\cup C\})}{support(\{A\})} = 66.6\% \)

## 三、强规则的“趣味性”探讨
### （一）核心问题
所有同时满足最小支持度和最小置信度阈值的强关联规则，是否都足够有趣并值得向用户提供？

### （二）案例验证
已知：\( |T|=100 \)（总事务数），\( |play|=75 \)（包含play的事务数），\( |dinner|=60 \)（包含dinner的事务数），\( |play and dinner|=40 \)（同时包含play和dinner的事务数）
规则：\( dinner \Rightarrow play \)
- 支持度 = 40%，置信度 = 66.7%
- 设定：min_sup=30%，min_conf=60%
- 结论：该规则为强规则，但需结合实际场景判断是否“有趣”

### （三）相关分析（判断规则趣味性的补充方法）
1. 相关系数公式：\( Corr(A,B) = \frac{P(A \cup B)}{P(A) \times P(B)} \)
2. 判定标准：
    - 正相关：\( Corr>1 \)
    - 独立关系：\( Corr=1 \)
    - 负相关：\( Corr<1 \)

## 四、关联规则挖掘的课程核心内容框架
1. 基本概念
2. 案例介绍
3. 问题分类
4. 应用领域
5. 经典算法

## 五、典型案例介绍
1. 数据分析：NBA赛场
2. 内容推荐：今日头条
3. 购物推荐：Amazon
4. 广告推荐：百度／京东／天猫
5. 垃圾短信狙击
6. 信贷公司业务应用
7. 全球反恐场景应用

## 六、问题分类
### （一）按规则中处理的值类型分类
1. 布尔关联规则（Boolean Association Rule）：规则考虑的关联是项的“存在”与“不存在”
    - 示例：\( buys(X, diaper) \Rightarrow buys(X, beer) \)
2. 量化关联规则（Quantitative Association Rule）：规则描述量化的项或属性之间的关联
    - 示例：\( age(X, [30,39]) \cap income(X, [8K, 9K]) \Rightarrow buys(X, home) \)

### （二）按规则中涉及的数据维数分类
1. 单维关联规则（Singe-Dimensional Association Rule）：规则中的项或属性仅涉及一个维度
    - 示例：\( buys(X, Computer) \Rightarrow buys(X, Windows OS) \)
2. 多维关联规则（Multi-Dimensional Association Rule）：规则涉及两个或多个维度
    - 示例：\( age(X, [20, 29]) \cap income(X, [0, 300]) \Rightarrow robs(X, money) \)

### （三）按规则中涉及的抽象层分类
1. 单层关联规则（Single-Level Association Rule）：规则不考虑项的分层结构
    - 示例：\( buys(X, milk) \Rightarrow buys(X, bread) \)
2. 多层关联规则（Multi-Level Association Rule）：规则考虑项的分层结构
    - 示例：\( buys(X, food) \Rightarrow buys(X, fruit) \)

## 七、频繁模式挖掘的分类
1. 频繁模式挖掘（Frequent Patterns Mining）
2. 交互挖掘（Interactive Mining）
3. 增量挖掘（Incremental Mining）
4. 效用频繁模式挖掘（Utility FPs Mining）
5. 最大频繁模式挖掘（Maximal FPs Mining）
6. 频繁闭合模式挖掘（Closed FPs Mining）
7. 并行/分布式挖掘（Parallel/Distributed Mining）
8. 其他扩展类型

## 八、频繁模式挖掘的应用领域
1. 关联规则构建
2. 分类模型训练
3. 序列模式识别
4. 聚类预测分析
5. 机器学习算法支撑
6. 神经网络模型应用

## 九、经典算法
### （一）基于侯选项生成与测试（candidate generation and test）的方法
1. 核心原理：任何非频繁模式的超集都是非频繁的；任何频繁模式的子集都是频繁的
    - 示例：如果 {AB} 是频繁模式，则 {A}、{B} 也一定是频繁模式
2. 代表作：Apriori 算法（1994年提出）

### （二）基于分而治之的模式增长（pattern growth）方法
1. 核心原理：采用分而治之的方法学，将数据挖掘任务分解为多个小任务
    - 示例：如果“abcdef”是频繁集，当且仅当“abcde”是频繁集，且“f”在包含“abcde”的事务中是频繁的
2. 代表作：FP-growth 算法（2000年提出）
