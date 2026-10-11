# 第3章 通过搜索进行问题求解 (Solving Problems by Searching)

## 核心思想
当正确动作不显然时，智能体应提前规划：把世界抽象为原子状态空间图，在其上叠加搜索树寻找从初始状态到目标的路径。所有搜索算法本质相同——反复选择边界上的节点扩展，区别只在选择顺序（评价函数 f）与内存策略；算法性能是完备性、代价最优性、时间、空间四者间的权衡，而启发式函数的质量决定能省掉多少搜索。

## 框架
- **标准问题形式化（Standard Problem Formulation）**：问题五元组 = 状态集合 + 初始状态 + Actions(s) + 转移模型 Result(s,a) + 目标测试 + 动作代价 c(s,a,s')
  - 何时使用：任何可当作"找动作序列"的任务（环境须回合式、单智能体、完全可观测、确定、静态、离散、已知）
  - 如何做：目标形式化 → 问题形式化（抽象出合理且可执行的抽象层级）→ 搜索 → 执行；解=初始到目标的路径，最优解=代价最小路径（代价可加、为正）
- **最佳优先搜索（Best-First Search）**：所有算法的统一模板——总是扩展边界中 f(n) 最小的节点
  - 何时使用：任何搜索算法的推导起点
  - 如何做：f=深度→BFS；f=路径代价 g→一致代价；f=h→贪心；f=g+h→A*；f=g+W·h→加权A*
- **图搜索 vs 树状搜索（Graph Search / Tree-like Search）**：维护已达（reached）状态表剔除冗余路径 vs 不维护以省内存
  - 何时使用：状态空间存在大量汇合/环路时用图搜索（可快百万倍）；天然无冗余路径时（如装配排序）用树状搜索省内存
  - 如何做：折中方案=只沿父指针链查环，不查一般冗余路径
- **可采纳/一致启发式（Admissible / Consistent Heuristic）**：h 从不高估到目标代价（乐观）；一致性 h(n) ≤ c(n,a,n')+h(n') 是三角不等式，蕴含可采纳
  - 何时使用：需要 A* 代价最优保证时用可采纳；需要"首次到达即最优、无需重开"时需一致
  - 如何做：构造法见下（松弛问题、模式数据库、地标、学习）
- **搜索等值线（Contours）**：A* 按 f=C* 同心带向目标延伸；f<C* 的节点必然扩展，f>C* 的节点永不扩展（剪枝）
  - 何时使用：分析/可视化启发式强度与 A* 效率
  - 如何做：好的 h 使等值线聚焦目标方向；一致代价的 g 等值线则各向均匀

## 关键概念
- **问题求解智能体（problem-solving agent）** — 用搜索确定动作序列的智能体；使用原子表示（对比第11章规划智能体的因子化/结构化表示）。
- **开环/闭环（open-loop / closed-loop）** — 确定+已知环境中解是固定动作序列，执行时可"闭眼"忽略感知；模型可能错时须监控感知。
- **状态空间 vs 搜索树（state space / search tree）** — 图描述世界；树描述候选路径，同一状态可对应多个节点。
- **边界与已达（frontier / reached）** — 已生成未扩展的节点集合；已生成过节点的状态集合。边界分离内部（已扩展）与外部（未到达）。
- **抽象（abstraction）** — 良好抽象=可细化（任何抽象解可细化为详细解）+ 动作易执行。
- **分支因子/深度（b, d, m）** — 后继数、最浅解深度、最大路径长度；复杂度分析的基本量。
- **占优（dominance）** — 对所有 n 有 h2(n)≥h1(n) 则 h2 占优 h1，A* 用 h2 扩展节点不多于 h1；一致启发式中"越高越好"。
- **有效分支因子（effective branching factor b*）** — 生成 n 个节点、解深 d 时均衡树所需的分支因子；衡量启发式质量。
- **满意解（satisficing）** — 放弃最优换取速度：有界次优（≤W·C*）、有界代价（≤C）、无界代价（speedy search）。
- **元级状态空间（metalevel state space）** — 把搜索程序自身内部状态当状态、扩展节点当动作；元级学习权衡计算开销与路径代价。

## 思维模型
- 动作代价不全相同时，把"深度分层"换成"代价波前"：BFS 按深度，一致代价按 g。
- 内存装不下而深度未知时，用迭代加深：以重复上层的小代价换 BFS 的完备性与 O(bd) 空间。
- 把 A* 看作"带方向感的一致代价搜索"：h 把等值线从圆拉成朝目标的锥形。
- 子问题可独立预计算时，把解代价存进模式数据库换运行时查表；地标场景用差分启发式给可采纳的强引导。
- 最优点不可负担时，显式选择放弃哪条性质：加权 A* 放弃最优性、束搜索放弃完备性、IDA*/RBFS 放弃时间。

## 反模式
- **负/零代价动作与负代价环**：环中循环无限次是"最优解"，本章算法全部失效；须假设代价 ≥ ε>0。
- **重复状态不去重**：8 邻接网格中 9 步路径超 1 亿条，平均超 100 万条冗余路径到达同一格——"不记得历史的算法注定重复历史"。
- **盲目 DFS 在无限/有环空间**：沿单条无限路径（4→4!→(4!)!…）走下去永不回头，不完备。
- **BFS 用于不等代价问题**：找到动作数最少解而非最便宜解；且 O(b^d) 内存（b=10、d=10 需 10 TB；d=14 时即使无限内存也要 3.5 年）。
- **贪心最佳优先求最优**：局部最优叠加成全局次优（Arad 例中多绕 32 英里：450 vs 418）。
- **不一致启发式下首次到达即停**：同一状态多次入边界、reached 反复改写（但实践中 Félner 等认为不必害怕）。
- **UCS 用早期目标测试**：生成 Bucharest(310) 时即返回会错过 278 的最优路径——必须扩展时才测试。

## 算法要点
公共结构：节点 = (STATE, PARENT, ACTION, PATH-COST)；Expand 用 Actions/Result/Action-Cost 生成子节点（图3-7）。

```
function EXPAND(problem, node) yields 节点                    # 图3-7
  s ← node.STATE
  for each action in problem.ACTIONS(s) do
    s' ← problem.RESULT(s, action)
    cost ← node.PATH-COST + problem.ACTION-COST(s, action, s')
    yield NODE(STATE=s', PARENT=node, ACTION=action, PATH-COST=cost)

function BEST-FIRST-SEARCH(problem, f) returns 解节点或 failure   # 图3-7
  node ← NODE(STATE=problem.INITIAL)
  frontier ← 以 f 排序的优先队列，其中一个元素为 node
  reached ← 查找表，一个条目键为 problem.INITIAL、值为 node
  while not IS-EMPTY(frontier) do
    node ← POP(frontier)
    if problem.IS-GOAL(node.STATE) then return node      # 后期目标测试
    for each child in EXPAND(problem, node) do
      s ← child.STATE
      if s 不在 reached 中 or child.PATH-COST < reached[s].PATH-COST then
        reached[s] ← child；将 child 添加到 frontier
  return failure
```

**广度优先搜索 BFS / 一致代价搜索 UCS（Dijkstra）**（图3-9）：
```
function BREADTH-FIRST-SEARCH(problem) returns 解节点或 failure
  node ← NODE(problem.INITIAL)
  if problem.IS-GOAL(node.STATE) then return node
  frontier ← FIFO 队列，其中一个元素为 node；reached ← {problem.INITIAL}
  while not IS-EMPTY(frontier) do
    node ← POP(frontier)
    for each child in EXPAND(problem, node) do
      s ← child.STATE
      if problem.IS-GOAL(s) then return child            # 早期测试：生成即检
      if s 不在 reached 中 then
        将 s 添加到 reached；将 child 添加到 frontier
  return failure

function UNIFORM-COST-SEARCH(problem) returns 解节点或 failure
  return BEST-FIRST-SEARCH(problem, PATH-COST)           # f=g，后期目标测试
```
性质：BFS 完备、动作等代价时代价最优，时间/空间 O(b^d)，内存是比时间更硬的瓶颈；UCS 完备（代价≥ε）、代价最优（按代价波前扩展），时空 O(b^(1+⌊C*/ε⌋))，等代价时退化为 BFS。

**深度优先搜索 DFS**（书中无伪代码图，依正文重构）：LIFO 栈的树状搜索、无 reached 表——弹出节点、目标测试、沿父指针查环后压入子节点。性质：无限空间不完备、非最优；时间 O(b^m)，空间 O(bm)。回溯变体一次只生成一个后继、原地改状态，空间 O(m)。

**迭代加深 / 深度受限搜索**（图3-12）：
```
function ITERATIVE-DEEPENING-SEARCH(problem) returns 解节点或 failure
  for depth = 0 to ∞ do
    result ← DEPTH-LIMITED-SEARCH(problem, depth)
    if result ≠ cutoff then return result

function DEPTH-LIMITED-SEARCH(problem, ℓ) returns 解节点或 failure 或 cutoff
  frontier ← LIFO 队列（栈），其中一个元素为 NODE(problem.INITIAL)
  result ← failure
  while not IS-EMPTY(frontier) do
    node ← POP(frontier)
    if problem.IS-GOAL(node.STATE) then return node
    if DEPTH(node) > ℓ then result ← cutoff
    else if not IS-CYCLE(node) do
      for each child in EXPAND(problem, node) do 将 child 添加到 frontier
  return result
```
性质：DLS 在 ℓ<d 时不完备，时间 O(b^ℓ)、空间 O(bℓ)；ℓ 界限可用状态空间直径（罗马尼亚 20 城，ℓ=19 有效，直径=9 更好）。IDS 完备、等代价时代价最优，时间 O(b^d)（b=10,d=5 时生成 123450 节点 vs BFS 111110，仅多约 11%）、空间 O(bd)；内存不足且解深未知时的首选无信息算法。混合法：先 BFS 到内存将满，再对边界上每个节点做 IDS。

**双向最佳优先搜索**（图3-14；需逆向后继推理，两边界+两 reached 表）：
```
function BIBF-SEARCH(problemF, fF, problemB, fB) returns 解节点或 failure
  nodeF ← NODE(problemF.INITIAL)    # 初始状态节点
  nodeB ← NODE(problemB.INITIAL)    # 目标状态节点
  frontierF ← 按 fF 排序的优先队列含 nodeF；frontierB ← 按 fB 排序的优先队列含 nodeB
  reachedF ← 查找表 {nodeF.STATE: nodeF}；reachedB ← 查找表 {nodeB.STATE: nodeB}
  solution ← failure
  while not TERMINATED(solution, frontierF, frontierB) do
    if fF(TOP(frontierF)) < fB(TOP(frontierB)) then
      solution ← PROCEED(F, problemF, frontierF, reachedF, reachedB, solution)
    else solution ← PROCEED(B, problemB, frontierB, reachedB, reachedF, solution)
  return solution

function PROCEED(dir, problem, frontier, reached, reached₂, solution) returns 解
  # 在 frontier 上扩展节点，对照另一边界 reached₂；dir 为方向（F 正向 / B 反向）
  node ← POP(frontier)
  for each child in EXPAND(problem, node) do
    s ← child.STATE
    if s 不在 reached 中 or PATH-COST(child) < PATH-COST(reached[s]) then
      reached[s] ← child；将 child 添加到 frontier
      if s ∈ reached₂ then
        solution₂ ← JOIN-NODES(dir, child, reached₂[s])
        if PATH-COST(solution₂) < PATH-COST(solution) then solution ← solution₂
  return solution
```
性质：动机 b^(d/2)≪b^d（b=d=10 时复杂性不到单向的五万分之一）；f=路径代价时找到的第一个解即最优，否则须记录并更新迄今最优直到 TERMINATED 证明不可能更优；双向一致代价不扩展代价 >C*/2 的节点，两半"在中间相遇"。

**贪心最佳优先搜索** — f(n)=h(n)，直奔看似最近的节点。性质：有限空间图搜索完备、无限不完备；非最优；最坏 O(b^m)，好 h 时 O(bm) 级，常极快。

**A\* 搜索** — f(n)=g(n)+h(n)：
```
function A*-SEARCH(problem, h) returns 解节点或 failure
  return BEST-FIRST-SEARCH(problem, λn: n.PATH-COST + h(n))
```
性质：h 可采纳则完备且代价最优（反证：次优解弹出时最优路径上必有更小 f 的节点在边界）；h 一致则首次到达即最优无需重开，且效率最优（任何用同 h 的算法都须扩展全部 f<C* 节点）；时空最坏指数。变体：加权 A* f=g+W·h，解 ≤W·C*；束搜索边界只留 k 个最佳（不完备但快）；IDA* 以 f 等值线为截断值迭代加深（空间 O(bd)）；SMA* 填满内存后丢弃最差叶节点并把其值备份给父节点。

**递归最佳优先搜索 RBFS**（线性空间 A*，图3-22）：
```
function RECURSIVE-BEST-FIRST-SEARCH(problem) returns 解或 failure
  solution, fvalue ← RBFS(problem, NODE(problem.INITIAL), ∞)
  return solution

function RBFS(problem, node, f_limit) returns 解或 failure，及一个新的 f 代价限制
  if problem.IS-GOAL(node.STATE) then return node
  successors ← LIST(EXPAND(node))
  if successors 为空 then return failure, ∞
  for each s in successors do                          # 用前一次搜索中的值更新 f
    s.f ← max(s.PATH-COST + h(s), node.f)
  while true do
    best ← successors 中 f 值最低的节点
    if best.f > f_limit then return failure, best.f
    alternative ← successors 中第二低的 f 值
    result, best.f ← RBFS(problem, best, min(f_limit, alternative))
    if result ≠ failure then return result, best.f
```
性质：空间 O(bd)；回溯时把 best.f 倒推更新（记住次优子树的 f 值），比 IDA* 高效，但仍会重复生成大量节点（图3-23 中 RV↔Fagaras 间反复"改变主意"）。

## 参考表
图3-15 搜索算法比较（针对不查重复状态的树状搜索版本；b=分支因子，m=搜索树最大深度，d=最浅层解深度（无解时为 m），ℓ=深度界限，C*=最优解代价，ε=动作代价下界）：

| 指标 | 广度优先 | 一致代价 | 深度优先 | 深度受限 | 迭代加深 | 双向（如适用） |
|---|---|---|---|---|---|---|
| 完备性 | 是¹ | 是¹˒² | 否 | 否 | 是¹ | 是¹˒⁴ |
| 代价最优 | 是³ | 是 | 否 | 否 | 是³ | 是³˒⁴ |
| 时间复杂性 | O(b^d) | O(b^(1+⌊C*/ε⌋)) | O(b^m) | O(b^ℓ) | O(b^d) | O(b^(d/2)) |
| 空间复杂性 | O(b^d) | O(b^(1+⌊C*/ε⌋)) | O(bm) | O(bℓ) | O(bd) | O(b^(d/2)) |

原注：¹ b 有限且状态空间要么有解要么有限；² 所有动作代价 ≥ ε>0；³ 动作代价都相同；⁴ 两个方向均使用广度优先或一致代价搜索。图搜索版本的主要区别：有限状态空间中 DFS 完备，时空受状态空间大小（顶点数+边数）限制。

图3-16 h_SLD（到 Bucharest 的直线距离）的值：

| 城市 | h_SLD | 城市 | h_SLD |
|---|---|---|---|
| Arad | 366 | Mehadia | 241 |
| Bucharest | 0 | Neamt | 234 |
| Craiova | 160 | Oradea | 380 |
| Drobeta | 242 | Pitesti | 100 |
| Eforie | 161 | Rimnicu Vilcea | 193 |
| Fagaras | 176 | Sibiu | 253 |
| Giurgiu | 77 | Timisoara | 329 |
| Hirsova | 151 | Urziceni | 80 |
| Iasi | 226 | Vaslui | 199 |
| Lugoj | 244 | Zerind | 374 |

评价函数谱系：贪心 f=h ＜ 加权 A* f=g+W·h ＜ A* f=g+h ＜ UCS f=g（h=0），从"只看未来"到"只看过去"的连续谱。

## 完整示例
**罗马尼亚寻路（Arad → Bucharest，图3-1）**，五要素：状态=当前城市；初始=Arad；Actions=驶向相邻城市；Result=到达该城市；目标=Bucharest；代价=公路英里数。
- 关键边：Arad–Sibiu 140，Arad–Timisoara 118，Arad–Zerind 75，Sibiu–Rimnicu Vilcea(RV) 80，Sibiu–Fagaras 99，Sibiu–Oradea 151，Oradea–Zerind 71，RV–Pitesti 97，RV–Craiova 146，Pitesti–Bucharest 101，Fagaras–Bucharest 211。h_SLD 全表见参考表（图3-16）。
- 一致代价（从 Sibiu 起的子问题演示后期测试）：RV(80)、Fagaras(99) 入边界 → 扩展 RV 得 Pitesti(177) → 扩展 Fagaras 生成 Bucharest(310)，不测试 → 扩展 Pitesti 得 Bucharest(80+97+101=278)，替换 reached → 扩展 Bucharest(278) 测试通过返回。若生成即测试会错误返回 310。
- 贪心（h_SLD）：Arad→Sibiu(253)→Fagaras(176)→Bucharest(0)，代价 450，非最优。
- A* 各阶段（图3-18，节点标 f=g+h）：
  (a) 初始：Arad 366=0+366；
  (b) 扩展 Arad：Sibiu 393=140+253，Timisoara 447=118+329，Zerind 449=75+374；
  (c) 扩展 Sibiu(393)：Arad 646=280+366，Fagaras 415=239+176，Oradea 671=291+380，RV 413=220+193；
  (d) 扩展 RV(413)：Craiova 526=366+160，Pitesti 417=317+100，Sibiu 553=300+253；
  (e) 扩展 Fagaras(415)：Bucharest 450=450+0 入边界但 f 非最小，不返回；
  (f) 扩展 Pitesti(417)：Bucharest 418=418+0 替换 reached 中的 450 → Bucharest(418) 弹出，目标测试通过。
  最优路径 **Arad–Sibiu–RV–Pitesti–Bucharest，代价 418**；Timisoara(447)、Zerind(449) 因 f>C* 永不扩展（图3-20 等值线 f=380/400/420）。

**8数码启发式对比（图3-26，每个解长度 d 为 100 多个实例的平均）**：h1=错位滑块数，h2=曼哈顿距离（h2 占优 h1）：

| d | BFS 节点数 | A*(h1) | A*(h2) | b* BFS | b* h1 | b* h2 |
|---|---|---|---|---|---|---|
| 6 | 128 | 24 | 19 | 2.01 | 1.42 | 1.34 |
| 8 | 368 | 48 | 31 | 1.91 | 1.40 | 1.30 |
| 10 | 1033 | 116 | 48 | 1.85 | 1.43 | 1.27 |
| 12 | 2672 | 279 | 84 | 1.80 | 1.45 | 1.28 |
| 14 | 6783 | 678 | 174 | 1.77 | 1.47 | 1.31 |
| 16 | 17270 | 1683 | 364 | 1.74 | 1.48 | 1.32 |
| 18 | 41558 | 4102 | 751 | 1.72 | 1.49 | 1.34 |
| 20 | 91493 | 9905 | 1318 | 1.69 | 1.50 | 1.34 |
| 22 | 175921 | 22955 | 2548 | 1.66 | 1.50 | 1.34 |
| 24 | 290082 | 53039 | 5733 | 1.62 | 1.50 | 1.36 |
| 26 | 395355 | 110372 | 10080 | 1.58 | 1.50 | 1.35 |
| 28 | 463234 | 202565 | 22055 | 1.53 | 1.49 | 1.36 |

（图3-25 的典型实例最短解需 26 步；8数码可达状态 9!/2=181400，15数码 16!/2 超 10 万亿须靠可采纳启发式。）

**启发式构造**：h1、h2 均可由松弛问题导出（删去"相邻""空格"限制后成为精确解长）；多个可采纳 h 取 max 仍可采纳且更强；模式数据库预存子问题（滑块1-2-3-4+空格）精确代价，不相交数据库使 15 数码毫秒级求解（节点数不到曼哈顿距离的万分之一，24 数码约百万倍加速）；地图场景用地标 h_L(n)=min_L[C*(n,L)+C*(L,goal)]（高效但不可采纳）或差分启发式 h_DH=C*(n,L)−C*(goal,L)（可采纳）。

## 要点
1. 先形式化再搜索：五元组定错（抽象层级不当），后面全白费；好的抽象=可细化+动作易执行。
2. 一切无信息/有信息搜索都是 BEST-FIRST-SEARCH 换 f：排队策略即算法身份。
3. 冗余路径处理是指数级加速点：图搜索 vs 树状搜索是内存与时间的买卖。
4. A* 的三条命脉：可采纳→最优；一致→最优+首次到达即最优+效率最优；f>C* 等值线外的都被剪枝。
5. 启发式可以系统制造：松弛问题、组合取 max、模式数据库、地标/差分、从经验学习——一致的 h 越高越好。
6. 内存是搜索的第一瓶颈：迭代加深、IDA*、RBFS、SMA*、束搜索都是对"内存-时间-最优性"三角的不同取舍。
7. 双向搜索把 b^d 变 b^(d/2)，但 h 很好时 A* 等值线已聚焦目标、双向增益不大；启发式一般时"中间相遇"的双向往往扩展更少节点，是首选。

## 关联
- **第2章**：搜索智能体=原子表示的基于目标的智能体；本章环境假设正是第2章任务环境维度的最简侧。
- **第4章**：放松确定性/完全可观测后的搜索（解变为分支策略，闭环执行）；**第5章**：对抗博弈；**第6章**：因子化表示的约束满足与回溯；**第11章**：规划（结构化动作）；**第17、22章**：MDP 与强化学习（转移模型未知时）。
- **外部概念**：Dijkstra 算法（图论）、NP 完全性与组合爆炸、动态规划（模式数据库反向构建）、Korf 的 IDA*（魔方求解）、在线地图服务的预计算+地标工程实践。
