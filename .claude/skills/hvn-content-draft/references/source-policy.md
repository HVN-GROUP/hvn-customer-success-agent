# Chính sách nguồn

Áp dụng theo Quy định an toàn bảo mật thông tin 01-01/11/2024/QyĐ và theo quyết định của CEO (Validation Log, Session 2): plugin công khai **không kết nối và không dùng dữ liệu WHMCS**.

## Được phép

| Nguồn | Điều kiện |
|---|---|
| Tài liệu chính thức của hãng (Google, Microsoft, Anthropic, OpenAI, Acronis, Kaspersky, Sectigo, DigiCert…) | Domain có trong `config/link-allowlist.yaml` của repo staging |
| Văn bản pháp luật chính thức (thuvienphapluat.vn, vanban.chinhphu.vn…) | Domain có trong allowlist |
| Outline HVN | Chỉ phần quy trình chuyên môn ở cấp Công khai hoặc Nội bộ không chứa dữ liệu khách hàng |
| dxt.vn | Nội dung khóa học công khai |
| Kinh nghiệm chuyên môn của người duyệt Service (#Witch) | Viết dưới dạng tình huống chung, không nêu khách hàng cụ thể |

## Bị cấm

- Mọi dữ liệu WHMCS: ticket, thông tin khách hàng, dịch vụ, đơn hàng, hóa đơn.
- Các connector `ID_HVN` và `ID_DEV_HVN`.
- Bảng giá vendor, giá gốc, chiến lược định giá, chính sách chiết khấu.
- Hợp đồng đối tác, dữ liệu nhân sự, kế hoạch marketing chi tiết.
- Mọi tài liệu gắn nhãn "Bí mật" hoặc "Tuyệt mật".

## Khi biên tập viên đưa nguồn bị cấm

Từ chối dùng nguồn đó. Giải thích ngắn rằng plugin là sản phẩm công khai và CEO đã quyết định không dùng dữ liệu WHMCS, rồi đề xuất nguồn thay thế trong bảng "Được phép".
