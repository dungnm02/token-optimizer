# Concise Output Prompting

**Addresses:** Cause 5.2 in [`../CAUSE.md`](../CAUSE.md) (with `diff-based-edits.md`)

**Idea:** Output is the most expensive token class *and* becomes input on
every later turn. Engineer the output contract — via prompts, structured
formats, verbosity parameters, and stop conditions — so the model delivers
the information, not the ceremony.

---

## How to apply

### 1. Give an explicit output contract in the system prompt

The reliable levers, in order of effect:

- **Lead-with-the-answer + selectivity framing**: *"Lead with the outcome.
  Keep output short by being selective about what you include — drop
  details that don't change what the reader does next — not by compressing
  the writing into fragments."* (Selectivity instructions beat "be concise",
  which models often satisfy by truncating style, not content.)
- **Ban the ceremony explicitly**: no preamble ("Certainly! Here is…"), no
  restating the question, no closing recap, no narrating routine actions in
  agent loops (*"default to silence between tool calls; one sentence when
  something changes"*).
- **Positive examples > prohibitions**: one or two examples of the desired
  answer shape outperform lists of don'ts.
- **Calibrate, don't minimize, where length is the product** — for reports
  and explanations, specify the *target shape* ("≤5 bullets, one line
  each") rather than a vague "short".

### 2. Use structured output as a length constraint

A JSON schema (provider structured-output modes) is the strongest verbosity
control available: the model can only emit the fields you defined. For
extraction/classification/routing routes, schema-constrained output
eliminates prose entirely — often cutting output 3–10× vs free text.

### 3. Use native verbosity/length parameters where offered

- OpenAI GPT-5 family: `verbosity: low|medium|high` — dials length without
  prompt surgery.
- Output caps (`max_tokens`) as a *backstop*, not a primary control —
  truncation is failure, not conciseness (`output-cap-sizing.md`).
- `stop` sequences to terminate known-terminal formats (e.g. end-of-JSON).

### 4. De-scaffold the prompts that *cause* verbosity

Forced progress summaries, "explain your reasoning" left over from
pre-reasoning-model days, mandatory section templates — audit and remove
what current models do adequately unprompted (`prompt-de-scaffolding.md`).

### 5. Watch the multi-turn echo

In agent loops, every verbose answer is re-billed as history each later
turn. Concision instructions therefore pay quadratically in long sessions —
prioritize them for your longest-running agents.

## SOTA tools

### Native — coding agents & provider APIs

| Provider / agent | Feature | Notes |
| --- | --- | --- |
| Anthropic API | Structured outputs (`output_config.format`) | Hard ceiling on output shape; the strongest lever for non-prose routes |
| OpenAI API | Structured outputs + `verbosity` parameter | Native length dial on supported models |
| Google Gemini API | `responseSchema` | Schema-constrained output |
| Claude Code | Output styles / `CLAUDE.md` conventions | Persistent output contract for the harness without per-prompt surgery |

### Third-party — agent-agnostic (open source preferred)

| Tool | License | Notes |
| --- | --- | --- |
| Caveman (`wilpel/caveman-compression`) | MIT | Semantic output compression — strips predictable grammar, keeps facts. Ships as a prompt skill usable across Claude Code, Codex, Gemini CLI, Cursor, Cline and other agents (skill variant claims up to ~75% output cut; the library's measured modes run 15–58%). Internal/agent traffic only — not for user-facing prose where tone matters |
| Instructor / Zod-typed outputs | MIT | Typed schemas that double as verbosity contracts, portable across providers |
| promptfoo / Langfuse evals | MIT | Regression-test that concision edits don't cut *content*; track output tokens per route; Braintrust is the commercial option |

## Trade-offs

- Over-tight contracts lose genuinely useful explanation — measure task
  success, not just token counts, when tightening.
- Aggressive terseness harms readability for human consumers; "selective,
  not compressed" is the framing that avoids fragment-speak.
- Style instructions drift across model generations (some models are
  naturally terser/wordier) — re-baseline on migration.

## Expected impact

- Structured-output conversion on extraction/routing routes: **3–10×**
  output reduction, plus parsing reliability.
- System-prompt output contracts on chat/agent routes typically cut output
  tokens **30–60%** at equal task success (measured via evals).
- In long agent sessions the same cut compounds through history: 40% less
  output ≈ 40% less *future input* from those turns as well.
