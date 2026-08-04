# Install Runbook — Agent

Machine-readable runbook for installing token-optimization tools in a
**restricted workspace** (no plugin marketplace, no extension install, no
`npm -g`, no admin rights).

> English-only and terse by design. This file is consumed by an agent, not a
> human — a bilingual copy would double the tokens of a doc whose whole
> purpose is saving them. The human version is
> [`install-without-package-managers.md`](install-without-package-managers.md).

## Scope rule — read first

Install only through paths the workspace policy already permits. **Do not**
attempt to bypass, disable, or work around a control. If a step is blocked,
record it and fall through to the next tier. If all tiers are blocked, stop
and emit the IT request block at the end — do not improvise an alternative
delivery mechanism.

Blocked is a valid terminal state. Report it; do not route around it.

## Tier 0 — Text only, no install, no runtime

### ponytail (MIT) — best evidence of any agent-specific tool here

Source files in `DietrichGebert/ponytail`. Plugin marketplace not required
for most agents — but distinguish documented paths from inferred ones and
report which you used.

| Target | Copy to | Status |
| --- | --- | --- |
| Cline | `.clinerules/` | documented |
| Cursor | `.cursor/rules/` | documented |
| Copilot Chat | `.github/copilot-instructions.md` or `~/.copilot/copilot-instructions.md` | documented |
| Claude Code | `CLAUDE.md` (project root) | **inferred** — project documents the plugin; file-copy works because the harness auto-loads `CLAUDE.md` |
| Codex | `AGENTS.md` (project root) | **inferred** — same mechanism |
| Gemini CLI | none | **blocked** — requires `gemini extensions install <url>` |

`gemini-extension.json` is an installer manifest, not a drop-in file. Do not
copy it into a project and report success. If Gemini CLI is the target and
extension install is blocked, the fallback is pasting the ruleset into
`GEMINI.md` — mark it as inferred and unmeasured when reporting.

Optional, no install required:

- env `PONYTAIL_DEFAULT_MODE` = `lite` | `full` | `ultra` | `off`
- `~/.config/ponytail/config.json` → `{ "defaultMode": "..." }`
  (Windows: `%USERPROFILE%\.config\ponytail\config.json`)

**Caveat to state when reporting:** the measured −10.3% cost result used
plugin + SessionStart hook injection. File-copy is a *different*
configuration — rules sit in context every turn by construction, so there is
no activation step to fail, but it is unmeasured and it adds fixed prompt
overhead per turn (cause 6.4). Do not quote −10.3% for a file-copy install.

### caveman skill (MIT)

A prompt. Paste the ruleset into the same rules file as above, or invoke
per-session. No install of any kind. Evidence: 8.5% measured under forced
activation (i.e. a ceiling), high tail-risk — see [`../PROOF.md`](../PROOF.md).
Do not install by default.

## Tier 1 — Own installer or single binary, user-space

### codegraph — highest-value tool that is still self-installable

**Does not require Node.** Standalone installers:

- Windows: `install.ps1` from
  `raw.githubusercontent.com/colbymchenry/codegraph/main/install.ps1`
- macOS/Linux: `install.sh` from the same path
- With an internal npm mirror: `npm i -g @colbymchenry/codegraph`

**Download the installer to a file, read it, then execute.** Do not run
`irm ... | iex` or `curl ... | sh`, even though the project advertises both.
Piping unread code into a shell is not an acceptable install path here.

Manual MCP config if the installer cannot write it (`~/.claude.json`):

```json
{
  "mcpServers": {
    "codegraph": {
      "type": "stdio",
      "command": "codegraph",
      "args": ["serve", "--mcp"]
    }
  }
}
```

Unsupported on Cline. Supported: Claude Code, Cursor, Codex CLI, opencode,
Hermes, Gemini CLI, Antigravity, Kiro.

### rtk (Apache-2.0)

Single Rust binary, zero dependencies, no admin required.

**Do not prioritize this.** Measured **+7.6% cost** on Claude Code
(p=0.004). If binary download is blocked, do not escalate — the evidence does
not justify the friction. Skip it.

If the workspace permits vetted binaries: download the release, verify the
published SHA256, place in `%LOCALAPPDATA%\Programs\rtk` (Windows) or
`~/.local/bin` (POSIX), add to user-scoped `PATH`. Never disable a scanner to
make this succeed.

## Tier 2 — Requires a package manager

Cannot be satisfied by file copy. If `pip`/`npm` are blocked and no internal
mirror exists, this tier is blocked — go to the IT request block.

| Tool | Install | Note |
| --- | --- | --- |
| headroom | `uv tool install --python 3.13 "headroom-ai[all]"` / `pip install "headroom-ai[all]"` / `ghcr.io/chopratejas/headroom:latest` | Package is **`headroom-ai`**, not `headroom`. Python 3.10+. Three GitHub forks and a fourth Docker namespace — verify source before installing |
| llmlingua | `pip install llmlingua` | Peer-reviewed. Downloads a model from huggingface.co on first run |
| routellm | `pip install "routellm[serve,eval]"` | Peer-reviewed. Needs training-data augmentation to perform; not drop-in |

MCP clients like Codex do not inherit shell `PATH` — configure the absolute
path from `Get-Command <tool>` (Windows) or `command -v <tool>` (POSIX).

**llmlingua and routellm are the only agent-independent tools here.** They
are the sole honest recommendation on Cline, where no agent-specific tool has
ever been benchmarked. Prefer them over headroom when pip is available.

If an internal mirror or artifact proxy exists (Artifactory, Nexus, internal
PyPI/npm), use it. That is the sanctioned path, not a bypass.

## Execution order

1. Confirm what is actually blocked. Do not assume — test one command.
2. Install Tier 0 by file copy. This needs no permission.
3. Verify Tier 0 is live before touching Tier 1 or 2.
4. Attempt codegraph (Tier 1) before opening any ticket — it is often
   installable without one.
5. Attempt remaining Tier 1/2 only via sanctioned channels.
6. On block: stop, emit the IT request block, report which tier failed.

## Verification

| Tool | Check |
| --- | --- |
| ponytail | Give the agent a task that invites over-building; confirm it ships the minimal version instead of scaffolding |
| caveman | Confirm narration is terse while code/diffs stay verbatim |
| rtk | Compare **billed input tokens** across a paired run, not `wc -c` of wrapped output |
| codegraph | Confirm a graph query returns without file reads |
| headroom | Enable its built-in 10% control group; read measured, not estimated, savings |

⚠️ Do not verify rtk by comparing raw vs wrapped output size. That measures
the text-compression ratio, which is real and irrelevant — it is the exact
metric that produced the false 60–90% claim. Only a paired before/after on
the bill counts.

Do not report a tool as installed until its check passes.

## Measure before and after

Any install claim without a before/after is the exact failure documented in
[`../PROOF.md`](../PROOF.md). Baseline with
[`../solutions/token-counting.md`](../solutions/token-counting.md) first.
Prefer tools that ship a control group (headroom does).

## IT request block — emit on block

```
Request: allow <tool> for AI coding agents

What: <tool>, <license>, <repo url>
Install: <file copy | user-directory installer | pip via internal mirror>
Data: source code stays local; no telemetry accounts required

Runtime network (state precisely; do not claim "all local"):
- ponytail, caveman, codegraph, rtk: none at runtime
- headroom: YES — proxy must reach your LLM API; setup also fetches
  cdn.pyke.io (ONNX Runtime) and huggingface.co (model)
- llmlingua: fetches a model from huggingface.co on first run

Evidence (independent, controlled benchmarks):
- ponytail: -10.3% cost, p=0.004, 80 paired tasks (JetBrains)
- codegraph: -69% tokens / -60% cost, 7 repos, reproducible methodology
- llmlingua: up to 20x compression, peer-reviewed (EMNLP 2023)
- Excluded on evidence: rtk (+7.6%), caveman (8.5%, high variance)

Requesting: <internal mirror entry | binary allowlist | rules-file exemption>
```

Attach [`../PROOF.md`](../PROOF.md). The request is stronger for naming the
tools that failed than for asking for all of them.
