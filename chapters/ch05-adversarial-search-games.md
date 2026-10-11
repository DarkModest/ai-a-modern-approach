# 第5章 对抗搜索和博弈 (Adversarial Search and Games)

## 核心思想
当环境中有智能体积极试图击败你时，最优行动由博弈树的极小化极大值决定；实践中用 α-β 剪枝加启发式评价函数、或蒙特卡罗树搜索，在有限时间内近似这一最优。

## 框架
- **极小化极大（Minimax Search）**：确定性、轮流、完美信息、双人零和博弈的最优决策算法
  - 何时使用：博弈树小到可以全部枚举（或作理论基准）
  - 如何做：终局取 Utility，Max 层取 max、Min 层取 min，深度优先自叶倒推；最优移动 = 指向极小化极大值最高后继的移动
- **α-β 剪枝（Alpha–Beta Pruning）**：剪除可证明不影响决策的子树
  - 何时使用：任何极小化极大场景（时间永远是瓶颈）
  - 如何做：维护 α（迄今 max 选择点的最佳值，"至少"）与 β（迄今 min 选择点的最佳值，"至多"），值越界即截断；效率取决于移动排序（绝招启发式、迭代加深、换位表）
- **启发式 α-β 树搜索（Heuristic Alpha–Beta）**：截断测试 + 评价函数
  - 何时使用：树搜不完的一切真实博弈
  - 如何做：以 Is-Cutoff（深度+终止）代替 Is-Terminal，以 Eval（特征加权线性函数）代替 Utility；只评静态局面（静态搜索），用单步延伸缓解视野效应；可加前向剪枝（ProbCut、后期移动缩减）
- **蒙特卡罗树搜索（Monte Carlo Tree Search, MCTS/UCT）**：用模拟到终局的平均值代替评价函数
  - 何时使用：分支因子极高或评价函数难定义（围棋 b=361）、全新博弈无先验知识
  - 如何做：选择→扩展→模拟(rollout)→反向传播循环至时间耗尽；UCB1 公式平衡探索/利用；返回模拟次数最多的移动
- **期望极小化极大（Expectiminimax）**：含机会节点的推广
  - 何时使用：随机博弈（西洋双陆棋等掷骰子）
  - 如何做：机会节点取概率加权平均；评价函数必须返回获胜概率的正线性变换值

## 关键概念
- **零和博弈（zero-sum game）** — 一方所得即另一方所失（严格说是常量和）；"完美信息"= 完全可观测的同义词。
- **效用函数（utility function）** — 终局状态给参与者 p 的数值收益（国际象棋赢/平/输 = 1/½/0）。
- **层（ply）** — 一个玩家的一次移动（博弈树深入一层）。
- **极小化极大值（minimax value）** — 双方此后都按最优策略行动时，终局对 max 的效用。
- **换位/换位表（transposition / transposition table）** — 不同移动序列到达同一局面；缓存其倒推值可让同等时间搜索深度翻倍。
- **评价函数（evaluation function）** — 非终局期望效用的估计，典型为加权线性函数 Eval=Σwᵢfᵢ（fᵢ=特征如子力）；只需与获胜机会单调相关，且计算要快。
- **静态局面/静态搜索（quiescence / quiescence search）** — 无大幅摇摆待着法（如吃后）的局面才可评估；非静态局面继续（常限于吃子着法）搜索。
- **视野效应（horizon effect）** — 用拖延着法把不可避免的损失推出搜索视野之外；缓解靠单步延伸（singular extension）。
- **前向剪枝（forward pruning）** — 剪掉"看似差"的着法（束搜索、ProbCut、后期移动缩减），以出错风险换时间；对应香农 B 型策略。
- **A 型/B 型策略（Type A / Type B strategy）** — 香农：全宽度浅搜 vs 选择性深搜；MCTS 本质是 B 型。
- **模拟（simulation / playout / rollout）** — 按模拟策略为双方快速选步直到终局，由规则（而非启发式）判分。
- **UCB1/UCT** — 应用于树搜索的置信上界选择策略：利用项 U(n)/N(n) + 探索项 C·√(ln N(Parent(n))/N(n))。
- **机会节点（chance node）** — 按概率分支随机事件（骰子结果）的节点；取期望值。
- **确保将死/概率将死（guaranteed / probabilistic checkmate）** — 部分可观测博弈的胜利标准：对信念状态中每种局面及每种感知序列都获胜 / 以概率 1 获胜（允许己方随机走）。
- **观测力平均（averaging over clairvoyance）** — 对每种发牌假设"之后双方全知"再求期望的近似方法；系统性缺陷是忽略未来的信念状态。
- **元推理（metareasoning）** — 关于"该做什么计算"的推理：按节点扩展的效用 vs 计算代价分配算力。

## 思维模型
- 当对手存在时，把搜索树看作 max/min 交替层；max 的"策略"是对 min 每种可能回应都有应手的条件规划（与或搜索对多值效用的推广）。
- 把 α/β 读作"至少/至多"：α = 路径上 max 已保证的值，β = min 已保证的值；当前节点值一旦越出 [α,β] 即无人会选它。
- 当评价函数难写时，把"评估状态"换成"从该状态模拟到终局再取平均"（MCTS）。
- 把不完美信息博弈看作信念状态上的博弈：收集信息、隐藏信息、随机化都是策略的一等成分。
- 把算力本身看作资源：搜索/模拟只是在"树的重要部分"上做非最优的元推理。

## 反模式
- **观测力平均**：假设发牌后自动全知——永不诈唬、永不收集信息；金子与公汽车的三天故事说明它在第三天会把智能体引向可能的死亡。
- **在非静态局面调用评价函数**：子力领先但下一手就丢后的局面会被高估；必须静态搜索或单步延伸。
- **用束搜索做前向剪枝**：无法保证最佳移动不被剪掉；ProbCut 用统计显著性降低剪错概率。
- **对次优对手机械使用极小化极大**：对最优对手它是最优的，且对次优对手"至少不差"；但当对手有 9/10 概率犯错时，冒险走法可能优于稳平。
- **MCTS 用于单步定生死的博弈**：随机模拟可能根本没有探索关键着法（B 型剪枝的代价）；对"明显胜负"的局面也要模拟很久确认。
- **机会博弈评价函数与胜率不是正线性变换**：保持叶值排序不变、只改量级（[1,2,3,4] → [1,20,30,400]）也会翻转机会节点处的决策。

## 算法要点

极小化极大（图5-3；Max-Value/Min-Value 返回 (utility, move) 对）
```
function Minimax-Search(game, state) returns 一个动作
    player ← game.To-Move(state)
    value, move ← Max-Value(game, state)
    return move

function Max-Value(game, state) returns 一个(utility, move)对
    if game.Is-Terminal(state) then return game.Utility(state, player), null
    v, move ← −∞, null
    for each a in game.Actions(state) do
        v2, a2 ← Min-Value(game, game.Result(state, a))
        if v2 > v then v, move ← v2, a
    return v, move

function Min-Value(game, state) returns 一个(utility, move)对
    if game.Is-Terminal(state) then return game.Utility(state, player), null
    v, move ← +∞, null
    for each a in game.Actions(state) do
        v2, a2 ← Max-Value(game, game.Result(state, a))
        if v2 < v then v, move ← v2, a
    return v, move
```
时间 O(b^m)；空间 O(m)（逐个生成动作）。

α-β 剪枝搜索（图5-7；与图5-3 相同，仅多维护 α、β 并在越界时截断）
```
function Alpha-Beta-Search(game, state) returns 一个动作
    player ← game.To-Move(state)
    value, move ← Max-Value(game, state, −∞, +∞)
    return move

function Max-Value(game, state, α, β) returns 一个(utility, move)对
    if game.Is-Terminal(state) then return game.Utility(state, player), null
    v, move ← −∞, null
    for each a in game.Actions(state) do
        v2, a2 ← Min-Value(game, game.Result(state, a), α, β)
        if v2 > v then
            v, move ← v2, a
            α ← Max(α, v)
        if v ≥ β then return v, move     // β 截断：上层 min 不会走这条路
    return v, move

function Min-Value(game, state, α, β) returns 一个(utility, move)对
    if game.Is-Terminal(state) then return game.Utility(state, player), null
    v, move ← +∞, null
    for each a in game.Actions(state) do
        v2, a2 ← Max-Value(game, game.Result(state, a), α, β)
        if v2 < v then
            v, move ← v2, a
            β ← Min(β, v)
        if v ≤ α then return v, move     // α 截断：上层 max 不会走这条路
    return v, move
```
（中文版图中 Min-Value 误印为 β←Max(β, v)，应为 Min。）
完美移动排序只需 O(b^{m/2}) 节点（有效分支因子 √b，国际象棋 35→6，同等时间深度翻倍）；简单排序（吃子→威胁→前进）即可达最好情况的约 2 倍以内。

启发式版：把 Is-Terminal 换成 Is-Cutoff(s,depth)（固定深度或迭代加深），Utility 换成 Eval(s,p)。

期望极小化极大（Expectiminimax 公式，5.3 节）
```
function Expectiminimax(s):
    if 终局: return Utility(s)
    if max 节点: return max_a Expectiminimax(Result(s,a))
    if min 节点: return min_a Expectiminimax(Result(s,a))
    if 机会节点: return Σ_r P(r) · Expectiminimax(Result(s,r))
```
O(b^m n^m)；西洋双陆棋 n=21、b≈20（可到 4000），实际只能搜约 3 层；机会节点剪枝需限定效用取值范围。

蒙特卡罗树搜索（图5-11；Select/Expand/Simulate/Back-Propagate 循环，常用 UCT 即 UCB1 作选择度量）
```
function Monte-Carlo-Tree-Search(state) returns 一个动作
    tree ← Node(state)
    while Is-Time-Remaining() do
        leaf ← Select(tree)           // 从根沿树下行, 每步选 UCB1 最大的子节点
        child ← Expand(leaf)          // 为 leaf 生成一个新子节点
        result ← Simulate(child)      // 按模拟策略 playout 到终局
        Back-Propagate(result, child) // 沿途 N ← N+1, U ← U+result
    return Actions(state) 中指向模拟次数最多的节点的移动

UCB1(n) = U(n)/N(n) + C·√(ln N(Parent(n)) / N(n))
```
单次模拟代价对深度是线性而非指数；返回访问最多而非均值最高的节点（胜 65/100 优于胜 2/3——后者不确定性大）。迭代示例（图5-10，已完成100次迭代）：(a) 选择沿 UCB1 走到叶 27/35；(b) 扩展新子节点 0/0，模拟得黑胜；(c) 反向传播 0/1→28/36→16/54→61/80→37/101（标注 = 胜场/场次）。

## 参考表

| 算法 | 节点/时间复杂度 | 前提 | 典型适用 |
|---|---|---|---|
| 极小化极大 | O(b^m) | 可枚举整树 | 小博弈、理论基准 |
| α-β（完美排序） | O(b^{m/2}) | 评价函数+好的移动排序 | 国际象棋类（分支因子适中） |
| 期望极小化极大 | O(b^m n^m) | 骰子概率已知 | 西洋双陆棋（约 3 层） |
| MCTS | 时间预算内迭代 | 只需规则+模拟策略 | 围棋（b=361）、无先验新博弈 |

书中算力对照：每秒评估约 100 万节点、b=32、平均 100 步的博弈、预算 10 亿状态——极小化极大约 6 层，完美排序 α-β 约 12 层，MCTS 约 1000 万次模拟。国际象棋实战：纯极小化极大约 5 层（35^5≈5000万节点/分钟，被平均水平的人类棋手击败）；α-β+换位表约 14 层（专家级）；Stockfish 深度超 30。

## 完整示例

**α-β 剪枝逐步演示（图5-2 / 图5-5）**：二层博弈树，根为 max 节点 A，动作 a₁/a₂/a₃ 分别到达 min 节点 B、C、D；B 的叶 = 3、12、8，C 的叶 = 2、4、6，D 的叶 = 14、5、2；极小化极大值 A=3、B=3、C=2、D=2。各步中每个节点可能的取值范围（图5-5 a–f）：

- (a) B 的第一个叶 = 3 → B ∈ (−∞, 3]；根仍 (−∞, +∞)。
- (b) B 的第二个叶 = 12 > 3 → B 不变，仍 (−∞, 3]。
- (c) B 的第三个叶 = 8 → B = [3, 3]；根更新为 [3, +∞)（α=3，"至少 3"）。
- (d) C 的第一个叶 = 2 → C ∈ (−∞, 2] ≤ α=3 → max 在根必不选 C，剪掉 C 其余两叶。
- (e) D 的第一个叶 = 14 → D ∈ (−∞, 14]；根的区间收缩为 [3, 14]。
- (f) D 的第二、三个叶 = 5、2 → D = [2, 2]；根 = max(3, ≤2, 2) = [3, 3] → 最优决策 a₁，min 的最佳回应是 b₁。

共评估 7 个叶而非 9 个。若移动排序让 D 的第三个叶（值 2）最先生成，D 还能再剪两叶——排序质量直接决定剪枝收益（"应先检查有可能是最佳选择的节点"）。

**叶值赋值改变最佳移动（图5-14）**：max 根下 a₁/a₂ 各到一个机会节点（P=0.9/0.1）再到 min 叶。叶值 [1,2,3,4] 时：a₁ = 0.9·2+0.1·3 = 2.1 > a₂ = 0.9·1+0.1·4 = 1.3，选 a₁；保持叶值排序不变改为 [1,20,30,400]：a₁ = 21 < a₂ = 0.9·1+0.1·400 = 40.9，改选 a₂。故机会博弈中评价函数必须返回获胜概率的正线性变换值。

**启发式极小化极大可能出错（图5-16）**：max 根下两个 min 节点，左叶 {99, 1000, 1000, 1000} → min = 99，右叶 {100, 101, 102, 100} → min = 100；启发式极小化极大选右（100 > 99），但若 min 可能犯错，左侧期望（均值 774.75）远优于右侧（100.75）——对次优对手，"极小化极大最优"未必实战最优。

**西洋双陆棋博弈树（图5-13）**：max → 机会节点（掷两骰：对子 1-1、6-6 各 P=1/36，非对子 1-2、6-5 各 P=1/18，共 21 种结果）→ min → 机会 → max → 终局。正因 n=21 的机会分支，期望极小化极大 O(b^m·n^m) 在此只能搜约 3 层。

## 要点
1. 博弈 6 元组形式化：S₀、To-Move、Actions、Result、Is-Terminal、Utility；极小化极大是与或搜索到多值效用的推广。
2. α-β 不改变决策只删节点：收益全靠移动排序——绝招启发式、迭代加深、换位表让有效分支因子从 b 降到约 √b。
3. 时间不够就截断+评价函数；评价函数是期望效用（特征加权和），只能用于静态局面，否则被水平线/视野效应欺骗。
4. MCTS 用规则判分代替启发式评估：四步循环+UCB1，天然适配自我对弈学习（AlphaZero 加神经网络先验）；返回访问次数最多的移动。
5. 随机博弈复杂度多一个因子 n^m；评价必须返回胜率的正线性变换，否则量纲错误会翻转决策。
6. 不完美信息博弈：用信念状态+与或搜索可求"确保将死"；最优玩法必须随机化（可预测的策略向对手泄密）；观测力平均是有系统缺陷的实用近似，配抽象与采样（Libratus 范式）。
7. 开局库与残局查表（逆向极小化极大，7 子残局 400 万亿状态）让程序两端免搜索；更深层的改进方向是元推理与层级规划。

## 关联
- **第4章**：与或搜索、信念状态、非确定性框架被直接复用；博弈是"对手带来的非确定性"的显式建模。
- **第16章**：效用理论解释"评价须为胜率的正线性变换"。
- **第17章**：UCB1/探索-利用的严格处理；均衡解为每人指定最优随机策略。
- **第18章**：博弈论——多人联盟、不完美信息博弈的均衡。
- **第22章**：MCTS 与强化学习同源（模拟未来、按结果评价行动）。
- **第19章**：机器学习取代人类专家知识（评价函数权重、模拟策略、AlphaZero 自我对弈）。
- **外部概念**：博弈论（Nash 均衡）、多臂老虎机（UCB）、AlphaZero/Libratus/AlphaStar 等里程碑系统。
