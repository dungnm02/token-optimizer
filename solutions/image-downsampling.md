# Giảm độ phân giải Ảnh & Ngân sách Thị giác (Tiếng Việt)

**Giải quyết:** Nguyên nhân 4.1 trong [`../CAUSE.md`](../CAUSE.md)

**Ý tưởng:** Hãy đưa mọi hình ảnh về độ phân giải nhỏ nhất mà tác vụ thực sự
cần, trước khi đưa vào request. Ở mọi nhà cung cấp lớn, chi phí token thị
giác đều tỷ lệ thuận với diện tích pixel/tile, nên độ phân giải chính là
một nút điều chỉnh chi phí trực tiếp.

---

## Mô hình chi phí (biết công thức của nhà cung cấp bạn dùng)

| Nhà cung cấp | Mô hình chi phí (xấp xỉ) | Đòn bẩy thực tế |
| --- | --- | --- |
| Anthropic | ~`(chiều rộng × chiều cao) / 750` token; các model độ phân giải cao chấp nhận tới cạnh dài 2576px (tới ~4.784 token/ảnh) | Giảm cạnh dài; 1568px khớp với giới hạn cũ (~1.600 token) |
| OpenAI | Dựa trên tile: chi phí gốc + chi phí mỗi tile 512px ở `detail: high`; cố định ~85 token ở `detail: low` | `detail: low` cho kiểm tra phân loại/hiện diện; resize để giảm số tile |
| Gemini | Dạng tile: ~258 token mỗi tile 768×768 | Vừa trong ít tile hơn |

Hai hệ quả:

1. Với cùng một nội dung *đọc được*, gửi screenshot ở 4K thay vì 1080p có
   thể tốn nhiều hơn **3–5×** số token.
2. Hình ảnh tồn tại lâu trong lịch sử hội thoại (nguyên nhân 2.1). Vì vậy
   một screenshot quá khổ sẽ bị tính phí lại ở mọi lượt sau đó, cho tới khi
   nó bị cắt tỉa.

## Cách áp dụng

1. **Luôn giảm độ phân giải tại ranh giới harness.** Cho mọi hình ảnh đi
   qua một bước resize trước khi tạo request:
   - Screenshot chung / vòng lặp computer-use: **1080p** là điểm cân bằng
     hiệu năng/chi phí trên các model hiện tại; **720p / 1366×768** cho các
     vòng lặp nhạy cảm về chi phí.
   - Tài liệu nhiều văn bản: giữ đủ DPI để OCR đọc được (~1.500px cạnh dài
     thường là đủ); kiểm tra trên các font khó đọc nhất.
   - Phân loại / "có hộp thoại trên màn hình không": chế độ chi tiết thấp
     của nhà cung cấp (OpenAI `detail: low` ≈ 85 token) hoặc giảm độ phân
     giải mạnh tay.
2. **Cắt (crop) trước khi gửi.** Nếu đã biết vùng quan tâm (bounding box
   của một phần tử, một vùng diff), hãy cắt vào đúng vùng đó — một crop
   300×200 chỉ tốn ~80 token so với hàng nghìn token cho toàn màn hình. Với
   các model thị giác agentic, có thể trang bị cho chúng một tool crop/zoom
   cùng một bản tổng quan toàn khung thu nhỏ, thay vì gửi một ảnh khổng lồ.
3. **Cắt tỉa screenshot cũ khỏi lịch sử.** Trong các vòng lặp computer-use,
   thay các screenshot cũ hơn 2–3 bước bằng một stub ("[screenshot bước
   12 — đã bị thay thế]") — trạng thái màn hình hiện tại mới là điều quan
   trọng. Đây là trường hợp thị giác của `context-editing.md`.
4. **Đừng gửi lại hình ảnh không đổi.** Hash các khung hình; nếu màn hình
   không đổi, hãy nói điều đó bằng văn bản thay vì đính kèm một screenshot
   giống hệt (`context-hygiene.md`).
5. **Ưu tiên văn bản khi có văn bản tồn tại.** Trích xuất DOM/cây trợ năng
   cho các agent trình duyệt, lớp văn bản PDF cho tài liệu — văn bản có
   cấu trúc thường rẻ hơn 5–20× so với pixel của cùng nội dung, và được
   phân tích đáng tin cậy hơn. Dùng thị giác khi bố cục/hình thức mới là
   trọng tâm.

## Công cụ hiện đại nhất (SOTA)

### Có sẵn — coding agent & API của nhà cung cấp

| Nhà cung cấp / agent | Tính năng | Ghi chú |
| --- | --- | --- |
| OpenAI API | Nút `detail: low\|high\|auto` theo từng ảnh | `low` cố định ~85 token — chế độ kiểm tra hiện diện rẻ nhất ở mọi nơi |
| Mọi nhà cung cấp | `count_tokens` trên các ảnh đại diện | Hiệu chỉnh lại sau khi nâng cấp model — công thức token ảnh thay đổi giữa các thế hệ |
| Tool computer-use của Anthropic / OpenAI | Hướng dẫn độ phân giải đã ghi tài liệu | Tuân theo độ phân giải chụp được nhà cung cấp khuyến nghị thay vì kích thước màn hình gốc |

### Bên thứ ba — không phụ thuộc agent (ưu tiên mã nguồn mở)

| Công cụ | Giấy phép | Ghi chú |
| --- | --- | --- |
| sharp (Node) / Pillow-SIMD (Python) / libvips | Apache-2.0 / PIL / LGPL | Resize/crop nhanh tại ranh giới harness, trước khi bất kỳ nhà cung cấp nào thấy ảnh |
| Cây trợ năng Playwright / trích xuất DOM của `browser-use` | Apache-2.0 / MIT | Biểu diễn trang ưu tiên văn bản cho agent trình duyệt; screenshot chỉ dùng dự phòng |
| pixelmatch / băm tri giác (perceptual hashing) | ISC / MIT | Bỏ qua các khung hình không đổi trong vòng lặp screenshot |

## Đánh đổi

- Giảm độ phân giải quá mạnh sẽ phá hỏng khả năng OCR và định vị các phần
  tử UI nhỏ; độ chính xác tọa độ khi click trong computer-use cũng giảm
  sút dưới ~720p. Hãy tự hiệu chỉnh trên UI thực tế của bạn.
- Bước resize thêm vài ms mỗi ảnh và một phụ thuộc xử lý.
- Các luồng dựa trên crop cần theo dõi vùng chính xác; một lần crop sai
  buộc phải chụp lại (thêm một round-trip).

## Tác động dự kiến

- Giảm độ phân giải screenshot 4K/gốc xuống 1080p thường cắt giảm input
  thị giác **3–5×**, mà độ chính xác tác vụ hầu như không giảm trên các UI
  tiêu chuẩn.
- Trong các vòng lặp computer-use chụp mỗi bước, kết hợp chụp 1080p + cắt
  tỉa screenshot cũ thường xuyên giảm tổng input phiên **50–80%**, vì
  screenshot chiếm phần lớn các transcript đó.
- Chuyển các agent trình duyệt từ ưu tiên screenshot sang ưu tiên DOM cắt
  giảm chi phí mỗi bước **cả một bậc độ lớn** trên các trang nhiều văn
  bản.

---

# Image Downsampling & Vision Budgeting

**Addresses:** Cause 4.1 in [`../CAUSE.md`](../CAUSE.md)

**Idea:** Size every image to the smallest resolution the task actually
needs before it enters the request — vision token cost scales with pixel
area/tiles on every major provider, so resolution is a direct cost dial.

---

## The cost model (know your provider's formula)

| Provider | Cost model (approximate) | Practical lever |
| --- | --- | --- |
| Anthropic | ~`(width × height) / 750` tokens; high-res models accept up to 2576px long edge (up to ~4,784 tokens/image) | Downscale long edge; 1568px matches the older cap (~1,600 tokens) |
| OpenAI | Tile-based: base cost + per-512px-tile cost at `detail: high`; flat ~85 tokens at `detail: low` | `detail: low` for classification/presence checks; resize to reduce tile count |
| Gemini | Tiled: ~258 tokens per 768×768 tile | Fit within fewer tiles |

Two consequences:

1. A screenshot sent at 4K vs 1080p can differ by **3–5×** in tokens for the
   same *legible* content.
2. Images persist in history (cause 2.1) — an oversized screenshot is
   re-billed on every subsequent turn until pruned.

## How to apply

1. **Downscale at the harness boundary, always.** Route every image through
   a resize step before request assembly:
   - General screenshots / computer-use loops: **1080p** is the
     performance/cost sweet spot on current models; **720p / 1366×768** for
     cost-sensitive loops.
   - Text-heavy documents: keep enough DPI for OCR-legibility (~1,500px long
     edge is usually sufficient); test on your worst fonts.
   - Classification / "is there a dialog on screen": provider low-detail
     modes (OpenAI `detail: low` ≈ 85 tokens) or aggressive downscale.
2. **Crop before you send.** If the region of interest is known (an element
   bounding box, a diff region), crop to it — a 300×200 crop is ~80 tokens
   vs thousands for the full screen. Agentic vision models can also be given
   a crop/zoom tool and a small full-frame overview instead of one huge
   image.
3. **Prune stale screenshots from history.** In computer-use loops, replace
   screenshots older than 2–3 steps with a stub ("[screenshot step 12 —
   superseded]") — the current screen state is what matters. This is the
   vision case of `context-editing.md`.
4. **Don't re-send unchanged images.** Hash frames; if the screen didn't
   change, say so in text instead of attaching an identical screenshot
   (`context-hygiene.md`).
5. **Prefer text when text exists.** DOM/accessibility-tree extraction for
   browser agents, PDF text layers for documents — structured text is
   usually 5–20× cheaper than pixels of the same content, and more reliably
   parsed. Use vision when layout/appearance is the point.

## SOTA tools

### Native — coding agents & provider APIs

| Provider / agent | Feature | Notes |
| --- | --- | --- |
| OpenAI API | `detail: low\|high\|auto` per-image dial | `low` is a flat ~85 tokens — the cheapest presence-check mode anywhere |
| All providers | `count_tokens` on representative images | Re-baseline after model upgrades — image token formulas change between generations |
| Anthropic / OpenAI computer-use tools | Documented resolution guidance | Follow the vendor's recommended capture resolution rather than native screen size |

### Third-party — agent-agnostic (open source preferred)

| Tool | License | Notes |
| --- | --- | --- |
| sharp (Node) / Pillow-SIMD (Python) / libvips | Apache-2.0 / PIL / LGPL | Fast resize/crop at the harness boundary, before any provider sees the image |
| Playwright accessibility tree / `browser-use` DOM extraction | Apache-2.0 / MIT | Text-first page representation for browser agents; screenshots only as fallback |
| pixelmatch / perceptual hashing | ISC / MIT | Skip unchanged frames in screenshot loops |

## Trade-offs

- Over-downscaling breaks OCR and small-UI-element grounding; coordinate
  precision for computer-use clicks degrades below ~720p. Calibrate on your
  actual UI.
- A resize step adds a few ms per image and a processing dependency.
- Crop-based flows need correct region tracking; a wrong crop forces a
  re-shot (extra round trip).

## Expected impact

- Downscaling 4K/native screenshots to 1080p typically cuts vision input
  **3–5×** with negligible task-accuracy loss on standard UIs.
- In screenshot-per-step computer-use loops, combining 1080p capture +
  stale-screenshot pruning routinely reduces total session input by
  **50–80%**, since screenshots dominate those transcripts.
- Switching browser agents from screenshot-primary to DOM-primary
  representations cuts per-step cost by **an order of magnitude** on
  text-dense pages.
