# lecture-report-skill

> 一个 [Claude Code](https://claude.com/claude-code) skill：把课程讲义 / slides / 教材章节 / PDF / 笔记，整理成一套**比原资料更详细、能脱离原资料独立学懂**的中文 HTML 学习报告。

把一堆资料丢给 Claude，它会**按章并行**地精读整理，产出一个带导航首页的 HTML 报告文件夹——每个定义讲透直觉、每个定理给完整逐步证明、每道例题/习题给逐步解答与「为什么这样做」，全程 MathJax 渲染公式，配统一的彩色盒子 + 步骤圆圈 + 速查表样式。

## 效果预览

`examples/probability-statistics/` 是用本 skill 整理的一份北大《概率统计》第 6–9 章学习报告（数理统计部分：抽样分布、参数估计、假设检验、方差分析与回归），含 1 个导航首页 + 4 章独立页面，**62 道例题全部逐步求解**。

把 `examples/probability-statistics/` 文件夹用浏览器打开 `index.html` 即可查看。

## 它做什么

- **比原资料更详细**：目标是「读者只看报告就能从头到尾顺畅学懂」，不是照抄/缩写。
- **定义**：精确陈述 + 大白话直觉 + 为什么这样定义 + 与已学概念的联系。
- **定理**：完整陈述 + **完整逐步证明**（补全原资料跳过的步骤，冗长细节折叠收纳）。
- **例题/习题**：一道不漏，完整题面 + 每一步代数 + 「为什么这么做」 + 结论；多解法都做完。
- **统一排版**：彩色盒子（定义/定理/例题/证明/注/直觉/易错）、步骤圆圈、关键结论条、折叠推导、数据速查表、粘性侧边栏导航（滚动高亮）、响应式、回到顶部。
- **自包含**：除 CDN 的 MathJax 外只依赖本地 `style.css` / `script.js`，可直接静态托管。
- 发现原资料的笔误/算错会**纠正并注明**，不默默照抄。

## 往年真题整理

除了把讲义整理成知识报告，还能**把历年试卷按章节归类、逐题详解**。试卷的特点是一份卷子跨多个章节，所以会先**拆题 + 判定每题属于哪一章**，再逐章汇编：

- 产出 `exam-chapN.html`（每章真题精解）+ `exam-index.html`（真题总览）；**可单独运行**——手里只有试卷、没有讲义也行。
- 每题含：**出处/题型/考点/难度徽章** + 考点定位 + 逐步解析（默认折叠在「先做后看」解答框里，方便自测）+ 最终答案高亮 + 易错点 + 举一反三。
- 总览页有**年份 × 章节分布矩阵**和高频考点统计，一眼看清复习重点。
- **试卷带答案**则核对扩充、纠正错误；**不带答案**则从零推导并自查。

## 安装

把本仓库内容放进 Claude Code 的 skills 目录：

```bash
git clone https://github.com/a1henu/lecture-report-skill.git
mkdir -p ~/.claude/skills/lecture-report
cp -r lecture-report-skill/{SKILL.md,reference,templates} ~/.claude/skills/lecture-report/
```

> `examples/` 不需要拷进 skill 目录，仅作效果演示。

## 使用

在 Claude Code 里：

```
/lecture-report
```

然后把资料（PDF / slides / 讲义路径）给它，或直接说：

> 把这几份讲义整理成学习报告

Claude 会按以下流水线工作：

1. **收集 & 通读**资料，搭好章节骨架、理清逻辑主线；
2. 新建 `report/` 文件夹，拷入共享 `style.css` / `script.js`；
3. **一章一个 subagent 并行**精读整理，各写一份 `chapN.html`；
4. 校验（锚点完整性 / 标签闭合 / 例题数）、补漏；
5. 生成导航首页 `index.html`，串联全部章节。

产出的 `report/` 文件夹是自包含的，可直接用任意静态托管（如 GitHub Pages）发布。

## 仓库结构

```
lecture-report-skill/
├── SKILL.md                      # 主文件：流程 + 硬性质量约束 + 排错
├── reference/
│   ├── subagent-prompt.md        # 知识报告·每章 subagent 的 prompt 模板
│   └── exam-subagent-prompt.md   # 往年真题·每章 subagent 的 prompt 模板
├── templates/
│   ├── style.css                 # 共享样式（彩色盒子 / 步骤 / 速查表 / 真题组件）
│   ├── script.js                 # 侧边栏滚动高亮
│   ├── chapter.html              # 知识报告·单章页骨架
│   ├── index.html                # 知识报告·导航首页骨架
│   ├── exam-chapter.html         # 往年真题·单章精解页骨架
│   └── exam-index.html           # 往年真题·总览页骨架
└── examples/
    └── probability-statistics/   # 示例：概率统计第 6–9 章学习报告
```

## 自定义

- **主题色**：默认北大红 `#94070a`，改 `templates/style.css` 顶部 `:root` 里的 `--pku-red` 等变量即可换配色。
- **触发词 / 默认文件夹名**：改 `SKILL.md` 的 `description` 与流程描述。

## License

[MIT](./LICENSE)
