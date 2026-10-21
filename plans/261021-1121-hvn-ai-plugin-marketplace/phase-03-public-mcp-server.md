---
phase: 3
title: "MCP server công khai (chỉ đọc, không xác thực)"
status: todo
priority: P1
effort: "4d"
dependencies: [1]
---

# Phase 3: MCP server công khai

## Goal

Một MCP server chỉ đọc, không lưu trạng thái, không xác thực, chạy trên HVN Cloud tại `https://mcp.hvn.vn` [UNVERIFIED: tên miền do HCS chốt]. Server phục vụ nội dung đã duyệt và catalog cho ChatGPT ngay ở đợt 1, cho Grok, Cursor, Antigravity và Claude connector ở các đợt sau. Mỗi kênh có mức up-sell và UTM riêng.

## Ranh giới bảo mật

- Server **chỉ đọc file `content-index.json` và `catalog/products.yaml` đóng gói sẵn lúc build.** Không gọi WHMCS, không gọi API nội bộ, không có biến môi trường bí mật.
- **Không ghi nội dung truy vấn vào log.** Log chỉ ghi thời điểm, kênh, tên công cụ, mã trạng thái và độ trễ, để chính sách quyền riêng tư (Phase 4) đơn giản và đúng sự thật.
- Có rate limit theo IP ở reverse proxy.

## Kiến trúc

- **Thư viện:** `@modelcontextprotocol/server` v2 (2.3.1) cùng adapter `@modelcontextprotocol/hono`, trên Node 22.19+ hoặc 24 LTS. `createMcpHandler` vừa phục vụ spec 2026-07-28 không trạng thái, vừa phục vụ client đời cũ. Nguồn: `plans/reports/researcher-261021-1130-chatgpt-apps-mcp-whmcs.md` §5.
- **Endpoint theo kênh:** `/mcp/{channel}` với `channel ∈ {chatgpt, grok, cursor, antigravity, claude}`. Kênh quyết định `utm_source` và mức up-sell lấy từ `config/upsell.yaml`. Kênh lạ trả về 404. Đây là cách đơn giản nhất để đo theo nền tảng mà không cần nhận dạng người dùng.
- **Endpoint phụ:**
  - `/healthz`.
  - `/.well-known/openai-apps-challenge`, trả token từ biến môi trường `OPENAI_DOMAIN_TOKEN`. Token này không phải bí mật nhưng vẫn không commit vào repo.
- **Tìm kiếm:** `minisearch` trong bộ nhớ, có bước chuẩn hóa bỏ dấu tiếng Việt (NFD rồi xóa dấu, thêm `đ→d`), để "sao luu" khớp với "sao lưu".

## Công cụ

Mọi công cụ khai báo annotation `readOnlyHint: true, destructiveHint: false, openWorldHint: false`.

| Tool | Input | Output |
|---|---|---|
| `search` | `query: string` | Tối đa 8 kết quả `{id, title, url, snippet}`. Theo schema chuẩn search/fetch của OpenAI để dùng được với "company knowledge". |
| `fetch` | `id: string` | `{id, title, text, url, metadata: {plugin, type, reviewed}}`. `text` là nội dung skill đã duyệt kèm phần "Nguồn". |
| `recommend_solutions` | `need: string` | Danh sách sản phẩm hoặc giải pháp khớp `needs`, tối đa `max_items` của kênh. Mỗi mục gồm tên, mô tả dữ kiện, mức phù hợp, link **trang thông tin** gắn UTM, và nhãn "Sản phẩm của HVN GROUP". |

**Mô tả công cụ:**

- Viết trung tính.
- Không bảo model ưu tiên HVN.
- Không dùng ngôn ngữ quảng cáo.
- Không nhắc giá, khuyến mãi hay dùng thử.
- Riêng `recommend_solutions` ghi rõ "Dùng khi người dùng hỏi nên chọn sản phẩm, dịch vụ hay nhà cung cấp nào".

**Mức up-sell theo kênh:**

- `relevant` (ChatGPT): `search` và `fetch` **không** chèn khối gợi ý; chỉ `recommend_solutions` trả về sản phẩm.
- `strong`: `fetch` thêm khối "Giải pháp liên quan từ HVN GROUP" vào cuối `text`, giống plugin trên Claude.

## Files to Create / Modify (repo `hvn-ai-marketplace`)

- Create: `server/main.ts` — dựng Hono app, gắn route `/mcp/:channel`, `/healthz` và `/.well-known/openai-apps-challenge`; khai báo `allowedHosts` từ biến môi trường `MCP_ALLOWED_HOSTS`.
- Create: `server/build-mcp-server.ts` — `build(channel)` trả về `McpServer` đã đăng ký 3 công cụ, bỏ qua các công cụ có tên trong `tools_disabled` của kênh (phương án dự phòng của Phase 4).
- Create: `server/content-store.ts` — nạp `content-index.json` và catalog, dựng chỉ mục minisearch, hàm chuẩn hóa tiếng Việt.
- Create: `server/recommend.ts` — khớp nhu cầu vào `needs`, áp `max_items`, gắn UTM bằng chung một hàm với `scripts/build.ts`.
- Create: `shared/utm.ts` — hàm `withUtm(url, {source, campaign, content})` dùng chung cho build script và server, để quy ước UTM chỉ nằm ở một chỗ.
- Create: `server/*.test.ts` — vitest.
- Create: `Dockerfile` (node:24-alpine, chạy bằng user không phải root, chỉ copy `server/`, `shared/`, `content-index.json`, `catalog/`, `config/`).
- Create: `deploy/Caddyfile` hoặc cấu hình nginx tương đương có TLS và rate limit 60 request/phút/IP. Chọn công cụ HCS đang dùng.
- Modify: `scripts/check.ts` — thêm luật: URL trong catalog không được chứa `cart.php`, `/checkout`, `a=add` hay `/cart`, tức chỉ trang thông tin.
- Modify: `.github/workflows/ci.yml` — thêm job build image và chạy test tích hợp.

## Tasks & Steps

1. Rút `withUtm` ra `shared/utm.ts`, sửa `scripts/build.ts` dùng lại hàm này, và chạy lại test của Phase 1.
2. Viết `content-store.ts` cùng test chuẩn hóa tiếng Việt: truy vấn "khoi phuc email bi xoa" phải tìm thấy skill có tiêu đề "Khôi phục email bị xóa".
3. Viết `build-mcp-server.ts` với 3 công cụ. Đối chiếu tên trường `annotations` của `registerTool` trong SDK v2 bằng README hoặc type của gói đã cài (hiện UNVERIFIED trong báo cáo).
4. Viết `main.ts` và `Dockerfile`. Chạy cục bộ, rồi kiểm tra bằng `npx @modelcontextprotocol/inspector@latest` qua Streamable HTTP tại `http://localhost:3000/mcp/chatgpt`.
5. Viết test tích hợp: dựng server ở cổng tạm, dùng MCP client của SDK v2 gọi cả 3 công cụ trên `/mcp/chatgpt` và `/mcp/grok`. Kiểm tra UTM và mức up-sell khác nhau theo kênh.
6. Deploy lên VPS của HVN Cloud bằng Docker kèm reverse proxy TLS. Trỏ DNS `mcp.hvn.vn` (HCS thực hiện).
7. Thử thêm `https://mcp.hvn.vn/mcp/claude` làm custom connector trên claude.ai ("No sign in") để xác nhận server tương thích với một client thứ hai.

## Verification

- `npm test` trả về exit 0, gồm các ca:
  - Bỏ dấu tiếng Việt.
  - `recommend_solutions` trên `/mcp/chatgpt` trả tối đa 1 mục, có `utm_source=chatgpt`.
  - `fetch` trên `/mcp/chatgpt` không có khối gợi ý.
  - `fetch` trên kênh `strong` có khối gợi ý.
  - Kênh lạ trả 404.
  - Khi `tools_disabled: [recommend_solutions]`, kênh đó chỉ liệt kê `search` và `fetch`.
- `npm run check` báo lỗi khi catalog có URL chứa `cart.php` (test âm tính).
- `curl -s https://mcp.hvn.vn/healthz` trả 200; `curl -s -o /dev/null -w '%{http_code}' https://mcp.hvn.vn/mcp/unknown` trả 404.
- MCP Inspector liệt kê đủ 3 công cụ, mỗi công cụ có đủ 3 annotation tường minh.
- Log của container sau một lượt gọi thử không chứa chuỗi truy vấn (grep chuỗi thử nghiệm không ra kết quả).

## Success Criteria

- [ ] Server chạy công khai qua HTTPS, chỉ đọc, không chứa bí mật.
- [ ] UTM và mức up-sell đúng theo từng kênh, có test chứng minh.
- [ ] Một client không phải ChatGPT (Claude custom connector) gọi được server.

## Risks

- **Không rõ ChatGPT dùng phiên bản spec MCP nào** [UNVERIFIED]. `createMcpHandler` hỗ trợ cả hai đời; xác nhận lại bằng thử nghiệm ở Phase 4.
- **API của SDK v2 còn mới.** Nếu tên trường hoặc adapter khác với báo cáo, đọc README của gói đã cài và cập nhật; không đoán.
