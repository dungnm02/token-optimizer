# Tinh chỉnh Truy xuất (Ưu tiên Độ chính xác hơn Đổ Ngập Recall) (Tiếng Việt)

**Giải quyết:** Nguyên nhân 4.2 trong [`../CAUSE.md`](../CAUSE.md) (cùng với
`document-reuse.md`)

**Ý tưởng:** Làm cho giai đoạn truy xuất làm đúng việc của nó — cung cấp một
context *nhỏ, chính xác* — thay vì đẩy gánh nặng lọc sang cho model bằng
cách đổ một top-k lớn. Mỗi chunk không liên quan đều bị trả tiền trên
request này và trên mọi lượt sau đó mà nó còn tồn tại.

---

## Cách áp dụng

### 1. Thêm một giai đoạn rerank

Truy xuất giai đoạn đầu (BM25/embedding) thiên về recall; hãy rerank các
ứng viên và chỉ gửi những ứng viên vượt qua vòng lọc đó:

```
truy xuất top-50 (rẻ) → rerank → gửi top-3..5 (chính xác)
```

Reranker cross-encoder là bản nâng cấp truy xuất có đòn bẩy cao nhất: chúng
cho phép bạn cắt giảm k 3–5× với chất lượng câu trả lời bằng hoặc tốt hơn.

### 2. Đặt đúng kích thước k và chunk — đo lường, không đoán

- Quét k trên bộ đánh giá của bạn. Với các pipeline được rerank tốt, chất
  lượng câu trả lời thường bão hòa ở k=3–8, trong khi chi phí lại tăng
  tuyến tính theo k.
- Kích thước chunk: các chunk nhỏ hơn (200–500 token) với **tiêu đề context** (breadcrumb tiêu đề/mục, hoặc context sinh ra kiểu công thức
  contextual-retrieval của Anthropic) đánh bại các chunk lớn 2K token —
  bạn gửi đoạn văn liên quan, không phải cả khu vực xung quanh nó.
- Loại bỏ trùng lặp các chunk gần giống hệt nhau (phổ biến với các cửa sổ
  chồng lấn) trước khi gửi.

### 3. Truy xuất khi cần, không phải luôn luôn

- **Kiểm soát truy xuất (gate)**: phân loại xem lượt này có thực sự cần
  truy xuất hay không (nhiều lượt là câu hỏi tiếp nối có thể trả lời từ
  context sẵn có). Một bộ phân loại rẻ hoặc ngưỡng tương đồng embedding
  ("truy vấn có trôi dạt khỏi những gì đã có trong context?") giúp tránh
  việc tự động gắn kèm truy xuất theo phản xạ ở mỗi lượt — xem
  `context-hygiene.md`.
- **Truy xuất agentic**: cho model một tool `search(query)` và để nó tự
  lấy những gì nó quyết định cần (1–2 lệnh gọi có mục tiêu), thay vì nhồi
  sẵn vào mọi request. Điều này biến chi phí truy xuất cố định mỗi lượt
  thành chi phí theo yêu cầu.

### 4. Nén những gì bạn gửi

- Gửi các đoạn trích ngắn (extractive snippet) — đoạn khớp ± một câu —
  thay vì toàn bộ chunk, khi tác vụ cho phép.
- Loại bỏ markup/boilerplate khỏi chunk tại thời điểm index, không phải
  tại thời điểm truy vấn.
- Các reranker nén context (kiểu LLMLingua) có thể ép context truy xuất
  2–5× cho các tác vụ hỏi-đáp, với một chút rủi ro về độ trung thực — hãy
  đánh giá trước khi áp dụng.

## Công cụ hiện đại nhất (SOTA)

### Có sẵn — coding agent & API của nhà cung cấp

| Nhà cung cấp / agent | Tính năng | Ghi chú |
| --- | --- | --- |
| Anthropic | Công thức contextual retrieval | Sinh context cho chunk tại thời điểm index; công bố giảm ~49% thất bại truy xuất (67% khi kết hợp rerank) |
| OpenAI API | `text-embedding-3` + vector store / file search | Truy xuất giai đoạn đầu được quản lý trong stack OpenAI |
| Tìm kiếm repo của coding agent (`Grep`/`Glob` của Claude Code, tool tìm kiếm của Codex/Gemini CLI) | Truy xuất agentic | Với codebase, tìm kiếm theo yêu cầu có giới hạn thường thắng hoàn toàn một pipeline RAG nhồi sẵn |

### Bên thứ ba — không phụ thuộc agent (ưu tiên mã nguồn mở)

| Công cụ | Giấy phép | Ghi chú |
| --- | --- | --- |
| BGE-reranker-v2 / mxbai-rerank | Apache-2.0 | Rerank cross-encoder tự host cấp SOTA; Cohere Rerank 3.5 / Voyage rerank-2 / Jina Reranker v2 là các lựa chọn thương mại host sẵn tương đương |
| Embedding BGE-M3 | MIT | Recall giai đoạn đầu mã nguồn mở mạnh; Voyage-3 là lựa chọn thương mại host sẵn |
| LLMLingua / LLMLingua-2 / LongLLMLingua | MIT | Họ nén prompt của Microsoft — nén tới ~20× với mức giảm chất lượng ~1,5 điểm trên các benchmark hỏi-đáp/suy luận (một trường hợp SaaS được báo cáo: $42K→$2,1K/tháng); LongLLMLingua cắt ~94% chi phí RAG trên LooGLE. **Lưu ý mạnh về độ trung thực trên code/văn bản có cấu trúc** — hãy đánh giá trước khi áp dụng, và ưu tiên loại bỏ ở Tier-0/1 cho các đội hình coding (`../setups/recommended-setup.md` Tier 3) |
| RAGAS / promptfoo | Apache-2.0 / MIT | Đo lường các phép quét k/chunk/rerank thay vì đoán; Braintrust là lựa chọn thương mại thay thế |

## Đánh đổi

- Rerank thêm một bước (~50–300ms) và một khoản phí mỗi truy vấn — hầu
  như luôn được hoàn trả bởi k nhỏ hơn.
- Tinh chỉnh độ chính xác quá mạnh tay có nguy cơ thất bại recall trên các
  câu hỏi đa bước; giữ một lối thoát (truy xuất tiếp nối agentic).
- Các kỹ thuật nén nhạy cảm với tác vụ — kiểm chứng trên đánh giá của bạn,
  không chỉ dựa trên benchmark.

## Tác động dự kiến

- Chuyển từ k=20 không rerank sang k=4 đã rerank giúp cắt giảm **~5×** tỷ
  trọng truy xuất trong input mỗi request, và mức tiết kiệm này còn cộng
  dồn qua mọi lượt mà các chunk đó lẽ ra sẽ còn tồn tại.
- Số liệu contextual-retrieval của Anthropic (−49% thất bại truy xuất,
  −67% khi kết hợp rerank) cho thấy tinh chỉnh độ chính xác *cải thiện
  chất lượng trong khi cắt giảm token* — các chunk không liên quan đang
  làm hại cả hai.
- Kiểm soát truy xuất (gate) thường loại bỏ được 30–70% số lần truy xuất
  bị gắn kèm không cần thiết trong các phiên hội thoại, vì hầu hết các
  lượt chỉ là câu hỏi tiếp nối.

---

# Retrieval Tuning (Precision Over Recall-Dumping)

**Addresses:** Cause 4.2 in [`../CAUSE.md`](../CAUSE.md) (with `document-reuse.md`)

**Idea:** Make the retrieval stage do its job — deliver a *small, precise*
context — instead of shifting the filtering burden onto the model by dumping
a large top-k. Every irrelevant chunk is paid for on this request and every
subsequent turn it survives in.

---

## How to apply

### 1. Add a reranking stage

First-stage retrieval (BM25/embeddings) is recall-oriented; rerank the
candidates and send only the survivors:

```
retrieve top-50 (cheap) → rerank → send top-3..5 (precise)
```

Cross-encoder rerankers are the single highest-leverage retrieval upgrade:
they let you cut k by 3–5× at equal-or-better answer quality.

### 2. Right-size k and chunks — measured, not guessed

- Sweep k on your eval set; answer quality typically plateaus at k=3–8 for
  well-reranked pipelines while cost grows linearly with k.
- Chunk size: smaller chunks (200–500 tokens) with **contextual headers**
  (title/section breadcrumbs, or generated context à la Anthropic's
  contextual-retrieval recipe) beat big 2K-token chunks — you send the
  relevant paragraph, not its whole neighborhood.
- Deduplicate near-identical chunks (common with overlapping windows) before
  sending.

### 3. Retrieve when needed, not always

- **Gate retrieval**: classify whether the turn needs retrieval at all
  (many turns are follow-ups answerable from context). A cheap classifier
  or embedding-similarity gate ("query drifted from what's already in
  context?") prevents reflexive per-turn attachment — see
  `context-hygiene.md`.
- **Agentic retrieval**: give the model a `search(query)` tool and let it
  pull what it decides it needs (1–2 targeted calls), instead of
  pre-stuffing every request. This converts fixed per-turn retrieval cost
  into on-demand cost.

### 4. Compress what you send

- Send extractive snippets (the matching passage ± a sentence) rather than
  full chunks where the task allows.
- Strip markup/boilerplate from chunks at indexing time, not at query time.
- Context-compression rerankers (e.g. LLMLingua-style) can squeeze retrieved
  context 2–5× for QA tasks, at some fidelity risk — eval before adopting.

## SOTA tools

### Native — coding agents & provider APIs

| Provider / agent | Feature | Notes |
| --- | --- | --- |
| Anthropic | Contextual retrieval recipe | Chunk-context generation at indexing time; published ~49% retrieval-failure reduction (67% with reranking) |
| OpenAI API | `text-embedding-3` + vector stores / file search | Managed first-stage retrieval inside the OpenAI stack |
| Coding agents' repo search (Claude Code `Grep`/`Glob`, Codex/Gemini CLI search tools) | Agentic retrieval | For codebases, bounded on-demand search often beats a pre-stuffed RAG pipeline entirely |

### Third-party — agent-agnostic (open source preferred)

| Tool | License | Notes |
| --- | --- | --- |
| BGE-reranker-v2 / mxbai-rerank | Apache-2.0 | Self-hosted SOTA-class cross-encoder reranking; Cohere Rerank 3.5 / Voyage rerank-2 / Jina Reranker v2 are hosted commercial equivalents |
| BGE-M3 embeddings | MIT | Strong open first-stage recall; Voyage-3 is the hosted commercial option |
| LLMLingua / LLMLingua-2 / LongLLMLingua | MIT | Microsoft prompt-compression family — up to ~20× compression at ~1.5-pt quality drop on QA/reasoning benchmarks (one reported SaaS case: $42K→$2.1K/mo); LongLLMLingua ~94% RAG-cost cut on LooGLE. **Strong fidelity caveat on code/structured text** — eval before adopting, and prefer Tier-0/1 removal on coding fleets (`../setups/recommended-setup.md` Tier 3) |
| RAGAS / promptfoo | Apache-2.0 / MIT | Measure the k/chunk/rerank sweeps instead of guessing; Braintrust is the commercial alternative |

## Trade-offs

- Reranking adds a hop (~50–300ms) and a per-query fee — almost always
  repaid by the smaller k.
- Aggressive precision tuning risks recall failures on multi-hop questions;
  keep an escape hatch (agentic follow-up retrieval).
- Compression techniques are task-sensitive — validate on your evals, not
  benchmarks alone.

## Expected impact

- k=20 unranked → k=4 reranked cuts the retrieval share of input **~5×**
  per request, compounding across every turn the chunks would have persisted.
- Anthropic's contextual-retrieval numbers (−49% retrieval failures, −67%
  with reranking) show precision tuning *improves quality while cutting
  tokens* — irrelevant chunks were hurting both.
- Retrieval gating typically eliminates 30–70% of retrieval attachments in
  conversational sessions (most turns are follow-ups).
