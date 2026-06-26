# Báo Cáo Thực Tập: Zabbix High Availability (Native HA Cluster)

---

## Mục Lục

1. [Tổng Quan về Zabbix HA](#1-tổng-quan-về-zabbix-ha)
2. [Mô Hình Lab và Hiện Trạng](#2-mô-hình-lab-và-hiện-trạng)
3. [Chuẩn Bị trên Node 1 (Máy đã có Zabbix Server)](#3-chuẩn-bị-trên-node-1-máy-đã-có-zabbix-server)
4. [Cài Đặt Zabbix Server trên Node 2 (Máy Agent)](#4-cài-đặt-zabbix-server-trên-node-2-máy-agent)
5. [Cấu Hình HA trên Node 1 — Kích Hoạt Chế Độ Active](#5-cấu-hình-ha-trên-node-1--kích-hoạt-chế-độ-active)
6. [Cấu Hình HA trên Node 2 — Kích Hoạt Chế Độ Standby](#6-cấu-hình-ha-trên-node-2--kích-hoạt-chế-độ-standby)
7. [Cấu Hình Zabbix Frontend cho HA](#7-cấu-hình-zabbix-frontend-cho-ha)
8. [Cập Nhật Cấu Hình Zabbix Agent trên Node 2](#8-cập-nhật-cấu-hình-zabbix-agent-trên-node-2)
9. [Kiểm Tra Trạng Thái Cluster](#9-kiểm-tra-trạng-thái-cluster)
10. [Kiểm Tra Failover](#10-kiểm-tra-failover)
11. [Các Lệnh Quản Trị HA Cluster](#11-các-lệnh-quản-trị-ha-cluster)

---

## 1. Tổng Quan về Zabbix HA

**High Availability (HA)** giúp hệ thống Zabbix tiếp tục hoạt động không gián đoạn khi một Zabbix Server bị lỗi. Nếu không có HA, khi máy chủ Zabbix gặp sự cố (crash, mất điện, hết disk...) thì toàn bộ quá trình thu thập dữ liệu, phát hiện sự cố và gửi cảnh báo đều dừng lại.

Từ **Zabbix 6.0 LTS**, tính năng **native HA** được tích hợp trực tiếp mà không cần phần mềm bên thứ ba (Pacemaker, Corosync...).

### Cơ Chế Hoạt Động

Native HA hoạt động theo mô hình **Active/Passive**:

- **Active node:** Node đang xử lý thực sự — thu thập dữ liệu, đánh giá trigger, gửi cảnh báo.
- **Standby node:** Node dự phòng, theo dõi Active node thông qua database dùng chung bằng heartbeat.
- **Database dùng chung:** Cả hai node kết nối đến **cùng một database**. Đây là kênh giao tiếp và heartbeat duy nhất, không cần mở thêm port giữa hai Zabbix Server.
- **Failover tự động:** Khi Active node ngừng cập nhật heartbeat vượt quá ngưỡng `FailoverDelay` (mặc định 1 phút), Standby node tự động tiếp quản vai trò Active.

```
┌──────────────────────────────┐     ┌──────────────────────────────┐
│  Node 1 — 192.168.197.148    │     │  Node 2 — 192.168.197.149    │
│                              │     │                              │
│  Zabbix Server (Active)      │     │  Zabbix Server (Standby)     │
│  Zabbix Frontend             │     │  Zabbix Agent                │
│  MariaDB (Shared DB)         │     │                              │
└──────────────┬───────────────┘     └──────────────┬───────────────┘
               │                                    │
               └──────── Kết nối cùng Database ─────┘
                         (192.168.197.148:3306)
```

### Điều Kiện Tiên Quyết

- Cả hai máy phải **ping được nhau**.
- **Đồng bộ thời gian hệ thống** (NTP) trên cả hai máy là bắt buộc — đồng hồ lệch sẽ gây lỗi heartbeat.
- Cả hai máy phải dùng **cùng phiên bản Zabbix**.

---

## 2. Mô Hình Lab và Hiện Trạng

### Sơ Đồ IP

| Máy | IP | Vai trò hiện tại | Vai trò sau khi cấu hình HA |
|---|---|---|---|
| Node 1 | `192.168.197.148` | Zabbix Server + Frontend + MariaDB + Agent | Zabbix Server **(Active)** + Frontend + DB + Agent |
| Node 2 | `192.168.197.149` | Zabbix Agent | Zabbix Server **(Standby)** + Agent |

> Database vẫn chạy trên Node 1 (`192.168.197.148`). Đây là điểm khác biệt so với kiến trúc 3 máy. Trong lab 2 máy, database **không** được HA — nếu Node 1 chết hoàn toàn (bao gồm cả DB), Node 2 cũng không thể failover. Tuy nhiên, nếu chỉ có **Zabbix Server process** trên Node 1 bị lỗi trong khi DB vẫn chạy, Node 2 sẽ failover thành công.


### Kiểm Tra Kết Nối Giữa Hai Máy

Trên cả hai máy, xác nhận ping được nhau:

```bash
# Trên Node 1
ping -c 3 192.168.197.149

# Trên Node 2
ping -c 3 192.168.197.148
```

Thêm hostname vào `/etc/hosts` trên **cả hai máy** để dễ quản lý:

```bash
sudo nano /etc/hosts
```

Thêm vào cuối file:

```
192.168.197.148  zabbix-node1
192.168.197.149  zabbix-node2
```

<img width="943" height="393" alt="image" src="https://github.com/user-attachments/assets/c8c84028-20fe-45bc-9cc3-8d7743b1d81f" />

---

## 3. Chuẩn Bị trên Node 1 (Máy đã có Zabbix Server)

Thực hiện trên `192.168.197.148`.

### 3.1 Cấp Quyền Database cho Node 2

Database hiện tại chỉ cho phép kết nối từ `localhost`. Cần mở thêm quyền cho IP của Node 2.

Đăng nhập MariaDB:

```bash
sudo mysql -u root -p
```

Tạo user cho Node 2 kết nối vào:

```sql
-- Cấp quyền cho Node 2 kết nối vào database zabbix
CREATE USER 'zabbix'@'192.168.197.149' IDENTIFIED BY 'zabbix_password';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'192.168.197.149';

-- Bật flag cần thiết khi thêm node mới vào cluster
SET GLOBAL log_bin_trust_function_creators = 1;

FLUSH PRIVILEGES;
EXIT;
```

> **Lưu ý:** Thay `zabbix_password` bằng mật khẩu database thực tế đang dùng trên hệ thống. Xem lại mật khẩu hiện tại trong file `/etc/zabbix/zabbix_server.conf` tại dòng `DBPassword=`.

<img width="957" height="653" alt="image" src="https://github.com/user-attachments/assets/07fe8f94-f83e-45cb-90ba-5eb898d783ee" />


### 3.2 Cấu Hình MariaDB Lắng Nghe Kết Nối Từ Bên Ngoài

Mặc định MariaDB chỉ lắng nghe `127.0.0.1`. Cần cho phép Node 2 kết nối được.

```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

Tìm dòng `bind-address` và sửa:

```ini
# Trước:
bind-address = 127.0.0.1

# Sau:
bind-address = 0.0.0.0
```
<img width="943" height="977" alt="image" src="https://github.com/user-attachments/assets/c6fa19dc-53b5-4816-b63a-d77f488358b2" />

Khởi động lại MariaDB:

```bash
sudo systemctl restart mariadb
```

### 3.3 Mở Port 3306 trên Tường Lửa cho Node 2

```bash
sudo ufw allow from 192.168.197.149 to any port 3306
sudo ufw reload
```

<img width="792" height="186" alt="image" src="https://github.com/user-attachments/assets/4f8f64be-2c10-47f6-b760-8062551d8877" />

Kiểm tra Node 2 đã kết nối được vào DB của Node 1 chưa (chạy lệnh này **từ Node 2**):

```bash
# Chạy trên Node 2 — 192.168.197.149
mysql -h 192.168.197.148 -u zabbix -p zabbix -e "SELECT 1;"
```

<img width="993" height="258" alt="image" src="https://github.com/user-attachments/assets/68ecff6e-add5-40ad-961f-2cd8800c9ef2" />

Kết quả mong đợi: `+---+` / `| 1 |` — kết nối thành công.

---

## 4. Cài Đặt Zabbix Server trên Node 2 (Máy Agent)

Thực hiện trên `192.168.197.149`. Node 2 đã có Zabbix Agent, chỉ cần cài thêm `zabbix-server-mysql` và `zabbix-frontend-php`.

### 4.1 Thêm Repo Zabbix (nếu chưa có)

Kiểm tra xem repo đã có chưa:

```bash
dpkg -l | grep zabbix-release
```

Nếu chưa có, thêm repo:

```bash
wget https://repo.zabbix.com/zabbix/7.2/release/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.2+ubuntu22.04_all.deb
sudo dpkg -i zabbix-release_latest_7.2+ubuntu22.04_all.deb
sudo apt update
```

### 4.2 Cài Đặt Zabbix Server và Frontend

```bash
sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf -y
```

> **Không chạy** `zabbix-sql-scripts` để import schema — database đã được khởi tạo từ Node 1, Node 2 dùng chung và **không được import lại** (sẽ gây lỗi hoặc xóa toàn bộ dữ liệu).

### 4.3 Không Khởi Động Zabbix Server Ngay

Chưa start service, sẽ cấu hình HA xong rồi mới start:

```bash
# Chỉ enable, CHƯA start
sudo systemctl enable zabbix-server
```

---

## 5. Cấu Hình HA trên Node 1 — Kích Hoạt Chế Độ Active

Thực hiện trên `192.168.197.148`.

### 5.1 Sửa File `zabbix_server.conf`

```bash
sudo nano /etc/zabbix/zabbix_server.conf
```

Dùng `Ctrl + W` để tìm từng tham số. Thêm hoặc sửa các dòng sau:

```ini
# Kết nối database (giữ nguyên nếu đã đúng)
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=a

# ===== THÊM MỚI — CẤU HÌNH HA =====

# Tên node này trong cluster — PHẢI UNIQUE
HANodeName=zabbix-node1

# Địa chỉ của node này — Frontend dùng để kết nối khi node này là Active
NodeAddress=192.168.197.148:10051
```
<img width="903" height="148" alt="image" src="https://github.com/user-attachments/assets/e8065858-8faa-4282-9a2c-cdc4c340364e" />


> **Quan trọng:** `HANodeName` là tham số kích hoạt chế độ HA. Nếu để trống hoặc bị comment, Zabbix Server chạy ở chế độ **standalone** (không HA).

### 5.2 Khởi Động Lại Zabbix Server trên Node 1

```bash
sudo systemctl restart zabbix-server
```

Kiểm tra log xác nhận HA đã lên Active:

```bash
sudo grep "HA" /var/log/zabbix/zabbix_server.log
```


Kết quả mong đợi:

```
starting HA manager
HA manager started in active mode
```
<img width="820" height="658" alt="image" src="https://github.com/user-attachments/assets/7ad643b4-a358-4fb7-834a-16e33d34671b" />

---

## 6. Cấu Hình HA trên Node 2 — Kích Hoạt Chế Độ Standby

Thực hiện trên `192.168.197.149`.

### 6.1 Sửa File `zabbix_server.conf`

```bash
sudo nano /etc/zabbix/zabbix_server.conf
```

Điền các tham số sau (database trỏ về Node 1):

```ini
# Kết nối đến database của Node 1
DBHost=192.168.197.148
DBName=zabbix
DBUser=zabbix
DBPassword=a

# ===== CẤU HÌNH HA =====

# Tên node — KHÁC với Node 1
HANodeName=zabbix-node2

# Địa chỉ của node này
NodeAddress=192.168.197.149:10051
```

<img width="980" height="945" alt="image" src="https://github.com/user-attachments/assets/c7674e64-7c88-4c82-b281-4995e9f2b558" />

### 6.2 Mở Port 10051 trên Tường Lửa Node 2

Port 10051 là port Zabbix Server dùng để nhận dữ liệu từ Agent (active check). Cần mở trên cả hai node.

```bash
# Trên Node 2
sudo ufw allow 10051/tcp
sudo ufw reload
```

Tương tự, kiểm tra Node 1 cũng đã mở port này:

```bash
# Trên Node 1
sudo ufw allow 10051/tcp
sudo ufw reload
```

### 6.3 Khởi Động Zabbix Server trên Node 2

```bash
sudo systemctl start zabbix-server
```

Kiểm tra log:

```bash
sudo grep "HA" /var/log/zabbix/zabbix_server.log
```

Kết quả mong đợi — Node 2 lên Standby vì Node 1 đã là Active:

```
starting HA manager
HA manager started in standby mode
```
<img width="765" height="269" alt="image" src="https://github.com/user-attachments/assets/3748ed5d-3b8f-4fe4-abed-8254030809db" />

---

## 7. Cấu Hình Zabbix Frontend cho HA

Frontend cần được cấu hình để **tự động phát hiện node đang Active** thay vì kết nối cứng đến một IP cụ thể.

### 7.1 Sửa File Cấu Hình Frontend trên Node 1

```bash
sudo nano /etc/zabbix/web/zabbix.conf.php
```

Tìm và **comment out** hai dòng sau (thêm `//` ở đầu nếu chưa có):

```php
// $ZBX_SERVER = '';
// $ZBX_SERVER_PORT = '';
```

Nếu file có nội dung kiểu:

```php
$ZBX_SERVER = '192.168.197.148';
$ZBX_SERVER_PORT = '10051';
```

Thì sửa thành:

```php
// $ZBX_SERVER = '192.168.197.148';
// $ZBX_SERVER_PORT = '10051';
```
<img width="895" height="118" alt="image" src="https://github.com/user-attachments/assets/2559c5c1-a172-4491-9f5a-90a5514d8a89" />


> **Giải thích:** Khi hai biến này được đặt giá trị, Frontend luôn kết nối cứng đến Node 1. Nếu Node 1 đang ở standby, Frontend sẽ báo lỗi "Zabbix server is not running" dù Node 2 đang chạy bình thường. Khi comment out, Frontend tự tra cứu database để tìm địa chỉ của node Active hiện tại.

### 7.2 Khởi Động Lại Apache

```bash
sudo systemctl restart apache2
```

---

## 8. Cập Nhật Cấu Hình Zabbix Agent trên Node 2

Agent trên Node 2 cần biết địa chỉ của cả hai Zabbix Server node, vì không thể biết trước node nào đang Active.

```bash
# Trên Node 2 — 192.168.197.149
sudo nano /etc/zabbix/zabbix_agentd.conf
```

Tìm và sửa các tham số:

```ini
# Passive checks — liệt kê cả hai node, cách nhau bởi dấu phẩy
Server=192.168.197.148,192.168.197.149

# Active checks — cả hai node trong cùng một cluster, cách nhau bởi dấu chấm phẩy
ServerActive=192.168.197.148;192.168.197.149

Hostname=zabbix-node2
```
<img width="655" height="401" alt="image" src="https://github.com/user-attachments/assets/2273233f-f201-422d-80be-5bc12c5de1d2" />

> **Lưu ý cú pháp:** `Server=` dùng dấu `,` (phẩy). `ServerActive=` dùng dấu `;` (chấm phẩy) để phân cách các node **trong cùng một cluster**. Đây là quy tắc bắt buộc, dùng sai dấu sẽ Agent không nhận cấu hình đúng.

Khởi động lại Agent:

```bash
sudo systemctl restart zabbix-agent
```

Tương tự, kiểm tra cấu hình Agent trên Node 1 (nếu có):

```bash
# Trên Node 1 — 192.168.197.148
sudo nano /etc/zabbix/zabbix_agentd.conf
```

```ini
Server=192.168.197.148,192.168.197.149
ServerActive=192.168.197.148;192.168.197.149
Hostname=zabbix-node1
```
<img width="799" height="465" alt="image" src="https://github.com/user-attachments/assets/3d26cee4-30f2-4e3d-985d-0055019f2d43" />

```bash
sudo systemctl restart zabbix-agent
```

---

## 9. Kiểm Tra Trạng Thái Cluster

### 9.1 Kiểm Tra qua Giao Diện Web

Đăng nhập vào `http://192.168.197.148/zabbix` → **Reports → System information**.

Phần **High availability cluster** hiển thị:

```
High availability cluster
Node name      Address                    Status    Last access
zabbix-node1   192.168.197.148:10051     active    2s ago
zabbix-node2   192.168.197.149:10051     standby   3s ago
```
<img width="1701" height="197" alt="image" src="https://github.com/user-attachments/assets/4701fc12-e76f-433c-9eee-a6d31bb4c80b" />

### 9.2 Kiểm Tra qua Command Line

Chạy lệnh sau trên bất kỳ node nào (Node 1 hoặc Node 2 đều được):

```bash
sudo zabbix_server -R ha_status
```

Kết quả mẫu:

```
Failover delay: 60 seconds
Cluster status:
# ID                          Name           Address                     Status    Last Access
1. ckvjyjy4o0001abc123def456  zabbix-node1  192.168.197.148:10051       active    2s
2. ckvjyjy4o0001xyz789uvw012  zabbix-node2  192.168.197.149:10051       standby   3s
```

<img width="965" height="305" alt="image" src="https://github.com/user-attachments/assets/b792d6ae-93b9-499a-95ed-c5ac57b1e236" />

### 9.3 Kiểm Tra Log

```bash
# Node 1
sudo grep "HA" /var/log/zabbix/zabbix_server.log

# Node 2
sudo grep "HA" /var/log/zabbix/zabbix_server.log

# Xem realtime trên Node 2
sudo tail -f /var/log/zabbix/zabbix_server.log | grep -i "ha\|fail\|active\|standby"
```

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/9d6c1a3d-febb-4c6e-9bf3-3db03bb65f99" />

### 9.4 Kiểm Tra Tiến Trình HA Manager

```bash
ps aux | grep zabbix_server
```

Nếu HA hoạt động đúng, sẽ thấy hai tiến trình — `zabbix_server` chính và `ha manager`:

```
/usr/sbin/zabbix_server -c /etc/zabbix/zabbix_server.conf
/usr/sbin/zabbix_server: ha manager
```

---

## 10. Kiểm Tra Failover

### 10.1 Mô Phỏng Node 1 Bị Lỗi

Trên Node 1 (`192.168.197.148`), dừng Zabbix Server:

```bash
# Dừng có kiểm soát (graceful — Node kịp báo trạng thái "stopped")
sudo systemctl stop zabbix-server
```

Hoặc mô phỏng crash đột ngột:

```bash
# Kill process không báo trước (mô phỏng crash)
sudo kill -9 $(pgrep -f "zabbix_server:" | head -1)
```

### 10.2 Quan Sát Node 2 Failover

Trên Node 2 (`192.168.197.149`), theo dõi log realtime:

```bash
sudo tail -f /var/log/zabbix/zabbix_server.log
```

Sau khoảng **5 giây** (graceful stop) hoặc **~65 giây** (crash đột ngột), Node 2 chuyển sang Active:

```
HA manager: standby node detected that active node stopped
HA manager: switching to active mode
HA manager started in active mode
```

Kiểm tra lại trạng thái:

```bash
sudo zabbix_server -R ha_status
```

Kết quả:

```
# ID    Name           Address                     Status    Last Access
1.      zabbix-node1  192.168.197.148:10051       stopped   75s
2.      zabbix-node2  192.168.197.149:10051       active    1s
```
sudo tail -f /var/log/zabbix/zabbix_server.log
<img width="1698" height="837" alt="image" src="https://github.com/user-attachments/assets/eed350ba-48fb-43ae-9521-16feb2737356" />

<img width="965" height="233" alt="image" src="https://github.com/user-attachments/assets/b91f36dc-7d85-45db-be8c-45436d736e0c" />

### 10.3 Kiểm Tra Frontend Sau Failover

Truy cập lại `http://192.168.197.148/zabbix` — Frontend vẫn hoạt động bình thường vì nó tự tra cứu database để tìm node Active mới (Node 2). Không cần làm gì thêm.

<img width="1920" height="921" alt="image" src="https://github.com/user-attachments/assets/23cb7c41-eeeb-4a7e-b00b-31b00a7ece29" />

### 10.4 Khôi Phục Node 1

Khởi động lại Zabbix Server trên Node 1:

```bash
sudo systemctl start zabbix-server
```

Node 1 tự động rejoin cluster ở trạng thái **standby** — không chiếm lại vai trò Active:

```
HA manager started in standby mode
```

Đây là hành vi đúng. Không có failback tự động để tránh flapping (dao động liên tục giữa hai node).

<img width="1680" height="808" alt="image" src="https://github.com/user-attachments/assets/898bacec-e595-4765-974e-7582a0c47c00" />

### 10.5 Bảng Thời Gian Failover

| Trường hợp | Thời gian failover |
|---|---|
| Graceful stop (`systemctl stop`) | ~5 giây |
| Crash đột ngột (`kill -9`) | `FailoverDelay` + 5 giây ≈ 65 giây (mặc định) |

---

## 11. Các Lệnh Quản Trị HA Cluster

### Xem trạng thái cluster

```bash
sudo zabbix_server -R ha_status
```
<img width="904" height="305" alt="image" src="https://github.com/user-attachments/assets/874ec96c-d770-4bbb-bfc0-50108e47044d" />

### Thay đổi Failover Delay

```bash
# Đặt 30 giây (phạm vi hợp lệ: 10s đến 15 phút)
sudo zabbix_server -R ha_set_failover_delay=30s

# Đặt 5 phút
sudo zabbix_server -R ha_set_failover_delay=5m
```

Failover delay ngắn hơn giúp phục hồi nhanh nhưng dễ false positive khi mạng chập chờn. Mặc định 1 phút là hợp lý cho hầu hết môi trường lab.

### Xóa Node Khỏi Cluster

Chỉ có thể xóa node ở trạng thái **stopped**:

```bash
# Lấy tên node trước
sudo zabbix_server -R ha_status

# Xóa theo tên
sudo zabbix_server -R ha_remove_node=zabbix-node1
```

---

