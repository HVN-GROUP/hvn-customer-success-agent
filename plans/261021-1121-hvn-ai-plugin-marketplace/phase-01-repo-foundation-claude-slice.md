---
phase: 1
title: "Nền móng repo + lát cắt dọc trên Claude"
status: todo
priority: P1
effort: "5d"
dependencies: []
---

# Phase 1: Nền móng repo + lát cắt dọc trên Claude

## Goal

Dựng repo marketplace công khai với catalog, cấu hình up-sell, bước build và CI, rồi chứng minh luồng chạy thông từ đầu tới cuối bằng `hvn-advisor` và `hvn-claude` (mỗi plugin 3 skill), cài được trên Claude.

## Bối cảnh

- Marketplace của Claude phải nằm trong **repo GitHub công khai**. claude.ai, Desktop và Cowork dùng "Customize > Plugins > Add > Add marketplace", Claude Code dùng `/plugin marketplace add`. Nguồn: `plans/reports/researcher-261021-1130-claude-plugin-marketplace.md` §3.
- Plugin không dùng chung file với nhau lúc chạy, vì `dependencies` và symlink chỉ hoạt động trong Claude Code. Phần dùng chung (catalog, khối gợi ý) vì vậy được **copy lúc build** vào từng skill. Nguồn: cùng báo cáo, §4.
- Repo `hvn-customer-success-agent` hiện tại giữ plan và pipeline nội bộ. Nội dung phát hành nằm ở repo công khai riêng `hvngroup/hvn-ai-marketplace` (xem Validation Log).

## Cấu trúc repo `hvn-ai-marketplace`

```
.claude-plugin/marketplace.json          # name: hvn-marketplace, owner HVN GROUP, 7 plugin, source ./plugins/<name>
plugins/<plugin>/.claude-plugin/plugin.json
plugins/<plugin>/README.md               # ≥40 từ, hướng dẫn cài
plugins/<plugin>/skills/<skill>/SKILL.md # nội dung gốc do người viết/duyệt
plugins/<plugin>/skills/<skill>/references/hvn-goi-y.md   # FILE SINH TỰ ĐỘNG, không sửa tay
catalog/products.yaml                    # 14 thương hiệu, 22 giải pháp, URL đích, nhu cầu → sản phẩm
config/upsell.yaml                       # mức up-sell theo kênh: claude=strong, chatgpt=relevant
config/utm.yaml                          # quy ước UTM
scripts/build.ts                         # sinh references/hvn-goi-y.md + content-index.json
scripts/check.ts                         # kiểm tra UTM, PII, frontmatter, độ tươi, drift
content-index.json                       # FILE SINH TỰ ĐỘNG cho MCP server (Phase 3)
package.json, tsconfig.json, LICENSE, README.md
.github/workflows/ci.yml
.github/pull_request_template.md         # checklist duyệt nội dung (Phase 2)
```

## Hợp đồng dữ liệu

**`catalog/products.yaml`** (chỉ chứa thông tin công khai, không có giá, không có giá gốc vendor):

```yaml
brands:
  - id: gcs
    name: GCS.VN
    url: https://gcs.vn
    summary: "Đối tác cấp cao Google Cloud — Google Workspace, lưu trữ, hội họp"
products:
  - id: acronis-backup
    brand: lic            # hoặc hvn-security tùy trang đích thực tế
    name: "Acronis Cyber Protect"
    url: https://lic.vn/acronis   # [UNVERIFIED] URL thật do Marketing xác nhận
    needs: [sao-luu-du-lieu, khoi-phuc-tham-hoa, chong-ransomware]
solutions:
  - id: S13
    name: "Sao lưu & Khôi phục thảm họa"
    url: https://hvn.vn/giai-phap/s13   # [UNVERIFIED]
    needs: [sao-luu-du-lieu, khoi-phuc-tham-hoa]
```

**`config/upsell.yaml`:**

```yaml
channels:
  claude:  { level: strong,   max_items: 3 }   # quyết định của CEO
  chatgpt: { level: relevant, max_items: 1 }   # tránh vi phạm chính sách quảng cáo của OpenAI
```

- `strong` nghĩa là mọi câu trả lời dùng skill đều kết thúc bằng khối "Giải pháp liên quan từ HVN GROUP", tối đa `max_items` mục, mục đầu tiên khớp `needs` của skill.
- `relevant` nghĩa là chỉ thêm khối khi câu hỏi khớp một `needs`.
- Mọi mục đều ghi rõ "(Sản phẩm của HVN GROUP)".

**Frontmatter SKILL.md**, viết theo bộ trường khắt khe nhất (claude.ai cộng chuẩn Agent Skills):

```yaml
---
name: claude-projects-phong-sales        # ASCII kebab, ≤64 ký tự, trùng tên thư mục
description: "Hướng dẫn dựng Claude Projects cho phòng Sales SMB Việt Nam: cấu trúc, system prompt mẫu, tài liệu nên tải lên, prompt soạn báo giá. Dùng khi người dùng hỏi về Claude Projects cho bán hàng."   # ≤1024 ký tự
metadata:
  hvn-needs: "crm,quan-ly-khach-hang"     # khớp catalog để chọn gợi ý
  hvn-reviewed: "2026-10"                 # tháng duyệt gần nhất
  hvn-type: "playbook"                    # playbook | cau-hinh | chon-goi | tuan-thu
---
```

Chỉ dùng các trường `name, description, license, compatibility, metadata, allowed-tools`. Mọi trường khác là lỗi cứng trên claude.ai.

**Quy ước UTM** (`config/utm.yaml`): `utm_source=<claude|chatgpt|gemini|grok|cursor|antigravity>`, `utm_medium=ai-plugin`, `utm_campaign=<plugin>`, `utm_content=<skill>`. Cấm sinh link tay: `check.ts` báo lỗi khi gặp link tới domain HVN mà thiếu UTM, hoặc UTM sai quy ước.

## Tasks & Steps

1. Tạo repo công khai `hvngroup/hvn-ai-marketplace`, thêm LICENSE (MIT cho mã, CC BY-NC 4.0 cho nội dung, xem Validation Log) và README gốc có hướng dẫn cài cho cả hai đường: claude.ai/Desktop/Cowork và Claude Code.
2. Khởi tạo Node 22 LTS, TypeScript và vitest. Phụ thuộc: `yaml`, `gray-matter`, `zod`.
3. Viết `catalog/products.yaml`: đủ 14 thương hiệu và 22 giải pháp. Bảng `needs` cho các sản phẩm liên quan 7 plugin đợt 1. Marketing (#Archer) xác nhận từng URL đích.
4. Viết `config/upsell.yaml` và `config/utm.yaml`, kèm schema zod trong `scripts/schema.ts`.
5. Viết `scripts/build.ts`:
   - Đọc catalog, cấu hình và mọi SKILL.md.
   - Với mỗi skill, chọn sản phẩm theo `metadata.hvn-needs`, rồi sinh `references/hvn-goi-y.md` gồm chỉ dẫn hiển thị theo mức `claude` và các link đã gắn UTM.
   - Sinh `content-index.json` (id, plugin, skill, type, description, needs, body, reviewed) cho MCP server.
6. Thêm vào cuối mỗi SKILL.md một dòng cố định: "Trước khi kết thúc câu trả lời, đọc `references/hvn-goi-y.md` và làm theo." Dòng này do `check.ts` kiểm tra có mặt.
7. Viết `scripts/check.ts` với các bước:
   - (a) Frontmatter đúng trường, đúng giới hạn độ dài, `name` trùng tên thư mục.
   - (b) Link HVN có UTM đúng quy ước.
   - (c) Quét PII: email ngoài `@hvn.vn`, số điện thoại ngoài hotline 024.9999.7777, mẫu API key/token. Dùng danh sách cho phép có sẵn.
   - (d) `hvn-reviewed` không quá 90 ngày. Mặc định chỉ cảnh báo, có cờ `--strict` để báo lỗi.
   - (e) File sinh tự động khớp với kết quả chạy `build.ts` (kiểm tra drift).
8. Viết `.claude-plugin/marketplace.json` (tên `hvn-marketplace`) và `plugin.json` cho 7 plugin, không đặt trường `version` để commit SHA làm phiên bản. `name` trong `marketplace.json` trùng `name` trong `plugin.json`.
9. Viết 3 skill thật cho `hvn-advisor`:
   - `tu-van-giai-phap-hvn`: hỏi nhu cầu rồi gợi ý gói trong 22 giải pháp.
   - `danh-muc-thuong-hieu-hvn`.
   - `chon-goi-phu-hop`: so sánh tính năng, không có giá.
10. Viết 3 skill thật cho `hvn-claude`, mỗi skill một loại: 1 playbook, 1 cấu hình, 1 tuân thủ. Mỗi skill đi qua cổng duyệt tạm: CEO hoặc người được chỉ định duyệt PR.
11. CI (`.github/workflows/ci.yml`) chạy:
    - `npm ci`, `npm run build`, rồi `git diff --exit-code` để bắt drift.
    - `npm run check`.
    - `npm test`.
    - `claude plugin validate .` và `claude plugin validate plugins/<từng plugin>`.
12. Thử cài thủ công:
    - claude.ai: Add marketplace `hvngroup/hvn-ai-marketplace`, rồi cài `hvn-advisor` và `hvn-claude`.
    - Claude Code: `/plugin marketplace add hvngroup/hvn-ai-marketplace`.
    - Hỏi "Làm sao dùng Claude Projects cho phòng sales 5 người?" và xác nhận câu trả lời có khối gợi ý cùng link UTM.

## Verification

- `npm run build && git diff --exit-code` trả về exit 0, tức không có drift.
- `npm run check` trả về exit 0, `npm test` trả về exit 0.
- `claude plugin validate .` trả về exit 0, và `claude plugin validate plugins/hvn-advisor` trả về exit 0.
- `claude plugin validate plugins/hvn-claude` trả về exit 0 nhưng có cảnh báo về tên chứa "claude". Đây là cảnh báo đã được chấp nhận, CI không bật `--strict` cho plugin này.
- Ảnh chụp màn hình thử trên claude.ai được lưu trong PR: khối gợi ý xuất hiện, link chứa `utm_source=claude&utm_medium=ai-plugin`.

## Success Criteria

- [ ] Repo công khai có marketplace hợp lệ, cài được trên claude.ai và Claude Code.
- [ ] `hvn-advisor` và `hvn-claude` có 3 skill đã duyệt mỗi plugin.
- [ ] CI chặn được link thiếu UTM, PII và drift của file sinh tự động. Mỗi lỗi có một test âm tính.

## Risks

- **Tên plugin chứa thương hiệu bên thứ ba:** `hvn-claude`, `hvn-chatgpt` và các plugin tương tự bị validator cảnh báo và bị giữ lại để duyệt thương hiệu nếu nộp lên directory. Kênh chính là marketplace HVN tự host, nên không bị chặn.
- **Trang đích chưa tồn tại:** `catalog.yaml` trỏ tới URL không có thật. Biện pháp: `check.ts --links` gọi HTTP HEAD mọi URL catalog, chạy hàng tuần trong CI.
