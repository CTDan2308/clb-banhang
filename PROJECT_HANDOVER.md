# 📘 PROJECT HANDOVER — CLB Quản Lý Bán Hàng

> **Self-contained document.** Paste toàn bộ file này vào đầu một phiên chat mới là đủ context để tiếp tục phát triển.
> Cập nhật lần cuối: **v1.2.1 — 2026-05-17**

---

## 🧭 TL;DR (đọc trong 30 giây)

- **App:** Hệ thống quản lý bán hàng nội bộ cho CLB Sinh viên Bán hàng Gây quỹ
- **Stack:** Single-file React 18 (CDN) + Babel Standalone + Supabase (PostgreSQL + Realtime)
- **Live:** https://ctdan2308.github.io/clb-banhang/
- **Repo:** https://github.com/CTDan2308/clb-banhang
- **Owner:** Lê Văn Trí — CEO, Learn to Leap (GitHub: `CTDan2308`)
- **Phiên hiện tại:** v1.2.1
- **Ngôn ngữ:** Tiếng Việt
- **File code chính:** `index.html` (~2994 dòng, KHÔNG tách module)
- **Folder local:** `C:\Users\HP\clb-banhang\`
- **Style giao tiếp:** Concise, đi thẳng vào việc, không dài dòng

---

## 1. Business context

CLB sinh viên bán đồ ăn/thức uống tại chỗ + giao hàng trong khuôn viên trường:

- **Sản phẩm:** Đồ pha chế (cà phê, trà sữa), đồ sẵn (bánh mì, snack), đồ dùng ngay (bánh tráng, khoai chiên)
- **Ca làm việc:** 5 ca cố định trong ngày (07:30 → 19:00)
- **Vai trò:** Trưởng ca, Trực đơn, Pha chế, Shipper, Admin
- **Thanh toán:** Tiền mặt hoặc VietQR
- **Giao hàng:** 3 zone theo khoảng cách

Quan trọng: **Tách `by` (người tạo đơn) và `kpi` (người nhận KPI)** — sale viên có thể mang khách về nhưng nhân viên quầy tạo đơn. KPI tính cho người bán, không phải người nhập.

---

## 2. Stack & deploy

```
┌──────────────────────────────────────┐
│ GitHub Pages (chỉ file tĩnh)         │
│  └─ index.html (React + Babel CDN)   │  ← deploy free, không cần build
└─────────────┬────────────────────────┘
              │ HTTPS REST + WebSocket
              ↓
┌──────────────────────────────────────┐
│ Supabase Cloud (free tier)           │
│  ├─ PostgreSQL                        │
│  ├─ Realtime via WebSocket            │
│  └─ Row Level Security (open policy) │
└──────────────────────────────────────┘
```

**Không có:** Node.js, npm, build step, bundler, server, framework backend. Giữ kiến trúc thuần CDN.

---

## 3. Supabase config

```js
const SUPABASE_URL = "https://ebfpxalvpksgnaeemcwl.supabase.co";
const SUPABASE_KEY = "sb_publishable_8HPd1lPa6i5TReqNmsre8A_NL32itNY";
```

> ✅ **SAFE to commit:** Anon (publishable) key public theo thiết kế Supabase. Bảo mật được kiểm soát qua RLS policies trong DB, không phải qua key bí mật. **KHÔNG BAO GIỜ commit `service_role` key.**

Dashboard: https://supabase.com/dashboard/project/ebfpxalvpksgnaeemcwl

---

## 4. Database schema (8 bảng)

```sql
products             — sản phẩm + công thức JSONB           [PK: id]
staff                — nhân sự + PIN 4 số                   [PK: id]
orders               — đơn hàng + items JSONB               [PK: num]
zones                — phí ship theo khoảng cách            [PK: id]
categories           — danh mục sản phẩm (editable)         [PK: key]
shifts               — 5 ca trong ngày                      [PK: id SMALLINT]
shift_assignments    — map shift→staff→role per ca          [PK: (shift_id,staff_id)]
shift                — LEGACY singleton (không dùng)        [PK: id=1]
```

### Key columns ⚠️

```
orders.num           BIGINT PK (display = "ĐH" + lpad(num,3,'0'))
orders.st            text - 8 trạng thái: moi, pha_che, san_sang, da_lay,
                                          dang_giao, hoan_thanh, hoan_hang, huy
orders.by_id         người TẠO đơn (auto từ currentUser.id)
orders.kpi_id        người NHẬN KPI (có thể khác by_id)
orders.sid           shipper id (khi đã phân công)
orders.return_reason text — lý do hoàn hàng (NEW v1.2)

products.on_menu     boolean - hiện trên menu Order không (đổi tên từ `on`)
products.recipe      JSONB { serves, prep, ing[], steps[] }

shift_assignments.role  override staff.role mặc định cho ca này
                        - "off" = đang nghỉ → RestScreen
```

### RLS (đã set sẵn — open access)

```sql
ALTER TABLE [tên] ENABLE ROW LEVEL SECURITY;
CREATE POLICY "open_all" ON [tên] FOR ALL TO anon, authenticated
  USING (true) WITH CHECK (true);
```

> Phù hợp demo nội bộ CLB sinh viên. KHÔNG dùng cho app public production. Khi cần lock-down, sửa policy thành `USING (auth.uid() IS NOT NULL)` + thêm Supabase Auth.

### Realtime publication

Tất cả 7 bảng (trừ `shift` legacy) đều `ADD TABLE` vào `supabase_realtime` publication.

File schema đầy đủ: `supabase-schema.sql` — **idempotent, chạy lại an toàn**.

---

## 5. State management pattern

### Pattern cốt lõi

```js
// Trong App component:
const [orders, _setOrders] = useState([]);
const ordersRef = useRef([]);

// Wrapper: diff với prev → tự INSERT/UPDATE/DELETE lên Supabase
const setOrders = useCallback((updater) => {
  const prev = ordersRef.current;
  const next = typeof updater === 'function' ? updater(prev) : updater;
  ordersRef.current = next;
  _setOrders(next);
  syncToDb(prev, next, "orders", "num", orderToDb);
}, []);
```

**Tóm tắt:**
- App giữ TOÀN BỘ state, truyền xuống qua props (không context, không Redux)
- Mỗi state có cặp `_setX` (raw) + `setX` (wrapper auto-sync DB)
- Realtime echoes back vào `_setX` để dedupe — tránh loop
- `shift_assignments` dùng composite key → `upsertAssignment` riêng

### Mappers (DB ↔ JS)

```js
// DB dùng snake_case, JS dùng camelCase/short
orderFromDb(r) → {id:'ĐH001', num, t, cus, ph, dFee, dT, by, kpi, returnReason, ...}
orderToDb(o)   → {num, t, cus, ph, d_fee, d_t, by_id, kpi_id, return_reason, ...}

productFromDb(r) → {..., on: r.on_menu}      // chuyển on_menu → on
productToDb(p)   → {..., on_menu: p.on}

shiftFromDb(r) → {name, start: r.start_at, end: r.end_at}
shiftToDb(s)   → {name, start_at: s.start, end_at: s.end}
```

Đa số bảng (staff, zones, categories) **không cần mapper** — keys trùng DB.

---

## 6. App component tree

```
App (root state + Supabase sync + realtime)
├── LoginScreen (PIN numpad — BẮT BUỘC vào)
├── RestScreen (khi role="off" theo ca hiện tại)
├── Sidebar (mobile drawer, nav lọc theo effectiveRole)
├── TopBar (current shift, notif bell, conn status)
├── 7 screens (lọc theo ROLE_PERMS):
│   ├── Dashboard          — stats + bar chart + bảng đơn gần đây
│   ├── OrderScreen        — tạo đơn (left panel) + đơn đang xử lý (right)
│   ├── KitchenScreen      — KDS 3 cột kanban + RecipeModal
│   ├── ShipperScreen      — 2 view: shipper-only OR admin-kanban
│   ├── StaffScreen        — bảng KPI + leaderboard
│   ├── ReportScreen       — metrics + xuất CSV
│   └── SettingsScreen tabs:
│       ├── StaffSettings        — add/edit/delete + PIN
│       ├── ProductSettings      — add/edit/delete + price/stock/toggle + recipe
│       ├── CategorySettings     — editable categories
│       ├── ShiftSettings        — Calendar OR Matrix view
│       └── GeneralSettings      — zones + shift info
├── Toast (bottom-right notifications)
└── HelpPopup (floating button)
```

### Color system (Shopee-inspired)

```css
--or:  #EE4D2D    /* Primary orange-red */
--orl: #FFF4F0    /* Light bg, sidebar active */
--ord: #C73D22    /* Hover */
--bg:  #F5F5F5    /* Page bg */
--bd:  #E8E8E8    /* Border */
--text:#212121
--mut: #767676
```

Role colors:

| Role | text | bg |
|---|---|---|
| `truong_ca` | `#1D4ED8` | `#EFF6FF` (blue) |
| `truc_don`  | `#EE4D2D` | `#FFF4F0` (orange) |
| `pha_che`   | `#D97706` | `#FFFBEB` (amber) |
| `shipper`   | `#16A34A` | `#F0FDF4` (green) |
| `admin`     | `#7C3AED` | `#F5F3FF` (purple) |

---

## 7. Login & role system

### Flow

```
[Load app]
   ↓
[Fetch all data từ Supabase]
   ↓
[Check localStorage clb_pin]
   ├─ match staff.pin + active=true → auto-login
   └─ không match → hiện LoginScreen numpad
```

### Static role vs Effective role

- **`staff.role`** = role mặc định (lưu vĩnh viễn)
- **`effectiveRole`** = role thực tế trong ca hiện tại (từ `shift_assignments`)
  - Tính: `getCurrentShift()` → query `shift_assignments` → fallback `staff.role`
  - `admin` LUÔN giữ admin role bất kể ca
  - `"off"` = đang nghỉ → show `RestScreen`

### ROLE_PERMS

```js
const ROLE_PERMS = {
  truong_ca: ["dashboard","order","kitchen","shipper","staff","report"],
  truc_don:  ["order","dashboard"],
  pha_che:   ["kitchen"],
  shipper:   ["shipper"],
  admin:     ["dashboard","order","kitchen","shipper","staff","report","settings"],
};
```

Sidebar lọc nav theo `effectiveRole`. Default screen = `allowed[0]`.

### PIN mặc định (seed)

| PIN | Vai trò | Tên |
|---|---|---|
| 1111 | Trưởng ca | Hoàng Khánh |
| 2222 | Trực đơn | Nguyễn Hà |
| 3333 | Pha chế | Trần Minh |
| 4444 | Pha chế | Lê Ngọc |
| 5555 | Shipper | Phạm Tuấn |
| 6666 | Shipper | Vũ Linh |

> ⚠️ Hiện chưa có PIN nào là `admin`. Để test admin: vào Supabase → table staff → đổi role 1 staff thành `admin`.

---

## 8. Order workflow

```
[Trực đơn] tạo đơn (st="moi")
   ↓
[Pha chế] bấm "Bắt đầu pha" → st="pha_che"
   ↓
[Pha chế] bấm "Hoàn thành pha" → st="san_sang"
   ↓
   ├─ Tại chỗ → "Hoàn thành" → st="hoan_thanh"
   │
   └─ Ship: chờ Trưởng ca/Admin phân shipper (sid)
        ↓
        [Shipper] "Đã lấy hàng" → st="da_lay"
        ↓
        [Shipper] "Bắt đầu giao" → st="dang_giao"
        ↓
        [Shipper] → "Hoàn thành" (st="hoan_thanh")
                  OR → "Hoàn hàng" + reason (st="hoan_hang")

Từ bất kỳ status nào → "Huỷ" (st="huy")
```

### Permission cho shipper actions

- **Chỉ truong_ca/admin** được gán shipper (set `sid`)
- **Shipper** chỉ thấy đơn của mình (`o.sid === currentUser.id`), tự đổi st `san_sang → da_lay → dang_giao → hoan_thanh/hoan_hang`
- Khi shipper nhận đơn (sid đổi sang chính họ) → **browser notification** tự động

---

## 9. Multi-shift system

### 5 ca cố định

| ID | Tên | Giờ |
|---|---|---|
| 1 | Ca 1 | 07:30 – 09:30 |
| 2 | Ca 2 | 09:30 – 12:30 |
| 3 | Ca 3 | 12:30 – 14:30 |
| 4 | Ca 4 | 14:30 – 17:00 |
| 5 | Ca 5 | 17:00 – 19:00 |

### Phân công role per ca

`shift_assignments` table override `staff.role` cho từng ca. Ví dụ Hà thường là Trực đơn nhưng ca 3 có thể là Trưởng ca.

### Admin UI (Settings → Lịch ca & Phân công)

2 view toggle:
- **Calendar view** (mặc định) — mỗi ca là card riêng, list nhân sự + role + nút "+ Thêm nhân sự"
- **Matrix view** — bảng nhân sự × ca, mỗi cell là dropdown role

---

## 10. Browser notifications

- Toggle bell icon trên TopBar (`clb_notif` trong localStorage)
- Permission: `Notification.requestPermission()`
- Trigger theo role + event type:

| Sự kiện | Notify ai |
|---|---|
| Đơn mới INSERT | pha_che, truong_ca |
| Đơn `st=san_sang` + ship | truong_ca, admin (để phân shipper) |
| Đơn `sid` đổi sang user hiện tại | shipper đó (cá nhân) |

Implement trong realtime subscription handler — không poll.

---

## 11. Responsive

CSS breakpoints:
- **≥768px**: layout đầy đủ, sidebar mở
- **<768px**: sidebar drawer (hamburger), order panels stack dọc, hide TopBar meta
- **<480px**: stats grid 2 cột thay 4

Classes:
```css
.app-shell, .app-main, .sidebar, .sb-overlay
.order-shell, .order-left, .order-right
.mobile-only, .desktop-only, .hide-mobile
.grid-stats-4, .grid-stats-5
```

Quan trọng: `.order-left` có `overflow:hidden; min-height:0` để button "Tạo đơn" cố định đáy không bị đẩy khỏi viewport (xem mục Layout fix bên dưới).

---

## 12. Order screen layout (đã fix nhiều lần — v1.2.1)

```
.order-left (flex column, overflow:hidden, min-height:0)
├─ Customer info     (flexShrink:0, maxHeight:40vh, overflowY:auto)
├─ Search + cat tabs (flexShrink:0)
├─ Products grid     (flex:1, overflowY:auto, minHeight:140)
├─ Cart panel        (flexShrink:0, maxHeight:40vh, overflowY:auto)
└─ 🟠 Footer button  (flexShrink:0, borderTop:2px var(--or), shadow)
                       ↑ LUÔN ở đáy panel, không phụ thuộc cart scroll
```

Bug đã fix:
1. v1.0: button trong cart panel, bị che khi cart đầy
2. v1.2.0: thêm `overflow:hidden` cho `.order-left`, vẫn nằm trong cart panel
3. v1.2.1: **tách button ra footer độc lập** ← final fix

---

## 13. Deployment & versioning

### File structure

```
clb-banhang/
├── index.html              # TẤT CẢ code app (~3000 dòng)
├── supabase-schema.sql     # DB schema + RLS + seed (idempotent)
├── README.md               # Hướng dẫn setup từ 0
├── CHANGELOG.md            # Lịch sử version (Keep a Changelog)
├── CLAUDE.md               # Memory cho Claude Code
├── PROJECT_HANDOVER.md     # File này — handover sang chat mới
├── PLAYBOOK.md             # Patterns tái sử dụng cho app khác
├── LICENSE                 # MIT
├── .nojekyll               # Tắt Jekyll trên GitHub Pages
└── .gitignore
```

### Quy ước version (SemVer)

- **PATCH** `v1.2.1`: bug fix
- **MINOR** `v1.x.0`: thêm tính năng tương thích
- **MAJOR** `v2.0.0`: breaking change

Có hằng `const VERSION = "1.2.1"` trong `index.html` + git tag tương ứng.

### Workflow release

```bash
cd ~/clb-banhang
# 1. Sửa code
# 2. Bump VERSION trong index.html
# 3. Update CHANGELOG.md (theme Keep a Changelog)
git add .
git commit -m "vX.Y.Z: <mô tả ngắn>"
git tag -a vX.Y.Z -m "Release vX.Y.Z"
git push && git push origin vX.Y.Z
```

GitHub Pages tự rebuild ~30-60s sau push.

### ⚠️ Gotcha quan trọng

Phải `cd ~/clb-banhang` TRƯỚC khi git commands. Đã từng nhầm chạy ở `~` (home directory) khiến có git repo accidentally tại home + lock file kẹt.

Prompt phải hiện: `HP@CTDan MINGW64 ~/clb-banhang (main)` — không phải `~ (master)`.

---

## 14. Security model

| Item | Public? | Lý do |
|---|---|---|
| `index.html` | ✅ public | Toàn bộ frontend |
| `SUPABASE_URL` | ✅ public | URL endpoint, không nhạy cảm |
| `SUPABASE_KEY` (anon/publishable) | ✅ public | Public by design, bảo mật qua RLS |
| `supabase-schema.sql` | ✅ public | Schema không phải secret |
| Staff PINs trong DB | ❌ trong DB | Plain text trong DB (acceptable cho demo nội bộ) |
| Supabase `service_role` key | ❌ TUYỆT ĐỐI KHÔNG commit | Bypass RLS, full DB access |

### Threat model hiện tại

- ✅ Ai vào URL cũng phải nhập PIN trước (LoginScreen)
- ✅ RLS bảo vệ ai có anon key không tự ý truy DB qua REST
- ❌ Anyone có PIN có thể login (chia sẻ PIN giữa CLB OK)
- ⚠️ RLS "open_all" → ai có anon key có thể đọc/ghi mọi bảng qua API
- ⚠️ Không có rate limiting → spam orders có thể
- ⚠️ PIN plain text trong DB → kẻ tấn công có DB access thấy được

### Bước nâng cấp bảo mật (cho future)

- Hash PIN với bcrypt (cần service_role function)
- Replace PIN bằng Supabase Auth + magic link/OTP
- RLS chặt: `auth.uid()` based policies
- Rate limit via Supabase Edge Function

---

## 15. Free tier Supabase limits

| Tài nguyên | Giới hạn | Bottleneck cho CLB? |
|---|---|---|
| Database storage | 500 MB | ❌ ~500k đơn được — đủ 10+ năm |
| Bandwidth | 5 GB/tháng | ❌ Đủ |
| Realtime connections | 200 | ❌ Đủ cho 100+ thành viên |
| **Realtime messages** | **2 triệu/tháng** | ⚠️ **Bottleneck thực** |
| API requests | Unlimited | ❌ |
| Auto-pause | 7 ngày không active | ⚠️ Khi nghỉ Tết/hè |

**Realtime message math:**
- Mỗi đơn full cycle = ~7 × N messages (N = số người online)
- Với 10 staff online: ~28k đơn/tháng max (≈ 950 đơn/ngày)
- Với 5 staff online: ~57k đơn/tháng (≈ 1900 đơn/ngày)

**Optimization khi gần trần:**
```sql
-- Tắt realtime cho bảng ít thay đổi
ALTER PUBLICATION supabase_realtime DROP TABLE shift, categories, zones;

-- Archive đơn cũ
DELETE FROM orders WHERE st = 'hoan_thanh' AND created_at < NOW() - INTERVAL '60 days';
```

---

## 16. Common issues & fixes

| Triệu chứng | Nguyên nhân | Cách sửa |
|---|---|---|
| Login PIN không vào được | Staff bị `active=false` hoặc PIN sai | Vào DB check staff table |
| TopBar "Mất kết nối" (đỏ) | WebSocket reset / tab nền lâu / Supabase paused | Reload tab; nếu paused → vào dashboard unpause |
| Đơn không hiện realtime | Bảng chưa add vào publication | Chạy `ALTER PUBLICATION supabase_realtime ADD TABLE X` |
| RLS error 401/403 | Policy chưa cho phép | Re-run schema.sql |
| `Cannot read properties of undefined` | Race condition với currentUser khi mount | Đã có loading guard; kiểm tra mới |
| Button "Tạo đơn" bị che | Layout flex thiếu min-height:0 hoặc button nằm trong cart panel có maxHeight | v1.2.1 đã fix: tách button ra footer riêng flexShrink:0 |
| Push GitHub fail "index.lock" | Có .git ở home directory nhầm | `cd ~/clb-banhang` trước khi run git |

---

## 17. Convention coding

### Style code

- **Inline styles** — không có CSS-in-JS framework. CSS variables trong `:root`.
- **Class names** chỉ dùng cho responsive: `.order-shell`, `.mobile-only`, `.hide-mobile`, ...
- **Component nhỏ** định nghĩa trong cùng file. Không tách module.
- **Naming:**
  - State: `[orders, setOrders]`
  - Refs: `ordersRef`
  - Raw setters: `_setOrders` (gạch dưới đầu)
  - Mappers: `orderFromDb`, `orderToDb`
  - Helpers: `nxSt`, `nxLb`, `iSub`, `iTot`, `fmt`, `nowS`, `todayS`

### Tránh khi sửa

- ❌ Đọc cả `index.html` — luôn dùng Grep + Read offset/limit
- ❌ Refactor sang nhiều file — single-file là feature
- ❌ Đổi tên column DB — code có mappers, đổi = refactor cả mapper + queries
- ❌ Xoá legacy table `shift` — giữ backward compat
- ❌ Add Node.js / build tools / npm
- ❌ Hardcode lại CL (categories) — đã dynamic từ DB
- ❌ Commit `service_role` key

---

## 18. Lịch sử version (tóm tắt)

| Version | Highlight |
|---|---|
| v3.0 | Demo single-file React + localStorage (6 màn) |
| v3.1 | Migrate sang Supabase + realtime sync |
| v1.0 | First production — PIN login, role-based UI, categories editable, responsive |
| v1.1 | Multi-shift role + browser notifications |
| v1.2.0 | 5 ca cố định + shipper workflow chi tiết (8 states + hoàn hàng) + shift calendar UI |
| v1.2.1 | Fix button "Tạo đơn" cố định đáy panel (tách footer riêng, không bị đẩy) |

Chi tiết: xem `CHANGELOG.md`.

---

## 19. Roadmap

- [x] v1.0 — PIN login, role-based UI, categories editable, responsive
- [x] v1.1 — Multi-shift, role thay đổi theo ca, browser notifications
- [x] v1.2 — 5 ca cố định + shipper workflow + calendar UI + button fix
- [ ] **v1.3** — Stored procedure cho stock decrement (atomic, tránh race condition khi nhiều người tạo đơn)
- [ ] **v1.4** — VietQR API tự sinh QR theo đơn
- [ ] **v1.5** — Lịch sử ca cumulative (reports across shifts/days), so sánh ca
- [ ] **v1.6** — Hash PIN với bcrypt (cần Edge Function)
- [ ] **v2.0** — Supabase Auth (magic link/OTP) thay PIN tự build
- [ ] **v3.0** — Migrate sang Next.js + Vercel nếu cần SSR/SEO

---

## 20. Khi bắt đầu phiên chat mới

### Prompt mẫu paste vào chat mới

```
Tôi đang phát triển app "CLB Quản Lý Bán Hàng" trên repo
https://github.com/CTDan2308/clb-banhang. File code chính ở
C:\Users\HP\clb-banhang\index.html.

Đây là context đầy đủ — đọc kỹ trước khi làm việc:

[Paste toàn bộ nội dung file PROJECT_HANDOVER.md này]

Nhiệm vụ tiếp theo: [Mô tả nhiệm vụ cụ thể]
```

### Cách Claude/người mới nên tiếp cận

1. Đọc TL;DR + section liên quan tới nhiệm vụ
2. `Grep` tìm component cần sửa (KHÔNG đọc cả file)
3. `Read` với `offset/limit` đúng đoạn cần
4. Edit chính xác bằng `Edit` tool với context đủ unique
5. Nếu cần migrate DB → cập nhật `supabase-schema.sql` + báo user chạy lại
6. Bump version → CHANGELOG → commit + tag → push

---

## 21. Liên hệ / Owner notes

- **Owner:** Lê Văn Trí — CEO, Learn to Leap
- **Bối cảnh:** EdTech STEM/Robotics/AI cho trường học Việt Nam
- **App này:** Side project cho CLB Sinh viên Bán hàng Gây quỹ (không phải sản phẩm thương mại)
- **Style trao đổi:** Tiếng Việt, concise, không cần xã giao dài dòng

---

*End of handover. Tài liệu này tự đủ — không cần đọc thêm file nào khác để hiểu app.*
