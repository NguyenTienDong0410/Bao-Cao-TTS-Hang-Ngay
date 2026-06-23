# BÁO CÁO THỰC TẬP: VẬN HÀNH VÀ QUẢN TRỊ EMAIL SERVER


---

## MỤC LỤC

1. [Tổng quan về Email Server](#1-tổng-quan-về-email-server)
2. [Kiến trúc hệ thống và luồng xử lý email](#2-kiến-trúc-hệ-thống-và-luồng-xử-lý-email)
   - 2.1. [Sơ đồ luồng email từ đầu đến cuối](#21-sơ-đồ-luồng-email-từ-đầu-đến-cuối)
   - 2.2. [Bốn vai trò chức năng chính](#22-bốn-vai-trò-chức-năng-chính)
3. [Các giao thức nền tảng](#3-các-giao-thức-nền-tảng)
   - 3.1. [SMTP - Giao thức gửi thư](#31-smtp---giao-thức-gửi-thư)
   - 3.2. [IMAP và POP3 - Giao thức nhận thư](#32-imap-và-pop3---giao-thức-nhận-thư)
4. [MTA - Mail Transfer Agent (Tác nhân chuyển phát thư)](#4-mta---mail-transfer-agent)
5. [MDA - Mail Delivery Agent (Tác nhân phân phối thư)](#5-mda---mail-delivery-agent)
6. [MUA - Mail User Agent và Webmail (Giao diện người dùng)](#6-mua---mail-user-agent-và-webmail)
7. [Bảo mật tầng truyền tải - TLS / STARTTLS](#7-bảo-mật-tầng-truyền-tải---tls--starttls)
8. [Hệ thống kiểm soát an ninh - Anti-Spam và Anti-Virus](#8-hệ-thống-kiểm-soát-an-ninh---anti-spam-và-anti-virus)
   - 8.1. [SpamAssassin - Cơ chế chấm điểm](#81-spamassassin---cơ-chế-chấm-điểm)
   - 8.2. [ClamAV - Diệt virus tệp đính kèm](#82-clamav---diệt-virus-tệp-đính-kèm)
   - 8.3. [Amavis - Phần mềm trung gian](#83-amavis---phần-mềm-trung-gian)
9. [Các bản ghi DNS quan trọng trong hệ thống Email](#9-các-bản-ghi-dns-quan-trọng)
   - 9.1. [MX Record](#91-mx---mail-exchanger)
   - 9.2. [PTR Record](#92-ptr---pointer-record--reverse-dns)
   - 9.3. [SPF Record](#93-spf---sender-policy-framework)
   - 9.4. [DKIM Record](#94-dkim---domainkeys-identified-mail)
   - 9.5. [DMARC Record](#95-dmarc---domain-based-message-authentication)
   - 9.6. [SRV Record](#96-srv---service-record)
10. [Bảo vệ chống Brute-force với Fail2Ban](#10-bảo-vệ-chống-brute-force-với-fail2ban)
11. [Giám sát hệ thống và phân tích Logs](#11-giám-sát-hệ-thống-và-phân-tích-logs)
    - 11.1. [Phân loại và ý nghĩa các file Logs](#111-phân-loại-và-ý-nghĩa-các-file-logs)
    - 11.2. [Đọc và phân tích một dòng Log thực tế](#112-đọc-và-phân-tích-một-dòng-log-thực-tế)
    - 11.3. [Công cụ CLI tra cứu Logs](#113-công-cụ-cli-tra-cứu-logs)
    - 11.4. [Hệ thống giám sát tập trung - ELK Stack](#114-hệ-thống-giám-sát-tập-trung---elk-stack)
12. [Quản lý hàng đợi thư - Mail Queue](#12-quản-lý-hàng-đợi-thư---mail-queue)
    - 12.1. [Cơ chế và các trạng thái của Mail Queue](#121-cơ-chế-và-các-trạng-thái-của-mail-queue)
    - 12.2. [Kiểm tra và xử lý Queue trên Postfix](#122-kiểm-tra-và-xử-lý-queue-trên-postfix)
    - 12.3. [Kiểm tra và xử lý Queue trên Exim](#123-kiểm-tra-và-xử-lý-queue-trên-exim)
    - 12.4. [Công cụ giám sát Queue chuyên nghiệp](#124-công-cụ-giám-sát-queue-chuyên-nghiệp)
13. [Tổng kết và bài học rút ra](#13-tổng-kết-và-bài-học-rút-ra)

---

## 1. TỔNG QUAN VỀ EMAIL SERVER

### 1.1. Định nghĩa

**Email server** (máy chủ thư điện tử) là một hệ thống máy tính được cấu hình chuyên biệt, đảm nhận toàn bộ quá trình tiếp nhận, xử lý, lưu trữ và chuyển phát thư điện tử trên môi trường mạng Internet. Không giống với người dùng phổ thông vốn chỉ nhìn thấy giao diện webmail đơn giản, phía sau đó là một hệ thống cực kỳ phức tạp với nhiều tầng phần mềm, giao thức và cơ chế bảo mật hoạt động song song.

Ẩn dụ quen thuộc nhất là so sánh email server với một hệ thống bưu điện kỹ thuật số. Người gửi thư (MUA) đến quầy viết và nộp thư, nhân viên bưu điện (MSA) tiếp nhận và kiểm tra xem thư có đủ điều kiện gửi không, trung tâm phân loại (MTA) đóng gói và định tuyến thư đến đúng bưu cục đích, nhân viên giao hàng nội bộ (MDA) mang thư đến đặt vào đúng hộp thư của người nhận, và cuối cùng người nhận mở hộp thư đọc thư của mình (qua IMAP/POP3).

### 1.2. Tại sao phải tự vận hành Email Server?

Nhiều tổ chức chọn tự xây dựng và vận hành email server thay vì dùng dịch vụ bên thứ ba (như Google Workspace hay Microsoft 365) vì các lý do sau:

*   **Kiểm soát dữ liệu hoàn toàn:** Toàn bộ nội dung thư điện tử lưu trữ nội bộ, tránh rủi ro rò rỉ dữ liệu ra ngoài.
*   **Tuân thủ quy định pháp lý:** Một số ngành (y tế, tài chính, nhà nước) bắt buộc phải lưu trữ dữ liệu trên hạ tầng do tổ chức kiểm soát.
*   **Tiết kiệm chi phí lâu dài:** Với số lượng người dùng lớn, chi phí thuê dịch vụ hàng tháng thường cao hơn nhiều so với chi phí tự vận hành.
*   **Tùy biến không giới hạn:** Tự do cấu hình các chính sách lọc spam, routing thư, tích hợp với các hệ thống nội bộ khác.

---

## 2. KIẾN TRÚC HỆ THỐNG VÀ LUỒNG XỬ LÝ EMAIL

### 2.1. Sơ đồ luồng email từ đầu đến cuối

Toàn bộ hành trình của một bức thư điện tử có thể được mô tả qua luồng sau đây:

```
[Người gửi - MUA]
      |
      | SMTP (Port 587 + TLS)
      v
[MSA - Mail Submission Agent]  <-- Xác thực người dùng (SASL)
      |
      | Nội bộ
      v
[MTA Gửi - Postfix/Exim]  <-- Kiểm tra SPF, DKIM ký vào thư
      |
      | DNS Lookup (MX Record của domain đích)
      | SMTP (Port 25, bắt buộc TLS)
      v
[MTA Nhận - Postfix/Exim của máy chủ đích]
      |
      | Nội bộ -> Amavis
      v
[Anti-Spam/Anti-Virus - SpamAssassin + ClamAV]
      |
      | Kết quả quét (OK / Spam / Virus)
      v
[MDA - Dovecot]  <-- Ghi vào hòm thư (Maildir/mbox)
      |
      | IMAP (Port 993) hoặc POP3 (Port 995)
      v
[Người nhận - MUA / Webmail]
```

Đây là toàn bộ hành trình của một email từ lúc bạn nhấn "Gửi" cho đến khi nó xuất hiện trong hộp thư đến của người nhận.

### 2.2. Bốn vai trò chức năng chính

Trong kiến trúc email hiện đại, mỗi thành phần phần mềm đảm nhận một vai trò chức năng được định nghĩa rõ ràng:

| Vai trò | Viết tắt | Chức năng chính | Ví dụ phần mềm |
|---|---|---|---|
| Mail User Agent | MUA | Giao diện người dùng để soạn/đọc thư | Thunderbird, Outlook, Roundcube |
| Mail Submission Agent | MSA | Tiếp nhận thư từ MUA, xác thực và chuyển cho MTA | Postfix (port 587) |
| Mail Transfer Agent | MTA | Định tuyến và chuyển phát thư giữa các máy chủ | Postfix, Exim, Sendmail |
| Mail Delivery Agent | MDA | Phân phối thư vào hòm thư cuối cùng của người dùng | Dovecot, Procmail |

---

## 3. CÁC GIAO THỨC NỀN TẢNG

### 3.1. SMTP 

**SMTP (Simple Mail Transfer Protocol)** là giao thức tiêu chuẩn để truyền tải email, được định nghĩa trong RFC 5321. Về bản chất, SMTP là một giao thức dạng văn bản (text-based), hoạt động theo mô hình yêu cầu - phản hồi (request-response) trên nền TCP.

#### Phân biệt các cổng SMTP

| Cổng | Tên | Mục đích | Mã hóa |
|---|---|---|---|
| 25 | SMTP | Liên lạc giữa các MTA (server-to-server) | Không bắt buộc (Opportunistic TLS) |
| 587 | Submission | Client gửi thư lên server (khuyến nghị hiện nay) | Bắt buộc STARTTLS |
| 465 | SMTPS | Client gửi thư, mã hóa ngay từ đầu (cũ hơn) | Implicit TLS (SSL từ đầu) |

Cổng 25 hiện nay bị hầu hết các ISP (nhà cung cấp dịch vụ Internet) chặn đối với các kết nối từ IP của người dùng thông thường để ngăn chặn spam. Chỉ có các máy chủ mail mới có thể dùng cổng 25 để giao tiếp với nhau.

#### Phiên làm việc SMTP thực tế

Dưới đây là một phiên SMTP hoàn chỉnh được mô phỏng qua lệnh `telnet`:

```
# Kết nối đến máy chủ SMTP
$ telnet mail.example.com 25

# Server chào hỏi
220 mail.example.com ESMTP Postfix (Ubuntu)

# Client tự giới thiệu
EHLO myserver.com

# Server phản hồi danh sách tính năng hỗ trợ
250-mail.example.com
250-PIPELINING
250-SIZE 52428800
250-STARTTLS
250-AUTH PLAIN LOGIN
250 HELP

# Client yêu cầu nâng cấp lên TLS
STARTTLS

# Server xác nhận sẵn sàng
220 2.0.0 Ready to start TLS

# (TLS Handshake diễn ra, toàn bộ nội dung tiếp theo được mã hóa)

# Khai báo người gửi
MAIL FROM:<sender@myserver.com>

# Server chấp nhận
250 2.1.0 Ok

# Khai báo người nhận
RCPT TO:<recipient@example.com>

# Server chấp nhận người nhận
250 2.1.5 Ok

# Client chuẩn bị gửi nội dung thư
DATA

# Server thông báo sẵn sàng nhận nội dung
354 End data with <CR><LF>.<CR><LF>

# Client gửi header và body của thư
From: sender@myserver.com
To: recipient@example.com
Subject: Test email
Date: Mon, 23 Jun 2026 15:00:00 +0700

Đây là nội dung của email thử nghiệm.

# Kết thúc nội dung bằng dấu chấm trên dòng riêng
.

# Server xác nhận đã nhận thư và gán Queue ID
250 2.0.0 Ok: queued as B4F1D3A8

# Client đóng kết nối
QUIT

# Server xác nhận đóng kết nối
221 2.0.0 Bye
```

#### Mã phản hồi SMTP thường gặp

| Mã | Ý nghĩa |
|---|---|
| 220 | Dịch vụ sẵn sàng (lời chào từ server) |
| 250 | Lệnh thực hiện thành công |
| 354 | Server sẵn sàng nhận nội dung thư |
| 421 | Dịch vụ tạm thời không khả dụng |
| 450 | Hòm thư người nhận tạm thời không truy cập được |
| 550 | Hành động bị từ chối (ví dụ: hòm thư không tồn tại) |
| 553 | Tên người dùng không hợp lệ |
| 554 | Giao dịch thất bại (thường do bị chặn spam) |
| 5xx | Lỗi vĩnh viễn, không nên thử lại |

### 3.2. IMAP và POP3 

Sau khi thư được lưu vào hòm thư, người dùng cần một giao thức để đọc nó. Có hai giao thức chính:

#### POP3 (Post Office Protocol version 3) - Port 110 / 995 (SSL)

POP3 là giao thức ra đời sớm, thiết kế cho thời kỳ internet còn chậm và đắt tiền. Logic cốt lõi của nó rất đơn giản: kết nối vào server, tải toàn bộ thư về máy tính, xóa thư trên server (hoặc giữ lại nếu cấu hình), ngắt kết nối.

*   **Điểm mạnh:** Thư được lưu offline trên thiết bị, không cần kết nối internet để đọc.
*   **Điểm yếu:** Thư chỉ tồn tại trên một thiết bị. Nếu bạn đọc thư trên điện thoại, máy tính xách tay sẽ không thấy thư đó nữa. Đây là vấn đề nghiêm trọng trong thời đại đa thiết bị.

#### IMAP (Internet Message Access Protocol) - Port 143 / 993 (SSL)

IMAP là giải pháp hiện đại và được khuyến nghị cho hầu hết mọi trường hợp. Không giống POP3, IMAP không tải thư về rồi xóa, mà luôn đồng bộ trạng thái giữa server và tất cả các thiết bị của người dùng.

*   **Điểm mạnh:** Đọc thư trên điện thoại và máy tính đều thấy đúng trạng thái (đã đọc/chưa đọc, đã xóa, đã di chuyển). Tất cả thay đổi được phản chiếu ngay lập tức trên mọi thiết bị.
*   **Điểm yếu:** Yêu cầu kết nối internet liên tục. Dung lượng hòm thư trên server là tài nguyên cần quản lý.

| Tiêu chí so sánh | POP3 | IMAP |
|---|---|---|
| Nơi lưu thư | Máy tính cục bộ | Server |
| Đồng bộ đa thiết bị | Không | Có |
| Cần kết nối internet | Chỉ lúc tải thư | Liên tục |
| Truy cập offline | Có | Hạn chế |
| Phù hợp với | Người dùng 1 thiết bị | Người dùng đa thiết bị (khuyến nghị) |

---

## 4. MTA - MAIL TRANSFER AGENT

MTA là trái tim của hệ thống email, đảm nhận vai trò định tuyến và chuyển phát thư giữa các máy chủ trên toàn Internet bằng giao thức SMTP. Khi nhận được thư cần gửi đi, MTA thực hiện tra cứu DNS để tìm bản ghi MX của domain đích, sau đó thiết lập kết nối TCP đến máy chủ đích và chuyển thư đi.

### 4.1. Postfix

Postfix được tạo ra bởi Wietse Venema vào cuối những năm 1990 với mục tiêu ban đầu là thay thế Sendmail, khắc phục các điểm yếu về bảo mật và độ phức tạp. Đến nay, Postfix là MTA được sử dụng rộng rãi nhất trên các hệ thống Linux.

**Kiến trúc module hóa của Postfix:**

Thay vì một tiến trình nguyên khối chạy với quyền root, Postfix chia nhỏ công việc thành nhiều tiến trình con độc lập, mỗi tiến trình chỉ có quyền tối thiểu cần thiết:

*   **smtpd:** Nhận kết nối SMTP từ bên ngoài, xác thực người gửi.
*   **pickup:** Thu thập thư từ hàng đợi `maildrop` (thư gửi từ local).
*   **cleanup:** Dọn dẹp, hoàn thiện header của thư trước khi đưa vào hàng đợi chính.
*   **qmgr (Queue Manager):** Bộ não điều phối toàn bộ hoạt động queue, quyết định thư nào được gửi đi theo thứ tự nào.
*   **smtp:** Là client SMTP, chịu trách nhiệm kết nối ra ngoài internet để chuyển thư đến máy chủ đích.
*   **local:** Phân phối thư đến các hòm thư người dùng trên chính máy chủ đó.
*   **virtual:** Xử lý phân phối cho các domain ảo (virtual domains).
*   **bounce:** Tạo ra thông báo lỗi (bounce message) khi thư không gửi được.

**Ưu điểm Postfix:**
*   Bảo mật cao do kiến trúc chia nhỏ tiến trình - nếu một module bị khai thác, attacker không có quyền root.
*   Cấu hình qua tệp `main.cf` và `master.cf` rõ ràng, có tài liệu phong phú.
*   Hiệu năng cực cao, có thể xử lý hàng triệu email mỗi ngày.
*   Tích hợp hoàn hảo với Dovecot, SpamAssassin và Amavis.

### 4.2. Exim 
Exim là lựa chọn mặc định của cPanel và là MTA thống trị trong thị trường web hosting chia sẻ. Điểm đặc biệt của Exim là khả năng tùy biến bộ định tuyến (router) và phương thức vận chuyển (transport) cực kỳ mạnh mẽ thông qua tệp cấu hình duy nhất `exim.conf`.

Ví dụ, quản trị viên có thể định nghĩa các quy tắc phức tạp như: "Thư gửi đến domain A thì chuyển qua relay server nội bộ, thư gửi đến domain B thì từ chối, thư có dung lượng trên 20MB thì chuyển vào hàng đợi ưu tiên thấp" - tất cả đều được cấu hình trong một tệp duy nhất bằng ngôn ngữ cấu hình riêng của Exim.

**Điểm yếu:** Cú pháp cấu hình của Exim nổi tiếng là phức tạp và khó học hơn Postfix nhiều.

### 4.3. Sendmail 

Sendmail là MTA nguyên thủy, ra mắt năm 1983 và từng là tiêu chuẩn của toàn bộ Internet suốt nhiều thập kỷ. Kiến trúc nguyên khối (monolithic) của nó chạy một tiến trình duy nhất với quyền root, điều này là mầm mống của vô số lỗ hổng bảo mật đã từng được khai thác.

Cấu hình của Sendmail được thực hiện qua các tệp macro `m4` - một ngôn ngữ có cú pháp cực kỳ khó đọc và dễ gây lỗi. Ngày nay, Sendmail được giữ lại chủ yếu trong các hệ thống kế thừa (legacy) không thể nâng cấp, chứ không ai dùng cho hệ thống mới.

### 4.4. qmail - Tượng đài bảo mật đã ngủ yên

qmail được phát triển bởi Daniel J. Bernstein vào những năm 1990 như một câu trả lời trực tiếp cho các vấn đề bảo mật của Sendmail. Bernstein đã thiết kế qmail với triết lý "chia nhỏ đặc quyền" (privilege separation) ở mức độ triệt để: mỗi công việc được giao cho một tiến trình riêng với quyền hệ thống tối thiểu. Ông thậm chí treo thưởng 1.000 USD (sau tăng lên 5.000 USD) cho bất kỳ ai tìm ra lỗ hổng bảo mật trong qmail - một thử thách không ai nhận được trong nhiều năm.

Tuy nhiên, qmail đã ngừng phát triển tích cực từ lâu, thiếu hỗ trợ cho IPv6, TLS hiện đại và nhiều tiêu chuẩn khác. Cộng đồng đã tạo ra các bản vá (netqmail, notqmail) để duy trì, nhưng nhìn chung, qmail không còn là lựa chọn thực tế cho hệ thống mới.

---

## 5. MDA - MAIL DELIVERY AGENT

Sau khi MTA nhận được thư từ Internet và xác định thư đó thuộc về domain nội bộ, nó bàn giao thư cho MDA. MDA thực hiện bước phân phối cuối cùng: đặt thư vào đúng hòm thư của người dùng trên ổ cứng, đồng thời cung cấp dịch vụ IMAP/POP3 để người dùng có thể kết nối vào đọc thư.

### 5.1. Hai định dạng lưu trữ thư phổ biến

Trước khi đi vào các phần mềm MDA, cần hiểu cách email được lưu trên đĩa cứng:

*   **mbox:** Toàn bộ thư của một hòm thư được lưu vào một file duy nhất, liên tiếp nhau. Đây là định dạng cũ, đơn giản nhưng có vấn đề nghiêm trọng về hiệu năng và khả năng phục hồi: khi file bị hỏng là mất hết thư, và đọc/ghi đồng thời gây ra tình huống tranh chấp (locking).

*   **Maildir:** Mỗi email là một file riêng biệt, được tổ chức trong cấu trúc thư mục `cur/` (thư đã đọc), `new/` (thư mới) và `tmp/` (đang xử lý). Đây là định dạng hiện đại, ưu việt hơn nhiều: thêm/xóa thư chỉ là tạo/xóa file, không ảnh hưởng đến các thư khác, và hỗ trợ truy cập đồng thời hoàn hảo.

### 5.2. Dovecot - Người đứng đầu hiện tại

Dovecot được phát triển bởi Timo Sirainen và hiện là MDA phổ biến nhất thế giới, được sử dụng trên hàng triệu máy chủ. Kiến trúc của Dovecot được chia thành nhiều tiến trình riêng biệt tương tự Postfix, đảm bảo tính bảo mật và ổn định.

**Các tính năng nổi bật của Dovecot:**
*   Hỗ trợ đầy đủ cả IMAP4rev1 và POP3 với TLS.
*   Hỗ trợ cả Maildir và mbox, dễ dàng chuyển đổi giữa hai định dạng.
*   Tích hợp với Postfix qua giao thức LMTP (Local Mail Transfer Protocol) - là cách giao tiếp hiệu quả hơn so với dùng lại SMTP nội bộ.
*   Hệ thống xác thực mạnh mẽ (SASL), hỗ trợ kết nối với LDAP, MySQL, PostgreSQL.
*   Tính năng Sieve scripting: cho phép người dùng tự định nghĩa các quy tắc lọc thư (chuyển thư từ địa chỉ A vào thư mục B, tự động trả lời khi vắng mặt...).

**Ví dụ cấu hình kết hợp Postfix + Dovecot:**

Trong `main.cf` của Postfix:
```
# Chuyển thư đến Dovecot qua LMTP thay vì trực tiếp vào mailbox
mailbox_transport = lmtp:unix:private/dovecot-lmtp
```

Trong `dovecot.conf`:
```
# Lắng nghe kết nối IMAP và POP3
protocols = imap pop3 lmtp

# Sử dụng định dạng Maildir
mail_location = maildir:~/Maildir

# Yêu cầu mã hóa SSL/TLS
ssl = required
ssl_cert = </etc/ssl/certs/dovecot.pem
ssl_key = </etc/ssl/private/dovecot.key
```

### 5.3. Cyrus IMAP - Giải pháp doanh nghiệp lớn

Cyrus được phát triển bởi Carnegie Mellon University và thiết kế đặc biệt cho môi trường doanh nghiệp quy mô lớn với hàng chục nghìn hòm thư. Kiến trúc "sealed-server" (máy chủ niêm phong) của nó có nghĩa là toàn bộ dữ liệu email được quản lý thông qua các công cụ nội bộ của Cyrus, không thể can thiệp trực tiếp bằng lệnh Linux thông thường như `cp`, `mv`, hay `cat`.

*   **Điểm mạnh:** Xử lý hàng triệu thư một cách ổn định, hỗ trợ tốt các tính năng doanh nghiệp như shared mailbox, ACL (Access Control List).
*   **Điểm yếu:** Phức tạp khi triển khai và bảo trì, không phù hợp với nhóm quản trị viên ít kinh nghiệm hoặc hệ thống vừa và nhỏ.

---

## 6. MUA - MAIL USER AGENT VÀ WEBMAIL

MUA (Mail User Agent) là điểm tiếp xúc trực tiếp giữa người dùng và hệ thống email. MUA kết nối với MDA (qua IMAP hoặc POP3) để nhận thư, và kết nối với MSA/MTA (qua SMTP) để gửi thư.

### 6.1. Roundcube - Webmail hiện đại thuần túy

Roundcube là ứng dụng webmail mã nguồn mở viết bằng PHP, nổi tiếng với giao diện người dùng mượt mà nhờ sử dụng AJAX - nội dung được cập nhật động mà không cần tải lại toàn trang. Trải nghiệm sử dụng Roundcube gần giống với Gmail hay Outlook Web.

Roundcube chỉ tập trung vào một việc duy nhất: quản lý email, và nó làm việc đó rất tốt. Không có lịch, không có danh bạ phức tạp - nhưng bù lại là sự gọn nhẹ và ổn định.

### 6.2. Horde Groupware - Giải pháp cộng tác toàn diện

Horde không chỉ là webmail mà là một bộ phần mềm cộng tác (Groupware) đầy đủ chức năng:

*   **IMP:** Module webmail chính.
*   **Kronolith:** Ứng dụng lịch dùng chung (tương tự Google Calendar).
*   **Turba:** Quản lý danh bạ tập trung.
*   **Nag:** Quản lý công việc và danh sách việc cần làm.
*   **Mnemo:** Ghi chú và memo.

Horde là lựa chọn phù hợp khi tổ chức cần một giải pháp thay thế Microsoft Exchange hoàn toàn mã nguồn mở và kiểm soát dữ liệu nội bộ.

### 6.3. Client desktop thông dụng

Ngoài webmail, người dùng cũng thường dùng các ứng dụng desktop hoặc di động:

*   **Mozilla Thunderbird:** Client email mã nguồn mở, đa nền tảng, hỗ trợ OpenPGP tích hợp để mã hóa email đầu cuối.
*   **Microsoft Outlook:** Phổ biến trong môi trường doanh nghiệp, hỗ trợ tốt Exchange và tích hợp với bộ Microsoft 365.

---

## 7. BẢO MẬT TẦNG TRUYỀN TẢI - TLS / STARTTLS

Ngay cả khi hệ thống email hoạt động hoàn hảo, nếu không có mã hóa, toàn bộ nội dung thư (bao gồm mật khẩu, dữ liệu nhạy cảm) di chuyển trên mạng dưới dạng văn bản thuần túy (plaintext) - bất kỳ ai ở giữa đường truyền đều có thể đọc được.

### 7.1. STARTTLS - Nâng cấp kết nối an toàn

STARTTLS là cơ chế cho phép một kết nối ban đầu ở dạng plaintext được "nâng cấp" thành kết nối mã hóa TLS. Quá trình như sau:

1.  Client kết nối đến server trên cổng 587 (plaintext ban đầu).
2.  Server thông báo trong danh sách tính năng EHLO: `250-STARTTLS`.
3.  Client gửi lệnh `STARTTLS`.
4.  Server phản hồi `220 Ready to start TLS`.
5.  Cả hai bên thực hiện TLS Handshake - trao đổi chứng chỉ, xác minh, đồng thuận khóa mã hóa.
6.  Từ thời điểm này, toàn bộ giao tiếp được mã hóa.

### 7.2. Implicit TLS (Cổng 465)

Khác với STARTTLS, Implicit TLS thiết lập kết nối mã hóa ngay từ byte đầu tiên. Không có giai đoạn plaintext nào cả. Cổng 465 hoạt động theo cơ chế này.

### 7.3. Thực hành tốt nhất về TLS

*   **Chỉ sử dụng TLS 1.2 hoặc 1.3:** Vô hiệu hóa hoàn toàn SSL 2.0, 3.0 và TLS 1.0, 1.1 vì chúng có lỗ hổng đã biết (POODLE, BEAST...).
*   **Chứng chỉ SSL hợp lệ:** Sử dụng chứng chỉ từ CA đáng tin cậy (Let's Encrypt cung cấp miễn phí), không dùng self-signed certificate cho môi trường production.
*   **Cấu hình bộ mật mã (cipher suite) mạnh:** Ưu tiên các cipher hỗ trợ Perfect Forward Secrecy (PFS) như ECDHE.

---

## 8. HỆ THỐNG KIỂM SOÁT AN NINH - ANTI-SPAM VÀ ANTI-VIRUS

Đây là lớp phòng thủ quan trọng nhất của hệ thống email. Mỗi email đến đều phải đi qua nhiều tầng kiểm tra trước khi được phép vào hòm thư người nhận.

### 8.1. SpamAssassin - Cơ chế chấm điểm

SpamAssassin không hoạt động theo kiểu "danh sách đen trắng" đơn giản mà sử dụng một hệ thống chấm điểm đa chiều cực kỳ tinh vi. Mỗi email được "đánh giá" qua hàng trăm quy tắc (rules), mỗi quy tắc có trọng số điểm riêng (dương hoặc âm).

#### Các cơ chế lọc của SpamAssassin

**a) Phân tích tiêu đề (Header Analysis):**
Kiểm tra các trường như `Received:`, `From:`, `Reply-To:`, `Subject:`. Ví dụ:
*   Trường `From:` và `Return-Path:` trỏ đến domain khác nhau (+2.5 điểm).
*   Subject có toàn chữ hoa (+1.5 điểm).
*   Header `Received:` cho thấy thư đi qua quá nhiều relay bất thường (+1.0 điểm).

**b) Phân tích nội dung (Content Analysis):**
Dùng các biểu thức chính quy để tìm kiếm các mẫu spam điển hình:
*   Các từ khóa như "MAKE MONEY FAST", "FREE VIAGRA", "CLICK HERE NOW" (+3.0 điểm).
*   Nội dung chứa quá nhiều link rút gọn (+1.5 điểm).
*   HTML có chứa hình ảnh tracking pixel ẩn (+1.0 điểm).

**c) Tra cứu danh sách đen DNS (DNSBL / RBL - DNS-based Blackhole List / Real-time Blackhole List):**
SpamAssassin truy vấn các cơ sở dữ liệu uy tín để kiểm tra địa chỉ IP của máy chủ gửi:
*   **Spamhaus (ZEN):** Danh sách đen được cập nhật liên tục bởi tổ chức chống spam lớn nhất thế giới.
*   **Barracuda Reputation Block List (BRBL):** Cơ sở dữ liệu về IP có lịch sử gửi spam.
*   **SORBS (Spam and Open Relay Blocking System):** Phát hiện các open relay (máy chủ cho phép bất kỳ ai relay thư qua).

Nếu IP của người gửi xuất hiện trong bất kỳ danh sách nào, SpamAssassin cộng thêm điểm đáng kể vào tổng điểm spam.

**d) Bộ lọc Bayesian (Bayesian Filter):**
Đây là thành phần thông minh nhất của SpamAssassin. Nó học từ dữ liệu thực tế theo nguyên lý thống kê Bayes:

1.  Quản trị viên hoặc người dùng "dạy" (train) cho bộ lọc bằng cách đánh dấu email là spam hoặc ham (không phải spam).
2.  Bộ lọc phân tích các "token" (từ, cụm từ, đặc điểm cấu trúc) và tính xác suất: token X xuất hiện trong spam với tần suất bao nhiêu % so với ham?
3.  Khi nhận email mới, bộ lọc tính xác suất tổng hợp dựa trên tất cả các token có trong email đó.

Điểm mạnh là bộ lọc Bayesian liên tục tự học và thích nghi với các chiến thuật spam mới mà không cần cập nhật quy tắc thủ công.

**e) Quy trình chấm điểm và hành động:**

```
# Ví dụ header X-Spam được SpamAssassin thêm vào email:

X-Spam-Status: Yes, score=7.8 required=5.0
    tests=BAYES_99=3.5,
          RCVD_IN_ZEN_BLOCKED=2.8,
          HTML_IMAGE_ONLY_04=1.5
X-Spam-Flag: YES

# Giải thích:
# - Tổng điểm: 7.8 (vượt ngưỡng 5.0 -> đánh dấu là Spam)
# - BAYES_99: Bộ lọc Bayesian tính xác suất 99% là spam (+3.5)
# - RCVD_IN_ZEN_BLOCKED: IP nguồn có trong danh sách đen Spamhaus (+2.8)
# - HTML_IMAGE_ONLY_04: Email chỉ chứa hình ảnh, không có chữ (+1.5)
```

### 8.2. ClamAV - Diệt virus tệp đính kèm

ClamAV là engine diệt virus mã nguồn mở được sử dụng rộng rãi nhất trong hệ sinh thái Linux. Nhiệm vụ chính của nó là quét tệp đính kèm trong email để phát hiện mã độc.

**Cách ClamAV hoạt động:**

1.  Nhận được nội dung email từ Amavis.
2.  Giải nén và phân tích tất cả các tệp đính kèm (kể cả file ZIP, RAR lồng nhau đến nhiều cấp độ).
3.  So sánh các mẫu nhị phân với cơ sở dữ liệu chữ ký virus (signature database) được cập nhật liên tục qua `freshclam`.
4.  Sử dụng phân tích heuristic để phát hiện các biến thể virus chưa có trong cơ sở dữ liệu.
5.  Trả kết quả về cho Amavis: sạch (CLEAN) hay phát hiện mối đe dọa (kèm tên virus).

**Lịch cập nhật database:**
```bash
# freshclam chạy tự động qua cron để cập nhật chữ ký virus
$ freshclam
ClamAV update process started at Mon Jun 23 2026
Downloading main.cvd [100%]
main.cvd updated (version: 62, sigs: 6647427)
Downloading daily.cvd [100%]
daily.cvd updated (version: 27394, sigs: 2059620)
```

### 8.3. Amavis - Phần mềm trung gian (Middleware)

Amavis (amavisd-new) đóng vai trò "cầu nối" giữa Postfix và các engine kiểm tra nội dung (SpamAssassin, ClamAV). Điều này rất quan trọng vì Postfix không biết cách tự gọi SpamAssassin hay ClamAV.

**Luồng xử lý qua Amavis:**

```
Postfix (smtpd) nhận thư từ Internet
        |
        | (Re-inject sang cổng 10024)
        v
Amavis lắng nghe trên cổng 10024
        |-- Gọi ClamAV quét virus
        |-- Gọi SpamAssassin tính điểm spam
        |
        | Tổng hợp kết quả:
        | - Sạch -> Chuyển thư về Postfix cổng 10025
        | - Spam -> Thêm header X-Spam, chuyển về Postfix
        | - Virus -> Cách ly (quarantine) hoặc xóa, gửi thông báo
        v
Postfix (smtpd) nhận lại thư trên cổng 10025
        |
        | Chuyển thư đến Dovecot (MDA) để phân phối vào hòm thư
        v
```

---

## 9. CÁC BẢN GHI DNS QUAN TRỌNG

Bước quan trọng không kém gì việc cài phần mềm là cấu hình DNS đúng đắn. DNS là hệ thống "thẻ căn cước" của domain trên Internet, giúp các máy chủ khác định danh và xác minh máy chủ email của bạn.

### 9.1. MX - Mail Exchanger

MX là bản ghi nền tảng nhất. Khi một máy chủ khác cần gửi email đến `user@tenmien.com`, trước tiên nó phải tra cứu DNS để hỏi: "Máy chủ email của `tenmien.com` là ai?" - câu trả lời chính là bản ghi MX.

```
; Ví dụ cấu hình DNS Zone cho bản ghi MX
tenmien.com.  IN MX 10  mail1.tenmien.com.   ; Ưu tiên cao (số nhỏ hơn)
tenmien.com.  IN MX 20  mail2.tenmien.com.   ; Máy chủ dự phòng

; Bản ghi A trỏ hostname về IP
mail1.tenmien.com.  IN A  203.0.113.1
mail2.tenmien.com.  IN A  203.0.113.2
```

**Giá trị Priority (mức ưu tiên):** Số nhỏ hơn = ưu tiên cao hơn. Máy chủ gửi thư sẽ luôn thử kết nối đến mail1 trước. Chỉ khi mail1 không phản hồi, nó mới chuyển sang mail2 (dự phòng).

### 9.2. PTR - Pointer Record / Reverse DNS

PTR là bản ghi "ngược" - thay vì từ tên miền ra IP (bản ghi A), PTR dịch từ IP ngược về tên miền. Đây là cơ chế xác minh danh tính quan trọng: khi máy chủ của bạn gửi thư đến Gmail, Gmail sẽ kiểm tra: "IP 203.0.113.1 này khi tra ngược có đúng là `mail1.tenmien.com` không?"

Nếu không có bản ghi PTR hoặc PTR không khớp với hostname, hầu hết các máy chủ lớn (Gmail, Yahoo, Outlook) sẽ từ chối nhận thư hoặc cho vào spam.

**Lưu ý quan trọng:** Bản ghi PTR không nằm trong DNS zone của bạn mà do nhà cung cấp hạ tầng (VPS, Cloud provider) quản lý. Bạn phải liên hệ hoặc cấu hình qua bảng điều khiển của nhà cung cấp.

### 9.3. SPF - Sender Policy Framework

SPF là bản ghi TXT trong DNS công bố danh sách chính thức các địa chỉ IP/hostname được phép gửi email mạo danh domain của bạn.

**Cú pháp SPF và các cơ chế:**

```
; Ví dụ bản ghi SPF phức tạp trong thực tế
tenmien.com.  IN TXT  "v=spf1 ip4:203.0.113.1 ip4:203.0.113.0/24 include:sendgrid.net mx -all"
```

Giải thích từng phần:

| Phần | Ý nghĩa |
|---|---|
| `v=spf1` | Phiên bản SPF, luôn phải là spf1 |
| `ip4:203.0.113.1` | Cho phép địa chỉ IP cụ thể này |
| `ip4:203.0.113.0/24` | Cho phép toàn bộ dải IP /24 |
| `include:sendgrid.net` | Cho phép các IP trong SPF record của sendgrid.net |
| `mx` | Cho phép các IP đã được khai báo trong bản ghi MX của domain |
| `-all` | Từ chối tất cả các IP còn lại (hard fail) |

**Các giá trị qualifier:**

| Qualifier | Ký hiệu | Hành động khi khớp |
|---|---|---|
| Pass | `+` (mặc định) | Cho phép |
| Soft Fail | `~` | Đánh dấu nghi ngờ nhưng không từ chối |
| Fail | `-` | Từ chối hoàn toàn (hard fail) |
| Neutral | `?` | Không có hành động |

**Giới hạn 10 DNS lookup:** RFC quy định mỗi bản ghi SPF chỉ được phép thực hiện tối đa 10 DNS lookup (do `include:`, `mx`, `ptr`...). Nếu vượt quá, bản ghi SPF bị coi là không hợp lệ (permerror).

### 9.4. DKIM - DomainKeys Identified Mail

DKIM giải quyết một vấn đề mà SPF không thể: SPF chỉ xác minh IP gửi thư, nhưng không đảm bảo nội dung thư không bị thay đổi trên đường truyền. DKIM sử dụng mã hóa khóa công khai (Public Key Cryptography) để "ký số" vào nội dung email.

**Quy trình ký và xác minh DKIM:**

```
BÊN GỬI (mail server của bạn):
─────────────────────────────
1. Server tạo một cặp khóa RSA:
   - Private Key: Lưu bí mật trên server
   - Public Key: Công bố lên DNS

2. Khi gửi thư, server tính giá trị hash (SHA-256)
   của một số header + body của email.

3. Server mã hóa hash đó bằng Private Key -> Digital Signature.

4. Thêm DKIM-Signature header vào email:
   DKIM-Signature: v=1; a=rsa-sha256; d=tenmien.com;
     s=selector1; c=relaxed/relaxed;
     h=from:to:subject:date;
     bh=BASE64_BODY_HASH;
     b=BASE64_DIGITAL_SIGNATURE

BÊN NHẬN (mail server của người nhận):
───────────────────────────────────────
5. Server nhận thấy DKIM-Signature header.
6. Tra cứu DNS để lấy Public Key:
   selector1._domainkey.tenmien.com  IN TXT  "v=DKIM1; k=rsa; p=PUBLIC_KEY_BASE64"
7. Dùng Public Key giải mã chữ ký trong header -> thu được hash gốc.
8. Tính lại hash từ email nhận được.
9. So sánh: nếu khớp -> email nguyên vẹn, không bị chỉnh sửa.
            nếu không khớp -> DKIM FAIL, email có thể bị giả mạo.
```

**Khái niệm Selector:** Cho phép bạn sử dụng nhiều cặp khóa DKIM khác nhau (ví dụ cho từng dịch vụ gửi mail), dễ dàng thu hồi một selector bị lộ mà không ảnh hưởng đến các selector khác.

**Thực hành tốt nhất:** Định kỳ (6-12 tháng) thực hiện xoay vòng khóa DKIM (key rotation) để giảm rủi ro nếu private key bị lộ.

### 9.5. DMARC - Domain-based Message Authentication

DMARC là tầng chính sách cuối cùng, đứng trên SPF và DKIM. Nó xác định rõ ràng cho máy chủ nhận biết phải làm gì khi email không vượt qua được SPF hoặc DKIM.

**Cú pháp DMARC:**

```
; Bản ghi DMARC đầy đủ
_dmarc.tenmien.com.  IN TXT  "v=DMARC1; p=quarantine; rua=mailto:dmarc-reports@tenmien.com; ruf=mailto:forensics@tenmien.com; fo=1; pct=100; sp=reject"
```

Giải thích các tag:

| Tag | Ý nghĩa | Ví dụ |
|---|---|---|
| `v=DMARC1` | Phiên bản DMARC | Bắt buộc |
| `p=` | Chính sách cho domain chính | none / quarantine / reject |
| `sp=` | Chính sách cho subdomain | none / quarantine / reject |
| `rua=` | Email nhận báo cáo tổng hợp (Aggregate) | mailto:dmarc@domain.com |
| `ruf=` | Email nhận báo cáo pháp y (Forensic) | mailto:forensics@domain.com |
| `pct=` | Phần trăm email áp dụng chính sách | 100 (mặc định) |
| `fo=` | Tùy chọn báo cáo forensic | 0 / 1 / d / s |



### 9.6. SRV - Service Record

Bản ghi SRV cung cấp thông tin cấu hình tự động (Autodiscover / Autoconfig) cho các ứng dụng email client như Outlook và Thunderbird. Khi người dùng nhập địa chỉ email vào ứng dụng, phần mềm tự động tra cứu DNS để biết chính xác thông tin kết nối mà không cần người dùng gõ tay.

```
; Bản ghi SRV cho Autodiscover (Outlook)
_autodiscover._tcp.tenmien.com.  IN SRV 0 1 443  mail.tenmien.com.

; Bản ghi SRV cho IMAP over TLS
_imaps._tcp.tenmien.com.  IN SRV 0 1 993  mail.tenmien.com.

; Bản ghi SRV cho SMTP Submission over TLS
_submissions._tcp.tenmien.com.  IN SRV 0 1 465  mail.tenmien.com.

; Cú pháp: _service._proto.domain TTL class SRV priority weight port target
```

---

## 11. GIÁM SÁT HỆ THỐNG VÀ PHÂN TÍCH LOGS

"Không thể quản lý thứ mình không đo lường được." - Triết lý này đặc biệt đúng với hệ thống email server. Logs là nguồn thông tin duy nhất để biết chuyện gì đang xảy ra với hàng nghìn email đi qua server mỗi ngày.

### 11.1. Phân loại và ý nghĩa các file Logs

**a) Logs gửi/nhận mail (Mail Logs):**

Đây là file quan trọng nhất, ghi lại toàn bộ hành trình của mỗi email:

| Hệ điều hành | Đường dẫn mặc định |
|---|---|
| Ubuntu / Debian | `/var/log/mail.log` |
| CentOS / RHEL / Fedora | `/var/log/maillog` |
| systemd (mọi distro) | `journalctl -u postfix` |

**b) Logs xác thực người dùng (Authentication Logs):**

Ghi lại các lần đăng nhập thành công và thất bại:

| Hệ điều hành | Đường dẫn mặc định |
|---|---|
| Ubuntu / Debian | `/var/log/auth.log` |

**c) Logs Dovecot (IMAP/POP3):**

Dovecot mặc định ghi log vào `mail.log`, nhưng có thể tách ra bằng cấu hình trong `dovecot.conf`:
```
log_path = /var/log/dovecot.log
info_log_path = /var/log/dovecot-info.log
```

**d) Logs Anti-Spam (Amavis + SpamAssassin):**

Amavis ghi log vào `mail.log`, nhưng cũng có log riêng tại `/var/log/amavis/`. Trong đó có thể thấy rõ lý do một email bị đánh dấu spam và điểm số chi tiết.

### 11.2. Đọc và phân tích một dòng Log thực tế

Một dòng log Postfix điển hình trông như thế này:

```
Jun 23 15:30:45 mailserver postfix/smtp[12345]: B4F1D3A8: to=<recipient@gmail.com>, relay=gmail-smtp-in.l.google.com[142.250.4.26]:25, delay=1.2, delays=0.1/0.0/0.8/0.3, dsn=2.0.0, status=sent (250 2.0.0 OK  1719153045 abc123-xyz.1 - gsmtp)
```

Phân tích từng phần:

| Phần | Giá trị | Ý nghĩa |
|---|---|---|
| Timestamp | `Jun 23 15:30:45` | Thời gian gửi |
| Hostname | `mailserver` | Tên máy chủ |
| Daemon | `postfix/smtp[12345]` | Module Postfix và PID |
| Queue ID | `B4F1D3A8` | ID duy nhất của email này trong queue |
| to= | `recipient@gmail.com` | Địa chỉ người nhận |
| relay= | `gmail-smtp-in...` | Máy chủ đích đã nhận thư |
| delay= | `1.2` | Tổng thời gian xử lý (giây) |
| delays= | `0.1/0.0/0.8/0.3` | Thời gian từng giai đoạn: nhận/queue/kết nối/gửi |
| dsn= | `2.0.0` | Delivery Status Notification code (2.x.x = thành công) |
| status= | `sent` | Kết quả: sent / deferred / bounced |

**Ví dụ log khi thư bị từ chối:**
```
Jun 23 15:31:02 mailserver postfix/smtp[12346]: C5G2E4B9: to=<user@yahoo.com>, relay=mta5.am0.yahoodns.net[98.137.65.52]:25, delay=5.3, delays=0.1/0.0/4.9/0.3, dsn=5.7.9, status=bounced (554 5.7.9 Message not accepted for policy reasons. See https://help.yahoo.com/kb/postmaster)
```

Đây là trường hợp Yahoo từ chối email do vi phạm chính sách (có thể do thiếu SPF/DKIM hoặc IP bị đưa vào blacklist).

### 11.3. Công cụ CLI tra cứu Logs

```bash
# Xem 50 dòng log gần nhất của Postfix theo thời gian thực
$ journalctl -u postfix -n 50 -f

# Tìm tất cả giao dịch liên quan đến một email cụ thể
$ grep "user@example.com" /var/log/mail.log

# Xem toàn bộ hành trình của một email bằng Queue ID
$ grep "B4F1D3A8" /var/log/mail.log

# Đếm số email bị bounce trong ngày hôm nay
$ grep "status=bounced" /var/log/mail.log | grep "$(date +"%b %d")" | wc -l

# Tìm các lỗi xác thực (brute force detection)
$ grep "authentication failed" /var/log/mail.log | awk '{print $NF}' | sort | uniq -c | sort -rn | head -20

# Tra cứu trong file log cũ đã nén mà không cần giải nén
$ zgrep "user@example.com" /var/log/mail.log.2.gz

# Lọc và hiển thị chỉ email bị deferred (kẹt queue)
$ grep "status=deferred" /var/log/mail.log | tail -50

# Thống kê nhanh: top 10 domain đích nhận thư từ server
$ grep "status=sent" /var/log/mail.log | grep -oP "relay=\K[^[]*" | sort | uniq -c | sort -rn | head -10
```

### 11.4. Hệ thống giám sát tập trung - ELK Stack

Khi hệ thống có nhiều máy chủ hoặc lượng log quá lớn để phân tích bằng tay, giải pháp tập trung hóa log là bắt buộc.

**ELK Stack** (Elasticsearch + Logstash + Kibana) là bộ công cụ tiêu chuẩn ngành:

*   **Logstash / Filebeat:** Chạy trên từng mail server, thu thập và chuyển tiếp log về server trung tâm. Filebeat nhẹ hơn và phù hợp hơn để chạy liên tục.
*   **Logstash (parser):** Phân tích cú pháp (parse) từng dòng log, tách ra các trường có cấu trúc (timestamp, Queue ID, sender, recipient, status...).
*   **Elasticsearch:** Cơ sở dữ liệu tìm kiếm phân tán, lưu trữ và index toàn bộ log đã được phân tích. Cho phép tìm kiếm cực nhanh trên hàng tỷ dòng log.
*   **Kibana:** Giao diện web trực quan để xây dựng Dashboard và tìm kiếm log.

**Ví dụ các Dashboard Kibana hữu ích cho email admin:**

*   Biểu đồ thời gian thực: số email gửi thành công / deferred / bounced theo giờ.
*   Top 10 địa chỉ IP gửi thư đến server (phát hiện spam nguồn).
*   Tỷ lệ email bị SpamAssassin đánh dấu là spam qua thời gian.
*   Bản đồ địa lý các IP đang tấn công brute-force vào Dovecot.
*   Cảnh báo (Alert) tự động gửi email/Slack khi queue vượt ngưỡng hoặc phát hiện spike đột biến.

---

## 12. QUẢN LÝ HÀNG ĐỢI THƯ - MAIL QUEUE

### 12.1. Cơ chế và các trạng thái của Mail Queue

**Mail Queue** là cơ chế đệm (buffer) cho phép hệ thống tiếp tục nhận thư ngay cả khi máy chủ đích tạm thời không truy cập được. Thay vì mất thư, MTA sẽ lưu thư vào queue và thử gửi lại theo lịch.

**Bốn hàng đợi nội bộ của Postfix:**

| Queue | Vị trí trên đĩa | Vai trò |
|---|---|---|
| `maildrop` | `/var/spool/postfix/maildrop/` | Nơi tiếp nhận thư từ local (lệnh `sendmail`) ban đầu |
| `incoming` | `/var/spool/postfix/incoming/` | Thư mới đến, chờ được xử lý bởi queue manager |
| `active` | `/var/spool/postfix/active/` | Thư đang được xử lý/gửi đi, kích thước có giới hạn |
| `deferred` | `/var/spool/postfix/deferred/` | Thư thất bại tạm thời, chờ thử lại |
| `bounce` | `/var/spool/postfix/bounce/` | Chứa thông tin của thư bị bounce |

**Lịch thử lại (Retry Schedule):**

Postfix không thử lại liên tục mà tăng dần khoảng thời gian chờ theo thuật toán exponential backoff:
*   Lần 1: Thử lại sau 5 phút.
*   Lần 2: Thử lại sau 10 phút.
*   Lần 3: Thử lại sau 30 phút.
*   Tiếp tục tăng dần...
*   Sau thời gian `maximal_queue_lifetime` (mặc định 5 ngày), thư bị coi là bounce và gửi thông báo lỗi về cho người gửi.

**Các nguyên nhân thư bị kẹt trong queue:**

| Trạng thái | Lý do phổ biến |
|---|---|
| Connection timed out | Máy chủ đích không phản hồi (sập hoặc chặn cổng 25) |
| Host not found | DNS của domain đích không có bản ghi MX hợp lệ |
| 4xx Temporary failure | Máy chủ đích báo lỗi tạm thời (đầy disk, quá tải...) |
| connect to... refused | Cổng 25 bị tường lửa đích chặn |
| 421 Too many connections | Máy chủ đích giới hạn số kết nối đồng thời |

### 12.2. Kiểm tra và xử lý Queue trên Postfix

**Xem nội dung queue:**
```bash
# Liệt kê tất cả thư trong queue
$ postqueue -p
# Hoặc tương đương:
$ mailq

# Output mẫu:
-Queue ID-  --Size-- ----Arrival Time---- -Sender/Recipient-------
B4F1D3A8       1420 Mon Jun 23 15:00:21  sender@domain.com
(connect to gmail-smtp-in.l.google.com[142.250.4.26]:25: Connection timed out)
                                          recipient@gmail.com

C5G2E4B9       2150 Mon Jun 23 15:05:10  marketing@domain.com
(Host or domain name not found)
                                          user@nonexistent-domain.com

-- 2 Kbytes in 2 Requests.
```

**Đọc nội dung chi tiết của một email trong queue:**
```bash
# Xem header và body đầy đủ của thư
$ postcat -q B4F1D3A8
```

**Các lệnh xử lý queue:**
```bash
# Ép gửi lại toàn bộ thư trong queue ngay lập tức
$ postqueue -f

# Xóa một thư cụ thể theo Queue ID
$ postsuper -d B4F1D3A8

# Xóa toàn bộ thư trong queue deferred (thư đang kẹt)
# CẢNH BÁO: Thao tác không thể hoàn tác!
$ postsuper -d ALL deferred

# Xóa toàn bộ queue (bao gồm cả active) - Rất nguy hiểm
$ postsuper -d ALL

# Di chuyển tất cả thư active về deferred (tạm dừng gửi)
$ postsuper -r ALL active

# Tái xếp hàng (re-queue) toàn bộ thư deferred để gửi lại ngay
$ postsuper -r ALL deferred
```

**Xử lý tình huống server bị hack spam hàng loạt:**
```bash
# Bước 1: Tìm địa chỉ email đang spam nhiều nhất
$ mailq | grep "^[A-F0-9]" | awk '{print $7}' | sort | uniq -c | sort -rn | head -20

# Bước 2: Xóa toàn bộ thư trong queue từ email đó
$ postqueue -p | grep -B1 "spammer@domain.com" | grep "^[A-F0-9]" | cut -d! -f1 | postsuper -d -

# Bước 3: Khóa tài khoản và đổi mật khẩu ngay lập tức
$ passwd username

# Bước 4: Kiểm tra xem tài khoản còn đang gửi thư không
$ tail -f /var/log/mail.log | grep "spammer@domain.com"
```

**Phân tích queue theo domain đích với qshape:**
```bash
$ qshape deferred
                           T  5 10 20 40 80
TOTAL                     15  0  0  3  5  7
  gmail.com                8  0  0  1  3  4
  yahoo.com                4  0  0  2  2  0
  hotmail.com              3  0  0  0  0  3
```

Công cụ `qshape` cho thấy có bao nhiêu thư đang kẹt gửi đến từng domain theo thời gian kẹt. Nếu tất cả thư kẹt đều hướng về `gmail.com`, nhiều khả năng Gmail đang chặn IP của bạn.

### 12.3. Kiểm tra và xử lý Queue trên Exim

```bash
# Xem queue
$ exim -bp

# Xem chi tiết một thư
$ exim -Mvh MESSAGE_ID
$ exim -Mvb MESSAGE_ID

# Xóa một thư cụ thể
$ exim -Mrm MESSAGE_ID

# Xóa toàn bộ queue (thận trọng!)
$ exim -bp | awk '/^ *[0-9]+[mhd]/{print "exim -Mrm " $3}' | bash

# Ép gửi lại một thư cụ thể
$ exim -M MESSAGE_ID

# Ép gửi lại toàn bộ queue
$ exim -qff

# Đếm số thư trong queue
$ exim -bpc
```

### 12.4. Công cụ giám sát Queue chuyên nghiệp

**pflogsumm - Báo cáo hàng ngày cho Postfix:**

```bash
# Cài đặt
$ apt install pflogsumm   # Ubuntu/Debian
$ yum install postfix-perl-scripts  # CentOS/RHEL

# Tạo báo cáo từ log hôm qua
$ pflogsumm /var/log/mail.log.1

# Ví dụ output mẫu:
Grand Totals
------------
messages

   3421   received
   3398   delivered
      8   forwarded
     15   deferred  (47 deferrals)
      0   bounced
     23   rejected (0%)

   6821   bytes received
  6.512m  bytes delivered

Senders by message count
------------------------
     215   marketing@domain.com
     189   no-reply@domain.com
      87   user@domain.com
```

**Thiết lập báo cáo pflogsumm tự động qua cron:**
```bash
# Mở crontab
$ crontab -e

# Gửi báo cáo email hàng ngày lúc 6:00 sáng
0 6 * * * /usr/sbin/pflogsumm -d yesterday /var/log/mail.log | mail -s "Mail Report $(date +%F)" admin@domain.com
```

**Postfix Admin - Giao diện web quản lý:**

Postfix Admin là ứng dụng web PHP cho phép quản lý các domain ảo, hòm thư và bí danh (alias) của Postfix qua giao diện đồ họa. Đây là giải pháp phổ biến cho các nhà cung cấp hosting nhỏ.

**MailWatch - Giám sát Amavis qua Web:**

MailWatch cung cấp giao diện web để theo dõi toàn bộ hoạt động của Amavis (Anti-Spam / Anti-Virus). Quản trị viên có thể:
*   Tìm kiếm email theo người gửi/nhận, thời gian, điểm số spam.
*   Xem lý do chi tiết tại sao một email bị đánh dấu là spam.
*   Giải phóng (release) email từ khu vực cách ly (quarantine).
*   Theo dõi các thống kê spam/virus theo thời gian thực.

---

