# 第4章 复杂环境中的搜索 (Search in Complex Environments)

## 核心思想
当只关心最终状态而不关心路径、或环境非确定/部分可观测/未知时，第3章的系统性搜索不再适用；本章给出三类替代武器：局部搜索（只要好状态不要路径）、与或/信念状态搜索（解从序列变为条件规划）、在线搜索（边走边学）。

## 框架
- **局部搜索（Local Search）**：从当前状态移动到邻居，不记路径、不记已访问集
  - 何时使用：只关心最终配置（8皇后、集成电路布图、车间调度、投资组合），或状态空间过大/无限
  - 如何做：用目标函数把状态铺成"状态空间地形图"；按爬山→模拟退火→束搜索→进化算法的顺序逐步引入随机性与多样性，对抗局部极大值、岭、平台区
- **与或搜索（AND–OR Search）**：非确定性环境下构造条件规划的深度优先搜索
  - 何时使用：Result(s,a) 返回状态集合（一个动作有多种可能结果）
  - 如何做：或节点=智能体的动作选择（只选一个），与节点=环境的全部结果（每个都要有子规划）；沿路径查环保证终止；解是一棵子树，即含 if–then–else（必要时 while 循环）的条件规划
- **信念状态搜索（Belief-State Search）**：在"可能状态集合"构成的空间中搜索
  - 何时使用：无传感器（conformant）或部分可观测环境
  - 如何做：把底层物理问题转成信念状态问题（状态数为 2^N、目标测试=所有成员均为目标），再套用第3章算法或与或搜索；转移分预测→可能感知→更新三阶段；用子集/超集剪枝与增量构造加速
- **在线搜索（Online Search）**：计算与动作交替进行
  - 何时使用：未知环境（不知 Result）、动态环境；动作本身即实验
  - 如何做：用 result[s,a] 表增量建图；Online-DFS 物理回溯；LRTA* 维护并更新各状态到目标代价的估计 H(s)，靠"不确定性下的乐观主义"探索；性能以竞争比度量

## 关键概念
- **局部搜索（local search）** — 只在邻居间移动、不记录路径与已访问集的算法家族；内存极省，可处理系统性算法无法应付的大/无限空间。
- **状态空间地形图（state-space landscape）** — 每个状态的"标高"由目标函数值定义的几何图景；找最高峰为爬山，找最低谷为梯度下降。
- **局部极大值/岭/平台区（local maxima / ridge / plateau）** — 爬山法三大困境：无更高邻居的峰、由局部极大值序列构成的山脊、平坦区域（可能是山肩）。
- **模拟退火（simulated annealing）** — 允许以指数递减概率接受"坏"移动的随机爬山；降温足够慢时以接近1的概率找到全局极大值。
- **局部束搜索（local beam search）** — 同时维护 k 个状态并在并行线程间传递有用信息（区别于 k 次独立随机重启）；随机束变体维持多样性。
- **遗传算法（genetic algorithm）** — 随机束搜索加杂交：按适应度选亲本、在杂交点交叉字符串、以突变率翻转位；用模式（schema）理论解释。
- **信念状态（belief state）** — 智能体认为自己可能位于的物理状态集合；部分可观测环境中搜索与维护的核心对象。
- **条件规划（conditional plan）** — 依据执行时收到的感知分支的解（又称应变规划/策略），形式为树或带标签循环。
- **循环解（cyclic solution）** — 用 while/标签表达的"反复尝试直到生效"的解，适用于动作可能失效的光滑环境。
- **无传感器问题（sensorless / conformant problem）** — 完全无感知时在信念状态空间求解；解仍是动作序列，可"强迫"世界到达目标。
- **过滤/状态评估（filtering / state estimation）** — 用预测-更新循环递归维护信念状态（式4-6）；任何智能系统的核心功能，又称监视。
- **定位（localization）** — 给定地图与感知/行动序列确定自身位置；预测扩大信念、更新收缩信念，通常迅速收敛到单点。
- **竞争比（competitive ratio）** — 在线搜索实际路径代价与已知环境最优路径代价之比。
- **可安全探索（safely explorable）** — 从每个可达状态都存在到目标的路径；在线探索算法的最低保证条件。
- **LRTA\*（learning real-time A\*）** — 存储 H(s)（到目标代价的当前最佳估计）并按邻居估计选择动作的在线智能体。
- **凸优化（convex optimization）** — 凸约束+凸目标（线性规划为其特例），多项式时间可解且实践中极高效。

## 思维模型
- 当路径不重要时，把搜索看作在地形图上找峰，而不是在图上找路。
- 当动作结果非确定时，把解看作树（if–then–else）；当动作可能失效时，把解看作循环（while）。
- 把部分可观测问题看作信念状态空间中的完全可观测问题：预测（动作）扩大信念，更新（感知）缩小信念。
- 把未知环境中的每个动作看作实验：选择规则+更新规则定义一族在线智能体。
- 把连续优化当作局部搜索的特例：离散化/经验梯度/牛顿-拉弗森只是"邻居生成器"不同。

## 反模式
- **纯最陡上升爬山**：86% 的 8 皇后实例被卡死；需受限横向移动（成功率 14%→94%）或随机性/重启。
- **把局部束搜索当 k 个独立重启**：k 个状态聚集在小区域后退化为 k 倍慢的爬山；需随机束/杂交维持多样性。
- **杂交作用在无结构表示上**：若基因位不对应有用组件（位置随机排列），杂交无任何优势——GA 成功依赖表示工程。
- **在不可逆/不可安全探索的空间用 Online-DFS 或随机游走**：可能跳进死胡同永不返回；一般状态空间不存在有界竞争比算法（对手论证）。
- **信念状态黑盒搜索不剪枝**：2^N 规模且每个元素是集合，朴素方法实践不可行；须紧凑表示、子集/超集剪枝或增量构造。

## 算法要点

爬山（最陡上升，图4-2）
```
function Hill-Climbing(problem) returns 一个位于局部极大值的状态
    current ← problem.Initial
    while true do
        neighbor ← current 的值最大的后继状态
        if Value(neighbor) ≤ Value(current) then return current
        current ← neighbor
```
内存 O(1)；变体：随机/首选/随机重启（期望重启次数 1/p）。8皇后用完整状态形式化（每列一个皇后，每状态 8×7=56 个后继）；启发式代价 h=相互攻击皇后对数（解 h=0），可把 −h 作为目标函数。

模拟退火（图4-5）
```
function Simulated-Annealing(problem, schedule) returns 一个解状态
    current ← problem.Initial
    for t = 1 to ∞ do
        T ← schedule(t)
        if T = 0 then return current
        next ← current 的一个随机选择的后继状态
        ΔE ← Value(next) − Value(current)
        if ΔE > 0 then current ← next
        else current ← next 仅以 e^{ΔE/T} 的概率
```
schedule 降温足够慢时以概率1收敛到全局极大（玻尔兹曼分布性质）。

局部束搜索
```
function LocalBeamSearch(problem, k):
    current ← k 个随机生成的状态
    loop:
        successors ← k 个状态的所有后继
        if 任一后继是目标: return 它
        current ← 其中最佳 k 个（随机束：按目标函数值加权抽取）
```
线程间传递信息："过来，这里的草更绿"，把资源集中到最有进展的路径。

遗传算法（图4-8）
```
function Genetic-Algorithm(population, fitness) returns 一个个体
    repeat
        weights ← Weighted-By(population, fitness)
        population2 ← 空列表
        for i = 1 to Size(population) do
            parent1, parent2 ← Weighted-Random-Choices(population, weights, 2)
            child ← Reproduce(parent1, parent2)
            if (小的随机概率) then child ← Mutate(child)
            将 child 添加到 population2
        population ← population2
    until 某个个体足够适应，或者已经经过了足够长的时间
    return 依据 fitness 选出的 population 中的最优个体

function Reproduce(parent1, parent2) returns 一个个体
    n ← Length(parent1);  c ← 从 1 到 n 的随机数
    return Append(Substring(parent1, 1, c), Substring(parent2, c+1, n))
```
8皇后编码：第 c 位数字 = 第 c 列皇后的行号（图4-7：第1行是最下面一行）；适应度 = 非攻击对数（解为 8×7/2=28）。图4-6 实例：初始种群 24748552、32752411、24415124、32543213，适应度 24、23、20、11（归一化概率 31%、29%、26%、14%），按权重选亲本→杂交点交叉→个别位突变。

与或图搜索（图4-11）
```
function And-Or-Search(problem) returns 一个条件规划或 failure
    return Or-Search(problem, problem.Initial, [])

function Or-Search(problem, state, path) returns 一个条件规划或 failure
    if problem.Is-Goal(state) then return 空规划
    if Is-Cycle(state, path) then return failure            // 环检查
    for each action 在 problem.Actions(state) 中 do
        plan ← And-Search(problem, Results(state, action), [state]+[path])
        if plan ≠ failure then return [action]+[plan]
    return failure

function And-Search(problem, states, path) returns 一个条件规划或 failure
    for each sᵢ in states do
        planᵢ ← Or-Search(problem, sᵢ, path)
        if planᵢ = failure then return failure
    return [if s₁ then plan₁ else if s₂ then plan₂ else … if sₙ₋₁ then planₙ₋₁ else planₙ]
```
有限状态空间中终止；解为条件规划子树，光滑环境下解为循环（while）。

在线 DFS 智能体（图4-21）
```
function Online-DFS-Agent(problem, s′) returns 一个动作     // s′=新感知到的状态
    persistent: s, a（之前的状态和动作，初始为空）
                result（将 (s,a) 映射到 s′ 的表，初始为空）
                untried（将 s 映射到未探索动作列表的表）
                unbacktracked（将 s 映射到未回溯状态队列的表）
    if problem.Is-Goal(s′) then return stop
    if s′ 是一个（不在 untried 中的）新状态 then untried[s′] ← problem.Actions(s′)
    if s 不空 then
        result[s,a] ← s′
        将 s 添加到 unbacktracked[s′] 的队首
    if untried[s′] 为空 then
        if unbacktracked[s′] 为空 then return stop          // 探索完成
        else a ← 满足 result[s′,b] = Pop(unbacktracked[s′]) 的动作 b（物理回溯）
    else a ← Pop(untried[s′])
    s ← s′;  return a
```
最坏情况每条连接恰好遍历两次（探索意义上最优）；仅当动作可逆时安全。

LRTA* 智能体（图4-24）
```
function LRTA*-Agent(problem, s′, h) returns 一个动作
    persistent: s, a；result 表；H（将 s 映射到代价估计值的表，初始为空）
    if Is-Goal(s′) then return stop
    if s′ 是一个（不在 H 中的）新状态 then H[s′] ← h(s′)
    if s 不空 then
        result[s,a] ← s′
        H[s] ← min_{b∈Actions(s)} LRTA*-Cost(problem, s, b, result[s,b], H)   // 更新刚离开的状态
    a ← argmin_{b∈Actions(s′)} LRTA*-Cost(problem, s′, b, result[s′,b], H)
    s ← s′;  return a

function LRTA*-Cost(problem, s, a, s′, H) returns 一个代价估计值
    if s′ 未定义 then return h(s)            // 未试动作乐观地按 h(s) 计
    else return problem.Action-Cost(s, a, s′) + H[s′]
```
"不确定性下的乐观主义"驱动探索；有限可安全探索环境中保证找到目标，最坏 O(n²) 步。

## 参考表

局部搜索算法对比（8 皇后实测与性质）：

| 算法 | 内存 | 完备性 | 关键数据/性质 |
|---|---|---|---|
| 最陡上升爬山 | O(1) | 否 | 14% 成功，成功平均 4 步、卡住平均 3 步 |
| 爬山+横向移动(≤100) | O(1) | 否 | 94% 成功，成功约 21 步、失败约 64 步 |
| 随机重启爬山 | O(1) | 概率 1 | 期望重启 1/p 次（p≈0.14 时约 7 次迭代、约 22 步） |
| 模拟退火 | O(1) | 概率 1（降温足够慢） | VLSI 布图、工厂调度等大规模优化 |
| 局部束搜索 | O(k) | 否 | 线程间共享信息；多样性不足则退化 |
| 遗传/进化算法 | 种群规模 | 否 | 杂交需有意义的表示（模式=可复用组件） |

## 完整示例

8皇后爬山实例（图4-3）：(a) 状态已接近解，仅第4、7列皇后沿对角线互攻；(b) 某状态 h=17，棋盘各格标出"把该列皇后移到该格"所得后继的 h 值，有 8 个并列最优移动 h=12，爬山法任选其一。

真空吸尘器世界的三个变体（贯穿 4.3–4.4 节）：8 个物理状态（图4-9；状态 7、8 为两格皆净的目标），动作 Left/Right/Suck。

1. 不稳定（erratic）版：脏格 Suck 有时顺带清邻格，净格 Suck 有时扬尘——非确定。状态 1 执行 Suck 得 Results(1,Suck)={5,7}。与或搜索给出条件规划（式4-3）：
   `Suck; if 当前状态=5 then [Right, Suck] else []`——若已两格皆净(7)则完成，若只净左格(5)则再走 Right、Suck 到达 8。
2. 无传感器版：初始信念 {1..8}，无感知也能获取信息：Right → {2,4,6,8}；[Right,Suck] → {4,8}；[Right,Suck,Left,Suck] → 强迫世界到达 {7}。解是动作序列（感知恒空、无偶发分支），在信念状态空间中问题完全可观测。
3. 光滑（slippery）版：移动有时失效（Right 可能原地不动），不存在无循环解；循环解：`while 不在最右格: Right`（配合吸尘），前提是失效是随机独立的（若皮带断了，重复无意义）。
4. 局部感知版（4.4）：感知如 [L, Dirty] 时初始信念 {1,3}；转移=预测→可能感知→更新，与或搜索在信念状态上返回测试信念（而非物理状态）的条件规划。

机器人定位（图4-18）：4×16 格世界，感知为 4 位串（北东南西是否有墙）。观测 E₁=1011 后剩 4 个可能位置（a）；向任一方向移动一格再观测 E₂=1010，信念收缩到唯一位置（b）——动作扩大不确定性、观测收缩不确定性的典型示范。

## 要点
1. 只要好状态不要路径时就换局部搜索：内存 O(1)，可对付无限空间。
2. 对抗局部极大值三板斧：受限横向移动、随机性（退火）、重启与多样性（束搜索、进化）。
3. 非确定性改变解的形态：序列 → 条件规划（与或树），动作可能失效时 → 循环规划。
4. 部分可观测 = 在信念状态空间搜索；预测扩大信念、更新（感知）缩小信念；维护信念状态即过滤/状态评估，是一切真实智能体的核心功能。
5. 信念空间 2^N 爆炸，须靠紧凑表示（第7章）、子集/超集剪枝、增量信念状态搜索。
6. 无传感器解常被低估：不依赖传感器可靠（类比广谱抗生素优于先化验再开药）。
7. 连续空间：离散化/经验梯度/牛顿-拉弗森；约束优化首选线性规划与凸优化（多项式时间、上千变量可行）。
8. 在线搜索中死胡同不可避免（对手论证）；LRTA* 用经验更新启发式"拉平"局部极小值，收敛后纯爬山即最优策略。

## 关联
- **第3章**：本章逐条放宽其"可观测、确定、静态、已知"假设；信念状态为单元素时即退化为第3章算法。
- **第5章**：与或搜索推广为极小化极大；信念状态直接用于不完美信息博弈。
- **第7章**：信念状态的大规模紧凑逻辑表示。
- **第12、14章**：用概率论处理随机环境的感知失效与连续状态过滤。
- **第17章**：在线搜索假设的放宽；探索/利用权衡。
- **第22章**：LRTA* 所属的强化学习在线智能体家族。
- **外部概念**：运筹学（线性规划/凸优化）、进化计算与元启发式（模拟退火源自冶金退火）、机器人定位与 SLAM。
