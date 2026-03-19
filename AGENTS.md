# OracleUniversity Repo Instructions

## Purpose

This repository is a Markdown-first study-notes repo built from Oracle University course source material, often including video transcripts.

The default task in this repo is not software development. It is turning rough transcript material into clean, readable, lesson-by-lesson study notes.

## Default Assumptions

- Treat pasted source text as material for a lesson note unless the user says otherwise.
- Prefer creating or updating one Markdown file per lesson.
- Follow the existing filename pattern in the target folder, such as `01_Course_Overview.md` or `04_Backup_and_Recovery_Configuration.md`.
- This repo is primarily documentation content. Do not spend time looking for build, test, or lint systems unless the user explicitly asks.
- Do not re-explore the whole repository on every task. Start with this file, then inspect only the target folder and one or two nearby examples if needed.

## Repo Shape

Common top-level content areas:

- `19cProCredential`
- `19cPL-SQLWorkshop`
- `ASMAdminWorkshop`
- `14cWeblogicServerAdmin1`

Common patterns inside course folders:

- Numbered lesson files
- `00_TABLE_OF_CONTENTS.md` in some courses
- `course-overview.md` in some courses
- Occasional `labs/` and `sops/` subfolders

## Writing Goal

Turn Oracle training transcripts into notes that are:

- technically accurate
- easy to scan
- useful for exam prep and review
- much more entertaining than the source material

## Voice And Tone

Use a witty, satirical, high-energy teaching voice with strong dry humor and sharp phrasing.

Target qualities:

- funny without becoming chaotic
- irreverent about boring technical material
- clear about what matters operationally
- willing to call out bad design, fragile setups, and failure modes
- readable enough that the humor helps retention instead of getting in the way

Aim for the general feel of sharp, satirical, high-energy explanatory comedy without directly imitating any specific living person.

## Lesson Structure

Default lesson format:

1. Lesson heading in this style:
   `## Lesson N - Title (short humorous subtitle)`
2. Short intro paragraph explaining what the lesson is about and why it matters
3. `By the end of this lesson, you should be able to:` followed by concise bullets
4. Major sections with numbered `##` headings or `## 1.`, `## 2.`, etc. depending on the existing local pattern
5. Bullets for concepts, risks, parameters, views, and takeaways
6. Numbered steps for procedures
7. Fenced code blocks for SQL, RMAN, shell, or other commands
8. A short wrap-up section at the end

Use `---` between major sections when it matches nearby files.

## Transcript Conversion Rules

- Remove filler, repetition, false starts, and presenter housekeeping.
- Preserve the real technical meaning.
- Keep Oracle terminology, parameter names, dynamic performance views, and commands exact.
- Convert rambling spoken explanations into structured notes.
- Highlight operational consequences: what breaks, what fills up, what fails over, what must be multiplexed, what requires downtime, and what actually matters in practice.
- If the source material mixes lecture and demo content, organize it cleanly instead of preserving the original disorder.
- If something in the source material is unclear, mark it carefully rather than inventing facts.
- Do not mention transcripts, transcript segments, or source-material mechanics inside the finished lesson notes. Present them as polished standalone study notes.

## Formatting Preferences

- Use clear Markdown headings and compact bullets.
- Use inline code for parameters, views, commands, and filenames.
- Keep lesson files self-contained.
- Prefer strong section subtitles and occasional parenthetical jokes.
- Avoid giant walls of text.

## Editing Behavior

- Match the conventions already present in the target course folder.
- Read nearby lessons before introducing a new pattern.
- Preserve existing humor-forward phrasing when editing older notes.
- When adding a brand-new lesson, keep it consistent with the folder's existing numbering and naming scheme.

## Priority Order

When there is a tradeoff, prioritize:

1. Technical accuracy
2. Study usefulness
3. Readability
4. Humor

Humor is important here, but not at the expense of correctness.
