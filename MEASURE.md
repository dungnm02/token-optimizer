# Đo lường: cách tự tạo ra bằng chứng (Tiếng Việt)

[`PROOF.md`](PROOF.md) cho biết **cái gì đã được đo**. Tài liệu này nói **cách
tự đo**.

Bạn cần nó vì lý do đã nêu ngay đầu `PROOF.md`: baseline chính là harness.
Mọi phần trăm trong kho này là giá trị biên so với *một* thiết lập cụ thể —
một phiên bản agent, một model, một repo. Thiết lập của bạn khác. Cách duy
nhất để biết một thay đổi có giúp **bạn** hay không là chạy thử.

Tin tốt: quy trình đã tạo ra con số đáng tin nhất trong kho này không hề bí
ẩn, và bạn có thể chạy phiên bản thu nhỏ của nó trong một buổi chiều.

> **Trạng thái bằng chứng:** ⚪ Đây là mô tả phương pháp, không phải kết quả.
> Quy trình bên dưới mô phỏng theo thiết kế nghiên cứu đã tạo ra các mục hạng
> 🟢 A trong [`PROOF.md`](PROOF.md).

## Luật số một: đo hóa đơn, đừng đo tỷ lệ nén

Đây là sai lầm đã hạ gục nhiều tuyên bố nhất trong kho này.

RTK quảng cáo giảm 60–90%. Con số đó **có thật** — nó là tỷ lệ nén tính trên
văn bản. Khi đem đo bằng hóa đơn, kết quả là **+7,6% chi phí** (p = 0,004):
output nén khiến model kém chắc chắn hơn nên nó làm nhiều hơn, kéo theo
+13,8% số lượt và +14,3% cache read.

Bài học tổng quát:

| Đừng đo | Hãy đo |
| --- | --- |
| Tỷ lệ nén văn bản | Tổng token của cả tác vụ |
| Token đã tiết kiệm ở một bước | Chi phí từ đầu tới lúc hoàn thành |
| Kích thước context | Số lượt cần để xong việc |
| Token input | Input + output + cache write + cache read |

Một tác vụ chỉ kết thúc khi nó **xong**. Nếu một công cụ cắt 40% mỗi kết quả
tool nhưng khiến agent phải gọi thêm lượt để bù, bạn vừa trả tiền cho việc
tối ưu đó.

## Luật số hai: xác nhận can thiệp thực sự đã kích hoạt

Trước khi đo bất cứ điều gì, hãy chứng minh thứ bạn đang thử nghiệm **thực sự
đang chạy**.

Đây không phải chi tiết vụn vặt. Bộ luật của ponytail khi cài ở chế độ thụ
động đã kích hoạt **0 lần trong 10 phiên**. Nếu nhóm nghiên cứu bỏ qua bước
này, họ đã đo được "0% cải thiện" và kết luận công cụ vô dụng — trong khi
thực tế nó chưa từng chạy. Nghiên cứu đã công bố kiểm toán mức áp dụng ở
**100% nhánh treatment / 0% nhánh baseline**.

Cách kiểm toán, theo mức độ tin cậy tăng dần:

1. Kiểm tra thủ công một phiên: dấu vết của can thiệp có xuất hiện không?
2. Ghi log ở tầng gắn kết — hook có chạy không, tool có được gọi không?
3. Đếm số lần kích hoạt trên **mọi** lần chạy ở cả hai nhánh, và công bố nó.

Nếu không đếm được số lần kích hoạt, bạn không đo được công cụ. Bạn chỉ đang
đo nhiễu.

## Quy trình

### 1 — Chọn đúng một thứ để thay đổi

Một biến. Một nút cấu hình, một công cụ, một quy ước quy trình. Nếu bạn đổi
`effortLevel` **và** cài CodeGraph cùng lúc, kết quả không cho biết cái nào
có tác dụng — hoặc liệu một cái có đang che lấp tác hại của cái kia không.

### 2 — Dựng bộ tác vụ từ backlog thật của bạn

10–30 tác vụ, lấy từ công việc thật, chứ không phải bài toán đồ chơi. Mỗi tác
vụ cần:

- Một mô tả cố định, viết sẵn (dán đúng nguyên văn ở cả hai nhánh).
- Một **điều kiện hoàn thành** kiểm chứng được — test pass, file tồn tại,
  lệnh trả về 0. Không có nó thì bạn không so sánh được chi phí, vì hai nhánh
  có thể đã làm hai lượng việc khác nhau.
- Một trạng thái repo ban đầu giống hệt nhau (một commit cụ thể, hoặc một
  worktree sạch cho mỗi lần chạy).

Hãy để bộ tác vụ phản ánh đúng phân bố công việc thật. Nếu 80% ngày làm việc
của bạn là sửa lỗi hẹp, đừng dựng bộ tác vụ toàn xây tính năng mới — ponytail
đo được **−31% ở các lần xây lớn và bằng không ở chỗ code vốn đã tối giản**.
Bộ tác vụ sai sẽ cho bạn con số đúng cho một người khác.

### 3 — Đóng băng mọi thứ còn lại

Ghi lại và giữ nguyên trong suốt quá trình:

| Biến | Vì sao |
| --- | --- |
| Phiên bản agent | Harness thay đổi hành vi giữa các bản phát hành |
| Model, chính xác từng ID | "Sonnet" không phải một biến cố định |
| Mức reasoning effort | Nút tốn kém nhất trong tất cả |
| Trạng thái repo | Một commit cụ thể |
| File chỉ dẫn | Đóng băng cả `CLAUDE.md`/`AGENTS.md` |
| MCP server đang bật | Schema đi vào mọi request |

### 4 — Chạy ở chế độ headless

Chạy không tương tác (`-p` trên Claude Code, `codex exec` trên Codex). Có ba
lý do, đều quan trọng:

- Loại bỏ bạn khỏi vòng lặp. Một người ngồi lái sẽ vô thức lái nhánh mình
  thích về đích tốt hơn.
- Cho phép lặp lại. Bạn sẽ chạy mỗi tác vụ nhiều lần.
- Làm cho trạng thái cache có thể dự đoán được — mỗi lần chạy bắt đầu nguội.

### 5 — Ghép cặp, đừng gộp trung bình riêng

Chạy **cùng một tác vụ** ở cả hai nhánh, rồi so sánh trong từng cặp. Đừng
tính trung bình nhánh A rồi trung bình nhánh B rồi trừ nhau.

Lý do: độ biến thiên giữa các *tác vụ* lớn hơn nhiều so với hiệu ứng bạn đang
tìm. Một tác vụ tốn 300k token và một tác vụ tốn 20k token nằm chung một
trung bình sẽ nhấn chìm mức chênh 10%. Ghép cặp loại bỏ nguồn nhiễu đó.

Hãy đảo thứ tự chạy: A trước rồi B ở tác vụ 1, B trước rồi A ở tác vụ 2. Điều
này triệt tiêu ảnh hưởng của việc cache còn ấm và của các thay đổi phía nhà
cung cấp trong ngày.

### 6 — Chạy đủ số lần

Đây là chỗ hầu hết các thử nghiệm tự làm sụp đổ. Con số **−10,3%** của
ponytail cần **80 cặp tác vụ và 251 lần chạy có tính phí** (tổng 246,09 đô)
mới đạt được p = 0,004.

Ý nghĩa thực tế với bạn:

| Số cặp | Bạn phát hiện được gì | Bạn **không** phát hiện được gì |
| --- | --- | --- |
| 3–5 | Gần như không gì — dùng để kiểm tra can thiệp có chạy | Mọi hiệu ứng thật |
| 10–20 | Hiệu ứng rất lớn (trên 30–40%) | Bất cứ thứ gì cỡ 10% |
| 30–50 | Hiệu ứng lớn và vừa | Hiệu ứng nhỏ |
| 80+ | Cỡ 10% với ý nghĩa thống kê | — |

Kết luận không dễ chịu: **nếu bạn chạy 5 tác vụ và thấy chênh 12%, bạn chưa
đo được gì cả.** Đó là nhiễu. Điều đúng đắn cần làm khi đó là tăng n hoặc nói
thẳng rằng kết quả không kết luận được.

Nếu ngân sách không cho phép đạt n cần thiết — hoàn toàn hợp lý — thì hãy
chọn cách rẻ hơn: chỉ tìm những hiệu ứng đủ lớn để hiện ra ở n nhỏ, và ghi
nhận phần còn lại là **chưa biết**.

### 7 — Đặt ngưỡng quyết định *trước khi* nhìn kết quả

Viết ra trước: *"Tôi sẽ giữ thay đổi này nếu nó giảm chi phí trung vị mỗi tác
vụ ít nhất 8% mà không làm hỏng tác vụ nào."*

Đặt ngưỡng sau khi đã thấy dữ liệu là cách bạn tự thuyết phục mình rằng +2%
là một chiến thắng.

## Những thứ sẽ đánh lừa bạn

- **Trạng thái cache.** Lần chạy đầu tiên trong ngày trả giá đầy đủ. Nếu
  nhánh A luôn chạy trước, nhánh A luôn chịu cache lạnh. Hãy đảo thứ tự.
- **TTL cache khác nhau theo cách bạn trả tiền.** Trên gói thuê bao là 1 giờ,
  trên API key là 5 phút — xem [`BILLING.md`](BILLING.md). Một thử nghiệm kéo
  dài cả ngày có thể vô tình vượt qua ranh giới đó.
- **Nhà cung cấp cập nhật giữa chừng.** Model và harness thay đổi. Hãy chạy
  cả hai nhánh trong cùng một cửa sổ thời gian, đừng chạy nhánh A tuần này và
  nhánh B tuần sau.
- **Chỉ số tự công bố.** Con số đô la trên màn hình được tính cục bộ theo giá
  niêm yết. Nó tốt để so sánh tương đối và không phải hóa đơn của bạn — trên
  gói thuê bao thì tài liệu chính thức nói thẳng nó không liên quan tới việc
  tính tiền.
- **Baseline chưa từng tồn tại.** "Công cụ này tiết kiệm 80% so với nếu bạn
  làm cách ngu ngốc nhất" không phải phép đo. `PROOF.md` xếp loại này là hạng
  🔴 D và loại bỏ.
- **Chất lượng âm thầm giảm.** Một thay đổi làm giảm token bằng cách làm ít
  việc hơn *không* phải khoản tiết kiệm. Điều kiện hoàn thành ở bước 2 tồn
  tại để chặn đúng chuyện này.

## Phiên bản rẻ: một buổi chiều, một người

Nếu bạn không định chạy nghiên cứu 80 cặp — hầu hết mọi người không — thì đây
là mức tối thiểu vẫn còn ý nghĩa:

1. Chọn 10 tác vụ thật từ backlog, có điều kiện hoàn thành rõ ràng.
2. Kiểm toán rằng can thiệp thực sự kích hoạt. Nếu không, dừng lại tại đây.
3. Chạy cả hai nhánh ở chế độ headless, đảo thứ tự, một lần mỗi bên.
4. So sánh **trung vị** chênh lệch theo từng cặp, không phải trung bình cộng.
5. Chỉ hành động nếu chênh lệch **lớn** (trên 25–30%) và cùng chiều ở đa số
   các cặp.
6. Ghi lại kết quả là hạng 🟠 C — "có số, phương pháp mỏng" — và ghi rõ n.

Như vậy trung thực, và nó vẫn bắt được những trường hợp đáng quan tâm nhất:
công cụ làm hóa đơn *tăng*.

## Lấy số liệu ở đâu

| Agent | Trong phiên | Xuất ra ngoài |
| --- | --- | --- |
| Claude Code | `/usage` (bí danh `/cost`), `/context` | Xuất OpenTelemetry — token và chi phí theo từng người, gần thời gian thực |
| Codex CLI | `/status` | Log phiên |
| Gemini CLI | `/stats` | — |
| Cline | Chi phí theo từng task, ngay trên UI | — |

Muốn theo dõi cho cả đội thay vì một máy, hãy đi qua một gateway ghi nhận chi
tiêu theo key (LiteLLM, Helicone, Langfuse, Portkey). Chi tiết ở
[`solutions/token-counting.md`](solutions/token-counting.md).

Lưu ý về phân bổ: trên gói thuê bao, phần "attribution" của `/usage` quy phần
trăm mức dùng gần đây về từng skill, subagent, plugin và từng MCP server. Đó
là cách rẻ nhất để trả lời "thứ gì đang ăn hạn mức của tôi" mà không cần dựng
gì cả — và là công cụ chính cho
[`solutions/mcp-server-audit.md`](solutions/mcp-server-audit.md).

## Ghi lại kết quả

Dùng đúng thang bằng chứng của [`PROOF.md`](PROOF.md) để kết quả của bạn so
sánh được với phần còn lại của kho:

| Hạng | Nghĩa là |
| --- | --- |
| 🟢 A | Bình duyệt, hoặc A/B ghép cặp độc lập có công bố phương pháp |
| 🟡 B | Do nhà cung cấp chạy nhưng **lặp lại được**: đã công bố model, cờ, truy vấn, số liệu thô |
| 🟠 C | Có số nhưng phương pháp mỏng hoặc mang tính định tính |
| 🔴 D | Tự công bố so với một baseline chưa từng tồn tại — **loại bỏ** |

Và hãy ghi lại những gì cần thiết để người khác lặp lại: phiên bản agent,
model, mức effort, n, thiết kế ghép cặp, tỷ lệ kích hoạt của can thiệp, và
số liệu thô. Nếu bạn đo được điều gì bác bỏ một mục trong `PROOF.md`, đó là
đóng góp giá trị nhất bạn có thể thêm vào kho này.

## Nguồn

- Phương pháp và các con số của nghiên cứu ponytail/RTK/Caveman: xem phần
  Nguồn trong [`PROOF.md`](PROOF.md)
- Claude Code — `/usage`, phân bổ, cờ hành vi, xuất OpenTelemetry:
  <https://code.claude.com/docs/en/costs>

---

# Measuring: how to produce your own evidence

[`PROOF.md`](PROOF.md) tells you **what has been measured**. This one tells
you **how to measure it yourself**.

You need it for the reason stated at the top of `PROOF.md`: the baseline is
the harness. Every percentage in this repo is a marginal value against *one*
specific setup — one agent version, one model, one repo. Yours is different.
The only way to know whether a change helps **you** is to run it.

The good news: the process that produced the most trustworthy number in this
repo isn't mysterious, and you can run a scaled-down version of it in an
afternoon.

> **Evidence status:** ⚪ This is a description of method, not a result. The
> procedure below mirrors the study design that produced the 🟢 A-tier entries
> in [`PROOF.md`](PROOF.md).

## Rule one: measure the bill, not the compression ratio

This is the mistake that has killed more claims in this repo than any other.

RTK advertised 60–90% reduction. That number is **real** — it's a compression
ratio measured on text. Measured against the bill, it came out at **+7.6%
cost** (p = 0.004): compressed output made the model less certain, so it did
more work, driving +13.8% turns and +14.3% cache reads.

The general lesson:

| Don't measure | Measure |
| --- | --- |
| Text compression ratio | Total tokens for the whole task |
| Tokens saved at one step | Cost from start to done |
| Context size | Turns required to finish |
| Input tokens | Input + output + cache writes + cache reads |

A task is only over when it's **done**. If a tool cuts 40% off each tool
result but makes the agent take extra turns to compensate, you just paid for
that optimization.

## Rule two: confirm the intervention actually fired

Before measuring anything, prove that the thing you're testing **is running
at all**.

This is not a nitpick. Ponytail's ruleset, installed passively, activated
**zero times across ten sessions**. Had the researchers skipped this check,
they would have measured "0% improvement" and concluded the tool was useless
— when in fact it had never run. The published study audited adoption at
**100% treatment / 0% baseline**.

How to audit, in increasing order of confidence:

1. Manually inspect one session: does the intervention leave a trace?
2. Log at the attachment layer — did the hook run, was the tool called?
3. Count activations across **every** run in both arms, and publish it.

If you can't count activations, you aren't measuring the tool. You're
measuring noise.

## The protocol

### 1 — Change exactly one thing

One variable. One config dial, one tool, one workflow convention. If you
change `effortLevel` **and** install CodeGraph at the same time, the result
tells you neither which one worked nor whether one is masking the other's
harm.

### 2 — Build a task set from your real backlog

10–30 tasks, drawn from real work, not toy problems. Each task needs:

- A fixed, pre-written description (pasted verbatim into both arms).
- A verifiable **done-condition** — tests pass, a file exists, a command
  exits 0. Without one you can't compare cost, because the two arms may have
  done different amounts of work.
- An identical starting repo state (a specific commit, or a clean worktree
  per run).

Make the task set reflect your actual distribution of work. If 80% of your
day is narrow bug fixes, don't build a task set entirely out of greenfield
features — ponytail measured **−31% on large builds and zero where the code
was already minimal**. The wrong task set gives you the right number for
somebody else.

### 3 — Freeze everything else

Record these and hold them constant throughout:

| Variable | Why |
| --- | --- |
| Agent version | Harness behavior changes between releases |
| Model, exact ID | "Sonnet" is not a fixed variable |
| Reasoning effort level | The single most expensive dial there is |
| Repo state | One specific commit |
| Instruction files | Freeze `CLAUDE.md`/`AGENTS.md` too |
| Enabled MCP servers | Their schemas enter every request |

### 4 — Run headless

Run non-interactively (`-p` on Claude Code, `codex exec` on Codex). Three
reasons, all of them load-bearing:

- It removes you from the loop. A human steering will unconsciously drive
  their preferred arm to a better outcome.
- It makes runs repeatable. You'll run each task more than once.
- It makes cache state predictable — every run starts cold.

### 5 — Pair the runs; don't average the arms separately

Run **the same task** through both arms and compare within each pair. Don't
average arm A, average arm B, and subtract.

The reason: variance between *tasks* is far larger than the effect you're
looking for. A 300k-token task and a 20k-token task in the same average will
drown a 10% difference. Pairing removes that source of noise.

Alternate the order: A-then-B on task 1, B-then-A on task 2. This cancels the
effect of a warm cache and of provider-side changes over the day.

### 6 — Run enough of them

This is where most home-grown experiments fall apart. Ponytail's **−10.3%**
required **80 paired tasks and 251 billed trials** ($246.09 total) to reach
p = 0.004.

What that means for you:

| Pairs | What you can detect | What you **cannot** detect |
| --- | --- | --- |
| 3–5 | Almost nothing — use it to check the intervention runs | Any real effect |
| 10–20 | Very large effects (above 30–40%) | Anything around 10% |
| 30–50 | Large and moderate effects | Small effects |
| 80+ | ~10% with statistical significance | — |

The uncomfortable conclusion: **if you run 5 tasks and see a 12% difference,
you have measured nothing.** That's noise. The correct move is to raise n or
to state plainly that the result is inconclusive.

If your budget won't reach the necessary n — entirely reasonable — then buy
the cheaper thing: look only for effects large enough to show up at small n,
and record the rest as **unknown**.

### 7 — Set your decision threshold *before* looking at results

Write it down first: *"I keep this change if it cuts median cost per task by
at least 8% with no task regressing to failure."*

Setting the threshold after seeing the data is how you talk yourself into
believing +2% is a win.

## What will fool you

- **Cache state.** The first run of the day pays full price. If arm A always
  runs first, arm A always eats the cold cache. Alternate.
- **Cache TTL differs by how you pay.** An hour on a subscription, five
  minutes on an API key — see [`BILLING.md`](BILLING.md). An experiment
  spread across a day can cross that boundary without you noticing.
- **Providers ship mid-experiment.** Models and harnesses change. Run both
  arms inside the same time window, not arm A this week and arm B next week.
- **Self-reported metrics.** The dollar figure on screen is computed locally
  at list prices. It's fine as a relative measure and it is not your bill —
  on a subscription the vendor documentation says outright that it isn't
  relevant for billing.
- **A baseline that never existed.** "This tool saves 80% versus doing it the
  dumbest possible way" is not a measurement. `PROOF.md` grades that 🔴 D and
  rejects it.
- **Quality quietly dropping.** A change that reduces tokens by doing less
  work is *not* a saving. The done-condition in step 2 exists to catch
  exactly this.

## The cheap version: one afternoon, one person

If you're not going to run an 80-pair study — and most people aren't — this
is the minimum that still means something:

1. Pick 10 real tasks from your backlog with clear done-conditions.
2. Audit that the intervention actually fires. If it doesn't, stop here.
3. Run both arms headless, alternating order, once per side.
4. Compare the **median** per-pair difference, not the mean.
5. Act only if the difference is **large** (above 25–30%) and points the same
   direction in most pairs.
6. Record the result as 🟠 C — "numbers exist, method is thin" — and state n.

That's honest, and it still catches the case that matters most: a tool that
makes the bill go *up*.

## Where the numbers come from

| Agent | In session | Export |
| --- | --- | --- |
| Claude Code | `/usage` (aliased as `/cost`), `/context` | OpenTelemetry export — per-user tokens and cost in near real time |
| Codex CLI | `/status` | Session logs |
| Gemini CLI | `/stats` | — |
| Cline | Per-task cost, right in the UI | — |

For fleet-level tracking rather than one machine, route through a gateway
that records spend per key (LiteLLM, Helicone, Langfuse, Portkey). Details in
[`solutions/token-counting.md`](solutions/token-counting.md).

One note on attribution: on a subscription, the `/usage` attribution view
assigns a share of recent usage to each skill, subagent, plugin, and
individual MCP server. It is the cheapest possible answer to "what is eating
my limits" with nothing to set up — and the primary instrument for
[`solutions/mcp-server-audit.md`](solutions/mcp-server-audit.md).

## Recording your result

Use [`PROOF.md`](PROOF.md)'s own evidence tiers so your result is comparable
with the rest of the repo:

| Tier | What it means |
| --- | --- |
| 🟢 A | Peer-reviewed, or independent paired A/B with published methodology |
| 🟡 B | Vendor-run but **reproducible**: model, flags, queries and raw figures published |
| 🟠 C | Numbers exist but the method is thin or qualitative |
| 🔴 D | Self-reported against a counterfactual that never existed — **rejected** |

And record what someone else would need to reproduce it: agent version,
model, effort level, n, the pairing design, the intervention's activation
rate, and the raw figures. If you measure something that contradicts an entry
in `PROOF.md`, that is the most valuable contribution you can make to this
repo.

## Sources

- Methodology and figures for the ponytail/RTK/Caveman studies: see the
  Sources section of [`PROOF.md`](PROOF.md)
- Claude Code — `/usage`, attribution, behavior flags, OpenTelemetry export:
  <https://code.claude.com/docs/en/costs>
