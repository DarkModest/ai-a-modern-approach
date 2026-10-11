# 第19章 样例学习 (Learning from Examples)

## 核心思想
学习的目标不是拟合训练集，而是泛化到未观测的样例：在"与数据一致"和"假设简单"之间权衡（奥卡姆剃刀），用留出的测试/验证数据做裁判。本章围绕这一主轴展开决策树、线性模型、非参数模型、SVM 与集成方法，并给出样本复杂性与工程实践的完整图景。

## 框架
- **决策树归纳（Decision Tree Learning，Learn-Decision-Tree，图19-5）**
  - 何时使用：离散/连续属性的分类与回归（CART），尤其需要可解释模型时
  - 如何做：贪心分治——每次选信息增益最大的属性分裂，递归处理子集；样例空则返回父样例多数类、属性空则返回本样例多数类；事后用 χ² 显著性检验剪枝（先生成后剪枝，优于提前停止）
  - 信息增益：熵 H(V)=−Σ_k P(v_k)·log₂P(v_k)；布尔熵 B(q)=−(q·log₂q+(1−q)·log₂(1−q))；Gain(A)=B(p/(p+n))−Remainder(A)，Remainder(A)=Σ_{k=1..d} ((p_k+n_k)/(p+n))·B(p_k/(p_k+n_k))。熵示例（原书行内公式图）：均匀硬币 H(Fair)=−(0.5·log₂0.5+0.5·log₂0.5)=1 位；均匀四面骰 H(Die4)=−(0.25·log₂0.25×4)=2 位；99% 正面的偏硬币 H(Loaded)=B(0.99)≈0.08 位
- **模型选择与交叉验证（Model Selection / k-fold Cross-Validation，图19-8）**
  - 何时使用：需要在多个假设空间或超参数值中挑选
  - 如何做：训练集训练、验证集选模型、测试集只碰一次（防研究者泄漏）；数据不足时 k 折交叉验证（k=5/10），超参数定后用全部数据重训
- **正则化与最小描述长度（Regularization / MDL）**
  - 何时使用：假设空间大、数据有限、防过拟合
  - 如何做：最小化总代价 Cost(h)=EmpLoss(h)+λ·Complexity(h)；线性函数取 Complexity(h_w)=L_q(w)=Σ_i|w_i|^q——q=1（L1）产生稀疏解（不相关特征权重归零），q=2（L2）平滑且旋转不变；MDL 直接最小化"假设编码位数+数据残余编码位数"
- **PAC 学习理论（Probably Approximately Correct Learning）**
  - 何时使用：分析"需要多少样例才能保证泛化"
  - 如何做：错误率 error(h)=GenLoss_{L_{0/1}}(h)=Σ_{x,y} L_{0/1}(y,h(x))·P(x,y)；N ≥ (1/ε)(ln(1/δ)+ln|H|) 个 i.i.d. 样例后，一致性假设以 ≥1−δ 概率误差 ≤ε；受限假设空间（如 k-DL 决策列表）样本复杂性是 n 的多项式，而全部布尔函数（|H|=2^(2^n)）需 2^n 级
- **线性模型家族（Linear Regression / 感知机 / 逻辑斯谛回归）**
  - 何时使用：数值输入的回归，或（近）线性可分的分类
  - 如何做：回归最小化 L2 损失（闭式正规方程 w*=(X^TX)^{-1}X^Ty，(X^TX)^{-1}X^T 为伪逆；或梯度下降）；分类用硬阈值得感知机更新规则（线性可分才收敛，α 需 O(1/t) 衰减），换软阈值 Logistic(z)=1/(1+e^{−z}) 得逻辑斯谛回归，梯度下降稳定收敛、输出可解释为概率
- **支持向量机与核技巧（SVM / Kernel Trick）**
  - 何时使用：中小规模数据、高维输入、无线性可分时
  - 如何做：解对偶二次规划求最大边距分离器（决策只由少数支持向量决定）；数据只以点积出现 → 用核函数 K(x,x') 替换点积（默瑟定理保证对应某特征空间），隐式在数十亿维甚至无限维空间中线性分离
- **集成学习（Ensemble Learning：Bagging / 随机森林 / Stacking / AdaBoost / 梯度提升）**
  - 何时使用：降方差（bagging、随机森林，基模型不稳定如决策树）或降偏差（boosting、stacking，基模型太弱如决策树桩）
  - 如何做：bagging 有放回采样+投票；随机森林再随机化属性选择（每次分裂只看 √n 或 n/3 个属性）+袋外误差评估；AdaBoost 迭代加错分样例权重+加权多数投票；梯度提升让新树拟合损失的负梯度

## 关键概念
- **监督/无监督/强化学习（supervised / unsupervised / reinforcement learning）** — 按反馈类型划分的三种学习：输入-输出对、无标签模式、奖励惩罚。
- **假设空间（hypothesis space）** — 学习器可输出的所有候选函数 h 的集合；选择它就是在表达能力与计算/泛化难度间权衡。
- **泛化（generalization）** — 假设对未见输入仍预测准确的能力，只能在独立测试集上度量。
- **偏差-方差权衡（bias-variance tradeoff）** — 假设空间约束带来的系统性偏离（偏差/欠拟合）与训练集波动带来的假设变化（方差/过拟合）之间的取舍。
- **信息增益（information gain）** — 按属性 A 分裂后熵的期望减少量（公式见框架），决策树选择分裂属性的判据。
- **超参数（hyperparameter）** — 定义模型类而非单个模型的"旋钮"（树节点数、正则化系数 λ、学习率、k），用验证集调优。
- **损失函数（loss function）** — L(x,y,ŷ)=Utility(x, 用 y)−Utility(x, 用 ŷ)：L1（绝对误差 |y−ŷ|，抗离群）、L2（平方误差 (y−ŷ)²，高斯噪声下最大似然）、L0/1（错=1 对=0，即错误率）；对数损失用于分类的梯度提升；代价不对称时直接赋不同值（如 L(spam,nospam)=1、L(nospam,spam)=10）。泛化损失 GenLoss_L(h)=Σ_{(x,y)} L(y,h(x))·P(x,y)，经验损失 EmpLoss_{L,E}(h)=(1/N)·Σ_{(x,y)∈E} L(y,h(x))，学习器以 ĥ*=argmin_{h∈H} EmpLoss 逼近 h*=argmin GenLoss
- **线性可分与决策边界（linear separability / decision boundary）** — 存在直线（超平面）将两类数据完全分开；该超平面即线性分离器。
- **参数/非参数模型（parametric / nonparametric model）** — 前者用固定个数参数概括数据（线性回归），后者用数据本身表示模型（最近邻、SVM、局部加权回归）。
- **维数灾难（curse of dimensionality）** — 高维空间中 k 邻域平均边长 (k/N)^{1/n} 趋近整个空间（17 维达 1/2、200 维达 94%），近邻推断退化为外推。
- **无悔学习（no-regret learning）** — 随机加权多数算法的错误数 M < [M*·ln(1/β)+ln K]/(1−β)（K 个专家、最优专家错 M* 次）；K=10 时取 β 得当 1.39M*+4.6——对任意样例序列成立，即使数据由对手选取。

## 思维模型
- 当属性少、数据多、内存装不下时，把数据本身当模型（非参数）；当数据少时，用强约束的参数模型防过拟合。
- 把正则化看作"给简单假设更高先验概率"的频率派镜像（第20章的 MAP/MDL 会严格化这一点）。
- 把 L1 看作"对坐标轴认真的"惩罚：菱形约束框的角落在轴上，解因此稀疏；L2 是旋转不变的球，适合轴可任意旋转的情形。
- 当验证误差曲线呈 U 形（决策树），选最低点；当它插值后二次下降（深度网络、随机森林、boosting），继续加复杂度反而更优——先认清手里的模型类属于哪一类。
- 把 boosting 看作"让下一个基模型专注上一轮的错题"，把梯度提升看作"让下一棵树拟合损失梯度的负方向"。

## 反模式
- **在训练集上评估、或反复用测试集调超参数**：单个假设没作弊，但整个过程泄漏了测试集信息；测试集必须完全锁定到最后。
- **把一致性当泛化**：高次多项式能完美穿过每个数据点，恰是过拟合的典型。
- **提前停止代替后剪枝**：XOR 的两个属性单独都无信息，提前停止会错过组合后高信息量的分裂；"先生成后剪枝"能正确处理。
- **固定学习率的感知机规则用于不可分数据**：永远震荡不收敛；学习率需按 O(1/t) 衰减。
- **高维中用 1-近邻/不归一化距离**：维度单位不统一会改变最近邻；k 过小过拟合、过大欠拟合，用交叉验证定 k。
- **在不平衡类上报告准确率**：全判多数类即得 99.99%；应重采样、加权损失或换 ROC/AUC 度量。
- **小数据上未做平滑的最大似然**（见第20章）：未出现事件的概率被置 0。
- **跳过上线前的工程化测试**（图19-28，Breck et al. 2016 的 ML 测试准则，共 4 类各 7 条）：特征与数据测试（特征被模式捕获、成本可控、隐私控制、新特征快速引入）；模型部署测试（模型规范入库存审、离线与线上指标关联、超参数经调优、关键数据切片表现足够好）；基础架构测试（训练可复现、全流程集成测试、上线前灰度测试、可快速安全回滚版本）；监控测试（依赖变更触发通知、训练/服务输入一致、模型不过陈旧、数值稳定、预测质量不倒退）

## 算法要点
**决策树归纳（图19-5，Learn-Decision-Tree）**
```
function Learn-Decision-Tree(examples, attributes, parent_examples) returns 一棵树
  if examples 为空: return Plurality-Value(parent_examples)
  else if 所有 examples 有相同的分类: return 该分类
  else if attributes 为空: return Plurality-Value(examples)
  else
    A ← argmax_{a∈attributes} Importance(a, examples)
    tree ← 以测试 A 为根的新决策树
    for each A 中的值 v:
      exs ← {e : e∈examples 且 e.A = v}
      subtree ← Learn-Decision-Tree(exs, attributes−A, examples)
      将带标签 (A=v) 和子树 subtree 的分支加入 tree
    return tree
剪枝：对仅挂叶的测试节点算总偏差 Δ = Σ_{k=1..d} [(p_k−p̂_k)²/p̂_k + (n_k−n̂_k)²/n̂_k]，其中 p̂_k=p·(p_k+n_k)/(p+n)、n̂_k=n·(p_k+n_k)/(p+n) 为零假设下的期望值；自由度 ν=d−1，若 Δ 低于 χ²_ν 阈值（如 d=4 即 ν=3、5% 置信水平下 Δ≥7.82 才拒绝"不相关"零假设；1% 水平下为 11.35）则删测试换叶
```

**模型选择与 k 折交叉验证（图19-8，Model-Selection）**
```
function Model-Selection(Learner, examples, k) returns (假设, 错误率)对
  err ← 以 size 为索引、存储验证集错误率的数组
  training_set, test_set ← examples 划分成两个集合
  for size = 1 to ∞:
    err[size] ← Cross-Validation(Learner, size, training_set, k)
    if err 开始显著增长:
      best_size ← 使 err[size] 最小的 size 值
      h ← Learner(best_size, training_set)
      return h, Error-Rate(h, test_set)

function Cross-Validation(Learner, size, examples, k) returns 错误率
  errs ← 0; N ← examples 的个数
  for i = 1 to k:
    validation_set ← examples[(i−1)×N/k : i×N/k]
    h ← Learner(size, examples − validation_set)
    errs ← errs + Error-Rate(h, validation_set)
  return errs / k            // 验证集平均错误率
```

**决策列表学习（图19-11，Decision-List-Learning）**
```
function Decision-List-Learning(examples) returns 决策列表或 failure
  if examples 为空: return 平凡的决策列表 No
  t ← 与 examples 的某个非空子集 examples_t 对应的测试，使 examples_t 全为正样例或全为负样例
  if 不存在这样的 t: return failure
  if examples_t 中样例均为正: o ← Yes else o ← No
  return 以 t 为初始测试、o 为输出、剩余测试由 Decision-List-Learning(examples − examples_t) 给出的决策列表
```

**梯度下降（单变量线性回归，L2 损失）**
```
w ← 权重空间任一点
repeat:  w_i ← w_i + α · Σ_j (y_j − h_w(x_j)) · x_j,i     # 批梯度下降，凸，收敛到全局最小
SGD: 每步随机取大小为 m 的小批量（N=10000、m=100 时计算量降为 1/100，梯度标准误差仅 ×10，整体仍快约 10 倍）；α 按 O(1/t) 衰减保证收敛
```

**感知机 / 逻辑斯谛回归更新**
```
感知机:  w_i ← w_i + α · (y − h_w(x)) · x_i        # 仅错分样例改变权重；线性可分才收敛
逻辑斯谛: h_w(x)=Logistic(w·x), Logistic(z)=1/(1+e^{−z}), g'=g(1−g)
         w ← w + α · (y − g(w·x)) · g'(w·x)·x     # 沿 y−预测 差方向；可微、噪声数据上稳定
```

**SVM 对偶与核技巧（式19-10、19-11）**
```
分离器: {x : w·x + b = 0}
对偶:  max_α Σ_j α_j − ½ Σ_{j,k} α_j α_k y_j y_k (x_j·x_k)   s.t. α_j ≥ 0, Σ_j α_j y_j = 0   # 凸二次规划
分离假设: h(x) = sign(Σ_j α_j y_j (x·x_j) − b)；最优权重 w = Σ_j α_j y_j x_j；非支持向量 α_j = 0
核技巧: 把每个 x_j·x_k 替换为 K(x_j,x_k)（如 (x_j·x_k)²、高斯核）；数据只以点积出现
       例: 映射 f₁=x₁², f₂=x₂², f₃=√2·x₁x₂ 满足 F(x_j)·F(x_k) = (x_j·x_k)²
软边距: 允许样例落在边界错误一侧，按移回正确一侧所需距离比例受罚
```

**AdaBoost（图19-25，自适应提升）**
```
function AdaBoost(examples, L, K) returns 一个假设
  w ← N 维样例权重向量，初始全 1/N；h、z ← K 维假设/权重向量；ε ← 小正数
  for k = 1 to K:
    h[k] ← L(examples, w)
    error ← Σ_{j: h[k](x_j)≠y_j} w[j]        // h[k] 的加权总错误率
    if error > 1/2: break from loop
    error ← min(error, 1−ε)
    for j = 1 to N:                           // 给错分样例更大权重
      if h[k](x_j) = y_j: w[j] ← w[j] · error/(1−error)
    w ← Normalize(w)
    z[k] ← ½ log((1−error)/error)             // 越准的假设投票权重越大
  return Function(x): Σ_k z_k·h_k(x)          // 加权多数（Weighted-Majority）
```

**非参数：kNN / 局部加权回归（回归+分类通用骨架）**
```
预测 x_q: 找 NN(k, x_q)；分类取多数票（k 取奇数），回归取均值/中位数/局部线性拟合
局部加权回归: 对每个查询点解 加权最小二乘 Σ_j K(Distance(x_q,x_j))·(y_j−w·x_j)²，书中核函数 K(d)=max(0, 1−(2|d|/w)²)（w 为核宽度，由交叉验证定）
加速: k-d 树（样例数≫维数）或局部敏感哈希 LSH（多随机投影+哈希桶并集，近似近邻）
```

## 参考表
损失函数速查：L1=|y−ŷ|（拉普拉斯噪声下最大似然、对离群值稳健）；L2=(y−ŷ)²（高斯噪声下最大似然、回归默认）；L0/1（最小化错误率、不可导）。正则化：L1=Σ|w_i|（稀疏解、自动特征选择，所需样例对不相关特征数是对数级）；L2=Σw_i²（平滑、旋转不变，样例需求对不相关特征数是线性级）。

集成方法对比：

| 方法 | 降偏差/方差 | 基模型训练数据 | 并行性 | 关键机制 |
|---|---|---|---|---|
| Bagging | 方差 | 自助重采样 | 可并行 | 多数投票/平均 |
| 随机森林 | 方差 | 重采样+随机属性子集（每分裂点分类取 √n、回归取 n/3 个） | 可并行 | 不剪枝；袋外误差评估；加树不增验证误差 |
| Stacking | 偏差 | 同一数据、不同模型类 | 可并行 | 用验证集训练元学习器组合基模型预测 |
| AdaBoost | 偏差 | 迭代重加权 | 序贯 | 错分样例加权+加权投票；弱学习器→训练集完美拟合 |
| 梯度提升 | 偏差 | 全数据残差/梯度 | 半并行 | 新树拟合损失负梯度；学习率+树大小正则化（XGBoost） |

## 完整示例
**餐厅等待问题**（图19-2）：预测布尔输出 WillWait，输入 10 个离散属性。训练集仅 12 个样例，而属性组合共 2^6·3^2·4·4=9216 种——归纳的本质是用 12 个已知输出对其余 9204 种组合做最佳猜测。

| 样例 | Alt | Bar | Fri/Sat | Hun | Pat | Price | Rain | Res | Type | Wait | WillWait |
|---|---|---|---|---|---|---|---|---|---|---|---|
| x1 | Yes | No | No | Yes | Some | $$$ | No | Yes | French | 0~10 | Yes |
| x2 | Yes | No | No | Yes | Full | $ | No | No | Thai | 30~60 | No |
| x3 | No | Yes | No | No | Some | $ | No | No | Burger | 0~10 | Yes |
| x4 | Yes | No | Yes | Yes | Full | $ | Yes | No | Thai | 10~30 | Yes |
| x5 | Yes | No | Yes | No | Full | $$$ | No | Yes | French | >60 | No |
| x6 | No | Yes | No | Yes | Some | $$ | Yes | Yes | Italian | 0~10 | Yes |
| x7 | No | Yes | No | No | None | $ | Yes | No | Burger | 0~10 | No |
| x8 | No | No | No | Yes | Some | $$ | Yes | Yes | Thai | 0~10 | Yes |
| x9 | No | Yes | Yes | No | Full | $ | Yes | No | Burger | >60 | No |
| x10 | Yes | Yes | Yes | Yes | Full | $$$ | No | Yes | Italian | 10~30 | No |
| x11 | No | No | No | No | None | $ | No | No | Thai | 0~10 | No |
| x12 | Yes | Yes | Yes | Yes | Full | $ | No | No | Burger | 30~60 | Yes |

根处 p=n=6，熵 B(0.5)=1 位。两个候选分裂的信息增益（图19-4）：
- Gain(Patrons) = 1 − [2/12·B(0/2) + 4/12·B(4/4) + 6/12·B(2/6)] ≈ **0.541 位**（全属性最大）
- Gain(Type) = 1 − [2/12·B(1/2) + 2/12·B(1/2) + 4/12·B(2/4) + 4/12·B(2/4)] = **0 位**（每个子集仍正负各半）

故 Patrons 被选为根。学出的树（图19-6，与作者手绘的图19-3 不同但更简单，省去 Raining、Reservation）：

```
Patrons? ─ None→No；Some→Yes；Full→
  Hungry? ─ No→No；Yes→
    Type? ─ French→Yes；Italian→No；Burger→Yes；Thai→
      Fri/Sat? ─ No→No；Yes→Yes
```

该树发现隐藏模式"周末会在泰国餐厅等位"。对应的决策列表（图19-10）：Patrons(x,Some)→Yes；否则 Patrons(x,Full)∧Fri/Sat(x)→Yes；否则 No——比树更紧凑但表达能力受限。学习曲线（图19-7）：100 个样例随机划分训练/测试、20 次重复取均值，准确率随训练集增大单调升至约 95%（"快乐图"）。同一数据上：决策列表稍逊于决策树（图19-12）；朴素贝叶斯（第20章）更低——真实函数是决策树、不满足条件独立；未提升的决策树桩仅 81%，AdaBoost(K=5) 提升到 93%，K=20 时训练误差归零（插值点）而测试准确率仍继续升至 0.98（K=137 达峰，后渐降至 0.95）。图19-24 展示提升的权重演化：初始 4 个样例等权重，每轮被错分样例的权重增高（矩形变高），下一轮树桩被迫专攻它们，最终集成为加权多数、越准的树桩投票权重越大。

**伯克利房价线性回归**（图19-13）：最小化平方误差和得唯一闭式解（式19-3）w₁=[NΣxy−ΣxΣy]/[NΣx²−(Σx)²]=0.232、w₀=(Σy−w₁Σx)/N=246；损失函数关于 (w₀,w₁) 的三维图是凸的，只有全局最小值。

## 要点
1. 模型选择与模型优化是两个任务：先选假设空间（超参数），再在其中训练；验证集管前者，测试集只在最后用一次。
2. 偏差-方差权衡无处不在：线性欠拟合、高次多项式过拟合，正则化和剪枝都是在两者间找平衡点。
3. 完全数据+表格 CPT 的最大似然就是条件频率，各参数可独立估计（第20章严格化）；多变量线性回归有闭式解（正规方程）。
4. 核技巧的本质：凡是只以点积形式使用数据的算法都可以"核化"，在极高维空间中高效学习非线性边界。
5. 集成的数学：5 个 75% 正确且独立的分类器多数投票正确率约 89%，17 个约 99%——前提是错误不相关，故随机森林靠随机化属性去相关。
6. 插值临界点之后验证误差仍可下降（boosting、随机森林、深度网络）：奥卡姆剃刀不是普适定律，"合适"比"简单"更重要。
7. 在线场景放弃 i.i.d. 假设也能学：随机加权多数算法对任意样例序列的遗憾有上界，可做到无悔。

## 关联
- **第20章**：本章的正则化=贝叶斯先验，L2 损失=固定方差高斯噪声的最大似然，MDL=MAP 学习的位数表述；朴素贝叶斯参数学习、EM（隐变量）在那里展开。
- **第21章**：感知机是神经网络的原型；SGD/小批量、学习率、验证误差二次下降直接延续到深度学习；迁移学习缓解"从零学习"的假设。
- **第13章**：贝叶斯网络给出朴素贝叶斯分类器的结构；SVM/逻辑斯谛回归作为判别模型与之对照。
- **第5章/第4章**：贝叶斯超参优化的探索-利用权衡同蒙特卡罗树搜索的上置信界；梯度下降是爬山法在连续空间的对应。
- **外部概念**：scikit-learn 工作流（train/test split、GridSearchCV、学习曲线、ROC-AUC、混淆矩阵）、统计学习理论（VC 维、结构风险最小化）、XGBoost/LightGBM 与随机森林在表格数据上的主导地位、LIME/可解释 AI。
