Chào mừng mọi người đã đến với buổi seminar ngày hôm nay do team FE tổ chức ạ. Hôm nay, em xin phép thay mặt team trình bày và giới thiệu đến mọi người một chủ đề mang tính "sống còn" đối với túi tiền của dự án: **Token và Chiến lược tối ưu chi phí khi dùng AI Agent**.

Như mọi người cũng biết, hiện nay nhân viên trong công ty mới được cấp phép sử dụng external AI (như Claude Code, Cline, Aider) cho việc coding assistant. Hôm trước, em mới thử múa vài đường prompt nhẹ nhàng bằng Claude Code thôi mà... úi giồi ôi, bay ngay 5 đô la chỉ trong nháy mắt!

Vâng, với cú sốc "đau ví" đó, việc tiết kiệm token không còn là lý thuyết suông nữa, mà là một kỹ năng bắt buộc phải áp dụng vào workflow. Mục tiêu của chúng ta là làm sao để "vắt kiệt" công năng của các external AI Agent này mà không làm thủng ngân sách.

---

### Phần 1: Lý thuyết cơ bản - Token là gì?

Đầu tiên, chúng ta sẽ đi vào một chút lý thuyết. Mọi người hay nói "tốn token này, token kia", nhưng token thực sự là gì và nó được tạo ra như thế nào?

AI không đọc văn bản theo từng chữ cái hay từng từ như chúng ta. Chúng xử lý luồng dữ liệu thông qua các mảnh ghép toán học được gọi là **token**, hình thành từ một thuật toán có tên là **Byte Pair Encoding (BPE)**.

Thuật toán BPE khởi đầu bằng việc coi mỗi byte (ký tự) là một token riêng biệt. Sau đó, nó quét qua một tập dữ liệu khổng lồ để đếm tần suất các cặp liền kề. Cặp nào xuất hiện nhiều nhất sẽ được "gộp" thành một token mới. Quá trình này lặp lại liên tục cho đến khi đạt được bộ từ vựng mong muốn.

**Ví dụ thực tế quá trình biến chữ thành token với cụm từ: "Data Platform"**

* **Lần 1 (Khởi tạo):** AI nhìn thấy các ký tự rời rạc: `D`, `a`, `t`, `a`, `_`, `P`, `l`, `a`, `t`, `f`, `o`, `r`, `m`. (Tốn 13 token).
* **Tần suất:** Nó nhận thấy cặp chữ "a" và "t" đứng cạnh nhau rất nhiều lần (trong chữ D**at**a và Pl**at**form).
* **Ghép token:** Nó tạo ra một token mới là `at`.
* **Lần 2 (Sau nhiều vòng lặp):** Qua hàng triệu văn bản, AI học được và gộp hẳn thành các token lớn hơn. Cuối cùng cụm từ trên có thể chỉ còn 2 token là `Data` và ` Platform`.

**Cạm bẫy khoảng trắng trong lập trình:**
Trong bối cảnh code phần mềm, cơ chế BPE này tạo ra những cái bẫy tốn tiền ngớ ngẩn. Code của chúng ta phụ thuộc rất nhiều vào khoảng trắng (whitespace) và thụt lề (indentation). Nhiều tokenizer truyền thống xử lý các chuỗi khoảng trắng hoặc tab thành các token riêng rẽ.
*Ví dụ:* Một file JSON định dạng đẹp mắt (pretty-printed) với thụt lề 4 dấu cách có thể tốn 19 token. Nhưng nếu nén lại (minified) bằng cách xóa khoảng trắng, nó chỉ tốn 11 token. Việc gửi JSON, log, hoặc config chưa nén cho API có thể làm tăng chi phí token đầu vào lên tới 40%!

---

### Phần 2: Agent xử lý Token thế nào? (Prefill vs. Decode)

Sau khi LLM nhận vào các mảnh token đó, chúng làm gì? Quá trình suy luận (Inference) của LLM chia làm 2 giai đoạn:

1. **Giai đoạn Prefill (Đọc/Nạp):** Hệ thống đọc và tính toán cho toàn bộ chuỗi token đầu vào *cùng một lúc*. Giai đoạn này tiêu thụ sức mạnh xử lý song song của GPU. Kết quả của bước này là một cấu trúc toán học khổng lồ được gọi là **KV Cache (Key-Value Cache)**, lưu trữ trên RAM của GPU để mô hình "nhớ" được ngữ cảnh.


2. **Giai đoạn Decode (Viết/Sinh):** Đây là lúc Agent sinh ra từng token để phản hồi cho chúng ta. Giai đoạn này diễn ra một cách tuần tự (autoregressive). Để đẻ ra token thứ N+1, mô hình phải chui vào KV Cache đọc lại N token trước đó. Do đó, quá trình sinh text bị giới hạn nghiêm trọng bởi băng thông bộ nhớ (memory-bandwidth-bound) thay vì tốc độ tính toán.



*(Nhấn mạnh)* VRAM của GPU lại là tài nguyên đắt đỏ nhất trên hành tinh hiện nay. Đó chính là lý do vì sao **giá của Output Token thường đắt gấp 3 đến 5 lần giá Input Token**.

---

### Phần 3: Tại sao chúng ta lại đốt quá nhiều Token trong quá trình Dev?

Ngoài những lý do chung như gửi quá nhiều file không liên quan, thì khi dùng các Agent như Claude Code hay Aider, chúng ta thường lãng phí token ở 3 giai đoạn:

**1. Giai đoạn Solution và Design (Phác thảo và Thiết kế hệ thống):**
Ở giai đoạn này, chúng ta thường yêu cầu AI phân tích kiến trúc để thêm tính năng mới hoặc tìm hiểu luồng dữ liệu. Lượng token lãng phí khổng lồ ở bước này thường đến từ hai sai lầm:

* **Nhồi nhét mã nguồn thô (Raw code dumping) và Tìm kiếm "mù":** Để Agent hiểu được sự liên kết giữa các module hoặc tìm xem một function cụ thể được gọi ở đâu, dev thường có thói quen copy-paste toàn bộ nội dung của 4-5 file code liên quan vào prompt. Nếu dùng Agent tự động (như Cline), nó cũng sẽ gọi các lệnh như `cat` hoặc `grep` để quét và đọc toàn bộ mã nguồn. Việc này "đốt" token kinh khủng vì 90% nội dung file là chi tiết triển khai (vòng lặp, logic xử lý nội bộ) – những thứ AI **không hề cần** ở giai đoạn thiết kế. Cái AI thực sự cần chỉ là: Hàm đó nhận input gì, trả output gì, và nó import/export từ đâu.

* **Vẽ biểu đồ bằng ASCII (ASCII art):** Lỗi phổ biến tiếp theo là yêu cầu AI sinh ra các biểu đồ cấu trúc bằng ký tự (`|`, `-`, `+`). Các ký hiệu này gây nhiễu loạn nghiêm trọng cho thuật toán phân tích BPE của AI. Một biểu đồ ASCII đơn giản ngốn tới 55 token, và dễ làm AI hiểu sai cấu trúc. *(Giải pháp thay thế là dùng cú pháp Mermaid.js - định hướng luồng suy luận của LLM tốt hơn và tốn cực ít token).*

**2. Giai đoạn Triển khai mã nguồn (Implementation):**
Các Agent có khả năng tự chạy lệnh trong Terminal (CLI). Lượng token lãng phí khổng lồ đến từ:

* **Thông tin từ Schema:** Khi Agent dùng Model Context Protocol (MCP) để gọi Tools, toàn bộ cái JSON Schema giải thích công cụ đó bị đính kèm vào *mỗi lượt* chat.
* **Dữ liệu rác từ Terminal:** Khi Agent chạy lệnh như `npm test`, toàn bộ luồng log (stdout) bị đẩy thẳng vào context window. Một lỗi vặt có thể in ra 5,000 token log, trong khi AI chỉ cần vài dòng báo lỗi cuối cùng!

**3. Giai đoạn Code Review:**
Chúng ta hay đưa lệnh `git diff` cho AI đọc. Nhưng `git diff` sinh ra để cho mắt người nhìn, chứa vô số các dòng không đổi (context lines) và tiêu đề (hunk headers) không mang lại giá trị logic cho AI, gây lãng phí token.

---

### Phần 4: Giải pháp - "Tam Giác Vàng" (CodeGraph, RTK, Caveman)

Để trị tận gốc các lỗi trên, team xin giới thiệu 3 tools tạo thành một "Tam giác vàng" bao phủ vòng đời token của AI Agent:

**1. CodeGraph - Giải quyết bài toán "Đầu vào tĩnh" (Planner Phase)**

* *Tại sao chọn?* Thay vì "nhồi nhét" toàn bộ thư mục code ngốn hàng trăm ngàn token, CodeGraph dùng thuật toán Tree-sitter và PageRank để nén kho mã nguồn thành một bản đồ (Repo Map) chỉ khoảng 1,000 - 2,000 token.
* *Cơ chế:* Mô hình sẽ chỉ nhìn thấy chữ ký hàm (function signatures) và các liên kết module thay vì đọc chi tiết từng dòng code. Rất lý tưởng khi mới mở phiên làm việc cần AI hiểu kiến trúc tổng thể.

**2. RTK - Giải quyết bài toán "Đầu vào động" (Implementer Phase)**

* *Tại sao chọn?* Để giảm 60-90% token lãng phí từ tiếng ồn Terminal.
* *Cơ chế:* RTK đứng chắn giữa Agent và Terminal. Khi Agent chạy `npm test`, RTK tự động xóa các biểu đồ ASCII, gộp dòng trắng, và khử trùng lặp (vd: gom 15 dòng lỗi giống nhau thành `[x15] Memory Warning`). *(Mở slide hình ảnh so sánh command thường vs command có RTK để thấy code gain).*

**3. Caveman - Giải quyết bài toán "Đầu ra đắt đỏ"**

* *Tại sao chọn?* Token đầu ra là loại đắt nhất. LLM rất hay nói nhảm (Ví dụ: "Sure, I can help with that. Here is the updated code...").
* *Cơ chế:* Caveman ép Agent (như Claude/Gemini) phản hồi bằng các câu cụt lủn "như người tối cổ", bỏ hết từ nối, lời chào, chỉ nhổ mã nguồn ra thôi. Chế độ Ultra có thể giảm 65% token đầu ra và giảm thời gian chờ đợi.

**Tips & Tricks Tối ưu (Synergy):**

* Dùng lệnh `/caveman-compress` kết hợp với Repo Map của CodeGraph để nén file tổng quan dự án thành văn phong tối cổ, tiết kiệm thêm 46% token trước khi nạp vào hệ thống.
* Có thể tùy chỉnh mức độ: Dùng Caveman Ultra khi Agent tự debug với CLI. Nhưng khi nhờ nó viết Docs cho con người đọc, hãy tắt đi để văn phong tự nhiên hơn.

**Khi nào KHÔNG nên dùng bộ 3 này?**

* Khi bạn cần AI tái cấu trúc (refactor) toàn diện một module lớn, nó bắt buộc phải đọc full file chi tiết chứ không chỉ đọc Map của CodeGraph.
* Khi bạn cần AI giải thích logic phức tạp cho một bạn Junior, việc ép nó dùng Caveman (nói cụt lủn) sẽ làm nội dung trở nên vô dụng.

---

### Phần 5: Bản chất của Caching và Hình phạt hết hạn (The Expiration Penalty)

Để hiểu cách tối ưu hóa đơn, chúng ta phải hiểu sự thật phũ phàng này: **LLM là các mô hình không có trí nhớ (Stateless)**.
Mỗi khi bạn ấn Enter, ứng dụng ở máy bạn phải gói *toàn bộ* lịch sử chat (bao gồm System Rules, Schema, lịch sử chat) và gửi lại cho API. Nếu không có cơ chế Cache, bạn sẽ trả nguyên giá cho hàng chục ngàn token cũ kỹ này ở mỗi lượt chat!

Nhưng may mắn, các hãng AI hiện nay hỗ trợ **Prompt Caching** (hay Radix Tree memory).

**Hãy xem vòng đời của 1 Agent Session:**

* **Turn 1 (Cold Start):** Bạn gửi nguyên 1 cục bự (System Prompt + User Msg 1). Server chưa từng thấy cục này, nên nó phải Prefill toàn bộ 10,000 tokens và lưu vào KV Cache. Bạn trả giá gốc (rất đắt).


* **Turn 2 (Cache Hit):** Bạn gửi lệnh tiếp theo. Payload bây giờ là (History cũ + User Msg 2). Server nhận ra: "À, phần đầu của đống này giống hệt cái tao vừa tính toán lúc nãy!". Nó lấy kết quả từ Cache ra, bỏ qua 98% quá trình xử lý, và chỉ tính toán cho vài chục token mới của Msg 2. Bạn được giảm giá tới 90% cho phần token cũ!



**Quy tắc Vàng (Exact Prefix Matching):**
Để Cache hoạt động, dữ liệu cũ phải khớp chính xác tuyệt đối từ trên xuống dưới (Exact prefix).
Do đó, khi thiết kế Prompt, bạn PHẢI xếp theo thứ tự:

1. **Lớp tĩnh (Trên cùng):** Chứa Repo Map của CodeGraph và System Prompt.


2. **Lớp động (Dưới cùng):** Log của RTK, hội thoại người dùng.
Nếu bạn đảo log của RTK lên trên, bạn phá vỡ cấu trúc "tiền tố", mọi thứ bên dưới sẽ phải tính toán lại ở mức giá gốc!



**Tại sao Cache chỉ sống được 5 phút? (The Expiration Penalty)**
Nhiều bạn thắc mắc sao đang code, đi vệ sinh 10 phút quay lại, chat tiếp thì tiền lại bị trừ 1 cục siêu to?
Vì KV Cache không thể lưu trên ổ cứng SSD rẻ tiền, nó phải nằm trong HBM (High Bandwidth Memory) của GPU - thứ tài nguyên đắt đỏ và khan hiếm nhất hiện nay.
API của Claude hay OpenAI phục vụ hàng triệu người. Nếu bạn đi uống cafe (quá 5 phút TTL), server sẽ **xóa (evict) toàn bộ Cache của bạn** để nhường RAM cho người khác.
Lúc bạn quay lại gửi Turn 11, server phải bắt đầu lại quá trình Cold Start trên toàn bộ lịch sử 10 turn trước đó. Đây gọi là "Hình phạt hết hạn" (Expiration Penalty).

=> **Mẹo:** Khi dùng Cline/Aider, hãy làm việc liên tục. Nếu lỡ rời đi quá lâu, tốt nhất hãy clear lịch sử và bắt đầu một session mới (tạo Context mới bằng CodeGraph) để tránh bị tính phí "Cold Start" khổng lồ trên một tệp lịch sử quá dài.

---

**Kết luận**
Việc dùng external AI cho code không phải là cầm tiền ném qua cửa sổ nếu chúng ta biết cách kiểm soát nó. Kết hợp tư duy cấu trúc Prompt (để tận dụng Caching) cùng bộ 3 công cụ CodeGraph, RTK và Caveman sẽ giúp team tận dụng được sức mạnh tuyệt đối của LLM với mức giá chỉ bằng một ly trà đá.

Cảm ơn mọi người đã lắng nghe! Sau đây, em xin phép chuyển sang phần Q&A ạ.
