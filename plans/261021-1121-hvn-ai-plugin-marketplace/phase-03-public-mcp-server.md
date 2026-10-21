---
phase: 3
title: "MCP server công khai (chỉ đọc, không xác thực)"
status: todo
priority: P1
effort: "5d"
dependencies: [1]
---

# Phase 3: MCP server công khai

<!-- Updated: Red Team 2026-10-21 - đợt 1 chỉ kênh chatgpt; index theo item; UTM theo channels.yaml; giới hạn input/body/thời gian; rate limit theo kênh; log không IP; staging + CD + rollback + giám sát -->

## Goal

Dựng một MCP server chỉ đọc, không lưu trạng thái, không xác thực. Server phục vụ nội dung đã duyệt cho ChatGPT tại `https://mcp.hvn.vn/mcp/chatgpt` [UNVERIFIED: tên miền do HCS chốt]. Có môi trường staging, tự deploy khi merge, rollback bằng một lệnh, và có giám sát.

## Ranh giới bảo mật

- Server chỉ đọc `dist/content-index.json`, `catalog/products.yaml` và `config/channels.yaml` được đóng gói trong image. Không gọi WHMCS hay API nội bộ, không có secret ứng dụng.
- **Không ghi nội dung truy vấn và không ghi IP vào log.**
  - Log ứng dụng chỉ gồm thời điểm, kênh, tên công cụ, mã trạng thái, độ trễ, và với `recommend_solutions` thêm số mục trả về. Số liệu này dùng tính CTR ở Phase 6.
  - Reverse proxy tắt access log, hoặc dùng định dạng không có IP và User-Agent.
  - Log xoay vòng, giữ tối đa 30 ngày.
- **Giới hạn đầu vào và tài nguyên:**
  - `query` và `need` tối đa 256 ký tự; `id` phải khớp `^[a-z0-9.-]{1,128}$`.
  - Body request tối đa 64 KB; mỗi handler tối đa 2 giây.
  - Tin IP client chỉ từ hop của proxy.
- **Rate limit theo kênh, không theo IP.** ChatGPT gọi từ một nhóm nhỏ IP của OpenAI, nên giới hạn theo IP sẽ chặn chung mọi người dùng. Đặt trần toàn cục cho `/mcp/chatgpt`, bắt đầu 600 request/phút rồi chỉnh theo load test, cộng giới hạn số request đồng thời.

## Kiến trúc

- **Thư viện:** `@modelcontextprotocol/server` v2 (2.3.1) cùng `@modelcontextprotocol/hono`, trên Node 24 LTS. `createMcpHandler` phục vụ được cả spec 2026-07-28 lẫn client đời cũ. Nguồn: `plans/reports/researcher-261021-1130-chatgpt-apps-mcp-whmcs.md` §5.
- **Endpoint:**
  - `/mcp/:channel`, chỉ nhận kênh có trong `config/channels.yaml` **và** đánh dấu `mcp: true`. Đợt 1 chỉ có `chatgpt`; kênh khác trả 404. Đợt 2 thêm kênh mới bằng cấu hình, ví dụ `claude-connector` với `utm_source` riêng.
  - `/healthz`.
  - `/.well-known/openai-apps-challenge`, đọc token từ biến môi trường `OPENAI_DOMAIN_TOKEN`; token không commit vào repo.
- **Tìm kiếm:** `minisearch` trên **từng item**, đánh trọng số cho `title`. Có chuẩn hóa tiếng Việt (NFD rồi bỏ dấu, `đ→d`), và tách cả **cụm 2 âm tiết** để giảm khớp nhiễu kiểu "sao" khớp mọi thứ.

## Công cụ

Mọi công cụ khai báo `readOnlyHint: true, destructiveHint: false, openWorldHint: false`.

| Tool | Input | Output |
|---|---|---|
| `search` | `query` (≤256) | Tối đa 8 kết quả `{id, title, url, snippet}`, theo schema chuẩn search/fetch của OpenAI. Bỏ qua item thuộc `exclude_types` của kênh. |
| `fetch` | `id` (theo pattern) | `{id, title, text, url, metadata: {plugin, type, reviewed}}`. `text` lấy từ index: không có khối gợi ý, không có câu chỉ dẫn; tham chiếu `[[hvn:…]]` hiện thành tên sản phẩm kèm link trang thông tin có UTM của kênh (`utm_campaign=<plugin>`, `utm_content=<id>`). |
| `recommend_solutions` | `need`: **enum** lấy từ `catalog.needs` | Tối đa `max_items` mục `{name, summary, matched_needs, url}`. `url` là trang thông tin có `utm_campaign=hvn-advisor&utm_content=recommend`; mỗi mục ghi "Sản phẩm của HVN GROUP". Không đăng ký công cụ này nếu nó nằm trong `tools_disabled`. |

Mô tả công cụ viết trung tính: không bảo model ưu tiên HVN, không dùng ngôn ngữ quảng cáo, không nhắc giá hay khuyến mãi. Riêng `recommend_solutions` ghi rõ "Dùng khi người dùng hỏi nên chọn sản phẩm hoặc nhà cung cấp nào".

## Files to Create / Modify (repo staging)

- Create: `server/main.ts` — Hono app, các route, `allowedHosts` lấy từ biến môi trường `MCP_ALLOWED_HOSTS`, giới hạn body và thời gian, giới hạn đồng thời.
- Create: `server/build-mcp-server.ts` — `build(channelConfig)` đăng ký các công cụ, bỏ qua công cụ trong `tools_disabled`.
- Create: `server/content-store.ts` — nạp index, dựng minisearch, chuẩn hóa tiếng Việt, áp `exclude_types`.
- Create: `server/recommend.ts` — khớp `need` với catalog, áp `max_items`, gắn UTM bằng `shared/utm.ts` của Phase 1.
- Create: `server/*.test.ts`, `server/search-golden.yaml` — 15 truy vấn tiếng Việt kèm id kỳ vọng nằm trong top 3.
- Create: `Dockerfile` — `node:24-alpine`, user không phải root, `npm run build` ngay trong lúc build image để sinh `dist/content-index.json`. Gắn tag image bằng git SHA.
- Create: `deploy/` — cấu hình reverse proxy mà HCS đang dùng (Caddy hoặc nginx): TLS, giới hạn theo kênh, log không IP. Thêm logrotate 30 ngày và `docker compose` với `restart: unless-stopped`.
- Create: `.github/workflows/deploy.yml` — khi push `main`: build image, đẩy lên registry, deploy **staging**, smoke test (`/healthz` + `tools/list` + `fetch` một id cố định), rồi deploy **production** qua GitHub Environment `production` có người duyệt. Giữ tag trước để rollback bằng `deploy/rollback.sh <tag>`. Secret registry và SSH chỉ nằm trong environment, không bao giờ ở `ci.yml`.
- Create: `docs/van-hanh-mcp.md` — cách deploy, rollback, đọc log, giám sát.

## Tasks & Steps

1. Viết `content-store.ts` và test golden: 15 truy vấn (không dấu, có dấu, viết tắt như "gg workspace", "m365"), mỗi truy vấn có item kỳ vọng nằm trong top 3.
2. Viết `build-mcp-server.ts` với 3 công cụ. Kiểm tra tên trường `annotations` của `registerTool` trong SDK v2 bằng type của gói đã cài (hiện UNVERIFIED).
3. Viết `main.ts` gồm giới hạn, route, kiểm tra kênh. Chạy cục bộ và kiểm tra bằng `npx @modelcontextprotocol/inspector@<phiên bản ghim>`.
4. Viết test tích hợp bằng MCP client của SDK v2, gọi `/mcp/chatgpt`.
5. Dựng `staging-mcp.hvn.vn` và `mcp.hvn.vn` trên VPS của HVN Cloud (HCS cấp DNS và VPS). Cài uptime monitor bên ngoài kiểm tra `/healthz` và `tools/list` mỗi phút, cảnh báo về `kythuat@hvn.vn` và nhóm kỹ thuật.
6. Load test trên staging: 50 phiên đồng thời × 3 lời gọi, rồi chỉnh trần rate limit để không phát sinh 429 ở mức đó.
7. Viết `deploy.yml`, `rollback.sh` và `docs/van-hanh-mcp.md`. Diễn tập: deploy một tag lỗi lên staging rồi rollback.

## Verification

- `npm test` trả về exit 0, gồm các ca:
  - bỏ dấu tiếng Việt;
  - 15/15 truy vấn golden có item kỳ vọng trong top 3;
  - `recommend_solutions` trên `chatgpt` trả tối đa 1 mục với `utm_source=chatgpt&utm_campaign=hvn-advisor`;
  - `fetch` không chứa `hvn-goi-y`, "Giải pháp liên quan" hay chuỗi chỉ dẫn;
  - item thuộc `exclude_types` không xuất hiện trong `search`;
  - `tools_disabled` làm `tools/list` mất công cụ tương ứng;
  - `/mcp/grok` trả 404;
  - `query` dài 10.000 ký tự bị từ chối;
  - body 1 MB trả 413.
- MCP Inspector liệt kê đủ công cụ, mỗi công cụ có 3 annotation tường minh.
- Sau một lượt gọi thử với chuỗi định danh, `grep` chuỗi đó **và** mẫu IPv4 trên cả log ứng dụng lẫn log proxy đều không có kết quả.
- Load test 50×3 không phát sinh 429 hay 5xx; p95 dưới 500 ms.
- Diễn tập rollback thành công: smoke test đạt trên tag cũ.
- Uptime monitor gửi cảnh báo khi dừng container staging có chủ đích.

## Success Criteria

- [ ] Server công khai qua HTTPS, chỉ đọc, không có secret ứng dụng, log không có truy vấn hay IP.
- [ ] Mỗi merge vào `main` tự đi tới staging; production cần một lần duyệt; rollback bằng một lệnh.
- [ ] Tìm kiếm tiếng Việt đạt bộ golden.

## Risks

- **Không rõ ChatGPT dùng phiên bản spec MCP nào** [UNVERIFIED]. `createMcpHandler` phục vụ được cả hai; xác nhận ở Phase 4.
- **API của SDK v2 còn mới.** Nếu tên trường khác báo cáo, đọc type hoặc README của gói đã cài; không đoán.
