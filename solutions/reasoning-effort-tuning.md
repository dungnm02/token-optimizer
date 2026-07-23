# Tinh chỉnh Effort Reasoning (Ngân sách Thinking theo từng Route) (Tiếng Việt)

**Giải quyết:** Nguyên nhân 5.1 trong [`../CAUSE.md`](../CAUSE.md)

**Ý tưởng:** Token reasoning được tính phí như output (loại đắt nhất) dù có
được hiển thị hay không. Đặt độ sâu reasoning **theo từng route** — sâu
cho công việc thực sự khó, nông hoặc tắt cho lưu lượng thường lệ — thay vì
một cài đặt toàn cục duy nhất.

---

## Các nút điều chỉnh của nhà cung cấp

| Nhà cung cấp | Nút điều chỉnh | Giá trị / ghi chú |
| --- | --- | --- |
| Anthropic | `output_config.effort` (+ adaptive thinking) | `low`/`medium`/`high`/`xhigh`/`max`; adaptive thinking quyết định *khi nào* suy nghĩ, effort quyết định *bao nhiêu*. Lưu ý: trên một số model hiện tại thinking bật **theo mặc định** — chi phí xuất hiện mà không cần bật |
| OpenAI | `reasoning_effort` (dòng o-series / GPT-5) | `minimal`/`low`/`medium`/`high`; token reasoning được báo cáo trong `usage.completion_tokens_details.reasoning_tokens` |
| Gemini | `thinking_budget` / `thinkingConfig` | Ngân sách theo số token bao gồm `0` (tắt, nơi hỗ trợ) và động |
| Model mở (kiểu R1) | Kiểm soát prompt/định dạng, ép ngân sách | Giới hạn độ dài suy nghĩ ở cấp serving |

## Cách áp dụng

1. **Kiểm kê các route theo nhu cầu reasoning.**

   | Loại route | Cài đặt |
   | --- | --- |
   | Phân loại, trích xuất, định dạng, tra cứu đơn giản | Tắt reasoning / `minimal` / `low` |
   | Chat tiêu chuẩn, tóm tắt | `low`–`medium` |
   | Coding, công việc agentic nhiều bước | `high` (quét `xhigh` trên bộ đánh giá) |
   | Quan trọng về tính đúng đắn, bài toán khó nhất | `high`+`xhigh`/`max` — có đo lường, không phải theo phản xạ |

2. **Quét, đừng đoán.** Chạy bộ đánh giá của bạn qua 2–3 mức effort và vẽ
   đồ thị chất lượng so với token output. Đường cong không đơn điệu theo
   chi phí: *effort cao hơn ngay từ đầu thường giảm tổng chi phí agentic*
   bằng cách hoàn thành trong ít lượt/lệnh gọi tool hơn — đánh giá bằng
   **chi phí mỗi tác vụ hoàn thành** (`token-counting.md`), không bao giờ
   chỉ dựa vào token output mỗi request.
3. **Định tuyến động cho lưu lượng hỗn hợp.** Một bộ phân loại độ phức tạp
   rẻ (hoặc router theo tier model từ `model-routing.md`) gán effort theo
   từng request; các truy vấn đơn giản đi theo đường nông.
4. **Dùng prompt để điều khiển khi nào thinking được kích hoạt**, ở những
   nơi nút điều chỉnh chỉ ở mức thô: các model adaptive-thinking chấp nhận
   hướng dẫn kiểu *"thinking làm tăng độ trễ và chỉ nên dùng khi nó cải
   thiện đáng kể chất lượng câu trả lời — khi không chắc chắn, hãy trả lời
   trực tiếp."*
5. **Để lại khoảng trống.** Ở effort cao, đặt giới hạn output rộng rãi —
   reasoning chia sẻ ngân sách output, và một giới hạn chặt sẽ cho ra câu
   trả lời bị cắt bớt sau khi đã tốn kém cho thinking (trường hợp tệ nhất
   của nguyên nhân 5.3).
6. **Theo dõi khoảng cách reasoning ẩn** trong đo lường: `output_tokens −
   visible_response_tokens` theo từng route chính là mức chi tiêu cho
   reasoning; hãy cảnh báo khi con số này tăng dần theo thời gian.

## Công cụ hiện đại nhất (SOTA)

### Có sẵn — coding agent & API của nhà cung cấp

| Nhà cung cấp / agent | Tính năng | Ghi chú |
| --- | --- | --- |
| Anthropic API · Claude Code | `output_config.effort` + adaptive thinking; chọn effort theo từng model trong harness | Effort quyết định *bao nhiêu*, adaptive thinking quyết định *khi nào* |
| OpenAI API · Codex CLI | Tham số `reasoning_effort`; `model_reasoning_effort` trong cấu hình Codex | Token reasoning được báo cáo trong `usage.completion_tokens_details` |
| Google Gemini API · Gemini CLI | `thinkingConfig` / `thinking_budget` | Ngân sách theo số token bao gồm `0` (tắt, nơi hỗ trợ) và động |

### Bên thứ ba — không phụ thuộc agent (ưu tiên mã nguồn mở)

| Công cụ | Giấy phép | Ghi chú |
| --- | --- | --- |
| LiteLLM | MIT | Chuẩn hóa các nút effort/ngân sách qua các nhà cung cấp để một cấu hình route→effort điều khiển mọi backend |
| promptfoo / harness đánh giá kiểu RAGAS | MIT / Apache-2.0 | Chạy các phép quét chất lượng-so-với-effort để bản đồ có bằng chứng |
| Langfuse / Helicone | MIT / Apache-2.0 | Theo dõi khoảng cách reasoning ẩn (`output − hiển thị`) theo từng route và cảnh báo khi leo thang |

## Đánh đổi

- Cấp thiếu reasoning cho các tác vụ khó sẽ tốn kém hơn số nó tiết kiệm
  được: câu trả lời sai, phải sửa thêm nhiều lượt, dùng tool hời hợt. Chỉ
  nên cắt effort ở những nơi đánh giá xác nhận chất lượng vẫn giữ vững.
- Cấu hình theo từng route là bề mặt cần bảo trì nhiều hơn; giữ bản đồ
  route → effort trong config, có phiên bản kèm đánh giá.
- Ngữ nghĩa giữa các nhà cung cấp khác nhau (ngân sách theo token, theo
  mức effort, hay luôn bật) — bản đồ route → effort cần tinh chỉnh lại mỗi
  khi đổi nhà cung cấp/model.

## Tác động dự kiến

- Token reasoning thường chiếm **30–70% chi tiêu output** trên các triển
  khai có bật reasoning; tắt hoặc giảm chúng trên các route thường lệ sẽ
  loại bỏ phần lớn tỷ trọng đó mà chất lượng vẫn không đổi.
- Hướng dẫn của OpenAI/Anthropic và các benchmark cộng đồng nhất quán cho
  thấy effort thấp so với cao chênh lệch **2–10× token output** trên cùng
  prompt — gán theo từng route nắm bắt được sự chênh lệch đó.
- Trên các tác vụ agentic thực sự khó, *tăng* effort có thể cắt giảm
  **tổng** chi phí (ít lượt hơn, ít thử lại hơn) — phép quét tìm ra những
  điểm đảo ngược này.

---

# Reasoning Effort Tuning (Per-Route Thinking Budgets)

**Addresses:** Cause 5.1 in [`../CAUSE.md`](../CAUSE.md)

**Idea:** Reasoning tokens are billed as output (the most expensive class)
whether or not they're displayed. Set the reasoning depth **per route** —
deep for genuinely hard work, shallow or off for routine traffic — instead
of one global setting.

---

## The provider dials

| Provider | Dial | Values / notes |
| --- | --- | --- |
| Anthropic | `output_config.effort` (+ adaptive thinking) | `low`/`medium`/`high`/`xhigh`/`max`; adaptive thinking decides *when* to think, effort decides *how much*. Note: on some current models thinking is on **by default** — cost appears without opt-in |
| OpenAI | `reasoning_effort` (o-series / GPT-5 family) | `minimal`/`low`/`medium`/`high`; reasoning tokens reported in `usage.completion_tokens_details.reasoning_tokens` |
| Gemini | `thinking_budget` / `thinkingConfig` | Token-count budget incl. `0` (off, where supported) and dynamic |
| Open models (R1-style) | Prompt/format control, budget forcing | Serving-level caps on thought length |

## How to apply

1. **Inventory routes by reasoning need.**

   | Route type | Setting |
   | --- | --- |
   | Classification, extraction, formatting, simple lookups | Reasoning off / `minimal` / `low` |
   | Standard chat, summarization | `low`–`medium` |
   | Coding, multi-step agentic work | `high` (sweep `xhigh` on evals) |
   | Correctness-critical, hardest problems | `high`+`xhigh`/`max` — measured, not reflexive |

2. **Sweep, don't guess.** Run your eval set across 2–3 effort levels and
   plot quality vs output tokens. The curve is not monotonic in cost:
   *higher effort up front often reduces total agentic cost* by finishing in
   fewer turns/tool calls — judge by **cost per completed task**
   (`token-counting.md`), never per-request output tokens alone.
3. **Route dynamically for mixed traffic.** A cheap complexity classifier
   (or the model-tier router from `model-routing.md`) assigns effort per
   request; simple queries take the shallow path.
4. **Steer thinking triggering with prompts** where the dial is coarse:
   adaptive-thinking models accept guidance like *"thinking adds latency
   and should only be used when it meaningfully improves answer quality —
   when in doubt, respond directly."*
5. **Leave headroom.** At high effort, size the output cap generously —
   reasoning shares the output budget, and a tight cap yields a truncated
   answer after expensive thinking (cause 5.3's worst case).
6. **Watch the hidden-reasoning gap** in telemetry: `output_tokens −
   visible_response_tokens` per route is the reasoning spend; alert on
   creep.

## SOTA tools

### Native — coding agents & provider APIs

| Provider / agent | Feature | Notes |
| --- | --- | --- |
| Anthropic API · Claude Code | `output_config.effort` + adaptive thinking; per-model effort selection in the harness | Effort decides *how much*, adaptive thinking decides *when* |
| OpenAI API · Codex CLI | `reasoning_effort` param; `model_reasoning_effort` in Codex config | Reasoning tokens reported in `usage.completion_tokens_details` |
| Google Gemini API · Gemini CLI | `thinkingConfig` / `thinking_budget` | Token-count budget incl. `0` (off, where supported) and dynamic |

### Third-party — agent-agnostic (open source preferred)

| Tool | License | Notes |
| --- | --- | --- |
| LiteLLM | MIT | Normalizes the effort/budget dials across providers so one route→effort config drives any backend |
| promptfoo / RAGAS-style eval harnesses | MIT / Apache-2.0 | Run the quality-vs-effort sweeps that make the map evidence-based |
| Langfuse / Helicone | MIT / Apache-2.0 | Track the hidden-reasoning gap (`output − visible`) per route and alert on creep |

## Trade-offs

- Under-provisioned reasoning on hard tasks costs more than it saves:
  wrong answers, extra correction turns, shallow tool use. Cut effort only
  where evals confirm quality holds.
- Per-route configuration is more surface to maintain; keep the route →
  effort map in config, versioned with evals.
- Provider semantics differ (a budget vs a level vs always-on) — the map
  needs re-tuning per provider/model migration.

## Expected impact

- Reasoning tokens are commonly **30–70% of output spend** on
  reasoning-enabled deployments; turning them off/down on routine routes
  removes most of that share for those routes at flat quality.
- OpenAI/Anthropic guidance and community benchmarks consistently show
  low-vs-high effort differing by **2–10× output tokens** on the same
  prompts — per-route assignment captures the spread.
- On genuinely hard agentic tasks, *raising* effort can cut **total** cost
  (fewer turns, fewer retries) — the sweep finds these inversions.
