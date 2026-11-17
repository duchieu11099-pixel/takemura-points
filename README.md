# Trang tích điểm 中華そば 竹むら — Hướng dẫn đưa lên mạng

Trang này gồm đúng 1 file `index.html`. Hiện tại trang chạy ở **chế độ demo**
(chưa liên kết LINE và Firebase): khách mở trang là xem điểm, quét QR cộng điểm,
đổi thưởng được ngay — dữ liệu tạm lưu trên chính điện thoại của người mở
(localStorage), mỗi máy một bộ dữ liệu riêng. Làm đủ 3 bước dưới đây thì
trang mới dùng chung dữ liệu thật cho mọi khách.

## Cách xem thử ngay

1. Mở file `index.html` bằng trình duyệt (trên điện thoại hoặc máy tính đều được).
2. Trang sẽ hiện ở chế độ demo: số điểm 0, mã hội viên dạng `TM-XXXXXX` kèm QR.
3. Thử cộng điểm: mở địa chỉ của trang kèm `?earn=takemura-visit` ở cuối
   (ví dụ `index.html?earn=takemura-visit`) — sẽ được cộng điểm 1 lần/ngày.
4. Thử đổi thưởng: sang tab 特典 (Đặc典), đủ điểm thì bấm 使う sẽ ra mã đổi thưởng 6 ký tự.

## Bước 1 — Firebase (lưu dữ liệu điểm)

1. Vào [console.firebase.google.com](https://console.firebase.google.com), đăng nhập Google, bấm tạo project mới (đặt tên tùy ý, ví dụ `takemura-points`).
2. Trong project, vào mục **Firestore Database** → **Create database** → chọn chế độ mặc định (production) → tạo.
3. Vào **Project settings** (biểu tượng bánh răng) → mục **Your apps** → chọn Web (`</>`) → đăng ký app → Firebase sẽ hiện một đoạn `firebaseConfig`.
4. Copy nguyên đoạn `firebaseConfig` đó, mở file `index.html`, tìm chữ `CONFIG` ở đầu phần script và dán đè vào dòng `firebaseConfig: null`.
5. Cũng trong Firestore, vào tab **Rules** và dán bộ rules giai đoạn 1 này, rồi bấm Publish:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId} {
      allow read, write: if true;
    }
    match /transactions/{docId} {
      allow read, write: if true;
    }
    match /settings/{docId} {
      allow read, write: if true;
    }
    match /rewards/{docId} {
      allow read, write: if true;
    }
  }
}
```

> ⚠️ **Cảnh báo:** bản đầu này chưa có máy chủ kiểm chứng danh tính LINE,
> nên rules đang để mở (`allow read, write: if true`) — ai biết địa chỉ
> project cũng có thể đọc/sửa dữ liệu điểm. Rủi ro được coi là thấp vì dữ liệu
> chỉ là điểm thưởng của quán, không có thông tin thanh toán hay cá nhân nhạy cảm.
> Sau này muốn chặt hơn (chống gian lận điểm nghiêm túc) thì cần thêm một
> backend nhỏ để kiểm chứng — cứ báo Muse khi cần.

Mẹo: muốn đổi token QR cộng điểm trên máy chủ thay vì trong file, tạo trong
Firestore một document `settings/shop` với field `earnToken` — trang sẽ ưu tiên
dùng giá trị đó. Tương tự, nếu tạo collection `rewards` (mỗi document gồm
`name` và `cost`) thì trang sẽ dùng danh sách đó thay cho danh sách trong CONFIG.

## Bước 2 — LINE (đăng nhập 1 chạm)

1. Tạo tài khoản **LINE Official Account** cho quán tại [business.line.me](https://business.line.me) (hoặc app LINE Official Account).
2. Vào [developers.line.biz](https://developers.line.biz) → tạo Provider → trong Provider đó tạo channel. LIFF nằm trong channel loại **Messaging API** (gắn với Official Account ở trên); nếu chỉ cần đăng nhập thì channel **LINE Login** cũng tạo được LIFF.
3. Trong channel, vào tab **LIFF** → **Add** → đặt tên, chọn size (Full), và điền **Endpoint URL** là địa chỉ GitHub Pages của trang (làm xong Bước 3 sẽ có địa chỉ này, quay lại điền sau cũng được).
4. Tạo xong sẽ có **LIFF ID** — copy, mở `index.html`, dán vào dòng `LIFF_ID: ""` trong CONFIG.
5. Từ lúc đó, khách mở trang từ trong LINE sẽ tự đăng nhập bằng tài khoản LINE của họ (hiện nút LINEでログイン màu xanh khi cần), điểm gắn đúng với từng khách.

## Bước 3 — GitHub Pages (đưa trang lên mạng)

1. Vào github.com, tạo **1 repo trống mới** (đặt tên ví dụ `takemura-points`, để Public).
2. Báo Muse tên repo — Muse sẽ đẩy file `index.html` lên và hướng dẫn bật Pages.
3. Bật Pages: trong repo vào **Settings → Pages** → Source chọn **Deploy from a branch** → branch `main`, thư mục `/ (root)` → Save.
4. Chờ khoảng 1 phút, địa chỉ trang sẽ là `https://<tên-github>.github.io/takemura-points/`.
5. Lấy địa chỉ này quay lại điền **Endpoint URL** cho LIFF ở Bước 2.

## Bước 4 — QR tại quán

1. In một mã QR trỏ tới địa chỉ: `https://<địa-chỉ-pages>/?earn=takemura-visit`
   (dán địa chỉ này vào bất kỳ trang tạo QR nào, in ra để ở quầy thu ngân/bàn ăn).
2. Khách quét QR này bằng camera/LINE → trang mở ra, tự cộng điểm (mỗi khách 1 lần/ngày), hiện thông báo ポイントを獲得しました！.
3. Muốn đổi token (tránh khách chụp ảnh QR mang về nhà quét), sửa `SHOP_EARN_TOKEN` trong CONFIG của `index.html` (hoặc field `earnToken` trong `settings/shop` trên Firestore) rồi in lại QR mới.

## Về tỉ lệ điểm và giá thưởng — đang là SỐ TẠM

Trong CONFIG ở đầu file `index.html` đang để tạm:

- `POINTS_PER_VISIT: 1` — 1 lần đến quán = 1 điểm.
- Giá các phần thưởng trong `REWARDS` (anh chốt lại 17/11/2026 15:40): topping tự chọn 3pt、丼もの 1品 free 5pt、中華そば hoặc 鳥家系ラーメン free 8pt, bát mì bất kỳ free 10pt. Ngoài ra có quà đăng nhập lần đầu (`SIGNUP_BONUS_NAME`): topping free hoặc 大盛り free, mỗi người 1 lần, không trừ điểm.

Đây chỉ là số ví dụ. Khi chủ quán chốt tỉ lệ thật, chỉ cần sửa đúng khối CONFIG này (1 chỗ duy nhất) là toàn trang cập nhật theo — hoặc tạo collection `rewards` trên Firestore như hướng dẫn ở Bước 1.
