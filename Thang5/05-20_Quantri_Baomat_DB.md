# BÁO CÁO THỰC TẬP 20/5



---

## MỤC LỤC

1. [User và Quản Lý Quyền Truy Cập](#1-user-và-quản-lý-quyền-truy-cập)
   - 1.1 [Khái niệm và mô hình phân quyền](#11-khái-niệm-và-mô-hình-phân-quyền)
   - 1.2 [Tạo và quản lý User](#12-tạo-và-quản-lý-user)
   - 1.3 [Cấp và thu hồi quyền](#13-cấp-và-thu-hồi-quyền)
   - 1.4 [Role-Based Access Control (RBAC)](#14-role-based-access-control-rbac)
   - 1.5 [Demo thực hành](#15-demo-thực-hành)

2. [Sao Lưu và Phục Hồi (Backup & Recovery)](#2-sao-lưu-và-phục-hồi-backup--recovery)
   - 2.1 [Phân loại Backup](#21-phân-loại-backup)
   - 2.2 [Công cụ Backup trong MySQL](#22-công-cụ-backup-trong-mysql)
   - 2.3 [Phục hồi dữ liệu](#23-phục-hồi-dữ-liệu)
   - 2.4 [Demo thực hành](#24-demo-thực-hành)

3. [Theo Dõi Hiệu Năng (Monitoring)](#3-theo-dõi-hiệu-năng-monitoring)
   - 3.1 [Các chỉ số hiệu năng quan trọng](#31-các-chỉ-số-hiệu-năng-quan-trọng)
   - 3.2 [Công cụ monitoring](#32-công-cụ-monitoring)
   - 3.3 [Demo thực hành](#33-demo-thực-hành)

4. [Tối Ưu Hóa Truy Vấn (Query Optimization)](#4-tối-ưu-hóa-truy-vấn-query-optimization)
   - 4.1 [Query Execution Plan](#41-query-execution-plan)
   - 4.2 [Index và chiến lược đánh index](#42-index-và-chiến-lược-đánh-index)
   - 4.3 [Các kỹ thuật tối ưu hóa](#43-các-kỹ-thuật-tối-ưu-hóa)
   - 4.4 [Demo thực hành](#44-demo-thực-hành)

5. [Quản Lý Transaction và Lock](#5-quản-lý-transaction-và-lock)
   - 5.1 [Transaction và tính chất ACID](#51-transaction-và-tính-chất-acid)
   - 5.2 [Isolation Level](#52-isolation-level)
   - 5.3 [Lock trong Database](#53-lock-trong-database)
   - 5.4 [Deadlock – Phát hiện và xử lý](#54-deadlock--phát-hiện-và-xử-lý)
   - 5.5 [Demo thực hành](#55-demo-thực-hành)

6. [Tổng Kết](#6-tổng-kết)

---

## 1. User và Quản Lý Quyền Truy Cập

### 1.1 Khái niệm và mô hình phân quyền

Quản lý quyền truy cập (Access Control) là nền tảng bảo mật của một hệ CSDL. Mục tiêu là đảm bảo **chỉ đúng người, đúng thời điểm, được làm đúng việc** trên dữ liệu.

**Các mô hình phổ biến:**

| Mô hình | Mô tả | Ví dụ |
|---------|-------|-------|
| DAC (Discretionary Access Control) | Chủ sở hữu dữ liệu cấp quyền tự do | MySQL GRANT cơ bản |
| MAC (Mandatory Access Control) | Hệ thống kiểm soát theo nhãn bảo mật | SELinux, Oracle Label Security |
| RBAC (Role-Based Access Control) | Quyền gắn vào vai trò, user được gán vai trò | MySQL Roles, PostgreSQL Roles |

**Nguyên tắc Least Privilege:** Mỗi user chỉ được cấp đúng quyền tối thiểu cần thiết để thực hiện công việc.

---

### 1.2 Tạo và quản lý User

**Trong MySQL:**

```sql
-- Tạo user mới (user chỉ kết nối từ localhost)
CREATE USER 'ten_user'@'localhost' IDENTIFIED BY 'MatKhau@2024';

-- Tạo user kết nối từ bất kỳ host nào
CREATE USER 'ten_user'@'%' IDENTIFIED BY 'MatKhau@2024';

-- Xem danh sách user
SELECT user, host, authentication_string FROM mysql.user;

-- Đổi mật khẩu
ALTER USER 'ten_user'@'localhost' IDENTIFIED BY 'MatKhauMoi@2024';

-- Khóa / mở khóa user
ALTER USER 'ten_user'@'localhost' ACCOUNT LOCK;
ALTER USER 'ten_user'@'localhost' ACCOUNT UNLOCK;

-- Xóa user
DROP USER 'ten_user'@'localhost';
```

---

### 1.3 Cấp và thu hồi quyền

**Các mức quyền trong MySQL:**

```
Global   → Database  → Table  → Column  → Routine
(*.*)    → (db.*)    → (db.t) → (cột)   → (thủ tục/hàm)
```

```sql
-- Cấp quyền SELECT trên toàn bộ database "sales"
GRANT SELECT ON sales.* TO 'nhanvien'@'localhost';

-- Cấp nhiều quyền trên một bảng cụ thể
GRANT SELECT, INSERT, UPDATE ON sales.orders TO 'nhanvien'@'localhost';

-- Cấp toàn quyền (KHÔNG nên dùng cho user thường)
GRANT ALL PRIVILEGES ON *.* TO 'admin'@'localhost' WITH GRANT OPTION;

-- Áp dụng quyền ngay lập tức
FLUSH PRIVILEGES;

-- Xem quyền của user
SHOW GRANTS FOR 'nhanvien'@'localhost';

-- Thu hồi quyền
REVOKE INSERT, UPDATE ON sales.orders FROM 'nhanvien'@'localhost';

-- Thu hồi toàn bộ quyền
REVOKE ALL PRIVILEGES ON sales.* FROM 'nhanvien'@'localhost';
```

---

### 1.4 Role-Based Access Control (RBAC)

Thay vì cấp quyền từng user, ta tạo **role** (vai trò) rồi gán role cho user. Dễ quản lý khi có nhiều user cùng chức năng.

```sql
-- Tạo role
CREATE ROLE 'role_readonly', 'role_editor', 'role_admin';

-- Cấp quyền cho role
GRANT SELECT ON sales.* TO 'role_readonly';
GRANT SELECT, INSERT, UPDATE, DELETE ON sales.* TO 'role_editor';
GRANT ALL PRIVILEGES ON sales.* TO 'role_admin';

-- Gán role cho user
GRANT 'role_readonly' TO 'user_bao_cao'@'localhost';
GRANT 'role_editor'   TO 'user_ke_toan'@'localhost';

-- Kích hoạt role mặc định khi đăng nhập
SET DEFAULT ROLE 'role_readonly' TO 'user_bao_cao'@'localhost';

-- Kích hoạt role trong phiên hiện tại
SET ROLE 'role_editor';

-- Xem role đang active
SELECT CURRENT_ROLE();
```

---

### 1.5 Demo thực hành

> **Mục tiêu:** Tạo cấu trúc user/role cho một hệ thống bán hàng nhỏ.

**Bước 1 – Tạo database và bảng mẫu**

```sql
CREATE DATABASE demo_sales;
USE demo_sales;

CREATE TABLE products (
    id   INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    price DECIMAL(10,2),
    stock INT
);

INSERT INTO products (name, price, stock) VALUES
('Laptop Dell', 15000000, 10),
('Chuột Logitech', 350000, 50),
('Bàn phím Keychron', 1800000, 30);
```
<img width="708" height="527" alt="image" src="https://github.com/user-attachments/assets/fae71ede-4102-46c6-9915-0457ba1b0f77" />

**Bước 2 – Tạo role và user**

```sql
-- Role chỉ xem
CREATE ROLE 'role_viewer';
GRANT SELECT ON demo_sales.* TO 'role_viewer';

-- Role nhập liệu
CREATE ROLE 'role_staff';
GRANT SELECT, INSERT, UPDATE ON demo_sales.* TO 'role_staff';

-- Tạo user
CREATE USER 'nv_bao_cao'@'localhost' IDENTIFIED BY 'Pass@1234';
CREATE USER 'nv_nhap_lieu'@'localhost' IDENTIFIED BY 'Pass@5678';

-- Gán role
GRANT 'role_viewer' TO 'nv_bao_cao'@'localhost';
GRANT 'role_staff'  TO 'nv_nhap_lieu'@'localhost';

SET DEFAULT ROLE ALL TO 'nv_bao_cao'@'localhost', 'nv_nhap_lieu'@'localhost';
```
<img width="1215" height="648" alt="image" src="https://github.com/user-attachments/assets/3a8326ef-8e48-4f67-87e8-f989bb4a91ae" />

**Bước 3 – Kiểm tra**

```sql
-- Đăng nhập bằng nv_bao_cao, thử lệnh INSERT (phải bị lỗi)
-- mysql -u nv_bao_cao -p
INSERT INTO demo_sales.products (name, price, stock) VALUES ('Test', 1, 1);
-- Kết quả mong đợi: ERROR 1142 (42000): INSERT command denied
```
<img width="1248" height="424" alt="image" src="https://github.com/user-attachments/assets/de267081-1c99-4c60-8c8d-ad208cedf447" />

nv báo cáo không thể thực hiện lệnh phần này và báo lỗi

---

## 2. Sao Lưu và Phục Hồi (Backup & Recovery)

### 2.1 Phân loại Backup

| Loại | Mô tả | Ưu điểm | Nhược điểm |
|------|-------|---------|-----------|
| **Full Backup** | Sao lưu toàn bộ dữ liệu | Đơn giản, phục hồi nhanh | Tốn dung lượng, thời gian dài |
| **Incremental Backup** | Chỉ sao lưu thay đổi so với lần backup trước | Nhỏ, nhanh | Phục hồi phức tạp hơn |
| **Differential Backup** | Sao lưu thay đổi kể từ lần Full Backup gần nhất | Cân bằng giữa hai loại trên | Lớn hơn Incremental |
| **Logical Backup** | Xuất ra file SQL (dump) | Dễ đọc, di chuyển được | Chậm với DB lớn |
| **Physical Backup** | Copy trực tiếp file nhị phân của DB | Nhanh, hiệu quả với DB lớn | Phụ thuộc phiên bản DBMS |

**Chiến lược 3-2-1:**
- **3** bản sao dữ liệu
- **2** loại phương tiện lưu trữ khác nhau
- **1** bản lưu ở nơi khác (offsite)

---
Đây là bản kết hợp: vừa giữ được sự ngắn gọn của lý thuyết, vừa chèn thêm các câu lệnh cốt lõi làm ví dụ minh họa để báo cáo của bạn có chiều sâu thực tế.

---

### 2.2. Các công cụ và phương pháp Backup trong MySQL

Trong quản trị cơ sở dữ liệu MySQL, có 3 công cụ và chiến lược chính thường được sử dụng tùy thuộc vào quy mô hệ thống và yêu cầu phục hồi:

#### a) `mysqldump` – Logical Backup (Sao lưu Logic)

**Khái niệm:**
Đây là công cụ mặc định phổ biến nhất. Cơ chế của nó không copy file vật lý mà đọc dữ liệu từ DB, sau đó sinh ra một file văn bản chứa các câu lệnh SQL (`CREATE TABLE`, `INSERT INTO`...).

**Đặc điểm:**
Linh hoạt, dễ nén để tiết kiệm dung lượng lưu trữ, nhưng tốc độ chậm (vì chỉ xử lý đơn luồng), phù hợp với hệ thống vừa và nhỏ.

**Các câu lệnh minh họa phổ biến:**

```bash
# Backup toàn bộ database (kèm theo tự động đặt ngày tháng vào tên file)
mysqldump -u root -p demo_sales > backup_sales_$(date +%Y%m%d).sql

# Nén backup ngay trong lúc xuất dữ liệu (tiết kiệm dung lượng)
mysqldump -u root -p demo_sales | gzip > backup_sales.sql.gz

# Tùy chọn: Chỉ backup cấu trúc (không lấy dữ liệu)
mysqldump -u root -p --no-data demo_sales > structure_only.sql

```

#### b) `mysqlpump` – Parallel Backup (Sao lưu Song song)

**Khái niệm:**
Được giới thiệu từ MySQL 5.7, công cụ này khắc phục điểm yếu tốc độ của `mysqldump` trên các hệ thống lớn bằng cách áp dụng cơ chế xử lý song song.

**Đặc điểm:**
Giảm đáng kể thời gian backup nhờ mở nhiều luồng (threads) cùng lúc. Tuy nhiên, từ bản MySQL 8.0.34, công cụ này bị đánh dấu lỗi thời (Deprecated) và dần được thay thế bởi *MySQL Shell Dump*.

**Câu lệnh minh họa:**

```bash
# Backup chia ra 4 luồng xử lý song song để tăng tốc
mysqlpump -u root -p --parallel-schemas=4 demo_sales > backup_pump.sql

```

#### c) Binary Log – Point-in-Time Recovery (Phục hồi theo thời điểm)

**Khái niệm:**
Để khắc phục lỗ hổng mất dữ liệu giữa các chu kỳ backup định kỳ, MySQL sử dụng Binary Log (Binlog). Đây là tệp nhật ký ghi lại mọi thao tác làm thay đổi dữ liệu (`INSERT`, `UPDATE`, `DELETE`) theo thời gian thực.

**Đặc điểm:**
Cho phép kết hợp bản backup gần nhất với file Binlog để khôi phục dữ liệu về đúng thời điểm trước khi xảy ra sự cố (đến từng giây). Đây là tính năng bắt buộc cho các hệ thống Production.

**Câu lệnh minh họa:**

```sql
-- Kiểm tra xem tính năng ghi nhật ký có đang được bật (ON) hay không
SHOW VARIABLES LIKE 'log_bin';

-- Xem danh sách các file nhật ký hiện có trên server
SHOW BINARY LOGS;

-- Đọc nội dung file log để truy vết các thao tác thay đổi dữ liệu
SHOW BINLOG EVENTS IN 'mysql-bin.000001' LIMIT 20;

```
### 2.4 Demo thực hành

> **Kịch bản:** Backup database → Xóa nhầm dữ liệu → Phục hồi.

**Bước 1 – Tạo backup**
trước tiên tạo một bảng data
<img width="766" height="648" alt="image" src="https://github.com/user-attachments/assets/6269864b-ba68-4329-a256-be2fa29df372" />

```bash
mysqldump -u root -p demo_sales > backup_demo_sales.sql
```

Kiểm tra file đã tạo:

```bash
ls -lh backup_demo_sales.sql
head -50 backup_demo_sales.sql
```
<img width="1481" height="776" alt="image" src="https://github.com/user-attachments/assets/cfcb1528-5372-4986-b69c-66ce5c567b8b" />

**Bước 2 – Giả lập sự cố (xóa dữ liệu)**

```sql
USE demo_sales;
DELETE FROM products WHERE id > 0;   -- Xóa toàn bộ dữ liệu
SELECT COUNT(*) FROM products;        -- Kết quả: 0
```
<img width="853" height="274" alt="image" src="https://github.com/user-attachments/assets/e74c94f2-f3f6-4cbf-a315-04bad2c2f049" />

<img width="700" height="285" alt="image" src="https://github.com/user-attachments/assets/e76a73b0-2dae-4569-87df-80ba09f7fec4" />

**Bước 3 – Phục hồi**

```bash
mysql -u root -p demo_sales < backup_demo_sales.sql
```

**Bước 4 – Xác nhận phục hồi thành công**

```sql
USE demo_sales;
SELECT * FROM products;
-- 
```
<img width="994" height="834" alt="image" src="https://github.com/user-attachments/assets/95201182-eccc-43a6-a82d-3458b8b6e024" />
Kết quả: dữ liệu đã trở lại đầy đủ

---

## 3. Theo Dõi Hiệu Năng (Monitoring)

### 3.1 Các chỉ số hiệu năng quan trọng

| Chỉ số | Ý nghĩa | Ngưỡng cảnh báo |
|--------|---------|-----------------|
| **QPS** (Queries per Second) | Số truy vấn xử lý mỗi giây | Tùy workload |
| **Buffer Pool Hit Rate** | Tỷ lệ đọc từ RAM thay vì đĩa | < 95% cần xem xét |
| **Slow Query** | Truy vấn chạy lâu hơn ngưỡng | > 1 giây (thường) |
| **Connections** | Số kết nối đang hoạt động | Gần max_connections |
| **Lock Waits** | Thời gian chờ lock | > 0 thường xuyên |
| **Replication Lag** | Độ trễ slave so với master | > 10 giây |

---

Dưới đây là bản viết lại theo phong cách báo cáo lý thuyết và tìm hiểu. Cấu trúc được làm gọn lại, giải thích rõ "Nó là gì?", "Dùng để làm gì?" và kèm theo các câu lệnh minh họa cốt lõi nhất.

---

### 3.2. Các công cụ và phương pháp Giám sát (Monitoring) trong MySQL

Việc giám sát (Monitoring) giúp Quản trị viên (DBA) nắm bắt được "sức khỏe" của Database Server theo thời gian thực, từ đó phát hiện sớm tình trạng quá tải, nghẽn cổ chai (bottleneck) hoặc các truy vấn kém hiệu quả để có phương án tối ưu kịp thời. MySQL cung cấp 3 bộ công cụ native (có sẵn) mạnh mẽ để thực hiện việc này:

#### a) Truy vấn trạng thái trực tiếp (Native Status Variables)

**Khái niệm:**
MySQL tự động duy trì hàng trăm biến số (variables) lưu trữ các thống kê hoạt động kể từ lúc service khởi động. Quản trị viên có thể gọi trực tiếp các biến này bằng lệnh `SHOW` để xem tình trạng hiện tại.

**Đặc điểm & Ứng dụng:**

* Nhanh, nhẹ, không gây thêm tải cho server.
* Thường dùng để kiểm tra ngay lập tức các "chỉ số sống còn" (Vital signs) như: Hệ thống đang có bao nhiêu kết nối (Connections)? Cache RAM (Buffer Pool) hoạt động có hiệu quả không? Hay có tiến trình nào đang bị treo (treo session) hay không.

**Các câu lệnh minh họa:**

```sql
-- 1. Kiểm tra số lượng kết nối đang mở
SHOW STATUS LIKE 'Threads_connected';

-- 2. Đánh giá hiệu quả dùng RAM của InnoDB (Hit Rate càng gần 100% càng tốt)
SHOW STATUS LIKE 'Innodb_buffer_pool_read_requests';
SHOW STATUS LIKE 'Innodb_buffer_pool_reads';

-- 3. Xem danh sách tất cả các tiến trình/truy vấn đang chạy ngay lúc này
SHOW FULL PROCESSLIST;

-- 4. Ép buộc đóng một tiến trình đang bị treo (ví dụ ID là 123)
KILL 123;

```

#### b) Performance Schema (Giám sát hiệu suất chuyên sâu)

**Khái niệm:**
Được kích hoạt mặc định từ MySQL 5.6 trở lên, `Performance Schema` là một engine đặc biệt chuyên giám sát các sự kiện hệ thống ở mức độ vi mô (low-level) bên trong MySQL.

**Đặc điểm & Ứng dụng:**

* Nó tổ chức các dữ liệu giám sát dưới dạng các "bảng" (tables) để DBA có thể dùng chính lệnh `SELECT` để truy vấn thống kê.
* Cực kỳ đắc lực khi cần phân tích chuyên sâu tìm nguyên nhân gốc rễ (Root Cause Analysis): Tìm xem bảng nào đang bị khóa (lock) nhiều nhất, tài nguyên CPU bị ngốn vào khâu nào.

**Các câu lệnh minh họa:**

```sql
-- Kiểm tra xem Performance Schema có đang được bật hay không
SHOW VARIABLES LIKE 'performance_schema';

-- Tìm top 10 câu lệnh SQL có thời gian chạy trung bình lâu nhất
SELECT DIGEST_TEXT, COUNT_STAR, AVG_TIMER_WAIT/1e12 AS avg_sec
FROM performance_schema.events_statements_summary_by_digest
ORDER BY AVG_TIMER_WAIT DESC
LIMIT 10;

```

#### c) Slow Query Log (Nhật ký truy vấn chậm)

**Khái niệm:**
Đúng như tên gọi, đây là một tính năng cấu hình cho phép MySQL tự động ghi chép lại (log) tất cả những câu lệnh SQL có thời gian thực thi vượt quá một ngưỡng quy định (Ví dụ: chạy quá 1 giây).

**Đặc điểm & Ứng dụng:**

* Là công cụ gối đầu giường để thực hiện Tối ưu hóa truy vấn (Query Optimization).
* Developer hoặc DBA sẽ gom file log này lại định kỳ, dùng tool phân tích để tìm ra những câu lệnh kém hiệu quả nhất, từ đó tiến hành đánh chỉ mục (Index) hoặc viết lại code cho tối ưu.

**Các lệnh thao tác và phân tích:**

```sql
-- Kích hoạt cấu hình ghi log trực tiếp mà không cần khởi động lại MySQL
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1; -- Cứ lệnh nào chạy > 1 giây thì ghi lại
SET GLOBAL slow_query_log_file = '/var/log/mysql/slow.log';

```


```bash
# Sử dụng công cụ mysqldumpslow để trích xuất Top 10 câu lệnh chậm nhất từ file log
mysqldumpslow -s t -t 10 /var/log/mysql/slow.log

```

### 3.3 Demo thực hành

> **Mục tiêu:** Bật slow query log, tạo query chậm, quan sát trong log.

**Bước 1 – Bật slow query log**

```sql
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 0;   -- Log mọi query (để test)
SHOW VARIABLES LIKE 'slow_query_log_file';
```
<img width="1061" height="854" alt="image" src="https://github.com/user-attachments/assets/c20bcc1c-50c4-45eb-b709-071434fcc169" />

/var/lib/mysql/dong-slow.log 

**Bước 2 – Tạo dữ liệu lớn để test**

```sql
USE demo_sales;
CREATE TABLE big_orders (
    id INT AUTO_INCREMENT PRIMARY KEY,
    customer VARCHAR(100),
    amount DECIMAL(12,2),
    order_date DATE
);

-- Chèn 100,000 dòng (dùng stored procedure)
DELIMITER $$
CREATE PROCEDURE gen_data()
BEGIN
    DECLARE i INT DEFAULT 0;
    WHILE i < 100000 DO
        INSERT INTO big_orders (customer, amount, order_date)
        VALUES (CONCAT('KH', i), RAND()*10000000,
                DATE_ADD('2023-01-01', INTERVAL FLOOR(RAND()*730) DAY));
        SET i = i + 1;
    END WHILE;
END$$
DELIMITER ;

CALL gen_data();
```
<img width="1503" height="736" alt="image" src="https://github.com/user-attachments/assets/1e3656ee-1b25-4531-9629-6ea897a7a565" />

**Bước 3 – Chạy query không có index (sẽ chậm)**

```sql
-- Query này sẽ full table scan
SELECT customer, SUM(amount)
FROM big_orders
WHERE order_date BETWEEN '2023-06-01' AND '2023-12-31'
GROUP BY customer
ORDER BY SUM(amount) DESC;
```

**Bước 4 – Xem slow query log**

```bash
tail -50 /var/log/mysql/slow.log
```

**Bước 5 – Xem processlist trong khi query chạy** (mở terminal thứ 2)

```sql
SHOW FULL PROCESSLIST;
```
<img width="1497" height="671" alt="image" src="https://github.com/user-attachments/assets/49d2d528-109a-495c-9273-5e027a8e4b4e" />


---

## 4. Tối Ưu Hóa Truy Vấn (Query Optimization)

### 4.1 Query Execution Plan

`EXPLAIN` cho thấy MySQL thực thi một query như thế nào. Đây là công cụ đầu tiên cần dùng khi tối ưu.

```sql
-- Xem execution plan
EXPLAIN SELECT customer, SUM(amount)
FROM big_orders
WHERE order_date BETWEEN '2023-06-01' AND '2023-12-31'
GROUP BY customer;

-- EXPLAIN dạng JSON (chi tiết hơn)
EXPLAIN FORMAT=JSON SELECT ...;

-- EXPLAIN ANALYZE (MySQL 8.0+): chạy thực sự và đo thời gian
EXPLAIN ANALYZE SELECT ...;
```

**Các cột quan trọng trong EXPLAIN:**

| Cột | Ý nghĩa | Cần chú ý khi |
|-----|---------|--------------|
| `type` | Loại join/scan | ALL (full scan) là tệ nhất |
| `key` | Index đang dùng | NULL = không dùng index |
| `rows` | Số hàng ước tính phải đọc | Càng nhỏ càng tốt |
| `Extra` | Thông tin thêm | "Using filesort", "Using temporary" = tốn kém |

**Thứ tự tốt → xấu của `type`:**
```
system → const → eq_ref → ref → range → index → ALL
```

---

### 4.2 Index và chiến lược đánh index

**Các loại index:**

```sql
-- B-Tree Index (mặc định) – dùng cho so sánh, range, ORDER BY
CREATE INDEX idx_order_date ON big_orders (order_date);

-- Composite Index – nhiều cột, thứ tự cột rất quan trọng
CREATE INDEX idx_date_customer ON big_orders (order_date, customer);

-- Unique Index
CREATE UNIQUE INDEX idx_unique_email ON users (email);

-- Full-Text Index – tìm kiếm văn bản
CREATE FULLTEXT INDEX idx_ft_name ON products (name);

-- Xem các index hiện có
SHOW INDEX FROM big_orders;

-- Xóa index
DROP INDEX idx_order_date ON big_orders;
```

**Quy tắc chọn index:**
- **Cột WHERE, JOIN ON, ORDER BY** thường cần index.
- **Selectivity cao** (nhiều giá trị khác nhau) → index hiệu quả hơn.
- **Không index cột thay đổi thường xuyên** (chi phí cập nhật index cao).
- **Composite index** tuân theo quy tắc **leftmost prefix**: index `(a, b, c)` hỗ trợ query trên `(a)`, `(a,b)`, `(a,b,c)` nhưng không hỗ trợ `(b)` hay `(c)`.

---

### 4.3 Các kỹ thuật tối ưu hóa

**a) Tránh SELECT \***

```sql
-- BAD
SELECT * FROM big_orders WHERE order_date = '2024-01-01';

-- GOOD: Chỉ lấy cột cần thiết → giảm I/O, tăng tốc covering index
SELECT id, customer, amount FROM big_orders WHERE order_date = '2024-01-01';
```

**b) Tránh hàm trên cột được index trong WHERE**

```sql
-- BAD: MySQL không thể dùng index trên order_date
SELECT * FROM big_orders WHERE YEAR(order_date) = 2024;

-- GOOD: Dùng range thay hàm
SELECT * FROM big_orders
WHERE order_date BETWEEN '2024-01-01' AND '2024-12-31';
```

**c) Tối ưu JOIN**

```sql
-- Đảm bảo cột JOIN có index ở cả hai bảng
-- Bảng nhỏ làm driving table (thường DBMS tự chọn)
SELECT o.id, c.name, o.amount
FROM customers c
JOIN orders o ON c.id = o.customer_id   -- customer_id phải có index
WHERE c.city = 'Hanoi';
```

**d) LIMIT để phân trang hiệu quả**

```sql
-- BAD: Với OFFSET lớn, MySQL vẫn phải đọc toàn bộ hàng trước
SELECT * FROM big_orders ORDER BY id LIMIT 100000, 10;

-- GOOD: Dùng keyset pagination
SELECT * FROM big_orders WHERE id > 100000 ORDER BY id LIMIT 10;
```

**e) Sử dụng Covering Index**

```sql
-- Index bao phủ: tất cả cột cần thiết đều nằm trong index
-- MySQL không cần đọc lại row data
CREATE INDEX idx_covering ON big_orders (order_date, customer, amount);

SELECT customer, amount
FROM big_orders
WHERE order_date = '2024-06-15';
-- Extra: "Using index" → covering index, rất nhanh
```

---

### 4.4 Demo thực hành

> **Mục tiêu:** So sánh tốc độ trước và sau khi tạo index.

**Bước 1 – Truy vấn không có index**

```sql
-- Kiểm tra không có index trên order_date
SHOW INDEX FROM big_orders;

-- Xem execution plan
EXPLAIN SELECT customer, SUM(amount)
FROM big_orders
WHERE order_date BETWEEN '2024-01-01' AND '2024-06-30'
GROUP BY customer;
-- Chú ý: type = ALL, key = NULL, rows = ~100000
```

**Bước 2 – Đo thời gian thực tế**

```sql
SET profiling = 1;

SELECT customer, SUM(amount)
FROM big_orders
WHERE order_date BETWEEN '2024-01-01' AND '2024-06-30'
GROUP BY customer;

SHOW PROFILES;
-- Ghi lại thời gian query
```
<img width="1535" height="437" alt="image" src="https://github.com/user-attachments/assets/1001614c-f5a8-4891-84f9-29870df34ba2" />

**Bước 3 – Tạo index và so sánh**

```sql
CREATE INDEX idx_order_date ON big_orders (order_date);

-- Chạy lại cùng query
SELECT customer, SUM(amount)
FROM big_orders
WHERE order_date BETWEEN '2024-01-01' AND '2024-06-30'
GROUP BY customer;

SHOW PROFILES;
-- So sánh thời gian trước và sau
```
<img width="1274" height="547" alt="image" src="https://github.com/user-attachments/assets/099e28da-b2da-4d7e-b461-02520258aef8" />

**Bước 4 – EXPLAIN sau khi có index**

```sql
EXPLAIN SELECT customer, SUM(amount)
FROM big_orders
WHERE order_date BETWEEN '2024-01-01' AND '2024-06-30'
GROUP BY customer;
-- Chú ý: type = range, key = idx_order_date, rows giảm đáng kể
```
thời gian đã giảm đáng kể , nhưng đó là khi đã đổi lại range truy vấn 
Có thể thấy sau khi tạo Index, thời gian chạy thực tế (Query ID 3: 0.30s) lại LÂU HƠN lúc chưa có Index (Query ID 1: 0.28s). Và ở lệnh EXPLAIN, cột type vẫn là ALL (quét toàn bộ bảng) chứ không thèm dùng Index (key vẫn là NULL).

Nguyên nhân: Sự "thông minh" của MySQL Optimizer
Trong MySQL có một bộ phận gọi là Query Optimizer (Bộ tối ưu hóa truy vấn). Khi bạn đưa một câu lệnh cho nó, nó sẽ phân tích xem dùng Index nhanh hơn hay quét toàn bộ bảng (Full Table Scan) nhanh hơn.

Hãy nhìn vào các thông số sau trong bài lab của bạn:

Bảng của bạn có khoảng 200.000 dòng (cột rows hiện 199978).

Khoảng thời gian bạn tìm kiếm: BETWEEN '2024-01-01' AND '2024-06-30' (nửa năm).

Cột filtered = 50.00%: Bộ tối ưu hóa dự đoán rằng kết quả thỏa mãn điều kiện thời gian này chiếm tới 50% tổng số dữ liệu của cả bảng (khoảng 100.000 dòng).

Quy tắc ngầm của MySQL: Khi một truy vấn sử dụng Index mà trả về số lượng dữ liệu quá lớn (thường > 20% - 30% tổng số dòng), MySQL sẽ quyết định bỏ qua Index và chọn Full Table Scan (type = ALL).

Tại sao? Bởi vì cấu trúc của Index lưu trữ theo dạng cây (B-Tree). Nếu dùng Index để đọc số lượng lớn, MySQL sẽ phải:

Đọc cây Index để tìm địa chỉ dòng.

Nhảy về ổ cứng đọc dữ liệu dòng đó.

Lặp lại thao tác "nhảy qua nhảy lại" đó 100.000 lần (Random I/O). Việc này tốn nhiều chi phí ổ đĩa hơn rất nhiều so với việc cứ thế quét một lèo từ đầu đến cuối bảng trên ổ cứng (Sequential I/O).


2. Chạy câu lệnh với phạm vi tìm kiếm NHỎ HƠN (ví dụ tìm trong đúng 1 ngày hoặc 1 tuần ta sẽ thấy rõ sự chênh lệch của nó ):

SQL
SELECT customer, SUM(amount)
FROM big_orders
WHERE order_date BETWEEN '2024-01-01' AND '2024-01-07' 
GROUP BY customer;


<img width="1288" height="709" alt="image" src="https://github.com/user-attachments/assets/9212fb20-104e-4dc6-bbd7-06331867657b" />


---

## 5. Quản Lý Transaction và Lock

### 5.1 Transaction và tính chất ACID

**Transaction** là một đơn vị công việc gồm nhiều thao tác, hoặc tất cả thành công hoặc tất cả thất bại.

**Tính chất ACID:**

| Tính chất | Ý nghĩa | Ví dụ |
|-----------|---------|-------|
| **A**tomicity (Nguyên tử) | Tất cả hoặc không có gì | Chuyển tiền: trừ A và cộng B phải cùng xảy ra |
| **C**onsistency (Nhất quán) | Dữ liệu luôn ở trạng thái hợp lệ | Số dư không bao giờ âm |
| **I**solation (Cô lập) | Các transaction không ảnh hưởng nhau | T1 không thấy thay đổi chưa commit của T2 |
| **D**urability (Bền vững) | Kết quả đã commit không bị mất | Dù server crash, dữ liệu đã commit vẫn còn |

**Cú pháp cơ bản:**

```sql
-- Bắt đầu transaction
START TRANSACTION;  -- hoặc BEGIN;

-- Thực hiện các thao tác
UPDATE accounts SET balance = balance - 500000 WHERE id = 1;
UPDATE accounts SET balance = balance + 500000 WHERE id = 2;

-- Kiểm tra điều kiện
SELECT balance FROM accounts WHERE id = 1;

-- Nếu OK: xác nhận
COMMIT;

-- Nếu lỗi: hủy bỏ
ROLLBACK;

-- SAVEPOINT – phục hồi về một điểm trung gian
SAVEPOINT sp1;
-- ... thực hiện thêm ...
ROLLBACK TO SAVEPOINT sp1;  -- về sp1, không hủy toàn bộ
RELEASE SAVEPOINT sp1;
```

---

### 5.2 Isolation Level

Isolation level quyết định mức độ cô lập giữa các transaction đang chạy song song.

**Các vấn đề có thể xảy ra:**

| Vấn đề | Mô tả |
|--------|-------|
| **Dirty Read** | Đọc dữ liệu chưa được commit của transaction khác |
| **Non-Repeatable Read** | Đọc cùng row hai lần trong một transaction nhưng kết quả khác nhau |
| **Phantom Read** | Đọc cùng điều kiện WHERE nhưng số lượng row khác nhau |

**Bảng so sánh Isolation Level:**

| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|----------------|-----------|--------------------|-----------  |
| READ UNCOMMITTED | Có thể xảy ra |  Có thể |  Có thể |
| READ COMMITTED | Ngăn chặn | Có thể | Có thể |
| REPEATABLE READ | Ngăn chặn | Ngăn chặn | Có thể |
| SERIALIZABLE | Ngăn chặn |  Ngăn chặn | Ngăn chặn |

> MySQL InnoDB mặc định dùng **REPEATABLE READ** và có cơ chế MVCC ngăn Phantom Read.

```sql
-- Xem isolation level hiện tại
SHOW VARIABLES LIKE 'transaction_isolation';

-- Thay đổi isolation level cho phiên hiện tại
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- Thay đổi global
SET GLOBAL TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```

---

### 5.3 Lock trong Database

**Phân loại lock:**

**a) Shared Lock (S-Lock) và Exclusive Lock (X-Lock)**

| | S-Lock (đọc) | X-Lock (ghi) |
|-|-------------|-------------|
| **S-Lock** | Tương thích (nhiều transaction có thể đọc cùng lúc) | Không tương thích |
| **X-Lock** | Không tương thích | Không tương thích |

```sql
-- Lấy Shared Lock (đọc)
SELECT * FROM accounts WHERE id = 1 LOCK IN SHARE MODE;
-- MySQL 8.0+:
SELECT * FROM accounts WHERE id = 1 FOR SHARE;

-- Lấy Exclusive Lock (chuẩn bị ghi)
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
```

**b) Row-Level Lock vs Table-Level Lock**

```sql
-- Row-Level Lock (InnoDB) – chỉ khóa hàng cụ thể
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;

-- Table-Level Lock – khóa cả bảng (MyISAM hoặc thủ công)
LOCK TABLES accounts WRITE;
-- ... thao tác ...
UNLOCK TABLES;
```

**c) Gap Lock và Next-Key Lock (InnoDB)**

InnoDB dùng **Next-Key Lock** = Row Lock + Gap Lock để ngăn Phantom Read trong REPEATABLE READ.

```sql
-- Câu này lock cả range (khoảng trống giữa các id thỏa điều kiện)
SELECT * FROM orders WHERE order_date = '2024-06-01' FOR UPDATE;
```

**d) Xem trạng thái lock hiện tại**

```sql
-- Xem lock đang giữ và chờ (MySQL 8.0+)
SELECT * FROM performance_schema.data_locks\G

-- Xem lock waits
SELECT * FROM performance_schema.data_lock_waits\G

-- Thông tin InnoDB tổng thể (bao gồm lock)
SHOW ENGINE INNODB STATUS\G
```

---

### 5.4 Deadlock – Phát hiện và xử lý

**Deadlock** xảy ra khi hai transaction chờ nhau giải phóng lock, tạo thành vòng tròn.

```
T1 giữ lock A, chờ lock B
T2 giữ lock B, chờ lock A
→ Deadlock!
```

**Cách InnoDB xử lý:** Tự động phát hiện deadlock và rollback transaction có "trọng lượng" nhỏ hơn (ít thay đổi hơn).

```sql
-- Xem deadlock gần nhất
SHOW ENGINE INNODB STATUS\G
-- Tìm phần "LATEST DETECTED DEADLOCK"
```

**Cách phòng tránh Deadlock:**

```sql
-- 1. Luôn lock theo thứ tự nhất quán
-- T1 và T2 đều lock id=1 trước, id=2 sau
-- → Không bao giờ deadlock

-- 2. Dùng SELECT ... FOR UPDATE sớm
START TRANSACTION;
SELECT * FROM accounts WHERE id IN (1, 2) ORDER BY id FOR UPDATE;
-- Lock cả 2 row ngay từ đầu, không có cơ hội deadlock

-- 3. Giảm thời gian transaction
-- Không để user input trong lúc transaction đang mở

-- 4. Cài đặt innodb_lock_wait_timeout
SET innodb_lock_wait_timeout = 5;  -- Timeout sau 5 giây chờ lock
```

---

### 5.5 Demo thực hành

 Mô phỏng chuyển tiền an toàn với transaction.  


---

 Chuyển tiền với Transaction

**Bước 1 – Chuẩn bị dữ liệu**

```sql
CREATE DATABASE demo_bank;
USE demo_bank;

CREATE TABLE accounts (
    id      INT PRIMARY KEY,
    name    VARCHAR(50),
    balance DECIMAL(15,2)
);

INSERT INTO accounts VALUES
(1, 'Nguyen Van A', 5000000),
(2, 'Tran Thi B',   2000000);

SELECT * FROM accounts;
```
<img width="604" height="236" alt="image" src="https://github.com/user-attachments/assets/499d4377-51b0-4f32-ac50-117b21473a40" />

**Bước 2 – Stored Procedure chuyển tiền an toàn**

```sql
DELIMITER $$
CREATE PROCEDURE transfer(
    IN from_id INT,
    IN to_id   INT,
    IN amount  DECIMAL(15,2)
)
BEGIN
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        SELECT 'LỖI: Giao dịch đã bị hủy' AS ket_qua;
    END;

    START TRANSACTION;

    -- Kiểm tra số dư
    IF (SELECT balance FROM accounts WHERE id = from_id FOR UPDATE) < amount THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Số dư không đủ';
    END IF;

    UPDATE accounts SET balance = balance - amount WHERE id = from_id;
    UPDATE accounts SET balance = balance + amount WHERE id = to_id;

    COMMIT;
    SELECT 'Chuyển tiền thành công' AS ket_qua;
END$$
DELIMITER ;
```

**Bước 3 – Thực hiện chuyển tiền**

```sql
-- Chuyển 1,000,000 từ A sang B
CALL transfer(1, 2, 1000000);

-- Kiểm tra kết quả
SELECT * FROM accounts;
-- A: 4,000,000 | B: 3,000,000
```
<img width="574" height="523" alt="image" src="https://github.com/user-attachments/assets/5678a6d5-9ec3-4c53-88d1-de3ce847f80b" />
```
-- Thử chuyển quá số dư
CALL transfer(1, 2, 99999999);
-- Kết quả: LỖI: Giao dịch đã bị hủy
```
<img width="588" height="305" alt="image" src="https://github.com/user-attachments/assets/5fea0769-3809-4c7f-8622-fa741608336b" />



---

## 6. Tổng Kết

| Chủ đề | Công cụ/Lệnh chính | Lưu ý quan trọng |
|--------|-------------------|-----------------|
| **User & Quyền** | `CREATE USER`, `GRANT`, `REVOKE`, `CREATE ROLE` | Luôn áp dụng Least Privilege |
| **Backup** | `mysqldump`, binary log | Test phục hồi định kỳ |
| **Monitoring** | `SHOW STATUS`, Performance Schema, Slow Query Log | Đặt alert cho các chỉ số ngưỡng |
| **Query Optimization** | `EXPLAIN`, `CREATE INDEX`, `SET profiling` | Index đúng chỗ, tránh full scan |
| **Transaction & Lock** | `START TRANSACTION`, `COMMIT`, `ROLLBACK`, `FOR UPDATE` | Lock thứ tự nhất quán, giảm thời gian transaction |

**Bộ tắt ngắn các best practices:**
- **Bảo mật:** Không dùng root cho ứng dụng; dùng role để phân quyền nhóm.
- **Backup:** Backup hàng ngày + binary log; test restore hàng tháng.
- **Monitoring:** Luôn bật slow query log; kiểm tra processlist thường xuyên.
- **Tối ưu:** EXPLAIN trước khi tạo index; tránh hàm trên cột WHERE.
- **Transaction:** Giữ transaction ngắn; lock theo thứ tự nhất quán để tránh deadlock.

---

*Báo cáo được biên soạn dựa trên MySQL 8.x. Các lệnh tương tự có thể áp dụng cho MariaDB và PostgreSQL với cú pháp điều chỉnh nhỏ.*
