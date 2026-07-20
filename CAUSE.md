# Causes of High Token Consumption

This document catalogs the identified causes of high token consumption in
**LLM-based agents and applications**. The causes are provider-agnostic: they
apply to any stack (Claude, GPT, Gemini, open-weight models, etc.) because
they stem from how LLM APIs fundamentally work — stateless requests, priced
per token, with a bounded context window. Provider-specific details appear
only as illustrative examples.

Each cause describes **what it is**, **why it inflates token usage**, and
**how to recognize it**. Link to a matching document in `solutions/` when a
mitigation exists.

> Status: living document. The catalog below covers the major cause *types*,
> grouped by category. Individual entries may be expanded with deeper
> analysis and measurements over time.

## Background: why these causes are universal

Three properties shared by essentially all LLM APIs generate every cause in
this catalog:

1. **Statelessness.** The API keeps no memory between calls — the client
   re-sends everything the model should know on every request. Anything that
   grows the "everything" grows every future request.
2. **Per-token pricing, asymmetric.** Input and output are billed per token,
   with output typically 3–5× the input price, and cached input typically
   ~10–25% of the input price (where the provider offers caching).
3. **Bounded context.** The context window is finite; approaching it forces
   truncation, summarization, or failure — and everything inside it is billed
   on every call.

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
| 1 | [Caching failures](#1-caching-failures) | Paying full price for prefix tokens that could be served at cache-read rates |
| 2 | [Context accumulation](#2-context-accumulation) | Conversation history grows without bound and is re-sent every turn |
| 3 | [Tool usage patterns](#3-tool-usage-patterns) | Tool calls and results flood the context with low-value content |
| 4 | [Expensive content types](#4-expensive-content-types) | Images, documents, and retrieved data cost far more than expected |
| 5 | [Generation-side spend](#5-generation-side-spend) | Output tokens (reasoning + response) — the most expensive tokens — are overspent |
| 6 | [Architectural choices](#6-architectural-choices) | System design multiplies token cost across requests and agents |

---

## 1. Caching Failures

Most major providers offer some form of **prompt/prefix caching** (explicit
breakpoints, automatic prefix caching, or pre-registered cached content).
All of them share the same core mechanic: the cache matches a **prefix** of
the request, and cached tokens are billed at a steep discount. These causes
are about failing to earn that discount.

> **Prefix caching is not the only caching layer.** It makes a *repeated
> request* cheaper, but still runs the model. A separate, orthogonal waste
> is answering **near-identical requests repeatedly across a fleet** (the
> same support question, the same analytic query, the same eval prompt) —
> each one a full model call that a *response-level* (semantic) cache could
> skip entirely. That's a distinct opportunity from the prefix discount
> below; see `solutions/semantic-caching.md`.

### 1.1 No prompt caching at all

**Summary:** Requests with a large, stable prefix (system prompt, tool
definitions, shared documents) are sent without using the provider's caching
mechanism — or on a provider/config where caching isn't available.

**Why it consumes tokens:** The API is stateless, so the full prompt is
processed at full input price on every request. With cache reads typically
priced at ~10–25% of normal input, skipping caching means paying 4–10× more
for every repeated prefix, on every single call.

**How to recognize it:** The provider's usage metadata reports zero cached
tokens (e.g. `cache_read_input_tokens` on Anthropic, `cached_tokens` on
OpenAI) on every response, while raw input token counts stay large and
roughly constant across requests.

**Related solution(s):** _planned — `solutions/prompt-caching.md`_

### 1.2 Silent cache invalidators

**Summary:** Caching is configured (or automatic) but never hits, because
something mutates the prompt prefix on every request.

**Why it consumes tokens:** Prefix caching is a **byte-exact prefix match** —
a single changed byte invalidates everything after it. Common culprits:
timestamps or "current date" interpolated into the system prompt, request
IDs/UUIDs early in the content, JSON serialized with non-deterministic key
order, per-user data placed at the front, and conditional prompt sections.
The result is the worst of both worlds: any cache-write surcharge is paid on
every request with zero reads.

**How to recognize it:** Usage metadata shows cache *writes* (or simply no
cached tokens) on every request despite requests looking identical. Diffing
the fully rendered prompts of two consecutive requests exposes the mutating
fragment.

**Related solution(s):** _planned — `solutions/prompt-caching.md`_

### 1.3 Mid-session changes that rebuild the cache

**Summary:** Editing the system prompt, switching models, or adding/
removing/reordering tool definitions mid-conversation invalidates the entire
cached prefix.

**Why it consumes tokens:** Tool definitions and the system prompt render at
the very front of the request. Changing anything at the front forces the
whole conversation history behind it to be re-processed uncached — in a long
agentic session that can be hundreds of thousands of tokens re-billed at
full price in one request. Caches are also model-scoped everywhere: swapping
models mid-session always starts cold.

**How to recognize it:** A sudden spike of uncached input (and collapse of
cached input) mid-session, correlated with a config change: mode switch,
tool set rebuild, model swap, or a "small edit" to the system prompt.

**Related solution(s):** _planned — `solutions/prompt-caching.md`,
`solutions/stable-prompt-architecture.md`_

### 1.4 Cache lifetime expiry between requests

**Summary:** Bursty or infrequent traffic lets the cache expire between
requests (typical lifetimes range from a few minutes to ~1 hour depending on
provider and configuration), so each burst starts cold.

**Why it consumes tokens:** An expired entry must be re-processed — and on
providers with an explicit cache-write surcharge, re-written at a premium.
Traffic whose gaps exceed the cache lifetime never benefits from earlier
requests.

**How to recognize it:** Cache hits succeed within a burst, but the first
request after each idle gap shows a full uncached (or cache-write) pass.

**Related solution(s):** _planned — `solutions/prompt-caching.md`_ (longer
TTLs, pre-warming, keep-alive traffic)

---

## 2. Context Accumulation

> **This category is a quality problem, not only a cost problem.**
> Controlled studies ("context rot") across 18 frontier models find every
> one degrades as input grows — and well *before* the window fills:
> practical high-accuracy budgets land around 150–400K tokens even on
> 2M-token models, and accuracy falls fastest when the accumulated noise is
> *semantically similar* to the answer (exactly the case in a long coding
> session full of near-miss exploration). So trimming history (below) buys
> accuracy as well as tokens — the two motivations point the same way.

### 2.1 Unbounded conversation history

**Summary:** The full message history is re-sent on every turn (the API is
stateless) and nothing ever trims, summarizes, or expires it.

**Why it consumes tokens:** Input cost per turn grows roughly linearly with
conversation length, so total cost across a session grows **quadratically**.
A 100-turn agentic session can spend most of its budget re-sending turns
1–99. Caching softens this (cached history is cheap) but does not remove it,
and every cache miss re-bills the whole history at full price.

**How to recognize it:** Total prompt size (cached + uncached input) climbs
steadily turn over turn; late-session requests are 10–100× larger than early
ones; sessions eventually hit the context-window limit.

**Related solution(s):** _planned — `solutions/compaction.md`,
`solutions/context-editing.md`_

### 2.2 Stale tool results kept in history

**Summary:** Old tool outputs (file dumps, search results, command output)
remain in the transcript long after they stopped being relevant.

**Why it consumes tokens:** In agentic loops, tool results usually dominate
the transcript. A file read at turn 3 that was superseded at turn 20 is
still re-sent (and re-billed) on turns 21–100.

**How to recognize it:** Inspecting the transcript shows large tool-result
blocks whose content is duplicated or superseded later; the tool-result
share of the prompt keeps growing while the useful share doesn't.

**Related solution(s):** _planned — `solutions/context-editing.md`_
(pruning old tool results / reasoning blocks)

### 2.3 Duplicate context injection

**Summary:** The same content (a file, a schema, retrieved documents) is
injected into the conversation multiple times.

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

**Summary:** Tools return far more content than the model needs — whole
files instead of relevant sections, full API responses instead of the needed
fields, unpaginated listings.

**Why it consumes tokens:** Every byte of a tool result is input tokens on
the next request *and on every subsequent request* while it stays in
history. An unfiltered 5,000-line file read costs its tokens dozens of times
over a long session.

**How to recognize it:** Individual tool results in the tens of thousands of
tokens; tool results of which the model visibly uses only a small fraction.

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

**Related solution(s):** _planned — `solutions/tool-composition.md`_
(code-executed tool orchestration, server-side composition, batching)

### 3.3 Retry and polling loops

**Summary:** Failed tool calls retried with the same context, or agents
polling an external state ("check again in a loop") with a full model
request per poll.

**Why it consumes tokens:** Every retry/poll re-bills the entire prompt.
A 10-iteration poll loop on a 100K-token context spends 1M input tokens to
learn "not done yet" nine times.

**How to recognize it:** Bursts of near-identical requests in usage logs;
repeated errored tool results with unchanged inputs; sleep-and-check
patterns implemented *through* the model instead of *around* it.

**Related solution(s):** _planned — `solutions/event-driven-waiting.md`_

### 3.4 Too many tool schemas loaded upfront

**Summary:** Every tool definition the application owns is included in every
request, even though only a few are relevant per task.

**Why it consumes tokens:** Tool schemas are serialized into every request.
Hundreds of JSON-schema definitions can add tens of thousands of tokens of
fixed overhead per call — and on providers with prefix caching, any change
to the set invalidates the whole cache (cause 1.3). This is especially
common in MCP-style setups where entire tool servers are attached wholesale.

**How to recognize it:** A large gap between input tokens on a
tools-included request vs. the same request without tools; a tool list much
larger than the set actually invoked.

**Related solution(s):** _planned — `solutions/tool-search.md`_ (dynamic
tool discovery / deferred loading)

---

## 4. Expensive Content Types

### 4.1 Full-resolution images

**Summary:** Images sent at native/high resolution when the task doesn't
need the fidelity.

**Why it consumes tokens:** Vision inputs are tokenized by area/tiles on
every major provider, so token cost scales with resolution — a
full-resolution image can cost several thousand tokens where a downscaled
one costs a few hundred. Screenshot-heavy loops (computer/browser use)
multiply this per step, and images also persist in history (cause 2.1).

**How to recognize it:** Image-bearing requests with input token counts far
above the text length; per-step cost of computer-use loops dominated by the
screenshot.

**Related solution(s):** _planned — `solutions/image-downsampling.md`_

### 4.2 Whole-document dumping (PDF / RAG over-retrieval)

**Summary:** Entire documents or over-broad retrieval results are attached
when only a small slice is relevant.

**Why it consumes tokens:** A few-hundred-page PDF or a top-20 chunk
retrieval can add tens or hundreds of thousands of input tokens per request.
Without caching, that is re-billed per question; without pruning, it rides
in history forever.

**How to recognize it:** Q&A flows where every question re-sends the same
large document; retrieval configured for recall (large k, large chunks)
with answers that cite a tiny fraction of what was sent.

**Related solution(s):** _planned — `solutions/document-reuse.md`,
`solutions/retrieval-tuning.md`_

### 4.3 Tokenizer-expensive content

**Summary:** Content that tokenizes inefficiently — dense code, minified
JSON, base64 blobs, many non-English scripts — or a model/tokenizer change
that counts the same text higher.

**Why it consumes tokens:** Token counts are tokenizer-specific, and
tokenizers differ across providers *and across model generations within a
provider* (upgrades have shifted counts by 30%+ for identical text).
Verbatim logs, minified bundles, and base64 payloads are among the worst
offenders per unit of useful information — base64 alone inflates byte count
by ~33% before tokenization even starts.

**How to recognize it:** Budgets calibrated on one model suddenly truncating
or costing more after a model swap; the provider's token-counting endpoint
showing counts far above chars/4 heuristics on your actual content.

**Related solution(s):** _planned — `solutions/token-counting.md`_
(re-baseline with the provider's counter for the exact target model)

---

## 5. Generation-Side Spend

### 5.1 Reasoning/thinking tokens

**Summary:** Reasoning-capable models generate (billed) chain-of-thought
tokens before the visible answer — often hidden or summarized in the
response, but paid either way.

**Why it consumes tokens:** Output tokens are typically 3–5× the input
price, and reasoning is billed as output on every major provider. High
reasoning-effort settings applied uniformly spend deep reasoning on routes
that don't need it; some providers enable reasoning by default, so cost
appears without any opt-in.

**How to recognize it:** Reported output tokens far exceed the visible
response length; simple routes (classification, lookups) showing the same
output spend as complex ones.

**Related solution(s):** _planned — `solutions/reasoning-effort-tuning.md`_
(per-route effort/budget settings)

### 5.2 Output verbosity

**Summary:** The model produces more prose than needed — re-emitting whole
files instead of diffs, narrating every step, long preambles and recaps.

**Why it consumes tokens:** Output is the most expensive token class, and in
multi-turn sessions every verbose answer also becomes *input* on all
subsequent turns — verbosity is billed once as output and N times as
history.

**How to recognize it:** Responses that restate the question, re-print
unchanged code, or summarize what was just shown; output token counts
disproportionate to the information delivered.

**Related solution(s):** _planned — `solutions/concise-output-prompting.md`,
`solutions/diff-based-edits.md`_

### 5.3 Truncation-and-retry cycles

**Summary:** An output-token cap set too low truncates the response
mid-generation, and the application retries the whole request.

**Why it consumes tokens:** Each retry re-bills the full input plus the
wasted partial output. Two truncated attempts before a successful third cost
roughly 3× the input and ~2 answers' worth of discarded output. The same
dynamic applies to schema-validation failures on structured output — each
invalid generation triggers a full re-request.

**How to recognize it:** Length/max-token stop reasons in logs followed by
near-identical retry requests; structured-output parse failures feeding a
retry loop.

**Related solution(s):** _planned — `solutions/output-cap-sizing.md`_

---

## 6. Architectural Choices

### 6.1 Cold-start subagents

**Summary:** Subagents are spawned per subtask with no shared cache or
context handoff, each re-deriving what the parent already knows.

**Why it consumes tokens:** Every spawn re-sends (and often re-discovers via
tool calls) context the orchestrator already paid for. Fork calls that
rebuild the system prompt / tool list / model choice with any difference
also miss the parent's prefix cache entirely.

**How to recognize it:** Subagent transcripts that open with the same
exploration the parent did; fan-out patterns where N workers each pay a
full cold context; zero cache hits on fork requests despite a warm parent.

**Related solution(s):** _planned — `solutions/subagent-context-handoff.md`,
`solutions/prompt-caching.md`_ (forks must reuse the parent's exact prefix)

### 6.2 Oversized model for the task

**Summary:** Every route uses the largest model, including routes a smaller
tier would serve well.

**Why it consumes tokens:** Strictly a *cost* multiplier rather than a
token-count one, but it compounds every other cause: frontier-tier models
across providers cost roughly 5–25× more per token than their small-tier
siblings, so the same token waste costs that much more.

**How to recognize it:** Simple classification/extraction/formatting traffic
flowing through the top-tier model; no routing layer; no use of the
provider's discounted async/batch tier for latency-insensitive work.

**Related solution(s):** _planned — `solutions/model-routing.md`,
`solutions/batch-processing.md`_

### 6.3 Concurrent cold-cache fan-out

**Summary:** N parallel requests with an identical prefix are fired at once
before any of them has populated the cache.

**Why it consumes tokens:** A cache entry generally becomes readable only
after the first request has been processed. All N parallel requests pay
full input price — none can read what the others are still writing.

**How to recognize it:** Fan-out batches where every request reports zero
cached tokens despite sharing a prefix, while sequential runs of the same
requests show hits from the second one on.

**Related solution(s):** _planned — `solutions/fan-out-warming.md`_ (warm
with one request, then fire the rest)

### 6.4 Over-prescriptive prompts and scaffolding

**Summary:** System prompts carry legacy scaffolding — forced progress
updates, step-by-step procedures, "double-check X" verification loops, long
few-shot batteries — tuned for older or different models.

**Why it consumes tokens:** The scaffolding itself is fixed prompt overhead
on every call, and it induces extra output: forced narration, redundant
verification tool calls, and re-explanations the current model would not
otherwise produce. Prompts accreted across model generations rarely get
re-audited.

**How to recognize it:** Prompt sections that duplicate current model
defaults ("after every 3 tool calls, summarize progress"); output containing
ritualized sections nobody reads; few-shot examples for behaviors the model
now does zero-shot.

**Related solution(s):** _planned — `solutions/prompt-de-scaffolding.md`_

---

## Measurement Primer

To attribute cost to a cause, the usage metadata returned with each response
is the ground truth. Names differ per provider, but the same four quantities
exist almost everywhere:

| Quantity | Typical relative price | Example field names |
| --- | --- | --- |
| Uncached input tokens | 1× input | Anthropic `input_tokens`; OpenAI `prompt_tokens` minus `cached_tokens`; Gemini `prompt_token_count` minus `cached_content_token_count` |
| Cached input tokens | ~0.1–0.25× input | Anthropic `cache_read_input_tokens`; OpenAI `prompt_tokens_details.cached_tokens`; Gemini `cached_content_token_count` |
| Cache-write tokens (where surcharged) | ~1.25–2× input | Anthropic `cache_creation_input_tokens` (others bill writes at normal input rates) |
| Output tokens, **including hidden reasoning** | ~3–5× input | Anthropic `output_tokens`; OpenAI `completion_tokens` (+ `reasoning_tokens` detail); Gemini `candidates_token_count` + `thoughts_token_count` |

Two universal rules:

- **Total prompt size = uncached + cached (+ cache-write) input.** Judging
  spend from the uncached field alone under- or over-attributes cost.
- **Count tokens with the provider's own counter for the exact target
  model.** Tokenizers are model-specific; a counter borrowed from another
  provider (or another model generation) is systematically wrong.
