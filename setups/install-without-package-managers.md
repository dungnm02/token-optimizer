# Cài đặt trong môi trường bị hạn chế (Tiếng Việt)

Dành cho nơi làm việc chặn plugin marketplace, chặn cài extension, chặn
`npm -g`, và không cho quyền admin.

**Tin tốt: xếp theo chất lượng bằng chứng, những công cụ đáng có lại đúng là
những công cụ cần ít quyền cài đặt nhất.** Công cụ có bằng chứng tốt nhất
(ponytail) chỉ là mấy file Markdown. Công cụ đòi cài binary nhiều nhất (RTK)
lại là công cụ đo ra **+7.6% chi phí**. Bạn gần như không mất gì khi ở trong
khuôn khổ chính sách.

> Trang này chỉ đi theo những đường mà chính sách vốn đã cho phép. Nó không
> hướng dẫn vượt rào, và không cần phải vượt rào — xem
> [Thứ tự ưu tiên](#thứ-tự-ưu-tiên-nên-làm-gì-trước).

## Bảng quyết định

Cột cuối là **phán quyết** (kết quả đo được), không phải hạng bằng chứng —
RTK có bằng chứng hạng A, chỉ là bằng chứng đó cho thấy nó có hại.

| Công cụ | Cần gì để cài | Bị chặn? | Phán quyết |
| --- | --- | --- | --- |
| **ponytail** | Chép file Markdown | ✅ Không, trừ Gemini CLI | ✅ −10.3% chi phí |
| **caveman** | Dán một prompt | ✅ Không bao giờ bị chặn | ⚠️ 8.5%, rủi ro đuôi |
| **codegraph** | Trình cài riêng, **không cần Node** | ⚠️ Có thể, nếu chặn tải binary | ✅ −60% chi phí |
| **rtk** | Một binary tĩnh, không cần admin | ⚠️ Có thể | ❌ +7.6% — bỏ qua |
| **headroom** | pip / npm / Docker | 🚫 Có, nếu bị chặn | ⚪ chưa có A/B độc lập |
| **llmlingua / routellm** | pip | 🚫 Có, nếu pip bị chặn | ✅ bình duyệt, 20× và −85% |

## Tầng 0 — Chỉ là văn bản, không cài gì

### ponytail — công cụ đáng giá nhất, và dễ nhất

**Không cần plugin marketplace** — nhưng chỉ đúng với một số agent. Hãy phân
biệt rõ đâu là đường dự án ghi nhận, đâu là đường chạy được nhưng do suy ra:

| Agent | Chép vào | Nguồn |
| --- | --- | --- |
| Cline | `.clinerules/` | ✅ dự án ghi rõ |
| Cursor | `.cursor/rules/` | ✅ dự án ghi rõ |
| Copilot Chat | `.github/copilot-instructions.md`, hoặc toàn cục `~/.copilot/copilot-instructions.md` | ✅ dự án ghi rõ |
| Claude Code | `CLAUDE.md` ở gốc dự án | ⚠️ dự án khuyên cài plugin; chép file vẫn chạy vì Claude Code tự nạp `CLAUDE.md`, nhưng **dự án không ghi nhận cách này** |
| Codex | `AGENTS.md` ở gốc dự án | ⚠️ như trên (Codex tự nạp `AGENTS.md`) |
| Gemini CLI | ❌ **không có đường chép file** | Dự án chỉ hỗ trợ `gemini extensions install <url>` — tức là đúng thứ bị chặn |

Về Gemini CLI: `gemini-extension.json` là **manifest cho trình cài đọc**,
không phải file bạn tự đặt vào dự án. (Bản trước của trang này ghi sai — đã
sửa.) Cách vòng khả dĩ là dán nội dung ruleset vào `GEMINI.md`, nhưng đây là
suy luận của chúng tôi, **chưa được dự án ghi nhận và chưa ai đo**.

Chỉnh mức độ mà không cần cài gì: biến môi trường `PONYTAIL_DEFAULT_MODE`
(`lite`/`full`/`ultra`/`off`), hoặc `~/.config/ponytail/config.json` (trên
Windows: `%USERPROFILE%\.config\ponytail\config.json`).

⚠️ **Một lưu ý trung thực:** con số −10.3% đo được là ở cấu hình plugin +
SessionStart hook. Cách chép file là cấu hình **khác**: ruleset nằm sẵn trong
context mỗi lượt nên không có bước kích hoạt nào để hỏng, nhưng nó **chưa
được đo**, và nó thêm chi phí prompt cố định mỗi lượt (nguyên nhân 6.4). Hãy
kỳ vọng có tác dụng, đừng trích con số −10.3% cho cách cài này.

### caveman — chỉ là một prompt

Dán ruleset vào cùng file rules ở trên, hoặc gọi theo từng phiên. Không cài
gì hết. Chỉ dùng cho luồng nhiều văn xuôi; xem [`../PROOF.md`](../PROOF.md).

## Tầng 1 — Trình cài riêng, không cần admin, không cần package manager

### codegraph — đáng giá nhất ở tầng này

**Không cần Node.** Dự án có trình cài độc lập, đây chính là điểm khiến nó
khả thi trong môi trường chặn npm:

```powershell
# Windows — KHÔNG chạy thẳng kiểu `irm ... | iex`
irm https://raw.githubusercontent.com/colbymchenry/codegraph/main/install.ps1 -OutFile install.ps1
# đọc file trước, rồi mới chạy
.\install.ps1
```

```bash
# macOS / Linux — tải rồi đọc, đừng pipe thẳng vào sh
curl -fsSL https://raw.githubusercontent.com/colbymchenry/codegraph/main/install.sh -o install.sh
sh install.sh
```

Nếu có Node và npm nội bộ: `npm i -g @colbymchenry/codegraph`.

⚠️ **Đừng pipe thẳng vào shell.** Dự án quảng cáo `irm ... | iex` và
`curl ... | sh`. Tải xuống, đọc, rồi chạy — mất thêm hai phút, và trong một
môi trường siết bảo mật thì đó là khác biệt giữa "cài có kiểm soát" và
"thực thi mã lạ không nhìn qua".

Cấu hình MCP thủ công (`~/.claude.json`), nếu trình cài không tự làm được:

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

**Không hỗ trợ Cline.** Danh sách: Claude Code, Cursor, Codex CLI, opencode,
Hermes, Gemini CLI, Antigravity, Kiro.

### rtk — có ở đây cho đủ, nhưng đừng cài

Một binary Rust duy nhất, không phụ thuộc gì, chạy được từ thư mục người
dùng. **Đừng dồn sức cho cái này.** Nó đo ra **+7.6% chi phí** trên Claude
Code (p=0.004). Nếu tải binary bị chặn, đừng leo thang xin phép — bằng chứng
không xứng với công sức. Bỏ qua.

Nếu vẫn muốn thử: tải bản release, đối chiếu SHA256 công bố, đặt vào
`%LOCALAPPDATA%\Programs\rtk` (Windows) hoặc `~/.local/bin` (POSIX), rồi thêm
vào `PATH` ở phạm vi user. Không bao giờ tắt trình quét để nó chạy được.

## Tầng 2 — Thật sự cần package manager

**headroom** — `uv tool install --python 3.13 "headroom-ai[all]"`, hoặc
`pip install "headroom-ai[all]"`, hoặc Docker
(`ghcr.io/chopratejas/headroom:latest`). Cần Python 3.10+. Với MCP client như
Codex (không thừa hưởng `PATH` của shell), cấu hình đường dẫn tuyệt đối lấy
từ `Get-Command headroom` (Windows) hoặc `command -v headroom` (POSIX).

⚠️ Gói pip tên là **`headroom-ai`**, không phải `headroom`; có ba fork trên
GitHub và image Docker lại nằm ở namespace thứ tư. Với một công cụ nằm giữa
agent và API key của bạn, hãy xác minh nguồn cho chắc trước khi cài.

Nếu công ty có mirror nội bộ (Artifactory, Nexus, PyPI/npm nội bộ) thì dùng
nó — đó là đường chính thống, không phải lách. Nếu không có, tầng này bị
chặn; dùng mẫu đề nghị bên dưới.

## Tầng ★ — Thư viện bình duyệt (bằng chứng mạnh nhất cả trang)

Đây là tầng duy nhất **độc lập với agent**: là thư viện Python bạn gọi từ code
của mình, nên chạy như nhau dưới Cline, Claude, Codex, Gemini. Với Cline —
nơi chưa công cụ chuyên biệt nào từng được benchmark — đây là thứ duy nhất
đề xuất được một cách trung thực.

- **LLMLingua** — `pip install llmlingua`. Nén tới 20× (EMNLP 2023). Lưu ý:
  nó **tải model về máy** ở lần chạy đầu (từ Hugging Face), nên vẫn cần một
  lần ra mạng; chạy CPU được nhưng chậm hơn GPU.
- **RouteLLM** — `pip install "routellm[serve,eval]"`. Giảm 85% chi phí MT
  Bench ở 95% chất lượng GPT-4. Không phải thứ cắm vào là chạy: chính nhóm
  tác giả nêu rõ router chạy kém nếu không bổ sung dữ liệu huấn luyện.

Cần pip, nên về mặt kỹ thuật vẫn là Tầng 2 — nhưng đây là **cuộc trao đổi
khác** với IT: một thư viện Python trong dự án của bạn, không phải một plugin
cắm vào agent hay một proxy giữ API key.

## Thứ tự ưu tiên: nên làm gì trước

1. **Cài ponytail bằng cách chép file.** Không cần xin phép ai, và đây là
   công cụ có bằng chứng tốt nhất trong nhóm chuyên biệt. Làm ngay hôm nay.
2. **Đo baseline** bằng
   [`../solutions/token-counting.md`](../solutions/token-counting.md) trước
   khi thêm bất cứ thứ gì khác.
3. **Thử codegraph** nếu repo của bạn trên ~1.000 file. Vì nó **không cần
   Node**, rất có thể bạn tự cài được mà không phải xin ai — hãy thử trước
   khi mở ticket.
4. **Bỏ qua rtk và caveman** trừ khi bạn có luồng nhiều văn xuôi. Đừng đánh
   nhau với IT vì hai công cụ đo ra +7.6% và 8.5%.
5. **Chỉ xin duyệt pip** nếu bạn muốn tầng bình duyệt hoặc headroom.

Nói cách khác: phần lớn giá trị đo được nằm ở thứ không cần cài gì cả, hoặc
tự cài được trong thư mục người dùng.

## Ghi chú cho Windows

Phần lớn tài liệu của các dự án này viết cho POSIX. Quy đổi nhanh:

| POSIX | Windows |
| --- | --- |
| `~/.local/bin` | `%LOCALAPPDATA%\Programs\<tool>` |
| `command -v <tool>` | `Get-Command <tool> \| Select-Object -ExpandProperty Source` |
| `~/.config/<tool>/` | `%USERPROFILE%\.config\<tool>\` |
| `export PATH=...` | `setx PATH "$env:PATH;<đường dẫn>"` (phạm vi user, không cần admin) |

## Mẫu đề nghị gửi IT

```
Đề nghị: cho phép <tên công cụ> dùng với coding agent

Là gì:   <công cụ>, <license>, <link repo>
Cài thế nào: <chép file | trình cài trong thư mục user | pip qua mirror nội bộ>
Mạng:    xem phần dưới — khác nhau tùy công cụ
Dữ liệu: mã nguồn không rời máy; không cần tài khoản telemetry

Yêu cầu mạng lúc chạy (nêu chính xác, đừng nói gộp "chạy local"):
- ponytail, caveman: không có. Thuần văn bản.
- codegraph: không có. Index nằm local trong SQLite.
- rtk: không có.
- headroom: CÓ. Là proxy nên phải gọi tới API LLM của bạn; lần cài đầu còn
  tải ONNX Runtime (cdn.pyke.io) và model nén (huggingface.co).
- llmlingua: tải model từ huggingface.co ở lần chạy đầu, sau đó chạy local.

Bằng chứng (benchmark độc lập, có đối chứng):
- ponytail: −10.3% chi phí, p=0.004, 80 cặp tác vụ (JetBrains)
- codegraph: −69% token / −60% chi phí, 7 repo, phương pháp tái lập được
- LLMLingua: nén tới 20×, bình duyệt EMNLP 2023
- Đã loại vì bằng chứng: rtk (+7.6%), caveman (8.5%, phương sai lớn)

Xin: <thêm vào mirror nội bộ | allowlist binary | miễn trừ cho file rules>
```

Đính kèm [`../PROOF.md`](../PROOF.md). Đề nghị của bạn **mạnh hơn** nhờ việc
nêu rõ hai công cụ đã bị loại — nó cho thấy bạn đã sàng lọc, chứ không xin
bừa cả gói. Và hãy khai báo yêu cầu mạng cho chính xác: nói "tất cả đều chạy
local" khi headroom là một proxy sẽ khiến bạn mất uy tín với đội bảo mật.

## Ba việc không nên làm

- Tắt hay né trình quét endpoint để một binary chạy được.
- Dùng registry lạ hoặc mirror ngoài luồng để vòng qua chặn npm/pip.
- Side-load extension để vượt qua chính sách.

Không việc nào cần thiết ở đây, và cả ba đều biến một khoản tiết kiệm ~10%
thành một sự cố bảo mật.

---

# Installing in a restricted workspace

For workplaces that block the plugin marketplace, extension installs,
`npm -g`, and admin rights.

**The good news: ranked by evidence, the tools worth having need the least
installation.** The best-evidenced tool (ponytail) is a few Markdown files.
The one demanding the most binary install (RTK) is the one that measured
**+7.6% cost**. Staying inside policy costs you almost nothing.

> This page only uses paths your policy already permits. It does not describe
> circumvention, and none is needed — see
> [Priority order](#priority-order-what-to-do-first).

## Decision table

The last column is the **verdict** (what was measured), not the evidence
tier — RTK has Tier A evidence; that evidence just says it hurts.

| Tool | What install needs | Blocked? | Verdict |
| --- | --- | --- | --- |
| **ponytail** | Copy Markdown files | ✅ No, except Gemini CLI | ✅ −10.3% cost |
| **caveman** | Paste a prompt | ✅ Never blocked | ⚠️ 8.5%, tail risk |
| **codegraph** | Own installer, **no Node needed** | ⚠️ Maybe, if binaries are blocked | ✅ −60% cost |
| **rtk** | One static binary, no admin | ⚠️ Maybe | ❌ +7.6% — skip |
| **headroom** | pip / npm / Docker | 🚫 Yes, if blocked | ⚪ no independent A/B |
| **llmlingua / routellm** | pip | 🚫 Yes, if pip is blocked | ✅ peer-reviewed, 20× and −85% |

## Tier 0 — Text only, nothing to install

### ponytail — the most valuable tool, and the easiest

**No plugin marketplace required** — but only for some agents. Keep the
project-documented paths separate from the ones that merely work:

| Agent | Copy to | Source |
| --- | --- | --- |
| Cline | `.clinerules/` | ✅ documented by the project |
| Cursor | `.cursor/rules/` | ✅ documented by the project |
| Copilot Chat | `.github/copilot-instructions.md`, or globally `~/.copilot/copilot-instructions.md` | ✅ documented by the project |
| Claude Code | `CLAUDE.md` in project root | ⚠️ the project recommends the plugin; file-copy works because Claude Code auto-loads `CLAUDE.md`, but **the project does not document it** |
| Codex | `AGENTS.md` in project root | ⚠️ same (Codex auto-loads `AGENTS.md`) |
| Gemini CLI | ❌ **no file-copy path** | The project supports only `gemini extensions install <url>` — exactly what's blocked |

On Gemini CLI: `gemini-extension.json` is a **manifest the installer reads**,
not a file you drop into your project. (An earlier version of this page said
otherwise — corrected.) The plausible workaround is pasting the ruleset text
into `GEMINI.md`, but that is our inference — **undocumented and unmeasured**.

Tune intensity with no install at all: env `PONYTAIL_DEFAULT_MODE`
(`lite`/`full`/`ultra`/`off`), or `~/.config/ponytail/config.json` (on
Windows: `%USERPROFILE%\.config\ponytail\config.json`).

⚠️ **One honest caveat:** the measured −10.3% came from a plugin +
SessionStart hook configuration. File-copy is a **different** setup: the
ruleset sits in context every turn by construction, so there is no activation
step that can fail — but it is **unmeasured**, and it adds fixed prompt
overhead per turn (cause 6.4). Expect it to help; don't quote −10.3% for it.

### caveman — just a prompt

Paste the ruleset into the same rules file above, or invoke it per session.
No install of any kind. Worth it only for prose-heavy routes; see
[`../PROOF.md`](../PROOF.md).

## Tier 1 — Own installer, no admin, no package manager

### codegraph — the one worth having at this tier

**No Node required.** The project ships a standalone installer, which is
exactly what makes it viable where npm is blocked:

```powershell
# Windows — do NOT run this as `irm ... | iex`
irm https://raw.githubusercontent.com/colbymchenry/codegraph/main/install.ps1 -OutFile install.ps1
# read it first, then run
.\install.ps1
```

```bash
# macOS / Linux — download and read it, don't pipe straight to sh
curl -fsSL https://raw.githubusercontent.com/colbymchenry/codegraph/main/install.sh -o install.sh
sh install.sh
```

If you do have Node and an internal npm mirror: `npm i -g @colbymchenry/codegraph`.

⚠️ **Don't pipe straight into a shell.** The project advertises
`irm ... | iex` and `curl ... | sh`. Download, read, then run — it costs two
minutes, and in a hardened environment it is the difference between a
reviewed install and executing unseen code.

Manual MCP config (`~/.claude.json`), if the installer can't write it:

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

**No Cline support.** The list is Claude Code, Cursor, Codex CLI, opencode,
Hermes, Gemini CLI, Antigravity, Kiro.

### rtk — listed for completeness, but don't install it

One Rust binary, zero dependencies, runnable from a user directory.
**Don't spend effort here.** It measured **+7.6% cost** on Claude Code
(p=0.004). If binary download is blocked, don't escalate — the evidence
doesn't justify the friction. Skip it.

If you want it anyway: download the release, verify the published SHA256,
place it in `%LOCALAPPDATA%\Programs\rtk` (Windows) or `~/.local/bin`
(POSIX), and add it to your user-scoped `PATH`. Never disable a scanner to
make it work.

## Tier 2 — Genuinely needs a package manager

**headroom** — `uv tool install --python 3.13 "headroom-ai[all]"`, or
`pip install "headroom-ai[all]"`, or Docker
(`ghcr.io/chopratejas/headroom:latest`). Needs Python 3.10+. For MCP clients
like Codex that don't inherit your shell `PATH`, configure the absolute path
from `Get-Command headroom` (Windows) or `command -v headroom` (POSIX).

⚠️ The pip package is **`headroom-ai`**, not `headroom`; there are three
GitHub forks and the Docker image sits in a fourth namespace. For a tool that
sits between your agent and your API key, confirm the source before
installing.

If your company runs an internal mirror (Artifactory, Nexus, internal
PyPI/npm), use it — that's the sanctioned route, not a bypass. If there isn't
one, this tier is blocked; use the request template below.

## Tier ★ — The peer-reviewed libraries (strongest evidence on the page)

The only **agent-independent** tier: Python libraries you call from your own
code, so they behave identically under Cline, Claude, Codex and Gemini. For
Cline — where no agent-specific tool has ever been benchmarked — this is the
only thing that can honestly be recommended.

- **LLMLingua** — `pip install llmlingua`. Up to 20× compression (EMNLP
  2023). Note it **downloads a model** on first run (from Hugging Face), so
  it needs one trip to the network; CPU works but is slower than GPU.
- **RouteLLM** — `pip install "routellm[serve,eval]"`. −85% cost on MT Bench
  at 95% of GPT-4 quality. Not drop-in: the authors state plainly that the
  router underperforms without training-data augmentation.

These need pip, so technically Tier 2 — but it's a **different conversation**
with IT: a Python library inside your project, not a plugin wired into your
agent or a proxy holding your API key.

## Priority order: what to do first

1. **Install ponytail by file copy.** Needs nobody's permission, and it's the
   best-evidenced agent-specific tool. Do it today.
2. **Baseline your usage** with
   [`../solutions/token-counting.md`](../solutions/token-counting.md) before
   adding anything else.
3. **Try codegraph** if your repo is above ~1,000 files. Because it needs
   **no Node**, you may be able to install it yourself without asking — try
   before you open a ticket.
4. **Skip rtk and caveman** unless you have prose-heavy routes. Don't fight
   IT over tools that measured +7.6% and 8.5%.
5. **Only request pip access** if you want the peer-reviewed tier or
   headroom.

Put differently: most of the measured value sits in things that need no
install at all, or that install into your own user directory.

## Windows notes

Most of these projects document POSIX paths. Quick translation:

| POSIX | Windows |
| --- | --- |
| `~/.local/bin` | `%LOCALAPPDATA%\Programs\<tool>` |
| `command -v <tool>` | `Get-Command <tool> \| Select-Object -ExpandProperty Source` |
| `~/.config/<tool>/` | `%USERPROFILE%\.config\<tool>\` |
| `export PATH=...` | `setx PATH "$env:PATH;<path>"` (user scope, no admin) |

## IT request template

```
Request: allow <tool> for AI coding agents

What:    <tool>, <license>, <repo url>
Install: <file copy | user-directory installer | pip via internal mirror>
Data:    source code never leaves the machine; no telemetry account needed

Runtime network access (state this precisely — do not say "all local"):
- ponytail, caveman: none. Plain text.
- codegraph: none. The index is local, in SQLite.
- rtk: none.
- headroom: YES. It is a proxy, so it must reach your LLM API; first-time
  setup also fetches ONNX Runtime (cdn.pyke.io) and a compression model
  (huggingface.co).
- llmlingua: downloads a model from huggingface.co on first run, local after.

Evidence (independent, controlled benchmarks):
- ponytail: -10.3% cost, p=0.004, 80 paired tasks (JetBrains)
- codegraph: -69% tokens / -60% cost, 7 repos, reproducible methodology
- LLMLingua: up to 20x compression, peer-reviewed at EMNLP 2023
- Excluded on evidence: rtk (+7.6%), caveman (8.5%, high variance)

Requesting: <internal mirror entry | binary allowlist | rules-file exemption>
```

Attach [`../PROOF.md`](../PROOF.md). Your request is **stronger** for naming
the two tools you rejected — it shows you filtered rather than asking for the
whole bundle. And describe the network requirements accurately: claiming
"everything runs locally" when headroom is a proxy will cost you credibility
with your security team.

## Three things not to do

- Disabling or evading endpoint scanning to get a binary running.
- Using an unofficial registry or off-channel mirror to route around an
  npm/pip block.
- Side-loading an extension to defeat policy.

None of these is necessary here, and each turns a ~10% saving into a security
incident.
