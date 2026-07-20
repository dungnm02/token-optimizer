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

**SOTA tools:** Two subsections, always in this order:
  - "Native — coding agents & provider APIs" — features built into the
    agents/vendors you already use (Claude Code/Anthropic, Codex/OpenAI,
    Gemini CLI/Google).
  - "Third-party — agent-agnostic (open source preferred)" — tools that
    work with *any* agent, with license noted; list open-source options
    first and name commercial alternatives inline.

**Trade-offs:** Costs, limitations, or risks to be aware of.

**Expected impact:** Rough sense of the token savings.
```

## Start Here

Not every solution is worth its setup cost for every system.
[`recommended-setup.md`](recommended-setup.md) distills the catalog into a
**necessary stack for a large codebase with many agents** — what to inherit
from the harness, the four pieces of custom work that matter, and what to
explicitly skip.

For concrete vendor instantiations, see
[`coding-setup-enterprise.md`](coding-setup-enterprise.md) — enterprise
coding-agent setups on **Claude, GPT, and Gemini**: access routes and their
feature trade-offs, harness choice, caching configuration, model/effort
maps, and mixed-fleet guidance.

For the harness we use day-to-day, see
[`coding-setup-cline.md`](coding-setup-cline.md) — the same tiering applied
to **Cline** (VS Code extension): provider/caching choice, Plan/Act model
split, `/smol`–`/newtask` context discipline, `.clinerules`/`.clineignore`/
MCP overhead trimming, and fleet telemetry via a gateway.

## Index

All 22 solutions are written, mapped to the causes in
[`../CAUSE.md`](../CAUSE.md). Each includes SOTA tool recommendations —
split into **agent/provider-native APIs** vs **third-party agent-agnostic
tools (open source preferred, licenses noted)** — and an expected-impact
assessment; several include mermaid diagrams of the mechanism.

### Category 1 — Caching failures

| Document | Addresses | Headline impact |
| --- | --- | --- |
| [`prompt-caching.md`](prompt-caching.md) | 1.1–1.4, 6.1 | Up to ~90% off cached input; 5–10× effective input reduction in long sessions |
| [`stable-prompt-architecture.md`](stable-prompt-architecture.md) | 1.3 | Makes mid-session cache rebuilds structurally impossible; sustains 80–95% cache-hit share |

### Category 2 — Context accumulation

| Document | Addresses | Headline impact |
| --- | --- | --- |
| [`compaction.md`](compaction.md) | 2.1 | Quadratic → linear session cost; 3–10× on long runs |
| [`context-editing.md`](context-editing.md) | 2.1, 2.2 | 2–5× steady-state context shrink with no summarization loss |
| [`context-hygiene.md`](context-hygiene.md) | 2.3 | Removes the 20–40% of history commonly wasted on duplicates |

### Category 3 — Tool usage patterns

| Document | Addresses | Headline impact |
| --- | --- | --- |
| [`tool-output-budgets.md`](tool-output-budgets.md) | 3.1 | 2–5× per-session input cut in tool-heavy workloads |
| [`tool-composition.md`](tool-composition.md) | 3.2 | K round-trips → ~1 context pass; intermediates never billed |
| [`event-driven-waiting.md`](event-driven-waiting.md) | 3.3 | Waiting cost O(polls × context) → ~0 |
| [`tool-search.md`](tool-search.md) | 3.4 | 10–50× reduction in per-request tool-schema overhead |

### Category 4 — Expensive content types

| Document | Addresses | Headline impact |
| --- | --- | --- |
| [`image-downsampling.md`](image-downsampling.md) | 4.1 | 3–5× vision input cut; 50–80% on screenshot loops |
| [`document-reuse.md`](document-reuse.md) | 4.2 | doc×questions → doc×1 + cheap cached reads |
| [`retrieval-tuning.md`](retrieval-tuning.md) | 4.2 | ~5× retrieval-share cut via reranked small-k; quality improves too |
| [`token-counting.md`](token-counting.md) | 4.3 (+ all) | The measurement layer — turns every other fix into an enforced invariant |

### Category 5 — Generation-side spend

| Document | Addresses | Headline impact |
| --- | --- | --- |
| [`reasoning-effort-tuning.md`](reasoning-effort-tuning.md) | 5.1 | 2–10× output-token spread captured by per-route effort |
| [`concise-output-prompting.md`](concise-output-prompting.md) | 5.2 | 30–60% output cut on chat/agent routes; 3–10× via structured output |
| [`diff-based-edits.md`](diff-based-edits.md) | 5.2 | 10–50× per-edit output cut for coding agents |
| [`output-cap-sizing.md`](output-cap-sizing.md) | 5.3 | Eliminates 2–3× truncate-retry waste; continuation salvages partials |

### Category 6 — Architectural choices

| Document | Addresses | Headline impact |
| --- | --- | --- |
| [`subagent-context-handoff.md`](subagent-context-handoff.md) | 6.1 | 50–90% of subagent re-discovery eliminated; artifact handoff keeps parents lean |
| [`model-routing.md`](model-routing.md) | 6.2 | 50–80% of volume moved to 5–25× cheaper tiers; RouteLLM/FrugalGPT-class results |
| [`batch-processing.md`](batch-processing.md) | 6.2 | Flat 2× on latency-insensitive traffic; 5–20× stacked with caching |
| [`fan-out-warming.md`](fan-out-warming.md) | 6.3 | N cold passes → 1 write + (N−1) cache reads (~85–90% off fan-out input) |
| [`prompt-de-scaffolding.md`](prompt-de-scaffolding.md) | 6.4 | 30–70% system-prompt cut + induced-output savings, often with quality gains |

### Suggested adoption order

1. **Measure first** — `token-counting.md` (you can't rank the rest without it)
2. **Free wins** — `prompt-caching.md` + `stable-prompt-architecture.md`, `batch-processing.md`
3. **Biggest structural levers** — `tool-output-budgets.md`, `compaction.md`/`context-editing.md`, `model-routing.md`
4. **Route-specific tuning** — the remaining docs, prioritized by what your telemetry attributes the spend to
