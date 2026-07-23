# Giải pháp (Tiếng Việt)

Thư mục này gồm các tài liệu, mỗi tài liệu ứng với một giải pháp hoặc chiến
lược để giảm mức tiêu tốn token. Mỗi giải pháp nên trỏ về một hoặc nhiều
nguyên nhân được liệt kê trong [`../CAUSE.md`](../CAUSE.md).

## Thêm một giải pháp

1. Tạo một file Markdown mới trong thư mục này, đặt tên theo chiến lược, ví
   dụ `prompt-caching.md` hoặc `context-trimming.md`.
2. Tuân theo mẫu (template) mục bên dưới để mọi giải pháp đều nhất quán.
3. Liên kết giải pháp từ nguyên nhân liên quan trong `../CAUSE.md`.

## Mẫu giải pháp

```
# <Tên giải pháp>

**Addresses (Giải quyết):** (Các) nguyên nhân nào từ CAUSE.md mà giải pháp
này khắc phục.

**Idea (Ý tưởng):** Mô tả ngắn gọn về cách tiếp cận.

**How to apply (Cách áp dụng):** Các bước cụ thể, khả thi.

**SOTA tools (Công cụ hiện đại nhất):** Luôn có hai phần con theo thứ tự
này:
  - "Native — coding agents & provider APIs" (Có sẵn — các coding agent &
    API của nhà cung cấp) — các tính năng có sẵn trong các agent/nhà cung
    cấp bạn đang dùng (Claude Code/Anthropic, Codex/OpenAI, Gemini
    CLI/Google).
  - "Third-party — agent-agnostic (open source preferred)" (Bên thứ ba —
    không phụ thuộc agent, ưu tiên mã nguồn mở) — các công cụ hoạt động với
    *bất kỳ* agent nào, có ghi giấy phép; liệt kê các lựa chọn mã nguồn mở
    trước và nêu tên các lựa chọn thương mại kèm theo.

**Trade-offs (Đánh đổi):** Chi phí, hạn chế, hoặc rủi ro cần lưu ý.

**Expected impact (Tác động dự kiến):** Ước lượng sơ bộ về mức tiết kiệm
token.
```

## Bắt đầu từ đâu

Không phải giải pháp nào cũng đáng công thiết lập cho mọi hệ thống.
[`recommended-setup.md`](../setups/recommended-setup.md) chắt lọc danh mục thành
**bộ công cụ cần thiết cho một codebase lớn với nhiều agent** — cái gì nên
kế thừa từ harness, bốn phần việc tùy chỉnh thực sự quan trọng, và cái gì
nên bỏ qua một cách rõ ràng.

Để xem cách áp dụng cụ thể theo từng nhà cung cấp, xem
[`coding-setup-enterprise.md`](../setups/coding-setup-enterprise.md) — trình bày các
thiết lập coding-agent cấp doanh nghiệp trên **Claude, GPT, và Gemini**: đường
truy cập và những đánh đổi tính năng đi kèm, cách chọn harness, cấu hình
caching, bản đồ model/effort, và hướng dẫn cho đội hình dùng nhiều nhà cung
cấp.

Với harness chúng ta dùng hàng ngày, xem
[`coding-setup-cline.md`](../setups/coding-setup-cline.md) — áp dụng cùng cách phân
tầng đó cho **Cline** (tiện ích mở rộng VS Code): cách chọn nhà cung cấp/
caching, tách model Plan/Act, kỷ luật quản lý context `/smol`–`/newtask`, cắt
giảm chi phí `.clinerules`/`.clineignore`/MCP, và đo lường toàn đội qua một
gateway.

## Mục lục

Cả 25 giải pháp đều đã được viết xong và ánh xạ tới các nguyên nhân trong
[`../CAUSE.md`](../CAUSE.md). Mỗi tài liệu đi kèm khuyến nghị công cụ hiện
đại nhất — chia thành **API có sẵn của agent/nhà cung cấp** và **công cụ bên
thứ ba không phụ thuộc agent (ưu tiên mã nguồn mở, có ghi giấy phép)** — cùng
một đánh giá tác động dự kiến; một số tài liệu còn có sơ đồ mermaid minh họa
cơ chế hoạt động.

### Danh mục 1 — Lỗi Caching

| Tài liệu | Giải quyết | Tác động nổi bật |
| --- | --- | --- |
| [`prompt-caching.md`](prompt-caching.md) | 1.1–1.4, 6.1 | Giảm tới ~90% giá input đã cache; giảm 5–10× input hiệu dụng trong các phiên dài |
| [`stable-prompt-architecture.md`](stable-prompt-architecture.md) | 1.3 | Khiến cache không thể bị làm mới giữa phiên vì lý do cấu trúc; duy trì tỷ lệ cache-hit 80–95% |

### Danh mục 2 — Tích lũy context

| Tài liệu | Giải quyết | Tác động nổi bật |
| --- | --- | --- |
| [`compaction.md`](compaction.md) | 2.1 | Chi phí phiên từ bậc hai → tuyến tính; giảm 3–10× trên các phiên chạy dài |
| [`context-editing.md`](context-editing.md) | 2.1, 2.2 | Thu nhỏ context ổn định 2–5× mà không mất mát do tóm tắt |
| [`context-hygiene.md`](context-hygiene.md) | 2.3 | Loại bỏ 20–40% lịch sử thường bị lãng phí vào trùng lặp |

### Danh mục 3 — Cách dùng tool

| Tài liệu | Giải quyết | Tác động nổi bật |
| --- | --- | --- |
| [`tool-output-budgets.md`](tool-output-budgets.md) | 3.1 | Giảm 2–5× input mỗi phiên trong khối lượng công việc nặng về tool |
| [`tool-output-compression.md`](tool-output-compression.md) | 3.1, 2.1 | Giảm 60–90% output CLI/log/JSON nhiễu, không cần thiết kế lại tool |
| [`tool-composition.md`](tool-composition.md) | 3.2 | K round-trip → ~1 lượt xử lý context; dữ liệu trung gian không bao giờ bị tính phí |
| [`event-driven-waiting.md`](event-driven-waiting.md) | 3.3 | Chi phí chờ đợi O(số lần poll × context) → ~0 |
| [`tool-search.md`](tool-search.md) | 3.4 | Giảm 10–50× chi phí schema tool mỗi request; Code Mode giảm ~99.9% trên các API lớn |

### Danh mục 4 — Loại nội dung đắt đỏ

| Tài liệu | Giải quyết | Tác động nổi bật |
| --- | --- | --- |
| [`image-downsampling.md`](image-downsampling.md) | 4.1 | Giảm 3–5× input thị giác; 50–80% trên các vòng lặp screenshot |
| [`document-reuse.md`](document-reuse.md) | 4.2 | doc×số câu hỏi → doc×1 + các lần đọc cache rẻ |
| [`retrieval-tuning.md`](retrieval-tuning.md) | 4.2 | Giảm ~5× tỷ trọng truy xuất nhờ rerank với k nhỏ; chất lượng cũng cải thiện |
| [`code-maps.md`](code-maps.md) | 4.2, 6.5, 6.1, 2.1 | Xóa bỏ 67–76% chi phí tìm file; tiết kiệm 25–60K token cold-start |
| [`token-counting.md`](token-counting.md) | 4.3 (+ tất cả) | Lớp đo lường — biến mọi giải pháp khác thành một bất biến được thực thi |

### Danh mục 5 — Chi tiêu phía sinh (generation)

| Tài liệu | Giải quyết | Tác động nổi bật |
| --- | --- | --- |
| [`reasoning-effort-tuning.md`](reasoning-effort-tuning.md) | 5.1 | Effort đặt theo từng route kiểm soát mức chênh lệch token output 2–10× |
| [`concise-output-prompting.md`](concise-output-prompting.md) | 5.2 | Giảm 30–60% output trên các route chat/agent; 3–10× qua structured output |
| [`diff-based-edits.md`](diff-based-edits.md) | 5.2 | Giảm 10–50× output mỗi lần chỉnh sửa cho coding agent |
| [`output-cap-sizing.md`](output-cap-sizing.md) | 5.3 | Loại bỏ lãng phí truncate-retry 2–3×; continuation cứu vãn phần đã sinh dở |

### Danh mục 6 — Lựa chọn kiến trúc

| Tài liệu | Giải quyết | Tác động nổi bật |
| --- | --- | --- |
| [`subagent-context-handoff.md`](subagent-context-handoff.md) | 6.1 | Loại bỏ 50–90% công sức subagent phải khám phá lại từ đầu; bàn giao qua artifact giúp agent cha luôn gọn nhẹ |
| [`model-routing.md`](model-routing.md) | 6.2 | Chuyển 50–80% khối lượng sang các tier rẻ hơn 5–25×; kết quả tương đương RouteLLM/FrugalGPT |
| [`batch-processing.md`](batch-processing.md) | 6.2 | Giảm cố định 2× trên lưu lượng không nhạy cảm về độ trễ; 5–20× khi kết hợp với caching |
| [`semantic-caching.md`](semantic-caching.md) | 6.6, 6.2 | Bỏ qua 100% chi phí model khi cache hit; 20–60% lưu lượng đọc lặp lại |
| [`fan-out-warming.md`](fan-out-warming.md) | 6.3 | N lượt xử lý lạnh → 1 lần ghi + (N−1) lần đọc cache (giảm ~85–90% input fan-out) |
| [`prompt-de-scaffolding.md`](prompt-de-scaffolding.md) | 6.4 | Giảm 30–70% system prompt, cộng thêm phần tiết kiệm nhờ output phát sinh giảm theo, thường kèm cải thiện chất lượng |

### Thứ tự áp dụng đề xuất

1. **Đo lường trước** — `token-counting.md` (bạn không thể xếp hạng phần
   còn lại nếu không có nó)
2. **Thắng lợi miễn phí** — `prompt-caching.md` + `stable-prompt-architecture.md`, `batch-processing.md`
3. **Đòn bẩy cấu trúc lớn nhất** — `tool-output-budgets.md`, `compaction.md`/`context-editing.md`, `model-routing.md`
4. **Tinh chỉnh theo từng route** — các tài liệu còn lại, ưu tiên theo nơi mà
   số liệu đo lường của bạn cho thấy chi phí đang đổ vào

---

# Solutions

This folder holds one document per solution or mitigation strategy for reducing
token consumption. Each solution should map back to one or more causes listed in
[`../CAUSE.md`](../CAUSE.md).

## Adding a Solution

1. Create a new Markdown file in this folder named after the strategy, e.g.
   `prompt-caching.md` or `context-trimming.md`.
2. Follow the entry template below so every solution is consistent.
3. Link the solution from the relevant cause in `../CAUSE.md`.

## Solution Template

```
# <Solution name>

**Addresses:** Which cause(s) from CAUSE.md this solves.

**Idea:** Short description of the approach.

**How to apply:** Concrete, actionable steps.

**SOTA tools:** Two subsections, always in this order:
  - "Native — coding agents & provider APIs" — features built into the
    agents/vendors you already use (Claude Code/Anthropic, Codex/OpenAI,
    Gemini CLI/Google).
  - "Third-party — agent-agnostic (open source preferred)" — tools that
    work with *any* agent, with license noted; list open-source options
    first and name commercial alternatives inline.

**Trade-offs:** Costs, limitations, or risks to be aware of.

**Expected impact:** Rough sense of the token savings.
```

## Start Here

Not every solution is worth its setup cost for every system.
[`recommended-setup.md`](../setups/recommended-setup.md) distills the catalog into a
**necessary stack for a large codebase with many agents** — what to inherit
from the harness, the four pieces of custom work that matter, and what to
explicitly skip.

For concrete vendor instantiations, see
[`coding-setup-enterprise.md`](../setups/coding-setup-enterprise.md) — enterprise
coding-agent setups on **Claude, GPT, and Gemini**: access routes and their
feature trade-offs, harness choice, caching configuration, model/effort
maps, and mixed-fleet guidance.

For the harness we use day-to-day, see
[`coding-setup-cline.md`](../setups/coding-setup-cline.md) — the same tiering applied
to **Cline** (VS Code extension): provider/caching choice, Plan/Act model
split, `/smol`–`/newtask` context discipline, `.clinerules`/`.clineignore`/
MCP overhead trimming, and fleet telemetry via a gateway.

## Index

All 25 solutions are written, mapped to the causes in
[`../CAUSE.md`](../CAUSE.md). Each includes SOTA tool recommendations —
split into **agent/provider-native APIs** vs **third-party agent-agnostic
tools (open source preferred, licenses noted)** — and an expected-impact
assessment; several include mermaid diagrams of the mechanism.

### Category 1 — Caching failures

| Document | Addresses | Headline impact |
| --- | --- | --- |
| [`prompt-caching.md`](prompt-caching.md) | 1.1–1.4, 6.1 | Up to ~90% off cached input; 5–10× effective input reduction in long sessions |
| [`stable-prompt-architecture.md`](stable-prompt-architecture.md) | 1.3 | Makes mid-session cache rebuilds structurally impossible; sustains 80–95% cache-hit share |

### Category 2 — Context accumulation

| Document | Addresses | Headline impact |
| --- | --- | --- |
| [`compaction.md`](compaction.md) | 2.1 | Quadratic → linear session cost; 3–10× on long runs |
| [`context-editing.md`](context-editing.md) | 2.1, 2.2 | 2–5× steady-state context shrink with no summarization loss |
| [`context-hygiene.md`](context-hygiene.md) | 2.3 | Removes the 20–40% of history commonly wasted on duplicates |

### Category 3 — Tool usage patterns

| Document | Addresses | Headline impact |
| --- | --- | --- |
| [`tool-output-budgets.md`](tool-output-budgets.md) | 3.1 | 2–5× per-session input cut in tool-heavy workloads |
| [`tool-output-compression.md`](tool-output-compression.md) | 3.1, 2.1 | 60–90% off noisy CLI/log/JSON output, no tool redesign |
| [`tool-composition.md`](tool-composition.md) | 3.2 | K round-trips → ~1 context pass; intermediates never billed |
| [`event-driven-waiting.md`](event-driven-waiting.md) | 3.3 | Waiting cost O(polls × context) → ~0 |
| [`tool-search.md`](tool-search.md) | 3.4 | 10–50× reduction in per-request tool-schema overhead; Code Mode ~99.9% on huge APIs |

### Category 4 — Expensive content types

| Document | Addresses | Headline impact |
| --- | --- | --- |
| [`image-downsampling.md`](image-downsampling.md) | 4.1 | 3–5× vision input cut; 50–80% on screenshot loops |
| [`document-reuse.md`](document-reuse.md) | 4.2 | doc×questions → doc×1 + cheap cached reads |
| [`retrieval-tuning.md`](retrieval-tuning.md) | 4.2 | ~5× retrieval-share cut via reranked small-k; quality improves too |
| [`code-maps.md`](code-maps.md) | 4.2, 6.5, 6.1, 2.1 | Kills the 67–76% file-finding tax; 25–60K cold-start tokens saved |
| [`token-counting.md`](token-counting.md) | 4.3 (+ all) | The measurement layer — turns every other fix into an enforced invariant |

### Category 5 — Generation-side spend

| Document | Addresses | Headline impact |
| --- | --- | --- |
| [`reasoning-effort-tuning.md`](reasoning-effort-tuning.md) | 5.1 | 2–10× output-token spread captured by per-route effort |
| [`concise-output-prompting.md`](concise-output-prompting.md) | 5.2 | 30–60% output cut on chat/agent routes; 3–10× via structured output |
| [`diff-based-edits.md`](diff-based-edits.md) | 5.2 | 10–50× per-edit output cut for coding agents |
| [`output-cap-sizing.md`](output-cap-sizing.md) | 5.3 | Eliminates 2–3× truncate-retry waste; continuation salvages partials |

### Category 6 — Architectural choices

| Document | Addresses | Headline impact |
| --- | --- | --- |
| [`subagent-context-handoff.md`](subagent-context-handoff.md) | 6.1 | 50–90% of subagent re-discovery eliminated; artifact handoff keeps parents lean |
| [`model-routing.md`](model-routing.md) | 6.2 | 50–80% of volume moved to 5–25× cheaper tiers; RouteLLM/FrugalGPT-class results |
| [`batch-processing.md`](batch-processing.md) | 6.2 | Flat 2× on latency-insensitive traffic; 5–20× stacked with caching |
| [`semantic-caching.md`](semantic-caching.md) | 6.6, 6.2 | 100% model-cost skip on cache hits; 20–60% of repetitive read-only traffic |
| [`fan-out-warming.md`](fan-out-warming.md) | 6.3 | N cold passes → 1 write + (N−1) cache reads (~85–90% off fan-out input) |
| [`prompt-de-scaffolding.md`](prompt-de-scaffolding.md) | 6.4 | 30–70% system-prompt cut + induced-output savings, often with quality gains |

### Suggested adoption order

1. **Measure first** — `token-counting.md` (you can't rank the rest without it)
2. **Free wins** — `prompt-caching.md` + `stable-prompt-architecture.md`, `batch-processing.md`
3. **Biggest structural levers** — `tool-output-budgets.md`, `compaction.md`/`context-editing.md`, `model-routing.md`
4. **Route-specific tuning** — the remaining docs, prioritized by what your telemetry attributes the spend to
