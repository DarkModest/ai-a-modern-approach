# 第7章 逻辑智能体 (Logical Agents)

## 核心思想
智能体可以把世界知识存储为知识库中的语句，通过**可靠且完备的逻辑推断**从已有语句推出新语句，从而在不直接感知的情况下（如 wumpus 世界的无底洞位置）得出保证为真的结论并决定动作。本章以命题逻辑为载体，给出从真值表枚举、归结定理证明到 DPLL/SAT 求解的完整推断工具链。

## 框架
- **基于知识的智能体（knowledge-based agent）**
  - 何时使用：环境部分可观测、需要一般规则而非逐状态列举时
  - 如何做：维护知识库 KB；每个感知周期执行三步——Tell(KB, 感知语句)、Ask(KB, 该做什么动作)、Tell(KB, 已执行动作)。只需在**知识层面（knowledge level）**描述其知识与目标即可预测其行为，与实现层面无关
- **模型检验（model checking / TT-Entails?）**
  - 何时使用：命题符号数较少、需要直接判定蕴含 KB ⊨ α 时
  - 如何做：深度优先枚举所有真值赋值，检验在 KB 为真的每个模型中 α 均为真；可靠且完备，但时间 O(2ⁿ)
- **归结（resolution / PL-Resolution）**
  - 何时使用：需要完备的定理证明、证明比枚举模型更短时
  - 如何做：把 KB ∧ ¬α 转为合取范式（CNF），反复对互补文字对归结生成新子句；推出**空子句**即证明蕴含，无新子句可加则不蕴含
- **DPLL（Davis-Putnam-Logemann-Loveland）**
  - 何时使用：大规模 SAT 求解（蕴含检验、SATplan 规划、硬件验证）
  - 如何做：在回溯枚举上叠加三个改进——早期终止、纯符号启发式、单元子句启发式（单元传播）；配合分量分析、冲突子句学习、随机重启、聪明索引
- **前向/反向链接（forward/backward chaining）**
  - 何时使用：知识库限于确定子句（霍恩子句）时
  - 如何做：前向链接从已知事实出发反复应用肯定前件（数据驱动，线性时间）；反向链接从查询出发递归证明其前提（目标导向，只涉及相关事实）

## 关键概念
- **知识库（knowledge base, KB）** — 用知识表示语言表达的语句集；直接给出而非推导的语句称公理（axiom）。
- **蕴含（entailment）** — KB ⊨ α 当且仅当 α 在所有 KB 为真的模型中为真；α 的模型集是 KB 模型集的超集。
- **可靠性（soundness）与完备性（completeness）** — 可靠算法只推出被蕴含的语句；完备算法能推出所有被蕴含的语句。
- **命题逻辑（propositional logic）** — 由命题符号与 5 个联结词（¬ ∧ ∨ ⇒ ⇔）构成的因子化表示；模型是对每个符号的真值赋值。
- **文字/子句/合取范式（literal/clause/CNF）** — 文字是原子或其否定；子句是文字的析取；CNF 是子句的合取；单元子句只含一个文字。
- **霍恩子句与确定子句（Horn clause / definite clause）** — 最多含一个正文字的子句是霍恩子句；恰含一个正文字是确定子句，可写为"合取前提 ⇒ 结论"。
- **有效性/可满足性（validity/satisfiability）** — 在所有模型中为真称有效（重言式）；在某些模型中为真称可满足；SAT 是首个被证明 NP 完全的问题。
- **流（fluent）与非时序变量（atemporal variable）** — 随时间变化的世界要素（如 L₁,₁ᵗ）是流；不变部分无需时间上标。
- **后继状态公理（successor-state axiom）** — 形如 Fᵗ⁺¹ ⇔ (ActionCausesFᵗ ∨ (Fᵗ ∧ ¬ActionKillsFᵗ))，按流（而非动作）写公理，把转移模型压缩到 O(mk)。
- **框架问题（frame problem）** — 效应公理不说明"动作未改变什么"，需用框架公理或后继状态公理补救；分为表示框架问题与推断框架问题。

## 思维模型
- 当你推出"针在草堆里"（蕴含是语义事实）时，推断算法是找针的过程——蕴含与推断必须区分。
- 把 KB ⊨ α 的检验看作反证法：证明 KB ∧ ¬α 不可满足，与数学中的归谬法完全一致（演绎定理：KB ⊨ α 当且仅当 KB ⇒ α 有效）。
- 当规则形如"对每个方格、每个时刻……"时，用确定子句+前向/反向链接；要证否定或需全功能推理时，用归结或 DPLL。
- 把 DPLL 看作"回溯 + 常识剪枝"：任一文字为真子句即真（提前终止）、总以同一符号位出现的符号可直接赋值（纯符号）、被迫的赋值要级联传播（单元传播）。

## 反模式
- **用命题逻辑直接公理化大世界**：每条"微风 ⇔ 相邻无底洞"规则须对每个方格展开，"每个时刻"再乘一个因子——100×100 世界、1000 时间步需上亿条语句。命题蕴含还是余 NP 完全的，最坏情况指数级。
- **效应公理堆砌**：只写"动作导致什么"会使未提及的流（如 HaveArrow¹）变成未知；用逐条框架公理修补则公理数爆炸为 O(mn)。
- **用 WalkSAT 判定蕴含**：局部搜索找得到模型却无法证明不可满足，不能用来"证明方格安全"——它只能给经验性证据。
- **SATplan 公理不全**：不写"智能体同一时刻只能在一个位置"、前提公理、动作排除公理，SAT 求解器会找出"同时位于两处""没箭也能射"的假规划——但这也是调试知识库缺失知识的利器。

## 算法要点
**TT-Entails?（图7-10，真值表枚举判定蕴含）**：
```
function TT-ENTAILS?(KB, α) returns true/false
  symbols ← KB 和 α 中的命题符号列表
  return TT-CHECK-ALL(KB, α, symbols, {})
function TT-CHECK-ALL(KB, α, symbols, model) returns true/false
  if EMPTY?(symbols) then
    if PL-TRUE?(KB, model) then return PL-TRUE?(α, model)
    else return true        // 当 KB 为 false 时，始终返回 true
  else
    P ← FIRST(symbols); rest ← REST(symbols)
    return TT-CHECK-ALL(KB, α, rest, model∪{P=true}) and
           TT-CHECK-ALL(KB, α, rest, model∪{P=false})
```
深度优先递归（and 是伪代码语言的关键字而非逻辑算符）；部分模型已使 KB 为假即剪枝。n 个符号 → 2ⁿ 模型，时间 O(2ⁿ)、空间 O(n)。

**CNF 转换四步**（归结前置）：①消去 ⇔：P⇔Q → (P⇒Q)∧(Q⇒P)；②消去 ⇒：P⇒Q → ¬P∨Q；③¬ 内移：双重否定律、德摩根律，直到 ¬ 只在文字前；④用分配律对 ∧ 分配 ∨。

**归结规则**：两子句中互补文字 lᵢ 与 mⱼ 消去，其余文字析取（去重=因子提取）；单元归结是特例。完备性由**基本归结定理**保证：子句集不可满足 ⇔ 其归结闭包含空子句。

**PL-Resolution（图7-13，命题归结）**：
```
function PL-RESOLUTION(KB, α) returns true/false
  clauses ← KB∧¬α 的 CNF 表示的子句集合
  new ← {}
  while true do
    for each 子句对 Cᵢ, Cⱼ in clauses do
      resolvents ← PL-RESOLVE(Cᵢ, Cⱼ)   // 返回两输入归结出的所有可能子句
      if resolvents 包含空子句 then return true
      new ← new ∪ resolvents
    if new ⊆ clauses then return false
    clauses ← clauses ∪ new
```

**PL-FC-Entails?（图7-15，前向链接，输入为确定子句集）**：
```
function PL-FC-ENTAILS?(KB, q) returns true/false
  count ← 表，count[c] 为子句 c 的前提符号数量
  inferred ← 表，初始对所有符号为 false
  queue ← 符号队列，初始为 KB 中已知为 true 的符号
  while queue 不空 do
    p ← POP(queue)
    if p = q then return true
    if inferred[p] = false then
      inferred[p] ← true
      for each 前提中出现 p 的子句 c do
        count[c] 减 1
        if count[c] = 0 then 将 c.CONCLUSION 加入 queue
  return false
```
线性时间；到达不动点时推得的原子集构成 KB 的一个模型，故对原子查询完备。

**DPLL-Satisfiable?（图7-17）**：
```
function DPLL-SATISFIABLE?(s) returns true/false
  clauses ← s 的 CNF 子句集；symbols ← s 中的命题符号列表
  return DPLL(clauses, symbols, {})
function DPLL(clauses, symbols, model) returns true/false
  if 在 model 中每个子句都为 true then return true
  if 在 model 中 clauses 中某个子句为 false then return false
  P, value ← FIND-PURE-SYMBOL(symbols, clauses, model)
  if P 非空 then return DPLL(clauses, symbols−P, model∪{P=value})
  P, value ← FIND-UNIT-CLAUSE(clauses, model)
  if P 非空 then return DPLL(clauses, symbols−P, model∪{P=value})
  P ← FIRST(symbols); rest ← REST(symbols)
  return DPLL(clauses, rest, model∪{P=true}) or
         DPLL(clauses, rest, model∪{P=false})
```
三改进：**早期终止**（任一文字真则子句真、所有文字假则子句假，模型未完成也可判定）；**纯符号启发式**（在未满足子句中只以正/只以负出现的符号直接赋值使其为真）；**单元子句启发式**（只剩一个未赋真文字的子句强制赋值，级联触发**单元传播**，确定子句下等价于前向链接）。

**WalkSAT（图7-18，随机翻转检验可满足性）**：
```
function WALKSAT(clauses, p, max_flips) returns 满足的模型或 failure
  model ← 对 clauses 中的符号随机赋值 true/false
  for i = 1 to max_flips do
    if model 满足 clauses then return model
    clause ← 从 clauses 中随机选择的在 model 中为 false 的子句
    if RANDOM(0,1) ≤ p then 在 model 中翻转一个从 clause 中随机选择的符号的值
    else 翻转 clause 中的符号以最大化已满足子句的数量
  return failure
```
p 通常约 0.5；返回模型即可满足，返回 failure 则无法判定（不可满足或需更多翻转）。

**SATPlan（图7-22）**：
```
function SATPLAN(init, transition, goal, T_max) returns 解或 failure
  for t = 0 to T_max do
    cnf ← TRANSLATE-TO-SAT(init, transition, goal, t)
    model ← SAT-SOLVER(cnf)
    if model 非空 then return EXTRACT-SOLUTION(model)
  return failure
```
目标断言在时刻 t 成立、到 t 为止每步含后继状态公理；从模型中为真的动作命题符号提取规划；t 递增保证最短规划；仅适用完全可观测/无传感器环境。

## 参考表
命题逻辑 BNF 文法（图7-7）：
```
语句 → 原子语句 | 复合语句
原子语句 → True | False | P | Q | R | …
复合语句 → (语句) | ¬语句 | 语句∧语句 | 语句∨语句 | 语句⇒语句 | 语句⇔语句
运算符优先级（高→低）：¬, ∧, ∨, ⇒, ⇔
```

5 个联结词真值表（图7-8）：

| P | Q | ¬P | P∧Q | P∨Q | P⇒Q | P⇔Q |
|---|---|---|---|---|---|---|
| false | false | true | false | false | true | true |
| false | true | true | false | true | true | false |
| true | false | false | false | true | false | false |
| true | true | false | true | true | true | true |

标准逻辑等价（图7-11，α、β、γ 为任意语句）：

| 等价式 | 名称 |
|---|---|
| (α∧β) ≡ (β∧α)；(α∨β) ≡ (β∨α) | ∧、∨ 交换律 |
| ((α∧β)∧γ) ≡ (α∧(β∧γ))；((α∨β)∨γ) ≡ (α∨(β∨γ)) | ∧、∨ 结合律 |
| ¬(¬α) ≡ α | 双重否定律 |
| (α⇒β) ≡ (¬β⇒¬α) | 假言易位 |
| (α⇒β) ≡ (¬α∨β) | 蕴涵消去 |
| (α⇔β) ≡ ((α⇒β)∧(β⇒α)) | 等价消去 |
| ¬(α∧β) ≡ (¬α∨¬β)；¬(α∨β) ≡ (¬α∧¬β) | 德摩根律 |
| (α∧(β∨γ)) ≡ ((α∧β)∨(α∧γ))；(α∨(β∧γ)) ≡ ((α∨β)∧(α∨γ)) | ∧对∨、∨对∧ 分配律 |

CNF/霍恩/确定/目标子句文法（图7-12）：
```
CNF语句 → 子句₁ ∧ ⋯ ∧ 子句ₙ        子句 → 文字₁ ∨ ⋯ ∨ 文字ₘ
事实 → 符号                        文字 → 符号 | ¬符号
符号 → P | Q | R | …
霍恩子句范式 → 确定子句范式 | 目标子句范式
确定子句范式 → Fact | (符号₁∧⋯∧符号ₗ) ⇒ 符号
目标子句范式 → (符号₁∧⋯∧符号ₗ) ⇒ False
```

关键语义桥接：

| 命题 | 等价刻画 |
|---|---|
| KB ⊨ α | M(KB) ⊆ M(α) |
| KB ⊨ α | KB ⇒ α 有效（演绎定理） |
| KB ⊨ α | KB∧¬α 不可满足（反证法） |
| α 有效 | ¬α 不可满足 |
| α、β 等价 | M(α)=M(β)，即互相蕴含 |

## 完整示例
**wumpus 世界的命题公理化与推理**。命题符号：Pₓ,ᵢ（方格有无底洞）、Bₓ,ᵢ（有微风）、Wₓ,ᵢ、Sₓ,ᵢ、Lₓ,ᵢ。知识库（对应图 7-3b 情形）：
- R1：¬P₁,₁（起点无无底洞）
- R2：B₁,₁ ⇔ (P₁,₂ ∨ P₂,₁)
- R3：B₂,₁ ⇔ (P₁,₁ ∨ P₂,₂ ∨ P₃,₁)
- R4：¬B₁,₁（在 [1,1] 无感知）
- R5：B₂,₁（在 [2,1] 感知微风）

模型检验视角（图7-5、图7-9）：只关心 [1,2]、[2,2]、[3,1] 三格的无底洞时有 8 个可能模型；KB（[1,1] 无观测、[2,1] 有微风）仅在其中 3 个为真（实线圈出）。(a) 虚线是 α₁=¬P₁,₂ 的模型——3 个 KB 模型全在其中，故 KB ⊨ ¬P₁,₂；(b) 虚线是 α₂=¬P₂,₂ 的模型——3 个 KB 模型中只有 2 个在其中（P₂,₂ 在 1 个为真），故 α₂ 与 ¬α₂ 均不可判定。枚举全部 7 个符号则是 128 行真值表，KB 只在 3 行为真（图7-9 最右列下划线标出）。

定理证明（5 步推出 ¬P₁,₂ ∧ ¬P₂,₁）：①对 R2 等价消去得 R6：(B₁,₁⇒(P₁,₂∨P₂,₁))∧((P₁,₂∨P₂,₁)⇒B₁,₁)；②对 R6 合取消去得 R7：(P₁,₂∨P₂,₁)⇒B₁,₁；③对 R7 假言易位得 R8：¬B₁,₁⇒¬(P₁,₂∨P₂,₁)；④对 R8 与感知 R4 用肯定前件得 ¬(P₁,₂∨P₂,₁)；⑤德摩根律得结论。证明只涉及相关命题，与知识库大小无关。

归结证明（图7-14）：对 KB∧¬α（¬α 即 P₁,₂）转 CNF 后运用 PL-Resolution。顶行子句 ¬P₂,₁∨B₁,₁、¬B₁,₁∨P₁,₂∨P₂,₁、¬P₁,₂∨B₁,₁、¬B₁,₁ 与 P₁,₂ 反复成对归结（多数归结句是 ¬B₁,₁∨P₁,₂∨B₁,₁、P₁,₂∨P₂,₁∨¬P₂,₁ 之类的重言式），关键路径推出 ¬P₂,₁ 与 ¬P₁,₂，最终 ¬P₁,₂ 与被否定的目标 P₁,₂ 归结出空子句。

前向链接示例（图7-16）：霍恩子句集 P⇒Q、L∧M⇒P、B∧L⇒M、A∧P⇒L、A∧B⇒L、A、B，对应与或图（弧线连接同一蕴涵式的多个前提）。从事实 A、B 出发，前向链接依次推得 L、M、P，最终推得查询 Q。

后续（图 7-4a）：在 [1,2] 感知臭味无微风，加入 S₁,₂、¬B₁,₂ 后可推得 W₁,₃（wumpus 位置）与 P₃,₁（无底洞在 [3,1]）——一次结合不同时间、不同地点信息的推断。带上流与后继状态公理（如 HaveArrowᵗ⁺¹ ⇔ (HaveArrowᵗ ∧ ¬Shootᵗ)）后，智能体可回答"我现在在哪、哪些方格 OK"，与 A* 结合构成混合智能体。

## 要点
1. 推断的黄金标准：可靠性保证"不捏造"，完备性保证"不遗漏"；KB 在真实世界为真 + 可靠推断 ⇒ 结论在真实世界为真。
2. 蕴含检验与 SAT 互相归约：KB ⊨ α ⇔ KB∧¬α 不可满足——定理证明、反证法、SAT 求解是同一枚硬币的三面。
3. 证明可以比枚举便宜：证明只触碰相关命题，模型检验必须遍历 2ⁿ 空间；反过来模型很多但证明很短时定理证明占优。
4. 限制语言换来效率：确定子句 KB 上前向/反向链接线性时间，这是逻辑编程（Prolog）的根基。
5. 写动态公理时按流不按动作：后继状态公理把 O(mn) 的框架公理压到 O(mk)，代价是公理变长。
6. 随机 3-CNF 在子句/符号比 ≈ 4.3 处有相变（图7-19：可满足概率在该比值附近从≈1 骤降到≈0，DPLL 与 WalkSAT 的运行时间中位数也都在阈值附近达到峰值），最难实例集中在阈值附近——欠约束问题易猜中解，过约束问题易排除，阈值问题最难。
7. 逻辑状态估计用 1-CNF 信念状态做保守近似：每步只保留可证明的文字，保证为真但可能丢失析取信息（如 P₃,₁∨P₂,₂）。

## 关联
- **第2章**：基于知识的智能体是内部状态智能体的推广（KB 即紧凑表示的世界状态与规则）。
- **第3章/第4章**：证明搜索可形式化为搜索问题（初始=KB，动作=推断规则，目标=待证语句）；逻辑状态估计延续第4章的信念状态维护。
- **第6章**：DPLL 是 CSP 回溯搜索的对应物（分量分析、MRV 度启发式、冲突学习一一对应）；WalkSAT 对应最小冲突局部搜索。
- **第8章**：命题逻辑无法表达"对所有方格、所有时刻"，推动一阶逻辑登场——任意大小的 wumpus 世界用约 10 条一阶语句即可描述。
- **第9章**：一阶逻辑的推断算法（含归结、前向链接与逻辑编程）。
- **第11章**：后继状态公理是规划（SATplan 思想的延续）的基础。
- **第12章**：概率论补足资格问题与"无法判定"的处理；WalkSAT 式经验证据由概率推断接手。
- **外部概念**：现代 SAT 求解器（CDCL、硬件与协议验证）、Prolog（霍恩子句逻辑编程）、自动定理证明。
