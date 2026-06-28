# PaperSet V1.1 配图功能变更说明

## （1）解决的问题

### 核心痛点
本次升级解决了原模板无标准化配图能力的核心问题：
- 原模板无配图插入的标准实现，用户生成带图试卷时需手动修改 HTML 插入图片，排版混乱，破坏原有样式统一性
- 手动插入的图片无统一排版规范，易出现错位、尺寸失控等问题，影响试卷专业度
- 不兼容题干配图、选项内配图、作答区辅助图等主流配图场景，大幅降低出卷效率

### 功能覆盖范围
- 支持三类配图位置：题干内嵌配图、选项内配图、作答区辅助图
- 支持三种排版模式：块级居中（默认）、文绕图（左浮/右浮）、多图并排
- 支持可选图注，有则展示、无则隐藏，不占用版面空间
- 支持可选样式扩展：边框样式、背景填充

---

## （2）修改的代码

### 样式层（CSS 新增类名）

所有新增样式位于 `exam-template.html` 的 `<style>` 标签内，在解答题样式（`.solve-line`）之后、页脚样式（`.paper-footer`）之前。

| 类名 | 类型 | 核心样式规则 |
|------|------|-------------|
| `.q-image` | 基础类 | `page-break-inside: avoid`，为所有配图提供分页保护 |
| `.q-image img` | 基础类 | `max-width: 100%; height: auto; object-fit: contain`，确保图片等比例缩放、不拉伸变形 |
| `.q-image-caption` | 图注类 | 字号 12px，颜色 `#666`，行高 1.4，水平居中，距图片 6px |
| `.q-image-block` | 排版扩展类（块级居中） | `display: block; margin: 12px auto; max-width: 80%`，单独占一行、水平居中、最大宽度 80%、上下外边距 12px |
| `.q-image-block img` | 排版扩展类 | `max-height: 70vh`，竖版长图限制最大高度为页面高度 70% |
| `.q-image-float-left` | 排版扩展类（文绕图左浮） | `float: left; max-width: 35%; margin: 4px 12px 8px 0` |
| `.q-image-float-right` | 排版扩展类（文绕图右浮） | `float: right; max-width: 35%; margin: 4px 0 8px 12px` |
| `.q-image-group` | 排版扩展类（多图并排容器） | `display: flex; justify-content: center; gap: 16px; flex-wrap: wrap` |
| `.opt .q-image` | 选项内配图专项 | `max-height: 80px; vertical-align: middle` |
| `.opt .q-image-caption` | 选项内配图专项 | `display: none`，选项内默认隐藏图注 |
| `.q-image-border` | 可选样式类 | 图片增加 1px 浅灰色（`#ccc`）边框 |
| `.q-image-bg` | 可选样式类 | 图片增加白色背景填充 |
| `.q-image-placeholder` | 占位类 | 图片加载失败/缺失时的灰色虚线占位框，提示文字居中 |

**补充修改：**
- `.q-body::after` 和 `.solve-area::after` 增加 `clear: both` 清除浮动，确保文绕图模式不影响后续元素排版
- `@media print` 中新增 `.q-image` 和 `.q-image-group` 的 `page-break-inside: avoid` 打印分页保护

### 结构层（DOM 结构规范）

配图组件与原有题目结构的嵌套关系：

1. **题干内嵌配图**：放置于 `.q-body` 内部，题干文字中间或末尾
2. **选项内配图**：放置于 `.opt` 内部的选项内容 `<span>` 中
3. **作答区辅助图**：放置于 `.solve-area` 内部、答题行（`.solve-line`）上方

### 兼容说明
本次升级为**增量修改**，完全向后兼容旧版试卷生成逻辑：
- 所有新增 CSS 类均为新增选择器，不修改原有类的样式规则
- 未引入任何外部依赖，保持模板轻量
- 原有无图试卷的渲染效果完全不受影响
- 原有题型结构、KaTeX 渲染、打印适配、信息栏、页脚等功能全部保留

---

## （3）新功能用法与使用规则

### 配图位置选择规则

| 配图位置 | 适用场景 | 放置层级 | 推荐排版模式 |
|---------|---------|---------|------------|
| 题干内嵌配图 | 题目描述中引用的示意图（几何图、实验装置图等） | `.q-body` 题干文字中 | 块级居中（默认）/ 文绕图 |
| 选项内配图 | 选择题选项为图形/图表（函数图像、结构图等） | `.opt` 选项容器内 | 无需额外排版类，自动适配 |
| 作答区辅助图 | 解答题答题区域提供辅助底图（空白坐标系等） | `.solve-area` 内、答题行上方 | 文绕图（左浮） |

### 图注使用规则

- **显隐逻辑**：原题提供图注文本时渲染 `.q-image-caption` 元素；未提供时省略该元素，不占用版面空间
- **格式要求**：完全沿用原题提供的图注文本，不自动编号、不补充内容
- **适用场景**：题干配图和多图并排推荐使用图注；选项内配图默认隐藏图注，如需标注统一纳入选项文字内容

### 图片资源放置规则

- 所有图片资源统一放置在 `images/` 文件夹中
- `images/` 文件夹需与 HTML 文件处于**同一目录**
- HTML 中图片引用统一使用相对路径：`images/文件名.后缀`
- 含配图的试卷交付时，需提醒用户将 `images/` 文件夹与 HTML 同目录放置，否则图片无法显示

### 样式扩展规则

| 扩展类 | 适用场景 | 启用方式 |
|--------|---------|---------|
| `.q-image-border` | 几何图、图表、坐标图等需要明确边界的图片 | 在 `.q-image` 类后追加 `.q-image-border` |
| `.q-image-bg` | 透明背景的 PNG 图片，避免打印时丢失细节 | 在 `.q-image` 类后追加 `.q-image-bg` |

可同时启用多个扩展类，例如：
```html
<div class="q-image q-image-block q-image-border q-image-bg">
```

### 输入处理规则

**保留规则（严格遵循）：**
- 严格保留图片的位置、顺序、图注文本
- 不擅自替换、裁剪、修改用户提供的图片内容
- 图注文本完全沿用用户提供内容，不自行编造、不自动编号

**缺失信息处理：**

| 缺失项 | 处理方式 |
|--------|----------|
| 图片无图注 | 不渲染 `.q-image-caption` 元素，不占用版面空间 |
| 用户描述配图但未提供图片文件 | 插入等尺寸占位框（`.q-image-placeholder`），显示"[ 请插入配图 ]"，灰色边框 |
| 未指定排版模式 | 默认使用"块级居中"模式（`.q-image-block`） |
| 选项内图片无尺寸说明 | 统一使用 80px 最大高度 |

### 异常与边界场景处理规则

1. **图片加载失败/路径错误**
   - 显示灰色占位框（`.q-image-placeholder`），尺寸与默认图一致，内部提示"图片加载失败"
   - 不影响题目整体布局与其他内容渲染

2. **超大尺寸图片**
   - 自动按容器最大宽度等比例缩放，不超出试卷内容区
   - 保留原图分辨率

3. **竖版长图**
   - 单张图片最大高度不超过页面高度的 70%（`max-height: 70vh`）
   - 超出自动等比缩放，避免单张图过长导致版面失衡

4. **分页截断问题**
   - 为 `.q-image`、`.q-image-group`、`.question` 容器添加 `page-break-inside: avoid` 打印属性
   - 保证题干与所属图片不会被分页截断，始终在同一页面内

---

## （4）新增快速复制代码块

### 块级居中配图（带图注）

```html
<div class="q-image q-image-block">
  <img src="images/xxx.png" alt="图片描述">
  <div class="q-image-caption">图注文字</div>
</div>
```

### 块级居中配图（无图注）

```html
<div class="q-image q-image-block">
  <img src="images/xxx.png" alt="图片描述">
</div>
```

### 文绕图配图（右浮）

```html
<div class="q-image q-image-float-right">
  <img src="images/xxx.png" alt="图片描述">
</div>
```

### 文绕图配图（左浮）

```html
<div class="q-image q-image-float-left">
  <img src="images/xxx.png" alt="图片描述">
</div>
```

### 多图并排配图

```html
<div class="q-image-group">
  <div class="q-image q-image-block">
    <img src="images/fig-a.png" alt="图A">
    <div class="q-image-caption">图A</div>
  </div>
  <div class="q-image q-image-block">
    <img src="images/fig-b.png" alt="图B">
    <div class="q-image-caption">图B</div>
  </div>
</div>
```

### 选项内配图

```html
<div class="opt">
  <span class="opt-key">A.</span>
  <span><div class="q-image q-image-border"><img src="images/fig-a.png" alt="选项A"></div></span>
</div>
```

### 作答区辅助图

```html
<div class="solve-area">
  <div class="q-image q-image-float-left q-image-border" style="max-width: 45%;">
    <img src="images/coordinate-blank.png" alt="空白坐标系">
    <div class="q-image-caption">答题辅助坐标系</div>
  </div>
  <span class="solve-line"></span>
  <span class="solve-line"></span>
  <span class="solve-line"></span>
  <span class="solve-line"></span>
  <span class="solve-line"></span>
  <span class="solve-line"></span>
  <span class="solve-line"></span>
  <span class="solve-line"></span>
</div>
```

### 带边框的配图（可选样式）

```html
<div class="q-image q-image-block q-image-border">
  <img src="images/xxx.png" alt="图片描述">
  <div class="q-image-caption">图注文字</div>
</div>
```

### 图片缺失占位框

```html
<div class="q-image q-image-block">
  <div class="q-image-placeholder">[ 请插入配图 ]</div>
</div>
```
