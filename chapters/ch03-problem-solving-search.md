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
- **抽象（abstraction）** — 从表示中剔除细节；良好抽象=可细化（任何抽象解可细化为详细解）+ 动作易执行。
- **分支因子/深度（b, d, m）** — 后继数、最浅解深度、最大路径长度；复杂度分析的基本量。
- **系统性的（systematic）** — 无限空间中完备的必要条件：保证最终到达每个可达状态（如螺旋覆盖网格）。
- **占优（dominance）** — 对所有 n 有 h2(n)≥h1(n) 则 h2 占优 h1，A* 用 h2 扩展节点不多于 h1；一致启发式中"越高越好"。
- **有效分支因子（effective branching factor b*）** — 生成 n 个节点、解深 d 时均衡树所需的分支因子；衡量启发式质量。
- **满意解（satisficing）** — 放弃最优换取速度：有界次优（≤W·C*）、有界代价（≤C）、无界代价（speedy search）。
- **元级状态空间（metalevel state space）** — 把搜索程序自身内部状态当状态、扩展节点当动作；元级学习权衡计算开销与路径代价。

## 思维模型
- 当动作代价不全相同时，把"深度优先"换成"代价波前"：BFS 按深度分层，一致代价按 g 分层。
- 当内存装不下而深度未知时，用迭代加深：以重复上层节点的小代价（约多 11%）换 BFS 的完备性。
- 把 A* 看作"带方向感的一致代价搜索"：h 的差值就是把等值线从圆拉成朝目标的锥形。
- 当子问题可独立预计算时，把解代价存进模式数据库换运行时查表；当地标预计算可行时，差分启发式给可采纳的强引导。
- 当最优点不可负担时，显式选择放弃哪条性质：加权 A* 放弃最优性、束搜索放弃完备性、IDA*/RBFS 放弃时间。

## 反模式
- **负/零代价动作与负代价环**：环中循环无限次是最优解，本章算法全部失效；须假设代价 ≥ ε>0。
- **重复状态不去重**：8 邻接网格 9 步路径超 8^9≈1 亿条，平均同一格子百万条冗余路径——"不记得历史的算法注定重复历史"。
- **盲目 DFS 在无限/有环空间**：沿单条无限路径（4→4!→(4!)!…）走下去，永不回头，不完备。
- **BFS 用于不等代价问题**：找到动作数最少解而非最便宜解；且 O(b^d) 内存（b=10,d=10 时需 10 TB）。
- **贪心最佳优先求最优**：每个局部最优选择叠加成全局次优（Arad 例中多绕 32 英里）。
- **不一致启发式下首次到达即停**：同一状态多次入边界、reached 反复改写，时空都受损（但实践中 Félner 等认为不必害怕）。
- **UCS 用早期目标测试**：生成 Bucharest(310) 时即返回会错过 278 的最优路径——必须扩展时才测试。

## 算法要点
公共结构：节点 = (State, Parent, Action, Path-Cost)；EXPAND 用 Actions/Result/Action-Cost 生成子节点。

```
function EXPAND(problem, node) yields 节点
  for action in problem.ACTIONS(node.state):
    s' ← problem.RESULT(node.state, action)
    cost ← node.path-cost + problem.ACTION-COST(node.state, action, s')
    yield NODE(s', node, action, cost)

function BEST-FIRST-SEARCH(problem, f) → 解或 failure    # 图搜索模板
  frontier ← 优先队列(按 f)，放入初始节点
  reached ← {初始状态: 初始节点}
  while frontier 非空:
    node ← POP(frontier)
    if IS-GOAL(node.state): return node        # 后期目标测试
    for child in EXPAND(problem, node):
      s ← child.state
      if s ∉ reached 或 child.path-cost < reached[s].path-cost:
          reached[s] ← child; ADD(child, frontier)
  return failure
```

**广度优先搜索 BFS** — f=深度，实践用 FIFO 队列 + 早期目标测试 + reached 为状态集：
```
function BREADTH-FIRST-SEARCH(problem)
  node ← 初始节点; if IS-GOAL: return node
  frontier ← FIFO队列[node]; reached ← {初始状态}
  while frontier 非空:
    node ← POP(frontier)
    for child in EXPAND(problem, node):
      if IS-GOAL(child.state): return child    # 早期测试：生成即检
      if child.state ∉ reached:
          reached.add; ADD(child, frontier)
  return failure
```
性质：完备；动作等代价时代价最优；时间/空间均 O(b^d)（内存是比时间更硬的瓶颈）。

**一致代价搜索 UCS / Dijkstra** — f(n)=g(n)=路径代价：
```
function UNIFORM-COST-SEARCH(problem)
  return BEST-FIRST-SEARCH(problem, f = n.path-cost)   # 后期目标测试
```
性质：完备（代价≥ε）；代价最优（按代价递增波前系统扩展）；时间/空间 O(b^(1+⌊C*/ε⌋))，等代价时退化为 BFS。

**深度优先搜索 DFS** — f=−深度，实践用 LIFO 栈、树状实现（无 reached 表）：
```
function DEPTH-FIRST-SEARCH(problem)      # 树状搜索
  frontier ← 栈[初始节点]
  while frontier 非空:
    node ← POP(frontier)
    if IS-GOAL(node.state): return node
    if 路径上无环:                          # 沿父指针查环（可选）
      for child in EXPAND(problem, node): PUSH(child, frontier)
  return failure
```
性质：无限空间不完备、非最优；时间 O(b^m)，空间 O(bm)（边界=球的半径而非表面）。回溯搜索变体一次只生成一个后继、原地修改状态，空间仅 O(m)。

**深度受限搜索** — 超过 ℓ 的节点视为无后继：
```
function DEPTH-LIMITED-SEARCH(problem, ℓ)
  node ← POP(栈)
  if IS-GOAL: return node
  if depth(node) < ℓ: 压入子节点（查环）
  结束返回 failure 或 cutoff（可能还有更深处的解）
```
性质：ℓ 选小则不完备；时间 O(b^ℓ)，空间 O(bℓ)；ℓ 可取状态空间直径（罗马尼亚图直径=9）。

**迭代加深搜索** — 对 ℓ=0,1,2,… 反复调用深度受限搜索，直到返回非 cutoff：
```
function ITERATIVE-DEEPENING-SEARCH(problem)
  for ℓ = 0, 1, 2, ...:
    result ← DEPTH-LIMITED-SEARCH(problem, ℓ)
    if result ≠ cutoff: return result
```
性质：完备（有限空间+查环）；等代价时代价最优；时间 O(b^d)（重复上层的开销仅约 11%），空间 O(bd)；解深未知且内存紧张时的首选无信息算法。

**双向搜索** — 正向从初始、反向从目标（需逆向后继推理），两个边界+两张 reached 表：
```
function BIDIRECTIONAL-SEARCH(problem)
  初始化正向、反向两个边界与 reached 表
  while 两边界未相遇:
    扩展两个边界中 f 最小的节点（任一侧）
    若新节点状态在对方 reached 中: JOIN-NODES 连接成候选解，记录最优
  直到 Terminated() 证明不可能更优
```
性质：动机是 b^(d/2) ≪ b^d（b=d=10 时快约 5 万倍）；双向一致代价（f=g）不扩展 g>C*/2 的节点。

**贪心最佳优先搜索** — f(n)=h(n)，直奔看似最近的节点。性质：有限空间图搜索完备、无限不完备；非最优；最坏 O(b^m)，好 h 时 O(bm) 级，常极快。

**A\* 搜索** — f(n)=g(n)+h(n)，g 为已付代价、h 为剩余估计：
```
function A*-SEARCH(problem, h)
  return BEST-FIRST-SEARCH(problem, f(n) = g(n) + h(n))
```
性质：h 可采纳则完备且代价最优（反证法：次优解弹出时最优路径上必有更小 f 的节点在边界）；h 一致则效率最优（任何用同 h 的算法都须扩展全部 f<C* 节点）、且首次到达即最优无需重开；时间/空间最坏指数（超强吸尘器世界中 2^N 个状态全被扩展）。变体：加权 A* f=g+W·h，解代价 ≤ W·C*，"有点贪心"；束搜索边界只留 k 个最佳（不完备次优但快）；IDA* 以 f 等值线为截断值迭代加深（空间 O(bd)）；RBFS 线性空间+倒推值回溯；SMA* 填满内存后丢弃最差叶节点并备份其值给父节点（可达解深度≤内存时完备）。

## 参考表
无信息搜索算法对比（树状搜索版本；b=分支因子，d=最浅解深度，m=最大深度，ℓ=深度界限，C*≤最优解代价，ε=代价下界）：

| 算法 | 完备性 | 代价最优性 | 时间 | 空间 |
|---|---|---|---|---|
| 广度优先 | 是 | 是（等代价） | O(b^d) | O(b^d) |
| 一致代价 | 是（代价≥ε） | 是 | O(b^(1+⌊C*/ε⌋)) | 同时间 |
| 深度优先 | 否（无限空间） | 否 | O(b^m) | O(bm) |
| 深度受限 | 否（ℓ<d 时） | 否 | O(b^ℓ) | O(bℓ) |
| 迭代加深 | 是 | 是（等代价） | O(b^d) | O(bd) |
| 双向 | 是 | 是（等代价） | O(b^(d/2)) | O(b^(d/2)) |

评价函数谱系：贪心 f=h ＜ 加权 A* f=g+W·h ＜ A* f=g+h ＜ UCS f=g（h=0），从"只看未来"到"只看过去"的连续谱。

## 完整示例
**罗马尼亚寻路（Arad → Bucharest）**，形式化五要素：
- 状态=当前城市；初始=Arad；Actions=驶向相邻城市；Result=到达该城市；目标=Bucharest；代价=公路里程（英里）。
- 关键边：Arad–Sibiu 140，Arad–Timisoara 118，Arad–Zerind 75，Sibiu–Rimnicu Vilcea(RV) 80，Sibiu–Fagaras 99，RV–Pitesti 97，Pitesti–Bucharest 101，Fagaras–Bucharest 211。
- 启发式 h_SLD（到 Bucharest 直线距离）：Arad 366，Sibiu 253，Timisoara 329，Zerind 374，RV 193，Fagaras 176，Pitesti 100，Bucharest 0。
- 一致代价（从 Sibiu 起的子问题演示后期测试）：RV(80)、Fagaras(99) 入边界 → 扩展 RV 得 Pitesti(177) → 扩展 Fagaras 生成 Bucharest(310)，不测试 → 扩展 Pitesti 得 Bucharest(278)，替换 reached → 扩展 Bucharest(278)，测试通过返回。若生成即测试会错误返回 310。
- 贪心：Arad→Sibiu(253)→Fagaras(176)→Bucharest(0)，代价 450，非最优。
- A* 展开（f=g+h）：Arad(366) → 子节点 Sibiu(393)、Timisoara(447)、Zerind(449) → 扩展 Sibiu：RV(413)、Fagaras(415)、Oradea(671) → 扩展 RV：Pitesti(417)、Craiova(526) → 扩展 Fagaras：Bucharest(450)，在边界但非最小 → 扩展 Pitesti(417)：Bucharest(278+140=418) 替换 → Bucharest(418) 最小，弹出测试为目标，返回最优路径 **Arad–Sibiu–RV–Pitesti–Bucharest，代价 418**。Timisoara(447)、Zerind(449) 被剪枝——f>C* 的节点永不扩展。

**启发式构造（8/15 数码）**：h1=错位滑块数、h2=曼哈顿距离，均可由松弛问题导出（删去"相邻""空格"限制后成为精确解长）；多个可采纳 h 取 max 仍可采纳且更强；模式数据库预存子问题（滑块1-2-3-4+空格）精确代价，不相交数据库可使 15 数码毫秒级求解（节点数降至万分之一）；地图场景用地标 h_L(n)=min_L[C*(n,L)+C*(L,goal)]（高效但不可采纳）或差分启发式 h_DH=C*(n,L)−C*(goal,L)（可采纳）。

## 要点
1. 先形式化再搜索：五元组定错（抽象层级不当），后面全白费；好的抽象=可细化+动作易执行。
2. 一切无信息/有信息搜索都是 BEST-FIRST-SEARCH 换 f：排队策略即算法身份。
3. 冗余路径处理是指数级加速点：图搜索 vs 树状搜索是内存与时间的买卖。
4. A* 的三条命脉：可采纳→最优；一致→最优+首次到达即最优+效率最优；f<C* 等值线外的都被剪枝。
5. 启发式可以系统制造：松弛问题、组合取 max、模式数据库、地标/差分、从经验学习——且"更高（一致的）h 永远不差"。
6. 内存是搜索的第一瓶颈：迭代加深、IDA*、RBFS、SMA*、束搜索都是对"内存-时间-最优性"三角的不同取舍。
7. 双向搜索把 b^d 变 b^(d/2)，但 h 很好时 A* 等值线已聚焦目标，双向收益有限——启发式越差双向越划算。

## 关联
- **第2章**：搜索智能体=原子表示的基于目标的智能体；本章环境假设（回合式、单、完全可观测、确定、静态、离散、已知）正是第2章任务环境维度的最简侧。
- **第4章**：放松确定性/完全可观测假设后的搜索（解变为分支策略，闭环执行）；**第5章**：对抗性多智能体（博弈）；**第6章**：因子化表示的约束满足与回溯搜索；**第11章**：规划（结构化动作）；**第17、22章**：MDP 与强化学习（转移模型未知时）。
- **外部概念**：Dijkstra 算法（图论）、NP 完全性与组合爆炸（第1章易处理性）、动态规划（模式数据库反向构建）、Korf 的 IDA*（魔方求解）、在线地图服务的预计算+地标工程实践。
