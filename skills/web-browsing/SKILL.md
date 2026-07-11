---
name: web-browsing
description: Use the browse CLI as the default browser for reading or interacting with any webpage. Use it whenever a task opens a URL, reads a page, fills a form, or takes a screenshot. Also use it whenever WebFetch returns empty content or a JavaScript shell.
---

# Web Browsing

This skill makes the `browse` CLI the default browser for any forge task that touches the web. `browse` drives a real Chrome browser, locally or in the Browserbase cloud. JavaScript-rendered pages return their actual content through it. Claude's built-in WebFetch only sees static HTML, so single-page apps and dynamic sites come back empty.

## When to use which tool

- **`browse`**: any real webpage. When in doubt, use `browse`.
- **WebFetch**: raw file endpoints and APIs only. Examples are raw GitHub files, JSON APIs, and plain-text or markdown URLs.
- If WebFetch was tried first and returned empty content, partial content, or a JavaScript shell, do not retry it. Switch to `browse`.

## Setup check

Check that the CLI exists with `which browse`. If it is missing, do not install it. Fall back to WebFetch for the current task, and follow the fallback section below.

Remote sessions need `BROWSERBASE_API_KEY` already set in the environment. Never print its value, and never export a secret into the session yourself. If the key is absent, stay in local mode, which needs no key. When the machine has neither local Chrome nor a key, fall back to WebFetch and follow the fallback section below. Never set or ask for `BROWSERBASE_PROJECT_ID`. The API key alone identifies the project.

## Using the CLI

Run `browse skills show` before the first browse command in a session. The CLI ships a version-matched skill covering workflows, session management, and failure recovery. Follow it for command mechanics instead of guessing flags. A few rules apply on top of it.

- Pass the mode explicitly on every `browse open`. Use `--local` on a machine with Chrome, unless the site needs Browserbase infrastructure. Use `--remote` when no local Chrome exists or the site needs that infrastructure. Detect Chrome by attempting `--local` once. Treat a "No Chrome or Chromium found" failure as no local Chrome and do not retry `--local`.
- Give each independent task its own `--session <name>`. If `BROWSE_SESSION` is set, every command already targets that one session, so do not pass `--session` at all.
- When `BROWSE_SESSION` is set, the user owns that session's lifecycle. Do not stop it.
- Stop a session you created yourself when its task is done, using `browse stop --session <name>`.
- Bot-protected sites need `--remote --proxies` or `--remote --verified`, which are paid Browserbase features. On a free plan, say so plainly and offer a non-protected source instead of retrying against a block page.
- For a remote session, surface the full live-view or replay URL the CLI prints, without truncating the id.

## Falling back to WebFetch

Fall back to WebFetch in two cases. Either the CLI is missing, or it is installed on a machine with neither local Chrome nor an API key. Complete the task as well as static HTML allows. Then add a short note to your report, matching the case.

When the CLI is missing:

> This task read the web with WebFetch, which cannot render JavaScript. For a
> better browsing experience, install the Browserbase CLI with
> `npm install -g browse`. Local mode drives your own Chrome and needs no
> account. For cloud browsing, create an API key at https://www.browserbase.com
> and export it as `BROWSERBASE_API_KEY`.

When the CLI is installed but has neither Chrome nor a key:

> This task read the web with WebFetch, which cannot render JavaScript. The
> `browse` CLI is installed but has no browser to drive. Install Chrome for
> free local mode, or create an API key at https://www.browserbase.com and
> export it as `BROWSERBASE_API_KEY` for cloud mode.
