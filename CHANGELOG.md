# Changelog

Tất cả thay đổi đáng chú ý của dự án **CLB Quản Lý Bán Hàng** sẽ được ghi ở đây.

Format theo [Keep a Changelog](https://keepachangelog.com/vi/1.1.0/), tuân thủ [Semantic Versioning](https://semver.org/lang/vi/).

---

## [1.5.0] — 2026-05-26

🚀 **Role-specific apps · Customer hybrid · Shift grid v2 · Self-claim shipper pool**

Tổng hợp 3 release candidate (rc1/rc2/rc3) thành release chính thức.

### Added
- 🛵 **Shipper standalone App** (Grab Driver style): full-screen, không sidebar/topbar admin. Header đen có Thoát + connection. 3 tab: **Đơn chờ nhận** / **Đang giao** / **Lịch sử**. Active order card to với tap-to-call, mở Google Maps deep-link, CTA gradient theo trạng thái
- 🆕 **Self-claim pool**: shipper tap "Nhận đơn" → `UPDATE WHERE sid IS NULL` (concurrency-safe). Trưởng ca vẫn có quyền gán thủ công
- ☕ **Pha chế standalone App**: header xanh, KitchenScreen full-screen, không sidebar
- 📋 **Trực đơn standalone App**: header cam, OrderScreen full-screen, không sidebar
- 🚨 **Hero "Đơn cần phân shipper"** ở màn Giao hàng (trưởng ca/admin): card cam pulse, quick-assign avatar chip (xanh = sẵn sàng, vàng = đang giao đơn khác)
- 🔔 **Customer hybrid duyệt 1-tap**:
  - Status mới `cho_xac_nhan` cho đơn khách tự đặt qua `?customer=1` hoặc `?qr=<bàn>`
  - Đơn vào trạng thái này **không trừ kho** (tránh lãng phí cho đơn bị từ chối)
  - OrderScreen có panel vàng "🔔 X đơn khách chờ duyệt" ở đầu danh sách
  - Tap **"✓ Duyệt · KPI về tôi"** → trừ kho + gán KPI=user hiện tại + chuyển sang `moi`
  - Tap **"✕ Từ chối"** → prompt lý do → set `huy` với `returnReason`
  - Tự động check đủ kho trước khi duyệt; thiếu → toast cảnh báo
  - Push notif cho trực đơn/trưởng ca/admin khi có đơn vào pool
  - Progress bar cho customer view có thêm bước "cho_xac_nhan"
- 📊 **Shift grid view (Ma trận) — cải tiến lớn**:
  - Click ô → cycle qua role tiếp theo (không cần mở dropdown)
  - Chuột phải → đặt nhanh role "Nghỉ"
  - Nút **📋 Copy ←** ở header mỗi cột → sao chép toàn bộ phân công từ ca trước (xoá ca hiện tại rồi paste)
  - Hint banner cam ở dưới giải thích thao tác

### Changed
- ST const: thêm status `cho_xac_nhan` (vàng amber) ở vị trí đầu chuỗi trạng thái
- App() có 3 early-return cho 3 role: shipper / pha_che / truc_don. Admin / Trưởng ca giữ shell hiện tại (sidebar + topbar + nav 7 màn)
- Notification: thêm event "đơn khách chờ duyệt" cho role truc_don/truong_ca/admin

### Notes
- Demo mô hình **Hybrid có duyệt** cho customer order — phù hợp scale CLB sinh viên
- Pha chế chưa có step-by-step checklist + quản lý nguyên liệu (giữ cho v2.0 theo yêu cầu)

---

## [1.5.0-rc3] — 2026-05-26 (preview)

🎨 **Pha chế + Trực đơn standalone apps · Assigner UI cải tiến**

### Added
- 🛠️ **RoleAppShell** — vỏ standalone tối giản (header + logout + status, không sidebar/topbar admin) dùng chung cho các role không phải admin/trưởng ca
- ☕ **Pha chế giao diện riêng** (`effectiveRole==='pha_che'`): header xanh đậm + KitchenScreen full-screen, không còn sidebar trái
- 📋 **Trực đơn giao diện riêng** (`effectiveRole==='truc_don'`): header cam đậm + OrderScreen full-screen
- 🚨 **Hero "Đơn cần phân shipper"** ở màn Giao hàng của trưởng ca/admin:
  - Card cam nổi bật ở đầu màn, badge pulse, hiển thị từng đơn `san_sang & sid IS NULL`
  - Quick-assign chip avatar: xanh = sẵn sàng, vàng = đang giao đơn khác
  - Cảnh báo đỏ nếu ca không có shipper nào (kiểm tra Lịch ca)

### Notes
- Còn cho v1.5.0 final: customer hybrid duyệt 1-tap + shift grid view

---

## [1.5.0-rc2] — 2026-05-26 (preview)

🛵 **Shipper App standalone + Self-claim pool**

### Changed
- 🏠 **Shipper giờ có app riêng full-screen** — KHÔNG còn dùng sidebar/topbar của admin. App() early-return `<ShipperApp/>` khi `effectiveRole==='shipper'`:
  - Header đen riêng có nút **Thoát**, connection status (Online / Mất kết nối)
  - Earnings bar đen sang, 3 metric (Đã giao / Hoàn hàng / Tỉ lệ %)
  - 3 tab: **Đơn chờ nhận** (pool) · **Đang giao** (active) · **Lịch sử**
  - Badge đỏ pulse khi pool có đơn mới
- 🆕 **Tab "Đơn chờ nhận" — Self-claim pool**:
  - Tự động lọc đơn `st=san_sang` và `sid IS NULL`
  - Card vàng có badge "MỚI", thông tin khách + địa chỉ + phí ship
  - Nút **"🛵 Nhận đơn này"** (xanh, full-width) — concurrency-safe (UPDATE WHERE sid IS NULL); nếu đơn đã bị shipper khác lấy → toast cảnh báo, không update state
  - Trưởng ca vẫn có quyền gán thủ công (view assigner giữ nguyên)
- 🔔 **Notification mới cho shipper**: khi có đơn vào pool, shipper online sẽ nhận thông báo "🆕 Đơn mới có thể nhận: ĐHxxx"

### Notes
- Vẫn là rc, còn 2 task cho v1.5.0 final: customer hybrid + shift grid
- Pha chế + Trực đơn vẫn dùng shell cũ (sẽ standalone trong release sau)

---

## [1.5.0-rc1] — 2026-05-26 (preview)

🛵 **Shipper UI rebuild — phong cách Grab Driver / ShopeeFood Driver**

### Changed
- 🛵 **ShipperScreen view khi `effectiveRole==='shipper'`** rebuild hoàn toàn theo focus 1 đơn/lần:
  - **Earnings bar** sticky trên cùng: avatar + tên + status "đang nhận đơn", phí ship hôm nay (highlight vàng), 3 metric chip (đã giao / hoàn hàng / tỉ lệ %)
  - **Tab switcher** "Đơn đang chạy" / "Lịch sử hôm nay" với badge số đơn
  - **Active order card** to, đầy đủ thông tin tách block:
    - Status header với icon + tiêu đề + hướng dẫn ngắn
    - Customer block với nút phone hình tròn xanh **tap-to-call** (`tel:` link)
    - Address block với icon map-pin + nút **"Mở Google Maps"** mở tab mới (`maps.google.com/?api=1&query=`)
    - Note block highlight vàng nếu có ghi chú khách
    - Items list + tổng phí ship (gradient cam) + số tiền khách phải trả
    - CTA button khổng lồ ở cuối (~50px), gradient theo trạng thái (tím→cam→xanh)
  - **Queue list** các đơn khác đang chạy: card compact, click để focus
  - **History tab**: 1 row mỗi đơn với icon trạng thái tròn, phí ship, giờ giao, lý do hoàn hàng (nếu có)
- 🔄 Sort active orders theo priority `dang_giao → da_lay → san_sang` rồi đến số đơn

### Why
- Shipper dùng điện thoại 1 tay, cần thông tin to + CTA dễ tap khi đang lái xe
- Grid view cũ dày, khó scan; giờ luôn có 1 đơn "đang focus" rõ ràng
- Tích hợp `tel:` + Maps deep-link giảm thao tác

### Notes
- ⚠️ Bản **rc1** chưa release final. Còn 2 task nữa cho v1.5.0:
  - Customer hybrid: `cho_xac_nhan` status + duyệt 1-tap ở OrderScreen
  - Shift grid view + click cycle role
- View Trưởng ca/Admin của ShipperScreen giữ nguyên (chỉ shipper view đổi)

---

## [1.4.0] — 2026-05-26

🔧 **Sửa loạt bug realtime + UX feedback từ owner**

### Added
- 🛍️ **Trang khách hàng tự đặt đơn** (`?customer=1` hoặc `?qr=<table_id>`) — không cần PIN, mobile-first
  - Menu lọc theo category với chip filter sticky
  - Floating cart bar dưới cùng với badge số món + tổng
  - Bottom sheet xác nhận đơn: tên, SĐT, loại đơn, địa chỉ + chọn zone phí ship, ghi chú
  - Sau khi đặt: màn status với progress bar 5 bước (Mới → Pha → Sẵn sàng → Đang giao → Hoàn thành), tự cập nhật realtime qua channel riêng `customer-order-<num>`
  - Lưu order vào localStorage để khách quay lại xem trạng thái
- 🟢 **Online presence trong Sidebar** — dùng Supabase Realtime Presence track ai đang login, hiển thị số người online + chip tên người gần nhất, dedupe theo `staff_id` (1 user mở nhiều tab vẫn đếm là 1)
- 🎯 **KPI combobox autocomplete** — thay `<select>` thuần bằng searchable picker với:
  - Search box (focus tự động khi mở)
  - Sort người trong ca lên trước, người ngoài ca xuống dưới
  - Chip cảnh báo đỏ "ngoài ca" khi chọn người không phân công ca hiện tại
  - Click outside để đóng dropdown

### Fixed
- 🔧 **Realtime UPDATE/DELETE thiếu `p.old` đầy đủ** — thêm `REPLICA IDENTITY FULL` cho `orders`, `products`, `staff`, `shift_assignments` trong `supabase-schema.sql`. Trước đây notify shipper khi `sid` đổi bị silent fail vì `p.old.sid` luôn null
- 🔧 **Huỷ đơn không hoàn kho** — sửa 2 chỗ `cancel()` ở OrderScreen + ShipperScreen: cộng lại stock cho từng product trong order. Có guard chống hoàn 2 lần (check `st` chưa thuộc `huy/hoan_thanh/hoan_hang`)
- 🔧 **Dashboard sub** — phân biệt rõ "X thành viên phân ca" vs "🟢 N đang online"

### Notes
- ⚠️ **Cần chạy lại `supabase-schema.sql`** trên Supabase Dashboard sau khi pull để áp `REPLICA IDENTITY FULL` (idempotent — chạy lại an toàn)
- ⚠️ Customer page hiện cho phép anon insert orders qua RLS `open_all`. Khi public thật cần thêm rate-limit (Edge Function) hoặc captcha

---

## [1.3.1] — 2026-05-17

🎨 **UI/UX overhaul tiếp theo (Nhóm C + D): KDS, Shipper, Dashboard, Login, Sidebar, TopBar + OrderScreen single-scroll**

### Changed
- 🛒 **OrderScreen restructure** — toàn bộ panel trái (form khách + sản phẩm + giỏ hàng) gộp vào **một vùng scroll duy nhất**, không còn 3 vùng overflow lồng nhau với `maxHeight:42vh` (trước đây dễ che bớt nội dung). Search/cats bar `position:sticky` ở đầu, header giỏ hàng `position:sticky`, footer "Tạo đơn" `flexShrink:0` luôn ở đáy panel
- 🛒 **Footer CTA** thêm dòng tóm tắt **"{N} món · {Tại chỗ/Ship} · Tổng {amount}"** phía trên button, button gọn hơn ("Tạo đơn ngay") để tránh trùng thông tin
- 📊 **Dashboard** stat cards có gradient accent ở góc trên-phải + icon shadow, số to (24px) + letter-spacing âm; bar chart highlight giờ hiện tại với gradient cam + shadow; status bars cao hơn (6px); table có header background xám, row hover, empty state
- 🍳 **KitchenScreen** — column header có icon + count chip màu theo status + sticky với backdrop-blur; thẻ đơn có 2 mức urgent (>10p = vàng "SẮP TRỄ", >15p = đỏ "TRỄ" + shake animation); badge thời gian có icon clock; CT button to hơn (pill); button advance dùng gradient theo cột
- 🚚 **ShipperScreen** — `ShipperOrderCard` có border-left màu theo status, layout block hơn (customer info + address có icon Tabler), payment chip; action buttons (Đã lấy/Bắt đầu giao/Hoàn thành) chuyển sang `btn-pri-grad` hoặc gradient xanh consistent
- 🔐 **LoginScreen** — background gradient 3-stop + 2 radial blobs cam (depth), card glass effect (backdrop-blur + translucent), logo 68px với gradient + sh-or, PIN dots có animation `pop` khi nhập, icon error có `ti-alert-circle`, numpad button hover state đổi sang orange tint với border
- 🧭 **Sidebar** — header có gradient nền + logo gradient với shadow cam, active nav item có pill accent bên trái + shadow + smooth hover, badge dùng gradient cam thay vì đỏ flat, logout button dùng `.btn-ghost`
- ⬆️ **TopBar** — clock chip có viền + background nhạt, connection status pill có ring glow ngoài, notification button to hơn (36px) với border màu khi active, profile chip có shadow

### Fixed
- ⚠️ **OrderScreen "che mất tính năng"** — trước đây customer info bị giới hạn 42vh có thể che phí ship/preset address khi scroll; cart panel 42vh có thể che dòng giảm giá. Giờ scroll thoáng toàn panel, button "Tạo đơn" vẫn sticky dưới cùng
- ⚠️ **KDS mobile** — 3 cột kanban trước đây bị squashed trên mobile; giờ scroll ngang với `min-width:280px` mỗi cột
- ⚠️ **OrderScreen mobile** — `.order-right` cap 46vh → 50vh để hiển thị thêm 1-2 đơn xử lý
- ⚠️ **Mobile menu button** — trước trông như link text, giờ có nền `var(--orl)` + radius rõ ràng (tap target tốt hơn)

---

## [1.3.0] — 2026-05-17

🎨 **UI/UX overhaul (Shopee-inspired): design tokens + OrderScreen polish**

### Added
- 🎨 **Design token system** — mở rộng `:root` với semantic colors (`--ok`, `--warn`, `--err`, `--info`), text scale (`--text2`, `--mut2`), radius scale (`--r-sm/md/lg/pill`), shadow scale (`--sh-sm/md/lg/or`), gradient brand (`--or-grad`), motion easing (`--ease`)
- ✨ **Hover/focus polish** — focus ring 3px orange (12% alpha) trên mọi input/select/textarea; `button:active` micro press-down
- 🎴 **Card hover lift** — class `.card-hover` cho subtle elevation khi rê chuột
- 🏷️ **Chip primitives** — class `.chip`, `.chip-or` để badge nhất quán
- 💫 **Animation library** — `shake`, `pop`, `shimmer` (skeleton); `fadeUp`/`fadeIn` upgraded với cubic-bezier
- 📐 **Text clamp utils** — `.clamp-1`, `.clamp-2` cho tên sản phẩm dài
- 🔘 **Button variants** — `.btn-pri-grad` (gradient CTA), `.btn-ghost` (secondary outline)

### Changed
- 🛒 **OrderScreen — Product card** Shopee-style: emoji to (38px) trong khung, tên 2-line clamp, giá nổi bật, stock badge góc trái, qty bubble góc phải khi đã chọn, hover lift + shadow cam khi active
- 🛒 **Cart panel header** mới với icon + count chip + nút "Xoá tất cả"; empty state có icon + hint
- 🛒 **Cart row** chuyển sang separator dashed, qty stepper to hơn (28→34px), nút xoá hover đỏ với icon `ti-trash`
- 🛒 **Search bar** có icon `ti-search` bên trái + nút clear (✕) bên phải khi có nội dung
- 🛒 **Category tabs** thêm shadow cam khi active, padding rộng hơn cho tap target mobile
- 🛒 **Footer CTA** gradient button với icon + đếm số món + tổng tiền in-line
- 📋 **Active orders panel** — border-left màu theo status, header sticky, customer info có icon, items list trong khung nền nhạt, note có background cam nhạt
- 🎨 **Bdg / TPill / StBdg** đồng bộ pill style (radius-pill, weight 700, kích thước tap-friendly)
- 🎚️ **Qty stepper** màu cam (thay vì xám), có border phân chia giữa số và nút +/−
- 🍞 **Toast** thêm border-left 4px (visual hierarchy), close button dạng pill nền translucent, shadow lớn hơn
- 🖱️ **Scrollbar** rộng hơn (8px), border trong suốt cho cảm giác "floating" — đỡ chiếm chỗ visual

### Fixed
- ⚠️ Focus state input trước đây chỉ đổi border, dễ miss → giờ có ring 3px rõ
- ⚠️ Stock badge `Còn N` ở stock thấp/cao trông giống nhau → giờ có dấu `●` + màu khác biệt
- ⚠️ Cart "xoá" trước đây dùng `✕` text bé, dễ nhầm → giờ là icon trash với hover đỏ
- ⚠️ "Loại đơn" buttons không có tap target rõ ràng → padding tăng + emoji size lớn hơn

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
