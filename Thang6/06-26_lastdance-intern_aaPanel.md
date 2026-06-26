# BÁO CÁO THỰC HÀNH
## Cài đặt và Cấu hình aaPanel trên Ubuntu Server 22.04 LTS

---

## MỤC LỤC

- [I. Giới thiệu về aaPanel](#i-giới-thiệu-về-aapanel)
- [II. Yêu cầu hệ thống](#ii-yêu-cầu-hệ-thống)
- [III. Chuẩn bị môi trường](#iii-chuẩn-bị-môi-trường)
- [IV. Cài đặt aaPanel](#iv-cài-đặt-aapanel)
- [V. Cấu hình Firewall và Truy cập Panel](#v-cấu-hình-firewall-và-truy-cập-panel)
- [VI. Cài đặt LNMP Stack](#vi-cài-đặt-lnmp-stack)
- [VII. Thêm Website và Cấu hình Nginx](#vii-thêm-website-và-cấu-hình-nginx)
- [VIII. Cấu hình SSL/TLS (HTTPS)](#viii-cấu-hình-ssltls-https)
- [IX. Quản lý Database](#ix-quản-lý-database)
- [X. Tăng cường Bảo mật Panel](#x-tăng-cường-bảo-mật-panel)
- [XI. Các lệnh quản lý qua CLI](#xi-các-lệnh-quản-lý-qua-cli)
- [XII. Xử lý sự cố thường gặp](#xii-xử-lý-sự-cố-thường-gặp)
- [XIII. Kết luận](#xiii-kết-luận)

---

## I. Giới thiệu về aaPanel

**aaPanel** (còn gọi là Bảo Tháp phiên bản quốc tế) là một control panel quản lý máy chủ web mã nguồn mở, miễn phí, được phát triển bởi aaPanel Technology Co., Ltd. Đây là phiên bản quốc tế của **BaoTa Panel (宝塔面板)** – một trong những control panel phổ biến nhất tại Trung Quốc.

aaPanel cung cấp giao diện web trực quan, cho phép quản trị viên hệ thống dễ dàng triển khai và quản lý các dịch vụ web mà không cần ghi nhớ nhiều câu lệnh phức tạp. Với aaPanel, người dùng có thể:

- Cài đặt nhanh LAMP hoặc LNMP stack chỉ với vài cú nhấp chuột.
- Quản lý nhiều website, domain, SSL certificate trên cùng một máy chủ.
- Tạo và quản lý database MySQL/MariaDB/MongoDB/Redis qua giao diện đồ họa.
- Quản lý tài khoản FTP, cron job, firewall và giám sát tài nguyên hệ thống theo thời gian thực.
- Cài đặt các ứng dụng phổ biến như phpMyAdmin, WordPress, Node.js, Docker, v.v.

---

## II. Yêu cầu hệ thống

Trước khi tiến hành cài đặt, cần đảm bảo máy chủ đáp ứng các yêu cầu tối thiểu sau:

| Thành phần | Yêu cầu tối thiểu |
|---|---|
| Hệ điều hành | Ubuntu 20.04 / **22.04 LTS** (khuyến nghị), Debian 10/11, CentOS 7/8 |
| RAM | Tối thiểu 512 MB (khuyến nghị 1 GB trở lên) |
| CPU | Tối thiểu 1 core (khuyến nghị 2 core trở lên) |
| Dung lượng ổ đĩa | Tối thiểu 5 GB trống (khuyến nghị 20 GB trở lên) |
| Kết nối mạng | Internet ổn định để tải các gói phần mềm |
| Quyền truy cập | Root hoặc sudo |
| Python | Python 3.x (có sẵn trên Ubuntu 22.04) |

> **Ghi chú:** Trong báo cáo này, môi trường thực hành là Ubuntu Server 22.04 LTS. Đảm bảo không có control panel nào khác (cPanel, Plesk...) đã được cài đặt trước đó.

---

## III. Chuẩn bị môi trường

### 3.1. Cập nhật hệ thống

Bước đầu tiên là cập nhật toàn bộ hệ thống để đảm bảo các gói phần mềm ở phiên bản mới nhất, tránh xung đột phiên bản và các lỗ hổng bảo mật đã biết:

```bash
# Đăng nhập với quyền root
sudo -i

# Cập nhật danh sách gói
apt update

# Nâng cấp tất cả các gói đã cài đặt
apt upgrade -y

# Xóa các gói không cần thiết
apt autoremove -y
```

### 3.2. Cấu hình hostname (tùy chọn)

Nên đặt hostname có ý nghĩa để dễ nhận biết khi quản lý nhiều máy chủ:

```bash
# Đặt hostname cho máy chủ
hostnamectl set-hostname aapanel-server

# Kiểm tra hostname
hostnamectl
```
<img width="1446" height="673" alt="image" src="https://github.com/user-attachments/assets/e93f66a0-1923-4e59-b604-03c3b2ce1f45" />

### 3.3. Kiểm tra trạng thái Firewall

Kiểm tra xem UFW (Uncomplicated Firewall) có đang chạy không:

```bash
# Kiểm tra trạng thái UFW
ufw status

# Nếu UFW chưa bật, cho phép SSH trước rồi mới bật
ufw allow 22/tcp
ufw enable
```


### 3.4. Cài đặt các gói phụ thuộc

Cài đặt một số gói tiện ích cơ bản cần thiết:

```bash
apt install -y wget curl vim net-tools
```

---

## IV. Cài đặt aaPanel

### Bước 1 – Tải xuống và chạy script cài đặt chính thức

aaPanel cung cấp script cài đặt tự động. Sử dụng lệnh sau dành riêng cho Ubuntu/Debian:

```bash
wget -O install.sh http://www.aapanel.com/script/install-ubuntu_6.0_en.sh \
  && sudo bash install.sh aapanel
```
<img width="1498" height="470" alt="image" src="https://github.com/user-attachments/assets/892b9bff-e38e-4382-9bbc-b2734553cc64" />

> **Ghi chú:** Script sẽ tự động phân tích hệ điều hành, cài đặt Python, Pip và toàn bộ các phụ thuộc. Quá trình thường mất từ 3 đến 8 phút tùy tốc độ Internet.

### Bước 2 – Theo dõi quá trình cài đặt

Trong quá trình cài đặt, script tự động thực hiện các tác vụ sau theo thứ tự:

1. Kiểm tra phiên bản hệ điều hành và kiến trúc CPU.
2. Cài đặt Python 3 và các module cần thiết (pip, setuptools, v.v.).
3. Tải xuống và cài đặt các file aaPanel vào `/www/server/panel`.
4. Cấu hình aaPanel chạy như một service hệ thống.
5. Thiết lập cron job để kiểm tra sức khỏe panel định kỳ.
6. Tạo tài khoản và đường dẫn đăng nhập ngẫu nhiên cho lần đầu.

### Bước 3 – Lưu thông tin đăng nhập

Sau khi cài đặt hoàn tất, terminal hiển thị thông tin đăng nhập. Ví dụ:

```
==================================================================
Congratulations! Installed successfully!
==================================================================
aaPanel Internet Address: http://203.xxx.xxx.xxx:8888/a1b2c3d4
aaPanel Internal Address: http://192.168.1.100:8888/a1b2c3d4
username: admin8x7k
password: Xk9mP2nQ
==================================================================
If you cannot access the panel,
release the following port (8888|888|80|443|20|21) in the security group
==================================================================
```

<img width="1045" height="606" alt="image" src="https://github.com/user-attachments/assets/60c80bbf-6eed-48c7-9e30-886636cdfc8b" />
```
SET_SSL: true
Stopping Bt-Tasks...    done
Stopping Bt-Panel...    done
Starting Bt-Panel... Bt-Panel (pid 59168) already running
Starting Bt-Tasks... Bt-Tasks (pid 59185) already running
==================================================================
Congratulations! Installed successfully!
==================================================================
aaPanel Internet IPv4 Address: https://14.248.82.194:35619/d5809538
aaPanel Internal Address: https://192.168.197.148:35619/d5809538
username: eftj1wmi
password: 443db4af
Warning:
If you cannot access the panel,
release the following port (35619|888|80|443|20|21) in the security group
==================================================================
Time consumed: 3 Minute!
```
> **Cảnh báo:** Sao chép và lưu lại ngay URL, username và password được hiển thị. Nếu bị mất, dùng lệnh `bt default` để xem lại.

Giao diện của aaPanel
<img width="1920" height="879" alt="image" src="https://github.com/user-attachments/assets/fa7fb2fe-5fbb-4f87-a687-2173f7334c35" />

---

## V. Cấu hình Firewall và Truy cập Panel

### 5.1. Mở các port cần thiết trên UFW

aaPanel cần một số port được mở để hoạt động đúng:

```bash
# Port truy cập aaPanel Web UI
ufw allow 8888/tcp

# Port Web HTTP và HTTPS
ufw allow 80/tcp
ufw allow 443/tcp

# Port FTP
ufw allow 20/tcp
ufw allow 21/tcp

# Port MySQL (nếu cần truy cập từ xa)
ufw allow 3306/tcp

# Áp dụng và kiểm tra
ufw reload
ufw status numbered
```
<img width="765" height="689" alt="image" src="https://github.com/user-attachments/assets/d57c7f29-ef7d-40ae-b9c3-16f07844ae34" />

### 5.2. Đăng nhập vào giao diện aaPanel

Mở trình duyệt web và truy cập URL được cung cấp sau khi cài đặt:

```
https://192.168.197.148:35619/d5809538

```

Nhập username và password để đăng nhập. Sau khi đăng nhập thành công lần đầu, aaPanel sẽ hiển thị màn hình chọn stack để cài đặt.

<img width="1916" height="960" alt="image" src="https://github.com/user-attachments/assets/19fb2b70-a590-406e-ad99-9eaeeb1de8f0" />

---

## VI. Cài đặt LNMP Stack

Sau khi đăng nhập lần đầu, aaPanel tự động hiển thị hộp thoại gợi ý cài đặt môi trường web.

### 6.1. So sánh LNMP và LAMP

| | LNMP (Khuyến nghị) | LAMP |
|---|---|---|
| Web server | Nginx | Apache |
| Hiệu năng | Cao hơn, nhẹ hơn | Thấp hơn |
| RAM tiêu thụ | Ít hơn | Nhiều hơn |
| Xử lý tải cao | Tốt hơn | Kém hơn |
| `.htaccess` | Không hỗ trợ native | Hỗ trợ tốt |
| Phù hợp với | WordPress, Laravel, ứng dụng mới | Ứng dụng cũ, shared hosting |

### 6.2. Cài đặt qua giao diện web

Thực hiện theo các bước sau trong giao diện aaPanel:

1. Trong hộp thoại xuất hiện sau đăng nhập, chọn **LNMP**.
2. Chọn phiên bản phần mềm:
   - **Nginx:** 1.24 (phiên bản ổn định mới nhất)
     <img width="1251" height="500" alt="image" src="https://github.com/user-attachments/assets/d671e5b2-4543-4e1b-bda8-61ccec1db4a3" />

   - **MySQL:** 5.7 hoặc 8.0
     <img width="1119" height="450" alt="image" src="https://github.com/user-attachments/assets/f933e5e2-6d03-4ff1-9a3f-f10152f2d5d4" />

   - **PHP:** 8.1 hoặc 8.2 (tùy ứng dụng)
   - **phpMyAdmin:** 5.x
3. Nhấn **One-click install** và chờ quá trình hoàn tất.
4. Quá trình cài đặt thường mất từ 10 đến 30 phút tùy cấu hình máy và tốc độ mạng.

Sau khi hoàn tất, truy cập Dashboard sẽ thấy các service Nginx, MySQL, PHP đang chạy (trạng thái xanh lá).

---

## VII. Thêm Website và Cấu hình Nginx

### 7.1. Tạo website mới

1. Vào menu **Website** trên thanh điều hướng bên trái.
2. Nhấn **Add site**.
3. Điền thông tin trong form:
   - **Domain:** `tdong.com`
   - **Root directory:** `/www/wwwroot/tdong.com` (mặc định)
   - **PHP Version:** chọn phiên bản PHP đã cài
   - **Database:** tùy chọn, có thể tạo kèm hoặc bỏ qua
   - **FTP:** bỏ qua trong môi trường lab
4. Nhấn **Submit** để tạo website.

### 7.2. Trỏ file hosts trên máy client

Do thực hành trên môi trường local (không có domain thật), cần chỉnh file `hosts` trên máy client để trình duyệt phân giải `tdong.com` về IP máy chủ `192.168.197.148`.

**Trên Windows:**

Mở Notepad với quyền Administrator, chỉnh sửa file:

```
C:\Windows\System32\drivers\etc\hosts
```

Thêm dòng sau vào cuối file:

```
192.168.197.148    tdong.com
```

Lưu file. Kiểm tra bằng lệnh:

```cmd
ping tdong.com
```

Kết quả trả về đúng IP `192.168.197.148` là thành công.



### 7.3. Upload source code

Sau khi tạo website, có thể upload source code theo các cách:

- **Qua File Manager trong aaPanel:** vào menu **Files**, điều hướng đến `/www/wwwroot/tdong.com` và upload.
- **Qua SSH/SCP:** dùng lệnh `scp` từ máy local.

```bash
# Upload qua SCP
scp -r /path/to/project root@192.168.197.148:/www/wwwroot/tdong.com/

# Phân quyền thư mục web
chown -R www:www /www/wwwroot/tdong.com
chmod -R 755 /www/wwwroot/tdong.com
```
<img width="1654" height="730" alt="image" src="https://github.com/user-attachments/assets/20cccb72-95d9-49fb-b7d5-7558ec46bc6e" />





## VIII. Cấu hình SSL/TLS (HTTPS)

Do thực hành trên môi trường local, không thể sử dụng Let's Encrypt (yêu cầu domain thật và IP public). Thay vào đó, sử dụng **Self-signed Certificate** (chứng chỉ tự ký).

> **Lưu ý:** Self-signed Certificate khiến trình duyệt hiển thị cảnh báo "Kết nối không an toàn" vì không được xác thực bởi CA (Certificate Authority) tin cậy. Đây là hành vi bình thường trong môi trường lab.

### 8.1. Cấu hình tự kí trên aaPanel 


<img width="1204" height="845" alt="image" src="https://github.com/user-attachments/assets/27583ead-2d37-46b2-9632-a124f57adc67" />

4. Nhấn **Save** để áp dụng.


### 8.4. Kiểm tra kết quả

Truy cập `https://tdong.com` trên trình duyệt. Trình duyệt sẽ hiển thị cảnh báo chứng chỉ không tin cậy — chọn **Advanced** > **Proceed to tdong.com** để tiếp tục.

Kiểm tra chứng chỉ đã được áp dụng đúng:

```bash
# Kiểm tra SSL từ máy chủ
openssl s_client -connect tdong.com:443 -servername tdong.com
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/50272953-2587-4b56-b68e-8f64c4ae82b8" />

---

## IX. Quản lý Database

### 9.1. Tạo database MySQL mới

1. Vào menu **Database** trên thanh điều hướng.
2. Nhấn **Add database**.
3. Điền thông tin:
   - **Database name:** tên database (vd: `myapp_db`)
   - **Username:** tên user (vd: `myapp_user`)
   - **Password:** mật khẩu mạnh (có thể dùng nút *Generate*)
   - **Access:** chọn *Local* (chỉ cho phép truy cập nội bộ) hoặc *Specified IP*
4. Nhấn **Submit**.

<img width="1093" height="694" alt="image" src="https://github.com/user-attachments/assets/e8ac3b7c-28f7-42c6-b85f-137f34742bb0" />

### 9.2. Quản lý qua phpMyAdmin

aaPanel tích hợp phpMyAdmin để quản lý database trực quan:

1. Vào **Database** > nhấn nút **phpMyAdmin** bên cạnh database cần quản lý.
2. Hệ thống tự động đăng nhập với đúng thông tin của database.
3. Từ đây có thể: import/export SQL, tạo bảng, chạy query, v.v.

<img width="1097" height="618" alt="image" src="https://github.com/user-attachments/assets/2b7bde54-3d2a-488f-ba34-a1b7976ecd53" />

   
<img width="1920" height="984" alt="image" src="https://github.com/user-attachments/assets/fc21aaf7-0c57-43af-907f-60e3582d1fe8" />

### 9.3. Backup và Restore database

```bash
# Backup database qua CLI
mysqldump -u root -p myapp_db > /root/backup_myapp_$(date +%Y%m%d).sql

# Restore database từ file backup
mysql -u root -p myapp_db < /root/backup_myapp_20250101.sql
```

Hoặc dùng tính năng Backup tích hợp trong aaPanel: **Database** > chọn database > **Backup**.

<img width="1467" height="540" alt="image" src="https://github.com/user-attachments/assets/fea47b8e-2f60-4536-a47a-b565c9d59984" />

Import

<img width="1549" height="766" alt="image" src="https://github.com/user-attachments/assets/6051f1b9-112e-4e36-83a6-daf2f70c2bf5" />


---

### 10.2. Đổi Security Entry (đường dẫn bí mật)

Đường dẫn bí mật ngăn chặn truy cập trái phép vào trang đăng nhập:

1. Vào **Panel Settings** > **Security**.
2. Tại mục **Security Entry**, nhập đường dẫn mới (vd: `/mysecretpath123`).
3. Lưu lại. URL mới sẽ là: `http://IP:PORT/mysecretpath123`.

<img width="1508" height="846" alt="image" src="https://github.com/user-attachments/assets/92ed376d-c887-404d-b3d9-775d633ab6b0" />

### 10.3. Bật xác thực hai yếu tố (2FA)

1. Vào **Panel Settings** > **Security** > **Two-Factor Authentication**.
2. Bật tính năng và quét mã QR bằng Google Authenticator hoặc Authy.
3. Nhập mã OTP để xác nhận kích hoạt thành công.

### 10.4. Giới hạn IP truy cập Panel

```bash
# Xóa rule cho phép tất cả truy cập port panel
ufw delete allow 8888/tcp

# Chỉ cho phép IP cụ thể truy cập
ufw allow from 192.168.1.100 to any port 8888
ufw reload
```

<img width="1610" height="536" alt="image" src="https://github.com/user-attachments/assets/820147b6-1d81-4d57-a9f3-7975e9d0b085" />


Hoặc qua giao diện: **Panel Settings** > **Security** > **IP Whitelist**.

### 10.5. Đổi mật khẩu admin

1. Nhấn vào tên tài khoản ở góc phải trên > **Account Settings**.
2. Nhập mật khẩu cũ và mật khẩu mới (tối thiểu 12 ký tự, kết hợp chữ hoa, chữ thường, số và ký tự đặc biệt).
3. Lưu thay đổi.

---

## XI. Các lệnh quản lý qua CLI

aaPanel cung cấp công cụ dòng lệnh `bt` với nhiều tùy chọn hữu ích:

```bash
# Mở menu quản lý tương tác
bt

# Xem thông tin đăng nhập mặc định
bt default

# Khởi động lại panel
bt restart

# Dừng panel
bt stop

# Xem log panel
bt log

# Xem trạng thái panel
bt status

# Lấy lại thông tin đăng nhập
bt 14

# Đổi port panel
bt 8

# Cập nhật aaPanel lên phiên bản mới nhất
bt 1
```

Quản lý các service web:

```bash
# Nginx
systemctl status nginx
systemctl restart nginx
systemctl reload nginx

# MySQL
systemctl status mysql
systemctl restart mysql

# PHP-FPM (ví dụ PHP 8.1)
systemctl status php8.1-fpm
systemctl restart php8.1-fpm
```

---

Qua quá trình thực hành cài đặt và cấu hình aaPanel trên Ubuntu Server 22.04 LTS, chúng ta đã hoàn thành được các mục tiêu sau:

1. Cài đặt thành công aaPanel sử dụng script tự động chính thức.
2. Cấu hình firewall UFW để bảo vệ máy chủ và chỉ mở các port cần thiết.
3. Triển khai LNMP Stack (Nginx + MySQL + PHP) qua giao diện
4. Tạo và quản lý website với cấu hình Nginx tùy chỉnh.
5. Cấu hình SSL/TLS
6. Quản lý database MySQL thông qua phpMyAdmin tích hợp.
7. Áp dụng các biện pháp bảo mật 

