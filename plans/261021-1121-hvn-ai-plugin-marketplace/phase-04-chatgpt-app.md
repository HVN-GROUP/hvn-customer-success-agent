---
phase: 4
title: "ChatGPT Plugin (trước đây gọi là App)"
status: todo
priority: P1
effort: "3d công + thời gian chờ duyệt (tuần 3–6)"
dependencies: [3]
---

# Phase 4: ChatGPT Plugin

## Goal

Đưa plugin HVN lên ChatGPT trên nền MCP server của Phase 3 (endpoint `/mcp/chatgpt`), chỉ trả văn bản, không đăng nhập. Nộp duyệt đúng hạn để có kết quả trong cửa sổ 8 tuần.

## Sự thật đã kiểm chứng

Nguồn: `plans/reports/researcher-261021-1130-chatgpt-apps-mcp-whmcs.md` §1–4.

- OpenAI đã đổi tên "Apps" thành **Plugins**. Tài liệu nằm ở `developers.openai.com/plugins/*`.
- Plugin được phép không đăng nhập, không có giao diện UI, và phải có annotation tường minh cho mọi công cụ.
- Bắt buộc: tổ chức đã xác minh trên OpenAI Platform, xác minh domain qua `/.well-known/openai-apps-challenge`, chính sách quyền riêng tư, đầu mối hỗ trợ, 5 ca thử thành công và 3 ca thử thất bại, video demo, icon, các URL website, hỗ trợ, quyền riêng tư và điều khoản.
- **Ràng buộc chính sách cần tuân thủ:**
  - Không phục vụ quảng cáo; không tồn tại chủ yếu như một kênh quảng cáo.
  - Không hiển thị gói đăng ký, không khởi tạo đăng ký mới, không quảng bá nâng cấp.
  - Chỉ link tới **trang thông tin**, không link tới trang thanh toán.
  - Mô tả plugin không nhắc giá, gói đăng ký, dùng thử hay khuyến mãi.
  - Có ghi nhận truy cập hoặc gắn UTM thì phải công bố trong chính sách quyền riêng tư.
- **Custom GPT không phải phương án:** sẽ ngừng chạy ngày 2026-12-11. Nguồn là đoạn trích Help Center, chưa đọc được trang gốc.

Thiết kế ở Phase 3 (mức `relevant`, không chèn khối gợi ý vào `fetch`, catalog chỉ có trang thông tin) đã được dựng để đáp ứng các ràng buộc trên.

## Files to Create / Modify

- Create (website hvn.vn, Marketing đăng sau khi duyệt):
  - `hvn.vn/chinh-sach-quyen-rieng-tu-ai-plugin`: dữ liệu thu thập (không thu thập dữ liệu cá nhân; log kỹ thuật không chứa nội dung truy vấn), mục đích, bên nhận (không có), thời hạn lưu log (đề xuất 30 ngày), việc gắn UTM trên link sang website HVN, quyền của người dùng, liên hệ `kythuat@hvn.vn`.
  - `hvn.vn/dieu-khoan-ai-plugin`: nội dung chỉ mang tính tham khảo, không thay thế tư vấn pháp lý, giới hạn trách nhiệm.
  - Cả hai trang dùng chung cho marketplace Claude (`privacyPolicyUrl` trong `plugin.json`, nếu sau này nộp directory).
- Create (repo công khai): `submission/chatgpt/` gồm:
  - `test-cases.md`: 5 ca thành công, ví dụ "Trỏ MX Google Workspace cho tên miền .vn", "Chọn gói Microsoft 365 cho 20 nhân viên". 3 ca thất bại, ví dụ câu hỏi ngoài phạm vi, yêu cầu báo giá (trả lời rằng plugin không cung cấp giá và dẫn tới trang thông tin), yêu cầu thao tác vào tài khoản.
  - `listing.md`: mô tả, release notes.
  - `icon.png`: logo HVN theo Brand Guideline, nền #EA4445 hoặc trắng.
- Create: video demo 2–3 phút (Marketing #Minion), lưu ngoài repo.

## Tasks & Steps

1. Tuần 1, chạy song song: xác định tài khoản tổ chức trên OpenAI Platform đứng tên **CÔNG TY CỔ PHẦN TẬP ĐOÀN HVN**, rồi bắt đầu xác minh tổ chức. Đây là đường găng vì thời gian xác minh không do HVN kiểm soát.
2. Kiểm tra project OpenAI không bật EU data residency, vì tùy chọn này chặn plugin dùng MCP server.
3. Thử nghiệm: ChatGPT > Plugins > "+" > Add custom MCP server > `https://mcp.hvn.vn/mcp/chatgpt` > No authentication > Create as a plugin. Chạy đủ 8 ca trong `test-cases.md` và ghi kết quả. Xác nhận gói ChatGPT nào dùng được (Business chắc chắn được; Plus và Pro chưa rõ, UNVERIFIED).
4. Gửi CMO duyệt trang chính sách quyền riêng tư và trang điều khoản, rồi đăng.
5. Đặt `OPENAI_DOMAIN_TOKEN` trên server và xác minh domain.
6. Soạn `listing.md` và đối chiếu từng dòng với quy định: không có giá, không khuyến mãi, không có từ ngữ quảng cáo. Đặt `commerce: false` và xác nhận Việt Nam có trong danh sách quốc gia phát hành (UNVERIFIED).
7. Nộp hồ sơ. Phản hồi reviewer trong vòng 2 ngày làm việc. Nếu bị từ chối vì `recommend_solutions`, chuyển sang phương án dự phòng: nộp chỉ với `search` và `fetch`, và tắt `recommend_solutions` cho kênh `chatgpt` bằng `config/upsell.yaml` (thêm `tools_disabled`), không cần sửa code công cụ.

## Verification

- Bản ghi chạy 8 ca thử trên ChatGPT, kèm ảnh chụp, đính trong issue theo dõi.
- `curl -s https://mcp.hvn.vn/.well-known/openai-apps-challenge` trả đúng token đã đăng ký (so sánh trên dashboard, không in token ra log công khai).
- Trạng thái nộp trên dashboard là "Submitted" trước hết tuần 4.

## Success Criteria

- [ ] Plugin chạy được trên ChatGPT qua custom MCP server, đạt 8/8 ca thử.
- [ ] Hồ sơ đã nộp trước hết tuần 4; được duyệt, hoặc đã có phương án dự phòng sẵn sàng.
- [ ] Trang chính sách quyền riêng tư và trang điều khoản đã đăng, mô tả đúng hành vi thật của server.

## Risks

- **Thời gian duyệt không được công bố.** Phase 7 không chờ được duyệt mới ra mắt; link ChatGPT được bổ sung sau.
- **Bị từ chối vì nội dung mang tính quảng cáo.** Đã có phương án dự phòng ở bước 7. Mức "chủ động mạnh" chỉ áp dụng trên marketplace Claude do HVN tự host.
