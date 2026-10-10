# ai-a-modern-approach

Agent skill generated from **《人工智能：现代方法（第4版）》** (*Artificial Intelligence: A Modern Approach*, 4th ed.) by Stuart Russell & Peter Norvig（中文版上下册，28 章）, converted with [book-to-skill](https://github.com/virgiliojr94/book-to-skill).

## 这是什么

把 AIMA 4e 全书提炼成可按需加载的智能体技能：核心框架速览、28 个章节研读摘要、术语表、技术模式库与算法选择速查。章节摘要覆盖：搜索（A\*、对抗搜索、CSP）、逻辑与知识表示、概率推理（贝叶斯网络、HMM）、决策论（MDP、POMDP、博弈论）、机器学习、深度学习、强化学习、NLP、计算机视觉、机器人学、AI 伦理与价值对齐。

> 内容为综合提炼的研读摘要，**不是**原书文本。原书插图与伪代码框未参与提取，各章伪代码由原书散文描述忠实重构。

## 文件清单

| 文件 | 内容 |
|---|---|
| `SKILL.md` | 核心框架（8 大主线）+ 章节索引 + 主题索引 |
| `chapters/ch01–ch28` | 28 个章节研读摘要（按需加载，每章约 3–4K tokens） |
| `glossary.md` | 66 个关键术语，中英对照 |
| `patterns.md` | 18 个可复用技术模式（何时用/怎么做/权衡） |
| `cheatsheet.md` | 算法选择决策表 + 常见坑速查 |

## 安装（任何支持 Agent Skills 的宿主）

```bash
npx skills add https://github.com/DarkModest/ai-a-modern-approach --skill ai-a-modern-approach
```

或直接克隆到你的技能目录（如 `~/.agents/skills/`）。

## 版权说明

本仓库为**私有**学习用途：技能内容派生自第三方版权书籍（《人工智能：现代方法（第4版）》，人民邮电出版社中文版 / Pearson 英文版），请勿公开传播仓库内容。
