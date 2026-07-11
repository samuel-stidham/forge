---
name: writing-style
description: Writing rules for all text forge produces, selected by context. The work context covers development output such as reports, PR bodies, commit messages, and READMEs. The creative context covers manuscript writing such as prose and poetry. Work is the default, and flags such as --creative override it. Use whenever writing output meant for a human reader.
---

# Writing Style

This skill defines the writing rules for every piece of text forge produces. A shared core applies everywhere. On top of it, one of two contexts applies, work or creative.

## Rules for all writing

These rules are not negotiable in either context. They keep the writing human. A creative type, defined under Flags and Types below, may adapt how a rule is measured, never whether it applies.

- **No em dashes.** Break the sentence into two sentences instead. Use a comma where a lighter pause fits.
- **No semicolons.** Break the sentence into two sentences instead. Use a comma where a lighter pause fits.
- **Cap sentences at twenty-two words.** Split any longer sentence into two or more shorter ones.

## Choosing the context

The work context is the default. It covers everything forge writes while doing development work. That includes reports, PR descriptions, commit messages, README content, and inline documentation.

The creative context covers manuscript content in a writing project. That includes chapters, scenes, essays, poems, and worldbuilding documents. `/forge:write` and `/forge:publish` apply it to manuscript content automatically. Even inside those commands, reports, commit messages, and PR descriptions stay in the work context.

### Flags

A command accepts these flags when its main content goes beyond development artifacts. A flag overrides the automatic choice for that main content. Development artifacts, such as reports, commit messages, and PR descriptions, always stay in the work context regardless of flags.

- `--work`: force the work context. This matches the default, so it exists to override a command's automatic creative choice.
- `--creative`: force the creative context.
- `--type-{name}`: refine the creative context with a form. `--type-prose` is the default and never needs to be passed. `--type-poetry` switches to the poetry rules. A type flag implies `--creative` on its own.

When `--work` appears alongside `--creative` or a type flag, `--work` wins and the others are ignored. Strip these flags from the arguments before interpreting the rest of the task.

## The work context

- **Use paragraph form.** Group prose under headings, and use backticks for file names, commands, and code.
- **Keep vocabulary consistent within a document.** Pick one word for a concept and reuse it instead of rotating synonyms. Use official or domain-specific terms only when the context requires them.

Apply these rules to every paragraph and list item you write, not just headline text. Numbered steps and checklists may stay as lists. Narrative explanation should read as paragraphs under headings.

## The creative context

### Voice

The repo owns the voice. Read the existing chapters or sections first and match what is there. Match the narrative distance, the vocabulary register, and the rhythm of the existing prose. Never impose a house style on a project that already has one. The shared rules above still win when the existing prose breaks them. Apply them to new content and note the difference in the report. If the project has no content yet, ask the user for tone and audience before writing.

### Continuity

Check every new passage against what already exists. Names, dates, places, and established facts must stay consistent. Track what each character knows and when they learned it. A character cannot react to something they have not yet discovered. When new content forces a contradiction with earlier material, flag it in the report instead of silently rewriting history.

### Pacing

Vary sentence length to control rhythm. Short sentences carry tension. Longer ones carry reflection. Do not let every paragraph run the same length. Give the reader a beat between scenes instead of jumping without transition.

### Types

`--type-prose` is the default. The voice, continuity, and pacing rules above apply as written.

`--type-poetry` adapts the mechanics to verse. Line breaks and stanza breaks outrank sentence structure. Apply the twenty-two-word cap per line instead of per sentence. The em dash and semicolon bans still hold. Match the form the project already uses, such as free verse, meter, or rhyme, before choosing one.

For any other type name, ask the user what rules that form needs instead of guessing.
