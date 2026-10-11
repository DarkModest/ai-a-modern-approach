# 第23章 自然语言处理 (Natural Language Processing)

## 核心思想
自然语言无法像形式语言那样做合法/非法的布尔判定，但可以用概率分布近似：语言模型给任意字符串赋概率，n 元统计、PCFG 句法分析与扩展文法语义解释构成"文本 → 句法树 → 逻辑形式"的经典概率管线。

## 框架
- **n 元语法模型（n-gram model）**：每个词的概率仅依赖前 n−1 个词（马尔可夫假设）
  - 何时使用：文本分类、语言识别、拼写校正、情感分析、作者归属等数据充足的统计任务
  - 如何做：语料库计数估计 + 平滑（回退/线性插值），低频词归并为 <UNK>
- **隐马尔可夫模型词性标注（HMM POS tagging）**：词性是隐藏状态、单词是观察序列的生成模型
  - 何时使用：序列标注，作为问答/翻译/语音合成的第一步（精度约 97%）
  - 如何做：转移概率 P(C_i|C_{i−1}) 与传感器概率 P(w|C) 来自平滑计数，维特比算法解码
- **概率上下文无关文法（PCFG）**：每条文法规则带概率，句法树概率 = 规则概率之积
  - 何时使用：需要层次短语结构并为多个分析排序
  - 如何做：CYK（乔姆斯基范式，O(n³m)）或 A*/束搜索/移位减少分析；可从树库计数学出
- **词汇化扩展文法（lexicalized/augmented grammar）**：非终结符带格、人称单复数、短语头（head）等参数
  - 何时使用：需要表达主谓一致、代词格、动词-宾语选择性等单词层知识
  - 如何做：NP(c, pn, n) 参数化；头对头的概率 P1(v, n) 配合平滑回退
- **组合语义解释（compositional semantics）**：短语语义 = 子短语语义的函数，用 λ 演算构造一阶逻辑形式
  - 何时使用：问答与语义分析（文法输出逻辑形式或答案）
  - 如何做：每条规则附语义变量，函数应用归约；可从句子-逻辑形式对或问答对中学习
- **歧义消解四模型**：世界模型 + 心理模型 + 语言模型 + 声学模型
  - 何时使用：文法概率只反映语料一般知识、而非当前情景时
  - 如何做：综合四类概率选最可能的解释

## 关键概念
**语言模型（language model）** — 描述任意字符串可能性的概率分布，是一切 NLP 任务的积木。
**词袋模型（bag-of-words）** — 朴素贝叶斯用于字符串：假设各类别下单词条件独立。
**平滑（smoothing）** — 为训练语料中未出现的 n 元保留一部分概率，以减小估计方差。
**未登录词（out-of-vocabulary）** — 测试中出现而训练中未见的词，用 <UNK>/<NUM>/<EMAIL> 等符号建模。
**跳字（skip-gram）** — 收集中间隔若干词的词对（如法语 "ne…pas"），捕获远程搭配信息。
**词性标注（part-of-speech tagging）** — 为每个词分配词汇范畴；Penn Treebank 定义 45 个标签。
**生成模型 vs 判别模型（generative/discriminative）** — 学联合分布 P(W,C) 可采样但错误率略高；判别模型学 P(C|W) 直接建模输出、易加特征。
**乔姆斯基范式（Chomsky Normal Form）** — 规则右侧恰为两个非终结符或一个终结符；任何 CFG 可自动转换，是 CYK 的前提。
**依存文法（dependency grammar）** — 用词汇项之间的二元关系表达句法、无成分结构；适合自由词序语言，Universal Dependencies 提供 70+ 语言标注。
**树库（treebank）** — 带句法分析树的句子的语料库（Penn Treebank 十万余句），对节点类型计数即可恢复 PCFG。
**准逻辑形式（quasi-logical form）** — 文法产出的中间表示，量词辖域由文法之外的优先规则决定。
**言语行为（speech act）** — 话语的意图类型（提问/命令/承诺/警告），属语用学。
**借代与隐喻（metonymy/metaphor）** — 用一对象代另一对象（Chrysler→其发言人）；隐喻是相似性版本的借代。

## 思维模型
- 把语言模型看作万能积木：补全、纠错、翻译、问答都归结为"选最高概率的字符串"。
- 当 n 增大时，把 n 元模型看作"更流畅但逐字复读训练语料"的补丁——真正的泛化需要结构（文法或词嵌入）。
- 把句法分析看作搜索：自顶向下/自底向上都会陷入死胡同回溯，图表（chart）动态规划存子串结果避免重复。
- 把歧义看作常态而非失败：几乎所有句子都有多种分析，消歧 = 文法概率 × 世界知识 × 说话者意图。

## 反模式
- **拉普拉斯（加 1）平滑用于大词表**：给海量未见事件错分概率质量，多数 NLP 任务效果差。
- **用纯 CFG 区分 I/me、banana/bandanna**：原子范畴表达不了格、一致性与词汇选择性——需扩展文法/词汇化概率。
- **贪心序列标注**：早期确定性选择无法被后面证据推翻——维特比或束搜索保留多假设。
- **在文法内硬编码量词辖域或借代**：规则组合爆炸——用准逻辑形式与文法外推理分层处理。
- **n 元模型做机器翻译**：n 再大信息也难从句子一端传到另一端，且学不到"英语形容词在名词前"这类组合规则。

## 算法要点
**n 元模型与平滑**
P(w_{1:N}) = ∏ P(w_i | 前 n−1 词)。计数估计后做线性插值回退：λ₃P(三元)+λ₂P(二元)+λ₁P(一元)，λ 固定或 EM 训练，也可随计数高低自适应；更精细的有 Witten-Bell、Kneser-Ney；"stupid backoff" 则靠海量语料配简单回退。句界加 <S>，低频词归并 <UNK>。

**HMM 词性标注（及判别式替代）**
转移 P(C_i|C_{i−1})（Penn Treebank 中 13124 个 MD 里 10471 个后接 VB，故 P(VB|MD)≈0.8）；传感器 P(w|C)（P(would|MD)=0.1）。维特比找最可能词性序列，精度约 97%。判别式：每词性一个逻辑斯谛回归模型，特征如 w_{i−1}="I"、w_i 以 "ous" 结尾、c_{i−1}=IN、w_{i−2} 属过去时等百万级稀疏二值特征，贪心左到右标注。

**CYK / PCFG 句法分析**
文法先转乔姆斯基范式（词法规则 X → word [p]；句法规则 X → Y Z [p]，右侧恰两范畴）。图23-5 CYK-Parse 逐行转录：

```
function CYK-PARSE(words, grammar) returns 句法分析树表
inputs: words 单词列表；grammar 含 LexicalRules 与 GrammarRules 结构
T ← 一个表    // T[X,i,k] 是涵盖 words_{i:k} 的最可能的 X 树
P ← 一个表，初始全为 0   // P[X,i,k] 表示树 T[X,i,k] 的概率
// 为每个单词插入词汇范畴
for i = 1 to LEN(words) do
    for each (X, p) in grammar.LEXICALRULES(words_i) do
        P[X,i,i] ← p
        T[X,i,i] ← TREE(X, words_i)
// 通过 Y_{i:j} + Z_{j+1:k} 构建 X_{i:k}，最短区间优先
for each (i, j, k) in SUBSPANS(LEN(words)) do
    for each (X, Y, Z, p) in grammar.GRAMMARRULES do
        PYZ ← P[Y,i,j] × P[Z,j+1,k] × p
        if PYZ > P[X,i,k] do
            P[X,i,k] ← PYZ
            T[X,i,k] ← TREE(X, T[Y,i,j], T[Z,j+1,k])
return T

function SUBSPANS(N) yields (i,j,k) 元组   // i≤j<k，按区间长度递增
for length = 2 to N do
    for i = 1 to N+1−length do
        k ← i+length−1
        for j = i to k−1 do
            yield (i, j, k)
```
空间 O(n²m)、时间 O(n³m)（n 词数、m 非终结符数）。加速：A* 搜索（状态=项目表，代价=已用规则概率倒数，可采纳启发式保证首个结果最优）；束搜索 O(n)；b=1 即确定性移位减少分析器（shift-reduce，逐词移栈或归约栈顶）。

**语义分析管道**
E0 文法 + 组合语义：算术文法给 "3+(4÷2)" 的树根标 Exp(5)；英语中 NP "Ali" → 逻辑项 Ali，VP "loves Bo" → 谓词 λx. Loves(x,Bo)，规则 S(pred(n)) → NP(n) VP(pred)，函数应用归约得 Loves(Ali,Bo)。扩展：时态用事件演算（loves ↔ During(Now, Extent(e))，loved ↔ After）。学习：Zettlemoyer & Collins 从句子-逻辑形式对学文法（79% 准确率）；问答对数据更易得，可训练更好的分析器（Liang et al.）。

## 参考表
句法分析算法对比：

| 算法 | 时间 | 保证 | 备注 |
|---|---|---|---|
| CYK / 图表分析 | O(n³m) | 全局最优 | 需乔姆斯基范式 |
| A* 搜索 | 通常快于 CYK | 最优（可采纳启发式） | 启发式从语料学习 |
| 束搜索（b>1） | O(n) | 近似最优 | b 调速度-精度 |
| 移位减少（b=1，确定性） | O(n) | 无保证 | 栈 + 移位/归约动作 |

平滑方法对比：

| 方法 | 思路 | 评价 |
|---|---|---|
| 拉普拉斯（加 1） | 均匀先验 | 方向正确但多数任务效果差 |
| 回退 / 线性插值 | 低计数高元回退到低元 | λ 可 EM 学习或随计数自适应 |
| Witten-Bell、Kneser-Ney | 更精细的折扣 | 更复杂 |
| stupid backoff | 简单回退 + 超大语料 | 目标相同：减方差 |

图23-2 E0 文法（每规则带概率与示例短语）：

| 左部 | 展开（概率，示例） |
|---|---|
| S | → NP VP [0.90] "I+feel a breeze" ｜ S Conj S [0.10] "I feel a breeze+and+It stinks" |
| NP | → Pronoun [0.25] I ｜ Name [0.10] Ali ｜ Noun [0.10] pits ｜ Article Noun [0.25] the+wumpus ｜ Article Adjs Noun [0.05] ｜ Digit Digit [0.05] 3 4 ｜ NP PP [0.10] ｜ NP RelClause [0.05] ｜ NP Conj NP [0.05] |
| VP | → Verb [0.40] stinks ｜ VP NP [0.35] feel+a breeze ｜ VP Adjective [0.05] ｜ VP PP [0.10] ｜ VP Adverb [0.10] |
| Adjs | → Adjective [0.80] ｜ Adjective Adjs [0.20] |
| PP | → Prep NP [1.00] to+the east |
| RelClause | → RelPro VP [1.00] that+is smelly |

图23-3 E0 词典（每类概率和为 1；名词/名称/动词/形容词/副词为开放类，其余封闭类）：

| 范畴 | 词条（概率） |
|---|---|
| Noun | stench [0.05]｜breeze [0.10]｜wumpus [0.15]｜pits [0.05]｜… |
| Verb | is [0.10]｜feel [0.10]｜smells [0.10]｜stinks [0.05]｜… |
| Article | the [0.40]｜a [0.30]｜an [0.10]｜every [0.05]｜… |
| Pronoun | me [0.10]｜you [0.03]｜I [0.10]｜it [0.10]｜… |
| 其余 | Adjective: right [0.10]、dead [0.05]…；Adverb: here [0.05]、ahead [0.05]…；RelPro: that [0.40]、which [0.15]…；Name: Ali/Bo [0.01]…；Prep: to [0.20]、in [0.10]、near [0.10]…；Conj: and [0.50]、but [0.20]…；Digit: 0–4 各 [0.20]… |

图23-1 Penn Treebank 词性标签（45 个，各带示例词）：动词 VB talk／VBD talked／VBG talking／VBN／VBP／VBZ talks；名词 NN kitten／NNS kittens／NNP Ali／NNPS Fords；JJ purple／JJR／JJS；RB quickly／RBR／RBS、小品词 RP off；PRP you／PRP$ your／WP who／WP$ whose；DT the／PDT all／WDT which；MD should；IN of；TO to；CC and；CD three；EX there；FW per se；LS 1；UH eureka；SYM +；POS 's；标点 $ # “ ” ( ) , : . 等。

词汇化 PCFG 片段（正文无编号图）：VP(v) → Verb(v) NP(n) [P1(v,n)] ｜ VP(v) → Verb(v) [P2(v)]；NP(n) → Article(a) Adjs(j) Noun(n) [P3(n,a)]；NP(n) → NP(n) Conjunction(c) NP(m) [P4(n,c,m)]；Verb(ate) → ate [0.002]；Noun(banana) → banana [0.0007]。P1 是 5000 动词×10000 名词的大表，靠平滑回退到仅依赖 v。

扩展文法片段（图23-9，省略概率）：S(v) → NP(Sbj,pn,n) VP(pn,v)；VP(pn,v) → Verb(pn,v) NP(Obj,pn,n)；PP(head) → Prep(head) NP(Obj,pn,h)；Pronoun(Sbj,1S,I) → I；Pronoun(Obj,1S,me) → me；Verb(3S,see) → see。

算术语义文法（图23-10）：Exp(op(x1,x2)) → Exp(x1) Operator(op) Exp(x2)；Exp(x) → ( Exp(x) )；Exp(x) → Number(x)；Number(10×x1+x2) → Number(x1) Digit(x2)；Operator(+) → + 等。

## 完整示例
E0 文法下 "Every wumpus smells" 的句法树（图23-6，内部节点标注规则概率）：

```
            S [0.90]
           /        \
      NP [0.25]      VP [0.40]
      /     \          |
Article[0.05] Noun[0.15] Verb[0.10]
    |          |         |
  Every     wumpus    smells
```
线性形式：[S [NP [Article every][Noun wumpus]] [VP [Verb smells]]]。整树概率 = 路径上规则概率之积 = 0.9 × 0.25 × 0.05 × 0.15 × 0.40 × 0.10 = 0.0000675。

图23-4 自顶向下分析 "The wumpus is dead" 的项目列表轨迹（每步应用一条规则替换最左非终结符）：
S →（S→NP VP）NP VP →（VP→VP Adjective）NP VP Adjective →（VP→Verb）NP Verb Adjective →（Adjective→dead）NP Verb dead →（Verb→is）NP is dead →（NP→Article Noun）Article Noun is dead →（Noun→wumpus）Article wumpus is dead →（Article→the）the wumpus is dead。

语义解释 "Ali loves Bo"（图23-12，λ 演算管线）：
1. 词汇：Name(Ali) → Ali；Verb(λy λx Loves(x,y)) → loves
2. 规则 VP(pred(n)) → Verb(pred) NP(n)：loves + Bo → λx. Loves(x, Bo)
3. 规则 S(pred(n)) → NP(n) VP(pred)
4. 函数应用归约：(λx. Loves(x, Bo))(Ali) ≡ Loves(Ali, Bo)

算术树对照（图23-11）："3 + (4÷2)" 根节点 Exp(5)，语义随句法自底向上组合。量词歧义示例："Every agent feels a breeze" 句法分析唯一，语义却有 ∀a∃b Feel(a,b) 与 ∃b∀a Feel(a,b) 两种——交给准逻辑形式处理。

## 要点
1. 先问"这个字符串概率多大"，再问"是否合法"——概率语言模型是各类 NLP 任务的通用积木与基准。
2. 低计数处方差大：平滑 + <UNK> 归并是 n 元模型的工程必需品。
3. 序列标注三档取舍：贪心（快、易错）→ 维特比（精确、慢）→ 束搜索（b 可调折中）。
4. 树库计数免费产出 PCFG；数据导向句法分析（data-oriented parsing）可自动泛化规则、压缩规则数量。
5. 扩展文法用参数化非终结符（格/人称/头）模块化表达一致性，优于把一切塞进概率表。
6. 消歧不能只靠语料概率：必须叠加世界模型与说话者心理模型。
7. n 元模型的天花板（长程信息、组合规则、逐字复读）正是第 24 章深度模型登场的动因。

## 关联
- **第 8 章**：一阶逻辑与 BNF 文法是语义表示与文法形式的基础。
- **第 14 章**：HMM 与维特比算法直接复用于词性标注。
- **第 12 / 19 章**：词袋 = 朴素贝叶斯；判别式标注 = 逻辑斯谛回归 + 特征工程。
- **第 21 / 24 章**：深度学习（词嵌入、RNN、Transformer）系统性解决本章模型的泛化与长程上下文局限。
- **外部概念**：Penn Treebank 与 Universal Dependencies 语料标准；Chomsky 范式与 λ 演算（形式语义学传统）；机器翻译、信息检索（搜索引擎）与信息抽取（NELL）等领域应用。
