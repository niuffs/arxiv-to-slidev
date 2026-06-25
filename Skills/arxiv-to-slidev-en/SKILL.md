---
name: arxiv-to-slidev-en
description: Automatically generate Slidev seminar presentations from arXiv LaTeX source code. Use this skill whenever the user provides an arXiv paper's LaTeX source (zip or folder) and asks to "make slides", "create a presentation", "seminar talk", "group meeting presentation", or "academic talk" — even if they don't explicitly mention this skill. This skill extracts abstract, introduction, method, experiments, conclusions etc. from LaTeX sources, builds a complete slidev project, auto-copies experimental figures, and produces a polished slides.md with proper math rendering and image paths.
---

# arxiv-to-slidev-en: From arXiv LaTeX to Slidev Presentation

## Core Philosophy

This skill transforms an arXiv paper's LaTeX source into a well-structured, beautifully rendered Slidev seminar presentation. The workflow follows "understand the paper → extract assets → build outline → write slides page by page", ensuring coverage of **background, contributions, algorithm principles, mathematical derivations, experimental validation, and conclusions**, with automatic handling of custom LaTeX command expansion, image copying, and formula rendering.

---

## Format Specification

The generated `slides.md` must strictly follow the format rules below.

### 1. Frontmatter

```markdown
---
theme: default
title: 'Paper Title — Seminar'
author: 'Author Name'
keywords: 'keywords'
class: text-center
transition: slide-left
monaco: false
lineNumbers: false
katex: true
---

<style>
.slidev-page { overflow-y: auto !important; }
</style>
```

**Notes:**
- `katex: true` — explicitly enables KaTeX math rendering (Slidev v52+ does NOT enable it by default; this field is required)
- Overflow and opacity: Slidev default theme applies `opacity: 0.5` to the first paragraph after `h1` (`.slidev-layout h1 + p`). Must add CSS override in the frontmatter:
  ```yaml
  head:
    - style: |
        .slidev-page, .slidev-slide-content { overflow-y: scroll !important; }
        .slidev-layout h1 + p { opacity: 1 !important; }
  ```
- Font size hierarchy: Maintain consistent font sizes across all slides. Formula sizes should match the surrounding text scale. Use `\displaystyle` only when necessary.
- Always add a space between text and `$...$` delimiters — text immediately adjacent to `$` causes KaTeX parsing failures:
  - ❌ `adapt$\mathbf{\Phi}$to the`
  - ✅ `adapt $\mathbf{\Phi}$ to the`
- Prefer `\boldsymbol{}` over `\mathbf{}` for Greek letters (`\boldsymbol{\Phi}`, `\boldsymbol{\Psi}`) for maximum KaTeX compatibility.

### 2. Slide Separators & Layout

- Each `---` separates one slide
- Cover page: use `layout: center` and `class: text-center`
- Grid layout for images: use `<div class="grid grid-cols-N gap-2">` (N = number of columns), with each image in its own `<div>` using native Markdown syntax:
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
- **No `<br>` tags** — use blank lines or two trailing spaces for line breaks

### 3. Math Formulas (Native Markdown)

Use KaTeX syntax (built into Slidev, requires `katex: true` in frontmatter). **Must use native Markdown syntax**:

- Inline: `$...$`
- Display: `$$...$$` (**only in body paragraphs, NEVER in tables or HTML tags**)
- Multi-line: `\begin{aligned}...\end{aligned}`

**Forbidden:** `\[...\]`, `\(...\)`, or any HTML-wrapped formulas.

#### ⚠️ Critical Rule: No Spaces Inside Inline Math `$...$`

Slidev v52+ requires the `$` delimiter to be **directly adjacent** to the formula content. Spaces break rendering:

```markdown
<!-- ❌ WRONG: spaces after/before $ → renders as literal text -->
$ \Xi(t) $   →   shows as "$ \Xi(t) $"
$ \mathbf{x} $   →   shows as "$ \mathbf{x} $"

<!-- ✅ CORRECT: $ touches content → renders as formula -->
$\Xi(t)$
$\mathbf{x}$
```

**Fix**: After generating `slides.md`, run:
```
Find: \$ (.+?) \$
Replace: $\1$
```

#### ⚠️ Critical: Formulas in Tables

Markdown tables and HTML tags **cannot use `$$...$$` display math**, or they will render as literal `$$` text.

**Rule 1: Always use inline `$...$` in tables**

```markdown
<!-- ❌ WRONG: $$ in table cell -->
| Method | Formula |
|--------|---------|
| DDIM | $$x_{t-1} = ...$$ |

<!-- ✅ CORRECT: use $...$ with \displaystyle for larger formulas -->
| Method | Formula |
|--------|---------|
| DDIM | $\displaystyle x_{t-1} = \frac{\alpha_{t-1}}{\alpha_t}x_t + \sigma_{t-1}\epsilon_\theta$ |
```

**Rule 2: Replace `|` with `\vert` in table formulas (CRITICAL)**

**Root cause:** Markdown table parsers process `|` as column separators BEFORE recognizing `$...$` math delimiters. Even `|` inside `$...$` will be treated as a table column separator, causing the cell to be split incorrectly and the formula to display as garbled text.

```markdown
<!-- ❌ WRONG: | inside $...$ is treated as table separator -->
| $\mathbf{x}_{T|t_n}$ | Conditional distribution |   ← THIS BREAKS THE TABLE

<!-- ✅ CORRECT: use \vert -->
| $\mathbf{x}_{T\vert t_n}$ | Conditional distribution |
```

**Checklist:** In every table cell that contains a formula, scan for `|` characters and replace ALL of them with `\vert`. This applies to complex formulas with subscript pipes like `\mathbf{x}_{0|\overline{\sigma}_n}` → `\mathbf{x}_{0\vert\overline{\sigma}_n}`.

**Rule 3: No line breaks `\\` or alignment `&` in table cells**

Use `\displaystyle` to flatten multi-line formulas into a single line.

**Rule 4: Only inline `$...$` inside HTML highlight boxes**

```markdown
<!-- ❌ WRONG: $$ inside div -->
<div class="p-4 bg-blue-50 border-l-4 border-blue-500 rounded-lg my-4">
Theorem: $$
E = mc^2
$$
</div>

<!-- ✅ CORRECT: use $...$ inside div -->
<div class="p-4 bg-blue-50 border-l-4 border-blue-500 rounded-lg my-4">

Theorem: $E = mc^2$

(For larger formulas, use $\displaystyle \int_0^\infty f(x)\,dx$)

</div>
```

> **Root cause:** Slidev's KaTeX renderer only processes `$$...$$` at the Markdown root level. `$$...$$` nested inside HTML tags (`<div>`, `<span>`, etc.) or Markdown table cells is NOT processed by KaTeX, and displays as literal `$$` text. Inline `$...$` does NOT have this limitation — it renders correctly everywhere.

### 4. Images (Native Markdown)

Images must use native Markdown syntax. **No `<img>` tags allowed**:

```markdown
![](./figures/xxx.png)
```

All images live in the project root `figures/` folder, referenced as `./figures/xxx.png`.

**Forbidden:** `<img src="...">`, `<div>` wrapping images, or any HTML image syntax.

### 5. Highlight Boxes

Only use these three styled `<div>` boxes for visual highlighting. **No other HTML tags allowed.**

```html
<!-- Blue info box (theorems, key formulas, notes) -->
<div class="p-4 bg-blue-50 border-l-4 border-blue-500 rounded-lg my-4">
content
</div>

<!-- Green result box (conclusions, contributions, results) -->
<div class="p-4 bg-green-50 border-l-4 border-green-500 rounded-lg my-4">
content
</div>

<!-- Red warning box (limitations, problems, cautions) -->
<div class="p-4 bg-red-50 border-l-4 border-red-500 rounded-lg my-4">
content
</div>
```

**Note:** Highlight boxes must NOT contain nested HTML tags — only Markdown text and formulas.

### 6. Horizontal Rules

Use `---` for section dividers within a slide. **No `<hr>` tags.**

### 7. Forbidden HTML Tags

| Tag | Replacement |
|-----|-------------|
| `<br>` | Blank line or two trailing spaces |
| `<img>` | `![](path)` |
| `<hr>` | `---` |
| `<p>` | Markdown paragraph (blank line) |
| `<span>` | Direct Markdown text |
| `<font>` | Not needed; Slidev defaults suffice |

### 8. Slide Length Budget

- Each slide: **15-25 lines** of content (including blanks)
- Slides exceeding 30 lines should be split
- Image-heavy slides can be standalone
- Large tables: wrap with `<div style="max-height:400px;overflow:auto;">` (only exception)
- Key mathematical derivations can span 2-3 slides

---

## Workflow

### Step 1: Environment Check & Slidev Project Initialization

If no Slidev project exists, initialize one:

1. **Check for `slides.md`** (Slidev project marker)
   - Exists → use it, go to Step 2
   - Does not exist → create new folder and initialize

2. **Initialize** (prefer global install, fall back to local):
   ```bash
   # Check if global slidev is available
   slidev --version 2>/dev/null || npm install -g @slidev/cli @slidev/theme-default playwright-chromium

   # Create project structure
   mkdir -p project-name
   cd project-name
   ```

3. **Create `figures/` folder** (single image directory):
   ```bash
   mkdir -p figures
   ```

4. **Delete `public/` folder** (if auto-generated during init):
   ```bash
   rm -rf public 2>/dev/null
   ```
   The project maintains only one `figures/` folder as the unified image directory.

5. **Create `.gitignore`**:
   ```
   node_modules/
   dist/
   .DS_Store
   *.log
   ```

### Step 2: Read & Analyze LaTeX Source

1. Locate the main `.tex` file (typically `icml_camera_ready.tex`, `neurips_2025.tex`, `main.tex`, etc.)

2. **Scan paper structure** using Grep/Read:
   - `\begin{abstract}` ... `\end{abstract}` → Abstract
   - `\section{Introduction}` → Background
   - `\section{Preliminaries}` → Preliminaries
   - `\section{Method/Methodology/Approach}` → Method
   - `\section{Related Work}` → Related Work
   - `\section{Experiments/Results}` → Experiments
   - `\section{Conclusion}` → Conclusion

3. **Extract key math**: Scan `\begin{equation}`, `$$`, `\(...\)` environments

4. **Extract terminology/abbreviations**: Record full names on first occurrence

5. **Find figure files**: Scan `\includegraphics{...}` commands

### Step 3: Handle LaTeX Custom Commands

This is critical for correct Markdown output.

1. **Locate the preamble**: Find everything before `\begin{document}` in the main `.tex` file, plus all files loaded via `\input{}` or `\include{}` (e.g., `preamble.tex`, `defs.tex`, `macros.tex`).

2. **Extract custom commands** from the preamble:
   - `\newcommand{\...}{...}`
   - `\newcommand*{\...}{...}`
   - `\renewcommand{\...}{...}`
   - `\DeclareMathOperator{\...}{...}`
   - `\def\...{...}` — TeX primitive
   - `\let\...\...` — command alias

3. **Focus on these command types**:
   - **Math symbol shorthands**: e.g., `\newcommand{\bX}{\mathbf{X}}`, `\newcommand{\hatY}{\hat{Y}}`, `\newcommand{\cA}{\mathcal{A}}`
   - **Operators**: e.g., `\DeclareMathOperator{\KL}{KL}`
   - **Paper-specific symbols**: e.g., `\newcommand{\s}{\boldsymbol{s}_\theta}`, `\newcommand{\ep}{\boldsymbol{\epsilon}}`

4. **Build a replacement map**:
   ```
   \bX → \mathbf{X}
   \hatY → \hat{Y}
   \cA → \mathcal{A}
   \KL → \operatorname{KL}
   \ep → \boldsymbol{\epsilon}
   ```

5. **Apply replacements in `slides.md`**:
   - Replace all custom LaTeX commands with KaTeX-compatible standard commands
   - Handle recursive expansion (e.g., `\newcommand{\bX}{\mathbf{X}}` referenced by `\newcommand{\X}{\bX}`)
   - Expand nested commands down to the lowest-level standard commands

6. **Verify replacements**:
   - Check that all replaced formulas render correctly in KaTeX
   - `\boldsymbol{}` is supported by KaTeX (keep as-is)
   - `\mathcal{}`, `\mathbb{}`, `\mathfrak{}` are all KaTeX-compatible

### Step 4: Copy Images (Preserving Subdirectory Structure)

1. Locate the paper's `figures/` folder (may also be `fig/`, `images/`, `imgs/`, etc.)

2. Identify PPT-worthy figures:
   - Architecture diagrams (network structures, flowcharts)
   - Experimental plots (line charts, bar charts, heatmaps)
   - Comparison result tables
   - Algorithm illustrations (TikZ or drawn)

3. **Recursively copy the entire figures folder, preserving subdirectory structure**:
   ```bash
   cp -r <paper-path>/figures/* figures/
   ```
   This ensures:
   - If the paper has subdirectories (e.g., `figures/uncond_sampling/`, `figures/cond_sampling/`), they are preserved
   - No filename collisions across subdirectories
   - Easy path distinction in `slides.md`

4. **Image paths in `slides.md`**:
   ```markdown
   <!-- No subdirectory -->
   ![](./figures/fig2_overview.png)

   <!-- With subdirectories -->
   ![](./figures/uncond_sampling/ddim_celebhq.png)
   ![](./figures/cond_sampling/50/rex_rk4/sample.png)
   ```
   Ensure every `./figures/...` path matches the actual directory structure exactly (case-sensitive).

5. **Important principle: All images must come from the original paper — do NOT draw any diagrams or flowcharts yourself.**
   - Use existing architecture diagrams, comparison figures, and experiment curves directly
   - Re-present tables as native Markdown tables, not screenshots
   - For TikZ-drawn figures: check for standalone `.tex` files in `tikz/` subdirectory, try compiling independently, or extract from the compiled PDF
   - **Do NOT** use third-party drawing tools (draw.io, Figma, PPT, etc.) to create any diagrams

6. **TikZ figures** (supplementary):
   - For TikZ figures that cannot be extracted, describe them in text only
   - Do NOT draw ASCII art or hand-drawn replacements

### Step 5: Plan Slide Structure

Standard structure (40-60 slides):

| Section | Slides | Content |
|---------|--------|---------|
| **Cover** | 1 | Title, authors, venue, date |
| **Outline** | 1 | Table of contents |
| **Task Intro** | 1-2 | Problem context, applications (for non-expert audience) |
| **Background** | 2-3 | Limitations of existing work, motivation |
| **Contributions** | 1 | Summary of core contributions |
| **Preliminaries** | 3-5 | Prerequisites, notation, background algorithms |
| **Method** | 8-15 | Core algorithm, math derivations, theoretical analysis |
| **Related Work** | 2-3 | Comparison methods, formula comparison, summary table |
| **Experiments** | 5-10 | Setup, quantitative/qualitative results, ablations |
| **Conclusion** | 1-2 | Summary and future work |
| **References** | 1 | Key references |
| **Thank You** | 1 | Q&A |

### Step 6: Write slides.md

Follow the **Format Specification** rules. Content guidelines per section:

#### Cover
- Paper title (original)
- Authors, affiliation
- Venue, year
- "Seminar · YYYY/MM/DD"

#### Task Introduction (Critical!)
Before diving into background, set context for non-expert audience:
- What problem is this?
- Why does it matter?
- Concrete application examples

#### Background & Motivation
- Limitations of existing approaches (red callout box)
- Why this paper's method is needed

#### Contributions
- Numbered list of 3-4 core contributions

#### Method (Core Section)
Page-by-page detailed exposition:
- **Each formula** with explanation of its physical/mathematical meaning
- **Key theorems** in blue info boxes
- **Important corollaries** in green result boxes
- Each major step gets its own slide

#### Experiments
- Overview table (dataset, model, metrics, key findings)
- Each experiment on its own slide(s)
- Quantitative results in tables, qualitative in image comparisons
- Key findings in green callout boxes

#### Conclusion
- Bullet points summarizing contributions and findings
- Bullet points for future work

### Step 7: Post-processing — Fix Inline Formula Spacing

After writing `slides.md`, run the following regex replacement to remove spaces between `$` and formula content.

#### Problem

Slidev v52+ requires `$` to directly touch the formula content — `$ x $` renders as literal text, not a formula.

#### Fix Command

```bash
cd <project>
sed -i 's/\$ \([^$]*\) \$/$\1$/g' slides.md
```

This replaces all `$ ... $` patterns (where `...` contains no `$`) with `$...$`, stripping the surrounding spaces.

#### Verify

```bash
# Count inline formulas before and after
grep -o '\$[^$]*\$' slides.md | wc -l
```

Ensure the count stays the same (no incorrect splitting of formulas that contain `$` internally).

#### Note
- This only removes one space on each side of `$`; internal spaces in the formula are untouched
- Manually spot-check 3-5 complex formulas after the fix

---

## Build Verification

After generating `slides.md`, **must** run these checks in order:

### Step 1: Fix formula spacing

```bash
cd <project>
sed -i 's/\$ \([^$]*\) \$/$\1$/g' slides.md
```

### Step 2: Build test

```bash
cd <project> && slidev build slides.md 2>&1
```

### Step 3: Fix errors

If build fails, locate KaTeX parse errors. Common issues:
- `$` still has surrounding spaces → re-run sed from Step 1
- Unexpanded LaTeX custom commands → check Step 3 replacement map
- Special characters (`&`, `\\`, `\|`, `_`) → fix manually
- `$$...$$` still inside `<div>` or table cells → change to `$...$`

### Step 4: Rebuild until clean

### Step 5: Check images in `dist/`

### Step 6: Spot-check formula rendering

Open `dist/index.html` (or run `slidev slides.md` dev server). Check 3-5 formula-heavy slides:
- No literal `$ ... $` or `$$` text visible
- All inline and display math renders as proper mathematical symbols

---

## Troubleshooting

### Q1: Images not displaying
- `./figures/xxx.png` must match the actual file path (case-sensitive)
- Check `figures/` directory exists and filenames match
- Only one `figures/` folder — no `public/` directory

### Q2: Formulas showing as literal `$$` text
**Most common causes (in order):**

1. **`$ content $` has spaces inside** ← most frequent
   - Fix: `sed -i 's/\$ \([^$]*\) \$/$\1$/g' slides.md`
   - See: Step 7

2. **`$$...$$` inside HTML tags (e.g., `<div>`)**
   - Fix: Move `$$...$$` outside, or switch to `$...$`
   - See: Format §3 Rule 4

3. **`$$...$$` inside Markdown table cells**
   - Fix: Use `$\displaystyle ...$` instead
   - See: Format §3 Rule 1

4. **`|` pipe in table cell formulas**
   - Fix: Replace with `\vert`
   - See: Format §3 Rule 2

5. **Unexpanded LaTeX custom commands**
   - Fix: Check preamble for `\newcommand` etc.
   - See: Step 3

### Q3: Slide content clipped
**No extra handling needed.** The frontmatter already includes:
```css
<style>
.slidev-page { overflow-y: auto !important; }
</style>
```
This auto-adds scrollbars when content overflows. If a slide still feels crowded, split it (15-20 lines per slide).

### Q4: `slidev` command not found
```bash
npm install -g @slidev/cli
```
Verify: `slidev --version`

### Q5: PDF export
```bash
cd <project>
slidev export slides.md
```
Requires `playwright-chromium` (install globally).

### Q6: All images from the original paper?
**Yes.** This skill strictly follows "all figures from original paper materials only." No self-drawn diagrams, no third-party drawing tools.

---

## Deliverables

Final project structure:

```
project-name/
├── slides.md              ← Main presentation file (strict format)
├── figures/               ← Single image directory (from original paper)
├── .gitignore
└── README.md              ← Quick start guide
```

(Note: only one `figures/` folder — no `public/` directory.)
