# CLAUDE.md — Project Memory for Claude Code

> File này được Claude Code đọc tự động ở đầu mỗi session. Nội dung cô đọng để tiết kiệm token.

## 🎯 Project

**CLB Quản Lý Bán Hàng** — hệ thống quản lý bán hàng nội bộ cho CLB Sinh viên Bán hàng Gây quỹ.

- **Owner:** Lê Văn Trí — CEO, Learn to Leap (CTDan2308 trên GitHub)
- **Stack:** Single-file React 18 (CDN) + Babel Standalone + Supabase (PostgreSQL + Realtime)
- **Deploy:** GitHub Pages → https://ctdan2308.github.io/clb-banhang/
- **Repo:** https://github.com/CTDan2308/clb-banhang
- **Supabase:** project `ebfpxalvpksgnaeemcwl` (URL + anon key đã nhúng trong `index.html`)
- **Ngôn ngữ giao tiếp:** Tiếng Việt
- **Style:** Concise, không dài dòng, đi thẳng vào việc

## 📁 Cấu trúc file (giữ TỐI THIỂU)

```
clb-banhang/
├── index.html               # TẤT CẢ code app (single file, ~2500 dòng)
├── supabase-schema.sql      # DB schema + RLS + realtime + seed (idempotent)
├── CHANGELOG.md             # Lịch sử version
├── CLAUDE.md                # File này
├── README.md                # Hướng dẫn deploy + setup
├── LICENSE                  # MIT
├── .gitignore, .nojekyll
```

Không tạo thêm file mới trừ khi thật cần. Mỗi file mới = thêm chi phí maintain.

## 🗄️ Database tables (Supabase)

| Table | Mô tả | PK |
|---|---|---|
| `products` | Sản phẩm bán + công thức (recipe JSONB) | `id` |
| `staff` | Nhân sự + PIN 4 số | `id` |
| `orders` | Đơn hàng + items JSONB | `num` |
| `zones` | Phí ship theo khoảng cách | `id` |
| `categories` | Danh mục sản phẩm (editable) | `key` |
| `shifts` | Các ca trong ngày (v1.1+) | `id` |
| `shift_assignments` | Map shift → staff → role (v1.1+) | `(shift_id, staff_id)` |
| `shift` | LEGACY singleton — không dùng nữa, giữ vì backward compat | `id=1` |

Cấu trúc cụ thể: xem `supabase-schema.sql`. Schema **idempotent** — chạy lại an toàn.

## 🧩 Kiến trúc React trong index.html

```
App (root state, Supabase sync, realtime subscriptions)
├── LoginScreen (PIN numpad - bắt buộc trước khi vào)
├── RestScreen (khi role=off / đang nghỉ)
├── Sidebar (lọc nav theo effectiveRole)
├── TopBar (current shift, notif toggle, status)
└── 7 screens:
    Dashboard, OrderScreen, KitchenScreen, ShipperScreen,
    StaffScreen, ReportScreen, SettingsScreen
    │
    └── SettingsScreen tabs:
        StaffSettings, ProductSettings, CategorySettings,
        ShiftSettings, GeneralSettings
```

## ⚙️ Conventions QUAN TRỌNG

### State management
- App giữ **toàn bộ state**, truyền xuống qua props
- Mỗi state có: `_setX` (raw setter) + `setX` (diff wrapper auto-sync Supabase)
- Trừ `shift_assignments`: composite key, dùng `upsertAssignment`/`deleteAssignment` thay vì diff wrapper
- Realtime echoes back vào `_setX` để dedupe

### Mappers (DB ↔ JS)
- `orderFromDb/orderToDb`, `productFromDb/productToDb`, `shiftFromDb/shiftToDb`
- Đa số bảng dùng snake_case trùng JS, không cần mapper
- Order ID hiển thị = `'ĐH' + str(num).padStart(3,'0')` — `num` là PK

### Role system (v1.1+)
- **Static role** (`staff.role`) = role mặc định
- **Effective role** = role theo ca hiện tại (từ `shift_assignments`), fallback về static
- Admin LUÔN giữ admin role bất kể ca
- Role `"off"` = đang nghỉ → hiển thị RestScreen
- Sidebar filter nav theo `effectiveRole` qua `ROLE_PERMS`

### Versioning
- SemVer trong const `VERSION`
- Mỗi release: bump version → update `CHANGELOG.md` → commit → `git tag -a vX.Y.Z` → push tag
- Commit message: `vX.Y.Z: <mô tả ngắn>`

### Style code
- Inline styles (không có CSS-in-JS framework). Token CSS variables trong `:root` (`--or`, `--bg`, ...).
- Class names CSS chỉ dùng cho responsive: `.order-shell`, `.order-left`, `.order-right`, `.mobile-only`, `.hide-mobile`
- Component nhỏ định nghĩa trong cùng file, không tách module

## 🎨 Color system (Shopee-inspired, đã chốt)

```
--or:  #EE4D2D    primary
--orl: #FFF4F0    light bg
--ord: #C73D22    hover
--bg:  #F5F5F5    page bg
--bd:  #E8E8E8    border
```

Role colors: blue (trưởng ca), orange (trực đơn), amber (pha chế), green (shipper), purple (admin).

## 🔁 Workflow phổ biến

### Khi user yêu cầu sửa UI / thêm tính năng
1. Đọc CHANGELOG để biết version hiện tại
2. Identify file(s) cần sửa — đa số là `index.html`
3. Dùng `Grep` để tìm component, `Read` lấy lines cụ thể (KHÔNG đọc cả file)
4. Edit chính xác bằng `Edit` tool với `old_string` đủ context để unique
5. Nếu thêm column/table: sửa `supabase-schema.sql` + báo user chạy lại
6. Bump version → update CHANGELOG → commit + tag

### Khi user nói "không chạy được"
1. Hỏi rõ: lỗi gì? Console browser có message gì?
2. `git log --oneline -5` xem commit gần đây
3. Đọc đúng file/dòng liên quan
4. KHÔNG đoán — verify bằng Read/Grep

### Push lên GitHub
```bash
cd ~/clb-banhang
git add .
git commit -m "vX.Y.Z: <mô tả>"
git tag -a vX.Y.Z -m "Release vX.Y.Z"
git push && git push origin vX.Y.Z
```

User phải đứng ở `~/clb-banhang`, KHÔNG phải `~` (đã từng nhầm).

## 🚫 Tránh

- **KHÔNG đọc toàn bộ index.html** — luôn dùng `Grep` + `Read` với `offset/limit`
- **KHÔNG refactor sang nhiều file** — single-file là feature, không phải bug (không cần build step, deploy GitHub Pages free)
- **KHÔNG đổi tên column DB** — code có mappers, đổi tên = refactor cả mappers + queries
- **KHÔNG xoá legacy table `shift`** — giữ để backward compat
- **KHÔNG add Node.js/build tools** — phá kiến trúc CDN React
- **KHÔNG commit secret keys** — Supabase anon key OK (public by design), nhưng nếu sau này có service_role key → tuyệt đối không commit

## 💡 Token-saving tips khi làm việc với Claude Code

- Bắt đầu session bằng `/clear` nếu task mới hoàn toàn
- Cho file path + dòng cụ thể thay vì để Claude search
- Yêu cầu Claude dùng sub-agent Explore cho research dài
- Nhiệm vụ phức tạp: vào plan mode (`/plan` hoặc Shift+Tab) trước, review plan, rồi mới implement
- Sau mỗi feature lớn: commit + bump version → context "đóng gói" được trong git log

## 📝 Roadmap

- [x] v1.0 — PIN login, role-based UI, categories editable, responsive
- [x] v1.1 — Multi-shift, role thay đổi theo ca, browser notifications
- [ ] v1.2 — Stored procedure cho stock decrement (tránh race condition)
- [ ] v1.3 — VietQR API tự sinh QR theo đơn
- [ ] v1.4 — Lịch sử ca (cumulative reports across shifts/days)
- [ ] v2.0 — Migrate sang Next.js + Vercel (nếu cần SSR/SEO)

## 🆘 Khi gặp lỗi thường gặp

| Triệu chứng | Có thể là |
|---|---|
| Login PIN không vào được | Staff bị `active=false`, hoặc PIN sai DB |
| "Mất kết nối" (đỏ) ở TopBar | Tab background quá lâu / WebSocket reset / Supabase paused |
| Đơn không hiện realtime | Bảng chưa add vào `supabase_realtime` publication |
| RLS error 401/403 | Policy chưa cho phép → check `open_all` policy trong schema.sql |
| `Cannot read properties of undefined (reading 'X')` | Race condition khi mount, thường là dùng currentUser trước khi load xong |
