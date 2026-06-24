# Hướng dẫn Demo Chi Tiết Zabbix

---
Bước 1: Tải cấu hình Repo và Cài đặt Zabbix Agent
Bạn chạy lần lượt các lệnh sau trên con máy khách (client) cần giám sát:

Bash
# 1. Tải gói cấu hình repo Zabbix 7.2 mới nhất
wget https://repo.zabbix.com/zabbix/7.2/release/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.2+ubuntu22.04_all.deb

# 2. Cài đặt gói repo
sudo dpkg -i zabbix-release_latest_7.2+ubuntu22.04_all.deb

# 3. Cập nhật danh sách phần mềm và cài đặt zabbix-agent
sudo apt update
sudo apt install zabbix-agent -y
Bước 2: Cấu hình Agent trỏ về Zabbix Server
Mở file cấu hình của Zabbix Agent:

Bash
sudo nano /etc/zabbix/zabbix_agentd.conf
Bạn sử dụng tổ hợp phím Ctrl + W để tìm và sửa đúng 3 dòng sau:

Server= Điền IP của máy Zabbix Server 192.168.197.148

ServerActive= Điền IP của máy Zabbix Server 192.168.197.148

Hostname= tên client

Sau khi sửa xong, nhấn Ctrl + O -> Enter để lưu, và Ctrl + X để thoát.

Triển khai thêm 1 máy client zabbix

<img width="881" height="847" alt="image" src="https://github.com/user-attachments/assets/a178d1f8-2d22-4c58-b69f-9249d201ff00" />

Bước 3: Mở Port tường lửa (Firewall)
Zabbix Agent cần mở port 10050 để Server có thể kết nối vào lấy dữ liệu.

Nếu bạn đang dùng UFW (tường lửa mặc định của Ubuntu), chạy lệnh sau:

Bash
sudo ufw allow 10050/tcp
sudo ufw reload

<img width="1011" height="190" alt="image" src="https://github.com/user-attachments/assets/cb2332c6-451b-4899-92e7-c161488eea20" />

Bước 4: Khởi động dịch vụ Zabbix Agent
Khởi động lại Agent để nhận cấu hình mới và thiết lập cho phép Agent tự động chạy mỗi khi máy chủ khởi động:

Bash
sudo systemctl restart zabbix-agent
sudo systemctl enable zabbix-agent

<img width="1007" height="270" alt="image" src="https://github.com/user-attachments/assets/50311b24-0f89-47ef-8de1-4bc143056365" />

(Bạn có thể kiểm tra xem Agent đã chạy xanh tốt chưa bằng lệnh: systemctl status zabbix-agent)

<img width="958" height="613" alt="image" src="https://github.com/user-attachments/assets/bbf3d8d4-e986-428f-bd61-18117ebb52df" />

Bước 5: Thêm máy khách lên giao diện Web Zabbix Server
Đăng nhập vào Web UI của Zabbix Server.

Truy cập Data collection -> Hosts -> Bấm nút Create host (Góc trên bên phải).

Điền các thông tin sau vào tab Host:

Host name: Nhập chính xác cái tên bạn đã đặt ở mục Hostname= trong file cấu hình (ví dụ: Ubuntu-Web-01).

Templates: Tìm và chọn Linux by Zabbix agent.

Host groups: Chọn một nhóm phù hợp (ví dụ: Linux servers).

Interfaces: Bấm chữ Add -> Chọn Agent -> Nhập địa chỉ IP của con máy khách này vào ô IP address.

Bấm nút Add ở dưới cùng để hoàn tất.

<img width="1700" height="336" alt="image" src="https://github.com/user-attachments/assets/5c883abf-fa70-4fdb-929b-ded4c93e8f5a" />


<img width="1564" height="781" alt="image" src="https://github.com/user-attachments/assets/4f64a980-2583-404d-869e-9d11b893ccf1" />

## 1. Quản lý Nhóm và Máy chủ (Host Groups & Hosts)




### 1.1 Tạo Host Group

1. Đăng nhập Zabbix Web UI → vào **Data collection → Host groups**
2. Click **Create host group** (góc trên phải)
3. Điền thông tin:
   - **Group name:** `group a`
4. Click **Add** để lưu
5. Kiểm tra: nhóm `group a` xuất hiện trong danh sách

<img width="1375" height="546" alt="image" src="https://github.com/user-attachments/assets/acdb8caf-776a-400e-8cac-2d9bf6d81b24" />

### 1.2 Tạo Host

1. Vào **Data collection → Hosts** → Click **Create host**
2. Tab **Host** — điền các trường:
   - **Host name:** `Ubuntu-Apache-Node1-A`
   - **Host groups:** chọn `group a` vừa tạo
  

   <img width="1540" height="780" alt="image" src="https://github.com/user-attachments/assets/3db9403c-5247-480d-b118-0bde0089ff5f" />

3. Mục **Interfaces** → Click **Add** → chọn type **Agent**:
   - **IP address:** `<IP máy Ubuntu chạy Zabbix Agent>`
   - **Port:** `10050`
4. Click **Add** để lưu

Đây là zabbix server có sẵn của zabbix dc cấu hình như trên 

<img width="1473" height="704" alt="image" src="https://github.com/user-attachments/assets/11164c57-1f0b-4393-8a76-4aa6644938be" />

---

## 2. Sử dụng Mẫu Cấu Hình (Templates & Template Groups)

### 2.1 Xem Template Groups

1. Vào **Data collection → Template groups**
2. Quan sát các nhóm có sẵn: `Templates/Applications`, `Templates/Operating Systems`, v.v.

<img width="1783" height="818" alt="image" src="https://github.com/user-attachments/assets/9fc0116d-3bd3-4511-bfe8-b0b4e5f06c15" />

### 2.2 Gắn Template vào Host

1. Vào **Data collection → Hosts** → Click vào host đã tạo
2. Chuyển sang tab **Templates**
3. Trong ô **Link new templates**, gõ `Apache` → chọn **Apache by Zabbix agent** từ gợi ý
4. Click **Update** để lưu
5. Kiểm tra: tab Templates hiển thị template vừa gắn; các Items/Triggers được kế thừa tự động từ template

---

## 3. Thu Thập Dữ Liệu và Phân Loại (Items & Tags)

### 3.1 Tạo Item thủ công

1. Vào **Data collection → Hosts** → Cột **Items** của `Ubuntu-Apache-Node1` → Click **Create item**
2. Điền thông tin tab **Item**:
   - **Name:** `Apache Service Port 80`
   - **Type:** `Zabbix agent`
   - **Key:** `net.tcp.service[http,,80]`
   - **Type of information:** `Numeric (unsigned)`
   - **Update interval:** `30s`
3. Chuyển sang tab **Tags** → Click **Add**:
   - **Name:** `Service` | **Value:** `Apache`
4. Click **Add** để lưu
5. Kiểm tra: vào **Monitoring → Latest data**, lọc theo host `Ubuntu-Apache-Node1`, tìm item `Apache Service Port 80` → giá trị trả về `1` (port mở) hoặc `0` (đóng)

---

## 4. Đánh Giá Trạng Thái và Ghi Nhận Sự Cố (Triggers & Events)

### 4.1 Tạo Trigger

1. Vào **Data collection → Hosts** → Cột **Triggers** của `Ubuntu-Apache-Node1` → Click **Create trigger**
2. Điền thông tin:
   - **Name:** `Apache is DOWN on Node 1`
   - **Severity:** `High`
3. Mục **Expression** → Click **Add**:
   - Chọn item `Apache Service Port 80`
   - Function: `last()`
   - Operator: `=`
   - Value: `0`
   - Expression tạo ra: `last(/Ubuntu-Apache-Node1/net.tcp.service[http,,80])=0`
4. Click **Add** → Click **Add** để lưu trigger
5. Kiểm tra: dừng dịch vụ Apache trên máy Ubuntu (`sudo systemctl stop apache2`), sau 30-60s vào **Monitoring → Problems** → trigger `Apache is DOWN on Node 1` xuất hiện với trạng thái **PROBLEM**, kèm timestamp

### 4.2 Xem Events

1. Vào **Monitoring → Problems**
2. Quan sát cột: **Time**, **Host**, **Problem**, **Severity**, **Duration**
3. Click vào tên sự cố để xem chi tiết Event (lịch sử trạng thái, acknowledgement)

---

## 5. Thông Báo Tự Động (Alerts / Actions)

### 5.1 Cấu hình Media Type (nếu chưa có)

**Với Telegram:**
1. Vào **Alerts → Media types** → Click **Telegram**
2. Điền **Token** của Telegram Bot vào tham số `Token`
3. Click **Update**

**Với Email:**
1. Vào **Alerts → Media types** → Click **Email**
2. Điền SMTP Server, port, tài khoản gửi
3. Click **Update**

### 5.2 Tạo Action

1. Vào **Alerts → Actions → Trigger actions** → Click **Create action**
2. Tab **Action**:
   - **Name:** `Notify Apache Down`
3. Tab **Conditions** → Click **Add**:
   - Type: `Trigger tag`
   - Tag name: `Service` | Value: `Apache`
4. Tab **Operations** → Click **Add**:
   - **Operation type:** `Send message`
   - **Send to users:** chọn tài khoản admin (đã cấu hình media)
   - **Send only to:** chọn `Telegram` hoặc `Email`
5. Click **Add** → Click **Add** để lưu
6. Kiểm tra: trigger Apache → kiểm tra Telegram/Email nhận được thông báo tự động

---

## 6. Trực Quan Hóa Dữ Liệu (Visualization / Dashboards)

### 6.1 Tạo Dashboard mới

1. Vào **Monitoring → Dashboards** → Click **Create dashboard**
2. Đặt tên: `Web Server Monitoring`
3. Click **Apply**

### 6.2 Thêm Widget biểu đồ băng thông (Graph)

1. Click **Add widget**
2. Chọn type: **Graph (classic)**
3. Điền:
   - **Name:** `HAProxy Bandwidth`
   - **Item:** chọn item băng thông của HAProxy host
4. Click **Apply**

### 6.3 Thêm Widget giá trị thời gian thực (Item value)

1. Click **Add widget**
2. Chọn type: **Item value**
3. Điền:
   - **Name:** `Active Requests`
   - **Item:** chọn item request count của Apache
4. Click **Apply**

5. Click **Save changes** để lưu dashboard
6. Kiểm tra: biểu đồ và giá trị cập nhật theo thời gian thực

---

## 7. Phân Quyền Hệ Thống (Users & User Groups)

### 7.1 Tạo User Group

1. Vào **Users → User groups** → Click **Create user group**
2. Tab **User group**:
   - **Group name:** `IT Support Team`
3. Tab **Host group permissions** → Click **Add**:
   - **Host groups:** `Web Servers HA`
   - **Permissions:** `Read-write`
4. Click **Add** để lưu

### 7.2 Tạo User mới

1. Vào **Users → Users** → Click **Create user**
2. Tab **User**:
   - **Username:** `it_support_01`
   - **Name:** (tên nhân viên)
   - **Groups:** chọn `IT Support Team`
   - Đặt **Password**
3. Tab **Media** → Click **Add**:
   - **Type:** `Email` hoặc `Telegram`
   - **Send to:** địa chỉ email hoặc Chat ID Telegram của nhân viên
   - Cấu hình thời gian nhận thông báo nếu cần
4. Tab **Permissions** → kiểm tra role: `User`
5. Click **Add** để lưu
6. Kiểm tra: đăng nhập bằng tài khoản `it_support_01` → chỉ thấy host thuộc nhóm `Web Servers HA`, không thấy host nhóm khác

---

## 8. Báo Cáo Định Kỳ (Scheduled Reports)

### 8.1 Tạo Scheduled Report

1. Vào **Reports → Scheduled reports** → Click **Create report**
2. Điền thông tin:
   - **Name:** `Weekly Web Server Performance`
   - **Dashboard:** chọn `Web Server Monitoring` đã tạo ở bước 6
   - **Period:** `Week`
   - **Start time:** `08:00`
   - **Repeat on:** `Monday`
3. Mục **Subscriptions** → Click **Add**:
   - **User:** chọn các user cần nhận (admin, manager, v.v.)
   - **Create like:** `Me` (gửi đúng định dạng PDF)
4. Click **Add** để lưu
5. Kiểm tra: đợi đến lịch hoặc dùng nút **Test** (nếu có) để gửi thử → kiểm tra email nhận được file PDF báo cáo dashboard

---

> **Lưu ý demo:** Nên chuẩn bị sẵn 1 máy Ubuntu có Zabbix Agent cài đặt và Apache đang chạy để minh họa trực tiếp trigger PROBLEM → RESOLVED khi tắt/bật lại dịch vụ Apache.
