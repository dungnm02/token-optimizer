# Model routing — nghiên cứu nền cho phần "giải pháp"

> Ngày khảo sát: **2026-08-11**. Mọi con số dưới đây đều ghi rõ nguồn và mức tin cậy.
> Quy ước như phần còn lại của kho: **đo** = có phương pháp công bố; **hãng công bố** = nhà cung cấp tự báo;
> **ước tính** = suy ra từ giá niêm yết, chưa ai đo độc lập.

---

## 0. Vì sao routing thuộc về phần "giải pháp", không phải phần "công cụ"

Routing là **thiết lập cấu hình**, không phải phần mềm phải cài. Nó nằm cùng tầng với
`reasoning effort` và giới hạn output: không thêm điểm hỏng, không thêm token vào context,
và nó tác động thẳng vào **đơn giá** của mọi token còn lại trong phiên.

Nhưng nó có một ràng buộc mà các nút cấu hình khác không có:

> **Đổi model giữa phiên là phá cache.** Cache được khóa theo model + mức effort.
> Một lần đổi model là toàn bộ tiền tố (system prompt, rules, tool schema, lịch sử) trả lại giá đầy đủ.

Đây là lý do routing trong coding agent **khác hẳn** routing kiểu API gateway. Với gateway,
mỗi request độc lập nên route từng request là tối ưu. Với coding agent, phiên là một chuỗi dài
chia sẻ một tiền tố cache — route quá tay thì phần tiết kiệm đơn giá bị phần mất cache ăn hết.

**Hệ quả thiết kế:** các chiến lược routing tốt trong coding agent đều tránh đổi model của *luồng chính*.
Chúng hoặc chọn một lần ở đầu phiên, hoặc đẩy việc sang một **context riêng** (subagent / advisor)
để luồng chính giữ nguyên cache.

---

## 1. Bốn chiến lược, xếp theo mức phá cache

| Chiến lược | Model mạnh chạy khi nào | Ai quyết định | Phá cache luồng chính? |
|---|---|---|---|
| **Chọn một lần đầu phiên** | cả phiên | người dùng | Không (chọn xong để yên) |
| **Advisor / leo thang** | tại các điểm quyết định giữa phiên | model tự gọi | **Không** — advisor chạy server-side, tiền tố luồng chính giữ nguyên |
| **Subagent** | trọn một việc con | model uỷ thác, hoặc người gọi | Không — subagent có context riêng |
| **Planner / Executor (opusplan, Plan/Act)** | giai đoạn lập kế hoạch | chuyển chế độ | **Có** — mỗi lần bật/tắt là một lần đổi model |
| **Router tự động theo từng lượt** | tuỳ độ khó từng lượt | bộ phân loại | **Có, nhiều nhất** — xem §5 |

### 1.1 Advisor (leo thang)

Model chính rẻ chạy phần lớn công việc; khi gặp điểm quyết định — trước khi chốt hướng làm,
khi lỗi lặp lại, trước khi tuyên bố xong — nó **tự gọi** một model mạnh hơn để xin ý kiến.
Advisor nhận **toàn bộ** hội thoại và trả về hướng dẫn (thường **400–700 token**).

Số của Anthropic (*hãng công bố*, blog "The advisor strategy"):

| Cặp | Benchmark | Điểm | Chi phí |
|---|---|---|---|
| Sonnet 4.6 + advisor Opus **vs** Sonnet 4.6 một mình | SWE-bench Multilingual | **+2.7 điểm** | **−11.9%** mỗi tác vụ |
| Haiku 4.5 + advisor Opus **vs** Haiku 4.5 một mình | BrowseComp | 19.7% → **41.2%** | — |
| Haiku 4.5 + advisor Opus **vs** Sonnet một mình | BrowseComp | **−29%** điểm | **−85%** chi phí |

Điểm đáng chú ý về cache (từ tài liệu Claude Code, *tài liệu chính thức*):

- Bật/tắt advisor giữa phiên **không** làm mất cache của model chính.
- Nhưng **lượt đọc của advisor không được cache** — mỗi lần gọi, advisor đọc lại toàn bộ transcript từ đầu.
  Nghĩa là chi phí một lần gọi advisor tăng tuyến tính theo độ dài phiên. Phiên càng dài, mỗi lời khuyên càng đắt.

Ràng buộc: advisor phải **mạnh ngang hoặc hơn** model chính; chỉ chạy trên Anthropic API
(không có trên Bedrock / Vertex / Foundry); là tính năng **thử nghiệm**.

### 1.2 Subagent

Uỷ thác một việc con cho model rẻ trong **context riêng**, chỉ trả về bản tóm tắt.
Lợi kép: đơn giá rẻ hơn **và** rác khám phá không lọt vào phiên chính.
Đây là chiến lược duy nhất vừa giảm đơn giá vừa giảm *số* token của luồng chính.

Trên Claude Code, mỗi subagent khai báo `model:` riêng trong file định nghĩa; subagent
kế thừa advisor đã cấu hình và tự kiểm tra lại cặp model của chính nó.

### 1.3 Planner / Executor

Model mạnh lập kế hoạch, model rẻ thực thi. `opusplan` (Claude Code) và
"Use different models for Plan and Act" (Cline) là hai hiện thân.

**Cảnh báo đã có sẵn trong slide:** mỗi lần bật/tắt plan mode là một lần đổi model → cache dựng lại từ đầu.
Chiến lược này chỉ đáng khi giai đoạn kế hoạch đủ dài để bù phần cache mất đi — hợp với refactor lớn,
không hợp với việc sửa vài dòng.

Con số "giảm 50–70% ngay ngày đầu" từ Plan/Act trên Cline là **hướng dẫn của cộng đồng**, không có phương pháp đo công bố → coi là giả thuyết.

---

## 2. Bảng gợi ý: việc nào, model nào

Nguyên tắc: **chọn theo chi phí mỗi tác vụ hoàn thành, không theo giá mỗi triệu token.**
Hai model cùng giá/token có thể chênh nhau một bậc độ lớn về tiền kết thúc một việc,
vì chúng đốt số token khác nhau để tới đích (suy luận, retry, vòng lặp tool).

| Loại việc | Ưu tiên | Thay thế rẻ hơn | Vì sao |
|---|---|---|---|
| Tìm file, grep, đọc hiểu bố cục | model rẻ nhất còn dùng được: Haiku 4.5 · Gemini 3 Flash · DeepSeek V3.2 | — | Việc thi hành, không cần suy luận. Đây cũng là loại việc **chiếm 67–76% ngân sách** một phiên |
| Sửa lỗi nhỏ, đổi tên, sinh test | Haiku 4.5 · Gemini 3 Flash · GLM-4.6 · Qwen3-Coder | DeepSeek V3.2 | Khối lượng lớn, sai thì phát hiện ngay bằng test |
| Viết code theo kế hoạch đã chốt | Sonnet 5 · GPT-5.5 · Gemini 3 Pro | Kimi K2.6 (open-weight, 80.2% SWE-bench) | Cần đúng cú pháp và bám kế hoạch, không cần tự nghĩ hướng |
| Lập kế hoạch, chọn kiến trúc, refactor nhiều file | Opus 5 · GPT-5.5 (mức suy luận cao) | Sonnet 5 **+ advisor Opus** | Chỗ duy nhất trả tiền cho model đắt là xứng: sai ở đây kéo theo cả chuỗi sai |
| Gỡ lỗi khi đã kẹt 2–3 vòng | Opus 5, hoặc gọi advisor | — | Vòng xoáy retry đắt hơn nhiều so với một lần hỏi model mạnh |
| Review trước khi chốt | advisor / một model khác đọc lại | Sonnet 5 | Ý kiến độc lập rẻ hơn sửa sau khi merge |

### 2.1 Model cũ vẫn đáng dùng

Đây là điểm dễ bỏ qua nhất: **model đời trước thường là điểm ngọt về hiệu quả chi phí**,
vì giá rơi nhưng năng lực cho việc thường ngày gần như không đổi.

| Model | Ra đời | SWE-bench Verified | $/1M vào · ra | Chi phí / tác vụ |
|---|---|---|---|---|
| DeepSeek V3.2 | cũ | 67.8% | $0.28 · $0.42 | **$0.028** |
| Qwen3-Coder 480B | cũ | 69.6% | $0.22 · $1.00 | $0.033 |
| GLM-4.6 | cũ | 68.0% | $0.39 · $1.74 | $0.059 |
| Kimi K2.6 | cũ | **80.2%** | $0.60 · $2.50 | $0.075 |
| Gemini 3 Flash | — | 78.0% | $0.50 · $3.00 | $0.078 |
| Claude Haiku 4.5 | cũ (10/2025) | 73.3% | $1.00 · $5.00 | $0.150 |
| GPT-5.1 | cũ | 76.3% | $1.25 · $10.00 | $0.239 |
| Claude Opus 4.5 | cũ | 80.9% | $5.00 · $25.00 | **$0.680** |

Nguồn: SSOJet, khảo sát 6/2026, đối chiếu tài liệu nhà cung cấp ngày 2026-06-07.

**Phương pháp — và giới hạn của nó (quan trọng):** bảng này cố định ngân sách token
(50.000 vào / 12.000 ra cho mọi model) rồi chia cho tỉ lệ giải được đã công bố.
Nghĩa là nó **không** đo được sự khác nhau về *số token mỗi model thật sự đốt* —
đúng cái yếu tố mà nguồn khác nói là quyết định. Nên đọc bảng này như **thứ tự ưu tiên để thử**,
không phải như hoá đơn dự báo.

Điểm đọc ra được: **Kimi K2.6 đạt 80.2%, kém Opus 4.5 đúng 0.7 điểm, với chi phí bằng ~1/9.**
Và Haiku 4.5 — model đã một năm tuổi — vẫn là model rẻ nhất tính trên mỗi điểm trong nhóm đóng.

### 2.2 Chi phí mỗi tác vụ trên tác vụ agentic thật

Một khảo sát khác (UsageBox, 6/2026, dựa trên Artificial Analysis Coding Agent Index)
đo trên vòng lặp agent thật thay vì ngân sách token cố định:

| Model | Điểm Coding Agent Index | Chi phí / tác vụ |
|---|---|---|
| Cursor Composer 2.5 | 62 | **$0.07** |
| GPT-5.5 | 65 | $4.82 |
| Claude Opus 4.7 | 66 | $4.10 |

Chênh lệch **59 lần tiền cho 4 điểm** — đây là lập luận mạnh nhất cho việc routing,
mạnh hơn mọi công cụ nén trong bộ slide này.

---

## 3. Cách bật — thủ công

| Harness | Chọn model phiên | Advisor / leo thang | Planner / Executor | Model cho việc con |
|---|---|---|---|---|
| **Claude Code** | `/model`, cờ `--model` | `/advisor opus`, cờ `--advisor`, setting `advisorModel` | alias `opusplan` | trường `model:` trong file subagent |
| **Codex CLI** | `/model`, `--model` / `-m` | không có | qua profile | — |
| **Gemini CLI** | `--model`, đổi trong phiên | không có | không có | — |
| **Cline** | chọn trong Settings | không có | bật "Use different models for Plan and Act" | — |

Cấu hình bền (Claude Code):

```json
{ "advisorModel": "opus" }
```

Lưu ý vận hành: Gemini CLI **tự hạ** Pro → Flash khi vượt hạn mức. Đó là routing tự động
mà bạn không yêu cầu — và nó giải thích vì sao chất lượng đôi khi tụt giữa phiên mà không rõ lý do.

## 4. Cách bật — tự động

Ba mức, từ nhẹ tới nặng:

1. **Để model tự leo thang** (advisor). Không cần hạ tầng, không cần phân loại. Model chính tự quyết định khi nào cần ý kiến.
2. **Quy tắc tĩnh theo loại việc**: khai báo subagent cho từng loại việc (tìm kiếm → Haiku; viết code → Sonnet;
   kiến trúc → Opus). Tất định, đọc được, không phụ thuộc bộ phân loại nào.
3. **Router học được** (RouteLLM, LiteLLM, OpenRouter, gateway thương mại): một bộ phân loại
   đoán độ khó từng request rồi chọn model.

## 5. Cảnh báo về router tự động — đừng bê thẳng con số vào coding agent

RouteLLM (UC Berkeley / Anyscale / Canva, ICLR 2025) là nguồn học thuật tốt nhất hiện có:
giảm **>85% chi phí trên MT Bench** trong khi giữ **95% chất lượng GPT-4**.

Nhưng đọc kỹ phần còn lại của chính bài đó:

- **45%** trên MMLU, **35%** trên GSM8K — con số 85% là của benchmark **dễ định tuyến nhất**, không phải mức chung.
- Benchmark đều là **hội thoại một lượt**. Coding agent là **một tác vụ nhiều lượt** dùng chung một tiền tố cache.
- Router chọn model **theo từng request**. Trong coding agent, đổi model theo từng lượt là
  **phá cache mỗi lượt** — thứ mà bộ slide này đã đo là một trong những nguồn rò rỉ đắt nhất.

**Kết luận:** trong coding agent, hãy route theo **ranh giới việc** (phiên, việc con, điểm quyết định),
đừng route theo **từng lượt**. Đây chính là lý do advisor và subagent thắng router per-request:
chúng đưa model mạnh vào mà không đụng tới tiền tố của luồng chính.

---

## 6. Rút gọn cho slide

1. Routing là **nút chỉnh đơn giá** — cùng tầng với effort, không phải công cụ phải cài.
2. Ràng buộc chi phối tất cả: **đổi model = phá cache**. Nên route theo ranh giới việc, không theo từng lượt.
3. Ba cách đưa model mạnh vào mà **không** phá cache luồng chính: chọn một lần đầu phiên · subagent · advisor.
4. Số của advisor (hãng công bố): Sonnet + advisor Opus = **+2.7 điểm, −11.9% chi phí**; Haiku + advisor = **−85% chi phí** so với Sonnet, đổi lại −29% điểm.
5. Model cũ là điểm ngọt: **Kimi K2.6 kém Opus 4.5 đúng 0.7 điểm với ~1/9 chi phí**; Haiku 4.5 rẻ nhất mỗi điểm trong nhóm đóng.
6. **Chi phí mỗi tác vụ**, không phải giá mỗi triệu token: 59× tiền cho 4 điểm (Composer 2.5 vs Opus 4.7).
7. Cảnh báo: 85% của RouteLLM là MT Bench một lượt — trên MMLU chỉ 45%, và nó chưa từng được đo trên vòng lặp coding agent.

---

## Nguồn

- Claude Code — Escalate hard decisions with the advisor tool: https://code.claude.com/docs/en/advisor
- Anthropic — The advisor strategy: https://claude.com/blog/the-advisor-strategy
- Claude Code — Model configuration: https://code.claude.com/docs/en/model-config
- Claude Code — Create custom subagents: https://code.claude.com/docs/en/sub-agents
- Cline — Plan & Act mode: https://docs.cline.bot/core-workflows/plan-and-act
- OpenAI — Codex models: https://developers.openai.com/codex/models
- SSOJet — 8 AI coding models ranked by cost-per-task (6/2026): https://ssojet.com/blog/cheapest-ai-coding-models
- UsageBox — Cost per task is the new AI benchmark (6/2026): https://usagebox.com/articles/cost-per-task-workhorse-models-2026
- LMSYS — RouteLLM: https://www.lmsys.org/blog/2024-07-01-routellm/ · bài báo: https://arxiv.org/pdf/2406.18665
