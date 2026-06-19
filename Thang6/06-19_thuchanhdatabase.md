# THỰC HÀNH DATABASE SERVER: MySQL/MariaDB
**Môi trường:** Ubuntu 22.04 LTS | **Engine:** MariaDB 10.11

---

## MỤC LỤC

1. [Cài đặt & Cấu hình](#1-cài-đặt--cấu-hình)
2. [Backup/Restore & Import/Export](#2-backuprestore--importexport)
3. [Tuning/Tối ưu hiệu năng](#3-tuningtối-ưu-hiệu-năng)
4. [HA – Master-Slave Replication](#4-ha--master-slave-replication)

---

## 1. Cài đặt & Cấu hình

### 1.1 Cài đặt MariaDB

```bash
# Cập nhật hệ thống
sudo apt update && sudo apt upgrade -y

# Cài MariaDB Server
sudo apt install -y mariadb-server mariadb-client

# Kiểm tra trạng thái
sudo systemctl status mariadb

# Bật autostart
sudo systemctl enable mariadb
sudo systemctl start mariadb
```
<img width="1503" height="756" alt="image" src="https://github.com/user-attachments/assets/2cafdd13-8be3-40fb-91d1-a5db77d5669b" />

### 1.2 Bảo mật ban đầu

```bash
sudo mysql_secure_installation
```

Trả lời các câu hỏi:
| Câu hỏi | Khuyến nghị |
|---------|-------------|
| Set root password? | Yes |
| Remove anonymous users? | Yes |
| Disallow root login remotely? | Yes |
| Remove test database? | Yes |
| Reload privilege tables? | Yes |

<img width="967" height="834" alt="image" src="https://github.com/user-attachments/assets/f16e5723-72aa-4400-8bfc-30ef4d795bf6" />

### 1.3 Đăng nhập và tạo user

```sql
-- Đăng nhập với root
sudo mysql -u root -p

-- Tạo database
CREATE DATABASE testdb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- Tạo user ứng dụng
CREATE USER 'appuser'@'localhost' IDENTIFIED BY 'StrongPass@123';
CREATE USER 'appuser'@'%' IDENTIFIED BY 'StrongPass@123';  -- cho phép remote

-- Cấp quyền
GRANT ALL PRIVILEGES ON testdb.* TO 'appuser'@'localhost';
GRANT ALL PRIVILEGES ON testdb.* TO 'appuser'@'%';
FLUSH PRIVILEGES;

-- Kiểm tra
SHOW GRANTS FOR 'appuser'@'localhost';
```

<img width="1449" height="727" alt="image" src="https://github.com/user-attachments/assets/98b7351b-269d-4df3-81de-fb70a109b045" />

### 1.4 Cấu hình file `/etc/mysql/mariadb.conf.d/50-server.cnf`

```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

Các thông số cơ bản cần điều chỉnh:

```ini
[mysqld]
# Bind address (0.0.0.0 để cho phép kết nối từ xa)
bind-address = 0.0.0.0

# Character set
character-set-server = utf8mb4
collation-server     = utf8mb4_unicode_ci

# Thư mục dữ liệu

datadir = /var/lib/mysql
# Log lỗi
log_error = /var/log/mysql/error.log

# Slow query log
slow_query_log       = 1
slow_query_log_file  = /var/log/mysql/slow.log
long_query_time      = 2   # giây

# General log (chỉ bật khi debug)
# general_log      = 1
# general_log_file = /var/log/mysql/general.log
```

```bash
# Khởi động lại sau khi chỉnh cấu hình
sudo systemctl restart mariadb
```

### 1.5 Cho phép firewall (nếu dùng UFW)

```bash
sudo ufw allow 3306/tcp
sudo ufw reload
```

### 1.6 Kiểm tra kết nối từ xa

```bash
# Từ máy khác
mysql -u appuser -p -h <IP_SERVER> testdb
```

---

## 2. Backup/Restore & Import/Export

### 2.1 Backup với `mysqldump`


```bash
# Backup một database
mysqldump -u root -p testdb > /backup/testdb_$(date +%F).sql
```
<img width="1049" height="635" alt="image" src="https://github.com/user-attachments/assets/57d49f32-19c3-4c0a-b1ca-6b84d07605fd" />

```
# Backup nhiều database
mysqldump -u root -p --databases testdb db2 db3 > /backup/multi_$(date +%F).sql

# Backup toàn bộ server
mysqldump -u root -p --all-databases > /backup/all_$(date +%F).sql

# Backup kèm stored procedures, triggers, events
mysqldump -u root -p \
  --routines \
  --triggers \
  --events \
  testdb > /backup/testdb_full_$(date +%F).sql

# Backup chỉ cấu trúc (không có data)
mysqldump -u root -p --no-data testdb > /backup/testdb_schema.sql

# Backup chỉ data (không có cấu trúc)
mysqldump -u root -p --no-create-info testdb > /backup/testdb_data.sql
```

Tạo dữ liệu cho db
<img width="929" height="750" alt="image" src="https://github.com/user-attachments/assets/bb9acc29-87ce-4925-a864-068dfa17eea4" />

<img width="1090" height="180" alt="image" src="https://github.com/user-attachments/assets/296906b3-a27c-4cf3-8388-da96b6447863" />

các file backup 

<img width="1054" height="261" alt="image" src="https://github.com/user-attachments/assets/5772db68-ad24-4563-ae02-0c80086b2bf5" />

<img width="885" height="200" alt="image" src="https://github.com/user-attachments/assets/a22510b3-3cc6-4660-9bc7-98bc2ff264aa" />

### 2.2 Restore từ file SQL



```bash
# Restore vào database đã tồn tại
mysql -u root -p testdb < /backup/testdb_2024-01-01.sql

# Restore toàn bộ (all-databases)
mysql -u root -p < /backup/all_2024-01-01.sql

# Restore với verbose output
mysql -u root -p testdb < /backup/testdb.sql --verbose
```
Giả sử thử drop db test và restore lại 
```
# Xóa testdb để giả lập mất data
mysql -u root -p -e "DROP DATABASE testdb;"

# Kiểm tra đã mất
mysql -u root -p -e "SHOW DATABASES;"

# Tạo lại và restore
mysql -u root -p -e "CREATE DATABASE testdb;"
mysql -u root -p testdb < /backup/testdb_2026-06-19.sql

# Verify
mysql -u root -p -e "SELECT * FROM testdb.students;"

```
<img width="1004" height="727" alt="image" src="https://github.com/user-attachments/assets/588fe3f5-8c21-4584-9710-4ea85bcc2ebe" />



### 2.3 Import/Export dữ liệu CSV



-- Đăng nhập vào mariadb trước
mysql -u root -p testdb
```
-- Export bảng ra CSV
SELECT * FROM students
INTO OUTFILE '/tmp/students.csv'
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n';

```

<img width="935" height="639" alt="image" src="https://github.com/user-attachments/assets/b9a3d3b6-739a-4a46-9311-68c8f486598c" />


```
-- Xóa data trong bảng để test import
DELETE FROM students;

-- Import lại từ file
LOAD DATA INFILE '/tmp/students.csv'
INTO TABLE students
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n';

-- Kiểm tra
SELECT * FROM students;
```

<img width="741" height="284" alt="image" src="https://github.com/user-attachments/assets/0c6b487e-61ee-4472-b557-ce2b48f63d79" />

<img width="1040" height="679" alt="image" src="https://github.com/user-attachments/assets/9838712c-c67e-4a41-a371-d614f58d015d" />


### 2.4 Tự động backup với Cron

```bash
# Tạo script backup
sudo nano /usr/local/bin/mysql_backup.sh
```

```bash
#!/bin/bash
BACKUP_DIR="/backup/mysql"
DATE=$(date +%F_%H%M)
USER="root"
PASS="your_root_password"
KEEP_DAYS=7

mkdir -p "$BACKUP_DIR"

# Backup
mysqldump -u "$USER" -p"$PASS" --all-databases \
  --single-transaction \
  --quick \
  --lock-tables=false \
  | gzip > "$BACKUP_DIR/all_$DATE.sql.gz"

# Xóa backup cũ hơn KEEP_DAYS ngày
find "$BACKUP_DIR" -name "*.sql.gz" -mtime +$KEEP_DAYS -delete

echo "Backup completed: $BACKUP_DIR/all_$DATE.sql.gz"
```
<img width="941" height="736" alt="image" src="https://github.com/user-attachments/assets/7a438e3a-1cbc-4fb7-b553-a230728b09e0" />

```bash
sudo chmod +x /usr/local/bin/mysql_backup.sh

```
```
# Thêm vào crontab (chạy lúc 2h sáng mỗi ngày)
sudo crontab -e
# Thêm dòng:
0 2 * * * /usr/local/bin/mysql_backup.sh >> /var/log/mysql_backup.log 2>&1
```
<img width="1224" height="679" alt="image" src="https://github.com/user-attachments/assets/d4dd8ec6-547d-4a00-9a60-d4a07e0d77c8" />

Đã nhận file auto backup

## 3. Tuning/Tối ưu hiệu năng

### 3.1 Xem trạng thái hiện tại

```sql
-- Các biến hệ thống đang chạy
SHOW GLOBAL VARIABLES LIKE 'innodb%';
SHOW GLOBAL VARIABLES LIKE 'key_buffer%';
SHOW GLOBAL VARIABLES LIKE 'max_connections';

-- Trạng thái thực tế
SHOW GLOBAL STATUS LIKE 'Threads_connected';
SHOW GLOBAL STATUS LIKE 'Slow_queries';
SHOW GLOBAL STATUS LIKE 'Innodb_buffer_pool%';

-- Xem các query đang chạy
SHOW FULL PROCESSLIST;

-- Xem bảng nào đang bị lock
SHOW OPEN TABLES WHERE In_use > 0;
```
<img width="979" height="756" alt="image" src="https://github.com/user-attachments/assets/c6d8c5c1-fbbd-4d18-b340-e3b62d2d4f44" />

<img width="1002" height="474" alt="image" src="https://github.com/user-attachments/assets/c955100e-61f6-4bd7-a741-e02f0f104fbf" />

<img width="1146" height="711" alt="image" src="https://github.com/user-attachments/assets/51b49131-5239-4e5c-9a13-87abdc7a75e7" />

<img width="863" height="434" alt="image" src="https://github.com/user-attachments/assets/e8977669-3078-4876-b10a-175c6bc9ffb5" />

<img width="1112" height="186" alt="image" src="https://github.com/user-attachments/assets/3aa2c6a1-ac65-4695-9371-15936506e2e0" />


### 3.2 Cấu hình tối ưu trong `50-server.cnf`

```ini
[mysqld]
# =====================
# InnoDB Buffer Pool
# =====================
# Đặt khoảng 60-70% RAM cho server chuyên DB
# Ví dụ server 4GB RAM → 2-3GB
innodb_buffer_pool_size     = 2G
innodb_buffer_pool_instances = 2      # 1 instance per 1GB

# InnoDB Log
innodb_log_file_size        = 256M
innodb_log_buffer_size      = 16M
innodb_flush_log_at_trx_commit = 2   # 1=an toàn nhất, 2=nhanh hơn, 0=nhanh nhất

# =====================
# Connections
# =====================
max_connections      = 200
max_connect_errors   = 1000000
wait_timeout         = 600
interactive_timeout  = 600

# Thread cache
thread_cache_size    = 50

# =====================
# Query Cache (MariaDB)
# =====================
query_cache_type     = 1
query_cache_size     = 64M
query_cache_limit    = 2M

# =====================
# Sort & Join Buffers (per-thread)
# =====================
sort_buffer_size     = 4M
join_buffer_size     = 4M
read_buffer_size     = 2M
read_rnd_buffer_size = 4M

# =====================
# Temp Tables
# =====================
tmp_table_size       = 64M
max_heap_table_size  = 64M

# =====================
# MyISAM Key Buffer
# =====================
key_buffer_size      = 32M
```
Phần này mang tính tham khảo , máy ảo của em không set dc tới tầm này :DDDD

### 3.3 Phân tích và tối ưu Query

```sql
-- EXPLAIN để phân tích query
EXPLAIN SELECT * FROM orders WHERE customer_id = 5;
EXPLAIN FORMAT=JSON SELECT * FROM orders WHERE customer_id = 5;

-- Tạo index
CREATE INDEX idx_customer_id ON orders(customer_id);
CREATE INDEX idx_created_date ON orders(created_at);

-- Index composite
CREATE INDEX idx_customer_date ON orders(customer_id, created_at);

-- Xem các index của bảng
SHOW INDEX FROM orders;

-- Kiểm tra bảng và tối ưu
ANALYZE TABLE orders;
OPTIMIZE TABLE orders;

-- Xem slow queries trong session
SET long_query_time = 1;
SET profiling = 1;
SELECT * FROM orders WHERE status = 'pending';
SHOW PROFILES;
SHOW PROFILE FOR QUERY 1;
```

### 3.4 Dùng `mysqltuner` để tự động gợi ý

```bash
# Cài đặt
sudo apt install -y mysqltuner

# Chạy (sau khi DB đã chạy ít nhất vài giờ)
sudo mysqltuner --user root --pass your_password
```

Đọc phần **Recommendations** ở cuối output để điều chỉnh cấu hình.

### 3.5 Monitoring nhanh

```bash
# Real-time monitoring
sudo mysqladmin -u root -p extended-status -i 1 | grep -E "Threads|Questions|Slow"

# Xem top queries (cần Performance Schema)
mysql -u root -p -e "
SELECT digest_text, count_star, avg_timer_wait/1e12 avg_sec
FROM performance_schema.events_statements_summary_by_digest
ORDER BY avg_timer_wait DESC
LIMIT 10;
"
```
<img width="1527" height="828" alt="image" src="https://github.com/user-attachments/assets/f164b43c-e72c-468e-bc41-6df201523431" />

<img width="609" height="474" alt="image" src="https://github.com/user-attachments/assets/42b253b9-94d5-4a19-8ddc-64f2a7ec54ce" />

---

## 4. HA – Master-Slave Replication

### Mô hình lab

```
[Master]  192.168.197.146  (Ubuntu VM 1)
    |
    |  Binary Log Replication
    ↓
[Slave]   192.168.197.147  (Ubuntu VM 2)
```

### 4.1 Chuẩn bị cả hai máy

```bash
# Thực hiện trên CẢ HAI máy
sudo apt update
sudo apt install -y mariadb-server
sudo systemctl enable --now mariadb
```

### 4.2 Cấu hình Master (192.168.197.146)

```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

```ini
[mysqld]
server-id               = 1
bind-address            = 0.0.0.0
log_bin                 = /var/log/mysql/mysql-bin.log
binlog_format           = ROW
expire_logs_days        = 7
max_binlog_size         = 100M
binlog_ignore_db        = information_schema
binlog_ignore_db        = performance_schema
binlog_ignore_db        = mysql
```

```bash
sudo systemctl restart mariadbsssws
```

Tạo user replication trên Master:

```sql
sudo mysql -u root -p

-- Tạo user cho slave kết nối
CREATE USER 'replicator'@'192.168.1.20' IDENTIFIED BY 'Repl@Pass123';
GRANT REPLICATION SLAVE ON *.* TO 'replicator'@'192.168.197.146';
FLUSH PRIVILEGES;

-- Lấy thông tin binary log hiện tại
-- GHI LẠI File và Position
FLUSH TABLES WITH READ LOCK;
SHOW MASTER STATUS;
```

Kết quả

<img width="1240" height="738" alt="image" src="https://github.com/user-attachments/assets/2b2a4f82-a1a4-40f1-b10c-6e4a9ae32a99" />


Nếu có data sẵn, dump và copy sang Slave:

```bash
# Terminal khác (giữ lock ở trên)
mysqldump -u root -p --all-databases --master-data > /tmp/master_dump.sql

# Sau khi dump xong, mở khóa
# (quay lại terminal đang lock)
UNLOCK TABLES;

# Copy dump sang Slave
scp /tmp/master_dump.sql user@192.168.1.20:/tmp/
```
<img width="1206" height="486" alt="image" src="https://github.com/user-attachments/assets/c48f649b-a460-4f74-a73c-cebfd7fc39ac" />

### 4.3 Cấu hình Slave (192.168.1.20)

```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

```ini
[mysqld]
server-id               = 2
bind-address            = 0.0.0.0
read_only               = 1
relay_log               = /var/log/mysql/relay-bin
relay_log_index         = /var/log/mysql/relay-bin.index
log_slave_updates       = 1
```
<img width="1085" height="400" alt="image" src="https://github.com/user-attachments/assets/1da3aa55-aae7-4d1d-a7f5-adbef376b2ef" />

```bash
sudo systemctl restart mariadb
```

Import data từ Master (nếu có):

```bash
mysql -u root -p < /tmp/master_dump.sql
```

Cấu hình kết nối tới Master:

```sql
sudo mysql -u root -p

STOP SLAVE;

CHANGE MASTER TO
  MASTER_HOST     = '192.168.1.10',
  MASTER_USER     = 'replicator',
  MASTER_PASSWORD = 'Repl@Pass123',
  MASTER_LOG_FILE = 'mysql-bin.000001',   -- từ SHOW MASTER STATUS
  MASTER_LOG_POS  = 745;                   -- từ SHOW MASTER STATUS

START SLAVE;

-- Kiểm tra trạng thái
SHOW SLAVE STATUS\G
```

### 4.4 Kiểm tra Replication hoạt động

Tìm hai dòng quan trọng trong `SHOW SLAVE STATUS\G`:

```
Slave_IO_Running: Yes      ← phải là Yes
Slave_SQL_Running: Yes     ← phải là Yes
Seconds_Behind_Master: 0   ← độ trễ (0 = realtime)
```
<img width="968" height="783" alt="image" src="https://github.com/user-attachments/assets/d7b15f53-8d75-4336-8024-405a8ddcf71b" />

Test thực tế:

```sql
-- Trên MASTER
CREATE DATABASE reptest;
USE reptest;
CREATE TABLE hello (id INT AUTO_INCREMENT PRIMARY KEY, msg VARCHAR(100));
INSERT INTO hello (msg) VALUES ('Hello from Master!');

-- Trên SLAVE (kiểm tra ngay)
USE reptest;
SELECT * FROM hello;
-- Phải thấy dòng dữ liệu từ Master
```
<img width="1227" height="866" alt="image" src="https://github.com/user-attachments/assets/7d31bb0c-a96a-4916-8a9a-95ebd4a6bc6e" />

<img width="1252" height="474" alt="image" src="https://github.com/user-attachments/assets/4531ac94-83e7-4bf2-a06e-5ddf8bf1463c" />



---

## TỔNG KẾT CÁC LỆNH QUAN TRỌNG

| Tác vụ | Lệnh |
|--------|------|
| Xem databases | `SHOW DATABASES;` |
| Xem tables | `SHOW TABLES;` |
| Xem cấu trúc bảng | `DESCRIBE tablename;` |
| Xem user | `SELECT user, host FROM mysql.user;` |
| Xem process | `SHOW FULL PROCESSLIST;` |
| Kill query | `KILL QUERY <id>;` |
| Xem biến | `SHOW VARIABLES LIKE 'pattern';` |
| Xem status | `SHOW STATUS LIKE 'pattern';` |
| Flush privileges | `FLUSH PRIVILEGES;` |
| Xem binlog | `SHOW BINARY LOGS;` |
| Xem master status | `SHOW MASTER STATUS;` |
| Xem slave status | `SHOW SLAVE STATUS\G` |

---
