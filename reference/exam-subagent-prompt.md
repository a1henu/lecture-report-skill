# 每章「往年真题精解」subagent 的 prompt 模板

派发前替换 `{{...}}`。**一个 subagent 负责一章的全部真题**，多个并行派发（`general-purpose`，模型继承主会话；做数学/证明务必 Opus 级）。每个 subagent 拿到的是**主 agent 在归类阶段已经切好、并标注好出处/题型/考点的「本章题目清单」**（含原始题面，以及参考答案——如果试卷带答案）。

---

你要为《{{COURSE_NAME}}》的 **{{CHAPTER_TITLE}}** 制作一份「往年真题精解」HTML 页面。

## 输入与输出
- 本章题目清单（已归类，逐题含：出处编号、题型、考点、原始题面、参考答案如有）：
{{QUESTION_LIST}}
- 如需核对题面/答案细节，可读原始试卷：`{{SOURCE_PATHS}}`（用 Read，PDF 每次 `pages` ≤20 页）。
- 输出文件：写入 `{{OUTPUT_DIR}}/exam-{{CHAP_FILE}}.html`（单一自包含 HTML）。
- 同目录已有 `style.css` / `script.js`，**直接引用，不要重建**。

## HTML 骨架
参照 `templates/exam-chapter.html`。要点：
- 页头用靛蓝渐变 `style="background:linear-gradient(135deg,#4f46e5 0%,#3730a3 100%)"`。
- 侧边栏 `brand` 写「往年真题 · {{CHAPTER_SHORT}}」，返回链接指向 `exam-index.html`；若本课程也整理了讲义知识报告，再加一条指向 `{{CHAP_FILE}}.html`（📖 看本章知识报告）。
- 开篇 `<h2 id="overview">本章考情概览`：一张 `table.data` 列出本章每道真题的「出处/题型/考点/难度」。
- 正文**按题型分组**（计算 / 证明 / 选择填空 / 应用）或按考点分组，每组一个 `<h2>`，组内逐题。
- 末尾 `<h2 id="summary">本章高频考点小结`：一张速查表汇总高频考点、典型套路、易错点。

## 每道真题的结构（务必逐题照此写）
```html
<div class="box exam">
  <div class="box-title"><span class="tag">真题</span> 第 N 题</div>
  <div class="exam-meta">
    <span class="exam-badge year">2023 期末·三</span>
    <span class="exam-badge type">计算题</span>
    <span class="exam-badge kaodian">考点：……</span>
    <span class="exam-badge diff">难度 ★★☆</span>
    <!-- 该考点在历年多次出现时再加： <span class="exam-badge freq">高频</span> -->
  </div>
  <p class="stem">原样转录的完整题面（公式 MathJax）。</p>
  <details class="solution"><summary></summary><div>
    <p><strong>考点定位：</strong>本题考查……，用到的知识点……。</p>
    <ol class="steps">
      <li><strong>第一步——做什么 &amp; 为什么：</strong>……</li>
      <li>……</li>
      <li><strong>结论：</strong>……</li>
    </ol>
    <p class="final-answer">$……$</p>
    <div class="box pitfall"><div class="box-title"><span class="tag">易错</span></div><p>……</p></div>
    <div class="box insight"><div class="box-title"><span class="tag">举一反三</span></div><p>同类变式 / 一题多解……</p></div>
  </div></details>
</div>
```

## 内容质量要求（最重要）
1. **题面原样转录**，不改数字、不删条件；公式一律 MathJax。
2. **解答放进 `<details class="solution">`**（默认折叠），方便先自测后对答案。
3. **每题先「考点定位」**：点明考哪个知识点、属于讲义哪一节，把真题和知识体系挂钩。
4. **逐步解析**：用 `<ol class="steps">`，每步说清「做什么 + 为什么这么做」，不跳步；该查表的写明查到的分位数/临界值。
5. **最终答案**用 `.final-answer` 高亮。
6. **有参考答案时**：核对；若答案有误或跳步，**纠正并在解析里注明差异**（`<span class="muted">原卷答案为…，但应为…</span>`），不照抄。
7. **无参考答案时**：从零完整推导，并在结论处做一次合理性自查（量纲/范围/特例验证）。
8. 每题尽量给 `pitfall`（易错点）与 `insight`（举一反三 / 一题多解）。
9. 全中文；用既定 CSS 组件，不自创样式；侧边栏 nav 与所有 `<h2 id>` 对应。

## 收尾自检
- 文件存在、以 `</html>` 结尾、`<div>`/`<details>`/`<table>` 标签平衡。
- 每个 `href="#x"` 都有 `id="x"`。
- 本章清单里的题**一道都不能漏**。

完成后返回简短摘要：本章收录题数、按题型/考点分布、发现的原卷答案错误（若有）。
