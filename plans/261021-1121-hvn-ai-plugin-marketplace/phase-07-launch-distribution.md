---
phase: 7
title: "Ra mắt & phân phối"
status: todo
priority: P1
effort: "1.5w (tuần 6–8)"
dependencies: [4, 5, 6]   # Phase 4 chỉ cần đã nộp hoặc đã có quyết định lùi, không cần chờ duyệt
---

# Phase 7: Ra mắt & phân phối

<!-- Updated: Red Team 2026-10-21 - ánh xạ theo kênh; mọi tin nhắn chỉ dẫn về hvn.vn/ai-plugin; hướng dẫn bật tự cập nhật và cách nhận biết HVN thật; thử cài trên nhiều loại tài khoản; sao lưu mẫu email; baseline lưu ngoài git -->

## Goal

Đưa link cài plugin tới đúng khách hiện hữu theo kênh AI họ dùng, qua email WHMCS và Zalo OA. Mọi tin nhắn chỉ trỏ về `hvn.vn/ai-plugin`. Ra mắt marketplace Claude đúng hạn, và bổ sung ChatGPT khi plugin được duyệt. Có baseline để đo ở mốc 90 ngày.

## Ánh xạ sản phẩm đã mua sang nội dung giới thiệu

| Nhóm sản phẩm WHMCS | Nội dung trong email và tin nhắn |
|---|---|
| Claude (Team/Enterprise/Pro) | Marketplace Claude: hvn-claude + hvn-advisor |
| ChatGPT | Plugin HVN trên ChatGPT, **chỉ sau khi được duyệt**. Trước đó không gửi. |
| Google Workspace (GCS.VN) | Marketplace Claude (hvn-google-workspace + hvn-gemini + hvn-advisor), kèm plugin ChatGPT nếu đã được duyệt |
| Microsoft 365 (MSO.VN) | Marketplace Claude (hvn-microsoft-365 + hvn-advisor), kèm plugin ChatGPT nếu đã được duyệt |
| SSL (CER.VN), Acronis, Kaspersky, dịch vụ HVN Security | Marketplace Claude (hvn-security + hvn-advisor), kèm plugin ChatGPT nếu đã được duyệt |

Tên nhóm sản phẩm thật được lấy bằng `product_groups_list` và `products_list` của WHMCS lúc triển khai, không đoán trước.

## Files to Create / Modify

- Create (repo staging): `docs/cai-dat.md`, nguồn của trang `hvn.vn/ai-plugin`. Nội dung:
  - **Hộp "Cách nhận biết đúng HVN"**: tên chính thức là `hvngroup/hvn-ai-marketplace`; tổ chức GitHub có huy hiệu domain đã xác minh `hvn.vn`; HVN không bao giờ gửi tên marketplace khác qua Zalo; liên hệ `kythuat@hvn.vn` khi nghi ngờ.
  - Cài trên claude.ai, Desktop và Cowork (Customize > Plugins > Add > Add marketplace), ghi gói Claude hỗ trợ theo kết quả spike Phase 1.
  - Cài trên Claude Code: `/plugin marketplace add hvngroup/hvn-ai-marketplace`, **kèm hướng dẫn bật tự cập nhật** trong `/plugin` > Marketplaces, vì tự cập nhật mặc định tắt.
  - Dành cho admin Team/Enterprise: bật "User-created skills", hoặc mirror repo riêng tư rồi org sync, hoặc upload zip.
  - Cài trên ChatGPT: link plugin khi đã được duyệt, ghi gói ChatGPT hỗ trợ theo kết quả Phase 4.
- Create: trang `hvn.vn/ai-plugin` do Marketing đăng sau khi **CMO duyệt**. Có nút "Sao chép lệnh cài" gắn sự kiện GA4, dùng làm chỉ số lượt cài đại diện (Phase 6).
- Modify (WHMCS admin):
  - **Trước khi sửa**, chụp lại nội dung từng mẫu email sẽ sửa.
  - Thêm khối "Dùng [sản phẩm] hiệu quả hơn với trợ lý AI của HVN" vào email chào mừng và email nhắc gia hạn của các nhóm trong bảng.
  - Link **chỉ** trỏ tới `hvn.vn/ai-plugin` với `utm_source=email&utm_medium=whmcs&utm_campaign=ai-plugin-onboarding` (thuộc enum trong `config/channels.yaml`).
  - Không bao giờ đặt chuỗi `owner/repo` trong email hay Zalo.
- Create: kịch bản broadcast Zalo OA theo từng tập khách, **CMO duyệt**, chỉ chứa link `hvn.vn/ai-plugin`.

## Tasks & Steps

1. **Baseline trước ra mắt**, lưu trên Drive hoặc Outline có phân quyền (CEO, CFO, COO), **không lưu trong git**, đánh dấu "BẢO MẬT":
   - Danh sách từ khóa và chủ đề ticket **cố định** cho các chủ đề plugin phủ.
   - Số ticket 30 ngày gần nhất theo danh sách đó.
   - Số dịch vụ và doanh thu 30 ngày gần nhất của các domain đích.
2. Kiểm tra trước ra mắt:
   - `npm run check -- --freshness`.
   - `claude plugin validate` cho marketplace và từng plugin.
   - Chạy lại eval 7 plugin (ngưỡng như Phase 5).
   - Thử cài từ đầu với tài khoản claude.ai **Free, Pro, thành viên Team, Owner Team**.
   - Thử cài đủ 7 plugin trên **Claude Code**.
3. Đăng `docs/cai-dat.md` và trang `hvn.vn/ai-plugin`.
4. Sửa mẫu email WHMCS. Gửi thử **mỗi mẫu cho mỗi nhóm sản phẩm** tới hộp thư nội bộ, kiểm tra hiển thị và link UTM trên GA4 Realtime. Nếu lỗi, khôi phục từ bản đã chụp.
5. Gửi broadcast Zalo OA theo tập khách (sau khi CMO duyệt).
6. **Khi ChatGPT Plugin được duyệt:** cập nhật trang cài đặt và bật phần ChatGPT trong email của các nhóm tương ứng.
7. Đặt lịch báo cáo mốc 30, 60 và 90 ngày theo bảng nguồn dữ liệu ở Phase 6.

## Verification

- Kết quả thử cài cho 4 loại tài khoản claude.ai và cho Claude Code, ảnh chụp đính trong issue release. Loại tài khoản nào không cài được thì `docs/cai-dat.md` ghi rõ cùng cách thay thế.
- Eval 7 plugin đạt ngưỡng tại ngày release.
- Mỗi mẫu email đã sửa có một lần gửi thử đạt; GA4 Realtime ghi nhận `utm_campaign=ai-plugin-onboarding`.
- `grep -R "hvngroup/" ` trên kịch bản Zalo và nội dung email không có kết quả; chuỗi `owner/repo` chỉ xuất hiện trên trang `hvn.vn/ai-plugin`.
- Baseline tồn tại trên Drive hoặc Outline, có ngày tạo trước ngày ra mắt.

## Success Criteria

- [ ] Marketplace Claude ra mắt; trang `hvn.vn/ai-plugin` có hướng dẫn cho từng loại tài khoản và hộp nhận biết.
- [ ] Email chào mừng và email gia hạn của các nhóm sản phẩm có link tới trang cài đặt; phần ChatGPT chỉ bật sau khi được duyệt.
- [ ] Đã có baseline và danh sách từ khóa cố định.

## Risks

- **Khách Team/Enterprise không thêm được marketplace.** Có hướng dẫn riêng cho admin; khách lớn được Service (#Witch) hỗ trợ trực tiếp.
- **Giả mạo marketplace.** Đã đăng ký các tên GitHub dễ nhầm (Phase 1), có hộp nhận biết, và tin nhắn không bao giờ chứa `owner/repo`.
