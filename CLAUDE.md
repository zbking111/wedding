# Trang cưới Trường & Gấm (27/12/2026)

Trang thiệp cưới online làm cho bạn của chủ repo. Trang tĩnh (HTML/CSS/JS thuần, không framework, không bước build).
Nội dung tách khỏi code: bạn cô dâu/chú rể tự sửa qua Pages CMS, không cần đụng code.

**Code (giao diện + JS) KHÔNG nằm ở repo này** mà ở repo dùng chung `zbking111/wedding-engine` (thư mục local `~/Downloads/wedding-engine`,
phục vụ tại https://wedding-engine.hpwd.workers.dev/v1/). Repo này chỉ có vỏ HTML + nội dung. Sửa giao diện/tính năng → sửa engine
(áp dụng cho mọi cặp đôi, xem CLAUDE.md của engine).

## Kiến trúc & luồng deploy

Pages CMS (sửa nội dung) ─┐
git push (sửa code) ──────┴─> GitHub `zbking111/wedding` (nhánh `main`) ─> Cloudflare Worker tự deploy (~30s–2 phút)

- Repo: https://github.com/zbking111/wedding (**public**), nhánh `main`. Thư mục local: `~/Downloads/wedding-site`.
- Hosting: Cloudflare **Worker** (static assets, nối Git) tên `gamtruong27122026wedding`, tài khoản Cloudflare nguyenzuanbka@…
  - Link: https://gamtruong27122026wedding.hpwd.workers.dev/ (subdomain tài khoản đã đổi thành `hpwd`; không bỏ được phần này).
  - Không có build: phục vụ thẳng file ở gốc repo. Kiểm tra deploy ở Workers & Pages → project → Versions.
  - Muốn link gọn: mua tên miền rồi Settings → Domains & Routes → Add custom domain.
- Dự án Pages cũ (upload tay) `gamtruong27122026wedding.pages.dev` đã bỏ, không dùng nữa.
- CMS: Pages CMS (https://app.pagescms.org), đăng nhập GitHub, cấu hình trong `.pages.yml`.
  Mời người sửa nội dung bằng email ở mục Collaborators (họ không cần tài khoản GitHub).
  Pages CMS tự commit "(via Pages CMS)" lên `main` → **luôn `git pull --rebase` trước khi push**.
- Upload qua Pages CMS giới hạn thực tế ~3,3 MB/file (CMS gửi file dạng base64 qua server của họ, giới hạn ~4,5 MB/request; file lớn hơn báo lỗi 4xx). File lớn (mp3, ảnh nặng) thì nén rồi commit thẳng bằng git vào `media/images/`.

## Cấu trúc file

- `index.html` — vỏ ~20 dòng: thẻ meta + tải `head.js`, `app.css`, `app.js` từ engine v1. `app.js` dựng trang rồi `fetch("content/site.json")`.
  Chỉ sửa file này khi đổi thẻ meta/og hoặc đổi phiên bản engine (v1 → v2).
- `content/site.json` — dữ liệu trang (CMS sửa file này).
- `.pages.yml` — schema form của Pages CMS. Thêm trường mới vào JSON thì thêm field ở đây; JS phải chịu được trường thiếu (dữ liệu cũ không có).
- `media/images/` — ảnh và nhạc do CMS tải lên (đường dẫn công khai `/media/images/...`). `ngay-cuoi.mp3` = nhạc nền đang chọn (đã nén 96 kbps).
- `tao-link.html` — vỏ trang tạo link thiệp mời theo tên khách (code ở engine: `tao-link.css`, `tao-link.js`).
- `content/site.ja.json` — bản dịch tiếng Nhật của các chữ trong site.json (KHÔNG sửa qua CMS; nhờ Claude dịch lại khi site.json đổi nội dung).
  Chỉ chứa trường chữ cần dịch; được ghép lên site.json (mảng ghép theo vị trí, thiếu/rỗng → giữ tiếng Việt).
  Không đưa dữ liệu thật (số tài khoản, ngân hàng, địa chỉ, ngày giờ, ảnh) vào đây, để luôn lấy từ site.json.

## Tiếng Nhật (`?lang=ja`, hoặc `lang=jp`)

- Cùng một trang cho cả hai ngôn ngữ (xử lý trong engine). Không có `lang=ja` → trang tiếng Việt, không tải thêm gì.
- Chữ cố định của giao diện được dịch trong engine; nội dung riêng của cặp đôi được dịch trong `content/site.ja.json`.
- Ngày kiểu 2026年12月27日（日）, ẩn âm lịch, tên khách tự thêm 様. Font Noto Sans/Serif JP chỉ tải khi lang=ja.
- Giá trị gửi lên Sheet (nhà trai/nhà gái, có/không tham dự) vẫn là tiếng Việt (option có `value` tiếng Việt).
- Thiệp chọn sự kiện bằng tên sự kiện tiếng Việt gốc (SVI), không dùng bản dịch.
- tao-link.html có ô "Khách là người" (mặc định Người Việt; chọn Người Nhật) → thêm `&lang=ja` vào link + tin nhắn mẫu tiếng Nhật.

## Các trường chính trong site.json

groomName, brideName, weddingDate (YYYY-MM-DD), weddingTime ("11:00"), lunarText, heroPhoto, music (trống → nhạc mặc định của engine `v1/ngay-cuoi.mp3`),
youtubeId, videoQuote, albumQuote, letter (đoạn cách nhau bằng dòng trống), letterPhoto, photos[], storyQuote, stories[{date,title,text,photo}],
events[{title,date,time,place,addr,map,side(trai|gai),invite(bool),lunar,colors[]}], groom/bride{photo,bio,father,mother},
bridesmaids[]/groomsmen[]{name,photo,intro}, gifts[{title,bank,no,name,qr}], wishSuggestions[], apiUrl,
sections{video,album,calendar,story,letter,events,couple,party,gifts,wishes,rsvp,hearts,invite} (false = ẩn; thiếu = hiện).

## Tính năng trên trang

Thiệp mời mở đầu theo tên khách (`?to=<tên>&s=trai|gai`; nút "Mở thiệp" đồng thời bật nhạc), hero + đếm ngược, 3 nút nhanh,
video YouTube, album + trình xem ảnh toàn màn hình (vuốt, zoom, tự chạy, dải thumbnail), lịch tháng, chuyện tình yêu (timeline),
lời ngỏ + ảnh, sự kiện (Google Calendar, chỉ đường, dress code), cô dâu chú rể + tên cha mẹ, phù dâu phù rể, hộp mừng cưới (mục + popup),
sổ lưu bút (gợi ý lời chúc, lời chúc nổi xoay vòng), xác nhận tham dự (modal), menu nổi, nhạc nền, hiệu ứng thả tim, ẩn/hiện từng mục.
Mục rỗng (album, chuyện tình yêu, phù dâu phù rể, sự kiện) tự ẩn.

## Backend lời chúc / xác nhận: Google Sheets + Apps Script

- Web App URL nằm ở `apiUrl` trong site.json (…/macros/s/AKfycbxHma0p…/exec), triển khai "Thực thi: Tôi", "Truy cập: Bất kỳ ai".
- `GET ?action=wishes` → `{ok, wishes:[{name,message}]}`; `POST` body JSON (Content-Type text/plain để tránh CORS preflight):
  `{type:"wish",name,message}` → tab `LoiChuc`; `{type:"rsvp",name,phone,side,count,attend,note}` → tab `XacNhan`.
- Ẩn một lời chúc: gõ `x` vào cột `An` trong Sheet.
- Sửa code Apps Script: Triển khai → Quản lý các bản triển khai → bút chì → Phiên bản mới (KHÔNG tạo "triển khai mới", vì sẽ đổi URL).

## Quy ước

- Sửa giao diện/tính năng: làm ở `~/Downloads/wedding-engine` (quy tắc trong CLAUDE.md của engine), không thêm code vào repo này.
- Thêm trường mới vào site.json (do engine hỗ trợ) → thêm field tương ứng vào `.pages.yml` ở đây.
- Thử trang với engine bản đang sửa trên máy: chạy engine ở cổng 8001, rồi chạy repo này ở 8000 với vỏ tạm trỏ tới
  `http://127.0.0.1:8001/v1/` (không commit vỏ tạm).
- Push: `git add … && git commit -m "…" && git pull --rebase && git push`. Xem kết quả sau ~1 phút, tải lại bằng Cmd+Shift+R.

## Việc còn mở

- Album chưa có ảnh (ảnh đang nằm ở heroPhoto) → thêm trong CMS mục "Album ảnh cưới".
- Thông tin thật còn để `[...]`: địa chỉ, tên cha mẹ, tài khoản ngân hàng, QR, giới thiệu.
- Có thể gắn tên miền riêng cho link gọn hơn.
