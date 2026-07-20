# Retrieval Tuning (Precision Over Recall-Dumping)

**Addresses:** Cause 4.2 in [`../CAUSE.md`](../CAUSE.md) (with `document-reuse.md`)

**Idea:** Make the retrieval stage do its job — deliver a *small, precise*
context — instead of shifting the filtering burden onto the model by dumping
a large top-k. Every irrelevant chunk is paid for on this request and every
subsequent turn it survives in.

---

## How to apply

### 1. Add a reranking stage

First-stage retrieval (BM25/embeddings) is recall-oriented; rerank the
candidates and send only the survivors:

```
retrieve top-50 (cheap) → rerank → send top-3..5 (precise)
```

Cross-encoder rerankers are the single highest-leverage retrieval upgrade:
they let you cut k by 3–5× at equal-or-better answer quality.

### 2. Right-size k and chunks — measured, not guessed

- Sweep k on your eval set; answer quality typically plateaus at k=3–8 for
  well-reranked pipelines while cost grows linearly with k.
- Chunk size: smaller chunks (200–500 tokens) with **contextual headers**
  (title/section breadcrumbs, or generated context à la Anthropic's
  contextual-retrieval recipe) beat big 2K-token chunks — you send the
  relevant paragraph, not its whole neighborhood.
- Deduplicate near-identical chunks (common with overlapping windows) before
  sending.

### 3. Retrieve when needed, not always

- **Gate retrieval**: classify whether the turn needs retrieval at all
  (many turns are follow-ups answerable from context). A cheap classifier
  or embedding-similarity gate ("query drifted from what's already in
  context?") prevents reflexive per-turn attachment — see
  `context-hygiene.md`.
- **Agentic retrieval**: give the model a `search(query)` tool and let it
  pull what it decides it needs (1–2 targeted calls), instead of
  pre-stuffing every request. This converts fixed per-turn retrieval cost
  into on-demand cost.

### 4. Compress what you send

- Send extractive snippets (the matching passage ± a sentence) rather than
  full chunks where the task allows.
- Strip markup/boilerplate from chunks at indexing time, not at query time.
- Context-compression rerankers (e.g. LLMLingua-style) can squeeze retrieved
  context 2–5× for QA tasks, at some fidelity risk — eval before adopting.

## SOTA tools

### Native — coding agents & provider APIs

| Provider / agent | Feature | Notes |
| --- | --- | --- |
| Anthropic | Contextual retrieval recipe | Chunk-context generation at indexing time; published ~49% retrieval-failure reduction (67% with reranking) |
| OpenAI API | `text-embedding-3` + vector stores / file search | Managed first-stage retrieval inside the OpenAI stack |
| Coding agents' repo search (Claude Code `Grep`/`Glob`, Codex/Gemini CLI search tools) | Agentic retrieval | For codebases, bounded on-demand search often beats a pre-stuffed RAG pipeline entirely |

### Third-party — agent-agnostic (open source preferred)

| Tool | License | Notes |
| --- | --- | --- |
| BGE-reranker-v2 / mxbai-rerank | Apache-2.0 | Self-hosted SOTA-class cross-encoder reranking; Cohere Rerank 3.5 / Voyage rerank-2 / Jina Reranker v2 are hosted commercial equivalents |
| BGE-M3 embeddings | MIT | Strong open first-stage recall; Voyage-3 is the hosted commercial option |
| LLMLingua-2 | MIT | Prompt/context compression 2–5× for QA-style tasks — eval before adopting |
| RAGAS / promptfoo | Apache-2.0 / MIT | Measure the k/chunk/rerank sweeps instead of guessing; Braintrust is the commercial alternative |

## Trade-offs

- Reranking adds a hop (~50–300ms) and a per-query fee — almost always
  repaid by the smaller k.
- Aggressive precision tuning risks recall failures on multi-hop questions;
  keep an escape hatch (agentic follow-up retrieval).
- Compression techniques are task-sensitive — validate on your evals, not
  benchmarks alone.

## Expected impact

- k=20 unranked → k=4 reranked cuts the retrieval share of input **~5×**
  per request, compounding across every turn the chunks would have persisted.
- Anthropic's contextual-retrieval numbers (−49% retrieval failures, −67%
  with reranking) show precision tuning *improves quality while cutting
  tokens* — irrelevant chunks were hurting both.
- Retrieval gating typically eliminates 30–70% of retrieval attachments in
  conversational sessions (most turns are follow-ups).
