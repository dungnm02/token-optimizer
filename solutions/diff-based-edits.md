# Chỉnh sửa dựa trên Diff (Không bao giờ Phát lại toàn bộ File) (Tiếng Việt)

**Giải quyết:** Nguyên nhân 5.2 trong [`../CAUSE.md`](../CAUSE.md) (phiên
bản chuyên biệt hóa cho coding agent của sự dài dòng output)

**Ý tưởng:** Với việc chỉnh sửa file — mẫu hình output chi phối của các
coding agent — hãy để model chỉ phát ra các **đoạn (hunk) đã thay đổi**
(khối search/replace, unified diff, lệnh gọi tool chỉnh sửa có mục tiêu),
không bao giờ toàn bộ file đã viết lại.

---

## Tại sao điều này quan trọng

Viết lại một file 1.000 dòng để đổi 5 dòng tốn ~1.000 dòng output (loại
token đắt nhất) cộng thêm, trong nhiều harness, file đã viết lại quay lại
ngữ cảnh dưới dạng input. Nhân điều này với mỗi lần chỉnh sửa trong một
phiên và việc viết lại toàn bộ file thường là chi phí *chi phối* của một
coding agent. Nó cũng chậm hơn (sinh nhiều token hơn = nhiều độ trễ hơn) và
rủi ro hơn (thay đổi vô tình ngoài ý muốn vào code chưa được động tới).

## Cách áp dụng

### 1. Chọn một định dạng chỉnh sửa mà model đáng tin cậy

| Định dạng | Hình dạng | Ghi chú |
| --- | --- | --- |
| **Search/replace (str-replace)** | `old_string` chính xác → `new_string`, được kiểm tra tính duy nhất | Được dùng bởi tool `Edit` của Claude Code và tool text-editor của Anthropic (`str_replace_based_edit_tool`); độ tin cậy cao nhất vì harness xác minh mỏ neo (anchor) tồn tại chính xác một lần |
| **Unified diff** | Các đoạn `-`/`+` chuẩn | Phát hiện dựa trên benchmark của Aider: các định dạng kiểu diff giảm mạnh việc lược bỏ "lười biếng" (`# ...phần còn lại không đổi`) so với toàn bộ file trên các model hàng đầu |
| **Thay thế/chèn theo khoảng dòng** | Chỉnh sửa theo số dòng | Đơn giản, nhưng dễ vỡ sau các chỉnh sửa đồng thời — đọc lại trước khi chỉnh sửa |
| **Toàn bộ file** | Nội dung đầy đủ | Chỉ dành cho file mới/nhỏ (dưới ~100 dòng, chi phí thấp hơn rủi ro lỗi định dạng) |

### 2. Thực thi trong harness, không chỉ trong prompt

- Cung cấp *tool chỉnh sửa* (`str_replace`, `apply_diff`) thay vì một
  "write file" chung chung để chỉnh sửa từng phần là con đường ít trở
  ngại nhất.
- Xác thực mỏ neo/đoạn phía harness (khớp đúng một lần; từ chối ngữ cảnh
  cũ) và trả về lỗi có thể hành động để một lần chỉnh sửa thất bại chỉ tốn
  một lần thử lại nhỏ, không phải quay về toàn bộ file.
- Theo dõi trạng thái file (hash tại lần đọc cuối) để harness có thể từ
  chối chỉnh sửa dựa trên nội dung cũ thay vì để model đọc lại + phát lại.

### 3. Dùng model fast-apply cho các lần gộp mờ (tùy chọn)

Một mẫu hình SOTA mới hơn: model hàng đầu phát ra một *bản phác thảo*
chỉnh sửa lỏng lẻo, và một "model áp dụng" chuyên biệt nhỏ gộp nó vào file
với tốc độ hàng nghìn token/giây. Điều này giữ output của model hàng đầu ở
mức tối thiểu trong khi vẫn chấp nhận các mỏ neo không hoàn hảo.

### 4. Cùng nguyên tắc ngoài phạm vi code

- Tài liệu dài: phát ra phần đã sửa, không phải toàn bộ tài liệu.
- Trạng thái Config/JSON: phát ra một patch (JSON Patch/merge patch),
  không phải toàn bộ đối tượng.
- Chỉnh sửa hội thoại: các phản hồi "đổi đoạn thứ hai thành…" nên xuất ra
  đoạn đó, không phải cả bài luận.

## Công cụ hiện đại nhất (SOTA)

### Có sẵn — coding agent & API của nhà cung cấp

| Nhà cung cấp / agent | Tính năng | Ghi chú |
| --- | --- | --- |
| Claude Code · Anthropic API | Tool `Edit` / `text_editor_20250728` | Search/replace được xác minh mỏ neo; triển khai tham chiếu |
| Codex CLI · OpenAI API | Định dạng chỉnh sửa `apply_patch`; predicted outputs | Định dạng diff có sẵn của Codex; predicted outputs tăng tốc/giảm giá việc tái sinh khi không thể tránh khỏi output toàn bộ file |
| Gemini CLI | Tool chỉnh sửa `replace` | Chỉnh sửa từng phần dựa trên mỏ neo trong harness |

### Bên thứ ba — không phụ thuộc agent (ưu tiên mã nguồn mở)

| Công cụ | Giấy phép | Ghi chú |
| --- | --- | --- |
| Định dạng chỉnh sửa của Aider (`diff`, `udiff`) + benchmark polyglot | Apache-2.0 | Dữ liệu benchmark công khai về độ tin cậy định dạng chỉnh sửa theo từng model — dùng nó để chọn định dạng cho mọi harness |
| Trình xác thực patch dựa trên tree-sitter | MIT | Kiểm tra cú pháp các đoạn trước khi ghi ra đĩa, bất kể agent nào đã phát ra chúng |
| Model fast-apply Morph / Relace | Thương mại | Gộp tốc độ cao chuyên biệt cho các chỉnh sửa lỏng lẻo (~hàng nghìn token/giây) đứng sau bất kỳ model hàng đầu nào |

## Đánh đổi

- Định dạng chỉnh sửa đôi khi thất bại (không tìm thấy mỏ neo, đoạn dị
  dạng) — mỗi thất bại tốn một lượt thử lại. Độ tin cậy khác nhau theo
  từng model; hãy benchmark (bảng của Aider) trước khi bắt buộc một định
  dạng.
- Search/replace cần mỏ neo duy nhất; các file lặp lại nhiều buộc phải
  dùng chuỗi ngữ cảnh lớn hơn.
- Fast-apply thêm một phụ thuộc model thứ hai và các chế độ lỗi riêng của
  nó.

## Tác động dự kiến

- Token output mỗi lần chỉnh sửa giảm khoảng theo **tỷ lệ kích thước file
  : kích thước đoạn** — thường **10–50×** trên các codebase thực tế
  (5–20 dòng thay đổi trong các file 500–2.000 dòng).
- Độ trễ mỗi lần chỉnh sửa giảm tương ứng (sinh output là nút thắt cổ
  chai), cộng dồn qua hàng chục lần chỉnh sửa mỗi phiên của một agent.
- Thắng lợi chất lượng phụ: không có hồi quy vô tình ở các vùng chưa được
  động tới, và diff review khớp với ý định.

---

# Diff-Based Edits (Never Re-Emit the File)

**Addresses:** Cause 5.2 in [`../CAUSE.md`](../CAUSE.md) (coding-agent
specialization of output verbosity)

**Idea:** For file modification — the dominant output pattern of coding
agents — have the model emit only the **changed hunks** (search/replace
blocks, unified diffs, targeted edit-tool calls), never the whole rewritten
file.

---

## Why it matters

Rewriting a 1,000-line file to change 5 lines costs ~1,000 lines of output
(the most expensive token class) plus, in many harnesses, the rewritten file
re-entering context as input. Multiply by every edit in a session and
whole-file rewrite is often *the* dominant cost of a coding agent. It's also
slower (more tokens to generate = more latency) and riskier (accidental
drive-by changes to untouched code).

## How to apply

### 1. Pick an edit format the model is reliable at

| Format | Shape | Notes |
| --- | --- | --- |
| **Search/replace (str-replace)** | Exact `old_string` → `new_string`, uniqueness-checked | Used by Claude Code's `Edit` tool and Anthropic's text-editor tool (`str_replace_based_edit_tool`); highest reliability because the harness verifies the anchor exists exactly once |
| **Unified diff** | Standard `-`/`+` hunks | Aider's benchmark-driven finding: diff-style formats sharply reduce "lazy" elisions (`# ...rest unchanged`) vs whole-file on frontier models |
| **Line-range replace / insert** | Edit by line numbers | Simple, but brittle after concurrent edits — re-read before editing |
| **Whole file** | Full content | Only for new/small files (below ~100 lines the overhead beats format failure risk) |

### 2. Enforce it in the harness, not just the prompt

- Expose *edit tools* (`str_replace`, `apply_diff`) rather than a generic
  "write file" so partial edits are the path of least resistance.
- Validate anchors/hunks harness-side (exactly-one-match; reject stale
  context) and return actionable errors so a failed edit costs one small
  retry, not a whole-file fallback.
- Track file state (hash at last read) so the harness can reject edits
  against stale content instead of letting the model re-read + re-emit.

### 3. Use fast-apply models for fuzzy merges (optional)

A newer SOTA pattern: the frontier model emits a *loose* edit sketch, and a
small specialized "apply model" merges it into the file at thousands of
tokens/sec. This keeps frontier output minimal while tolerating imperfect
anchors.

### 4. Same principle beyond code

- Long documents: emit the revised section, not the whole document.
- Config/JSON state: emit a patch (JSON Patch / merge patch), not the full
  object.
- Conversational revisions: "change the second paragraph to…" responses
  should output the paragraph, not the essay.

## SOTA tools

### Native — coding agents & provider APIs

| Provider / agent | Feature | Notes |
| --- | --- | --- |
| Claude Code · Anthropic API | `Edit` tool / `text_editor_20250728` | Anchor-verified search/replace; the reference implementation |
| Codex CLI · OpenAI API | `apply_patch` edit format; predicted outputs | Codex's native diff format; predicted outputs speed/discount regeneration when whole-file output is unavoidable |
| Gemini CLI | `replace` (edit) tool | Anchor-based partial edits in the harness |

### Third-party — agent-agnostic (open source preferred)

| Tool | License | Notes |
| --- | --- | --- |
| Aider edit formats (`diff`, `udiff`) + polyglot benchmark | Apache-2.0 | Public benchmark data on edit-format reliability per model — use it to pick formats for any harness |
| tree-sitter-based patch validators | MIT | Syntax-check hunks before writing to disk, regardless of which agent emitted them |
| Morph / Relace fast-apply models | Commercial | Specialized high-speed merge of loose edits (~thousands of tokens/s) behind any frontier model |

## Trade-offs

- Edit formats fail sometimes (anchor not found, malformed hunk) — each
  failure costs a retry round trip. Reliability varies by model; benchmark
  (Aider's tables) before mandating a format.
- Search/replace needs unique anchors; highly repetitive files force larger
  context strings.
- Fast-apply adds a second model dependency and its own error modes.

## Expected impact

- Output tokens per edit drop roughly by the **file-size : hunk-size ratio**
  — commonly **10–50×** on real codebases (5–20 changed lines in
  500–2,000-line files).
- Latency per edit drops proportionally (output generation is the
  bottleneck), which compounds across an agent's dozens of edits per
  session.
- Secondary quality win: no drive-by regressions in untouched regions, and
  review diffs match intent.
