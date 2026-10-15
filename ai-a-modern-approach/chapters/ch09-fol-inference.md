# 第9章 一阶逻辑中的推断 (Inference in First-Order Logic)

## 核心思想
一阶推断的核心技巧是用合一（unification）把命题推断规则"提升"到带变量的形式——只在必要时才实例化变量，而非盲目命题化；前向链接、反向链接、归结分别支撑演绎数据库/产生式系统、逻辑编程（Prolog）和完备的定理证明。

## 框架
- **一般化肯定前件（Generalized Modus Ponens）**：一切确定子句推断的引擎
  - 何时使用：所有基于确定子句（前件为正文字合取、后件为单个正文字）的推断
  - 如何做：若有 p₁′…pₙ′ 与 ∀v (p₁∧…∧pₙ ⇒ q)，存在置换 θ 使 Subst(θ, pᵢ′) = Subst(θ, pᵢ) 对所有 i 成立，则推断 Subst(θ, q)
- **前向链接（FOL-FC-Ask）**：数据驱动
  - 何时使用：事实持续到达、需维护全部结论（演绎数据库、产生式系统）
  - 如何做：规则前提被事实满足即触发，结论入 KB，直至答出查询或不动点（图 9-3）
- **反向链接（FOL-BC-Ask）**：目标驱动
  - 何时使用：只答特定查询、规则远多于事实（Prolog 型场景）
  - 如何做：用可与目标合一的子句递归证明前件（与或搜索），累积传递置换（图 9-6）
- **归结（Resolution）**：唯一适用于任意（非仅确定子句）KB 的反演完备方法
  - 何时使用：需要完备性、或 KB 非霍恩形式（定理证明、验证与合成）
  - 如何做：KB 与查询的否定转为 CNF（含斯科伦化），反复归结并因子提取，直至推出空子句（反证法）
- **Rete 算法**：增量前向链接
  - 何时使用：规则数巨大、事实持续更新（Xcon、Ops-5）
  - 如何做：规则集预处理为数据流网络，网络状态保存全部部分匹配，避免重复计算

## 关键概念
- **全称量词实例化（UI）/存在量词实例化（EI）** — UI 用基本项置换全称变量，可反复应用；EI 用新常量（斯科伦常量）替换存在变量，一次性应用。
- **命题化（Propositionalization）** — 枚举所有基本项实例化把一阶 KB 转为命题 KB；埃尔布朗定理保证若有证明则有限子集足够，但函数符号导致无穷项空间。
- **半可判定（Semidecidable）** — 存在能证明所有被蕴含语句的算法，但不存在判定"不被蕴含"的算法（同停机问题）。
- **合一（Unification）与最一般合一子（MGU）** — 求使两表达式相同的置换；MGU 在变量重命名意义下唯一。
- **出现检验（Occur Check）** — 检查变量是否出现在待匹配项内（S(x) 与 S(S(x)) 不可合一）；Prolog 略去它换取速度（可能不可靠）。
- **标准化分离（Standardizing Apart）** — 合一前对两语句之一的变量重命名，避免同名变量冲突。
- **数据日志（Datalog）** — 无函数符号的一阶确定子句语言；蕴含可判定，前向链接对它完备且多项式时间。
- **斯科伦化（Skolemization）** — CNF 转换中消去存在量词：存在变量换成以辖域内全部全称变量为参数的新函数。

## 思维模型
- 事实多、要全部后果时用前向链接；规则多、只有一个查询时用反向链接。
- 归结 ="命题化 + 反证"的智能版：不预先实例化，而在归结时用合一即席实例化。
- 深度优先反向链接的死循环 = 重复状态问题：解法是缓存（表格化），即引入前向链接的动态规划思想。

## 反模式
- **盲目命题化**：函数符号生成无穷嵌套项 Father(Father(Father(John)))…；查询无答案时永不停止。
- **子句顺序不当的深度优先反向链接（图 9-8/9-9 的教训）**：path(X,Z):-link(X,Z) 与 path(X,Z):-path(X,Y),link(Y,Z) 配事实 link(a,b)、link(b,c)，正确子句顺序下 path(a,c) 可证（图 9-9a），递归子句居前则生成无限证明树（图 9-9b）——Prolog 深度优先且不查无限递归，对确定子句也不完备。图 9-8b 分层随机图上找 A₁→J₄ 需 877 次推断（多为重推不可达节点，重复状态问题），前向链接仅需 62 次；缓存子问题的表格化逻辑编程结合两者优点，对数据日志完备。
- **略去出现检验**：换来效率但产生不可靠推断；Prolog 的自觉取舍，不应在通用定理证明器中模仿。
- **朴素前向链接**：每轮用每条规则匹配每条事实、生成与目标无关的事实。应做增量匹配（只碰上一轮新事实触发的前提）、用 Rete 保留部分匹配、用魔法集限定相关绑定。

## 算法要点
**合一 Unify（图 9-1）**：
```
function Unify(x, y, θ=empty) returns 使 x 和 y 相同的置换，或 failure
  if θ = failure then return failure
  else if x = y then return θ
  else if Variable?(x) then return Unify-Var(x, y, θ)
  else if Variable?(y) then return Unify-Var(y, x, θ)
  else if Compound?(x) and Compound?(y) then
      return Unify(Args(x), Args(y), Unify(Op(x), Op(y), θ))
  else if List?(x) and List?(y) then
      return Unify(Rest(x), Rest(y), Unify(First(x), First(y), θ))
  else return failure

function Unify-Var(var, x, θ) returns 一个置换
  if 对某些 val 有 {var/val} ∈ θ then return Unify(val, x, θ)
  else if 对某些 val 有 {x/val} ∈ θ then return Unify(var, val, θ)
  else if Occur-Check?(var, x) then return failure
  else return 将 {var/x} 添加到 θ
```

**前向链接 FOL-FC-Ask（图 9-3）**：
```
function FOL-FC-Ask(KB, α) returns 一个置换或 false
inputs: KB，知识库，一个一阶确定子句集
        α，查询，一个原子语句
while true do
  new ← {}        // 每次迭代推断出的新语句集
  for each rule in KB do
    (p₁ ∧ … ∧ pₙ ⇒ q) ← Standardize-Variables(rule)
    for each θ 使得对 KB 中某些 p₁′,…,pₙ′ 有 Subst(θ, p₁∧…∧pₙ) = Subst(θ, p₁′∧…∧pₙ′) do
      q′ ← Subst(θ, q)
      if q′ 不能与已在 KB 或 new 中的语句合一 then
        将 q′ 添加到 new
        φ ← Unify(q′, α)
        if φ 不为 failure then return φ
  if new = {} then return false
  将 new 添加到 KB
```

**反向链接 FOL-BC-Ask（图 9-6）**：
```
function FOL-BC-Ask(KB, query) returns 置换生成器
  return FOL-BC-Or(KB, query, {})

function FOL-BC-Or(KB, goal, θ) returns 一个置换
  for each rule in Fetch-Rules-For-Goal(KB, goal) do
    (lhs ⇒ rhs) ← Standardize-Variables(rule)
    for each θ′ in FOL-BC-And(KB, lhs, Unify(rhs, goal, θ)) do
      yield θ′

function FOL-BC-And(KB, goals, θ) returns 一个置换
  if θ = failure then return
  else if Length(goals) = 0 then yield θ
  else
    first, rest ← First(goals), Rest(goals)
    for each θ′ in FOL-BC-Or(KB, Subst(θ, first), θ) do
      for each θ″ in FOL-BC-And(KB, rest, θ′) do
        yield θ″
```

**CNF 转换六步**（例：∀x [(∀y Animal(y)⇒Loves(x,y)) ⇒ ∃z Loves(z,x)]）：①消蕴涵；②¬内移；③变量标准化；④斯科伦化 → (Animal(F(x)) ∧ ¬Loves(x,F(x))) ∨ Loves(G(x),x)；⑤删全称量词；⑥∨对∧分配 → 两子句 Animal(F(x))∨Loves(G(x),x) 与 ¬Loves(x,F(x))∨Loves(G(x),x)。

**归结规则**：子句 C₁∨A 与 ¬B∨C₂，若 A 与 B 可合一（MGU θ），得 Subst(θ, C₁∨C₂)。二元归结单独不完备，必须配**因子提取**（子句内可合一的重复文字合一为一个）；两者组合反演完备。

**等词处理三法**：(1) 公理化等词（自反/对称/传递 + 每个谓词、函数一条置换公理）——可靠但产生大量无用结论；(2) 解调（用 x=y 定向重写，如 z+0→z）、超解调（在可合一处以等词置换，完备）；(3) 等词合一（为交换律、结合律等定制）。

**归结策略**：单元优先（对霍恩子句完备）；支撑集（归结总涉及含否定查询的支撑集；其余子句可满足则完备，且产生目标导向证明树）；输入归结（每步一个输入子句，对霍恩 KB 完备）；线性归结（完备）；包容（删除被蕴含的更具体子句）；学习（DeepHOL 用神经网络嵌入选证明步骤，至少需 10 000 条证明样本）。

## 参考表

| 方法 | 完备性 | 适用 KB | 典型系统 | 主要代价 |
|---|---|---|---|---|
| 命题化+命题推断 | 完备（但半可判定）| 任意 FOL | — | 无穷项空间，极慢 |
| 前向链接 | 确定子句完备；数据日志多项式 | 确定子句/数据日志 | 演绎数据库、Xcon/Ops-5（Rete）| 无关事实膨胀 |
| 反向链接 | 深度优先时不完备 | 确定子句 | Prolog、表格化逻辑编程 | 冗余推断、死循环 |
| 归结+因子提取 | 反演完备 | 任意 FOL | Otter、软硬件验证器 | 搜索空间大，需策略 |

Prolog 偏离纯逻辑之处：数据库语义（唯一名称+封闭世界）；内置算术按代码执行；assert/retract 副作用；无出现检验；按子句顺序深度优先且不查无限递归。

## 完整示例
**犯罪知识库（West 上校，贯穿全章，数据日志）**：∀x,y,z American(x)∧Weapon(y)∧Sells(x,y,z)∧Hostile(z) ⇒ Criminal(x)；Missile(M₁)、Owns(Nono,M₁)（∃ 实例化为新常量 M₁）；∀x Missile(x)⇒Sells(West,x,Nono)；∀x Missile(x)⇒Weapon(x)；∀x Enemy(x,America)⇒Hostile(x)；American(West)；Enemy(Nono,America)。

**前向链接证明树（图 9-4）**：底层为初始事实 American(West)、Missile(M₁)、Owns(Nono,M₁)、Enemy(Nono,America)；中层为第 1 轮推断 Weapon(M₁) ← Missile(M₁)，Sells(West,M₁,Nono) ← Missile(M₁)∧Owns(Nono,M₁)，Hostile(Nono) ← Enemy(Nono,America)；顶层为第 2 轮 Criminal(West) ← 四前提合取，随后到达不动点。

**反向链接证明树（图 9-7，深度优先从左到右读，成功绑定标在节点旁）**：Criminal(West) 分解为 4 个子目标——American(West) 命中 {}；Weapon(y) 回溯 Missile(y) 得 {y/M₁}，Missile(M₁) 命中 {}；Sells(West,M₁,z) 得 {z/Nono}，子目标 Missile(M₁)、Owns(Nono,M₁) 命中 {}；Hostile(Nono) 回溯 Enemy(Nono,America) 命中 {}。

**归结证明（图 9-10，输入归结的单条"主线"，对应反向链接的目标序列；⊕ 表示归结）**：
1. ¬American(x)∨¬Weapon(y)∨¬Sells(x,y,z)∨¬Hostile(z)∨Criminal(x) ⊕ ¬Criminal(West) → ¬American(West)∨¬Weapon(y)∨¬Sells(West,y,z)∨¬Hostile(z)
2. ⊕ American(West) → ¬Weapon(y)∨¬Sells(West,y,z)∨¬Hostile(z)
3. ⊕ ¬Missile(x)∨Weapon(x) → ¬Missile(y)∨¬Sells(West,y,z)∨¬Hostile(z)
4. ⊕ Missile(M₁) → ¬Sells(West,M₁,z)∨¬Hostile(z)
5. ⊕ ¬Missile(x)∨¬Owns(Nono,x)∨Sells(West,x,Nono) → ¬Missile(M₁)∨¬Owns(Nono,M₁)∨¬Hostile(Nono)
6. ⊕ Missile(M₁) → ¬Owns(Nono,M₁)∨¬Hostile(Nono)
7. ⊕ Owns(Nono,M₁) → ¬Hostile(Nono)
8. ⊕ ¬Enemy(x,America)∨Hostile(x) → ¬Enemy(Nono,America)
9. ⊕ Enemy(Nono,America) → □（空子句，矛盾）

**好奇心害死猫（图 9-11，斯科伦化 + 非确定子句 + 因子提取）**。CNF 子句集：Cat(Tuna)；¬Cat(x)∨Animal(x)；Kills(Jack,Tuna)∨Kills(Curiosity,Tuna)；否定查询 ¬Kills(Curiosity,Tuna)；¬Loves(y,x)∨¬Animal(z)∨¬Kills(x,z)；Animal(F(x))∨Loves(G(x),x) 与 ¬Loves(x,F(x))∨Loves(G(x),x)；¬Animal(x)∨Loves(Jack,x)。证明链：
1. Cat(Tuna) ⊕ ¬Cat(x)∨Animal(x) → Animal(Tuna)
2. Kills(Jack,Tuna)∨Kills(Curiosity,Tuna) ⊕ ¬Kills(Curiosity,Tuna) → Kills(Jack,Tuna)
3. Animal(Tuna) ⊕ ¬Loves(y,x)∨¬Animal(z)∨¬Kills(x,z) → ¬Loves(y,x)∨¬Kills(x,Tuna)　{z/Tuna}
4. Kills(Jack,Tuna) ⊕ 上式 → ¬Loves(y,Jack)　{x/Jack}
5. ¬Loves(x,F(x))∨Loves(G(x),x) ⊕ ¬Animal(x)∨Loves(Jack,x) → ¬Animal(F(Jack))∨Loves(G(Jack),Jack)　{x/Jack, x′/F(Jack)}（需先标准化分离变量）
6. 上式 ⊕ Animal(F(x))∨Loves(G(x),x) → Loves(G(Jack),Jack)∨Loves(G(Jack),Jack)，因子提取 → Loves(G(Jack),Jack)
7. ¬Loves(y,Jack) ⊕ Loves(G(Jack),Jack) → □　{y/G(Jack)}

泛化查询 ∃w Kills(w,Tuna)：取否定后重跑证明，置换中 w←Curiosity 即得答案（归结可能给出非构造性证明）。

## 要点
1. 提升的全部收益：只在必要时用合一实例化变量，取代命题化的全面枚举。
2. 数据日志是甜点区：无函数符号 ⟹ 蕴含可判定、前向链接多项式、表格化反向链接也完备。
3. 规则前提的合取子句排序是 NP 难（等价于 CSP 匹配）；树状结构的子句可线性求解。
4. 增量是前向链接的第一优化：第 t 轮只匹配与第 t−1 轮新事实合一的前提；Rete 缓存部分匹配。
5. 魔法集（magic set）：用目标反向重写规则、附加约束变量绑定——前向推断+反向预处理的混合。
6. 等词推理优先选解调/超解调/专用合一，而非把等词公理化塞给归结。

## 关联
- **第7章**：命题前向/反向链接与归结是本章算法的原型；**第8章**：一阶逻辑语法、置换与数据库语义是前提。
- **第11章**：规划的后继状态公理依赖本章推断；**第27章**：哥德尔不完全性对 AI 意义之争论。
- **外部概念**：Prolog 与表格化 Prolog（XSB）、Datalog/演绎数据库、Rete（CLIPS、OPS-5）、归结定理证明器（Otter、Vampire）、程序验证与演绎合成（Boyer-Moore）、DeepHOL 神经定理证明。
