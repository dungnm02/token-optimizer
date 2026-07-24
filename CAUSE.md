# Nguyên nhân gây tiêu tốn token cao (Tiếng Việt)

Tài liệu này liệt kê các nguyên nhân đã biết khiến **agent và ứng dụng dùng
LLM** tiêu tốn nhiều token. Chúng không phụ thuộc nhà cung cấp
(provider-agnostic): stack nào cũng gặp (Claude, GPT, Gemini, model mã nguồn
mở…), vì gốc rễ nằm ở chính cách API LLM vận hành — request không lưu trạng
thái (stateless), tính tiền theo token, và context window có giới hạn. Chi
tiết riêng của từng nhà cung cấp chỉ xuất hiện làm ví dụ minh họa.

Mỗi nguyên nhân trả lời ba câu hỏi: **nó là gì**, **vì sao nó tốn token**,
và **nhận biết nó bằng cách nào**. Khi có cách khắc phục, mục đó dẫn link
sang tài liệu tương ứng trong `solutions/`; nguyên nhân nào phụ thuộc hoàn
toàn vào nhà cung cấp thì ghi chú thẳng như vậy, không cần tài liệu
solution riêng.

> Trạng thái: tài liệu sống. Danh mục dưới đây bao quát các *loại* nguyên
> nhân chính, gom theo nhóm. Từng mục có thể được bổ sung phân tích và số
> liệu đo lường theo thời gian.

## Bối cảnh: vì sao các nguyên nhân này mang tính phổ quát

Hầu như mọi API LLM đều có chung ba đặc điểm, và chính chúng sinh ra mọi
nguyên nhân trong danh mục này:

1. **Không lưu trạng thái (stateless).** API không nhớ gì giữa các lần gọi
   — client phải gửi lại mọi thứ model cần biết trong từng request. Cái
   "mọi thứ" đó phình ra bao nhiêu thì mọi request về sau phình theo bấy
   nhiêu.
2. **Tính tiền theo token, giá bất đối xứng.** Cả input lẫn output đều tính
   theo token: output thường đắt gấp 3–5 lần input, còn input đọc từ cache
   thường chỉ bằng ~10–25% giá input (khi nhà cung cấp có caching).
3. **Context có giới hạn.** Context window là hữu hạn. Càng gần giới hạn,
   hệ thống càng buộc phải cắt bớt, tóm tắt, hoặc chịu lỗi — và mọi thứ
   đang nằm trong đó đều bị tính tiền ở mỗi lần gọi.

## Mẫu cho một mục mới

Sao chép mẫu này khi thêm một nguyên nhân mới.

```
### <Tên nguyên nhân>

**Tóm tắt:** Mô tả nguyên nhân trong một câu.

**Vì sao nó tốn token:** Giải thích cơ chế làm tăng lượng token.

**Cách nhận biết:** Triệu chứng, chỉ số, hoặc mẫu hình cho thấy nguyên nhân
này đang xảy ra.

**Giải pháp liên quan:** Link đến (các) tài liệu trong `solutions/`. Nếu
nguyên nhân phụ thuộc nhà cung cấp (phía mình không tự sửa được), ghi rõ
như vậy thay vì link.
```

---

## Danh mục

Các nguyên nhân được gom thành sáu nhóm:

| # | Nhóm | Vấn đề cốt lõi |
| --- | --- | --- |
| 1 | Lỗi caching | Trả nguyên giá cho những token prefix lẽ ra chỉ tốn giá cache-read |
| 2 | Tích lũy context | Lịch sử hội thoại phình không giới hạn và bị gửi lại ở mỗi lượt |
| 3 | Cách dùng tool | Lệnh gọi tool và kết quả của chúng nhồi vào context toàn nội dung ít giá trị |
| 4 | Loại nội dung đắt đỏ | Ảnh, tài liệu, và dữ liệu truy xuất tốn hơn nhiều so với hình dung |
| 5 | Chi tiêu phía sinh (generation) | Token output (reasoning + câu trả lời) — loại token đắt nhất — bị tiêu quá tay |
| 6 | Lựa chọn kiến trúc | Thiết kế hệ thống nhân chi phí token lên qua nhiều request và agent |

## Giải pháp nằm ở đâu: bên thứ ba hay nhà cung cấp

Trước khi vào danh mục, cần định vị nhanh một điều: có những nguyên nhân
xử lý được trọn vẹn bằng **công cụ bên thứ ba, không phụ thuộc agent**
(thường là mã nguồn mở, mang theo được qua các nhà cung cấp). Có những
nguyên nhân lại nằm ở **năng lực và bảng giá mà chỉ nhà cung cấp kiểm
soát** — công cụ phía client có thể *tận dụng* một mức giảm giá hay một
nút chỉnh của nhà cung cấp, nhưng không bao giờ *tự tạo ra* được nó.

| Nhóm | Ai giải quyết là chính | Xử lý được bằng công cụ bên thứ ba / kiến trúc của bạn | Phần phụ thuộc nhà cung cấp |
| --- | --- | --- | --- |
| 1 Caching | **Nhà cung cấp** — vệ sinh prompt chỉ giúp bạn *đủ điều kiện* nhận giảm giá, không tạo ra nó | Kỷ luật giữ prompt ổn định, render tất định, test byte trong CI; đo cache-hit (Langfuse/Helicone/LiteLLM); tự host để tái dùng prefix (vLLM/SGLang) | Bản thân mức giảm giá: có cache hay không, giá đọc (~0.1×), TTL, breakpoint, phụ phí ghi |
| 2 Tích lũy context | **Bên thứ ba** — gần như nằm trọn trong tay bạn | Cắt tỉa/nén trong harness hoặc framework (LangGraph, LlamaIndex), registry hash, kho bộ nhớ | API nén/quản lý context phía server chỉ là tiện, không bắt buộc |
| 3 Dùng tool | **Bên thứ ba** — phần lớn bạn tự kiểm soát được | Ngân sách output, proxy nén (RTK/Headroom), tỉa MCP, hạ tầng hướng sự kiện (Temporal, webhook) | Tải tool trì hoãn / tool search, gọi tool bằng code (Code Mode), định dạng gọi tool tiết kiệm token |
| 4 Loại nội dung | **Bên thứ ba** — giảm độ phân giải/trích xuất không cần nhà cung cấp nhúng tay | Giảm độ phân giải (sharp/Pillow), trích xuất (Docling/unstructured), reranker mã nguồn mở (BGE), bản đồ code (aider/Repomix) | Công thức token cho ảnh và nút `detail`, API file/caching, endpoint `count_tokens` |
| 5 Phía sinh (generation) | **Nhà cung cấp** — đòn bẩy lớn nhất (ngân sách reasoning, `verbosity`) chỉ họ mới có | Hợp đồng output trong prompt, sửa file theo diff (Aider), retry có kiểm định (Instructor), nén output (Caveman) | Các nút chính đều của nhà cung cấp: ngân sách reasoning/thinking, `verbosity`, chế độ structured output |
| 6 Kiến trúc | **Cả hai** — chia đôi: định tuyến/khử trùng lặp là của bạn, bậc giá/fine-tuning là của họ | Định tuyến/gateway (RouteLLM/LiteLLM/Portkey), semantic caching (GPTCache), framework điều phối, làm ấm rồi mới fan-out, gói context | Thang giá giữa các tier model, tier batch giảm 50%, có fine-tuning/distillation hay không |

Quy tắc nằm lòng: **vệ sinh là việc của bạn, giảm giá là việc của họ.**
Mọi thứ giúp giữ token *đứng ngoài* request (context, output tool, prompt,
nội dung trùng lặp) đều mang theo được và xử lý được bằng bên thứ ba — vì
vậy nhóm 2, 3, 4 nghiêng hẳn về phía bạn. Còn các đòn bẩy làm cho số token
còn lại *rẻ đi* — giá cache prefix, tier batch, tier model rẻ hơn, nút
effort/verbosity — đều nằm trong tay nhà cung cấp, nên nhóm 1 và 5 nghiêng
hẳn về phía họ. Ở hai nhóm đó, việc của bạn chỉ là làm cho mình đủ điều
kiện hưởng ưu đãi, và lựa chọn nhà cung cấp sẽ định trần mức ưu đãi bạn
nhận được. Nhóm 6 chia đôi: nửa trong tay bạn, nửa trong tay họ.

---

## 1. Lỗi Caching

Hầu hết nhà cung cấp lớn đều có một dạng **prompt/prefix caching**
(breakpoint tường minh, prefix caching tự động, hoặc nội dung cache đăng
ký trước). Tất cả chung một cơ chế lõi: cache khớp theo **prefix** của
request, và token nằm trong cache được tính giá giảm rất sâu. Các nguyên
nhân dưới đây đều là những kiểu để vuột mất khoản giảm giá đó.

> **Prefix caching không phải lớp cache duy nhất.** Nó làm một *request
> lặp lại* rẻ đi, nhưng model vẫn phải chạy. Có một kiểu lãng phí khác,
> tách biệt hoàn toàn: cả hệ thống trả lời đi trả lời lại **những request
> gần giống hệt nhau** (cùng một câu hỏi hỗ trợ, cùng một truy vấn phân
> tích, cùng một prompt đánh giá) — lần nào cũng là một lệnh gọi model
> trọn vẹn, trong khi một cache ở *mức câu trả lời* (semantic cache) có
> thể bỏ qua hẳn. Đó là một cơ hội khác, nằm ở nguyên nhân 6.6; xem
> `solutions/semantic-caching.md`.

### 1.1 Không dùng prompt caching

**Tóm tắt:** Request có phần prefix lớn và ổn định (system prompt, định
nghĩa tool, tài liệu dùng chung) nhưng không dùng cơ chế caching của nhà
cung cấp — hoặc chạy trên provider/cấu hình không có caching.

**Vì sao nó tốn token:** API không lưu trạng thái, nên toàn bộ prompt bị
xử lý ở nguyên giá input trong mỗi request. Giá cache-read thường chỉ bằng
~10–25% giá input, nên bỏ qua caching nghĩa là trả đắt hơn 4–10 lần cho
mỗi đoạn prefix lặp lại, ở mọi lần gọi.

**Cách nhận biết:** Metadata usage của nhà cung cấp báo số token cache
bằng 0 (ví dụ `cache_read_input_tokens` bên Anthropic, `cached_tokens`
bên OpenAI) trên mọi phản hồi, trong khi tổng token input vẫn lớn và gần
như không đổi giữa các request.

**Giải pháp liên quan:** Không có tài liệu riêng — nguyên nhân này phụ
thuộc nhà cung cấp: hãy chọn nhà cung cấp/cấu hình có prompt caching và
bật nó lên; mức giảm giá là thứ chỉ họ tạo ra được.

### 1.2 Cache bị vô hiệu hóa âm thầm

**Tóm tắt:** Caching đã được cấu hình (hoặc chạy tự động) nhưng không bao
giờ trúng (hit), vì có thứ gì đó làm prefix của prompt thay đổi ở mỗi
request.

**Vì sao nó tốn token:** Prefix caching khớp **chính xác đến từng byte**
— chỉ một byte khác đi là mọi thứ phía sau nó mất hiệu lực. Thủ phạm quen
mặt: timestamp hay "ngày hôm nay" được chèn vào system prompt, request
ID/UUID nằm ở đầu nội dung, JSON serialize với thứ tự khóa không cố định,
dữ liệu riêng của người dùng đặt lên đầu, và các đoạn prompt render theo
điều kiện. Kết cục là tệ cả đôi đường: phụ phí ghi cache (nếu có) bị trả
ở mỗi request mà không có lấy một lần đọc.

**Cách nhận biết:** Metadata usage cho thấy toàn *ghi* cache (hoặc đơn
giản là không có token cache nào) ở mọi request, dù các request nhìn
giống hệt nhau. Diff bản prompt đã render đầy đủ của hai request liên
tiếp sẽ chỉ ra ngay đoạn nào đang thay đổi.

**Giải pháp liên quan:** `solutions/prompt-caching.md`

### 1.3 Thay đổi giữa phiên khiến cache phải xây lại

**Tóm tắt:** Sửa system prompt, đổi model, hoặc thêm/bớt/đảo thứ tự định
nghĩa tool giữa cuộc hội thoại làm toàn bộ prefix đã cache mất hiệu lực.

**Vì sao nó tốn token:** Định nghĩa tool và system prompt được render ở
ngay đầu request. Đổi bất cứ thứ gì ở phần đầu là toàn bộ lịch sử hội
thoại phía sau phải được xử lý lại không qua cache — trong một phiên
agentic dài, đó có thể là hàng trăm nghìn token bị tính lại nguyên giá
chỉ trong một request. Cache cũng luôn gắn với từng model: đổi model giữa
phiên là chắc chắn bắt đầu lại từ cache nguội.

**Cách nhận biết:** Input không cache tăng vọt (và input cache sụp xuống)
giữa phiên, trùng thời điểm với một thay đổi cấu hình: chuyển chế độ,
dựng lại bộ tool, đổi model, hay một lần "sửa nhỏ" system prompt.

**Giải pháp liên quan:** `solutions/prompt-caching.md`,
`solutions/stable-prompt-architecture.md`

### 1.4 Cache hết hạn giữa các request

**Tóm tắt:** Lưu lượng dồn cục hoặc thưa thớt để cache kịp hết hạn giữa
hai request (thời gian sống điển hình từ vài phút đến ~1 giờ, tùy nhà
cung cấp và cấu hình), nên mỗi đợt lại bắt đầu từ cache nguội.

**Vì sao nó tốn token:** Mục cache đã hết hạn phải được xử lý lại — và ở
những nhà cung cấp có phụ phí ghi cache, còn phải ghi lại với giá cao
hơn. Lưu lượng có khoảng nghỉ dài hơn thời gian sống của cache thì không
bao giờ hưởng lợi gì từ các request trước.

**Cách nhận biết:** Trong một đợt thì cache hit đều đặn, nhưng request
đầu tiên sau mỗi khoảng nghỉ lại là một lượt xử lý không cache (hoặc ghi
cache) trọn vẹn.

**Giải pháp liên quan:** Không có tài liệu riêng — TTL và phụ phí ghi là
cấu hình/bảng giá của nhà cung cấp: chọn TTL dài hơn ở nơi họ cho phép.

---

## 2. Tích lũy context

> **Nhóm này là vấn đề chất lượng, không chỉ là vấn đề chi phí.** Các
> nghiên cứu có kiểm soát về "context rot" trên 18 model frontier cho
> thấy model nào cũng kém đi khi input dài ra — và kém đi từ *trước khi*
> cửa sổ đầy: ngân sách thực dụng để giữ độ chính xác cao chỉ quanh mức
> 150–400K token, kể cả trên model có cửa sổ 2M. Độ chính xác rơi nhanh
> nhất khi phần nhiễu tích tụ *na ná về nghĩa* với câu trả lời — đúng
> kiểu một phiên lập trình dài đầy những lần dò sai suýt đúng. Vì vậy cắt
> tỉa lịch sử (bên dưới) vừa được chất lượng vừa được token — hai động cơ
> cùng chiều.

### 2.1 Lịch sử hội thoại không giới hạn

**Tóm tắt:** Toàn bộ lịch sử tin nhắn bị gửi lại ở mỗi lượt (API không
lưu trạng thái) mà không có gì cắt tỉa, tóm tắt, hay cho nó hết hạn.

**Vì sao nó tốn token:** Chi phí input mỗi lượt tăng gần như tuyến tính
theo độ dài hội thoại, nên tổng chi phí cả phiên tăng theo **bậc hai
(quadratic)**. Một phiên agentic 100 lượt có thể tiêu phần lớn ngân sách
chỉ để gửi đi gửi lại các lượt 1–99. Caching làm dịu chuyện này (lịch sử
nằm trong cache thì rẻ) nhưng không xóa được nó, và mỗi lần cache trượt
là toàn bộ lịch sử bị tính lại nguyên giá.

**Cách nhận biết:** Tổng kích thước prompt (input cache + không cache)
tăng đều qua từng lượt; các request cuối phiên lớn gấp 10–100 lần các
request đầu; phiên chạy đủ lâu thì chạm trần context window.

**Giải pháp liên quan:** `solutions/compaction.md`,
`solutions/context-editing.md`

### 2.2 Kết quả tool cũ vẫn nằm lại trong lịch sử

**Tóm tắt:** Những output tool cũ (dump file, kết quả tìm kiếm, output
lệnh) vẫn nằm trong transcript rất lâu sau khi đã hết liên quan.

**Vì sao nó tốn token:** Trong vòng lặp agentic, kết quả tool thường
chiếm phần lớn transcript. Một lần đọc file ở lượt 3, dù đã bị thay thế ở
lượt 20, vẫn bị gửi lại (và tính tiền lại) suốt các lượt 21–100.

**Cách nhận biết:** Soi transcript thấy các khối kết quả tool lớn có nội
dung trùng nhau hoặc đã bị phiên bản mới hơn thay thế; tỷ trọng kết quả
tool trong prompt cứ tăng, còn tỷ trọng thông tin hữu ích thì không.

**Giải pháp liên quan:** `solutions/context-editing.md`
(cắt tỉa kết quả tool / khối reasoning cũ)

### 2.3 Chèn context trùng lặp

**Tóm tắt:** Cùng một nội dung (một file, một schema, một bộ tài liệu
truy xuất) bị chèn vào cuộc hội thoại nhiều lần.

**Vì sao nó tốn token:** Mỗi lần chèn được tính tiền riêng, và mỗi bản
sao sau đó lại nằm trong lịch sử cho đến hết phiên (nguyên nhân 2.1
khuếch đại thêm).

**Cách nhận biết:** Cùng một file bị đọc lại nhiều lần qua các lượt;
pipeline truy xuất gắn lại đúng top-k tài liệu cũ vào mỗi tin nhắn người
dùng; nội dung vốn thuộc cấp hệ thống bị dán vào lượt người dùng "cho
chắc là model nhìn thấy".

**Giải pháp liên quan:** `solutions/context-hygiene.md`

---

## 3. Cách dùng Tool

### 3.1 Output tool quá lớn

**Tóm tắt:** Tool trả về nhiều hơn hẳn mức model cần — nguyên cả file
thay vì phần liên quan, nguyên phản hồi API thay vì vài trường cần thiết,
danh sách không phân trang.

**Vì sao nó tốn token:** Mỗi byte của kết quả tool là token input ở
request kế tiếp *và ở mọi request sau đó* chừng nào nó còn trong lịch sử.
Một lần đọc file 5.000 dòng không lọc sẽ bị tính số token của nó hàng
chục lần trong một phiên dài.

**Cách nhận biết:** Có những kết quả tool đơn lẻ lên tới hàng chục nghìn
token; nhìn là thấy model chỉ dùng một phần rất nhỏ của kết quả.

**Giải pháp liên quan:** `solutions/tool-output-budgets.md`,
`solutions/tool-output-compression.md`

### 3.2 Round-trip vụn vặt thay vì gộp lại (composition)

**Tóm tắt:** Nhiều lệnh gọi tool nối đuôi nhau, kết quả trung gian nào
cũng chảy qua context của model, dù model chỉ cần đáp số cuối cùng.

**Vì sao nó tốn token:** Mỗi round-trip gửi lại cả khối lịch sử đang
phình và bỏ thêm một kết quả trung gian vào đó. Ba lần tra cứu nối tiếp
(hồ sơ → đơn hàng → tồn kho) là ba lượt xử lý trọn context, và dữ liệu
trung gian thường chẳng bao giờ cần đến lần thứ hai.

**Cách nhận biết:** Chuỗi dài các lệnh gọi tool lặt vặt cho mỗi yêu cầu
người dùng; transcript đầy dữ liệu trung gian mà câu trả lời cuối không
hề nhắc tới.

**Giải pháp liên quan:** `solutions/tool-composition.md`
(điều phối tool bằng code, gộp phía server, gộp lô)

### 3.3 Vòng lặp thử lại và polling

**Tóm tắt:** Lệnh gọi tool thất bại được thử lại với nguyên context cũ,
hoặc agent poll trạng thái bên ngoài ("cứ kiểm tra lại trong vòng lặp")
và mỗi lần poll là một request model đầy đủ.

**Vì sao nó tốn token:** Mỗi lần thử lại/poll đều tính lại toàn bộ
prompt. Một vòng poll 10 lần trên context 100K token đốt 1 triệu token
input chỉ để nghe "chưa xong" chín lần. Người anh em phía agentic là
**doom loop**: model cứ lặp một bản sửa đang thất bại (sửa → test → fail
→ sửa gần giống), mỗi vòng tính lại cả khối context đang phình rồi nhét
thêm một lần thất bại nữa vào — chi phí cộng dồn trong khi tiến độ đứng
yên.

**Cách nhận biết:** Từng đợt request gần giống hệt nhau trong log usage;
kết quả tool lỗi lặp lại với input y nguyên; các mẫu sleep-and-check chạy
*xuyên qua* model thay vì *vòng ngoài* model; nhiều diff/lần test fail na
ná liên tiếp trong một phiên (hãy giới hạn số lần thử trong harness và
buộc lập kế hoạch lại hoặc leo thang).

**Giải pháp liên quan:** `solutions/event-driven-waiting.md`

### 3.4 Tải sẵn quá nhiều schema tool

**Tóm tắt:** Mọi định nghĩa tool mà ứng dụng có đều bị nhét vào từng
request, dù mỗi tác vụ chỉ cần vài cái.

**Vì sao nó tốn token:** Schema tool được serialize vào mỗi request. Hàng
trăm định nghĩa JSON-schema có thể cộng thêm hàng chục nghìn token chi
phí cố định mỗi lần gọi — và ở nhà cung cấp có prefix caching, chỉ cần bộ
tool thay đổi là cả cache mất hiệu lực (nguyên nhân 1.3). Chuyện này đặc
biệt hay gặp ở các thiết lập kiểu MCP, nơi nguyên cả tool server được gắn
vào một cục.

**Cách nhận biết:** Chênh lệch lớn giữa token input của request có kèm
tool và cùng request đó không kèm tool; danh sách tool dài hơn hẳn tập
tool thực sự được gọi.

**Giải pháp liên quan:** `solutions/tool-search.md` (khám phá tool động /
tải trì hoãn)

---

## 4. Loại nội dung đắt đỏ

### 4.1 Ảnh giữ nguyên độ phân giải gốc

**Tóm tắt:** Ảnh được gửi ở độ phân giải gốc/cao trong khi tác vụ không
cần độ nét đến vậy.

**Vì sao nó tốn token:** Mọi nhà cung cấp lớn đều token hóa ảnh theo diện
tích/tile, nên chi phí token tỷ lệ với độ phân giải — một tấm ảnh nguyên
cỡ có thể tốn vài nghìn token trong khi bản thu nhỏ chỉ tốn vài trăm. Các
vòng lặp nhiều screenshot (computer/browser use) nhân con số đó theo từng
bước, và ảnh cũng nằm lại trong lịch sử (nguyên nhân 2.1).

**Cách nhận biết:** Request có ảnh với số token input cao hơn hẳn phần
chữ; chi phí mỗi bước của vòng lặp computer-use bị screenshot chiếm gần
hết.

**Giải pháp liên quan:** `solutions/image-downsampling.md`

### 4.2 Đổ nguyên tài liệu (PDF / RAG truy xuất quá rộng)

**Tóm tắt:** Nguyên cả tài liệu, hoặc kết quả truy xuất quá rộng, được
đính kèm trong khi chỉ một lát cắt nhỏ là liên quan.

**Vì sao nó tốn token:** Một file PDF vài trăm trang hay một lần truy
xuất top-20 chunk có thể cộng thêm hàng chục đến hàng trăm nghìn token
input mỗi request. Không có caching thì khoản đó bị tính lại theo từng
câu hỏi; không cắt tỉa thì nó nằm trong lịch sử mãi.

**Cách nhận biết:** Các luồng hỏi-đáp mà câu hỏi nào cũng gửi lại đúng
tài liệu lớn đó; truy xuất được cấu hình thiên về recall (k lớn, chunk
to) trong khi câu trả lời chỉ trích dẫn một phần tí xíu những gì đã gửi.

**Giải pháp liên quan:** `solutions/document-reuse.md`,
`solutions/retrieval-tuning.md`, `solutions/code-maps.md` (cho codebase)

### 4.3 Nội dung tốn kém khi tokenize

**Tóm tắt:** Nội dung tokenize kém hiệu quả — code đậm đặc, JSON minify,
blob base64, nhiều hệ chữ ngoài tiếng Anh — hoặc một lần đổi
model/tokenizer làm cùng một đoạn văn bản bị đếm ra nhiều token hơn.

**Vì sao nó tốn token:** Số token phụ thuộc vào từng tokenizer, và
tokenizer khác nhau giữa các nhà cung cấp *lẫn giữa các thế hệ model của
cùng một nhà* (đã có lần nâng cấp làm lệch số token hơn 30% trên cùng một
văn bản). Log nguyên văn, bundle minify, và payload base64 thuộc nhóm tệ
nhất tính trên mỗi đơn vị thông tin hữu ích — riêng base64 đã thổi phồng
số byte thêm ~33% trước cả khi tokenize.

**Cách nhận biết:** Ngân sách hiệu chỉnh trên model này bỗng bị cắt cụt
hoặc đắt lên sau khi đổi model; endpoint đếm token của nhà cung cấp cho
con số cao hơn hẳn ước lượng ký-tự/4 trên nội dung thật của bạn.

**Giải pháp liên quan:** `solutions/token-counting.md`
(hiệu chỉnh lại bằng bộ đếm của chính nhà cung cấp, đúng model đích)

---

## 5. Chi tiêu phía sinh (Generation-Side)

### 5.1 Token reasoning/thinking

**Tóm tắt:** Model có khả năng suy luận sinh ra (và tính tiền) các token
chain-of-thought trước phần trả lời hiển thị — thường bị ẩn hoặc tóm tắt
trong phản hồi, nhưng kiểu gì cũng phải trả tiền.

**Vì sao nó tốn token:** Token output thường đắt gấp 3–5 lần input, và
reasoning được tính là output ở mọi nhà cung cấp lớn. Đặt mức
reasoning-effort cao đồng loạt nghĩa là tiêu suy luận sâu cho cả những
route không cần; vài nhà cung cấp còn bật reasoning mặc định, nên chi phí
xuất hiện dù bạn chưa hề bật gì.

**Cách nhận biết:** Số token output báo cáo vượt xa độ dài câu trả lời
nhìn thấy; các route đơn giản (phân loại, tra cứu) tiêu output ngang các
route phức tạp.

**Giải pháp liên quan:** Không có tài liệu riêng — ngân sách
reasoning/effort là nút chỉnh của nhà cung cấp: đặt mức effort phù hợp
cho từng route ngay trong cấu hình gọi API.

### 5.2 Output dài dòng

**Tóm tắt:** Model viết nhiều hơn mức cần — in lại nguyên file thay vì
diff, tường thuật từng bước, mở bài và tổng kết lê thê.

**Vì sao nó tốn token:** Output là loại token đắt nhất, và trong phiên
nhiều lượt, mỗi câu trả lời dài dòng còn quay lại làm *input* cho mọi
lượt sau — sự dài dòng bị tính một lần là output và N lần nữa dưới dạng
lịch sử.

**Cách nhận biết:** Câu trả lời nhắc lại câu hỏi, in lại code không đổi,
hay tóm tắt thứ vừa hiển thị xong; số token output không tương xứng với
lượng thông tin thực nhận được.

**Giải pháp liên quan:** `solutions/concise-output-prompting.md`,
`solutions/diff-based-edits.md`

### 5.3 Bị cắt output giữa chừng rồi thử lại

**Tóm tắt:** Trần token output đặt quá thấp làm câu trả lời bị cắt giữa
chừng, và ứng dụng thử lại nguyên cả request.

**Vì sao nó tốn token:** Mỗi lần thử lại tính lại toàn bộ input, cộng
thêm phần output dở dang đã vứt đi. Hai lần bị cắt trước khi lần thứ ba
thành công tốn cỡ 3 lần input và ~2 câu trả lời output bỏ phí. Cùng cơ
chế đó áp cho các lỗi kiểm định schema của structured output — mỗi lần
sinh không hợp lệ là một request lại từ đầu.

**Cách nhận biết:** Log có lý do dừng length/max-token, theo sau là các
request thử lại gần giống hệt; lỗi parse structured output đổ vào một
vòng thử lại.

**Giải pháp liên quan:** `solutions/output-cap-sizing.md`

---

## 6. Lựa chọn kiến trúc

### 6.1 Subagent khởi động nguội

**Tóm tắt:** Subagent được sinh ra cho từng tác vụ con mà không có cache
chung hay bàn giao context, con nào cũng tự mò lại những gì agent cha đã
biết.

**Vì sao nó tốn token:** Mỗi lần sinh subagent là gửi lại (và thường là
khám phá lại qua các lệnh gọi tool) phần context mà orchestrator đã trả
tiền rồi. Lệnh fork nào dựng lại system prompt / danh sách tool / lựa
chọn model mà lệch đi dù chỉ một chút cũng trượt hoàn toàn khỏi prefix
cache của agent cha.

**Cách nhận biết:** Transcript của subagent mở màn bằng đúng chuỗi khám
phá agent cha vừa làm; mẫu fan-out mà N worker đều trả nguyên giá một
context nguội; request fork không có cache hit nào dù agent cha đang ấm.

**Giải pháp liên quan:** `solutions/subagent-context-handoff.md`,
`solutions/prompt-caching.md` (fork phải dùng lại đúng y nguyên prefix
của agent cha)

### 6.2 Model quá cỡ so với tác vụ

**Tóm tắt:** Route nào cũng dùng model lớn nhất, kể cả những route mà một
tier nhỏ hơn thừa sức phục vụ.

**Vì sao nó tốn token:** Đây thuần túy là hệ số nhân về *chi phí* chứ
không phải về *số token*, nhưng nó khuếch đại mọi nguyên nhân khác: model
tier frontier ở các nhà cung cấp đắt hơn khoảng 5–25 lần mỗi token so với
đàn em tier nhỏ cùng dòng, nên cùng một lượng token lãng phí sẽ đắt lên
chừng đó lần.

**Cách nhận biết:** Lưu lượng phân loại/trích xuất/định dạng đơn giản vẫn
chạy qua model tier cao nhất; không có lớp định tuyến; không dùng tier
async/batch giảm giá của nhà cung cấp cho việc không cần phản hồi ngay.

**Giải pháp liên quan:** `solutions/model-routing.md`,
`solutions/batch-processing.md`, `solutions/semantic-caching.md`

### 6.3 Fan-out song song khi cache còn nguội

**Tóm tắt:** N request song song, chung một prefix, được bắn ra cùng lúc
trước khi có cái nào kịp ghi vào cache.

**Vì sao nó tốn token:** Một mục cache thường chỉ đọc được sau khi
request đầu tiên xử lý xong. Cả N request song song đều trả nguyên giá
input — không cái nào đọc được thứ mà những cái kia còn đang ghi dở.

**Cách nhận biết:** Các đợt fan-out mà request nào cũng báo 0 token cache
dù chung prefix, trong khi chạy tuần tự đúng các request đó thì từ cái
thứ hai đã hit.

**Giải pháp liên quan:** `solutions/fan-out-warming.md` (làm ấm bằng một
request, rồi mới bắn phần còn lại)

### 6.4 Prompt và scaffolding thừa chỉ dẫn

**Tóm tắt:** System prompt cõng theo scaffolding đời cũ — bắt cập nhật
tiến độ, quy trình từng bước, vòng "kiểm tra lại X", những bộ few-shot
dài — vốn được tinh chỉnh cho các model đời trước hoặc model khác.

**Vì sao nó tốn token:** Bản thân scaffolding là chi phí prompt cố định
mỗi lần gọi, và nó còn kéo theo output thừa: tường thuật bắt buộc, các
lệnh gọi tool kiểm tra lại không cần thiết, và những lời giải thích mà
model hiện tại vốn sẽ không tự viết ra. Prompt bồi đắp qua nhiều thế hệ
model thì hiếm khi được rà soát lại.

**Cách nhận biết:** Có những đoạn prompt lặp lại hành vi mặc định của
model hiện tại ("cứ 3 lệnh gọi tool thì tóm tắt tiến độ"); output chứa
các phần nghi thức chẳng ai đọc; ví dụ few-shot cho những hành vi model
giờ làm được không cần ví dụ.

**Giải pháp liên quan:** `solutions/prompt-de-scaffolding.md`

### 6.5 Khởi động nguội giữa các phiên (kiến thức không được giữ lại)

**Tóm tắt:** Phiên mới nào cũng bắt đầu từ con số không — agent khám phá
lại codebase hay lĩnh vực và suy ra lại những hiểu biết mà các phiên
trước đã trả tiền mua rồi.

**Vì sao nó tốn token:** Tính không trạng thái áp dụng *giữa* các phiên,
chứ không chỉ bên trong một phiên. Để định vị trên một repo lớn, agent
phải tốn rất nhiều lệnh gọi tool nặng (thường 25–60K token trước khi công
việc thật bắt đầu), và hóa đơn đó lặp lại cho từng phiên mới, từng kỹ sư,
từng agent trong hệ thống — cùng một mớ kiến thức bị mua đi mua lại.

**Cách nhận biết:** Transcript phiên nào cũng mở màn bằng đúng một chuỗi
khám phá (liệt kê → đọc → grep đúng mấy file lõi ấy); chi tiêu N lượt đầu
bị chiếm bởi những lần đọc mà phiên trước đã đọc rồi; những gì học được
trong phiên này không phiên sau nào dùng lại được.

**Giải pháp liên quan:** `solutions/code-maps.md` (bản đồ/gói context
commit sẵn), `solutions/subagent-context-handoff.md` (kho artifact),
`solutions/compaction.md` (mang bản tóm tắt vượt qua ranh giới phiên)

### 6.6 Request trùng lặp trên toàn hệ thống (fleet)

**Tóm tắt:** Cùng một request, hoặc gần giống hệt, được trả lời nhiều lần
trên nhiều người dùng, nhiều phiên, hay các lượt chạy theo lịch — lần nào
cũng nguyên giá model.

**Vì sao nó tốn token:** Prefix caching giảm giá cho phần *prompt* lặp
lại nhưng model vẫn chạy; không có tái sử dụng ở mức câu trả lời thì N
câu hỏi gần trùng nhau tốn N lượt sinh trọn vẹn. Hỏi-đáp hỗ trợ/tài liệu,
các câu hỏi phân tích định kỳ, và bộ eval chạy đi chạy lại là những hình
mẫu kinh điển.

**Cách nhận biết:** Log usage vón thành từng cụm prompt gần giống nhau
với câu trả lời gần giống nhau; độ trùng truy vấn cao trong lưu lượng
thiên về đọc; các job theo lịch sinh lại những output gần như không đổi.

**Giải pháp liên quan:** `solutions/semantic-caching.md`;
`solutions/batch-processing.md` cho biến thể chạy lại theo lịch

---

## Cẩm nang đo lường

Muốn quy chi phí về đúng nguyên nhân, hãy bám vào metadata usage trả về
cùng mỗi phản hồi — đó là nguồn số liệu chuẩn (ground truth). Tên trường
mỗi nhà một kiểu, nhưng bốn đại lượng sau thì gần như đâu cũng có:

| Đại lượng | Giá tương đối điển hình | Tên trường ví dụ |
| --- | --- | --- |
| Token input không cache | 1× input | Anthropic `input_tokens`; OpenAI `prompt_tokens` trừ `cached_tokens`; Gemini `prompt_token_count` trừ `cached_content_token_count` |
| Token input đã cache | ~0.1–0.25× input | Anthropic `cache_read_input_tokens`; OpenAI `prompt_tokens_details.cached_tokens`; Gemini `cached_content_token_count` |
| Token ghi cache (nơi có phụ phí) | ~1.25–2× input | Anthropic `cache_creation_input_tokens` (nơi khác tính phí ghi theo giá input thường) |
| Token output, **gồm cả reasoning ẩn** | ~3–5× input | Anthropic `output_tokens`; OpenAI `completion_tokens` (+ chi tiết `reasoning_tokens`); Gemini `candidates_token_count` + `thoughts_token_count` |

Hai quy tắc phổ quát:

- **Tổng kích thước prompt = input không cache + input cache (+ phần ghi
  cache).** Chỉ nhìn vào trường không cache là quy chi phí thiếu hoặc
  thừa.
- **Đếm token bằng bộ đếm của chính nhà cung cấp, đúng model đích.**
  Tokenizer gắn với từng model; mượn bộ đếm của nhà khác (hay của thế hệ
  model khác) là sai một cách có hệ thống.

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
mitigation exists; causes that hinge entirely on the provider say so
instead of linking to a solution doc.

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
If the cause is provider-dependent (nothing to build on our side), say so
instead of linking.
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

| Category | Primarily solved by | Solvable with third-party tools / your architecture | Depends on the provider |
| --- | --- | --- | --- |
| 1 Caching | **Provider** — hygiene only makes you *eligible* for the discount, it doesn't create it | Prompt-stability discipline, deterministic rendering, CI byte-tests; cache-hit telemetry (Langfuse/Helicone/LiteLLM); self-hosted prefix reuse (vLLM/SGLang) | The discount itself: cache existence, read pricing (~0.1×), TTLs, breakpoints, write surcharges |
| 2 Context accumulation | **Third-party** — almost entirely in your hands | Almost fully third-party: pruning/compaction in the harness or framework (LangGraph, LlamaIndex), hash registries, memory stores | Server-side compaction/context-management APIs are a convenience, not a requirement |
| 3 Tool usage | **Third-party** — mostly under your control | Mostly third-party: output budgets, compression proxies (RTK/Headroom), MCP trimming, event-driven infra (Temporal, webhooks) | Deferred tool loading / tool search, programmatic tool calling (Code Mode), token-efficient tool-call formats |
| 4 Content types | **Third-party** — downscaling/extraction need no provider involvement | Downscaling (sharp/Pillow), extraction (Docling/unstructured), open rerankers (BGE), code maps (aider/Repomix) | Vision token formulas and `detail` dials, file/caching APIs, `count_tokens` endpoints |
| 5 Generation-side | **Provider** — the biggest lever (reasoning-effort budgets, `verbosity`) exists only on the provider side | Output contracts in prompts, diff edit formats (Aider), validation-aware retries (Instructor), output compression (Caveman) | The main dials are provider knobs: reasoning-effort/thinking budgets, `verbosity`, structured-output modes |
| 6 Architecture | **Both** — split down the middle: routing/dedup is yours, price tiers/fine-tuning are theirs | Routing/gateways (RouteLLM/LiteLLM/Portkey), semantic caching (GPTCache), orchestration frameworks, warm-then-fan logic, context packs | The price ladder across model tiers, the 50% batch tier, fine-tuning/distillation availability |

Rule of thumb: **hygiene is yours, discounts are theirs.** Everything that
keeps tokens *out* of requests (context, tool output, prompts, duplication)
is portable and third-party-solvable — which is why categories 2, 3, and 4
tilt firmly your way. The levers that make remaining tokens *cheaper*
(prefix caching rates, batch tier, cheaper model tiers, effort/verbosity
dials) are provider-controlled — which is why categories 1 and 5 tilt
firmly theirs. Your job there is to qualify for those discounts, and a
provider choice caps how much they can give you. Category 6 is the even
split: half in your hands, half in theirs.

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

**Related solution(s):** None — this cause is provider-dependent: pick a
provider/config that offers prompt caching and enable it; the discount
itself is theirs to create.

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

**Related solution(s):** None — cache TTLs and write surcharges are
provider configuration/pricing: pick a longer TTL where offered.

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

**Related solution(s):** None — reasoning-effort/thinking budgets are
provider knobs: set the right effort per route in your API call config.

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
