# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Repository purpose

This is a **C++ Primer (5th Edition) learning notes repository** — a collection of structured Markdown study notes and exercise solutions organized by chapter. It is not a software project: there is no build system, no tests, and nothing to compile or run.

## Content organization

```
ChapterNN_topic/
├── README.md                          # Chapter navigation with knowledge-point index
├── note_N_M_descriptive_name.md       # Per-section study notes
├── note_N_chapter_summary.md          # Chapter summary & exam quick-reference
└── exercises/                         # C++ exercise source files
    └── exercise_N_M.cpp               # Naming: exercise_<chapter>_<number>.cpp
```

- Chapters 1–16 are complete; Chapter 17 is an empty placeholder.
- Each chapter README links to its notes and to the previous/next chapter and root README.
- Note filenames use the pattern `note_<chapter>_<section>_<topic_slug>.md` (e.g., `note_7_1_defining_abstract_data_type.md`).
- Exercise filenames use the pattern `exercise_<chapter>_<number>.cpp`; grouped exercises use ranges (e.g., `exercise_12_6-7.cpp`).

## Note format

Every section note follows a strict template defined in the root [README.md](README.md):
- **Header**: chapter/section number, title, English original name
- **核心概念总览** (Core Concepts Overview): ordered list of knowledge points exactly matching the textbook's paragraph order — no logical reordering allowed
- **知识点详解** (Knowledge Point Details): each point has **Theory** (summary, technical accuracy, bold for key terms), **Textbook Code Examples** (verbatim copies), and **注意点** (warnings, tips, cross-references with emoji markers)
- **核心要点总结** (Key Points Summary): 3–5 distilled takeaways
- **考试速记版** (Exam Quick-Reference): condensed rules, comparison tables, common pitfalls, a mnemonic

Key principles enforced by the template:
- **Strict original-order fidelity** — no merging or reordering of textbook content
- **Bilingual terminology** — key terms include English originals (e.g., 迭代器(iterator))
- **All tables, code examples, and warnings** must be preserved

## Available skills for content generation

These skills are registered and can be invoked with `Skill`:

| Skill | Purpose |
|---|---|
| `course-notes-generator` | Generate a single section note from textbook content (PDF, images, URLs, transcripts). Produces one `note_N_M_*.md` file following the template above. |
| `chapter-nav-readme` | Generate a chapter-level `README.md` navigation page from a set of completed notes. |
| `course-root-readme` | Generate or update the root `README.md` with chapter navigation, repo structure, and embedded generation prompts. |
| `verify` | Bootstraps a project verify skill (not currently applicable — this is a docs repo). |

## Adding a new chapter or section

1. Create the chapter directory: `ChapterNN_topic/`
2. Use `course-notes-generator` with the textbook content to produce each section note.
3. Use `chapter-nav-readme` to generate the chapter `README.md`.
4. Add the chapter entry to the root `README.md` chapter navigation list (section under the appropriate part: 基础, 标准库, or 类和面向对象).
5. Add exercise `.cpp` files to `exercises/` following the `exercise_N_M.cpp` naming convention.

## Editing notes

When editing existing notes, preserve the template structure. Notes reference each other via relative links and the root README via `../README.md`. Chapter READMEs link to adjacent chapters with `← 上一章` / `→ 下一章` footer links.
