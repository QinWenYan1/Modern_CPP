# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Repository purpose

This is a **C++ learning notes repository** — structured Markdown study notes and exercise solutions organized by chapter. It is not a software project: there is no build system, no tests, and nothing to compile or run.

Content has two sources:

- **Chapters 1–16 (Parts 1–3: 基础 / 标准库 / 类和面向对象)** — notes for *C++ Primer (5th Edition)*, complete.
- **Chapter 17+ (Part 4: C++ 拓展)** — extension content beyond the textbook. Currently the dev toolchain (Makefile, GDB, static/dynamic libraries); *Effective Modern C++* is planned. These notes are self-curated from courses/videos, not tied to any book's section numbering.

## Content organization

```
ChapterNN_topic/
├── README.md                          # Chapter navigation with knowledge-point index
├── note_N_M_descriptive_name.md       # Per-section study notes
├── note_N_chapter_summary.md          # Chapter summary & exam quick-reference (textbook chapters)
└── exercises/                         # C++ exercise source files
    └── exercise_N_M.cpp               # Naming: exercise_<chapter>_<number>.cpp
```

- Chapters 1–16 are complete. Chapter 17 (`Chapter17_extension/`) is in progress — notes are being built incrementally from user-provided screenshots.
- Each chapter README links to its notes and to the previous/next chapter and root README.
- Note filenames use the pattern `note_<chapter>_<section>_<topic_slug>.md` (e.g., `note_7_1_defining_abstract_data_type.md`).
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

- Content often arrives as **batches of screenshots** saved to a folder in the repo (e.g., `Chapter17_extension/_screenshots/`).
- Screenshots may be PNG data with `.jpg` extensions — check with `file` and rename before reading. If the Read tool can't render an image, fall back to OCR: `tesseract <img> stdout -l chi_sim+eng` (Chinese + English language packs are installed via Homebrew).
- Workflow: generate a **draft note** in the chapter folder → user reviews and gives corrections → revise. Do a final global pass only when the user confirms the material is complete.
- Do not create the final chapter README navigation or update the root README until the chapter's notes are stable.

## Editing notes

When editing existing notes, preserve the template structure. Notes reference each other via relative links and the root README via `../README.md`. Chapter READMEs link to adjacent chapters with `← 上一章` / `→ 下一章` footer links.
