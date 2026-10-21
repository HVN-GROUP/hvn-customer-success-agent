---
phase: 6
title: "Đo lường & quy doanh thu về plugin"
status: todo
priority: P1
effort: "5d (spike tuần 1–2, triển khai tuần 5–6)"
dependencies: [1]
---

# Phase 6: Đo lường & quy doanh thu về plugin

<!-- Updated: Red Team 2026-10-21 - spike giỏ hàng WHMCS sớm; 2 hook chỉ ở trang giỏ hàng, kill switch, Throwable, kiểm tra input bằng pattern; cookie 30 ngày; định dạng custom field cố định; nguồn dữ liệu cho từng chỉ số -->

## Goal

Mỗi lượt click từ plugin được GA4 ghi nhận theo nền tảng, plugin và nội dung. Mỗi dịch vụ mua sau lượt click đó lưu nguồn UTM ở phía server, kể cả đơn chuyển khoản. Mọi chỉ số 90 ngày đều có nguồn dữ liệu và công thức rõ ràng.

## Sự thật đã kiểm chứng

Nguồn: `plans/reports/researcher-261021-1130-chatgpt-apps-mcp-whmcs.md` §7.

- WHMCS không tự lưu UTM.
- Addon GA4 của WHMCS không ghi được đơn chuyển khoản.
- Không có API `UpdateOrder`; UTM được ghi vào dịch vụ bằng `UpdateClientProduct`.
- Link UTM trỏ tới website thương hiệu, nên website phải tự chuyển tiếp UTM sang giỏ hàng.

## Spike tuần 1–2 trên WHMCS dev (`ID_DEV_HVN`) — chặn thiết kế

Câu hỏi cần trả lời: `cart.php?a=add&pid=X&utm_…` có redirect trước khi `ClientAreaPageCart` chạy hay không, tức query string có bị mất không. Thử bằng một URL thật qua luồng giỏ hàng.

- **Phương án A** (nếu hook đọc được `$_GET` ở trang giỏ hàng): `ClientAreaPageCart` lưu first-touch vào session.
- **Phương án B** (nếu bị mất):
  - Thêm endpoint nhỏ `hvn_utm.php` trên host WHMCS: nhận `utm_*` và `next`, ghi cookie first-party 30 ngày trên domain WHMCS, rồi redirect.
  - `next` **chỉ** được là đường dẫn tương đối bắt đầu bằng `/cart.php` hoặc `/store/`, để chặn open redirect.
  - `utm-forward.js` đưa link giỏ hàng đi qua endpoint này.

Kết quả spike ghi vào `plans/reports/spike-261021-platform-checks.md` (cùng file với spike của Phase 1).

## Luồng dữ liệu

```
Plugin → link UTM → website thương hiệu (GA4 ghi phiên)
       → utm-forward.js: cookie first-touch 30 ngày trên domain brand, gắn utm_* vào link giỏ hàng
       → WHMCS: ClientAreaPageCart (A) hoặc hvn_utm.php (B) lưu first-touch
       → AfterShoppingCartCheckout: ghi custom field `utm_order` của từng ServiceID (chỉ admin xem)
       → Báo cáo: dịch vụ có utm_medium=ai-plugin (khớp chính xác), chia theo source, campaign
```

**Định dạng `utm_order` cố định:** `utm_source=…&utm_medium=…&utm_campaign=…&utm_content=…`.

- Mọi giá trị phải khớp `^[a-z0-9.-]{1,64}$`; `source` và `medium` phải thuộc enum trong `config/channels.yaml`, kể cả `email` và `whmcs`.
- Không dùng danh sách giá trị cho phép cho `campaign` và `content`, chỉ kiểm tra pattern, vì danh sách này thay đổi theo mỗi PR nội dung.

## Files to Create / Modify

- Create (repo staging): `snippets/utm-forward.js`, khoảng 40 dòng.
  - Đọc `utm_*` và kiểm tra theo pattern.
  - Lưu first-touch vào cookie 30 ngày (`SameSite=Lax`).
  - Dùng `URL` và `URLSearchParams.set()` để gắn UTM vào link giỏ hàng WHMCS, theo phương án A hoặc B; không bao giờ ghi đè tham số không phải `utm_*`.
  - Dùng MutationObserver để xử lý cả link được chèn bằng JavaScript.
  - Mỗi website thương hiệu **tự host một bản ghim theo phiên bản**, không hot-link từ GitHub.
- Create (mã nguồn WHMCS, theo chuẩn `mj-whmcs-dev-standard`): `includes/hooks/hvn_utm_attribution.php` gồm **2 hook**: `ClientAreaPageCart` (chỉ phương án A) và `AfterShoppingCartCheckout`. Thêm `hvn_utm.php` nếu dùng phương án B. Yêu cầu:
  - Dòng đầu kiểm tra hằng `HVN_UTM_ATTRIBUTION_ENABLED` (kill switch).
  - Toàn bộ thân hàm nằm trong `try { } catch (\Throwable $e) { logActivity('hvn_utm: ' . get_class($e)); }`, chỉ ghi loại lỗi, không ghi giá trị input.
  - Không đổi luồng checkout, không trả gì về cho template.
- Modify (WHMCS admin): tạo custom field sản phẩm `utm_order` (Admin Only) cho **mọi sản phẩm** thuộc các nhóm trong catalog đợt 1. Không tạo field cấp khách hàng.
- Create: tài liệu rollback, gồm xóa file hook, tắt hằng, xóa cache template.
- Create: báo cáo GA4 Exploration (`utm_medium=ai-plugin`, chia theo source, campaign, content) và báo cáo WHMCS (lọc `utm_order` chứa đúng chuỗi `utm_medium=ai-plugin&`). Báo cáo doanh thu đánh dấu "BẢO MẬT", lưu trên Drive hoặc Outline có phân quyền, **không lưu trong git**.

## Nguồn dữ liệu cho chỉ số 90 ngày

| Chỉ số | Nguồn | Công thức |
|---|---|---|
| Lượt cài (đại diện) | GA4: phiên trên `hvn.vn/ai-plugin` có click nút "Sao chép lệnh cài"; GitHub Traffic (clone, view) của repo công khai, chụp hằng tuần bằng job trong staging, lưu CSV | Số người dùng duy nhất click nút cài; tổng clone duy nhất theo tuần |
| CTR ChatGPT | Bộ đếm `recommend_solutions` trong log server (Phase 3) và click GA4 có `utm_source=chatgpt&utm_campaign=hvn-advisor` | click ÷ số lần gọi có ít nhất 1 mục trả về |
| Click Claude | GA4 `utm_source=claude` | Tổng click theo plugin và nội dung. **Không tính CTR**, vì Claude không gửi số lần hiển thị về HVN. |
| Doanh thu | WHMCS `utm_order` | Số dịch vụ và doanh thu có `utm_medium=ai-plugin` |
| Ticket | WHMCS: **danh sách từ khóa và chủ đề cố định**, lưu cùng baseline (Phase 7) | Số ticket khớp danh sách, so với baseline |
| Tỷ lệ thất thoát | So phiên GA4 có `utm_medium=ai-plugin` tới trang giỏ hàng với số đơn ghi được UTM trong 1 tuần thử | Phát hiện sớm việc mất dấu |

**Giới hạn đã biết:**

- Đơn do nhân viên tạo tay trong WHMCS admin không có UTM.
- Đơn tên miền không có `ServiceID` nên không được quy doanh thu.
- UTM có thể bị giả mạo (ai cũng tự thêm được vào URL). Báo cáo ghi rõ đây là số liệu tự khai, không dùng làm căn cứ thưởng.

## Tasks & Steps

1. **Tuần 1–2:** chạy spike giỏ hàng và chốt phương án A hoặc B.
2. Liệt kê domain đích của catalog đợt 1. Marketing xác nhận GA4 trên từng website.
3. Viết `utm-forward.js` cùng test (vitest + jsdom):
   - link giỏ hàng nhận đủ `utm_*`;
   - tham số khác không đổi;
   - giá trị sai pattern bị bỏ;
   - first-touch không bị ghi đè;
   - link chèn sau vẫn được xử lý;
   - chuỗi `x%26a%3Dadd` không chèn được tham số giỏ hàng.
4. Viết hook (và `hvn_utm.php` nếu dùng B). Chạy `mj-security-gate`, gồm kiểm tra XSS ở mọi màn hình admin hiển thị custom field và open redirect của `next`. Sau đó chạy `mj-staging-uat`.
5. Trên WHMCS dev: chụp lại các mẫu email sẽ sửa ở Phase 7 để có bản khôi phục.
6. Triển khai hook lên production theo quy trình phát hành, kill switch bật sẵn. Theo dõi log lỗi 24 giờ.
7. Gắn bản ghim `utm-forward.js` lên các website thương hiệu đợt 1.
8. Chạy thử từ đầu tới cuối: click link trong `hvn-security` trên claude.ai → cer.vn → giỏ hàng → đơn chuyển khoản trên WHMCS dev → `utm_order` = `utm_source=claude&utm_medium=ai-plugin&utm_campaign=hvn-security&utm_content=…`.
9. Đo tỷ lệ thất thoát trong 1 tuần trước ra mắt, bằng traffic nội bộ và thử nghiệm.
10. Dựng job chụp GitHub Traffic hằng tuần cùng các báo cáo GA4 và WHMCS.

## Verification

- `npm test` của staging đạt các ca thử cho `utm-forward.js`.
- Trên WHMCS dev: `services_get` trả `utm_order` đúng định dạng cho cả đơn chuyển khoản lẫn đơn qua cổng online.
- Ném lỗi giả trong hook: trang giỏ hàng và checkout vẫn chạy, `logActivity` chỉ ghi tên lớp lỗi.
- Tắt kill switch thì hook không ghi gì.
- `mj-security-gate` đạt.
- GA4 Realtime ghi nhận phiên `utm_medium=ai-plugin` từ một lượt click thật.
- Tỷ lệ thất thoát được đo và ghi lại; nếu trên 30% thì xử lý trước khi ra mắt.

## Success Criteria

- [ ] Phương án A hoặc B đã được spike chứng minh trước tuần 3.
- [ ] Mọi domain đích đợt 1 có GA4 và `utm-forward.js`.
- [ ] Đơn chuyển khoản vẫn quy được về plugin; có báo cáo cho mọi chỉ số trong bảng.

## Risks

- **Hook nằm trong hệ thống vận hành chính.** Hook chỉ chạy ở trang giỏ hàng và checkout, có kill switch, bắt `Throwable`, và đi qua staging cùng security gate.
- **Website thương hiệu dùng nhiều CMS khác nhau.** Ưu tiên các domain được catalog đợt 1 trỏ tới.
