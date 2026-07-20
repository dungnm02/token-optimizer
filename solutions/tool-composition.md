# Tool Composition (Code-Executed Orchestration)

**Addresses:** Cause 3.2 in [`../CAUSE.md`](../CAUSE.md)

**Idea:** Replace chains of model-mediated tool round-trips with a single
step in which the model **writes code that composes the tools**; the code
runs in a sandbox, intermediate results flow between tool calls inside the
runtime, and only the final distilled answer returns to the model's context.

---

## Why it wins

```mermaid
sequenceDiagram
    participant M as Model
    participant T as Tools
    rect rgb(255, 235, 235)
    Note over M,T: ❌ Round-trip style — 3 full context passes
    M->>T: get_profile(user)
    T-->>M: 4K-token profile → lands in context
    M->>T: get_orders(profile.id)
    T-->>M: 12K-token order list → lands in context
    M->>T: check_inventory(order_ids)
    T-->>M: 6K-token result → lands in context
    end
    rect rgb(235, 255, 235)
    Note over M,T: ✅ Composed — 1 pass, intermediates never billed
    M->>T: run_code("p=get_profile(u); o=get_orders(p.id);<br/>print(summarize(check_inventory(o)))")
    T-->>M: 300-token final answer
    end
```

Every round-trip re-sends the entire (growing) history; composition sends it
once. Intermediate payloads (often the bulk of the data) never enter the
context at all — they live and die inside the sandbox.

## How to apply

1. **Provider-native programmatic tool calling** — *Anthropic PTC*: declare
   the code-execution tool and mark your custom tools with
   `allowed_callers: ["code_execution_20260120"]`; Claude writes a script
   that calls your tools as functions inside the sandbox, and only the
   script's stdout returns to context. *OpenAI*: code interpreter +
   function-calling composition on the Responses API.
2. **Code-as-action agent frameworks** — Hugging Face **smolagents**
   (`CodeAgent`) makes code the *default* action format: instead of emitting
   one JSON tool call per step, the model emits a Python snippet that can
   loop, branch, and call several tools. The CodeAct research line behind
   this reports up to ~30% fewer steps than JSON-tool-call agents on
   multi-tool benchmarks — fewer steps = fewer full-context passes.
3. **Deterministic pre-composition** — if the chain is *always* the same
   (profile → orders → inventory), it isn't an LLM decision at all: fuse it
   into one tool (`get_user_order_status`) in the harness. The cheapest
   token is the one the model never orchestrates.
4. **Batch independent calls in one turn** — when composition isn't
   available, at minimum exploit parallel tool use: N independent calls in
   one assistant turn + all results in one user turn is one context pass
   instead of N.
5. **Filter inside the sandbox** — the composed script should end with a
   `summarize`/`select` step so the return value is the answer, not the raw
   data (pair with `tool-output-budgets.md`).

## SOTA tools

| Tool | Scope | Notes |
| --- | --- | --- |
| Anthropic programmatic tool calling (`code_execution` + `allowed_callers`) | API | Intermediates return to the running code, not to context; token cost scales with final output |
| Hugging Face smolagents `CodeAgent` | Framework | Code-as-action; multi-call snippets per step; ~30% fewer steps reported vs JSON tool-calling |
| OpenAI code interpreter + function calling | API | Sandbox-side composition on the Responses API |
| LangGraph | Framework | Express deterministic chains as graph edges (code), reserving the model for genuine decisions |
| E2B / Modal / Daytona sandboxes | Infra | Self-hosted execution environments for code-as-action agents |

## Trade-offs

- Requires a sandbox — infrastructure, security review, and (for hosted
  options) execution pricing.
- Code-writing itself costs output tokens; for a single trivial call, plain
  tool use is cheaper. Composition pays off from ~3 chained calls or any
  large intermediate.
- Harder-to-inspect failures: an error inside the composed script needs the
  script + traceback surfaced well, or the model burns turns debugging
  blind.
- Provider-native PTC restricts some features (e.g. incompatible with strict
  schemas / forced tool choice on some stacks) — check the current matrix.

## Expected impact

- Token cost of a K-step chain drops from **K full context passes to ~1**;
  with a 100K-token context and K=5, that's ~500K input tokens → ~100K.
- Intermediate results (often 10–100× the size of the final answer) are
  removed from context *permanently* — compounding with history persistence.
- Step-count reductions (~30% in CodeAct-style evaluations) also cut
  latency and error surface, not just tokens.
