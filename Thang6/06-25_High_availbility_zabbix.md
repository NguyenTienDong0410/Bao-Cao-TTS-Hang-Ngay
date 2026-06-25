# Hướng Dẫn Thực Hành: Zabbix High Availability (Native HA Cluster)

---

## Mục Lục

1. [Tổng Quan về Zabbix HA](#1-tổng-quan-về-zabbix-ha)
2. [Mô Hình Lab và Yêu Cầu](#2-mô-hình-lab-và-yêu-cầu)
3. [Chuẩn Bị Database Server](#3-chuẩn-bị-database-server)
4. [Cài Đặt Zabbix Server trên Cả Hai Node](#4-cài-đặt-zabbix-server-trên-cả-hai-node)
5. [Cấu Hình HA trên Node 1 (Active)](#5-cấu-hình-ha-trên-node-1-active)
6. [Cấu Hình HA trên Node 2 (Standby)](#6-cấu-hình-ha-trên-node-2-standby)
7. [Cấu Hình Zabbix Frontend](#7-cấu-hình-zabbix-frontend)
8. [Cấu Hình Zabbix Agent cho Môi Trường HA](#8-cấu-hình-zabbix-agent-cho-môi-trường-ha)
9. [Kiểm Tra Trạng Thái Cluster](#9-kiểm-tra-trạng-thái-cluster)
10. [Kiểm Tra Failover](#10-kiểm-tra-failover)
11. [Các Lệnh Quản Trị HA Cluster](#11-các-lệnh-quản-trị-ha-cluster)

---

## 1. Tổng Quan về Zabbix HA

### HA là gì?

**High Availability (HA)** là kiến trúc giúp hệ thống tiếp tục hoạt động không gián đoạn ngay cả khi một thành phần bị lỗi. Trong Zabbix, nếu không có HA, khi Zabbix Server gặp sự cố (crash, mất điện, hết disk...) thì toàn bộ quá trình thu thập dữ liệu, phát hiện sự cố và gửi cảnh báo đều dừng lại.

### Native HA của Zabbix (từ phiên bản 6.0)

Trước phiên bản 6.0, người dùng phải dùng các giải pháp HA bên thứ ba (Pacemaker, Corosync, Keepalived...) để đảm bảo tính sẵn sàng, vốn phức tạp và đòi hỏi nhiều chuyên môn. Từ **Zabbix 6.0 LTS**, tính năng **native HA** được tích hợp trực tiếp, không cần phần mềm bổ sung.

### Cơ chế hoạt động

Native HA của Zabbix hoạt động theo mô hình **Active/Passive**:

- **Active node:** Node đang thực sự xử lý công việc — thu thập dữ liệu, đánh giá trigger, gửi cảnh báo.
- **Standby node:** Node dự phòng, ở chế độ chờ, liên tục theo dõi node Active thông qua database dùng chung.
- **Database dùng chung:** Tất cả các node trong cluster cùng kết nối đến một database. Đây là kênh giao tiếp và heartbeat giữa các node.
- **Failover tự động:** Khi Active node ngừng cập nhật heartbeat vượt quá ngưỡng `FailoverDelay` (mặc định 1 phút), Standby node tự động lên thành Active và tiếp quản toàn bộ.

```
┌─────────────────┐         ┌─────────────────┐
│  Zabbix Node 1  │         │  Zabbix Node 2  │
│  (Active)       │         │  (Standby)      │
│  192.168.1.11   │         │  192.168.1.12   │
└────────┬────────┘         └────────┬────────┘
         │                           │
         └──────────┬────────────────┘
                    │  Shared Database
              ┌─────▼──────┐
              │  MariaDB   │
              │ 192.168.1.10│
              └────────────┘
```

### Những gì HA bảo vệ và không bảo vệ

| Bảo vệ | Không bảo vệ |
|---|---|
| Zabbix Server process crash | Database bị lỗi (cần giải pháp riêng cho DB) |
| Máy chủ chạy Zabbix Server hỏng | Zabbix Frontend (cần cấu hình riêng) |
| Quá trình bảo trì / nâng cấp node | Mất dữ liệu do disk failure trên DB |
| OS-level failure | Network partition (split-brain được xử lý qua DB) |

---

## 2. Mô Hình Lab và Yêu Cầu

### Sơ đồ IP

| Máy | Vai trò | IP | OS |
|---|---|---|---|
| `zabbix-db` | Database Server (MariaDB) | `192.168.1.10` | Ubuntu 22.04 |
| `zabbix-node1` | Zabbix Server Node 1 (Active) | `192.168.1.11` | Ubuntu 22.04 |
| `zabbix-node2` | Zabbix Server Node 2 (Standby) | `192.168.1.12` | Ubuntu 22.04 |

> Trong lab VMware Workstation, có thể dùng Network Adapter chế độ **Host-only** hoặc **NAT** và điều chỉnh IP phù hợp với dải mạng thực tế.

### Yêu cầu hệ thống (tối thiểu cho lab)

| Máy | RAM | CPU | Disk |
|---|---|---|---|
| `zabbix-db` | 2 GB | 2 vCPU | 20 GB |
| `zabbix-node1` | 2 GB | 2 vCPU | 20 GB |
| `zabbix-node2` | 2 GB | 2 vCPU | 20 GB |

### Yêu cầu tiên quyết

- Ba máy ảo Ubuntu 22.04 đã được cài đặt và có thể ping nhau.
- **Thời gian hệ thống đồng bộ** trên cả ba máy (dùng NTP/chrony). Đây là yêu cầu bắt buộc — đồng hồ lệch sẽ gây lỗi cluster và heartbeat.
- Tên hostname đã được đặt đúng trên từng máy.

Thiết lập hostname (chạy trên từng máy tương ứng):

```bash
# Trên zabbix-db
sudo hostnamectl set-hostname zabbix-db

# Trên zabbix-node1
sudo hostnamectl set-hostname zabbix-node1

# Trên zabbix-node2
sudo hostnamectl set-hostname zabbix-node2
```

Thêm vào `/etc/hosts` trên **cả ba máy**:

```bash
sudo nano /etc/hosts
```

Thêm các dòng sau:

```
192.168.1.10  zabbix-db
192.168.1.11  zabbix-node1
192.168.1.12  zabbix-node2
```

---

## 3. Chuẩn Bị Database Server

Thực hiện toàn bộ phần này trên máy `zabbix-db` (192.168.1.10).

### 3.1 Cài đặt MariaDB

```bash
sudo apt update
sudo apt install mariadb-server -y
sudo systemctl enable mariadb
sudo systemctl start mariadb
```

Chạy bảo mật ban đầu:

```bash
sudo mysql_secure_installation
```

Làm theo hướng dẫn: đặt mật khẩu root, xóa user ẩn danh, tắt remote root login, xóa test database.

### 3.2 Tạo Database và User cho Zabbix

Đăng nhập vào MariaDB:

```bash
sudo mysql -u root -p
```

Thực hiện lần lượt các lệnh SQL sau:

```sql
-- Tạo database
CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;

-- Tạo user và cấp quyền cho cả hai Zabbix Server node
CREATE USER 'zabbix'@'192.168.1.11' IDENTIFIED BY 'zabbix_password';
CREATE USER 'zabbix'@'192.168.1.12' IDENTIFIED BY 'zabbix_password';

GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'192.168.1.11';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'192.168.1.12';

-- Cần thiết cho việc import schema
SET GLOBAL log_bin_trust_function_creators = 1;

FLUSH PRIVILEGES;
EXIT;
```

> **Lưu ý:** Cấp quyền riêng cho từng IP của hai Zabbix Server node, không dùng `'zabbix'@'%'` trong môi trường production vì lý do bảo mật.

### 3.3 Mở Port MariaDB cho Hai Node

Nếu đang dùng UFW:

```bash
sudo ufw allow from 192.168.1.11 to any port 3306
sudo ufw allow from 192.168.1.12 to any port 3306
sudo ufw reload
```

Cấu hình MariaDB lắng nghe kết nối từ bên ngoài (mặc định chỉ nghe `localhost`):

```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

Tìm dòng `bind-address` và sửa thành IP của máy DB:

```ini
bind-address = 192.168.1.10
```

Khởi động lại MariaDB:

```bash
sudo systemctl restart mariadb
```

---

## 4. Cài Đặt Zabbix Server trên Cả Hai Node

Thực hiện **đồng thời** các bước dưới đây trên cả `zabbix-node1` và `zabbix-node2`.

### 4.1 Thêm Repo Zabbix 7.2

```bash
wget https://repo.zabbix.com/zabbix/7.2/release/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.2+ubuntu22.04_all.deb
sudo dpkg -i zabbix-release_latest_7.2+ubuntu22.04_all.deb
sudo apt update
```

### 4.2 Cài Đặt Zabbix Server, Frontend và Agent

```bash
sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent -y
```

### 4.3 Import Database Schema (CHỈ thực hiện trên Node 1)

Bước này **chỉ chạy một lần duy nhất** trên `zabbix-node1`. Node 2 dùng chung database nên **không import lại**.

```bash
# Trên zabbix-node1
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | \
  mysql --default-character-set=utf8mb4 \
  -h 192.168.1.10 -u zabbix -p zabbix
```

Nhập mật khẩu `zabbix_password` khi được hỏi. Quá trình này mất vài phút.

Sau khi import xong, tắt `log_bin_trust_function_creators` trên DB server:

```bash
# Trên zabbix-db
sudo mysql -u root -p -e "SET GLOBAL log_bin_trust_function_creators = 0;"
```

---

## 5. Cấu Hình HA trên Node 1 (Active)

Thực hiện trên `zabbix-node1` (192.168.1.11).

### 5.1 Cấu Hình File `zabbix_server.conf`

```bash
sudo nano /etc/zabbix/zabbix_server.conf
```

Tìm và sửa (hoặc thêm) các dòng sau:

```ini
# Kết nối database
DBHost=192.168.1.10
DBName=zabbix
DBUser=zabbix
DBPassword=zabbix_password

# ===== CẤU HÌNH HA =====
# Tên node trong cluster — phải UNIQUE, không được trùng với node khác
HANodeName=zabbix-node1

# Địa chỉ và port mà Frontend dùng để kết nối đến node này khi nó là Active
# Format: <IP>:<port>
NodeAddress=192.168.1.11:10051
```

> **Quan trọng:** Tham số `HANodeName` là tham số kích hoạt chế độ HA. Nếu tham số này để trống hoặc bị comment, Zabbix Server sẽ chạy ở chế độ **standalone** (không HA).

### 5.2 Khởi Động Zabbix Server trên Node 1

```bash
sudo systemctl enable zabbix-server zabbix-agent apache2
sudo systemctl start zabbix-server zabbix-agent apache2
```

Kiểm tra log để xác nhận HA đã khởi động thành công:

```bash
sudo grep "HA" /var/log/zabbix/zabbix_server.log
```

Kết quả mong đợi:

```
starting HA manager
HA manager started in active mode
```

---

## 6. Cấu Hình HA trên Node 2 (Standby)

Thực hiện trên `zabbix-node2` (192.168.1.12).

### 6.1 Cấu Hình File `zabbix_server.conf`

```bash
sudo nano /etc/zabbix/zabbix_server.conf
```

Cấu hình **giống Node 1** nhưng thay đổi `HANodeName` và `NodeAddress`:

```ini
# Kết nối database — GIỐNG HỆT Node 1 (cùng database)
DBHost=192.168.1.10
DBName=zabbix
DBUser=zabbix
DBPassword=zabbix_password

# ===== CẤU HÌNH HA =====
# Tên node — KHÁC với Node 1
HANodeName=zabbix-node2

# Địa chỉ của Node 2
NodeAddress=192.168.1.12:10051
```

### 6.2 Khởi Động Zabbix Server trên Node 2

```bash
sudo systemctl enable zabbix-server zabbix-agent apache2
sudo systemctl start zabbix-server zabbix-agent apache2
```

Kiểm tra log:

```bash
sudo grep "HA" /var/log/zabbix/zabbix_server.log
```

Kết quả mong đợi cho Node 2:

```
starting HA manager
HA manager started in standby mode
```

> Node 2 khởi động ở chế độ **standby** vì Node 1 đã đăng ký là Active trước đó. Đây là hành vi bình thường và đúng.

---

## 7. Cấu Hình Zabbix Frontend

Phần này đặc biệt quan trọng. Frontend cần được cấu hình để **tự động kết nối đến node đang Active**, không hardcode IP của một node cụ thể.

### 7.1 Cấu Hình qua Trình Duyệt (lần đầu)

Truy cập `http://192.168.1.11/zabbix` hoặc `http://192.168.1.12/zabbix` để chạy Web Setup Wizard.

Tại bước **Configure DB connection**, điền:

| Trường | Giá trị |
|---|---|
| Database host | `192.168.1.10` |
| Database port | `3306` |
| Database name | `zabbix` |
| User | `zabbix` |
| Password | `zabbix_password` |

Tại bước **Zabbix server details**, **để trống** cả `Host` và `Port` — Frontend sẽ tự động phát hiện node Active qua database.

### 7.2 Sửa File Cấu Hình Frontend (bắt buộc nếu nâng cấp từ phiên bản cũ)

Nếu đã có cài đặt trước đó, cần kiểm tra và sửa file cấu hình frontend:

```bash
sudo nano /etc/zabbix/web/zabbix.conf.php
```

Đảm bảo **comment out hoặc xóa** hai dòng sau (nếu có):

```php
// $ZBX_SERVER = '';        // <-- phải được comment out
// $ZBX_SERVER_PORT = '';   // <-- phải được comment out
```

Nếu hai biến này được đặt giá trị cụ thể, Frontend sẽ luôn kết nối cứng đến một node và sẽ báo lỗi "Zabbix server is not running" khi node đó ở standby. Khi để trống, Frontend tự tra cứu database để tìm địa chỉ của node Active.

### 7.3 Khởi Động Lại Apache

```bash
sudo systemctl restart apache2
```

---

## 8. Cấu Hình Zabbix Agent cho Môi Trường HA

Tất cả các máy khách (Zabbix Agent) cần biết địa chỉ của **cả hai node**, vì không biết node nào đang Active vào bất kỳ thời điểm nào.

```bash
sudo nano /etc/zabbix/zabbix_agentd.conf
```

Sửa các tham số sau:

```ini
# Passive checks — liệt kê cả hai node, cách nhau bởi dấu phẩy
Server=192.168.1.11,192.168.1.12

# Active checks — liệt kê cả hai node, cách nhau bởi dấu chấm phẩy
# (Active checks dùng dấu ; để phân cách các node trong cùng cluster)
ServerActive=192.168.1.11;192.168.1.12

Hostname=<tên máy client>
```

> **Lưu ý cú pháp:** Trong `ServerActive`, các node của **cùng một cluster** phân cách bằng dấu `;`. Nếu có nhiều server/cluster khác nhau thì phân cách bằng dấu `,`. Ví dụ: `ServerActive=node1;node2,other_server` — cluster gồm node1 và node2, cộng với một server độc lập khác.

Khởi động lại Agent:

```bash
sudo systemctl restart zabbix-agent
```

---

## 9. Kiểm Tra Trạng Thái Cluster

### 9.1 Kiểm Tra qua Giao Diện Web

Đăng nhập vào Zabbix Web UI → **Reports → System information**.

Phần **High availability cluster** hiển thị trạng thái tất cả các node:

```
High availability cluster
Node name    Address              Status    Last access
zabbix-node1 192.168.1.11:10051  active    2s ago
zabbix-node2 192.168.1.12:10051  standby   3s ago
```

### 9.2 Kiểm Tra qua Command Line

Chạy lệnh sau trên bất kỳ node nào:

```bash
sudo zabbix_server -R ha_status
```

Kết quả mẫu:

```
Failover delay: 60 seconds
Cluster status:
# ID                         Name           Address                  Status    Last Access
1. ckvjyjy4o0001v6renrp0001  zabbix-node1  192.168.1.11:10051       active    2s
2. ckvjyjy4o0001v6renrp0002  zabbix-node2  192.168.1.12:10051       standby   3s
```

### 9.3 Kiểm Tra Log

```bash
# Xem log HA trên Node 1
sudo grep "HA" /var/log/zabbix/zabbix_server.log

# Xem log realtime
sudo tail -f /var/log/zabbix/zabbix_server.log | grep -i "ha\|fail\|active\|standby"
```

---

## 10. Kiểm Tra Failover

Đây là bước quan trọng nhất để xác nhận HA hoạt động đúng.

### 10.1 Mô Phỏng Node 1 Bị Lỗi

Trên `zabbix-node1`, dừng Zabbix Server:

```bash
# Dừng có kiểm soát (graceful stop)
sudo systemctl stop zabbix-server
```

Hoặc mô phỏng crash đột ngột:

```bash
# Tắt process đột ngột (không graceful)
sudo kill -9 $(pgrep -f zabbix_server | head -1)
```

### 10.2 Quan Sát Quá Trình Failover

Trên `zabbix-node2`, theo dõi log:

```bash
sudo tail -f /var/log/zabbix/zabbix_server.log
```

Sau khoảng **5–65 giây** (tùy cách dừng), Node 2 sẽ lên Active:

```
HA manager: standby node detected that active node stopped
HA manager: switching to active mode
HA manager started in active mode
```

Kiểm tra lại trạng thái trên Node 2:

```bash
sudo zabbix_server -R ha_status
```

Kết quả:

```
# ID    Name           Address                  Status    Last Access
1.      zabbix-node1  192.168.1.11:10051       stopped   75s
2.      zabbix-node2  192.168.1.12:10051       active    1s
```

### 10.3 Khôi Phục Node 1 và Quan Sát Rejoining

Khởi động lại Zabbix Server trên Node 1:

```bash
sudo systemctl start zabbix-server
```

Node 1 sẽ tự động rejoin cluster ở trạng thái **standby** (không chiếm lại vai trò Active):

```
HA manager started in standby mode
```

Đây là hành vi đúng — node nào đang chạy Active sẽ giữ nguyên vai trò, không có failback tự động để tránh flapping.

### 10.4 Kiểm Tra Thời Gian Failover

Thời gian failover phụ thuộc vào cách node Active bị dừng:

| Trường hợp | Thời gian failover |
|---|---|
| Graceful stop (`systemctl stop`) | ~5 giây (node kịp báo trạng thái "stopped") |
| Crash đột ngột (`kill -9`) | `FailoverDelay` + 5 giây = ~65 giây (mặc định) |

---

## 11. Các Lệnh Quản Trị HA Cluster

### Xem trạng thái cluster

```bash
sudo zabbix_server -R ha_status
```

### Thay đổi Failover Delay

```bash
# Đặt failover delay thành 30 giây (range: 10s đến 15 phút)
sudo zabbix_server -R ha_set_failover_delay=30s

# Đặt thành 5 phút
sudo zabbix_server -R ha_set_failover_delay=5m
```

> Failover delay ngắn hơn → phục hồi nhanh hơn nhưng dễ false positive khi mạng chập chờn. Failover delay dài hơn → ổn định hơn nhưng downtime khi có lỗi thực sẽ lâu hơn. Mặc định 1 phút là cân bằng hợp lý cho hầu hết môi trường.

### Xóa node khỏi cluster

Chỉ có thể xóa node ở trạng thái **stopped**. Node đang active hoặc standby không thể xóa.

```bash
# Lấy tên node trước
sudo zabbix_server -R ha_status

# Xóa node theo tên
sudo zabbix_server -R ha_remove_node=zabbix-node1
```

### Kiểm tra tiến trình HA Manager

```bash
ps aux | grep zabbix_server
```

Kết quả đúng sẽ thấy tiến trình `ha manager` chạy song song với tiến trình chính:

```
/usr/sbin/zabbix_server -c /etc/zabbix/zabbix_server.conf
/usr/sbin/zabbix_server: ha manager
```

---

