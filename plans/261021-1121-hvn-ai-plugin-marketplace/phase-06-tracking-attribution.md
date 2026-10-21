---
phase: 6
title: "Đo lường & quy doanh thu về plugin"
status: todo
priority: P1
effort: "4d"
dependencies: [1]
---

# Phase 6: Đo lường & quy doanh thu về plugin

## Goal

Mỗi lượt click từ plugin được GA4 ghi nhận theo nền tảng, plugin và skill. Mỗi dịch vụ mua sau lượt click đó được WHMCS lưu nguồn UTM ở phía server, kể cả đơn thanh toán bằng chuyển khoản.

## Sự thật đã kiểm chứng

Nguồn: `plans/reports/researcher-261021-1130-chatgpt-apps-mcp-whmcs.md` §7.

- WHMCS không tự lưu UTM.
- Addon GA4 của WHMCS chỉ ghi chuyển đổi khi cổng thanh toán online xác nhận, và **không ghi được đơn chuyển khoản**. Vì vậy dữ liệu UTM lưu phía server mới là nguồn sự thật.
- Link UTM trỏ tới website thương hiệu, không trỏ thẳng tới WHMCS. Mỗi website thương hiệu phải chuyển tiếp `utm_*` sang link giỏ hàng WHMCS, nếu không WHMCS không bao giờ nhìn thấy UTM.
- WHMCS không có API `UpdateOrder`. UTM được lưu vào custom field của khách hàng (`UpdateClient`) và custom field của dịch vụ (`UpdateClientProduct`).

## Luồng dữ liệu

```
Plugin → link UTM → website thương hiệu (GA4 ghi phiên)
       → script chuyển tiếp utm_* sang link giỏ hàng WHMCS
       → WHMCS hook ClientAreaPage(Cart): lưu first-touch UTM vào session
       → ClientAdd: ghi UTM vào custom field khách hàng (chỉ admin xem)
       → AfterShoppingCartCheckout: ghi UTM vào custom field từng dịch vụ (chỉ admin xem)
       → Báo cáo: dịch vụ có utm_medium=ai-plugin, theo utm_source / utm_campaign
```

## Files to Create / Modify

- Create: `snippets/utm-forward.js` (repo công khai, khoảng 30 dòng). Đọc `utm_*` từ URL hiện tại, lưu `sessionStorage` theo nguyên tắc first-touch, và gắn vào mọi link có host là cổng WHMCS của HVN [UNVERIFIED: host giỏ hàng thực tế, ví dụ `id.hvn.vn`]. Script không thu thập dữ liệu nào khác.
- Modify: website của các thương hiệu xuất hiện trong catalog đợt 1 (gcs.vn, mso.vn, lic.vn, cer.vn, hvn.vn, cùng các website khác được catalog trỏ tới). Gắn `utm-forward.js` và xác nhận đã có GA4. Phòng Marketing hoặc HVN Digital thực hiện.
- Create (mã nguồn WHMCS, theo chuẩn `mj-whmcs-dev-standard`): hook `includes/hooks/hvn_utm_attribution.php` gồm 3 hàm:
  - `ClientAreaPage`: lưu first-touch `utm_source`, `utm_medium`, `utm_campaign`, `utm_content` vào session. Lọc theo danh sách giá trị cho phép và giới hạn độ dài 64 ký tự để chặn dữ liệu rác.
  - `ClientAdd`: dùng `localAPI('UpdateClient', …)` ghi vào custom field khách hàng chỉ admin xem.
  - `AfterShoppingCartCheckout`: với mỗi `ServiceID`, dùng `localAPI('UpdateClientProduct', …)` ghi vào custom field dịch vụ chỉ admin xem.
- Modify (WHMCS admin): tạo custom field khách hàng `utm_first_touch` và custom field sản phẩm `utm_order` (Admin Only) cho các nhóm sản phẩm trong catalog.
- Create: báo cáo GA4 Exploration và báo cáo WHMCS lọc dịch vụ theo `utm_order` chứa `ai-plugin`. Báo cáo doanh thu thuộc cấp nội bộ, đánh dấu "BẢO MẬT".

## Tasks & Steps

1. Liệt kê toàn bộ domain đích trong `catalog/products.yaml` và kiểm tra từng website đã có GA4 chưa (Marketing xác nhận).
2. Viết `utm-forward.js` cùng test chạy trên trình duyệt (vitest + jsdom): link giỏ hàng nhận đủ `utm_*`; link ngoài không bị đổi; lần truy cập sau không ghi đè first-touch.
3. Viết hook WHMCS và triển khai trên **WHMCS dev** (`ID_DEV_HVN`) trước. Chạy `mj-security-gate`, rồi `mj-staging-uat`. Thử cả hai luồng: khách mới đăng ký và khách cũ đăng nhập rồi mua. Xác nhận `ClientAdd` vẫn đọc được session khi khách đăng ký trong lúc checkout (chưa kiểm chứng, theo báo cáo).
4. Triển khai lên production WHMCS theo quy trình phát hành hiện có.
5. Dựng báo cáo GA4 (theo `utm_medium=ai-plugin`, chia theo source, campaign, content) và báo cáo WHMCS.
6. Chạy thử từ đầu tới cuối: click link trong plugin hvn-security trên claude.ai → cer.vn → giỏ hàng → đặt đơn thử bằng chuyển khoản trên WHMCS dev → custom field dịch vụ có `source=claude, medium=ai-plugin, campaign=hvn-security`.

## Verification

- `npm test` của repo công khai đạt các ca thử cho `utm-forward.js`.
- Trên WHMCS dev: sau đơn thử, `services_get` của dịch vụ vừa tạo có custom field `utm_order` đúng giá trị. Thử cả đơn chuyển khoản lẫn đơn qua cổng online.
- GA4 Realtime ghi nhận phiên `utm_medium=ai-plugin` từ một lượt click thật trong plugin.
- `mj-security-gate` đạt với hook (không có lỗ hổng chèn dữ liệu, input đã được lọc).

## Success Criteria

- [ ] 100% domain đích trong catalog có GA4 và đã gắn `utm-forward.js`.
- [ ] Đơn chuyển khoản vẫn quy được về plugin qua custom field.
- [ ] Có báo cáo doanh thu theo plugin, chỉ nội bộ được xem.

## Risks

- **Website thương hiệu dùng nhiều CMS khác nhau.** Gắn script ở từng site tốn công; ưu tiên các site được catalog đợt 1 trỏ tới.
- **Hook WHMCS chạy trong hệ thống vận hành chính.** Bắt buộc đi qua staging và security gate. Hook chỉ ghi custom field, không chặn checkout, và mọi lỗi được nuốt rồi ghi log để không làm hỏng việc đặt hàng.
