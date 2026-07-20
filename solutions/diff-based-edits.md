# Diff-Based Edits (Never Re-Emit the File)

**Addresses:** Cause 5.2 in [`../CAUSE.md`](../CAUSE.md) (coding-agent
specialization of output verbosity)

**Idea:** For file modification — the dominant output pattern of coding
agents — have the model emit only the **changed hunks** (search/replace
blocks, unified diffs, targeted edit-tool calls), never the whole rewritten
file.

---

## Why it matters

Rewriting a 1,000-line file to change 5 lines costs ~1,000 lines of output
(the most expensive token class) plus, in many harnesses, the rewritten file
re-entering context as input. Multiply by every edit in a session and
whole-file rewrite is often *the* dominant cost of a coding agent. It's also
slower (more tokens to generate = more latency) and riskier (accidental
drive-by changes to untouched code).

## How to apply

### 1. Pick an edit format the model is reliable at

| Format | Shape | Notes |
| --- | --- | --- |
| **Search/replace (str-replace)** | Exact `old_string` → `new_string`, uniqueness-checked | Used by Claude Code's `Edit` tool and Anthropic's text-editor tool (`str_replace_based_edit_tool`); highest reliability because the harness verifies the anchor exists exactly once |
| **Unified diff** | Standard `-`/`+` hunks | Aider's benchmark-driven finding: diff-style formats sharply reduce "lazy" elisions (`# ...rest unchanged`) vs whole-file on frontier models |
| **Line-range replace / insert** | Edit by line numbers | Simple, but brittle after concurrent edits — re-read before editing |
| **Whole file** | Full content | Only for new/small files (below ~100 lines the overhead beats format failure risk) |

### 2. Enforce it in the harness, not just the prompt

- Expose *edit tools* (`str_replace`, `apply_diff`) rather than a generic
  "write file" so partial edits are the path of least resistance.
- Validate anchors/hunks harness-side (exactly-one-match; reject stale
  context) and return actionable errors so a failed edit costs one small
  retry, not a whole-file fallback.
- Track file state (hash at last read) so the harness can reject edits
  against stale content instead of letting the model re-read + re-emit.

### 3. Use fast-apply models for fuzzy merges (optional)

A newer SOTA pattern: the frontier model emits a *loose* edit sketch, and a
small specialized "apply model" merges it into the file at thousands of
tokens/sec. This keeps frontier output minimal while tolerating imperfect
anchors.

### 4. Same principle beyond code

- Long documents: emit the revised section, not the whole document.
- Config/JSON state: emit a patch (JSON Patch / merge patch), not the full
  object.
- Conversational revisions: "change the second paragraph to…" responses
  should output the paragraph, not the essay.

## SOTA tools

### Native — coding agents & provider APIs

| Provider / agent | Feature | Notes |
| --- | --- | --- |
| Claude Code · Anthropic API | `Edit` tool / `text_editor_20250728` | Anchor-verified search/replace; the reference implementation |
| Codex CLI · OpenAI API | `apply_patch` edit format; predicted outputs | Codex's native diff format; predicted outputs speed/discount regeneration when whole-file output is unavoidable |
| Gemini CLI | `replace` (edit) tool | Anchor-based partial edits in the harness |

### Third-party — agent-agnostic (open source preferred)

| Tool | License | Notes |
| --- | --- | --- |
| Aider edit formats (`diff`, `udiff`) + polyglot benchmark | Apache-2.0 | Public benchmark data on edit-format reliability per model — use it to pick formats for any harness |
| tree-sitter-based patch validators | MIT | Syntax-check hunks before writing to disk, regardless of which agent emitted them |
| Morph / Relace fast-apply models | Commercial | Specialized high-speed merge of loose edits (~thousands of tokens/s) behind any frontier model |

## Trade-offs

- Edit formats fail sometimes (anchor not found, malformed hunk) — each
  failure costs a retry round trip. Reliability varies by model; benchmark
  (Aider's tables) before mandating a format.
- Search/replace needs unique anchors; highly repetitive files force larger
  context strings.
- Fast-apply adds a second model dependency and its own error modes.

## Expected impact

- Output tokens per edit drop roughly by the **file-size : hunk-size ratio**
  — commonly **10–50×** on real codebases (5–20 changed lines in
  500–2,000-line files).
- Latency per edit drops proportionally (output generation is the
  bottleneck), which compounds across an agent's dozens of edits per
  session.
- Secondary quality win: no drive-by regressions in untouched regions, and
  review diffs match intent.
