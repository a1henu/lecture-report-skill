# 每章 subagent 的 prompt 模板

派发时，把 `{{...}}` 占位符替换为具体内容，逐字复制其余部分。**一个 subagent 负责一个章节/部分**，多个 subagent 在同一条消息里并行派发（`subagent_type: general-purpose`，模型继承主会话——做数学/证明时务必用 Opus 级模型保证质量）。

---

你要为《{{COURSE_NAME}}》的 **{{CHAPTER_TITLE}}** 制作一份详尽的中文 HTML 学习报告。

## 输入与输出
- 阅读源材料：`{{SOURCE_PATH}}`（{{SOURCE_DESC}}，例如「PDF 共 N 页」）。
- 输出文件：写入 `{{OUTPUT_DIR}}/{{CHAP_FILE}}.html`（单一自包含 HTML）。
- 同目录已存在 `style.css` 与 `script.js`，**直接引用，不要重建**。

## 第一步：完整读完源材料
用 Read 工具分批读取（PDF 每次 `pages` 参数 ≤20 页，如 1-20、21-40……），**通读全部内容，不遗漏任何一页/任何例题**。逐一记录：章节标题、定义、定理、公式、每道例题/习题。
本章预计结构（以源材料实际内容为准，据实补全/修正）：
{{OUTLINE}}

## 第二步：写出 HTML（务必使用下面骨架）
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>{{CHAPTER_TITLE}} · {{COURSE_NAME}}学习报告</title>
<link rel="stylesheet" href="style.css">
<script>
window.MathJax = { tex: { inlineMath: [['$','$'],['\\(','\\)']], displayMath: [['$$','$$'],['\\[','\\]']], processEscapes: true, tags: 'none' }, svg: { fontCache: 'global' } };
</script>
<script src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js" async></script>
</head>
<body>
<div class="layout">
  <aside class="sidebar">
    <div class="brand">{{COURSE_NAME}}<small>学习报告 · {{CHAPTER_SHORT}}</small></div>
    <a class="home-link" href="index.html">← 返回首页目录</a>
    <nav><!-- 每个 <h2 id> 一条 <a href="#id">；重要 <h3 id> 用 class="lvl2" --></nav>
  </aside>
  <main class="content">
    <header class="page-header"><div class="inner">
      <div class="eyebrow">{{COURSE_EN}} · {{CHAPTER_EN}}</div>
      <h1>{{CHAPTER_TITLE}}</h1>
      <p>一句话概述。</p>
    </div></header>
    <div class="content-inner"><!-- 正文 --></div>
  </main>
</div>
<a class="back-top" href="#" title="回到顶部">↑</a>
<script src="script.js"></script>
</body>
</html>
```

## 可用的 CSS 组件（必须使用这些 class，不要自创样式）
- 盒子：`<div class="box TYPE"><div class="box-title"><span class="tag">标签</span> 名称</div> 内容</div>`
  TYPE ∈ `definition`(定义) / `theorem`(定理) / `example`(例题) / `proof`(证明) / `note`(注) / `insight`(直觉) / `pitfall`(易错)；`.tag` 文字相应改写。
- 分步解答：`<ol class="steps"><li>…</li></ol>`（自动编号成圆圈步骤，每步以「做什么+为什么」开头）。
- 关键结论：`<p class="conclusion">…</p>`（自动加 📌 前缀）。
- 冗长推导折叠：`<details class="derive"><summary>展开…</summary><div>…</div></details>`。
- 表格：`<div class="tbl-wrap"><table class="data"><thead>…</thead><tbody>…</tbody></table></div>`。
- 行内：`<span class="term">术语</span>`、`<span class="hl">高亮</span>`；二级标题编号 `<h2><span class="secnum">1</span>标题</h2>`；证毕 `<span class="qed">∎</span>`。

## 内容质量要求（最重要 —— 目标是脱离原始资料也能完全学懂）
1. **比原始资料更详细、更易懂**，不能只是照抄。
2. **每个定义**：精确陈述 + 用大白话解释直觉、为什么这样定义、有什么用、与已学概念的联系。
3. **每个定理/性质**：完整陈述（条件+结论）+ **完整逐步证明**，补全原资料跳过的步骤；过长的细节放进 `<details class="derive">`，保证主线清爽。
4. **每道例题/习题（务必全部覆盖，一道都不能漏）**：完整题面 + **每一步代数都写出来并解释「为什么这么做」** + 最终结论；若涉及查表，写明查到的临界值。同一问题有多种方法时把每种都做完并点评。
5. 适当加入 `insight`（直觉洞察）与 `pitfall`（易错点）。
6. 开篇写「本章导读」（本章在整套资料中的位置与作用），章末写「本章小结」（主线回顾 + 一张速查表 `table.data`）。
7. 全中文；所有数学符号用 MathJax（行内 `$…$`，行间 `$$…$$`），**绝不**把公式只用文字描述。
8. 侧边栏 `nav` 必须与正文所有 `<h2 id>` 及关键 `<h3 id>` 一一对应（id 用英文短横线）。
9. **发现原资料的错误/笔误**（如数据算错、结论矛盾）时，按正确结果重做，并用 `<span class="muted">` 注明差异，不要默默照抄。

## 收尾自检（写完必须做）
- 文件存在、以 `</html>` 结尾、`<div>`/`<details>`/`<table>` 标签开闭平衡。
- 每个 `href="#x"` 都有对应的 `id="x"`（可用 grep 自查）。
- 可分多次写入（先 Write 前半段，再 Edit 续写），但**最终必须是一个完整文件**。

完成后返回简短摘要：实际覆盖的章节标题、定理数、例题数、以及发现的原资料错误（若有）。
