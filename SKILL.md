---
name: exam-template
description: >
  Use this skill whenever the user wants to create, generate, or fill in an HTML exam paper
  (试卷) using the provided exam-template.html layout. Triggers include: "帮我出一套试卷",
  "生成试卷", "制作考试题", "用模板出题", "help me make an exam", "generate exam paper",
  or any request to produce a printable quiz/test in HTML with math formulas, multiple-choice,
  fill-in-the-blank, or free-response sections. Also trigger when the user pastes or references
  exam-template.html and asks for a populated version. Always use this skill — do NOT write
  exam HTML from scratch when this template and these instructions are available.
---

# Exam Template Skill

This skill produces a fully populated HTML exam paper by filling in `exam-template.html`.
The template uses KaTeX (bundled locally) for math rendering and produces clean print output.

**Asset locations (both in `assets/`):**
- `assets/exam-template.html` — the base template to copy and populate
- `assets/katex/` — the KaTeX library folder

---

## File Delivery Rules

- **Copy** `assets/exam-template.html` and `assets/katex/` to a working path (e.g. `/workspace/<title>-exam.html`).  
  Do **not** edit the asset in place.
- When done, present **only the HTML file** to the user with `present_files`.  

---

## Input Handling

Users typically upload a **Markdown or DOCX file** containing the exam questions.

**For provided content — preserve exactly:**
- All question stems, wording, and options
- The question numbering as given
- Section groupings and their order
- All formulas, symbols, and notation (convert to KaTeX if not already)
- Image positions, order, and captions — preserve exactly as provided
- Do not replace, crop, or modify any user-provided images
- Caption text must be used verbatim; do not auto-number or fabricate captions

**For missing information — generate sensibly or use defaults:**

| Missing field | Action |
|--------------|--------|
| Main title (`<h1>`) | Infer from subject/context (e.g. "高等数学期末考试"), or use "试卷" as fallback |
| Exam duration / 满分 | Leave as `__分钟` / `__分` placeholders in the subtitle |
| Per-question score (`q-score`) | Distribute points proportionally if total given; else use `（__分）` |
| Section score summary in section-title | Compute if per-question score is known; else use `共__分` |
| Info-bar fields | Keep default: 班级 / 姓名 / 学号 / 得分 |
| Footer text | Use default: `— 试卷结束，请检查作答 —` |
| Image caption | Omit `.q-image-caption` element entirely — do not render, do not occupy space |
| Image file missing (but referenced) | Insert placeholder `<div class="q-image-placeholder">[ 请插入配图 ]</div>` |
| Layout mode unspecified | Default to block-centered (`.q-image-block`) |
| Option image size unspecified | Default to 80px max-height |

Never invent or alter question content. Only generate structural/metadata fields.

---

## Workflow

1. Read the uploaded file (Markdown or DOCX) to extract all questions, images, and captions.
   - **Markdown**: Parse `![caption](image-path)` syntax to extract image paths and corresponding captions
   - **DOCX**: Extract embedded images and corresponding caption text, match positions by question order
2. Identify sections, question types, and image positions — follow the user's structure exactly; preserve image order and placement as given.
3. Copy `assets/exam-template.html` and `assets/katex/` to `/workspace/<name>-exam.html` and `/workspace/katex/`. When images are present, create `/workspace/images/` folder and move all image resources there.
4. Fill in the HTML following the rules below, using relative paths `images/filename.ext` for all images.
5. Present only the HTML file with `present_files`.
6. Remind the user:
   - `katex/` folder must be alongside the HTML file for formulas to render
   - `images/` folder (if present) must be alongside the HTML file for images to display

---

## Header Customization

### Paper title
```html
<div class="paper-header">
  <h1>2024 年秋季高等数学期末考试</h1>
  <p class="subtitle">考试时间：120分钟 &nbsp;|&nbsp; 满分：100分 &nbsp;|&nbsp; 请将答案填写在作答区域内</p>
</div>
```
- `<h1>`: main title, centered bold
- `.subtitle`: duration, total score, instructions — separate items with `&nbsp;|&nbsp;`

### Info bar (student fields)
```html
<div class="info-bar">
  <div class="info-field">班级：<span class="underline" style="min-width:100px;"></span></div>
  <div class="info-field">姓名：<span class="underline" style="min-width:80px;"></span></div>
  <div class="info-field">学号：<span class="underline" style="min-width:120px;"></span></div>
  <div class="info-field">得分：<span class="underline" style="min-width:60px;"></span></div>
</div>
```
- Add / remove `<div class="info-field">` lines freely
- Control underline length with `min-width` (px)

---

## Section Title

```html
<div class="section-title">一、选择题（第 1—5 题，每题 4 分，共 20 分）</div>
```

- Use Chinese ordinals: 一 二 三 四 五 …
- Pattern: `[序号]、[题型名称]（第 A—B 题，每题 X 分，共 Y 分）`
- **Follow the user's actual section names and order.** Do not default to 选择题/填空题/解答题 if the user's exam uses different groupings (e.g. 判断题, 计算题, 证明题, 名词解释, 论述题, 实验题, etc.).

---

## Question Types & Layout Elements

The template provides a set of building blocks. **Combine them freely** to match any question format — the three classic types are not the only valid structure.

### Core question wrapper (all types)
```html
<div class="question">
  <div class="q-row">
    <span class="q-no">N.</span>
    <div class="q-body">
      <!-- stem + any inline elements here -->
    </div>
    <span class="q-score">（X分）</span>
  </div>
  <!-- optional: .solve-area goes here, outside .q-row -->
</div>
```

---

### Element A — Multiple-Choice Options

Add inside `.q-body`, after the stem. End the stem with （&ensp;&ensp;）.

**Two-column (default — short options):**
```html
<div class="options">
  <div class="opt"><span class="opt-key">A.</span><span>选项 A</span></div>
  <div class="opt"><span class="opt-key">B.</span><span>选项 B</span></div>
  <div class="opt"><span class="opt-key">C.</span><span>选项 C</span></div>
  <div class="opt"><span class="opt-key">D.</span><span>选项 D</span></div>
</div>
```

**Single-column (long options or formula-heavy):**
```html
<div class="options options-single">
  ...
</div>
```

---

### Element B — Inline Fill Blank

Place inside `.q-body` where the blank appears. Adjust `min-width` to match expected answer length (60–160px).

```html
$f(x) = $<span class="blank-inline" style="min-width:80px;"></span>
```

For a blank inside a formula, use LaTeX underline instead:
```html
$\lim_{x \to 0} \dfrac{\sin x}{x} = \underline{\hspace{80px}}$
```

---

### Element C — Below-stem Answer Row

Place inside `.q-body`, after the stem text. Use for blanks that need a dedicated line below the question.

```html
<div class="blank-area"></div>
```

---

### Element D — Free-Response Answer Lines

Place **outside** `.q-row` but inside `.question`. Each `<span>` is one dashed line.

```html
<div class="solve-area">
  <span class="solve-line"></span>
  <span class="solve-line"></span>
  <!-- repeat as needed -->
</div>
```

**Line count guide:**

| Points | Suggested lines |
|--------|----------------|
| 4–5 分  | 5–6 行 |
| 6–8 分  | 8–10 行 |
| 10–12 分 | 12–15 行 |
| 15 分以上 | 16–20 行 |

---

### Element E — Question Images

Images are placed within `.q-body` (for stem images), inside `.opt` (for option images), or inside `.solve-area` (for answer-area images).

**Layout Mode 1 — Block Centered (default):**
```html
<div class="q-image q-image-block">
  <img src="images/geometry-1.png" alt="Triangle ABC diagram">
  <div class="q-image-caption">图1 三角形ABC示意图</div>
</div>
```
- Standalone block, horizontally centered
- Max-width: 80% of content area
- Margin: 12px top/bottom
- Use for main diagrams like geometry figures, experimental setups

**Layout Mode 2 — Float Left/Right (text wrap):**
```html
<div class="q-image q-image-float-right">
  <img src="images/device.png" alt="Experimental apparatus">
</div>
如右图所示的实验装置中...
```
- `q-image-float-left` or `q-image-float-right`
- Max-width: 35% of content area
- Text wraps around the image
- Use for small diagrams referenced in text like "如图所示..."

**Layout Mode 3 — Multiple Images Side by Side:**
```html
<div class="q-image-group">
  <div class="q-image q-image-block">
    <img src="images/fig-a.png" alt="Figure A">
    <div class="q-image-caption">图A</div>
  </div>
  <div class="q-image q-image-block">
    <img src="images/fig-b.png" alt="Figure B">
    <div class="q-image-caption">图B</div>
  </div>
</div>
```
- Images arranged horizontally with 16px gap
- Each image can have its own caption
- Use for comparison figures, experimental groups

**Option Images (inside `.opt`):**
```html
<div class="opt">
  <span class="opt-key">A.</span>
  <span><div class="q-image"><img src="images/fig-a.png" alt="Option A"></div></span>
</div>
```
- Max-height: 80px, auto width
- Captions hidden by default
- Compatible with both `.options` (2-column) and `.options-single` (1-column)

**Answer Area Images (inside `.solve-area`):**
```html
<div class="solve-area">
  <div class="q-image q-image-float-left q-image-border" style="max-width: 45%;">
    <img src="images/coordinate-blank.png" alt="Blank coordinate system">
    <div class="q-image-caption">答题辅助坐标系</div>
  </div>
  <span class="solve-line"></span>
  <span class="solve-line"></span>
</div>
```
- Place images above answer lines
- Recommended: `q-image-float-left` at 40-50% max-width

**Optional Styles:**
- `.q-image-border` — adds 1px light gray border (for charts, diagrams)
- `.q-image-bg` — adds white background (for transparent PNGs to prevent print issues)
- Combine: `class="q-image q-image-block q-image-border q-image-bg"`

**Image Placeholder (when file is missing):**
```html
<div class="q-image q-image-block">
  <div class="q-image-placeholder">[ 请插入配图 ]</div>
</div>
```

---

### Combining Elements

Elements A–D can be combined in any question. Examples:

**Choice + display formula in stem:**
```html
<div class="q-body">
  设 $f(x) = x^2 + 1$，则 $\int_0^1 f(x)\,dx = $（&ensp;&ensp;）。
  $$\text{（备注：此为块级公式示例）}$$
  <div class="options">...</div>
</div>
```

**Fill-blank + answer lines (calculation fill-blank):**
```html
<div class="q-body">
  $\lim_{x\to 0}\dfrac{\sin 3x}{x} = $<span class="blank-inline" style="min-width:80px;"></span>
</div>
<!-- give working space -->
<div class="solve-area">
  <span class="solve-line"></span>
  <span class="solve-line"></span>
  <span class="solve-line"></span>
</div>
```

**True/False (判断题) — no options element needed, just inline blank:**
```html
<div class="q-body">
  若 $f'(x_0)=0$，则 $x_0$ 一定是 $f(x)$ 的极值点。（&ensp;&ensp;）
</div>
```

**Short-answer / 名词解释 — stem + blank-area:**
```html
<div class="q-body">
  请解释"极限"的定义。
  <div class="blank-area"></div>
</div>
```

---

## Math Formulas (KaTeX)

KaTeX auto-renders `$…$` (inline) and `$$…$$` (display block).
Always use LaTeX for all mathematical expressions — never plain text.

### Inline formula
```
已知 $f(x) = \sin x + \cos x$，求 $f'(x)$。
```

### Display (block) formula
```
$$\int_0^{\pi} \sin x\,dx = 2$$
```

### Common LaTeX reference

| Output | LaTeX |
|--------|-------|
| Fraction (display) | `\dfrac{a}{b}` |
| Fraction (text) | `\frac{a}{b}` |
| Square root | `\sqrt{x}` · n-th root `\sqrt[n]{x}` |
| Definite integral | `\int_a^b f(x)\,dx` |
| Double integral | `\iint_D f\,dA` |
| Partial derivative | `\dfrac{\partial f}{\partial x}` |
| Evaluated partial | `\left.\dfrac{\partial f}{\partial x}\right|_{(1,2)}` |
| Limit | `\lim_{x \to 0}` |
| Sum | `\sum_{k=1}^{n}` |
| Vector | `\vec{a}` · `\overrightarrow{AB}` |
| Matrix | `\begin{pmatrix} a & b \\ c & d \end{pmatrix}` |
| System of eqs | `\begin{cases} x+y=1 \\ x-y=0 \end{cases}` |
| Inline fill blank | `\underline{\hspace{80px}}` |
| Absolute value | `\left| x \right|` |
| Infinity | `\infty` |
| Greek letters | `\alpha \beta \gamma \delta \varepsilon \theta \lambda \mu \pi \sigma \varphi \omega` |
| ≤ ≥ ≠ ≈ | `\le \ge \ne \approx` |
| → ∈ ⊂ ∪ ∩ | `\to \in \subset \cup \cap` |

> Always add `\,` before `dx`, `dy`, `dA` etc. in integrals.

---

## Footer

```html
<div class="paper-footer">— 试卷结束，请检查作答 —</div>
```

Customize the text as needed.

---

## Quick-Copy Blocks

### Multiple-choice question
```html
<div class="question">
  <div class="q-row">
    <span class="q-no">N.</span>
    <div class="q-body">
      STEM（&ensp;&ensp;）。
      <div class="options">
        <div class="opt"><span class="opt-key">A.</span><span>OPT_A</span></div>
        <div class="opt"><span class="opt-key">B.</span><span>OPT_B</span></div>
        <div class="opt"><span class="opt-key">C.</span><span>OPT_C</span></div>
        <div class="opt"><span class="opt-key">D.</span><span>OPT_D</span></div>
      </div>
    </div>
    <span class="q-score">（X分）</span>
  </div>
</div>
```

### Fill-in-the-blank question
```html
<div class="question">
  <div class="q-row">
    <span class="q-no">N.</span>
    <div class="q-body">
      STEM = <span class="blank-inline" style="min-width:80px;"></span>。
    </div>
    <span class="q-score">（X分）</span>
  </div>
</div>
```

### Free-response question (8 lines)
```html
<div class="question">
  <div class="q-row">
    <span class="q-no">N.</span>
    <div class="q-body">STEM</div>
    <span class="q-score">（X分）</span>
  </div>
  <div class="solve-area">
    <span class="solve-line"></span><span class="solve-line"></span>
    <span class="solve-line"></span><span class="solve-line"></span>
    <span class="solve-line"></span><span class="solve-line"></span>
    <span class="solve-line"></span><span class="solve-line"></span>
  </div>
</div>
```

### True/False question (判断题)
```html
<div class="question">
  <div class="q-row">
    <span class="q-no">N.</span>
    <div class="q-body">
      STEM（&ensp;&ensp;）
    </div>
    <span class="q-score">（X分）</span>
  </div>
</div>
```

### Short-answer / 名词解释 question
```html
<div class="question">
  <div class="q-row">
    <span class="q-no">N.</span>
    <div class="q-body">
      STEM
      <div class="blank-area"></div>
    </div>
    <span class="q-score">（X分）</span>
  </div>
</div>
```

### Multiple-choice with stem image (block centered)
```html
<div class="question">
  <div class="q-row">
    <span class="q-no">N.</span>
    <div class="q-body">
      STEM with reference to image below（&ensp;&ensp;）。
      <div class="q-image q-image-block q-image-border">
        <img src="images/xxx.png" alt="Image description">
        <div class="q-image-caption">图注文字</div>
      </div>
      <div class="options">
        <div class="opt"><span class="opt-key">A.</span><span>OPT_A</span></div>
        <div class="opt"><span class="opt-key">B.</span><span>OPT_B</span></div>
        <div class="opt"><span class="opt-key">C.</span><span>OPT_C</span></div>
        <div class="opt"><span class="opt-key">D.</span><span>OPT_D</span></div>
      </div>
    </div>
    <span class="q-score">（X分）</span>
  </div>
</div>
```

### Multiple-choice with image options
```html
<div class="question">
  <div class="q-row">
    <span class="q-no">N.</span>
    <div class="q-body">
      STEM（&ensp;&ensp;）。
      <div class="options">
        <div class="opt"><span class="opt-key">A.</span><span><div class="q-image"><img src="images/fig-a.png" alt="图A"></div></span></div>
        <div class="opt"><span class="opt-key">B.</span><span><div class="q-image"><img src="images/fig-b.png" alt="图B"></div></span></div>
        <div class="opt"><span class="opt-key">C.</span><span><div class="q-image"><img src="images/fig-c.png" alt="图C"></div></span></div>
        <div class="opt"><span class="opt-key">D.</span><span><div class="q-image"><img src="images/fig-d.png" alt="图D"></div></span></div>
      </div>
    </div>
    <span class="q-score">（X分）</span>
  </div>
</div>
```

### Fill-in-the-blank with float image
```html
<div class="question">
  <div class="q-row">
    <span class="q-no">N.</span>
    <div class="q-body">
      <div class="q-image q-image-float-right">
        <img src="images/xxx.png" alt="Image">
      </div>
      STEM with answer blank <span class="blank-inline" style="min-width:80px;"></span>。
    </div>
    <span class="q-score">（X分）</span>
  </div>
</div>
```

### Fill-in-the-blank with multiple images
```html
<div class="question">
  <div class="q-row">
    <span class="q-no">N.</span>
    <div class="q-body">
      STEM <span class="blank-inline" style="min-width:60px;"></span>。
      <div class="q-image-group">
        <div class="q-image q-image-block q-image-border">
          <img src="images/fig-a.png" alt="图A">
          <div class="q-image-caption">图A</div>
        </div>
        <div class="q-image q-image-block q-image-border">
          <img src="images/fig-b.png" alt="图B">
          <div class="q-image-caption">图B</div>
        </div>
      </div>
    </div>
    <span class="q-score">（X分）</span>
  </div>
</div>
```

### Free-response with answer-area image
```html
<div class="question">
  <div class="q-row">
    <span class="q-no">N.</span>
    <div class="q-body">STEM</div>
    <span class="q-score">（X分）</span>
  </div>
  <div class="solve-area">
    <div class="q-image q-image-float-left q-image-border" style="max-width: 45%;">
      <img src="images/coordinate-blank.png" alt="Blank coordinate">
      <div class="q-image-caption">答题辅助坐标系</div>
    </div>
    <span class="solve-line"></span><span class="solve-line"></span>
    <span class="solve-line"></span><span class="solve-line"></span>
    <span class="solve-line"></span><span class="solve-line"></span>
    <span class="solve-line"></span><span class="solve-line"></span>
  </div>
</div>
```
