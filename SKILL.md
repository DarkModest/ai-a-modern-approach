---
name: ai-a-modern-approach
description: "Knowledge base from \"人工智能：现代方法（第4版）\" (Artificial Intelligence: A Modern Approach, 4th ed.) by Stuart Russell & Peter Norvig. Use when studying or applying AI concepts: search (A*, minimax, CSP), logic & knowledge representation, probability (Bayesian networks, HMM), decision theory (MDP, POMDP, game theory), machine learning, deep learning, reinforcement learning, NLP, computer vision, robotics, or AI ethics/alignment (价值对齐). 中文问答与复习首选。"
---

<!-- argument-hint: [主题、框架名或章节号，如 "A*搜索"、"贝叶斯网络"、ch17] -->

# 人工智能：现代方法（第4版）
**作者**: Stuart Russell & Peter Norvig（斯图尔特·罗素、彼得·诺维格）| **章数**: 28（上下册）| **生成日期**: 2026-10-10

## 如何使用本技能

- **无参数** — 浏览下方核心框架速览
- **带主题** — 问 "A*搜索"、"贝叶斯网络"、"强化学习" 等，我会读对应章节文件再回答
- **带章节号** — 要 `ch17`，我直接加载该章
- **浏览** — 问 "有哪些章节？" 查看完整索引

问到的主题若不在下方速览中，我会先读相关章节文件再作答。

---

## 核心框架与思维模型

### 1. 理性智能体主线（全书的组织原则）
把 AI 定义为研究**理性智能体 (rational agent)** 的学科：对每个可能的感知序列，选择最大化期望性能度量的行动。
- 用 **PEAS**（性能/环境/执行器/传感器）定义智能体任务；用 8 个**任务环境维度**（完全/部分可观测、单/多智能体、确定/随机、片段式/序贯、静态/动态、离散/连续、已知/未知）决定设计选型。
- 智能体程序四层级：简单反射 → 基于模型 → 基于目标 → 基于效用；学习型智能体 = 性能+批评+学习元素+问题生成器。
- 表示三层级：原子（搜索）→ 因子化（CSP/贝叶斯网）→ 结构化（逻辑/一阶）——全书各部分对应不同层级。
- 第4版核心更新：**标准模型**（机器优化固定目标）不充分，AI 须对人类目标保持不确定并追求人类真正目标（价值对齐，Ch 1、27）。

### 2. 搜索范式（Ch 3–6）
- 一切搜索 = 按 f 反复扩展**边界**：f=g 一致代价、f=h 贪婪、f=g+h 即 **A\***。**h 可采纳 ⇒ 最优**；一致 ⇒ 图搜索无需重开节点；h 支配关系与 max 组合决定剪枝强度。
- 不要路径只要好状态 → **局部搜索**（爬山/退火/遗传），O(1) 内存；非确定环境 → **AND-OR 条件规划**；部分可观测 → **信念状态**搜索；未知环境 → 在线搜索 LRTA\*。
- 对抗 → **极小化极大 + α-β 剪枝 + 评价函数**；大分支 → **MCTS**；随机性 → 期望极小化极大。
- **CSP**：变量+约束的因子化让领域无关技术生效——MRV/度启发式、前向检验/MAC、冲突导向回跳；约束图**树宽**决定问题难度（树分解线性求解）。

### 3. 知识与推理（Ch 7–11）
- 基于知识的智能体：KB 语句 + 蕴含 + 推断。命题层用**模型检验/DPLL/归结**；一阶逻辑引入对象、关系、量词，一般规律一条语句搞定。
- 一阶推断三路径：**合一 + 前向/反向链接**（Prolog/产生式系统）与**归结**（反演完备、半可判定）——避免盲目命题化。
- 知识表示：上层本体论（类别/事件/时间/信念）+ 描述逻辑 + **非单调推理**（限定、缺省）+ 真值维护。
- 规划 = 因子化的搜索：**PDDL** 动作模式（前提+效果）自动导出领域无关启发式（松弛问题）；规模大用 SATPlan/HTN。

### 4. 不确定性推理（Ch 12–15）
- 概率 = 信念度的数学（柯尔莫哥洛夫公理；违反则可被 Dutch book 套利）；**决策论 = 概率 + 效用**。
- **条件独立性**是可计算性的钥匙：贝叶斯网络把联合分布压缩为局部 CPT 乘积。精确推断用**变量消元**（结构好则线性）；一般 NP 难 → 似然加权/吉布斯采样近似。
- 图判据：**D-分离**、马尔可夫毯；因果方向区分观察与干预（do 演算入门）。
- 时间维度：滤波 = 预测+更新递归；平滑 = 前向-反向；解码 = 维特比。线性高斯 → 卡尔曼滤波；一般 → 粒子滤波。
- 概率编程（PPL/RPM/OUPM）：模型写成生成程序，对象数不定也能推断。

### 5. 决策与博弈（Ch 16–18）
- **MEU 原则**：理性 = 最大化期望效用。vNM 公理给出效用存在性；风险态度由效用曲率刻画；**VPI** 决定是否值得收集信息。
- 序贯决策：**MDP** 的解是**策略**不是动作序列——贝尔曼方程 + 值迭代/策略迭代；部分可观测 → **POMDP** = 信念空间上的连续 MDP，最优行为内含信息收集。
- 多智能体：**纳什均衡**（无人单方面获益）预测交互结果；**机制设计**反向构造规则（VCG 说真话占优、拍卖、Shapley 值、合同网）。

### 6. 学习（Ch 19–22）
- 监督学习第一原理：目标是**泛化**不是训练一致性——奥卡姆剃刀 + 交叉验证 + 正则化（MDL/L1/L2）；偏差-方差分解。
- 模型族：决策树（信息增益）、线性/逻辑回归（梯度下降）、SVM（最大间隔+核）、kNN、集成（bagging 降方差 / boosting 降偏差）。
- 概率视角：贝叶斯学习（后验加权预测最优）→ 实践退化为 MAP/ML；隐变量用 **EM**；生成式 vs 判别式。
- 深度学习：计算图 + 反向传播 + softmax/交叉熵；CNN（权重共享）、残差（抗梯度消失）、LSTM、VAE/GAN；正则化靠暂退/批归一化/权重衰减。
- 强化学习：模型可采样 → ADP；不可 → **TD/Q 学习**（off-policy：Q←Q+α[r+γmax Q′−Q]）；探索用 GLIE；大状态 → 函数近似（DQN）/策略梯度/逆强化学习。

### 7. 感知、行动与机器（Ch 23–26）
- NLP 两条线：概率管线（n 元语法+平滑、PCFG/CYK、组合语义）→ 深度管线（词向量、注意力 seq2seq、Transformer、预训练-微调 / BERT-GPT 谱系）。
- 视觉 = **逆图形学**：成像正问题良定义、逆问题有歧义，靠先验与学习消歧（边缘→特征→CNN 分类/检测；双目视差恢复深度）。
- 机器人：构形空间运动规划（PRM/RRT）+ 概率滤波定位（蒙特卡罗）+ SLAM + 反馈控制（PID/计算扭矩），分层分解各击破。

### 8. 伦理与未来（Ch 27–28）
- 与其争论机器是否思考（图灵测试、中文屋），不如解决**价值对齐**：能力与目标正交、工具性收敛目标、奖励黑客都是真实风险。
- 可实现理性标准 = **有界最优性**（固定架构上的最好程序），而非完美理性；AI 审计（六组件）与辅助博弈是工程化出路。

---

## 章节索引

| # | 标题 | 关键框架 |
|---|------|----------|
| [ch01](chapters/ch01-introduction.md) | 绪论 | 四种AI定义象限, 图灵测试, 标准模型, 价值对齐 |
| [ch02](chapters/ch02-intelligent-agents.md) | 智能体 | PEAS, 任务环境维度, 四类智能体程序, 学习型智能体 |
| [ch03](chapters/ch03-problem-solving-search.md) | 通过搜索进行问题求解 | 问题形式化, 最佳优先搜索, A*/可采纳启发式, 加权A* |
| [ch04](chapters/ch04-search-complex-environments.md) | 复杂环境中的搜索 | 局部搜索/退火/遗传, AND-OR搜索, 信念状态, LRTA* |
| [ch05](chapters/ch05-adversarial-search-games.md) | 对抗搜索和博弈 | 极小化极大, α-β剪枝, 评价函数, MCTS |
| [ch06](chapters/ch06-csp.md) | 约束满足问题 | AC-3, 回溯+MRV/前向检验, 最少冲突, 树分解/树宽 |
| [ch07](chapters/ch07-logical-agents.md) | 逻辑智能体 | 知识库/蕴含, 模型检验, DPLL, 归结, wumpus世界 |
| [ch08](chapters/ch08-first-order-logic.md) | 一阶逻辑 | FOL语法/语义, 量词, 数据库语义, 知识工程7步法 |
| [ch09](chapters/ch09-fol-inference.md) | 一阶逻辑中的推断 | 合一, 一般化肯定前件, 前/反向链接, 归结, Rete |
| [ch10](chapters/ch10-knowledge-representation.md) | 知识表示 | 上层本体论, 事件演算, 描述逻辑, 缺省推理, TMS |
| [ch11](chapters/ch11-automated-planning.md) | 自动规划 | PDDL, 前向/回归搜索, 松弛启发式, SATPlan, HTN |
| [ch12](chapters/ch12-quantifying-uncertainty.md) | 不确定性的量化 | 概率公理, 贝叶斯法则, 条件独立, 朴素贝叶斯, MEU |
| [ch13](chapters/ch13-probabilistic-reasoning.md) | 概率推理 | 贝叶斯网络, 变量消元, D-分离, 似然加权, 吉布斯采样 |
| [ch14](chapters/ch14-prob-reasoning-over-time.md) | 时间上的概率推理 | HMM, 前向-反向, 维特比, 卡尔曼滤波, 粒子滤波 |
| [ch15](chapters/ch15-probabilistic-programming.md) | 概率编程 | RPM/OUPM, 生成程序, PPL, 提升推断, 数据关联 |
| [ch16](chapters/ch16-simple-decisions.md) | 做简单决策 | MEU, vNM效用公理, 多属性效用, 决策网络, VPI |
| [ch17](chapters/ch17-complex-decisions.md) | 做复杂决策 | MDP, 贝尔曼方程, 值迭代/策略迭代, 探索-利用, POMDP |
| [ch18](chapters/ch18-multiagent-decisions.md) | 多智能体决策 | 纳什均衡, 子博弈完美, VCG机制, Shapley值, 社会选择 |
| [ch19](chapters/ch19-learning-from-examples.md) | 样例学习 | 决策树, 交叉验证/正则化/MDL, PAC, SVM/核, 集成学习 |
| [ch20](chapters/ch20-learning-probabilistic-models.md) | 概率模型学习 | 贝叶斯/MAP/ML学习, 共轭先验, EM, 结构学习 |
| [ch21](chapters/ch21-deep-learning.md) | 深度学习 | 计算图/反向传播, CNN, 残差网络, LSTM, VAE/GAN |
| [ch22](chapters/ch22-reinforcement-learning.md) | 强化学习 | TD/Q学习, ADP, GLIE探索, 策略梯度, 函数近似/DQN |
| [ch23](chapters/ch23-nlp.md) | 自然语言处理 | n元语法+平滑, HMM词性标注, PCFG/CYK, 组合语义 |
| [ch24](chapters/ch24-deep-learning-for-nlp.md) | NLP中的深度学习 | 词嵌入, seq2seq+注意力, Transformer, 预训练-微调 |
| [ch25](chapters/ch25-computer-vision.md) | 计算机视觉 | 逆图形学, 边缘/特征, CNN分类/检测, 双目立体视觉 |
| [ch26](chapters/ch26-robotics.md) | 机器人学 | 构形空间, 蒙特卡罗定位, PRM/RRT, PID控制, SLAM |
| [ch27](chapters/ch27-philosophy-ethics-safety.md) | 哲学、伦理和安全性 | 中文屋, 价值对齐, 辅助博弈, 公平不可能定理 |
| [ch28](chapters/ch28-future-of-ai.md) | 人工智能的未来 | 有界最优性, 元推理, 可微编程, AGI讨论 |

## 主题索引

- **A\* / 启发式搜索** → ch03, ch04
- **贝叶斯法则 / 概率基础** → ch12
- **贝叶斯网络 / D-分离 / 变量消元** → ch13
- **贝尔曼方程 / 值迭代 / 策略迭代** → ch17
- **博弈论 / 纳什均衡 / 机制设计 / 拍卖** → ch18, ch05
- **CSP / 回溯 / 弧一致性 / 树宽** → ch06
- **深度学习 / CNN / RNN / 注意力 / Transformer** → ch21, ch24
- **决策网络 / 效用 / VPI / 风险厌恶** → ch16
- **EM / 隐变量 / 混合模型** → ch20
- **规划 / PDDL / HTN / SATPlan** → ch11
- **过滤 / 平滑 / HMM / 卡尔曼 / 粒子滤波** → ch14
- **过拟合 / 交叉验证 / 正则化 / 偏差-方差** → ch19
- **归纳逻辑编程 / 决策树 / SVM / 集成** → ch19
- **机器翻译 / 语言模型 / 词性标注 / 句法分析** → ch23, ch24
- **机器人 / SLAM / 运动规划 / 控制** → ch26
- **计算机视觉 / 物体识别 / 立体视觉** → ch25
- **强化学习 / Q学习 / 探索-利用 / DQN / 策略梯度** → ch22
- **可信度 / 归结 / 前向反向链接 / 合一** → ch07, ch09
- **蒙特卡罗树搜索 / 评价函数** → ch05
- **POMDP / 信念状态** → ch17, ch04
- **PEAS / 任务环境 / 智能体设计** → ch02
- **人工智能历史 / 学科基础** → ch01
- **图灵测试 / 中文屋 / 意识** → ch01, ch27
- **树分解** → ch06
- **搜索算法对比（BFS/DFS/UCS/IDS）** → ch03
- **网络安全 / 公平 / 对齐 / 存在风险** → ch27
- **一阶逻辑 / 知识表示 / 本体 / 非单调推理** → ch08, ch10
- **遗传算法 / 模拟退火 / 局部搜索** → ch04
- **语音识别 / 期望极小化极大** → ch05, ch14
- **智能体学习类型** → ch02, ch19–22

## 支持文件

- [glossary.md](glossary.md) — 全书关键术语（中英对照，按字母序）
- [patterns.md](patterns.md) — 18 个可复用技术模式（何时用/怎么做/权衡）
- [cheatsheet.md](cheatsheet.md) — 算法选择决策表与常见坑速查

---

## 范围与限制

本技能只覆盖原书内容（中文版上下册，28 章）。章节文件是从原文综合提炼的研读摘要，非逐字原文。原书约 3,196 幅插图（含伪代码框与图表）在文本提取中被丢弃，各章伪代码系从原书散文描述忠实重构。实际动手实现请结合项目代码与官方代码库 (aima-python)；超出本书范围的主题请直接询问或查相关技能。
