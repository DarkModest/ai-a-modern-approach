# 第15章 概率编程 (Probabilistic Programming)

## 核心思想
贝叶斯网络是命题式的因子化表示，无法紧凑描述有大量对象和重复结构的域；本章给出两条通往通用表达能力的路径——基于逻辑的关系/开宇宙概率模型（RPM/OUPM）与基于程序的生成程序（PPL），并配备通用推断算法（落地、提升推断、MCMC），使概率模型像图灵机一样"计算上通用"。

## 框架
- **关系概率模型（Relational Probability Model, RPM）**
  - 何时使用：对象集合与身份已知（数据库语义：唯一命名假设 + 域闭包），但对象间关系与属性有不确定性
  - 如何做：带类型签名的函数/谓词（如 Honest: Customer→{true,false}、Kindness: Customer→{1..5}、Quality: Book→{1..5}、Recommendation: Customer×Book→{1..5}）+ 依赖声明（父变量与共享 CPT 模板），例如 Recommendation(c,b) ~ if Honest(c) then HonestRecCPT(Kindness(c), Quality(b)) else ⟨0.4, 0.1, 0.0, 0.1, 0.4⟩，先验如 Kindness(c) ~ ⟨0.1, 0.1, 0.2, 0.3, 0.3⟩、Quality(b) ~ ⟨0.05, 0.2, 0.4, 0.2, 0.15⟩；对常量的所有组合实例化得到**基本随机变量**（basic random variable），展开为等价贝叶斯网络；依赖必须无环且有依据（无无限祖先链）
- **开宇宙概率模型（Open-Universe Probability Model, OUPM）**
  - 何时使用：对象数量未知（存在不确定性）或哪些术语指同一对象未知（身份不确定性），如引文去重、多 ISBN、女巫攻击（sybil attack）
  - 如何做：一阶逻辑标准语义之上加**数字语句**（number statement，指定各类型对象数量的条件分布，常用泊松或数量级分布）+ 起源函数（origin function，记录对象由谁生成）；对象即生成历史，保证每个世界恰有一个生成序列，世界概率 = 各次采样值概率之积（图15-4：一个含 #Customer=2、#Book=3 的世界，其概率为 Honest/Kindness/Quality/#LoginID 与全部 Recommendation 取值概率的连乘）
- **生成程序（Generative Program）/ 概率编程语言（PPL）**
  - 何时使用：需要完整编程语言的表达能力（递归、复杂数据结构、高阶函数），或模型含任意结构的噪声过程
  - 如何做：在普通程序中引入随机源；可能世界 = 执行轨迹，轨迹概率 = 各次随机选择概率之积；可计算通用（能表示任何可由停机概率图灵机采样的分布）
- **数据关联（Data Association）多目标追踪**
  - 何时使用：多个对象产生观测且不知道哪个观测来自哪个对象（时序化的身份不确定性）
  - 如何做：把 Source(b) 等关联变量作为随机变量纳入 OUPM，补充虚假警报（false alarm/杂波）、检测故障（detection failure）、目标进出；推断用最近邻滤波器→匈牙利算法（最优赋值）→粒子滤波/MCMC 多假设→Rao-Blackwellization（给定关联假设后逐对象跑精确卡尔曼滤波）
- **模型-证据-查询三件套（OUPM 标准用法）**：模型 = 类型 + 类型签名 + 数字语句 + 依赖语句；证据 = 已知事实；查询 = 可含自由逻辑变量的任意表达式；答案 = 替换的联合后验分布

## 关键概念
- **数据库语义（database semantics）** — 唯一命名假设 + 域闭包：每个常量对应唯一已知对象，世界有限，RPM 的基础。
- **依赖声明（dependency statement）** — 声明某函数对每个对象元组的父变量与条件分布，是 RPM 的"CPT 模板"。
- **特定上下文的独立性（context-specific independence）** — 用 if–then–else 表达"条件测试本身是随机变量、推断无需知道其值"的依赖细化（如不诚实顾客的评分与质量无关）。
- **多路选择器（multiplexer）** — 当某函数值（如 Author(b)）未知时，用随机变量作选择器决定哪些候选父变量真正生效，建模关系不确定性。
- **存在不确定性（existence uncertainty）/ 身份不确定性（identity uncertainty）** — 观测不指明真实对象的存在（多少本书/间谍）与同一性（哪些术语指同一对象），OUPM 的核心动机。
- **数量级分布（order-of-magnitude distribution）** — 对数（以 10 为底）呈正态的离散分布，如 OM(3,1) 均值 10³、标准差一个数量级，适合无界对象数。
- **落地/展开（grounding/unrolling）** — 把 RPM 实例化为等价贝叶斯网络，对一阶逻辑命题化的精确模拟；可能产生巨大网络。
- **提升推断（lifted inference）** — 一个提升因子代表一整族仅常量不同的落地因子，每个提升步骤完成许多基础步骤的工作。
- **最小自支撑实例化（minimal self-supporting instantiation）** — 证据与查询变量的祖先闭包；MCMC 对 OUPM 采样的就是这样的部分世界。
- **执行轨迹（execution trace）** — 生成程序各次随机选择的取值序列，构成 PPL 的样本空间。
- **保证对象（guaranteed object）** — 被保证存在且彼此不同的对象（如追踪场景中的已知飞行器）。

## 思维模型
- 把 RPM 看作"展开成贝叶斯网络的模板"：一阶谓词 + 共享 CPT 把指数级网络压缩到几百个参数（1000 万本书 × 10 亿顾客 ≈ 10^(7×10^15) 个世界，参数 < 300）。
- 把 OUPM 看作"逐事件生成世界"：生成事件不是变量赋值而是对象的存在本身；每个对象记住生成历史以避免多条生成路径重复计数。
- 把生成程序看作"带随机选择点的程序"：轨迹即世界，概率即随机选择之积——改模型（如字母独立→字母二元马尔可夫）比改推断更易改善结果。
- 观测不带唯一 ID 时（引文、雷达光点、车辆图像），把它当作"未知对象按物理过程产生模糊感知"的开宇宙问题；多个感知共享来源会互相增强推断。

## 反模式
- **手工为大规模对象域画贝叶斯网络**：节点与 CPT 随对象数爆炸，既不可手工构造也不可从数据学习。
- **假设世界封闭、对象身份已知**：一个"逻辑实体"多版本（多 ISBN）、一人多 ID（女巫攻击）、一论文多引文字符串，闭宇宙假设导致系统性错误（2002 年 CiteSeer 把 Russell & Norvig 列为 120 多本不同的书）。
- **对落地网络硬做精确推断**：应只实例化查询/证据的祖先（相关变量），缓存重复因子（可加速 3 个数量级），或用提升推断/MCMC。
- **每步只接受单一最佳关联赋值**（最近邻/匈牙利）：一旦选错，后续预测连锁错误；混淆场景必须维护多假设并允许修改历史赋值。
- **把模型改进与推断改进混为一谈**：OCR 例中先改进推断（更多迭代、更好提议），再改进模型（字母二元模型）才是正解。

## 算法要点
**RPM → 等价贝叶斯网络（落地；忠实转录，图书推荐例，B 本书、C 位顾客）**
```
for b = 1 to B do
    增加不带父节点的节点 Quality_b，先验为 ⟨0.05, 0.2, 0.4, 0.2, 0.15⟩
for c = 1 to C do
    增加不带父节点的节点 Honest_c，先验为 ⟨0.99, 0.01⟩
    增加不带父节点的节点 Kindness_c，先验为 ⟨0.1, 0.1, 0.2, 0.3, 0.3⟩
    for b = 1 to B do
        增加带父节点的节点 Recommendation_{c,b}，
        其父节点为 Honest_c、Kindness_c、Quality_b，
        且服从条件分布 RecCPT(Honest_c, Kindness_c, Quality_b)
```
推断捷径：从查询和证据出发"链接"模型，只构造相关的隐式网络片段。

**OUPM 的推断（MCMC 为主）**
- 状态空间 = 对象与关系的可能世界集合；转移可改关系/函数值、**增删对象**、改变常量符号的解释
- 每步概率计算完全局部（吉布斯/MH 概率比只取决于固定大小的变化子图），逻辑查询在每个访问世界**递增**评估
- 采样对象是部分世界：证据与查询变量的最小自支撑实例化（如只取时间序列的有限前缀）

**图15-11 OCR 生成程序（忠实转录；本章核心示例）**
```
function GENERATE-IMAGE() returns 一张包含一些字母的图像
  letters ← GENERATE-LETTERS(10)
  return RENDER-NOISY-IMAGE(letters, 32, 128)

function GENERATE-LETTERS(λ) returns 一个由字母组成的向量
  n ~ Poisson(λ)
  letters ← []
  for i = 1 to n do
      letters[i] ~ UniformChoice({a, b, c, …})
  return letters

function RENDER-NOISY-IMAGE(letters, width, height) returns 一张包含这些字母的带噪声的图像
  clean_image ← RENDER(letters, width, height, text_top = 10, text_left = 10)
  noisy_image ← []
  noise_variance ~ UniformReal(0.1, 1)
  for row = 1 to width do
      for col = 1 to height do
          noisy_image[row, col] ~ N(clean_image[row, col], noise_variance)
  return noisy_image
```

**图15-15 改进的 OCR 生成程序（字母二元马尔可夫模型替代独立字母，忠实转录）**
```
function GENERATE-MARKOV-LETTERS(λ) returns 一个字母的向量
  n ~ Poisson(λ)
  letters ← []
  letter_probs ← MARKOV-INITIAL()
  for i = 1 to n do
      letters[i] ~ Categorical(letter_probs)
      letter_probs ← MARKOV-TRANSITION(letters[i])
  return letters
```

**生成程序的推断**
- 拒绝采样（只保留与证据一致的轨迹）/ 似然加权（权重 = 沿途观测概率之积）/ MCMC（采样并修改执行轨迹，if 分支改变会使后续轨迹失效）
- 加速：编译推断（2–3 个数量级）、专用蒙特卡罗硬件（100–10000 倍）、自适应提议分布、用模型合成数据训练深度网络充当提议
- 可判定性：仅含无限精度连续随机变量时推断不可判定；实际用有限精度 + 平滑分布时仍可判定

## 参考表
| 维度 | 贝叶斯网络 | RPM | OUPM | 生成程序 |
|---|---|---|---|---|
| 语义 | 命题（变量赋值） | 一阶 + 数据库语义 | 一阶标准语义（开宇宙） | 执行轨迹 |
| 可能世界 | 有限、固定变量集 | 有限（常量已知） | 可无限（对象数无界） | 可无限（随机选择数不定） |
| 表达能力 | 因子化 | 结构化（对象+关系） | + 存在/身份不确定性 | 计算通用 |
| 典型推断 | 变量消元/聚类 | 落化后同左；提升推断 | MCMC over worlds | 轨迹 MCMC/重要性采样 |

## 完整示例
**图书推荐 RPM**：变量 Honest(c)、Kindness(c)、Quality(b)（1–5）、Recommendation(c,b)（共享 RecCPT，2×5×5=50 行）；粉丝假说的依赖声明：Recommendation(c,b) ~ if Honest(c) then (if Fan(c, Author(b)) then Exactly(5) else HonestRecCPT(Kindness(c), Quality(b))) else ⟨0.4, 0.1, 0.0, 0.1, 0.4⟩。当 Author(B₂) 未知时（图15-3），Recommendation(C₁,B₂) 的父变量含 Author(B₂) 与 Fan(C₁,A₁)、Fan(C₁,A₂)（多路选择器：作者是谁决定哪个 Fan 生效）——若 3 位 A₁ 的粉丝都给 B₂ 打 5 分而其他人打低分，则"A₁ 是作者"的后验大增，且评估 A₁ 的书时会折扣其粉丝的 5 分。

**引文匹配 OUPM（Pasula et al., 2003；图15-5 忠实转录）**：
```
type Researcher, Paper, Citation
random String Name(Researcher)
random String Title(Paper)
random Paper CitedPaper(Citation)
random String Text(Citation)
random Boolean Professor(Researcher)
origin Researcher Author(Paper)

#Researcher ~ OM(3, 1)
Name(r) ~ NamePrior()
Professor(r) ~ Boolean(0.2)
#Paper(Author = r) ~ if Professor(r) then OM(1.5, 0.5) else OM(1, 0.5)
Title(p) ~ PaperTitlePrior()
CitedPaper(c) ~ UniformChoice({Paper p})
Text(c) ~ HMMGrammar(Name(Author(CitedPaper(c))), Title(CitedPaper(c)))
```
仅以引文字符串为证据做推断，错误率为 CiteSeer 的 1/2–1/3，且呈现"集体消歧"：引用越多解析越准。

**多目标追踪 OUPM（图15-9 忠实转录；图15-8：观测光点不带对象身份，n 对象 T 步共 (n!)ᵀ 种关联假设）**：
```
#Aircraft(EntryTime = t) ~ Poisson(λ_a)
Exits(a, t) ~ if InFlight(a, t) then Boolean(α_e)
InFlight(a, t) = (t = EntryTime(a)) ∨ (InFlight(a, t−1) ∧ ¬Exits(a, t−1))
X(a, t) ~ if t = EntryTime(a) then InitX()
          else if InFlight(a, t) then N(F X(a, t−1), Σ_x)
#Blip(Source = a, Time = t) ~ if InFlight(a, t) then Bernoulli(DetectionProb(X(a, t)))
#Blip(Time = t) ~ Poisson(λ_f)
Z(b) ~ if Source(b) = null then UniformZ(R) else N(H X(Source(b), Time(b)), Σ_z)
```
推断升级路径：最近邻滤波器（对象分离好时）→ 匈牙利算法（当前预测下最大化联合概率的赋值）→ 粒子滤波（维持大量当前赋值）/ MCMC（探索历史赋值空间）→ Rao-Blackwellization：给定关联假设后逐对象精确卡尔曼滤波，换假设时只重算受影响对象；可实时处理数百目标。

**NET-VISA（CTBTO 核监约监控；图15-6 简化版忠实转录）**：
```
#SeismicEvents ~ Poisson(T·λ_e)
Time(e) ~ UniformReal(0, T)
EarthQuake(e) ~ Boolean(0.999)
Location(e) ~ if Earthquake(e) then SpatialPrior() else UniformEarth()
Depth(e) ~ if Earthquake(e) then UniformReal(0, 700) else Exactly(0)
Magnitude(e) ~ Exponential(log(10))
Detected(e, p, s) ~ Logistic(weights(s, p), Magnitude(e), Depth(e), Dist(e, s))
#Detections(site = s) ~ Poisson(T·λ_f(s))
#Detections(event=e, phase=p, station=s) = if Detected(e, p, s) then 1 else 0
OnsetTime(a, s)：event(a)=null 时 ~ UniformReal(0, T)；否则 = Time(event(a)) + GeoTT(Dist, Depth, phase) + Laplace(μ_t(s), σ_t(s))
Amplitude / Azimuth / Slowness：事件为 null 时用噪声/均匀模型（NoiseAmpModel(s)、UniformReal(0, 360)、UniformReal(0, 20)）；否则 = 地球物理模型 + Laplace 噪声
ObservedPhase(a, s) ~ CategoricalPhaseModel(phase(a))
```
2009 年震级 3–4 的 27 294 个事件中，联合国 SEL3 公告漏检 27.4%，NET-VISA 漏检 11.1%，比人类专家最终公告多发现 50% 的真实事件；2018 年起实际部署。

**OCR 推断结果（图15-13/15-14）**：一条含 "ocflwe" 的轨迹展开有 4104 个节点（1 个字母数 + 6 个字母 + 1 个噪声方差 + 4096 个像素）。中等噪声下 3 轮 MCMC（各 25 次迭代）均收敛到 "uncertainty"；极端噪声下独立字母模型与二元模型都有歧义，但二元模型的解释更接近真值——模型先验知识替代推断算力。

## 要点
1. 表示能力阶梯：命题（贝叶斯网络）→ 关系（RPM，数据库语义）→ 开宇宙（OUPM，存在/身份不确定性）→ 生成程序（PPL，计算通用）。
2. RPM 的参数量与对象数无关——这是它能建模十亿级对象域的唯一原因。
3. 关系不确定性使依赖结构本身随机（多路选择器），这是命题模型无法表达的。
4. 推断以查询为中心：落地整个隐式网络几乎总是不必要的。
5. 概率一旦用 PPL 表达，通用推断机即免除为每个模型重新做数学推导（多目标追踪文献中上千篇论文的推导可被一次模型编写替代）。
6. "模型改进 vs 推断改进"是 PPL 工作流的核心循环：先诊断（混合不足还是模型不对），再选择改模型或改提议；未观测对象也可被推断存在（天王星摄动→海王星）：存在性经由对已观测对象的影响传播。

## 关联
- **第13–14章**：贝叶斯网络、DBN/卡尔曼滤波是本章的因子化基础；多目标追踪是第14章时序推理 + 身份不确定性的结合。
- **第8章**：一阶逻辑语义（标准 vs 数据库）直接决定 RPM/OUPM 的世界观。
- **第9章**：说明性 PPL 与逻辑编程之于通用 PPL 的关系，如同 Prolog 之于通用语言。
- **外部概念**：Church/WebPPL/BLOG 等概率编程语言；微软 TrueSkill（Skill→Performance、团队表现=个人之和）；NET-VISA/CTBTO；数据关联与多目标追踪（雷达、计算机视觉）。
