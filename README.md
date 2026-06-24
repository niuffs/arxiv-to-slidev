
# arxiv-to-slidev

> 从 arXiv 论文 LaTeX 源码自动生成 Slidev 讨论班演示文稿的 Claude Code Skill

---

## 🇨🇳 中文介绍

### 简介

**arxiv-to-slidev** 是一个 Claude Code Skill，用于从 arXiv 论文的 LaTeX 源码自动生成 [Slidev](https://github.com/slidevjs/slidev) 格式的讨论班演示文稿。

### 解决的问题

Markdown 格式的幻灯片（如 Marp）格式单一，排版能力有限；而 HTML 格式的幻灯片虽然展示美观，但不易于人为阅读和修改元素。

通过 [Slidev 项目](https://github.com/slidevjs/slidev)，本技能实现 **Markdown 文件内容** 和 **HTML 格式渲染** 的分离，兼顾了以下优势：

- **md 文件易于人为阅读修改** — 可直接在文本编辑器或 LLM 中修改内容
- **HTML 格式展示美观** — 支持 KaTeX 公式、主题切换、动画过渡等
- **节省 LLM Tokens** — 修改和打磨 Markdown 内容文件时无需处理 HTML/JS 代码，大幅节省 tokens

### 功能

- 从 arXiv LaTeX 源码提取摘要、引言、方法、实验、结论等全结构内容
- 自动复制论文中的数值实验图到幻灯片目录
- 自动展开 LaTeX 自定义命令（如 `\bX` → `\mathbf{X}`）
- 支持 KaTeX 数学公式渲染
- 支持中英文幻灯片生成

### 幻灯片操作

| 操作 | 快捷键 |
|------|--------|
| 下一页 | `←` `→` 方向键 或 `Space` |
| 上一页 | `←` 方向键 |
| 概览模式 | `O` |
| 黑暗模式 | `D` |
| 绘图模式 | `B` (按住) |
| 全屏 | `F` |

### 示例

- [中文版讨论班 PPT](https://niuffs.github.io/arxiv-to-slidev/cn/)
- [English Seminar PPT](https://niuffs.github.io/arxiv-to-slidev/en/)

### 安装

将 `Skills/arxiv-to-slidev-cn/` 或 `Skills/arxiv-to-slidev-en/` 放入 Claude Code 项目的 `.claude/skills/` 目录。

---

## 🇬🇧 English

### Introduction

**arxiv-to-slidev** is a Claude Code Skill that automatically generates [Slidev](https://github.com/slidevjs/slidev) seminar presentations from arXiv paper LaTeX source code.

### Problem Statement

Markdown-based slide tools (e.g., Marp) offer limited formatting options, while HTML-based slides are visually rich but difficult to edit and maintain manually.

By leveraging the [Slidev project](https://github.com/slidevjs/slidev), this skill separates **Markdown content** from **HTML rendering**, achieving both:

- **Easy-to-edit Markdown** — modify slides in any text editor or via LLM without touching HTML/JS
- **Beautiful HTML rendering** — KaTeX math, theme switching, animation transitions
- **LLM Token efficiency** — editing Markdown content consumes far fewer tokens than editing rendered HTML

### Features

- Extracts full paper structure: abstract, introduction, method, experiments, conclusion
- Auto-copies experimental figures from LaTeX source
- Expands LaTeX custom commands (e.g., `\bX` → `\mathbf{X}`)
- KaTeX math formula rendering
- Supports both Chinese and English presentations

### Slide Navigation

| Action | Shortcut |
|--------|----------|
| Next slide | `←` `→` Arrow keys or `Space` |
| Previous slide | `←` Arrow key |
| Overview mode | `O` |
| Dark mode | `D` |
| Drawing mode | `B` (hold) |
| Fullscreen | `F` |

### Examples

- [Chinese Seminar PPT](https://niuffs.github.io/arxiv-to-slidev/cn/) (Rex Paper)
- [English Seminar PPT](https://niuffs.github.io/arxiv-to-slidev/en/) (Rex Paper)

### Installation

Place `Skills/arxiv-to-slidev-cn/` or `Skills/arxiv-to-slidev-en/` into your Claude Code project's `.claude/skills/` directory.

---

## Repository Structure

```
├── Skills/
│   ├── arxiv-to-slidev-cn/       # Chinese Skill
│   └── arxiv-to-slidev-en/       # English Skill
├── examples/
│   ├── cn/                       # CN example source
│   └── en/                       # EN example source
└── .github/workflows/deploy.yml  # GitHub Pages auto-deploy
```
