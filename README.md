# token-optimizer (Tiếng Việt)

Đây là kho tài liệu để khám phá **nguyên nhân gây tiêu tốn token cao** và
**cách giảm thiểu nó**. Repo này được cố tình thiết kế tối giản: nó chỉ chứa
ghi chú, phân tích, và tài liệu tham khảo — không phải mã ứng dụng.

## Mục đích

- Xác định và ghi lại các nguyên nhân gốc rễ gây tiêu tốn token cao.
- Thu thập các giải pháp cụ thể, khả thi để giảm mức sử dụng token.
- Đóng vai trò là kho kiến thức chung, ngày càng phát triển khi có phát hiện mới.

## Cấu trúc kho lưu trữ

| Đường dẫn | Mô tả |
| --- | --- |
| `README.md` | Tệp này — tổng quan và quy ước. |
| `CAUSE.md` | Danh mục các nguyên nhân gây tiêu tốn token cao đã xác định. |
| `solutions/` | Mỗi tài liệu tương ứng với một giải pháp / chiến lược giảm thiểu. |
| `setups/` | Hướng dẫn thiết lập chuyên biệt cho các nhà cung cấp và công cụ. |

## Cách sử dụng

1. Đọc `CAUSE.md` để hiểu các nguyên nhân đã biết.
2. Duyệt `solutions/` để tìm các giải pháp tương ứng với những nguyên nhân đó.
3. Xem `setups/` để có hướng dẫn thiết lập chuyên biệt cho nhà cung cấp hoặc công cụ cụ thể của bạn.
4. Khi bạn phát hiện điều gì mới, hãy thêm vào `CAUSE.md` và, nếu có cách khắc
   phục, hãy thêm tài liệu tương ứng trong `solutions/`.

## Ghi chú đóng góp

Kho lưu trữ này **chỉ dành cho tài liệu**. Không có bước build, không có
kiểm thử, và không có ứng dụng nào để chạy — chỉ có Markdown.

- **Không cần nhánh (branch).** Làm việc trực tiếp trên `main`.
- **Không cần pull request.** Commit và push thẳng lên `main`.
- Giữ mỗi thay đổi tập trung vào chất lượng và độ chính xác của tài liệu.

Xem [Quy tắc thông điệp commit](#quy-tắc-thông-điệp-commit) bên dưới trước khi commit.

## Quy tắc thông điệp commit

Vì repo này chỉ dành riêng cho tài liệu, thông điệp commit tuân theo một
quy ước đơn giản, nhất quán.

### Định dạng

```
<type>: <tóm tắt ngắn gọn ở thể mệnh lệnh>

<phần thân tùy chọn: cái gì và tại sao, ngắt dòng ở ~72 ký tự>
```

### Type

Sử dụng một trong các tiền tố sau:

| Type | Dùng cho |
| --- | --- |
| `docs` | Thêm hoặc chỉnh sửa nội dung tài liệu (mặc định cho repo này). |
| `cause` | Thêm hoặc cập nhật một mục trong `CAUSE.md`. |
| `solution` | Thêm hoặc cập nhật một tài liệu trong `solutions/`. |
| `chore` | Việc dọn dẹp repo (cấu trúc, đổi tên, định dạng, metadata). |

### Quy tắc

1. **Dòng tiêu đề**
   - Dùng thể mệnh lệnh: "add", "fix", "clarify" — không dùng "added" / "adds".
   - Giữ dưới ~72 ký tự.
   - Viết thường sau tiền tố `type:`; không có dấu chấm cuối câu.
2. **Phạm vi** thông điệp cho một thay đổi logic duy nhất.
3. **Phần thân** (tùy chọn) giải thích *cái gì* đã thay đổi và *tại sao*,
   không phải *cách* thực hiện.
4. Tham chiếu đến tệp bị ảnh hưởng khi điều đó giúp rõ ràng hơn, ví dụ:
   `cause: document large-context re-sends in CAUSE.md`.

### Ví dụ

```
docs: add repository overview and layout
cause: document redundant system-prompt re-sends
solution: add prompt-caching guide
chore: restructure solutions folder
```

---

# token-optimizer

A documentation repository for exploring **what causes high token consumption**
and **how to reduce it**. This repo is intentionally lightweight: it holds
notes, analysis, and reference material — not application code.

## Purpose

- Identify and document the root causes of high token consumption.
- Collect concrete, actionable solutions to reduce token usage.
- Serve as a shared knowledge base that grows as new findings emerge.

## Repository Layout

| Path | Description |
| --- | --- |
| `README.md` | This file — overview and conventions. |
| `CAUSE.md` | Catalog of identified causes of high token consumption. |
| `solutions/` | One document per solution / mitigation strategy. |
| `setups/` | Specialized setup guides for vendors and tools. |

## How to Use

1. Read `CAUSE.md` to understand the known causes.
2. Browse `solutions/` for mitigations mapped to those causes.
3. See `setups/` for specialized setup guides for your vendor or tool.
4. When you discover something new, add it to `CAUSE.md` and, if you have a
   fix, add a matching document under `solutions/`.

## Contributing Notes

This repository is **documentation-only**. There is no build step, no tests,
and no application to run — just Markdown.

- **No branches required.** Work directly on `main`.
- **No pull requests required.** Commit and push straight to `main`.
- Keep every change focused on documentation quality and accuracy.

See [Commit Message Rules](#commit-message-rules) below before committing.

## Commit Message Rules

Because this repo is strictly for documentation, commit messages follow a
simple, consistent convention.

### Format

```
<type>: <short summary in imperative mood>

<optional body: what and why, wrapped at ~72 chars>
```

### Type

Use one of the following prefixes:

| Type | Use for |
| --- | --- |
| `docs` | Adding or editing documentation content (the default for this repo). |
| `cause` | Adding or updating an entry in `CAUSE.md`. |
| `solution` | Adding or updating a document under `solutions/`. |
| `chore` | Repo housekeeping (structure, renames, formatting, metadata). |

### Rules

1. **Subject line**
   - Use the imperative mood: "add", "fix", "clarify" — not "added" / "adds".
   - Keep it under ~72 characters.
   - Lowercase after the `type:` prefix; no trailing period.
2. **Scope** the message to a single logical change.
3. **Body** (optional) explains *what* changed and *why*, not *how*.
4. Reference the affected file when it adds clarity, e.g.
   `cause: document large-context re-sends in CAUSE.md`.

### Examples

```
docs: add repository overview and layout
cause: document redundant system-prompt re-sends
solution: add prompt-caching guide
chore: restructure solutions folder
```
