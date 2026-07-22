---
name: writing-style
description: Writing rules for all text forge produces, selected by context. The work context covers development output such as reports, PR bodies, commit messages, and READMEs. The creative context covers manuscript writing such as prose and poetry. Work is the default, and flags such as --creative override it. Use whenever writing output meant for a human reader.
---

# Writing Style

This skill defines the writing rules for every piece of text forge produces. A shared core applies everywhere. On top of it, one of two contexts applies, work or creative.

## Rules for all writing

These rules are not negotiable in either context. They govern every sentence you write, and they keep the writing human. A creative type, defined under Flags and Types below, may adapt how a rule is measured, never whether it binds.

- **No em dashes.** This covers the character itself, an en dash used as one, and a double hyphen standing in for either. Break the sentence into two sentences instead. Use a comma where a lighter pause fits.
- **No semicolons.** Break the sentence into two sentences instead. Use a comma where a lighter pause fits.
- **Cap sentences at twenty-two words.** Split any longer sentence into two or more shorter ones.

### What these rules do not touch

These rules govern prose you write. They never apply to material you reproduce. Quote the following verbatim, even when it breaks every rule above.

- Tool output, test failures, lint results, stack traces, and error messages.
- Code, config, diffs, and file contents.
- Command lines and their flags.
- Existing text you are citing, such as a commit message or an issue title.

Rewriting quoted material to fit the style corrupts the evidence. Put it in a fenced block and leave it alone. Your own prose around the block still follows every rule.

## Habits to avoid

These are the tells of machine-written text. They apply in full to the work context. In the creative context they yield to the Voice and Types rules below. That happens only where the project's existing prose or chosen form already establishes the pattern. Absent that evidence, avoid them in creative writing too.

- **No "not X, but Y" framing.** State what is true. Skip the contrast with what it is not.
- **No throat-clearing openers.** Cut phrases like "it is worth noting" and "let's dive in." Start with the content.
- **No closing paragraph that repeats what you just said.** Stop when the point is made.
- **No inflated stakes.** Skip words like crucial, vital, seamless, robust, and game-changing unless they are literally accurate.
- **No stacked hedges.** Pick one qualifier. "This may possibly cause" becomes "this may cause."
- **No rule-of-three padding.** List three items only when there are three items.

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

- **Lead with the answer.** Open each section with its conclusion, and let the supporting detail follow. Never build up to a point you could state first. When a command mandates its own order, such as a verdict that must come last, the command wins.
- **Use paragraph form.** Write narrative explanation as prose, and use backticks for file names, commands, and code. Numbered steps and checklists may stay as lists.
- **Scale headings to the artifact.** A commit body, a short PR description, or a two-paragraph report needs no headings. A README, a `TEST_PLAN.md`, or a multi-section report needs them. Never add a heading to organize three sentences.
- **Keep vocabulary consistent within a document.** Pick one word for a concept and reuse it instead of rotating synonyms. Use official or domain-specific terms only when the context requires them.
- **Match the target repo's spelling.** Read the existing README and docs, and follow the convention already there. Default to American English only when the repo has no prose to read.

Apply these rules to every paragraph and list item you write, not just headline text.

## The creative context

### Voice

The repo owns the voice. Read the existing chapters or sections first and match what is there. Match the narrative distance, the vocabulary register, and the rhythm of the existing prose. Match its spelling conventions too, including British or other regional spelling. Never impose a house style on a project that already has one. The mechanical rules under Rules for all writing still win when the existing prose breaks them. Apply them to new content and note the difference in the report. The habits under Habits to avoid yield to an established pattern in the existing prose. If the project has no content yet, ask the user for tone and audience before writing.

### Continuity

Check every new passage against what already exists. Names, dates, places, and established facts must stay consistent. Track what each character knows and when they learned it. A character cannot react to something they have not yet discovered. When new content forces a contradiction with earlier material, flag it in the report instead of silently rewriting history.

### Pacing

Vary sentence length to control rhythm. Short sentences carry tension. Longer ones carry reflection. Do not let every paragraph run the same length. Give the reader a beat between scenes instead of jumping without transition.

### Types

`--type-prose` is the default. The voice, continuity, and pacing rules above apply as written.

`--type-poetry` adapts the mechanics to verse. Line breaks and stanza breaks outrank sentence structure. Apply the twenty-two-word cap per line instead of per sentence. The em dash and semicolon bans still hold. The habits under Habits to avoid yield to the form. Match the form the project already uses, such as free verse, meter, or rhyme, before choosing one.

For any other type name, ask the user what rules that form needs instead of guessing.

## Before delivering

Scan what you wrote for the mechanical failures before the content leaves the command. These are the ones a reader notices first and the ones easiest to miss while drafting.

1. Search the prose for `—`, for `–`, for `--` standing in for a dash, and for `;` outside code. A flag name such as `--work` is not a dash, and a number range is not one either. Rewrite every real hit as two sentences or a comma.
2. Find the longest sentence. Split it if it runs past twenty-two words.
3. Confirm every fenced block holds quoted material. Prose moved into a block to dodge these rules does not count.

Run this check on your own output, not on files you only read.
