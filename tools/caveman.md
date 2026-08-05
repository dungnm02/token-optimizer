# Caveman (Tiếng Việt)

**Là gì:** hai thứ **khác nhau** cùng mang tên "caveman" — một *skill* prompt
ép agent viết cộc lốc, và một *thư viện* Python nén văn bản bạn tự đưa vào.
Chúng bị nhầm lẫn với nhau ở khắp nơi, kể cả trong kho này trước đây.

**Giải quyết:** skill → nguyên nhân 5.2; thư viện → nguyên nhân 6.4 và 4.2
trong [`../CAUSE.md`](../CAUSE.md). Xem
[`../solutions/concise-output-prompting.md`](../solutions/concise-output-prompting.md)

**Bằng chứng:** 🟢 Tier A cho skill — quảng cáo **65%**, đo được **8.5%**, và
đó là **trần** chứ không phải mức thường gặp. Thư viện chưa có A/B độc lập.
Xem [`../PROOF.md`](../PROOF.md)

---

## Trước hết: phân biệt hai thứ

| | Thư viện (`wilpel/caveman-compression`) | Skill ("caveman mode") |
| --- | --- | --- |
| Là gì | Thư viện Python, MIT, 3 backend | Một prompt/ruleset ép agent viết cộc lốc |
| Nén cái gì | Văn bản **bạn** đưa vào | Output **model** viết ra |
| Quảng cáo | 15–58% tùy backend | 65% |
| Đã được A/B độc lập? | ❌ Chưa | ✅ Rồi — kết quả **8.5%** |
| Nguyên nhân nhắm tới | 6.4, 4.2 | 5.2 |

Thư viện **không ghi nhận tích hợp với bất kỳ agent framework nào** — và nó
không cần. Nó chạy trên văn bản của bạn *trước khi* có request nào, nên hoạt
động y hệt trên mọi harness, mọi nhà cung cấp model, kể cả khi bạn không dùng
agent nào cả. Nếu bạn đọc ở đâu đó rằng "caveman hoạt động trên Claude Code,
Cursor, Cline…" thì đó là nói về **skill** — thứ duy nhất trong hai cái phụ
thuộc harness.

## Ý tưởng chung

Ngữ pháp là thứ dự đoán được. "Tôi nghĩ rằng có lẽ chúng ta nên xem xét việc
kiểm tra xem hàm này có xử lý giá trị null hay không" mang đúng lượng thông
tin như "hàm này xử lý null?" — với gấp năm lần số token. Người tiền sử nói
bằng danh từ và động từ; caveman ép văn bản về đúng dạng đó.

Áp dụng vào **văn bản tĩnh bạn kiểm soát** (system prompt, tài liệu, chunk
RAG), đó là một khoản tiết kiệm cố định trên mọi request. Áp dụng vào
**output model**, đó là ép model bớt kể lể.

## Skill "caveman mode"

### Cách hoạt động

Một ruleset dán vào rules file của agent (`CLAUDE.md`, `.clinerules/`,
`AGENTS.md`, `.cursor/rules/`…) hoặc bật theo từng phiên. Nội dung đại ý:
bỏ mở đầu, bỏ chuyển tiếp, bỏ tóm tắt lại việc vừa làm, nói bằng câu cụt.
Không cài đặt gì, không có binary, không có runtime.

Điểm mấu chốt: nó **chỉ chạm được vào phần văn xuôi giữa các lần gọi tool**.
Code, diff, output tool đều phải giữ nguyên văn — và đó là phần lớn token.

### Vì sao 65% thành 8.5%

- Baseline đã làm sẵn phần lớn việc. System prompt của Claude Code vốn đã
  dập phần dẫn nhập và kể lể, nên "chỉ còn phần văn xuôi giữa các lần gọi
  tool được nén, mà phần đó không nhiều".
- **Lần chạy đầu 10 tác vụ cho −29.5%**, rồi con số tan biến khi mẫu lớn
  lên (86 tác vụ, 82 cặp sạch). Benchmark n nhỏ ở lĩnh vực này là vô giá
  trị.
- JetBrains **ép kích hoạt** skill. Nên 8.5% là **trần**; khi để nó tự kích
  hoạt theo ngữ cảnh, mức tiết kiệm "ít hơn hoặc bằng không".

### 8.5% là con số *so với một baseline cụ thể*

Cả ba lý do trên đều nói về **harness**, không nói về skill. Dư địa còn lại
bằng đúng phần văn xuôi mà system prompt của bạn *chưa* dập:

| Bạn đang chạy trên | Dư địa còn lại |
| --- | --- |
| Harness đã ép ngắn gọn sẵn (Claude Code, phần lớn agent code hiện đại) | Ít — 8.5% và đó là trần |
| Harness không nói gì về độ dài | Lớn hơn, nhưng chưa ai đo |
| App tự dựng trên API | **Bạn tự viết system prompt** — đừng cài skill, hãy viết thẳng yêu cầu ngắn gọn vào đó rồi nén nó bằng thư viện bên dưới |

Hàng cuối là kết luận thực dụng: nếu bạn kiểm soát system prompt, "skill" chỉ
là vài câu bạn tự viết. Không có gì để cài, và cũng không có gì che giấu
phương sai bên dưới.

### Rủi ro đuôi — phần nguy hiểm hơn con số trung bình

Một tác vụ kiểm toán dependency vọt lên **$8.29 so với baseline $0.33**,
**lật ngược toàn bộ lợi thế chi phí của cả lần chạy**. Một khoản tiết kiệm
kỳ vọng ~10% với phương sai như vậy thì không phải là khoản tiết kiệm.

Chất lượng là kết quả rỗng: 8 tốt hơn, 10 tệ hơn, 64 hòa (sign test p=0.82).

### Cách dùng

Dán ruleset vào rules file của agent, hoặc gọi theo phiên. Không có bước cài
đặt nào. Nếu vẫn muốn thử:

- Bật **theo route**, không bật toàn cục — chỉ ở những đường đi nặng văn
  xuôi (tóm tắt, giải thích, báo cáo), không phải ở tác vụ agentic dài.
- Đặt trần chi phí cho mỗi tác vụ. Rủi ro đuôi là chế độ hỏng đã được ghi
  nhận, không phải giả thuyết.
- Đo output token theo từng route trước và sau. Đừng đo cảm giác.

## Thư viện `wilpel/caveman-compression`

### Ba backend

| Backend | Mức giảm | Chi phí | Tốc độ | Cơ chế |
| --- | --- | --- | --- | --- |
| LLM (`caveman_compress.py`) | 40–58% | cần OpenAI API key | ~2s/request | Nén có nhận thức ngữ cảnh, chất lượng tốt nhất |
| MLM (`caveman_compress_mlm.py`) | 20–30% | miễn phí, offline | ~1–5s/tài liệu | Bỏ top-k token dễ đoán nhất theo xác suất masked LM |
| NLP (`caveman_compress_nlp.py`) | 15–30% | miễn phí, theo luật | <100ms | Luật ngữ pháp, 15+ ngôn ngữ |

Ví dụ đo được: system prompt 171→72 token (58%), tài liệu API 137→79 (42%),
CV 201→156 (22%); 13/13 sự kiện được bảo toàn.

### Cách dùng

```bash
pip install -r requirements.txt
cp .env.example .env
# backend NLP: pip install -r requirements-nlp.txt
# backend MLM: pip install -r requirements-mlm.txt
```

```bash
python caveman_compress.py compress "Văn bản dài dòng của bạn"
python caveman_compress_nlp.py compress -f input.txt -o output.txt
python caveman_compress_mlm.py compress -f input.txt -k 30
```

Cách dùng đúng là **nén một lần, lưu kết quả, dùng lại mãi** — không phải
gọi nó trên đường đi của mỗi request. Nén system prompt trong lúc build, kiểm
tra bằng mắt, rồi commit bản đã nén.

## Dùng tốt nhất khi

**Thư viện** — và đây mới là chỗ caveman thực sự có giá trị:

- **System prompt và ruleset**: mỗi token cắt được ở đây là token cắt được
  trên *mọi* request suốt vòng đời (nguyên nhân 6.4). Nén 58% một system
  prompt là khoản tiết kiệm cố định, không có phương sai.
- **Chunk RAG và tài liệu tham chiếu** (nguyên nhân 4.2): nội dung máy đọc,
  đưa vào hàng loạt.
- **Backend NLP trước, LLM sau.** NLP dưới 100ms, theo luật, miễn phí và có
  thể chạy trong CI. Chỉ leo lên backend LLM khi 15–30% không đủ.

**Skill** — biên hẹp hơn nhiều:

- Route nặng văn xuôi trên harness **chưa** có system prompt ép ngắn gọn.
  Trên Claude Code, phần lớn khoản tiết kiệm đã bị baseline lấy mất.

## Không dùng khi

- **Nội dung người dùng đọc, marketing, văn bản pháp lý, giao tiếp có sắc
  thái cảm xúc** — đây là khuyến cáo của chính dự án, và nó đúng.
- **Bất cứ thứ gì cần đúng từng chữ**: code, diff, câu lệnh, thông báo lỗi,
  giá trị cấu hình. Skill phải được nói rõ là giữ nguyên những phần đó.
- **Skill trên tác vụ agentic dài** — rủi ro đuôi $8.29-so-với-$0.33 xuất
  hiện đúng ở loại tác vụ này.
- **Khi bạn kỳ vọng 65%.** Con số đó không tồn tại trên một harness hiện đại.

## Đánh đổi

- Nén ngữ pháp làm giảm độ dư thừa, mà độ dư thừa đôi khi chính là thứ giúp
  model không hiểu sai. Hãy kiểm tra chất lượng, đừng chỉ đếm token.
- Với thư viện: backend LLM gửi văn bản của bạn tới OpenAI. Với system prompt
  nội bộ, đó là một quyết định cần cân nhắc — backend MLM và NLP chạy offline.
- Với skill: bạn đang đánh đổi output token lấy phương sai. Trên khối lượng
  nhỏ, phương sai thắng.

## Kiểm chứng trên hệ thống của bạn

1. Thư viện: đếm token trước/sau bằng
   [`../solutions/token-counting.md`](../solutions/token-counting.md), rồi
   chạy một bộ eval hồi quy để chắc chắn không mất *nội dung* — promptfoo
   hoặc Langfuse.
2. Skill: đo **output token theo từng route**, và theo dõi cả **phân vị 95**
   của chi phí mỗi tác vụ, không chỉ trung bình. Trung bình sẽ nói dối với
   bạn ở đây.

---

# Caveman

**What it is:** two **different** artifacts sharing the name "caveman" — a
prompt *skill* that makes the agent write tersely, and a Python *library*
that compresses text you pass in. They are conflated everywhere, including
in this repo previously.

**Addresses:** the skill → cause 5.2; the library → causes 6.4 and 4.2 in
[`../CAUSE.md`](../CAUSE.md). See
[`../solutions/concise-output-prompting.md`](../solutions/concise-output-prompting.md)

**Evidence:** 🟢 Tier A for the skill — advertised **65%**, measured
**8.5%**, and that is the **ceiling**, not the typical case. The library has
no independent A/B. See [`../PROOF.md`](../PROOF.md)

---

## First: these are two different things

| | The library (`wilpel/caveman-compression`) | The skill ("caveman mode") |
| --- | --- | --- |
| What | Python library, MIT, 3 backends | A prompt/ruleset making the agent terse |
| Compresses | Text **you** pass in | Output the **model** writes |
| Claim | 15–58% by backend | 65% |
| Independently A/B'd? | ❌ No | ✅ Yes — result **8.5%** |
| Cause targeted | 6.4, 4.2 | 5.2 |

The library **documents no integration with any agent framework** — and
doesn't need one. It runs on your text *before* any request exists, so it
behaves identically on every harness, every model provider, and with no agent
at all. If you read somewhere that "caveman works on Claude Code, Cursor,
Cline…", that's about the **skill** — the only one of the two that depends on
the harness.

## The shared idea

Grammar is predictable. "I think that perhaps we should consider checking
whether this function handles null values" carries the same information as
"function handle null?" — at five times the tokens. Cavemen speak in nouns
and verbs; caveman forces text into that shape.

Applied to **static text you control** (system prompts, docs, RAG chunks),
that's a fixed saving on every request. Applied to **model output**, it's
pressure on the model to narrate less.

## The "caveman mode" skill

### How it works

A ruleset pasted into your agent's rules file (`CLAUDE.md`, `.clinerules/`,
`AGENTS.md`, `.cursor/rules/`…) or enabled per session. The gist: drop
preamble, drop transitions, drop the recap of what you just did, speak in
clipped sentences. No install, no binary, no runtime.

The crucial constraint: it can only touch **the prose between tool calls**.
Code, diffs and tool output must stay verbatim — and that's where most of the
tokens are.

### Why 65% became 8.5%

- The baseline already did most of the work. Claude Code's system prompt
  already suppresses preamble and narration, so "only the narration between
  tool calls gets compressed, and there is not much of it."
- **The first 10-task run showed −29.5%**, and that dissolved as the sample
  grew (86 tasks, 82 clean pairs). Small-n benchmarks in this space are
  worthless.
- JetBrains **force-activated** the skill. So 8.5% is the **ceiling**;
  left to auto-trigger, savings are "less or nothing."

### 8.5% is a figure *against one specific baseline*

All three reasons above are about the **harness**, not the skill. The
remaining headroom is exactly the prose your system prompt hasn't already
suppressed:

| What you're running on | Headroom left |
| --- | --- |
| A harness that already enforces concision (Claude Code, most modern coding agents) | Little — 8.5%, and that's the ceiling |
| A harness that says nothing about length | More, but nobody has measured it |
| Your own app on the API | **You write the system prompt** — don't install a skill, write the concision requirement into it directly, then compress it with the library below |

That last row is the pragmatic conclusion: if you control the system prompt,
the "skill" is a few sentences you write yourself. Nothing to install — and
nothing hiding the variance underneath either.

### Tail risk — more dangerous than the mean

One dependency-audit task spiked to **$8.29 against a $0.33 baseline**,
**inverting the entire run's cost advantage**. A ~10% expected saving with
that variance is not a saving.

Quality was a null result: 8 better, 10 worse, 64 tied (sign test p=0.82).

### How to use it

Paste the ruleset into your agent's rules file, or invoke it per session.
There is no install step. If you want to try it anyway:

- Enable it **per route**, not globally — only on prose-heavy paths
  (summaries, explanations, reports), not long agentic tasks.
- Put a per-task cost ceiling in place. The tail risk is a documented failure
  mode, not a hypothetical.
- Measure output tokens per route, before and after. Don't measure vibes.

## The `wilpel/caveman-compression` library

### Three backends

| Backend | Reduction | Cost | Speed | Mechanism |
| --- | --- | --- | --- | --- |
| LLM (`caveman_compress.py`) | 40–58% | needs an OpenAI API key | ~2s/request | Context-aware compression, best quality |
| MLM (`caveman_compress_mlm.py`) | 20–30% | free, offline | ~1–5s/doc | Drops the top-k most predictable tokens by masked-LM probability |
| NLP (`caveman_compress_nlp.py`) | 15–30% | free, rule-based | <100ms | Grammar rules, 15+ languages |

Measured examples: a system prompt 171→72 tokens (58%), an API doc 137→79
(42%), a resume 201→156 (22%); 13/13 facts preserved.

### How to use it

```bash
pip install -r requirements.txt
cp .env.example .env
# NLP backend: pip install -r requirements-nlp.txt
# MLM backend: pip install -r requirements-mlm.txt
```

```bash
python caveman_compress.py compress "Your verbose text here"
python caveman_compress_nlp.py compress -f input.txt -o output.txt
python caveman_compress_mlm.py compress -f input.txt -k 30
```

The right usage pattern is **compress once, store the result, reuse forever**
— not calling it in each request's hot path. Compress your system prompt at
build time, eyeball the output, commit the compressed version.

## Best use cases

**The library** — and this is where caveman genuinely earns its keep:

- **System prompts and rulesets**: every token cut here is cut on *every*
  request for the lifetime of the app (cause 6.4). Compressing a system
  prompt 58% is a fixed saving with no variance.
- **RAG chunks and reference docs** (cause 4.2): machine-read content,
  ingested in bulk.
- **NLP backend first, LLM backend later.** NLP is sub-100ms, rule-based,
  free, and CI-runnable. Escalate to the LLM backend only when 15–30% isn't
  enough.

**The skill** — a much narrower window:

- Prose-heavy routes on a harness whose system prompt does **not** already
  enforce concision. On Claude Code, the baseline has already taken most of
  the saving.

## When not to use it

- **User-facing content, marketing copy, legal documents, emotionally
  nuanced communication** — the project's own caveats, and they're right.
- **Anything that must be exact**: code, diffs, commands, error messages,
  config values. The skill must be told explicitly to leave those verbatim.
- **The skill on long agentic tasks** — the $8.29-vs-$0.33 tail risk shows up
  precisely there.
- **When you're expecting 65%.** That number doesn't exist on a modern
  harness.

## Trade-offs

- Compressing grammar removes redundancy, and redundancy is sometimes exactly
  what keeps a model from misreading. Test quality, don't just count tokens.
- For the library: the LLM backend sends your text to OpenAI. For an internal
  system prompt that's a decision worth making deliberately — the MLM and NLP
  backends run offline.
- For the skill: you're trading output tokens for variance. At low volume,
  variance wins.

## Verify it on your own system

1. Library: count tokens before/after with
   [`../solutions/token-counting.md`](../solutions/token-counting.md), then
   run a regression eval to confirm no *content* was lost — promptfoo or
   Langfuse.
2. Skill: measure **output tokens per route**, and track the **p95** of
   per-task cost, not just the mean. The mean will lie to you here.
