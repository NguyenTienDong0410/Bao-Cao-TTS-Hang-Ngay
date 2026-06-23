## BÁO CÁO THỰC TẬP NGÀY 22/06
## THỰC HÀNH DEMO SSL/TLS

---

## 1. Thực hành

### 1.1 Tạo SSL

Để có chứng chỉ SSL đưa lên máy chủ (Apache/Nginx/Tomcat), chúng ta có 2 phương pháp ký (sign) khác nhau: Tự ký (Self-Signed) dùng cho test nội bộ, và Xin chữ ký từ CA (ZeroSSL) dùng cho môi trường thực tế.

#### 1.1.1 Self-Signed Certificate (Chứng chỉ tự ký)

**Giới thiệu:** Đây là chứng chỉ do chính máy chủ của bạn tự sinh ra và tự xác nhận (không thông qua CA nào). Khi truy cập, trình duyệt sẽ hiện cảnh báo đỏ "Kết nối của bạn không an toàn". Phương pháp này bỏ qua bước tạo CSR mà đi thẳng ra file chứng chỉ (CRT).

**Các bước thực hiện (trên Linux):**
Sử dụng lệnh OpenSSL với tham số `-x509` để sinh ra Private Key và Chứng chỉ cùng một lúc:

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout server.key -out server.crt
```

*Giải thích:*
*   `req -x509`: Báo cho OpenSSL tạo trực tiếp chứng chỉ tự ký (bỏ qua bước sinh file yêu cầu CSR).
*   `-nodes`: Không đặt mật khẩu cho Private Key (để Web server tự khởi động lại được).
*   `-days 365`: Thời hạn 1 năm.
*   Kết quả thu được 2 file: `server.key` (Private key) và `server.crt` (Chứng chỉ công khai).

<img width="1443" height="808" alt="image" src="https://github.com/user-attachments/assets/4ed48a98-34fc-4559-873c-c6ec874d2b23" />


#### 1.1.2 ZeroSSL Certificate (Chứng chỉ do CA ký)

**Giới thiệu:** Để đưa lên môi trường thực tế (Apache, Nginx) mà không bị trình duyệt cảnh báo, quy trình chuẩn (Best Practice) là bạn phải tạo một Yêu cầu cấp chứng chỉ (CSR) trên server, gửi CSR đó cho Tổ chức cấp phát (như ZeroSSL) để họ "ký" (Sign) và trả lại file Chứng chỉ hợp lệ.

**Bước 1: Tạo Private Key và CSR trên Server**
Sử dụng OpenSSL để tạo bộ khóa và file yêu cầu:

```bash
openssl req -new -newkey rsa:2048 -nodes -keyout private.key -out request.csr
```

*Lệnh này sẽ hỏi các thông tin doanh nghiệp. **Đặc biệt lưu ý:** Phần `Common Name (CN)` bạn phải nhập chính xác tên miền thật của bạn (VD: demo.yourdomain.com).*
Kết quả thu được file `private.key` (giữ tuyệt mật trên server) và `request.csr` (để gửi cho CA).

<img width="1915" height="894" alt="image" src="https://github.com/user-attachments/assets/a72b4ee4-2a28-4fd8-a2a0-3ee73f3d69c6" />

Dùng lệnh `cat request.csr` và copy toàn bộ nội dung văn bản bên trong.

<img width="796" height="567" alt="image" src="https://github.com/user-attachments/assets/8b56a93c-1504-4e24-9683-fb78e6f84ce5" />

-----BEGIN CERTIFICATE REQUEST-----
MIIDEDCCAfgCAQAwgZwxCzAJBgNVBAYTAlZOMQswCQYDVQQIDAJITjEPMA0GA1UE
BwwGSGEgTm9pMREwDwYDVQQKDAhOaGFuIEhvYTERMA8GA1UECwwITmhhbiBIb2Ex
GjAYBgNVBAMMEXd3dy50ZG9uZzQxLmlkLnZuMS0wKwYJKoZIhvcNAQkBFh5kb25n
a2hvbmduZ3UwNDEwMjAwNEBnbWFpbC5jb20wggEiMA0GCSqGSIb3DQEBAQUAA4IB
DwAwggEKAoIBAQCtOyj42/AKigNK2vcAC94/+/8LPLSVJ0CtBjhN8n2JtenNRtM4
801nn5iGgFK/ano9CVj82z3W76oTZgUI5r+j6yRgk7MUILjWyUXxRESZnEq9Lqi+
ygK30y7v9xrExGeEmtADQDmXV1NsCUAmmvwZU6HhPPiCHxJ2LxwZTzYwuhonqthJ
bNJx5MXea2F5rMLBLzIrdgolR0LejOM19aJR3fcF2kJIEi7lZIAyt/WOcmcVjZ9Y
ZjVBkh7+i93Cton3rrOR+Kf60gx+BeTIkN/lZgJDsf7nBr0PBe/Diix3B53q21yG
ULm+62dTiLmeuOezcqwg6YQxxOeFb6jM+JKLAgMBAAGgLjATBgkqhkiG9w0BCQcx
BgwEYWFhYTAXBgkqhkiG9w0BCQIxCgwITmhhbiBIb2EwDQYJKoZIhvcNAQELBQAD
ggEBABtLOArEdDEPnuhoklpGbz/THB0QE7HfXVZmkjXzdp1vWJAKVEvURjZhjap6
DLbL/gyx3xKknIB+shTFdmuDbQagM4jlIxX/5c8xqAhpOZg743qUy75PnQBZ3uXo
ERDvgm/UmNsgdSxnpW3V8TcqcUlEq8bYcgyKzaez2cxHNui7an1wqq7Ep97ZVY8m
FMx0jRHn/j9Q8Uo5n4Q+jq4ulRgxL9xzmGDh184XEbmEB2DDtaVHSJhEHhfjY344
0xI3D2gEnbB5fRfNGbU5Xi4X1E/Eo0F62hCYbVBzQbebZ/4ns7a45nGKFzb5r5kE
3gTr4w2zH+zsuSkoALEg/eOn8rE=
-----END CERTIFICATE REQUEST-----

**Bước 2: Xin chữ ký từ ZeroSSL**
1. Truy cập [ZeroSSL.com](https://zerossl.com/) tạo tài khoản và chọn **New Certificate**. Nhập tên miền của bạn.

   <img width="1920" height="873" alt="image" src="https://github.com/user-attachments/assets/55d16ec1-af36-4c30-91a5-c3e2a4e5e259" />

3. Tại phần CSR & Contact, thay vì để Auto-Generate (kém an toàn), bạn chọn **Paste Existing CSR** và dán nội dung file `request.csr` đã copy ở Bước 1 vào.
4. Hoàn thành quá trình xác minh quyền sở hữu tên miền (thường dùng cách tạo bản ghi CNAME trên hệ thống quản lý DNS).
   <img width="1125" height="776" alt="image" src="https://github.com/user-attachments/assets/50f33da4-d5c5-412f-b351-7a83b91cf5a7" />

add qua cloudflare 

<img width="1199" height="829" alt="image" src="https://github.com/user-attachments/assets/0f3cd626-c7e4-4dad-9421-9eeae07baeac" />

6. Sau khi xác minh thành công, bấm **Download Certificate**.
   <img width="1027" height="853" alt="image" src="https://github.com/user-attachments/assets/fcae8863-0224-4b90-96d7-353b7e6b86cf" />

8. Giải nén file tải về, bạn sẽ nhận được 2 file quan trọng: `certificate.crt` (Chứng chỉ đã được ZeroSSL ký) và `ca_bundle.crt` (Chuỗi chứng chỉ Root/Intermediate). Lúc này bạn không cần lấy private key từ ZeroSSL nữa vì file `private.key` đã nằm an toàn trên server của bạn từ Bước 1.

---

trang web ban đầu chưa được ký ssl/tls
<img width="1639" height="646" alt="image" src="https://github.com/user-attachments/assets/47e32242-121a-48fa-a187-7f49326143c8" />

Hiện tại web đang được triển khai trên direct admin, truy cập và add file đã tải trên zerossl

<img width="1844" height="794" alt="image" src="https://github.com/user-attachments/assets/09dc1acf-11a2-4364-b036-b50989a5b32c" />

Add key vào trong ssl cer Dán toàn bộ nội dung file private.key.

Ngay bên dưới: Dán tiếp toàn bộ nội dung file certificate.crt.
<img width="1184" height="843" alt="image" src="https://github.com/user-attachments/assets/bf1cfb25-3fb7-4087-988b-dbae304f169c" />

Gắn chứng chỉ trung gian (CA Bundle)

 <img width="1350" height="898" alt="image" src="https://github.com/user-attachments/assets/f9e7b6fb-7355-4c22-9598-419ba79b5607" />

sau khi thiết lập, trang web đã được cấp chứng chỉ

<img width="1920" height="812" alt="image" src="https://github.com/user-attachments/assets/32b76d79-594a-411a-9d67-7455f3a347e4" />

Dưới đây là bản trình bày chi tiết theo định dạng văn bản báo cáo kỹ thuật. Bạn có thể sử dụng cấu trúc này để đưa trực tiếp vào tài liệu thực hành hoặc báo cáo bài tập của mình.

---


**1. Mục đích triển khai**

* Thiết lập môi trường thử nghiệm (Local Environment) cho dịch vụ Web Server sử dụng giao thức HTTPS.
* Tích hợp bộ chứng chỉ bảo mật (SSL Certificate) đã được cấp phát bởi ZeroSSL cho tên miền `tdong41.id.vn` lên máy chủ cục bộ.
* Giả lập hệ thống phân giải tên miền (DNS Resolution) trên máy khách để kiểm thử tính hợp lệ của chứng chỉ (hiển thị ổ khóa bảo mật xanh) mà không làm gián đoạn dịch vụ đang chạy trên môi trường Internet (Production).

**2. Yêu cầu chuẩn bị**

* **Máy chủ Web (Local Server):** Hệ điều hành Ubuntu (đã cài đặt Apache hoặc NGINX), có địa chỉ IP LAN tĩnh (Ví dụ: `192.168.1.50`).
* **Máy khách (Client):** Máy tính sử dụng hệ điều hành Windows để truy cập và kiểm thử.
* **Tài nguyên SSL:** Bộ 3 file chứng chỉ của tên miền `tdong41.id.vn` bao gồm:
* `private.key` (Khóa bí mật tạo trên máy chủ).
* `certificate.crt` (Chứng chỉ gốc cấp bởi ZeroSSL).
* `ca_bundle.crt` (Chứng chỉ trung gian CA).



---

**3. Các bước tiến hành**

### Phân hệ 3.1: Quản lý và lưu trữ File Chứng chỉ trên Ubuntu Server

Để đảm bảo tính bảo mật và tổ chức hệ thống chuẩn xác, bộ khóa SSL cần được lưu trữ tại thư mục hệ thống với quyền truy cập được kiểm soát.

**Bước 1:** Di chuyển hoặc sao chép 3 file chứng chỉ (`private.key`, `certificate.crt`, `ca_bundle.crt`) vào thư mục `/etc/ssl/`.
<img width="851" height="134" alt="image" src="https://github.com/user-attachments/assets/3cfcf633-d6b8-4d40-94a7-5c872a3eb6d8" />


**Bước 2:** (Tùy chọn dành riêng cho NGINX) Tiến hành gộp file chứng chỉ gốc và chứng chỉ trung gian thành một chuỗi duy nhất:

```bash
sudo cat /etc/ssl/certificate.crt /etc/ssl/ca_bundle.crt > /etc/ssl/fullchain.crt

```
<img width="924" height="160" alt="image" src="https://github.com/user-attachments/assets/7371365b-cd38-45c4-8387-1f752d6f5d0d" />


---

**1. Mục đích triển khai**

* Thiết lập môi trường thử nghiệm (Local Environment) cho dịch vụ Web Server sử dụng giao thức HTTPS.
* Tích hợp bộ chứng chỉ bảo mật (SSL Certificate) đã được cấp phát bởi ZeroSSL cho tên miền `tdong41.id.vn` lên máy chủ cục bộ.
* Giả lập hệ thống phân giải tên miền (DNS Resolution) trên máy khách để kiểm thử tính hợp lệ của chứng chỉ (hiển thị ổ khóa bảo mật xanh) mà không làm gián đoạn dịch vụ đang chạy trên môi trường Internet (Production).

**2. Yêu cầu chuẩn bị**

* **Máy chủ Web (Local Server):** Hệ điều hành Ubuntu (đã cài đặt Apache hoặc NGINX), có địa chỉ IP LAN tĩnh (Ví dụ: `192.168.1.50`).
* **Máy khách (Client):** Máy tính sử dụng hệ điều hành Windows để truy cập và kiểm thử.
* **Tài nguyên SSL:** Bộ 3 file chứng chỉ của tên miền `tdong41.id.vn` bao gồm:
* `private.key` (Khóa bí mật tạo trên máy chủ).
* `certificate.crt` (Chứng chỉ gốc cấp bởi ZeroSSL).
* `ca_bundle.crt` (Chứng chỉ trung gian CA).



---

**3. Các bước tiến hành**

### Phân hệ 3.1: Quản lý và lưu trữ File Chứng chỉ trên Ubuntu Server

Để đảm bảo tính bảo mật và tổ chức hệ thống chuẩn xác, bộ khóa SSL cần được lưu trữ tại thư mục hệ thống với quyền truy cập được kiểm soát.

**Bước 1:** Di chuyển hoặc sao chép 3 file chứng chỉ (`private.key`, `certificate.crt`, `ca_bundle.crt`) vào thư mục `/etc/ssl/`.
**Bước 2:** (Tùy chọn dành riêng cho NGINX) Tiến hành gộp file chứng chỉ gốc và chứng chỉ trung gian thành một chuỗi duy nhất:

```bash
sudo cat /etc/ssl/certificate.crt /etc/ssl/ca_bundle.crt > /etc/ssl/fullchain.crt

```

### Phân hệ 3.2: Khởi tạo mã nguồn Web thử nghiệm (Document Root)

Để kiểm chứng việc truy cập thành công, cần tạo một trang HTML đơn giản làm nội dung phản hồi cho tên miền thay vì dùng trang mặc định của Web Server.

**Bước 1:** Đảm bảo thư mục lưu trữ mã nguồn đã tồn tại và phân quyền đúng:

```bash
sudo mkdir -p /var/www/html
sudo chown -R www-data:www-data /var/www/html
sudo chmod -R 755 /var/www/html

```

**Bước 2:** Tạo tệp tin `index.html` với nội dung nhận diện riêng:

```bash
echo "<h1>Kiem thu thanh cong SSL Local cho tdong41.id.vn</h1>" | sudo tee /var/www/html/index.html

```
<img width="1304" height="354" alt="image" src="https://github.com/user-attachments/assets/1d1ab691-783a-411a-86b1-732186c9c633" />

### Phân hệ 3.3: Khai báo VirtualHost / Server Block trên Web Server

Cấu hình Web Server để lắng nghe kết nối mã hóa qua cổng 443 và trỏ đường dẫn đến bộ khóa SSL.

**Tùy chọn A: Dành cho Apache Server**
Mở và chỉnh sửa file cấu hình tại `/etc/apache2/sites-available/tdong41-ssl.conf`:

```apache
<VirtualHost *:443>
    ServerName tdong41.id.vn
    DocumentRoot /var/www/html

    # Kích hoạt module SSL
    SSLEngine on

    # Khai báo đường dẫn chứng chỉ
    SSLCertificateFile /etc/ssl/certificate.crt
    SSLCertificateKeyFile /etc/ssl/private.key
    SSLCertificateChainFile /etc/ssl/ca_bundle.crt

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

```
<img width="1285" height="495" alt="image" src="https://github.com/user-attachments/assets/58d690b8-fed7-444a-9222-bc1dbd29e3fc" />

Thực thi lệnh kiểm tra và khởi động lại dịch vụ:

```bash
sudo a2enmod ssl
sudo a2ensite tdong41-ssl.conf
sudo systemctl restart apache2

```
<img width="736" height="385" alt="image" src="https://github.com/user-attachments/assets/d44212cd-85fa-4e21-b1f4-07cde08ccde4" />

**Tùy chọn B: Dành cho NGINX Server**
Mở và chỉnh sửa file cấu hình tại `/etc/nginx/sites-available/tdong41`:

```nginx
server {
    listen 443 ssl;
    server_name tdong41.id.vn;
    root /var/www/html;
    index index.html;

    # Khai báo đường dẫn chứng chỉ (sử dụng file đã gộp)
    ssl_certificate /etc/ssl/fullchain.crt;
    ssl_certificate_key /etc/ssl/private.key;

    # Tối ưu hóa bảo mật SSL
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
}

```
<img width="1132" height="472" alt="image" src="https://github.com/user-attachments/assets/9dc1fd82-1609-4858-a99b-413a04781461" />

Thực thi lệnh kiểm tra và khởi động lại dịch vụ:

```bash
sudo ln -s /etc/nginx/sites-available/tdong41 /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx

```

### Phân hệ 3.4: Điều hướng DNS cục bộ trên Máy khách (Windows)

Đây là kỹ thuật cốt lõi để giả lập môi trường thực tế. Cần can thiệp vào file `hosts` của Windows để ép trình duyệt phân giải tên miền `tdong41.id.vn` về IP của máy chủ Local thay vì IP Public.

**Bước 1:** Khởi chạy ứng dụng Notepad dưới quyền Quản trị viên (Run as Administrator).
**Bước 2:** Mở tệp tin cấu hình hệ thống theo đường dẫn:
`C:\Windows\System32\drivers\etc\hosts`
**Bước 3:** Bổ sung bản ghi định tuyến cục bộ vào cuối tệp tin (thay IP `192.168.197.148` bằng IP thực tế của máy chủ Ubuntu):

```text
# Local Testing cho Môi trường SSL
192.168.197.148   tdong41.id.vn
192.168.197.148   www.tdong41.id.vn

```
<img width="571" height="80" alt="image" src="https://github.com/user-attachments/assets/af610ede-0a66-45da-8f4d-445b3d2b677d" />

**Bước 4:** Lưu tệp tin (`Ctrl + S`).

---

**4. Kết quả và Quy trình kiểm thử**

* **Môi trường kiểm thử:** Trình duyệt web (Google Chrome / Microsoft Edge) sử dụng chế độ Ẩn danh (Incognito) để tránh lưu Cache.
* **Thao tác:** Truy cập vào URL `https://tdong41.id.vn`.
* **Kết quả thu được:**
1. Trình duyệt hiển thị thành công dòng chữ: *"Kiem thu thanh cong SSL Local cho tdong41.id.vn"*.
2. Thanh địa chỉ hiển thị trạng thái bảo mật hợp lệ (Biểu tượng ổ khóa màu xanh / Connection is secure).
3. Kiểm tra chi tiết chứng chỉ (Certificate Details) cho thấy thông tin nhà phát hành (Issuer) là **ZeroSSL RSA Domain Secure Site CA** và tên miền được cấp (Common Name) hoàn toàn trùng khớp.
<img width="1633" height="324" alt="image" src="https://github.com/user-attachments/assets/bbdfd69e-4ca7-4563-a1ff-bf1cbb3a7c60" />

<img width="991" height="912" alt="image" src="https://github.com/user-attachments/assets/45c47a54-d99d-46a4-842a-a0a3d3778c62" />




