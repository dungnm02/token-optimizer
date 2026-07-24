# Bảng tổng hợp: nguyên nhân → giải pháp → công cụ (Tiếng Việt)

Một trang để tra nhanh. Mỗi hàng ghép một **nguyên nhân** tốn token với
**cách khắc phục** và **công cụ** giúp bạn làm điều đó. Chi tiết đầy đủ nằm
trong [`CAUSE.md`](CAUSE.md) và thư mục [`solutions/`](solutions/).

**Cột "NCC" (nhà cung cấp):** dấu ▲ nghĩa là bạn chỉ nắm được khoản tiết kiệm
này nếu **nhà cung cấp** hỗ trợ — giá cache, nút reasoning-effort, các tier
model/batch rẻ hơn. Đó là những thứ bạn không tự tạo ra được, chỉ có thể tận
dụng. Ô trống nghĩa là bạn tự giải quyết được bằng công cụ bên thứ ba hoặc
kiến trúc của mình. Cột "Công cụ" để trống khi cách khắc phục thuần là kỷ
luật prompt/kiến trúc, không có công cụ rời để cài.

### 1 — Lỗi caching

| Nguyên nhân | Giải pháp | Công cụ | NCC |
| --- | --- | --- | --- |
| 1.1 Không có caching prompt | [`prompt-caching`](solutions/prompt-caching.md) | Caching của nhà cung cấp (native); telemetry: Langfuse, Helicone, LiteLLM | ▲ |
| 1.2 Vô hiệu hóa cache âm thầm | [`prompt-caching`](solutions/prompt-caching.md) | Render tất định + kiểm thử byte trong CI (phía bạn) | ▲ |
| 1.3 Thay đổi giữa phiên làm mới cache | [`prompt-caching`](solutions/prompt-caching.md), [`stable-prompt-architecture`](solutions/stable-prompt-architecture.md) | Kiến trúc prompt ổn định (phía bạn); tự host vLLM/SGLang | ▲ |
| 1.4 Cache hết hạn giữa các request | [`prompt-caching`](solutions/prompt-caching.md) | TTL dài hơn / lưu lượng giữ ấm (cấu hình nhà cung cấp) | ▲ |

### 2 — Tích lũy context

| Nguyên nhân | Giải pháp | Công cụ | NCC |
| --- | --- | --- | --- |
| 2.1 Lịch sử hội thoại không giới hạn | [`compaction`](solutions/compaction.md), [`context-editing`](solutions/context-editing.md) | API compaction/context-editing (native); LangGraph, LlamaIndex | |
| 2.2 Kết quả tool cũ còn trong lịch sử | [`context-editing`](solutions/context-editing.md) | Cắt tỉa trong harness; LangGraph, LlamaIndex | |
| 2.3 Chèn context trùng lặp | [`context-hygiene`](solutions/context-hygiene.md) | Registry hash (phía bạn) | |

### 3 — Cách dùng tool

| Nguyên nhân | Giải pháp | Công cụ | NCC |
| --- | --- | --- | --- |
| 3.1 Output tool quá lớn | [`tool-output-budgets`](solutions/tool-output-budgets.md), [`tool-output-compression`](solutions/tool-output-compression.md) | jq, Trafilatura; RTK, Headroom (Apache-2.0), Caveman | |
| 3.2 Nhiều round-trip thay vì kết hợp | [`tool-composition`](solutions/tool-composition.md) | Code Mode (native); kết hợp phía server | |
| 3.3 Vòng lặp thử lại và polling | [`event-driven-waiting`](solutions/event-driven-waiting.md) | Temporal, webhook | |
| 3.4 Quá nhiều schema tool tải sẵn | [`tool-search`](solutions/tool-search.md) | Tải tool trì hoãn / tìm kiếm tool (native); Code Mode | |

### 4 — Loại nội dung đắt đỏ

| Nguyên nhân | Giải pháp | Công cụ | NCC |
| --- | --- | --- | --- |
| 4.1 Hình ảnh độ phân giải đầy đủ | [`image-downsampling`](solutions/image-downsampling.md) | sharp, Pillow | |
| 4.2 Đổ nguyên tài liệu (PDF/RAG rộng) | [`document-reuse`](solutions/document-reuse.md), [`retrieval-tuning`](solutions/retrieval-tuning.md), [`code-maps`](solutions/code-maps.md) | Docling, unstructured; reranker BGE, LLMLingua; aider, Repomix, Codesight | |
| 4.3 Nội dung tốn kém khi tokenize | [`token-counting`](solutions/token-counting.md) | Endpoint `count_tokens` của nhà cung cấp (native) | |

### 5 — Chi tiêu phía sinh (generation)

| Nguyên nhân | Giải pháp | Công cụ | NCC |
| --- | --- | --- | --- |
| 5.1 Token reasoning/thinking | [`reasoning-effort-tuning`](solutions/reasoning-effort-tuning.md) | Nút reasoning-effort / ngân sách thinking (native) | ▲ |
| 5.2 Output dài dòng | [`concise-output-prompting`](solutions/concise-output-prompting.md), [`diff-based-edits`](solutions/diff-based-edits.md) | Structured output (native); định dạng diff của Aider | |
| 5.3 Chu kỳ cắt bớt và thử lại | [`output-cap-sizing`](solutions/output-cap-sizing.md) | Retry có kiểm định: Instructor | |

### 6 — Lựa chọn kiến trúc

| Nguyên nhân | Giải pháp | Công cụ | NCC |
| --- | --- | --- | --- |
| 6.1 Subagent cold-start | [`subagent-context-handoff`](solutions/subagent-context-handoff.md), [`prompt-caching`](solutions/prompt-caching.md) | Kho lưu artifact (phía bạn) | |
| 6.2 Model quá lớn so với tác vụ | [`model-routing`](solutions/model-routing.md), [`batch-processing`](solutions/batch-processing.md), [`semantic-caching`](solutions/semantic-caching.md) | RouteLLM, LiteLLM, Portkey; tier batch của nhà cung cấp | ▲ |
| 6.3 Fan-out cache lạnh đồng thời | [`fan-out-warming`](solutions/fan-out-warming.md) | Điều phối warm-then-fan (phía bạn) | |
| 6.4 Prompt và scaffolding quá quy định | [`prompt-de-scaffolding`](solutions/prompt-de-scaffolding.md) | Rà soát prompt (phía bạn) | |
| 6.5 Cold-start giữa các phiên | [`code-maps`](solutions/code-maps.md), [`subagent-context-handoff`](solutions/subagent-context-handoff.md), [`compaction`](solutions/compaction.md) | aider, Repomix, Codesight | |
| 6.6 Request trùng lặp trên toàn hệ thống | [`semantic-caching`](solutions/semantic-caching.md), [`batch-processing`](solutions/batch-processing.md) | GPTCache; semantic cache của LiteLLM/Portkey | |

Đo lường trước tiên: [`token-counting`](solutions/token-counting.md) là lớp
cho biết nguyên nhân nào đang thực sự tốn tiền. Muốn có thứ tự áp dụng đề
xuất, xem [`solutions/README.md`](solutions/README.md).

---

# Summary: cause → solution → tool

A one-page lookup. Each row pairs a **cause** of token spend with the **fix**
and the **tool** that helps you apply it. Full detail lives in
[`CAUSE.md`](CAUSE.md) and the [`solutions/`](solutions/) folder.

**The "Provider" column:** a ▲ means you can only capture this saving if your
**provider** offers it — the caching discount, the reasoning-effort dial, the
cheaper model/batch tiers. Those are things you can't create yourself, only
qualify for. A blank means you can solve it with third-party tooling or your
own architecture. The "Tool(s)" column is left blank when the fix is purely
prompt/architecture discipline with no separate tool to install.

### 1 — Caching failures

| Cause | Fix | Tool(s) | Provider |
| --- | --- | --- | --- |
| 1.1 No prompt caching at all | [`prompt-caching`](solutions/prompt-caching.md) | Provider prompt caching (native); telemetry: Langfuse, Helicone, LiteLLM | ▲ |
| 1.2 Silent cache invalidators | [`prompt-caching`](solutions/prompt-caching.md) | Deterministic render + CI byte-tests (your side) | ▲ |
| 1.3 Mid-session cache rebuilds | [`prompt-caching`](solutions/prompt-caching.md), [`stable-prompt-architecture`](solutions/stable-prompt-architecture.md) | Stable prompt architecture (your side); self-hosted vLLM/SGLang | ▲ |
| 1.4 Cache expiry between requests | [`prompt-caching`](solutions/prompt-caching.md) | Longer TTL / keep-alive traffic (provider config) | ▲ |

### 2 — Context accumulation

| Cause | Fix | Tool(s) | Provider |
| --- | --- | --- | --- |
| 2.1 Unbounded conversation history | [`compaction`](solutions/compaction.md), [`context-editing`](solutions/context-editing.md) | Compaction/context-editing APIs (native); LangGraph, LlamaIndex | |
| 2.2 Stale tool results kept in history | [`context-editing`](solutions/context-editing.md) | Harness-side pruning; LangGraph, LlamaIndex | |
| 2.3 Duplicate context injection | [`context-hygiene`](solutions/context-hygiene.md) | Hash registry (your side) | |

### 3 — Tool usage patterns

| Cause | Fix | Tool(s) | Provider |
| --- | --- | --- | --- |
| 3.1 Oversized tool outputs | [`tool-output-budgets`](solutions/tool-output-budgets.md), [`tool-output-compression`](solutions/tool-output-compression.md) | jq, Trafilatura; RTK, Headroom (Apache-2.0), Caveman | |
| 3.2 Chatty round-trips vs. composition | [`tool-composition`](solutions/tool-composition.md) | Code Mode (native); server-side composition | |
| 3.3 Retry and polling loops | [`event-driven-waiting`](solutions/event-driven-waiting.md) | Temporal, webhooks | |
| 3.4 Too many tool schemas upfront | [`tool-search`](solutions/tool-search.md) | Deferred tool loading / tool search (native); Code Mode | |

### 4 — Expensive content types

| Cause | Fix | Tool(s) | Provider |
| --- | --- | --- | --- |
| 4.1 Full-resolution images | [`image-downsampling`](solutions/image-downsampling.md) | sharp, Pillow | |
| 4.2 Whole-document dumping (PDF/RAG) | [`document-reuse`](solutions/document-reuse.md), [`retrieval-tuning`](solutions/retrieval-tuning.md), [`code-maps`](solutions/code-maps.md) | Docling, unstructured; BGE reranker, LLMLingua; aider, Repomix, Codesight | |
| 4.3 Tokenizer-expensive content | [`token-counting`](solutions/token-counting.md) | Provider `count_tokens` endpoint (native) | |

### 5 — Generation-side spend

| Cause | Fix | Tool(s) | Provider |
| --- | --- | --- | --- |
| 5.1 Reasoning/thinking tokens | [`reasoning-effort-tuning`](solutions/reasoning-effort-tuning.md) | Reasoning-effort / thinking-budget dials (native) | ▲ |
| 5.2 Output verbosity | [`concise-output-prompting`](solutions/concise-output-prompting.md), [`diff-based-edits`](solutions/diff-based-edits.md) | Structured output (native); Aider diff format | |
| 5.3 Truncation-and-retry cycles | [`output-cap-sizing`](solutions/output-cap-sizing.md) | Validation-aware retries: Instructor | |

### 6 — Architectural choices

| Cause | Fix | Tool(s) | Provider |
| --- | --- | --- | --- |
| 6.1 Cold-start subagents | [`subagent-context-handoff`](solutions/subagent-context-handoff.md), [`prompt-caching`](solutions/prompt-caching.md) | Artifact store (your side) | |
| 6.2 Oversized model for the task | [`model-routing`](solutions/model-routing.md), [`batch-processing`](solutions/batch-processing.md), [`semantic-caching`](solutions/semantic-caching.md) | RouteLLM, LiteLLM, Portkey; provider batch tier | ▲ |
| 6.3 Concurrent cold-cache fan-out | [`fan-out-warming`](solutions/fan-out-warming.md) | Warm-then-fan orchestration (your side) | |
| 6.4 Over-prescriptive prompts/scaffolding | [`prompt-de-scaffolding`](solutions/prompt-de-scaffolding.md) | Prompt audit (your side) | |
| 6.5 Cross-session cold starts | [`code-maps`](solutions/code-maps.md), [`subagent-context-handoff`](solutions/subagent-context-handoff.md), [`compaction`](solutions/compaction.md) | aider, Repomix, Codesight | |
| 6.6 Fleet-duplicate requests | [`semantic-caching`](solutions/semantic-caching.md), [`batch-processing`](solutions/batch-processing.md) | GPTCache; LiteLLM/Portkey semantic cache | |

Measure first: [`token-counting`](solutions/token-counting.md) is the layer
that tells you which cause is actually costing you. For a suggested adoption
order, see [`solutions/README.md`](solutions/README.md).
