## BÁO CÁO THỰC TẬP 23/6

I. Vận hành & Quản trị
Email server là gì ?
Hiểu một cách đơn giản, email server là gì? Nó hoạt động giống như một "bưu điện kỹ thuật số". Trước khi thư của bạn đến tay đối tác, nó bắt buộc phải đi qua bưu điện này để kiểm duyệt, phân loại và chuyển phát đúng địa chỉ.

1. MTA (Mail Transfer Agent) - Bưu điện trung tâm
MTA chịu trách nhiệm nhận email từ người dùng (hoặc từ máy chủ khác) và định tuyến nó đến đích.

Postfix: Đây là MTA phổ biến nhất hiện nay trên các máy chủ Linux. Postfix được thiết kế với kiến trúc module hóa để thay thế Sendmail. Ưu điểm lớn nhất của nó là bảo mật cao, hiệu năng tuyệt vời và cấu hình tương đối dễ hiểu. Nó là lựa chọn mặc định cho hầu hết các hệ thống email tự xây dựng ngày nay.

Exim: Rất phổ biến trên các hệ thống Shared Hosting (đặc biệt là máy chủ cài cPanel). Điểm mạnh của Exim là khả năng tùy biến bộ định tuyến (routing) cực kỳ mạnh mẽ thông qua các tệp cấu hình linh hoạt, nhưng cú pháp cấu hình của nó phức tạp hơn Postfix.

Sendmail: Là MTA "đồ cổ" và là tiêu chuẩn của internet những năm 90. Nó sử dụng kiến trúc nguyên khối (monolithic) và cấu hình qua các tệp macro m4 cực kỳ phức tạp. Hiện nay, Sendmail ít được sử dụng cho các hệ thống mới vì lý do bảo mật và sự rườm rà.

qmail: Từng là một tượng đài về bảo mật. Tác giả của qmail từng treo thưởng cho bất kỳ ai tìm được lỗ hổng bảo mật của nó. Nó chia nhỏ các tiến trình để giảm thiểu rủi ro. Tuy nhiên, qmail hiện tại đã ngừng phát triển tích cực và thiếu sự hỗ trợ cho các tiêu chuẩn hiện đại (như IPv6 hay TLS mới) trừ khi dùng các bản vá (patch) từ cộng đồng.

2. MDA (Mail Delivery Agent) - Người đưa thư nội bộ
Sau khi MTA nhận được thư đến, nó sẽ giao cho MDA để phân loại và thả vào đúng thư mục (Inbox, Spam...) của người dùng trên ổ cứng. MDA cũng thường kiêm luôn vai trò cung cấp dịch vụ IMAP/POP3 để người dùng kết nối vào đọc thư.

Dovecot: Là MDA "vua" ở thời điểm hiện tại. Dovecot cực kỳ nhẹ, bảo mật, hiệu năng cao và hỗ trợ hoàn hảo các định dạng lưu trữ thư (Maildir, mbox). Nó tương thích tuyệt vời với Postfix.

Cyrus: Được thiết kế cho các doanh nghiệp lớn với kiến trúc "hộp đen" (sealed-server). Người quản trị không thể can thiệp trực tiếp vào file lưu trữ email bằng lệnh Linux thông thường mà phải thông qua các công cụ riêng của Cyrus. Nó rất mạnh trong việc quản lý hòm thư khổng lồ, nhưng quá phức tạp cho các hệ thống vừa và nhỏ.

3. Webmail - Giao diện người dùng
Đây thực chất là một MUA (Mail User Agent) chạy trên nền tảng web, kết nối với MDA (qua IMAP) và MTA (qua SMTP) ở backend.

Roundcube: Giao diện webmail hiện đại, dùng AJAX nên mang lại trải nghiệm mượt mà giống như một ứng dụng trên máy tính. Nó tập trung thuần túy vào việc gửi/nhận email và là giao diện mặc định được yêu thích nhất.

Horde: Không chỉ là webmail mà là một "Groupware" (phần mềm cộng tác). Ngoài email, nó đi kèm lịch (Calendar), danh bạ (Contacts), ghi chú (Notes) và quản lý công việc (Tasks). Giao diện của Horde có phần cũ và cồng kềnh hơn Roundcube, phù hợp nếu bạn cần một giải pháp thay thế Microsoft Exchange.

4. Anti-Spam / Anti-Virus - Trạm kiểm soát an ninh
Để ngăn chặn thư rác và mã độc, hệ thống cần các bộ lọc quét nội dung liên tục.

SpamAssassin: Hệ thống chống thư rác dựa trên điểm số (scoring). Nó phân tích tiêu đề, nội dung email, kiểm tra các từ khóa đáng ngờ, và tra cứu các danh sách đen (DNSBL/RBL). Nếu điểm số vượt quá một ngưỡng nhất định (ví dụ: > 5.0), thư sẽ bị đánh dấu là Spam.

ClamAV: Một engine diệt virus mã nguồn mở. Nó sẽ quét các tệp đính kèm trong email (như PDF, file ZIP, file thực thi) để tìm kiếm malware, trojan hoặc virus trước khi cho phép email đi tiếp.

Amavis (amavisd-new): Đây không phải là phần mềm diệt virus hay spam, mà là phần mềm trung gian (middleware). Postfix không tự nói chuyện được với SpamAssassin và ClamAV; nó đẩy thư cho Amavis. Amavis sẽ "gọi" SpamAssassin và ClamAV để quét thư, tổng hợp kết quả và trả lại lệnh cho Postfix (chấp nhận, từ chối, hoặc cách ly thư).

5. Các bản ghi DNS
Việc cài đặt phần mềm xong chưa đủ, thế giới Internet cần xác minh máy chủ của bạn qua hệ thống tên miền (DNS).

MX (Mail Exchanger): Bản ghi nền tảng nhất. Nó chỉ đường cho các máy chủ khác biết: "Nếu muốn gửi email cho @tenmien.com, hãy gửi đến địa chỉ IP của máy chủ này". Bạn có thể có nhiều bản ghi MX với mức độ ưu tiên (Priority) khác nhau để dự phòng.

PTR (Pointer Record / Reverse DNS): Ngược lại với bản ghi A (từ tên miền ra IP), PTR dịch từ IP ra tên miền. Khi máy chủ của bạn gửi thư đi, máy chủ nhận sẽ kiểm tra xem địa chỉ IP của bạn có thực sự khớp với tên miền máy chủ (Hostname) hay không. Nếu không có PTR, 99% email của bạn sẽ bị Gmail/Yahoo từ chối. Bản ghi này thường do nhà cung cấp máy chủ (VPS/Cloud) cấu hình cho bạn.

SPF (Sender Policy Framework): Một bản ghi TXT công bố danh sách các địa chỉ IP được phép gửi email bằng tên miền của bạn.

Ví dụ: v=spf1 ip4:192.168.1.100 -all (Chỉ IP 192.168.1.100 mới được gửi email cho tên miền này, các IP khác sẽ bị từ chối).

DKIM (DomainKeys Identified Mail): Bản ghi TXT chứa một khóa công khai (Public Key). Mỗi khi máy chủ của bạn gửi thư đi, nó sẽ dùng khóa bí mật (Private Key) để ký điện tử vào bức thư. Máy chủ nhận sẽ dùng khóa công khai trên DNS để giải mã. Việc này đảm bảo thư không bị hacker chỉnh sửa nội dung trên đường truyền.

DMARC (Domain-based Message Authentication): Kẻ gác cổng tối cao. Nó dựa vào kết quả của SPF và DKIM để ra lệnh cho máy chủ nhận. DMARC định nghĩa chính sách: "Nếu một email giả mạo tên miền của tôi (thất bại SPF hoặc DKIM), hãy Từ chối nó (reject), hoặc Đưa vào thư mục Spam (quarantine)". Nó cũng gửi báo cáo định kỳ về cho bạn để bạn biết ai đang cố tình mạo danh tên miền của mình.

SRV (Service Record): Bản ghi này không giúp chống spam, mà giúp trải nghiệm người dùng tốt hơn. Nó chứa thông tin tự động cấu hình (Auto-discover). Khi người dùng nhập email vào Outlook hoặc Thunderbird, ứng dụng sẽ tự tìm bản ghi SRV để biết chính xác cổng IMAP là 993, SMTP là 465 mà người dùng không cần gõ tay.

Quản trị một hệ thống email server không chỉ dừng lại ở việc cài đặt xong là chạy. Khâu giám sát (Monitor) và xử lý sự cố (Troubleshoot) thông qua Logs và Mail Queue mới là công việc thường ngày của một System Admin.

Dưới đây là chi tiết về cách "khám bệnh" và điều phối hệ thống email của bạn.

### 2. Logs & Giám sát hệ thống 

Bất kỳ thao tác nào từ việc có người cố gắng đăng nhập sai mật khẩu, một email gửi đi thành công, hay một tệp đính kèm bị chặn lại vì có virus, đều được ghi lại cẩn thận.

* **Logs gửi/nhận mail:** Đây là nơi bạn tra cứu hành trình của một bức thư.
* Trên các bản phân phối như Ubuntu/Debian, file này thường nằm ở `/var/log/mail.log`. Với CentOS/RHEL, nó là `/var/log/maillog`.
* Mỗi dòng log sẽ chứa mã ID của bức thư (Message-ID), địa chỉ IP nguồn/đích, và trạng thái (status=sent, deferred, hoặc bounced).


* **Logs authentication (Xác thực người dùng):** Rất quan trọng để phát hiện các cuộc tấn công Brute-force (dò mật khẩu).
* Đường dẫn phổ biến: `/var/log/auth.log` (Ubuntu) hoặc `/var/log/secure` (CentOS).
* Nếu bạn thấy hàng loạt IP lạ liên tục báo lỗi "Failed password", máy chủ đang bị tấn công.


* **Logs truy cập IMAP/POP (Dovecot):** Giám sát việc người dùng kéo thư về thiết bị của họ. Dovecot thường bắn log chung vào `mail.log`, nhưng bạn có thể cấu hình tách ra file riêng để dễ quản lý.
* **Logs chống spam/virus:** * Các công cụ như ClamAV hay SpamAssassin ghi lại nguyên nhân tại sao một email bị đánh dấu là Spam (ví dụ: điểm số bay lên mức 7.5 do chứa link blacklist).

**Công cụ tra cứu và giám sát Logs:**
Thay vì mở thủ công các file text khổng lồ, quản trị viên sử dụng các công cụ mạnh mẽ:

* **Lệnh CLI cơ bản:** * `grep "user@domain.com" /var/log/mail.log` (Tìm mọi giao dịch của một user).
* `awk` để lọc các cột dữ liệu cụ thể (như lọc riêng địa chỉ IP).
* `zgrep` để tìm kiếm trực tiếp trong các file log cũ đã được nén (`.gz`) mà không cần giải nén.
* `journalctl -u postfix -f` hoặc `journalctl -u dovecot -f` để theo dõi log theo thời gian thực trực tiếp từ systemd.


* **Hệ thống giám sát tập trung (Centralized Logging):**
* Khi hệ thống lớn lên, việc dùng **ELK Stack (Elasticsearch, Logstash, Kibana)** là giải pháp tối ưu. Logstash sẽ thu thập `mail.log`, phân tích cú pháp, và đẩy lên Elasticsearch. Sau đó, bạn có thể dùng Kibana để vẽ biểu đồ trực quan (Dashboard), ví dụ như biểu đồ hiển thị lượng email Spam bị chặn trong ngày hoặc top IP đang spam máy chủ.



### 3. Quản lý Queue Mail 

**Queue (Hàng đợi)** là nơi lưu trữ tạm thời các email đang chờ xử lý. Nếu máy chủ đích bị sập, hoặc mạng bị đứt, MTA của bạn không vứt email đó đi mà sẽ đưa vào queue "deferred" (trì hoãn) để thử gửi lại sau.

**Cách kiểm tra Queue:**

* **Postfix:** Dùng lệnh `postqueue -p` hoặc `mailq`. Lệnh này sẽ in ra danh sách các email đang kẹt, kèm theo Queue ID, kích thước, người gửi, và lý do bị kẹt (ví dụ: *Connection timed out*).
* **Exim:** Dùng lệnh `exim -bp`.
* **qmail:** Dùng lệnh `qmail-qstat` (chỉ xem số lượng) hoặc các công cụ phụ trợ để xem chi tiết.

**Xử lý thư bị kẹt (Mail Stuck):**
Đôi khi, một tài khoản bị hack và spam hàng nghìn email ra ngoài, làm đầy cứng Queue khiến email hợp lệ không thể gửi đi (gây nghẽn cổ chai).

* **Postfix:** * Xóa một thư cụ thể: `postsuper -d <Queue_ID>`
* Xóa toàn bộ hàng đợi (Cẩn thận!): `postsuper -d ALL`
* Ép gửi lại ngay lập tức (thay vì chờ đến lượt): `postqueue -f`


* **Exim:**
* Xóa thư: `exim -Mrm <Message_ID>`
* Xóa toàn bộ: `exim -bp | awk '/^ *[0-9]+[mhd]/{print "exim -Mrm " $3}' | bash`



**Công cụ giám sát Queue chuyên nghiệp:**

* **pflogsumm:** Một script bằng Perl cực kỳ xuất sắc dành cho Postfix. Nó đọc file `mail.log` và tạo ra một báo cáo tổng kết mỗi ngày (Gửi thành công bao nhiêu, bị từ chối bao nhiêu, user nào gửi nhiều nhất) và gửi thẳng vào email của Admin.
* **Web UI:** Nếu không muốn dùng giao diện dòng lệnh, bạn có thể tích hợp **Postfix Admin** hoặc **MailWatch**. Chúng cung cấp giao diện web trực quan để bạn nhìn thấy Queue đang có gì và click chuột để xóa/gửi lại thư kẹt dễ dàng.





