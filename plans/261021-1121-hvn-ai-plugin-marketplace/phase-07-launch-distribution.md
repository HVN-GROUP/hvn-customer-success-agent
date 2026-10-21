---
phase: 7
title: "Ra mắt & phân phối"
status: todo
priority: P1
effort: "1.5w (tuần 6–8)"
dependencies: [4, 5, 6]   # Phase 4 chỉ cần ở trạng thái đã nộp, không cần chờ duyệt
---

# Phase 7: Ra mắt & phân phối

## Goal

Đưa link cài plugin tới đúng khách hiện hữu theo sản phẩm họ đã mua, qua email WHMCS và Zalo OA. Ra mắt công khai marketplace trên Claude và app trên ChatGPT. Ghi nhận baseline để đo kết quả sau 90 ngày.

## Ánh xạ sản phẩm đã mua sang plugin

| Nhóm sản phẩm WHMCS | Plugin gửi kèm |
|---|---|
| Claude (Team/Enterprise/Pro) | hvn-claude + hvn-advisor |
| ChatGPT | hvn-chatgpt + hvn-advisor |
| Google Workspace (GCS.VN) | hvn-google-workspace + hvn-gemini + hvn-advisor |
| Microsoft 365 (MSO.VN) | hvn-microsoft-365 + hvn-advisor |
| SSL (CER.VN), Acronis, Kaspersky, dịch vụ HVN Security | hvn-security + hvn-advisor |

Tên nhóm sản phẩm thực tế được lấy bằng `products_list` và `product_groups_list` của WHMCS lúc triển khai, không đoán trước.

## Files to Create / Modify

- Create (repo công khai): `docs/cai-dat.md` — hướng dẫn cài dành cho người không chuyên. Gồm 3 đường:
  - claude.ai/Desktop/Cowork: Customize > Plugins > Add > Add marketplace > `hvngroup/hvn-ai-marketplace`.
  - Claude Code: `/plugin marketplace add hvngroup/hvn-ai-marketplace`.
  - ChatGPT: link app trong directory, hoặc link developer mode nếu app chưa được duyệt.
  - Thêm ảnh chụp từng bước và phần dành cho admin Team/Enterprise: tạo bản mirror repo riêng tư hoặc upload zip, do org sync chỉ nhận repo riêng tư.
- Create: trang `hvn.vn/ai-plugin`, nội dung lấy từ `docs/cai-dat.md`. Đây là đích chung cho mọi email và tin nhắn. Phòng Marketing đăng trên CMS website hiện có. **CMO duyệt trước khi đăng.**
- Modify (WHMCS admin, nhân sự có quyền thực hiện): thêm khối "Dùng [sản phẩm] hiệu quả hơn với trợ lý AI của HVN" vào email chào mừng sản phẩm và email nhắc gia hạn của các nhóm sản phẩm trong bảng ánh xạ. Link mang `utm_source=email&utm_medium=whmcs&utm_campaign=ai-plugin-onboarding`.
- Create: kịch bản broadcast Zalo OA, phân theo tập khách đang dùng từng nhóm sản phẩm. **CMO duyệt.**

## Tasks & Steps

1. Chụp baseline **trước** khi ra mắt, lưu vào `plans/reports/` của repo nội bộ, đánh dấu "BẢO MẬT":
   - Số ticket 30 ngày gần nhất theo chủ đề mà plugin phủ (WHMCS `tickets_list` theo phòng ban hoặc từ khóa).
   - Số đơn hàng và doanh thu 30 ngày gần nhất từ các trang đích.
2. Gắn nhãn "release" cho commit trên `main` của repo công khai. Kiểm tra lần cuối bằng `claude plugin validate`, `npm run check -- --strict` và thử cài lại từ đầu trên claude.ai bằng một tài khoản sạch.
3. Đăng `docs/cai-dat.md` và trang `hvn.vn/ai-plugin` (sau khi CMO duyệt).
4. Cập nhật mẫu email WHMCS. Gửi thử tới hộp thư nội bộ và kiểm tra link UTM trong GA4 Realtime.
5. Gửi broadcast Zalo OA theo từng tập khách (sau khi CMO duyệt).
6. Sau khi ChatGPT App được duyệt (Phase 4), bổ sung link app vào trang cài đặt và vào email.
7. Ghi lịch báo cáo ở mốc 30, 60 và 90 ngày theo chỉ số trong `plan.md`.

## Verification

- Thử cài lại từ đầu trên claude.ai với tài khoản Pro mới: cài được đủ 7 plugin, có ảnh chụp lưu trong issue release.
- Email thử từ WHMCS có link đúng UTM, và GA4 Realtime ghi nhận phiên có `utm_campaign=ai-plugin-onboarding`.
- File baseline tồn tại trước ngày phát hành, có ngày tạo sớm hơn tag release.

## Success Criteria

- [ ] Marketplace công khai được giới thiệu trên trang `hvn.vn/ai-plugin`.
- [ ] Email chào mừng và email gia hạn của 5 nhóm sản phẩm có link cài plugin tương ứng.
- [ ] Đã có baseline để so sánh ở mốc 90 ngày.

## Risks

- **Khách Team/Enterprise không thêm được marketplace,** vì admin tắt "User-created skills" hoặc vì org sync chỉ nhận repo riêng tư. `docs/cai-dat.md` có hướng dẫn riêng cho admin, và khách lớn được Service (#Witch) hỗ trợ trực tiếp.
- **Gói Free của Claude có thể không có "Add marketplace"** [UNVERIFIED]. Trang cài đặt ghi rõ gói yêu cầu sau khi đã thử thực tế ở bước 2.
