# Quy trình soạn và duyệt nội dung plugin AI

Tài liệu dành cho biên tập viên và người duyệt nội dung của HVN AI Plugin Marketplace. Plan gốc: `plans/261021-1121-hvn-ai-plugin-marketplace/`.

## Ba repo

| Repo | Quyền | Dùng để |
|---|---|---|
| `hvngroup/hvn-customer-success-agent` | Riêng tư | Plan và skill soạn nội dung `hvn-content-draft` |
| `hvngroup/hvn-ai-marketplace-staging` | Riêng tư | Soạn nội dung, mở PR, chạy CI, duyệt. **Mọi PR mở ở đây.** |
| `hvngroup/hvn-ai-marketplace` | Công khai | Bản sao tự động của `main` staging. Không ai push tay hay mở PR vào repo này. |

## Biên tập viên

1. Clone `hvn-ai-marketplace-staging` vào một thư mục **nằm ngoài** repo `hvn-customer-success-agent`, rồi chạy `npm ci`.
2. Mở Claude Code trong repo `hvn-customer-success-agent` và gọi skill `hvn-content-draft` với đủ đầu vào: plugin, skill, loại nội dung, chủ đề, nguồn, và đường dẫn tới bản checkout staging.
3. Skill soạn item, chạy `npm run build && npm run check`, rồi mở PR. Biên tập viên đọc lại bản nháp trước khi giao cho người duyệt.
4. Mỗi PR chỉ chứa một skill, tối đa 10 item.

**Nguồn:** chỉ dùng nguồn được phép trong `.claude/skills/hvn-content-draft/references/source-policy.md`. Không dùng bất kỳ dữ liệu WHMCS nào.

## Người duyệt

| Vai trò | Người | Duyệt gì |
|---|---|---|
| Biên tập (#Valkyrie) | _COO/CMO chỉ định_ | Giọng văn, brand, độ rõ ràng |
| Kỹ thuật (#Witch) | _COO chỉ định_ | Độ đúng kỹ thuật, đối chiếu nguồn |
| HVN Security | _CEO HSC chỉ định_ | Item loại `tuan-thu` (bắt buộc qua CODEOWNERS) |

**Cách duyệt:** đi qua checklist trong PR template từng mục một. Không approve nếu còn mục chưa tích. Mỗi item mất khoảng 30 phút, kể cả thời gian đối chiếu nguồn.

## Lịch theo dõi

- **Thứ Sáu hằng tuần:** người phụ trách gửi kết quả `npm run check -- --summary` cho CEO.
- **Cuối tuần 4:** tổng số item đã merge của 6 plugin chuyên sâu phải đạt ít nhất 105. Nếu chưa đạt, CEO chọn thêm người duyệt hoặc hạ N.
- **Hằng tháng:** workflow freshness cập nhật một issue ghim duy nhất, liệt kê các item đã quá hạn duyệt lại.

## Khi phát hiện nội dung sai hoặc nhạy cảm

Làm theo `docs/runbook-go-noi-dung.md` trong repo staging. SLA là 4 giờ làm việc.
