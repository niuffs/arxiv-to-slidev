# arxiv-to-slidev：arXiv 论文 → Slidev 讨论班 PPT

## 功能概述

从 arXiv 论文的完整 LaTeX 源码自动生成 Slidev 格式的学术讨论班演示文稿。覆盖**背景、贡献、预备知识、方法推导、实验验证、结论**等全结构内容。

## 使用方法

### 环境要求

```bash
# 全局安装 Slidev（仅需一次）
npm install -g @slidev/cli @slidev/theme-default playwright-chromium
```

### 调用 skill

在 Claude Code 中，提供 LaTeX 源码文件夹路径，说"把这篇论文做成讨论班PPT"即可触发本 skill。

或手动引用：

```bash
# 在你的项目文件夹中运行
slidev slides.md        # 启动开发服务器
slidev build slides.md   # 构建静态网站
slidev export slides.md  # 导出 PDF
```

### 输入

- arXiv 论文的 LaTeX 源码（文件夹或压缩包）
- 包含主 `.tex` 文件和 `figures/` 图片目录

### 输出

```
output-folder/
├── slides.md              ← 主演示文稿文件
├── figures/               ← 图片（md 预览用）
├── styles
```

## Skill 文件结构

```
arxiv-to-slidev/
├── SKILL.md               ← 核心指令（中文版）
├── README.md              ← 本文件
```

## 安装 Skill

将 `arxiv-to-slidev/` 文件夹放入 Claude Code 的 skills 目录，或在项目中通过 `.claude/settings.json` 引用。
