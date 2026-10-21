---
phase: 6
title: "Đo lường bằng GA4"
status: todo
priority: P1
effort: "2d"
dependencies: [1]
---

# Phase 6: Đo lường bằng GA4

<!-- Updated: Red Team 2026-10-21 - nguồn dữ liệu cho từng chỉ số -->
<!-- Updated: Validation Session 2 - bỏ toàn bộ tích hợp WHMCS (hook, custom field, spike giỏ hàng, báo cáo doanh thu); đo lường dừng ở GA4 -->

## Goal

GA4 trên website thương hiệu ghi nhận mỗi lượt click từ plugin, phân theo nền tảng, plugin và nội dung. Mọi chỉ số 90 ngày đều có nguồn dữ liệu và công thức rõ ràng. Việc đo lường **không** kết nối với WHMCS (quyết định của CEO).

## Phạm vi đo

- **Đo được:**
  - Lượt click từ plugin tới trang thông tin.
  - Hành vi sau click trên website thương hiệu: phiên, trang xem, sự kiện chuyển đổi mà website đang có, ví dụ gửi form liên hệ hoặc bấm "Mua ngay".
- **Không đo:** doanh thu và đơn hàng trong WHMCS. Đây là giới hạn đã được chấp nhận. Muốn đo doanh thu sau này phải lập plan riêng.

## Files to Create / Modify

- Modify (website của các thương hiệu được catalog đợt 1 trỏ tới, do Marketing hoặc HVN Digital thực hiện):
  - Xác nhận GA4 đã gắn trên từng website.
  - Đánh dấu các sự kiện chuyển đổi sẵn có (form liên hệ, nút "Mua ngay", click hotline hoặc Zalo) thành key event trong GA4.
- Create: báo cáo GA4 Exploration lọc `utm_medium=ai-plugin`, chia theo `utm_source`, `utm_campaign` (plugin) và `utm_content` (nội dung). Báo cáo đặt trên GA4 hoặc Looker Studio, chia sẻ cho CEO, CMO và COO.
- Create (repo staging): `.github/workflows/github-traffic.yml` — chạy hàng tuần, lưu số clone và view của repo công khai thành CSV artifact. Workflow chỉ có quyền đọc.
- Create (trên `hvn.vn/ai-plugin`, cùng Phase 7): sự kiện GA4 `copy_install_command` cho nút "Sao chép lệnh cài".

## Nguồn dữ liệu cho chỉ số 90 ngày

| Chỉ số | Nguồn | Công thức |
|---|---|---|
| Lượt cài (đại diện) | GA4 sự kiện `copy_install_command`; GitHub Traffic CSV hằng tuần | Số người dùng duy nhất bấm nút cài; số clone duy nhất theo tuần |
| CTR ChatGPT | Bộ đếm `recommend_solutions` trong log server (Phase 3) và click GA4 có `utm_source=chatgpt&utm_campaign=hvn-advisor` | Số click ÷ số lần gọi có ít nhất 1 mục trả về |
| Click Claude | GA4 `utm_source=claude` | Tổng click theo plugin và theo nội dung. **Không tính CTR**, vì Claude không gửi số lần hiển thị về HVN. |
| Chuyển đổi trên website | GA4 key events trong các phiên `utm_medium=ai-plugin` | Số key event và tỷ lệ chuyển đổi theo plugin |
| Lượt mở lộ trình onboard | GA4 click có `utm_content` thuộc item loại `lo-trinh`; log `fetch` của item `lo-trinh` trên MCP | Số click và số lần fetch theo plugin |

**Giới hạn đã biết:**
- UTM có thể bị giả mạo, vì ai cũng tự thêm được vào URL. Báo cáo ghi rõ đây là số liệu chỉ báo, không dùng làm căn cứ thưởng.
- Click trên Claude chỉ đo được khi người dùng bấm link.

## Tasks & Steps

1. Liệt kê domain đích của catalog đợt 1 và nhờ Marketing xác nhận GA4 trên từng website.
2. Đánh dấu key event trên từng website.
3. Dựng báo cáo GA4 Exploration và workflow `github-traffic.yml`.
4. Chạy thử từ đầu tới cuối: bấm link trong `hvn-security` trên claude.ai, sang cer.vn, rồi kiểm tra GA4 Realtime thấy phiên có `utm_source=claude&utm_medium=ai-plugin&utm_campaign=hvn-security`.

## Verification

- GA4 Realtime ghi nhận phiên `utm_medium=ai-plugin` từ một lượt click thật trong plugin Claude, và một lượt từ ChatGPT ở staging.
- Báo cáo GA4 Exploration hiển thị đủ 3 chiều source, campaign và content.
- `github-traffic.yml` chạy tay (`workflow_dispatch`) và tạo được artifact CSV.
- Repo staging không có file hook WHMCS, script WHMCS hay cấu hình giỏ hàng: `grep -rn "whmcs\|cart.php" snippets server scripts` không trả kết quả ngoài luật chặn link trang thanh toán của `check.ts`.

## Success Criteria

- [ ] Mọi domain đích đợt 1 đã có GA4 và key event.
- [ ] Có báo cáo cho từng chỉ số trong bảng.
- [ ] Không có tích hợp nào với WHMCS.

## Risks

- **Website thương hiệu thiếu GA4 hoặc key event** thì click mất dấu. Ưu tiên các domain mà catalog đợt 1 trỏ tới.
- **Không đo được doanh thu.** CEO ra quyết định về kênh dựa trên click, chuyển đổi trên website và lượt cài.
