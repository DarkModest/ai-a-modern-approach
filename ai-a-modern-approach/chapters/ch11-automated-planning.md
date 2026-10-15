# 第11章 自动规划 (Automated Planning)

## 核心思想
经典规划＝在离散、确定、静态、完全可观测的环境中寻找达成目标的动作序列；PDDL 的因子化表示（状态＝流的合取、动作＝前提＋效果）使一个动作模式顶指数级命题公理，并让领域无关的启发式与算法自动推导成为可能——这是控制组合爆炸的根本工具。

## 框架
- **PDDL（Planning Domain Definition Language）**
  - 何时使用：表示经典规划领域（扩展版可处理连续、部分可观测、并发、多智能体）
  - 如何做：状态＝基本原子流的合取（封闭世界假设＋唯一名称假设）；动作模式＝名称＋变量表＋前提＋效果（文字合取）；执行结果 Result(s,a) = s − Del(a) ∪ Add(a)；领域＝动作模式集，问题＝领域＋Init（基本流合取）＋Goal（可含变量的文字合取）
- **前向／反向状态空间搜索（forward progression / backward regression search）**
  - 何时使用：有强领域无关启发式时用前向（当前主流）；分支因子巨大或目标含变量时用反向
  - 如何做：前向——用当前状态与各模式前提合一生成适用基本动作；反向——只考虑"相关动作"（其效果与目标某文字合一且不否定目标任何部分），对目标做回归 g′ = (g − Add(a)) ∪ Pre(a)
- **SATPlan（SAT 规划）**
  - 何时使用：可利用现代 SAT 求解器的强大效率时
  - 如何做：①动作命题化（变量×常量×时间步）②动作排斥公理 ③前提公理 ④定义初始状态 ⑤目标命题化为基本实例析取 ⑥后继状态公理 F^{t+1} ⟺ ActionCausesF^t ∨ (F^t ∧ ¬ActionCausesNotF^t)
- **分层任务网络规划（HTN, hierarchical task network）**
  - 何时使用：解需数千至数千万个动作（生产规划、航天器控制），或规划须人类可读可验证
  - 如何做：高层动作（HLA）＋细化库；Hierarchical-Search 在细化空间做广度优先；Angelic-Search 用可达集（乐观 Reach⁺／悲观 Reach⁻ 近似描述）先证明抽象规划可行、再专注并细化，复杂度从 O(b^d) 降到近似解深度的 k 次方根级
- **关键路径法（CPM, critical path method）**
  - 何时使用：动作偏序与时长给定、暂无资源约束的调度（"先规划，后调度"的调度阶段）
  - 如何做：动态规划求每动作最早开始 ES 与最晚开始 LS；ES(B)=max_{A≺B}[ES(A)+Duration(A)]，从 Finish 反向求 LS；零松弛动作构成关键路径；复杂度仅 O(Nb)；引入资源"不可重叠"约束（线性不等式析取）后变为 NP 难

## 关键概念
- **经典规划（classical planning）** — 在离散、确定、静态、完全可观测环境中找达成目标的动作序列的任务。
- **流（fluent）** — 随时间变化的世界方面；状态是基本（无变量）原子流的合取。
- **添加／删除列表（add list / delete list）** — 效果中的正／负文字；执行动作即对状态做集合减加。
- **回归（regression）** — 反向搜索中由目标 g 与动作 a 求前驱目标描述 g′ 的运算。
- **相关动作（relevant action）** — 效果能与目标某文字合一、且不否定目标任何部分的动作（反向搜索的"适用动作"对应物）。
- **后继状态公理（successor-state axiom）** — SATPlan 中刻画流随动作真值变迁的命题公式。
- **松弛问题（relaxed problem）** — 通过加边（忽略前提／删除列表）或合并节点（状态抽象）得到的易解问题，其解代价用作可容许启发式。
- **可序列化子目标（serializable subgoals）** — 存在一种完成子目标而不互相撤销的顺序，使规划器无须回溯（如积木塔自底向上；NASA 深空一号"远程智能体"借此实时控制航天器）。
- **高层动作（HLA）与细化（refinement）** — HLA 可展开为动作序列；全由基元动作组成的细化称实现（implementation）。
- **可达集（reachable set）** — Reach(s,h)：HLA 所有实现可达的状态集合；天使语义下智能体自选落点，与恶魔非确定论（对手选择）相对。
- **信念状态（belief state）** — 智能体可能处于的具体状态集；无传感器／应变规划在其上搜索。
- **条件效果（conditional effect）** — "when condition: effect" 语法，效果依赖当前状态；破坏 1-CNF 信念状态的封闭性。
- **松弛（slack）与最大完工时间（makespan）** — LS−ES 为动作调度窗口；调度总时长为 makespan。

## 思维模型
- 把动作看作集合运算：新状态 = 旧状态 − 删除列表 ＋ 添加列表。
- 当分支因子是天文数字（如 1 万亿个 Buy 实例）时，从目标反向回归——目标自动筛出相关动作，只考虑 1 个而非 1 万亿个。
- 把多细化 HLA 看作"天使非确定"动作：智能体自己选实现，故"至少一个实现达成目标"即规划成功；环境非确定性才是"恶魔的"，须所有结果都成功。
- 把信念状态当作文字合取（1-CNF）：在无传感器更新下封闭，用 O(n) 规模表示 2^n 个状态的子集；遇到条件效果则失守。
- 把调度看作偏序约束图：关键路径决定总工期，非关键动作有松弛窗口。

## 反模式
- **前向／反向裸搜索**：10 机场空运问题分支因子约 2000、深度 41，搜索图约 2000^41 节点——没有自动推导的启发式就绝望。
- **恶魔式 HLA 描述**（要求效果对所有实现成立，如"无现金也能到机场"被否决）：过于保守；正确做法是天使语义＋可达集。
- **1-CNF 被条件效果破坏后死守保守近似**：可靠（不会产生错误规划）但不完备（可能找不到确实存在的解，如 [Suck] 即可到干净方格）。
- **子目标代价直接相加**：子规划有冗余动作时不可容许（悲观）；有负相互作用时乐观错误；取 max 可容许但常过低，独立时求和才既可容许又准。
- **信赖模型绝对正确而不监控**：模型可能缺失前提／效果／流，或遭外因事件（含目标改变）——行为脆弱，重蹈掘土黄蜂式徒劳重复；应监视＋重规划，并从每次预测失败中学习更好模型。

## 算法要点
- **前向状态空间搜索**：状态＝基本流集合；适用动作＝把当前状态与每个模式的前提合一，对每个成功置换实例化出基本动作；用 A*／贪心最佳优先配自动启发式。
- **反向（回归）搜索**：选相关动作 a（效果与 g 中文字合一带标准化分离的变量置换，且 a 的效果不否定 g）；g′ = (g − Add(a)) ∪ Pre(a)；含变量时注意标准化分离以防变量名冲突。
- **SATPlan 翻译六步**：命题化动作 → 排斥公理 A_t ⇒ ¬B_t → 前提公理 A_t ⇒ Pre → Init 断言（未提及流断言为假）→ 目标基本实例析取 → 后继状态公理。
- **分层前向规划搜索（图 11-8，广度优先实现）**：最初提供给算法的规划是 [Act]；Refinements 函数返回一个动作序列集——HLA 的每个由状态 outcome 满足其前提的细化对应一个序列。
  ```
  function Hierarchical-Search(problem, hierarchy) returns 一个解或failure
    frontier ← 一个元素仅为[Act]的先入先出队列
    while true do
      if Is-Empty(frontier) then return failure
      plan ← Pop(frontier)                    // 选择frontier中最浅的plan
      hla ← plan中的第一个HLA，若无则为null
      prefix, suffix ← plan中hla之前和之后的动作序列
      outcome ← Result(problem.Initial, prefix)
      if hla为null then                       // plan是基元的且结果为outcome
        if problem.Is-Goal(outcome) then return plan
      else for each sequence in Refinements(hla, outcome, hierarchy) do
        添加 Append(prefix, sequence, suffix) 到frontier
  ```
  复杂度：解 d 个基元动作、每非基元动作 r 个细化、每细化 k 步，则细化树数约 r^{(d−1)/(k−1)}——小 r 大 k 时指数变 k 次方根。
- **天使语义分层规划（图 11-11）**：用天使语义识别并执行可行的高层规划、避免不可行高层规划；谓词 Making-Progress 检查并确保不会陷入细化的无限回归。
  ```
  function Angelic-Search(problem, hierarchy, initialPlan) returns 解或fail
    frontier ← 一个initialPlan为唯一元素的先入先出队列
    while true do
      if Empty?(frontier) then return fail
      plan ← Pop(frontier)                    // 选择frontier中最浅的节点
      if Reach⁺(problem.Initial, plan)与problem.Goal相交 then
        if plan是基元的 then return plan      // Reach⁺对于基元规划是精确的
        guaranteed ← Reach⁻(problem.Initial, plan) ∩ problem.Goal
        if guaranteed ≠ {} 且Making-Progress(plan, initialPlan) then
          finalState ← guaranteed中的任意元素
          return Decompose(hierarchy, problem.Initial, plan, finalState)
        hla ← plan中的某个HLA
        prefix, suffix ← plan中hla之前和之后的动作序列
        outcome ← Result(problem.Initial, prefix)
        for each sequence in Refinements(hla, outcome, hierarchy) do
          将 Append(prefix, sequence, suffix) 添加到frontier

  function Decompose(hierarchy, s₀, plan, s_f) returns 一个解
    solution ← 空规划
    while plan非空 do
      action ← Remove-Last(plan)
      s_i ← Reach⁻(s₀, plan)中的一个状态，使得 s_f ∈ Reach⁺(s_i, action)
      problem ← 一个Initial=s_i且Goal=s_f的问题
      solution ← Append(Angelic-Search(problem, hierarchy, action), solution)
      s_f ← s_i
    return solution
  ```
  判定规则：乐观可达集与目标不相交 ⇒ 规划行不通（剪除）；悲观可达集与目标相交 ⇒ 规划有效（Decompose 按回归分解为子问题细化）；两者之间的中间地带 ⇒ 继续细化以消除不确定性。
- **信念状态更新（无传感器）**：b′ = (b − Del(a)) ∪ Add(a)，未知文字：动作添加则为真、删除则为假、不影响则保持未知；应变规划在动作更新后再用感知模式并入感知文字（其前提需以析取并入）。
- **CPM**：拓扑序前向算 ES、反向算 LS；资源约束下用最小松弛（minimum slack）贪婪启发式（同 MRV 思想；对装配例得 130 分钟而非最优 115 分钟）。

## 参考表
经典规划方法对比：

| 方法 | 搜索空间 | 优点 | 局限 |
|---|---|---|---|
| 前向状态空间搜索 | 基本状态 | 可配强领域无关启发式，主流 | 无启发式时分支因子巨大 |
| 反向（回归）搜索 | 含变量的目标描述 | 分支因子小（相关动作） | 启发式难求 |
| SATPlan | 命题公式 | 借力现代 SAT 求解器 | 命题化后规模膨胀 |
| Graphplan | 规划图（planning graph） | 编码前提／效果与互斥约束 | — |
| CSP 编码 | 每步一个 Action_t 变量 | 免去动作排除公理 | 有界规划 |
| 偏序规划（POP） | 规划空间（图） | 独立子问题、人类可读（航天器/火星车） | 经典问题上已被前向+SAT 超越 |
| HTN 规划 | 细化空间 | 大规模、线性化深度、可读 | 需要知识工程（规划库） |

在线监视三级：

| 监视方式 | 检查内容 | 代价／收益 |
|---|---|---|
| 动作监视 | 执行前各前提仍成立 | 简单，但可能白白执行注定失败的步骤 |
| 规划监视 | 剩余规划仍会成功（各步前提除将被达成者外均成立） | 尽早中断、允许意外收获 |
| 目标监视 | 是否有更好的目标集 | 最主动 |

## 完整示例
航空货物运输（图 11-1，3 个动作模式）：

```
Init(At(C1,SFO) ∧ At(C2,JFK) ∧ At(P1,SFO) ∧ At(P2,JFK)
     ∧ Cargo(C1) ∧ Cargo(C2) ∧ Plane(P1) ∧ Plane(P2)
     ∧ Airport(JFK) ∧ Airport(SFO))
Goal(At(C1,JFK) ∧ At(C2,SFO))

Action(Load(c,p,a),
  Precond: At(c,a) ∧ At(p,a) ∧ Cargo(c) ∧ Plane(p) ∧ Airport(a)
  Effect: ¬At(c,a) ∧ In(c,p))
Action(Unload(c,p,a),
  Precond: In(c,p) ∧ At(p,a) ∧ Cargo(c) ∧ Plane(p) ∧ Airport(a)
  Effect: At(c,a) ∧ ¬In(c,p))
Action(Fly(p,from,to),
  Precond: At(p,from) ∧ Plane(p) ∧ Airport(from) ∧ Airport(to)
  Effect: ¬At(p,from) ∧ At(p,to))
```

关键建模约定：PDDL 无全称量词，故规定"In 飞机内的货物不同时 At 任何地方"——At 的真正含义是"在给定地点可用"；货物随机飞行、卸载后才 At 新机场。
解（6 步）：[Load(C1,P1,SFO), Fly(P1,SFO,JFK), Unload(C1,P1,JFK), Load(C2,P2,JFK), Fly(P2,JFK,SFO), Unload(C2,P2,SFO)]。

备用轮胎问题（图 11-2；注意 LeaveOvernight 是"世界捣乱"动作）：

```
Init(Tire(Flat) ∧ Tire(Spare) ∧ At(Flat,Axle) ∧ At(Spare,Trunk))
Goal(At(Spare,Axle))
Action(Remove(obj,loc),
  Precond: At(obj,loc)
  Effect: ¬At(obj,loc) ∧ At(obj,Ground))
Action(PutOn(t,Axle),
  Precond: Tire(t) ∧ At(t,Ground) ∧ ¬At(Flat,Axle) ∧ ¬At(Spare,Axle)
  Effect: ¬At(t,Ground) ∧ At(t,Axle))
Action(LeaveOvernight,
  Precond: （空）
  Effect: ¬At(Spare,Ground) ∧ ¬At(Spare,Axle) ∧ ¬At(Spare,Trunk)
        ∧ ¬At(Flat,Ground) ∧ ¬At(Flat,Axle) ∧ ¬At(Flat,Trunk))
```

积木世界（图 11-4，建造 A 在 B 上、B 在 C 上的塔；一个解是 [MoveToTable(C,A), Move(B,Table,C), Move(A,Table,B)]）：

```
Init(On(A,Table) ∧ On(B,Table) ∧ On(C,A)
     ∧ Block(A) ∧ Block(B) ∧ Block(C) ∧ Clear(B) ∧ Clear(C) ∧ Clear(Table))
Goal(On(A,B) ∧ On(B,C))
Action(Move(b,x,y),
  Precond: On(b,x) ∧ Clear(b) ∧ Clear(y) ∧ Block(b) ∧ Block(y) ∧
           (b≠x) ∧ (b≠y) ∧ (x≠y)
  Effect: On(b,y) ∧ Clear(x) ∧ ¬On(b,x) ∧ ¬Clear(y))
Action(MoveToTable(b,x),
  Precond: On(b,x) ∧ Clear(b) ∧ Block(b) ∧ Block(x)
  Effect: On(b,Table) ∧ Clear(x) ∧ ¬On(b,x))
```

建模细节：Clear(y) 意为"y 上方有容身空间"；Table 永远 Clear，故另设 MoveToTable（若给 Move 加 Block(b)∧Block(y) 前提，也可阻止把积木堆到桌子这种冗余动作）。

HLA 细化示例（图 11-7："前往旧金山机场"与真空吸尘器世界的递归导航）：

```
Refinement(Go(Home,SFO),
  Steps: [Drive(Home,SFOLongTermParking), Shuttle(SFOLongTermParking,SFO)])
Refinement(Go(Home,SFO),
  Steps: [Taxi(Home,SFO)])

Refinement(Navigate([a,b],[x,y]),
  Precond: a=x ∧ b=y
  Steps: [])
Refinement(Navigate([a,b],[x,y]),
  Precond: Connected([a,b],[a−1,b])
  Steps: [Left, Navigate([a−1,b],[x,y])])
Refinement(Navigate([a,b],[x,y]),
  Precond: Connected([a,b],[a+1,b])
  Steps: [Right, Navigate([a+1,b],[x,y])])
```

## 要点
1. 因子化表示是规划的力量之源：一个 Fly 模式＝4Tn² 条命题公理，且无须领域特定启发式。
2. 回归搜索用"相关动作"砍分支：1 万亿个 Buy 基本动作的问题反向只需考虑 1 个动作。
3. 启发式可从动作模式自动推导：忽略前提 → 集合覆盖（NP 难，贪心 log n 近似但不可容许）；只忽略部分前提可复现曼哈顿距离；忽略删除列表 → 单调无局部极小，爬山（FF 的强制爬山＋广度优先逃逸）即可解。
4. 剪枝三招：对称约简、优先动作（松弛规划中的步骤）、可序列化子目标排序。
5. HTN 把指数化为 k 次方根：少量长细化的规划库＋天使可达集，可得可证明正确的抽象规划（Angelical-Search 在多房间吸尘世界上近线性，而广度优先 5^d）。
6. 无传感器／应变规划在 1-CNF 信念状态上搜索；故意选择让信念状态保持简单的动作序列（Right,Suck,Left,Suck），或用"Result(b₀,动作序列)"惰性表示＋SAT 蕴含测试。
7. 先规划后调度：CPM 以 O(Nb) 求出全部 ES/LS；资源聚合为数量（防 9! 类枚举）；资源约束使问题 NP 难（1963 年 10×10 作业车间问题悬置 23 年）。

## 关联
- **第3章／第4章**：问题求解搜索与与或搜索是本章前向／应变规划的原子表示前身。
- **第7章**：命题逻辑智能体的后继状态公理、封闭世界假设与信念状态被 SATPlan 和 11.5 节复用。
- **第6章**：CSP 编码有界规划（每时间步一个 Action_t 变量）。
- **第17章／第22章**：随机环境（MDP/POMDP）与强化学习是本章非确定性扩展的概率化续篇。
- **外部概念**：STRIPS／PDDL 标准（Ghallab et al. 1998）、Graphplan、FF（Hoffmann 2005）、O-Plan（日立生产规划）、NASA 深空一号 Remote Agent、情景演算（situation calculus）、关键路径法／作业车间调度。
