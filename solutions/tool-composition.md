# Kết hợp Tool (Điều phối thực thi bằng code) (Tiếng Việt)

**Giải quyết:** Nguyên nhân 3.2 trong [`../CAUSE.md`](../CAUSE.md)

**Ý tưởng:** Thay các chuỗi round-trip tool phải qua trung gian của model
bằng một bước duy nhất, trong đó model **viết code để kết hợp các tool**.
Đoạn code này chạy trong sandbox, kết quả trung gian chảy thẳng giữa các
lệnh gọi tool bên trong runtime, và chỉ câu trả lời cô đọng cuối cùng mới
quay lại context của model.

---

## Tại sao nó thắng

```mermaid
sequenceDiagram
    participant M as Model
    participant T as Tool
    rect rgb(255, 235, 235)
    Note over M,T: ❌ Round-trip — 3 lượt xử lý context đầy đủ
    M->>T: get_profile(user)
    T-->>M: profile 4K token → vào context
    M->>T: get_orders(profile.id)
    T-->>M: danh sách đơn hàng 12K token → vào context
    M->>T: check_inventory(order_ids)
    T-->>M: kết quả 6K token → vào context
    end
    rect rgb(235, 255, 235)
    Note over M,T: ✅ Kết hợp — 1 lượt, dữ liệu trung gian không bao giờ bị tính phí
    M->>T: run_code("p=get_profile(u); o=get_orders(p.id);<br/>print(summarize(check_inventory(o)))")
    T-->>M: câu trả lời cuối 300 token
    end
```

Mỗi round-trip gửi lại toàn bộ lịch sử (đang lớn dần); kết hợp chỉ gửi
một lần. Payload trung gian (thường là phần lớn dữ liệu) không bao giờ vào
context — chúng sinh ra và mất đi bên trong sandbox.

## Cách áp dụng

1. **Gọi tool theo chương trình có sẵn từ nhà cung cấp** — *Anthropic
   PTC*: khai báo tool thực thi code và đánh dấu các tool tùy chỉnh của bạn
   với `allowed_callers: ["code_execution_20260120"]`; Claude viết một
   script gọi các tool của bạn như các hàm bên trong sandbox, và chỉ stdout
   của script quay lại context. *OpenAI*: code interpreter + kết hợp
   function-calling trên Responses API.
2. **Framework agent code-as-action** — **smolagents** của Hugging Face
   (`CodeAgent`) biến code thành định dạng hành động *mặc định*: thay vì
   phát một lệnh gọi tool JSON mỗi bước, model phát ra một đoạn Python có
   thể lặp, rẽ nhánh, và gọi nhiều tool. Dòng nghiên cứu CodeAct đứng sau
   điều này báo cáo giảm tới ~30% số bước so với agent gọi-tool-JSON trên
   các benchmark nhiều tool — ít bước hơn = ít lượt xử lý context đầy đủ
   hơn.
3. **Kết hợp trước một cách tất định** — nếu chuỗi *luôn luôn* giống nhau
   (profile → đơn hàng → tồn kho), đó hoàn toàn không phải là một quyết
   định của LLM: hợp nhất nó thành một tool (`get_user_order_status`)
   trong harness. Token rẻ nhất là token model không bao giờ phải điều
   phối.
4. **Gộp các lệnh gọi độc lập trong một lượt** — khi chưa thể kết hợp, ít
   nhất hãy tận dụng khả năng dùng tool song song: N lệnh gọi độc lập
   trong một lượt assistant, cộng toàn bộ kết quả trong một lượt user, chỉ
   tính là một lượt xử lý context thay vì N.
5. **Lọc bên trong sandbox** — script kết hợp nên kết thúc bằng một bước
   `summarize`/`select` để giá trị trả về là câu trả lời, không phải dữ
   liệu thô (kết hợp với `tool-output-budgets.md`).

## Công cụ hiện đại nhất (SOTA)

### Có sẵn — coding agent & API của nhà cung cấp

| Nhà cung cấp / agent | Tính năng | Ghi chú |
| --- | --- | --- |
| Anthropic API | Gọi tool theo chương trình (`code_execution` + `allowed_callers`) | Dữ liệu trung gian quay lại code đang chạy, không vào context; chi phí token tỷ lệ với output cuối cùng |
| OpenAI API · Codex | Code interpreter + gọi hàm trên Responses API | Kết hợp phía sandbox |
| Claude Code / Codex CLI / Gemini CLI | Kết hợp có sẵn qua shell (pipeline `Bash`, script) | Các coding agent đã kết hợp qua shell — pipe/lọc trong bash thay vì round-trip với output thô |

### Bên thứ ba — không phụ thuộc agent (ưu tiên mã nguồn mở)

| Công cụ | Giấy phép | Ghi chú |
| --- | --- | --- |
| Hugging Face smolagents `CodeAgent` | Apache-2.0 | Code-as-action là định dạng hành động *mặc định*; báo cáo giảm ~30% số bước so với gọi-tool-JSON |
| LangGraph | MIT | Biểu diễn các chuỗi tất định thành cạnh của đồ thị (code), dành model cho các quyết định thực sự |
| E2B | Apache-2.0 | Sandbox tự host cho các agent code-as-action; Modal / Daytona là các lựa chọn thương mại/host sẵn thay thế |

## Đánh đổi

- Đòi hỏi một sandbox — hạ tầng, rà soát bảo mật, và (với các lựa chọn
  host sẵn) chi phí thực thi.
- Bản thân việc viết code tốn token output; với một lệnh gọi đơn lẻ tầm
  thường, dùng tool trực tiếp rẻ hơn. Kết hợp đáng giá từ ~3 lệnh gọi nối
  chuỗi trở lên hoặc bất kỳ dữ liệu trung gian lớn nào.
- Các thất bại khó kiểm tra hơn: một lỗi bên trong script kết hợp cần
  hiển thị tốt cả script lẫn traceback, nếu không model sẽ phải tốn nhiều
  lượt gỡ lỗi trong mù mờ.
- PTC có sẵn từ nhà cung cấp hạn chế một số tính năng (ví dụ không tương
  thích với schema chặt/buộc chọn tool trên một số stack) — kiểm tra ma
  trận hiện tại.

## Tác động dự kiến

- Chi phí token của một chuỗi K bước giảm từ **K lượt xử lý context đầy đủ
  xuống ~1**; với context 100K token và K=5, đó là ~500K token input →
  ~100K.
- Kết quả trung gian (thường lớn gấp 10–100× câu trả lời cuối cùng) được
  loại bỏ khỏi context *vĩnh viễn* — cộng dồn với việc lịch sử tồn tại
  lâu.
- Giảm số bước (~30% trong các đánh giá kiểu CodeAct) cũng cắt giảm độ trễ
  và bề mặt lỗi, không chỉ token.

---

# Tool Composition (Code-Executed Orchestration)

**Addresses:** Cause 3.2 in [`../CAUSE.md`](../CAUSE.md)

**Idea:** Replace chains of model-mediated tool round-trips with a single
step in which the model **writes code that composes the tools**; the code
runs in a sandbox, intermediate results flow between tool calls inside the
runtime, and only the final distilled answer returns to the model's context.

---

## Why it wins

```mermaid
sequenceDiagram
    participant M as Model
    participant T as Tools
    rect rgb(255, 235, 235)
    Note over M,T: ❌ Round-trip style — 3 full context passes
    M->>T: get_profile(user)
    T-->>M: 4K-token profile → lands in context
    M->>T: get_orders(profile.id)
    T-->>M: 12K-token order list → lands in context
    M->>T: check_inventory(order_ids)
    T-->>M: 6K-token result → lands in context
    end
    rect rgb(235, 255, 235)
    Note over M,T: ✅ Composed — 1 pass, intermediates never billed
    M->>T: run_code("p=get_profile(u); o=get_orders(p.id);<br/>print(summarize(check_inventory(o)))")
    T-->>M: 300-token final answer
    end
```

Every round-trip re-sends the entire (growing) history; composition sends it
once. Intermediate payloads (often the bulk of the data) never enter the
context at all — they live and die inside the sandbox.

## How to apply

1. **Provider-native programmatic tool calling** — *Anthropic PTC*: declare
   the code-execution tool and mark your custom tools with
   `allowed_callers: ["code_execution_20260120"]`; Claude writes a script
   that calls your tools as functions inside the sandbox, and only the
   script's stdout returns to context. *OpenAI*: code interpreter +
   function-calling composition on the Responses API.
2. **Code-as-action agent frameworks** — Hugging Face **smolagents**
   (`CodeAgent`) makes code the *default* action format: instead of emitting
   one JSON tool call per step, the model emits a Python snippet that can
   loop, branch, and call several tools. The CodeAct research line behind
   this reports up to ~30% fewer steps than JSON-tool-call agents on
   multi-tool benchmarks — fewer steps = fewer full-context passes.
3. **Deterministic pre-composition** — if the chain is *always* the same
   (profile → orders → inventory), it isn't an LLM decision at all: fuse it
   into one tool (`get_user_order_status`) in the harness. The cheapest
   token is the one the model never orchestrates.
4. **Batch independent calls in one turn** — when composition isn't
   available, at minimum exploit parallel tool use: N independent calls in
   one assistant turn + all results in one user turn is one context pass
   instead of N.
5. **Filter inside the sandbox** — the composed script should end with a
   `summarize`/`select` step so the return value is the answer, not the raw
   data (pair with `tool-output-budgets.md`).

## SOTA tools

### Native — coding agents & provider APIs

| Provider / agent | Feature | Notes |
| --- | --- | --- |
| Anthropic API | Programmatic tool calling (`code_execution` + `allowed_callers`) | Intermediates return to the running code, not to context; token cost scales with final output |
| OpenAI API · Codex | Code interpreter + function calling on the Responses API | Sandbox-side composition |
| Claude Code / Codex CLI / Gemini CLI | Shell-native composition (`Bash` pipelines, scripts) | Coding agents already compose via the shell — pipe/filter in bash instead of round-tripping raw output |

### Third-party — agent-agnostic (open source preferred)

| Tool | License | Notes |
| --- | --- | --- |
| Hugging Face smolagents `CodeAgent` | Apache-2.0 | Code-as-action as the *default* action format; ~30% fewer steps reported vs JSON tool-calling |
| LangGraph | MIT | Express deterministic chains as graph edges (code), reserving the model for genuine decisions |
| E2B | Apache-2.0 | Self-hostable sandbox for code-as-action agents; Modal / Daytona are commercial/hosted alternatives |

## Trade-offs

- Requires a sandbox — infrastructure, security review, and (for hosted
  options) execution pricing.
- Code-writing itself costs output tokens; for a single trivial call, plain
  tool use is cheaper. Composition pays off from ~3 chained calls or any
  large intermediate.
- Harder-to-inspect failures: an error inside the composed script needs the
  script + traceback surfaced well, or the model burns turns debugging
  blind.
- Provider-native PTC restricts some features (e.g. incompatible with strict
  schemas / forced tool choice on some stacks) — check the current matrix.

## Expected impact

- Token cost of a K-step chain drops from **K full context passes to ~1**;
  with a 100K-token context and K=5, that's ~500K input tokens → ~100K.
- Intermediate results (often 10–100× the size of the final answer) are
  removed from context *permanently* — compounding with history persistence.
- Step-count reductions (~30% in CodeAct-style evaluations) also cut
  latency and error surface, not just tokens.
