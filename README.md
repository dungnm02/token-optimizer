# token-optimizer

A documentation repository for exploring **what causes high token consumption**
and **how to reduce it**. This repo is intentionally lightweight: it holds
notes, analysis, and reference material — not application code.

## Purpose

- Identify and document the root causes of high token consumption.
- Collect concrete, actionable solutions to reduce token usage.
- Serve as a shared knowledge base that grows as new findings emerge.

## Repository Layout

| Path | Description |
| --- | --- |
| `README.md` | This file — overview and conventions. |
| `CAUSE.md` | Catalog of identified causes of high token consumption. |
| `solutions/` | One document per solution / mitigation strategy. |

## How to Use

1. Read `CAUSE.md` to understand the known causes.
2. Browse `solutions/` for mitigations mapped to those causes.
3. When you discover something new, add it to `CAUSE.md` and, if you have a
   fix, add a matching document under `solutions/`.

## Contributing Notes

This repository is **documentation-only**. There is no build step, no tests,
and no application to run — just Markdown.

- **No branches required.** Work directly on `main`.
- **No pull requests required.** Commit and push straight to `main`.
- Keep every change focused on documentation quality and accuracy.

See [Commit Message Rules](#commit-message-rules) below before committing.

## Commit Message Rules

Because this repo is strictly for documentation, commit messages follow a
simple, consistent convention.

### Format

```
<type>: <short summary in imperative mood>

<optional body: what and why, wrapped at ~72 chars>
```

### Type

Use one of the following prefixes:

| Type | Use for |
| --- | --- |
| `docs` | Adding or editing documentation content (the default for this repo). |
| `cause` | Adding or updating an entry in `CAUSE.md`. |
| `solution` | Adding or updating a document under `solutions/`. |
| `chore` | Repo housekeeping (structure, renames, formatting, metadata). |

### Rules

1. **Subject line**
   - Use the imperative mood: "add", "fix", "clarify" — not "added" / "adds".
   - Keep it under ~72 characters.
   - Lowercase after the `type:` prefix; no trailing period.
2. **Scope** the message to a single logical change.
3. **Body** (optional) explains *what* changed and *why*, not *how*.
4. Reference the affected file when it adds clarity, e.g.
   `cause: document large-context re-sends in CAUSE.md`.

### Examples

```
docs: add repository overview and layout
cause: document redundant system-prompt re-sends
solution: add prompt-caching guide
chore: restructure solutions folder
```
