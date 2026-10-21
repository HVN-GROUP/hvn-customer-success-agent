---
phase: 5
title: "Sản xuất nội dung cho 7 plugin"
status: todo
priority: P1
effort: "5w (song song tuần 2–6)"
dependencies: [2]
---

# Phase 5: Sản xuất nội dung cho 7 plugin

## Goal

Mỗi plugin chuyên sâu đạt tối thiểu **N nội dung đã duyệt**, chia theo 4 loại. `hvn-advisor` phủ đủ 14 thương hiệu và 22 giải pháp. Tất cả đi qua pipeline của Phase 2.

## Định mức (đề xuất N = 35, chờ CEO chốt ở Validation)

| Plugin | Playbook | Cấu hình & sự cố | Chọn gói | Tuân thủ | Tổng |
|---|---|---|---|---|---|
| hvn-claude | 10 | 15 | 5 | 5 | 35 |
| hvn-chatgpt | 10 | 15 | 5 | 5 | 35 |
| hvn-gemini | 10 | 15 | 5 | 5 | 35 |
| hvn-google-workspace | 10 | 15 | 5 | 5 | 35 |
| hvn-microsoft-365 | 10 | 15 | 5 | 5 | 35 |
| hvn-security | 10 | 15 | 5 | 5 | 35 |
| hvn-advisor | Mỗi giải pháp S01–S22 có 1 mô tả nhu cầu → gói; 1 skill danh mục thương hiệu; 1 skill tư vấn chọn gói | | | | ~6 skill, tham chiếu toàn bộ catalog |

Tổng cộng khoảng **216 nội dung** cần duyệt trong 5 tuần, tức khoảng 43 nội dung mỗi tuần. Với 2 người duyệt, mỗi người khoảng 4–5 nội dung mỗi ngày làm việc.

## Gom nội dung thành skill

Một SKILL.md có thể chứa nhiều nội dung cùng chủ đề, đặt trong `references/*.md` để giữ SKILL.md dưới 500 dòng. Ví dụ `gws-email-su-co` gồm 15 sự cố email, mỗi sự cố là một file reference. Đơn vị đếm N là **nội dung đã duyệt**, không phải số thư mục skill. Cách gom này giữ danh sách skill gọn, để Claude chọn skill chính xác hơn.

## Chủ đề ưu tiên (gợi ý, biên tập viên chốt danh sách ở tuần 2)

- **hvn-claude:** Projects theo phòng ban (Sales, Kế toán, HR, Marketing), viết prompt tiếng Việt, Claude Team/Enterprise quản trị thành viên, SSO, chính sách lưu trữ dữ liệu, Claude Code cho đội dev, Connectors với Google Workspace/M365.
- **hvn-chatgpt:** GPTs nội bộ, ChatGPT Team/Enterprise admin, quản lý dữ liệu và quyền riêng tư, Projects, Deep Research cho nghiên cứu thị trường.
- **hvn-gemini:** Gemini trong Gmail/Docs/Sheets/Meet (khách Workspace), Gems, NotebookLM, bật/tắt Gemini theo OU.
- **hvn-google-workspace:** trỏ MX/SPF/DKIM/DMARC, chuyển dữ liệu email, phân quyền Drive dùng chung, bảo mật tài khoản 2 lớp, khôi phục email/file bị xóa.
- **hvn-microsoft-365:** cấu hình domain, Exchange Online, Teams, SharePoint/OneDrive, Intune cơ bản, chuyển đổi từ Google.
- **hvn-security:** cài SSL lên cPanel/Nginx/IIS, gia hạn SSL, Acronis backup và khôi phục, Kaspersky endpoint, checklist Nghị định 13 về dữ liệu cá nhân, Luật An ninh mạng, ứng phó ransomware.

## Tasks & Steps

1. Tuần 2: chốt danh sách chủ đề cho từng plugin trong một issue theo dõi ở repo công khai, mỗi chủ đề một checkbox. Gán người duyệt.
2. Tuần 2–6: sinh nháp qua `hvn-content-draft` theo lô 10 nội dung. Mỗi lô là một PR theo plugin để người duyệt xem gọn.
3. Nội dung loại tuân thủ: HVN Security (CEO Trần Văn Hoà hoặc người được ủy quyền) duyệt bổ sung trước khi merge.
4. Mỗi tuần, người phụ trách báo cáo tiến độ bằng `npm run stats`, script đếm nội dung theo plugin và theo loại từ `content-index.json`. Thêm script này vào `scripts/stats.ts` của repo công khai.
5. Tuần 6: rà soát chéo, mỗi người duyệt đọc ngẫu nhiên 10% nội dung của người kia.

## Verification

- `npm run stats` cho thấy mỗi plugin chuyên sâu có ít nhất N nội dung, đủ tối thiểu theo từng loại trong bảng định mức.
- `npm run check -- --strict` trả về exit 0, nghĩa là không có PII, link đều có UTM, và không có nội dung quá 90 ngày.
- 100% nội dung loại tuân thủ có approval của người duyệt HVN Security trên PR.

## Success Criteria

- [ ] 6 plugin chuyên sâu đạt định mức N đã chốt.
- [ ] `hvn-advisor` phủ đủ 14 thương hiệu và 22 giải pháp trong catalog.
- [ ] Rà soát chéo 10% không phát hiện lỗi nghiêm trọng, hoặc mọi lỗi phát hiện đã được sửa.

## Risks

- **Năng lực duyệt là nút thắt.** Nếu đến cuối tuần 4 đạt dưới 50% định mức, CEO quyết định một trong hai hướng: thêm người duyệt, hoặc ra mắt với N thấp hơn rồi bổ sung sau. Không tự cắt định mức khi chưa có quyết định.
- **Tính năng của hãng thay đổi nhanh,** đặc biệt Claude, ChatGPT và Gemini. Mỗi nội dung có `hvn-reviewed`, và workflow freshness của Phase 2 nhắc duyệt lại sau 90 ngày.
