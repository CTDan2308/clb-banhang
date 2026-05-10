# 🛒 CLB Quản Lý Bán Hàng v3.1

> Hệ thống quản lý bán hàng nội bộ cho **Câu lạc bộ Sinh viên bán hàng Gây quỹ** — tạo đơn, pha chế, giao hàng, theo dõi KPI và báo cáo doanh thu trong từng ca.

**Single-page app** chạy trên trình duyệt + **Supabase Cloud** làm database/realtime.  
Nhiều người cùng dùng sẽ thấy đơn/trạng thái cập nhật **tức thì** giữa các thiết bị.

---

## ✨ Tính năng

- 📋 **Tạo đơn nhanh** — tách rõ *Người tạo đơn* và *Người nhận KPI*
- ☕ **KDS (Kitchen Display)** — kanban 3 cột Mới / Đang pha / Sẵn sàng, có công thức từng món
- 🛵 **Quản lý giao hàng** — phân công shipper, theo dõi trạng thái
- 🎯 **KPI cá nhân** — bảng xếp hạng theo doanh thu KPI
- 📊 **Báo cáo cuối ca** — xuất CSV (Excel mở được, có BOM UTF-8)
- ⚙️ **Cài đặt linh hoạt** — thêm/sửa/xoá nhân sự, sản phẩm, công thức, phí ship
- ☁️ **Realtime sync qua Supabase** — đơn tạo ở máy A hiện ngay ở máy B/KDS/Shipper
- 👤 **User switcher** — chuyển vai nhanh không cần PIN (demo)

---

## 🏗️ Kiến trúc

```
┌─────────────────────────────┐
│ GitHub Pages (index.html)    │  ← UI tĩnh, deploy free
│ React 18 + Babel CDN          │
└──────────────┬──────────────┘
               │ HTTPS REST + WebSocket
               ↓
┌─────────────────────────────┐
│ Supabase Cloud (free tier)  │
│ • PostgreSQL (5 bảng)        │
│ • Realtime channels          │
│ • RLS policies (open access) │
└─────────────────────────────┘
```

5 bảng: `products`, `staff`, `orders`, `zones`, `shift` (xem `supabase-schema.sql`).

---

## 🚀 Cách chạy lần đầu (đầy đủ từ 0)

### 1. Tạo project Supabase

1. Đăng ký miễn phí tại https://supabase.com (đăng nhập bằng GitHub cho nhanh)
2. **New Project** → đặt tên `clb-banhang` → region **Southeast Asia (Singapore)** → tạo password DB → **Create**
3. Đợi ~1-2 phút project được provision

### 2. Chạy schema SQL

1. Mở project → menu trái **SQL Editor** (icon ⚡)
2. Bấm **+ New query**
3. Mở file [`supabase-schema.sql`](supabase-schema.sql) trong repo, **copy toàn bộ**, paste vào SQL Editor
4. Bấm **Run** (hoặc Ctrl+Enter)
5. Phải thấy `Success. No rows returned` ở cuối — DB đã có 5 bảng + dữ liệu mẫu (10 sản phẩm, 6 nhân viên, 3 zones, 1 ca)

### 3. Lấy URL + anon key

1. Menu trái **Settings** (bánh răng) → **API**
2. Copy 2 giá trị:
   - **Project URL** (dạng `https://xxx.supabase.co`)
   - **anon / public key** (dạng `sb_publishable_...` hoặc `eyJ...`)

### 4. Cập nhật trong `index.html`

Mở `index.html`, tìm dòng (gần đầu):

```js
const SUPABASE_URL = "https://ebfpxalvpksgnaeemcwl.supabase.co";
const SUPABASE_KEY = "sb_publishable_8HPd1lPa6i5TReqNmsre8A_NL32itNY";
```

Thay 2 giá trị bằng thông tin của project bạn.

### 5. Mở trong trình duyệt

```bash
# Mở trực tiếp
start index.html

# Hoặc qua local server
python -m http.server 8000
```

App sẽ kết nối Supabase, tải dữ liệu, và hiện indicator **● Realtime** ở góc trên bên phải.

---

## 🌐 Deploy lên GitHub Pages

```bash
cd C:\Users\HP\clb-banhang
git add .
git commit -m "v3.1: Tich hop Supabase realtime"
git push
```

GitHub Pages tự rebuild ~30s. App live tại:
```
https://<USERNAME>.github.io/clb-banhang/
```

> Anon key của Supabase được nhúng trong `index.html` — đây là khoá public, **an toàn** để commit lên GitHub. Bảo mật được kiểm soát qua **RLS policies** trong DB.

---

## 🧪 Kiểm tra realtime hoạt động

1. Mở app trên 2 trình duyệt khác nhau (hoặc 2 thiết bị)
2. Tạo đơn ở máy A → máy B sẽ thấy đơn xuất hiện trong vòng ~200ms
3. Bấm "Bắt đầu pha" ở máy A → trạng thái cập nhật ngay ở máy B
4. Sửa giá sản phẩm trong Settings ở máy A → menu Order ở máy B đổi giá ngay

Nếu indicator "Realtime" báo **đỏ**, kiểm tra:
- Đã chạy `ALTER PUBLICATION supabase_realtime ADD TABLE ...` trong schema chưa?
- Console browser (F12) có lỗi WebSocket không?

---

## 📁 Cấu trúc

```
clb-banhang/
├── index.html              # Toàn bộ app (React + Supabase JS via CDN)
├── supabase-schema.sql     # Schema DB + RLS + Realtime + seed
├── README.md               # File này
├── LICENSE                 # MIT
├── .nojekyll               # Tắt Jekyll trên GitHub Pages
└── .gitignore
```

---

## 🎨 Hệ màu (Shopee-inspired)

| Token  | Mã        | Dùng cho                        |
|--------|-----------|---------------------------------|
| `--or` | `#EE4D2D` | Primary (button, active, total) |
| `--orl`| `#FFF4F0` | Background nhẹ, sidebar active  |
| `--ord`| `#C73D22` | Hover state                     |
| `--bg` | `#F5F5F5` | Background chính                |
| `--bd` | `#E8E8E8` | Border                          |

---

## 👥 Vai trò

| Role        | Mô tả                                            |
|-------------|--------------------------------------------------|
| `truong_ca` | Trưởng ca — giám sát toàn bộ, báo cáo, phân công |
| `truc_don`  | Trực đơn — tạo đơn, tiếp nhận khách, thu tiền    |
| `pha_che`   | Pha chế — xem KDS, pha, đánh dấu hoàn thành      |
| `shipper`   | Shipper — xem đơn giao, xác nhận đã giao         |
| `admin`     | Admin — toàn quyền, cấu hình hệ thống            |

---

## 🛠️ Bảo trì & xử lý sự cố

### Reset toàn bộ dữ liệu

Vào Supabase Dashboard → SQL Editor:

```sql
TRUNCATE TABLE orders, products, staff, zones RESTART IDENTITY CASCADE;
DELETE FROM shift WHERE id = 1;
-- Sau đó chạy lại supabase-schema.sql để có data mẫu
```

### Backup dữ liệu

Supabase Dashboard → **Database** → **Backups** (free tier giữ 7 ngày).  
Hoặc trong app, vào Báo cáo → **Xuất CSV** để có file Excel của ngày hiện tại.

### Khi cần đổi sang project Supabase khác

Chỉ cần đổi `SUPABASE_URL` và `SUPABASE_KEY` ở đầu `index.html`, commit & push. Không cần build lại.

---

## 🛣️ Roadmap

- [x] **v3.0** — Single-file React + localStorage
- [x] **v3.1** — Tích hợp Supabase + realtime sync
- [ ] **v3.2** — PIN login thật (Supabase Auth + RLS chặt hơn)
- [ ] **v3.3** — VietQR API tự sinh QR theo từng đơn
- [ ] **v3.4** — Multi-shift, lưu lịch sử nhiều ca
- [ ] **v4.0** — Chuyển sang Next.js + Vercel cho UX tốt hơn

---

## 📝 License

MIT — dùng tự do cho mục đích học tập và CLB sinh viên.

---

*Phát triển cho Câu lạc bộ Sinh viên bán hàng Gây quỹ — Learn to Leap*
