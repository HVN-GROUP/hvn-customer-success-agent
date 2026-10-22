# Kiểm chứng nền tảng (Phase 1, bước 1)

Plan: `plans/261021-1121-hvn-ai-plugin-marketplace/`. Ngày chạy: 2026-10-21.

## (a) Thêm marketplace trên claude.ai theo loại tài khoản — CHƯA CHẠY

Spike này cần repo công khai `hvngroup/hvn-ai-marketplace` và 4 tài khoản thật (Free, Pro, thành viên Team, Owner Team). Chạy sau khi CEO tạo repo. Kết quả ghi vào bảng sau:

| Tài khoản | Thêm marketplace được? | Bản cài tự cập nhật khi repo đổi? | Ghi chú |
|---|---|---|---|
| Free | | | |
| Pro | | | |
| Thành viên Team | | | |
| Owner Team | | | |

## (b) `claude plugin validate` không cần đăng nhập — ĐẠT

**Cách chạy:** dùng Claude Code 2.1.306 với `HOME` trỏ tới một thư mục rỗng, tức không có phiên đăng nhập. Repo thử gồm marketplace và 2 plugin (`hvn-claude`, `hvn-advisor`), mỗi plugin 1 skill.

**Kết quả `claude plugin validate`:** exit 0 ở cả 3 đích (marketplace, `plugins/hvn-claude`, `plugins/hvn-advisor`). Các cảnh báo và lỗi đi kèm:

- **Cảnh báo:**
  - Tên `hvn-claude` "reads as one of Anthropic's own". Cảnh báo này đã lường trước.
  - Thiếu `version` (chủ ý: plan dùng commit SHA làm phiên bản).
  - Thiếu `author`.
- **Lỗi kiểu directory** (không làm validate fail, chỉ chặn khi chạy `--profile directory`): thiếu README và LICENSE ở thư mục plugin. Một README hoặc LICENSE ở gốc repo cũng được tính.
- **Cảnh báo kiểu directory:** thiếu icon.

**Kết quả `--strict` trên `hvn-claude`:** exit 1, vì strict coi cảnh báo là lỗi.

**Hệ quả cho CI:**

- **Chạy validate:** CI chạy `claude plugin validate` **không có** `--strict` và không cần secret.
- **Ghim phiên bản:** ghim `@anthropic-ai/claude-code@2.1.306`, trừ khi lúc dựng CI đã có bản mới hơn được kiểm tra.
- **`plugin.json` do build sinh ra phải có:**
  - `author: {name: "HVN GROUP", url: "https://hvn.vn"}`;
  - `license`, trỏ tới `LICENSE-CONTENT.md` hoặc một SPDX id;
  - `homepage`.
- **README:** mỗi plugin có `README.md` từ 40 từ trở lên, để giảm cảnh báo và sẵn sàng nếu sau này nộp directory.

## (c) Quyền sở hữu tổ chức GitHub `hvngroup` — CHƯA CHẠY

Việc này thuộc quản trị GitHub của HVN: xác nhận quyền sở hữu tổ chức, đăng ký các tên dễ nhầm, bật verified domain `hvn.vn` và bắt buộc 2FA.
