# BÁO CÁO THỰC TẬP NGÀY 16/6

---

# BẢO MẬT VÀ HIGH AVAILABILITY CHO DATABASE SERVER

---

## MỤC LỤC

- [Phần V: Bảo Mật Database Server](#phần-v-bảo-mật-database-server)
  - [V.1. Các mối đe dọa bảo mật phổ biến](#v1-các-mối-đe-dọa-bảo-mật-phổ-biến)
    - [V.1.1. SQL Injection](#v11-sql-injection)
    - [V.1.2. Privilege Escalation (Leo thang đặc quyền)](#v12-privilege-escalation-leo-thang-đặc-quyền)
    - [V.1.3. Data Breaches (Rò rỉ dữ liệu)](#v13-data-breaches-rò-rỉ-dữ-liệu)
    - [V.1.4. Insider Threats (Mối đe dọa nội bộ)](#v14-insider-threats-mối-đe-dọa-nội-bộ)
    - [V.1.5. Brute Force và Credential Stuffing](#v15-brute-force-và-credential-stuffing)
    - [V.1.6. Misconfiguration (Cấu hình sai)](#v16-misconfiguration-cấu-hình-sai)
  - [V.2. Mã hóa dữ liệu (Encryption)](#v2-mã-hóa-dữ-liệu-encryption)
    - [V.2.1. Encryption at Rest (Mã hóa dữ liệu lưu trữ)](#v21-encryption-at-rest-mã-hóa-dữ-liệu-lưu-trữ)
    - [V.2.2. Encryption in Transit (Mã hóa dữ liệu truyền tải)](#v22-encryption-in-transit-mã-hóa-dữ-liệu-truyền-tải)
  - [V.3. Audit và theo dõi truy cập](#v3-audit-và-theo-dõi-truy-cập)
    - [V.3.1. Các sự kiện cần ghi lại](#v31-các-sự-kiện-cần-ghi-lại)
    - [V.3.2. Công cụ Audit theo từng hệ quản trị](#v32-công-cụ-audit-theo-từng-hệ-quản-trị)
    - [V.3.3. Tích hợp SIEM và cảnh báo tập trung](#v33-tích-hợp-siem-và-cảnh-báo-tập-trung)
  - [V.4. Patch management và cập nhật bảo mật](#v4-patch-management-và-cập-nhật-bảo-mật)
    - [V.4.1. Quy trình patch management](#v41-quy-trình-patch-management)
    - [V.4.2. Thực tiễn tốt trong patch management](#v42-thực-tiễn-tốt-trong-patch-management)

- [Phần VI: High Availability và Scalability](#phần-vi-high-availability-và-scalability)
  - [VI.1. Replication (Đồng bộ hóa dữ liệu)](#vi1-replication-đồng-bộ-hóa-dữ-liệu)
    - [VI.1.1. Replication trong MySQL](#vi11-replication-trong-mysql)
    - [VI.1.2. Replication trong SQL Server](#vi12-replication-trong-sql-server)
    - [VI.1.3. Replication trong MongoDB](#vi13-replication-trong-mongodb)
  - [VI.2. Cluster và Failover](#vi2-cluster-và-failover)
    - [VI.2.1. MySQL InnoDB Cluster](#vi21-mysql-innodb-cluster)
    - [VI.2.2. MongoDB Sharded Cluster và Failover](#vi22-mongodb-sharded-cluster-và-failover)
  - [VI.3. Sharding (Phân mảnh dữ liệu)](#vi3-sharding-phân-mảnh-dữ-liệu)
    - [VI.3.1. Khái niệm và kiến trúc Sharding](#vi31-khái-niệm-và-kiến-trúc-sharding)
    - [VI.3.2. Các chiến lược Sharding](#vi32-các-chiến-lược-sharding)
    - [VI.3.3. Lưu ý khi thiết kế Shard Key](#vi33-lưu-ý-khi-thiết-kế-shard-key)
  - [VI.4. Load Balancing cho Database](#vi4-load-balancing-cho-database)
    - [VI.4.1. Kiến trúc Load Balancing trong hệ thống Database](#vi41-kiến-trúc-load-balancing-trong-hệ-thống-database)
    - [VI.4.2. Các công cụ Load Balancing phổ biến](#vi42-các-công-cụ-load-balancing-phổ-biến)

---

## PHẦN V: BẢO MẬT DATABASE SERVER

Database server là trung tâm lưu trữ toàn bộ dữ liệu quan trọng của một hệ thống, từ thông tin người dùng, tài chính, đến dữ liệu vận hành. Vì vậy, đây cũng là mục tiêu tấn công hàng đầu của các tác nhân độc hại. Bảo mật database không chỉ đơn giản là đặt mật khẩu mạnh — nó đòi hỏi một chiến lược đa lớp bao gồm kiểm soát truy cập, mã hóa, giám sát liên tục, và quản lý vá lỗi chủ động.

---

### V.1. Các mối đe dọa bảo mật phổ biến

#### V.1.1. SQL Injection

SQL Injection (SQLi) là một trong những lỗ hổng bảo mật lâu đời nhất nhưng vẫn xuất hiện phổ biến nhất trong các ứng dụng web hiện nay. Kẻ tấn công lợi dụng việc ứng dụng không kiểm tra kỹ đầu vào từ người dùng để chèn mã SQL độc hại vào các câu truy vấn được gửi đến database.

**Cơ chế tấn công:**

Giả sử ứng dụng có đoạn code đăng nhập như sau:

```sql
SELECT * FROM users WHERE username = '$input_user' AND password = '$input_pass';
```

Nếu kẻ tấn công nhập vào trường `username` giá trị:

```
' OR '1'='1
```

Câu query trở thành:

```sql
SELECT * FROM users WHERE username = '' OR '1'='1' AND password = '...';
```

Do điều kiện `'1'='1'` luôn đúng, hệ thống sẽ trả về toàn bộ bảng `users`, bỏ qua xác thực mật khẩu. Ở mức độ nguy hiểm hơn, kẻ tấn công có thể dùng `UNION SELECT` để đọc dữ liệu từ các bảng khác, hoặc gọi các hàm hệ thống để thực thi lệnh trên máy chủ.

**Phòng chống SQL Injection:**

- Sử dụng **Prepared Statements / Parameterized Queries**: Tách biệt hoàn toàn phần cấu trúc SQL và phần dữ liệu người dùng nhập vào.
- Áp dụng **Stored Procedures** được viết an toàn.
- Thực hiện **Input Validation** và **Whitelist** các ký tự được phép.
- Cấp quyền tối thiểu cho tài khoản kết nối database (Principle of Least Privilege).
- Sử dụng **WAF (Web Application Firewall)** để lọc các request có dấu hiệu SQLi.

**Demo:**
Giả sử đăng nhập và sử dụng sqli để tấn công trang sau, như có thể thấy noti đằng sau hệ thống không đăng nhập được 

<img width="1703" height="718" alt="image" src="https://github.com/user-attachments/assets/42ed881c-1a4c-4400-bd98-f93e6b72a0cd" />

Nếu thêm 1 dấu ' đằng sau password , truy vấn sẽ như thế này, báo lỗi  
<img width="1645" height="650" alt="image" src="https://github.com/user-attachments/assets/a98b1bf0-ea4e-4599-8620-2bc0f780ef2f" />


<img width="1720" height="700" alt="image" src="https://github.com/user-attachments/assets/92be4593-9c5e-4f34-8dbb-ed7de09174b1" />

sử dụng câu lệnh sqli để pass đăng nhập
<img width="1625" height="698" alt="image" src="https://github.com/user-attachments/assets/bc1d197a-5668-4083-82e6-29ea56043619" />


<img width="1625" height="698" alt="image" src="https://github.com/user-attachments/assets/69a402ec-c4d0-41e1-afe7-373de5466b7f" />
Do điều kiện `'1'='1'` luôn đúng, hệ thống sẽ trả về toàn bộ bảng `users`, bỏ qua xác thực mật khẩu. Ở mức độ nguy hiểm hơn, kẻ tấn công có thể dùng `UNION SELECT` để đọc dữ liệu từ các bảng khác, hoặc gọi các hàm hệ thống để thực thi lệnh trên máy chủ.

<img width="1681" height="589" alt="image" src="https://github.com/user-attachments/assets/0c864edc-27fc-44b8-ae2f-712fce57c902" />

---

#### V.1.2. Privilege Escalation (Leo thang đặc quyền)

Privilege Escalation xảy ra khi một người dùng hoặc tiến trình cố gắng giành được mức quyền truy cập cao hơn so với quyền ban đầu được cấp. Trong môi trường database, điều này có thể xảy ra theo hai hướng:

- **Vertical Escalation**: Người dùng thông thường leo thang lên quyền `DBA` hoặc `SUPERUSER`.
- **Horizontal Escalation**: Người dùng truy cập vào dữ liệu của tài khoản khác cùng cấp.

**Nguyên nhân phổ biến:**

- Cấu hình quyền quá rộng (ví dụ: cấp `GRANT ALL PRIVILEGES` thay vì chỉ `SELECT`).
- Lỗ hổng trong stored procedure hoặc trigger cho phép thực thi với quyền của owner.
- Tài khoản service bị lợi dụng để chạy lệnh với quyền cao hơn.
- Lỗ hổng trong phần mềm database chưa được vá.

**Phòng chống:**

- Áp dụng **Principle of Least Privilege** — mỗi tài khoản chỉ có đúng quyền cần thiết cho công việc của mình.
- Kiểm tra định kỳ danh sách quyền của tất cả tài khoản.
- Phân tách tài khoản theo vai trò: tài khoản đọc, tài khoản ghi, tài khoản quản trị.
- Vô hiệu hóa các tài khoản mặc định có quyền cao (ví dụ: `sa` trong SQL Server, `root` trong MySQL nên bị hạn chế truy cập từ xa).

---

#### V.1.3. Data Breaches (Rò rỉ dữ liệu)

Data Breach là sự kiện dữ liệu nhạy cảm bị tiết lộ, truy cập, hoặc đánh cắp bởi người không có thẩm quyền. Database là mục tiêu chính vì nó tập trung toàn bộ tài sản thông tin của tổ chức.

**Nguyên nhân phổ biến dẫn đến Data Breach:**

- Database để lộ cổng kết nối (port) ra internet mà không có tường lửa bảo vệ.
- Sử dụng mật khẩu mặc định hoặc mật khẩu yếu.
- Không mã hóa dữ liệu nhạy cảm khi lưu trữ.
- Backup database không được bảo vệ và bị truy cập trái phép.
- Lỗ hổng trong ứng dụng cho phép đọc dữ liệu ngoài phạm vi cho phép.

**Biện pháp phòng ngừa:**

- Không bao giờ expose database port trực tiếp ra internet. Database chỉ nên nhận kết nối từ application server thông qua mạng nội bộ hoặc VPN.
- Mã hóa toàn bộ dữ liệu nhạy cảm (xem mục V.2).
- Bảo vệ file backup bằng mã hóa và kiểm soát truy cập chặt chẽ.
- Thực hiện kiểm tra bảo mật định kỳ (Penetration Testing).

---

#### V.1.4. Insider Threats (Mối đe dọa nội bộ)

Insider Threat là mối đe dọa đến từ chính những người bên trong tổ chức — nhân viên, nhà thầu, cựu nhân viên — những người có hoặc từng có quyền truy cập hợp lệ vào hệ thống. Đây là loại mối đe dọa khó phát hiện nhất vì tác nhân đã vượt qua lớp xác thực ngoài cùng.

**Phân loại Insider Threat:**

- **Malicious Insider**: Cố ý đánh cắp, phá hoại, hoặc tiết lộ dữ liệu vì động cơ cá nhân.
- **Negligent Insider**: Vô ý gây ra sự cố bảo mật do thiếu cẩn thận (ví dụ: để lộ thông tin đăng nhập, tải dữ liệu về thiết bị cá nhân không được bảo mật).
- **Compromised Insider**: Tài khoản của nhân viên bị đánh cắp và bị kẻ bên ngoài sử dụng.

**Phòng chống:**

- Phân quyền theo nguyên tắc **Need-to-Know**: chỉ cho phép truy cập dữ liệu mà người đó thực sự cần cho công việc.
- Theo dõi hành vi truy cập bất thường qua audit log (xem mục V.3).
- Thu hồi quyền ngay lập tức khi nhân viên nghỉ việc hoặc chuyển vai trò.
- Áp dụng **Data Loss Prevention (DLP)** để ngăn xuất dữ liệu ra ngoài qua email, USB, hoặc upload.

---

#### V.1.5. Brute Force và Credential Stuffing

**Brute Force** là phương pháp thử lần lượt tất cả các tổ hợp mật khẩu cho đến khi tìm đúng. **Credential Stuffing** là một biến thể tinh vi hơn: kẻ tấn công sử dụng danh sách tên đăng nhập và mật khẩu đã bị rò rỉ từ các vụ breach trước đó để thử đăng nhập vào hệ thống khác, khai thác thói quen dùng lại mật khẩu của người dùng.

**Phòng chống:**

- Giới hạn số lần đăng nhập sai (Account Lockout Policy): sau N lần thất bại sẽ khóa tài khoản tạm thời.
- Bật **Multi-Factor Authentication (MFA)** cho tất cả tài khoản quản trị.
- Giám sát và cảnh báo khi có nhiều lần đăng nhập thất bại liên tiếp từ cùng một IP.
- Không cho phép truy cập database management port (3306, 1433, 27017...) từ địa chỉ IP công cộng.
- Sử dụng mật khẩu mạnh, duy nhất cho mỗi hệ thống, và quản lý bằng Password Manager.

---

#### V.1.6. Misconfiguration (Cấu hình sai)

Misconfiguration là nguyên nhân gốc rễ của rất nhiều sự cố bảo mật database. Một hệ thống được cấu hình sai sẽ tạo ra lỗ hổng ngay cả khi phần mềm không có lỗi.

**Các lỗi cấu hình phổ biến:**

- Để nguyên mật khẩu mặc định của tài khoản `root`, `sa`, hoặc `admin`.
- Mở port database ra internet mà không cần thiết.
- Cấp quyền `GRANT ALL` thay vì quyền tối thiểu cần thiết.
- Không tắt các tính năng nguy hiểm không dùng đến (ví dụ: `xp_cmdshell` trong SQL Server, `FILE` privilege trong MySQL).
- Không bật mã hóa kết nối (cho phép kết nối không mã hóa).
- Lưu file cấu hình chứa thông tin đăng nhập database dưới dạng plaintext và không giới hạn quyền đọc file.

**Phòng chống:**

- Tuân theo **CIS Benchmarks** hoặc **DISA STIGs** — bộ hướng dẫn cấu hình bảo mật tiêu chuẩn cho từng loại database.
- Thực hiện **Configuration Audit** định kỳ để phát hiện các cấu hình lệch khỏi baseline.
- Dùng công cụ tự động như **Lynis**, **DBScan**, hoặc **AWS Config Rules** để kiểm tra liên tục.

---

### V.2. Mã hóa dữ liệu (Encryption)

Mã hóa là lớp bảo vệ cuối cùng: ngay cả khi kẻ tấn công truy cập được vào file dữ liệu hoặc đánh chặn được lưu lượng mạng, họ cũng không thể đọc được nội dung nếu dữ liệu đã được mã hóa đúng cách. Chiến lược mã hóa toàn diện bao gồm hai phạm vi chính: mã hóa khi lưu trữ và mã hóa khi truyền tải.

#### V.2.1. Encryption at Rest (Mã hóa dữ liệu lưu trữ)

Encryption at Rest bảo vệ dữ liệu được lưu trên ổ đĩa khỏi bị đọc khi thiết bị lưu trữ bị đánh cắp hoặc bị truy cập vật lý trái phép.

**Transparent Data Encryption (TDE)**

TDE là phương pháp mã hóa toàn bộ file database một cách tự động, minh bạch với ứng dụng — tức là ứng dụng không cần thay đổi bất kỳ dòng code nào. Khi database engine ghi dữ liệu xuống đĩa, nó tự động mã hóa; khi đọc lên, nó tự động giải mã. Toàn bộ quá trình xảy ra trong bộ nhớ của database engine.

- **MySQL / MariaDB**: Hỗ trợ TDE từ MySQL 5.7.11 (gọi là InnoDB Tablespace Encryption). Key được quản lý qua plugin keyring (`keyring_file`, `keyring_okv`, v.v.).
- **SQL Server**: Hỗ trợ TDE từ SQL Server 2008. Sử dụng cơ chế phân cấp khóa: Database Encryption Key (DEK) được bảo vệ bởi certificate của master database.
- **PostgreSQL**: Không có TDE built-in ở mức engine; thường dùng mã hóa ở mức filesystem (dm-crypt/LUKS trên Linux) hoặc các extension của bên thứ ba.

**Column-level Encryption (Mã hóa theo cột)**

Trong nhiều trường hợp, không cần thiết phải mã hóa toàn bộ database. Thay vào đó, chỉ mã hóa các cột chứa thông tin cực kỳ nhạy cảm như số CMND/CCCD, số thẻ tín dụng (PAN), mật khẩu, số điện thoại. Đây là cách tiếp cận linh hoạt và có hiệu năng cao hơn TDE.

- **MySQL**: Sử dụng hàm `AES_ENCRYPT()` và `AES_DECRYPT()` tích hợp sẵn.
- **SQL Server**: Sử dụng `Always Encrypted` — cho phép mã hóa ngay tại phía client, database engine không bao giờ thấy dữ liệu plaintext.
- **MongoDB**: Sử dụng **Client-Side Field Level Encryption (CSFLE)** — mã hóa được thực hiện trong MongoDB driver trước khi gửi lên server.

**Chuẩn mã hóa AES-256**

AES (Advanced Encryption Standard) với độ dài khóa 256-bit là chuẩn mã hóa đối xứng được sử dụng phổ biến nhất, được chính phủ Mỹ và nhiều tổ chức chuẩn hóa quốc tế công nhận (FIPS 140-2). Với 2^256 tổ hợp khóa có thể có, việc bẻ khóa bằng brute force là bất khả thi về mặt tính toán với công nghệ hiện tại.

**Quản lý khóa mã hóa (Key Management)**

Chất lượng bảo mật của mã hóa phụ thuộc hoàn toàn vào việc bảo vệ khóa. Nguyên tắc cơ bản: **không bao giờ lưu khóa mã hóa cùng chỗ với dữ liệu được mã hóa**.

Các giải pháp quản lý khóa chuyên dụng:

| Giải pháp | Mô tả |
|-----------|-------|
| **HSM (Hardware Security Module)** | Thiết bị phần cứng chuyên dụng, lưu và xử lý khóa trong môi trường vật lý cách ly, không bao giờ xuất khóa ra ngoài |
| **AWS KMS** | Dịch vụ quản lý khóa của Amazon Web Services, tích hợp tốt với RDS, S3 |
| **Azure Key Vault** | Dịch vụ tương tự của Microsoft Azure, hỗ trợ cả secrets và certificates |
| **HashiCorp Vault** | Giải pháp open-source phổ biến cho môi trường on-premise và hybrid cloud |



---

#### V.2.2. Encryption in Transit (Mã hóa dữ liệu truyền tải)

Khi dữ liệu di chuyển qua mạng giữa ứng dụng và database, hoặc giữa các node database với nhau, nó có thể bị đánh chặn qua kỹ thuật Man-in-the-Middle (MitM). Encryption in Transit bảo vệ dữ liệu trong quá trình truyền.

**TLS/SSL**

Transport Layer Security (TLS) là giao thức tiêu chuẩn để mã hóa kết nối mạng. TLS là phiên bản kế nhiệm và an toàn hơn của SSL (SSL đã bị deprecated từ năm 2015 do nhiều lỗ hổng). Hầu hết các database hiện đại đều hỗ trợ bật kết nối TLS:

- **MySQL**: Cấu hình `require_secure_transport = ON` trong `my.cnf`.
- **SQL Server**: Bật "Force Encryption" trong SQL Server Configuration Manager.
- **MongoDB**: Cấu hình `net.tls.mode: requireTLS` trong `mongod.conf`.

Phiên bản TLS tối thiểu nên dùng là **TLS 1.2**; nên cập nhật lên **TLS 1.3** vì cải thiện đáng kể về bảo mật và hiệu năng so với phiên bản cũ.

**Certificate Validation**

Chỉ bật TLS chưa đủ — ứng dụng kết nối đến database cần xác thực certificate của server để đảm bảo đang kết nối đúng với database thật, không phải với một server giả mạo trong tấn công MitM. Client cần cấu hình để kiểm tra: certificate phải được ký bởi CA tin cậy, tên miền trong certificate phải khớp với địa chỉ server, và certificate chưa hết hạn.

**Mutual TLS (mTLS)**

Trong TLS thông thường, chỉ có client xác thực server. Mutual TLS (mTLS) yêu cầu cả hai chiều: server cũng xác thực certificate của client. Điều này đảm bảo rằng ngay cả khi ai đó biết địa chỉ và port của database, họ cũng không thể kết nối nếu không có certificate hợp lệ được cấp phát bởi tổ chức.

mTLS đặc biệt hữu ích trong môi trường microservices hoặc khi database nhận kết nối từ nhiều service khác nhau.

**VPN / Private Network**

Một lớp bảo vệ bổ sung quan trọng là đặt database trong mạng riêng (private network / VPC) và chỉ cho phép truy cập từ bên trong thông qua VPN. Điều này thu hẹp bề mặt tấn công (attack surface) xuống đến mức tối thiểu: ngay cả khi cấu hình TLS có lỗi, kẻ tấn công từ internet cũng không thể tiếp cận database.

---

### V.3. Audit và theo dõi truy cập

Audit log là cơ chế ghi lại các hoạt động xảy ra trên database. Audit phục vụ hai mục đích chính: phát hiện các hành vi đáng ngờ trong thời gian thực (detective control), và cung cấp bằng chứng để điều tra khi có sự cố (forensic evidence). Nhiều tiêu chuẩn tuân thủ như PCI DSS, HIPAA, ISO 27001 cũng yêu cầu bắt buộc phải có audit log.

#### V.3.1. Các sự kiện cần ghi lại

Không phải mọi hoạt động đều cần ghi lại — ghi quá nhiều sẽ gây tốn tài nguyên và khó phân tích. Dưới đây là danh sách các sự kiện quan trọng cần audit:

| Loại sự kiện | Thông tin cần ghi |
|---|---|
| **Đăng nhập / đăng xuất** | Thành công và thất bại, timestamp, địa chỉ IP nguồn, tên user |
| **DDL Operations** | CREATE, DROP, ALTER, TRUNCATE trên bảng, view, stored procedure |
| **DML nhạy cảm** | INSERT, UPDATE, DELETE trên các bảng chứa dữ liệu quan trọng |
| **Thay đổi quyền** | GRANT, REVOKE, thay đổi role, tạo/xóa tài khoản |
| **Truy cập dữ liệu nhạy cảm** | SELECT trên bảng chứa PII, dữ liệu tài chính, dữ liệu y tế |
| **Thay đổi cấu hình** | Thay đổi các tham số cấu hình hệ thống database |
| **Lỗi bảo mật** | Cố gắng truy cập bị từ chối, vi phạm quyền |

**Thông tin tối thiểu cần có trong mỗi bản ghi audit:**

- Timestamp (bao gồm múi giờ)
- Tên người dùng và tài khoản database
- Địa chỉ IP và port nguồn
- Tên đối tượng bị tác động (tên bảng, stored procedure...)
- Loại hành động
- Kết quả (thành công / thất bại)
- Nội dung câu lệnh (nếu có thể)

#### V.3.2. Công cụ Audit theo từng hệ quản trị

**MySQL Audit**

MySQL cung cấp tính năng audit qua hai cơ chế:

- **Audit Plugin chính thức** (thuộc MySQL Enterprise Edition): Ghi log chi tiết ra file, hỗ trợ lọc theo user, host, database, event type.
- **MariaDB Audit Plugin**: Là giải pháp tương tự nhưng miễn phí và tương thích với MySQL Community Edition. Cấu hình trong `my.cnf`:

```ini
[mysqld]
plugin-load-add = server_audit
server_audit_logging = ON
server_audit_events = CONNECT, QUERY, TABLE
server_audit_file_path = /var/log/mysql/audit.log
```

**SQL Server Audit**

SQL Server cung cấp tính năng Audit tích hợp sẵn ở hai cấp độ:

- **Server Audit**: Ghi các sự kiện ở cấp độ server (đăng nhập, thay đổi cấu hình...).
- **Database Audit Specification**: Ghi các sự kiện ở cấp độ database cụ thể (truy cập bảng, thay đổi dữ liệu...).

Log có thể được ghi vào Windows Event Log, Security Log, hoặc file nhị phân. Có thể dùng T-SQL để truy vấn log:

```sql
SELECT * FROM sys.fn_get_audit_file('C:\AuditLogs\*.sqlaudit', DEFAULT, DEFAULT);
```

**MongoDB Audit Log**

MongoDB Enterprise hỗ trợ Audit Log với khả năng ghi ra nhiều định dạng và đích khác nhau. Cấu hình trong `mongod.conf`:

```yaml
auditLog:
  destination: file
  format: JSON
  path: /var/log/mongodb/audit.json
  filter: '{ atype: { $in: ["authenticate", "createCollection", "dropCollection", "createUser", "dropUser", "grantRolesToUser"] } }'
```


#### V.3.3. Tích hợp SIEM và cảnh báo tập trung

Ghi audit log trên từng database riêng lẻ chỉ là bước đầu. Trong môi trường production với nhiều database server, việc xem xét từng file log thủ công là không khả thi. Giải pháp là tập trung log về một hệ thống **SIEM (Security Information and Event Management)**.

**Luồng xử lý log điển hình:**

```
Database Audit Log
      |
      v
Log Shipper (Filebeat / Fluentd / Telegraf)
      |
      v
Log Aggregator (Logstash / Vector)
      |
      +---> SIEM (Splunk / IBM QRadar / Microsoft Sentinel)
      |          (Phân tích, tương quan sự kiện, cảnh báo)
      |
      +---> Log Storage (Elasticsearch / Loki / S3)
                 (Lưu trữ dài hạn để điều tra)
```

**Các quy tắc cảnh báo nên thiết lập:**

- Đăng nhập thất bại liên tiếp hơn N lần trong M phút từ cùng một IP.
- Đăng nhập từ địa lý bất thường (GeoIP alert).
- Truy vấn SELECT trả về số lượng row bất thường lớn (có thể là data exfiltration).
- Truy cập database ngoài giờ làm việc bởi tài khoản không phải service account.
- Thay đổi quyền hoặc tạo tài khoản mới.
- DROP TABLE hoặc TRUNCATE TABLE bất kỳ.

---

### V.4. Patch management và cập nhật bảo mật

Phần mềm database — cũng như mọi phần mềm khác — liên tục được phát hiện có lỗ hổng bảo mật mới. Patch management là quy trình có hệ thống để theo dõi, đánh giá và áp dụng các bản vá bảo mật nhằm đóng các lỗ hổng đó trước khi kẻ tấn công khai thác.

#### V.4.1. Quy trình patch management

**Bước 1: Theo dõi CVE (Common Vulnerabilities and Exposures)**

CVE là hệ thống định danh chuẩn cho các lỗ hổng bảo mật đã được công khai. Mỗi lỗ hổng được gán một mã CVE (ví dụ: CVE-2024-21096 là lỗ hổng trong MySQL Server năm 2024).

Nguồn theo dõi:
- **NVD (National Vulnerability Database)**: nvd.nist.gov — cơ sở dữ liệu lỗ hổng toàn diện nhất.
- **Security advisories của vendor**: MySQL Security Advisories, Microsoft SQL Server Security Bulletin, MongoDB Security Advisories.
- **Các dịch vụ theo dõi CVE**: Tenable, Qualys, Rapid7 cung cấp feed tự động theo hệ thống đang dùng.

**Bước 2: Đánh giá mức độ rủi ro**

Không phải mọi CVE đều cần xử lý ngay lập tức. Cần đánh giá theo các tiêu chí:

- **CVSS Score (Common Vulnerability Scoring System)**: Thang điểm từ 0-10, phân loại Critical (9.0-10.0), High (7.0-8.9), Medium (4.0-6.9), Low (0-3.9).
- **Khả năng khai thác**: Lỗ hổng có exploit công khai chưa? Khai thác có cần xác thực không?
- **Mức độ ảnh hưởng**: Lỗ hổng này ảnh hưởng đến tính năng/module nào đang được sử dụng?
- **Biện pháp giảm thiểu tạm thời**: Có workaround nào không cần patch ngay không?

**Bước 3: Test trên môi trường phi production**

Trước khi triển khai lên production, bắt buộc phải test patch trên môi trường staging/dev:

- Áp dụng patch và kiểm tra database khởi động bình thường.
- Chạy bộ test tự động (regression tests) của ứng dụng để đảm bảo không có tính năng bị hỏng.
- Kiểm tra hiệu năng trước và sau khi patch.
- Kiểm tra tính tương thích với các driver, connector phía ứng dụng.

**Bước 4: Triển khai có kiểm soát lên production**

- Lên lịch trong **maintenance window** — khung giờ ít người dùng nhất, thường ban đêm hoặc cuối tuần.
- Thực hiện **full backup** database trước khi bắt đầu.
- Chuẩn bị **rollback plan** chi tiết: nếu có sự cố sau khi patch, cần khôi phục trong bao nhiêu phút?
- Với các hệ thống có High Availability: patch lần lượt từng node (rolling update) để không gây downtime.

**Bước 5: Xác minh và ghi nhật ký thay đổi**

- Xác nhận phiên bản mới đã được cài đặt thành công.
- Kiểm tra lại toàn bộ chức năng của ứng dụng sau khi patch.
- Ghi lại bản ghi Change Management: thời gian thực hiện, người thực hiện, phiên bản trước/sau, kết quả kiểm tra.
- Cập nhật vào hệ thống inventory/asset management.


#### V.4.2. Thực tiễn tốt trong patch management

- **Bật tự động cập nhật minor patches** (patch bảo mật không thay đổi tính năng): Hầu hết các distro Linux cho phép cấu hình `unattended-upgrades` để tự động áp dụng security patches.
- **Định kỳ review major version**: Không chỉ patch, cần lên kế hoạch nâng cấp lên major version mới khi version cũ đến End of Life.
- **Patch cả hệ điều hành và middleware**: Lỗ hổng trong OS, OpenSSL, hoặc các thư viện dependency cũng có thể ảnh hưởng đến bảo mật database.
- **Sử dụng IDS/IPS**: Triển khai hệ thống phát hiện/ngăn chặn xâm nhập để có thêm thời gian phản ứng trước khi patch kịp thời.

---

## PHẦN VI: HIGH AVAILABILITY VÀ SCALABILITY

**High Availability (HA)** là khả năng hệ thống duy trì hoạt động liên tục, tối thiểu hóa thời gian ngừng hoạt động (downtime) kể cả khi có sự cố phần cứng, phần mềm, hoặc lỗi vận hành. **Scalability** là khả năng mở rộng năng lực xử lý của hệ thống khi lượng dữ liệu và tải truy vấn tăng lên.

Mức độ sẵn sàng (availability) thường được đo bằng tỷ lệ phần trăm thời gian hoạt động trong năm. Ví dụ:

| Mức SLA | Uptime | Downtime cho phép/năm |
|---------|--------|----------------------|
| 99% | 99% | ~3.65 ngày |
| 99.9% ("Three nines") | 99.9% | ~8.76 giờ |
| 99.99% ("Four nines") | 99.99% | ~52.6 phút |
| 99.999% ("Five nines") | 99.999% | ~5.26 phút |

Để đạt được các mức SLA cao, hệ thống cần kết hợp Replication, Cluster, và Load Balancing.

---

### VI.1. Replication (Đồng bộ hóa dữ liệu)

Replication là cơ chế sao chép dữ liệu từ một database server (Primary/Master) sang một hoặc nhiều server khác (Secondary/Replica/Slave) theo thời gian thực hoặc gần thời gian thực. Replication phục vụ hai mục đích chính: tăng tính sẵn sàng (nếu Primary down, Replica có thể tiếp quản) và tăng khả năng mở rộng đọc (các truy vấn SELECT có thể được phân phối ra các Replica).

#### VI.1.1. Replication trong MySQL

MySQL hỗ trợ nhiều kiểu replication khác nhau, mỗi loại có đặc điểm và trường hợp sử dụng phù hợp.

**Kiến trúc cơ bản: Master-Slave (Source-Replica)**

Cơ chế hoạt động:

1. Primary (Master) ghi mọi thay đổi dữ liệu vào **Binary Log (binlog)** — đây là nhật ký tuần tự ghi lại tất cả các sự kiện thay đổi dữ liệu.
2. Secondary (Slave) có một tiến trình **I/O Thread** kết nối đến Primary và liên tục đọc các sự kiện mới từ binlog, lưu vào **Relay Log** ở phía Slave.
3. Slave có một tiến trình thứ hai là **SQL Thread** đọc từ Relay Log và thực thi lại các sự kiện đó để áp dụng thay đổi vào database của Slave.

**Các chế độ Replication:**

**Asynchronous Replication (Mặc định)**

Primary ghi dữ liệu và commit giao dịch mà không cần chờ xác nhận từ bất kỳ Slave nào. Slave nhận và áp dụng dữ liệu một cách bất đồng bộ, có thể bị trễ (replication lag) so với Primary.

- Ưu điểm: Hiệu năng ghi cao nhất, latency thấp cho ứng dụng.
- Nhược điểm: Nếu Primary đột ngột down, các giao dịch gần nhất chưa kịp replicate sẽ bị mất.

**Semi-Synchronous Replication**

Primary ghi dữ liệu vào binlog và gửi đến Slave, nhưng chỉ commit khi **ít nhất một Slave xác nhận đã nhận được** (không cần chờ Slave áp dụng). Nếu không có Slave nào xác nhận trong timeout, hệ thống tự động chuyển về Asynchronous.

- Ưu điểm: Đảm bảo dữ liệu đã được sao chép đến ít nhất một nơi khác trước khi commit.
- Nhược điểm: Tăng latency ghi một chút so với Async.

**GTID-based Replication (Global Transaction Identifier)**

Mỗi giao dịch được gán một GTID duy nhất trong toàn cluster (định dạng: `server_uuid:transaction_id`). Slave tự biết mình đã nhận đến GTID nào và cần lấy tiếp từ đâu, giúp đơn giản hóa đáng kể quá trình failover — không cần xác định thủ công binlog position như phương pháp truyền thống.

**Group Replication**

Đây là chế độ replication nâng cao nhất của MySQL, cho phép triển khai kiến trúc **multi-primary** — tất cả các node trong group đều có thể nhận giao dịch ghi. Các node đồng bộ với nhau qua giao thức **Paxos consensus** để đảm bảo nhất quán. Đây là nền tảng của MySQL InnoDB Cluster.

| Chế độ | Consistency | Hiệu năng ghi | Độ phức tạp |
|--------|-------------|---------------|-------------|
| Async | Yếu nhất | Cao nhất | Thấp |
| Semi-sync | Trung bình | Trung bình | Trung bình |
| GTID | Tùy thuộc chế độ | Tương tự chế độ nền | Thấp (dễ failover) |
| Group Replication | Mạnh nhất | Thấp hơn | Cao |

---

#### VI.1.2. Replication trong SQL Server

SQL Server cung cấp nhiều tính năng để đồng bộ dữ liệu, phù hợp với các kịch bản khác nhau từ HA đến phân phối dữ liệu.

**Always On Availability Groups (AG)**

Đây là giải pháp HA và DR (Disaster Recovery) hiện đại nhất của SQL Server, được giới thiệu từ SQL Server 2012. Một Availability Group bao gồm:

- **Primary Replica**: Nhận mọi kết nối đọc/ghi.
- **Secondary Replicas**: Nhận dữ liệu đồng bộ từ Primary, có thể được cấu hình để nhận truy vấn đọc (Read-Only Replica) nhằm phân tải.
- **Listener**: Một virtual name/IP cố định, ứng dụng kết nối vào Listener mà không cần biết node nào đang là Primary.

Dữ liệu được đồng bộ qua log stream từ Primary đến Secondary. Hỗ trợ cả chế độ Synchronous Commit (đảm bảo zero data loss, dùng trong cùng datacenter) và Asynchronous Commit (dùng cho Secondary ở datacenter xa).

**Demo:**

*(Hình ảnh minh họa: sơ đồ kiến trúc Always On AG với Listener, Primary, Secondary Replicas)*

**Transactional Replication**

Đây là phương pháp phân phối dữ liệu cụ thể theo cơ chế Publisher-Distributor-Subscriber:

- **Publisher**: Database nguồn, định nghĩa các Articles (bảng, stored procedure cần replicate) và Publication.
- **Distributor**: Trung gian lưu trữ và chuyển tiếp các thay đổi. Có thể là server riêng hoặc cùng server với Publisher.
- **Subscriber**: Database đích nhận dữ liệu được replicate.

Transactional Replication cho phép lọc dữ liệu theo row (row filter) hoặc column (column filter), rất phù hợp cho bài toán phân phối một tập con dữ liệu đến các nhánh hoặc chi nhánh.

**Log Shipping**

Log Shipping là phương pháp đơn giản nhất: định kỳ (thường từng 15-60 phút), Primary tự động backup transaction log và chuyển file backup đó đến Standby Server để restore. Đây là giải pháp DR cơ bản, chi phí thấp, nhưng không thể đạt Near-Zero RTO vì phụ thuộc vào chu kỳ backup.

---

#### VI.1.3. Replication trong MongoDB

MongoDB được thiết kế với Replica Set là đơn vị cơ bản của triển khai production, không có khái niệm "standalone server" trong môi trường production.

**Replica Set**

Một Replica Set gồm tối thiểu 3 node:

- **Primary**: Nhận mọi thao tác ghi. Ghi vào **Oplog (Operations Log)** — một capped collection đặc biệt ghi lại mọi thao tác thay đổi dữ liệu theo dạng idempotent.
- **Secondary (một hoặc nhiều)**: Liên tục sao chép từ Oplog của Primary và áp dụng lại.
- **Arbiter (tùy chọn)**: Node không lưu dữ liệu, chỉ tham gia bỏ phiếu trong quá trình bầu chọn Primary mới. Tiết kiệm tài nguyên.

**Cơ chế Oplog**

Oplog là capped collection (có giới hạn kích thước) trong database `local`. Mỗi thao tác ghi được ghi vào oplog dưới dạng một operation có thể áp dụng lại nhiều lần mà không thay đổi kết quả (idempotent). Secondary giữ con trỏ đến vị trí cuối cùng đã sync trong oplog của Primary và liên tục kéo thêm.

Nếu Secondary bị offline quá lâu và oplog của Primary đã bị ghi đè (vì oplog có giới hạn kích thước), Secondary sẽ rơi vào trạng thái **RECOVERING** và cần resync toàn bộ (initial sync) từ đầu.

**Bầu chọn Primary (Election)**

Khi Primary down hoặc không phản hồi, các Secondary tự động khởi động quá trình bầu chọn Primary mới theo giao thức **Raft consensus**:

1. Secondary phát hiện Primary không phản hồi sau `electionTimeoutMillis` (mặc định 10 giây).
2. Secondary tự đề cử mình làm ứng viên và yêu cầu bỏ phiếu từ các node khác.
3. Node được **đa số phiếu** (majority = n/2 + 1) sẽ trở thành Primary mới.
4. Quá trình thường hoàn thành trong vòng 10-30 giây.

Đây là lý do Replica Set cần tối thiểu 3 node (hoặc 2 node + 1 Arbiter): để có thể đạt majority vote khi một node down.

**Read Preference**

MongoDB cho phép cấu hình cách ứng dụng đọc dữ liệu qua `readPreference`:

| Read Preference | Hành vi |
|-----------------|---------|
| `primary` (mặc định) | Chỉ đọc từ Primary, đảm bảo đọc dữ liệu mới nhất |
| `primaryPreferred` | Ưu tiên Primary, dùng Secondary nếu Primary không khả dụng |
| `secondary` | Chỉ đọc từ Secondary, giảm tải cho Primary |
| `secondaryPreferred` | Ưu tiên Secondary, dùng Primary nếu không có Secondary |
| `nearest` | Đọc từ node có latency thấp nhất, bất kể Primary hay Secondary |

---

### VI.2. Cluster và Failover

Cluster là sự kết hợp của nhiều database node hoạt động như một hệ thống thống nhất, với khả năng tự động chuyển giao công việc (Failover) khi một node bị lỗi mà không cần can thiệp thủ công.

#### VI.2.1. MySQL InnoDB Cluster

MySQL InnoDB Cluster là giải phán HA tích hợp hoàn chỉnh của MySQL, bao gồm ba thành phần chính phối hợp với nhau:

**Group Replication**

Đây là lớp đồng bộ dữ liệu. Tất cả các node trong cluster đều là thành viên của một Group Replication group, đồng bộ dữ liệu qua Paxos consensus protocol. Mặc định chạy ở chế độ **Single-Primary** (chỉ một node nhận ghi tại một thời điểm). Cũng hỗ trợ chế độ **Multi-Primary** cho phép tất cả node nhận ghi đồng thời, phù hợp khi cần scale write.

**MySQL Router**

Đây là proxy layer đứng giữa ứng dụng và cluster. MySQL Router tự động:

- Phát hiện topology hiện tại của cluster (node nào là Primary, node nào là Secondary).
- Định tuyến các kết nối ghi đến Primary.
- Định tuyến các kết nối đọc đến Secondary (nếu cấu hình read-write splitting).
- Tự cập nhật routing khi topology thay đổi do failover.

Ứng dụng chỉ cần kết nối đến địa chỉ của MySQL Router, không cần biết địa chỉ IP cụ thể của từng node.

**MySQL Shell**

Công cụ command-line dùng để quản lý, deploy, và monitor InnoDB Cluster. Ví dụ khởi tạo cluster:

```javascript
// Trong MySQL Shell
var cluster = dba.createCluster('myCluster');
cluster.addInstance('user@node2:3306');
cluster.addInstance('user@node3:3306');
cluster.status();
```

**Cơ chế Failover**

Khi Primary node bị lỗi:

1. Các node còn lại phát hiện Primary không phản hồi.
2. Group Replication tổ chức bầu chọn Primary mới trong số các Secondary.
3. MySQL Router tự phát hiện thay đổi topology và chuyển hướng kết nối đến Primary mới.
4. Thời gian failover thường trong khoảng 20-30 giây.

**Yêu cầu tối thiểu**: InnoDB Cluster cần tối thiểu 3 node để đảm bảo quorum (đa số phiếu). Với 3 node, hệ thống chịu được tối đa 1 node lỗi; với 5 node chịu được 2 node lỗi.



#### VI.2.2. MongoDB Sharded Cluster và Failover

MongoDB Sharded Cluster là kiến trúc mở rộng quy mô ngang (horizontal scaling) cho MongoDB, phù hợp khi dữ liệu quá lớn để lưu trên một Replica Set đơn.

**Thành phần kiến trúc:**

**mongos (Query Router)**

Tiến trình mongos là điểm tiếp nhận mọi request từ ứng dụng. mongos không lưu trữ dữ liệu — nó đọc metadata từ Config Server để xác định request cần đến shard nào, sau đó định tuyến query và hợp nhất kết quả trả về cho client. Thường triển khai nhiều instance mongos để tránh single point of failure.

**Config Server Replica Set (CSRS)**

Config Server lưu trữ metadata của toàn bộ cluster: thông tin về các shard, bản đồ phân phối dữ liệu (chunk map), thông tin routing. Bản thân Config Server cũng phải là một Replica Set để đảm bảo HA của metadata. Thường triển khai 3 node Config Server.

**Shard (Replica Set)**

Mỗi shard là một Replica Set MongoDB độc lập, chứa một phần dữ liệu của toàn bộ database. Mỗi shard có Primary và Secondary của riêng nó, với cơ chế failover độc lập.

**Cơ chế Failover trong Sharded Cluster**

Khi Primary của một shard bị lỗi:
- Các Secondary trong Replica Set đó tự bầu chọn Primary mới (trong vòng 10-30 giây).
- mongos phát hiện sự thay đổi và bắt đầu route request đến Primary mới của shard.
- Trong thời gian failover, các write đến shard đó sẽ thất bại; read từ Secondary vẫn có thể tiếp tục nếu `readPreference` cho phép.


### VI.3. Sharding (Phân mảnh dữ liệu)

Sharding là kỹ thuật phân chia dữ liệu của một database thành nhiều phần nhỏ hơn (gọi là shard), mỗi phần được lưu trên một server riêng biệt. Khác với Replication (sao chép toàn bộ dữ liệu ra nhiều nơi), Sharding chia dữ liệu ra — mỗi shard chỉ chứa một tập con của toàn bộ dữ liệu.

**Khi nào cần Sharding?**

- Bộ dữ liệu quá lớn để lưu trên một server (vượt quá dung lượng ổ đĩa hoặc RAM).
- Tải ghi (write throughput) quá lớn để một server xử lý.
- Cần phân tán dữ liệu theo địa lý (data locality).

#### VI.3.1. Khái niệm và kiến trúc Sharding

**Shard Key**

Shard Key là trường (hoặc tổ hợp trường) trong document/row dùng để xác định dữ liệu thuộc shard nào. Việc chọn shard key là quyết định quan trọng nhất khi thiết kế sharding — một shard key tốt sẽ phân phối dữ liệu đều giữa các shard và không gây hotspot.

**Chunk**

MongoDB chia dữ liệu thành các **chunk** — mỗi chunk là một khoảng (range) của shard key values. Mặc định mỗi chunk có kích thước tối đa 128MB. Khi chunk quá lớn, **Balancer** (tiến trình chạy nền) sẽ tự động chia chunk và di chuyển chunk giữa các shard để duy trì phân phối đều.

#### VI.3.2. Các chiến lược Sharding

**Range Sharding (Phân mảnh theo khoảng)**

Dữ liệu được chia theo khoảng liên tục của shard key. Ví dụ: với shard key là `userID`, shard 1 chứa `userID` từ 1 đến 1,000,000; shard 2 chứa từ 1,000,001 đến 2,000,000; v.v.

- Ưu điểm: Hiệu quả cao với các truy vấn range query (`WHERE userID BETWEEN 500000 AND 700000` chỉ cần query một shard).
- Nhược điểm: Dễ gây **hotspot** nếu shard key có giá trị tăng dần theo thời gian (ví dụ: timestamp, auto-increment ID) — mọi insert mới đều vào shard cuối cùng.

**Hash Sharding (Phân mảnh theo hash)**

Giá trị của shard key được hash trước, sau đó kết quả hash được dùng để xác định shard. Ví dụ: `hash(userID) % số_shard`.

- Ưu điểm: Phân phối dữ liệu gần như hoàn toàn đều, không có hotspot.
- Nhược điểm: Không thể thực hiện range query hiệu quả — truy vấn `WHERE userID BETWEEN 500000 AND 700000` phải fan-out đến tất cả shard vì các giá trị liên tiếp bị hash đến các shard khác nhau.

**Zone Sharding / Tag-based Sharding (Phân mảnh theo vùng địa lý)**

Gắn các shard vào các "zone" địa lý hoặc logic, và chỉ định dữ liệu nào thuộc zone nào. Ví dụ: tất cả dữ liệu của khách hàng Việt Nam (`country = "VN"`) luôn được lưu trên shard đặt tại Asia data center; dữ liệu châu Âu lưu trên shard tại European data center.

- Ưu điểm: Giảm latency cho người dùng (dữ liệu gần với người dùng hơn), hỗ trợ data sovereignty requirements.
- Nhược điểm: Phân phối có thể không đều nếu tải không cân bằng giữa các vùng.

| Chiến lược | Phân phối đều | Range Query | Phù hợp với |
|------------|---------------|-------------|-------------|
| Range | Không (có thể hotspot) | Hiệu quả cao | Time-series data với query theo range |
| Hash | Tốt | Kém | OLTP workload với random access |
| Zone | Phụ thuộc | Trung bình | Yêu cầu data locality |

#### VI.3.3. Lưu ý khi thiết kế Shard Key

Đây là khuyến nghị thực tiễn quan trọng nhất khi triển khai sharding:

- **Tránh monotonically increasing key**: Timestamp, auto-increment ID luôn tăng dần → tất cả write vào shard cuối → hotspot nghiêm trọng. Nếu cần dùng timestamp, hãy kết hợp với một trường khác để tạo Compound Shard Key.
- **Chọn key có cardinality cao**: Shard key với ít giá trị phân biệt (ví dụ: trường `status` chỉ có 3 giá trị) sẽ không thể phân phối đều.
- **Cân nhắc query patterns**: Shard key nên xuất hiện trong phần lớn các câu query để tránh scatter-gather (query phải fan-out đến tất cả shard).
- **MongoDB và Vitess (MySQL) hỗ trợ resharding động**: Cho phép thay đổi shard key sau khi đã triển khai, tuy nhiên đây là thao tác tốn kém tài nguyên và cần lên kế hoạch cẩn thận.

---

### VI.4. Load Balancing cho Database

Load Balancing trong context của database là việc phân phối các truy vấn đến nhiều node database khác nhau nhằm tối ưu hóa việc sử dụng tài nguyên, tránh quá tải cho một node đơn lẻ, và tăng khả năng chịu lỗi.

**Điểm khác biệt quan trọng so với Load Balancing web server**: Database có trạng thái (stateful) và phân biệt giữa ghi và đọc. Không thể gửi ngẫu nhiên một lệnh `INSERT` đến bất kỳ node nào — lệnh ghi phải đi đến Primary. Đây là lý do Load Balancing cho database phức tạp hơn và đòi hỏi các proxy thông minh, hiểu được ngữ nghĩa của SQL.

#### VI.4.1. Kiến trúc Load Balancing trong hệ thống Database

Kiến trúc điển hình của một hệ thống database có Load Balancing:

```
┌─────────────────────────────────┐
│         Application Layer        │
│   (App Server 1, 2, 3, ...)     │
└────────────────┬────────────────┘
                 │ Kết nối duy nhất đến proxy
                 v
┌─────────────────────────────────┐
│    Database Proxy / Load         │
│    Balancer (ProxySQL / HAProxy) │
│                                  │
│  ┌─────────────────────────────┐ │
│  │  Read/Write Splitting Logic │ │
│  │  - Query bắt đầu bằng      │ │
│  │    SELECT → Read Pool       │ │
│  │  - INSERT/UPDATE/DELETE     │ │
│  │    → Write Pool             │ │
│  └─────────────────────────────┘ │
└──────┬───────────┬───────────────┘
       │           │
       v           v
┌──────────┐  ┌────────────────────┐
│  Write   │  │    Read Pool        │
│  Node    │  │ (Secondary 1, 2, 3)│
│ (Primary)│  │  Round-robin hoặc  │
│          │  │  Least Connections  │
└──────────┘  └────────────────────┘
```

**Read/Write Splitting** là kỹ thuật then chốt: các truy vấn đọc (`SELECT`) được phân phối đều ra các Replica, trong khi các truy vấn ghi (`INSERT`, `UPDATE`, `DELETE`, `BEGIN TRANSACTION`) luôn được gửi đến Primary. Điều này cho phép scale theo chiều ngang cho read workload bằng cách chỉ cần thêm Replica.

**Demo:**

*(Hình ảnh minh họa: sơ đồ kiến trúc tổng thể Application → Proxy → Write Node + Read Pool)*

#### VI.4.2. Các công cụ Load Balancing phổ biến

**ProxySQL**

ProxySQL là proxy layer mã nguồn mở phổ biến nhất cho MySQL và MariaDB. Điểm mạnh:

- **Query Rules**: Cho phép định tuyến query dựa trên regex pattern (ví dụ: query khớp `^SELECT` đi vào read pool; query khớp `^(INSERT|UPDATE|DELETE)` đi vào write pool).
- **Connection Pooling**: Duy trì một pool kết nối đến backend, giảm chi phí tạo kết nối mới cho mỗi request.
- **Query Caching**: Có thể cache kết quả các câu SELECT thường gặp.
- **Query Mirroring**: Nhân bản query đến một backend thứ hai để test.
- **Failover**: Tự động phát hiện node down và loại khỏi pool.

Cấu hình ProxySQL hoàn toàn qua SQL-like interface, không cần restart để thay đổi cấu hình.

**MySQL Router**

MySQL Router là proxy nhẹ được phát triển bởi MySQL (Oracle), tích hợp trực tiếp với InnoDB Cluster. Điểm khác biệt so với ProxySQL:

- Tự động phát hiện và cập nhật topology của InnoDB Cluster qua metadata cache — khi failover xảy ra, Router tự cập nhật mà không cần cấu hình thủ công.
- Ít tính năng hơn ProxySQL nhưng setup đơn giản hơn nhiều.
- Lý tưởng khi đã dùng InnoDB Cluster.

**HAProxy**

HAProxy là TCP/HTTP load balancer phổ biến, hỗ trợ tất cả các loại database. Ở layer TCP, HAProxy không hiểu ngữ nghĩa SQL — nó chỉ cân bằng tải ở cấp kết nối TCP, không thể tự động phân biệt read/write. Vì vậy, khi dùng HAProxy cho database, cần cấu hình hai backend pool riêng biệt và ứng dụng phải tự chọn endpoint phù hợp (hoặc dùng kết hợp với một middleware layer khác).

HAProxy phù hợp hơn trong vai trò health check và failover detection đơn giản, hoặc khi cần load balance cho các database không có proxy riêng (PostgreSQL, Redis, Cassandra...).

**mongos (MongoDB)**

Trong MongoDB Sharded Cluster, `mongos` đóng vai trò là query router tích hợp sẵn. Không cần cài đặt phần mềm proxy bên ngoài — chỉ cần triển khai nhiều instance `mongos` trước cluster, và cấu hình ứng dụng kết nối đến danh sách các `mongos` instance (MongoDB driver sẽ tự cân bằng tải giữa các mongos).

**Bảng so sánh tổng hợp:**

| Công cụ | Hỗ trợ DB | Read/Write Split | Connection Pooling | Auto Failover | Độ phức tạp |
|---------|-----------|-------------------|--------------------|---------------|-------------|
| **ProxySQL** | MySQL, MariaDB | Có (query rules) | Có | Có | Trung bình |
| **MySQL Router** | MySQL (InnoDB Cluster) | Có (tự động) | Có | Có (tự động) | Thấp |
| **HAProxy** | Tất cả (TCP level) | Không (cần manual) | Không | Có (basic) | Thấp |
| **mongos** | MongoDB only | Không áp dụng | Có | Có (tự động) | Thấp |
| **PgBouncer** | PostgreSQL | Không | Có | Không | Thấp |
| **Vitess** | MySQL | Có | Có | Có | Cao |


---
