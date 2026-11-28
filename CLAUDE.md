# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 任务交接

从[仓库协作入口与技能索引](AGENTS.md#agent-handoff)开始，读取[项目现状与授权边界](agent/STATE.md)及[当前计划](agent/PLAN.md)。相关问题查[问题账本](agent/AUDIT.md)，第 17 章逐点材料查[来源对照](agent/sources/chapter17.md)。项目技能由入口按任务导航并显式读取。

---

## Repository purpose

This is a **C++ learning notes repository** — structured Markdown study notes and exercise solutions organized by chapter. It is not a software project: there is no build system, no tests, and nothing to compile or run.

Content has two sources:

- **Chapters 1–16 (Parts 1–3: 基础 / 标准库 / 类和面向对象)** — notes for *C++ Primer (5th Edition)*, complete.
- **第 17 章及以后（C++ 拓展）**。原第 17 章笔记参考《现代 C++ 教程：高速上手 C++》第 7 章。并发笔记的重组使用[当前已授权材料](agent/STATE.md)。第 18 章介绍静态库、动态库和 CMake。

## Content organization

```
ChapterNN_topic/
├── README.md                          # Chapter navigation with knowledge-point index
├── note_N_M_descriptive_name.md       # Per-section study notes
├── note_N_chapter_summary.md          # Chapter summary & exam quick-reference (textbook chapters)
└── exercises/                         # C++ exercise source files
    └── exercise_N_M.cpp               # Naming: exercise_<chapter>_<number>.cpp
```

- 第 1–16 章的基础笔记已完成。第 17 章原三篇和重组稿并存，交接位置见[项目状态](agent/STATE.md)。第 18 章按材料分步整理。
- Each chapter README links to its notes and to the previous/next chapter and root README.
- Derive each section note's filename from its H1 using `note_<chapter>_<section>_<full_english_title>.md`. Keep every word of the parenthesized English title in its original order and spelling, lowercase it, and use underscores between words. For example, `17.4 异步任务与结果传递 (Asynchronous Tasks and Futures)` maps to `note_17_4_asynchronous_tasks_and_futures.md`. Check the match before delivery.
- Exercise filenames use the pattern `exercise_<chapter>_<number>.cpp`; grouped exercises use ranges (e.g., `exercise_12_6-7.cpp`).

## Note format

Every section note follows a strict template defined in the root [README.md](README.md):
- **Header**: chapter/section number, title, English original name
- **核心概念总览** (Core Concepts Overview): ordered list of knowledge points with anchor links
- **知识点详解** (Knowledge Point Details): each point has a bold tagline, **Theory** (summary, technical accuracy, bold for key terms), **示例/实践** (verbatim code examples), and **注意点** (`>` blockquotes with ⚠️ warnings, 💡 tips, 🔄 cross-references, 📋 terminology)
- **核心要点总结** (Key Points Summary): 3–5 distilled takeaways
- **考试速记版** (Exam Quick-Reference): condensed rules, comparison tables, common pitfalls, a mnemonic

Key principles enforced by the template:
- **Bilingual terminology** — key terms include English originals (e.g., 迭代器(iterator))
- **All tables, code examples, and warnings** must be preserved

Ordering and merging policy differs by content source:
- **Textbook notes (Ch 1–16)**: strict original-order fidelity — no merging or reordering of textbook content.
- **Extension notes (Ch 17+)**: user prefers **merged knowledge points** (e.g., 概念+特点 as one point, 制作+使用 as one point), and the knowledge-point order must **match the original source material's order** — if a pedagogical reorder seems better, confirm with the user first.

## Available skills for content generation

These skills are registered and can be invoked with `Skill`:

| Skill | Purpose |
|---|---|
| `course-notes-generator` | Generate a single section note from learning content (PDF, images, URLs, transcripts). Produces one `note_N_M_*.md` file following the template above. |
| `chapter-nav-readme` | Generate a chapter-level `README.md` navigation page from a set of completed notes. |
| `course-root-readme` | Generate or update the root `README.md` with chapter navigation, repo structure, and embedded generation prompts. |

## Adding a new chapter or section

1. Create the chapter directory: `ChapterNN_topic/`
2. Use `course-notes-generator` with the source content to produce each section note.
3. Use `chapter-nav-readme` to generate the chapter `README.md`.
4. Add the chapter entry to the root `README.md` chapter navigation list (under the appropriate part: 基础, 标准库, 类和面向对象, or C++ 拓展).
5. Add exercise `.cpp` files to `exercises/` following the `exercise_N_M.cpp` naming convention.

## Working with user-provided source material

For extension chapters, the user provides source content incrementally and reviews drafts:

- Content often arrives as **batches of screenshots** saved to a folder in the repo (e.g., `Chapter18_extension/images/`).
- Screenshots may be PNG data with `.jpg` extensions — check with `file` and rename before reading. If the Read tool can't render an image, fall back to OCR: `tesseract <img> stdout -l chi_sim+eng` (Chinese + English language packs are installed via Homebrew).
- Workflow: generate a **draft note** in the chapter folder → user reviews and gives corrections → revise. Do a final global pass only when the user confirms the material is complete.
- Do not create the final chapter README navigation or update the root README until the chapter's notes are stable.

## Editing notes

When editing existing notes, preserve the template structure. Notes reference each other via relative links and the root README via `../README.md`. Chapter READMEs link to adjacent chapters with `← 上一章` / `→ 下一章` footer links.
