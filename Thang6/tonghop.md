# TÀI LIỆU ÔN TẬP — BUỔI REVIEW NHÂN HÒA GROUP
> Tổng hợp từ báo cáo thực tập Tháng 4–6 | Nguyễn Tiến Đông

---

## 1. LINUX — PHÂN QUYỀN, USER, NETWORK

### Kiến thức cốt lõi
- Linux gồm 3 layer: **Hardware → Kernel → User Process (Shell, Utilities, App)**
- File quan trọng: `/etc/passwd` (thông tin user), `/etc/shadow` (hash mật khẩu, chỉ root đọc được), `/etc/group` (thông tin nhóm)

### Phân quyền
| Quyền | Số | Ý nghĩa |
|---|---|---|
| `chmod 755` | rwxr-xr-x | Owner full, group/other đọc+chạy — dùng cho thư mục, file thực thi |
| `chmod 644` | rw-r--r-- | Owner đọc/ghi, group/other chỉ đọc — dùng cho file config |
| `chmod 1777` | drwxrwxrwt | Sticky Bit — chỉ owner mới được xóa file của mình |

**Quyền đặc biệt:**
- `SUID`: File chạy với quyền của owner, không phải user hiện tại
- `SGID`: File chạy với quyền của group; thư mục → file tạo trong đó kế thừa group
- `Sticky Bit`: Thư mục dùng chung — chỉ owner file mới xóa được file của mình

### Lệnh hay dùng
```bash
chmod [mode] file        # đổi quyền
chown user:group file    # đổi owner và group cùng lúc
chgrp group file         # chỉ đổi group
useradd -m alice         # tạo user + tạo /home/alice
usermod -aG devteam alice # thêm alice vào group devteam (không xóa group cũ)
userdel -r alice         # xóa user + xóa /home/alice
ss -tlnp                 # xem port đang lắng nghe + PID
top / htop               # xem process, CPU, RAM realtime
df -h / du -sh           # kiểm tra dung lượng disk
```

### NFS vs SMB
- **NFS**: Dùng trên Linux/Unix, hiệu năng tốt trong môi trường thuần Linux, mount qua `/etc/fstab`
- **SMB (Samba)**: Tương thích Windows, cross-platform tốt hơn

### LVM
- **PV (Physical Volume)** → **VG (Volume Group)** → **LV (Logical Volume)**
- Ưu điểm: thay đổi kích thước không cần reboot, gộp nhiều disk, snapshot dễ dàng

---

## 2. LOGS VÀ SECURITY

### Các file log quan trọng
| File | Nội dung |
|---|---|
| `/var/log/auth.log` | SSH login, sudo, su — phát hiện brute force |
| `/var/log/syslog` | Log hệ thống tổng hợp |
| `/var/log/nginx/access.log` | Request HTTP đến Nginx |
| `/var/log/nginx/error.log` | Lỗi Nginx |
| `/var/log/mail.log` | Log gửi/nhận mail |

### Lệnh hay dùng
```bash
journalctl -u nginx -f              # xem log Nginx realtime
journalctl --since "1 hour ago"     # lọc theo thời gian
tail -200 /var/log/auth.log         # xem 200 dòng cuối log

# Phát hiện brute force SSH
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn

# Ai đã đăng nhập gần đây
last
lastb   # đăng nhập thất bại
```

### Công cụ bảo mật
- **logrotate**: Tự động xoay vòng log, tránh đầy disk — config tại `/etc/logrotate.d/`
- **fail2ban**: Đọc log, tự động block IP có hành vi bất thường (brute force SSH, HTTP...)
- **rsyslog / syslog-ng**: Log daemon — rsyslog phổ biến hơn, syslog-ng linh hoạt hơn cho môi trường enterprise

---

## 3. FIREWALL

### Kiến trúc firewall Linux
```
Kernel (Netfilter) → iptables / nftables (low-level) → UFW / Firewalld (frontend)
```

### iptables — Chain và Table
| Chain | Áp dụng cho |
|---|---|
| `INPUT` | Traffic đi vào server |
| `OUTPUT` | Traffic đi ra từ server |
| `FORWARD` | Traffic đi qua server (khi làm router/gateway) |

**Target hành động:**
- `ACCEPT`: Cho đi qua
- `DROP`: Loại bỏ, không phản hồi
- `REJECT`: Loại bỏ, có phản hồi (reset/unreachable)
- `LOG`: Ghi log

```bash
# Chặn một IP cụ thể
iptables -A INPUT -s 1.2.3.4 -j DROP

# Cho phép port 443
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Lưu rule vĩnh viễn
iptables-save > /etc/iptables/rules.v4
```

### UFW (Ubuntu)
```bash
ufw allow 443/tcp
ufw deny 21/tcp
ufw status verbose
ufw enable
```

### Stateful vs Stateless
- **Stateless**: Kiểm tra từng packet riêng lẻ theo rule
- **Stateful**: Theo dõi trạng thái kết nối, thông minh và bảo mật hơn

### tcpdump
```bash
tcpdump -i eth0 port 80           # bắt HTTP
tcpdump -i eth0 host 192.168.1.1  # bắt từ/đến IP
tcpdump -i eth0 -w capture.pcap   # ghi ra file, mở bằng Wireshark
```

---

## 4. WEB SERVER

### Apache vs Nginx
| | Apache | Nginx |
|---|---|---|
| Mô hình | Multi-process/thread | Event-driven (async) |
| Tải cao | Tốn RAM hơn | Hiệu năng tốt hơn |
| File tĩnh | Trung bình | Rất tốt |
| Config | `/etc/apache2/` | `/etc/nginx/` |
| Virtual host | `/etc/apache2/sites-available/` | `/etc/nginx/sites-available/` |

### HTTP Status Code
| Code | Ý nghĩa |
|---|---|
| 200 | OK |
| 301 | Redirect vĩnh viễn |
| 302 | Redirect tạm thời |
| 403 | Forbidden — không có quyền |
| 404 | Not Found |
| 500 | Internal Server Error |
| 502 | Bad Gateway — backend không phản hồi |
| 503 | Service Unavailable — server quá tải |

### Các tính năng quan trọng
- **Virtual host**: Host nhiều website trên 1 server — mỗi domain 1 server block
- **Reverse proxy**: Nginx đứng trước nhận request, forward về backend (Apache, Node.js...)
- **Load balancing**: Nginx phân phối request đến nhiều backend — thuật toán: round-robin, least_conn, ip_hash
- **Caching**: `proxy_cache_path` — trả response từ cache thay vì gọi backend mỗi lần

```bash
# Kiểm tra syntax Nginx
nginx -t

# Reload config không downtime
nginx -s reload

# Xem access log realtime
tail -f /var/log/nginx/access.log
```

### Bảo mật Web Server
- **Rate limiting**: `limit_req_zone` trong Nginx — giới hạn request/giây
- **Connection limiting**: `limit_conn_zone`
- **fail2ban**: Block IP brute force tự động
- **ModSecurity**: WAF — lọc SQLi, XSS, LFI

---

## 5. FTP SERVER

### Active Mode vs Passive Mode
| | Active Mode | Passive Mode |
|---|---|---|
| Kết nối data | Server kết nối ngược về client | Client kết nối đến server |
| Port | Server: 20 → Client: random | Server mở port random, client kết nối vào |
| Firewall | Client khó nhận kết nối từ ngoài | Dễ qua NAT/firewall hơn |

**Port 21**: Control connection (lệnh)
**Port 20**: Data connection (Active mode)

### So sánh giao thức
| Giao thức | Port | Mã hóa | Kênh |
|---|---|---|---|
| FTP | 21/20 | Không | 2 kênh |
| FTPS | 990/989 | SSL/TLS trên FTP | 2 kênh (vẫn khó qua NAT) |
| SFTP | 22 | SSH toàn bộ | 1 kênh |
| SCP | 22 | SSH | 1 kênh, nhanh hơn SFTP |

**Khuyến nghị**: Dùng **SFTP** cho môi trường production — bảo mật, đơn giản, qua firewall dễ

### vsftpd — cấu hình quan trọng
```bash
# /etc/vsftpd.conf
anonymous_enable=NO         # tắt anonymous
local_enable=YES            # cho phép local user
write_enable=YES            # cho phép upload
chroot_local_user=YES       # giới hạn user trong /home của họ
pasv_enable=YES             # bật passive mode
pasv_min_port=40000         # dải port passive
pasv_max_port=50000
```

---

## 6. DATABASE SERVER

### So sánh MySQL, MariaDB, Redis
| | MySQL/MariaDB | Redis |
|---|---|---|
| Loại | RDBMS (quan hệ) | In-memory Key-Value |
| Lưu trữ | Disk | RAM |
| Tốc độ | Chậm hơn | Cực nhanh |
| Dùng khi | Dữ liệu có quan hệ, cần ACID | Cache, session, queue |

### InnoDB vs MyISAM
| | InnoDB | MyISAM |
|---|---|---|
| Transaction | Có | Không |
| Foreign key | Có | Không |
| Lock | Row-level | Table-level |
| Khuyến nghị | ✅ Production | Chỉ đọc nhiều |

### Backup/Restore MySQL
```bash
# Backup một database
mysqldump -u root -p dbname > backup.sql

# Backup toàn bộ server
mysqldump -u root -p --all-databases > full_backup.sql

# Restore
mysql -u root -p dbname < backup.sql

# Backup nén
mysqldump -u root -p dbname | gzip > backup.sql.gz
```

### Master-Slave Replication
- Master ghi vào **Binary Log**
- Slave đọc Binary Log từ master qua **IO Thread**, ghi vào **Relay Log**
- Slave áp dụng Relay Log qua **SQL Thread**

### ACID
- **A**tomicity: Tất cả hoặc không có gì
- **C**onsistency: Dữ liệu luôn hợp lệ trước và sau transaction
- **I**solation: Các transaction không ảnh hưởng lẫn nhau
- **D**urability: Dữ liệu được lưu vĩnh viễn sau khi commit

### Bảo mật Database
- Không expose port 3306 ra internet — chỉ cho phép kết nối từ app server
- Principle of Least Privilege — cấp quyền tối thiểu
- Dùng Prepared Statements để chống SQL Injection
- Mã hóa dữ liệu nhạy cảm khi lưu trữ

---

## 7. SSL/TLS

### TLS Handshake (đơn giản hóa)
1. Client gửi `ClientHello` (danh sách cipher hỗ trợ)
2. Server gửi `ServerHello` + Certificate
3. Client xác minh Certificate với CA
4. Trao đổi session key
5. Bắt đầu mã hóa dữ liệu

### Phân loại chứng chỉ
| Loại | Xác minh | Dùng cho |
|---|---|---|
| DV | Chỉ domain | Blog, cá nhân |
| OV | Domain + tổ chức | Doanh nghiệp |
| EV | Xác minh toàn diện | Ngân hàng, tài chính |
| Self-signed | Tự ký | Lab, nội bộ |
| Wildcard `*.domain.com` | Tất cả subdomain | Nhiều subdomain cùng domain |

### File chứng chỉ
- `private.key`: Private key — giữ bí mật tuyệt đối
- `certificate.crt`: Chứng chỉ đã được CA ký
- `ca_bundle.crt`: Intermediate + Root CA chain
- `request.csr`: Yêu cầu cấp chứng chỉ — gửi cho CA

**Với Nginx**: Gộp `certificate.crt` + `ca_bundle.crt` thành một file chain

### Lệnh hay dùng
```bash
# Tạo self-signed certificate
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365

# Tạo CSR
openssl req -new -newkey rsa:2048 -nodes -keyout private.key -out request.csr

# Kiểm tra certificate hết hạn khi nào
openssl x509 -in certificate.crt -noout -dates

# Kiểm tra kết nối SSL
openssl s_client -connect domain.com:443
```

### Các cấu hình bảo mật SSL
- **HSTS**: Buộc trình duyệt dùng HTTPS — `add_header Strict-Transport-Security "max-age=31536000"`
- **OCSP Stapling**: Server chủ động gửi kèm trạng thái chứng chỉ — giảm thời gian verify
- Tắt TLS 1.0/1.1, chỉ dùng TLS 1.2/1.3

---

## 8. EMAIL SERVER (ZIMBRA)

### Kiến trúc Email
- **MTA (Mail Transfer Agent)**: Gửi/nhận mail giữa server — Postfix, Exim
- **MDA (Mail Delivery Agent)**: Đưa mail vào mailbox user — Dovecot
- **MUA (Mail User Agent)**: Giao diện người dùng — Outlook, Thunderbird, Webmail

### Giao thức
| Giao thức | Port | Dùng cho |
|---|---|---|
| SMTP | 25 | Server → Server |
| SMTP Submission | 587 | Client → Server (có auth) |
| IMAP | 143 / 993 (SSL) | Đồng bộ mail với server |
| POP3 | 110 / 995 (SSL) | Tải mail về, xóa trên server |

**IMAP vs POP3**: IMAP đồng bộ nhiều thiết bị, mail vẫn trên server — POP3 tải về một thiết bị

### DNS Record cho Email
| Record | Tác dụng |
|---|---|
| MX | Chỉ server nhận mail |
| SPF (TXT) | Xác định server được phép gửi mail thay mặt domain |
| DKIM (TXT) | Ký điện tử — xác minh mail không bị sửa |
| DMARC (TXT) | Chính sách xử lý khi fail SPF/DKIM: none/quarantine/reject |
| PTR | Reverse DNS — IP → hostname, quan trọng để không bị spam |

### Mail bị spam — kiểm tra gì?
1. SPF/DKIM/DMARC có cấu hình đúng không
2. PTR record có khớp với hostname không
3. IP có bị blacklist không (kiểm tra MXToolbox)
4. Nội dung mail có chứa từ khóa spam không

### Quản lý queue Postfix
```bash
postqueue -p          # xem queue
postqueue -f          # flush, thử gửi lại tất cả
postsuper -d ALL      # xóa toàn bộ queue
```

### Zimbra — thao tác thường dùng
- **Admin Console**: `https://IP:7071`
- **Webmail**: `https://IP`
- **Class of Service (CoS)**: Gom cấu hình (quota, feature) áp dụng cho nhóm user
- Kiểm tra log: `/var/log/zimbra.log`

---

## 9. ZABBIX MONITOR

### Kiến trúc
- **Zabbix Server**: Xử lý trung tâm — polling, alerting, trigger evaluation
- **Zabbix Agent**: Cài trên host được monitor, thu thập metric
- **Zabbix Proxy**: Node trung gian — giảm tải server, monitor mạng khác subnet
- **Zabbix Web**: Giao diện quản trị

### Active vs Passive Check
| | Passive | Active |
|---|---|---|
| Ai kết nối | Server kết nối đến Agent | Agent kết nối đến Server |
| Firewall | Cần mở port từ Server đến Agent | Chỉ cần Agent ra được Server |
| Cấu hình | Mặc định | Thêm `ServerActive=` trong agent config |

### Các khái niệm
- **Item**: Metric cần thu thập (CPU, RAM, disk, process...)
- **Trigger**: Điều kiện cảnh báo dựa trên giá trị item
- **Action**: Hành động khi trigger kích hoạt (gửi email, Telegram, chạy script)
- **Template**: Bộ item + trigger + graph dùng chung — gán vào nhiều host
- **LLD (Low-Level Discovery)**: Tự động phát hiện và tạo item cho đối tượng động (nhiều disk, nhiều interface)
- **Maintenance period**: Tắt cảnh báo trong thời gian bảo trì

### Zabbix HA (Native HA Cluster)
- Tất cả node dùng **chung một database**
- Node `Active`: Đang xử lý toàn bộ
- Node `Standby`: Theo dõi heartbeat của Active node qua database
- Khi Active mất heartbeat vượt `FailoverDelay` (mặc định 1 phút) → Standby tự lên Active
- Kiểm tra trạng thái: **Administration → High Availability** trên Web UI

**Lab thực tế:**
- Node 1 (Active): `192.168.197.148`
- Node 2 (Standby): `192.168.197.149`
- Database: MariaDB trên Node 1, bind-address `0.0.0.0`, grant quyền cho Node 2

---

## 10. AAPANEL

### Giới thiệu
- Control panel mã nguồn mở, miễn phí — phiên bản quốc tế của BaoTa Panel
- Cài LNMP/LAMP stack chỉ vài click

### LNMP Stack
- **L**inux + **N**ginx + **M**ariaDB + **P**HP

### Thao tác cơ bản
- **Thêm website**: Website → Add Site → điền domain, chọn PHP version
- **Gán SSL**: Vào site → tab SSL → Let's Encrypt hoặc upload thủ công
- **Quản lý firewall**: Security → Firewall (thực chất là UFW phía sau)
- **phpMyAdmin**: Database → phpMyAdmin

### aaPanel vs cPanel
| | aaPanel | cPanel |
|---|---|---|
| Chi phí | Miễn phí | Trả phí (license) |
| Phù hợp | VPS cá nhân, SMB | Hosting thương mại |
| Tính năng | Đủ dùng | Đầy đủ hơn |

---

## 11. CLOUDFLARE & CONTROL PANEL

### Cloudflare hoạt động như thế nào?
- Đứng trước server như một **Reverse Proxy**
- Ẩn IP thực của server — chống DDoS, rà quét
- **CDN**: Cache nội dung tĩnh tại Edge Server gần người dùng
- **WAF**: Lọc traffic độc hại (SQLi, XSS, bot)
- **DNS**: Phân giải nhanh, cập nhật gần như ngay lập tức

### Cấu hình Cloudflare
1. Thêm domain vào Cloudflare
2. Cloudflare quét DNS record hiện tại
3. Đổi Nameserver về Cloudflare (tại nhà cung cấp domain)
4. Cấu hình thêm A record, CNAME trên Cloudflare

### DirectAdmin / cPanel — liên quan đến Nhân Hòa
- Control panel cho khách hàng hosting
- Quản lý domain, email, FTP, database, SSL
- Nhân Hòa dùng DirectAdmin và cPanel cho dịch vụ hosting

---

## TỔNG KẾT — CÂU HỎI ĐỂ CHUẨN BỊ

**Khi khách hàng báo website down:**
1. Ping domain/IP — kiểm tra server có phản hồi không
2. `systemctl status nginx` / `apache2` — web server có chạy không
3. Xem `error.log` — có lỗi gì không
4. Kiểm tra CPU/RAM/Disk — có hết tài nguyên không
5. Kiểm tra database — service có chạy không
6. Thông báo cho khách và cập nhật tiến độ xử lý

**Khi mail bị spam:**
1. Kiểm tra SPF/DKIM/DMARC record
2. Kiểm tra PTR record
3. Tra cứu IP blacklist trên MXToolbox
4. Xem log mail để tìm nguyên nhân

**Phân biệt shared hosting vs VPS:**
- Shared hosting: Nhiều khách dùng chung server, tài nguyên bị giới hạn, giá rẻ, phù hợp website nhỏ
- VPS: Môi trường ảo hóa riêng, tài nguyên cố định, toàn quyền quản trị, phù hợp khi cần hiệu năng và tùy biến
