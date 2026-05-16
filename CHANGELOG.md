# Changelog

Tất cả thay đổi đáng chú ý của dự án **CLB Quản Lý Bán Hàng** sẽ được ghi ở đây.

Format theo [Keep a Changelog](https://keepachangelog.com/vi/1.1.0/), tuân thủ [Semantic Versioning](https://semver.org/lang/vi/).

---

## [1.2.1] — 2026-05-17

🔧 **Fix nhỏ: button "Tạo đơn" cố định đáy panel**

### Changed
- 🎯 Tách button "Tạo đơn" ra khỏi cart panel, đặt thành **footer độc lập với `flexShrink:0`** ở cuối `.order-left` — button giờ **luôn hiển thị ở đáy panel** bất kể cart đầy/rỗng, không bị ảnh hưởng bởi scroll của cart hay products
- Cart panel scrollable maxHeight giảm `45vh` → `40vh` (vì button đã tách ra ngoài)
- Button có `box-shadow` viền cam phía trên để nổi bật như sticky CTA

---

## [1.2.0] — 2026-05-17

🚚 **Shipper workflow chi tiết + Lịch ca calendar + 5 ca cố định**

### Added
- 🛵 **Shipper status flow chi tiết**: `san_sang` (chờ lấy) → `da_lay` (đã lấy hàng) → `dang_giao` → `hoan_thanh` / **`hoan_hang`** (kèm lý do)
- 📝 **ReturnReasonModal** — modal nhập lý do hoàn hàng (5 preset + custom)
- 👤 **Shipper personal view** — màn hình riêng cho shipper, chỉ thấy đơn được giao cho mình, với 4 panel: Chờ lấy / Đã lấy / Đang giao / Lịch sử
- 🔔 **Browser notification cho shipper** — khi bị Trưởng ca gán đơn (`r.sid` đổi sang chính họ)
- 📅 **ShiftSettings v2 — Calendar view** — mỗi ca là một card, hiển thị nhân sự đã gán + role, có nút "Thêm nhân sự" với picker
- 🔄 Toggle **Calendar ↔ Matrix view** trong Cài đặt → Lịch ca
- 🗓️ **5 ca cố định** mặc định: 07:30-09:30, 09:30-12:30, 12:30-14:30, 14:30-17:00, 17:00-19:00

### Changed
- 🐛 **Fix Order screen lần 2**: thêm `overflow:hidden` + `min-height:0` cho `.order-left`, customer info có scroll riêng `maxHeight:40vh`, products grid có `minHeight:140`, cart panel giảm xuống `maxHeight:45vh`. Button "Tạo đơn" giờ **luôn hiển thị** dù cart đầy
- 🔒 **Permission shipper**: chỉ `truong_ca` / `admin` được phân công shipper. Trực đơn / pha chế không thấy nút assign
- 📊 ShipperScreen với role admin/trưởng ca: 4 cột Kanban (Chờ / Đã lấy / Đang giao / Hoàn thành) + panel hoàn hàng riêng
- Status panel shipper hiển thị real workload (`da_lay` + `dang_giao` = busy)

### Database migration (cần chạy lại `supabase-schema.sql`)
- `ALTER TABLE orders ADD COLUMN IF NOT EXISTS return_reason TEXT`
- Auto reset shifts từ 3 ca cũ (`Ca sáng/trưa/chiều`) sang 5 ca mới — chỉ TRUNCATE nếu phát hiện schedule 3-ca cũ

---

## [1.1.0] — 2026-05-11

🕐 **Multi-shift role system + Push notifications**

### Added
- 🗓️ **Bảng `shifts` (plural)** — nhiều ca trong ngày (Ca sáng, Ca trưa, Ca chiều)
- 👥 **Bảng `shift_assignments`** — phân công role cho từng staff trong từng ca
- 🎯 **Effective role** — role thực tế trong ca hiện tại, **tự thay đổi theo giờ**
- 🛏️ **RestScreen** — màn hình "Đang nghỉ" cho thành viên không có ca hiện tại, kèm thông tin ca tiếp theo
- ⚙️ **Settings → Lịch ca & Phân công** — admin grid để gán role per shift per staff với UI ma trận trực quan
- 🔔 **Browser Notifications** — bell icon trên TopBar, trigger:
  - Pha chế / Trưởng ca: nhận thông báo khi có đơn mới
  - Shipper / Trưởng ca: nhận thông báo khi đơn ready ship
- 📍 Sidebar hiển thị **ca hiện tại** + badge "theo ca" khi role đã override
- 🧠 **CLAUDE.md** — project memory để future Claude Code sessions load context nhanh

### Changed
- Sidebar/TopBar dùng **effectiveRole** thay vì staff.role tĩnh
- Admin được hardcode giữ role admin xuyên ca
- Dashboard sub-text show ca hiện tại + số người trong ca thay vì ca mặc định

### Migration cần làm
- Chạy lại `supabase-schema.sql` (idempotent) → tạo 2 bảng mới + seed 3 ca + auto-gán role mặc định cho Ca sáng

---

## [1.0.0] — 2026-05-10

🎉 **Bản chính thức đầu tiên** — chuyển từ Demo sang Version 1.

### Added
- 🔐 **PIN Login** — màn đăng nhập bắt buộc khi vào app, mỗi PIN map đến một thành viên/role
- 🎯 **Role-based UI** — mỗi vai trò chỉ thấy các màn liên quan để tập trung:
  - Trưởng ca: Tổng quan + tất cả màn vận hành + Báo cáo + Nhân sự
  - Trực đơn: Trực đơn + Tổng quan
  - Pha chế: KDS
  - Shipper: Giao hàng
  - Admin: Toàn quyền + Cài đặt
- 🗂️ **Categories chỉnh sửa được** — bảng `categories` mới trong DB, tab Cài đặt → Danh mục cho phép thêm/xoá/đổi tên/đổi thứ tự
- 🚪 **Đăng xuất** — nút logout ở sidebar
- 📱 **Responsive design** — sidebar drawer trên mobile, panels tự stack, font/padding tự co
- 📋 **CHANGELOG.md** — bắt đầu đánh dấu version từ v1.0

### Changed
- 🐛 **Fix layout Order screen** — button "Tạo đơn" không còn bị che khi cart đầy (tách thành footer cố định, vùng cart có scroll riêng)
- 🚫 Loại bỏ tính năng "Switch User" — thay bằng login/logout đúng nghĩa
- 🔢 Version hiển thị trong sidebar và Login screen

### Removed
- Sidebar bỏ tag "v3.0 — Quản lý nội bộ" cũ

---

## [3.1.0] — 2026-05-10 (DEMO cuối)

### Added
- ☁️ **Tích hợp Supabase** — chuyển từ localStorage sang Postgres + realtime sync
- 📡 Realtime subscriptions trên 5 bảng (orders, products, staff, zones, shift)
- ⚡ Indicator kết nối realtime trên TopBar (xanh / vàng / đỏ)
- 📋 `supabase-schema.sql` — file SQL hoàn chỉnh để khởi tạo DB

### Changed
- Lưu trữ chuyển từ `localStorage` → Supabase Cloud (chỉ giữ PIN login ở localStorage)
- Loading screen khi đang fetch lần đầu

### Removed
- Nút "Reset dữ liệu mẫu" trong Settings (giờ data ở cloud, không thể reset client-side)

---

## [3.0.0] — 2026-05-10 (DEMO ban đầu)

### Added
- 7 màn hình hoàn chỉnh: Dashboard, Trực đơn, KDS, Giao hàng, KPI, Báo cáo, Cài đặt
- Hệ màu Shopee (`#EE4D2D`)
- Tách `by` (người tạo) và `kpi` (người nhận KPI)
- Màn Cài đặt 3 tabs: Nhân sự, Sản phẩm, Cài đặt chung
- Help popup floating button
- Recipe modal với View/Edit + import .txt + sao chép
- Export CSV với BOM UTF-8
- Mock data: 10 sản phẩm, 6 nhân viên, 5 đơn

---

## Quy ước đặt version

- **MAJOR** (1.x.x): thay đổi không tương thích ngược, refactor lớn
- **MINOR** (x.1.x): thêm tính năng tương thích ngược
- **PATCH** (x.x.1): sửa lỗi, không thêm tính năng

Khi push lên GitHub, dùng tag git để đánh dấu version:
```bash
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin v1.0.0
```
