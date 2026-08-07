# Cấu hình agent: tầng tiết kiệm miễn phí (Tiếng Việt)

[`HARNESS.md`](HARNESS.md) trả lời *nên cài công cụ nào*. Tài liệu này trả lời
câu hỏi đến trước đó: **nên đặt nút nào, ở giá trị nào**, trước khi cài bất cứ
thứ gì.

Đây là tầng rẻ nhất trong cả kho tài liệu. Không có gì để cài, không có phụ
thuộc mới, không có điểm hỏng mới. Trên Codex và Gemini, một dòng cấu hình xử
lý đúng nguyên nhân đắt nhất (3.1 — output tool quá lớn) mà cả một công cụ bên
thứ ba đã đo ra **+7.6%** khi cố làm thay.

> ⚪ **Trạng thái bằng chứng: chưa đo.** Kho này chưa chạy A/B trên bất kỳ giá
> trị nào dưới đây. Tên khóa và **giá trị mặc định** lấy từ tài liệu nhà cung
> cấp (tháng 8/2026, có nguồn ở cuối trang). **Giá trị đề xuất** là điểm khởi
> đầu suy ra từ [`CAUSE.md`](CAUSE.md), không phải kết quả đo. Hãy đọc
> [`PROOF.md`](PROOF.md) để hiểu vì sao sự phân biệt này quan trọng ở đây.

Một lưu ý trước khi vặn bất cứ nút nào: **nút nào đáng vặn tùy thuộc bạn đang
tiêu đơn vị gì.** Trên hạn ngạch tính theo request, siết trần output không
giúp bạn tiết kiệm gì cả. Xem [`BILLING.md`](BILLING.md).

---

## Ba nút quan trọng, theo đúng thứ tự

Chỉnh theo thứ tự này. Mỗi nút sau chỉ đáng động tới khi nút trước đã đặt xong.

| # | Nút | Nguyên nhân | Vì sao đứng ở vị trí này |
| --- | --- | --- | --- |
| 1 | **Trần output tool** | 3.1 | Một lần `npm test` ồn ào có thể đắt hơn cả ngày prompt |
| 2 | **Điểm kích hoạt nén** | 2.1 + 1.3 | Nén quá sớm phá cache liên tục; quá muộn thì trả tiền cho lịch sử chết |
| 3 | **Reasoning + verbosity** | 5.1, 5.2 | Token sinh ra đắt gấp ~5 lần token đọc vào |

Còn một nút thứ tư không nằm trong file cấu hình nào: **tắt server MCP bạn
không dùng hôm nay** (nguyên nhân 3.4). Trên Cline nó thường là khoản tiết
kiệm lớn nhất, vì schema MCP bị nhét vào *mọi* request.

### Bẫy đơn vị: cùng một con số, bốn ý nghĩa

Trước khi chép giá trị từ agent này sang agent khác, hãy kiểm tra đơn vị.

| Agent | Trần output đếm bằng | `20000` nghĩa là |
| --- | --- | --- |
| Claude Code (`BASH_MAX_OUTPUT_LENGTH`) | **ký tự** | ~5.000 token |
| Claude Code (`MAX_MCP_OUTPUT_TOKENS`) | **token** | 20.000 token |
| Codex CLI (`tool_output_token_limit`) | **token** | 20.000 token |
| Gemini CLI (`truncateToolOutputThreshold`) | **ký tự** | ~5.000 token |

Chênh lệch bốn lần giữa hai cột. Đây là cách phổ biến nhất để vô tình đặt trần
chặt hơn dự định gấp bốn lần — rồi rơi thẳng vào bẫy ở mục
[Chỉnh quá tay](#chỉnh-quá-tay-tốn-tiền-theo-chiều-ngược-lại) bên dưới.

---

## Claude Code

Cấu hình ở `~/.claude/settings.json` (hoặc `.claude/settings.json` theo dự án).
Các biến môi trường đặt trong khối `env` để chúng được áp dụng cho mọi phiên.

```json
{
  "effortLevel": "medium",
  "autoCompactEnabled": true,
  "claudeMdExcludes": ["**/node_modules/**", "**/vendor/**"],
  "env": {
    "BASH_MAX_OUTPUT_LENGTH": "15000",
    "MAX_MCP_OUTPUT_TOKENS": "10000",
    "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": "1",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "claude-haiku-4-5-20251001"
  }
}
```

| Khóa | Mặc định | Đề xuất | Vì sao | Dấu hiệu bạn đặt sai |
| --- | --- | --- | --- | --- |
| `BASH_MAX_OUTPUT_LENGTH` | `30000` ký tự (tối đa `150000`) | `15000` | Log build/test là nguồn output rác lớn nhất | Agent chạy lại lệnh kèm `\| tail` — bạn vừa trả tiền hai lần (5.3) |
| `MAX_MCP_OUTPUT_TOKENS` | `25000` token | `10000` | Bằng đúng ngưỡng cảnh báo cố định của Claude Code | Kết quả MCP bị cắt mất phần bạn cần |
| `effortLevel` | không đặt | `medium` | `high`/`xhigh` cho việc sửa lặt vặt là 5.1 dạng thuần túy | Chất lượng tụt ở việc khó → nâng theo từng phiên, không đổi mặc định |
| `autoCompactEnabled` | `true` | **giữ `true`** | Tắt đi chỉ dời chi phí sang phiên mới, không xóa nó | — |
| `claudeMdExcludes` | không đặt | loại thư mục vendor | File memory bị nạp lại mỗi phiên (2.3) | Agent quên quy ước thật của dự án |
| `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` | tắt | `1` | Bỏ request nền không bắt buộc | Mất vài tiện ích hiển thị |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | `claude-3-5-haiku-latest` | model Haiku hiện hành | Việc nền chạy model rẻ (6.2) | — |

**Nút nén, dùng khi cần:** `autoCompactWindow` nhận số token trong khoảng
`100000`–`1000000`, và `CLAUDE_CODE_AUTOCOMPACT_PCT_OVERRIDE` (1–100) đặt phần
trăm cửa sổ đó là điểm kích hoạt. Mặc định được điều chỉnh theo model — hãy để
yên trừ khi bạn *đã đo* rằng nén đang chạy quá sớm hoặc quá muộn. Xem
[Chỉnh quá tay](#chỉnh-quá-tay-tốn-tiền-theo-chiều-ngược-lại).

**Nút thinking:** `MAX_THINKING_TOKENS=0` tắt hẳn extended thinking. Đây là con
dao cùn — nó tiết kiệm token reasoning bằng cách đánh đổi lấy rủi ro làm lại. Với
Opus 4.6/Sonnet 4.6, `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING=1` chuyển từ ngân
sách thích ứng sang ngân sách cố định; adaptive thường đã là lựa chọn rẻ hơn.

**Kiểm chứng:** `/context` cho thấy cái gì đang chiếm chỗ, `/cost` cho thấy tỷ
lệ cache read.

---

## Codex CLI

Cấu hình ở `~/.codex/config.toml`. Đây là harness có **nhiều nút số nhất** — ba
nguyên nhân đắt nhất đều sửa được ở đây, không cần cài gì.

```toml
model                          = "gpt-5.5"
model_reasoning_effort         = "medium"
model_verbosity                = "low"
model_reasoning_summary        = "concise"
tool_output_token_limit        = 8000
model_auto_compact_token_limit = 300000   # ≤ 90% của model_context_window
```

| Khóa | Mặc định | Đề xuất | Vì sao |
| --- | --- | --- | --- |
| `tool_output_token_limit` | không đặt | `8000` | Ngân sách token cho **một** kết quả tool. Đây là bản sửa trực tiếp cho nguyên nhân 3.1 |
| `model_auto_compact_token_limit` | không đặt | ≤ 90% cửa sổ | Trên 90% thì lượt cuối trước khi nén có nguy cơ không đủ chỗ |
| `model_auto_compact_token_limit_scope` | `total` | `total` | `body_after_prefix` chỉ đếm phần tăng thêm sau tiền tố — hữu ích khi prompt hệ thống của bạn rất lớn |
| `model_reasoning_effort` | không đặt | `medium` | `xhigh` tồn tại vì đôi khi cần, không phải để dùng mặc định (5.1) |
| `model_verbosity` | không đặt | `low` | Không harness nào khác cho bạn nút này. Nó thay thế trọn vẹn một "skill viết cộc lốc" (5.2) |
| `model_reasoning_summary` | không đặt | `concise` hoặc `none` | Bản tóm tắt reasoning **là token sinh ra và bị tính tiền**. `none` là khoản tiết kiệm thật |

**Profile chính là bản đồ model/effort của bạn.** Codex đọc override theo
profile từ `$CODEX_HOME/<tên>.config.toml`, chọn bằng `--profile <tên>`. Đó
chính là "bản đồ model & effort tĩnh theo vai trò" trong
[`setups/recommended-setup.md`](setups/recommended-setup.md), nhưng đã có sẵn:

```toml
# ~/.codex/review.config.toml — vai trò đọc-hiểu, không sửa file
model_reasoning_effort = "low"
model_verbosity        = "low"
tool_output_token_limit = 4000
```

**Cạm bẫy:** `hide_agent_reasoning` **không** tiết kiệm token. Nó chỉ ẩn sự kiện
reasoning khỏi TUI và output của `codex exec`; model vẫn sinh ra và bạn vẫn trả
tiền. Muốn cắt thật thì dùng `model_reasoning_effort` hoặc
`model_reasoning_summary`. Tương tự, `history.max_bytes` giới hạn *file lịch sử
trên đĩa*, không liên quan gì tới context.

**Kiểm chứng:** `/status`.

---

## Gemini CLI

Cấu hình ở `~/.gemini/settings.json`. Đây là harness có **nhiều tầng quản lý
context nhất** — và cũng là nơi mặc định dễ khiến bạn tốn tiền nhất.

```json
{
  "tools": {
    "truncateToolOutputThreshold": 20000
  },
  "model": {
    "compressionThreshold": 0.75
  },
  "contextManagement": {
    "historyWindow": {
      "maxTokens": 150000,
      "retainedTokens": 40000
    },
    "tools": {
      "distillation": {
        "maxOutputTokens": 10000,
        "summarizationThresholdTokens": 20000
      }
    }
  }
}
```

| Khóa | Mặc định | Đề xuất | Vì sao |
| --- | --- | --- | --- |
| `tools.truncateToolOutputThreshold` | `40000` ký tự | `20000` | ~10.000 token cho một kết quả tool vẫn là rất rộng rãi |
| `model.compressionThreshold` | `0.5` | `0.75` | **Nút đáng chú ý nhất trong tài liệu này** — xem bên dưới |
| `model.summarizeToolOutput` | không đặt | bật *sau*, theo từng tool | Tóm tắt bằng LLM tự nó tốn token; cắt thì miễn phí |
| `contextManagement.historyWindow.retainedTokens` | `40000` | giữ | Phần luôn được giữ lại sau khi nén |
| `distillation.summarizationThresholdTokens` | `20000` | giữ | Ngưỡng để output đã cắt được đem đi tóm tắt |
| `model.maxSessionTurns` | `-1` (không giới hạn) | `-1` | Cắt theo lượt là công cụ cùn; các nút token phía trên chính xác hơn |

**Vì sao `compressionThreshold` đáng để ý.** Mặc định `0.5` nghĩa là Gemini nén
khi context đầy một nửa. Mỗi lần nén làm hai việc tốn tiền: nó **phá tiền tố
cache** (lượt kế tiếp trả giá đầy đủ, nguyên nhân 1.3) và bản thân nó là một
lượt gọi LLM để tóm tắt. Kích hoạt ở 50% khiến bạn trả cả hai khoản đó thường
xuyên hơn mức cần thiết. Nâng lên `0.75` cắt bớt đáng kể số lần nén.

Đừng nâng quá `0.8`: bạn cần đủ chỗ trống để lượt cuối cùng trước khi nén vẫn
vừa cửa sổ. Và hãy nhớ `historyWindow.maxTokens` (`150000`) là một ngưỡng
*riêng* — ⚪ kho này chưa xác minh cái nào thắng khi hai ngưỡng mâu thuẫn, nên
hãy đặt chúng nhất quán với nhau.

**Thứ tự bật rất quan trọng:** cắt (`truncateToolOutputThreshold`) trước, tóm
tắt (`summarizeToolOutput`, distillation) sau. Cắt là thao tác cục bộ, miễn phí.
Tóm tắt là một lượt gọi model. Bật tóm tắt trước khi đặt trần cắt tức là bạn
đang trả tiền để tóm tắt thứ lẽ ra chỉ cần vứt đi.

**Kiểm chứng:** `/stats`.

---

## Cline

Cline là ngoại lệ, và cần nói thẳng: **nó gần như không có nút số nào.** Không
có trần output tool cấu hình được, không có ngưỡng nén cấu hình được. Auto
Compact chạy theo logic nội bộ dựa trên cửa sổ context của model; yêu cầu thêm
giới hạn dòng output terminal vẫn đang mở
([cline/cline#4002](https://github.com/cline/cline/issues/4002)).

Vì vậy trên Cline, "cấu hình" nghĩa là **các lựa chọn**, không phải các con số.
Theo thứ tự tác động:

1. **Nhà cung cấp có caching.** Cline là BYO-provider, nên đây là quyết định
   đắt nhất bạn đưa ra. Provider không hỗ trợ prompt caching khiến bạn trả giá
   đầy đủ cho mỗi lượt gửi lại (nguyên nhân 1.1) — lớn hơn mọi nút chỉnh trong
   cả tài liệu này.
2. **Tắt server MCP không dùng hôm nay.** Schema MCP bị nhét vào *mọi* request
   và Cline không có tải tool trì hoãn (nguyên nhân 3.4). Đây là khoản tiết
   kiệm lớn nhất tốn 0 công.
3. **Plan/Act với hai model.** Model mạnh để lập kế hoạch, model rẻ để thực
   thi. Đây là router có sẵn của bạn (nguyên nhân 6.2) — không cần công cụ định
   tuyến nào khác.
4. **`.clineignore`** để `build/`, `dist/`, `node_modules/`, snapshot và file
   khóa không bao giờ lọt vào context.
5. **`.clinerules` gọn, và đóng băng giữa task.** Sửa rules giữa phiên là buộc
   xây lại cache toàn phiên (nguyên nhân 1.3).

**Kiểm chứng:** Cline có khả năng quan sát tốt nhất trong bốn agent — cache
read/write và chi phí hiển thị ngay theo từng task. Dùng nó.

Chi tiết đầy đủ: [`setups/coding-setup-cline.md`](setups/coding-setup-cline.md).

---

## Cùng một nút, bốn cái tên

| Nguyên nhân | Claude Code | Codex CLI | Gemini CLI | Cline |
| --- | --- | --- | --- | --- |
| **3.1** trần output tool | `BASH_MAX_OUTPUT_LENGTH`, `MAX_MCP_OUTPUT_TOKENS` | `tool_output_token_limit` | `tools.truncateToolOutputThreshold` | ❌ không có |
| **2.1** điểm nén | `autoCompactWindow`, `CLAUDE_CODE_AUTOCOMPACT_PCT_OVERRIDE` | `model_auto_compact_token_limit` | `model.compressionThreshold`, `historyWindow.maxTokens` | ❌ tự động |
| **2.2** giữ lại sau nén | ⚪ tự động | ⚪ theo ngân sách lưu trữ | `historyWindow.retainedTokens` | ❌ |
| **5.1** reasoning | `effortLevel`, `MAX_THINKING_TOKENS` | `model_reasoning_effort`, `model_reasoning_summary` | ⚪ | ngân sách thinking (model Anthropic) |
| **5.2** verbosity | `outputStyle` | `model_verbosity` | ⚪ | ❌ chỉ qua prompt |
| **3.4** chi phí schema | tắt server MCP, `claudeMdExcludes` | tắt server MCP | hook trước khi chọn tool | tắt server MCP (**quan trọng nhất ở đây**) |
| **6.2** định tuyến | `model`, `fallbackModel`, `ANTHROPIC_DEFAULT_HAIKU_MODEL` | profile (`--profile`) | ⚪ theo phiên | Plan/Act |

Cột Cline gần như trống, và đó chính là điều bảng này cần nói ra. Trên ba agent
kia, phần lớn công việc tối ưu là **đặt giá trị**. Trên Cline, đó là **kỷ luật
vận hành** — cùng những nguyên nhân đó, nhưng không có nút nào để vặn.

---

## Chỉnh quá tay: tốn tiền theo chiều ngược lại

Mọi nút trong tài liệu này đều có rủi ro hai chiều. Nguyên nhân **5.3 — bị cắt
output giữa chừng rồi thử lại** tồn tại chính là vì lý do này.

| Chỉnh quá tay | Chuyện gì xảy ra | Bạn trả giá thế nào |
| --- | --- | --- |
| Trần output quá thấp | Agent chạy lại lệnh với `head`/`grep`/`tail` | Hai vòng tool thay vì một |
| Ngưỡng nén quá thấp | Nén chạy liên tục | Mỗi lần đều phá cache (1.3) **và** tốn một lượt gọi LLM |
| Tắt hẳn auto-compact | Phiên đâm vào trần cứng | Bạn phải bắt đầu lại — cold start còn đắt hơn (6.5) |
| Reasoning về `minimal` cho việc khó | Đáp án sai, phải sửa lại | Ba lượt rẻ đắt hơn một lượt đắt |
| `MAX_THINKING_TOKENS=0` toàn cục | Không còn extended thinking ở đâu cả | Chi phí làm lại, ẩn mình dưới dạng "phiên dài hơn" |

**Quy tắc:** nới rộng khi thấy dấu hiệu chạy lại, siết chặt khi thấy dấu hiệu
lãng phí. Đừng đặt cả hai cùng lúc rồi đoán cái nào có tác dụng.

Và một nhóm thiết lập *trông như* tiết kiệm nhưng không:

| Thiết lập | Nó thực sự làm gì |
| --- | --- |
| `hide_agent_reasoning` (Codex) | Ẩn reasoning khỏi màn hình. Token vẫn được sinh ra và tính tiền |
| `history.max_bytes` (Codex) | Giới hạn file lịch sử **trên đĩa**. Không ảnh hưởng context |
| `CLAUDE_CODE_SKIP_PROMPT_HISTORY` | Không ghi transcript. Tiết kiệm ổ đĩa, không tiết kiệm token |
| `DISABLE_TELEMETRY` | Tắt gửi telemetry. Bạn đang tắt đúng thứ giúp mình *đo* chi phí |

---

## Cách kiểm chứng một thiết lập đã có tác dụng

Đặt giá trị rồi tin là xong thì cũng giống như cài ruleset ở tầng file rules —
tầng đã đo được là **kích hoạt 0/10 phiên**. Hãy quan sát nó chạy.

1. **Trần output:** chạy một lệnh in ~50.000 dòng (`seq 1 50000`). Model có
   nhìn thấy toàn bộ không? Có chuỗi báo cắt không?
2. **Điểm nén:** chạy một phiên dài, ghi lại vị trí nén thực sự kích hoạt. So
   với giá trị bạn đặt.
3. **Reasoning/verbosity:** giao cùng một tác vụ hai lần, hai giá trị khác
   nhau, so số token output.
4. **Nói chung:** `/context` + `/cost` (Claude Code), `/status` (Codex),
   `/stats` (Gemini), thanh chi phí theo task (Cline).

Quy trình đo lường đầy đủ:
[`solutions/token-counting.md`](solutions/token-counting.md).

Đặt xong tầng này rồi thì phần dư địa còn lại nằm ở chỗ khác: **cách bạn làm
việc trong mỗi phiên**. Cấu hình chỉ đặt trần cho thiệt hại —
[`WORKFLOW.md`](WORKFLOW.md) quyết định bạn có chạm tới trần đó hay không.

---

## Nguồn

- [Claude Code settings](https://code.claude.com/docs/en/settings) ·
  [Claude Code environment variables](https://code.claude.com/docs/en/env-vars) ·
  [Claude Code MCP](https://code.claude.com/docs/en/mcp)
- [Codex config reference](https://learn.chatgpt.com/docs/config-file/config-reference)
- [Gemini CLI configuration](https://geminicli.com/docs/reference/configuration/)
- [Cline docs](https://docs.cline.bot/) ·
  [cline/cline#4002 — giới hạn dòng output terminal](https://github.com/cline/cline/issues/4002)

---

# Agent configuration: the free tier

[`HARNESS.md`](HARNESS.md) answers *which tools to install*. This document
answers the question that comes before it: **which dials to set, and to what**,
before you install anything at all.

This is the cheapest tier in the whole repo. Nothing to install, no new
dependency, no new failure mode. On Codex and Gemini, one line of configuration
handles the most expensive cause there is (3.1 — oversized tool output) — the
same job a third-party tool measured at **+7.6%** trying to do.

> ⚪ **Evidence status: unmeasured.** This repo has run no A/B on any value
> below. Key names and **defaults** come from vendor documentation (August
> 2026, sourced at the bottom). **Suggested values** are starting points
> derived from [`CAUSE.md`](CAUSE.md), not measured results. Read
> [`PROOF.md`](PROOF.md) for why that distinction matters here.

One note before turning any dial: **which dials are worth turning depends on
the unit you spend.** On a request-based quota, tightening output caps saves
you nothing at all. See [`BILLING.md`](BILLING.md).

---

## The three dials that matter, in order

Set them in this order. Each one is only worth touching once the one before it
is in place.

| # | Dial | Cause | Why it ranks here |
| --- | --- | --- | --- |
| 1 | **Tool-output cap** | 3.1 | One noisy `npm test` can cost more than a day of prompts |
| 2 | **Compaction trigger point** | 2.1 + 1.3 | Too early and you shred the cache; too late and you pay for dead history |
| 3 | **Reasoning + verbosity** | 5.1, 5.2 | Generated tokens cost roughly 5× what read tokens cost |

There's a fourth dial that lives in no config file: **turn off MCP servers you
aren't using today** (cause 3.4). On Cline it's usually the single largest
saving available, because MCP schemas are injected into *every* request.

### The units trap: one number, four meanings

Before copying a value from one agent to another, check the unit.

| Agent | Output cap counts | `20000` means |
| --- | --- | --- |
| Claude Code (`BASH_MAX_OUTPUT_LENGTH`) | **characters** | ~5,000 tokens |
| Claude Code (`MAX_MCP_OUTPUT_TOKENS`) | **tokens** | 20,000 tokens |
| Codex CLI (`tool_output_token_limit`) | **tokens** | 20,000 tokens |
| Gemini CLI (`truncateToolOutputThreshold`) | **characters** | ~5,000 tokens |

A 4× difference between those columns. This is the most common way to
accidentally set a cap four times tighter than you meant — and land straight in
[Over-tuning](#over-tuning-paying-in-the-other-direction) below.

---

## Claude Code

Configured in `~/.claude/settings.json` (or `.claude/settings.json` per
project). Environment variables go in the `env` block so they apply to every
session.

```json
{
  "effortLevel": "medium",
  "autoCompactEnabled": true,
  "claudeMdExcludes": ["**/node_modules/**", "**/vendor/**"],
  "env": {
    "BASH_MAX_OUTPUT_LENGTH": "15000",
    "MAX_MCP_OUTPUT_TOKENS": "10000",
    "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": "1",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "claude-haiku-4-5-20251001"
  }
}
```

| Key | Default | Suggested | Why | How you'd know it's wrong |
| --- | --- | --- | --- | --- |
| `BASH_MAX_OUTPUT_LENGTH` | `30000` chars (max `150000`) | `15000` | Build and test logs are the largest source of junk output | The agent re-runs the command with `\| tail` — you just paid twice (5.3) |
| `MAX_MCP_OUTPUT_TOKENS` | `25000` tokens | `10000` | Matches Claude Code's own fixed warning threshold | MCP results get cut off before the part you needed |
| `effortLevel` | unset | `medium` | `high`/`xhigh` on routine edits is cause 5.1 in its purest form | Quality drops on hard work → raise it per session, not as the default |
| `autoCompactEnabled` | `true` | **leave `true`** | Turning it off relocates the cost to a new session, it doesn't remove it | — |
| `claudeMdExcludes` | unset | exclude vendor trees | Memory files are re-read every session (2.3) | The agent forgets conventions that genuinely matter |
| `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` | off | `1` | Skips optional background requests | You lose some display niceties |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | `claude-3-5-haiku-latest` | current Haiku | Background work runs on the cheap model (6.2) | — |

**Compaction dials, when you need them:** `autoCompactWindow` takes a token
count between `100000` and `1000000`, and `CLAUDE_CODE_AUTOCOMPACT_PCT_OVERRIDE`
(1–100) sets the percentage of that window where compaction fires. The defaults
are model-tuned — leave them alone unless you have *measured* that compaction
fires too early or too late. See
[Over-tuning](#over-tuning-paying-in-the-other-direction).

**Thinking dials:** `MAX_THINKING_TOKENS=0` disables extended thinking
entirely. It's a blunt instrument — it trades reasoning tokens for rework risk.
On Opus 4.6/Sonnet 4.6, `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING=1` switches from
an adaptive budget to a fixed one; adaptive is usually already the cheaper
choice.

**Verify with:** `/context` for what's occupying the window, `/cost` for the
cache-read ratio.

---

## Codex CLI

Configured in `~/.codex/config.toml`. This is the harness with the **most
numeric dials** — all three of the most expensive causes are fixable here with
nothing installed.

```toml
model                          = "gpt-5.5"
model_reasoning_effort         = "medium"
model_verbosity                = "low"
model_reasoning_summary        = "concise"
tool_output_token_limit        = 8000
model_auto_compact_token_limit = 300000   # ≤ 90% of model_context_window
```

| Key | Default | Suggested | Why |
| --- | --- | --- | --- |
| `tool_output_token_limit` | unset | `8000` | Token budget for a **single** tool result. The direct fix for cause 3.1 |
| `model_auto_compact_token_limit` | unset | ≤ 90% of the window | Above 90%, the last turn before compaction risks not fitting |
| `model_auto_compact_token_limit_scope` | `total` | `total` | `body_after_prefix` counts only growth past the prefix — useful when your system prompt is very large |
| `model_reasoning_effort` | unset | `medium` | `xhigh` exists because it's occasionally needed, not as a default (5.1) |
| `model_verbosity` | unset | `low` | No other harness gives you this dial. It fully replaces a "be terse" skill (5.2) |
| `model_reasoning_summary` | unset | `concise` or `none` | Reasoning summaries **are generated, billed tokens**. `none` is a real saving |

**Profiles are your model/effort map.** Codex reads per-profile overrides from
`$CODEX_HOME/<name>.config.toml`, selected with `--profile <name>`. That is
precisely the "static model & effort map per agent role" from
[`setups/recommended-setup.md`](setups/recommended-setup.md), already built in:

```toml
# ~/.codex/review.config.toml — a read-only reviewing role
model_reasoning_effort  = "low"
model_verbosity         = "low"
tool_output_token_limit = 4000
```

**The trap:** `hide_agent_reasoning` does **not** save tokens. It suppresses
reasoning events in the TUI and in `codex exec` output; the model still
generates them and you are still billed. To cut them for real, use
`model_reasoning_effort` or `model_reasoning_summary`. Likewise
`history.max_bytes` caps the history file *on disk* and has nothing to do with
context.

**Verify with:** `/status`.

---

## Gemini CLI

Configured in `~/.gemini/settings.json`. This is the **most layered**
context-management system of the four — and the one whose defaults are most
likely to cost you money.

```json
{
  "tools": {
    "truncateToolOutputThreshold": 20000
  },
  "model": {
    "compressionThreshold": 0.75
  },
  "contextManagement": {
    "historyWindow": {
      "maxTokens": 150000,
      "retainedTokens": 40000
    },
    "tools": {
      "distillation": {
        "maxOutputTokens": 10000,
        "summarizationThresholdTokens": 20000
      }
    }
  }
}
```

| Key | Default | Suggested | Why |
| --- | --- | --- | --- |
| `tools.truncateToolOutputThreshold` | `40000` chars | `20000` | ~10,000 tokens for one tool result is still generous |
| `model.compressionThreshold` | `0.5` | `0.75` | **The most consequential dial in this document** — see below |
| `model.summarizeToolOutput` | unset | enable *after*, per tool | LLM summarization costs tokens itself; truncation is free |
| `contextManagement.historyWindow.retainedTokens` | `40000` | keep | What always survives a compression pass |
| `distillation.summarizationThresholdTokens` | `20000` | keep | Threshold above which truncated output gets summarized |
| `model.maxSessionTurns` | `-1` (unlimited) | `-1` | Turn caps are a blunt instrument; the token dials above are more precise |

**Why `compressionThreshold` matters most.** The default of `0.5` means Gemini
compresses when the context is half full. Every compression does two expensive
things: it **breaks the cache prefix** (the next turn pays full price, cause
1.3) and it is itself an LLM call to write the summary. Firing at 50% makes you
pay both of those far more often than necessary. Raising it to `0.75`
meaningfully cuts the number of compressions.

Don't push past `0.8`: you need enough headroom that the final turn before
compaction still fits the window. And note that `historyWindow.maxTokens`
(`150000`) is a *separate* threshold — ⚪ this repo hasn't verified which one
wins when the two disagree, so set them consistently with each other.

**Enablement order matters:** truncate (`truncateToolOutputThreshold`) first,
summarize (`summarizeToolOutput`, distillation) second. Truncation is a free
local operation. Summarization is a model call. Enabling summarization before
setting the truncation cap means paying to summarize what you only needed to
discard.

**Verify with:** `/stats`.

---

## Cline

Cline is the exception, and it deserves to be said plainly: **it has almost no
numeric dials.** No configurable tool-output cap, no configurable compaction
threshold. Auto Compact runs on internal logic derived from the model's context
window; the request for a terminal-output line limit is still open
([cline/cline#4002](https://github.com/cline/cline/issues/4002)).

So on Cline, "configuration" means **choices**, not numbers. In impact order:

1. **A provider with caching.** Cline is BYO-provider, which makes this the
   most expensive decision you make. A provider without prompt caching means
   paying full price on every re-send (cause 1.1) — larger than every dial in
   this document combined.
2. **Turn off MCP servers you aren't using today.** MCP schemas are injected
   into *every* request and Cline has no deferred tool loading (cause 3.4).
   This is the largest zero-effort saving available.
3. **Plan/Act with two models.** Strong model to plan, cheap model to execute.
   This is your router, already built in (cause 6.2) — no routing tool needed.
4. **`.clineignore`** so `build/`, `dist/`, `node_modules/`, snapshots and lock
   files never reach context.
5. **A lean `.clinerules`, frozen mid-task.** Editing rules mid-session forces
   a session-wide cache rebuild (cause 1.3).

**Verify with:** Cline has the best observability of the four — cache
read/write and cost are displayed per task. Use it.

Full detail: [`setups/coding-setup-cline.md`](setups/coding-setup-cline.md).

---

## The same dial, four names

| Cause | Claude Code | Codex CLI | Gemini CLI | Cline |
| --- | --- | --- | --- | --- |
| **3.1** tool-output cap | `BASH_MAX_OUTPUT_LENGTH`, `MAX_MCP_OUTPUT_TOKENS` | `tool_output_token_limit` | `tools.truncateToolOutputThreshold` | ❌ none |
| **2.1** compaction point | `autoCompactWindow`, `CLAUDE_CODE_AUTOCOMPACT_PCT_OVERRIDE` | `model_auto_compact_token_limit` | `model.compressionThreshold`, `historyWindow.maxTokens` | ❌ automatic |
| **2.2** retention after compaction | ⚪ automatic | ⚪ via storage budget | `historyWindow.retainedTokens` | ❌ |
| **5.1** reasoning | `effortLevel`, `MAX_THINKING_TOKENS` | `model_reasoning_effort`, `model_reasoning_summary` | ⚪ | thinking budget (Anthropic models) |
| **5.2** verbosity | `outputStyle` | `model_verbosity` | ⚪ | ❌ prompt only |
| **3.4** schema overhead | disable MCP servers, `claudeMdExcludes` | disable MCP servers | pre-tool-selection hooks | disable MCP servers (**matters most here**) |
| **6.2** routing | `model`, `fallbackModel`, `ANTHROPIC_DEFAULT_HAIKU_MODEL` | profiles (`--profile`) | ⚪ per session | Plan/Act |

The Cline column is nearly empty, and that's what this table is for. On the
other three, most of the optimization work is **setting values**. On Cline it's
**operational discipline** — the same causes, with no dials to turn.

---

## Over-tuning: paying in the other direction

Every dial here carries two-sided risk. Cause **5.3 — truncation-and-retry
cycles** exists for exactly this reason.

| Over-tuned | What happens | How you pay |
| --- | --- | --- |
| Output cap too low | The agent re-runs the command with `head`/`grep`/`tail` | Two tool round-trips instead of one |
| Compaction threshold too low | Compaction fires constantly | Each one breaks the cache (1.3) **and** costs an LLM call |
| Auto-compaction disabled entirely | The session hits the hard limit | You start over — cold start costs more (6.5) |
| Reasoning at `minimal` for hard work | Wrong answer, then rework | Three cheap turns cost more than one expensive one |
| `MAX_THINKING_TOKENS=0` globally | No extended thinking anywhere | Rework cost, disguised as "longer sessions" |

**The rule:** loosen when you see retries, tighten when you see waste. Don't set
both at once and guess which one worked.

And a category of settings that *look* like savings but aren't:

| Setting | What it actually does |
| --- | --- |
| `hide_agent_reasoning` (Codex) | Hides reasoning from the screen. The tokens are still generated and billed |
| `history.max_bytes` (Codex) | Caps the history file **on disk**. No effect on context |
| `CLAUDE_CODE_SKIP_PROMPT_HISTORY` | Stops transcript writes. Saves disk, not tokens |
| `DISABLE_TELEMETRY` | Stops telemetry submission. You're disabling the thing that lets you *measure* cost |

---

## How to verify a setting took effect

Setting a value and trusting it is the same mistake as installing a ruleset at
the rules-file tier — the tier measured at **0 activations in 10 sessions**.
Watch it work.

1. **Output caps:** run something that prints ~50,000 lines (`seq 1 50000`).
   Does the model see all of it? Is there a truncation marker?
2. **Compaction point:** run a long session and record where compaction
   actually fired. Compare against the value you set.
3. **Reasoning/verbosity:** give the same task twice at two different values
   and compare output token counts.
4. **In general:** `/context` + `/cost` (Claude Code), `/status` (Codex),
   `/stats` (Gemini), the per-task cost bar (Cline).

Full measurement procedure:
[`solutions/token-counting.md`](solutions/token-counting.md).

Once this tier is set, the remaining headroom is somewhere else: **how you work
within each session**. Configuration only caps the damage —
[`WORKFLOW.md`](WORKFLOW.md) decides whether you approach the cap at all.

---

## Sources

- [Claude Code settings](https://code.claude.com/docs/en/settings) ·
  [Claude Code environment variables](https://code.claude.com/docs/en/env-vars) ·
  [Claude Code MCP](https://code.claude.com/docs/en/mcp)
- [Codex config reference](https://learn.chatgpt.com/docs/config-file/config-reference)
- [Gemini CLI configuration](https://geminicli.com/docs/reference/configuration/)
- [Cline docs](https://docs.cline.bot/) ·
  [cline/cline#4002 — terminal output line limit](https://github.com/cline/cline/issues/4002)
