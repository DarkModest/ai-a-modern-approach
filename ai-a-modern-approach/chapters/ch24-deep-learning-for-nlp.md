# 第24章 自然语言处理中的深度学习 (Deep Learning for Natural Language Processing)

## 核心思想
把单词从原子记号换成可学习的稠密向量（词嵌入），用 RNN/LSTM 捕获序列上下文，用带注意力的序列到序列模型与 Transformer 完成翻译和生成，再以"海量未标注文本预训练 + 少量数据微调"的迁移学习范式通吃各类 NLP 任务。

## 框架
- **词嵌入（word embedding）**：每个词对应低维稠密向量，相似词在空间中邻近
  - 何时使用：一切 NLP 任务的输入表示（取代独热编码）
  - 如何做：随机初始化 v×d 嵌入矩阵 E 随任务端到端训练；或直接用 word2vec/GloVe/fastText 预训练向量
- **循环神经网络语言模型（RNN language model）**：隐藏状态跨时间步传递，参数 O(1) 且各位置权重共享
  - 何时使用：需要不定长上下文的下一词预测、序列标注、分类
  - 如何做：输入词嵌入，输出 softmax 词分布；训练目标 = 预测偏移一位的下一词，通过时间反向传播（BPTT）
- **长短期记忆（LSTM）**：带门控单元的 RNN，可选择记忆/遗忘，跨长距离原样搬运信息
  - 何时使用：主语与动词远隔时的单复数一致等长程依赖
- **注意力序列到序列模型（attentional seq2seq）**：编码器-解码器架构，解码每步用注意力加权读取全部源隐状态
  - 何时使用：机器翻译及几乎一切"序列→序列"任务（问答、摘要、看图说话）
  - 如何做：上下文向量 c_i = Σ_j a_ij·s_j，a_ij 为 softmax 归一化的注意力概率；解码用束搜索
- **Transformer（自注意力架构）**：Q/K/V 三投影 + 缩放点积自注意力 + 前馈层 + 残差连接 + 位置嵌入，全并行计算
  - 何时使用：预训练大模型（BERT/GPT/T5 谱系）的骨架；需要远距上下文与硬件效率
- **预训练与迁移学习（pretraining & transfer learning）**：掩码语言模型等无监督目标在海量通用文本上训练，再用少量领域数据微调
  - 何时使用：标注昂贵而未标注文本充足（NLP 的常态）

## 关键概念
**独热向量（one-hot vector）** — 词表大小的稀疏编码，无法表达单词间相似性。
**向量类比（vector analogy）** — D = C + (B−A) 可解 "Athens 之于 Greece 如 Oslo 之于 ?" → Norway；向量差编码关系。
**双向 RNN（bidirectional RNN）** — 左到右与右到左两个模型的隐状态拼接，利用右侧上下文。
**平均池化（average pooling）** — 对全部隐向量取元素均值，聚合出句子级表示（用于情感分析等）。
**梯度消失（vanishing gradient）** — 信息沿时间步逐层衰减，如"传话游戏"；LSTM 门控是对策。
**邻近上下文偏差（nearby context bias）** — RNN 隐状态更新时新信息挤掉旧信息，偏向近期、丢失远距。
**固定上下文大小限制（fixed context size limit）** — 整个源句被压进定长隐向量（约 1024 维）的瓶颈。
**注意力（attention）** — 无自身可学参数、可微、可解释的加权平均组件；概率常对应翻译词对齐。
**自注意力（self-attention）** — 序列对自身的 Q/K/V 注意力；与词序无关、可矩阵并行。
**多头注意力（multiheaded attention）** — m 组独立注意力并行后拼接，让重要子块在结果中脱颖而出。
**位置嵌入（positional embedding）** — 为每个位置学习的向量，补回自注意力丢失的词序信息。
**掩码语言模型（MLM）** — 掩住输入词再让模型预测，无监督习得双向上下文表示（图24-12："The river [masked] five feet" 经两层双向 RNN，被掩位输出接专门的 MLM 嵌入 → 前馈 → 预测 "rose"）。
**上下文表示（contextual representation）** — 词向量随上下文变化（图24-11：左到右 RNN 每步输出当前词的上下文表示并预测下一词），使多义词 rose 的花义与升起义分开聚类。

## 思维模型
- 把词嵌入看作"压缩的共现统计"：10^25 维稀疏 n 元计数压成几百维稠密空间，换来泛化能力。
- 当 RNN 像传话游戏丢信息时：用 LSTM 门控修补，或干脆用注意力建立到任意源词的直连。
- 把注意力概率矩阵看作"软对齐"：翻译中每行目标词对每列源词的权重，颜色越深≈人类给出的对齐。
- 把一切 NLP 任务编码为 seq2seq：问答 = 问题序列 + 答案序列，摘要 = 长文→短文；T5 干脆把任务名写进输入前缀（"translate English to German: That is good" → "Das ist gut"）。

## 反模式
- **贪心解码做生成**：每步取最高概率词、无纠错机制，"La entrada"式局部最优毁掉整句；应使用束搜索。
- **把 RNN 隐状态当无限记忆**：1024 维装不下 64 词句子（每词仅摊 16 维），扩维又训练慢且过拟合——注意力才解开此结。
- **依赖一次一词的顺序处理**：RNN 无法批量并行，浪费矩阵加速硬件——这正是 Transformer 的动机之一。
- **单向量表示多义词**：rose 无法同时邻近 dahlia 与 upsurge——需要上下文/掩码表示。
- **用原始点积做自注意力**：向量与自身的点积恒高，导致偏向自关注；必须做 Q/K/V 三投影并加缩放因子。
- **拿类比演示当嵌入质量证明**：嵌入的价值在于下游任务表现，而非能否解类比题。

## 算法要点
**词嵌入 + 前馈词性标注器（24.1 的完整流程）**
(1) 取窗口宽 w=5，语料切成重叠窗口，中间词的词性为标签；(2) 词表 = 出现≥5 次的词，共 v 个；(3)–(4) 排序词表、选嵌入维度 d；(5) 建嵌入矩阵 E∈R^{v×d}，随机初始化或用预训练向量；(6) 搭两层隐藏 + softmax 网络，输出中间词在 45 个词性上的分布；(7) 窗口内 5 个嵌入拼接成 5d 维输入——每个位置连到第一层不同权重，隐式编码相对位置；(8) 梯度下降同时训练 E 与各层权重，凭 "yesterday"“they” 等证据把 "cut" 标为过去时动词。

**GloVe 预训练词向量**
统计窗口内共现次数 X_ij（窗口约 5 词），P_ij = X_ij/X_i。核心直觉：比值 P(ice,w)/P(steam,w) 在 w=solid 时高、w=gas 时低、w=the 时≈1，从而区分词义。数学推导得约束 E_i·E_j = log P_ij（点积 = 共现概率的对数）。每个词学两个向量最后相加以防过拟合；普通台式机 CPU 数小时即可从几十亿词训练完成。领域版本可还原知识：材料科学语料的嵌入能预测未被发现的热电材料。

**RNN/LSTM：语言模型、分类与生成**
结构（图24-4）：隐藏层 z 带延迟 Δ 的自环；z_t 经权重 w_xz 接词嵌入 x_t、经 w_zz 接 z_{t−1}，输出 y_t = softmax(w_zy·z_t)；按时间展开（z_0→z_1→z_2→z_3）即所有时间步共享权重的前馈网。双向 RNN 词性标注（图24-5）：每位置左到右与右到左两个 RNN 的隐向量拼接 → 前馈 → 词类；"Yesterday they cut the rope" → Adverb / Pronoun / PastTenseVerb / Determiner / Noun。训练：x_t 为词嵌入，观察目标是右移一位的词序列，BPTT 且所有时间步共享权重。生成：从输出分布采样→反馈为下一输入→迭代（采样策略是超参数）。标注/共指分类：每步一个 softmax（输出词性或先行词分布）；共指需双向 RNN 看全句。句子级分类（情感分析）：取末位隐状态或平均池化聚合后接前馈层。LSTM 门控可在 "The athletes, who …, now ___" 中把复数特征原样前传，正确选 "compete" 而非 "competes"。

**注意力 seq2seq 与解码**
编码器读完源句，其最终隐状态作为解码器初值；<start>/<end> 标记边界（图24-6：源 LSTM 链读 "The man is tall" + <start>，目标链生成 "El hombre es alto" + <end>）。注意力计算（图24-7a 虚线）：r_ij = h_{i−1}·s_j（目标状态与源隐状态点积得分）→ a_ij = e^{r_ij} / Σ_k e^{r_ik} → c_i = Σ_j a_ij·s_j → 解码器 h_i = RNN(h_{i−1}, [x_i, c_i])（无注意力时 h_i = RNN(h_{i−1}, x_i)）。图24-7b 注意力矩阵：行=源词 The/front/door/is/red，列=目标词 La/puerta/de/entrada/es/roja，每列概率和为 1，深色格即词级对齐 La↔The、puerta↔door、es↔is、roja↔red；entrada 兼看 front 与 door，对应西语语序反转。softmax 三作用：使注意力可微（梯度可回传）、允许捕获远距上下文、可表示不确定性。解码：贪心（每步一个选择）或束搜索（每步保留 k 个最优假设；现代 NMT 用 k=4–8，老式统计 MT 需 ≥100）。

**Transformer 层**
自注意力（精确式）：q_i = W_q x_i（注意力来源）、k_i = W_k x_i（去向）、v_i = W_v x_i（内容）；r_ij = (q_i·k_j)/√d，a_ij = e^{r_ij} / Σ_k e^{r_ik}，c_i = Σ_j a_ij·v_j；缩放因子 √d 稳数值，注意力不对称（r_ij ≠ r_ji）。多头：句子的 m 个副本各用独立权重做注意力再拼接（拼接而非求和，让重要子块脱颖而出）。单层结构（图24-9）：输入向量 → 自注意力 → 残差相加 → 逐位置前馈（ReLU）→ 残差相加 → 输出向量；层数常 ≥6。词性标注管线（图24-10）：词嵌入 + 位置嵌入_i 相加 → 3 层 Transformer → 每位置前馈 + softmax。解码器另加因果自注意力（只看左侧）与对编码器输出的注意力模块。

## 参考表
表示与模型演进对比：

| 模型 | 有效上下文 | 参数量级 | 主要缺陷 |
|---|---|---|---|
| n 元模型 | 前 n−1 词 | O(vⁿ) | 长程断裂、逐字复读训练数据 |
| 前馈窗口网络 | 固定 w 词窗口 | O(n) | 窗口有限、位置不对称 |
| RNN | 理论无限 | O(1) | 传话游戏式信息衰减 |
| LSTM（门控） | 长程 | O(1) | 邻近偏差仍存、顺序处理慢 |
| seq2seq + 注意力 | 全源句 | — | 解开定长瓶颈；仍受限于 RNN 骨架 |
| Transformer | 全句、可并行 | — | 上下文窗口仍限于几百词 |

seq2seq 三大缺陷 vs 对策：邻近上下文偏差（注意力可直达任意源词）、固定上下文大小限制（加权平均取代单向量）、缓慢顺序处理（Transformer 根治）。

解码方式对比：贪心解码（快、无纠错）vs 束搜索（保留 k 个假设、牺牲速度换全局概率）。

## 完整示例
**词嵌入可视化与类比（图24-1、24-2）**：60 亿词训练的 100 维 GloVe 向量投影到二维后自然聚类——{france, greece, germany}、{nephew, niece, uncle, aunt}、{car, bicycle, truck}、{banana, pizza, apple}。类比 D = C + (B − A) 取最近邻的结果（A / B / C / D / 关系）：Athens / Greece / Oslo / Norway / 首都；Astana / Kazakhstan / Harare / Zimbabwe / 首都；Angola / kwanza / Iran / rial / 货币；copper / Cu / gold / Au / 元素符号；Microsoft / Windows / Google / Android / 操作系统；New York / New York Times / Baltimore / Baltimore Sun / 报纸；Berlusconi / Silvio / Obama / Barack / 名；Switzerland / Swiss / Cambodia / Cambodian / 国籍；Einstein / scientist / Picasso / painter / 职业；brother / sister / grandson / granddaughter / 亲属关系；Chicago / Illinois / Stockton / California / 州；possibly / impossibly / ethical / unethical / 否定；mouse / mice / dollar / dollars / 复数；easy / easiest / lucky / luckiest / 最高级；walking / walked / swimming / swam / 过去式。

**束解码数值轨迹（图24-8，b=2，词得分=softmax 对数概率，假设得分=累加和）**：译 "The front door is red" → "La puerta de entrada es roja"。t1：La −0.3｜Una −2.1。t2：La entrada −1.1｜La puerta −1.2（Una 支掉队）。t3：La entrada 只能续 de −1.5 / puerta −1.9，而 La puerta 续 de −0.5 / del −0.7 → 束变为 La puerta de −1.7 与 La puerta del −1.9——贪心路径 "La entrada"（西语名词须先于修饰语）从束上脱落。全部生成 <end> 后输出得分最高的假设。

## 要点
1. 独热→词嵌入是 NLP 深度化的第一次免费午餐：相似词共享统计结构，"a black cat" 的证据即可泛化到 "the fulvous kitten"。
2. RNN 用 O(1) 参数换来不定长上下文，但记忆有损；LSTM 门控是修补，注意力是根治。
3. 注意力本身没有可学参数，却因 softmax 而可微、可回传梯度，且概率可解释为对齐——这是它能量产洞察的原因。
4. 生成质量靠搜索：束解码的假设数 k 是精度-速度旋钮（现代模型 4–8 已够）。
5. 自注意力必须配套：Q/K/V 三投影防自关注偏差、缩放因子稳数值、位置嵌入补词序、多头让关键信息脱颖而出。
6. 预训练让无标注文本自带标签（下一词、被掩词），是 2018 年"NLP 的 ImageNet 时刻"的引擎；RoBERTa 训 2.2 万亿词仍能涨分，数据越多越好。
7. 数据驱动模型取代手工文法的理由很朴素：更易开发维护、基准分更高；混合方法（注意力 + 传统句法分析器，如 Kitaev & Klein、SLING）可能是回摆方向。

## 关联
- **第 21 章**：RNN、LSTM、梯度下降、迁移学习等全部工具来自深度学习基础章。
- **第 23 章**：本章是对 n 元模型（泛化差）与手写文法（难维护）局限的直接回应；本章模型也被反过来改进传统句法分析。
- **外部概念**：word2vec/GloVe/fastText 词向量库；BERT/RoBERTa（掩码预训练）、GPT-2（15 亿参数自回归生成）、T5（text-to-text 统一接口）谱系；机器翻译史（Sutskever seq2seq → Vaswani "Attention is all you need"）；Aristo 系统（八年级科学考试 91.6 分）与 Winograd 模式挑战。
