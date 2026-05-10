# Changelog

Tất cả thay đổi đáng chú ý của dự án **CLB Quản Lý Bán Hàng** sẽ được ghi ở đây.

Format theo [Keep a Changelog](https://keepachangelog.com/vi/1.1.0/), tuân thủ [Semantic Versioning](https://semver.org/lang/vi/).

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
