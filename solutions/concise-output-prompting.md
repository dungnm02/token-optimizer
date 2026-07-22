# Prompt Output Ngắn gọn (Tiếng Việt)

**Giải quyết:** Nguyên nhân 5.2 trong [`../CAUSE.md`](../CAUSE.md) (cùng với
`diff-based-edits.md`)

**Ý tưởng:** Output là loại token đắt nhất *và* trở thành input trên mọi
lượt sau đó. Thiết kế hợp đồng output — qua prompt, định dạng có cấu trúc,
tham số dài dòng, và điều kiện dừng — để model cung cấp thông tin, không
phải nghi thức rườm rà.

---

## Cách áp dụng

### 1. Đưa ra một hợp đồng output tường minh trong system prompt

Các đòn bẩy đáng tin cậy, theo thứ tự hiệu quả:

- **Dẫn dắt bằng câu trả lời + khung chọn lọc**: *"Dẫn dắt bằng kết quả.
  Giữ output ngắn bằng cách chọn lọc những gì đưa vào — bỏ các chi tiết
  không thay đổi việc người đọc làm tiếp theo — không phải bằng cách nén
  văn bản thành các mảnh vụn."* (Hướng dẫn chọn lọc thắng "hãy ngắn gọn",
  điều mà các model thường thỏa mãn bằng cách cắt bớt văn phong, không
  phải nội dung.)
- **Cấm nghi thức một cách tường minh**: không mở đầu ("Chắc chắn rồi! Đây
  là…"), không nhắc lại câu hỏi, không tóm tắt kết thúc, không tường thuật
  các hành động thường lệ trong vòng lặp agent (*"mặc định im lặng giữa
  các lệnh gọi tool; một câu khi có gì đó thay đổi"*).
- **Ví dụ tích cực hơn lệnh cấm**: một hoặc hai ví dụ về hình dạng câu trả
  lời mong muốn hiệu quả hơn danh sách những điều không nên làm.
- **Hiệu chỉnh, đừng tối thiểu hóa, khi độ dài chính là sản phẩm** — với
  báo cáo và giải thích, hãy chỉ rõ *hình dạng mục tiêu* ("≤5 gạch đầu
  dòng, mỗi dòng một ý") thay vì một từ "ngắn gọn" mơ hồ.

### 2. Dùng structured output như một ràng buộc độ dài

Một schema JSON (chế độ structured-output của nhà cung cấp) là kiểm soát
dài dòng mạnh nhất hiện có: model chỉ có thể phát ra các trường bạn đã
định nghĩa. Với các route trích xuất/phân loại/định tuyến, output ràng
buộc bởi schema loại bỏ hoàn toàn văn xuôi — thường cắt giảm output 3–10×
so với văn bản tự do.

### 3. Dùng tham số dài dòng/độ dài có sẵn khi được cung cấp

- Dòng GPT-5 của OpenAI: `verbosity: low|medium|high` — điều chỉnh độ dài
  mà không cần can thiệp prompt.
- Giới hạn output (`max_tokens`) như một *lưới an toàn*, không phải kiểm
  soát chính — cắt bớt là thất bại, không phải sự ngắn gọn
  (`output-cap-sizing.md`).
- Chuỗi `stop` để kết thúc các định dạng đã biết điểm dừng (ví dụ kết thúc
  JSON).

### 4. Gỡ bỏ khung sườn gây ra sự dài dòng

Các bản tóm tắt tiến độ bắt buộc, "giải thích lý luận của bạn" sót lại từ
thời trước-model-reasoning, các mẫu section bắt buộc — hãy rà soát và loại
bỏ những gì model hiện tại đã làm tốt mà không cần yêu cầu
(`prompt-de-scaffolding.md`).

### 5. Chú ý tiếng vọng nhiều lượt

Trong các vòng lặp agent, mỗi câu trả lời dài dòng đều bị tính phí lại như
lịch sử ở mỗi lượt sau đó. Vì vậy các chỉ dẫn ngắn gọn trả giá theo kiểu
bậc hai trong các phiên dài — ưu tiên chúng cho các agent chạy dài nhất
của bạn.

## Công cụ hiện đại nhất (SOTA)

### Có sẵn — coding agent & API của nhà cung cấp

| Nhà cung cấp / agent | Tính năng | Ghi chú |
| --- | --- | --- |
| Anthropic API | Structured output (`output_config.format`) | Giới hạn cứng về hình dạng output; đòn bẩy mạnh nhất cho các route không phải văn xuôi |
| Anthropic API | Token-efficient tool use (header beta) | Nén định dạng output *lệnh gọi tool* — giảm tới ~70% token output trên các lượt nặng về tool, trung bình ~14%; có trên API/Bedrock/Vertex |
| OpenAI API | Structured output + tham số `verbosity` | Nút điều chỉnh độ dài có sẵn trên các model hỗ trợ |
| Google Gemini API | `responseSchema` | Output ràng buộc bởi schema |
| Claude Code | Output style / quy ước `CLAUDE.md` | Hợp đồng output bền vững cho harness mà không cần can thiệp từng prompt |

### Bên thứ ba — không phụ thuộc agent (ưu tiên mã nguồn mở)

| Công cụ | Giấy phép | Ghi chú |
| --- | --- | --- |
| Caveman (`wilpel/caveman-compression`) | MIT | Nén output ngữ nghĩa — loại bỏ ngữ pháp có thể đoán trước, giữ lại sự kiện. Cung cấp dưới dạng một prompt skill dùng được trên Claude Code, Codex, Gemini CLI, Cursor, Cline và các agent khác (biến thể skill công bố cắt tới ~75% output; các chế độ đo được của thư viện chạy ở mức 15–58%). Chỉ dùng cho lưu lượng nội bộ/agent — không dùng cho văn xuôi hướng người dùng nơi giọng điệu quan trọng |
| Instructor / output có kiểu Zod | MIT | Schema có kiểu đồng thời đóng vai trò hợp đồng dài dòng, di động qua các nhà cung cấp |
| Đánh giá promptfoo / Langfuse | MIT | Kiểm thử hồi quy để đảm bảo các chỉnh sửa ngắn gọn không cắt mất *nội dung*; theo dõi token output theo từng route; Braintrust là lựa chọn thương mại |

## Đánh đổi

- Hợp đồng quá chặt làm mất giải thích thực sự hữu ích — đo lường thành
  công tác vụ, không chỉ số token, khi thắt chặt.
- Sự ngắn gọn quá mạnh tay làm hại khả năng đọc cho người tiêu thụ; "chọn
  lọc, không nén" là cách đóng khung tránh được lối viết mảnh vụn.
- Các chỉ dẫn văn phong trôi dạt qua các thế hệ model (một số model tự
  nhiên ngắn gọn/dài dòng hơn) — hiệu chỉnh lại khi di chuyển.

## Tác động dự kiến

- Chuyển sang structured output trên các route trích xuất/định tuyến:
  giảm output **3–10×**, cộng thêm độ tin cậy khi phân tích.
- Hợp đồng output trong system prompt trên các route chat/agent thường cắt
  giảm token output **30–60%** với thành công tác vụ tương đương (đo qua
  đánh giá).
- Trong các phiên agent dài, cùng mức cắt giảm đó cộng dồn qua lịch sử:
  giảm 40% output ≈ giảm khoảng 40% *input tương lai* từ các lượt đó nữa.

---

# Concise Output Prompting

**Addresses:** Cause 5.2 in [`../CAUSE.md`](../CAUSE.md) (with `diff-based-edits.md`)

**Idea:** Output is the most expensive token class *and* becomes input on
every later turn. Engineer the output contract — via prompts, structured
formats, verbosity parameters, and stop conditions — so the model delivers
the information, not the ceremony.

---

## How to apply

### 1. Give an explicit output contract in the system prompt

The reliable levers, in order of effect:

- **Lead-with-the-answer + selectivity framing**: *"Lead with the outcome.
  Keep output short by being selective about what you include — drop
  details that don't change what the reader does next — not by compressing
  the writing into fragments."* (Selectivity instructions beat "be concise",
  which models often satisfy by truncating style, not content.)
- **Ban the ceremony explicitly**: no preamble ("Certainly! Here is…"), no
  restating the question, no closing recap, no narrating routine actions in
  agent loops (*"default to silence between tool calls; one sentence when
  something changes"*).
- **Positive examples > prohibitions**: one or two examples of the desired
  answer shape outperform lists of don'ts.
- **Calibrate, don't minimize, where length is the product** — for reports
  and explanations, specify the *target shape* ("≤5 bullets, one line
  each") rather than a vague "short".

### 2. Use structured output as a length constraint

A JSON schema (provider structured-output modes) is the strongest verbosity
control available: the model can only emit the fields you defined. For
extraction/classification/routing routes, schema-constrained output
eliminates prose entirely — often cutting output 3–10× vs free text.

### 3. Use native verbosity/length parameters where offered

- OpenAI GPT-5 family: `verbosity: low|medium|high` — dials length without
  prompt surgery.
- Output caps (`max_tokens`) as a *backstop*, not a primary control —
  truncation is failure, not conciseness (`output-cap-sizing.md`).
- `stop` sequences to terminate known-terminal formats (e.g. end-of-JSON).

### 4. De-scaffold the prompts that *cause* verbosity

Forced progress summaries, "explain your reasoning" left over from
pre-reasoning-model days, mandatory section templates — audit and remove
what current models do adequately unprompted (`prompt-de-scaffolding.md`).

### 5. Watch the multi-turn echo

In agent loops, every verbose answer is re-billed as history each later
turn. Concision instructions therefore pay quadratically in long sessions —
prioritize them for your longest-running agents.

## SOTA tools

### Native — coding agents & provider APIs

| Provider / agent | Feature | Notes |
| --- | --- | --- |
| Anthropic API | Structured outputs (`output_config.format`) | Hard ceiling on output shape; the strongest lever for non-prose routes |
| Anthropic API | Token-efficient tool use (beta header) | Compacts the *tool-call* output format — up to ~70% fewer output tokens on tool-heavy turns, ~14% average; available on API/Bedrock/Vertex |
| OpenAI API | Structured outputs + `verbosity` parameter | Native length dial on supported models |
| Google Gemini API | `responseSchema` | Schema-constrained output |
| Claude Code | Output styles / `CLAUDE.md` conventions | Persistent output contract for the harness without per-prompt surgery |

### Third-party — agent-agnostic (open source preferred)

| Tool | License | Notes |
| --- | --- | --- |
| Caveman (`wilpel/caveman-compression`) | MIT | Semantic output compression — strips predictable grammar, keeps facts. Ships as a prompt skill usable across Claude Code, Codex, Gemini CLI, Cursor, Cline and other agents (skill variant claims up to ~75% output cut; the library's measured modes run 15–58%). Internal/agent traffic only — not for user-facing prose where tone matters |
| Instructor / Zod-typed outputs | MIT | Typed schemas that double as verbosity contracts, portable across providers |
| promptfoo / Langfuse evals | MIT | Regression-test that concision edits don't cut *content*; track output tokens per route; Braintrust is the commercial option |

## Trade-offs

- Over-tight contracts lose genuinely useful explanation — measure task
  success, not just token counts, when tightening.
- Aggressive terseness harms readability for human consumers; "selective,
  not compressed" is the framing that avoids fragment-speak.
- Style instructions drift across model generations (some models are
  naturally terser/wordier) — re-baseline on migration.

## Expected impact

- Structured-output conversion on extraction/routing routes: **3–10×**
  output reduction, plus parsing reliability.
- System-prompt output contracts on chat/agent routes typically cut output
  tokens **30–60%** at equal task success (measured via evals).
- In long agent sessions the same cut compounds through history: 40% less
  output ≈ 40% less *future input* from those turns as well.
