## BÁO CÁO THỰC TẬP NGÀY 22/06
# PHẦN 12: THỰC HÀNH DEMO TẠO VÀ CÀI ĐẶT SSL/TLS

Tài liệu này hướng dẫn chi tiết các bước thực hành tạo chứng chỉ SSL (bao gồm tự ký và sử dụng dịch vụ miễn phí) cũng như cách triển khai cấu hình chúng trên các Web Server phổ biến.

---

## 12.1 Tạo SSL

### 12.1.1 Tạo Self-Signed Certificate (Chứng chỉ tự ký)

**Giới thiệu:** Chứng chỉ tự ký là chứng chỉ do chính bạn sinh ra mà không thông qua bất kỳ Tổ chức cấp phát (CA) nào. Khi truy cập, trình duyệt sẽ hiện cảnh báo đỏ "Kết nối của bạn không an toàn". Nó chỉ phù hợp cho mục đích test nội bộ (Localhost) hoặc môi trường Dev.

**Công cụ thực hiện:** Sử dụng `OpenSSL` (thường có sẵn trên các hệ điều hành Linux).

**Các bước thực hiện:**
Mở Terminal và chạy câu lệnh sau để vừa sinh ra Private Key, vừa tạo Chứng chỉ x509:

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout server.key -out server.crt
```

**Giải thích các tham số:**
*   `req -x509`: Báo cho OpenSSL biết ta muốn tạo trực tiếp chứng chỉ X.509 thay vì tạo file yêu cầu (CSR).
*   `-nodes`: (No DES) Không mã hóa Private Key bằng mật khẩu, giúp Web Server có thể tự động khởi động mà không cần người quản trị nhập pass.
*   `-days 365`: Thời hạn của chứng chỉ là 1 năm.
*   `-newkey rsa:2048`: Tự động tạo một Private Key mới bằng thuật toán RSA độ dài 2048-bit.
*   `-keyout`: Tên file Private Key sẽ được xuất ra (`server.key`).
*   `-out`: Tên file Chứng chỉ sẽ được xuất ra (`server.crt`).

Sau khi chạy lệnh, hệ thống sẽ yêu cầu bạn nhập các thông tin cho chứng chỉ:
```text
Country Name (2 letter code) [XX]: VN
State or Province Name (full name) []: Hanoi
Locality Name (eg, city) [Default City]: Hanoi
Organization Name (eg, company) [Default Company Ltd]: IT Demo Company
Organizational Unit Name (eg, section) []: IT Dept
Common Name (eg, your name or your server's hostname) []: test.local
Email Address []: admin@test.local
```
*(Lưu ý: Quan trọng nhất là phần **Common Name** phải điền chính xác tên miền hoặc IP bạn dùng để test).*

**Kết quả:** Bạn nhận được 2 file `server.key` và `server.crt`.

---

### 12.1.2 Tạo ZeroSSL Certificate (Chứng chỉ miễn phí tin cậy)

**Giới thiệu:** ZeroSSL cung cấp chứng chỉ SSL miễn phí thời hạn 90 ngày (tương tự Let's Encrypt). Chứng chỉ này được các trình duyệt tin tưởng hoàn toàn (hiển thị ổ khóa xanh an toàn).

**Các bước thực hiện trên Web ZeroSSL:**
1. Truy cập [ZeroSSL.com](https://zerossl.com/) và tạo một tài khoản miễn phí.
2. Tại màn hình Dashboard, click vào **New Certificate**. Nhập tên miền thật của bạn (ví dụ: `demo.yourdomain.com`).
3. Chọn gói **90-Day Certificate** (Miễn phí) và bật tính năng **Auto-Generate CSR**.
4. **Xác minh quyền sở hữu tên miền (Domain Verification):** ZeroSSL sẽ yêu cầu bạn chứng minh bạn là chủ sở hữu tên miền bằng 1 trong 3 cách:
    *   **Email Verification:** Gửi email đến `admin@yourdomain.com`.
    *   **DNS (CNAME):** Tạo một bản ghi CNAME trên hệ thống quản lý DNS của tên miền. *(Cách phổ biến và khuyên dùng nhất)*.
    *   **HTTP File Upload:** Upload một file text dạng `.txt` lên hosting theo đường dẫn ZeroSSL chỉ định (ví dụ: `http://yourdomain.com/.well-known/pki-validation/...`).
5. Sau khi xác minh thành công, bấm **Download Certificate**.
6. Giải nén file tải về, bạn sẽ nhận được 3 file:
    *   `certificate.crt` (Chứng chỉ của tên miền)
    *   `ca_bundle.crt` (Chứng chỉ của Intermediate CA và Root CA)
    *   `private.key` (Private key của bạn)

---

## 12.2 Cài đặt và cấu hình SSL trên các Webserver

Phần này hướng dẫn cách đưa các file Chứng chỉ và Private key (vừa tạo ở phần 12.1) lên các máy chủ web phổ biến.

### 12.2.1 Cài đặt trên Apache

**Bước 1:** Đảm bảo module SSL của Apache đã được bật.
```bash
sudo a2enmod ssl
sudo systemctl restart apache2
```

**Bước 2:** Copy các file `certificate.crt`, `private.key`, `ca_bundle.crt` vào thư mục an toàn trên server (ví dụ `/etc/ssl/`).

**Bước 3:** Chỉnh sửa file cấu hình VirtualHost (thường nằm ở `/etc/apache2/sites-available/default-ssl.conf` hoặc cấu hình web của bạn):

```apache
<VirtualHost *:443>
    ServerName demo.yourdomain.com
    DocumentRoot /var/www/html

    # Bật SSL Engine
    SSLEngine on

    # Đường dẫn tới file Chứng chỉ
    SSLCertificateFile /etc/ssl/certificate.crt

    # Đường dẫn tới Private Key
    SSLCertificateKeyFile /etc/ssl/private.key

    # Đường dẫn tới Chuỗi chứng chỉ (CA Bundle)
    SSLCertificateChainFile /etc/ssl/ca_bundle.crt
</VirtualHost>
```

**Bước 4:** Kiểm tra lỗi cú pháp và khởi động lại Apache:
```bash
sudo apachectl configtest
sudo systemctl restart apache2
```

---

### 12.2.2 Cài đặt trên NGINX

Khác với Apache, Nginx yêu cầu bạn phải **gộp chung** file Chứng chỉ của domain (`certificate.crt`) và file Chuỗi chứng chỉ (`ca_bundle.crt`) thành một file duy nhất (gọi là `fullchain.crt`).

**Bước 1:** Gộp file chứng chỉ (Lưu ý: File chứng chỉ domain phải nằm trên, CA bundle nằm dưới).
```bash
cat certificate.crt ca_bundle.crt > fullchain.crt
```

**Bước 2:** Di chuyển `fullchain.crt` và `private.key` vào thư mục an toàn (VD: `/etc/nginx/ssl/`).

**Bước 3:** Mở file cấu hình Nginx (thường ở `/etc/nginx/sites-available/default` hoặc `/etc/nginx/conf.d/domain.conf`):

```nginx
server {
    # Lắng nghe cổng 443 và bật tính năng SSL
    listen 443 ssl;
    server_name demo.yourdomain.com;

    # Cấu hình đường dẫn chứng chỉ
    ssl_certificate /etc/nginx/ssl/fullchain.crt;
    ssl_certificate_key /etc/nginx/ssl/private.key;

    # Cấu hình các giao thức bảo mật (Chỉ dùng TLS 1.2 và 1.3)
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    location / {
        root /var/www/html;
        index index.html index.htm;
    }
}
```

**Bước 4:** Kiểm tra cú pháp và khởi động lại Nginx:
```bash
sudo nginx -t
sudo systemctl restart nginx
```

---

### 12.2.3 Cài đặt trên Tomcat (Java)

Tomcat không sử dụng trực tiếp các file định dạng PEM (`.crt`, `.key`) như Nginx hay Apache mà sử dụng Java Keystore (`.jks`) hoặc PKCS12 (`.p12`).

**Bước 1: Chuyển đổi định dạng sang PKCS12 (.p12)**
Bạn cần dùng OpenSSL để đóng gói cả 3 file (Cert, Key, CA Bundle) vào 1 file `.p12`.

```bash
openssl pkcs12 -export -out keystore.p12 -inkey private.key -in certificate.crt -certfile ca_bundle.crt -name "tomcat"
```
Hệ thống sẽ yêu cầu bạn tạo một mật khẩu xuất (Export Password). Hãy nhớ mật khẩu này (ví dụ: `matkhau123`).

**Bước 2:** Di chuyển file `keystore.p12` vào thư mục cấu hình của Tomcat (ví dụ `/opt/tomcat/conf/`).

**Bước 3: Sửa cấu hình server.xml**
Mở file `/opt/tomcat/conf/server.xml`, tìm thẻ `<Connector>` có cổng `443` (hoặc `8443` tùy thiết lập) và chỉnh sửa như sau:

```xml
<Connector port="443" 
           protocol="org.apache.coyote.http11.Http11NioProtocol"
           maxThreads="150" 
           SSLEnabled="true" 
           scheme="https" 
           secure="true"
           clientAuth="false" 
           sslProtocol="TLS">
    
    <SSLHostConfig>
        <Certificate certificateKeystoreFile="conf/keystore.p12"
                     certificateKeystorePassword="matkhau123"
                     certificateKeystoreType="PKCS12" />
    </SSLHostConfig>
</Connector>
```

**Bước 4:** Khởi động lại Tomcat để áp dụng.
```bash
/opt/tomcat/bin/shutdown.sh
/opt/tomcat/bin/startup.sh
```

---

## Tổng kết thực hành
Sau khi cài đặt xong trên một trong các Webserver kể trên, bạn truy cập vào tên miền qua giao thức `https://`.
*   Đối với bản **Self-Signed**: Trình duyệt sẽ hiện cảnh báo. Để tiếp tục, bạn nhấn "Advanced" (Nâng cao) -> "Proceed to site" (Tiếp tục truy cập).
*   Đối với bản **ZeroSSL**: Trình duyệt sẽ nhận diện ngay lập tức và hiển thị biểu tượng ổ khóa an toàn, chứng minh quá trình cài đặt Demo đã hoàn tất thành công.
