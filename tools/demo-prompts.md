# Prompt demo: nhìn thấy từng công cụ thực sự làm gì (Tiếng Việt)

Bốn hồ sơ công cụ trong thư mục này nói **cái gì đã được đo**. Trang này cho
bạn những prompt cụ thể để **tự nhìn thấy cơ chế hoạt động** trong mười phút.

Đây không phải phép đo. Một cặp chạy là n = 1, và
[`../MEASURE.md`](../MEASURE.md) giải thích vì sao con số từ n = 1 không đáng
tin. Mục đích ở đây khác và vẫn có giá trị: **thấy công cụ thay đổi hành vi
gì**, và quan trọng hơn — xác nhận nó có kích hoạt hay không.

> Bước xác nhận đó không phải hình thức. Bộ luật của ponytail khi cài thụ động
> đã kích hoạt **0 lần trong 10 phiên**. Nếu bạn không chứng minh được công cụ
> đang chạy, mọi thứ bạn quan sát sau đó đều vô nghĩa.

## Chuẩn bị chung

Áp dụng cho cả bốn demo:

1. **Phiên mới, hoàn toàn sạch.** `/clear` hoặc mở terminal mới. Context sót
   lại từ phiên trước sẽ làm hỏng cả hai nhánh theo những cách khác nhau.
2. **Chạy nhánh đối chứng trước, trên repo sạch.** `git stash` hoặc
   `git worktree add` để hai nhánh cùng xuất phát từ một điểm.
3. **Ghi số trước và sau.** `/context` trước khi gõ prompt, `/usage` sau khi
   xong. Trên Codex là `/status`, Gemini là `/stats`, Cline hiện ngay trên UI.
4. **Đổi đúng một biến.** Chỉ bật/tắt công cụ. Giữ nguyên model, mức effort,
   file chỉ dẫn.
5. **Dán prompt nguyên văn ở cả hai nhánh.** Diễn đạt lại là đã đổi hai biến.

---

## 1 — Ponytail: xem agent quyết định xây ít hơn

**Cơ chế cần thấy:** ponytail không nén gì cả. Nó thay đổi **agent quyết định
xây cái gì**, trước khi có dòng code nào tồn tại.

**Prompt demo — chọn một tác vụ mời gọi xây thừa:**

```
Thêm chức năng cho phép người dùng xuất danh sách đơn hàng ra file CSV.
```

```
Thêm cơ chế retry cho client gọi API thanh toán.
```

```
Viết một hàm đọc file cấu hình YAML và trả về các giá trị cần thiết.
```

**Cách chạy:**

```
/ponytail off      → chạy prompt → git diff --stat → ghi lại → git checkout .
/ponytail full     → chạy đúng prompt đó → git diff --stat → so sánh
```

**Nhìn vào đâu:**

| Tín hiệu | Nhánh `off` thường thấy | Nhánh `full` thường thấy |
| --- | --- | --- |
| Số dòng trong `git diff --stat` | Nhiều hơn | Ít hơn |
| Số file mới tạo | 3–6 (interface, config, factory, test cho từng lớp) | 1–2 |
| Lớp trừu tượng không ai yêu cầu | `ExporterFactory`, `RetryPolicyConfig` | Không có |
| Xử lý trường hợp chưa ai đề cập | Có | Không |

Chạy `/ponytail-review` trên diff của nhánh `off` để công cụ tự chỉ ra những
chỗ nó coi là xây thừa. Đây là cách nhanh nhất để thấy nó đang "nghĩ" gì.

**Demo này chứng minh gì và không chứng minh gì:**

Nó cho thấy cơ chế có thật và có kích hoạt. Nó **không** cho bạn con số. Con
số đo được là −10,3% chi phí trên 80 cặp tác vụ, và khoản tiết kiệm tập trung
gần như toàn bộ vào các tình huống xây thừa: **−31% ở các lần xây lớn, bằng
không ở chỗ code vốn đã tối giản**.

Nghĩa là nếu bạn demo bằng một tác vụ hẹp ("sửa lỗi off-by-one ở dòng 42"),
bạn sẽ thấy **không có gì thay đổi cả** — và đó là kết quả đúng, không phải
demo hỏng.

---

## 2 — CodeGraph: xem phần khám phá biến mất

**Cơ chế cần thấy:** thay chuỗi grep-rồi-đọc-nhiều-file bằng vài truy vấn vào
một graph đã dựng sẵn.

**Điều kiện bắt buộc:** một repo **thật, lớn** — trên khoảng 1.000 file. Demo
trên repo đồ chơi sẽ cho kết quả ngược, vì chi phí dựng index lớn hơn phần
tiết kiệm. Đó cũng là kết quả đúng.

**Prompt demo — câu hỏi bắt buộc phải truy vết xuyên repo:**

```
Khi người dùng gửi form đăng nhập, hãy liệt kê theo thứ tự mọi hàm được gọi,
từ handler HTTP cho tới lúc ghi session, kèm đường dẫn file và số dòng.
```

```
Hàm `validateToken` được gọi từ những chỗ nào, và mỗi chỗ truyền vào gì?
```

```
Nếu tôi đổi chữ ký của `UserRepository.findById`, những file nào phải sửa?
```

**Cách chạy:** hỏi một lần khi chưa bật CodeGraph, một lần khi đã bật.

**Nhìn vào đâu:**

| Tín hiệu | Không có CodeGraph | Có CodeGraph |
| --- | --- | --- |
| Số lần gọi tool đọc/tìm file | Hàng chục | Vài lần |
| Token input tổng cộng | Cao — mỗi file đọc vào đều tính tiền | Thấp hơn nhiều |
| Nội dung transcript | Đầy file đọc toàn bộ chỉ để lấy vài dòng | Kết quả truy vấn gọn |
| Câu trả lời có bỏ sót nhánh không? | Thường có | Ít hơn |

Cột cuối đáng chú ý: đây là công cụ hiếm hoi mà việc tiết kiệm token đi kèm
**câu trả lời đầy đủ hơn**, vì graph thấy được những chỗ gọi mà grep bỏ lỡ.

**Cảnh báo khi demo:** lần chạy đầu tiên sau khi cài phải dựng index. Đừng
tính thời gian và chi phí của lần đó vào so sánh — hãy dựng index xong rồi mới
bắt đầu demo.

---

## 3 — Caveman: xem output co lại, rồi xem chỗ nó gây hại

Công cụ này có **hai** thứ khác nhau. Demo dưới đây là cho **skill** (ép agent
viết cộc lốc), không phải thư viện nén văn bản.

**Cơ chế cần thấy:** chỉ tác động lên **văn xuôi mà model sinh ra**. Nó không
đụng gì tới kết quả tool, file đọc vào, hay token reasoning.

**Prompt demo — chọn tác vụ nặng về văn xuôi:**

```
Giải thích module xác thực này hoạt động thế nào và luồng đăng nhập đi qua
những bước nào.
```

```
Tóm tắt những thay đổi trong 20 commit gần nhất và ý nghĩa của chúng.
```

**Nhìn vào đâu:** số **output token** cho cùng một câu hỏi. Đó là con số duy
nhất công cụ này có thể tác động.

**Và giờ là phần quan trọng hơn — demo cho thấy nó gây hại:**

```
Hãy sửa lỗi test đang fail trong `tests/test_orders.py`. Chạy test, đọc lỗi,
sửa code, chạy lại cho tới khi xanh.
```

Chạy tác vụ này với skill bật và tắt, **ba lần mỗi bên**, và ghi lại chi phí
từng lần.

| Nhìn vào | Vì sao |
| --- | --- |
| Không phải trung bình — mà là **lần đắt nhất** | Rủi ro đuôi là chế độ hỏng đã được ghi nhận, không phải giả thuyết |
| Agent có phải hỏi lại vì diễn đạt cụt không | Cộc lốc quá thì mất thông tin, và phải bù bằng round-trip |
| Số lượt để hoàn thành | Đây mới là thứ quyết định hóa đơn |

**Kỳ vọng đúng:** quảng cáo −65% output; đo được **8,5%**, kèm rủi ro đuôi
nặng. Trên tác vụ agentic dài, phần văn xuôi chỉ chiếm một tỷ trọng nhỏ trong
tổng chi phí, nên 8,5% của một phần nhỏ gần như không thấy được trên hóa đơn.

---

## 4 — RTK: hai demo, và chúng cho hai kết luận ngược nhau

Đây là demo có giá trị nhất trong trang này, vì nó cho bạn thấy tận mắt cách
một con số thật lại dẫn tới kết luận sai.

### Demo A — "demo bán hàng"

```bash
# chạy trực tiếp, không qua agent
git log --stat -n 100 | rtk
npm test 2>&1 | rtk
cargo build 2>&1 | rtk
```

Đếm ký tự vào và ký tự ra. Bạn sẽ thấy mức giảm **60–90%**. Con số này **có
thật**. Nó là một tỷ lệ nén tính trên văn bản, và nó đúng.

### Demo B — "demo kiểm chứng"

Bây giờ đo thứ thực sự bị tính tiền: chi phí để **hoàn thành một tác vụ**.

```
Bộ test trong repo này đang fail. Chạy nó, tìm nguyên nhân, sửa, và chạy lại
cho tới khi toàn bộ test xanh.
```

Chạy trọn vẹn tác vụ này với RTK bật và tắt. Đừng đo kích thước output tool —
đo tổng chi phí phiên từ đầu tới lúc test xanh.

**Nhìn vào đâu:**

| Tín hiệu | Vì sao nó quan trọng |
| --- | --- |
| Tổng chi phí tới lúc hoàn thành | Đây là hóa đơn |
| **Số lượt** để xong việc | Đo được +13,8% — model kém chắc chắn hơn nên làm nhiều hơn |
| Cache read | Đo được +14,3% — thêm lượt nghĩa là thêm lần gửi lại lịch sử |
| Agent có đọc file bằng tool riêng không? | Nếu có, RTK không bao giờ nhìn thấy lưu lượng đó |

**Kết quả đã đo trên Claude Code: +7,6% chi phí** (p = 0,004). Nén 60–90% văn
bản, mà hóa đơn vẫn tăng.

Demo A và Demo B đều đúng. Chúng chỉ đo hai đại lượng khác nhau — và chỉ một
trong hai là thứ bạn phải trả tiền. Nếu bạn chỉ nhớ một điều từ cả trang này,
hãy nhớ điều đó.

---

## Bảng tổng hợp: mỗi demo chứng minh được gì

| Công cụ | Demo cần loại tác vụ nào | Tín hiệu chính | Cạm bẫy khi demo |
| --- | --- | --- | --- |
| Ponytail | Xây tính năng mới, mời gọi xây thừa | LOC, số file mới, lớp trừu tượng thừa | Demo trên tác vụ hẹp sẽ không thấy gì — đúng như dự kiến |
| CodeGraph | Truy vết xuyên repo, repo trên ~1.000 file | Số lần gọi tool đọc file, token input | Đừng tính lần dựng index vào so sánh |
| Caveman (skill) | Nặng văn xuôi để thấy lợi, agentic dài để thấy hại | Output token; rồi **lần đắt nhất** | Trung bình sẽ nói dối bạn ở đây |
| RTK | Chạy thẳng để thấy tỷ lệ; tác vụ trọn vẹn để thấy hóa đơn | Tổng chi phí và **số lượt**, không phải tỷ lệ nén | Chỉ đo tỷ lệ nén là cách tự lừa mình |

## Sau khi demo xong

Demo cho bạn biết công cụ **làm gì**. Nó không cho bạn biết công cụ có đáng
dùng trên repo của bạn hay không — muốn biết điều đó cần 10–30 cặp tác vụ và
một ngưỡng quyết định đặt trước. Quy trình đầy đủ nằm ở
[`../MEASURE.md`](../MEASURE.md).

Và trước khi bỏ công đo: kiểm tra bạn có đang thực sự trả tiền theo token
không. Trên gói cố định chưa chạm trần, cả bốn công cụ này đều tiết kiệm cho
bạn **không đồng nào** — xem [`../BILLING.md`](../BILLING.md).

---

# Demo prompts: seeing each tool actually work

The four tool profiles in this folder tell you **what has been measured**.
This page gives you concrete prompts to **see the mechanism yourself** in ten
minutes.

This is not measurement. One paired run is n = 1, and
[`../MEASURE.md`](../MEASURE.md) explains why a number from n = 1 can't be
trusted. The purpose here is different and still worthwhile: **see what
behavior the tool changes**, and more importantly — confirm that it fires at
all.

> That confirmation step isn't a formality. Ponytail's ruleset, installed
> passively, activated **zero times across ten sessions**. If you can't prove
> the tool is running, everything you observe afterwards is meaningless.

## Shared setup

Applies to all four demos:

1. **A brand-new, clean session.** `/clear` or a fresh terminal. Leftover
   context from a previous session corrupts both arms in different ways.
2. **Run the control arm first, on a clean repo.** `git stash` or
   `git worktree add` so both arms start from the same point.
3. **Record numbers before and after.** `/context` before you type the
   prompt, `/usage` when it's done. On Codex that's `/status`, Gemini
   `/stats`, Cline shows it in the UI.
4. **Change exactly one variable.** Only the tool on/off. Keep the model,
   effort level, and instruction files identical.
5. **Paste the prompt verbatim in both arms.** Rephrasing means you changed
   two variables.

---

## 1 — Ponytail: watch the agent decide to build less

**The mechanism to see:** ponytail compresses nothing. It changes **what the
agent decides to build**, before a line of code exists.

**Demo prompt — pick a task that invites over-building:**

```
Add a feature that lets users export their order list to a CSV file.
```

```
Add retry logic to the payment API client.
```

```
Write a function that reads a YAML config file and returns the values we need.
```

**How to run it:**

```
/ponytail off      → run the prompt → git diff --stat → record → git checkout .
/ponytail full     → run the exact same prompt → git diff --stat → compare
```

**What to look at:**

| Signal | Typical `off` arm | Typical `full` arm |
| --- | --- | --- |
| Lines in `git diff --stat` | More | Fewer |
| New files created | 3–6 (interface, config, factory, a test per layer) | 1–2 |
| Abstractions nobody asked for | `ExporterFactory`, `RetryPolicyConfig` | None |
| Handling for cases nobody mentioned | Present | Absent |

Run `/ponytail-review` against the `off` arm's diff and the tool will point at
what it considers over-building. That's the fastest way to see what it's
"thinking."

**What this demo proves and doesn't:**

It shows the mechanism is real and that it fires. It does **not** give you a
number. The measured figure is −10.3% cost across 80 paired tasks, and the
savings concentrate almost entirely in over-build scenarios: **−31% on large
builds, zero where the code was already minimal**.

Which means that if you demo with a narrow task ("fix the off-by-one on line
42"), you'll see **nothing change at all** — and that's the correct result,
not a broken demo.

---

## 2 — CodeGraph: watch the exploration disappear

**The mechanism to see:** replacing a grep-then-read-many-files sequence with
a few queries against a pre-built graph.

**Hard requirement:** a **real, large** repo — above roughly 1,000 files. A
demo on a toy repo will show the opposite result, because the indexing cost
exceeds the saving. That is also the correct result.

**Demo prompt — a question that forces a cross-repo trace:**

```
When a user submits the login form, list in order every function that gets
called, from the HTTP handler through to the session being written, with file
paths and line numbers.
```

```
Where is `validateToken` called from, and what does each call site pass in?
```

```
If I change the signature of `UserRepository.findById`, which files have to
change?
```

**How to run it:** ask once with CodeGraph off, once with it on.

**What to look at:**

| Signal | Without CodeGraph | With CodeGraph |
| --- | --- | --- |
| File-read/search tool calls | Dozens | A handful |
| Total input tokens | High — every file read is billed | Much lower |
| Transcript contents | Full files read to extract a few lines | Compact query results |
| Does the answer miss a branch? | Often | Less often |

That last row is worth noting: this is the rare tool where saving tokens comes
with a **more complete answer**, because the graph sees call sites that grep
misses.

**Demo caveat:** the first run after installing has to build the index. Don't
count that run's time or cost in the comparison — build the index first, then
start the demo.

---

## 3 — Caveman: watch output shrink, then watch it hurt

This tool is **two** different things. The demo below is for the **skill**
(forcing terse agent prose), not the text-compression library.

**The mechanism to see:** it only affects **prose the model generates**. It
touches nothing in tool results, file reads, or reasoning tokens.

**Demo prompt — pick a prose-heavy task:**

```
Explain how this authentication module works and what steps the login flow
goes through.
```

```
Summarize the changes in the last 20 commits and what they mean.
```

**What to look at:** the **output token** count for the same question. That's
the only number this tool can move.

**And now the more important part — the demo that shows it hurting:**

```
Fix the failing tests in `tests/test_orders.py`. Run the tests, read the
errors, fix the code, and re-run until green.
```

Run this task with the skill on and off, **three times each side**, and record
the cost of each run.

| Look at | Why |
| --- | --- |
| Not the mean — the **most expensive run** | Tail risk is a documented failure mode, not a hypothetical |
| Whether the agent had to ask again because of clipped phrasing | Too terse loses information, and that's paid back in round-trips |
| Turns to completion | This is what actually decides the bill |

**Correct expectation:** advertised −65% output; measured **8.5%**, with
severe tail risk. On long agentic tasks, prose is a small share of total cost,
so 8.5% of a small share is essentially invisible on the bill.

---

## 4 — RTK: two demos that reach opposite conclusions

This is the most valuable demo on this page, because it shows you first-hand
how a true number leads to a false conclusion.

### Demo A — the one that sells it

```bash
# run directly, no agent involved
git log --stat -n 100 | rtk
npm test 2>&1 | rtk
cargo build 2>&1 | rtk
```

Count characters in and characters out. You'll see a **60–90%** reduction.
That number is **real**. It's a compression ratio measured on text, and it's
correct.

### Demo B — the one that tests it

Now measure the thing that's actually billed: the cost of **completing a
task**.

```
The test suite in this repo is failing. Run it, find the cause, fix it, and
re-run until every test passes.
```

Run this task end to end with RTK on and off. Don't measure tool-output size
— measure total session cost from start until the tests are green.

**What to look at:**

| Signal | Why it matters |
| --- | --- |
| Total cost to completion | This is the bill |
| **Turns** to finish | Measured at +13.8% — a less certain model does more work |
| Cache reads | Measured at +14.3% — more turns means more history re-sent |
| Does the agent read files with its own tools? | If so, RTK never sees that traffic |

**Measured result on Claude Code: +7.6% cost** (p = 0.004). It compresses
60–90% of the text and the bill still goes up.

Demo A and Demo B are both correct. They just measure two different
quantities — and only one of them is the one you pay for. If you remember one
thing from this page, remember that.

---

## Summary: what each demo can prove

| Tool | Task type the demo needs | Primary signal | Demo pitfall |
| --- | --- | --- | --- |
| Ponytail | Greenfield feature work that invites over-building | LOC, new files, gratuitous abstractions | A narrow task shows nothing — as expected |
| CodeGraph | Cross-repo tracing, repo above ~1,000 files | File-read tool calls, input tokens | Don't count the indexing run in the comparison |
| Caveman (skill) | Prose-heavy to see the gain, long agentic to see the harm | Output tokens; then the **worst run** | The mean will lie to you here |
| RTK | Piped directly for the ratio; a full task for the bill | Total cost and **turns**, not the compression ratio | Measuring only the ratio is how you fool yourself |

## After the demo

A demo tells you what a tool **does**. It doesn't tell you whether the tool is
worth running on your repo — that takes 10–30 paired tasks and a
decision threshold set in advance. The full procedure is in
[`../MEASURE.md`](../MEASURE.md).

And before spending effort on measurement: check whether you're actually
paying per token. On a flat plan you never max out, all four of these tools
save you **nothing at all** — see [`../BILLING.md`](../BILLING.md).
