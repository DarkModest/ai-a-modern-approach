# 第8章 一阶逻辑 (First-Order Logic)

## 核心思想
一阶逻辑（FOL，亦称一阶谓词演算）在命题逻辑之上引入**对象、关系与量词**，使"所有国王都是人"这类一般规律用一条而非一百万条语句表达。表达能力来自本体论约定——世界由对象及对象间关系组成。

## 框架
- **一阶逻辑语法（项—原子—复合—量化语句）**
  - 何时使用：需要指代对象、表达对象间关系或对全体/部分对象做断言时
  - 如何做：常量命名对象；函数构成复合项（对象的名称，非子程序调用）；谓词+参数构成原子语句；再用联结词与 ∀/∃ 组合（BNF 见参考表）
- **一阶逻辑语义（model / domain / interpretation）**
  - 何时使用：判定语句在模型中的真值、理解蕴含为何不可枚举检验时
  - 如何做：模型 = 非空域 + 解释（常量→对象、谓词→元组集、函数→全函数）；原子为真 ⇔ 参数元组落在谓词所指关系中
- **数据库语义（database semantics）**
  - 何时使用：全部对象与事实已知、要让"列出的事实即全部事实"直白成立时（数据库、逻辑编程）
  - 如何做：叠加唯一名称假设、封闭世界假设、域闭包；模型数从天文数字降为有限
- **知识工程过程（knowledge engineering，7 步法）**
  - 何时使用：为特定论域（电路、wumpus、亲属关系）构建专用知识库
  - 如何做：①确定问题 ②收集知识 ③确定词汇表（本体论）④编码通用公理 ⑤编码实例 ⑥提出查询 ⑦调试（推理链中断处即缺失公理）

## 关键概念
- **合成性（compositionality）** — 语句含义是其各部分含义的函数；好的表示语言的必要性质。
- **本体论约定（ontological commitment）** — 逻辑对世界的假设：命题逻辑假设事实；一阶逻辑假设对象与关系；模糊逻辑假设 0–1 真实度。
- **认识论约定（epistemological commitment）** — 每个事实允许的知识状态：命题/一阶逻辑为真、假、未知；概率论为 0–1 信任度——与真实度不可混淆。
- **项（term）** — 指代对象的逻辑表达式：常量、变量或函数作用于项；无变量的项称基本项（ground term）。
- **原子语句（atomic sentence）** — 谓词符号后接一列项；等词 t₁=t₂ 表示两项指代同一对象。
- **全称量词（∀）/存在量词（∃）** — ∀x P 对每个对象的扩展解释均为真时为真；∃x P 至少一个为真即为真；∃! 表"恰好一个"。
- **解释（interpretation）与预期解释** — 固定所有符号指代物的映射；排除非预期模型是知识库的职责。
- **公理与定理（axiom / theorem）** — 公理是直接给出的语句（定义/部分刻画/直白事实）；定理被公理蕴含，缓存以省推导。

## 思维模型
- **∀ 配 ⇒，∃ 配 ∧**：全称量词用蕴涵只约束满足前提的对象；存在量词用合取确保各分句谈同一对象。∀x(King(x)∧Person(x)) 断言万物皆为国王（过强），∃x(Crown(x)⇒…) 被非王冠对象弄真（过弱）。
- **把模型看作"对象集 + 一张解释表"**（图 8-2 见完整示例）：模型比命题真值赋值丰富，故蕴含无法枚举检验——图 8-4 的情形中，仅用不超过 6 个对象就有 137 506 194 466 个模型。
- **量词顺序就是语义**：∀x∃y Loves(x,y)（每个人都有所爱）≠ ∃y∀x Loves(x,y)（有人被所有人爱）；嵌套时用不同变量名——变量属于最内层量词。

## 反模式
- **量词配错联结词**：∀x(King(x)∧Person(x)) 断言理查的左腿也是国王；∃x(Crown(x)⇒OnHead(x,John)) 只要存在非王冠对象即为真——最常见的翻译错误。
- **用 ∃x∃y 表达"至少两个"却忘了不等**：∃x,y Brother(x,Richard)∧Brother(y,Richard) 在 x、y 绑定同一对象时也为真；必须加 x≠y。
- **嵌套量词复用变量名**：∀x∃x Brother(Richard,x) 中外层 ∀x 是无效装饰，内层量词独占该变量。
- **以为存在"正确的"语义**：标准语义与数据库语义各有适用面；全部事实已知时数据库语义直白，否则会把未知误当为假。

## 算法要点
**语法构成**：完整 BNF 见参考表（图 8-3）。运算符优先级从高到低：¬、=、∧、∨、⇒、⇔；量词优先级高于其右侧一切。

**语义求值**：项 f(t₁,…,tₙ) 的指代物 = 函数值 F(d₁,…,dₙ)（dᵢ 为各项指代的对象）；原子 P(t₁,…,tₙ) 为真 ⇔ 〈d₁,…,dₙ〉 ∈ 谓词所指关系元组集；∀x P 在每个域元素的扩展解释下均真，∃x P 至少一个为真。函数须为全函数（无左腿之物用"不可见对象"补齐）。

**量词等价式（德摩根式对偶）**：¬∀x P ≡ ∃x ¬P；¬∃x P ≡ ∀x ¬P。连续同类量词可折叠：∀x∀y → ∀x,y。

**数据库语义三假设**：唯一名称假设 + 封闭世界假设 + 域闭包 ⇒ "列出的兄弟就是全部兄弟"；两常量 R、J 加二元关系时，4 个元组的子集共 2⁴=16 个模型（图 8-5），对比标准语义的天文数字。

**知识工程 7 步**：见框架；错误公理可独立于 KB 判真伪（如 ∀x,y Brother(x,y)⇒Opposite(x,y) 对桌子不成立）。

## 参考表
形式化语言的两类约定（据图 8-1）：

| 语言 | 本体论约定（世界由什么组成） | 认识论约定（知识状态） |
|---|---|---|
| 命题逻辑 | 事实 | 真 / 假 / 未知 |
| 一阶逻辑 | 对象、关系、函数 | 真 / 假 / 未知 |
| 时态逻辑 | 事实 + 有序时间 | 真 / 假 / 未知 |
| 高阶逻辑 | 关系与函数亦为对象 | 真 / 假 / 未知 |
| 模糊逻辑 | 有真实度的事实 | [0,1] 真实度 |
| 概率论 | 事实 | [0,1] 信任度 |

一阶逻辑 BNF 语法（图 8-3，含等价）：
```
语句     → 原子语句 | 复合语句
原子语句 → 谓词 | 谓词(项,…) | 项=项
复合语句 → (语句) | ¬语句 | 语句∧语句 | 语句∨语句 | 语句⇒语句 | 语句⇔语句 | 量词 变量,… 语句
项       → 函数(项,…) | 常量 | 变量
量词     → ∀ | ∃
常量     → A | X₁ | John | …
变量     → a | x | s | …
谓词     → True | False | After | Loves | Raining | …
函数     → Mother | LeftLeg | …
```

## 完整示例
**图 8-2 模型与真语句**：5 个对象——狮心理查、约翰国王、两人的左腿、王冠。二元关系：兄弟={〈理查,约翰〉,〈约翰,理查〉}、在头顶={〈王冠,约翰〉}；一元关系：人={理查,约翰}、国王={约翰}、王冠={王冠}；一元函数：左腿={〈理查〉↦理查左腿, 〈约翰〉↦约翰左腿, …}。真语句示例：¬Brother(LeftLeg(Richard),John)；Brother(Richard,John)∧Brother(John,Richard)；King(Richard)∨King(John)；¬King(Richard)⇒King(John)。

**亲属关系论域**（定义即公理，形如"⇔"）：母亲 ∀m,c Mother(c)=m ⇔ Female(m)∧Parent(m,c)；丈夫 ∀h,w Husband(h,w) ⇔ Male(h)∧Spouse(h,w)；父母-孩子互反 ∀p,c Parent(p,c) ⇔ Child(c,p)；祖父母 ∀g,c Grandparent(g,c) ⇔ ∃p Parent(g,p)∧Parent(p,c)；兄弟姐妹 ∀x,y Sibling(x,y) ⇔ x≠y ∧ ∃p Parent(p,x)∧Parent(p,y)。
**数的论域（皮亚诺公理）**：NatNum(0)；∀n NatNum(n) ⇒ NatNum(S(n))；∀m,n m=n ⇔ S(m)=S(n)；加法递归在首参数：∀m NatNum(m) ⇒ +(0,m)=m；∀m,n NatNum(m)∧NatNum(n) ⇒ +(S(m),n)=S(+(m,n))。四条公理即长出整个数论。

**wumpus 世界**（对比第 7 章的百万条命题公理）：
- 感知公理：∀t,s,g,w,c Percept([s,Breeze,g,w,c],t) ⇒ Breeze(t)；∀t,s,g,w,c Percept([s,None,g,w,c],t) ⇒ ¬Breeze(t)（Glitter 位同理两条）
- 相邻定义：∀x,y,a,b Adjacent([x,y],[a,b]) ⇔ (x=a ∧ (y=b−1∨y=b+1)) ∨ (y=b ∧ (x=a−1∨x=a+1))
- 微风公理（命题版 R2/R3 压缩为一条）：∀s Breezy(s) ⇔ ∃r Adjacent(r,s) ∧ Pit(r)
- 箭的后继状态公理：∀t HaveArrow(t+1) ⇔ HaveArrow(t) ∧ ¬Shoot(t)
- 动作查询：AskVars(KB, ∃a BestAction(a,5)) 返回 {a/Grab}

**电子电路论域**（7 步法示范；图 8-6 一位全加器 C₁：2 异或门、2 与门、1 或门，输入 i₁、i₂ 与进位 i₃，输出和位与进位）：
- 词汇表：谓词 Circuit/Gate/Terminal/Connected；函数 Type/In/Out/Signal/Arity；常量 0、1、AND、OR、XOR、NOT、Nothing。
- 12 条通用公理：①连通端子信号相同 ∀t₁,t₂ Connected(t₁,t₂) ⇒ Signal(t₁)=Signal(t₂)；②每个端子信号非 1 即 0；③Connected 可交换；④门类型恰 4 种；⑤与门 ∀g Gate(g)∧Type(g)=AND ⇒ Signal(Out(1,g))=0 ⇔ ∃n Signal(In(n,g))=0；⑥或门 ∀g Gate(g)∧Type(g)=OR ⇒ Signal(Out(1,g))=1 ⇔ ∃n Signal(In(n,g))=1；⑦异或门 ∀g Gate(g)∧Type(g)=XOR ⇒ Signal(Out(1,g))=1 ⇔ Signal(In(1,g))≠Signal(In(2,g))；⑧非门输出≠输入；⑨元数 ∀g Gate(g)∧Type(g)=NOT ⇒ Arity(g,1,1)，∀g Gate(g)∧k=Type(g)∧(k=AND∨k=OR∨k=XOR) ⇒ Arity(g,2,1)；⑩端子不超元数 ∀c,i,j Circuit(c)∧Arity(c,i,j) ⇒ ∀n (n≤i ⇒ Terminal(In(n,c))) ∧ (n>i ⇒ In(n,c)=Nothing) ∧ ∀n (n≤j ⇒ Terminal(Out(n,c))) ∧ (n>j ⇒ Out(n,c)=Nothing)；⑪门、端子、信号互异；⑫门是电路。
- 实例编码：Circuit(C₁)∧Arity(C₁,3,2)；Gate(X₁)∧Type(X₁)=XOR；Gate(X₂)∧Type(X₂)=XOR；Gate(A₁)∧Type(A₁)=AND；Gate(A₂)∧Type(A₂)=AND；Gate(O₁)∧Type(O₁)=OR；12 条连线 Connected(Out(1,X₁),In(1,X₂))、Connected(Out(1,X₁),In(2,A₂))、Connected(Out(1,A₂),In(1,O₁))、Connected(Out(1,A₁),In(2,O₁))、Connected(Out(1,X₂),Out(1,C₁))、Connected(Out(1,O₁),Out(2,C₁))、Connected(In(1,C₁),In(1,X₁))、Connected(In(1,C₁),In(1,A₁))、Connected(In(2,C₁),In(2,X₁))、Connected(In(2,C₁),In(2,A₁))、Connected(In(3,C₁),In(2,X₂))、Connected(In(3,C₁),In(1,A₂))。
- 查询①"哪些输入使和位 0、进位 1"：∃i₁,i₂,i₃ Signal(In(1,C₁))=i₁ ∧ Signal(In(2,C₁))=i₂ ∧ Signal(In(3,C₁))=i₃ ∧ Signal(Out(1,C₁))=0 ∧ Signal(Out(2,C₁))=1 → AskVars 返回 3 置换：{i₁/1,i₂/1,i₃/0}、{i₁/1,i₂/0,i₃/1}、{i₁/0,i₂/1,i₃/1}。
- 查询②求全输入输出表 ∃i₁,i₂,i₃,o₁,o₂ Signal(In(1,C₁))=i₁ ∧ … ∧ Signal(Out(2,C₁))=o₂ —— 即**电路验证**。
- 调试：忘断言 1≠0 时系统除输入 000、110 外无法证明输出；逐门查询 ∃i₁,i₂,o Signal(In(1,C₁))=i₁ ∧ Signal(In(2,C₁))=i₂ ∧ Signal(Out(1,X₁))=o 定位缺失公理。

## 要点
1. FOL 的简洁力来自对象+关系+量词：任意大小的 wumpus 世界约 10 条一阶语句，命题逻辑需上亿条。
2. 选语言先看两类约定：本体论约定决定能谈什么，认识论约定决定能信什么。
3. 标准语义下蕴含不可枚举（模型无穷），推断须走语法证明路线（第 9 章）。
4. 数据库语义用三个假设换直白性：适合事实齐全的封闭世界，不适合部分信息推理。
5. 知识工程的核心是第 3 步本体论决策：无底洞是对象还是谓词、朝向是函数还是谓词——风格决定公理成败。

## 关联
- **第7章**：命题逻辑的每方格/每时刻公理爆炸直接动机了一阶逻辑；wumpus 公理在此压缩为数条。
- **第9章**：合一、广义归结、前向/反向链接、Prolog 与数据库语义的正式使用。
- **第10章**：经典规划用 FOL 后继状态公理表示；通用本体的构建。
- **第12章**：概率论补足认识论维度（信任度），与 FOL 的三值知识状态互补。
- **外部概念**：关系数据库与 SQL（数据库语义）、Prolog/Datalog、描述逻辑（OWL 本体）、SMT 求解器。
