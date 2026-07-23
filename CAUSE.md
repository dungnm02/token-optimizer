# Nguyên nhân gây tiêu tốn Token cao (Tiếng Việt)

Tài liệu này liệt kê các nguyên nhân đã xác định gây tiêu tốn token cao trong
**các agent và ứng dụng dựa trên LLM**. Các nguyên nhân này không phụ thuộc
vào nhà cung cấp (provider-agnostic): chúng áp dụng cho mọi stack (Claude,
GPT, Gemini, các model mã nguồn mở, v.v.) vì bắt nguồn từ chính cách các API
LLM hoạt động — request không trạng thái (stateless), tính phí theo token, và
context window có giới hạn. Các chi tiết riêng của từng nhà cung cấp chỉ xuất
hiện như ví dụ minh họa.

Mỗi nguyên nhân mô tả **nó là gì**, **tại sao nó làm tăng mức sử dụng
token**, và **cách nhận biết nó**. Liên kết đến tài liệu tương ứng trong
`solutions/` khi có giải pháp khắc phục.

> Trạng thái: tài liệu sống (living document). Danh mục dưới đây bao quát
> các *loại* nguyên nhân chính, được nhóm theo danh mục. Các mục riêng lẻ có
> thể được mở rộng với phân tích và số liệu đo lường sâu hơn theo thời gian.

## Bối cảnh: tại sao các nguyên nhân này mang tính phổ quát

Hầu như mọi API LLM đều có chung ba đặc điểm, và chính ba đặc điểm này tạo ra
mọi nguyên nhân trong danh mục này:

1. **Không trạng thái (Statelessness).** API không lưu bộ nhớ giữa các lần
   gọi — client phải gửi lại mọi thứ mà model cần biết trong mỗi request.
   Hễ "mọi thứ" đó phình to thì mọi request về sau cũng phình to theo.
2. **Định giá theo token, bất đối xứng.** Input và output được tính phí theo
   token, với output thường đắt gấp 3–5× giá input, và input được cache
   thường chỉ bằng ~10–25% giá input (khi nhà cung cấp hỗ trợ caching).
3. **Context có giới hạn.** Context window là hữu hạn. Khi gần chạm giới
   hạn, hệ thống buộc phải cắt bớt, tóm tắt, hoặc thất bại — và mọi thứ nằm
   bên trong nó đều bị tính phí trên mỗi lần gọi.

## Mẫu (Template) cho một mục mới

Sao chép mẫu này khi thêm một nguyên nhân mới.

```
### <Tên nguyên nhân>

**Tóm tắt:** Mô tả nguyên nhân trong một câu.

**Tại sao nó tiêu tốn token:** Giải thích cơ chế làm tăng mức sử dụng token.

**Cách nhận biết:** Các triệu chứng, chỉ số, hoặc mẫu hình cho thấy nguyên
nhân này đang hiện diện.

**Giải pháp liên quan:** Liên kết đến (các) tệp liên quan trong `solutions/`.
```

---

## Danh mục

Các nguyên nhân được nhóm thành sáu danh mục:

| # | Danh mục | Vấn đề cốt lõi |
| --- | --- | --- |
| 1 | Lỗi caching | Trả giá đầy đủ cho các token tiền tố (prefix) lẽ ra có thể được phục vụ với giá cache-read |
| 2 | Tích lũy context | Lịch sử hội thoại tăng trưởng không giới hạn và được gửi lại mỗi lượt |
| 3 | Cách dùng tool | Các lệnh gọi tool và kết quả làm tràn ngập context bằng nội dung ít giá trị |
| 4 | Loại nội dung đắt đỏ | Hình ảnh, tài liệu, và dữ liệu truy xuất tốn kém hơn nhiều so với dự kiến |
| 5 | Chi tiêu phía sinh (generation) | Token output (suy luận + phản hồi) — loại token đắt nhất — bị chi tiêu quá mức |
| 6 | Lựa chọn kiến trúc | Thiết kế hệ thống nhân bội chi phí token trên nhiều request và agent |

## Giải pháp nằm ở đâu: Bên thứ ba so với Nhà cung cấp

Trước khi vào danh mục, cần định hướng nhanh một điều: một số nguyên nhân có
thể được giải quyết hoàn toàn bằng **công cụ bên thứ ba, không phụ thuộc
agent** (thường mã nguồn mở, di động qua các nhà cung cấp). Những nguyên nhân
khác lại phụ thuộc vào **năng lực và giá cả mà chỉ nhà cung cấp mới kiểm
soát được** — một công cụ phía client có thể *tận dụng* một mức giảm giá hay
nút điều chỉnh của nhà cung cấp, nhưng không bao giờ *tạo ra* được nó.

| Danh mục | Giải quyết được bằng công cụ bên thứ ba / kiến trúc của bạn | Phụ thuộc vào nhà cung cấp |
| --- | --- | --- |
| 1 Caching | Kỷ luật ổn định prompt, render tất định, kiểm thử byte trong CI; đo lường cache-hit (Langfuse/Helicone/LiteLLM); tái sử dụng prefix tự host (vLLM/SGLang) | Chính mức giảm giá: sự tồn tại của cache, giá đọc (~0.1×), TTL, breakpoint, phụ phí ghi |
| 2 Tích lũy context | Gần như hoàn toàn bên thứ ba: cắt tỉa/nén trong harness hoặc framework (LangGraph, LlamaIndex), registry hash, kho lưu bộ nhớ | Các API nén/quản lý context phía server chỉ là tiện lợi, không phải bắt buộc |
| 3 Dùng tool | Chủ yếu bên thứ ba: ngân sách output, proxy nén (RTK/Headroom), cắt gọt MCP, hạ tầng hướng sự kiện (Temporal, webhook) | Tải tool trì hoãn / tìm kiếm tool, gọi tool theo chương trình (Code Mode), định dạng gọi tool tiết kiệm token |
| 4 Loại nội dung | Giảm độ phân giải (sharp/Pillow), trích xuất (Docling/unstructured), reranker mã nguồn mở (BGE), bản đồ mã nguồn (aider/Repomix) | Công thức token thị giác và nút `detail`, API file/caching, endpoint `count_tokens` |
| 5 Phía sinh (generation) | Hợp đồng output trong prompt, định dạng edit dạng diff (Aider), retry có kiểm định (Instructor), nén output (Caveman) | Các nút điều chỉnh chính là của nhà cung cấp: ngân sách reasoning-effort/thinking, `verbosity`, chế độ structured-output |
| 6 Kiến trúc | Định tuyến/gateway (RouteLLM/LiteLLM/Portkey), semantic caching (GPTCache), framework điều phối, logic warm-then-fan, gói context | Bậc thang giá qua các tier model, tier batch giảm 50%, khả năng fine-tuning/distillation |

Quy tắc chung: **vệ sinh (hygiene) là việc của bạn, giảm giá là việc của họ.**
Mọi thứ giúp giữ token *không* lọt vào request (context, output tool, prompt,
trùng lặp) đều di động và giải quyết được bằng bên thứ ba. Còn các đòn bẩy
làm cho những token còn lại *rẻ hơn* — giá cache tiền tố, tier batch, tier
model rẻ hơn, nút effort/verbosity — đều do nhà cung cấp kiểm soát. Việc của
bạn ở phần đó chỉ là đủ điều kiện để nhận các ưu đãi ấy, và nhà cung cấp bạn
chọn sẽ giới hạn mức họ có thể cho bạn.

---

## 1. Lỗi Caching

Hầu hết các nhà cung cấp lớn đều cung cấp một dạng **caching prompt/prefix**
nào đó (breakpoint tường minh, prefix caching tự động, hoặc nội dung cache
đăng ký sẵn). Tất cả đều có chung cơ chế cốt lõi: cache khớp với một
**prefix** của request, và các token được cache sẽ được tính phí ở mức giảm
giá sâu. Các nguyên nhân dưới đây đều xoay quanh việc không đạt được mức
giảm giá đó.

> **Caching prefix không phải là lớp caching duy nhất.** Nó làm cho một
> *request lặp lại* rẻ hơn, nhưng vẫn chạy model. Có một dạng lãng phí khác,
> hoàn toàn tách biệt: toàn hệ thống (fleet) phải trả lời đi trả lời lại
> **các request gần giống hệt nhau** (cùng một câu hỏi hỗ trợ, cùng một truy
> vấn phân tích, cùng một prompt đánh giá) — mỗi lần đều tốn một lệnh gọi
> model đầy đủ, trong khi một cache ở *cấp độ phản hồi* (semantic) lẽ ra có
> thể bỏ qua hoàn toàn. Đây là một cơ hội khác, tách biệt với mức giảm giá
> prefix nói trên — được liệt kê là nguyên nhân 6.6; xem
> `solutions/semantic-caching.md`.

### 1.1 Không có caching prompt

**Tóm tắt:** Các request có prefix lớn, ổn định (system prompt, định nghĩa
tool, tài liệu chung) được gửi mà không sử dụng cơ chế caching của nhà cung
cấp — hoặc trên một provider/cấu hình mà caching không khả dụng.

**Tại sao nó tiêu tốn token:** API không trạng thái, nên toàn bộ prompt được
xử lý ở giá input đầy đủ trên mỗi request. Giá cache-read thường chỉ bằng
~10–25% giá input thông thường, nên nếu bỏ qua caching, bạn phải trả nhiều
hơn 4–10× cho mỗi prefix lặp lại, ở mọi lệnh gọi.

**Cách nhận biết:** Metadata sử dụng của nhà cung cấp báo cáo số token được
cache bằng 0 (ví dụ `cache_read_input_tokens` trên Anthropic, `cached_tokens`
trên OpenAI) trên mọi phản hồi, trong khi số token input thô vẫn lớn và
tương đối ổn định qua các request.

**Giải pháp liên quan:** `solutions/prompt-caching.md`

### 1.2 Yếu tố vô hiệu hóa cache âm thầm

**Tóm tắt:** Caching được cấu hình (hoặc tự động) nhưng không bao giờ trúng
(hit), vì một thứ gì đó làm thay đổi prefix của prompt trên mỗi request.

**Tại sao nó tiêu tốn token:** Caching prefix là khớp **chính xác từng
byte** — chỉ cần một byte thay đổi là vô hiệu hóa mọi thứ phía sau nó. Các
thủ phạm phổ biến: timestamp hoặc "ngày hiện tại" được nội suy vào system
prompt, ID request/UUID ở đầu nội dung, JSON được serialize với thứ tự khóa
không xác định, dữ liệu riêng của người dùng đặt ở đầu, và các phần prompt
có điều kiện. Kết quả là tệ nhất của cả hai thế giới: bất kỳ phụ phí ghi
cache nào cũng bị trả trên mỗi request mà không có lần đọc nào.

**Cách nhận biết:** Metadata sử dụng cho thấy các lần *ghi* cache (hoặc đơn
giản là không có token được cache) trên mỗi request dù các request trông
giống hệt nhau. So sánh (diff) các prompt đã render đầy đủ của hai request
liên tiếp sẽ lộ ra đoạn gây thay đổi.

**Giải pháp liên quan:** `solutions/prompt-caching.md`

### 1.3 Thay đổi giữa phiên làm mới lại cache

**Tóm tắt:** Chỉnh sửa system prompt, đổi model, hoặc thêm/xóa/sắp xếp lại
định nghĩa tool giữa cuộc hội thoại sẽ vô hiệu hóa toàn bộ prefix đã cache.

**Tại sao nó tiêu tốn token:** Định nghĩa tool và system prompt được render
ở ngay đầu request. Thay đổi bất cứ điều gì ở đầu sẽ buộc toàn bộ lịch sử
hội thoại phía sau phải được xử lý lại không cache — trong một phiên agentic
dài, đó có thể là hàng trăm nghìn token bị tính lại ở giá đầy đủ trong một
request. Cache cũng luôn gắn với model cụ thể: đổi model giữa phiên luôn bắt
đầu lại từ đầu (cold).

**Cách nhận biết:** Đột ngột tăng vọt input không được cache (và sụt giảm
input được cache) giữa phiên, tương quan với một thay đổi cấu hình: chuyển
chế độ, xây lại bộ tool, đổi model, hoặc "chỉnh sửa nhỏ" vào system prompt.

**Giải pháp liên quan:** `solutions/prompt-caching.md`,
`solutions/stable-prompt-architecture.md`

### 1.4 Cache hết hạn giữa các request

**Tóm tắt:** Lưu lượng dồn cục hoặc thưa thớt khiến cache hết hạn giữa các
request (thời gian sống điển hình từ vài phút đến ~1 giờ tùy nhà cung cấp và
cấu hình), nên mỗi đợt lại bắt đầu lại từ đầu (cold).

**Tại sao nó tiêu tốn token:** Một mục cache đã hết hạn phải được xử lý lại
— và trên các nhà cung cấp có phụ phí ghi cache tường minh, được ghi lại với
giá cao hơn. Lưu lượng có khoảng cách vượt quá thời gian sống của cache sẽ
không bao giờ hưởng lợi từ các request trước đó.

**Cách nhận biết:** Cache hit thành công trong một đợt, nhưng request đầu
tiên sau mỗi khoảng nghỉ cho thấy một lượt xử lý không cache (hoặc ghi
cache) đầy đủ.

**Giải pháp liên quan:** `solutions/prompt-caching.md` (TTL dài hơn,
pre-warming, lưu lượng giữ ấm)

---

## 2. Tích lũy context

> **Danh mục này là một vấn đề chất lượng, không chỉ là vấn đề chi phí.**
> Các nghiên cứu có kiểm soát ("context rot") trên 18 model frontier cho
> thấy mọi model đều suy giảm khi input tăng lên, và điều này xảy ra *trước
> khi* cửa sổ đầy. Ngân sách thực tế để giữ độ chính xác cao chỉ nằm trong
> khoảng 150–400K token, ngay cả trên các model có cửa sổ 2M token. Độ chính
> xác giảm nhanh nhất khi nhiễu tích lũy *tương đồng về mặt ngữ nghĩa* với
> câu trả lời — đúng là tình huống thường gặp trong một phiên lập trình dài
> đầy các bước khám phá gần đúng. Vì vậy, cắt tỉa lịch sử (xem bên dưới) vừa
> cải thiện độ chính xác vừa tiết kiệm token — hai động lực này cùng hướng
> về một phía.

### 2.1 Lịch sử hội thoại không giới hạn

**Tóm tắt:** Toàn bộ lịch sử tin nhắn được gửi lại trên mỗi lượt (API không
trạng thái) và không có gì từng cắt tỉa, tóm tắt, hoặc hết hạn nó.

**Tại sao nó tiêu tốn token:** Chi phí input mỗi lượt tăng gần như tuyến
tính theo độ dài hội thoại, nên tổng chi phí trên một phiên tăng theo **bậc
hai (quadratic)**. Một phiên agentic 100 lượt có thể tiêu phần lớn ngân sách
để gửi lại các lượt 1–99. Caching làm dịu bớt điều này (lịch sử được cache
thì rẻ) nhưng không loại bỏ nó, và mỗi lần cache trượt (miss) sẽ tính lại
toàn bộ lịch sử ở giá đầy đủ.

**Cách nhận biết:** Kích thước prompt tổng (input cached + uncached) tăng
đều đặn qua từng lượt; các request cuối phiên lớn hơn 10–100× so với các
request đầu; các phiên cuối cùng chạm giới hạn context window.

**Giải pháp liên quan:** `solutions/compaction.md`,
`solutions/context-editing.md`

### 2.2 Kết quả tool cũ vẫn được giữ trong lịch sử

**Tóm tắt:** Các output tool cũ (dump file, kết quả tìm kiếm, output lệnh)
vẫn còn trong bản ghi (transcript) rất lâu sau khi chúng không còn liên quan.

**Tại sao nó tiêu tốn token:** Trong các vòng lặp agentic, kết quả tool
thường chiếm phần lớn transcript. Một lần đọc file ở lượt 3 mà đã bị thay
thế ở lượt 20 vẫn được gửi lại (và tính phí lại) ở các lượt 21–100.

**Cách nhận biết:** Kiểm tra transcript cho thấy các khối kết quả tool lớn
có nội dung trùng lặp hoặc đã bị thay thế sau đó; tỷ trọng kết quả tool
trong prompt tiếp tục tăng trong khi tỷ trọng hữu ích thì không.

**Giải pháp liên quan:** `solutions/context-editing.md`
(cắt tỉa các kết quả tool / khối reasoning cũ)

### 2.3 Chèn context trùng lặp

**Tóm tắt:** Cùng một nội dung (một file, một schema, tài liệu truy xuất)
được chèn vào cuộc hội thoại nhiều lần.

**Tại sao nó tiêu tốn token:** Mỗi lần chèn được tính phí độc lập, và mỗi
bản sao sau đó tiếp tục tồn tại trong lịch sử suốt phần còn lại của phiên
(nguyên nhân 2.1 làm trầm trọng thêm điều này).

**Cách nhận biết:** Đọc lại cùng một file nhiều lần qua các lượt; pipeline
truy xuất gắn lại cùng top-k tài liệu vào mỗi tin nhắn người dùng; nội dung
cấp hệ thống được dán vào lượt người dùng "để đảm bảo model nhìn thấy nó".

**Giải pháp liên quan:** `solutions/context-hygiene.md`

---

## 3. Cách dùng Tool

### 3.1 Output tool quá lớn

**Tóm tắt:** Tool trả về nhiều nội dung hơn nhiều so với những gì model cần
— toàn bộ file thay vì phần liên quan, toàn bộ phản hồi API thay vì các
trường cần thiết, danh sách không phân trang.

**Tại sao nó tiêu tốn token:** Mỗi byte của kết quả tool là token input
trên request tiếp theo *và trên mọi request sau đó* khi nó vẫn còn trong
lịch sử. Một lần đọc file 5.000 dòng không lọc sẽ tốn số token của nó hàng
chục lần trong một phiên dài.

**Cách nhận biết:** Các kết quả tool riêng lẻ lên đến hàng chục nghìn token;
các kết quả tool mà model chỉ rõ ràng sử dụng một phần nhỏ.

**Giải pháp liên quan:** `solutions/tool-output-budgets.md`,
`solutions/tool-output-compression.md`

### 3.2 Nhiều round-trip thay vì kết hợp (composition)

**Tóm tắt:** Nhiều lệnh gọi tool tuần tự nơi mỗi kết quả trung gian đều chảy
qua context của model, dù model chỉ cần câu trả lời cuối cùng.

**Tại sao nó tiêu tốn token:** Mỗi round-trip gửi lại lịch sử đang lớn
dần và đưa thêm một kết quả trung gian vào đó. Ba lần tra cứu nối tiếp
(profile → đơn hàng → tồn kho) tốn ba lượt xử lý context đầy đủ, và dữ liệu
trung gian thường không bao giờ cần lại nữa.

**Cách nhận biết:** Chuỗi dài các lệnh gọi tool nhỏ cho mỗi request người
dùng; transcript đầy dữ liệu trung gian mà câu trả lời cuối không tham
chiếu đến.

**Giải pháp liên quan:** `solutions/tool-composition.md`
(điều phối tool thực thi bằng code, kết hợp phía server, gộp lô)

### 3.3 Vòng lặp thử lại và polling

**Tóm tắt:** Các lệnh gọi tool thất bại được thử lại với cùng context, hoặc
agent poll trạng thái bên ngoài ("kiểm tra lại trong vòng lặp") với một
request model đầy đủ cho mỗi lần poll.

**Tại sao nó tiêu tốn token:** Mỗi lần thử lại/poll đều tính lại toàn bộ
prompt. Một vòng lặp poll 10 lần trên context 100K token tiêu tốn 1M token
input chỉ để biết "chưa xong" tới chín lần. Một biến thể tương tự trong bối
cảnh agentic là **doom loop**: model lặp đi lặp lại một bản sửa lỗi thất bại
(sửa → test → thất bại → sửa tương tự), mỗi vòng đều tính phí lại context
ngày càng lớn rồi cộng thêm một lần thất bại nữa vào đó — chi phí cứ tăng
dồn trong khi tiến độ không nhích lên.

**Cách nhận biết:** Các đợt request gần giống hệt nhau trong log sử dụng;
các kết quả tool bị lỗi lặp lại với input không đổi; các mẫu hình
sleep-and-check được triển khai *thông qua* model thay vì *xung quanh* nó;
nhiều diff/thất bại test tương tự liên tiếp trong một phiên (giới hạn số
lần thử trong harness và buộc lập lại kế hoạch hoặc leo thang thay thế).

**Giải pháp liên quan:** `solutions/event-driven-waiting.md`

### 3.4 Quá nhiều schema tool được tải sẵn từ đầu

**Tóm tắt:** Mọi định nghĩa tool mà ứng dụng sở hữu đều được đưa vào mỗi
request, dù chỉ một vài trong số đó liên quan đến từng tác vụ.

**Tại sao nó tiêu tốn token:** Schema tool được serialize vào mỗi request.
Hàng trăm định nghĩa JSON-schema có thể thêm hàng chục nghìn token chi phí
cố định cho mỗi lệnh gọi — và trên các nhà cung cấp có prefix caching, bất
kỳ thay đổi nào với tập hợp đó đều vô hiệu hóa toàn bộ cache (nguyên nhân
1.3). Điều này đặc biệt phổ biến trong các thiết lập kiểu MCP nơi toàn bộ
tool server được gắn vào một cách trọn gói.

**Cách nhận biết:** Khoảng cách lớn giữa token input của một request có kèm
tool so với cùng request không kèm tool; danh sách tool lớn hơn nhiều so
với tập hợp thực sự được gọi.

**Giải pháp liên quan:** `solutions/tool-search.md` (khám phá tool động /
tải trì hoãn)

---

## 4. Loại nội dung đắt đỏ

### 4.1 Hình ảnh độ phân giải đầy đủ

**Tóm tắt:** Hình ảnh được gửi ở độ phân giải gốc/cao trong khi tác vụ
không cần độ trung thực đó.

**Tại sao nó tiêu tốn token:** Input thị giác được token hóa theo diện
tích/tile trên mọi nhà cung cấp lớn, nên chi phí token tỷ lệ với độ phân
giải — một hình ảnh độ phân giải đầy đủ có thể tốn vài nghìn token trong
khi một hình đã giảm độ phân giải chỉ tốn vài trăm. Các vòng lặp nhiều
screenshot (computer/browser use) nhân điều này lên theo từng bước, và hình
ảnh cũng tồn tại trong lịch sử (nguyên nhân 2.1).

**Cách nhận biết:** Các request có hình ảnh với số token input cao hơn
nhiều so với độ dài văn bản; chi phí mỗi bước của các vòng lặp computer-use
bị chi phối bởi screenshot.

**Giải pháp liên quan:** `solutions/image-downsampling.md`

### 4.2 Đổ nguyên tài liệu (PDF / RAG truy xuất quá rộng)

**Tóm tắt:** Toàn bộ tài liệu hoặc kết quả truy xuất quá rộng được đính kèm
khi chỉ một phần nhỏ là liên quan.

**Tại sao nó tiêu tốn token:** Một PDF vài trăm trang hoặc truy xuất top-20
chunk có thể thêm hàng chục hoặc hàng trăm nghìn token input mỗi request.
Không có caching, điều đó bị tính lại cho mỗi câu hỏi; không có cắt tỉa, nó
tồn tại trong lịch sử mãi mãi.

**Cách nhận biết:** Các luồng hỏi-đáp nơi mỗi câu hỏi gửi lại cùng tài liệu
lớn; truy xuất được cấu hình cho recall (k lớn, chunk lớn) với câu trả lời
chỉ trích dẫn một phần rất nhỏ những gì đã gửi.

**Giải pháp liên quan:** `solutions/document-reuse.md`,
`solutions/retrieval-tuning.md`, `solutions/code-maps.md` (cho codebase)

### 4.3 Nội dung tốn kém khi tokenize

**Tóm tắt:** Nội dung tokenize không hiệu quả — code đậm đặc, JSON đã
minify, blob base64, nhiều script không phải tiếng Anh — hoặc một thay đổi
model/tokenizer khiến cùng văn bản tính ra số token cao hơn.

**Tại sao nó tiêu tốn token:** Số token phụ thuộc vào tokenizer cụ thể, và
các tokenizer khác nhau giữa các nhà cung cấp *và giữa các thế hệ model
trong cùng một nhà cung cấp* (các bản nâng cấp đã làm thay đổi số token hơn
30% cho cùng một văn bản). Log nguyên văn, bundle đã minify, và payload
base64 nằm trong số những thứ tệ nhất tính trên mỗi đơn vị thông tin hữu ích
— chỉ riêng base64 đã làm phình to số byte thêm ~33% trước khi tokenization
bắt đầu.

**Cách nhận biết:** Ngân sách được hiệu chỉnh trên một model đột nhiên bị
cắt bớt hoặc tốn kém hơn sau khi đổi model; endpoint đếm token của nhà cung
cấp cho ra số cao hơn nhiều so với ước lượng chars/4 trên nội dung thực tế
của bạn.

**Giải pháp liên quan:** `solutions/token-counting.md`
(hiệu chỉnh lại bằng bộ đếm của nhà cung cấp cho đúng model mục tiêu)

---

## 5. Chi tiêu phía sinh (Generation-Side)

### 5.1 Token reasoning/thinking

**Tóm tắt:** Các model có khả năng suy luận tạo ra (và bị tính phí) các
token chuỗi suy nghĩ (chain-of-thought) trước câu trả lời hiển thị — thường
bị ẩn hoặc tóm tắt trong phản hồi, nhưng vẫn phải trả tiền dù sao.

**Tại sao nó tiêu tốn token:** Token output thường đắt gấp 3–5× giá input,
và reasoning được tính phí như output trên mọi nhà cung cấp lớn. Các thiết
lập reasoning-effort cao áp dụng đồng loạt sẽ chi tiêu suy luận sâu cho các
route không cần đến nó; một số nhà cung cấp bật reasoning theo mặc định,
nên chi phí xuất hiện mà không cần bật tùy chọn nào.

**Cách nhận biết:** Số token output báo cáo vượt xa độ dài phản hồi hiển
thị; các route đơn giản (phân loại, tra cứu) cho thấy cùng mức chi tiêu
output như các route phức tạp.

**Giải pháp liên quan:** `solutions/reasoning-effort-tuning.md`
(thiết lập effort/ngân sách theo từng route)

### 5.2 Output dài dòng

**Tóm tắt:** Model tạo ra nhiều văn xuôi hơn cần thiết — phát lại toàn bộ
file thay vì diff, tường thuật từng bước, lời mở đầu và tóm tắt dài dòng.

**Tại sao nó tiêu tốn token:** Output là loại token đắt nhất, và trong các
phiên nhiều lượt, mỗi câu trả lời dài dòng cũng trở thành *input* trên mọi
lượt sau đó — sự dài dòng bị tính phí một lần dưới dạng output và N lần
dưới dạng lịch sử.

**Cách nhận biết:** Các phản hồi lặp lại câu hỏi, in lại code không đổi,
hoặc tóm tắt những gì vừa được hiển thị; số token output không tương xứng
với lượng thông tin được cung cấp.

**Giải pháp liên quan:** `solutions/concise-output-prompting.md`,
`solutions/diff-based-edits.md`

### 5.3 Chu kỳ cắt bớt và thử lại

**Tóm tắt:** Giới hạn token output đặt quá thấp làm cắt bớt phản hồi giữa
chừng sinh, và ứng dụng thử lại toàn bộ request.

**Tại sao nó tiêu tốn token:** Mỗi lần thử lại tính lại toàn bộ input cộng
với phần output bị lãng phí. Hai lần thử bị cắt bớt trước khi thành công ở
lần thứ ba tốn khoảng 3× input và ~2 câu trả lời output bị bỏ đi. Động lực
tương tự áp dụng cho các thất bại kiểm định schema trên structured output —
mỗi lần sinh không hợp lệ kích hoạt một request lại đầy đủ.

**Cách nhận biết:** Lý do dừng length/max-token trong log theo sau bởi các
request thử lại gần giống hệt nhau; các thất bại phân tích structured-output
dẫn vào một vòng lặp thử lại.

**Giải pháp liên quan:** `solutions/output-cap-sizing.md`

---

## 6. Lựa chọn kiến trúc

### 6.1 Subagent cold-start

**Tóm tắt:** Các subagent được sinh ra cho từng tác vụ con mà không có
cache hay bàn giao context chung, mỗi cái tự suy ra lại những gì agent cha
đã biết.

**Tại sao nó tiêu tốn token:** Mỗi lần sinh gửi lại (và thường khám phá lại
qua các lệnh gọi tool) context mà orchestrator đã trả tiền. Các lệnh gọi
fork xây lại system prompt / danh sách tool / lựa chọn model với bất kỳ khác
biệt nào cũng bỏ lỡ hoàn toàn cache prefix của agent cha.

**Cách nhận biết:** Transcript của subagent mở đầu bằng đúng những khám phá
mà agent cha đã làm; mẫu hình fan-out nơi N worker đều phải trả giá cho một
context lạnh đầy đủ; cache-hit bằng 0 trên các request fork dù agent cha
đang "ấm" (warm).

**Giải pháp liên quan:** `solutions/subagent-context-handoff.md`,
`solutions/prompt-caching.md` (các fork phải tái sử dụng chính xác prefix
của agent cha)

### 6.2 Model quá lớn so với tác vụ

**Tóm tắt:** Mọi route đều dùng model lớn nhất, kể cả những route mà một
tier nhỏ hơn vẫn phục vụ tốt.

**Tại sao nó tiêu tốn token:** Đây thuần túy là một hệ số nhân *chi phí*
chứ không phải *số lượng token*, nhưng nó khuếch đại mọi nguyên nhân khác:
các model tier frontier trên các nhà cung cấp có giá cao hơn khoảng 5–25×
mỗi token so với các model tier nhỏ cùng dòng, nên cùng một sự lãng phí
token lại tốn kém hơn nhiều đến vậy.

**Cách nhận biết:** Lưu lượng phân loại/trích xuất/định dạng đơn giản chạy
qua model tier cao nhất; không có lớp định tuyến; không sử dụng tier
async/batch giảm giá của nhà cung cấp cho công việc không nhạy cảm về độ
trễ.

**Giải pháp liên quan:** `solutions/model-routing.md`,
`solutions/batch-processing.md`, `solutions/semantic-caching.md`

### 6.3 Fan-out cache lạnh đồng thời

**Tóm tắt:** N request song song với cùng một prefix được bắn ra cùng lúc
trước khi bất kỳ request nào kịp điền vào cache.

**Tại sao nó tiêu tốn token:** Một mục cache thường chỉ có thể đọc được sau
khi request đầu tiên đã được xử lý. Cả N request song song đều trả giá input
đầy đủ — không ai đọc được thứ mà những cái khác vẫn đang ghi.

**Cách nhận biết:** Các đợt fan-out nơi mọi request đều báo cáo 0 token
được cache dù chia sẻ chung một prefix, trong khi các lượt chạy tuần tự của
cùng các request đó cho thấy hit từ lần thứ hai trở đi.

**Giải pháp liên quan:** `solutions/fan-out-warming.md` (làm ấm bằng một
request, sau đó mới bắn phần còn lại)

### 6.4 Prompt và scaffolding quá quy định

**Tóm tắt:** System prompt mang theo scaffolding cũ — cập nhật tiến độ bắt
buộc, quy trình từng bước, vòng lặp xác minh "kiểm tra lại X", các bộ
few-shot dài — được tinh chỉnh cho các model cũ hơn hoặc khác.

**Tại sao nó tiêu tốn token:** Bản thân scaffolding là chi phí prompt cố
định trên mỗi lệnh gọi, và nó gây ra thêm output: tường thuật bắt buộc, các
lệnh gọi tool xác minh dư thừa, và giải thích lại những gì model hiện tại
sẽ không tự sinh ra. Các prompt tích lũy qua nhiều thế hệ model hiếm khi
được rà soát lại.

**Cách nhận biết:** Các phần prompt lặp lại hành vi mặc định của model hiện
tại ("sau mỗi 3 lệnh gọi tool, tóm tắt tiến độ"); output chứa các phần mang
tính nghi thức không ai đọc; ví dụ few-shot cho các hành vi mà model hiện đã
làm được mà không cần ví dụ.

**Giải pháp liên quan:** `solutions/prompt-de-scaffolding.md`

### 6.5 Cold-start giữa các phiên (không có kiến thức bền vững)

**Tóm tắt:** Mỗi phiên mới đều bắt đầu từ số không — agent khám phá lại
codebase hoặc lĩnh vực và suy luận lại những hiểu biết mà các phiên trước đã
trả tiền cho nó rồi.

**Tại sao nó tiêu tốn token:** Tính không trạng thái áp dụng *giữa các
phiên*, không chỉ bên trong chúng. Để định hướng trên một repo lớn, agent
phải thực hiện rất nhiều lệnh gọi tool lớn (thường tốn 25–60K token trước
khi công việc thực sự bắt đầu), và hóa đơn đó lặp lại cho mỗi phiên mới, mỗi
kỹ sư, mỗi agent trong hệ thống — cùng một kiến thức bị mua đi mua lại nhiều
lần.

**Cách nhận biết:** Transcript phiên mở đầu bằng cùng một chuỗi khám phá
(liệt kê → đọc → grep cùng các file cốt lõi); chi tiêu N lượt đầu tiên bị
chi phối bởi các lần đọc mà một phiên trước đã từng làm; không có gì học
được trong một phiên khả dụng cho phiên tiếp theo.

**Giải pháp liên quan:** `solutions/code-maps.md` (bản đồ/gói context đã
lưu sẵn), `solutions/subagent-context-handoff.md` (kho lưu artifact),
`solutions/compaction.md` (mang tóm tắt sang các ranh giới phiên)

### 6.6 Request trùng lặp trên toàn hệ thống (fleet)

**Tóm tắt:** Cùng một request hoặc gần giống hệt nhau được trả lời nhiều
lần trên các người dùng, phiên, hoặc lượt chạy theo lịch — mỗi lần đều ở giá
model đầy đủ.

**Tại sao nó tiêu tốn token:** Caching prefix giảm giá cho *prompt* lặp lại
nhưng vẫn chạy model; không có tái sử dụng ở cấp độ phản hồi, N câu hỏi gần
giống hệt nhau tốn N lần sinh đầy đủ. Hỏi-đáp hỗ trợ/tài liệu, các câu hỏi
phân tích lặp lại, và các bộ đánh giá (eval) chạy lại là những hình mẫu điển
hình.

**Cách nhận biết:** Log sử dụng gom cụm thành các nhóm prompt gần giống hệt
nhau với câu trả lời gần giống hệt nhau; độ trùng lặp truy vấn cao trong lưu
lượng chủ yếu đọc; các job theo lịch tái sinh output hầu như không đổi.

**Giải pháp liên quan:** `solutions/semantic-caching.md`;
`solutions/batch-processing.md` cho biến thể chạy lại theo lịch

---

## Cẩm nang Đo lường

Để quy chi phí về đúng nguyên nhân, hãy dựa vào metadata sử dụng đi kèm mỗi
phản hồi — đó chính là sự thật nền tảng (ground truth). Tên trường khác nhau
tùy nhà cung cấp, nhưng cùng bốn đại lượng này tồn tại ở hầu hết mọi nơi:

| Đại lượng | Giá tương đối điển hình | Tên trường ví dụ |
| --- | --- | --- |
| Token input không cache | 1× input | Anthropic `input_tokens`; OpenAI `prompt_tokens` trừ `cached_tokens`; Gemini `prompt_token_count` trừ `cached_content_token_count` |
| Token input đã cache | ~0.1–0.25× input | Anthropic `cache_read_input_tokens`; OpenAI `prompt_tokens_details.cached_tokens`; Gemini `cached_content_token_count` |
| Token ghi cache (nơi có phụ phí) | ~1.25–2× input | Anthropic `cache_creation_input_tokens` (nơi khác tính phí ghi theo giá input thông thường) |
| Token output, **bao gồm cả reasoning ẩn** | ~3–5× input | Anthropic `output_tokens`; OpenAI `completion_tokens` (+ chi tiết `reasoning_tokens`); Gemini `candidates_token_count` + `thoughts_token_count` |

Hai quy tắc phổ quát:

- **Tổng kích thước prompt = input không cache + đã cache (+ ghi cache).**
  Nếu chỉ đánh giá chi tiêu dựa vào trường không cache, bạn sẽ quy chi phí
  thiếu hoặc thừa.
- **Đếm token bằng bộ đếm riêng của nhà cung cấp cho đúng model mục tiêu.**
  Tokenizer là đặc thù theo từng model; mượn bộ đếm của nhà cung cấp khác
  (hoặc của thế hệ model khác) sẽ luôn cho kết quả sai một cách có hệ thống.

---

# Causes of High Token Consumption

This document catalogs the identified causes of high token consumption in
**LLM-based agents and applications**. The causes are provider-agnostic: they
apply to any stack (Claude, GPT, Gemini, open-weight models, etc.) because
they stem from how LLM APIs fundamentally work — stateless requests, priced
per token, with a bounded context window. Provider-specific details appear
only as illustrative examples.

Each cause describes **what it is**, **why it inflates token usage**, and
**how to recognize it**. Link to a matching document in `solutions/` when a
mitigation exists.

> Status: living document. The catalog below covers the major cause *types*,
> grouped by category. Individual entries may be expanded with deeper
> analysis and measurements over time.

## Background: why these causes are universal

Three properties shared by essentially all LLM APIs generate every cause in
this catalog:

1. **Statelessness.** The API keeps no memory between calls — the client
   re-sends everything the model should know on every request. Anything that
   grows the "everything" grows every future request.
2. **Per-token pricing, asymmetric.** Input and output are billed per token,
   with output typically 3–5× the input price, and cached input typically
   ~10–25% of the input price (where the provider offers caching).
3. **Bounded context.** The context window is finite; approaching it forces
   truncation, summarization, or failure — and everything inside it is billed
   on every call.

## Entry Template

Copy this template when adding a new cause.

```
### <Cause name>

**Summary:** One-sentence description of the cause.

**Why it consumes tokens:** Explain the mechanism that drives up token usage.

**How to recognize it:** Symptoms, metrics, or patterns that indicate this
cause is present.

**Related solution(s):** Link to the relevant file(s) under `solutions/`.
```

---

## Catalog

The causes are grouped into six categories:

| # | Category | Core problem |
| --- | --- | --- |
| 1 | [Caching failures](#1-caching-failures) | Paying full price for prefix tokens that could be served at cache-read rates |
| 2 | [Context accumulation](#2-context-accumulation) | Conversation history grows without bound and is re-sent every turn |
| 3 | [Tool usage patterns](#3-tool-usage-patterns) | Tool calls and results flood the context with low-value content |
| 4 | [Expensive content types](#4-expensive-content-types) | Images, documents, and retrieved data cost far more than expected |
| 5 | [Generation-side spend](#5-generation-side-spend) | Output tokens (reasoning + response) — the most expensive tokens — are overspent |
| 6 | [Architectural choices](#6-architectural-choices) | System design multiplies token cost across requests and agents |

## Where the Fixes Live: Third-Party vs Provider

A quick orientation before the catalog: some causes can be attacked
entirely with **agent-agnostic third-party tooling** (usually open source,
portable across providers), while others hinge on **capabilities and
pricing only the provider controls** — a client-side tool can *exploit* a
provider discount or dial, but it can never *create* one.

| Category | Solvable with third-party tools / your architecture | Depends on the provider |
| --- | --- | --- |
| 1 Caching | Prompt-stability discipline, deterministic rendering, CI byte-tests; cache-hit telemetry (Langfuse/Helicone/LiteLLM); self-hosted prefix reuse (vLLM/SGLang) | The discount itself: cache existence, read pricing (~0.1×), TTLs, breakpoints, write surcharges |
| 2 Context accumulation | Almost fully third-party: pruning/compaction in the harness or framework (LangGraph, LlamaIndex), hash registries, memory stores | Server-side compaction/context-management APIs are a convenience, not a requirement |
| 3 Tool usage | Mostly third-party: output budgets, compression proxies (RTK/Headroom), MCP trimming, event-driven infra (Temporal, webhooks) | Deferred tool loading / tool search, programmatic tool calling (Code Mode), token-efficient tool-call formats |
| 4 Content types | Downscaling (sharp/Pillow), extraction (Docling/unstructured), open rerankers (BGE), code maps (aider/Repomix) | Vision token formulas and `detail` dials, file/caching APIs, `count_tokens` endpoints |
| 5 Generation-side | Output contracts in prompts, diff edit formats (Aider), validation-aware retries (Instructor), output compression (Caveman) | The main dials are provider knobs: reasoning-effort/thinking budgets, `verbosity`, structured-output modes |
| 6 Architecture | Routing/gateways (RouteLLM/LiteLLM/Portkey), semantic caching (GPTCache), orchestration frameworks, warm-then-fan logic, context packs | The price ladder across model tiers, the 50% batch tier, fine-tuning/distillation availability |

Rule of thumb: **hygiene is yours, discounts are theirs.** Everything that
keeps tokens *out* of requests (context, tool output, prompts, duplication)
is portable and third-party-solvable; the levers that make remaining tokens
*cheaper* (prefix caching rates, batch tier, cheaper model tiers,
effort/verbosity dials) are provider-controlled — your job there is to
qualify for them, and a provider choice caps how much they can give you.

---

## 1. Caching Failures

Most major providers offer some form of **prompt/prefix caching** (explicit
breakpoints, automatic prefix caching, or pre-registered cached content).
All of them share the same core mechanic: the cache matches a **prefix** of
the request, and cached tokens are billed at a steep discount. These causes
are about failing to earn that discount.

> **Prefix caching is not the only caching layer.** It makes a *repeated
> request* cheaper, but still runs the model. A separate, orthogonal waste
> is answering **near-identical requests repeatedly across a fleet** (the
> same support question, the same analytic query, the same eval prompt) —
> each one a full model call that a *response-level* (semantic) cache could
> skip entirely. That's a distinct opportunity from the prefix discount
> below — cataloged as cause 6.6; see `solutions/semantic-caching.md`.

### 1.1 No prompt caching at all

**Summary:** Requests with a large, stable prefix (system prompt, tool
definitions, shared documents) are sent without using the provider's caching
mechanism — or on a provider/config where caching isn't available.

**Why it consumes tokens:** The API is stateless, so the full prompt is
processed at full input price on every request. With cache reads typically
priced at ~10–25% of normal input, skipping caching means paying 4–10× more
for every repeated prefix, on every single call.

**How to recognize it:** The provider's usage metadata reports zero cached
tokens (e.g. `cache_read_input_tokens` on Anthropic, `cached_tokens` on
OpenAI) on every response, while raw input token counts stay large and
roughly constant across requests.

**Related solution(s):** `solutions/prompt-caching.md`

### 1.2 Silent cache invalidators

**Summary:** Caching is configured (or automatic) but never hits, because
something mutates the prompt prefix on every request.

**Why it consumes tokens:** Prefix caching is a **byte-exact prefix match** —
a single changed byte invalidates everything after it. Common culprits:
timestamps or "current date" interpolated into the system prompt, request
IDs/UUIDs early in the content, JSON serialized with non-deterministic key
order, per-user data placed at the front, and conditional prompt sections.
The result is the worst of both worlds: any cache-write surcharge is paid on
every request with zero reads.

**How to recognize it:** Usage metadata shows cache *writes* (or simply no
cached tokens) on every request despite requests looking identical. Diffing
the fully rendered prompts of two consecutive requests exposes the mutating
fragment.

**Related solution(s):** `solutions/prompt-caching.md`

### 1.3 Mid-session changes that rebuild the cache

**Summary:** Editing the system prompt, switching models, or adding/
removing/reordering tool definitions mid-conversation invalidates the entire
cached prefix.

**Why it consumes tokens:** Tool definitions and the system prompt render at
the very front of the request. Changing anything at the front forces the
whole conversation history behind it to be re-processed uncached — in a long
agentic session that can be hundreds of thousands of tokens re-billed at
full price in one request. Caches are also model-scoped everywhere: swapping
models mid-session always starts cold.

**How to recognize it:** A sudden spike of uncached input (and collapse of
cached input) mid-session, correlated with a config change: mode switch,
tool set rebuild, model swap, or a "small edit" to the system prompt.

**Related solution(s):** `solutions/prompt-caching.md`,
`solutions/stable-prompt-architecture.md`

### 1.4 Cache lifetime expiry between requests

**Summary:** Bursty or infrequent traffic lets the cache expire between
requests (typical lifetimes range from a few minutes to ~1 hour depending on
provider and configuration), so each burst starts cold.

**Why it consumes tokens:** An expired entry must be re-processed — and on
providers with an explicit cache-write surcharge, re-written at a premium.
Traffic whose gaps exceed the cache lifetime never benefits from earlier
requests.

**How to recognize it:** Cache hits succeed within a burst, but the first
request after each idle gap shows a full uncached (or cache-write) pass.

**Related solution(s):** `solutions/prompt-caching.md` (longer
TTLs, pre-warming, keep-alive traffic)

---

## 2. Context Accumulation

> **This category is a quality problem, not only a cost problem.**
> Controlled studies ("context rot") across 18 frontier models find every
> one degrades as input grows — and well *before* the window fills:
> practical high-accuracy budgets land around 150–400K tokens even on
> 2M-token models, and accuracy falls fastest when the accumulated noise is
> *semantically similar* to the answer (exactly the case in a long coding
> session full of near-miss exploration). So trimming history (below) buys
> accuracy as well as tokens — the two motivations point the same way.

### 2.1 Unbounded conversation history

**Summary:** The full message history is re-sent on every turn (the API is
stateless) and nothing ever trims, summarizes, or expires it.

**Why it consumes tokens:** Input cost per turn grows roughly linearly with
conversation length, so total cost across a session grows **quadratically**.
A 100-turn agentic session can spend most of its budget re-sending turns
1–99. Caching softens this (cached history is cheap) but does not remove it,
and every cache miss re-bills the whole history at full price.

**How to recognize it:** Total prompt size (cached + uncached input) climbs
steadily turn over turn; late-session requests are 10–100× larger than early
ones; sessions eventually hit the context-window limit.

**Related solution(s):** `solutions/compaction.md`,
`solutions/context-editing.md`

### 2.2 Stale tool results kept in history

**Summary:** Old tool outputs (file dumps, search results, command output)
remain in the transcript long after they stopped being relevant.

**Why it consumes tokens:** In agentic loops, tool results usually dominate
the transcript. A file read at turn 3 that was superseded at turn 20 is
still re-sent (and re-billed) on turns 21–100.

**How to recognize it:** Inspecting the transcript shows large tool-result
blocks whose content is duplicated or superseded later; the tool-result
share of the prompt keeps growing while the useful share doesn't.

**Related solution(s):** `solutions/context-editing.md`
(pruning old tool results / reasoning blocks)

### 2.3 Duplicate context injection

**Summary:** The same content (a file, a schema, retrieved documents) is
injected into the conversation multiple times.

**Why it consumes tokens:** Each injection is billed independently, and each
copy then rides along in the history for the rest of the session (cause 2.1
compounds it).

**How to recognize it:** Repeated re-reads of the same file across turns;
retrieval pipelines that re-attach the same top-k documents on every user
message; system-level content pasted into user turns "to make sure the model
sees it".

**Related solution(s):** `solutions/context-hygiene.md`

---

## 3. Tool Usage Patterns

### 3.1 Oversized tool outputs

**Summary:** Tools return far more content than the model needs — whole
files instead of relevant sections, full API responses instead of the needed
fields, unpaginated listings.

**Why it consumes tokens:** Every byte of a tool result is input tokens on
the next request *and on every subsequent request* while it stays in
history. An unfiltered 5,000-line file read costs its tokens dozens of times
over a long session.

**How to recognize it:** Individual tool results in the tens of thousands of
tokens; tool results of which the model visibly uses only a small fraction.

**Related solution(s):** `solutions/tool-output-budgets.md`,
`solutions/tool-output-compression.md`

### 3.2 Chatty round-trips instead of composition

**Summary:** Many sequential tool calls where each intermediate result flows
through the model's context, even though the model only needs the final
answer.

**Why it consumes tokens:** Each round trip re-sends the growing history and
lands another intermediate result in it. Three chained lookups
(profile → orders → inventory) cost three full context passes, and the
intermediate data is usually never needed again.

**How to recognize it:** Long chains of small tool calls per user request;
transcripts full of intermediate data the final answer doesn't reference.

**Related solution(s):** `solutions/tool-composition.md`
(code-executed tool orchestration, server-side composition, batching)

### 3.3 Retry and polling loops

**Summary:** Failed tool calls retried with the same context, or agents
polling an external state ("check again in a loop") with a full model
request per poll.

**Why it consumes tokens:** Every retry/poll re-bills the entire prompt.
A 10-iteration poll loop on a 100K-token context spends 1M input tokens to
learn "not done yet" nine times. The agentic cousin is the **doom loop**:
the model repeatedly attempts a failing fix (edit → test → fail → similar
edit), re-billing the ever-growing context on each lap and adding another
failed attempt to it — cost compounds while progress stays flat.

**How to recognize it:** Bursts of near-identical requests in usage logs;
repeated errored tool results with unchanged inputs; sleep-and-check
patterns implemented *through* the model instead of *around* it; many
consecutive similar diffs/test failures in one session (cap attempts in the
harness and force a replan or escalation instead).

**Related solution(s):** `solutions/event-driven-waiting.md`

### 3.4 Too many tool schemas loaded upfront

**Summary:** Every tool definition the application owns is included in every
request, even though only a few are relevant per task.

**Why it consumes tokens:** Tool schemas are serialized into every request.
Hundreds of JSON-schema definitions can add tens of thousands of tokens of
fixed overhead per call — and on providers with prefix caching, any change
to the set invalidates the whole cache (cause 1.3). This is especially
common in MCP-style setups where entire tool servers are attached wholesale.

**How to recognize it:** A large gap between input tokens on a
tools-included request vs. the same request without tools; a tool list much
larger than the set actually invoked.

**Related solution(s):** `solutions/tool-search.md` (dynamic
tool discovery / deferred loading)

---

## 4. Expensive Content Types

### 4.1 Full-resolution images

**Summary:** Images sent at native/high resolution when the task doesn't
need the fidelity.

**Why it consumes tokens:** Vision inputs are tokenized by area/tiles on
every major provider, so token cost scales with resolution — a
full-resolution image can cost several thousand tokens where a downscaled
one costs a few hundred. Screenshot-heavy loops (computer/browser use)
multiply this per step, and images also persist in history (cause 2.1).

**How to recognize it:** Image-bearing requests with input token counts far
above the text length; per-step cost of computer-use loops dominated by the
screenshot.

**Related solution(s):** `solutions/image-downsampling.md`

### 4.2 Whole-document dumping (PDF / RAG over-retrieval)

**Summary:** Entire documents or over-broad retrieval results are attached
when only a small slice is relevant.

**Why it consumes tokens:** A few-hundred-page PDF or a top-20 chunk
retrieval can add tens or hundreds of thousands of input tokens per request.
Without caching, that is re-billed per question; without pruning, it rides
in history forever.

**How to recognize it:** Q&A flows where every question re-sends the same
large document; retrieval configured for recall (large k, large chunks)
with answers that cite a tiny fraction of what was sent.

**Related solution(s):** `solutions/document-reuse.md`,
`solutions/retrieval-tuning.md`, `solutions/code-maps.md` (for codebases)

### 4.3 Tokenizer-expensive content

**Summary:** Content that tokenizes inefficiently — dense code, minified
JSON, base64 blobs, many non-English scripts — or a model/tokenizer change
that counts the same text higher.

**Why it consumes tokens:** Token counts are tokenizer-specific, and
tokenizers differ across providers *and across model generations within a
provider* (upgrades have shifted counts by 30%+ for identical text).
Verbatim logs, minified bundles, and base64 payloads are among the worst
offenders per unit of useful information — base64 alone inflates byte count
by ~33% before tokenization even starts.

**How to recognize it:** Budgets calibrated on one model suddenly truncating
or costing more after a model swap; the provider's token-counting endpoint
showing counts far above chars/4 heuristics on your actual content.

**Related solution(s):** `solutions/token-counting.md`
(re-baseline with the provider's counter for the exact target model)

---

## 5. Generation-Side Spend

### 5.1 Reasoning/thinking tokens

**Summary:** Reasoning-capable models generate (billed) chain-of-thought
tokens before the visible answer — often hidden or summarized in the
response, but paid either way.

**Why it consumes tokens:** Output tokens are typically 3–5× the input
price, and reasoning is billed as output on every major provider. High
reasoning-effort settings applied uniformly spend deep reasoning on routes
that don't need it; some providers enable reasoning by default, so cost
appears without any opt-in.

**How to recognize it:** Reported output tokens far exceed the visible
response length; simple routes (classification, lookups) showing the same
output spend as complex ones.

**Related solution(s):** `solutions/reasoning-effort-tuning.md`
(per-route effort/budget settings)

### 5.2 Output verbosity

**Summary:** The model produces more prose than needed — re-emitting whole
files instead of diffs, narrating every step, long preambles and recaps.

**Why it consumes tokens:** Output is the most expensive token class, and in
multi-turn sessions every verbose answer also becomes *input* on all
subsequent turns — verbosity is billed once as output and N times as
history.

**How to recognize it:** Responses that restate the question, re-print
unchanged code, or summarize what was just shown; output token counts
disproportionate to the information delivered.

**Related solution(s):** `solutions/concise-output-prompting.md`,
`solutions/diff-based-edits.md`

### 5.3 Truncation-and-retry cycles

**Summary:** An output-token cap set too low truncates the response
mid-generation, and the application retries the whole request.

**Why it consumes tokens:** Each retry re-bills the full input plus the
wasted partial output. Two truncated attempts before a successful third cost
roughly 3× the input and ~2 answers' worth of discarded output. The same
dynamic applies to schema-validation failures on structured output — each
invalid generation triggers a full re-request.

**How to recognize it:** Length/max-token stop reasons in logs followed by
near-identical retry requests; structured-output parse failures feeding a
retry loop.

**Related solution(s):** `solutions/output-cap-sizing.md`

---

## 6. Architectural Choices

### 6.1 Cold-start subagents

**Summary:** Subagents are spawned per subtask with no shared cache or
context handoff, each re-deriving what the parent already knows.

**Why it consumes tokens:** Every spawn re-sends (and often re-discovers via
tool calls) context the orchestrator already paid for. Fork calls that
rebuild the system prompt / tool list / model choice with any difference
also miss the parent's prefix cache entirely.

**How to recognize it:** Subagent transcripts that open with the same
exploration the parent did; fan-out patterns where N workers each pay a
full cold context; zero cache hits on fork requests despite a warm parent.

**Related solution(s):** `solutions/subagent-context-handoff.md`,
`solutions/prompt-caching.md` (forks must reuse the parent's exact prefix)

### 6.2 Oversized model for the task

**Summary:** Every route uses the largest model, including routes a smaller
tier would serve well.

**Why it consumes tokens:** Strictly a *cost* multiplier rather than a
token-count one, but it compounds every other cause: frontier-tier models
across providers cost roughly 5–25× more per token than their small-tier
siblings, so the same token waste costs that much more.

**How to recognize it:** Simple classification/extraction/formatting traffic
flowing through the top-tier model; no routing layer; no use of the
provider's discounted async/batch tier for latency-insensitive work.

**Related solution(s):** `solutions/model-routing.md`,
`solutions/batch-processing.md`, `solutions/semantic-caching.md`

### 6.3 Concurrent cold-cache fan-out

**Summary:** N parallel requests with an identical prefix are fired at once
before any of them has populated the cache.

**Why it consumes tokens:** A cache entry generally becomes readable only
after the first request has been processed. All N parallel requests pay
full input price — none can read what the others are still writing.

**How to recognize it:** Fan-out batches where every request reports zero
cached tokens despite sharing a prefix, while sequential runs of the same
requests show hits from the second one on.

**Related solution(s):** `solutions/fan-out-warming.md` (warm
with one request, then fire the rest)

### 6.4 Over-prescriptive prompts and scaffolding

**Summary:** System prompts carry legacy scaffolding — forced progress
updates, step-by-step procedures, "double-check X" verification loops, long
few-shot batteries — tuned for older or different models.

**Why it consumes tokens:** The scaffolding itself is fixed prompt overhead
on every call, and it induces extra output: forced narration, redundant
verification tool calls, and re-explanations the current model would not
otherwise produce. Prompts accreted across model generations rarely get
re-audited.

**How to recognize it:** Prompt sections that duplicate current model
defaults ("after every 3 tool calls, summarize progress"); output containing
ritualized sections nobody reads; few-shot examples for behaviors the model
now does zero-shot.

**Related solution(s):** `solutions/prompt-de-scaffolding.md`

### 6.5 Cross-session cold starts (no persistent knowledge)

**Summary:** Every new session starts from zero — the agent re-explores the
codebase or domain and re-derives understanding that previous sessions
already paid for.

**Why it consumes tokens:** Statelessness applies *across* sessions, not
just within them. Orienting on a large repo is many large tool calls
(commonly 25–60K tokens before real work begins), and that bill repeats for
every new session, every engineer, every agent in the fleet — the same
knowledge purchased over and over.

**How to recognize it:** Session transcripts open with the same exploration
sequence (list → read → grep of the same core files); first-N-turn spend is
dominated by reads that a previous session already made; nothing learned in
one session is available to the next.

**Related solution(s):** `solutions/code-maps.md` (checked-in maps/context
packs), `solutions/subagent-context-handoff.md` (artifact stores),
`solutions/compaction.md` (carry summaries forward at session boundaries)

### 6.6 Fleet-duplicate requests

**Summary:** The same or near-identical request is answered many times
across users, sessions, or scheduled runs — each time at full model price.

**Why it consumes tokens:** Prefix caching discounts the repeated *prompt*
but still runs the model; with no response-level reuse, N near-identical
questions cost N full generations. Support/docs Q&A, recurring analytics
questions, and re-run eval suites are the classic shapes.

**How to recognize it:** Usage logs cluster into groups of near-identical
prompts with near-identical answers; high query overlap in read-mostly
traffic; scheduled jobs re-generating mostly-unchanged outputs.

**Related solution(s):** `solutions/semantic-caching.md`;
`solutions/batch-processing.md` for the scheduled-rerun variant

---

## Measurement Primer

To attribute cost to a cause, the usage metadata returned with each response
is the ground truth. Names differ per provider, but the same four quantities
exist almost everywhere:

| Quantity | Typical relative price | Example field names |
| --- | --- | --- |
| Uncached input tokens | 1× input | Anthropic `input_tokens`; OpenAI `prompt_tokens` minus `cached_tokens`; Gemini `prompt_token_count` minus `cached_content_token_count` |
| Cached input tokens | ~0.1–0.25× input | Anthropic `cache_read_input_tokens`; OpenAI `prompt_tokens_details.cached_tokens`; Gemini `cached_content_token_count` |
| Cache-write tokens (where surcharged) | ~1.25–2× input | Anthropic `cache_creation_input_tokens` (others bill writes at normal input rates) |
| Output tokens, **including hidden reasoning** | ~3–5× input | Anthropic `output_tokens`; OpenAI `completion_tokens` (+ `reasoning_tokens` detail); Gemini `candidates_token_count` + `thoughts_token_count` |

Two universal rules:

- **Total prompt size = uncached + cached (+ cache-write) input.** Judging
  spend from the uncached field alone under- or over-attributes cost.
- **Count tokens with the provider's own counter for the exact target
  model.** Tokenizers are model-specific; a counter borrowed from another
  provider (or another model generation) is systematically wrong.
