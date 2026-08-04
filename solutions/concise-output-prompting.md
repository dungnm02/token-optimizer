# Prompt Output Ngắn gọn (Tiếng Việt)

**Giải quyết:** Nguyên nhân 5.2 trong [`../CAUSE.md`](../CAUSE.md) (cùng với
`diff-based-edits.md`)

**Ý tưởng:** Output là loại token đắt nhất, và nó còn trở thành input ở mọi
lượt sau đó. Hãy thiết kế một hợp đồng output rõ ràng — qua prompt, định
dạng có cấu trúc, tham số điều chỉnh độ dài, và điều kiện dừng — để model
trả về đúng thông tin cần thiết, chứ không phải những câu chữ rườm rà.

---

## Cách áp dụng

### 1. Đưa ra một hợp đồng output tường minh trong system prompt

Các đòn bẩy đáng tin cậy nhất, xếp theo mức độ hiệu quả:

- **Dẫn dắt bằng câu trả lời, kèm nguyên tắc chọn lọc**: *"Hãy mở đầu bằng
  kết quả. Giữ output ngắn gọn bằng cách chọn lọc nội dung đưa vào — bỏ
  những chi tiết không làm thay đổi việc người đọc làm tiếp theo — chứ
  không phải bằng cách nén câu chữ thành các mảnh vụn."* (Hướng dẫn chọn
  lọc kiểu này hiệu quả hơn nhiều so với câu "hãy ngắn gọn" chung chung,
  vì các model thường thỏa mãn yêu cầu đó bằng cách cắt bớt văn phong chứ
  không cắt nội dung.)
- **Cấm nghi thức rườm rà một cách tường minh**: không câu mở đầu ("Chắc
  chắn rồi! Đây là…"), không nhắc lại câu hỏi, không tóm tắt ở cuối, không
  tường thuật lại các hành động thường lệ trong vòng lặp agent (*"mặc định
  im lặng giữa các lệnh gọi tool; chỉ nói một câu khi có gì đó thay đổi"*).
- **Ví dụ minh họa hiệu quả hơn liệt kê điều cấm**: đưa ra một hoặc hai ví
  dụ về hình dạng câu trả lời mong muốn sẽ hiệu quả hơn nhiều so với một
  danh sách những điều không nên làm.
- **Hãy hiệu chỉnh, đừng cố tối thiểu hóa, khi độ dài chính là một phần
  của sản phẩm** — với báo cáo và các bài giải thích, hãy nêu rõ *hình
  dạng mục tiêu* (ví dụ "tối đa 5 gạch đầu dòng, mỗi dòng một ý") thay vì
  chỉ nói mơ hồ rằng "hãy ngắn gọn".

### 2. Dùng structured output như một ràng buộc về độ dài

Một schema JSON (dùng chế độ structured-output của nhà cung cấp) là công
cụ kiểm soát độ dài mạnh nhất hiện có: model chỉ có thể trả về đúng các
trường bạn đã định nghĩa. Với các route trích xuất/phân loại/định tuyến,
output ràng buộc bởi schema loại bỏ hoàn toàn phần văn xuôi thừa — thường
cắt giảm output từ 3 đến 10 lần so với văn bản tự do.

### 3. Dùng tham số điều chỉnh độ dài/độ chi tiết có sẵn khi được cung cấp

- Dòng GPT-5 của OpenAI: `verbosity: low|medium|high` — điều chỉnh độ dài
  mà không cần can thiệp vào prompt.
- Giới hạn output (`max_tokens`) chỉ nên dùng như một *lưới an toàn dự
  phòng*, không phải công cụ kiểm soát chính — bị cắt bớt là một thất bại,
  chứ không phải là sự ngắn gọn (`output-cap-sizing.md`).
- Chuỗi `stop` để kết thúc các định dạng đã biết trước điểm dừng (ví dụ
  điểm kết thúc của JSON).

### 4. Gỡ bỏ những phần scaffolding gây ra sự dài dòng

Các bản tóm tắt tiến độ bắt buộc, câu "hãy giải thích lý luận của bạn" còn
sót lại từ thời trước khi có model reasoning, hay các mẫu section bắt
buộc — hãy rà soát và loại bỏ những gì model hiện tại đã tự làm tốt mà
không cần được yêu cầu (`prompt-de-scaffolding.md`).

### 5. Chú ý hiệu ứng lặp lại qua nhiều lượt

Trong các vòng lặp agent, mỗi câu trả lời dài dòng sẽ bị tính phí lại như
một phần của lịch sử ở tất cả các lượt sau đó. Vì vậy, các chỉ dẫn giúp
ngắn gọn sẽ mang lại lợi ích tăng theo cấp số nhân trong các phiên dài —
nên hãy ưu tiên áp dụng chúng cho những agent chạy dài nhất của bạn.

## Công cụ hiện đại nhất (SOTA)

### Có sẵn — coding agent & API của nhà cung cấp

| Nhà cung cấp / agent | Tính năng | Ghi chú |
| --- | --- | --- |
| Anthropic API | Structured output (`output_config.format`) | Giới hạn cứng về hình dạng output; đòn bẩy mạnh nhất cho các route không dùng văn xuôi |
| Anthropic API | Token-efficient tool use (header beta) | Nén định dạng output *của lệnh gọi tool* — giảm tới ~70% token output ở các lượt dùng nhiều tool, trung bình khoảng ~14%; có trên API/Bedrock/Vertex |
| OpenAI API | Structured output + tham số `verbosity` | Nút điều chỉnh độ dài có sẵn trên các model hỗ trợ |
| Google Gemini API | `responseSchema` | Output ràng buộc bởi schema |
| Claude Code | Output style / quy ước trong `CLAUDE.md` | Hợp đồng output bền vững cho harness mà không cần chỉnh sửa từng prompt |

### Bên thứ ba — không phụ thuộc agent (ưu tiên mã nguồn mở)

| Công cụ | Giấy phép | Ghi chú |
| --- | --- | --- |
| Caveman — **skill** ("caveman mode") | MIT | Prompt skill ép agent viết cộc lốc, dùng được trên Claude Code, Codex, Gemini CLI, Cursor, Cline. Quảng cáo 65%; A/B độc lập trên Claude Code đo được **8.5%** — và đó là **trần** (skill bị ép kích hoạt), kèm rủi ro đuôi nặng: một tác vụ vọt lên $8.29 so với $0.33 ở nhánh đối chứng. Xem [`../PROOF.md`](../PROOF.md) |
| Caveman — **thư viện** (`wilpel/caveman-compression`) | MIT | ⚠️ **Không phải cùng một thứ với skill ở trên.** Đây là thư viện Python nén **văn bản bạn tự đưa vào** (system prompt, tài liệu, chunk RAG): LLM 40–58%, MLM 20–30%, NLP 15–30%. README của nó **không ghi nhận tích hợp với bất kỳ agent framework nào** — nên nó thuộc nguyên nhân 6.4/4.2, không phải 5.2. Chính họ khuyến cáo tránh dùng cho nội dung người dùng đọc, marketing, văn bản pháp lý |
| Instructor / output có kiểu Zod | MIT | Schema có kiểu dữ liệu, đồng thời đóng vai trò như một hợp đồng về độ dài, dùng được trên nhiều nhà cung cấp |
| Đánh giá promptfoo / Langfuse | MIT | Kiểm thử hồi quy để đảm bảo các chỉnh sửa cho ngắn gọn không cắt mất *nội dung*; theo dõi số token output theo từng route; Braintrust là lựa chọn thương mại |

## Đánh đổi

- Hợp đồng quá chặt có thể làm mất đi những giải thích thực sự hữu ích —
  khi thắt chặt, hãy đo bằng mức độ thành công của tác vụ chứ không chỉ
  dựa vào số token.
- Sự ngắn gọn quá mức làm hại đến khả năng đọc của người dùng; nguyên tắc
  "chọn lọc, chứ không nén" chính là cách đóng khung giúp tránh lối viết
  rời rạc, thiếu mạch lạc.
- Các chỉ dẫn về văn phong có thể trôi dạt qua từng thế hệ model (một số
  model vốn đã ngắn gọn hoặc dài dòng hơn) — nên hiệu chỉnh lại mỗi khi di
  chuyển sang model mới.

## Tác động dự kiến

- Chuyển sang dùng structured output trên các route trích xuất/định
  tuyến: giảm output **3–10 lần**, đồng thời tăng thêm độ tin cậy khi phân
  tích kết quả.
- Hợp đồng output đặt trong system prompt trên các route chat/agent
  thường cắt giảm token output **30–60%** trong khi mức độ thành công của
  tác vụ vẫn tương đương (đo qua các bài đánh giá).
- Trong các phiên agent dài, cùng một mức cắt giảm đó sẽ cộng dồn qua cả
  lịch sử: giảm 40% output đồng nghĩa với giảm khoảng 40% *input trong
  tương lai* phát sinh từ chính các lượt đó.

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
| Caveman — the **skill** ("caveman mode") | MIT | Prompt skill making the agent terse, usable across Claude Code, Codex, Gemini CLI, Cursor, Cline. Advertised 65%; an independent A/B on Claude Code measured **8.5%** — and that is the **ceiling** (the skill was force-activated), with severe tail risk: one task spiked to $8.29 against a $0.33 baseline. See [`../PROOF.md`](../PROOF.md) |
| Caveman — the **library** (`wilpel/caveman-compression`) | MIT | ⚠️ **Not the same artifact as the skill above.** A Python library that compresses **text you pass in** (system prompts, docs, RAG chunks): LLM 40–58%, MLM 20–30%, NLP 15–30%. Its README **documents no integration with any agent framework** — so it belongs to causes 6.4/4.2, not 5.2. Its own caveats: avoid user-facing content, marketing copy, legal text |
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
