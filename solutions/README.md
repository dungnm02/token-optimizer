# Solutions

This folder holds one document per solution or mitigation strategy for reducing
token consumption. Each solution should map back to one or more causes listed in
[`../CAUSE.md`](../CAUSE.md).

## Adding a Solution

1. Create a new Markdown file in this folder named after the strategy, e.g.
   `prompt-caching.md` or `context-trimming.md`.
2. Follow the entry template below so every solution is consistent.
3. Link the solution from the relevant cause in `../CAUSE.md`.

## Solution Template

```
# <Solution name>

**Addresses:** Which cause(s) from CAUSE.md this solves.

**Idea:** Short description of the approach.

**How to apply:** Concrete, actionable steps.

**Trade-offs:** Costs, limitations, or risks to be aware of.

**Expected impact:** Rough sense of the token savings.
```

## Index

_No solutions written yet. The following documents are planned, referenced
from the causes in [`../CAUSE.md`](../CAUSE.md):_

| Planned document | Addresses (cause #) |
| --- | --- |
| `prompt-caching.md` | 1.1, 1.2, 1.3, 1.4, 3.4, 6.1 |
| `stable-prompt-architecture.md` | 1.3 |
| `compaction.md` | 2.1 |
| `context-editing.md` | 2.1, 2.2 |
| `context-hygiene.md` | 2.3 |
| `tool-output-budgets.md` | 3.1 |
| `programmatic-tool-calling.md` | 3.2 |
| `batching.md` / `batch-api.md` | 3.2, 6.2 |
| `event-driven-waiting.md` | 3.3 |
| `tool-search.md` | 3.4 |
| `image-downsampling.md` | 4.1 |
| `files-api-and-caching.md` | 4.2 |
| `retrieval-tuning.md` | 4.2 |
| `token-counting.md` | 4.3 |
| `effort-tuning.md` | 5.1 |
| `concise-output-prompting.md` | 5.2 |
| `diff-based-edits.md` | 5.2 |
| `max-tokens-sizing.md` | 5.3 |
| `subagent-context-handoff.md` | 6.1 |
| `model-routing.md` | 6.2 |
| `fan-out-warming.md` | 6.3 |
| `prompt-de-scaffolding.md` | 6.4 |
