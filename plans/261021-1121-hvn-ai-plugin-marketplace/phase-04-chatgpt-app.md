---
phase: 4
title: "ChatGPT Plugin (trước đây gọi là App)"
status: todo
priority: P1
effort: "4d công + thời gian chờ duyệt (tuần 1–8)"
dependencies: [3]
---

# Phase 4: ChatGPT Plugin

<!-- Updated: Red Team 2026-10-21 - mốc quyết định tuần 2; ca thử chỉ từ nội dung chắc chắn có; trang pháp lý từ tuần 1, HVN Security cùng duyệt; chính sách quyền riêng tư đúng thực tế; workspace ChatGPT Business -->

## Goal

Đưa plugin HVN lên ChatGPT trên `/mcp/chatgpt`, chỉ trả văn bản, không đăng nhập. Nộp hồ sơ trước cuối tuần 4, kèm mốc quyết định đi tiếp hay dừng rõ ràng.

## Sự thật đã kiểm chứng

Nguồn: `plans/reports/researcher-261021-1130-chatgpt-apps-mcp-whmcs.md` §1–4.

- OpenAI đã đổi tên "Apps" thành **Plugins**. Plugin được phép không đăng nhập, không có giao diện UI, và phải có annotation tường minh cho mọi công cụ.
- Hồ sơ bắt buộc gồm:
  - tổ chức đã xác minh;
  - xác minh domain;
  - chính sách quyền riêng tư nêu loại dữ liệu, mục đích, bên nhận, thời hạn lưu và quyền kiểm soát;
  - đầu mối hỗ trợ;
  - 5 ca thử thành công và 3 ca thử thất bại;
  - video demo, icon;
  - các URL website, hỗ trợ, quyền riêng tư, điều khoản.
- **Ràng buộc chính sách:**
  - Không quảng cáo; không tồn tại chủ yếu như một kênh quảng cáo.
  - Không hiển thị gói đăng ký và không quảng bá nâng cấp.
  - Chỉ link trang thông tin, không link trang thanh toán.
  - Mô tả plugin không nhắc giá hay khuyến mãi.
  - Việc theo dõi (UTM, log) phải được công bố.
- Custom GPT sẽ ngừng chạy ngày 2026-12-11 (đoạn trích Help Center), nên không phải phương án.

Thiết kế đã áp dụng để đáp ứng các ràng buộc trên:

- Mức `relevant`.
- `fetch` không có khối gợi ý và không có câu chỉ dẫn.
- Catalog chỉ chứa trang thông tin.
- `exclude_types` và `tools_disabled` cấu hình được theo kênh.

## Files to Create / Modify

- Create (website hvn.vn, đăng sau khi **CMO và HVN Security cùng duyệt**):
  - `hvn.vn/chinh-sach-quyen-rieng-tu-ai-plugin`. Nội dung phải mô tả **đúng thực tế**:
    - server không lưu nội dung truy vấn và không lưu IP, chỉ log kỹ thuật tối đa 30 ngày;
    - link sang website HVN có UTM;
    - website HVN dùng Google Analytics 4 (Google là bên nhận dữ liệu);
    - khi khách đặt mua trên WHMCS, nguồn UTM được lưu kèm dịch vụ để HVN đo hiệu quả kênh;
    - quyền của người dùng; liên hệ `kythuat@hvn.vn`.
  - `hvn.vn/dieu-khoan-ai-plugin`: nội dung chỉ để tham khảo, không thay thế tư vấn pháp lý, giới hạn trách nhiệm.
- Create (repo staging): `submission/chatgpt/` gồm:
  - `test-cases.md`. **5 ca thành công chỉ lấy từ nội dung chắc chắn có trước tuần 3**: Phase 1 (`hvn-claude`, `hvn-advisor`) và đợt nội dung đầu tiên của Phase 5 (ưu tiên `hvn-google-workspace` và `hvn-microsoft-365`). **Không dùng ca so sánh gói.** 3 ca thất bại: câu hỏi ngoài phạm vi, yêu cầu báo giá (plugin trả lời rằng không cung cấp giá), yêu cầu thao tác vào tài khoản.
  - `listing.md`: mô tả, release notes.
  - `icon.png`: logo HVN theo Brand Guideline.
- Create: video demo 2–3 phút (Marketing #Minion), lưu ngoài repo.

## Tasks & Steps

1. **Ngày 1 tuần 1:**
   - Bắt đầu xác minh tổ chức trên OpenAI Platform dưới tên **CÔNG TY CỔ PHẦN TẬP ĐOÀN HVN**.
   - Kiểm tra project OpenAI không bật EU data residency.
   - Xác nhận HVN có workspace **ChatGPT Business** để thử nghiệm; nếu chưa có thì đăng ký.
   - Người chịu trách nhiệm: CEO chỉ định.
2. **Tuần 1:** soạn chính sách quyền riêng tư và điều khoản. Hạn cho CMO và HVN Security duyệt: **cuối tuần 2**.
3. **Tuần 3:**
   - Thử trên `staging-mcp.hvn.vn/mcp/chatgpt` qua ChatGPT > Plugins > "+" > Add custom MCP server > No authentication > Create as a plugin.
   - Chạy 8 ca thử và ghi kết quả.
   - Thử thêm trên tài khoản Plus và Pro để biết gói nào dùng được (UNVERIFIED), rồi ghi vào `docs/cai-dat.md`.
4. Đặt `OPENAI_DOMAIN_TOKEN` trên production và xác minh domain.
5. Soạn `listing.md`, đối chiếu từng dòng với quy định; đặt `commerce: false`; xác nhận Việt Nam có trong danh sách quốc gia phát hành (UNVERIFIED).
6. **Mốc quyết định cuối tuần 2** (CEO): nếu tổ chức chưa xác minh xong hoặc trang pháp lý chưa được duyệt, ChatGPT rời khỏi Done means của đợt 1 và nộp ngay khi đủ điều kiện. Phase 7 vẫn ra mắt Claude đúng hạn.
7. Nộp hồ sơ **trước cuối tuần 4** theo cấu hình kênh đã chốt ở Validation Log. Phản hồi reviewer trong 2 ngày làm việc.
8. Lần nộp đầu đã loại `chon-goi` (`exclude_types: [chon-goi]`) và giữ `recommend_solutions` (Validation Log). Nếu vẫn bị từ chối vì nội dung mang tính quảng cáo, thêm `recommend_solutions` vào `tools_disabled` rồi nộp lại, tức chỉ còn `search` và `fetch`. Đây là một thay đổi cấu hình đi qua PR, CD và deploy. Lưu ý: nộp lại sẽ bắt đầu một vòng duyệt mới, thời gian không được công bố.

## Verification

- Bản ghi 8 ca thử trên ChatGPT, kèm ảnh chụp, đính trong issue theo dõi.
- Trang chính sách quyền riêng tư khớp hành vi thực của server: đối chiếu với mục "Ranh giới bảo mật" của Phase 3 và luồng dữ liệu của Phase 6. HVN Security ký xác nhận.
- `curl -s -o /dev/null -w '%{http_code}' https://mcp.hvn.vn/.well-known/openai-apps-challenge` trả 200. Token được so trên dashboard, không in ra log.
- Trạng thái trên dashboard là "Submitted" trước cuối tuần 4, hoặc biên bản quyết định ở mốc tuần 2 ghi lý do lùi.

## Success Criteria

- [ ] Plugin chạy được trên ChatGPT qua custom MCP server, đạt 8/8 ca thử.
- [ ] Hồ sơ đã nộp trước cuối tuần 4, hoặc đã có quyết định lùi tại mốc tuần 2.
- [ ] Chính sách quyền riêng tư đã đăng và đúng với thực tế.

## Risks

- **Thời gian duyệt không được công bố.** Phase 7 không chờ được duyệt; khách ChatGPT chỉ nhận link sau khi plugin được duyệt.
- **Bị từ chối.** Đã có phương án dự phòng ở bước 8. Mức "chủ động mạnh" chỉ áp dụng trên marketplace Claude do HVN tự host.
