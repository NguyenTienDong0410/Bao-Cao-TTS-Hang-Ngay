# 🚀 System Engineer Internship Workspace - Nhan Hoa Group

Chào mừng đến với Repository lưu trữ toàn bộ tài liệu, script và tiến độ thực tập của mình tại **Nhân Hòa Group**. Repo này đóng vai trò như một cuốn nhật ký kỹ thuật, ghi lại quá trình tìm hiểu, cấu hình và triển khai các hệ thống từ cơ bản đến nâng cao dưới sự hướng dẫn của các mentor.

**👨‍💻 Thực tập sinh:** Nguyễn Tiến Đông  
**🏢 Vị trí:** System Engineer Intern  
**📅 Thời gian báo cáo hàng ngày:** 17h30 
**📩 Mentors:** `duydm@`, `lampk@`, `annt@`, `anvt@`, `chinhtran@` (nhanhoa.com.vn)

---

## 🛠️ Công cụ & Môi trường làm việc
- **Giao tiếp & Báo cáo:** Telegram, Zalo, Gmail, GitHub (Markdown).
- **Remote & SSH:** PuTTY, SecureCRT, MobaXterm, Remote Desktop Manager.
- **Ảo hóa & Lab:** VMware Workstation.
- **Thiết kế & Document:** Visio, Draw.io, LightShot.

---

## 📚 Lộ trình & Nội dung thực hành (Lab Modules)

Dưới đây là chi tiết các hạng mục công việc và kiến thức đã hoàn thiện trong quá trình thực tập:

### 📍 Module 1: Quản trị Hệ điều hành & Lưu trữ (Linux/Windows)
- [x] **Linux Foundation:** Kiến trúc OS, các distro phổ biến, hệ thống phân quyền (rwx, Sticky bit).
- [x] **CLI & Tools:** Bash shell, Nano, Vi, quản lý tiến trình, thao tác với log cơ bản, CLI (`grep`, `awk`).
- [x] **LVM (Logical Volume Manager):** - Khởi tạo, mở rộng, xóa `pv`, `vg`, `lv`.
  - Cấu hình Filesystem và tự động mount qua `/etc/fstab`.
  - Thay thế/Loại bỏ ổ cứng vật lý khỏi hệ thống LVM an toàn.
- [x] **NFS (Network File System):** Cài đặt NFS Server và cấu hình mount trên NFS Client.
- [x] **Windows Server OS:** Quản trị Event Viewer (lọc Event ID), Task Scheduler, phân quyền NTFS (ngắt kế thừa).

### 📍 Module 2: Triển khai Web Server & Load Balancing
- [x] **Linux Web Stack:**
  - Cài đặt và triển khai **LAMP** / **LEMP** stack.
  - Triển khai site WordPress (Mô hình All-in-one & Mô hình tách node Web/DB).
  - Viết Bash script tự động hóa cài đặt LAMP/LEMP.
- [x] **Windows Web Stack (IIS):**
  - Triển khai đa dạng mã nguồn: HTML basic, ASP classic, .NET (3.5, 4.x), PHP trên IIS.
- [x] **Kiến trúc Web Server Nâng cao:**
  - Nắm vững kiến trúc lõi Apache, Nginx, IIS, LiteSpeed. So sánh bản OSS vs Commercial.
  - Giao thức & Bảo mật: HTTP/2, QUIC, WebSocket, HTTPS (SSL/TLS).
  - Giao tiếp trung gian: CGI, FastCGI, WSGI.
  - Tối ưu hóa: Khởi tạo Virtual Host, Nginx Reverse Proxy cho cụm Apache, cấu hình Load Balancer, nén Gzip, Caching, lấy Real IP từ CDN, tinh chỉnh (Worker, Connection, Keep-alive).

### 📍 Module 3: High Availability (HA) & Monitoring
- [x] **HAProxy & Keepalived:**
  - Tìm hiểu thuật ngữ (Proxy, ACL, Backend, Frontend, Sticky Sessions).
  - Các giải thuật cân bằng tải.
  - Triển khai cụm HAProxy + Keepalived phân tải cho Apache trên Ubuntu 22.04.
- [x] **Monitoring System (Prometheus + Grafana):**
  - Cài đặt Prometheus & Grafana trên Ubuntu 22.04.
  - Viết query Prometheus.
  - Xây dựng Dashboard Grafana giám sát: RAM, CPU, Uptime, Load, MySQL Service.

### 📍 Module 4: Hệ thống Mail Server
- [x] **Zimbra Mail Server (Ubuntu 22.04):**
  - Cài đặt, tạo user, chính sách mật khẩu, chữ ký, forwarding.
  - Quản trị: Tìm ID mailbox, đổi pass admin, phân tích log gửi/nhận, thay đổi logo/title web, cấu hình Quota.
  - Nâng cao: Backup/Restore và Migrate data sang node khác.
- [x] **MDaemon Mail Server (Windows Server):**
  - Cài đặt, quản lý port, tạo Domain/User/Group/Alias/Mailing lists.
  - Phân quyền Admin (Global/Domain).
  - Security: Content Filter (Spam, Antivirus, Attach/Message Filters), Dynamic screening.
  - Đọc log hệ thống và thực hành Backup/Restore.

### 📍 Module 5: Networking & Security
- [x] **Hệ thống DNS:** Nắm vững kiến trúc và các loại bản ghi DNS.
- [x] **Firewall trong Linux/Windows:**
  - Linux: Thực hành cấu hình `UFW`, `Firewalld`, `iptables`, `nftables`, `apf`.
  - Windows: Cấu hình Windows Defender Firewall qua GUI và PowerShell (Inbound/Outbound rules).
- [x] **pfSense Firewall Router:**
  - Cấu hình NAT Local Internet, Firewall Rules, DHCP Server.
  - Setup OpenVPN (chế độ TAP và TUN).
- [x] **Network Analysis & Troubleshooting:** - Cài đặt và thực hành kiểm tra gói tin mạng bằng `tcpdump` và `Wireshark`.
  - Quản lý bản vá hệ thống tự động (APT).

---
*Repo này được cập nhật liên tục hàng ngày để báo cáo tiến độ và lưu trữ các đoạn code/script cấu hình phục vụ cho công việc tại lab Nhân Hòa.*
