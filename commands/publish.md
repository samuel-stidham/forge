---
description: Compile a writing project into publishable ebook formats, such as EPUB for Amazon KDP and Barnes and Noble Nook. Builds local files only and does not upload.
argument-hint: "[optional target format or platform, e.g. \"kindle\", \"nook\", or \"epub\"]"
---

# /forge:publish

You are compiling a writing project in this repo into publishable ebook formats. `/forge:publish` is the prose equivalent of `/forge:ship`. `/forge:ship` pushes code and opens a PR. `/forge:publish` builds the manuscript into files ready for upload to publishing platforms.

`/forge:publish` targets Amazon KDP and Barnes and Noble Nook. It handles manuscript assembly, metadata, formatting, and file generation. It does not handle the upload. Uploading needs logging into each platform, setting pricing, and completing platform-specific forms. That is a manual step the user handles.

The requested format or platform is: $ARGUMENTS

If `$ARGUMENTS` is empty, generate all supported formats.

## Orientation

Before building anything, read the repo to understand the manuscript structure.

- Look for `CLAUDE.md`, `README.md`, or a metadata file that defines the book's title, author name, description, language, and any ISBNs. If these are missing, ask the user for them before proceeding.
- Identify the chapter ordering. Look for a table of contents file, an outline, numbered filenames, or a manifest that defines the reading order. If the order is ambiguous, ask instead of guessing.
- Look for a cover image file. KDP and Nook both expect one. If none exists, note it in the report. Do not generate one.
- Check for front matter, such as a dedication, a preface, or acknowledgments. Check for back matter, such as an appendix, a bibliography, or an about-the-author page. Include them in the correct position if they exist.

## Supported output formats

- **EPUB** (`.epub`): the standard ebook format. Nook accepts EPUB directly. KDP also accepts EPUB. This is the primary output format.
- **KDP-ready EPUB**: an EPUB file validated against KDP's specific requirements. This covers their metadata expectations, their table of contents structure, and their cover image embedding rules.
- **MOBI** (`.mobi`): the legacy Kindle format. Only generate this when the user explicitly asks for it. KDP now accepts EPUB directly, so MOBI is rarely needed.

## Workflow

Follow the `token-budget` skill in this plugin for session management. Default to Opus for this command.

1. Read the manuscript files in order. Assemble them into a single logical document, following the chapter ordering found during orientation.
2. Apply basic typographic formatting. This includes chapter headings, paragraph spacing, and italics and bold preserved from the markdown source. It also includes proper front and back matter placement.
3. Generate the EPUB file, using Pandoc or a similar tool available in the environment. Embed the cover image if one exists. Include metadata such as title, author, language, and description.
4. Validate the generated EPUB. Use `epubcheck` if it is available. Report any validation errors, and fix what can be fixed automatically.
5. Place the output files in a `dist/` or `build/` directory at the repo root. Do not commit them. Add `dist/` and `build/` to `.gitignore` if they are not already there.
6. Report what was generated, where the files are, the file sizes, and any issues found during validation. Include a checklist of what the user needs to do next for each platform.
   - For KDP: log into kdp.amazon.com, create or update a title, upload the EPUB, and set pricing.
   - For Nook: log into press.barnesandnoble.com, create or update a title, upload the EPUB, and set pricing.

## What this command does not do

- It does not upload to any platform. That needs authentication and account decisions only the user can make.
- It does not set pricing, categories, or keywords. Those are business decisions, not manuscript concerns.
- It does not generate cover art. Cover design is a separate discipline.
- It does not commit or push the generated files. Build artifacts belong in `dist/` or `build/` and stay out of version control.

## Dependencies

This command needs Pandoc, or a similar EPUB generation tool, installed on the machine. If Pandoc is not found, tell the user how to install it for their platform, instead of failing silently. If `epubcheck` is available, use it for validation. If it is not, skip validation and note that the user should validate manually before uploading.
