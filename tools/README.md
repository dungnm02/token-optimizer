# Hồ sơ từng công cụ (Tiếng Việt)

Trong khi [`../solutions/`](../solutions/README.md) mô tả *kỹ thuật* và
[`../PROOF.md`](../PROOF.md) tổng hợp *bằng chứng*, thư mục này đi sâu vào
**từng công cụ cụ thể**: nó là gì, chạy thế nào, cài ra sao, dùng lúc nào và
khi nào thì đừng.

| Công cụ | Cơ chế | Đã đo | Nên dùng? |
| --- | --- | --- | --- |
| [`ponytail.md`](ponytail.md) | Ruleset ngăn agent xây thừa | **−10.3% chi phí** (p=0.004) | ✅ Thử đầu tiên |
| [`codegraph.md`](codegraph.md) | Knowledge graph local, truy vấn qua MCP | **−69% token, −60% chi phí** | ✅ Nếu repo trên ~1.000 file |
| [`caveman.md`](caveman.md) | Nén văn bản (thư viện) / ép viết cộc lốc (skill) | **8.5%** cho skill, rủi ro đuôi nặng | ⚠️ Chỉ dùng thư viện, cho văn bản tĩnh |
| [`rtk.md`](rtk.md) | Nén output shell qua hook | **+7.6% chi phí** trên Claude Code | ❌ Đừng cài trên Claude Code |

**Quy luật xuyên suốt cả bốn:** *ngăn công việc thắng nén công việc.* Hai
công cụ đứng đầu bảng ngăn không cho token được sinh ra; hai công cụ cuối
bảng bóp nhỏ token sau khi đã sinh ra, và cả hai đều hụt so với quảng cáo
5–10 lần.

> Mọi con số ở đây đều là **giá trị biên so với Claude Code**, không phải
> thuộc tính của công cụ. Đọc phần "baseline chính là harness" trong
> [`../PROOF.md`](../PROOF.md) trước khi mang các con số này sang harness
> khác.

## Công cụ này có hợp với harness của bạn không?

Cả bốn công cụ đều được đo trên Claude Code, đơn giản vì đó là nơi có phép đo
— **không** vì chúng là công cụ dành cho Claude Code. Kết quả của chúng phụ
thuộc vào sáu thuộc tính của harness, và harness của bạn có thể khác ở bất kỳ
thuộc tính nào.

| Thuộc tính harness | Vì sao quan trọng | Ảnh hưởng tới |
| --- | --- | --- |
| **Tự cắt bớt output tool cỡ lớn?** | Nếu đã cắt sẵn, bộ nén output chỉ đang nén thứ đằng nào cũng bị vứt | RTK |
| **Có tool đọc/tìm file riêng, không qua shell?** | Lớp bọc ở tầng shell không bao giờ nhìn thấy lưu lượng đó | RTK |
| **Bật prompt caching mặc định?** | Byte gửi lại chỉ bị tính ~1/10 giá, nên đếm byte sẽ thổi phồng khoản tiết kiệm | RTK, Caveman |
| **Nói được MCP?** | Là đường agent truy vấn graph | CodeGraph |
| **Tiêm rule tất định (hook/plugin), không chỉ rules file?** | Rules file thụ động đã kích hoạt **0/10 phiên** | Ponytail, skill Caveman |
| **System prompt đã ép ngắn gọn sẵn?** | Lấy mất phần dư địa mà một rule "viết cộc lốc" định khai thác | skill Caveman |

Bảng đối chiếu — ô ghi *kiểm tra* nghĩa là kho này chưa xác minh được, hãy tự
chạy phép thử bên dưới:

| Harness | Cắt output | Tool file riêng | MCP | Tiêm tất định |
| --- | --- | --- | --- | --- |
| Claude Code | ✅ có (đã đo) | ✅ Read/Grep/Glob | ✅ | ✅ hook |
| Codex CLI | kiểm tra | phần lớn qua shell | ✅ | ✅ plugin |
| Cursor / Windsurf | kiểm tra | ✅ | ✅ | ❌ chỉ rules file |
| Cline / Roo / Kilo | kiểm tra | ✅ read_file/search_files | ✅ | ❌ chỉ rules file |
| Gemini CLI | kiểm tra | ✅ | ✅ | ✅ extension |
| Copilot | kiểm tra | ✅ | kiểm tra | ❌ chỉ instructions |
| opencode | kiểm tra | ✅ | ✅ | kiểm tra |
| Aider | ít liên quan (bạn tự dán) | repo map, không có vòng lặp tool | ❌ | bạn tự viết prompt |
| App tự dựng trên API | **bạn quyết định** | **bạn quyết định** | **bạn quyết định** | **bạn quyết định** |

Cột cuối cùng mới là điểm mấu chốt: nếu bạn tự gọi API, không có harness nào
cắt output thay bạn, không có system prompt nào ép ngắn gọn thay bạn, và
không có gì tự kích hoạt ruleset. Cả bốn kết luận đo trên Claude Code đều có
thể đảo dấu, và **RTK là ví dụ rõ nhất** — thứ làm hóa đơn tăng ở đây lại có
dư địa thật ở đó.

### Tự kiểm tra harness của bạn trong 15 phút

1. **Có cắt output không?** Chạy một lệnh in ~50.000 dòng
   (`seq 1 50000`). Xem model có nhìn thấy toàn bộ không, hoặc tìm chuỗi kiểu
   "output truncated" trong log. Có cắt → bộ nén output mất phần lớn giá trị.
2. **Bao nhiêu phần context đi qua shell?** Ghi log lệnh tool trong một phiên
   thật, cộng số ký tự kết quả theo từng loại tool. Trên Claude Code, Bash chỉ
   mang chưa tới 20% — đó là lý do trần của RTK sụp xuống ≈3%.
3. **Ruleset có thực sự kích hoạt không?** Giao một tác vụ mời gọi xây thừa và
   xem agent có tuân ruleset không. Nếu không quan sát được, coi như nó chưa
   được cài.

---

# Tool profiles

Where [`../solutions/`](../solutions/README.md) describes *techniques* and
[`../PROOF.md`](../PROOF.md) aggregates *evidence*, this folder goes deep on
**individual tools**: what each one is, how it works, how to install it, when
to use it, and when not to.

| Tool | Mechanism | Measured | Adopt? |
| --- | --- | --- | --- |
| [`ponytail.md`](ponytail.md) | Ruleset preventing over-building | **−10.3% cost** (p=0.004) | ✅ Try this first |
| [`codegraph.md`](codegraph.md) | Local knowledge graph, queried over MCP | **−69% tokens, −60% cost** | ✅ If your repo is above ~1,000 files |
| [`caveman.md`](caveman.md) | Text compression (library) / terse output (skill) | **8.5%** for the skill, severe tail risk | ⚠️ Library only, on static text |
| [`rtk.md`](rtk.md) | Shell-output compression via hook | **+7.6% cost** on Claude Code | ❌ Don't install it on Claude Code |

**The pattern across all four:** *preventing work beats compressing it.* The
top two stop tokens from being produced; the bottom two squeeze tokens after
they exist, and both underdelivered by 5–10×.

> Every number here is a **marginal value against Claude Code**, not a
> property of the tool. Read "the baseline is the harness" in
> [`../PROOF.md`](../PROOF.md) before carrying these figures to another
> harness.

## Does any of this apply to your harness?

All four tools were measured on Claude Code because that's where the
measurements exist — **not** because they're Claude Code tools. Their results
turn on six properties of the harness, and yours may differ on any of them.

| Harness property | Why it matters | Affects |
| --- | --- | --- |
| **Truncates large tool output?** | If it already truncates, an output compressor is compressing what was going to be discarded | RTK |
| **Native file read/search tools, not via shell?** | A shell-level wrapper never sees that traffic | RTK |
| **Prompt caching on by default?** | Re-sent bytes bill at ~1/10, so byte counts overstate the saving | RTK, Caveman |
| **Speaks MCP?** | It's how the agent queries the graph | CodeGraph |
| **Deterministic rule injection (hooks/plugins), not just a rules file?** | A passive rules file fired **0 times in 10 sessions** | Ponytail, the Caveman skill |
| **System prompt already enforces concision?** | Removes the headroom a "be terse" rule would exploit | The Caveman skill |

Where things stand — *check* means this repo hasn't verified it, so run the
tests below yourself:

| Harness | Truncates | Own file tools | MCP | Deterministic injection |
| --- | --- | --- | --- | --- |
| Claude Code | ✅ yes (measured) | ✅ Read/Grep/Glob | ✅ | ✅ hooks |
| Codex CLI | check | mostly via shell | ✅ | ✅ plugins |
| Cursor / Windsurf | check | ✅ | ✅ | ❌ rules files only |
| Cline / Roo / Kilo | check | ✅ read_file/search_files | ✅ | ❌ rules files only |
| Gemini CLI | check | ✅ | ✅ | ✅ extensions |
| Copilot | check | ✅ | check | ❌ instructions only |
| opencode | check | ✅ | ✅ | check |
| Aider | mostly moot (you paste) | repo map, no tool loop | ❌ | you write the prompt |
| Your own app on the API | **your call** | **your call** | **your call** | **your call** |

That last row is the real point: if you're calling the API yourself, no
harness truncates for you, no system prompt enforces concision for you, and
nothing auto-activates a ruleset. Every Claude Code conclusion here can flip
sign, and **RTK is the clearest case** — the tool that raised the bill here
has genuine headroom there.

### Test your own harness in 15 minutes

1. **Does it truncate?** Run something that prints ~50,000 lines
   (`seq 1 50000`). Check whether the model sees all of it, or grep the logs
   for an "output truncated" marker. If it truncates, output compressors lose
   most of their value.
2. **How much context flows through the shell?** Log tool calls for one real
   session and sum result characters per tool type. On Claude Code, Bash
   carried under 20% — that's why RTK's ceiling collapsed to ≈3%.
3. **Does a ruleset actually activate?** Give the agent a task that invites
   over-building and see whether it follows the rules. If you can't observe
   it firing, assume it isn't installed.
