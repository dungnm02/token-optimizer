# Context Editing (Pruning Stale Turns)

**Addresses:** Causes 2.1 and 2.2 in [`../CAUSE.md`](../CAUSE.md)

**Idea:** Instead of (or before) summarizing, **delete** content from the
transcript that is provably stale — old tool results, superseded file reads,
completed reasoning blocks — while keeping the conversation's structural
skeleton intact. Pruning is cheaper than summarizing (no extra model call)
and lossless for content that genuinely no longer matters.

---

## Compaction vs. context editing

| | Context editing (prune) | Compaction (summarize) |
| --- | --- | --- |
| Mechanism | Remove/clear blocks | Replace span with generated summary |
| Extra model call | No | Yes |
| Information loss | Only what you choose to drop | Whatever the summary misses |
| Best target | Bulky, superseded tool results | Long narrative history near the window limit |
| Typical order | Continuously, throughout the session | When approaching the context budget |

They compose: prune continuously, compact when the pruned transcript still
approaches the limit.

## How to apply

1. **Provider-native (preferred where available)** — *Anthropic context
   management* (beta `context-management-2025-06-27`):

   ```json
   "context_management": {
     "edits": [
       {"type": "clear_tool_uses_20250919", "clear_tool_inputs": true},
       {"type": "clear_thinking_20251015"}
     ]
   }
   ```

   The API clears old tool results (and optionally the tool-call inputs) and
   spent thinking blocks based on configurable thresholds — no client
   bookkeeping.

2. **Harness-level pruning rules** (any provider):
   - *Supersession*: when a file is re-read or re-written, replace earlier
     reads of the same path with a stub: `"[content superseded — re-read if
     needed]"`.
   - *TTL by turn distance*: tool results older than K turns collapse to a
     one-line digest (`"grep: 14 matches in 3 files (pruned)"`).
   - *Keep the call, drop the payload*: preserve the `tool_use` /
     `tool_result` pairing (many APIs require matched IDs) but shrink the
     result content.
3. **Offload instead of inline**: have tools write large outputs to a file
   and return the path + a preview; the model reads slices on demand. This
   prevents the bloat rather than cleaning it up (see
   `tool-output-budgets.md`).
4. **Framework support**: LangGraph `trim_messages` / message-filtering
   hooks, LlamaIndex memory buffers with token limits, OpenAI Agents SDK
   session trimming.

## SOTA tools

### Native — coding agents & provider APIs

| Provider / agent | Feature | Notes |
| --- | --- | --- |
| Anthropic API | `context_management` edits (`clear_tool_uses`, `clear_thinking`) | Server-side clearing of stale tool results and spent thinking blocks; no client bookkeeping |
| Claude Code / Claude Agent SDK | Built-in stale-tool-result pruning (microcompaction) | Applied inside the harness automatically |
| OpenAI Agents SDK | Session trimming hooks | Part of the OpenAI/Codex stack |

### Third-party — agent-agnostic (open source preferred)

| Tool | License | Notes |
| --- | --- | --- |
| LangGraph `trim_messages` / message-filtering hooks | MIT | Composable pruning rules for custom loops on any provider |
| LlamaIndex token-limited memory buffers | MIT | Budget-enforced history for any backend |
| Supersession/TTL pruning rules (custom, ~100 LOC) | — | The harness-level rules in §2 are trivially implementable in any stack |

## Trade-offs

- Pruning heuristics can drop something the model still needed — the model
  will re-fetch (cheap if the tool is cheap, bad if it isn't). Prefer
  *stub-with-pointer* over silent deletion so recovery is one tool call.
- Editing the middle of the transcript invalidates the message-level cache
  from the edit point on (the head stays cached). Batch prunes rather than
  pruning every turn.
- Provider-native clearing gives less control over *what* is cleared than
  hand-rolled rules.

## Expected impact

- In tool-heavy agent transcripts, tool results are commonly **70–90% of
  history tokens**; pruning superseded ones typically shrinks steady-state
  context by **2–5×** with no summarization loss.
- Anthropic reports context editing extends effective task length
  substantially while *improving* quality on long tasks (stale context is
  distraction, not just cost).
- Zero additional model-call cost, unlike compaction.
