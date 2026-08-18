# Kịch bản thuyết trình — Tiết kiệm token cho AI coding agent

Thời lượng gợi ý: 20–25 phút, chưa gồm Q&A. Kịch bản được viết theo ngôn ngữ nói; không cần đọc nguyên văn mọi gạch đầu dòng trên slide.

## Slide 1 — Tiết kiệm token cho AI coding agent

Chào mọi người. Hôm nay tôi muốn nói về một thứ rất dễ bị xem là chi tiết kỹ thuật, nhưng thực tế lại ảnh hưởng trực tiếp đến chi phí và chất lượng khi dùng coding agent: token.

Điểm tôi muốn mọi người giữ lại sau buổi này không phải là một con số giá cụ thể. Giá model có thể thay đổi. Điều quan trọng hơn là hiểu token đi đâu trong một phiên làm việc, vì sao một yêu cầu có vẻ đơn giản vẫn có thể tốn nhiều token, và chúng ta có thể can thiệp ở những đâu.

Trong sơ đồ nhỏ bên phải có ba thành phần: người dùng, agent và tool. Phần lớn chi phí phát sinh từ cách ba thành phần này trao đổi với nhau, chứ không chỉ từ câu trả lời cuối cùng mà chúng ta nhìn thấy.

*Chuyển ý:* Trước hết, hãy nhìn vào một tình huống rất quen thuộc.

## Slide 2 — Một yêu cầu ngắn không đồng nghĩa với việc tiêu tốn ít token

Ví dụ người dùng chỉ yêu cầu: “Thêm trường `preferredLanguage` vào hồ sơ người dùng.” Nhìn từ bên ngoài, đây là một yêu cầu ngắn và cuối cùng cũng chỉ có một câu trả lời.

Nhưng bên trong, agent có thể phải tìm model và API, đọc các file phụ thuộc, sửa code, chạy test, đọc log, sửa lại rồi mới trả kết quả. Mỗi bước có thể tạo thêm một lần gọi model; context ở những lần sau thường còn lớn hơn lần trước vì đã có thêm lịch sử và kết quả tool.

Vì vậy, đơn vị cần quan sát không phải chỉ là “một user message”. Một user message có thể dẫn tới nhiều lần gọi model. Đây là lý do câu trả lời cuối nhìn rất ngắn nhưng tổng chi phí của task vẫn cao.

*Chuyển ý:* Muốn hiểu chi phí đó, trước tiên cần thống nhất token thực chất là gì.

## Slide 3 — Model đọc các mảnh token

Model không đọc văn bản theo cách con người đọc từng chữ. Văn bản được chia thành các mảnh token theo vocabulary của model.

Một cụm phổ biến có thể được biểu diễn bằng rất ít token. Ngược lại, một chuỗi hiếm như UUID, hash hoặc mã định danh có thể bị chia thành rất nhiều mảnh nhỏ. Cách chia còn phụ thuộc vào tokenizer, ngôn ngữ và loại nội dung.

Điểm cần nhớ ở đây là: chúng ta trả tiền theo số mảnh token, không phải theo số ký tự, số dòng hay diện tích văn bản trên màn hình.

*Chuyển ý:* Vì vậy, hai đoạn trông dài tương đương chưa chắc có chi phí tương đương.

## Slide 4 — Mật độ token khác nhau theo loại nội dung

Bảng này minh họa cùng một lượng ký tự nhưng số token có thể chênh lệch gần năm lần. Văn xuôi thông thường thường “dễ nén” thành token hơn. JSON, bundle đã minify, Base64, UUID hoặc git SHA thường có mật độ token cao hơn.

Điều này giải thích vì sao dán một đoạn log, một payload JSON hoặc dữ liệu Base64 vào prompt có thể đắt hơn cảm giác khi nhìn bằng mắt. Chúng ta không nên ước lượng bằng số dòng hoặc số ký tự; nếu cần quyết định chính xác thì phải đo bằng tokenizer phù hợp với model đang dùng.

Các con số trên slide chỉ là mẫu minh họa. Điều quan trọng là thứ tự tương đối: dữ liệu có cấu trúc lộn xộn, chuỗi hiếm và nội dung máy sinh thường đáng kiểm tra trước.

*Chuyển ý:* Sau khi biết token được tạo ra thế nào, bước tiếp theo là hiểu token được tính tiền ở đâu.

## Slide 5 — Một lần gọi model tính tiền ở cả hai chiều

Mỗi lần gọi model có hai phía. Phía input là context được gửi vào model. Kết quả tool của vòng trước cũng trở thành input ở vòng sau. Cache có thể giảm đơn giá cho phần được sử dụng lại, nhưng phần đó vẫn nằm trong context window.

Phía output gồm phần suy luận nếu nhà cung cấp tính, yêu cầu gọi tool do model sinh ra và câu trả lời cuối. Ở các ví dụ trong bảng, đơn giá output cao hơn input nhiều lần. Giá cụ thể có thể thay đổi, nhưng đây là lý do chúng ta nên tránh cả hai loại lãng phí: gửi vào quá nhiều và để model sinh ra quá nhiều.

Một lưu ý quan trọng: câu trả lời cuối ngắn chưa chắc làm toàn bộ task rẻ hơn. Nếu để có câu trả lời ngắn đó, agent đã gọi model nhiều lần với context ngày càng lớn, tổng chi phí vẫn cao.

*Chuyển ý:* Để biết ai kiểm soát input và số lần gọi, cần tách model ra khỏi agent.

## Slide 6 — Model chỉ biến payload thành output

Bản thân model chỉ nhận một payload hiện tại và sinh ra output. Nó không tự có bộ nhớ làm việc bền vững, không trực tiếp đọc file và cũng không tự chạy command.

Khi model sinh ra một tool call, đó mới chỉ là mô tả một hành động mong muốn. Một thành phần khác phải kiểm tra lời gọi, thực thi tool, thu kết quả rồi quyết định đưa phần nào trở lại model.

Sự phân biệt này rất quan trọng, vì nếu chúng ta chỉ tối ưu prompt gửi cho model mà bỏ qua phần điều phối xung quanh thì sẽ bỏ sót phần lớn nguyên nhân làm token tăng.

*Chuyển ý:* Thành phần điều phối đó chính là harness.

## Slide 7 — Harness biến model thành agent

Harness là phần mềm nằm giữa người dùng, model và môi trường làm việc. Nó nhận task, lắp context, gọi model, chạy tool, giữ history, compact và quyết định ranh giới của phiên.

Nói cách khác, model tạo ra khả năng suy luận; harness biến khả năng đó thành một agent có thể làm việc với code và tool. Harness quyết định gửi gì, gọi model khi nào và giữ lại điều gì.

Đây cũng là tin tốt: nhiều nguyên nhân làm tốn token không phải là đặc tính bất biến của model. Chúng nằm ở workflow và cấu hình client mà chúng ta có thể thay đổi.

*Chuyển ý:* Nơi mọi quyết định của harness hội tụ là context window.

## Slide 8 — Context window là bộ nhớ làm việc của một lần gọi model

Context window là một ngân sách hữu hạn. System content, tool schemas, user message, history, file, kết quả tool và phần output còn lại đều cùng cạnh tranh không gian.

Nội dung không liên quan vẫn chiếm chỗ. Lịch sử cũng có thể được mang qua nhiều lần gọi model, nên một kết quả tool lớn không chỉ tốn token một lần; nó có thể tiếp tục được gửi lại ở các vòng sau.

Vì vậy, “nhiều context hơn” không mặc định là “model hiểu tốt hơn”. Context có ích khi nó liên quan, cập nhật và đủ để giải quyết task hiện tại. Context cũ hoặc trùng lặp vừa tăng chi phí, vừa làm tín hiệu quan trọng khó nổi bật.

*Chuyển ý:* Cache giúp một phần trong số token lặp lại rẻ hơn, nhưng không thay thế việc dọn context.

## Slide 9 — Cache giảm chi phí cho nội dung được sử dụng lại

Ở lần gọi model thứ hai, các phần như system, rules, tool schemas hoặc history có thể trùng với lần trước. Nếu provider và harness hỗ trợ cache đúng cách, phần trùng đó có thể được tính với đơn giá thấp hơn.

Nhưng cache không làm context nhỏ đi. Nội dung được cache vẫn chiếm không gian và vẫn có thể làm model phải xử lý nhiều thông tin không cần thiết. Vì vậy có hai việc khác nhau: dùng cache để giảm giá của phần cần tái sử dụng, và tối ưu context để loại bỏ phần không nên tiếp tục tồn tại.

Đừng xem cache như giấy phép để giữ mọi thứ mãi trong phiên. Nó là một tối ưu về đơn giá, không phải một cơ chế quản lý vòng đời thông tin.

*Chuyển ý:* Bây giờ chúng ta ghép các khái niệm này vào toàn bộ vòng đời của một yêu cầu.

## Slide 10 — Vòng đời sau khi người dùng gửi yêu cầu

Đây là sơ đồ trung tâm của phần còn lại. Người dùng đưa task vào. Harness xây context và gọi model. Nếu model cần tool, harness thực thi tool, chọn phần kết quả cần giữ rồi gọi model lại. Vòng này có thể lặp nhiều lần trước khi có final response.

Chúng ta sẽ đi qua sáu vị trí có thể kiểm soát: yêu cầu đầu vào; bước client xây context; lần gọi model; quyết định chọn tool; cạnh tool trả kết quả vào context; và vòng lặp retry.

Mỗi slide tiếp theo có cùng cấu trúc: bên trái là nguyên nhân phía client có thể xử lý, bên phải là giải pháp ưu tiên. Tên tool, nếu có, được đặt ở cuối slide để thể hiện nó chỉ là một cách triển khai giải pháp, không phải mục tiêu tự thân.

*Chuyển ý:* Bắt đầu từ nơi sớm nhất và thường rẻ nhất để sửa: yêu cầu đầu vào.

## Slide 11 — Yêu cầu đầu vào quyết định số vòng lặp phía sau

Một yêu cầu mơ hồ không trực tiếp tạo ra token, nhưng nó khuếch đại việc tìm kiếm, hỏi lại, sửa lại và chạy test lại. Đặc biệt tốn kém là gộp nhiều việc không liên quan, đưa ràng buộc quá muộn hoặc không nói rõ điều kiện hoàn thành.

Ví dụ “sửa auth, tiện thể refactor module và bổ sung test” mở ra rất nhiều cách hiểu. Phiên bản tốt hơn chỉ rõ file, phạm vi, ràng buộc dependency và điều kiện test phải qua. Agent không cần đoán đâu là phần được phép thay đổi và khi nào nên dừng.

Một nguyên tắc thực dụng là một task cho một phiên. Nếu kết quả có thể được xác định bằng compiler hoặc test, hãy để agent chạy công cụ đó thay vì yêu cầu model dự đoán.

*Chuyển ý:* Yêu cầu rõ vẫn chưa đủ nếu client tiếp tục đưa quá nhiều thứ vào context.

## Slide 12 — Những gì client đưa vào context đều trở thành input

Ở bước build context, các nguyên nhân dễ kiểm soát nhất là để một phiên kéo dài qua nhiều task, giữ instruction files dài hoặc trùng lặp, bật quá nhiều tool và MCP, hoặc tiếp tục mang theo kết quả tool đã cũ.

Phần nguy hiểm là các nội dung này không chỉ xuất hiện một lần. Nếu chúng nằm trong history, chúng có thể được gửi lại ở nhiều lần gọi model tiếp theo.

Giải pháp là mở phiên mới khi đổi sang task ít liên quan; giữ instruction files ngắn; tắt tool không dùng; compact tại ranh giới công việc rõ ràng; và lưu kết quả quan trọng ra file trước khi restart. Mục tiêu không phải xóa mọi context, mà giữ đúng phần cần cho task hiện tại.

*Chuyển ý:* Sau khi context đã được xây, chúng ta vẫn có thể lãng phí ngay trong lần gọi model.

## Slide 13 — Lãng phí từ phạm vi xây dựng và độ dài câu trả lời

Tại node gọi model có hai loại lãng phí khác nhau. Loại thứ nhất là agent xây quá nhiều: tạo thêm abstraction, scaffolding hoặc thay đổi ngoài phạm vi. Loại thứ hai là model nói quá nhiều: tường thuật tiến độ, mở bài dài hoặc in cả file khi chỉ cần diff.

Giải pháp phía trên vì vậy cũng tách làm hai: giới hạn rõ phạm vi code được phép tạo và đặt contract đầu ra ngắn, tập trung vào kết quả. Output cap và verbosity nên được chọn theo loại task, không nên dùng một mức cố định cho mọi trường hợp.

Hai công cụ ở cuối slide nằm cùng node nhưng giải quyết hai việc khác nhau. Ponytail là ruleset buộc agent ưu tiên tái sử dụng và chỉ xây phần tối thiểu cần thiết. Caveman là một skill rút ngắn phần văn xuôi; nó không nén code, diff hay kết quả tool. Nói ngắn gọn: Ponytail kiểm soát phần được xây, Caveman kiểm soát phần được nói.

*Chuyển ý:* Nếu model cần tool, lựa chọn cách tìm thông tin cũng quyết định số vòng lặp.

## Slide 14 — Chọn sai cách tìm code làm tăng số vòng tool

Trong repo lớn, agent có thể liên tục search từ khóa, đọc file, phát hiện thiếu một quan hệ rồi lại search tiếp. Điều này xảy ra khi agent chưa biết symbol liên quan, caller, callee hoặc phạm vi ảnh hưởng.

Trước hết, nếu người dùng đã biết vị trí thì nên chỉ đúng file hoặc symbol. Nếu chưa biết, hãy thu hẹp search trước khi đọc nguyên file. Và với câu hỏi có đáp án tất định, như code có compile hay test có qua không, hãy dùng compiler hoặc test.

CodeGraph phù hợp khi nút thắt là hiểu quan hệ trong repo lớn. Nó xây một graph code local để agent hỏi symbol, caller và impact trước, sau đó chỉ đọc những file thực sự cần. Với repo nhỏ hoặc code phụ thuộc nhiều vào reflection và hành vi động, lợi ích có thể không đủ lớn.

*Chuyển ý:* Chọn đúng tool giúp giảm số vòng; nhưng kết quả tool trả về vẫn có thể làm context phình ra.

## Slide 15 — Tool output dài trở thành input ở vòng tiếp theo

Một test log tám nghìn dòng có thể chỉ chứa vài dòng lỗi thực sự hữu ích. Nếu toàn bộ log được append vào history, lần gọi model tiếp theo phải nhận lại toàn bộ phần đó. Điều tương tự xảy ra với nguyên file, API response lớn hoặc search result không phân trang.

Giải pháp tốt nhất là giảm dữ liệu trước khi nó vào context: lọc tại nguồn, giới hạn log, chỉ giữ error summary, exit code, test name và các dòng liên quan. Tuy nhiên, không nên đặt trần quá thấp đến mức agent thiếu dữ liệu rồi phải chạy lại tool; khi đó chúng ta chỉ đổi một payload lớn thành nhiều vòng gọi nhỏ nhưng tổng chi phí có thể còn tệ hơn.

RTK nằm trên cạnh tool trả kết quả. Nó lọc, gom nhóm, cắt và khử trùng lặp shell output trước khi output đi vào context. Công cụ này phù hợp khi harness đang đưa raw build, test hoặc linter output vào model; nếu harness đã có cơ chế giới hạn tốt thì không nhất thiết phải thêm RTK.

*Chuyển ý:* Sau khi kết quả đã được append, rủi ro cuối cùng là tiếp tục lặp mà không tạo thêm thông tin mới.

## Slide 16 — Mỗi vòng lặp mang context cũ quay lại model

Retry chỉ có giá trị khi vòng mới có giả thuyết mới hoặc thêm bằng chứng mới. Nếu agent lặp lại cùng một cách sửa và cùng một kiểu thất bại, mỗi vòng chỉ kéo context cũ quay lại model thêm một lần.

Một quy tắc vận hành dễ áp dụng là: cùng một cách thất bại hai lần thì dừng và lập kế hoạch lại. Có thể đổi giả thuyết, bỏ kết quả cũ hoặc mở phiên mới với một handoff ngắn.

Compact cũng cần đúng thời điểm. Không compact giữa lúc đang điều tra lỗi vì có thể làm mất manh mối và khiến agent đọc lại. Hãy compact sau khi hoàn thành một phần rõ ràng, và lưu checkpoint hoặc handoff trước khi restart.

*Chuyển ý:* Sáu nhóm giải pháp vừa rồi khá nhiều, nên ba slide cuối sẽ chuyển chúng thành checklist có thể dùng ngay.

## Slide 17 — Checklist workflow

Checklist đầu tiên đi theo vòng đời của một phiên.

Khi mở phiên: một task, phạm vi rõ, điều kiện hoàn thành rõ và ràng buộc xuất hiện ngay từ đầu. Trong phiên: đưa đường dẫn thay vì dán nội dung agent có thể tự đọc; chỉ đưa dòng lỗi liên quan; để compiler và test trả lời các câu hỏi tất định; và tránh thay đổi rules, model hoặc tool setup giữa task.

Khi không tiến triển: dừng sau hai lần thất bại cùng kiểu, chỉ compact ở ranh giới rõ ràng và lưu kết quả quan trọng trước khi restart. Khi task kết thúc: ghi lại đã đổi gì, đã kiểm tra gì, còn gì chưa xong; nếu task tiếp theo ít liên quan thì mở phiên mới.

Không nhất thiết áp dụng tất cả cùng lúc. Chỉ cần nhóm thống nhất vài quy tắc workflow này, số vòng lặp không cần thiết đã có thể giảm đáng kể.

*Chuyển ý:* Checklist thứ hai dành cho người cấu hình harness.

## Slide 18 — Checklist cấu hình harness

Ưu tiên đầu tiên là trần output tool: đủ để giữ phần chẩn đoán cần thiết nhưng không mang cả log vào context. Tiếp theo là điểm compact: không quá sớm, cũng không đợi đến khi context chạm trần.

Reasoning effort và output verbosity nên ở mức vừa phải cho công việc thường ngày, chỉ tăng khi task thật sự cần. Tool và MCP không dùng nên được tắt hoặc nạp theo nhu cầu. Model nên được chọn phù hợp ngay từ đầu phiên để tránh thay đổi hành vi giữa task.

Cuối cùng, instruction files như `CLAUDE.md`, `AGENTS.md` hoặc rules file nên ngắn và không trùng lặp. Nếu có tài liệu chi tiết, hãy dẫn tới tài liệu đó thay vì nhúng toàn bộ vào instruction file được gửi đi lặp lại.

*Chuyển ý:* Checklist cuối cùng trả lời câu hỏi thực tế nhất: nên dùng tool nào trong trường hợp nào.

## Slide 19 — Chọn tool theo vấn đề cần giải quyết

Đừng bắt đầu bằng câu hỏi “tool nào tiết kiệm token nhất”. Hãy bắt đầu bằng vấn đề đang xảy ra.

Nếu agent xây quá phạm vi, cân nhắc Ponytail. Nếu phần văn xuôi quá dài, cân nhắc Caveman hoặc cơ chế verbosity có sẵn của harness. Nếu agent liên tục search và đọc file trong repo lớn, cân nhắc CodeGraph. Nếu raw shell output quá lớn, cân nhắc RTK hoặc trần output sẵn có.

Bảng phía dưới cho thấy cùng một tool không phù hợp như nhau với mọi harness. Ví dụ RTK không nên chồng lên một harness đã truncate shell output tốt; còn CodeGraph chỉ đáng dùng khi quy mô và cấu trúc repo tạo ra nhu cầu hiểu quan hệ. Mục tiêu là chọn ít tool nhất nhưng đúng nút thắt nhất.

*Chuyển ý:* Tôi xin kết thúc bằng chính sơ đồ vòng đời, để mọi người có thể chọn điểm muốn trao đổi sâu hơn.

## Slide 20 — Q&A

Nếu chỉ giữ lại một ý, tôi muốn đó là: token không chỉ nằm trong prompt và final response. Token tích lũy qua toàn bộ vòng đời — từ cách viết task, cách client xây context, số lần gọi model, lựa chọn tool, kích thước tool output cho đến cách retry.

Sáu vị trí trên sơ đồ tương ứng với sáu nhóm nguyên nhân và giải pháp mà chúng ta vừa đi qua. Trong phần hỏi đáp, mọi người có thể chọn trực tiếp một node hoặc một cạnh để quay lại slide chi tiết.

Repo ở cuối slide chứa tài liệu và các tool được nhắc tới. Cảm ơn mọi người. Chúng ta muốn bắt đầu trao đổi từ vị trí nào trong vòng đời này?

## Gợi ý trả lời một số câu hỏi thường gặp

**“Có phải cứ giảm token là tốt?”**  
Không. Mục tiêu là giảm token không cần thiết trong khi vẫn hoàn thành task đúng chất lượng. Giới hạn quá thấp có thể khiến agent thiếu thông tin và phải gọi lại nhiều lần.

**“Cache có giải quyết được context lớn không?”**  
Không hoàn toàn. Cache giảm đơn giá cho phần được sử dụng lại; nội dung đó vẫn chiếm context window. Context không liên quan vẫn nên được loại bỏ.

**“Có nên cài cả bốn tool?”**  
Không. Chỉ chọn tool khớp với vấn đề đang thấy trên harness và repo của mình. Nếu harness đã có giải pháp tương đương thì không nên chồng thêm một lớp khác.

**“Caveman có phải tool nén output không?”**  
Không. Caveman là một skill/ruleset hướng model viết văn xuôi ngắn hơn; nó không nén code, diff hoặc kết quả tool.

**“Khi nào nên mở phiên mới?”**  
Khi task mới ít liên quan đến task cũ, hoặc khi vòng lặp hiện tại không còn tạo thêm thông tin mới. Trước khi mở phiên mới, ghi một handoff ngắn nếu còn việc cần tiếp tục.
