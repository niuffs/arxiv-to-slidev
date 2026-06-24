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
katex: true
head:
  - style: |
      .slidev-page, .slidev-slide-content { overflow-y: scroll !important; }
---
```

**说明：**
- `katex: true` —— 显式启用 KaTeX 数学渲染（Slidev v52+ 默认不启用，必须设置此项）
- `head:` —— 通过 frontmatter 的 `head` 选项注入全局 CSS

#### 滚动条设置（重要）

Slidev 默认在 `.slidev-slide-container` 和 `.slidev-slide-content` 上设置了 `overflow: hidden`，导致内容超出时被截断且无法滚动。

需要在项目根目录下创建 `styles/index.css` 文件，Slidev 会自动加载该文件中的全局样式：

```css
/* styles/index.css：覆盖 Slidev 默认隐藏溢出行为 */
.slidev-slide-container { overflow: visible !important; }
.slidev-slide-content { overflow: visible !important; }
.slidev-page { overflow-y: scroll !important; }
```

**三层容器的含义：**
- `.slidev-slide-container` —— 最内层包裹容器，默认 `overflow: hidden` → 改为 `visible`
- `.slidev-slide-content` —— 幻灯片内容容器（绝对定位、居中），默认 `overflow: hidden` → 改为 `visible`
- `.slidev-page` —— 最外层页面容器，添加 `overflow-y: scroll` 固定显示垂直滚动条

**为什么不能只用 `<style>` 块？** Slidev 会将 `slides.md` 文件体中的 `<style>` 块作为 Vue 作用域样式编译（添加 `data-v-*` 属性选择器），由于作用域 ID 与 Slidev 组件的 DOM 元素不匹配，样式无法生效。`styles/index.css` 是全局样式文件，不受 Vue 作用域限制。`head:` 作为补充手段。
- 这些目录均为构建产物/辅助文件，**创建 slides 项目后可以一并删除**，仅保留 `slides.md` + `figures/` 即可
- 见 Step 1 中的清理说明

### 2. 分隔与布局

- 每个 `---` 分隔一页幻灯片
- 封面页使用 `layout: center` 和 `class: text-center`
- 双栏布局使用 HTML `<div class="grid grid-cols-2 gap-4">` + 每列各一个 `<div>`（此为幻灯片列布局需要，例外地使用 div）
- **禁止使用 `<br>` 标签**——换行通过空行或在 Markdown 行末加两个空格实现

### 3. 数学公式（原生 Markdown）

使用 KaTeX 语法（Slidev 内建支持，需在 frontmatter 中设置 `katex: true`），**必须使用原生 Markdown 语法**：

- 行内公式：`$...$`
- 行间公式：`$$...$$`（**只能用于正文段落中，禁止在表格和 HTML 标签内使用**）
- 多行公式：`\begin{aligned}...\end{aligned}`

**禁止使用** `\[...\]`、`\(...\)` 或任何 HTML 包裹公式。

#### ⚠️ 关键规则：行内公式的 `$` 与内容之间不能有空格

Slidev v52+ 要求行内公式的 `$` 与公式内容**紧邻，不能有空格**。这是最常见的问题：

```markdown
<!-- ❌ 错误：$ 与内容之间有空格，不会渲染，显示为文字 -->
$ \Xi(t) $   →  显示为 "$ \Xi(t) $"
$ \mathbf{x} $   →  显示为 "$ \mathbf{x} $"

<!-- ✅ 正确：$ 紧邻内容，正常渲染为公式 -->
$\Xi(t)$
$\mathbf{x}$
```

**修复方法**：生成 `slides.md` 后，用以下正则全局替换所有行内公式首尾的空格：

```
查找：\$ (.+?) \$
替换：$\1$
```

> **注意**：此规则仅针对 `$` 与公式内容之间紧邻的空格。公式**内部**的空格不受影响（如 `$\mathbf{x}_{T\vert t_n}$` 内部可以有空格）。

#### 其他注意事项

- **中文标点**：确保 `$...$` 前后无中文标点紧邻。如果有中文标点跟在 `$` 后，在标点前加空格，如 `$公式$ ；`。

#### ⚠️ 关键：表格中的公式处理

Markdown 表格和 HTML 标签中**不能使用 `$$...$$` 行间公式**，否则公式会被渲染为原文 `$$` 字符串。以下为具体规则：

**规则 1：表格中的公式一律使用行内格式 `$...$`**

```markdown
<!-- ❌ 错误：在表格中使用 $$...$$ -->
| 方法 | 公式 |
|------|------|
| DDIM | $$x_{t-1} = ...$$ |

<!-- ✅ 正确：表格中使用 $...$（如需行间样式，用 \displaystyle） -->
| 方法 | 公式 |
|------|------|
| DDIM | $\displaystyle x_{t-1} = \frac{\alpha_{t-1}}{\alpha_t}x_t + \sigma_{t-1}\epsilon_\theta$ |
```

**规则 2：表格公式中的 `|` 管道符用 `\vert` 替代**

```markdown
<!-- ❌ 错误：公式中的 | 与表格列分隔符冲突 -->
| $\mathbf{x}_{T|t_n}$ | 条件分布 |

<!-- ✅ 正确：用 \vert 替代 | -->
| $\mathbf{x}_{T\vert t_n}$ | 条件分布 |
```

**规则 3：表格公式中的换行和 `&` 对齐符不可用**

`\\` 换行和 `&` 对齐符在表格单元格中会破坏 Markdown 解析。改用 $\displaystyle$ 将公式展开为单行。

**规则 4：HTML 高亮框内只能使用行内公式 `$...$`**

```markdown
<!-- ❌ 错误：在 div 中使用 $$...$$ -->
<div class="p-4 bg-blue-50 border-l-4 border-blue-500 rounded-lg my-4">
定理：$$
E = mc^2
$$
</div>

<!-- ✅ 正确：div 中使用 $...$，多公式可分行写 -->
<div class="p-4 bg-blue-50 border-l-4 border-blue-500 rounded-lg my-4">

定理：$E = mc^2$

（如需较大公式，此处的 $\displaystyle \int_0^\infty f(x)\,dx$ 用 \displaystyle 放大）

</div>
```

> **根本原因：** Slidev 中的 KaTeX 渲染器只处理位于 Markdown 根层级的 `$$...$$` 语法。嵌套在 HTML 标签（`<div>`、`<span>`、`<p>` 等）或 Markdown 表格中的 `$$...$$` 不会被 KaTeX 处理，从而原样显示为 `$$` 文本。
>
> 行内公式 `$...$` 则不受此限制——Markdown 表格和 HTML 标签内的 `$...$` 均可被正确渲染。

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

1. **处理 arXiv 源码压缩包（如 `.tar.gz`、`.zip`）**：
   - 检查用户提供的源码路径是否为压缩包（以 `.tar.gz`、`.tgz`、`.zip` 结尾）
   - 如果是压缩包，先解压到一个独立的 `paper_source/` 文件夹：
     ```bash
     # 创建 paper_source 文件夹存放解压后的源码
     mkdir -p paper_source
     
     # 根据后缀选择解压方式
     tar -xzf <arxiv-file>.tar.gz -C paper_source/   # 适用于 .tar.gz
     unzip <arxiv-file>.zip -d paper_source/          # 适用于 .zip
     
     # 解压后 paper_source/ 中即包含 .tex 文件和 figures/ 等
     ```
   - 解压后，后续步骤中的"源码路径"均指向 `paper_source/` 目录
   - **注意**：`paper_source/` 仅用于存放原始源码，slides 项目建在**独立的文件夹**中（见下一步）

2. **创建独立的 slides 项目文件夹**（与 `paper_source/` 平级）：
   ```bash
   # 在 paper_source/ 同一父目录下，创建 slides 项目文件夹
   mkdir -p rex-seminar
   cd rex-seminar
   ```
   - slides 项目文件夹与 `paper_source/` 平级，互不干扰
   - 这样做的好处：解压的源码仅用于读取素材，slides 项目独立干净

3. **检查环境**（优先尝试全局安装，失败则本地安装）：
   ```bash
   # 先检查全局 slidev 是否可用
   slidev --version 2>/dev/null || npm install -g @slidev/cli @slidev/theme-default playwright-chromium
   ```

4. **创建 `figures/` 文件夹**（唯一图片目录）：
   ```bash
   mkdir -p figures
   ```

5. **创建 `styles/` 目录并写入全局滚动条样式**（确保所有页面可滚动）：
   ```bash
   mkdir -p styles
   cat > styles/index.css << 'EOF'
.slidev-slide-container { overflow: visible !important; }
.slidev-slide-content { overflow: visible !important; }
.slidev-page { overflow-y: scroll !important; }
EOF
   ```
   Slidev 会自动加载 `styles/index.css` 作为全局样式。

6. **清理非必要文件和目录**（`styles/` 是全局样式，如需保留则跳过删除）：
   ```bash
   rm -rf public node_modules dist 2>/dev/null
   ```
   **说明：** 创建 slides 项目后，`node_modules/`、`dist/`、`public/`、`styles/` 均为构建产物或辅助文件，**非必要保留**。最终交付物只需 `slides.md` + `figures/` 两个核心文件。
   - `node_modules/` — npm 依赖包（`npm install` 可重建）
   - `dist/` — `slidev build` 的构建输出
   - `public/` — Slidev 默认静态资源目录（我们统一用 `figures/` 代替）
   - `styles/` — 全局样式辅助文件（可按需保留）

7. **创建 `.gitignore`**：
   ```
   node_modules/
   dist/
   public/
   styles/
   .DS_Store
   *.log
   ```

### Step 2：读取并分析 LaTeX 源码

1. 找到用户提供的 LaTeX 源码文件夹或压缩包中的主要 `.tex` 文件（通常是 `icml_camera_ready.tex`、`neurips_2025.tex`、`main.tex` 等）
   - 如果是解压到 `paper_source/` 的，源码目录为 `../paper_source/`
   - 如果是直接提供的文件夹，使用该文件夹路径

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

### Step 4：复制图片素材（保留子文件夹结构）

1. 找到 LaTeX 源码包中的 `figures/` 文件夹（可能也叫 `fig/`、`images/`、`imgs/` 等）

2. 识别 PPT 需要使用的图片类型：
   - **模型架构图**（如网络结构、流程图）
   - **数值实验图**（折线图、柱状图、热力图等）
   - **对比实验结果表格**（截图或自动生成）
   - **算法示意图**（TikZ 或绘制的图）

3. **将整个图片文件夹递归复制到 `figures/` 目录，保留原有子文件夹结构**：
   ```bash
   # 如果解压到了 paper_source/（与 slides 项目平级）
   cp -r ../paper_source/figures/* figures/

   # 如果是直接指定了源码文件夹路径
   cp -r <源码文件夹路径>/figures/* figures/
   ```
   这样做的目的是：
   - 如果原文的 `figures/` 下有子文件夹（如 `figures/uncond_sampling/`、`figures/cond_sampling/`、`figures/al3/` 等），这些子文件夹及层次结构会被**原样保留**
   - 避免因同名文件在不同子文件夹中产生冲突
   - 方便在 `slides.md` 中通过子文件夹路径区分不同实验的图片

4. **图片引用路径**：在 `slides.md` 中按子文件夹路径引用图片：
   ```markdown
   <!-- 无子文件夹 -->
   ![](./figures/fig2_rex_overview.png)

   <!-- 有子文件夹 -->
   ![](./figures/uncond_sampling/ddim_celebhq.png)
   ![](./figures/cond_sampling/50/rex_rk4/a_close_up.png)
   ```
   确保 `slides.md` 中的每个 `./figures/...` 路径与 `figures/` 目录下的实际路径完全一致（包括大小写、子文件夹层次）。

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

### Step 7：后处理——修复行内公式空格

撰写完 `slides.md` 后，必须执行以下正则替换，修复行内公式首尾空格问题。

#### 问题现象

Slidev v52+ 要求行内公式 `$` 紧邻公式内容，`$ x $` 不会渲染而会显示为文字。

#### 修复命令

```bash
cd <project>
# 使用 sed 修复所有行内公式首尾空格
sed -i 's/\$ \([^$]*\) \$/$\1$/g' slides.md
```

这条命令会将所有 `$ ... $` 模式（其中 `...` 不包含 `$`）替换为 `$...$`，去除首尾空格。

#### 验证修复

修复后检查公式数量是否一致：
```bash
# 统计行内公式数量
grep -o '\$[^$]*\$' slides.md | wc -l
```
确保修复前后公式数量不变（没有因为公式内部包含 `$` 而错误拆分）。

#### 注意
- 该命令仅去除 `$` 与内容之间紧邻的一个空格，不改变公式内部的空格
- 如果公式本身包含 `$` 字符（极少见），需要手动检查
- 运行后手动抽查几处复杂公式，确认渲染正常

---

## 分页原则

- 每页幻灯片控制在 **15-25 行内容**（含空白行）
- 超过 30 行的页面需要拆分
- 图片页面可以单独一页
- 大型表格使用 `<div style="max-height:400px;overflow:auto;">` 包裹
- 关键的数学推导可以独立占用 2-3 页

---

## 编译验证

生成 `slides.md` 后，**必须**依次执行以下检查和修复：

### 第 1 步：修复行内公式空格

```bash
cd <project>
sed -i 's/\$ \([^$]*\) \$/$\1$/g' slides.md
```

去除所有 `$ 公式 $` 中 `$` 与公式内容之间的首尾空格，确保 KaTeX 能正常渲染。

### 第 2 步：编译测试

```bash
cd <project> && slidev build slides.md 2>&1
```

### 第 3 步：定位并修复错误

如果编译报错，定位 KaTeX 无法解析的公式。常见问题：
- `$` 与内容之间仍有空格 → 重新运行第 1 步的 sed 命令
- 未展开的 LaTeX 自定义命令 → 检查 Step 3 的替换映射表是否完整
- 特殊字符使用不当（`&`、`\\`、`\|`、`_` 等）→ 手动修复
- `$$...$$` 仍在 `<div>` 或表格中 → 改为 `$...$`

### 第 4 步：重新编译直到通过

修复后重新运行 `slidev build slides.md`，直到编译成功。

### 第 5 步：检查图片

编译通过后检查 `dist/` 中是否包含正确的图片文件（路径应为 `./figures/xxx.png`）。

### 第 6 步：抽检公式渲染

打开生成的 `dist/index.html`（或启动 `slidev slides.md` 开发服务器），抽查 3-5 处复杂公式：
- 确认没有 `$ 文字 $` 或 `$$` 原文显示
- 确认行内公式和行间公式均正确渲染为数学符号

---

## 常见问题处理

### Q1: 图片路径不显示
确保图片路径正确：
- `./figures/xxx.png` → `figures/xxx.png`（项目根目录）
- 如果还是不显示：检查 `figures/` 目录是否存在以及文件名是否匹配（注意大小写）
- **不要**同时保留 `public/` 和 `figures/`——项目仅维护一个 `figures/` 文件夹

### Q2: 公式渲染为 `$$` 文本
**最常见原因及解决方案（按出现频率排序）：**

1. **行内公式 `$ 内容 $` 含有首尾空格** ← 最常见
   - 原因：Slidev v52+ 要求行内公式 `$` 紧邻内容，`$ x $` 不会渲染
   - 解决：运行 `sed -i 's/\$ \([^$]*\) \$/$\1$/g' slides.md`
   - 参考：Step 7

2. **`$$...$$` 放在 HTML 标签（如 `<div>`）内部**
   - 原因：KaTeX 不处理 HTML 标签内的 `$$...$$`
   - 解决：将 `$$...$$` 移出 HTML 标签，或改用 `$...$` 行内公式
   - 参考：格式规范第 3 节规则 4

3. **`$$...$$` 放在 Markdown 表格单元格中**
   - 原因：表格单元格属于 HTML 上下文，KaTeX 不处理
   - 解决：改用 `$\displaystyle ...$` 行内格式
   - 参考：格式规范第 3 节规则 1

4. **公式中的 `|` 与 Markdown 表格列分隔符冲突**
   - 解决：用 `\vert` 替代 `|`：`$\mathbf{x}_{T\vert t_n}$`
   - 参考：格式规范第 3 节规则 2

5. **公式紧邻中文标点**
   - 解决：在标点前加空格，如 `$公式$ ；`
   - 参考：格式规范第 3 节

6. **LaTeX 自定义命令未展开**
   - 解决：检查导言区命令（`\newcommand` 等）是否全部替换为 KaTeX 标准命令
   - 参考：Step 3

### Q3: 页面内容被截断 / 滑动轴不生效
需要在项目中创建 `styles/index.css` 文件（Slidev 自动加载），写入：
```css
.slidev-slide-container { overflow: visible !important; }
.slidev-slide-content { overflow: visible !important; }
.slidev-page { overflow-y: scroll !important; }
```
同时 frontmatter 中通过 `head:` 注入样式作为补充：
```yaml
head:
  - style: |
      .slidev-page, .slidev-slide-content { overflow-y: scroll !important; }
```

**不能使用 slides.md 文件体中的 `<style>` 块**：Slidev 会将其作为 Vue 作用域样式编译（添加 `data-v-*` 属性选择器），因作用域 ID 不匹配而无法生效。

**不能只设 `overflow-y` 而忽略 `overflow: hidden`**：Slidev 默认在 `.slidev-slide-container` 和 `.slidev-slide-content` 上设置了 `overflow: hidden`，即使给外层加滚动条，内容仍然被内层容器裁剪。需要将内层容器改为 `overflow: visible`，再给外层加 `overflow-y: scroll`。

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
