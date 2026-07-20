# Context Hygiene (Deduplicating Injection)

**Addresses:** Cause 2.3 in [`../CAUSE.md`](../CAUSE.md)

**Idea:** Guarantee that any given piece of content (a file, a schema, a
retrieved document) enters the conversation **at most once**, and is
referenced — not re-pasted — afterwards.

---

## How to apply

1. **Track what's already in context.** The harness keeps a registry of
   injected artifacts (path/URL/doc-ID → content hash + turn injected).
   Before injecting, check the registry:
   - Same hash already present → inject a reference instead: *"`schema.sql`
     is already in context (turn 4, unchanged)."*
   - Present but changed on disk → inject a **diff** against the in-context
     version, not the whole file.
2. **Fix retrieval pipelines that re-attach every turn.** Naive RAG glues
   the top-k chunks to *every* user message; in a session about one topic
   that's the same chunks re-billed each turn. Retrieve once per topic
   shift, keep results as a stable (cacheable) context block, and re-run
   retrieval only when the query drifts (embedding-distance gate).
3. **Reference by identifier where the provider supports it.** Upload once
   and reference by ID — Anthropic Files API `file_id`, OpenAI Files /
   vector stores, Gemini File API — instead of re-embedding the bytes into
   every request (see `document-reuse.md`).
4. **De-duplicate harness reminders.** System-reminder blocks, safety
   boilerplate, and schema recaps that the harness auto-injects should be
   injected once, or collapsed when repeated — audit what your framework
   adds per turn.
5. **Make tools idempotent-aware.** A `read_file` tool can answer from the
   registry ("unchanged since turn 4") instead of returning the full content
   again — the model gets its confirmation for ~10 tokens instead of 10,000.

## SOTA tools

| Tool | Scope | Notes |
| --- | --- | --- |
| Claude Code / Claude Agent SDK file-state tracking | Harness | Tracks read/edit state; discourages redundant re-reads ("file state is current in your context") |
| Anthropic Files API / OpenAI Files / Gemini File API | API | Upload-once, reference-by-ID |
| LlamaIndex / LangChain retrieval with query-similarity gating | Framework | Re-retrieve only on topic drift |
| Content-hash registry (custom, ~50 LOC) | Harness | The core mechanism; trivially implementable in any stack |

## Trade-offs

- The registry must be invalidated correctly when the underlying artifact
  changes — a stale "already in context" reference is a correctness bug, not
  just a token bug. Hash the current bytes, don't trust timestamps.
- Diff-injection assumes the model can apply diffs mentally; for heavily
  changed files, a fresh full read is safer.

## Expected impact

- Sessions with repeated file access commonly waste **20–40%** of history
  tokens on duplicate content; a hash registry eliminates nearly all of it.
- Retrieval dedup turns per-turn RAG cost from `k_chunks × turns` into
  `k_chunks × topic_shifts` — often a **5–20×** reduction on the retrieval
  share of input in long sessions.
