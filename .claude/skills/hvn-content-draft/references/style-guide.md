# Giọng văn và template nội dung

## Giọng văn HVN GROUP

- **Vai trò:** chuyên gia đáng tin cậy kiêm người đồng hành nhiệt huyết.
- **Câu văn:** ngôn ngữ chủ động, câu ngắn, đi thẳng vào lợi ích và cách làm. Tránh câu bị động, từ mơ hồ, biệt ngữ không giải thích.
- **Thuật ngữ:** giữ nguyên tên tính năng theo giao diện của hãng (ví dụ "Projects", "Admin console"). Lần đầu nhắc tới thì giải thích ngắn bằng tiếng Việt.
- **Cách viết tên:** luôn viết "HVN GROUP", không viết "Hvn Group". Dùng "công ty thành viên", không dùng "công ty con".
- **Số liệu:** chỉ dùng số liệu có nguồn trong `sources`. Không bịa số.
- **Định dạng:** căn lề trái, tiêu đề ngắn, danh sách đánh số cho các bước.
- **Độ dài:** mỗi item từ 150 đến 900 từ. Chủ đề dài thì tách thành nhiều item.

## Template theo loại

### `playbook`

```markdown
# <Tiêu đề>

**Dành cho:** <vai trò / phòng ban / quy mô>
**Kết quả:** <người dùng đạt được gì sau khi làm xong>

## Bối cảnh
<2–3 câu: vấn đề thường gặp và lợi ích>

## Các bước
1. ...

## Prompt mẫu
<1–3 prompt tiếng Việt dùng được ngay, đặt trong code block>

## Kết quả mong đợi
<mô tả đầu ra, tiêu chí "làm đúng">

## Nguồn
- <url>
```

### `cau-hinh` (cấu hình & sự cố)

```markdown
# <Tiêu đề>

## Điều kiện trước
- <quyền cần có, gói dịch vụ cần có>

## Các bước
1. ...

## Kiểm tra kết quả
- <cách xác nhận đã chạy đúng>

## Lỗi thường gặp
| Triệu chứng | Nguyên nhân | Cách sửa |
|---|---|---|

## Nguồn
- <url>
```

### `chon-goi` (không có giá)

```markdown
# <Tiêu đề>

## Tiêu chí chọn
- <quy mô người dùng, nhu cầu lưu trữ, bảo mật, tuân thủ...>

## So sánh tính năng
| Tính năng | Gói A | Gói B |
|---|---|---|

## Gợi ý theo tình huống
- <tình huống> → <gói phù hợp và lý do>

## Nguồn
- <url>
```

Không ghi giá, không ghi khuyến mãi, không giục nâng cấp. Item loại này **không được phát hành trên ChatGPT**, vì `exclude_types: [chon-goi]`.

### `tuan-thu`

```markdown
# <Tiêu đề>

> Nội dung mang tính tham khảo, không thay thế tư vấn pháp lý.

## Quy định liên quan
- <văn bản, điều, khoản>

## Doanh nghiệp cần làm gì
1. ...

## Checklist
- [ ] ...

## Nguồn
- <url văn bản pháp luật chính thức>
```

HVN Security phải approve trước khi merge.

### `lo-trinh` (lộ trình onboard)

```markdown
# Lộ trình 90 ngày dùng <sản phẩm> hiệu quả

**Dành cho:** người mới bắt đầu dùng <sản phẩm>

## Ngày 1 — <mục tiêu>
1. <việc> → [<tiêu đề item>](./<item>.md)

**Hoàn thành khi:** <dấu hiệu>

## Tuần 1 — <mục tiêu>
...

## Tháng 1 — <mục tiêu>
...

## Tháng 3 — <mục tiêu>
...
```

Mỗi mốc có 3–5 việc. Mỗi việc dẫn tới một item **đã có** trong cùng plugin.
