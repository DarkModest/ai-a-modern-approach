# 第6章 约束满足问题 (Constraint Satisfaction Problems)

## 核心思想
把状态从黑盒分解为变量+约束（因子化表示）后，就能用领域无关的推断（约束传播）与启发式（变量/值排序）一次性消除大片搜索空间，并由约束图的结构（树宽）预测并利用问题的求解难度。

## 框架
- **约束传播（Constraint Propagation）**：用约束缩减变量域、再引发连锁缩减的推断
  - 何时使用：搜索前预处理（有时直接解出或证明无解），以及搜索中与每次赋值交替
  - 如何做：逐级增强局部一致性——节点一致 → 弧一致（AC-3）→ 路径一致 → k 一致；全局约束（Alldiff、资源约束）用专用传播算法，整数大域用边界传播
- **回溯搜索（Backtracking Search）**：CSP 的深度优先搜索
  - 何时使用：传播后仍有变量取多值
  - 如何做：利用可交换性每层只选一个变量（叶从 n!·d^n 降到 d^n）；配 MRV/度启发式选变量、最少约束值排序、Inference（前向检验/MAC）删值；失败用冲突导向回跳+约束学习（no-good）
- **最少冲突局部搜索（Min-Conflicts）**：完整赋值上每次改一个变量
  - 何时使用：解稠密问题（n 皇后）、需要在线最小改动修复的调度
  - 如何做：随机选冲突变量，赋冲突最少的值；平台区用禁忌搜索、约束加权、模拟退火逃逸
- **结构分解（Structure Exploitation）**：把约束图化归为树
  - 何时使用：图由独立分量组成，或"几乎是一棵树"（小环割集/小树宽）
  - 如何做：独立分量各自求解；树状 CSP 定向弧一致后线性求解；一般图用割集调整 O(d^c·(n−c)·d²)（线性内存）或树分解 O(nd^{w+1})（指数内存）

## 关键概念
- **CSP 三元组（X, D, C）** — 变量集、各变量的域、约束集（约束=scope+rel）；解 = 一致（不违反任何约束）且完整（每变量已赋值）的赋值；一般 CSP 求解是 NP 完全的。
- **约束图（constraint graph）** — 节点=变量、边=共享二元约束；高阶约束用约束超图或对偶图变换表示。
- **全局约束（global constraint）** — 涉及任意多个变量的约束（Alldiff、Atmost/资源约束）；比拆成二元约束更易描述且有更强专用推理。
- **偏好约束（preference constraint）/COP** — 编码为代价的软约束，引出约束优化问题（线性规划是一类 COP）。
- **弧一致（arc consistency）** — Xi 相对 Xj 弧一致：Di 的每个值在 Dj 中都有满足约束的伴；AC-3 在 O(cd³) 内达成。
- **路径一致（path consistency）** — {Xi,Xj} 的每个一致赋值都能被某 Xm 的赋值同时满足；对二元图即 3 一致。
- **强 k 一致（strongly k-consistent）** — 1 到 k 级一致齐备；强 n 一致可 O(n²d) 无回溯解出，但建立的时间与空间均为 n 的指数级。
- **可交换性（commutativity）** — 赋值动作的顺序不影响结果；据此搜索每层只处理一个变量。
- **最少剩余值（MRV）** — 选合法值最少的未赋变量（"最受约束变量"/"失败优先"）。
- **度启发式（degree heuristic）** — 选与最多未赋变量相连的变量，用于 MRV 全并列时（如澳大利亚先着 SA，度=5）。
- **最少约束值（least-constraining value）** — 优先给相邻变量留最多选择的值（"失败延后"）。
- **前向检验（forward checking）** — 赋值 X=x 后立即从邻居域中删不相容值；顺带免费提供冲突集。
- **MAC（maintaining arc consistency）** — 赋值后从相邻弧 (Xj,Xi) 起跑 AC-3；严格强于前向检验（递归传播）。
- **冲突集/回跳/冲突导向回跳（conflict set / backjumping / conflict-directed backjumping）** — 记录造成失败的赋值集合，直接回溯到根源变量而非上一个变量。
- **约束学习/无用赋值（constraint learning / no-good）** — 缓存引起矛盾的最小变量-值组合；现代 CSP 求解器最重要的技术之一。
- **树宽（tree width）** — 最优树分解中最大节点大小减 1；树宽 w 的 CSP 可在 O(nd^{w+1}) 内求解，找最优分解是 NP 难的。
- **值对称/对称性破缺（value symmetry / symmetry-breaking）** — 颜色名排列产生 d! 个等价解；加排序约束只留其一，空间砍 d! 倍。

## 思维模型
- 把状态看作约束图上的一组"变量-域"快照：推断 = 沿边收缩域，赋值 = 固定一个节点。
- 变量选择"失败优先"、值选择"失败延后"：所有变量终须赋值（选谁先失败都一样），但只需一个解（好值优先试）。
- 把结构看作复杂度：不连通→线性；树→线性；树宽 w→d^{w+1}；NP 完全只是最坏情形。
- 把矛盾的起因看作可复用知识（no-good），而不是只用来一次性回溯。

## 反模式
- **用朴素 DFS 解 CSP**：不用可交换性会产生 n!·d^n 片叶子（赋值只有 d^n 种，纯因顺序爆炸）。
- **时序回溯**：SA 无值却回头改塔斯马尼亚的颜色——回到与失败无关的变量毫无意义。
- **以为前向检验足够**：WA=red、Q=green 后 NT 与 SA 都只剩蓝（二者相邻必冲突），前向检验看不见，MAC 能看见。
- **一味追求强 k 一致**：时间与空间都是指数级；实践中 2-3 级一致是常规折中。
- **忽略值对称**：同一解族的 d! 个排列被逐个搜索。
- **树分解节点过大**：节点含 30 个布尔变量意味着 d=2³⁰，"需要几个世纪"；此时割集调整的线性内存反而占优。
- **静态/随机变量排序**：MRV 与度启发式常有数量级的效率差。

## 算法要点

AC-3（弧一致）（图6-3）
```
function AC-3(csp) returns false（若发现不一致）或 true（其他情况）
    queue ← 弧的队列，初始化时包含 csp 中的所有弧
    while queue 非空 do
        (Xi, Xj) ← Pop(queue)
        if Revise(csp, Xi, Xj) then
            if Di 的规模 = 0 then return false
            for each Xk ∈ Xi.Neighbors − {Xj} do
                将 (Xk, Xi) 添加到 queue 中
    return true

function Revise(csp, Xi, Xj) returns true 当且仅当 Xi 的域被修改
    revised ← false
    for each x ∈ Di do
        if Dj 中不存在使 (x,y) 满足 (Xi,Xj) 约束的值 then
            将 x 从 Di 中删除；revised ← true
    return revised
```
n 变量、域≤d、c 条弧时 O(cd³)；结果与原 CSP 同解但域更小（有时直接解出/证明无解）。

回溯搜索（含启发式与推断）（图6-5）
```
function Backtracking-Search(csp) returns 一个解或 failure
    return Backtrack(csp, {})

function Backtrack(csp, assignment) returns 一个解或 failure
    if assignment 是完整的 then return assignment
    var ← Select-Unassigned-Variable(csp, assignment)   // MRV；并列用度
    for each value ∈ Order-Domain-Values(csp, var, assignment) do   // 最少约束值优先
        if value 与 assignment 一致 then
            将 {var = value} 添加到 assignment 中
            inferences ← Inference(csp, var, assignment)   // 前向检验或 MAC
            if inferences ≠ failure then
                将 inferences 添加到 csp 中
                result ← Backtrack(csp, assignment)
                if result ≠ failure then return result
            从 csp 中删除 inferences                  // 原地单状态修改
            从 assignment 中删除 {var = value}
    return failure
```
MRV+前向检验互补：前向检验恰好增量算出 MRV 所需的域大小信息。

最少冲突局部搜索（图6-9）
```
function Min-Conflicts(csp, max_steps) returns 一个解或 failure
    inputs: csp，约束满足问题；max_steps，放弃前允许的步数
    current ← csp 的一个初始完整赋值      // 随机选取或贪心逐变量取最少冲突值
    for i = 1 to max_steps do
        if current 是 csp 的一个解 then return current
        var ← 从 csp.Variables 中随机选取的冲突变量
        value ← 使 Conflicts(csp, var, v, current) 取得最小值的 v
        在 current 中设置 var = value
    return failure
```
n 皇后：初始赋值后平均约 50 步可解百万皇后（与规模基本无关）；哈勃望远镜周调度从 3 周缩至约 10 分钟。

树求解器（Tree-CSP-Solver）（图6-11）
```
function Tree-CSP-Solver(csp) returns 一个解或 failure
    inputs: csp，具有 X, D, C 的 CSP（约束图为树）
    n ← X 中变量个数
    assignment ← 一个空的赋值
    root ← X 中任一变量
    X ← TopologicalSort(X, root)         // 以 root 为根、父先于子的线性排序（图6-10）
    for j = n down to 2 do
        Make-Arc-Consistent(Parent(Xj), Xj)   // 删父域中无伴的值
        if 不能取得一致 then return failure
    for i = 1 to n do
        assignment[Xi] ← Di 中任一一致的值    // 定向弧一致 ⇒ 无需回溯
        if 没有一致值 then return failure
    return assignment
```
O(nd²)；配合割集调整/树分解可推广到近似树状的图。

## 参考表

回溯搜索的四个领域无关改进点（6.3 节四问）：

| 决策点 | 启发式/技术 | 作用 |
|---|---|---|
| 选哪个变量 | MRV（最少剩余值） | 失败优先，尽早剪枝 |
| 初始/并列破僵 | 度启发式 | 缩小后续分支因子 |
| 按什么顺序试值 | 最少约束值 | 给邻居留最大灵活性 |
| 赋值后做什么推断 | 前向检验 < MAC（AC-3） | 删不相容值、早发现死路 |
| 失败回溯到哪 | 回跳 / 冲突导向回跳 | 直回冲突根源（前向检验下简单回跳冗余） |
| 失败之后 | 约束学习（no-good 缓存） | 永不再蹈同一覆辙 |

一致性层级与结构方法：

| 概念 | 强度/复杂度 | 备注 |
|---|---|---|
| 1 一致 = 节点一致 | 消一元约束 | 预处理即可完成 |
| 2 一致 = 弧一致（AC-3） | O(cd³) | 解不了澳大利亚 2 色问题 |
| 3 一致 = 路径一致（二元图） | 更贵（数独需查 255 960 条路径约束） | 可证明 2 色无解 |
| 强 n 一致 | O(n²d) 求解，但建立成本/空间指数 | 理论上限 |
| 独立分量分解 | O(d^c·n/c)，线性 | 100 布尔变量拆 4 份：宇宙寿命 → <1 秒 |
| 割集调整（割集 c） | O(d^c·(n−c)·d²)，线性内存 | c=20 时几分钟；最坏 c 可达 n−2 |
| 树分解（树宽 w） | O(nd^{w+1})，内存指数于 w | 有割集 c 则树宽 ≤ c+1，常更小 |

## 完整示例

澳大利亚地图着色回溯 + 前向检验（图 6-1/6-6/6-7）：变量 WA、NT、SA、Q、NSW、V、T，域 {red, green, blue}，相邻不同色（约束图 9 条边：WA-NT、WA-SA、NT-SA、NT-Q、Q-SA、Q-NSW、SA-NSW、SA-V、NSW-V；T 孤立）。图6-6 给出部分搜索树：根下 WA 三分支（r/g/b），WA=red 下 NT∈{green,blue}，再展开 Q——利用可交换性每层只展开一个变量。

1. WA=red → 前向检验从 NT、SA 的域删 red：NT={g,b}，SA={g,b}。
2. Q=green → 从 NT、SA、NSW 删 green：NT={b}，SA={b}，NSW={r,b}。
3. V=blue → 从 NSW、SA 删 blue：NSW={r}，**SA=∅** ← 空域！前向检验立即报告不一致，算法回溯（换 V 的颜色或更早的赋值）。
4. 配 MRV：第 2 步后 NT、SA 只剩 1 值会被优先选中，完全消除这些变量上的分支——前向检验正是 MRV 所需信息的增量计算。

前向检验域缩减全过程（图6-7，R/G/B=红/绿/蓝，加粗为新赋值）：

| 步骤 | WA | NT | Q | NSW | V | SA | T |
|---|---|---|---|---|---|---|---|
| 初始域 | RGB | RGB | RGB | RGB | RGB | RGB | RGB |
| WA=red 后 | **R** | GB | RGB | RGB | RGB | GB | RGB |
| Q=green 后 | R | B | **G** | RB | RGB | B | RGB |
| V=blue 后 | R | B | G | R | **B** | **∅** | RGB |

密码算术（图6-2）：TWO + TWO = FOUR。主变量 F,T,U,W,R,O ∈ {0..9}，辅助进位变量 C1,C2,C3 ∈ {0,1}；约束为 Alldiff(F,T,U,W,R,O) 加四个按列方程：O+O=R+10·C1、C1+W+W=U+10·C2、C2+T+T=O+10·C3、C3=F，且 T≠0、F≠0（不允许前导零）。约束超图（图6-2b）中方块表示高阶约束、圆表示变量。

最少冲突求解 8 皇后（图6-8，两步）：左棋盘随机选中最右列的冲突皇后 Q8，该列各方格冲突数（相互攻击的皇后对数）自上而下为 2,2,1,2,3,1,2（当前在底行），两格同为最小值 1，随机取 Q8=3；中棋盘再选 Q6，列上冲突数为 3,3,—,2,3,2,3,0，移至第 8 行（0 冲突）即得右棋盘的解。

互补视角：
- 冲突与回跳：按 Q,NSW,V,T,SA,WA,NT 顺序赋值，Q=r,NSW=g,V=b,T=r 后 SA 无值——时序回溯会去改 T（塔斯马尼亚！）；SA 的冲突集 {Q,NSW,V} 指向真凶，回跳直达 V。
- 约束学习：{WA=red, NT=green, Q=blue} 是 no-good（SA 必无值），若外层还对 V、T 赋值，则缓存它可避免反复撞墙。
- 结构：删去 SA 后约束图成为两棵树组成的森林（图 6-12）——SA 是大小 1 的环割集，固定 SA=red 后整树 O(nd²) 解出，对所有 SA 取值共 d^1 次即可；树分解（图 6-13）把图拆成 {WA,NT,SA}、{NT,Q,SA}、{Q,SA,NSW}、{SA,NSW,V}、{T} 五个节点（各取该变量组的联合值），最大节点含 3 个变量 ⇒ 树宽 2。

## 要点
1. 形式化为 CSP 的三大回报：表示自然、通用求解器高效、约束传播一次砍掉大片空间（选 SA=blue 后邻居组合从 3^5=243 降到 2^5=32，省 87%）。
2. 推断与搜索交替（AC-3/MAC）通常远胜纯搜索；传播有时直接解出（简单数独）或证明无解（空域/2 色地图的路径一致论证）。
3. 变量排序失败优先（MRV、度），值排序失败延后（最少约束值）；只在求一个解时值排序才有意义，枚举所有解时无所谓。
4. MAC 严格强于前向检验；前向检验已使简单回跳冗余，但冲突导向回跳与约束学习仍是现代求解器（含 SAT 的 CDCL 思想）的核心。
5. min-conflicts 在解稠密问题上惊人有效（百万皇后 ≈50 步），且天然适合在线重调度（从当前方案出发最少改动修复）。
6. 复杂度由结构而非规模决定：树宽有界 ⇒ 多项式可解；最小割集/最优分解虽 NP 难，但近似算法实践够用；割集调整省内存、树分解省时间。
7. 打破值对称可把搜索空间砍 d! 倍（如强制 WA<NT<SA 的颜色字母序）。

## 关联
- **第3章**：回溯 = 利用可交换性受限的深度优先搜索；启发式从领域特定变为领域无关。
- **第4章**：min-conflicts 就是 CSP 上的局部搜索（最少冲突计数即目标函数，平台区对策同源）；调度问题也可用连续优化。
- **第7章**：难/易问题与相变现象——为何 n 皇后对局部搜索如此容易。
- **第13章**：割集调整同样用于概率推理中的图结构利用。
- **外部概念**：SAT 求解（子句学习=no-good 学习）、运筹学与调度、数据库查询优化、参数化复杂度（树宽）、数独/密码算术等经典谜题。
