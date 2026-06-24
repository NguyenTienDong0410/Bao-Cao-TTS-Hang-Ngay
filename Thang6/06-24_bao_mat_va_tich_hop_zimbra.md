# BÁO CÁO THỰC TẬP 24/6

---

## II. BẢO MẬT & AUDIT (SECURITY & AUDITING)

### 4. Xác thực & Chống giả mạo (Anti-Spoofing Deep-Dive)
Cần hiểu rõ: SMTP ban đầu là một giao thức "ngây thơ", ai cũng có thể khai báo `From: billgates@microsoft.com`. Để khắc phục, Internet sinh ra 3 "lá chắn", và Zimbra triển khai chúng rất chặt chẽ.

*   **SPF (Sender Policy Framework - Chống giả mạo IP):**
    *   *Cơ chế hoạt động:* SPF kiểm tra địa chỉ **Envelope Sender** (giai đoạn `MAIL FROM` trong phiên SMTP), không phải trường `From:` hiển thị cho người dùng. Khi Zimbra nhận thư, công cụ `policyd-spf` hoặc SpamAssassin sẽ phân giải DNS của domain người gửi, tìm record TXT để xem IP hiện tại có nằm trong danh sách cho phép không.
    *   *Zimbra config:* Trong Zimbra, SpamAssassin được cấu hình mặc định để cộng/trừ điểm (Score) cực mạnh nếu SPF pass/fail.
*   **DKIM (DomainKeys Identified Mail - Chống sửa đổi nội dung):**
    *   *Cơ chế hoạt động:* Dựa trên mã hóa phi đối xứng. Máy chủ gửi dùng Private Key băm (hash) toàn bộ nội dung + tiêu đề email và nhét vào header `DKIM-Signature`. Máy chủ nhận dùng Public Key (lấy từ DNS) để giải mã và so khớp chuỗi hash. Chỉ cần 1 dấu phẩy bị thay đổi trên đường truyền, chữ ký sẽ bị phá vỡ (Fail).
    *   *Zimbra triển khai:* Zimbra tích hợp sẵn module DKIM. Khi tạo key (`zmdkimkeyutil`), Zimbra lưu Private Key vào LDAP. Khi Postfix đẩy mail ra ngoài, nó đưa qua Amavis. Amavis đọc Private Key từ LDAP và tự động chèn header DKIM vào mail trước khi phát đi.
*   **DMARC (Domain-based Message Authentication, Reporting, and Conformance):**
    *   *Cơ chế hoạt động:* SPF và DKIM có lỗ hổng là kẻ gian vẫn có thể dùng thủ thuật làm sai lệch giữa "Envelope Sender" và "Header From". DMARC ra đời để ép 2 giá trị này phải **Alignment (Khớp nhau)**. DMARC cũng cho phép chủ domain nhận các báo cáo (rua/ruf) về tình trạng giả mạo domain của mình.
    *   *Zimbra triển khai:* Từ bản Zimbra 8.8.x, **OpenDMARC** được tích hợp sẵn. Zimbra sẽ đọc policy của DMARC (`p=reject`, `p=quarantine`) và quyết định xem có nên đá mail giả mạo ngay từ cửa Postfix (trả lỗi 550) hay cho vào thư mục Junk.
*   **SMTP AUTH & SASL (Xác thực người gửi nội bộ):**
    *   *Mục đích:* Để chống Open-Relay (bị lợi dụng làm trạm trung chuyển thư rác), Postfix yêu cầu client phải xác thực trước khi gửi ra ngoài.
    *   *Zimbra triển khai:* Postfix không chứa mật khẩu. Khi client kết nối Port 587 và gõ user/pass, Postfix đưa thông tin này cho tiến trình **Cyrus-SASL** (`saslauthd`). `saslauthd` trong Zimbra được lập trình để query trực tiếp vào **Zimbra LDAP**. Nếu LDAP báo OK, Postfix mới cấp quyền `Relay Access Granted`.

### 5. Mã hóa kết nối (Encryption Mechanics)
*   **Cơ chế TLS/SSL:** Giúp vô hiệu hóa các cuộc tấn công bắt gói tin (Packet Sniffing) trên mạng LAN/Wi-Fi công cộng.
*   **Phân biệt Port 465 và 587:**
    *   `Port 465 (Implicit SSL)`: Vừa mở kết nối là bắt tay TLS (Handshake) ngay lập tức. Client/Server nói chuyện bằng mã hóa từ giây đầu tiên.
    *   `Port 587 (Explicit TLS / STARTTLS)`: Mở kết nối bằng dạng chữ rõ (plain-text). Client sẽ gõ lệnh `STARTTLS`. Nếu Zimbra trả lời `220 Ready to start TLS`, hai bên mới bắt đầu sinh khóa và mã hóa.
*   **Kiến trúc mã hóa của Zimbra:**
    *   Zimbra dùng **Nginx** làm Proxy bọc bên ngoài. Mọi chứng chỉ SSL (quản lý qua công cụ `zmcertmgr`) đều được cài lên Nginx.
    *   Nginx đứng ra chịu tải mã hóa SSL từ Client. Sau đó, Nginx kết nối ngược vào Postfix/Mailboxd ở vòng trong bằng plain-text (HTTP/IMAP) để tiết kiệm CPU cho server lõi.
*   **Kiểm tra và Debug:**
    *   Dùng lệnh `openssl s_client -connect mail.domain.com:465 -servername mail.domain.com` để xem Nginx trả về chứng chỉ của domain nào, dùng thuật toán mã hóa (Cipher Suite) nào.

### 6. Audit & Theo dõi hệ thống (System Auditing)
*   **Ai đăng nhập, từ đâu (Truy vết người dùng):**
    *   Mọi tương tác đăng nhập Webmail, IMAP, POP3, ActiveSync đều được Java Mailboxd ghi cực kỳ rõ ràng vào file `/opt/zimbra/log/audit.log`. Bạn sẽ thấy các dòng `INFO [qtp...] [ip=1.2.3.4;] security - cmd=Auth; account=user@domain; protocol=imap; status=oap;`.
*   **Ai gửi nhiều mail bất thường (Chống tài khoản bị hack):**
    *   Một tài khoản bị lộ pass có thể gửi 10.000 mail rác/giờ, làm IP server bị liệt vào Blacklist.
    *   Zimbra sử dụng module **cbpolicyd (Policy Daemon)**. Bạn có thể set rule: "Một tài khoản chỉ được gửi tối đa 100 mail / 1 giờ". Vượt quá, Postfix sẽ tự drop.
*   **Tích hợp Fail2ban chặn IP Brute-force:**
    *   *Vấn đề:* Kẻ tấn công dùng Tool thử hàng ngàn mật khẩu/phút. Nếu chỉ ghi log thì server vẫn kiệt sức.
    *   *Triển khai:* Cài đặt Fail2ban, viết Regex (biểu thức chính quy) để rình file `audit.log`. Nếu dòng `security - cmd=Auth; ... status=error` xuất hiện 5 lần từ 1 IP trong 10 phút, Fail2ban sẽ gọi `iptables` hoặc `firewalld` khóa IP đó 24 giờ.

---

## III. TÍCH HỢP & HƯỚNG DẪN NGƯỜI DÙNG

### 7. Tích hợp 3rd-party Client (Outlook, Apple Mail, Mobile)
*   **Giao thức IMAP vs POP3 ở cấp độ hệ thống:**
    *   *IMAP:* Đồng bộ cờ trạng thái (Flags) như `\Seen` (đã đọc), `\Deleted` (đã xóa), `\Answered` (đã trả lời). Zimbra xử lý IMAP qua Nginx Proxy.
    *   *ActiveSync (EAS):* Giao thức của Microsoft, được Zimbra Network Edition hoặc Z-Push (trên bản FOSS) hỗ trợ. Nó "Push" mail tức thời (Real-time) và đồng bộ cả Lịch, Danh bạ xuống Native App của điện thoại mà không cần thiết lập nhiều.
*   **Tự động cấu hình (Autodiscover & Autoconfig):**
    *   *Cơ chế:* Thay vì bắt người dùng nhớ "Máy chủ thư đến là gì, cổng bao nhiêu", hệ thống cung cấp 1 file XML.
    *   *Zimbra hoạt động:* Khi User nhập mail vào Outlook, Outlook tự ngầm gửi 1 request HTTP `POST` đến `https://autodiscover.domain.com/autodiscover/autodiscover.xml`. Nginx của Zimbra sẽ bắt request này, chuyển cho Mailboxd xử lý. Mailboxd trả về file XML chứa toàn bộ thông số Port 993/587 và báo cho Outlook tự động setup. Người dùng chỉ cần gõ Email và Password.

### 8. Hướng dẫn sử dụng & Logic bên dưới hệ thống
*   **Bộ lọc thư (Filters - Sieve scripts):**
    *   *Cơ chế:* Khi người dùng tạo rule "Nếu tiêu đề có chữ Khuyến mãi thì bỏ vào Thư rác", Zimbra sử dụng ngôn ngữ **Sieve**. Kịch bản Sieve này chạy trực tiếp trên server (Mailboxd) ở khoảnh khắc mail vừa được nhận, đảm bảo mail được di chuyển dẫu người dùng có đang tắt máy tính.
*   **Out-of-office (Autoresponder):**
    *   Cũng được xử lý bởi Sieve. Mailboxd sẽ tự động sinh ra một mail hồi đáp và đưa ngược lại cho Postfix gửi đi. Nó thông minh ở chỗ nó ghi nhớ IP/Email người gửi để không gửi phản hồi "Tôi đi vắng" quá 1 lần/ngày cho cùng 1 người, tránh vòng lặp mail (Mail Loop).
*   **Quota (Dung lượng):**
    *   Zimbra lưu hạn mức (Quota) trong LDAP và lưu dung lượng thực tế đang dùng trong MariaDB. Lệnh check nhanh của Admin: `zmprov gqu server.domain.com`.

### 9. Các dịch vụ bổ trợ
*   **Lịch & Danh bạ (CalDAV / CardDAV):**
    *   Zimbra thực tế cũng đóng vai trò là 1 Web Server. CalDAV/CardDAV chạy trên nền HTTP/WebDAV (Port 443). Lịch được lưu dưới dạng file `.ics` (iCalendar), danh bạ lưu dạng `.vcf` (vCard) bên trong database của Zimbra. Thiết bị (iPhone/Mac) kết nối vào URL `/dav` của Zimbra để lấy dữ liệu.
*   **Rspamd (Anti-spam thế hệ mới):**
    *   Trong môi trường lớn, Amavis/SpamAssassin bằng Perl khá nặng và chậm. Xu hướng hiện nay (và Zimbra 10 hỗ trợ) là thay thế bằng **Rspamd**. Rspamd viết bằng C, dùng Redis để cache, có Web UI siêu đẹp, tốc độ quét mail nhanh gấp nhiều lần và nhận diện spam chính xác hơn nhờ Machine Learning.

---

## IV. KHÁC (KIẾN TRÚC MỞ RỘNG & VẬN HÀNH NÂNG CAO)

Phần này thể hiện sức mạnh thực sự của Zimbra so với các hệ thống Mail tự build nhỏ lẻ.

### 10. Kiến trúc Cluster & High Availability (HA)
Hệ thống Zimbra có thể tách rời các dịch vụ để chịu lỗi (Fault Tolerance) và chia tải (Load Balancing).
*   **Mô hình phổ biến (Zimbra Multi-Server):**
    *   **LDAP Node:** 1 Server chạy LDAP Master. Có thể cắm thêm LDAP Replica để nếu Master chết, các server khác vẫn có chỗ xác thực User.
    *   **MTA Node (Postfix/Amavis):** 2 Server đứng sau Load Balancer. Chúng không giữ dữ liệu. Cứ có mail đến, Load Balancer rải đều cho 2 MTA này lọc Spam/Virus.
    *   **Proxy Node (Nginx):** Đứng trước hướng Internet. Nó hứng traffic Webmail (HTTPS), IMAP/POP3.
    *   **Mailbox Node (Dữ liệu thật):** Nhiều server khác nhau. *Điểm thông minh của Zimbra:* Nginx liên tục hỏi dịch vụ **Memcached** (Zimbra Route Lookup) để biết "User A đang nằm ở Mailbox Node 1 hay Node 2", từ đó Nginx sẽ điều hướng traffic (Proxy) của User A vào đúng Node chứa dữ liệu của họ.

### 11. Backup & Restore (Chiến lược sao lưu)
*   **Backup:**
    *   Bản quyền Zimbra Network Edition có tính năng **Real-Time Backup (Redo logs)**. Mọi thao tác click chuột/nhận mail đều được ghi log theo thời gian thực, có thể khôi phục trạng thái server về chính xác từng giây.
    *   Bản Zimbra FOSS (Miễn phí) không có tính năng này. Admin thường phải viết shell script chạy cronjob dùng `zmsimbra` (công cụ mã nguồn mở) hoặc `rsync /opt/zimbra` và snapshot máy ảo định kỳ.
*   **Migration (Chuyển đổi với Zero-Downtime):**
    *   Công cụ thần thánh: **imapsync**. Khi chuyển từ cPanel/Exchange sang Zimbra, `imapsync` sẽ kết nối bằng tài khoản IMAP của cả 2 server, copy toàn bộ cấu trúc thư mục và trạng thái mail (đã đọc/chưa đọc) sang Zimbra.
    *   Kỹ năng khó nhất là việc đồng bộ DNS (giảm TTL của MX record) và định tuyến song song để đảm bảo trong thời gian di dời (có thể mất nhiều ngày cho dữ liệu lớn), không một email nào của đối tác bị từ chối.
*   **Cách imapsync hoạt động:** Nó không can thiệp vào Database. Nó hoạt động như 1 Email Client, đọc thư ở bên này và "kéo-thả" (Append) sang bên kia, giữ nguyên các lá cờ (Đã đọc, Quan trọng). Do đó nó cực kỳ an toàn và dùng được cho mọi loại Mail Server (Exchange, M365, Google Workspace -> Zimbra).
