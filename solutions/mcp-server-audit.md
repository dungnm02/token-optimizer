# Kiểm toán MCP server (Tiếng Việt)

**Giải quyết:** Nguyên nhân 3.4 và 2.3 trong [`../CAUSE.md`](../CAUSE.md)

**Ý tưởng:** Mỗi MCP server bạn bật đều đưa schema tool của nó vào context —
với vài agent là vào **mọi request**, kể cả khi bạn không hề gọi tới nó. Hãy
đo chi phí thường trực đó, rồi tắt những server không kiếm đủ chỗ chúng đang
chiếm.

---

## Vì sao đây là kiểm toán, không phải một lần dọn dẹp

MCP server được cài theo kiểu tích lũy. Bạn thêm một cái cho Jira, một cái cho
Postgres, một cái cho Sentry — mỗi cái đều hợp lý ở thời điểm đó. Không ai gỡ
cái nào ra cả. Sáu tháng sau, ba server bạn dùng mỗi tuần một lần vẫn đang trả
tiền thuê chỗ ở mỗi lượt.

Điểm mấu chốt là **chi phí của một server hoàn toàn không liên quan tới việc
bạn có dùng nó hay không**. Một server có 40 tool mà bạn chưa từng gọi vẫn có
giá đúng bằng một server có 40 tool bạn dùng liên tục — trên các harness nạp
sẵn toàn bộ schema.

## Chi phí thật của bạn tùy thuộc vào harness

Đây là chỗ số tiền chênh nhau nhiều nhất, và nó không phải thuộc tính của MCP:

| Agent | Schema tool được nạp thế nào | Hệ quả |
| --- | --- | --- |
| Claude Code | **Trì hoãn mặc định** — chỉ tên tool vào context cho tới khi Claude thực sự dùng một tool cụ thể | Chi phí thường trực thấp; vẫn nên tắt server không dùng |
| Gemini CLI | Có hook lọc tool trước khi chọn | Kiểm soát được, nhưng phải tự cấu hình |
| Codex CLI | Chưa rõ tài liệu về việc nạp trì hoãn | Coi như nạp sẵn cho tới khi bạn tự đo |
| Cline | **Nhét vào mọi request**, không có cơ chế trì hoãn | Đây là nút tốn kém nhất bạn có |

Nếu bạn dùng Cline, hãy đọc phần này trước mọi thứ khác trong thư mục
`solutions/`. Nếu bạn dùng Claude Code, harness đã lo phần lớn — việc kiểm
toán vẫn đáng làm nhưng lợi ích nhỏ hơn nhiều. Đó chính là quy tắc hai điều
kiện trong [`../HARNESS.md`](../HARNESS.md) áp dụng vào một trường hợp cụ thể.

## Cách áp dụng

1. **Đo trước đã.** Trên Claude Code, chạy `/context` — nó phân tách chỗ đang
   bị chiếm và gắn cờ các tool ngốn context. Ghi lại con số đó trước khi động
   vào bất cứ thứ gì.
2. **Quy chi phí về từng server.** Trên gói thuê bao, phần phân bổ trong
   `/usage` quy phần trăm mức dùng gần đây về **từng MCP server**. Đây là
   cách trực tiếp nhất để trả lời "cái nào đang ăn tiền của tôi". Lưu ý cách
   tính: phần của một server chỉ đếm những request đã thực sự tiêu thụ kết
   quả tool của nó.
3. **Liệt kê những gì đang bật.** `/mcp` trên Claude Code mở danh sách tương
   tác. Với mỗi server, hỏi ba câu:
   - Tôi đã dùng nó lần cuối khi nào?
   - Nó phơi ra bao nhiêu tool, và tôi thực sự cần bao nhiêu trong số đó?
   - Có CLI nào làm được việc tương đương không?
4. **Thay bằng CLI khi có thể.** `gh`, `aws`, `gcloud`, `sentry-cli` tiết
   kiệm context hơn MCP server tương ứng vì chúng **không thêm dòng liệt kê
   tool nào cả**. Agent gọi chúng qua shell. Bạn đổi một chi phí thường trực
   lấy một lần gọi tool duy nhất khi cần.
5. **Tắt theo mặc định, bật theo dự án.** Đừng để mọi server bật ở cấu hình
   người dùng toàn cục. Bật ở cấp dự án đúng những cái mà dự án đó cần. Trên
   Claude Code, `/mcp disable <tên>` và `/mcp disable all` đổi trạng thái
   ngay mà không cần mở hộp thoại.
6. **Chặn output tràn.** Schema là chi phí thường trực; kết quả tool là chi
   phí theo lượt và thường lớn hơn nhiều. Đặt `MAX_MCP_OUTPUT_TOKENS` (Claude
   Code) hoặc giới hạn tương đương — xem [`../CONFIG.md`](../CONFIG.md) và
   [`tool-output-budgets.md`](tool-output-budgets.md).
7. **Ưu tiên server hẹp hơn server "mọi thứ".** Một server phơi ra 60 tool để
   bạn dùng 3 là lựa chọn tệ. Nếu server hỗ trợ lọc tool, hãy lọc.
8. **Kiểm toán lại định kỳ.** Mỗi quý, hoặc mỗi lần ai đó thêm một server
   mới. Đây là thứ trôi dạt trở lại.

## Thứ tự quyết định

Với mỗi server, theo đúng thứ tự này:

| Câu hỏi | Nếu đúng |
| --- | --- |
| Có CLI làm được việc này không? | Dùng CLI, gỡ server |
| Dùng ít hơn mỗi tuần một lần? | Tắt; bật lại theo phiên khi cần |
| Chỉ cần ở một dự án? | Chuyển từ cấu hình toàn cục xuống cấp dự án |
| Phơi ra nhiều tool hơn mức bạn cần? | Lọc danh sách tool nếu server hỗ trợ |
| Trả về kết quả lớn? | Đặt trần output, đừng gỡ server |

## Công cụ hiện đại nhất (SOTA)

### Có sẵn — coding agent & API của nhà cung cấp

| Nhà cung cấp / agent | Tính năng | Ghi chú |
| --- | --- | --- |
| Claude Code | Nạp tool trì hoãn / tìm kiếm tool | Bật mặc định; chỉ tên tool vào context cho tới khi cần |
| Claude Code | `/context` | Phân tách chỗ bị chiếm, gắn cờ tool ngốn context |
| Claude Code | `/usage` (phân bổ) | Phần trăm mức dùng gần đây theo **từng** MCP server |
| Claude Code | `/mcp enable\|disable <tên\|all>` | Đổi trạng thái kết nối, dùng được cả ở chế độ `-p` |
| Claude Code | `MAX_MCP_OUTPUT_TOKENS` | Trần cho kết quả tool MCP |
| Gemini CLI | Hook lọc tool | Lọc tool trước khi model chọn |
| Cline | — | Không có cơ chế trì hoãn; chỉ còn cách bật/tắt server |

### Bên thứ ba — không phụ thuộc agent (ưu tiên mã nguồn mở)

| Công cụ | Giấy phép | Ghi chú |
| --- | --- | --- |
| CLI của chính nhà cung cấp (`gh`, `aws`, `gcloud`, `sentry-cli`) | Khác nhau | Không tốn chỗ liệt kê tool; thường là lựa chọn rẻ nhất |
| Bộ đếm token của nhà cung cấp | — | Đo trực tiếp khối schema; xem [`token-counting.md`](token-counting.md) |
| Gateway MCP / proxy lọc tool | Khác nhau | Gộp nhiều server sau một bề mặt hẹp hơn; thêm một thành phần vận hành |

## Đánh đổi

- **Tắt một server không miễn phí về mặt con người.** Khi agent thiếu công cụ
  cần thiết, nó sẽ tìm cách vòng — thường là nhiều lượt shell hơn, tốn nhiều
  token hơn số bạn vừa tiết kiệm. Hãy tắt những cái *không dùng*, không phải
  những cái *có vẻ đắt*.
- **CLI đổi một chi phí lấy một chi phí khác.** Không tốn chỗ liệt kê tool,
  nhưng agent phải nhớ cú pháp và đôi khi đoán sai — dẫn tới vài lượt thử.
  Với công cụ phổ biến thì đây là lựa chọn tốt; với API nội bộ ít gặp thì
  MCP server có schema rõ ràng lại rẻ hơn.
- **Cấu hình theo dự án tăng chi phí bảo trì.** Nhiều nơi hơn để cấu hình
  trôi dạt.

## Tác động dự kiến

⚪ **Chưa đo trong kho này.** Mức tiết kiệm dao động cực lớn tùy harness và
tùy bạn đã tích tụ bao nhiêu server.

Điều có thể nói chắc chắn về mặt cấu trúc: trên harness nạp sẵn toàn bộ
schema, chi phí tỷ lệ thuận với **tổng số tool đang bật**, không phải số tool
bạn dùng. Trên harness nạp trì hoãn, phần lớn khoản đó đã bị harness thu hồi
sẵn — nên hãy đo trước khi bỏ công.

Cách đo, cụ thể: ghi lại chỉ số `/context` khi bật tất cả server, tắt hết,
mở phiên mới, ghi lại lần nữa. Chênh lệch chính là chi phí thường trực bạn
đang trả. Xem [`../MEASURE.md`](../MEASURE.md) trước khi kết luận rằng việc
tắt server đã giúp bạn tiết kiệm.

---

# MCP server audit

**Addresses:** Causes 3.4 and 2.3 in [`../CAUSE.md`](../CAUSE.md)

**Idea:** Every MCP server you enable puts its tool schemas into context — on
some agents, into **every request**, whether or not you ever call it. Measure
that standing cost, then turn off the servers that aren't earning the space
they occupy.

---

## Why this is an audit, not a one-time cleanup

MCP servers accumulate. You add one for Jira, one for Postgres, one for
Sentry — each reasonable at the time. Nobody ever removes one. Six months
later, three servers you use once a week are paying rent on every turn.

The key point is that **a server's cost is entirely unrelated to whether you
use it**. A server exposing 40 tools you never call costs exactly what one
exposing 40 tools you call constantly costs — on harnesses that load all
schemas upfront.

## Your actual cost depends on the harness

This is where the money differs most, and it isn't a property of MCP:

| Agent | How tool schemas load | Consequence |
| --- | --- | --- |
| Claude Code | **Deferred by default** — only tool names enter context until Claude actually uses a specific tool | Low standing cost; still worth disabling unused servers |
| Gemini CLI | Hooks can filter tools before selection | Controllable, but you have to configure it |
| Codex CLI | Deferred loading undocumented | Assume upfront until you measure it yourself |
| Cline | **Injected into every request**, no deferral | This is the most expensive dial you have |

If you're on Cline, read this before anything else in `solutions/`. If you're
on Claude Code, the harness already handles most of it — the audit is still
worth doing, but the payoff is much smaller. That's the two-condition rule
from [`../HARNESS.md`](../HARNESS.md) applied to one concrete case.

## How to apply

1. **Measure first.** On Claude Code, run `/context` — it breaks down what's
   occupying space and flags context-heavy tools. Record that number before
   touching anything.
2. **Attribute cost to individual servers.** On a subscription, the `/usage`
   attribution view assigns a share of recent usage to **each MCP server**.
   That's the most direct answer to "which one is costing me." Note how it
   counts: a server's share covers only the requests that actually consumed
   one of its tool results.
3. **List what's on.** `/mcp` on Claude Code opens the interactive list. For
   each server, ask three questions:
   - When did I last use it?
   - How many tools does it expose, and how many do I actually need?
   - Is there a CLI that does the same job?
4. **Replace with CLIs where you can.** `gh`, `aws`, `gcloud` and
   `sentry-cli` are more context-efficient than the equivalent MCP server
   because they add **no per-tool listing at all**. The agent calls them
   through the shell. You trade a standing cost for a single tool call when
   you need it.
5. **Off by default, on per project.** Don't leave every server enabled in
   your global user config. Enable exactly what a project needs at the
   project level. On Claude Code, `/mcp disable <name>` and `/mcp disable all`
   change state without opening the dialog.
6. **Cap the output too.** Schemas are a standing cost; tool results are a
   per-turn cost and usually much larger. Set `MAX_MCP_OUTPUT_TOKENS` (Claude
   Code) or the equivalent — see [`../CONFIG.md`](../CONFIG.md) and
   [`tool-output-budgets.md`](tool-output-budgets.md).
7. **Prefer narrow servers to "everything" servers.** A server exposing 60
   tools so you can use 3 is a bad trade. If the server supports tool
   filtering, filter it.
8. **Re-audit periodically.** Every quarter, or whenever someone adds a new
   server. This is a thing that drifts back.

## Decision order

For each server, in this order:

| Question | If yes |
| --- | --- |
| Is there a CLI that does this? | Use the CLI, remove the server |
| Used less than once a week? | Disable it; enable per session when needed |
| Only needed in one project? | Move it from global to project config |
| Exposes more tools than you need? | Filter the tool list if the server supports it |
| Returns large results? | Cap the output; don't remove the server |

## SOTA tools

### Native — coding agents & provider APIs

| Provider / agent | Feature | Notes |
| --- | --- | --- |
| Claude Code | Deferred tool loading / tool search | On by default; only tool names enter context until needed |
| Claude Code | `/context` | Breaks down occupied space, flags context-heavy tools |
| Claude Code | `/usage` (attribution) | Share of recent usage per **individual** MCP server |
| Claude Code | `/mcp enable\|disable <name\|all>` | Change connection state; also works in `-p` mode |
| Claude Code | `MAX_MCP_OUTPUT_TOKENS` | Ceiling for MCP tool results |
| Gemini CLI | Tool-filtering hooks | Filter tools before the model selects |
| Cline | — | No deferral mechanism; enabling/disabling servers is the only lever |

### Third-party — agent-agnostic (open source preferred)

| Tool | License | Notes |
| --- | --- | --- |
| Vendor CLIs (`gh`, `aws`, `gcloud`, `sentry-cli`) | Various | No tool-listing footprint; usually the cheapest option |
| Provider token counters | — | Measure the schema block directly; see [`token-counting.md`](token-counting.md) |
| MCP gateways / tool-filtering proxies | Various | Collapse several servers behind a narrower surface; adds an operational component |

## Trade-offs

- **Disabling a server isn't free in human terms.** When the agent lacks a
  tool it needs, it works around it — usually with more shell turns, costing
  more tokens than you saved. Disable what's *unused*, not what *looks
  expensive*.
- **CLIs trade one cost for another.** No tool-listing footprint, but the
  agent has to remember the syntax and sometimes guesses wrong, costing a few
  retry turns. For widely-known tools this is a good trade; for an obscure
  internal API, an MCP server with an explicit schema is cheaper.
- **Per-project config raises maintenance cost.** More places for
  configuration to drift.

## Expected impact

⚪ **Unmeasured in this repo.** The saving varies enormously by harness and by
how many servers you've accumulated.

What can be said structurally: on a harness that loads all schemas upfront,
the cost scales with the **total number of enabled tools**, not the number you
use. On a harness with deferred loading, most of that has already been
captured by the harness — so measure before you spend effort.

How to measure it, concretely: record `/context` with all servers enabled,
disable them all, open a new session, and record again. The difference is the
standing cost you were paying. See [`../MEASURE.md`](../MEASURE.md) before
concluding that disabling servers saved you anything.
