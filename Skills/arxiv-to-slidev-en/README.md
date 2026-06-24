# arxiv-to-slidev-en: arXiv Paper → Slidev Seminar Presentation

## Overview

Automatically generate well-structured Slidev seminar presentations from arXiv paper LaTeX source code. Covers **background, contributions, preliminaries, method derivations, experimental validation, and conclusions** in a complete presentation workflow.

## Usage

### Prerequisites

```bash
# Global install of Slidev (one-time only)
npm install -g @slidev/cli @slidev/theme-default playwright-chromium
```

### Invoking the Skill

In Claude Code, provide a LaTeX source folder path and say "make a presentation from this paper" — the skill will trigger automatically.

Manual commands:

```bash
# In your project folder
slidev slides.md        # Start dev server
slidev build slides.md   # Build static site
slidev export slides.md  # Export as PDF
```

### Input

- arXiv paper LaTeX source (folder or zip archive)
- Contains main `.tex` file and `figures/` directory

### Output

```
output-folder/
├── slides.md              ← Main presentation file
├── figures/               ← Images (for md preview)
├── styles
```

## Skill File Structure

```
arxiv-to-slidev-en/
├── SKILL.md               ← Core instructions (English)
├── README.md              ← This file
```

## Installing the Skill

Place the `arxiv-to-slidev-en/` folder into Claude Code's skills directory, or reference it via `.claude/settings.json` in your project.
