# Research: ChatGPT Plugin trên MCP công khai chỉ đọc, client AI khác, và UTM trong WHMCS

Ngày 2026-10-21. Nguồn lấy trực tiếp trong ngày, trừ khi ghi chú khác. File do phiên điều phối lưu lại từ báo cáo của researcher.

## Kết luận chính
1. **Kỹ thuật khả thi.** OpenAI đã đổi tên "Apps" thành **Plugins**: `developers.openai.com/apps-sdk/*` chuyển hướng sang `/plugins/*`. Plugin được phép chỉ trả văn bản, không đăng nhập, dùng Streamable HTTP công khai tại `/mcp`. Cả 3 công cụ đều chỉ đọc.
2. **Rủi ro chính là chính sách, tập trung ở `recommend_solutions`.** Nguyên văn quy định: "Selling digital products or services—including subscriptions… is not allowed, whether offered directly or indirectly", và "must not exist primarily as an advertising vehicle". Được phép link tới trang thông tin; không được link tới trang thanh toán (checkout).
3. **Custom GPT đang bị khai tử**, thay bằng plugins. Ngày ngừng là 2026-12-11; Enterprise được lùi đến 2027-02-11 (theo đoạn trích từ Help Center).
4. **MCP TypeScript SDK v2:** `@modelcontextprotocol/server` 2.3.1, cần Node >=20. Spec 2026-07-28 bỏ session; `createMcpHandler` phục vụ được cả client đời cũ lẫn đời mới.
5. **WHMCS không tự lưu UTM.** Cần 3 hook kèm custom field chỉ admin xem được. Addon GA4 không ghi nhận được đơn trả bằng chuyển khoản ngân hàng.

## 1. Yêu cầu với MCP server của ChatGPT plugin
Nguồn: https://developers.openai.com/plugins/build/mcp-server, /plugins/plugin-guidelines, /plugins/deploy/app-review, /plugins/deploy/submission
- **Kết nối:** Streamable HTTP, thường đặt tại `/mcp`, chạy HTTPS công khai và ổn định. Dùng tunnel chỉ hợp lệ khi thử nghiệm.
- **Xác thực:** cổng nộp chấp nhận "no authentication, OAuth, and mixed".
- **Annotation bắt buộc:** đặt `readOnlyHint`, `destructiveHint`, `openWorldHint` thành boolean tường minh. Với HVN là true/false/false. Lý do cho `openWorldHint: false`: "A tool confined to a bounded private account, workspace, or catalog may use false".
- **Giao diện:** không bắt buộc có widget UI. CSP chỉ áp dụng khi trả UI (`_meta.ui.csp`).
- **Xác minh domain:** đặt token tại `https://<host>/.well-known/openai-apps-challenge`.
- **Giới hạn khác:** project có EU data residency không nộp được plugin dùng MCP server; khi nộp công khai, header của server phải để trống.
- **Tùy chọn:** nếu công cụ theo schema chuẩn `search`/`fetch`, tính năng "company knowledge" của ChatGPT dùng được chúng.
- **Sau khi phát hành:** OpenAI quét server hằng ngày. Thay đổi ở công cụ cũ lên sống sau kiểm tra tự động; công cụ mới phải chờ duyệt.

## 2. Quy định khi nộp (trích nguyên văn /plugins/plugin-guidelines)
- **Tài khoản:** bắt buộc tổ chức đã xác minh; tên doanh nghiệp chưa xác minh bị từ chối.
- **Hồ sơ phải có:**
  - Chính sách quyền riêng tư nêu rõ loại dữ liệu, mục đích, bên nhận, thời hạn lưu và quyền kiểm soát.
  - Đầu mối hỗ trợ.
  - 5 ca thử thành công và 3 ca thử thất bại.
  - Video demo, release notes, icon.
  - Các URL website, hỗ trợ, quyền riêng tư và điều khoản.
- **Thời gian duyệt:** không công bố.
- **Quảng cáo:** "Plugins must not serve advertisements and must not exist primarily as an advertising vehicle."
- **Mô tả plugin:** "Do not advertise pricing, subscriptions, free trials, discounts, or promotions."
- **Thương mại:** chỉ được bán hàng hóa vật lý. "Plugins must not display subscription plans, initiate new subscriptions, or promote upgrades."
- **Link ra ngoài:**
  - Được: "Link to an informational page describing available plans or entitlement options."
  - Không được: "Link directly to a checkout or other transactional page."
- **Không chèn nội dung lạc đề.** Mô tả công cụ không được bảo model ưu tiên HVN; tên công cụ tránh ngôn ngữ quảng cáo.
- **Theo dõi:** việc ghi log truy vấn hoặc gắn UTM phải được công bố trong chính sách quyền riêng tư.
- **Nhận định của researcher (suy luận, chưa có văn bản OpenAI xác nhận):** `recommend_solutions` thuần quảng cáo có khả năng bị từ chối.
- **Phương án đề xuất:** lấy hướng dẫn làm giá trị cốt lõi; `recommend_solutions` chỉ chạy khi người dùng hỏi về sản phẩm, trả về mức độ phù hợp theo dữ kiện, link tới trang thông tin, không bao giờ link `cart.php`.

## 3. Thử trước khi nộp
Nguồn: /plugins/deploy/connect-chatgpt
- **Các bước:** ChatGPT Plugins → "+" → Add custom MCP server → nhập URL có `/mcp` → Create as a plugin. Khi dùng, gõ `@` trong chat.
- **Công cụ thử khác:** MCP Inspector `npx @modelcontextprotocol/inspector@latest`; API Playground (Tools → MCP Server).
- **Gói ChatGPT nào thêm được custom MCP:** UNVERIFIED (Business/Enterprise có đầy đủ; Pro chỉ read/fetch; Plus chưa rõ).

## 4. Custom GPT: không nên làm
Nguồn: https://learn.chatgpt.com/docs/migrate-custom-gpts
- Custom GPT ngừng chạy khi bị khai tử, và custom actions không chuyển sang plugin được.

## 5. MCP TypeScript SDK
- **Gói v2:** `@modelcontextprotocol/server` 2.3.1 (2026-10-05); adapter `@modelcontextprotocol/hono`, `/express`, `/node`.
- **Gói v1:** `@modelcontextprotocol/sdk` 1.32.1, được sửa lỗi ít nhất 6 tháng.
- **Spec 2026-07-28:** bỏ session và `initialize`, thêm `server/discover`.
- **Khung code tối thiểu:** dùng `createMcpHandler(build)` với `createMcpHonoApp({host:'0.0.0.0', allowedHosts:[...]})`, rồi `app.all('/mcp', c => handler.fetch(c.req.raw))`.
- **`allowedHosts`:** bắt buộc khi chạy production.
- **Chưa xác nhận:** key `annotations` trong `registerTool` của v2 (UNVERIFIED); ChatGPT dùng phiên bản spec nào (UNVERIFIED).
- **Node:** 22.19+ hoặc 24 LTS.

## 6. Client khác dùng được MCP công khai không xác thực
| Client | Cấu hình | Phù hợp người dùng VN | Nguồn |
|---|---|---|---|
| Claude custom connector | Customize > Connectors > Add custom connector, chọn "No sign in"; gói Free chỉ được 1 connector | Tốt | support.claude.com/en/articles/11175166 |
| Gemini app | Custom apps: chỉ cho người ≥18 tuổi ở Mỹ, chỉ tiếng Anh | **Không dùng được ở VN** | support.google.com/gemini/answer/17209137 |
| Gemini CLI | `"mcpServers":{"hvn":{"httpUrl":"https://…/mcp"}}`; đã bị Antigravity CLI thay thế với gói miễn phí từ 2026-06-18 | Hạn chế | github.com/google-gemini/gemini-cli |
| Antigravity | `~/.gemini/config/mcp_config.json`: `"serverUrl"`, không nhận `url`/`httpUrl` | Lập trình viên | antigravity.google/docs/mcp |
| Grok app | grok.com/connectors > Custom > URL; mọi người dùng; gói Business cần admin cấp | OK | docs.x.ai/grok/connectors |
| Grok API | `tools:[{type:"mcp", server_url, server_label}]` | Lập trình viên | docs.x.ai |
| Cursor | `.cursor/mcp.json`: `{"mcpServers":{"hvn":{"url":"https://…/mcp"}}}` | Lập trình viên | cursor.com/docs/context/mcp |

## 7. WHMCS: lưu UTM vào khách hàng và dịch vụ
Nguồn: developers.whmcs.com hooks-reference; API UpdateClient, UpdateClientProduct; docs.whmcs.com/9-0/addon-modules/google-analytics
- **Điều kiện tiên quyết:** website thương hiệu phải chuyển tiếp `utm_*` sang link giỏ hàng WHMCS.
- **Bước 1:** hook `ClientAreaPage` / `ClientAreaPageCart` đọc `$_GET['utm_*']` và lưu first-touch vào session hoặc cookie.
- **Bước 2:** hook `ClientAdd` gọi `UpdateClient` với `customfields` (base64 serialize), ghi vào custom field khách hàng chỉ admin xem.
- **Bước 3:** hook `AfterShoppingCartCheckout` (có `OrderID` và `ServiceIDs`, không có ClientID) gọi `UpdateClientProduct` cho từng dịch vụ, ghi vào custom field sản phẩm chỉ admin xem. Không có API `UpdateOrder`.
- **GA4:** addon chỉ ghi chuyển đổi khi cổng thanh toán online xác nhận, không ghi được đơn chuyển khoản. Vì vậy custom field phía server là nguồn sự thật.
- **Cần thử trên staging:** `ClientAdd` có chạy trước khi mất session của khách checkout không đăng ký hay không.

Status: DONE_WITH_CONCERNS
