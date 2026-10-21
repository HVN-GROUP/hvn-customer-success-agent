---
phase: 2
title: "Pipeline nội dung AI + cổng duyệt"
status: todo
priority: P1
effort: "4d"
dependencies: [1]
---

# Phase 2: Pipeline nội dung AI + cổng duyệt

## Goal

Biên tập viên chỉ cần một lệnh để AI sinh bản nháp skill từ nguồn đã được phép, ẩn danh dữ liệu khách hàng, rồi mở PR vào repo công khai. Nội dung chỉ được phát hành sau khi người duyệt merge.

## Nguyên tắc bảo mật (theo Quy định 01-01/11/2024/QyĐ)

- Pipeline và prompt nằm trong repo **nội bộ** `hvn-customer-success-agent`, không nằm ở repo công khai.
- Ticket WHMCS là dữ liệu cấp Nội bộ hoặc Bí mật. Pipeline chỉ trích **mẫu sự cố và cách khắc phục**, tuyệt đối không giữ tên khách, domain, email, số điện thoại, mã dịch vụ hay nội dung nguyên văn của khách.
- Bảo vệ hai lớp: AI ẩn danh khi soạn, sau đó `check.ts` (Phase 1) quét PII trong CI của repo công khai. PR không qua được lớp 2 thì không merge được.
- Không bao giờ đưa giá gốc vendor, chiến lược định giá hay thông tin nội bộ vào nội dung.

## Files to Create / Modify

Repo `hvn-customer-success-agent` (nội bộ):

- Create: `.claude/skills/hvn-content-draft/SKILL.md` — skill soạn nháp. Đầu vào là plugin, chủ đề, loại nội dung và danh sách nguồn. Đầu ra là thư mục skill mới trong một bản checkout của `hvn-ai-marketplace`.
- Create: `.claude/skills/hvn-content-draft/references/style-guide.md` — giọng văn HVN (chuyên nghiệp, chủ động, có số liệu), cấu trúc bắt buộc cho từng loại `playbook | cau-hinh | chon-goi | tuan-thu`, quy tắc viết "HVN GROUP" và "công ty thành viên".
- Create: `.claude/skills/hvn-content-draft/references/anonymize-rules.md` — danh sách trường cần loại bỏ, kèm ví dụ trước và sau khi ẩn danh.
- Create: `.claude/skills/hvn-content-draft/references/source-policy.md` — nguồn được phép dùng: tài liệu chính thức của hãng (kèm URL), Outline HVN, Notion BQT phần công khai được, ticket WHMCS đã ẩn danh, dxt.vn. Nguồn bị cấm: bảng giá vendor, hợp đồng, dữ liệu nhân sự.
- Create: `docs/content-pipeline.md` — hướng dẫn cho biên tập viên và người duyệt.

Repo `hvn-ai-marketplace` (công khai):

- Create: `.github/pull_request_template.md` — checklist duyệt (xem bên dưới).
- Create: `.github/CODEOWNERS` — `plugins/**` yêu cầu một trong hai người duyệt đã chỉ định.
- Create: `.github/workflows/freshness.yml` — chạy cron hàng tháng `npm run check -- --freshness-report`, tạo issue liệt kê các skill có `hvn-reviewed` quá 90 ngày.
- Modify: cài đặt nhánh `main` với branch protection: bắt buộc PR, bắt buộc CI xanh, 1 approval từ CODEOWNERS, không cho push trực tiếp.

## Checklist duyệt trong PR

- [ ] Nội dung đúng về kỹ thuật: người duyệt đã đối chiếu các link nguồn chính thức trong phần "Nguồn" cuối SKILL.md.
- [ ] Không còn tên khách, domain, email, số điện thoại hay mã dịch vụ.
- [ ] Không có giá, không có giá gốc vendor, không có thông tin nội bộ.
- [ ] Giọng văn và thuật ngữ theo Brand Guideline HVN.
- [ ] `metadata.hvn-needs` hợp lý, để khối gợi ý chọn đúng sản phẩm.
- [ ] Nội dung loại tuân thủ (Nghị định 13, Luật An ninh mạng): HVN Security đã xác nhận.

## Tasks & Steps

1. Viết `hvn-content-draft/SKILL.md` theo luồng sau:
   - Nhận yêu cầu, rồi đọc nguồn qua các connector sẵn có (Outline HVN, WHMCS tickets chỉ đọc, web chính thức của hãng).
   - Soạn SKILL.md theo template của loại nội dung, ghi `metadata.hvn-reviewed` bằng tháng hiện tại, thêm phần "Nguồn" có URL.
   - Tự kiểm tra theo `anonymize-rules.md`.
   - Chạy `npm run build && npm run check` trong bản checkout công khai.
   - Tạo nhánh `content/<plugin>-<skill>` và mở PR.
2. Viết 4 template trong `style-guide.md`:
   - Playbook: bối cảnh, các bước, prompt mẫu, kết quả mong đợi.
   - Cấu hình: điều kiện, các bước, kiểm tra kết quả, lỗi thường gặp.
   - Chọn gói: tiêu chí và bảng so sánh tính năng, không có giá.
   - Tuân thủ: điều luật, nghĩa việc phải làm, checklist, lưu ý không thay thế tư vấn pháp lý.
3. Viết quy trình lấy mẫu sự cố từ ticket: lọc ticket đã đóng theo phòng ban hoặc từ khóa sản phẩm, gom nhóm theo vấn đề, chỉ giữ "triệu chứng, nguyên nhân, cách sửa".
4. Viết PR template, CODEOWNERS và workflow freshness. Bật branch protection.
5. Chạy thử: sinh 2 skill nháp cho `hvn-google-workspace`, trong đó 1 skill lấy từ ticket, rồi đi hết vòng PR, CI, duyệt và merge.

## Verification

- Đưa vào một bản nháp có cố tình chứa email khách: CI của PR **đỏ** ở bước quét PII.
- 2 skill thử nghiệm được merge qua PR có approval của CODEOWNERS. Không có commit nào vào `main` ngoài PR (đối chiếu `git log --first-parent main`).
- `freshness.yml` chạy thủ công (`workflow_dispatch`) và tạo được issue khi có skill nhãn `hvn-reviewed` cũ.

## Success Criteria

- [ ] Biên tập viên tạo được PR nội dung bằng một lệnh trong Claude Code.
- [ ] Không có đường nào để phát hành nội dung mà không qua người duyệt.
- [ ] Hai lớp chặn PII hoạt động, mỗi lớp có ca thử âm tính.

## Risks

- **Ẩn danh bằng AI có thể sót.** Lớp 2 bằng regex chỉ bắt được email, số điện thoại và token, không bắt được tên công ty viết tự do. Biện pháp: người duyệt bắt buộc tích mục ẩn danh, và với nội dung lấy từ ticket chỉ dùng cấu trúc "triệu chứng, nguyên nhân, cách sửa", không trích dẫn nguyên văn.
- **Người duyệt quá tải.** Xem Phase 5 về năng lực duyệt.
