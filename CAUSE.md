# Causes of High Token Consumption

This document catalogs the identified causes of high token consumption. Each
cause describes **what it is**, **why it inflates token usage**, and
**how to recognize it**. Link to a matching document in `solutions/` when a
mitigation exists.

> Status: living document. The catalog below covers the major cause *types*,
> grouped by category. Individual entries may be expanded with deeper analysis
> and measurements over time.

## Entry Template

Copy this template when adding a new cause.

```
### <Cause name>

**Summary:** One-sentence description of the cause.

**Why it consumes tokens:** Explain the mechanism that drives up token usage.

**How to recognize it:** Symptoms, metrics, or patterns that indicate this
cause is present.

**Related solution(s):** Link to the relevant file(s) under `solutions/`.
```

---

## Catalog

The causes are grouped into six categories:

| # | Category | Core problem |
| --- | --- | --- |
| 1 | [Caching failures](#1-caching-failures) | Paying full price for tokens that could be ~0.1× via cache reads |
| 2 | [Context accumulation](#2-context-accumulation) | Conversation history grows without bound and is re-sent every turn |
| 3 | [Tool usage patterns](#3-tool-usage-patterns) | Tool calls and results flood the context with low-value content |
| 4 | [Expensive content types](#4-expensive-content-types) | Images, PDFs, and retrieved documents cost far more than expected |
| 5 | [Generation-side spend](#5-generation-side-spend) | Output tokens (thinking + response) — the most expensive tokens — are overspent |
| 6 | [Architectural choices](#6-architectural-choices) | System design multiplies token cost across requests and agents |

---

## 1. Caching Failures

### 1.1 No prompt caching at all

**Summary:** Requests with a large, stable prefix (system prompt, tool
definitions, shared documents) are sent without any `cache_control`
breakpoints.

**Why it consumes tokens:** The API is stateless — the full prompt is
processed at full input price on every request. Cache reads cost ~0.1× the
base input price; skipping caching means paying ~10× more for every repeated
prefix, on every single call.

**How to recognize it:** `usage.cache_read_input_tokens` and
`usage.cache_creation_input_tokens` are both `0` on every response while
`input_tokens` stays large and roughly constant across requests.

**Related solution(s):** _planned — `solutions/prompt-caching.md`_

### 1.2 Silent cache invalidators

**Summary:** Caching is configured but never hits, because something mutates
the prompt prefix on every request.

**Why it consumes tokens:** Prompt caching is a **prefix match** — a single
byte change anywhere in the prefix invalidates everything after it. Common
culprits: timestamps (`datetime.now()` / `Date.now()`) interpolated into the
system prompt, UUIDs / request IDs early in content, JSON serialized without
sorted keys, per-user data in the system prompt, and conditional prompt
sections. Every request then pays the ~1.25× cache-*write* premium with zero
reads — strictly worse than no caching.

**How to recognize it:** `cache_creation_input_tokens` is large on *every*
request while `cache_read_input_tokens` stays `0` across identical-looking
requests. Diffing the rendered prompt bytes of two consecutive requests
exposes the mutating fragment.

**Related solution(s):** _planned — `solutions/prompt-caching.md`_

### 1.3 Mid-session changes that rebuild the cache

**Summary:** Editing the system prompt, switching models, or adding/removing/
reordering tools mid-conversation invalidates the entire cached prefix.

**Why it consumes tokens:** Tools render at position 0 of the prompt, system
comes next, then messages. Changing anything at the front forces the whole
conversation history to be re-processed uncached — in a long agentic session
that can be hundreds of thousands of tokens re-billed at full price in one
request.

**How to recognize it:** A sudden spike of `cache_creation_input_tokens`
(and drop of `cache_read_input_tokens`) mid-session, correlated with a
config change: mode switch, tool set rebuild, model swap, or a "small edit"
to the system prompt.

**Related solution(s):** _planned — `solutions/prompt-caching.md`,
`solutions/stable-prompt-architecture.md`_

### 1.4 Cache TTL expiry between requests

**Summary:** Bursty or infrequent traffic lets the cache (default 5-minute
TTL) expire between requests, so each burst re-pays the cache write.

**Why it consumes tokens:** A cache entry that expired must be rewritten at
~1.25× (5m TTL) or 2× (1h TTL) the base input price. Traffic with gaps longer
than the TTL never benefits from earlier writes.

**How to recognize it:** Cache reads succeed within a burst but the first
request after each idle gap shows a full `cache_creation_input_tokens` write.

**Related solution(s):** _planned — `solutions/prompt-caching.md`_ (1-hour
TTL, pre-warming)

---

## 2. Context Accumulation

### 2.1 Unbounded conversation history

**Summary:** The full message history is re-sent on every turn (the API is
stateless) and nothing ever trims, summarizes, or expires it.

**Why it consumes tokens:** Input cost per turn grows roughly linearly with
conversation length, so total cost across a session grows **quadratically**.
A 100-turn agentic session can spend most of its budget re-sending turns
1–99. Caching softens this (cached history is ~0.1×) but does not remove it,
and every cache miss re-bills the whole history at full price.

**How to recognize it:** `input_tokens + cache_read_input_tokens` climbs
steadily turn over turn; late-session requests are 10–100× larger than early
ones; sessions eventually hit `model_context_window_exceeded`.

**Related solution(s):** _planned — `solutions/compaction.md`,
`solutions/context-editing.md`_

### 2.2 Stale tool results kept in history

**Summary:** Old tool outputs (file dumps, search results, command output)
remain in the transcript long after they stopped being relevant.

**Why it consumes tokens:** In agentic loops, tool results usually dominate
the transcript. A file read at turn 3 that was superseded at turn 20 is still
re-sent (and re-billed) on turns 21–100.

**How to recognize it:** Inspecting the transcript shows large
`tool_result` blocks whose content is duplicated or superseded later; the
tool-result share of the prompt keeps growing while the useful share doesn't.

**Related solution(s):** _planned — `solutions/context-editing.md`_
(`clear_tool_uses` strategy)

### 2.3 Duplicate context injection

**Summary:** The same content (a file, a schema, retrieved docs) is injected
into the conversation multiple times.

**Why it consumes tokens:** Each injection is billed independently, and each
copy then rides along in the history for the rest of the session (cause 2.1
compounds it).

**How to recognize it:** Repeated re-reads of the same file across turns;
retrieval pipelines that re-attach the same top-k documents on every user
message; system-level content pasted into user turns "to make sure the model
sees it".

**Related solution(s):** _planned — `solutions/context-hygiene.md`_

---

## 3. Tool Usage Patterns

### 3.1 Oversized tool outputs

**Summary:** Tools return far more content than the model needs — whole files
instead of relevant sections, full API responses instead of the needed
fields, unpaginated listings.

**Why it consumes tokens:** Every byte of a tool result is input tokens on
the next request *and on every subsequent request* while it stays in history.
An unfiltered 5,000-line file read costs its tokens dozens of times over a
long session.

**How to recognize it:** Individual `tool_result` blocks in the tens of
thousands of tokens; tool results that the model visibly uses only a small
fraction of.

**Related solution(s):** _planned — `solutions/tool-output-budgets.md`_

### 3.2 Chatty round-trips instead of composition

**Summary:** Many sequential tool calls where each intermediate result flows
through the model's context, even though the model only needs the final
answer.

**Why it consumes tokens:** Each round trip re-sends the growing history and
lands another intermediate result in it. Three chained lookups
(profile → orders → inventory) cost three full context passes, and the
intermediate data is usually never needed again.

**How to recognize it:** Long chains of small tool calls per user request;
transcripts full of intermediate data the final answer doesn't reference.

**Related solution(s):** _planned — `solutions/programmatic-tool-calling.md`,
`solutions/batching.md`_

### 3.3 Retry and polling loops

**Summary:** Failed tool calls retried with the same context, or agents
polling an external state ("check again in a loop") with a full model request
per poll.

**Why it consumes tokens:** Every retry/poll re-bills the entire prompt.
A 10-iteration poll loop on a 100K-token context spends 1M input tokens to
learn "not done yet" nine times.

**How to recognize it:** Bursts of near-identical requests in usage logs;
repeated `is_error: true` tool results with unchanged inputs; sleep-and-check
patterns implemented through the model instead of around it.

**Related solution(s):** _planned — `solutions/event-driven-waiting.md`_

### 3.4 Too many tool schemas loaded upfront

**Summary:** Every tool definition the application owns is included in every
request, even though only a few are relevant per task.

**Why it consumes tokens:** Tool schemas render at the very front of the
prompt on every request. Hundreds of JSON-schema definitions can add tens of
thousands of tokens of fixed overhead — and any change to the set invalidates
the whole cache (cause 1.3).

**How to recognize it:** A large gap between `input_tokens` on a
tools-included request vs. the same request without tools; a `tools` array
much larger than the set actually invoked.

**Related solution(s):** _planned — `solutions/tool-search.md`_ (deferred
loading), `solutions/prompt-caching.md`_

---

## 4. Expensive Content Types

### 4.1 Full-resolution images

**Summary:** Images sent at native/high resolution when the task doesn't need
the fidelity.

**Why it consumes tokens:** On high-res-capable models (2576px long edge), a
full-resolution image can cost up to ~4,784 tokens — roughly 3× the older
~1,600-token cap. Screenshot-heavy loops (computer use) multiply this per
step, and images also persist in history (cause 2.1).

**How to recognize it:** Image-bearing requests with input token counts far
above the text length; per-step cost of computer-use loops dominated by the
screenshot.

**Related solution(s):** _planned — `solutions/image-downsampling.md`_
(1080p / 720p guidance for computer use)

### 4.2 Whole-document dumping (PDF / RAG over-retrieval)

**Summary:** Entire PDFs or over-broad retrieval results are attached when
only a small slice is relevant.

**Why it consumes tokens:** A few-hundred-page PDF or a top-20 chunk
retrieval can add tens or hundreds of thousands of input tokens per request.
Without caching, that is re-billed per question; without pruning, it rides in
history forever.

**How to recognize it:** Q&A flows where every question re-sends the same
large document; retrieval configured for recall (large k, large chunks) with
answers that cite a tiny fraction of what was sent.

**Related solution(s):** _planned — `solutions/files-api-and-caching.md`,
`solutions/retrieval-tuning.md`_

### 4.3 Tokenizer-expensive content

**Summary:** Content that tokenizes inefficiently — dense code, minified
JSON, base64 blobs, some non-English text — or a model whose tokenizer counts
the same text higher.

**Why it consumes tokens:** Token counts are model-specific. Newer
tokenizers (e.g. the Opus 4.7-family tokenizer) count the same text up to
~1.35× higher than earlier ones; Sonnet 5's tokenizer ~30% higher than Sonnet
4.6's. Verbatim logs, minified bundles, and base64 payloads are among the
worst offenders per unit of useful information.

**How to recognize it:** Budgets calibrated on an older model suddenly
truncating or costing more after a model upgrade; `count_tokens` on a sample
showing token counts far above chars/4 heuristics.

**Related solution(s):** _planned — `solutions/token-counting.md`_
(re-baseline with `count_tokens`, never `tiktoken`)

---

## 5. Generation-Side Spend

### 5.1 Thinking tokens

**Summary:** Extended/adaptive thinking generates (billed) reasoning tokens
before the visible answer — invisible when `display` is `"omitted"`, but paid
either way.

**Why it consumes tokens:** Output tokens are ~5× the price of input tokens.
High effort settings on hard-or-easy-alike routes spend deep reasoning on
tasks that don't need it. Thinking is billed identically whether or not it is
displayed.

**How to recognize it:** `usage.output_tokens` far exceeds the visible
response length; simple routes (classification, lookups) showing the same
output spend as complex ones.

**Related solution(s):** _planned — `solutions/effort-tuning.md`_ (per-route
`output_config.effort`)

### 5.2 Output verbosity

**Summary:** The model produces more prose than needed — re-emitting whole
files instead of diffs, narrating every step, long preambles and recaps.

**Why it consumes tokens:** Output is the most expensive token class, and in
multi-turn sessions every verbose answer also becomes *input* on all
subsequent turns — verbosity is billed once as output and N times as history.

**How to recognize it:** Responses that restate the question, re-print
unchanged code, or summarize what was just shown; output token counts
disproportionate to the information delivered.

**Related solution(s):** _planned — `solutions/concise-output-prompting.md`,
`solutions/diff-based-edits.md`_

### 5.3 Truncation-and-retry cycles

**Summary:** `max_tokens` set too low causes `stop_reason: "max_tokens"`
truncation, and the application retries the whole request.

**Why it consumes tokens:** Each retry re-bills the full input plus the
wasted partial output. Two truncated attempts before a successful third cost
roughly 3× the input and ~2 answers' worth of discarded output.

**How to recognize it:** `stop_reason: "max_tokens"` in logs followed by
near-identical retry requests.

**Related solution(s):** _planned — `solutions/max-tokens-sizing.md`_

---

## 6. Architectural Choices

### 6.1 Cold-start subagents

**Summary:** Subagents are spawned per subtask with no shared cache or
context, each re-deriving what the parent already knows.

**Why it consumes tokens:** Every spawn re-sends (and often re-discovers via
tool calls) context the orchestrator already paid for. Fork calls that
rebuild `system`/`tools`/`model` with any difference also miss the parent's
prompt cache entirely.

**How to recognize it:** Subagent transcripts that open with the same
exploration the parent did; fan-out patterns where N workers each pay a full
cold context; zero cache reads on fork requests despite a warm parent.

**Related solution(s):** _planned — `solutions/subagent-context-handoff.md`,
`solutions/prompt-caching.md`_ (fork must reuse the parent's exact prefix)

### 6.2 Oversized model for the task

**Summary:** Every route uses the largest model, including routes a smaller
tier would serve well.

**Why it consumes tokens:** This is strictly a *cost* multiplier rather than
a token-count one, but it compounds every other cause: the same token waste
costs 5–10× more per token on a frontier model ($10/$50 per MTok on Fable 5
vs. $1/$5 on Haiku 4.5).

**How to recognize it:** Simple classification/extraction/formatting traffic
flowing through the top-tier model; no routing layer.

**Related solution(s):** _planned — `solutions/model-routing.md`,
`solutions/batch-api.md`_ (50% discount for async work)

### 6.3 Concurrent cold-cache fan-out

**Summary:** N parallel requests with an identical prefix are fired at once
before any of them has written the cache.

**Why it consumes tokens:** A cache entry becomes readable only after the
first response begins streaming. All N parallel requests pay full input
price — none can read what the others are still writing.

**How to recognize it:** Fan-out batches where every request reports
`cache_creation_input_tokens > 0` and `cache_read_input_tokens: 0` despite
sharing a prefix.

**Related solution(s):** _planned — `solutions/fan-out-warming.md`_ (send 1,
await first token, then fire N−1)

### 6.4 Over-prescriptive prompts and scaffolding

**Summary:** System prompts carry legacy scaffolding — forced progress
updates, step-by-step procedures, "double-check X" verification loops —
tuned for older models.

**Why it consumes tokens:** The scaffolding itself is fixed prompt overhead,
and it induces extra output: forced narration, redundant verification tool
calls, and re-explanations the current model would not otherwise produce.

**How to recognize it:** Prompt sections that duplicate current model
defaults ("after every 3 tool calls, summarize progress"); output containing
ritualized sections nobody reads.

**Related solution(s):** _planned — `solutions/prompt-de-scaffolding.md`_

---

## Measurement Primer

To attribute cost to a cause, the response `usage` object is the ground
truth:

| Field | Meaning | Relative price |
| --- | --- | --- |
| `input_tokens` | Uncached prompt tokens | 1× input |
| `cache_read_input_tokens` | Prompt tokens served from cache | ~0.1× input |
| `cache_creation_input_tokens` | Prompt tokens written to cache | ~1.25× (5m) / 2× (1h) input |
| `output_tokens` | Generated tokens, **including thinking** | ~5× input |

Total prompt size = `input_tokens + cache_read_input_tokens +
cache_creation_input_tokens`. For pre-flight measurement, use the
`count_tokens` API endpoint with the exact target model — token counts are
model-specific, and third-party estimators (e.g. `tiktoken`) are wrong for
Claude.
