# Bằng chứng: điều gì thực sự đo được (Tiếng Việt)

Phần còn lại của kho này nói *nên* làm gì. Trang này chỉ hỏi một câu: **có ai
đo thật chưa, và đo ra bao nhiêu?**

Mỗi mục dưới đây ghép một tuyên bố quảng cáo với con số đo được trong một
thử nghiệm có đối chứng. Ở đâu không có phép đo, trang này nói thẳng là
không có — một ô trống ở đây có giá trị hơn một con số mượn từ trang
marketing.

> Trạng thái: tài liệu sống. Cập nhật lần cuối 2026-08-06. Chỉ nhận nguồn
> có công bố phương pháp; xem [Thang bằng chứng](#thang-bằng-chứng).

Trang này cho biết **cái gì đã được đo**. Muốn tự đo trên thiết lập của
mình — thiết kế ghép cặp, cần bao nhiêu lần chạy, những thứ sẽ đánh lừa bạn
— xem [`MEASURE.md`](MEASURE.md).

## Cách đọc những con số này: baseline chính là harness

**Đây là phần quan trọng nhất của trang.** Mọi phép đo có đối chứng hiện có
đều chạy trên Claude Code. Điều đó có nghĩa mỗi con số dưới đây là **giá trị
biên so với một baseline cụ thể**, không phải thuộc tính của công cụ.

Công thức đúng là:

> **Mức tiết kiệm đo được = tiềm năng của công cụ − phần harness đã tự lo**

Hãy nhìn ba cơ chế khiến RTK thất bại. Không cơ chế nào mô tả RTK cả — cả ba
đều mô tả Claude Code:

| Cơ chế thất bại | Thực chất là gì |
| --- | --- |
| "Claude Code cắt bớt kết quả tool trước ngưỡng của rtk" | Harness **đã lấy** khoản tiết kiệm đó rồi |
| "`Read`/`Grep`/`Glob` không đi qua Bash hook" | Tool file có sẵn của Claude Code, không phải shell |
| Trần: chỉ 33% lệnh Bash, ~20% ký tự kết quả tool → tối đa ≈3% | Trần này là tính chất của *bộ tool của harness này* |

RTK không hề nén kém. **Nó nén thứ mà Claude Code vốn đã vứt đi.** Trên một
harness đổ nguyên output tool vào context, chính mức nén 89% đó còn nguyên
dư địa.

Chuyện tương tự với hai công cụ kia. Caveman chỉ đạt 8.5% vì "chỉ phần kể
chuyện giữa các lần gọi tool được nén, mà phần đó không nhiều" — system
prompt của Claude Code vốn đã dập phần dẫn nhập và kể lể. Ponytail đạt
−10.3% khi đối đầu một baseline vốn đã khuyên đừng over-engineer.

Hệ quả trực tiếp: **một công cụ đạt ~0 trên Claude Code và một công cụ vô
dụng là không phân biệt được** trong các kết quả này.

Điều này khớp với cấu trúc kho đã có — "Tier 0 — thừa hưởng gì từ harness"
trong [`setups/recommended-setup.md`](setups/recommended-setup.md) và "Tier 0
— Cline đã có sẵn những gì" trong
[`setups/coding-setup-cline.md`](setups/coding-setup-cline.md). Khái niệm đã
có sẵn; các benchmark này là bằng chứng cứng đầu tiên cho nó.

### Ba giới hạn, để lập luận này không thành cái cớ

1. **Nó giải thích việc tiết kiệm bằng 0 — không giải thích việc chi phí
   *tăng*.** RTK về đích ở +7.6%, do +13.8% lượt và +14.3% cache read. Output
   bị nén khiến model kém chắc chắn và phải làm nhiều hơn. Đó là tác hại từ
   phía công cụ, nhiều khả năng theo nó sang harness khác.
2. **Vấn đề bypass nhiều khả năng phổ quát.** Cline, Roo và Codex đều có tool
   đọc/tìm file riêng. Agent chỉ dùng shell là ngoại lệ vào 2026, không phải
   thông lệ.
3. **"Claude Code là harness tốt hơn" bản thân nó chưa được đo.** Hợp lý —
   có sẵn truncation, context editing, compaction, caching, tool search —
   nhưng chưa ai chạy đối đầu trực tiếp. Hãy coi đó là lời giải thích khả dĩ,
   không phải sự thật đã xác lập.

Và mặt ngược lại, cần nói thẳng: lập luận này **không thể minh oan cho một
công cụ**. "Có thể nó chạy tốt hơn trên Cline" là một giả thuyết. Nó không
phải bằng chứng, và trang này không cho nó đóng vai bằng chứng.

## Thang bằng chứng

| Hạng | Nghĩa là gì |
| --- | --- |
| 🟢 **A** | Bình duyệt (peer-reviewed), hoặc A/B ghép cặp độc lập, có công bố phương pháp |
| 🟡 **B** | Nhà phát triển tự chạy nhưng **tái lập được**: công bố model, cờ, truy vấn, số liệu thô |
| 🟠 **C** | Có số nhưng phương pháp mỏng hoặc chỉ định tính |
| 🔴 **D** | Tự báo cáo dựa trên một counterfactual không tồn tại — **bị loại** |

## Bảng phán quyết

| Công cụ | Quảng cáo | **Đo được** | Baseline | Hạng |
| --- | --- | --- | --- | --- |
| [Ponytail](#1--ponytail) | −20% chi phí | **−10.3% chi phí** (p=0.004) | Claude Code 2.1.201 | 🟢 A |
| [CodeGraph](#2--codegraph) | — | **−69% token, −60% chi phí** | Claude Code + Opus 4.8 | 🟡 B |
| [Headroom](#3--headroom) | −20% token (coding agent) | chưa có A/B độc lập | — | 🟡 B |
| [Caveman (skill)](#4--caveman) | −65% output | **−8.5% output** | Claude Code 2.1.200 | 🟢 A |
| [RTK](#5--rtk) | −60…90% *(tỉ lệ nén văn bản)* | **+7.6% chi phí** (p=0.004) | Claude Code 2.1.201 | 🟢 A |
| [LLMLingua](#6--tầng-bình-duyệt) | 20× nén | **20× nén** | độc lập với agent | 🟢 A |
| [RouteLLM](#6--tầng-bình-duyệt) | — | **−85% chi phí @ 95% chất lượng** | độc lập với agent | 🟢 A |

⚠️ **Hàng RTK so hai đại lượng khác nhau, và đó chính là vấn đề.** 60–90% là
tỉ lệ nén *văn bản*; +7.6% là *hóa đơn*. Trang này để cạnh nhau để chỉ ra
chỗ lẫn lộn, không phải để coi chúng tương đương — xem [mục 5](#5--rtk). Các
hàng khác so cùng đơn vị với nhau.

Đọc mọi hàng cùng với phần
[baseline chính là harness](#cách-đọc-những-con-số-này-baseline-chính-là-harness).

---

## 1 — Ponytail

**Công cụ duy nhất trong nhóm "skill lan truyền" trụ được qua kiểm định.**
MIT · 95.7k sao · `DietrichGebert/ponytail`

**Nguồn:** JetBrains (Denis Shiryaev), phần 3 trong loạt bài — 80 cặp A/B,
251 lượt chạy tính phí, $246.09, Claude Code 2.1.201 headless
`bypassPermissions`, `claude-sonnet-5` ở reasoning effort **medium**,
ponytail 4.8.4, tiêm ruleset qua SessionStart hook (kiểm toán 100% nhánh
treatment / 0% nhánh baseline).

| Chỉ số | Quảng cáo | Đo được | Ý nghĩa thống kê |
| --- | --- | --- | --- |
| **Chi phí** | −20% | **−10.3%** | **p=0.004 ✅** |
| Lượng code viết ra | −54% | −15.4% | p=0.088 ✗ |
| Thời gian | −27% | −11% | — |

**Vì sao nó chạy được trong khi hai cái kia không:** Ponytail không nén gì
cả. Nó thay đổi thứ agent **quyết định xây** trước khi có dòng code nào. Cái
tiết kiệm được là output thật, không phải một counterfactual dán nhãn lại —
nên không có baseline ma nào để thổi phồng.

**Benchmark của chính dự án** cũng trung thực bất thường: 12 tác vụ tính năng
trên repo FastAPI + React, Haiku 4.5, chạy 4 lần mỗi bên, đo bằng `git diff`
→ LOC −54%, token −22%, chi phí −20%, thời gian −27%, safety 100%. Dự án
**đã tự rút lại** tuyên bố cũ "ít code hơn 80–94%" vì baseline đó có chèn
văn xuôi thừa. Tự đính chính như vậy là hiếm.

### Ba cảnh báo trước khi áp dụng

1. **Cài thụ động thì không chạy.** Khi cài mà không tiêm qua hook, ruleset
   kích hoạt **0 lần trên 10 phiên**. Bắt buộc phải có SessionStart hook.
2. **Vì thế Cline là vấn đề.** Ponytail chỉ liệt kê Cline ở nhóm *adapter chỉ
   có instruction* (chép file vào `.clinerules/`) — đúng là chế độ thụ động
   đã kích hoạt 0% ở trên. Chưa được đo, và về cấu trúc thì đáng ngờ.
3. **Chất lượng là kết quả rỗng, không phải giấy chứng nhận sạch** — 9 tác vụ
   tệ hơn, 6 tốt hơn, 65 y hệt. Khoản tiết kiệm dồn hết vào tình huống
   over-build: −31% ở bản dựng lớn, **bằng 0 khi code vốn đã tối giản**.

**Tiết kiệm khi:** agent có xu hướng xây thừa (scaffold UI, lớp abstraction
đón đầu, tự viết lại thứ stdlib đã có). **Vô ích khi:** tác vụ vốn đã tối
giản, hoặc harness không cho tiêm qua hook.

---

## 2 — CodeGraph

`colbymchenry/codegraph` — knowledge graph dựng sẵn, chạy 100% local, agent
truy vấn qua MCP.

**Nguồn:** benchmark của chính dự án, nhưng **tái lập được**: công bố model
(Claude Opus 4.8), cờ (`claude -p`, `--strict-mcp-config`), 4 lần chạy mỗi
nhánh lấy trung vị, truy vấn chính xác cho từng repo và số liệu thô. Kiểm
định lại 2026-07-21. Nhánh WITHOUT vẫn có đủ `Read`/`Grep`/`Bash`.

**Tổng hợp: −89% lượt gọi tool · −69% token · −60% chi phí · số lần đọc file
về 0 trên cả 7 repo.**

Mọi ô đều đọc là **có CodeGraph vs không có** — cột trái là nhánh dùng
CodeGraph.

| Repo | File | Token (có vs không) | Chi phí (có vs không) |
| --- | --- | --- | --- |
| Tokio (Rust) | 790 | 386k vs 4.3M | $0.44 vs $3.04 |
| Alamofire (Swift) | 110 | 316k vs 3.1M | $0.35 vs $2.51 |
| Excalidraw (TS) | 640 | 324k vs 2.9M | $0.40 vs $1.81 |
| VS Code (TS) | ~11k | 265k vs 1.5M | $0.36 vs $1.41 |
| Django (Python) | ~3k | 254k vs 1.2M | $0.35 vs $1.13 |
| Gin (Go) | 110 | 246k vs 300k | $0.27 vs $0.46 |
| **OkHttp (Java)** | 645 | 156k vs 233k | **$0.23 vs $0.20 — đắt hơn** |

−69% và −60% ở trên là **trung bình mức giảm theo từng repo** (không phải
gộp chung), nên Gin và OkHttp kéo con số xuống đúng như phải thế.

OkHttp nằm trong chính bảng của họ: ít token hơn nhưng **hóa đơn cao hơn**.
Việc không giấu con số đó là tín hiệu mạnh nhất cho độ tin cậy của benchmark.

Chi phí index khiêm tốn: Swift compiler (27k file) ~100 giây; Linux kernel
(70k file, 6.4M quan hệ) dưới 12 phút trên VPS 2 nhân/6GB; đồng bộ tăng dần
~0.3–0.4 giây.

**Dưới lăng kính baseline-là-harness, đây là con số ấn tượng nhất cả trang:**
−69% token thắng được một Claude Code **còn đầy đủ vũ khí**, không phải một
đối thủ rơm.

**Vô ích hoặc đắt hơn khi:**

- Dưới ~150 file — vòng lặp grep của model mạnh xong sớm hơn về thời gian
  thực; và OkHttp cho thấy chi phí có thể đảo chiều ngay ở 645 file.
- **Agent giao việc khám phá cho sub-agent đọc file — index bị bỏ qua hoàn
  toàn.** Điều này đe dọa trực tiếp profile "codebase lớn, nhiều agent" của
  [`setups/recommended-setup.md`](setups/recommended-setup.md).
- Reflection, DI container, routing theo quy ước và dynamic dispatch vô hình
  với phân tích tĩnh (độ phủ framework 74–100%).
- SQLite WAL không đáng tin trên network share và một số cấu hình WSL2.

**Không hỗ trợ Cline.** Danh sách: Claude Code, Cursor, Codex CLI, opencode,
Hermes, Gemini CLI, Antigravity, Kiro.

---

## 3 — Headroom

`headroomlabs-ai/headroom` · Apache-2.0 · library, proxy, và MCP server

Công cụ trung thực về phương pháp nhất trong cả đợt khảo sát này. Tiêu đề
README của chính họ tách bạch: **"ít hơn 20% token cho coding agent, ít hơn
60–95% token cho JSON"** — tự tay tách con số thực tế khiêm tốn khỏi con số
JSON hào nhoáng.

Có công bố phần bảo toàn độ chính xác: GSM8K ±0.000, TruthfulQA +0.030,
SQuAD v2 97% ở mức nén 19%, BFCL 97% ở mức nén 32%.

**Điểm đáng nói nhất:** nó có sẵn **nhóm đối chứng** — để 10% hội thoại không
bị xử lý, nhằm có số đo thật thay vì số ước lượng. Đó đúng là cách sửa lỗi
đã hạ gục RTK. Một công cụ tự cài sẵn cơ chế phản chứng cho chính nó là công
cụ đáng tin.

Chưa có A/B ghép cặp độc lập.

⚠️ **Vấn đề danh tính, cần cẩn thận trước khi cài.** Có ba fork trên GitHub
(`headroomlabs-ai`, `gglucass`, `RaghavRD`), gói pip tên là **`headroom-ai`**
(không phải `headroom`), còn image Docker chính thức lại nằm ở
**`ghcr.io/chopratejas/headroom`** — một namespace thứ tư nữa. Với một công cụ
nằm giữa agent và API key của bạn, ba tên khác nhau là rủi ro chuỗi cung ứng
thật sự. Hãy đối chiếu đúng nguồn trước khi cài, và ghim phiên bản.

---

## 4 — Caveman

### Trước hết: đây là **hai** thứ khác nhau

| | Thư viện (`wilpel/caveman-compression`) | Skill ("caveman mode") |
| --- | --- | --- |
| Là gì | Thư viện Python, MIT, 3 backend | Prompt skill bắt agent viết cộc lốc |
| Nén cái gì | Văn bản **bạn** đưa vào | Output do model viết ra |
| Tuyên bố | 15–58% tùy backend | 65% |
| JetBrains có test không? | ❌ Không | ✅ Có |

Kho này **từng** gán độ phủ đa-agent và con số của *skill* cho *thư viện*.
Theo README của thư viện, nó **không ghi nhận tích hợp với bất kỳ agent
framework nào**. Đã sửa — xem [phần đã sửa](#những-chỗ-đã-sửa-trong-kho-này).

### Skill — đã đo

**Nguồn:** JetBrains, SkillsBench qua Harbor 0.17, 86 tác vụ, ~240 lượt chạy
tính phí, ~$106, Claude Code 2.1.200 headless, `claude-sonnet-5` effort
thấp, 3 lần chạy, 82 cặp sạch.

**Quảng cáo 65% → đo được 8.5%** output (592k → 542k).

Hai cảnh báo về phương pháp: lần chạy 10 tác vụ đầu cho **−29.5%**, rồi tan
biến khi cỡ mẫu tăng — benchmark cỡ mẫu nhỏ ở đây là vô giá trị. Và JetBrains
**ép kích hoạt** skill, nên 8.5% là **trần, không phải trường hợp thường**;
để tự động kích hoạt thì tiết kiệm "ít hơn hoặc bằng không".

**Rủi ro đuôi hủy luôn giá trị trung bình:** một tác vụ audit dependency vọt
lên **$8.29 so với $0.33** ở nhánh baseline — *đảo ngược lợi thế chi phí của
cả đợt chạy*. Tiết kiệm kỳ vọng ~10% với phương sai như vậy không phải là
tiết kiệm.

Chất lượng: 8 tốt hơn, 10 tệ hơn, 64 hòa; sign test p=0.82.

### Thư viện — hữu ích, nhưng ở chỗ khác

Backend LLM 40–58%, MLM 20–30%, NLP 15–30%. Ví dụ đo được: system prompt
171→72 token (58%), API doc 137→79 (42%), CV 201→156 (22%); bảo toàn 13/13
dữ kiện.

**Đây là công cụ cho nguyên nhân 6.4 và 4.2 (văn bản tĩnh bạn kiểm soát:
system prompt, tài liệu, chunk RAG, tập chỉ dẫn), không phải 5.2.** Chính họ
khuyến cáo tránh: nội dung người dùng đọc, marketing, văn bản pháp lý, giao
tiếp cảm xúc.

---

## 5 — RTK

`rtk-ai/rtk` · Apache-2.0 · "Rust Token Killer"

**Nguồn:** JetBrains, 425 lượt chạy tính phí, ~$320, 4 nhánh ghép cặp (10
tác vụ thử, 10 tác vụ k=3, 86 tác vụ effort thấp, 86 tác vụ effort cao),
SkillsBench 86/87 tác vụ, Claude Code 2.1.201 headless, `claude-sonnet-5`,
rtk v0.43.0 ghim SHA256.

| Điều kiện | Chi phí | Lượt | Cache read |
| --- | --- | --- | --- |
| Effort thấp (80 cặp sạch) | **+7.6%** (p=0.004) | +13.8% (p=0.03) | +14.3% (p=0.008) |
| Effort cao | +0.1% (p=0.99) | +0.0 (p=0.74) | — |
| Nhóm phơi nhiễm hook nặng | **~+24%** | — | — |

**`rtk gain` tự báo cáo tiết kiệm 96.2 triệu token — 99.8% mọi thứ nó chạm
vào — trong khi hóa đơn đo được lại tăng.** Ba cơ chế thổi phồng bảng điểm:

1. Đếm output thô đầy đủ làm counterfactual, trong khi Claude Code **vốn đã
   cắt bớt** trước ngưỡng đó.
2. Ước lượng token bằng `bytes/4` lúc thực thi, bỏ qua việc đọc lại từ cache
   chỉ bị tính ~1/10 giá.
3. Hook không nhìn thấy phần lớn context.

**Phân tích trần:** chỉ 33% lệnh Bash mang chưa tới 20% ký tự kết quả tool.
Dù có bóp toàn bộ phần rtk chạm tới đi 70% thì trần vẫn là **≈3% token
input**. Khoảng 60–90% chưa bao giờ khả thi về mặt vật lý trên harness này.

**Cần ghi nhận:** README của RTK trung thực — nó nói rõ "output bash chỉ là
*một* phần đóng góp vào token input, bên cạnh prompt của bạn, system prompt
và lịch sử hội thoại", rằng token đếm bằng `bytes/4` nên "phần trăm đáng tin
nhưng con số tuyệt đối chỉ là xấp xỉ", và rằng `Read`/`Grep`/`Glob` không đi
qua Bash hook. **60–90% là tỉ lệ nén văn bản, không phải mức giảm hóa đơn.**
Marketing và các blog phái sinh đã trộn lẫn hai thứ.

Chất lượng không đổi (5 tốt hơn / 4 tệ hơn / 71 hòa, p=1.0). Hai lỗi thật:
vị từ `find` phức hợp gây lỗi dùng sai và phải thử lại; binary không khởi
động được trên một image vì yêu cầu glibc.

**Có thể vẫn tiết kiệm khi:** harness **không** tự cắt bớt output tool, và
lệnh dài thật sự đi qua Bash. Tỉ lệ nén trên chính văn bản là có thật:
pytest/cargo test/go test −90%, cargo build/clippy −80%, ruff −80%. Chưa ai
đo trên Cline, Codex hay Gemini.

**Lưu ý mâu thuẫn nguồn:** README của RTK xếp Codex ở tầng hook, nhưng bài
viết chuyên về Codex mô tả tầng rules-file (`RTK.md` + vá `AGENTS.md`). Với
Cline, cả hai nguồn đều thống nhất là tầng rules-file `.clinerules` — tức là
**model phải tự nguyện tuân theo**. Yêu cầu tích hợp hook gốc cho Codex
([openai/codex#19001](https://github.com/openai/codex/issues/19001)) vẫn
đang mở, không có phản hồi từ maintainer và không có số liệu.

---

## 6 — Tầng bình duyệt

Độc lập với agent — là thư viện và hạ tầng, nên chạy như nhau dưới Cline,
Claude, Codex, Gemini. Điều đó né được đúng lỗ hổng độ phủ đã hạ gục RTK.

| Công trình | Kết quả | Đánh giá trên |
| --- | --- | --- |
| **LLMLingua** (EMNLP 2023, [arXiv:2310.05736](https://arxiv.org/abs/2310.05736)) | nén tới **20×**, hao hụt nhỏ | GSM8K, BBH, ShareGPT, Arxiv-March23 |
| **LongLLMLingua** ([arXiv:2310.06839](https://arxiv.org/abs/2310.06839)) | ít token hơn **và hiệu năng +21.4%** | NaturalQuestions, LongBench |
| **LLMLingua-2** | nén 2–5×, độ trễ đầu-cuối giảm tới 2.9× | task-agnostic |
| **RouteLLM** ([OpenReview](https://openreview.net/forum?id=8sSqNntaMr)) | **−85% chi phí MT Bench** ở 95% hiệu năng GPT-4 | MT Bench, MMLU, GSM8K |

LongLLMLingua đáng chú ý vì kết quả *tăng* hiệu năng — nén mà cải thiện độ
chính xác là thứ rất khó ngụy tạo.

RouteLLM công bố toàn bộ code và dataset. Hạn chế họ tự nêu (hiếm và đáng
tin): phần lớn câu hỏi MMLU nằm ngoài phân phối huấn luyện nên router chạy
kém nếu **không** bổ sung dữ liệu. Routing không phải thứ cắm vào là chạy.

**Chưa xác minh:** FrugalGPT (Stanford) được nhắc tới với mức giảm chi phí
tới 98%. Chưa đọc bài báo gốc — chưa nhận vào trang này.

**Chưa xác minh:** chưa tìm được bản tái lập độc lập cho LLMLingua. Đã bình
duyệt là mạnh hơn tự báo cáo, nhưng không đồng nghĩa đã được nhân rộng.

---

## 7 — Native và thương mại

**Anthropic — code execution với MCP:** 150.000 → 2.000 token (**98.7%**)
trên **một** workflow Google Drive → Salesforce, bằng cách tải định nghĩa
tool theo nhu cầu và giữ kết quả trung gian trong môi trường thực thi. Chính
Anthropic lưu ý chi phí sandbox, giới hạn tài nguyên và hạ tầng. Đây là một
ví dụ minh họa, không phải bộ benchmark.

**JetBrains Context:** khối lượng tác vụ lớn nhất trong cả trang — 205 tác vụ
SWE-bench + 175 tác vụ monorepo production + 1.953 tác vụ code-localization →
giảm tới 68% lượt agent, 59% độ trễ, **48% chi phí**. Hỗ trợ Claude Code,
Codex, Junie. Không mã nguồn mở, cần license JetBrains AI. **Không** được đo
bằng chính phương pháp A/B ghép cặp SkillsBench mà họ dùng để bác bỏ RTK và
Caveman.

**serena-slim** (`mcpslim/serena-slim`): 29 tool ≈ **23.878 token** overhead
schema gom lại còn 18 thao tác ngữ nghĩa, **−50.3%**. Hẹp nhưng cụ thể và
kiểm chứng được — nhắm thẳng nguyên nhân 3.4.

**Serena** (`oraios/serena`, MIT) **không** công bố con số giảm token nào.
Đánh giá của họ là ~20 tác vụ thường ngày, thẩm định bằng *phản hồi định tính
của agent*. Đó là lời chứng, không phải phép đo.

---

## 8 — Bị loại

| Nguồn | Vì sao loại |
| --- | --- |
| jCodeMunch MCP — "giảm 99.6%", "đã tiết kiệm 313 tỷ token" | Đúng mẫu `rtk gain`: đếm ngược một counterfactual mà harness vốn đã cắt |
| MadPlay — "111k → 23.2k token (80%)" trên Claude Code | Dùng số `rtk gain` tự báo cáo, **không phải token bị tính tiền** |
| "90.7%, $1.12 → $0.10" trên Codex | Trộn lẫn RTK **và** Serena MCP — không quy được cho công cụ nào |
| Bifrost Code Mode — 92.8% ở 500+ tool | Nhà phát triển tự chạy, chưa có A/B (dù cơ chế là cơ chế đúng) |
| tekai.dev, dashen-tech, knightli, toolgenix, agentconn, arceapps, servicesground, thepromptshelf, lennypeters, rushis, và các bản đăng lại trên Medium/DEV | Nội dung SEO nhai lại tuyên bố nhà cung cấp, không đo gì. Số sao GitHub họ đưa cho ponytail (74k/78.7k/94.3k) đều sai — số thật là 95.7k |

---

## Ma trận độ phủ theo agent

| Công cụ | Claude | Cline | Codex | Gemini |
| --- | --- | --- | --- | --- |
| Ponytail | ✅ **−10.3% đo được** | ⚠️ chỉ instruction (tầng đã kích hoạt 0%) | ⚪ hỗ trợ plugin, chưa đo | ⚪ hỗ trợ plugin, chưa đo |
| CodeGraph | ✅ **−60% chi phí đo được** | 🚫 không hỗ trợ | ⚪ hỗ trợ, chưa đo | ⚪ hỗ trợ, chưa đo |
| Headroom | ⚪ chưa có A/B | ⚪ proxy, chưa đo | ⚪ proxy, chưa đo | ⚪ proxy, chưa đo |
| Caveman (skill) | ⚠️ **8.5% đo được** | ⚪ chưa đo | ⚪ chưa đo | ⚪ chưa đo |
| RTK | ❌ **+7.6% đo được** | ⚪ tầng rules-file, chưa đo | ⚪ tầng rules-file, chưa đo | ⚪ tầng hook, chưa đo |
| LLMLingua / RouteLLM | ✅ | ✅ | ✅ | ✅ |

Ký hiệu ở bảng này là **trạng thái đo lường**, không phải hạng bằng chứng:
✅ đã đo và có lợi · ⚠️ đã đo nhưng yếu, hoặc đường tích hợp đáng ngờ ·
❌ đã đo và có hại · ⚪ chưa đo · 🚫 không hỗ trợ. Caveman vẫn là bằng chứng
**hạng A**; thứ đáng thất vọng là kết quả, không phải phương pháp.

**Cline là vùng trắng về đo lường.** Chưa có công cụ chuyên biệt nào từng
được benchmark trên nó. Chỉ tầng bình duyệt độc-lập-với-agent là có thể đề
xuất một cách trung thực ở đó.

## Hai mẫu hình lặp lại

**1. Ngăn việc thắng nén việc.** Ponytail (−10.3%) và CodeGraph (−60% chi
phí) đều *ngăn công việc xảy ra*. RTK và Caveman đều bóp output sau khi nó đã
được tạo ra, và cả hai hụt 5–10 lần so với quảng cáo.

**2. Cái bẫy counterfactual.** Cả RTK lẫn jCodeMunch đều đo mình dựa trên
"lẽ ra đã gửi bao nhiêu" — một baseline mà harness vốn đã không gửi. Đây là
kiểm tra duy nhất đáng giá cho một tuyên bố tiết kiệm: **họ đo hóa đơn thật,
hay đo một thế giới giả định?**

Hệ quả: chất lượng bằng chứng tỉ lệ nghịch với độ hào nhoáng của công cụ. Nén
prompt và định tuyến model — nhàm chán, bình duyệt, độc lập với agent — trụ
được ở 20× và 85%. Các binary lan truyền hứa 60–90% với zero-config đo ra
**+7.6%** và **8.5%**.

## Xung đột lợi ích cần công bố

JetBrains ra mắt **JetBrains Context** vào 2026-07 — cùng tháng với cả hai
bài bác bỏ, và là đối thủ trực tiếp trong cùng phân khúc. Phương pháp của họ
chắc chắn và kết quả tiêu cực có bằng chứng vững; nhưng một trang tự nhận nói
về nguồn *đáng tin* thì phải công bố rằng bên bác bỏ có bán sản phẩm cạnh
tranh. Cả hai benchmark cũng dùng một harness (SkillsBench), một model
(`claude-sonnet-5`), và kích hoạt ép/tối đa.

## Những chỗ đã sửa trong kho này

Trang này từng mâu thuẫn với các con số ở nơi khác. **Đã xử lý xong
2026-08-05 và 2026-08-06**, ghi lại để truy vết:

| Vị trí | Vấn đề | Đã sửa thành |
| --- | --- | --- |
| `solutions/tool-output-compression.md` | RTK "60–90%" trình bày như mức giảm hóa đơn | Ghi rõ là tỉ lệ nén *văn bản*; thêm cảnh báo +7.6% và điều kiện harness không tự cắt bớt |
| `solutions/tool-output-compression.md` | Lẫn lộn thư viện/skill Caveman | Tách rõ hai artifact |
| `solutions/concise-output-prompting.md` | Gán độ phủ đa-agent và ~75% của *skill* cho thư viện `wilpel` | Tách thành hai hàng riêng, kèm con số đo được |
| `setups/coding-setup-cline.md` | Khẳng định 60–90% riêng cho Cline; bộ ba "RTK + Headroom + Caveman" | Nêu rõ chưa từng đo trên Cline; hạ bộ ba xuống mức giả thuyết, yêu cầu bật từng cái kèm đo |
| `solutions/README.md`, `solutions/tool-search.md` | "Code Mode ~99.9%" | Sửa thành 98.7% trên **một** workflow của Anthropic; đánh dấu con số 99.9% là không nguồn |
| Mọi nơi nhắc Headroom | Trích "60–95%" | Đổi sang **~20% cho coding agent**; 60–95% ghi rõ là con số JSON |
| `setups/recommended-setup.md` | Caveman "cắt token output trên lưu lượng agent" | Thêm 8.5% đo được và rủi ro đuôi $8.29 vs $0.33 |
| `solutions/code-maps.md`, `CAUSE.md` | "25–60K token" không có nguồn | Đánh dấu là ước lượng không nguồn; thêm CodeGraph làm lựa chọn có bằng chứng |
| `README.md`, `SUMMARY.md`, `solutions/README.md` | Không có đường dẫn tới trang này | Đã liên kết, kèm cảnh báo đọc trước khi tin phần trăm |
| Cả ba tài liệu trong `setups/` | Chỉ nêu các công cụ đo ra ~0 hoặc âm (RTK, Caveman, Headroom); **không hề nhắc** hai công cụ duy nhất đo ra có lợi | Ponytail và CodeGraph đã vào cả ba, xếp theo bằng chứng: `recommended-setup.md` thêm cả hai vào Tier 2 + bộ công cụ tham chiếu kèm cảnh báo bypass subagent; `coding-setup-cline.md` nêu rõ cả hai **không tới được Cline** và chỉ ra cách tự làm lấy cơ chế; `coding-setup-enterprise.md` thêm ma trận add-on theo từng nhà cung cấp |

## Nguồn

- [JetBrains — rtk Claude Code Token Savings: A Skill Trial Benchmark](https://blog.jetbrains.com/ai/2026/07/rtk-claude-code-token-savings/)
- [JetBrains — Speaking to AI Agents like Cavemen Saves 65% of Tokens. We Test.](https://blog.jetbrains.com/ai/2026/07/speak-to-ai-agents-like-cavemen-tosave-tokens/)
- [JetBrains — Ponytail Skill for Claude Code: Does It Really Cut Tokens](https://blog.jetbrains.com/ai/2026/07/ponytail-skill-claude-tested/)
- [JetBrains — Introducing JetBrains Context](https://blog.jetbrains.com/ai/2026/07/introducing-jetbrains-context-repository-intelligence-for-coding-agents/)
- [github.com/DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail)
- [github.com/colbymchenry/codegraph](https://github.com/colbymchenry/codegraph)
- [github.com/headroomlabs-ai/headroom](https://github.com/headroomlabs-ai/headroom)
- [github.com/rtk-ai/rtk](https://github.com/rtk-ai/rtk)
- [github.com/wilpel/caveman-compression](https://github.com/wilpel/caveman-compression)
- [github.com/oraios/serena](https://github.com/oraios/serena)
- [LLMLingua — arXiv:2310.05736 (EMNLP 2023)](https://arxiv.org/abs/2310.05736)
- [LongLLMLingua — arXiv:2310.06839](https://arxiv.org/abs/2310.06839)
- [RouteLLM — OpenReview](https://openreview.net/forum?id=8sSqNntaMr) · [LMSYS](https://www.lmsys.org/blog/2024-07-01-routellm/)
- [Anthropic — Code execution with MCP](https://www.anthropic.com/engineering/code-execution-with-mcp)
- [RTK + Codex CLI](https://codex.danielvaughan.com/2026/05/19/rtk-codex-cli-token-optimisation-shell-output-compression/) · [openai/codex#19001](https://github.com/openai/codex/issues/19001)

---

# Proof: what has actually been measured

The rest of this repo says what you *should* do. This page asks one question:
**has anyone measured it, and what did they get?**

Each entry below pairs an advertised claim with the number a controlled trial
produced. Where no measurement exists, the page says so — a blank here is
worth more than a figure borrowed from a marketing page.

> Status: living document. Last updated 2026-08-06. Only sources with a
> published methodology are admitted; see [Evidence tiers](#evidence-tiers).

This page tells you **what has been measured**. To measure something on your
own setup — the paired design, how many runs it takes, what will fool you —
see [`MEASURE.md`](MEASURE.md).

## Reading these numbers: the baseline is the harness

**This is the most important section on the page.** Every controlled
measurement that exists was run on Claude Code. That means each number below
is a **marginal value against one specific baseline**, not a property of the
tool.

The honest formula is:

> **Measured saving = the tool's potential − what the harness already
> captured**

Look at the three mechanisms behind RTK's failure. None of them describes
RTK — all three describe Claude Code:

| Failure mechanism | What it actually is |
| --- | --- |
| "Claude Code truncates tool results before rtk's threshold" | The harness **already captured** that saving |
| "`Read`/`Grep`/`Glob` bypass the Bash hook" | Claude Code's built-in file tools, not shell |
| Ceiling: only 33% of Bash calls, ~20% of tool-result chars → max ≈3% | That ceiling is a property of *this harness's tool mix* |

RTK didn't fail to compress. **It compressed something Claude Code had
already thrown away.** On a harness that dumps raw tool output into context,
that same 89% compression has real headroom.

The same holds for the other two. Caveman measured 8.5% because "only the
narration between tool calls gets compressed, and there is not much of it" —
Claude Code's system prompt already suppresses preamble and narration.
Ponytail got −10.3% against a baseline that already discourages
over-engineering.

The direct consequence: **a tool scoring ~0 on Claude Code and a tool that
does nothing are indistinguishable** in these results.

This maps onto structure the repo already has — "Tier 0 — what to inherit
from the harness" in [`setups/recommended-setup.md`](setups/recommended-setup.md)
and "Tier 0 — what Cline gives you natively" in
[`setups/coding-setup-cline.md`](setups/coding-setup-cline.md). The concept
was already here; these benchmarks are the first hard evidence for it.

### Three limits, so this doesn't become an excuse

1. **It explains zero savings — it does not explain a cost *increase*.** RTK
   landed at +7.6%, driven by +13.8% turns and +14.3% cache reads. Compressed
   output made the model less certain and it worked harder. That is a
   tool-side harm likely to follow it to any harness.
2. **The bypass problem probably generalizes.** Cline, Roo and Codex all have
   their own file-read/search tools. Shell-only agents are the exception in
   2026, not the rule.
3. **"Claude Code is the better harness" is itself unmeasured.** Plausible —
   native truncation, context editing, compaction, caching, tool search — but
   nobody has run these harnesses head-to-head. Treat it as the likely
   explanation, not established fact.

And the flip side, stated plainly: this argument **cannot rehabilitate a
tool**. "It might do better on Cline" is a hypothesis. It is not evidence,
and this page does not let it act as one.

## Evidence tiers

| Tier | What it means |
| --- | --- |
| 🟢 **A** | Peer-reviewed, or independent paired A/B with published methodology |
| 🟡 **B** | Vendor-run but **reproducible**: model, flags, queries and raw figures published |
| 🟠 **C** | Numbers exist but the method is thin or qualitative |
| 🔴 **D** | Self-reported against a counterfactual that never existed — **rejected** |

## Verdict table

| Tool | Advertised | **Measured** | Baseline | Tier |
| --- | --- | --- | --- | --- |
| [Ponytail](#1--ponytail-1) | −20% cost | **−10.3% cost** (p=0.004) | Claude Code 2.1.201 | 🟢 A |
| [CodeGraph](#2--codegraph-1) | — | **−69% tokens, −60% cost** | Claude Code + Opus 4.8 | 🟡 B |
| [Headroom](#3--headroom-1) | −20% tokens (coding agents) | no independent A/B | — | 🟡 B |
| [Caveman (skill)](#4--caveman-1) | −65% output | **−8.5% output** | Claude Code 2.1.200 | 🟢 A |
| [RTK](#5--rtk-1) | −60…90% *(text-compression ratio)* | **+7.6% cost** (p=0.004) | Claude Code 2.1.201 | 🟢 A |
| [LLMLingua](#6--the-peer-reviewed-tier) | 20× compression | **20× compression** | agent-independent | 🟢 A |
| [RouteLLM](#6--the-peer-reviewed-tier) | — | **−85% cost @ 95% quality** | agent-independent | 🟢 A |

⚠️ **The RTK row compares two different quantities — and that is the point.**
60–90% is a *text* compression ratio; +7.6% is a *bill*. They sit side by side
here to expose the conflation, not to treat them as commensurable — see
[section 5](#5--rtk-1). Every other row compares like with like.

Read every row together with
[the baseline is the harness](#reading-these-numbers-the-baseline-is-the-harness).

---

## 1 — Ponytail

**The only viral skill that survived a controlled test.** MIT · 95.7k stars ·
`DietrichGebert/ponytail`

**Source:** JetBrains (Denis Shiryaev), part 3 of the series — 80 paired A/B
tasks, 251 billed trials, $246.09, Claude Code 2.1.201 headless
`bypassPermissions`, `claude-sonnet-5` at **medium** reasoning effort,
ponytail 4.8.4, ruleset injected via SessionStart hook (audited at 100%
treatment / 0% baseline).

| Metric | Advertised | Measured | Significance |
| --- | --- | --- | --- |
| **Cost** | −20% | **−10.3%** | **p=0.004 ✅** |
| Code written | −54% | −15.4% | p=0.088 ✗ |
| Time | −27% | −11% | — |

**Why it works where the others don't:** Ponytail compresses nothing. It
changes what the agent **decides to build** before a line of code exists. The
saving is real output, not a re-labelled counterfactual — so there is no
phantom baseline to inflate.

**Its own benchmark** is unusually honest too: 12 feature tasks on a FastAPI
+ React repo, Haiku 4.5, 4 runs each side, measured via `git diff` → LOC
−54%, tokens −22%, cost −20%, time −27%, safety 100%. The project
**explicitly retired** its older "80–94% less code" claim because that
baseline included prose padding. Self-correction like that is rare.

### Three warnings before adopting

1. **A passive install does nothing.** Installed without hook injection, the
   ruleset activated **zero times across ten sessions**. A SessionStart hook
   is required.
2. **Which makes Cline a problem.** Ponytail lists Cline only as an
   *instruction-only adapter* (copy files into `.clinerules/`) — precisely
   the passive mode that fired 0% above. Unmeasured, and structurally
   suspect.
3. **Quality is a null result, not a clean bill of health** — 9 tasks worse,
   6 better, 65 identical. Savings concentrate entirely in over-build
   scenarios: −31% on large builds, **zero where the code is already
   minimal**.

**Saves when:** the agent tends to over-build (UI scaffolding, speculative
abstraction layers, re-implementing what stdlib already has). **Useless
when:** the task is already minimal, or the harness offers no hook injection.

---

## 2 — CodeGraph

`colbymchenry/codegraph` — a pre-indexed knowledge graph, fully local,
queried by the agent over MCP.

**Source:** vendor-run, but **reproducible**: published model (Claude Opus
4.8), flags (`claude -p`, `--strict-mcp-config`), 4 runs per arm reporting
the median, the exact query per repo, and raw figures. Re-validated
2026-07-21. The WITHOUT arm still had `Read`/`Grep`/`Bash`.

**Aggregate: −89% tool calls · −69% tokens · −60% cost · file reads cut to
zero on all seven repos.**

Every cell reads **with CodeGraph vs without** — the left value is the
CodeGraph arm.

| Repo | Files | Tokens (with vs without) | Cost (with vs without) |
| --- | --- | --- | --- |
| Tokio (Rust) | 790 | 386k vs 4.3M | $0.44 vs $3.04 |
| Alamofire (Swift) | 110 | 316k vs 3.1M | $0.35 vs $2.51 |
| Excalidraw (TS) | 640 | 324k vs 2.9M | $0.40 vs $1.81 |
| VS Code (TS) | ~11k | 265k vs 1.5M | $0.36 vs $1.41 |
| Django (Python) | ~3k | 254k vs 1.2M | $0.35 vs $1.13 |
| Gin (Go) | 110 | 246k vs 300k | $0.27 vs $0.46 |
| **OkHttp (Java)** | 645 | 156k vs 233k | **$0.23 vs $0.20 — cost more** |

The −69% and −60% headline figures are the **mean of the per-repo
reductions**, not a pooled total — so Gin and OkHttp drag them down, as they
should.

OkHttp sits in their own table: fewer tokens but a **higher bill**. Not
hiding it is the strongest signal for the benchmark's credibility.

Indexing overhead is modest: the Swift compiler (27k files) in ~100 seconds;
the Linux kernel (70k files, 6.4M relationships) in under 12 minutes on a
2-core/6GB VPS; incremental sync ~0.3–0.4 seconds.

**Under the baseline-is-the-harness lens this is the most impressive number
on the page:** −69% tokens was won against a **fully-equipped** Claude Code,
not a strawman.

**Useless or costs more when:**

- Under ~150 files — a strong model's raw grep loop finishes sooner in
  wall-clock terms; and OkHttp shows cost can invert even at 645 files.
- **The agent delegates exploration to file-reading sub-agents — the index is
  bypassed entirely.** This directly threatens the "large codebase, many
  agents" profile in
  [`setups/recommended-setup.md`](setups/recommended-setup.md).
- Reflection, DI containers, convention-based routing and dynamic dispatch
  are invisible to static analysis (framework coverage 74–100%).
- SQLite WAL is unreliable on network shares and some WSL2 configurations.

**No Cline support.** The list is Claude Code, Cursor, Codex CLI, opencode,
Hermes, Gemini CLI, Antigravity, Kiro.

---

## 3 — Headroom

`headroomlabs-ai/headroom` · Apache-2.0 · library, proxy, and MCP server

The most methodologically honest tool in this survey. Its own README headline
separates the two cases: **"20% fewer tokens for coding agents, 60–95% fewer
tokens for JSON"** — voluntarily splitting the modest real-world figure from
the flashy JSON one.

It publishes accuracy preservation: GSM8K ±0.000, TruthfulQA +0.030, SQuAD v2
97% at 19% compression, BFCL 97% at 32% compression.

**The part that matters most:** it ships a built-in **control group** — leave
10% of conversations unshaped to get measured rather than estimated savings.
That is exactly the fix for the flaw that sank RTK. A tool that builds in its
own falsification is a tool worth trusting.

No independent paired A/B exists.

⚠️ **An identity problem worth care before installing.** There are three
GitHub forks (`headroomlabs-ai`, `gglucass`, `RaghavRD`), the pip package is
**`headroom-ai`** (not `headroom`), and the official Docker image lives at
**`ghcr.io/chopratejas/headroom`** — a fourth namespace. For a tool that sits
between your agent and your API key, three different names is a real
supply-chain risk. Confirm which source you are pulling from, and pin the
version.

---

## 4 — Caveman

### First: these are **two** different things

| | The library (`wilpel/caveman-compression`) | The skill ("caveman mode") |
| --- | --- | --- |
| What | Python library, MIT, 3 backends | Prompt skill making the agent terse |
| Compresses | Text **you** pass in | Output the model writes |
| Claim | 15–58% by backend | 65% |
| Tested by JetBrains? | ❌ No | ✅ Yes |

This repo **used to** attribute the *skill's* cross-agent coverage and
figures to the *library*. Per the library's README, it **documents no
integration with any agent framework**. Fixed — see
[corrections](#corrections-applied-to-this-repo).

### The skill — measured

**Source:** JetBrains, SkillsBench via Harbor 0.17, 86 tasks, ~240 billed
trials, ~$106, Claude Code 2.1.200 headless, `claude-sonnet-5` at low effort,
3 runs, 82 clean pairs.

**Advertised 65% → measured 8.5%** output reduction (592k → 542k).

Two methodology warnings: the first 10-task run showed **−29.5%**, which
dissolved as the sample grew — small-n benchmarks here are worthless. And
JetBrains **force-activated** the skill, so 8.5% is the **ceiling, not the
usual case**; auto-triggered usage saves "less or nothing."

**Tail risk destroys the mean:** one dependency-audit task spiked to **$8.29
against a $0.33 baseline** — *inverting the entire run's cost advantage*. A
~10% expected saving with that variance is not a saving.

Quality: 8 better, 10 worse, 64 tied; sign test p=0.82.

### The library — useful, but elsewhere

LLM backend 40–58%, MLM 20–30%, NLP 15–30%. Measured examples: a system
prompt 171→72 tokens (58%), an API doc 137→79 (42%), a resume 201→156 (22%);
13/13 facts preserved.

**This is a tool for causes 6.4 and 4.2 (static text you control: system
prompts, docs, RAG chunks, instruction sets), not 5.2.** Its own caveats:
avoid user-facing content, marketing copy, legal documents, emotional
communication.

---

## 5 — RTK

`rtk-ai/rtk` · Apache-2.0 · "Rust Token Killer"

**Source:** JetBrains, 425 billed trials, ~$320, four paired arms (10-task
smoke, 10 tasks at k=3, 86 tasks at low effort, 86 at high effort),
SkillsBench 86 of 87 tasks, Claude Code 2.1.201 headless, `claude-sonnet-5`,
rtk v0.43.0 pinned by SHA256.

| Condition | Cost | Turns | Cache reads |
| --- | --- | --- | --- |
| Low effort (80 clean pairs) | **+7.6%** (p=0.004) | +13.8% (p=0.03) | +14.3% (p=0.008) |
| High effort | +0.1% (p=0.99) | +0.0 (p=0.74) | — |
| Heavily hook-exposed pairs | **~+24%** | — | — |

**`rtk gain` self-reported 96.2 million tokens saved — 99.8% of everything it
touched — while the measured bill went up.** Three mechanisms inflate the
scoreboard:

1. It counts full raw output as the counterfactual, while Claude Code
   **already truncates** before that threshold.
2. It estimates tokens as `bytes/4` at execution time, ignoring that cached
   re-reads bill at ~1/10 the price.
3. The hook never sees the majority of context.

**Ceiling analysis:** only 33% of Bash calls carried just under 20% of
tool-result chars. Even squeezing rtk's entire share by 70% caps out at
**≈3% of input tokens**. The 60–90% range was never physically reachable on
this harness.

**Credit where due:** RTK's README is honest — it states that bash output is
"*one* contributor to input tokens, alongside your prompt, the system prompt
and conversation history," that token counts are estimated as `bytes/4` so
"the percentages are reliable but the absolute token numbers are
approximate," and that `Read`/`Grep`/`Glob` do not pass through the Bash
hook. **60–90% is a text-compression ratio, not a bill reduction.** The
marketing and the downstream blogs conflated the two.

Quality was unchanged (5 better / 4 worse / 71 ties, p=1.0). Two genuine
failures: compound `find` predicates caused usage errors and retries; the
binary refused to start on one task image over a glibc requirement.

**May still save when:** the harness does **not** truncate tool output, and
verbose commands genuinely route through Bash. The compression ratios on the
text itself are real: pytest/cargo test/go test −90%, cargo build/clippy
−80%, ruff −80%. Nobody has measured this on Cline, Codex or Gemini.

**Source conflict to note:** RTK's README places Codex at the hook tier, but
the dedicated Codex writeup describes a rules-file tier (`RTK.md` plus an
`AGENTS.md` patch). For Cline both sources agree on the `.clinerules`
rules-file tier — meaning **the model must voluntarily comply**. The request
for native Codex hooks
([openai/codex#19001](https://github.com/openai/codex/issues/19001)) is still
open, with no maintainer response and no data.

---

## 6 — The peer-reviewed tier

Agent-independent — these are libraries and infrastructure, so they behave
identically under Cline, Claude, Codex and Gemini. That sidesteps the very
coverage gap that sank RTK.

| Work | Result | Evaluated on |
| --- | --- | --- |
| **LLMLingua** (EMNLP 2023, [arXiv:2310.05736](https://arxiv.org/abs/2310.05736)) | up to **20× compression**, little performance loss | GSM8K, BBH, ShareGPT, Arxiv-March23 |
| **LongLLMLingua** ([arXiv:2310.06839](https://arxiv.org/abs/2310.06839)) | fewer tokens **and up to +21.4% performance** | NaturalQuestions, LongBench |
| **LLMLingua-2** | 2–5× compression, up to 2.9× lower end-to-end latency | task-agnostic |
| **RouteLLM** ([OpenReview](https://openreview.net/forum?id=8sSqNntaMr)) | **−85% cost on MT Bench** at 95% of GPT-4 performance | MT Bench, MMLU, GSM8K |

LongLLMLingua is notable for reporting a performance *gain* — compression
that improves accuracy is a hard thing to fake.

RouteLLM released all code and datasets. Its self-stated limitation (rare and
credible): most MMLU questions were out-of-distribution, so the router
performed poorly **without** data augmentation. Routing is not drop-in.

**Unverified:** FrugalGPT (Stanford) is cited with cost reductions up to 98%.
The paper has not been read — not admitted to this page.

**Unverified:** no independent replication of LLMLingua was located. Peer
review is stronger than vendor self-report, but it is not the same as having
been reproduced.

---

## 7 — Native and commercial

**Anthropic — code execution with MCP:** 150,000 → 2,000 tokens (**98.7%**)
on **one** Google Drive → Salesforce workflow, by loading tool definitions on
demand and keeping intermediates inside the execution environment. Anthropic
itself flags sandboxing, resource limits and infrastructure overhead. This is
an illustrative example, not a benchmark suite.

**JetBrains Context:** the largest task volume on this page — 205 SWE-bench
tasks + 175 production-monorepo tasks + 1,953 code-localization tasks → up to
68% fewer agent turns, 59% less latency, **48% lower cost**. Supports Claude
Code, Codex, Junie. Closed source, requires a JetBrains AI licence. **Not**
evaluated with the same paired-A/B SkillsBench methodology they used to
debunk RTK and Caveman.

**serena-slim** (`mcpslim/serena-slim`): 29 tools ≈ **23,878 tokens** of
schema overhead collapsed into 18 semantic operations, **−50.3%**. Narrow but
concrete and verifiable — aimed straight at cause 3.4.

**Serena** (`oraios/serena`, MIT) publishes **no** token-reduction figures at
all. Its evaluation was ~20 routine tasks assessed via *qualitative agent
feedback*. That is testimonial, not measurement.

---

## 8 — Rejected

| Source | Why rejected |
| --- | --- |
| jCodeMunch MCP — "99.6% reduction", "313B+ tokens saved" | The `rtk gain` pattern exactly: counting against a counterfactual the harness would have truncated |
| MadPlay — "111k → 23.2k tokens (80%)" on Claude Code | Uses rtk's self-reported figure, **not billed tokens** |
| "90.7%, $1.12 → $0.10" on Codex | Confounds RTK **and** Serena MCP — attributable to neither |
| Bifrost Code Mode — 92.8% at 500+ tools | Vendor-run, no A/B (though the mechanism is the sound one) |
| tekai.dev, dashen-tech, knightli, toolgenix, agentconn, arceapps, servicesground, thepromptshelf, lennypeters, rushis, and the Medium/DEV reposts | SEO content restating vendor claims with no measurement. Their ponytail star counts (74k/78.7k/94.3k) are all wrong — the real figure is 95.7k |

---

## Agent coverage matrix

| Tool | Claude | Cline | Codex | Gemini |
| --- | --- | --- | --- | --- |
| Ponytail | ✅ **−10.3% measured** | ⚠️ instruction-only (the tier that fired 0%) | ⚪ plugin-supported, unmeasured | ⚪ plugin-supported, unmeasured |
| CodeGraph | ✅ **−60% cost measured** | 🚫 unsupported | ⚪ supported, unmeasured | ⚪ supported, unmeasured |
| Headroom | ⚪ no A/B | ⚪ proxy, unmeasured | ⚪ proxy, unmeasured | ⚪ proxy, unmeasured |
| Caveman (skill) | ⚠️ **8.5% measured** | ⚪ unmeasured | ⚪ unmeasured | ⚪ unmeasured |
| RTK | ❌ **+7.6% measured** | ⚪ rules-file tier, unmeasured | ⚪ rules-file tier, unmeasured | ⚪ hook tier, unmeasured |
| LLMLingua / RouteLLM | ✅ | ✅ | ✅ | ✅ |

Symbols here are **measurement status**, not evidence tiers: ✅ measured and
beneficial · ⚠️ measured but weak, or the integration path is suspect ·
❌ measured and harmful · ⚪ unmeasured · 🚫 unsupported. Caveman is still
**Tier A** evidence; it is the result that disappoints, not the method.

**Cline is a measurement desert.** No agent-specific tool has ever been
benchmarked on it. Only the agent-independent peer-reviewed tier can be
honestly recommended there.

## Two recurring patterns

**1. Preventing work beats compressing it.** Ponytail (−10.3%) and CodeGraph
(−60% cost) both *stop work from happening*. RTK and Caveman both squeeze
output after it exists, and both underdelivered by 5–10×.

**2. The counterfactual trap.** RTK and jCodeMunch both measure themselves
against "how much would have been sent" — a baseline the harness was never
going to send. This is the single check worth applying to any savings claim:
**are they measuring a real bill, or a hypothetical world?**

The consequence: proof quality is inversely proportional to how glamorous the
tool is. Prompt compression and model routing — unglamorous, peer-reviewed,
agent-agnostic — hold up at 20× and 85%. The viral binaries promising 60–90%
with zero config measured **+7.6%** and **8.5%**.

## Conflict of interest to disclose

JetBrains launched **JetBrains Context** in 2026-07 — the same month as both
debunking posts, and a direct competitor in the same space. Their methodology
is sound and their negative results well-evidenced; but a page claiming to be
about *reliable* sources must disclose that the debunker sells a rival
product. Both benchmarks also used one harness (SkillsBench), one model
(`claude-sonnet-5`), and forced/maximal activation.

## Corrections applied to this repo

This page used to contradict figures stated elsewhere. **Resolved
2026-08-05 and 2026-08-06**; logged here for traceability:

| Location | Problem | Now reads |
| --- | --- | --- |
| `solutions/tool-output-compression.md` | RTK "60–90%" presented as a bill reduction | Labelled a *text*-compression ratio; +7.6% warning added, plus the condition that the harness must not already truncate |
| `solutions/tool-output-compression.md` | Caveman library/skill conflation | The two artifacts split apart |
| `solutions/concise-output-prompting.md` | Attributed the *skill's* cross-agent coverage and ~75% to the `wilpel` library | Split into two rows with their measured figures |
| `setups/coding-setup-cline.md` | Asserted 60–90% specifically for Cline; the "RTK + Headroom + Caveman" trio | States plainly that nothing here was measured on Cline; trio demoted to a hypothesis, enable one at a time with measurement |
| `solutions/README.md`, `solutions/tool-search.md` | "Code Mode ~99.9%" | Corrected to Anthropic's 98.7% on **one** workflow; the 99.9% figure marked unsourced |
| Everywhere Headroom appears | Quoted "60–95%" | Now **~20% for coding agents**; 60–95% identified as the JSON figure |
| `setups/recommended-setup.md` | Caveman "output-token cut on agent traffic" | Measured 8.5% plus the $8.29-vs-$0.33 tail risk |
| `solutions/code-maps.md`, `CAUSE.md` | Unsourced "25–60K tokens" | Marked an unsourced estimate; CodeGraph added as the evidenced alternative |
| `README.md`, `SUMMARY.md`, `solutions/README.md` | No route to this page | Linked, with a warning to read it before trusting any percentage |
| All three `setups/` documents | Named only the tools that measured ~0 or negative (RTK, Caveman, Headroom); the two that measured beneficial were **absent entirely** | Ponytail and CodeGraph added to all three, ranked by evidence: `recommended-setup.md` adds both to Tier 2 + the reference stack with the subagent-bypass warning; `coding-setup-cline.md` states plainly that neither **reaches Cline** and gives the hand-rolled mechanism instead; `coding-setup-enterprise.md` gains a per-vendor add-on matrix |

## Sources

- [JetBrains — rtk Claude Code Token Savings: A Skill Trial Benchmark](https://blog.jetbrains.com/ai/2026/07/rtk-claude-code-token-savings/)
- [JetBrains — Speaking to AI Agents like Cavemen Saves 65% of Tokens. We Test.](https://blog.jetbrains.com/ai/2026/07/speak-to-ai-agents-like-cavemen-tosave-tokens/)
- [JetBrains — Ponytail Skill for Claude Code: Does It Really Cut Tokens](https://blog.jetbrains.com/ai/2026/07/ponytail-skill-claude-tested/)
- [JetBrains — Introducing JetBrains Context](https://blog.jetbrains.com/ai/2026/07/introducing-jetbrains-context-repository-intelligence-for-coding-agents/)
- [github.com/DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail)
- [github.com/colbymchenry/codegraph](https://github.com/colbymchenry/codegraph)
- [github.com/headroomlabs-ai/headroom](https://github.com/headroomlabs-ai/headroom)
- [github.com/rtk-ai/rtk](https://github.com/rtk-ai/rtk)
- [github.com/wilpel/caveman-compression](https://github.com/wilpel/caveman-compression)
- [github.com/oraios/serena](https://github.com/oraios/serena)
- [LLMLingua — arXiv:2310.05736 (EMNLP 2023)](https://arxiv.org/abs/2310.05736)
- [LongLLMLingua — arXiv:2310.06839](https://arxiv.org/abs/2310.06839)
- [RouteLLM — OpenReview](https://openreview.net/forum?id=8sSqNntaMr) · [LMSYS](https://www.lmsys.org/blog/2024-07-01-routellm/)
- [Anthropic — Code execution with MCP](https://www.anthropic.com/engineering/code-execution-with-mcp)
- [RTK + Codex CLI](https://codex.danielvaughan.com/2026/05/19/rtk-codex-cli-token-optimisation-shell-output-compression/) · [openai/codex#19001](https://github.com/openai/codex/issues/19001)
