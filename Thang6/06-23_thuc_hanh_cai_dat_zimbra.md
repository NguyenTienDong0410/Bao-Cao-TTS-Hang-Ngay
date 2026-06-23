---

# Cài đặt Zimbra OSE 10.1.4 trên Ubuntu 22.04

Zimbra là giải pháp hàng đầu về email, lịch và cộng tác, có thể được triển khai trên đám mây công cộng hoặc riêng tư. Với giao diện dựa trên trình duyệt được thiết kế lại, nó mang đến trải nghiệm sáng tạo và ấn tượng nhất hiện nay, kết nối người dùng cuối với thông tin và hoạt động trong đám mây riêng của họ. Vì vậy, ứng dụng này phù hợp để sử dụng trong các công ty nhỏ, vừa và lớn, trường học, cơ quan, v.v.

## Các bước cài đặt Zimbra OSE 10.1.4

### 1. Cài đặt các gói phần mềm cần thiết

```bash
sudo apt-get update
sudo apt-get install nano wget bind9 bind9utils telnet perl ufw tar resolvconf net-tools -y

```

### 2. Thiết lập múi giờ

```bash
timedatectl set-timezone Asia/Ho_Chi_Minh

```

### 3. Thiết lập tên máy chủ (Hostname)

```bash
hostnamectl set-hostname mail.domain.com

```

### 4. Dừng và vô hiệu hóa Postfix (Tránh xung đột cổng mail)

```bash
systemctl stop postfix
systemctl disable postfix

```

### 5. Cấu hình file Hosts

Mở file `/etc/hosts`:

```bash
nano /etc/hosts

```
<img width="813" height="414" alt="image" src="https://github.com/user-attachments/assets/5e21fbc5-69d5-4a56-847c-2e4d92df809b" />

<img width="1817" height="663" alt="image" src="https://github.com/user-attachments/assets/4d5402b2-c799-4310-b78a-890c55f53f46" />


Thêm dòng ánh xạ IP và Hostname của bạn vào nội dung file (Lưu lại bằng `Ctrl + O`, thoát bằng `Ctrl + X`):

```text
192.168.197.148 mail.domain.com mail

```

### 6. Cấu hình resolv.conf

```bash
systemctl enable resolvconf
systemctl start resolvconf
cp /etc/resolv.conf /etc/resolv.conf.backup

echo "search domain.com" >> /etc/resolvconf/resolv.conf.d/head
echo "nameserver 192.168.197.148" >> /etc/resolvconf/resolv.conf.d/head

sudo resolvconf --enable-updates
sudo resolvconf -u

```

---

## Tạo máy chủ DNS cục bộ (Local DNS Server)

### 7. Tạo vùng (Zone) trên máy chủ DNS bind

Sao lưu cấu hình mặc định:

```bash
cp /etc/bind/named.conf.local /etc/bind/named.conf.local.backup
cp /etc/bind/named.conf.options /etc/bind/named.conf.options.backup
> /etc/bind/named.conf.local
sed -i '/directory*/a\        forwarders {8.8.8.8; 8.8.4.4;};' /etc/bind/named.conf.options

```

Chỉnh sửa tệp `/etc/bind/named.conf.local`:

```bash
nano /etc/bind/named.conf.local 

```

Nhập vào nội dung sau:

```text
zone "domain.com" {
        type master;
        file "/var/lib/bind/domain.com.hosts";
        allow-transfer {
                127.0.0.1;
                localnets;
                };
        };

```
<img width="1329" height="468" alt="image" src="https://github.com/user-attachments/assets/8d182860-36b0-4900-b69f-24378981f7c7" />

### 8. Tạo file cơ sở dữ liệu zone `domain.com.hosts`

Tạo tệp tin cấu hình bản ghi bằng trình soạn thảo văn bản:

```bash
nano /var/lib/bind/domain.com.hosts

```

Nhập chính xác các thông số cấu hình DNS sau:

```text
$ttl 3600
@       IN      SOA     mail.domain.com. root.mail.domain.com. (
                        1615364925
                        3600
                        600
                        1209600
                        3600 )
domain.com.       IN      NS      mail.domain.com.
mail.domain.com.  IN      A       192.168.197.148
domain.com.       IN      MX      10 mail

```

### 9. Khởi động lại và kích hoạt dịch vụ DNS Bind

```bash
systemctl restart named
systemctl enable named

```
<img width="795" height="215" alt="image" src="https://github.com/user-attachments/assets/5356cb05-cd08-49df-83ea-d5ffbb5dec39" />

### 10. Kiểm tra phân giải tên miền (DNS Check)

```bash
nslookup mail.domain.com

```

Nếu kết quả trả về khớp với IP cục bộ của bạn như bên dưới là cấu hình chính xác:

```text
Server:         192.168.197.148
Address:        192.168.197.148#53

Name:   mail.domain.com
Address: 192.168.197.148

```
<img width="534" height="168" alt="image" src="https://github.com/user-attachments/assets/14641539-8a9a-467b-b954-eed9578223e3" />

---

## Tải xuống và cài đặt Zimbra Mail Server

### 1. Tải xuống gói cài đặt Zimbra

```bash
cd /opt/
wget https://github.com/maldua/zimbra-foss-builder/releases/download/zimbra-foss-build-ubuntu-22.04/10.1.16/zcs-10.1.16_GA_4200001.UBUNTU22_64.20260310121616.tgz

```

### 2. Giải nén gói cài đặt

```bash
tar -zxvf zcs-10.1.16_GA_4200001.UBUNTU22_64.20260310121616.tgz
cd zcs-10.1.16_GA_4200001.UBUNTU22_64.20260310121616

```
<img width="1179" height="517" alt="image" src="https://github.com/user-attachments/assets/40eaa48d-ad43-487b-9c69-5c664356f9f9" />

### 3. Khởi chạy Script cài đặt

```bash
./install.sh

```

### 4. Chấp nhận điều khoản bản quyền

Khi màn hình hiển thị nội dung EULA, gõ `y` rồi nhấn **Enter**:

```text
Do you agree with the terms of the software license agreement? [N] y

```

### 5. Sử dụng Kho lưu trữ gói của Zimbra

Gõ `y` rồi nhấn **Enter**:

```text
Use Zimbra's package repository [Y] y

```

### 6. Lựa chọn các Package để cài đặt

Lựa chọn các gói cài đặt theo mẫu bên dưới (Gõ `y` hoặc `n` tương ứng):

```text
Select the packages to install

Install zimbra-ldap [Y] y
Install zimbra-logger [Y] y
Install zimbra-mta [Y] y
Install zimbra-dnscache [Y] n
Install zimbra-snmp [Y] y
Install zimbra-store [Y] y
Install zimbra-apache [Y] y
Install zimbra-spell [Y] y
Install zimbra-memcached [Y] y
Install zimbra-proxy [Y] y

```
<img width="891" height="1001" alt="image" src="https://github.com/user-attachments/assets/2067618a-8433-4d09-b440-10ca9322b219" />

### 7. Xác nhận thay đổi hệ thống

Gõ `y` rồi nhấn **Enter** để bắt đầu tiến trình cài đặt các gói vào hệ thống:

```text
The system will be modified. Continue? [N] y

```

### 8. Xử lý cấu hình Domain (Sửa lỗi MX Record)

Hệ thống sẽ kiểm tra MX record và đưa ra cảnh báo cấu hình. Gõ `y` để đổi lại tên miền chính xác:

```text
DNS ERROR resolving MX for mail.domain.com
It is suggested that the domain name have an MX record configured in DNS
Change domain name? [Yes] y
Create domain: [mail.domain.com] domain.com

```

### 9. Cấu hình mật khẩu Quản trị viên (Admin Password)

<img width="1027" height="1076" alt="image" src="https://github.com/user-attachments/assets/0f790188-091b-4ff0-930a-92249ede045f" />

Khi màn hình hiển thị **Main menu**, bạn cần cấu hình password cho tài khoản admin bị thiếu (`UNSET`):

```text
Main menu
   1) Common Configuration:                                                  
   2) zimbra-ldap:                             Enabled                       
   3) zimbra-logger:                           Enabled                       
   4) zimbra-mta:                              Enabled                       
   5) zimbra-snmp:                             Enabled                       
   6) zimbra-store:                            Enabled                       
        +Create Admin User:                    yes                           
        +Admin user to create:                 admin@domain.com             
******* +Admin Password                        UNSET                         
...
Address unconfigured (**) items (? - help) 6

```

* Gõ `6` (hoặc số tương ứng với mục `zimbra-store`) rồi nhấn **Enter**.
<img width="957" height="734" alt="image" src="https://github.com/user-attachments/assets/d2708a97-50e1-4312-ac80-d00d0a9575a7" />

Tiếp theo, chọn mục cấu hình mật khẩu:

```text
Store configuration
   1) Status:                                 Enabled                       
   2) Create Admin User:                      yes                           
   3) Admin user to create:                   admin@domain.com             
** 4) Admin Password                          UNSET                         
...
Select, or 'r' for previous menu [r] 4

```

* Gõ `4` rồi nhấn **Enter**.
* Tiến hành nhập mật khẩu admin mong muốn của bạn (ví dụ: `rahasia10`) rồi nhấn **Enter**:

```text
Password for admin@domain.com (min 6 characters): [EHO3AXUi7] rahasia10

```

### 10. Áp dụng cấu hình hệ thống

Quay trở lại menu chính bằng cách gõ `r` và nhấn **Enter**:

```text
Select, or 'r' for previous menu [r] r

```

Tại **Main menu**, gõ `a` để lưu và áp dụng toàn bộ cấu hình:

```text
Select from menu, or press 'a' to apply config (? - help) a
Save configuration data to a file? [Yes] y
Save config in file: [/opt/zimbra/config.11662] nhấn Enter
Saving config in /opt/zimbra/config.11662...done.
The system will be modified - continue? [No] y

```

### 11. Từ chối gửi thông báo cài đặt về hệ thống Zimbra

Gõ `n` rồi nhấn **Enter**:

```text
Notify Zimbra of your installation? [Yes] n

```

### 12. Hoàn tất cài đặt

Khi hệ thống chạy xong cấu hình cơ sở dữ liệu và khởi động dịch vụ, nhấn **Enter** để kết thúc quá trình cài đặt:

```text
Configuration complete - press return to exit

```
<img width="883" height="802" alt="image" src="https://github.com/user-attachments/assets/77326b97-fad1-4855-ac15-6cac6f7a9774" />

---

## Cấu hình Tường lửa (UFW)

### 13. Kích hoạt UFW

```bash
sudo ufw enable

```

### 14. Mở các cổng dịch vụ cần thiết cho Zimbra Mail

Sao chép toàn bộ khối lệnh dưới đây, dán vào terminal và nhấn **Enter**:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 25/tcp
sudo ufw allow 80/tcp
sudo ufw allow 110/tcp
sudo ufw allow 143/tcp
sudo ufw allow 443/tcp
sudo ufw allow 465/tcp
sudo ufw allow 587/tcp
sudo ufw allow 993/tcp
sudo ufw allow 995/tcp
sudo ufw allow 3443/tcp
sudo ufw allow 5222/tcp
sudo ufw allow 5223/tcp
sudo ufw allow 9071/tcp
sudo ufw allow 8443/tcp
sudo ufw allow 7071/tcp
sudo ufw allow 53/tcp
sudo ufw allow 53/udp

```

---

## Đường dẫn truy cập hệ thống

* **Trang quản trị dành cho Admin (Admin Console):** `https://192.168.197.148:7071`
* **Trang đăng nhập kiểm tra thư của người dùng (Webmail):** `https://192.168.197.148`
<img width="1911" height="892" alt="image" src="https://github.com/user-attachments/assets/431d25fa-2801-42bb-aeee-b5cbf3cca3ad" />
