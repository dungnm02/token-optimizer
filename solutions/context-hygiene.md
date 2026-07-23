# Vệ sinh Context (Loại bỏ chèn trùng lặp) (Tiếng Việt)

**Giải quyết:** Nguyên nhân 2.3 trong [`../CAUSE.md`](../CAUSE.md)

**Ý tưởng:** Đảm bảo mỗi nội dung (một file, một schema, một tài liệu
truy xuất) chỉ đi vào cuộc hội thoại **nhiều nhất một lần**. Sau đó, chỉ
tham chiếu đến nó — không dán lại nội dung nữa.

---

## Cách áp dụng

1. **Theo dõi những gì đã có trong context.** Harness giữ một registry
   các artifact đã chèn (đường dẫn/URL/doc-ID → hash nội dung + lượt đã
   chèn). Trước khi chèn, kiểm tra registry:
   - Cùng hash đã có sẵn → chèn một tham chiếu thay thế: *"`schema.sql` đã
     có trong context (lượt 4, không đổi)."*
   - Có mặt nhưng đã thay đổi trên đĩa → chèn một **diff** so với phiên
     bản trong context, không phải toàn bộ file.
2. **Sửa các pipeline truy xuất cứ tự động gắn lại dữ liệu mỗi lượt.** RAG
   kiểu ngây thơ dán top-k chunk vào *mọi* tin nhắn của người dùng; trong
   một phiên chỉ xoay quanh một chủ đề, đó là cùng một tập chunk bị tính
   phí lặp lại ở từng lượt. Thay vào đó, chỉ nên truy xuất lại mỗi khi chủ
   đề thay đổi, giữ kết quả đó làm một khối context ổn định (có thể cache),
   và chỉ chạy lại truy xuất khi truy vấn trôi dạt (dùng ngưỡng khoảng cách
   embedding để phát hiện).
3. **Tham chiếu bằng định danh khi nhà cung cấp hỗ trợ.** Tải lên một lần
   và tham chiếu bằng ID — `file_id` của Anthropic Files API, OpenAI
   Files/vector store, Gemini File API — thay vì nhúng lại các byte vào mỗi
   request (xem `document-reuse.md`).
4. **Loại bỏ trùng lặp các reminder của harness.** Các khối
   system-reminder, boilerplate an toàn, và tóm lược schema mà harness tự
   động chèn nên chỉ được chèn một lần, hoặc được thu gọn khi lặp lại — hãy
   rà soát những gì framework của bạn thêm vào mỗi lượt.
5. **Làm cho tool nhận biết tính bất biến (idempotent-aware).** Một tool
   `read_file` có thể trả lời từ registry ("không đổi kể từ lượt 4") thay
   vì trả lại toàn bộ nội dung — model nhận được xác nhận với ~10 token
   thay vì 10.000.

## Công cụ hiện đại nhất (SOTA)

### Có sẵn — coding agent & API của nhà cung cấp

| Nhà cung cấp / agent | Tính năng | Ghi chú |
| --- | --- | --- |
| Claude Code / Claude Agent SDK | Theo dõi trạng thái file | Theo dõi trạng thái đọc/sửa; giúp tránh đọc lại dư thừa ("trạng thái file hiện đang cập nhật trong context của bạn") |
| Anthropic Files API / OpenAI Files / Gemini File API | Tải lên một lần, tham chiếu bằng ID | Không cần truyền lại byte; kết hợp với caching để giảm chi phí xử lý |

### Bên thứ ba — không phụ thuộc agent (ưu tiên mã nguồn mở)

| Công cụ | Giấy phép | Ghi chú |
| --- | --- | --- |
| Kiểm soát tương đồng truy vấn của LlamaIndex / LangChain | MIT | Chỉ truy xuất lại khi trôi dạt chủ đề; độc lập với nhà cung cấp |
| Registry hash nội dung (tùy chỉnh, ~50 dòng code) | — | Cơ chế cốt lõi; triển khai dễ dàng trong mọi stack, mọi agent |

## Đánh đổi

- Registry phải được vô hiệu hóa đúng lúc khi artifact gốc thay đổi. Nếu
  tham chiếu "đã có trong context" bị cũ, đó là lỗi về tính đúng đắn, không
  chỉ là lỗi lãng phí token — hãy hash nội dung byte hiện tại, đừng dựa vào
  timestamp.
- Việc chèn dạng diff giả định rằng model có thể tự áp dụng diff trong đầu;
  với các file bị thay đổi nhiều, đọc lại toàn bộ file mới sẽ an toàn hơn.

## Tác động dự kiến

- Các phiên có truy cập file lặp lại thường lãng phí **20–40%** token
  lịch sử vào nội dung trùng lặp; một registry hash loại bỏ gần như toàn
  bộ điều đó.
- Loại bỏ trùng lặp trong truy xuất giúp chi phí RAG mỗi lượt giảm từ
  `k_chunks × số lượt` xuống còn `k_chunks × số lần đổi chủ đề` — thường
  giảm **5–20×** tỷ trọng truy xuất trong input của các phiên dài.

---

# Context Hygiene (Deduplicating Injection)

**Addresses:** Cause 2.3 in [`../CAUSE.md`](../CAUSE.md)

**Idea:** Guarantee that any given piece of content (a file, a schema, a
retrieved document) enters the conversation **at most once**, and is
referenced — not re-pasted — afterwards.

---

## How to apply

1. **Track what's already in context.** The harness keeps a registry of
   injected artifacts (path/URL/doc-ID → content hash + turn injected).
   Before injecting, check the registry:
   - Same hash already present → inject a reference instead: *"`schema.sql`
     is already in context (turn 4, unchanged)."*
   - Present but changed on disk → inject a **diff** against the in-context
     version, not the whole file.
2. **Fix retrieval pipelines that re-attach every turn.** Naive RAG glues
   the top-k chunks to *every* user message; in a session about one topic
   that's the same chunks re-billed each turn. Retrieve once per topic
   shift, keep results as a stable (cacheable) context block, and re-run
   retrieval only when the query drifts (embedding-distance gate).
3. **Reference by identifier where the provider supports it.** Upload once
   and reference by ID — Anthropic Files API `file_id`, OpenAI Files /
   vector stores, Gemini File API — instead of re-embedding the bytes into
   every request (see `document-reuse.md`).
4. **De-duplicate harness reminders.** System-reminder blocks, safety
   boilerplate, and schema recaps that the harness auto-injects should be
   injected once, or collapsed when repeated — audit what your framework
   adds per turn.
5. **Make tools idempotent-aware.** A `read_file` tool can answer from the
   registry ("unchanged since turn 4") instead of returning the full content
   again — the model gets its confirmation for ~10 tokens instead of 10,000.

## SOTA tools

### Native — coding agents & provider APIs

| Provider / agent | Feature | Notes |
| --- | --- | --- |
| Claude Code / Claude Agent SDK | File-state tracking | Tracks read/edit state; discourages redundant re-reads ("file state is current in your context") |
| Anthropic Files API / OpenAI Files / Gemini File API | Upload-once, reference-by-ID | Removes byte re-transmission; pair with caching for processing cost |

### Third-party — agent-agnostic (open source preferred)

| Tool | License | Notes |
| --- | --- | --- |
| LlamaIndex / LangChain query-similarity gating | MIT | Re-retrieve only on topic drift; provider-independent |
| Content-hash registry (custom, ~50 LOC) | — | The core mechanism; trivially implementable in any stack, any agent |

## Trade-offs

- The registry must be invalidated correctly when the underlying artifact
  changes — a stale "already in context" reference is a correctness bug, not
  just a token bug. Hash the current bytes, don't trust timestamps.
- Diff-injection assumes the model can apply diffs mentally; for heavily
  changed files, a fresh full read is safer.

## Expected impact

- Sessions with repeated file access commonly waste **20–40%** of history
  tokens on duplicate content; a hash registry eliminates nearly all of it.
- Retrieval dedup turns per-turn RAG cost from `k_chunks × turns` into
  `k_chunks × topic_shifts` — often a **5–20×** reduction on the retrieval
  share of input in long sessions.
