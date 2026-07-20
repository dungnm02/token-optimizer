# Reasoning Effort Tuning (Per-Route Thinking Budgets)

**Addresses:** Cause 5.1 in [`../CAUSE.md`](../CAUSE.md)

**Idea:** Reasoning tokens are billed as output (the most expensive class)
whether or not they're displayed. Set the reasoning depth **per route** —
deep for genuinely hard work, shallow or off for routine traffic — instead
of one global setting.

---

## The provider dials

| Provider | Dial | Values / notes |
| --- | --- | --- |
| Anthropic | `output_config.effort` (+ adaptive thinking) | `low`/`medium`/`high`/`xhigh`/`max`; adaptive thinking decides *when* to think, effort decides *how much*. Note: on some current models thinking is on **by default** — cost appears without opt-in |
| OpenAI | `reasoning_effort` (o-series / GPT-5 family) | `minimal`/`low`/`medium`/`high`; reasoning tokens reported in `usage.completion_tokens_details.reasoning_tokens` |
| Gemini | `thinking_budget` / `thinkingConfig` | Token-count budget incl. `0` (off, where supported) and dynamic |
| Open models (R1-style) | Prompt/format control, budget forcing | Serving-level caps on thought length |

## How to apply

1. **Inventory routes by reasoning need.**

   | Route type | Setting |
   | --- | --- |
   | Classification, extraction, formatting, simple lookups | Reasoning off / `minimal` / `low` |
   | Standard chat, summarization | `low`–`medium` |
   | Coding, multi-step agentic work | `high` (sweep `xhigh` on evals) |
   | Correctness-critical, hardest problems | `high`+`xhigh`/`max` — measured, not reflexive |

2. **Sweep, don't guess.** Run your eval set across 2–3 effort levels and
   plot quality vs output tokens. The curve is not monotonic in cost:
   *higher effort up front often reduces total agentic cost* by finishing in
   fewer turns/tool calls — judge by **cost per completed task**
   (`token-counting.md`), never per-request output tokens alone.
3. **Route dynamically for mixed traffic.** A cheap complexity classifier
   (or the model-tier router from `model-routing.md`) assigns effort per
   request; simple queries take the shallow path.
4. **Steer thinking triggering with prompts** where the dial is coarse:
   adaptive-thinking models accept guidance like *"thinking adds latency
   and should only be used when it meaningfully improves answer quality —
   when in doubt, respond directly."*
5. **Leave headroom.** At high effort, size the output cap generously —
   reasoning shares the output budget, and a tight cap yields a truncated
   answer after expensive thinking (cause 5.3's worst case).
6. **Watch the hidden-reasoning gap** in telemetry: `output_tokens −
   visible_response_tokens` per route is the reasoning spend; alert on
   creep.

## SOTA tools

### Native — coding agents & provider APIs

| Provider / agent | Feature | Notes |
| --- | --- | --- |
| Anthropic API · Claude Code | `output_config.effort` + adaptive thinking; per-model effort selection in the harness | Effort decides *how much*, adaptive thinking decides *when* |
| OpenAI API · Codex CLI | `reasoning_effort` param; `model_reasoning_effort` in Codex config | Reasoning tokens reported in `usage.completion_tokens_details` |
| Google Gemini API · Gemini CLI | `thinkingConfig` / `thinking_budget` | Token-count budget incl. `0` (off, where supported) and dynamic |

### Third-party — agent-agnostic (open source preferred)

| Tool | License | Notes |
| --- | --- | --- |
| LiteLLM | MIT | Normalizes the effort/budget dials across providers so one route→effort config drives any backend |
| promptfoo / RAGAS-style eval harnesses | MIT / Apache-2.0 | Run the quality-vs-effort sweeps that make the map evidence-based |
| Langfuse / Helicone | MIT / Apache-2.0 | Track the hidden-reasoning gap (`output − visible`) per route and alert on creep |

## Trade-offs

- Under-provisioned reasoning on hard tasks costs more than it saves:
  wrong answers, extra correction turns, shallow tool use. Cut effort only
  where evals confirm quality holds.
- Per-route configuration is more surface to maintain; keep the route →
  effort map in config, versioned with evals.
- Provider semantics differ (a budget vs a level vs always-on) — the map
  needs re-tuning per provider/model migration.

## Expected impact

- Reasoning tokens are commonly **30–70% of output spend** on
  reasoning-enabled deployments; turning them off/down on routine routes
  removes most of that share for those routes at flat quality.
- OpenAI/Anthropic guidance and community benchmarks consistently show
  low-vs-high effort differing by **2–10× output tokens** on the same
  prompts — per-route assignment captures the spread.
- On genuinely hard agentic tasks, *raising* effort can cut **total** cost
  (fewer turns, fewer retries) — the sweep finds these inversions.
