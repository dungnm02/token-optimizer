# Chuẩn bị trước buổi trình bày — "Tiết kiệm token cho coding agent"

Gửi danh sách này cho người tham dự trước buổi ít nhất một ngày. Phần bắt buộc
mất khoảng 15–20 phút; làm xong thì mọi bài tập trong buổi đều áp dụng được
ngay trên setup thật của bạn thay vì chỉ nghe lý thuyết.

## Bắt buộc (15–20 phút)

1. **Cài sẵn một coding agent và đăng nhập được.**
   Lý tưởng là Claude Code — mọi con số trong bộ slide được đo trên nó.
   Codex CLI, Gemini CLI hoặc Cline cũng dùng được. Cập nhật lên bản mới
   nhất, vì Phần 1 so sánh năng lực harness theo bản hiện hành.

2. **Biết mình đang trả tiền kiểu gì.**
   API tính theo token hay gói thuê bao? Mở được trang xem usage/chi phí
   chưa (ví dụ lệnh `/cost` trong Claude Code)? Không biết hóa đơn của
   mình ở đâu thì cả buổi chỉ là lý thuyết.

3. **Chạy thử một tác vụ quen thuộc và ghi lại số token/chi phí.**
   Chọn một việc bạn hay giao cho agent (sửa một bug, thêm một endpoint…),
   chạy như bình thường, ghi lại con số. Đây là baseline "trước" — slide
   kết luận yêu cầu *đo lại trước khi cài thêm*, nên phải có số để so.

4. **Mang theo một repo thật đang làm việc hằng ngày.**
   Không phải repo demo. Các khuyến nghị như CodeGraph chỉ có nghĩa với
   repo >1.000 file, và phần "vặn nút cấu hình" cần thử trên project thật.

## Khuyến khích (5 phút)

5. **Liệt kê MCP server / extension đang bật** (`/mcp` hoặc xem file
   config). Trong buổi có bước "tắt server MCP không dùng hôm nay" —
   chuẩn bị sẵn danh sách thì làm ngay được.

6. **Biết file cấu hình của agent mình nằm ở đâu**
   (`settings.json`, `config.toml`, `.clinerules`…). Phần 4 —
   "Cùng một nút, bốn cái tên" — sẽ chỉ vào đúng các file này.

## Không cần chuẩn bị

- **Không cần đọc trước tài liệu gì.** Bộ slide đứng độc lập; thuật ngữ
  nền (token, context, hook, MCP…) được giải thích ở hai slide đầu.
- **Không cần kiến thức prompt engineering.** Đây là buổi nhập môn.
- **Không cài trước Ponytail, CodeGraph hay công cụ nào khác.** Kết luận
  của buổi chính là *đừng cài gì trước khi xét harness của bạn* — cài
  trước là đi ngược thông điệp.
