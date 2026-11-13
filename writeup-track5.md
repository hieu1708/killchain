# Writeup Track 5 — Bài 2 và Bài 3


## Bài 2 — BOSS: Leo thang đặc quyền

### Mục tiêu

Tìm cấu hình sai cho phép chạy lệnh với quyền `root`, sau đó đọc nội dung file `/root/flag` trong môi trường bài tập.

![Đề bài 2 — Leo thang đặc quyền](writeup-track5-images/bai2-de-bai.png)

### Bước 1: Khảo sát chức năng chạy lệnh

Đề bài cung cấp đường dẫn chạy lệnh:

```text
/app/shell?cmd=<lệnh>
```

![Giao diện chạy lệnh với tài khoản www-data](writeup-track5-images/bai2-shell.png)

Truy cập đường dẫn này cho thấy tài khoản hiện tại là `www-data`. Tiếp tục kiểm tra bằng các lệnh được đề bài gợi ý:

```bash
whoami
sudo -l
```

### Bước 2: Phát hiện cấu hình sudo không an toàn

Kết quả của `sudo -l` có dòng:

```text
(root) NOPASSWD: /usr/bin/python3
```

![Kết quả sudo -l](writeup-track5-images/bai2-sudo.png)

Cấu hình này cho phép `www-data` chạy `/usr/bin/python3` với quyền `root` mà không cần mật khẩu. Vì Python có thể thực thi mã và đọc file, quyền này cho phép tài khoản hiện tại thực hiện các thao tác vượt quá quyền thông thường của nó.

### Bước 3: Đọc flag

Tài liệu gốc ghi nhận lệnh sau và ảnh chụp kết quả trả về flag:

```bash
sudo python3 cat /root/flag
```

**Lưu ý:** thực ra câu lệnh trên chưa đúng cú pháp nhưng lab vẫn xử lý lệnh

Flag hiển thị trong ảnh:

```text
FLAG{m10-926822c17962}
```

![Flag bài 2 trong tài liệu gốc](writeup-track5-images/bai2-flag.png)

### Nguyên nhân

Lỗi nằm ở việc cấp quyền sudo quá rộng cho tài khoản chạy ứng dụng web: cho phép chạy một trình thông dịch đa dụng bằng quyền `root` mà không cần mật khẩu.

## Bài 3 

![Đề bài 3 — Két sắt APT](writeup-track5-images/bai3-de-bai.png)

### Bước 1: Kiểm tra mã nguồn phía trình duyệt

Mở trang `/app/boss`. Giao diện hiển thị két AcmeVault ở trạng thái đang tải.

![Giao diện AcmeVault](writeup-track5-images/bai3-giao-dien.png)

Mở Developer Tools bằng `F12`, vào tab **Sources** và xem mã JavaScript trong `bundle`.

![Mã JavaScript trong bundle](writeup-track5-images/bai3-bundle.png)

Trong mã nguồn có đường dẫn lấy payload:

```javascript
const VAULT_GATE = '/app/boss/gate';
```

![Chú thích mô tả quy trình giải mã](writeup-track5-images/bai3-quy-trinh.png)

Các chú thích cũng mô tả quy trình mã hóa:

```text
Dữ liệu gốc → đảo ngược byte → Base64 → XOR với khóa lặp → hex
```

Do đó, quy trình giải mã là:

```text
hex → chuyển thành byte → XOR với khóa lặp → giải mã Base64 → đảo ngược byte
```

### Bước 2: Lấy payload đã mã hóa

![Payload dạng hex tại gate](writeup-track5-images/bai3-payload.png)

Truy cập `/app/boss/gate`, thu được payload dạng hex:

```text
536c57492872615d7d6664573d0b2f526c510403286206026201772936065805
```

### Bước 3: Tìm KEY

Trong `bundle` còn sót lại dữ liệu kiểm thử riêng cho tầng XOR:

```text
pt = ACME-VAULT-SELFTEST-OK-2026
ct = 747b7976486376607c651c37212d236c706b601e2a7e1a07000307
```

![Cặp bản rõ và bản mã trong self-test](writeup-track5-images/bai3-self-test.png)

Ta thấy pt XOR KEY = ct
-> KEY = pt XOR ct (tính chất tự nghịch đảo của xor)

![Khôi phục khóa bằng XOR](writeup-track5-images/bai3-khoi-phuc-khoa.png)

```text
Đem pt XOR ct thu được 5843e575011ddae85843e575011
```

Đoạn sau bị lặp rồi nên ta lấy phần đầu thôi

```text
KEY: 5843e575011ddae8
```

### Bước 4: XOR payload với KEY


![XOR payload với khóa lặp](writeup-track5-images/bai3-xor-payload.png)

Kết quả là chuỗi Base64 đã bị mã hóa như để nói:

```text
fTczMGVhMWU3YjJjYi00MW17R0FMRg==
```


Đem giải mã Base64 thu được:


![Kết quả giải mã Base64](writeup-track5-images/bai3-base64.png)

```text
}730ea1e7b2cb-41m{GALF
```

Đảo ngược thứ tự:

```text
FLAG{m14-bc2b7e1ae037}
```

![Flag bài 3 sau khi đảo ngược byte](writeup-track5-images/bai3-flag.png)


