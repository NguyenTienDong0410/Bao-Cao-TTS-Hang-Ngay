# Báo cáo Thực Tập 24/6 : Cài đặt và cấu hình Zabbix 7.2 trên Ubuntu 22.04

---

## Mục lục

1. [Giới thiệu](#1-giới-thiệu)
2. [Yêu cầu hệ thống](#2-yêu-cầu-hệ-thống)
3. [Các bước cài đặt](#3-các-bước-cài-đặt)
   - [Bước 1: Cập nhật hệ thống](#bước-1-cập-nhật-hệ-thống)
   - [Bước 2: Cài đặt Web Server Apache](#bước-2-cài-đặt-web-server-apache)
   - [Bước 3: Cài đặt và cấu hình MariaDB](#bước-3-cài-đặt-và-cấu-hình-mariadb)
   - [Bước 4: Cài đặt PHP và các module cần thiết](#bước-4-cài-đặt-php-và-các-module-cần-thiết)
   - [Bước 5: Cài đặt Zabbix repository, Server, Frontend và Agent](#bước-5-cài-đặt-zabbix-repository-server-frontend-và-agent)
   - [Bước 6: Cấu hình cơ sở dữ liệu cho Zabbix](#bước-6-cấu-hình-cơ-sở-dữ-liệu-cho-zabbix)
   - [Bước 7: Cấu hình Firewall](#bước-7-cấu-hình-firewall)
   - [Bước 8: Truy cập và hoàn tất cài đặt qua giao diện Web](#bước-8-truy-cập-và-hoàn-tất-cài-đặt-qua-giao-diện-web)
4. [Kết quả](#4-kết-quả)

---

## 1. Giới thiệu

Zabbix là một trong những công cụ giám sát hệ thống và mạng mạnh mẽ, phổ biến nhất hiện nay. Nền tảng này cho phép theo dõi hiệu suất của máy chủ, dịch vụ, ứng dụng và thiết bị mạng theo thời gian thực. Với khả năng thu thập, phân tích và cảnh báo sự cố một cách linh hoạt, Zabbix là lựa chọn lý tưởng cho các quản trị viên hệ thống trong môi trường doanh nghiệp.

Báo cáo này trình bày toàn bộ quy trình cài đặt Zabbix 7.2 trên Ubuntu 22.04, bao gồm: cài đặt các dịch vụ nền (Web Server, Database, PHP), thiết lập Zabbix Server và Zabbix Agent, cũng như hoàn tất cài đặt qua giao diện web.

---

## 2. Yêu cầu hệ thống

| Thành phần | Yêu cầu |
|---|---|
| Hệ điều hành | Ubuntu 22.04 LTS |
| Web Server | Apache hoặc Nginx |
| Cơ sở dữ liệu | MySQL hoặc MariaDB (10.5.00 – 11.5.x) |
| PHP | Phiên bản tương thích với Zabbix 7.2 |

> Tham khảo đầy đủ yêu cầu hệ thống tại: https://www.zabbix.com/documentation/current/en/manual/installation/requirements

---

## 3. Các bước cài đặt

### Bước 1: Cập nhật hệ thống

Trước khi bắt đầu cài đặt, cần cập nhật toàn bộ các gói phần mềm trên máy chủ lên phiên bản mới nhất nhằm đảm bảo tính tương thích và bảo mật.

```bash
sudo apt update -y
sudo apt upgrade -y
```

---

### Bước 2: Cài đặt Web Server Apache

Zabbix Frontend yêu cầu một Web Server để phục vụ giao diện người dùng. Trong bài hướng dẫn này, Apache được sử dụng.

```bash
sudo apt install apache2 -y
sudo systemctl enable apache2
sudo systemctl start apache2
sudo systemctl status apache2
```

Lệnh `systemctl enable` đảm bảo Apache tự khởi động cùng hệ thống sau mỗi lần reboot.

![Kiểm tra trạng thái Apache](https://github.com/user-attachments/assets/63f6978c-11d4-4e9c-87a5-58b08f11e349)

*Hình 1: Trạng thái hoạt động của dịch vụ Apache2*

---

### Bước 3: Cài đặt và cấu hình MariaDB

Zabbix 7.2 yêu cầu MariaDB phiên bản từ 10.5.00 đến 11.5.x. Để đảm bảo tương thích, cần thêm kho lưu trữ chính thức của MariaDB trước khi cài đặt.

**3.1. Thêm kho lưu trữ MariaDB**

```bash
sudo apt update
sudo curl -LsS https://downloads.mariadb.com/MariaDB/mariadb_repo_setup | sudo bash -s -- --mariadb-server-version=10.11
```

**3.2. Cài đặt MariaDB**

```bash
sudo apt update
sudo apt install mariadb-server mariadb-client -y
```

**3.3. Kiểm tra phiên bản sau khi cài đặt**

```bash
mariadb --version
```

![Kiểm tra phiên bản MariaDB](https://github.com/user-attachments/assets/40c9e763-e69d-4c7b-a9da-919c05052cc1)

*Hình 2: Phiên bản MariaDB sau khi cài đặt*

**3.4. Bảo mật MariaDB**

Chạy script `mysql_secure_installation` để thiết lập mật khẩu root, xóa người dùng ẩn danh, vô hiệu hóa đăng nhập root từ xa và xóa cơ sở dữ liệu test.

```bash
mysql_secure_installation
```

![Cấu hình bảo mật MariaDB - phần 1](https://github.com/user-attachments/assets/b91075e0-b2f3-4c69-9a97-e9cf9fce7482)

*Hình 3: Quá trình bảo mật MariaDB (phần 1)*

![Cấu hình bảo mật MariaDB - phần 2](https://github.com/user-attachments/assets/20efdefc-17ee-4f6c-a1b0-cc3ebe834253)

*Hình 4: Quá trình bảo mật MariaDB (phần 2)*

**3.5. Kích hoạt và khởi động lại dịch vụ MariaDB**

```bash
sudo systemctl enable mariadb
sudo systemctl restart mariadb
sudo systemctl status mariadb
```

![Trạng thái dịch vụ MariaDB](https://github.com/user-attachments/assets/09a9ae34-f22e-4c9b-99e6-0f95c802c926)

*Hình 5: Trạng thái hoạt động của dịch vụ MariaDB*

**3.6. Tạo cơ sở dữ liệu cho Zabbix**

Đăng nhập vào MariaDB với tài khoản root và thực hiện các lệnh sau để tạo database, user và phân quyền:

```sql
mysql -uroot -p

CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE USER zabbix@localhost IDENTIFIED BY 'a';
GRANT ALL PRIVILEGES ON zabbix.* TO zabbix@localhost;
FLUSH PRIVILEGES;
QUIT;
```

> **Lưu ý:** Câu lệnh `GRANT ALL PRIVILEGES` cần trỏ đúng tên database vừa tạo (`zabbix.*`). Đây là lỗi thường gặp nếu tên database không nhất quán giữa các bước.

> **Khuyến nghị bảo mật:** Mật khẩu trong ví dụ trên được đặt là `a` chỉ để minh họa. Trong môi trường thực tế, nên sử dụng mật khẩu đủ mạnh (ít nhất 12 ký tự, kết hợp chữ hoa, chữ thường, số và ký tự đặc biệt).

![Tạo cơ sở dữ liệu Zabbix](https://github.com/user-attachments/assets/4ca6f391-b025-4137-a9fd-6bba7c07b760)

*Hình 6: Tạo database và user cho Zabbix trong MariaDB*

---

### Bước 4: Cài đặt PHP và các module cần thiết

Zabbix Frontend được viết bằng PHP và yêu cầu một số module PHP bổ sung để hoạt động đầy đủ.

```bash
sudo apt install php php-mysql php-ldap php-bcmath php-gd php-xml libapache2-mod-php -y
```

Giải thích các module:
- `php-mysql`: kết nối PHP với MariaDB/MySQL
- `php-ldap`: hỗ trợ xác thực qua LDAP/Active Directory
- `php-bcmath`: xử lý số học độ chính xác cao
- `php-gd`: xử lý đồ họa (vẽ biểu đồ)
- `php-xml`: xử lý XML (cần thiết cho import/export cấu hình)
- `libapache2-mod-php`: tích hợp PHP với Apache

---

### Bước 5: Cài đặt Zabbix repository, Server, Frontend và Agent

**5.1. Thêm Zabbix repository**

```bash
wget https://repo.zabbix.com/zabbix/7.2/release/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.2+ubuntu22.04_all.deb
dpkg -i zabbix-release_latest_7.2+ubuntu22.04_all.deb
apt update
```

![Thêm Zabbix repository](https://github.com/user-attachments/assets/adb53495-3f7d-4a1d-91a7-14ee11a228e3)

*Hình 7: Thêm Zabbix repository thành công*

**5.2. Cài đặt các thành phần Zabbix**

```bash
apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent -y
```

Giải thích các gói:
- `zabbix-server-mysql`: Zabbix Server với backend MySQL/MariaDB
- `zabbix-frontend-php`: giao diện web quản trị Zabbix
- `zabbix-apache-conf`: cấu hình Apache cho Zabbix Frontend
- `zabbix-sql-scripts`: script SQL khởi tạo schema database
- `zabbix-agent`: Zabbix Agent (giám sát chính máy chủ Zabbix)

**5.3. Kiểm tra phiên bản Zabbix đã cài đặt**

```bash
apt-cache policy zabbix-server-mysql
```

![Kiểm tra phiên bản Zabbix](https://github.com/user-attachments/assets/12d675c0-9caf-41b6-a764-079de14a286f)

*Hình 8: Xác nhận phiên bản Zabbix Server đã cài đặt*

---

### Bước 6: Cấu hình cơ sở dữ liệu cho Zabbix

**6.1. Import schema database**

Zabbix cần import cấu trúc bảng (schema) và dữ liệu khởi tạo vào database đã tạo ở Bước 3. Thực hiện bằng lệnh:

```bash
zcat /usr/share/zabbix/sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
```

> Quá trình này có thể mất vài phút tùy cấu hình máy chủ.

**6.2. Cấu hình kết nối database trong Zabbix Server**

Mở file cấu hình Zabbix Server và điền thông tin kết nối database:

```bash
sudo vi /etc/zabbix/zabbix_server.conf
```

Tìm và chỉnh sửa các dòng sau:

```ini
DBName=zabbix
DBUser=zabbix
DBPassword=a
```

![Cấu hình file zabbix_server.conf](https://github.com/user-attachments/assets/ed802151-8d5a-4514-a9ed-7d94ee5fd4cd)

*Hình 9: Cấu hình thông tin kết nối database trong zabbix\_server.conf*

**6.3. Khởi động lại Zabbix Server**

```bash
sudo systemctl restart zabbix-server
sudo systemctl status zabbix-server
```

![Trạng thái Zabbix Server](https://github.com/user-attachments/assets/d4ece803-c0af-4e12-8684-534e0d5fed13)

*Hình 10: Zabbix Server hoạt động bình thường sau khi cấu hình*

**6.4. Cấu hình múi giờ cho Apache**

Zabbix Frontend yêu cầu khai báo múi giờ trong cấu hình Apache. Mở file và tìm dòng `php_value date.timezone`, sau đó đặt đúng múi giờ:

```bash
sudo nano /etc/zabbix/apache.conf
```

Chỉnh sửa:

```apache
php_value date.timezone Asia/Ho_Chi_Minh
```

![Cấu hình múi giờ trong apache.conf](https://github.com/user-attachments/assets/018fe1cf-237e-4681-a776-b09c2ec12e9a)

*Hình 11: Cấu hình múi giờ Asia/Ho\_Chi\_Minh trong apache.conf*

**6.5. Kích hoạt và khởi động lại toàn bộ dịch vụ**

```bash
sudo systemctl restart zabbix-server zabbix-agent apache2
sudo systemctl enable zabbix-server zabbix-agent apache2
```

![Kích hoạt các dịch vụ Zabbix](https://github.com/user-attachments/assets/d1224f07-f135-4926-8626-8ddd766f38b9)

*Hình 12: Kích hoạt tự khởi động cho các dịch vụ Zabbix*

---

### Bước 7: Cấu hình Firewall

Zabbix Server sử dụng một số port mặc định cần được mở trên Firewall:
- **Port 80/443**: giao diện web Zabbix Frontend (HTTP/HTTPS)
- **Port 10051**: Zabbix Server nhận dữ liệu từ Agent
- **Port 10050**: Zabbix Agent (nếu cần truy vấn chủ động từ server)

Trong môi trường này sử dụng UFW, thực hiện mở port qua giao diện quản trị CSF hoặc bằng lệnh:

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 10051/tcp
sudo ufw allow 10050/tcp
sudo ufw reload
```

![Cấu hình mở port trên Firewall](https://github.com/user-attachments/assets/a7d954b8-011c-4060-8a71-576bb1ce227d)

*Hình 13: Mở các port cần thiết cho Zabbix trên Firewall*

---

### Bước 8: Truy cập và hoàn tất cài đặt qua giao diện Web

**8.1. Truy cập Zabbix Frontend**

Mở trình duyệt và truy cập theo địa chỉ IP của máy chủ:

```
http://192.168.197.148/zabbix
```

![Giao diện khởi động Zabbix 7.2](https://github.com/user-attachments/assets/a2f896ec-e974-45a2-85c2-18c6f8d61354)

*Hình 14: Giao diện wizard cài đặt Zabbix 7.2*

**8.2. Các bước hoàn tất qua wizard**

Quá trình cài đặt qua giao diện web gồm các bước sau:

1. **Welcome**: Nhấn **Next step** để bắt đầu.
2. **Check of pre-requisites**: Hệ thống kiểm tra các yêu cầu PHP và module. Nhấn **Next step** nếu tất cả hiển thị màu xanh (OK).
3. **Configure DB connection**: Điền thông tin kết nối cơ sở dữ liệu đã tạo ở Bước 3, sau đó nhấn **Next step**.

![Cấu hình kết nối Database trong wizard](https://github.com/user-attachments/assets/6eae54d7-5b24-424d-872b-24edbf622e8c)

*Hình 15: Nhập thông tin kết nối database trong wizard cài đặt*

4. **Settings**: Đặt tên cho Zabbix Server (Zabbix server name) và chọn múi giờ mặc định (`Asia/Ho_Chi_Minh`), sau đó nhấn **Next step**.
5. **Pre-installation summary**: Xem lại toàn bộ cấu hình, nhấn **Next step** để tiến hành cài đặt.
6. **Install**: Nhấn **Finish** để hoàn tất.

**8.3. Đăng nhập lần đầu**

Sau khi cài đặt hoàn tất, hệ thống chuyển đến trang đăng nhập. Sử dụng thông tin tài khoản mặc định:

| Trường | Giá trị |
|---|---|
| Username | `Admin` |
| Password | `zabbix` |

> **Khuyến nghị bảo mật:** Sau lần đăng nhập đầu tiên, nên đổi ngay mật khẩu tài khoản `Admin` để tránh rủi ro bảo mật.

---

## 4. Kết quả

Sau khi hoàn tất các bước trên, Zabbix 7.2 đã được cài đặt và hoạt động thành công trên Ubuntu 22.04. Giao diện tổng quan (Dashboard) hiển thị đầy đủ trạng thái hệ thống, sẵn sàng cho việc thêm host và cấu hình giám sát.

![Giao diện đăng nhập Zabbix](https://github.com/user-attachments/assets/684fd72a-cdae-40de-9894-f8bef1150e76)

*Hình 16: Trang đăng nhập Zabbix*

![Giao diện tổng quan Zabbix Dashboard](https://github.com/user-attachments/assets/218b2761-0561-418b-a7da-4a6389ade2c2)

*Hình 17: Giao diện Dashboard sau khi đăng nhập thành công*
