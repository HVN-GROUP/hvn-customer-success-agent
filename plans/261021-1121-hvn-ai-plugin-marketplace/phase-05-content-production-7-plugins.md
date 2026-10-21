---
phase: 5
title: "Sản xuất nội dung cho 7 plugin"
status: todo
priority: P1
effort: "4.5w (giữa tuần 2 – tuần 6)"
dependencies: [2]
---

# Phase 5: Sản xuất nội dung cho 7 plugin

<!-- Updated: Red Team 2026-10-21 - đơn vị là item; advisor sinh từ catalog; tính lại năng lực duyệt; người duyệt có tên; mốc tuần 4 bằng số tuyệt đối; eval cho 5 plugin còn lại; ưu tiên nội dung phục vụ ca thử ChatGPT -->

## Goal

Mỗi plugin chuyên sâu đạt tối thiểu **N item đã duyệt** (giá trị N ghi ở Validation Log của `plan.md`), chia theo 4 loại. `hvn-advisor` phủ đủ 14 thương hiệu và 22 giải pháp, sinh tự động từ catalog. Cả 7 plugin đạt ngưỡng eval.

## Đơn vị đếm

- **Một item** là một file `content/<plugin>/<skill>/<item>.md`, có `type`, `needs`, `reviewed` riêng. Lệnh `npm run check -- --summary` đếm item theo plugin và theo loại; đây là nguồn số liệu duy nhất.
- Một skill gom các item cùng chủ đề, ví dụ `gws-email-su-co` gồm 15 item sự cố. Item `tuan-thu` nằm trong skill có tiền tố `tuan-thu-` để CODEOWNERS bắt buộc HVN Security duyệt.
- `hvn-advisor` **không tính vào N**: danh mục giải pháp được sinh từ catalog, nội dung viết tay chỉ có 2 skill tư vấn (khoảng 6 item).

## Định mức theo loại (tỷ lệ áp vào N)

| Loại | Tỷ lệ | Với N=35 |
|---|---|---|
| Playbook | ~30% | 10 |
| Cấu hình & sự cố | ~40% | 15 |
| Chọn gói | ~15% | 5 |
| Tuân thủ | ~15% | 5 |

Ví dụ với N=35: 6 plugin × 35 = **210 item** trong khoảng 4,5 tuần, tức khoảng 47 item mỗi tuần. 6 item đã có từ Phase 1 không đổi đáng kể con số này. Năng lực cần có:

- **2 người duyệt nội dung**: 1 biên tập viên Content (#Valkyrie) lo giọng văn và brand, 1 kỹ thuật viên Service (#Witch) lo độ đúng kỹ thuật. Mỗi người khoảng 24 item mỗi tuần, tương đương khoảng 5 item mỗi ngày làm việc. Ước tính 30 phút mỗi item kể cả đối chiếu nguồn, tức khoảng **2,5 giờ mỗi ngày mỗi người**.
- **Người duyệt tuân thủ (HVN Security):** 30 item trong 4,5 tuần, khoảng 7 item mỗi tuần.
- Tên người và số giờ mỗi tuần được ghi ở Validation Log, và CEO duyệt trong tuần 1.

## Thứ tự sản xuất

1. **Nửa sau tuần 2 đến tuần 3, ưu tiên 1:** item phục vụ ca thử ChatGPT (Phase 4): `hvn-google-workspace` (trỏ MX/SPF/DKIM, khôi phục email) và `hvn-microsoft-365` (cấu hình domain, Exchange).
2. **Tuần 3–5:** các plugin còn lại, mỗi tuần đẩy đều 6 plugin để không plugin nào về 0.
3. **Tuần 6:** lấp chỗ thiếu theo `--summary`, rà soát chéo.

## Chủ đề gợi ý (biên tập viên chốt trong issue theo dõi ở tuần 2)

- **hvn-claude:** Projects theo phòng ban, viết prompt tiếng Việt, quản trị Team/Enterprise, SSO, lưu trữ dữ liệu, Claude Code cho đội dev, Connectors với Google Workspace và M365.
- **hvn-chatgpt:** GPTs nội bộ và việc chuyển sang plugins, quản trị Business/Enterprise, quyền riêng tư, Projects, Deep Research.
- **hvn-gemini:** Gemini trong Gmail, Docs, Sheets và Meet; Gems; NotebookLM; bật hoặc tắt Gemini theo OU.
- **hvn-google-workspace:** MX/SPF/DKIM/DMARC, chuyển dữ liệu email, Drive dùng chung, xác minh 2 lớp, khôi phục email và file.
- **hvn-microsoft-365:** domain, Exchange Online, Teams, SharePoint/OneDrive, Intune cơ bản, chuyển từ Google sang.
- **hvn-security:** SSL trên cPanel, Nginx và IIS; gia hạn SSL; Acronis; Kaspersky; Nghị định 13; Luật An ninh mạng; ứng phó ransomware.

## Tasks & Steps

1. Tuần 2: tạo issue theo dõi trong staging, mỗi chủ đề một checkbox, gán người duyệt.
2. Sinh nháp bằng `hvn-content-draft`, **mỗi PR một skill** (tối đa 10 item), để diff gọn và hạn chế xung đột. Bật merge queue (Phase 1).
3. Item `tuan-thu`: HVN Security approve theo CODEOWNERS.
4. Thứ Sáu hằng tuần: người phụ trách gửi kết quả `npm run check -- --summary` cho CEO.
5. Viết `evals/<plugin>.yaml` cho 5 plugin còn lại (30 câu hỏi mỗi plugin) và chạy eval với cả 7 plugin đã cài.
6. Rải đều ngày `reviewed`: `review_sla_days` là 60 cho playbook AI (`hvn-claude`, `hvn-chatgpt`, `hvn-gemini`) và 120 cho các loại khác. Nhờ vậy lượt duyệt lại không dồn vào cùng một tuần.
7. Tuần 6: rà soát chéo, mỗi người duyệt đọc ngẫu nhiên 10% item của người kia.

## Mốc quyết định cuối tuần 4 (số tuyệt đối)

- Nếu tổng item đã merge của 6 plugin **dưới 0,5 × 6 × N** (với N=35 là 105), CEO chọn một trong hai:
  - (a) Thêm người duyệt từ tuần 5.
  - (b) Ra mắt với N thấp hơn rồi bổ sung sau, ghi vào Validation Log.
- Không tự cắt định mức khi chưa có quyết định.

## Verification

- `npm run check -- --summary`: mỗi plugin chuyên sâu ≥ N item, đúng tỷ lệ tối thiểu theo loại (cho phép lệch ±1).
- `npm run check -- --freshness` trả về exit 0 tại ngày release.
- 100% item `tuan-thu` có approval của HVN Security trên PR (CODEOWNERS bắt buộc).
- Eval cả 7 plugin: chọn đúng skill ≥80%, có khối gợi ý ≥90%.
- `danh-muc-giai-phap-hvn` chứa đủ mọi id thương hiệu và giải pháp trong catalog (`check.ts` kiểm tra).

## Success Criteria

- [ ] 6 plugin chuyên sâu đạt định mức N đã chốt.
- [ ] `hvn-advisor` phủ đủ 14 thương hiệu và 22 giải pháp, sinh từ catalog.
- [ ] Eval 7 plugin đạt ngưỡng; rà soát chéo không còn lỗi chưa sửa.

## Risks

- **Năng lực duyệt là nút thắt.** Có mốc tuần 4 bằng số tuyệt đối. Không ép người duyệt chạy qua checklist cho đủ số.
- **Tính năng của hãng đổi nhanh.** Đã có SLA duyệt lại theo loại, và workflow freshness cập nhật một issue ghim duy nhất.
