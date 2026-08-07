# token-optimizer (Tiếng Việt)

Hóa đơn LLM của bạn phần lớn là những token lẽ ra không cần gửi đi. Đây là
cẩm nang chỉ ra chỗ chúng rò rỉ — và cách bịt lại.

Không có gì để cài đặt, không có code để chạy. Đây là một tập hợp ghi chú
viết bằng ngôn ngữ dễ hiểu: điều gì khiến lượng token phình to, và những cách
cụ thể để kéo nó xuống.

## Vì sao có kho tài liệu này

Mỗi lần gọi một LLM đều bị tính tiền theo token, và hầu hết ứng dụng lặng lẽ
lãng phí rất nhiều — gửi lại cùng một context, đổ ra kết quả tool quá khổ,
trả giá đầy đủ cho những thứ mà một bộ cache lẽ ra đã lo được. Lãng phí kiểu
này thường vô hình, cho đến khi hóa đơn về tới.

Kho này làm ba việc:

- Gọi tên các nguyên nhân gốc rễ gây tốn token, gom về một chỗ.
- Ghép mỗi nguyên nhân với cách khắc phục cụ thể, làm được ngay.
- Lớn dần theo hiểu biết của chúng ta — đây là một cuốn sổ tay sống, không
  phải cẩm nang đã đóng khung.

## Bên trong có gì

| Ở đâu | Bạn sẽ thấy gì |
| --- | --- |
| [`CAUSE.md`](CAUSE.md) | Danh mục những thứ đẩy lượng token lên cao — hãy bắt đầu từ đây. |
| [`BILLING.md`](BILLING.md) | **Tiết kiệm token thực ra mua được gì.** Trả theo token, theo hạn mức gói, hay theo số request — đơn vị đó quyết định nguyên nhân nào thực sự đắt với bạn. |
| [`solutions/`](solutions/README.md) | Mỗi cách khắc phục là một tài liệu ngắn, đều dẫn ngược về một nguyên nhân. |
| [`setups/`](setups/) | Các thiết lập dựng sẵn cho từng nhà cung cấp và công cụ cụ thể. |
| [`tools/`](tools/README.md) | Hồ sơ chi tiết từng công cụ — ý tưởng, cách hoạt động, cách cài, dùng lúc nào và khi nào thì đừng. |
| [`HARNESS.md`](HARNESS.md) | Harness là gì, nó tiêu token ở đâu, và bảng đối chiếu năng lực của Claude Code, Codex CLI, Gemini CLI và Cline. |
| [`CONFIG.md`](CONFIG.md) | Đặt nút nào, ở giá trị nào — cấu hình đề xuất cho từng agent, kèm những nút chỉnh quá tay sẽ làm hóa đơn tăng. |
| [`WORKFLOW.md`](WORKFLOW.md) | Tầng cuối cùng: bạn làm gì ở mỗi phiên. Phạm vi, chỉ đường, nén có chủ đích, và lúc nào nên bắt đầu lại. |
| [`PROOF.md`](PROOF.md) | **Cái gì đã thực sự được đo.** Đối chiếu tuyên bố quảng cáo với kết quả A/B có đối chứng — và loại những tuyên bố không trụ được. Đọc trước khi tin bất kỳ con số phần trăm nào trong kho này. |
| [`MEASURE.md`](MEASURE.md) | Cách tự tạo ra bằng chứng đó: thiết kế ghép cặp, cần bao nhiêu lần chạy, và những thứ sẽ đánh lừa bạn. |
| [`SUMMARY.md`](SUMMARY.md) | Bảng tra nhanh nguyên nhân → giải pháp → công cụ. |

## Dùng nó thế nào

1. **Xác định bạn đang tiêu đơn vị gì** — [`BILLING.md`](BILLING.md). Trên
   gói cố định, token tiết kiệm đổi thành dư địa chứ không phải tiền, và thứ
   tự ưu tiên thay đổi theo.
2. **Đọc `CAUSE.md`** để nhận ra vấn đề nào khớp với điều bạn đang gặp.
3. **Theo các liên kết sang `solutions/`** để tìm cách khắc phục tương ứng.
4. **Đối chiếu với [`PROOF.md`](PROOF.md)** trước khi cài thêm công cụ. Nhiều
   công cụ nổi tiếng đo ra gần bằng 0 — có cái còn làm hóa đơn *tăng*. Muốn
   tự kiểm chứng trên thiết lập của mình, xem [`MEASURE.md`](MEASURE.md).
5. **Đặt các nút trong harness trước** — xem [`CONFIG.md`](CONFIG.md). Tầng
   này miễn phí, và trên vài agent nó xử lý luôn nguyên nhân đắt nhất. Rồi
   [`WORKFLOW.md`](WORKFLOW.md) cho phần còn lại: cách bạn làm việc mỗi phiên.
6. **Ghé `setups/`** nếu bạn muốn một cấu hình dựng sẵn cho nhà cung cấp hoặc
   công cụ của mình. Nếu nơi làm việc chặn plugin/extension/npm, xem
   [`setups/install-without-package-managers.md`](setups/install-without-package-managers.md).
7. **Phát hiện điều gì mới?** Thêm vào `CAUSE.md`, và nếu đã biết cách sửa,
   hãy để lại ghi chú trong `solutions/`.

## Đóng góp

Đây là kho chỉ gồm tài liệu — thuần Markdown, không build, không test, không
có gì để chạy.

- Làm việc thẳng trên `main`. Không nhánh, không pull request.
- Giữ mỗi thay đổi gọn và tập trung, và giữ cho nó chính xác.

### Thông điệp commit

Ngắn gọn và nhất quán. Định dạng:

```
<type>: <bạn đã làm gì, ở thể mệnh lệnh — "add", "fix", "clarify">
```

Chọn một type:

| Type | Dùng khi bạn đang... |
| --- | --- |
| `docs` | Viết hoặc sửa tài liệu nói chung (trường hợp thường gặp nhất). |
| `cause` | Đụng tới một mục trong `CAUSE.md`. |
| `solution` | Đụng tới một tài liệu trong `solutions/`. |
| `chore` | Dọn dẹp — cấu trúc, đổi tên, định dạng. |

Vài ví dụ:

```
docs: add repository overview and layout
cause: document redundant system-prompt re-sends
solution: add prompt-caching guide
chore: restructure solutions folder
```

Về cơ bản chỉ vậy. Giữ dòng tiêu đề dưới ~72 ký tự, viết thường sau tiền tố,
không dấu chấm cuối. Nếu một commit cần giải thích "tại sao", thêm một đoạn
thân ngắn bên dưới dòng tiêu đề.

---

# token-optimizer

Your LLM bill is mostly tokens you never needed to send. This is a field
guide to where they leak — and how to plug the leaks.

There's nothing to install and no code to run. It's a collection of
plain-language notes: what makes token usage balloon, and the concrete moves
that bring it back down.

## Why this exists

Every call to an LLM is priced by the token, and most apps quietly waste a
lot of them — re-sending the same context, dumping oversized tool output,
paying full price for things a cache could have covered. The waste is usually
invisible until the bill arrives.

This repo does three things:

- Names the root causes of high token usage, all in one place.
- Pairs each cause with concrete fixes you can actually apply.
- Grows as we learn more — it's a living notebook, not a finished manual.

## What's inside

| Where | What you'll find |
| --- | --- |
| [`CAUSE.md`](CAUSE.md) | The catalog of what drives token usage up — start here. |
| [`BILLING.md`](BILLING.md) | **What saving tokens actually buys you.** Metered, plan-limited, or request-quotaed — the unit you spend decides which causes are genuinely expensive for you. |
| [`solutions/`](solutions/README.md) | One short guide per fix, each mapped back to a cause. |
| [`setups/`](setups/) | Ready-made setups for specific vendors and tools. |
| [`tools/`](tools/README.md) | Deep profiles of individual tools — the idea, how it works, how to install, when to use it and when not to. |
| [`HARNESS.md`](HARNESS.md) | What a harness is, where it spends your tokens, and a capability matrix across Claude Code, Codex CLI, Gemini CLI and Cline. |
| [`CONFIG.md`](CONFIG.md) | Which dials to set and to what — recommended configuration per agent, plus the ones that raise your bill when over-tuned. |
| [`WORKFLOW.md`](WORKFLOW.md) | The last layer: what you do every session. Scoping, pointing, deliberate compaction, and when to start over. |
| [`PROOF.md`](PROOF.md) | **What has actually been measured.** Advertised claims against controlled A/B results — and the claims that didn't survive. Read it before trusting any percentage in this repo. |
| [`MEASURE.md`](MEASURE.md) | How to produce that evidence yourself: the paired design, how many runs it takes, and what will fool you. |
| [`SUMMARY.md`](SUMMARY.md) | Quick cause → solution → tool lookup. |

## How to use it

1. **Work out which unit you're spending** — [`BILLING.md`](BILLING.md). On a
   flat plan, saved tokens become headroom rather than money, and the priority
   order changes accordingly.
2. **Read `CAUSE.md`** to spot which problems match what you're seeing.
3. **Follow the links into `solutions/`** for the matching fixes.
4. **Check them against [`PROOF.md`](PROOF.md)** before installing anything.
   Several well-known tools measured near zero — one made the bill *go up*. To
   verify anything on your own setup, see [`MEASURE.md`](MEASURE.md).
5. **Set your harness's own dials first** — see [`CONFIG.md`](CONFIG.md). That
   tier is free, and on some agents it closes the most expensive cause outright.
   Then [`WORKFLOW.md`](WORKFLOW.md) for what's left: how you work each session.
6. **Check `setups/`** if you want a ready-made configuration for your vendor
   or tool. If your workplace blocks plugins/extensions/npm, see
   [`setups/install-without-package-managers.md`](setups/install-without-package-managers.md).
7. **Found something new?** Add it to `CAUSE.md`, and if you know the fix,
   leave a note in `solutions/`.

## Contributing

This is a docs-only repo — just Markdown, no build, no tests, nothing to run.

- Work straight on `main`. No branches, no pull requests.
- Keep each change small and focused, and keep it accurate.

### Commit messages

Short and consistent. Format:

```
<type>: <what you did, in the imperative — "add", "fix", "clarify">
```

Pick a type:

| Type | Use it when you're... |
| --- | --- |
| `docs` | Writing or editing general documentation (the usual case). |
| `cause` | Touching an entry in `CAUSE.md`. |
| `solution` | Touching a guide in `solutions/`. |
| `chore` | Tidying up — structure, renames, formatting. |

A few examples:

```
docs: add repository overview and layout
cause: document redundant system-prompt re-sends
solution: add prompt-caching guide
chore: restructure solutions folder
```

That's basically it. Keep subject lines under ~72 characters, lowercase after
the prefix, no trailing period. If a commit needs a "why," add a short body
below the subject line.
