II. Bảo mật & Audit
4. Xác thực & chống giả mạo
SPF, DKIM, DMARC: Cách tạo, kiểm tra và debug.
SMTP AUTH: Xác thực người gửi.
SASL: Giao thức hỗ trợ SMTP AUTH.
5. Mã hóa
TLS/SSL cho SMTP, IMAP, POP.
Kiểm tra bằng openssl s_client, testssl.sh.
6. Audit & theo dõi hoạt động
Ai đăng nhập, đăng xuất, từ đâu.
Ai gửi nhiều mail bất thường.
Theo dõi file log, alert qua Zabbix, Grafana, ELK.
Tích hợp fail2ban để chặn IP brute-force.
III. Tích hợp & Hướng dẫn người dùng
7. Cài đặt email trên ứng dụng 3rd-party
Outlook, Thunderbird, Apple Mail, điện thoại Android/iOS.
IMAP/POP/SMTP port & SSL settings.
Tự động cấu hình: Autodiscover, autoconfig.xml.
8. Hướng dẫn sử dụng
Tạo chữ ký, lọc thư, forward.
Thiết lập out-of-office reply.
Giải thích quota mailbox.
9. Các dịch vụ bổ trợ
Webmail: Roundcube, RainLoop.
Lịch (calendar) & danh bạ (CardDAV, CalDAV).
Anti-spam nâng cao: Rspamd, MailScanner.
IV. Khác (nâng cao)
Cluster mail server.
High availability (HA).
Backup & restore mailbox.
Migration (chuyển mail server hoặc domain).
