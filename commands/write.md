---
description: Write long-form prose for a writing project in this repo, such as fiction, nonfiction, essays, or worldbuilding documents. Adapts to the project's existing voice and structure.
argument-hint: "[chapter outline, scene description, section topic, or revision request]"
---

# /forge:write

You are writing long-form prose for a writing project in this repo. `/forge:write` works for fiction, nonfiction, commentary, essays, worldbuilding documents, fanfiction, technical books, and any other writing project structured in Git. It makes no assumptions about genre, tone, or subject matter. It adapts to the project by reading the repo's existing content and any project-level guidance files.

The Git repo is the workspace, not the publishing destination. The final output may go to Amazon KDP, Gumroad, a personal site, or any other platform. `/forge:write` only manages the writing and revision process. Publishing is a separate concern, handled outside this command.

The task is: $ARGUMENTS

If `$ARGUMENTS` is empty, ask the user what to write next before doing anything else.

## Orientation

Before writing anything, orient yourself in this repo, the same way `/forge:do-work` orients in a codebase.

- Look for `CLAUDE.md`, `README.md`, or any file describing the project's subject, tone, audience, structure, or conventions. Treat these as editorial guidelines. Follow them the way `/forge:do-work` follows coding conventions.
- Read existing chapters, sections, or documents in the repo. Look for the voice, the pacing, the argument style, the narrative tone, and the terminology already established. Match them. Do not introduce a new voice or style that clashes with what exists.
- Look for an outline file, a table of contents, or a chapter plan. Use it to see where the current task fits in the larger structure. Check for continuity with what comes before and what comes after.
- If the repo has a project-specific skill file, load it and follow it. Examples include a worldbuilding bible, a style guide, a character reference, or a theological framework.

## Workflow

### 1. Restate the task

Restate the task in your own words. Confirm what you are writing, where it fits in the project, and any constraints from the project's guidelines. State your assumptions. Keep this brief.

### 2. Create a branch

Create a branch following the same convention as `/forge:do-work`. Use `feat` for new content, `fix` for corrections or continuity fixes, `refactor` for restructuring, and `docs` for metadata or outline updates.

### 3. Write the content

Write the content. Follow the `writing-style` skill in this plugin for the prose. Match the voice and conventions you found during orientation. Produce real, complete prose. Do not leave placeholder paragraphs or notes like "TODO: expand this section."

### 4. Commit

Commit using the `conventional-commits` skill in this plugin. Scope each commit logically. For example, commit one section or one scene at a time, instead of one giant commit for an entire chapter.

### 5. Review your own work

Review your own work before you report. Check for continuity with existing content. Check for tone consistency. Check that the new content fits the outline or structural plan, if one exists. Flag anything you are uncertain about.

### 6. Report

Finish with a concise report. Follow the `writing-style` skill in this plugin for the report's prose. Cover what was written, where it fits in the project, and any continuity concerns or decisions you made. Note that the user should run `/forge:review` or read the diff before shipping.

## What this command does not do

- It does not push or open a PR. That is `/forge:ship`'s job.
- It does not define the project's voice, genre, or subject matter. The repo does that, through its existing content and guidance files. `/forge:write` adapts to what it finds.
- It does not generate outlines or project plans, unless the user explicitly asks for one as the task. Its default mode is producing prose, not planning prose.
- It does not handle publishing or formatting for any platform. The repo is the workspace. Getting the final manuscript onto KDP, Gumroad, or anywhere else is outside this command's scope.
