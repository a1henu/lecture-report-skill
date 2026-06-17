---
name: lecture-report
description: 把课程讲义 / slides / 教材章节 / PDF / 笔记等学习资料，整理成一套「比原资料更详细、能脱离原资料独立学懂」的中文 HTML 学习报告 —— 生成一个 report 文件夹，含一个导航 index.html 和每章一份独立 HTML。每个定义讲透直觉、每个定理给完整逐步证明、每道例题/习题给逐步解答与「为什么这样做」，全程 MathJax 渲染公式，统一的彩色盒子 + 步骤圆圈 + 速查表样式。用 subagent 按章并行整理。当用户说「把这份资料/讲义/slides 整理成学习笔记 / report / 学习报告」时调用。
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

`templates/` 里有四份现成资源（拷贝 / 当骨架用）：
- `style.css` —— 共享样式：彩色盒子（定义蓝/定理紫/例题青/证明灰/注橙/直觉·易错红）、步骤圆圈 `ol.steps`、关键结论 `p.conclusion`、折叠推导 `details.derive`、数据表 `table.data`、粘性侧边栏、首页卡片、响应式、回到顶部。北大红主题（改 `:root` 可换色）。
- `script.js` —— 侧边栏 scrollspy。
- `chapter.html` —— 单章页骨架（含各类盒子用法示例）。
- `index.html` —— 首页骨架。

`reference/subagent-prompt.md` —— 派给每章 subagent 的完整 prompt 模板（含质量要求与自检清单）。

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

## 约束与排错

- 主题色默认北大红 `#94070a`，改 `templates/style.css` 顶部 `:root` 的 `--pku-red` 等变量即可换主题。
- 资料在远程服务器（如 `yl:`）：先 `scp -O -r yl:<路径> .` 拉到本地再处理。
- 体量很大的课程（十几章）：分批派 subagent，避免一次并发过多；每批之间校验补漏。
- 整理完若用户要分享，衔接 `/view-html`：本 skill 的输出天然自包含，可直接部署。
