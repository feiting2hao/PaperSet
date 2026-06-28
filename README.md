# PaperSet

一个 AI Skill，能将你准备好的题目文稿自动排版为可打印的 HTML 试卷——支持 LaTeX 数学公式、灵活的题型组合与整洁的版面，所有内容输出为单个 HTML 文件。

---

## 功能简介

上传包含试题的 Markdown 或 Word（`.docx`）文件，AI 即可生成格式规范的 HTML 试卷。支持：

- 完全遵循你提供的大题结构与题目顺序
- 通过离线 KaTeX 库渲染 LaTeX 数学公式，无需联网
- 选择题（双列/单列）、填空题、解答题、判断题、简答/名词解释，以及上述题型的任意组合
- 对缺失的元信息（标题、满分、时长）给出合理默认值，题目内容严格保持原样

---

## 文件结构

```
exam-template-skill/
├── README.md                ← 本文件
├── SKILL.md                 ← AI 读取的排版指令
└── assets/
    ├── exam-template.html   ← 基础 HTML 模板
    └── katex/               ← 离线 KaTeX 库（数学公式渲染）
        ├── katex.min.js
        ├── katex.min.css
        ├── contrib/
        └── fonts/
```

---

## 安装方法

1. 下载 Skill 文件
2. 在支持的AI中进入 **设置 → Skills**，安装该 `.skill` 文件

---

## 使用方式

1. 将试题整理为 Markdown（`.md`）或 Word（`.docx`）文件
2. 上传至 Claude，并发送类似以下的指令：
   - `帮我把这份题目排成试卷`
   - `用模板出一套试卷`
   - `Generate an exam paper from this file`
3. AI 生成并输出 HTML 文件
4. 将 `katex/` 文件夹与下载的 HTML 文件放在**同一目录**下，数学公式方可正常渲染


---

## 内容处理规则

**严格保留原文的内容：**
- 所有题目题干、表述及选项文字
- 原有题目编号
- 大题名称及顺序
- 所有公式与数学符号（自动转换为 KaTeX 语法）

**缺失信息的处理方式：**

| 字段 | 处理方式 |
|------|---------|
| 试卷标题 | 根据学科/上下文推断，无法推断时使用"试卷"作为默认值 |
| 考试时长 / 满分 | 保留为占位符 `__分钟` / `__分` |
| 各题分值 | 若已知总分则按比例分配；否则使用 `（__分）` |
| 考生信息栏 | 默认显示：班级 / 姓名 / 学号 / 得分 |
| 页脚文字 | 默认显示：`— 试卷结束，请检查作答 —` |

---

## 支持的题型

模板由可自由组合的排版元素构成，不限于固定题型：

| 题型 | 实现方式 |
|------|---------|
| 选择题 | 选项块，双列或单列布局 |
| 填空题 | 行内下划线，或公式内 `\underline{\hspace{Xpx}}` |
| 解答 / 计算题 | 虚线作答横线，行数与分值成比例 |
| 判断题 | 题干后行内括号，无选项块 |
| 简答题 / 名词解释 | 题干 + 单行宽作答区 |
| 混合题型（如计算型填空） | 行内空格 + 作答横线组合 |

---

## 数学公式渲染

模板使用 [KaTeX](https://katex.org/) 从本地 `katex/` 文件夹加载，文件就位后无需网络连接。

使用标准 LaTeX 语法书写公式：

```
行内公式：$f(x) = \sin x$
独立公式：$$\int_0^1 x^2\,dx = \frac{1}{3}$$
```

---

## 版权声明与许可证

### 本项目（Skill 指令与 HTML 模板）

本项目的 Skill 指令（`SKILL.md`）与 HTML 模板（`exam-template.html`）以 **MIT 许可证**开源发布，你可以自由使用、修改和再分发。

---

### KaTeX（`assets/katex/` 目录下除字体外的文件）

本项目捆绑了 [KaTeX](https://katex.org/) 的 JavaScript 与 CSS 文件，遵循 **MIT 许可证**。

```
MIT License

Copyright (c) 2013-2020 Khan Academy and other contributors

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

KaTeX 项目地址：<https://github.com/KaTeX/KaTeX>

---

### KaTeX 字体文件（`assets/katex/fonts/` 目录）

`fonts/` 目录下的所有字体文件（`.ttf`、`.woff`、`.woff2`）遵循 **SIL 开放字体许可证 1.1 版（SIL Open Font License v1.1，OFL）**。

字体版权归属：

```
Copyright (c) 2009-2010, Design Science, Inc. (www.mathjax.org)
Copyright (c) 2014, Khan Academy (www.khanacademy.org)
with Reserved Font Name KaTeX_Main
```

这些字体由 Khan Academy 基于 Design Science, Inc.（MathJax 项目）的字体工作衍生制作，内嵌于各字体文件的元数据中可查阅完整授权声明。

OFL 核心条款摘要：

- **允许**：在任何软件（包括商业软件）中免费使用、捆绑和再分发这些字体
- **禁止**：将字体文件单独出售
- **要求**：若对字体进行修改后再发布，衍生版本须继续以 OFL 授权，且不得使用原字体的保留字体名称（Reserved Font Name，如 `KaTeX_Main` 等）

OFL 完整许可证文本：<https://openfontlicense.org/open-font-license-official-text/>

---

### 合规说明

分发或使用本项目时，请注意以下事项：

1. **保留本 README**：其中包含了所有必要的版权声明，满足 MIT 许可证的署名要求。
2. **不得移除 KaTeX 版权注释**：不得从 `katex.min.js`、`katex.min.css` 等文件中删除版权注释头。
3. **字体文件不可单独销售**：`fonts/` 目录中的字体文件只能作为软件包的一部分随项目分发，不可单独售卖。
4. **修改字体须改名**：若对字体文件进行任何修改，衍生字体须重命名（不可继续使用 `KaTeX_` 保留前缀），且须以 OFL 授权发布。
5. **无担保声明**：本项目及其所有捆绑的第三方组件均按"现状"提供，不附带任何明示或暗示的担保。