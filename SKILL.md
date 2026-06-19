---
name: lecture-report
description: 把课程讲义 / slides / 教材章节 / PDF / 笔记 / 往年试卷等学习资料，整理成一套「比原资料更详细、能脱离原资料独立学懂」的中文 HTML 学习报告 —— 生成一个 report 文件夹，含导航 index.html 和每章一份独立 HTML。每个定义讲透直觉、每个定理给完整逐步证明、每道例题/习题给逐步解答与「为什么这样做」，全程 MathJax 渲染公式，统一的彩色盒子 + 步骤圆圈 + 速查表样式。**还能整理往年真题**：读入历年试卷，把每道题按章节归类，逐章生成「往年真题精解」页（含出处/考点/难度徽章、逐步解析、易错点、举一反三、年份×章节分布矩阵），有答案则核对扩充、无答案则从零推导。用 subagent 按章并行整理。当用户说「把这份资料/讲义/slides 整理成学习笔记 / report / 学习报告」或「整理往年题 / 真题 / 按章节归类历年考题」时调用。
---

# lecture-report — 把学习资料整理成 HTML 学习报告

用户的工作流：给一份或多份学习资料（讲义 PDF、slides、教材章节、扫描件、笔记……），希望得到一套**排版精良、比原资料更详尽、能独立学懂**的 HTML 学习报告。本 skill 把「拆解 → 并行精读整理 → 汇总导航」固化成一条流水线。

## 北极星目标

> 读者**只看报告就能从头到尾顺畅学懂全部内容**，只在自己想深挖某点时才回去翻原资料。

每一个设计都服务于此：先直觉后公式、每个符号都解释、每道例题不跳步、关键证明写全、长推导折叠收纳、易错点显式提醒。**报告必须比原资料更详细**，绝不是照抄/缩写。

## 输出结构

在当前工作目录（或用户指定处）新建一个文件夹（默认名 `report/`）：

```
report/
├── index.html       # 纯导航首页：课程简介 + 学习主线 + 每章卡片 + 使用说明
├── chap1.html       # 每章/每部分一份独立、自包含的报告
├── chap2.html
├── …
├── style.css        # 共享样式（从本 skill templates/ 复制，勿改）
└── script.js        # 侧边栏滚动高亮 + 回到顶部（从 templates/ 复制）
```

- 文件名用英文/数字（`chap6.html`、`index.html`）；正文一律中文。
- 章节怎么切分：**有多个文件就一文件一章**；单个大文件按其内在章节切分；用户另有指定则听用户。

## 执行流程（七步）

### 1. 收集与理解资料
找到所有源文件（`ls`、`find`）。PDF 先看页数：
```bash
mdls -name kMDItemNumberOfPages -raw file.pdf    # macOS 取页数
```

### 2. 亲自通读，搭好骨架（关键，别跳过）
**主 agent 先自己读一遍所有资料**（PDF 用 Read 工具，每次 `pages` ≤20 页，分批读完）。目的：
- 确定每章标题、节结构、每个定义/定理/例题清单；
- 理清各章之间的逻辑主线（写 index 和给 subagent 派活都要用）；
- PDF 并行读取时图片可能错位/串章，**以标题页和清晰可辨的内容为准**核对每章归属，不要凭猜测。

> 如果资料体量极大、主 agent 通读成本太高，可只读各章标题页/目录页确定结构，把「通读+整理」整个交给 subagent；但只要预算允许，主 agent 至少粗读一遍能显著提升 index 质量和派活准确度。

### 3. 建 report 文件夹 + 拷入共享资源
```bash
mkdir -p report
cp ~/.claude/skills/lecture-report/templates/style.css report/style.css
cp ~/.claude/skills/lecture-report/templates/script.js  report/script.js
```
`style.css` / `script.js` 已调好，**直接用，不要重写**。需要微调配色（如非红色主题）再改 `:root` 变量。

### 4. 并行派发 subagent —— 一章一个
在**同一条消息**里并行调用多个 `Agent`（`subagent_type: general-purpose`，模型继承主会话；做数学/证明务必 Opus 级）。每个 subagent 的 prompt 用 `reference/subagent-prompt.md` 模板，替换占位符并填入第 2 步得到的**该章 outline**。各 subagent 读各自的源、写各自的 `chapN.html`，文件互不冲突，无需 worktree 隔离。

### 5. 兜底：没写出文件就补写
subagent 可能读完资料后**耗尽轮次还没写文件**（最终消息停在「Now I'll write…」）。第 6 步校验若发现某章缺失，**重新派一个 agent 把该章写出来**（让它重读源材料并写文件，或继续之前的 agent）。

### 6. 校验（每次必做）
```bash
cd report
for f in chap*.html; do
  echo -n "$f | $(wc -c <$f)B | </html>:$(grep -c '</html>' $f) | h2:$(grep -co '<h2' $f) | 例题:$(grep -co 'box example' $f) | "
  miss=0
  for h in $(grep -o 'href="#[^"]*"' $f | sed 's/href="#//;s/"//' | sort -u); do
    [ -z "$h" ] && continue; grep -q "id=\"$h\"" $f || { echo -n "缺id:$h "; miss=1; }
  done
  [ $miss = 0 ] && echo "锚点OK"
done
```
逐项确认：文件存在、以 `</html>` 结尾、导航锚点零缺失、引用了 `style.css`/`script.js`、有 `href="index.html"` 返回链接。

### 7. 建 index.html + 收尾
用 `templates/index.html` 骨架写首页：课程名、学习主线流程图（`.flow`）、每章一张 `.chap-card`（标题+简介+3~4 个要点）、使用说明、一句话主线 `.conclusion`。校验首页每个 `href="chapN.html"` 都指向真实文件，然后 `open report/index.html`。

完成后报告产物清单 + 总例题数，并**主动提示可用 `/view-html` 一键部署到公网**。

## 模板与组件清单

`templates/` 里有现成资源（拷贝 / 当骨架用）：
- `style.css` —— 共享样式：彩色盒子（定义蓝/定理紫/例题青/证明灰/注橙/直觉·易错红）、步骤圆圈 `ol.steps`、关键结论 `p.conclusion`、折叠推导 `details.derive`、数据表 `table.data`、粘性侧边栏、首页卡片、响应式、回到顶部；**以及真题专用组件**（真题盒子 `.box.exam`、出处/题型/考点/难度徽章 `.exam-badge`、可折叠解答 `details.solution`、答案高亮 `.final-answer`、分布矩阵 `table.matrix`）。北大红主题（改 `:root` 可换色）。
- `script.js` —— 侧边栏 scrollspy。
- `chapter.html` —— 知识报告·单章页骨架（含各类盒子用法示例）。
- `index.html` —— 知识报告·首页骨架。
- `exam-chapter.html` —— 往年真题·单章精解页骨架（逐题结构示例）。
- `exam-index.html` —— 往年真题·总览页骨架（分布矩阵 + 按章入口）。

`reference/` 里有两份 subagent prompt 模板：
- `subagent-prompt.md` —— 知识报告·每章 subagent 的完整 prompt（含质量要求与自检）。
- `exam-subagent-prompt.md` —— 往年真题·每章 subagent 的完整 prompt（含逐题结构与质量要求）。

## 硬性质量约束（写进每个 subagent，也是验收标准）

1. **比原资料更详细**，能脱离原资料独立学懂。
2. **定义** = 精确陈述 + 直觉 + 为什么 + 联系。
3. **定理** = 完整陈述 + **完整逐步证明**（补全跳过的步骤；冗长细节进 `<details>`）。
4. **每道例题/习题全覆盖，一道不漏** = 完整题面 + 每步代数 + 「为什么这么做」 + 结论；多解法都做完。
5. 全中文；公式一律 MathJax（`$…$` / `$$…$$`），绝不只用文字描述公式。
6. 用既定 CSS 组件，不自创样式；侧边栏 nav 与所有 `h2`/关键 `h3` 的 `id` 一一对应。
7. 每章含「本章导读」+「本章小结（含速查表）」。
8. **发现原资料错误就纠正并注明**，不默默照抄。
9. 自包含：除 CDN 的 MathJax 外，只依赖本地 `style.css`/`script.js`（这样能被 `/view-html` 直接部署）。

## 章末真题嵌入（知识 + 真题一体化）

当同时拥有讲义和往年试卷时，**每章知识报告末尾必须追加「本章对应往年真题」段落**。目标：让学生在学完知识点后立即用真题检验理解，并看到「这道题考的是本章哪个定理/方法」的显式对应。

### 嵌入位置与结构
在每章 `chapN.html` 的「本章小结」之后、文件结尾标签之前，插入：
```html
<hr class="sep">
<h2 id="exam-practice"><span class="secnum">★</span> 本章对应往年真题</h2>
<p class="muted">以下真题考查了本章知识点。建议先独立作答，再展开对照解析。</p>

<div class="box exam">
  <div class="box-title"><span class="tag">真题</span> 出处 · 题号</div>
  <div class="exam-meta">…</div>
  <div class="stem">完整题面（MathJax）</div>
  <details class="solution"><summary></summary><div>
    <p><strong>考点对应：</strong>本章第 X 节「xxx」中的定理/性质 Y。</p>
    <p><strong>解题方法：</strong>直接应用前文第 X 节的公式/步骤…</p>
    逐步解答…
    <div class="final-answer">最终答案</div>
  </div></details>
</div>
```
同时在侧边栏 `nav` 最后加 `<a href="#exam-practice">★ 往年真题</a>`。

### 嵌入质量约束
1. 每道题必须有**「考点对应」**（指明本章哪一节的哪个定理/公式）和**「解题方法」**（本章前文哪里教了这个方法）。
2. 让学生明确看到「学 → 考」的闭环。
3. 每章追加 3-8 道最具代表性的真题（全量在独立整卷页里）。

### 执行流程
知识报告各章 + 独立真题页都生成之后，分批派 subagent，每个负责读取试卷 + 章节文件，用 Edit 在末尾追加真题段落并更新侧边栏。

## 往年真题整理（独立整卷页）

除了章末嵌入，还需**按整套试卷组织的独立真题精解页**（`exam-mid.html`、`exam-final.html` 等），方便学生做整卷模拟练习。两种组织方式互补：
- **章末嵌入** → 学完即练，理解知识点如何出题
- **独立整卷页** → 模拟考试、把握时间、训练综合应用

当用户给的是**历年试卷**、要求「整理往年题 + 详细解析」时，走这条平行流程。它**可单独运行**（手里只有试卷、没有讲义也行），也可与上面的知识报告并存、互相链接。

**与知识报告的区别**：试卷一份卷子**跨多个章节**，所以核心多了一步「拆题 + 归类」——先把每道题判定到对应章节，再逐章汇编详解。

### 输出结构（在同一个 `report/` 文件夹内）
```
report/
├── exam-index.html      # 真题总览：年份×章节分布矩阵 + 高频考点 + 按章入口 + 备考建议
├── exam-chap6.html      # 第6章往年真题精解（按题型/考点分组，逐题详解）
├── exam-chap7.html
├── …
├── style.css            # 与知识报告共用（已含真题专用组件）
└── script.js
```
若同时也整理了讲义知识报告（`chap6.html`…），两边互相加跳转链接（真题页→知识报告、首页→真题总览）。

### 执行流程

1. **确定章节体系**：真题归类需要一份「章节清单」。优先用已整理的讲义结构；若只有试卷，则从课程大纲/教材目录推断，**章节划分不明确时先问用户**。

2. **拆题 + 归类（主 agent 亲自做，关键）**：完整读完所有试卷，把卷子拆成**单道题**，逐题记录：
   - 出处编号（年份 + 卷别 + 题号，如 `2023-期末-三`）、题型（选择/填空/计算/证明/应用）、完整题面、参考答案（若试卷带答案）、**归属章节 + 考点**。
   - 产出一张**归类清单/矩阵**（题 → 章节），并据此统计年份×章节分布、高频考点。一题跨多章时归主考点章、并在另一章备注。

3. **建文件夹 + 拷资源**：同知识报告流程（`mkdir -p report` + 拷 `style.css`/`script.js`）。

4. **并行派发 subagent —— 一章一个**：用 `reference/exam-subagent-prompt.md` 模板，把**该章的题目清单（含题面与已有答案）**填进去。各 subagent 写各自的 `exam-chapN.html`：逐题给「考点定位 + 逐步解析（折叠在 `<details class="solution">` 里便于自测）+ 最终答案 + 易错点 + 举一反三」。

5. **兜底补漏 + 校验**：同知识报告（文件存在、`</html>` 结尾、锚点零缺失、本章题目一道不漏）。校验时统计 `box exam` 数量与归类清单核对。

6. **建 `exam-index.html`**：用 `templates/exam-index.html` 骨架——年份×章节分布矩阵（`table.matrix`，命中格加 `class="hit"`）、高频考点、每章一张 `.chap-card.exam-card` 入口、备考建议。

### 真题专属质量约束（写进每个 exam-subagent）
- **题面原样转录**，不改数字不删条件；公式一律 MathJax。
- 解答放 `<details class="solution">`（默认折叠，先自测后对答案）；最终答案用 `.final-answer` 高亮。
- 每题先「**考点定位**」，把真题挂回知识体系；逐步解析不跳步，查表写明临界值。
- **有答案**：核对，错了就纠正并注明；**无答案**：从零推导 + 合理性自查。
- 尽量配 `pitfall`（易错）与 `insight`（举一反三 / 一题多解）。

## 约束与排错

- 主题色默认北大红 `#94070a`，改 `templates/style.css` 顶部 `:root` 的 `--pku-red` 等变量即可换主题。
- 资料在远程服务器（如 `yl:`）：先 `scp -O -r yl:<路径> .` 拉到本地再处理。
- 体量很大的课程（十几章）：分批派 subagent，避免一次并发过多；每批之间校验补漏。
- 整理完若用户要分享，衔接 `/view-html`：本 skill 的输出天然自包含，可直接部署。
