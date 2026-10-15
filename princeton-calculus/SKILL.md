---
name: princeton-calculus
description: "Knowledge base from \"普林斯顿微积分读本（修订版）\" (The Calculus Lifesaver) by Adrian Banner. Use when studying or solving single-variable calculus problems: limits, derivatives, integrals, series, Taylor expansions, and applications (极限、导数、积分、级数、泰勒展开及其应用)."
---

<!-- argument-hint: [topic, method name, or chapter number] -->

# 普林斯顿微积分读本 (The Calculus Lifesaver)
**作者**: Adrian Banner（杨爽、赵晓婷、高璞 译，人民邮电出版社第2版） | **页数**: ~648 | **章节**: 30章 + 2附录 | **生成日期**: 2026-10-15

## 如何使用本技能

- **不带参数** — 加载下方核心框架作为参考
- **按主题提问** — 问"换元法"、"洛必达"、"收敛半径"等，我会找到并读取相关章节
- **按章节** — 直接问 `ch14` 或"第14章"，我加载该章
- **浏览** — 问"有哪些章节？"查看完整索引
- **速查** — 解题决策流程看 [cheatsheet.md](cheatsheet.md)；方法库看 [patterns.md](patterns.md)；术语看 [glossary.md](glossary.md)

当你问到核心框架未覆盖的主题时，我会先读对应章节文件再回答。

---

## 核心框架与心智模型

### 极限
- **先代入，再分类**：任何极限第一步是直接代入；得 b/0 是渐近线情形，得不定式才进入方法选择。——第3、14章
- **首项主导法**：x→±∞ 时多项式型函数只由最高次项决定；次数相等→系数比。——第4章
- **三明治定理**：振荡项（sin、cos 乘任何趋零量）用 −1≤sin≤1 构造夹逼。——第3章
- **洛必达归化思想**：所有不定式（B1差、B2积、C幂指）都化归为类型A（分式）再分别求导；对数永远留分子。——第14章
- **重要极限三件套**：sin(x)/x→1、(e^x−1)/x→1、(1+1/x)^x→e（x→0/∞）。——第7、9章
- 用 **ε-δ 正式定义** 严格化极限（挑战-回应游戏：给定 ε 找 δ）。——附录A

### 微分
- **导数即瞬时变化率/切线斜率**：f′(x)=lim_{h→0}[f(x+h)−f(x)]/h；速度是位移的导数，加速度是速度的导数。——第5、6章
- **三大求导法则**：乘积法则、商法则、链式法则（dy/dx = dy/du·du/dx）——识别复合结构是核心技能。——第6章
- **可导 ⟹ 连续，反之不然**；导数不存在于角点、尖点、间断点。——第5章
- **符号表格**：用 f′、f″ 的零点分区标正负 → 单调性、极值、凹凸、拐点一网打尽，是画图的系统方法。——第11、12章
- **f′(c)=0 ⇏ 极值；f″(c)=0 ⇏ 拐点**；二阶导判别法在 f″(c)=0 时失效，退回一阶导。——第11章
- **相关变化率头号规则：先求导，后代入**瞬时数值。——第8章
- **最优化别忘端点**：全局最值在临界点与定义域端点中比较产生。——第13章
- **线性化 = 一阶泰勒**：L(x)=f(a)+f′(a)(x−a)，误差 ≈ ½f″(c)(x−a)²。——第13、24章

### 积分
- **FTC 双向桥**：积分上限函数求导回到被积函数；定积分 = 反导数两端差 F(b)−F(a)。——第17章
- **不定积分必写 +C**；面积与有向面积不同——面积要把 x 轴下方部分取绝对值。——第16、17章
- **换元法**：找"函数及其导数"结构；定积分换元必换上下限。——第18章
- **分部积分**：∫u dv = uv − ∫v du；u 选求导变简单的（ln、反三角优先）。——第18章
- **三角换元三型对照**：√(a²−x²)→sin、√(a²+x²)→tan、√(x²−a²)→sec，配三角形回代。——第19章

### 反常积分与级数（统一类比）
- **瑕点拆分原则**：每个瑕点（±∞、分母零点）单独成段且位于端点；一段发散全发散。区间内部瑕点最易漏。——第20、21章
- **p 判别法双版本方向相反**：∞ 处 p>1 收敛；0 处 p<1 收敛。——第20章
- **增长阶梯**：ln 幂 ≪ 多项式 ≪ 指数 ≪ 阶乘——极限比较判别法的取主部依据。——第21、23章
- **第 n 项判别法是单向的**：lim aₙ≠0 → 发散；lim aₙ=0 什么都不能说（调和级数是反例）。——第22章
- **判别法选型直觉**：阶乘/指数→比式；n 次幂→根式；像 1/nᵖ→比较+p；递减正函数→积分判别；交错→交错级数判别+绝对收敛检验。——第23章
- **幂级数端点必须逐个单查**：收敛半径只告诉你内部绝对收敛。——第26章

### 泰勒级数
- **泰勒定理**：f(x)=P_N(x)+R_N(x)，|R_N|≤M|x−a|ᴺ⁺¹/(N+1)!——估算问题的误差控制核心。——第24、25章
- **五个麦克劳林级数是母机**（e^x、sin、cos、1/(1−x)、ln(1+x)）：代换、求导、积分、乘除可合成新级数，比硬算高阶导快。——第26章
- **泰勒级数收敛 ⇏ 收敛到 f(x)**：必须验证 R_N→0（反例 e^{−1/x²}）。——第24章

### 应用
- **体积选型**：切片⊥轴用圆盘法 ∫πr²，柱壳∥轴用壳法 ∫2πrh；一个积分变量到底。——第29章
- **微分方程选型**：能分离先分离 → 线性一阶用积分因子 → 常系数用特征方程；特解与齐次解冲突乘 x。——第30章
- **复数极式**：e^{iθ}=cos θ+i sin θ，解 zⁿ=w 得 n 个均布在单位圆缩放上的根。——第28章

---

## 章节索引

| # | 标题 | 核心方法 |
|---|------|----------|
| [ch01](chapters/ch01-functions-graphs-lines.md) | 函数、图像和直线 | 垂线/水平线检验、定义域值域、反函数、复合、奇偶性 |
| [ch02](chapters/ch02-trig-review.md) | 三角学回顾 | ASTC 方法、扩展定义域、三角恒等式 |
| [ch03](chapters/ch03-intro-to-limits.md) | 极限导论 | 左右极限、∞处极限、渐近线、三明治定理 |
| [ch04](chapters/ch04-polynomial-limits.md) | 求解多项式的极限问题 | 首项主导法、因式分解、有理化、绝对值极限 |
| [ch05](chapters/ch05-continuity-differentiability.md) | 连续性和可导性 | 介值定理、最大最小值定理、导数定义 |
| [ch06](chapters/ch06-differentiation-rules.md) | 求解微分问题 | 乘积/商/链式法则、切线方程、速度加速度、分段函数求导 |
| [ch07](chapters/ch07-trig-limits-derivatives.md) | 三角函数的极限和导数 | sin(x)/x 极限、小数/大数情况、三角导数 |
| [ch08](chapters/ch08-implicit-related-rates.md) | 隐函数求导和相关变化率 | 隐函数求导（含二阶）、相关变化率四步法 |
| [ch09](chapters/ch09-exponentials-logarithms.md) | 指数函数和对数函数 | e 的定义、对数法则、取对数求导法、指数增长衰变、双曲函数 |
| [ch10](chapters/ch10-inverse-trig.md) | 反函数和反三角函数 | 反函数求导、六个反三角函数定义域值域与导数 |
| [ch11](chapters/ch11-derivatives-and-graphs.md) | 导数和图像 | 极值定理、罗尔定理、中值定理、一阶/二阶导数判别法 |
| [ch12](chapters/ch12-sketching-graphs.md) | 绘制函数图像 | 符号表格、绘图完整十一步法 |
| [ch13](chapters/ch13-optimization-linearization.md) | 最优化和线性化 | 最优化一般方法、线性化与微分、误差、牛顿法 |
| [ch14](chapters/ch14-lhopital.md) | 洛必达法则及极限问题总结 | 类型A/B1/B2/C 归化、极限方法总决策 |
| [ch15](chapters/ch15-integration-intro.md) | 积分 | 求和符号、伸缩求和、有向面积、位移与面积 |
| [ch16](chapters/ch16-definite-integrals.md) | 定积分 | 黎曼和定义、定积分性质、两曲线间面积、平均值与中值定理 |
| [ch17](chapters/ch17-ftc.md) | 微积分基本定理 | FTC1/FTC2、四种变形、不定积分 |
| [ch18](chapters/ch18-integration-techniques-1.md) | 积分的方法 I | 换元法、分部积分、部分分式 |
| [ch19](chapters/ch19-integration-techniques-2.md) | 积分的方法 II | 三角幂次策略、约化公式、三角换元三型 |
| [ch20](chapters/ch20-improper-integrals-basics.md) | 反常积分：基本概念 | 收敛发散、比较/极限比较/p/绝对收敛判别法 |
| [ch21](chapters/ch21-improper-integrals-solving.md) | 反常积分：如何解题 | 瑕点拆分、函数在 ∞/0 附近的渐近行为 |
| [ch22](chapters/ch22-sequences-series-basics.md) | 数列和级数：基本概念 | 第n项/比式/根式/积分/交错级数判别法 |
| [ch23](chapters/ch23-solving-series.md) | 求解级数问题 | 几何级数求值、判别法选择流程、负项级数 |
| [ch24](chapters/ch24-taylor-intro.md) | 泰勒多项式、泰勒级数和幂级数导论 | 泰勒定理与误差项、幂级数收敛性 |
| [ch25](chapters/ch25-estimation-problems.md) | 求解估算问题 | 误差项估算一般方法、五个精讲例子 |
| [ch26](chapters/ch26-taylor-power-series-solving.md) | 泰勒级数和幂级数：如何解题 | 收敛半径与区域、级数合成六操作、级数求极限 |
| [ch27](chapters/ch27-parametric-polar.md) | 参数方程和极坐标 | 参数求导、坐标互换、极坐标切线与面积 |
| [ch28](chapters/ch28-complex-numbers.md) | 复数 | 复平面、棣莫弗、zⁿ=w、欧拉恒等式 |
| [ch29](chapters/ch29-volumes-arc-length.md) | 体积、弧长和表面积 | 圆盘法、壳法、弧长、旋转体表面积 |
| [ch30](chapters/ch30-differential-equations.md) | 微分方程 | 分离变量、积分因子、特征方程、IVP、建模 |
| [ch31](chapters/ch31-appendix-a-limit-proofs.md) | 附录A：极限及其证明 | ε-δ 定义、极限运算法则证明、MVT 证明 |
| [ch32](chapters/ch32-appendix-b-estimating-integrals.md) | 附录B：估算积分 | 梯形法则、辛普森法则、误差界 |

## 主题索引

- **ASTC 方法** → ch02
- **ε-δ 定义** → ch31
- **p 判别法** → ch20, ch22, ch23
- **凹性/拐点** → ch11, ch12
- **比式/根式判别法** → ch22, ch23
- **分部积分** → ch18
- **部分分式** → ch18
- **参数方程** → ch27, ch29
- **Chain rule 链式法则** → ch06, ch08
- **导数定义** → ch05, ch06, ch17（变形4）
- **对数法则/求导** → ch09
- **反函数/反三角** → ch01, ch10
- **反常积分** → ch20, ch21
- **复合函数** → ch01, ch06
- **复利与 e** → ch09
- **符号表格/绘图** → ch12
- **极值/最优化** → ch11, ch13
- **极坐标** → ch27
- **介值定理** → ch05, ch31
- **近似与误差** → ch13, ch24, ch25, ch32
- **绝对收敛** → ch20, ch22
- **壳法/圆盘法** → ch29
- **L'Hôpital 洛必达** → ch14
- **临界点和分类** → ch11
- **洛朗/幂级数收敛半径** → ch26
- **连续性** → ch05, ch31
- **麦克劳林级数** → ch24, ch25, ch26
- **牛顿法** → ch13
- **平均值/积分中值定理** → ch16
- **三明治定理** → ch03, ch07
- **三角恒等式** → ch02, ch19
- **三角换元** → ch19
- **数列极限** → ch22
- **Taylor 泰勒定理/级数** → ch24, ch25, ch26, ch28
- **体积/弧长/表面积** → ch29
- **微分方程** → ch30
- **微积分基本定理 FTC** → ch17
- **线性化/微分** → ch13, ch24
- **相关变化率** → ch08
- **斜率/切线** → ch05, ch06, ch27
- **隐函数求导** → ch08, ch13
- **有向面积/位移** → ch15, ch16, ch17
- **增长速度比较** → ch09, ch21, ch23
- **指数增长与衰变** → ch09, ch30
- **中值定理/罗尔定理** → ch11, ch31
- **重要极限** → ch07, ch09
- **复数/欧拉公式** → ch28

## 支撑文件

- [glossary.md](glossary.md) — 关键术语及定义（按拼音排序）
- [patterns.md](patterns.md) — 全部解题方法与技巧（何时用/怎么做/注意）
- [cheatsheet.md](cheatsheet.md) — 决策规则与速查表

---

## 范围与限制

本技能仅覆盖该书内容。用于实际解题时结合具体题目要求；超出本书的主题（多变量微积分、实分析严格化等）请查阅相关技能或直接询问。
