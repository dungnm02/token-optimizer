# Chỉnh sửa Context (Cắt tỉa các lượt cũ) (Tiếng Việt)

**Giải quyết:** Nguyên nhân 2.1 và 2.2 trong [`../CAUSE.md`](../CAUSE.md)

**Ý tưởng:** Thay vì (hoặc trước khi) tóm tắt, hãy **xóa** nội dung khỏi
transcript mà có thể chứng minh là đã cũ — kết quả tool cũ, các lần đọc file
đã bị thay thế, các khối reasoning đã hoàn tất — trong khi vẫn giữ nguyên bộ
khung cấu trúc của hội thoại. Cắt tỉa rẻ hơn tóm tắt (không cần lệnh gọi
model thêm) và không mất mát đối với nội dung thực sự không còn quan trọng.

---

## Nén (compaction) so với chỉnh sửa context (context editing)

| | Chỉnh sửa context (cắt tỉa) | Nén (tóm tắt) |
| --- | --- | --- |
| Cơ chế | Xóa/dọn các khối | Thay thế đoạn bằng bản tóm tắt sinh ra |
| Lệnh gọi model thêm | Không | Có |
| Mất mát thông tin | Chỉ những gì bạn chọn bỏ | Bất cứ điều gì bản tóm tắt bỏ sót |
| Mục tiêu tốt nhất | Kết quả tool cồng kềnh, đã bị thay thế | Lịch sử tường thuật dài gần giới hạn cửa sổ |
| Thứ tự điển hình | Liên tục, suốt cả phiên | Khi tiệm cận ngân sách context |

Chúng kết hợp với nhau: cắt tỉa liên tục, nén khi transcript đã cắt tỉa vẫn
còn tiệm cận giới hạn.

## Cách áp dụng

1. **Có sẵn từ nhà cung cấp (ưu tiên nếu có)** — *Anthropic context
   management* (beta `context-management-2025-06-27`):

   ```json
   "context_management": {
     "edits": [
       {"type": "clear_tool_uses_20250919", "clear_tool_inputs": true},
       {"type": "clear_thinking_20251015"}
     ]
   }
   ```

   API dọn các kết quả tool cũ (và tùy chọn cả input của lệnh gọi tool) và
   các khối thinking đã dùng xong dựa trên các ngưỡng có thể cấu hình —
   không cần sổ sách phía client.

2. **Quy tắc cắt tỉa cấp harness** (mọi nhà cung cấp):
   - *Thay thế (Supersession)*: khi một file được đọc lại hoặc ghi lại,
     thay các lần đọc trước đó của cùng đường dẫn bằng một stub: `"[nội
     dung đã bị thay thế — đọc lại nếu cần]"`.
   - *TTL theo khoảng cách lượt*: các kết quả tool cũ hơn K lượt được thu
     gọn thành một bản tóm lược một dòng (`"grep: 14 kết quả trong 3 file
     (đã cắt tỉa)"`).
   - *Giữ lệnh gọi, bỏ payload*: bảo toàn cặp `tool_use` / `tool_result`
     (nhiều API yêu cầu ID khớp nhau) nhưng thu nhỏ nội dung kết quả.
3. **Offload thay vì đưa trực tiếp vào**: cho các tool ghi output lớn vào
   một file và trả về đường dẫn + bản xem trước; model đọc từng phần theo
   yêu cầu. Điều này ngăn chặn sự phình to thay vì phải dọn nó sau (xem
   `tool-output-budgets.md`).
4. **Hỗ trợ từ framework**: LangGraph `trim_messages` / hook lọc tin nhắn,
   bộ đệm bộ nhớ LlamaIndex có giới hạn token, cắt tỉa phiên của OpenAI
   Agents SDK.

## Công cụ hiện đại nhất (SOTA)

### Có sẵn — coding agent & API của nhà cung cấp

| Nhà cung cấp / agent | Tính năng | Ghi chú |
| --- | --- | --- |
| Anthropic API | Chỉnh sửa `context_management` (`clear_tool_uses`, `clear_thinking`) | Dọn phía server các kết quả tool cũ và khối thinking đã dùng; không cần sổ sách phía client |
| Claude Code / Claude Agent SDK | Cắt tỉa kết quả tool cũ có sẵn (microcompaction) | Được áp dụng tự động bên trong harness |
| OpenAI Agents SDK | Hook cắt tỉa phiên | Một phần của stack OpenAI/Codex |

### Bên thứ ba — không phụ thuộc agent (ưu tiên mã nguồn mở)

| Công cụ | Giấy phép | Ghi chú |
| --- | --- | --- |
| LangGraph `trim_messages` / hook lọc tin nhắn | MIT | Quy tắc cắt tỉa có thể kết hợp cho các vòng lặp tùy chỉnh trên mọi nhà cung cấp |
| Bộ đệm bộ nhớ giới hạn token của LlamaIndex | MIT | Lịch sử được thực thi theo ngân sách cho mọi backend |
| Quy tắc cắt tỉa Supersession/TTL (tùy chỉnh, ~100 dòng code) | — | Các quy tắc cấp harness ở mục §2 có thể triển khai dễ dàng trong mọi stack |

## Đánh đổi

- Các heuristic cắt tỉa có thể xóa nhầm thứ gì đó model vẫn cần — model sẽ
  lấy lại (rẻ nếu tool rẻ, tệ nếu không). Ưu tiên *stub-kèm-con-trỏ* thay vì
  xóa âm thầm để việc phục hồi chỉ là một lệnh gọi tool.
- Chỉnh sửa giữa transcript vô hiệu hóa cache cấp tin nhắn từ điểm chỉnh
  sửa trở đi (phần đầu vẫn được cache). Gộp các đợt cắt tỉa thay vì cắt tỉa
  mỗi lượt.
- Dọn dẹp có sẵn từ nhà cung cấp cho ít quyền kiểm soát hơn về *cái gì*
  được dọn so với quy tắc tự viết tay.

## Tác động dự kiến

- Trong các transcript agent nặng về tool, kết quả tool thường chiếm
  **70–90% token lịch sử**; cắt tỉa những cái đã bị thay thế thường thu
  nhỏ context ổn định **2–5×** mà không mất mát do tóm tắt.
- Anthropic báo cáo rằng chỉnh sửa context kéo dài đáng kể độ dài tác vụ
  hiệu dụng trong khi *cải thiện* chất lượng trên các tác vụ dài (context
  cũ là sự phân tâm, không chỉ là chi phí).
- **Đây là một đòn bẩy chất lượng, không chỉ là chi phí — "context rot"**.
  Các nghiên cứu có kiểm soát trên 18 model
  frontier cho thấy mỗi model đều suy giảm khi input tăng lên, và điều này
  xảy ra *trước khi* cửa sổ gần đầy: ngân sách thực tế cho độ chính xác
  cao nằm trong khoảng 150–400K token ngay cả trên các model 2M token, và
  độ chính xác giảm nhanh nhất khi nhiễu tích lũy *tương đồng về mặt ngữ
  nghĩa* với câu trả lời (chính xác là trường hợp trong một phiên lập
  trình đầy các bước khám phá gần đúng). Cắt tỉa nhiễu đó mang lại cả độ
  chính xác lẫn token.
- Không tốn thêm chi phí lệnh gọi model nào, khác với nén.

---

# Context Editing (Pruning Stale Turns)

**Addresses:** Causes 2.1 and 2.2 in [`../CAUSE.md`](../CAUSE.md)

**Idea:** Instead of (or before) summarizing, **delete** content from the
transcript that is provably stale — old tool results, superseded file reads,
completed reasoning blocks — while keeping the conversation's structural
skeleton intact. Pruning is cheaper than summarizing (no extra model call)
and lossless for content that genuinely no longer matters.

---

## Compaction vs. context editing

| | Context editing (prune) | Compaction (summarize) |
| --- | --- | --- |
| Mechanism | Remove/clear blocks | Replace span with generated summary |
| Extra model call | No | Yes |
| Information loss | Only what you choose to drop | Whatever the summary misses |
| Best target | Bulky, superseded tool results | Long narrative history near the window limit |
| Typical order | Continuously, throughout the session | When approaching the context budget |

They compose: prune continuously, compact when the pruned transcript still
approaches the limit.

## How to apply

1. **Provider-native (preferred where available)** — *Anthropic context
   management* (beta `context-management-2025-06-27`):

   ```json
   "context_management": {
     "edits": [
       {"type": "clear_tool_uses_20250919", "clear_tool_inputs": true},
       {"type": "clear_thinking_20251015"}
     ]
   }
   ```

   The API clears old tool results (and optionally the tool-call inputs) and
   spent thinking blocks based on configurable thresholds — no client
   bookkeeping.

2. **Harness-level pruning rules** (any provider):
   - *Supersession*: when a file is re-read or re-written, replace earlier
     reads of the same path with a stub: `"[content superseded — re-read if
     needed]"`.
   - *TTL by turn distance*: tool results older than K turns collapse to a
     one-line digest (`"grep: 14 matches in 3 files (pruned)"`).
   - *Keep the call, drop the payload*: preserve the `tool_use` /
     `tool_result` pairing (many APIs require matched IDs) but shrink the
     result content.
3. **Offload instead of inline**: have tools write large outputs to a file
   and return the path + a preview; the model reads slices on demand. This
   prevents the bloat rather than cleaning it up (see
   `tool-output-budgets.md`).
4. **Framework support**: LangGraph `trim_messages` / message-filtering
   hooks, LlamaIndex memory buffers with token limits, OpenAI Agents SDK
   session trimming.

## SOTA tools

### Native — coding agents & provider APIs

| Provider / agent | Feature | Notes |
| --- | --- | --- |
| Anthropic API | `context_management` edits (`clear_tool_uses`, `clear_thinking`) | Server-side clearing of stale tool results and spent thinking blocks; no client bookkeeping |
| Claude Code / Claude Agent SDK | Built-in stale-tool-result pruning (microcompaction) | Applied inside the harness automatically |
| OpenAI Agents SDK | Session trimming hooks | Part of the OpenAI/Codex stack |

### Third-party — agent-agnostic (open source preferred)

| Tool | License | Notes |
| --- | --- | --- |
| LangGraph `trim_messages` / message-filtering hooks | MIT | Composable pruning rules for custom loops on any provider |
| LlamaIndex token-limited memory buffers | MIT | Budget-enforced history for any backend |
| Supersession/TTL pruning rules (custom, ~100 LOC) | — | The harness-level rules in §2 are trivially implementable in any stack |

## Trade-offs

- Pruning heuristics can drop something the model still needed — the model
  will re-fetch (cheap if the tool is cheap, bad if it isn't). Prefer
  *stub-with-pointer* over silent deletion so recovery is one tool call.
- Editing the middle of the transcript invalidates the message-level cache
  from the edit point on (the head stays cached). Batch prunes rather than
  pruning every turn.
- Provider-native clearing gives less control over *what* is cleared than
  hand-rolled rules.

## Expected impact

- In tool-heavy agent transcripts, tool results are commonly **70–90% of
  history tokens**; pruning superseded ones typically shrinks steady-state
  context by **2–5×** with no summarization loss.
- Anthropic reports context editing extends effective task length
  substantially while *improving* quality on long tasks (stale context is
  distraction, not just cost).
- **This is a quality lever, not only a cost one — "context rot."**
  Controlled studies across 18 frontier models find every one degrades as
  input grows, and *before* the window is anywhere near full: practical
  high-accuracy budgets land around 150–400K tokens even on 2M-token
  models, and accuracy drops fastest when the accumulated noise is
  *semantically similar* to the answer (exactly the case in a coding
  session full of near-miss exploration). Pruning that noise buys accuracy
  as well as tokens.
- Zero additional model-call cost, unlike compaction.
