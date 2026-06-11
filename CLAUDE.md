# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file HTML web app (`companion.html`) — a reading companion for six philosophy papers by Jan Plate. Visitors ask questions; the app routes to the right paper(s) and answers using the Anthropic API, grounding responses in the full paper text.

There is no build system, no framework, no server. Open the file in a browser (or serve it statically) and it works. CDN dependencies: KaTeX 0.16.9, marked.js 12.0.0, DOMPurify 3.0.8.

## Architecture

**Corpus storage** — The full text of all six papers is embedded in the file as `PAYLOAD`: a gzip-compressed, base64-encoded JSON blob (line 313). This is decoded at runtime via `DecompressionStream`. To regenerate the payload after editing paper content, re-compress the JSON and re-encode it.

**Paper metadata** — `PAPER_TOCS` (lines ~320–490) is the human-editable table of contents for each paper. Section titles may contain KaTeX math (`$...$`). `EXAMPLES` holds the four starter questions shown on load.

**Two-stage routing** — `routePapers(question)` first tries an LLM call (`api()`) that returns JSON `{"papers": ["id1"]}` or `{"papers": ["id1","id2"]}`. On failure or zero score it falls back to `keywordRoute()`, a simple title/keyword scorer. If the user has pinned papers, routing is bypassed entirely.

**Answering** — `buildSystem(ids)` constructs the system prompt, injecting the full paper XML and the table of contents. `answer()` calls `api()` with the accumulated `history` (capped at 12 turns). The assistant is asked to end each reply with a `SOURCES:` line that `renderAnswerHTML()` strips out and formats separately.

**Rendering** — Markdown via `marked.parse`, math via `renderMathInElement` (KaTeX auto-render), sanitized with `DOMPurify`. Math is stashed and re-injected around the markdown pass to prevent mangling.

## Deployment

This file is a **Claude artifact** created with Claude Fable 5, intended to be run on claude.ai. When opened there, claude.ai automatically supplies the user's API key, which is why the `api()` function has no `x-api-key` header. Users need a Claude account. The model is `claude-sonnet-4-20250514`.

## Theming

CSS custom properties handle light/dark/auto. The `[data-theme]` attribute on `<html>` overrides `prefers-color-scheme`. A `?theme=light|dark` query param sets the initial theme. Token names mirror `jplate.github.io/home`.

## Paper IDs

`simple`, `intrinsic`, `qualitative`, `quovadis`, `ott`, `roles` — used throughout as keys in `PAPER_TOCS`, in the routing system prompt catalog, and in `SOURCES:` lines.
