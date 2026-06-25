---
name: arxiv-to-slidev
description: 从 arXiv 论文的 LaTeX 源码自动生成 Slidev 讨论班演示文稿。当用户提供 arXiv 论文的 LaTeX 源码压缩包或文件夹、并说"做 PPT"、"做幻灯片"、"做 slides"、"讨论班报告"、"组会汇报"、"学术报告"时，务必使用此 skill——即使未明确说"用 skill"。本 skill 从 LaTeX 源码中提取摘要、引言、方法、实验、结论等全结构内容，自动构建 slidev 项目并生成 slides.md，数值实验图自动复制到 figures/ 目录并配以文字说明。
---

# arxiv-to-slidev：从 arXiv LaTeX 到 Slidev 演示文稿

## 核心理念

本 skill 将一篇 arXiv 论文的 LaTeX 源码自动转化为结构完整、排版精美的 Slidev 讨论班幻灯片。流程基于"理解论文 → 提取素材 → 构建大纲 → 逐页撰写"的思路，确保 PPT 覆盖论文的**背景、贡献、算法原理、数学推导、实验验证、结论**等全部关键环节，并自动处理图片复制、公式替换、自定义命令展开等问题。

---

## 格式规范（参考 slides.md）

生成的 `slides.md` 必须遵循以下格式规范。以下规则均源自本 skill 配套的格式参考文件 `slides.md`。

### 1. 前导配置

```markdown
---
theme: default
title: '论文标题 — 讨论班'
author: '作者名'
keywords: '关键词'
class: text-center
transition: slide-left
monaco: false
lineNumbers: false
---

<style>
.slidev-page { overflow-y: auto !important; }
</style>
```

`overflow-y: auto !important` 确保内容超出页面高度时自动出现滚动条，避免内容被截断。

**注意：** Slidev 默认主题对 `h1` 后的段落施加 `opacity: 0.5`（灰色半透明效果），必须在 `head:` 中添加 CSS 覆盖：
```yaml
head:
  - style: |
      .slidev-page, .slidev-slide-content { overflow-y: scroll !important; }
      .slidev-layout h1 + p { opacity: 1 !important; }
```

### 1.1 字体和层级统一
- 同一套 PPT 中，标题字号、正文字号、公式大小需保持一致
- 数学公式和段落之间应有适当间距空格，避免文本紧贴 `$` 符号：
  - ❌ `adapt$\mathbf{\Phi}$to the` — 文本紧贴 `$`，KaTeX 可能解析错误
  - ✅ `adapt $\mathbf{\Phi}$ to the` — `$` 前后各留空格
- 使用 `\boldsymbol{}` 而非 `\mathbf{}` 对希腊字母加粗以保证兼容性

### 2. 分隔与布局

- 每个 `---` 分隔一页幻灯片
- 封面页使用 `layout: center` 和 `class: text-center`
- 双栏布局使用 HTML `<div class="grid grid-cols-2 gap-4">` + 每列各一个 `<div>`（此为幻灯片列布局需要，例外地使用 div）
- 多图横排布局：使用 `<div class="grid grid-cols-N gap-2">`（N 为列数），每张图放在独立的 `<div>` 中，图片仍用原生 Markdown 语法：
  ```html
  <div class="grid grid-cols-5 gap-2">
  <div>

  ![](./figures/img1.png)

  </div>
  <div>

  ![](./figures/img2.png)

  </div>
  </div>
  ```
- **禁止使用 `<br>` 标签**——换行通过空行或在 Markdown 行末加两个空格实现

### 3. 数学公式（原生 Markdown）

使用 KaTeX 语法（Slidev 内建支持），**必须使用原生 Markdown 语法**：

- 行内公式：`$...$`
- 行间公式：`$$...$$`
- 多行公式：`\begin{aligned}...\end{aligned}`

**禁止使用** `\[...\]`、`\(...\)` 或任何 HTML 包裹公式。

**注意**：确保 `$...$` 前后无中文标点紧邻。如果有中文标点跟在 `$` 后，在标点前加空格，如 `$公式$ ；`。

**表格中的公式**：若公式包含 `|` 管道符，用 `\vert` 替代：`$\mathbf{x}_{T\vert t_n}^\theta$`。注意 `|` 在 Markdown 表格中即使位于 `$...$` 内也会被部分解析器误认为列分隔符，因此在表格单元格的公式中**必须**将所有 `|` 替换为 `\vert`。

### 4. 图片引用（原生 Markdown）

图片必须使用原生 Markdown 语法，**禁止使用 `<img>` 标签**：

```markdown
![](./figures/xxx.png)
```

路径规则：所有图片统一放在项目根目录的 `figures/` 文件夹中，引用路径为 `./figures/xxx.png`。

**禁止：** `<img src="./figures/xxx.png">`、`<div>...</div>` 包裹图片、HTML 格式的图片引用。

### 5. 高亮显示框

仅为了显示美观为某些段落分块高亮显示时，才引入以下三种显示框。**禁止在其他场合使用 HTML 标签**。

```html
<!-- 蓝色信息框（用于定理、关键公式、说明） -->
<div class="p-4 bg-blue-50 border-l-4 border-blue-500 rounded-lg my-4">
内容
</div>

<!-- 绿色结果框（用于结论、贡献、实验亮点） -->
<div class="p-4 bg-green-50 border-l-4 border-green-500 rounded-lg my-4">
内容
</div>

<!-- 红色警告框（用于问题、局限性、注意事项） -->
<div class="p-4 bg-red-50 border-l-4 border-red-500 rounded-lg my-4">
内容
</div>
```

**注意：** 高亮框内不得嵌套其他 HTML 标签，仅可包含 Markdown 文本和公式。

### 6. 分隔线

页面内分隔内容用 `---` 分割线，**禁止使用 `<hr>` 标签**。

### 7. 禁止使用的 HTML 标签

以下 HTML 标签在 `slides.md` 中**一律禁止**使用：

| 禁止标签 | 替代方案 |
|---------|---------|
| `<br>` | 空行或行末两个空格 |
| `<img>` | `![](path)` |
| `<hr>` | `---` |
| `<p>` | Markdown 段落（空行分隔） |
| `<span>` | 直接 Markdown 文本 |
| `<font>` | 不需要；Slidev 默认样式即可 |

### 8. 分页原则

- 每页幻灯片控制在 **15-25 行内容**（含空白行）
- 超过 30 行的页面需要拆分
- 图片页面可以单独一页
- 大型表格使用 `<div style="max-height:400px;overflow:auto;">` 包裹（此为唯一的例外场景）
- 关键的数学推导可以独立占用 2-3 页

---

## 工作流程

### Step 1：检查环境与初始化 Slidev 项目

如果当前没有 Slidev 项目，需要初始化：

1. **检查是否有 `slides.md`**（Slidev 项目标志）
   - 有 → 直接使用，进入 Step 2
   - 无 → 在用户指定的位置新建文件夹，初始化 Slidev 项目

2. **初始化方式**（优先尝试全局安装，失败则本地安装）：
   ```bash
   # 先检查全局 slidev 是否可用
   slidev --version 2>/dev/null || npm install -g @slidev/cli @slidev/theme-default playwright-chromium

   # 创建项目结构
   mkdir -p project-name
   cd project-name
   ```

3. **创建 `figures/` 文件夹**（唯一图片目录）：
   ```bash
   mkdir -p figures
   ```

4. **删除 `public/` 文件夹**（如果初始化后自动生成）：
   ```bash
   rm -rf public 2>/dev/null
   # 如果存在 public/figures/ 等目录，一并删除
   ```
   项目只维护一个 `figures/` 文件夹作为统一图片目录。

5. **创建 `.gitignore`**：
   ```
   node_modules/
   dist/
   .DS_Store
   *.log
   ```

### Step 2：读取并分析 LaTeX 源码

1. 找到用户提供的 LaTeX 源码文件夹或压缩包中的主要 `.tex` 文件（通常是 `icml_camera_ready.tex`、`neurips_2025.tex`、`main.tex` 等）

2. **扫描论文结构**——使用 Grep/Read 工具提取以下章节：
   - `\begin{abstract}` ... `\end{abstract}` → 摘要
   - `\section{Introduction}` → 引言/背景
   - `\section{Preliminaries}` → 预备知识
   - `\section{Method/Methodology/Approach}` → 方法
   - `\section{Related Work}` → 相关工作
   - `\section{Experiments/Results}` → 实验
   - `\section{Conclusion}` → 结论

3. **提取关键数学公式**：扫描 `\begin{equation}`、`$$`、`\(...\)` 等环境，理解核心算法的数学原理

4. **提取专有名词/缩略语**：首次出现的专有名词记录其全称，需要在 PPT 中插入解释

5. **定位图片文件**：扫描 `\includegraphics{...}` 命令，找到引用的图片文件路径

### Step 3：处理 LaTeX 自定义命令

这是生成正确 Markdown 的关键步骤。

1. **定位导言区**：找到 main tex 文件中的 `\begin{document}` 之前的部分，以及所有通过 `\input{}` 或 `\include{}` 引入的样式/宏包文件（如 `preamble.tex`、`defs.tex`、`macros.tex` 等）。

2. **提取自定义命令**：在导言区中扫描以下模式：
   - `\newcommand{\...}{...}` —— 普通命令
   - `\newcommand*{\...}{...}` —— 带 `*` 的短命令
   - `\renewcommand{\...}{...}`
   - `\DeclareMathOperator{\...}{...}`
   - `\def\...{...}` —— TeX 原始定义
   - `\let\...\...` —— 命令别名

3. **重点关注的命令类型**：
   - **数学符号缩写**：如 `\newcommand{\bX}{\mathbf{X}}`、`\newcommand{\hatY}{\hat{Y}}`、`\newcommand{\cA}{\mathcal{A}}`
   - **运算符**：如 `\DeclareMathOperator{\KL}{KL}`
   - **论文特定符号**：如 `\newcommand{\s}{\boldsymbol{s}_\theta}`、`\newcommand{\ep}{\boldsymbol{\epsilon}}`

4. **创建替换映射表**：将收集到的自定义命令整理为替换表，例如：
   ```
   \bX → \mathbf{X}
   \hatY → \hat{Y}
   \cA → \mathcal{A}
   \KL → \operatorname{KL}
   \ep → \boldsymbol{\epsilon}
   ```

5. **在 `slides.md` 中进行替换**：
   - 将所有 LaTeX 自定义命令替换为 Markdown/KaTeX 可识别的标准命令
   - 某些命令可能需要递归展开（如 `\newcommand{\bX}{\mathbf{X}}` 再被 `\newcommand{\X}{\bX}` 引用）
   - 对于嵌套自定义命令，展开到最底层标准命令为止

6. **验证替换正确性**：
   - 检查替换后的公式是否能在 KaTeX 下正常渲染
   - 特别注意：`\boldsymbol{}` 在 KaTeX 中需保持为 `\boldsymbol{}`（KaTeX 支持）
   - 注意 `\mathcal{}`、`\mathbb{}`、`\mathfrak{}` 等在 KaTeX 中均支持

### Step 4：复制图片素材

1. 找到 LaTeX 源码包中的 `figures/` 文件夹（可能也叫 `fig/`、`images/`、`imgs/` 等）

2. 识别 PPT 需要使用的图片类型：
   - **模型架构图**（如网络结构、流程图）
   - **数值实验图**（折线图、柱状图、热力图等）
   - **对比实验结果表格**（截图或自动生成）
   - **算法示意图**（TikZ 或绘制的图）

3. **将选中的图片复制到统一图片目录**：
   ```bash
   cp <paper-path>/figures/*.png figures/
   cp <paper-path>/figures/*.jpg figures/
   ```

4. **图片引用路径**：在 `slides.md` 中统一使用 `./figures/xxx.png` 路径：
   ```markdown
   ![](./figures/xxx.png)
   ```

5. **重要原则：所有图片和图示均来自原文素材，不自行绘制任何框图或流程图。**
   - 论文中已有的架构图、对比图、实验曲线图直接使用
   - 论文中的表格内容以原生 Markdown 表格形式重新呈现，而非截图
   - 论文中由 TikZ 代码直接绘制的图：检查 LaTeX 包的 `tikz/` 文件夹是否有独立的 `.tex` 文件，可尝试独立编译生成 PDF 后提取；或者从已编译的 PDF 中截图保存
   - **不得**使用第三方绘图工具（draw.io、Figma、PPT 等）自行绘制任何示意图

6. **TikZ 图片处理**（作为补充）：
   - 对于无法从 PDF 截图或独立编译提取的 TikZ 图，在 slides.md 中直接用文字描述
   - 不要在幻灯片中画 ASCII 图或手工绘制替代图

### Step 5：规划幻灯片结构

根据论文内容，规划合理的幻灯片顺序。标准结构（约 40-60 页）：

| 板块 | 页数 | 内容 |
|------|------|------|
| **封面** | 1 | 标题、作者、会议、日期 |
| **目录** | 1 | 报告提纲 |
| **任务介绍** | 1-2 | 问题背景、应用场景（为非专业听众提供上下文） |
| **背景与动机** | 2-3 | 现有方法不足、本文动机 |
| **贡献总结** | 1 | 论文核心贡献概览 |
| **预备知识** | 3-5 | 前置概念、符号体系、基础算法原理（为后续推导铺垫） |
| **方法** | 8-15 | 核心算法、数学推导、理论分析（含收敛性、稳定性） |
| **相关工作** | 2-3 | 对比方法介绍、公式对比、综合对比表 |
| **实验** | 5-10 | 实验设置、定量/定性结果、消融实验 |
| **结论** | 1-2 | 总结与展望 |
| **参考文献** | 1 | 核心文献 |
| **致谢** | 1 | 感谢聆听 |

### Step 6：撰写 slides.md

按照**格式规范**一节中的规则，逐页撰写幻灯片。以下为各节的内容要点。

#### 封面页
- 论文标题（原文 + 中文翻译）
- 作者、单位
- 会议名、年份
- "讨论班报告 · YYYY 年 MM 月 DD 日"

#### 任务场景介绍（关键！）
在正式进入背景之前，先为非专业听众介绍**论文所解决问题的上下文**：
- 这是什么问题？
- 为什么重要？
- 具体应用场景有哪些？（用简短例子说明）

#### 背景与动机
- 现有方法的不足（用红色提示框标注）
- 为什么需要本论文的方法
- 可以画对比图说明差距

#### 贡献总结
- 用有序列表列出 3-4 个核心贡献
- 每个贡献用粗体标注

#### 预备知识
根据论文涉及的领域，补充必要的背景知识：
- 每个概念配以数学公式和简短说明

#### 方法（核心部分）
这是 PPT 的**核心章节**，需要逐页详细展开：
- **每个公式**都要配合文字解释其物理/数学意义
- **关键定理**放入蓝色提示框重点展示
- **重要推论**放入绿色提示框
- 方法部分的每个重要步骤都应该单独一页或多页

#### 理论性质
- 收敛性分析、稳定性分析
- 每个数学定义配文字解释

#### 相关工作
- 分类介绍对比方法
- 对每个对比方法给出算法公式
- 用**综合对比表**汇总各方法的属性

#### 实验
- 实验概览表（数据集、模型、指标、核心发现）
- 每个实验独立一页或多页
- 定量结果用表格展示，定性结果用图片对比
- 重要发现放入绿色提示框

#### 结论
- 用项目符号总结论文的主要贡献和发现
- 用项目符号列出未来工作方向

#### 参考文献
- 列出 6-10 篇核心参考文献
- 在引用处的上下文简要说明引用理由

---

## 分页原则

- 每页幻灯片控制在 **15-25 行内容**（含空白行）
- 超过 30 行的页面需要拆分
- 图片页面可以单独一页
- 大型表格使用 `<div style="max-height:400px;overflow:auto;">` 包裹
- 关键的数学推导可以独立占用 2-3 页

---

## 编译验证

生成 `slides.md` 后，**必须**执行以下检查：

1. 编译：`cd <project> && slidev build slides.md`
2. 如果编译报错，定位 KaTeX 无法解析的公式（通常是 `&`、`\\`、`\|`、`_`、未展开的 LaTeX 自定义命令等特殊字符使用不当）
3. 修复后重新编译，直到编译通过
4. 编译通过后检查 `dist/` 中是否包含正确的图片文件（图片引用路径应为 `./figures/xxx.png`）

---

## 常见问题处理

### Q1: 图片路径不显示
确保图片路径正确：
- `./figures/xxx.png` → `figures/xxx.png`（项目根目录）
- 如果还是不显示：检查 `figures/` 目录是否存在以及文件名是否匹配（注意大小写）
- **不要**同时保留 `public/` 和 `figures/`——项目仅维护一个 `figures/` 文件夹

### Q2: 公式渲染为 `$$` 文本
原因通常是：
- 公式中的 `|` 与 Markdown 表格冲突 → 用 `\vert` 替代
- 公式紧邻中文标点 → 在标点前加空格
- LaTeX 自定义命令未展开 → 检查导言区命令是否全部替换
- 公式在 HTML 标签内不被处理 → 移出 HTML 或使用 Markdown 原生表格

### Q3: 页面内容被截断
在 frontmatter 后添加：
```css
<style>
.slidev-page { overflow-y: auto !important; }
</style>
```
或者将长页面拆分为多个页面（每页 15-20 行内容为宜）。

### Q4: `slidev` 命令找不到
全局安装：`npm install -g @slidev/cli`
验证安装：`slidev --version`

### Q5: 如何导出 PDF
```bash
cd <project>
slidev export slides.md
```
需要安装 `playwright-chromium`（全局安装）。

### Q6: 图片全部来自原文吗？
**是的。** 本 skill 严格遵循"所有图片和图示均来自原文素材"的原则，不自行绘制任何框图或流程图。论文中已有的架构图、对比图、实验曲线图直接复制使用；论文中的表格以原生 Markdown 表格形式重新呈现。

---

## 交付物

skill 完成后输出的项目结构：

```
project-name/
├── slides.md              ← 主演示文稿文件（严格遵循格式规范）
├── figures/               ← 唯一图片目录（来自原文素材）
├── .gitignore
└── README.md              ← 简要使用说明
```

（注意：项目仅维护一个 `figures/` 文件夹，**不包含** `public/` 目录。）
