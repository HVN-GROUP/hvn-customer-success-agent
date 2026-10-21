---
phase: 1
title: "Nền móng repo + lát cắt dọc trên Claude"
status: todo
priority: P1
effort: "6d"
dependencies: []
---

# Phase 1: Nền móng repo + lát cắt dọc trên Claude

<!-- Updated: Validation Session 2 - thêm loại nội dung lo-trinh; không tích hợp WHMCS -->
<!-- Updated: Red Team 2026-10-21 - repo staging riêng tư + mirror công khai; nguồn tách khỏi plugin sinh tự động; đơn vị nội dung; UTM/needs một nguồn; whitelist link và loại file; CI an toàn; eval -->

## Goal

Dựng repo staging (riêng tư) và repo phát hành (công khai) cùng hợp đồng dữ liệu, bước build, CI và eval. Chứng minh luồng chạy thông với `hvn-advisor` và `hvn-claude` cài được trên Claude. Xác minh sớm các giả định nền tảng còn chưa kiểm chứng.

## Hai repo, một chiều

| Repo | Quyền | Vai trò |
|---|---|---|
| `hvngroup/hvn-ai-marketplace-staging` | **Riêng tư** | Nơi soạn, PR, CI, duyệt. Chỉ cho squash-merge vào `main`. |
| `hvngroup/hvn-ai-marketplace` | **Công khai** | Bản sao `main` của staging, do GitHub Action đẩy bằng deploy key sau mỗi merge. Không nhận PR. Không ai push tay. |
| `hvn-customer-success-agent` (repo này) | Nội bộ | Plan, skill soạn nháp (Phase 2), runbook. |

Nội dung nháp, kể cả bản có lỗi PII, **không bao giờ** xuất hiện ở repo công khai. Repo công khai chỉ nhận cây đã qua CI và người duyệt.

Repo công khai vẫn bắt buộc vì claude.ai và Claude Code thêm marketplace bằng `owner/repo`. Org sync của Team/Enterprise lại đòi repo riêng tư, nên khách lớn tự mirror hoặc upload zip (Phase 7). Nguồn: `plans/reports/researcher-261021-1130-claude-plugin-marketplace.md` §3.

## Cấu trúc repo staging

```
content/<plugin>/plugin.yaml                  # displayName, description, keywords
content/<plugin>/<skill>/SKILL.md             # NGUỒN: name, description, phần mở đầu chung của skill
content/<plugin>/<skill>/<item>.md            # NGUỒN: một nội dung = một file (đơn vị đếm N)
catalog/products.yaml                         # needs (từ vựng), brands, products, solutions
config/channels.yaml                          # kênh: up-sell, UTM, loại nội dung bị loại, công cụ tắt
config/link-allowlist.yaml                    # domain ngoài HVN được phép link (tài liệu chính thức của hãng)
scripts/build.ts                              # sinh plugins/** và dist/content-index.json
scripts/check.ts                              # kiểm tra (xem bên dưới)
scripts/schema.ts                             # schema zod cho mọi file trên
evals/<plugin>.yaml                           # 30 câu hỏi kèm skill kỳ vọng (Phase 1 cho 2 plugin, Phase 5 cho phần còn lại)
.claude-plugin/marketplace.json               # SINH TỰ ĐỘNG
plugins/<plugin>/**                           # SINH TỰ ĐỘNG, commit để Claude cài được
dist/content-index.json                       # SINH TỰ ĐỘNG, KHÔNG commit (CI và Docker tự build)
docs/                                         # cai-dat.md, runbook-go-noi-dung.md, content-style.md
.github/workflows/ci.yml, mirror.yml
.github/CODEOWNERS, pull_request_template.md
LICENSE, LICENSE-CONTENT.md, README.md          # MIT cho mã; nội dung © HVN GROUP, cho phép cài/dùng/mirror nội bộ, cấm tái phân phối công khai hay bán lại
```

Nội dung trong `plugins/<plugin>/` chỉ được phép gồm `.claude-plugin/plugin.json`, `README.md`, `skills/*/SKILL.md` và `skills/*/references/*.md`. Build chỉ sinh đúng các file này, và `check.ts` từ chối mọi file khác: `hooks/`, `bin/`, `.mcp.json`, `commands/`, `agents/`, `settings.json`, symlink.

## Hợp đồng dữ liệu (nguồn duy nhất)

**`catalog/products.yaml`**: chỉ chứa thông tin công khai, không có giá, không có giá gốc vendor.

```yaml
needs:                                   # TỪ VỰNG DUY NHẤT cho nhu cầu
  - { id: sao-luu-du-lieu, label: "Sao lưu dữ liệu" }
  - { id: quan-ly-khach-hang, label: "Quản lý khách hàng (CRM)" }
brands:
  - { id: gcs, name: GCS.VN, url: https://gcs.vn, summary: "Đối tác cấp cao Google Cloud — Google Workspace, lưu trữ, hội họp" }
products:
  - id: acronis-cyber-protect
    brand: lic                            # khóa ngoại, zod kiểm tra tồn tại
    name: "Acronis Cyber Protect"
    summary: "Sao lưu và khôi phục máy chủ, máy trạm, Microsoft 365, Google Workspace"
    url: https://lic.vn/acronis           # TRANG THÔNG TIN; Marketing xác nhận ở bước 3
    needs: [sao-luu-du-lieu]
solutions:
  - { id: S13, name: "Sao lưu & Khôi phục thảm họa", summary: "...", url: https://hvn.vn/giai-phap/s13, needs: [sao-luu-du-lieu] }
```

**`config/channels.yaml`**:

```yaml
claude:  { upsell: strong,   max_items: 3, utm_source: claude }                      # plugin Claude (build)
chatgpt: { upsell: relevant, max_items: 1, utm_source: chatgpt, mcp: true,
           exclude_types: [chon-goi], tools_disabled: [] }                           # MCP /mcp/chatgpt — đã chốt ở Validation Log
email:   { utm_source: email, utm_medium: whmcs }      # chỉ dùng cho link trong email (Phase 7)
utm:
  source_enum: [claude, chatgpt, email]                 # thêm kênh mới = thêm vào đây
  medium_enum: [ai-plugin, whmcs]
  medium: ai-plugin
  campaign: "<plugin của nội dung>; recommend_solutions dùng hvn-advisor; email dùng ai-plugin-onboarding"
  content: "<id nội dung dạng plugin.skill.item>; recommend_solutions dùng recommend"
  pattern: "^[a-z0-9.-]{1,64}$"                         # mọi giá trị utm_* phải khớp
```

- Phase 3 và Phase 6 đọc chung file này.
- Kênh không có trong file thì không tồn tại: server trả 404, CI báo lỗi.
- ChatGPT loại nội dung `chon-goi` ngay từ lần nộp đầu, vẫn giữ `recommend_solutions` (Validation Log, Session 1).

**Frontmatter của SKILL.md nguồn** (bộ trường hợp lệ trên claude.ai):

```yaml
---
name: gws-email-su-co                     # ASCII kebab, ≤64 ký tự, trùng tên thư mục
description: "Khắc phục sự cố email Google Workspace: không nhận được thư, thư vào spam, lỗi DKIM/SPF, khôi phục email bị xóa. Dùng khi người dùng hỏi về lỗi Gmail doanh nghiệp."
---
```

**Frontmatter của một nội dung `<item>.md`:**

```yaml
---
title: "Khôi phục email bị xóa trong Google Workspace"
type: cau-hinh                            # playbook | cau-hinh | chon-goi | tuan-thu | lo-trinh
needs: [sao-luu-du-lieu]                  # phải thuộc catalog.needs
reviewed: 2026-11-03                      # ngày đầy đủ
review_sla_days: 90                       # theo loại: AI playbook 60; khác 120
sources: [https://support.google.com/a/answer/...]   # domain phải thuộc link-allowlist
---
```

**Liên kết trong nội dung nguồn:**

- **Cấm link tới domain HVN.** Muốn nhắc sản phẩm thì dùng tham chiếu `[[hvn:acronis-cyber-protect]]`; build sẽ thay bằng tên sản phẩm kèm link có UTM đúng kênh.
- Link ngoài phải thuộc `link-allowlist.yaml`.

## Build sinh ra gì

1. **Cho Claude** (`plugins/**`):
   - Mỗi `<item>.md` được copy sang `references/`. Cuối file build nối thêm khối "Giải pháp liên quan từ HVN GROUP": tối đa 3 mục, chọn theo `needs` của item, link có `utm_source=claude&utm_medium=ai-plugin&utm_campaign=<plugin>&utm_content=<plugin.skill.item>`, mỗi mục ghi "(Sản phẩm của HVN GROUP)".
   - SKILL.md được sinh gồm: frontmatter, **ngay dưới tiêu đề** một dòng mô tả "Mỗi tài liệu tham chiếu kết thúc bằng khối Giải pháp liên quan; hiển thị khối đó nguyên văn ở cuối câu trả lời", phần mở đầu nguồn, và mục lục các item (tiêu đề cùng đường dẫn). Item loại `lo-trinh` luôn đứng đầu mục lục, để người dùng mới được dẫn vào lộ trình onboard trước.
   - `hvn-advisor` có skill `danh-muc-giai-phap-hvn`, sinh **toàn bộ** từ catalog: 14 thương hiệu và 22 giải pháp, mỗi giải pháp một reference. Nội dung viết tay của advisor chỉ là cách tư vấn.
   - `marketplace.json` và `plugin.json` được sinh từ `plugin.yaml`.
2. **Cho MCP** (`dist/content-index.json`): mỗi item một bản ghi `{id, plugin, skill, title, type, needs, reviewed, url, text}`.
   - `url` là link GitHub blob của item trên repo công khai.
   - `text` là nội dung nguồn, tham chiếu `[[hvn:…]]` chỉ hiện tên sản phẩm, **không** có khối gợi ý và không có link HVN. Server tự gắn link theo kênh.

## `check.ts`

Các luật **luôn chặn**, dùng cho cả hotfix:

- (a) Schema zod của mọi file nguồn; mọi khóa ngoại (`brand`, `needs`) tồn tại; `name` trùng tên thư mục; độ dài trường đúng giới hạn; mỗi liên kết nội bộ trong item `lo-trinh` trỏ tới item có thật trong cùng plugin.
- (b) Không có link HVN trong nguồn; mọi link ngoài thuộc allowlist; URL trong catalog không chứa `cart.php`, `/checkout`, `/cart`, `a=add`.
- (c) Quét PII:
  - email ngoài `@hvn.vn` và ngoài `example.*`;
  - số điện thoại ngoài hotline;
  - FQDN không thuộc allowlist, catalog hay `example.com/.vn`;
  - IPv4 và IPv6;
  - mẫu mã dịch vụ hoặc hóa đơn WHMCS (`#\d{4,}`, `ID dịch vụ`);
  - mẫu API key và token.
- (d) Lệnh nguy hiểm: `curl … | sh|bash`, `iwr … | iex`, tải script từ xa trong code block.
- (e) File trong `plugins/**` đúng whitelist; `plugins/**` khớp với kết quả chạy build (drift).
- (f) Mọi `utm_*` sinh ra khớp `pattern`.

Luật **cảnh báo**, chỉ chặn khi chạy với `--freshness`: item quá `reviewed + review_sla_days`.

Lệnh `--summary` in số item theo plugin và theo loại; đây là nguồn số liệu cho N ở Phase 5.

## Tasks & Steps

1. **Ngày 1–2, các spike xác minh** (ghi kết quả vào `plans/reports/spike-261021-platform-checks.md` của repo nội bộ):
   - (a) Thêm marketplace thử nghiệm trên claude.ai với 4 loại tài khoản: Free, Pro, thành viên Team và Owner Team. Ghi lại ai thêm được, và khi repo đổi thì bản cài trên claude.ai có tự cập nhật hay không.
   - (b) Chạy `claude plugin validate` trên runner GitHub mà không đăng nhập, với phiên bản CLI ghim.
   - (c) Xác nhận HVN sở hữu tổ chức GitHub `hvngroup`. Đăng ký các tên dễ nhầm (`hvn-group`, `hvngroup-vn`). Bật verified domain `hvn.vn` và bắt buộc 2FA.
2. Tạo 2 repo theo bảng trên. Thêm `LICENSE` (MIT, cho mã) và `LICENSE-CONTENT.md` (nội dung © HVN GROUP; được cài, dùng và mirror nội bộ, kể cả doanh nghiệp khách hàng; cấm tái phân phối công khai hay bán lại). `plugin.json` đặt `license` trỏ tới file này.
   - Staging: bật branch protection (bắt buộc PR, CI xanh, 1 approval của CODEOWNERS, chỉ squash-merge, bật merge queue, áp cho cả admin, tự hủy approval khi có commit mới).
   - Công khai: chỉ deploy key của workflow mirror được push; tắt Issues và PR từ bên ngoài, trỏ hỗ trợ về `kythuat@hvn.vn`.
3. Khởi tạo Node 24 LTS, TypeScript và vitest. Phụ thuộc: `yaml` (parse frontmatter bằng cách tự tách khối `---`; **không dùng gray-matter**, và từ chối mọi fence khác `---`), `zod`. Lockfile commit.
4. Viết `catalog/products.yaml`: 14 thương hiệu, 22 giải pháp, từ vựng `needs`, các sản phẩm liên quan 7 plugin. **Marketing xác nhận mọi URL là trang thông tin đang tồn tại trước cuối tuần 1.** URL nào chưa có trang thì tạm trỏ về trang chủ thương hiệu.
5. Viết `config/channels.yaml`, `config/link-allowlist.yaml` và `scripts/schema.ts`.
6. Viết `scripts/build.ts` theo mục "Build sinh ra gì", cùng `shared/utm.ts` (hàm `withUtm`) và `shared/render-refs.ts` (thay `[[hvn:id]]` theo kênh). Phase 3 dùng lại hai module này.
7. Viết `scripts/check.ts`. Mỗi luật có ít nhất 1 test âm tính, dùng fixture **tổng hợp** (không phải dữ liệu thật).
8. CODEOWNERS:
   - `*` thuộc nhóm kỹ thuật.
   - `/content/**` thuộc nhóm duyệt nội dung: 1 biên tập viên Content (#Valkyrie) và 1 kỹ thuật viên Service (#Witch), tên cụ thể ghi ở Validation Log.
   - `/content/**/tuan-thu-*` hoặc item `type: tuan-thu` thuộc HVN Security. CODEOWNERS khớp theo đường dẫn, nên nội dung tuân thủ đặt trong skill có tiền tố `tuan-thu-`.
   - `/.github/`, `/scripts/`, `/shared/`, `/server/`, `/config/`, `/catalog/`, `/Dockerfile`, `/deploy/` thuộc nhóm kỹ thuật kèm người phụ trách bảo mật.
9. CI `ci.yml`:
   - Trigger `pull_request` và `push` trên `main`.
   - `permissions: contents: read`, không có secret.
   - Action ghim theo SHA.
   - Các bước: `npm ci` → `npm run build` → `git diff --exit-code plugins .claude-plugin` → `npm run check` → `npm test` → `npx @anthropic-ai/claude-code@<phiên bản ghim> plugin validate .` và cho từng `plugins/*`.
   - Thêm một job hàng tuần chạy CLI bản mới nhất, chỉ cảnh báo, không chặn.
10. `mirror.yml`: khi push lên `main` của staging, đẩy cây đã build sang `main` của repo công khai. Workflow dùng deploy key lưu trong GitHub Environment `public-mirror`, chỉ cho nhánh `main` dùng.
11. Viết nguồn 3 skill cho `hvn-advisor`:
    - `tu-van-giai-phap-hvn`: cách hỏi nhu cầu và chọn giải pháp.
    - `danh-muc-giai-phap-hvn`: sinh từ catalog.
    - `chon-goi-phu-hop`: tiêu chí chọn gói theo quy mô, không có giá.
12. Viết nguồn 3 skill cho `hvn-claude`, khoảng 6 item, đủ 4 loại.
13. Viết `evals/hvn-advisor.yaml` và `evals/hvn-claude.yaml`, mỗi file 30 câu hỏi tiếng Việt thực tế kèm skill kỳ vọng. Chạy với **cả 7 plugin** đã cài (5 plugin còn lại là khung rỗng), ghi 2 chỉ số: tỷ lệ chọn đúng skill và tỷ lệ có khối gợi ý trong các câu trả lời đã chọn đúng skill. Cách chạy:
    - Nếu spike (b) cho phép, dùng `claude -p` headless.
    - Nếu không, dùng bảng chạy tay trên claude.ai, ghi kết quả vào `evals/results/<ngày>.csv`.
14. Thử cài: claude.ai (các loại tài khoản đạt ở spike (a)) và Claude Code (`/plugin marketplace add hvngroup/hvn-ai-marketplace`), cài `hvn-advisor` và `hvn-claude`.

## Verification

- `npm run build && git diff --exit-code plugins .claude-plugin` trả về exit 0.
- `npm run check` trả về exit 0, và `npm test` trả về exit 0 với các test âm tính:
  - PII (email, domain, IP, mã dịch vụ);
  - link HVN viết tay;
  - domain ngoài allowlist;
  - `curl | sh`;
  - file `hooks/` trong plugin;
  - `needs` không có trong catalog;
  - frontmatter `---js`;
  - `utm_*` sai pattern.
- `claude plugin validate` đạt exit 0 cho marketplace và từng plugin. `hvn-claude` chỉ có cảnh báo về tên, đã chấp nhận.
- Eval của `hvn-advisor` và `hvn-claude`: **chọn đúng skill ≥80%**, **có khối gợi ý ≥90%** trong các câu đã chọn đúng skill. Chưa đạt thì sửa `description` hoặc vị trí dòng hướng dẫn rồi chạy lại; không hạ ngưỡng.
- Repo công khai sau khi mirror chỉ chứa commit của `main` staging. Kiểm tra bằng `git log` của repo công khai: không có commit nào không có trong `main` staging.
- File spike tồn tại, có kết quả cho các mục (a), (b), (c).

## Success Criteria

- [ ] Marketplace công khai cài được trên các loại tài khoản đạt ở spike, và trên Claude Code.
- [ ] Hợp đồng dữ liệu có nguồn duy nhất, mọi khóa ngoại và UTM đều được CI kiểm tra.
- [ ] Eval 2 plugin đạt ngưỡng.
- [ ] Không có đường nào đưa nội dung chưa duyệt lên repo công khai.

## Risks

- **Tên plugin chứa thương hiệu bên thứ ba** (`hvn-claude`, `hvn-chatgpt`…) bị validator cảnh báo và bị giữ để duyệt thương hiệu nếu nộp directory. Kênh phát hành là marketplace tự host, nên không bị chặn.
- **Model cập nhật làm giảm tỷ lệ hiện khối gợi ý.** Chạy lại eval ở Phase 7 và sau mỗi lần Anthropic ra model mới.
