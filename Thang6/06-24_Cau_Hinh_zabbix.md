# Báo Cáo Thực Hành: Demo Hệ Thống Giám Sát Zabbix

---

## Mục Lục

1. [Cài Đặt và Cấu Hình Zabbix Agent trên Máy Khách](#1-cài-đặt-và-cấu-hình-zabbix-agent-trên-máy-khách)
2. [Quản Lý Nhóm và Máy Chủ (Host Groups & Hosts)](#2-quản-lý-nhóm-và-máy-chủ-host-groups--hosts)
3. [Sử Dụng Mẫu Cấu Hình (Templates & Template Groups)](#3-sử-dụng-mẫu-cấu-hình-templates--template-groups)
4. [Thu Thập Dữ Liệu và Phân Loại (Items & Tags)](#4-thu-thập-dữ-liệu-và-phân-loại-items--tags)
5. [Đánh Giá Trạng Thái và Ghi Nhận Sự Cố (Triggers & Events)](#5-đánh-giá-trạng-thái-và-ghi-nhận-sự-cố-triggers--events)
6. [Trực Quan Hóa Dữ Liệu (Dashboards)](#6-trực-quan-hóa-dữ-liệu-dashboards)
7. [Phân Quyền Hệ Thống (Users & User Groups)](#7-phân-quyền-hệ-thống-users--user-groups)
8. [Báo Cáo Định Kỳ (Scheduled Reports)](#8-báo-cáo-định-kỳ-scheduled-reports)

---

## 1. Cài Đặt và Cấu Hình Zabbix Agent trên Máy Khách

### Tổng Quan

Zabbix hoạt động theo mô hình server–agent: **Zabbix Server** là trung tâm xử lý và lưu trữ dữ liệu, còn **Zabbix Agent** là tiến trình chạy trên mỗi máy cần giám sát, có nhiệm vụ thu thập số liệu (CPU, RAM, dịch vụ, v.v.) và gửi về Server. Phần này trình bày các bước cài đặt và cấu hình Agent trên máy Ubuntu, sau đó đăng ký máy đó lên giao diện Web của Zabbix Server.

---

### Bước 1: Tải và Cài Đặt Zabbix Agent

Thực hiện lần lượt các lệnh sau trên máy khách (client) cần giám sát:

```bash
# 1. Tải gói cấu hình repo Zabbix 7.2 cho Ubuntu 22.04
wget https://repo.zabbix.com/zabbix/7.2/release/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.2+ubuntu22.04_all.deb

# 2. Cài đặt gói repo vừa tải
sudo dpkg -i zabbix-release_latest_7.2+ubuntu22.04_all.deb

# 3. Cập nhật danh sách phần mềm và cài đặt Zabbix Agent
sudo apt update
sudo apt install zabbix-agent -y
```

---

### Bước 2: Cấu Hình Agent Trỏ Về Zabbix Server

Mở file cấu hình của Zabbix Agent:

```bash
sudo nano /etc/zabbix/zabbix_agentd.conf
```

Sử dụng tổ hợp phím `Ctrl + W` để tìm kiếm và sửa đúng **3 dòng** sau:

| Tham số | Giá trị cần điền | Giải thích |
|---|---|---|
| `Server=` | `192.168.197.148` | IP của Zabbix Server — chấp nhận kết nối passive từ Server |
| `ServerActive=` | `192.168.197.148` | IP của Zabbix Server — Agent chủ động gửi dữ liệu (active check) |
| `Hostname=` | `<tên client>` | Tên định danh của máy này; phải khớp với `Host name` khi tạo host trên Web UI |

Sau khi sửa xong, nhấn `Ctrl + O` → `Enter` để lưu, rồi `Ctrl + X` để thoát.

> **Lưu ý:** Giá trị `Hostname` ở đây phải trùng khớp chính xác với trường **Host name** khi thêm máy vào giao diện Web (Bước 5). Nếu không khớp, Zabbix Server sẽ không nhận dữ liệu từ Agent.

Minh họa: triển khai thêm một máy client Zabbix:

![Triển khai thêm máy client Zabbix](https://github.com/user-attachments/assets/a178d1f8-2d22-4c58-b69f-9249d201ff00)

---

### Bước 3: Mở Port Tường Lửa (Firewall)

Zabbix Agent lắng nghe trên **port 10050/TCP**. Server cần kết nối vào port này để lấy dữ liệu (passive check). Nếu máy khách đang dùng UFW (tường lửa mặc định của Ubuntu), chạy lệnh sau để mở port:

```bash
sudo ufw allow 10050/tcp
sudo ufw reload
```

![Mở port 10050 trên UFW](https://github.com/user-attachments/assets/cb2332c6-451b-4899-92e7-c161488eea20)

---

### Bước 4: Khởi Động Dịch Vụ Zabbix Agent

Khởi động lại Agent để nhận cấu hình mới và thiết lập tự động chạy khi máy khởi động:

```bash
sudo systemctl restart zabbix-agent
sudo systemctl enable zabbix-agent
```

![Khởi động và enable Zabbix Agent](https://github.com/user-attachments/assets/50311b24-0f89-47ef-8de1-4bc143056365)

Kiểm tra trạng thái Agent:

```bash
systemctl status zabbix-agent
```

Kết quả mong đợi: trạng thái `active (running)` hiển thị màu xanh.

![Trạng thái Zabbix Agent đang chạy](https://github.com/user-attachments/assets/bbf3d8d4-e986-428f-bd61-18117ebb52df)

---

### Bước 5: Thêm Máy Khách Lên Giao Diện Web Zabbix Server

Sau khi Agent đã chạy thành công trên máy khách, cần đăng ký máy đó lên Zabbix Server để Server bắt đầu thu thập dữ liệu.

**Thực hiện:**

1. Đăng nhập vào Web UI của Zabbix Server.
2. Truy cập **Data collection → Hosts** → Click **Create host** (góc trên bên phải).
3. Điền các thông tin sau vào tab **Host**:
   - **Host name:** Nhập chính xác tên đã đặt ở mục `Hostname=` trong file cấu hình (ví dụ: `Ubuntu-Web-01`).
   - **Templates:** Tìm và chọn `Linux by Zabbix agent`.
   - **Host groups:** Chọn nhóm phù hợp (ví dụ: `Linux servers`).
   - **Interfaces:** Click **Add** → Chọn **Agent** → Nhập địa chỉ IP của máy khách vào ô **IP address**, port mặc định là `10050`.
4. Click **Add** ở cuối trang để hoàn tất.

![Danh sách hosts trên Zabbix Web UI](https://github.com/user-attachments/assets/5c883abf-fa70-4fdb-929b-ded4c93e8f5a)

![Tạo host mới trên Zabbix Web UI](https://github.com/user-attachments/assets/4f64a980-2583-404d-869e-9d11b893ccf1)

---

## 2. Quản Lý Nhóm và Máy Chủ (Host Groups & Hosts)

### Tổng Quan

**Host Groups** là cơ chế phân loại, tổ chức các máy chủ (hosts) trong Zabbix theo tiêu chí tùy chọn (theo phòng ban, môi trường, chức năng, v.v.). Việc nhóm hosts giúp đơn giản hóa việc quản lý, áp dụng chính sách phân quyền, và lọc dữ liệu giám sát. Một host có thể thuộc nhiều nhóm đồng thời.

---

### 2.1 Tạo Host Group

1. Đăng nhập Zabbix Web UI → vào **Data collection → Host groups**.
2. Click **Create host group** (góc trên bên phải).
3. Điền thông tin:
   - **Group name:** `group a`
4. Click **Add** để lưu.
5. Kiểm tra: nhóm `group a` xuất hiện trong danh sách.

![Tạo Host Group trong Zabbix](https://github.com/user-attachments/assets/acdb8caf-776a-400e-8cac-2d9bf6d81b24)

---

### 2.2 Tạo Host

1. Vào **Data collection → Hosts** → Click **Create host**.
2. Tab **Host** — điền các trường:
   - **Host name:** `Ubuntu-Apache-Node1-A`
   - **Host groups:** chọn `group a` vừa tạo.

![Tạo Host mới - điền thông tin](https://github.com/user-attachments/assets/3db9403c-5247-480d-b118-0bde0089ff5f)

3. Mục **Interfaces** → Click **Add** → chọn type **Agent**:
   - **IP address:** `<IP máy Ubuntu chạy Zabbix Agent>`
   - **Port:** `10050`
4. Click **Add** để lưu.

Zabbix Server có sẵn sau khi cấu hình như trên:

![Zabbix Server hiển thị host vừa tạo](https://github.com/user-attachments/assets/11164c57-1f0b-4393-8a76-4aa6644938be)

---

## 3. Sử Dụng Mẫu Cấu Hình (Templates & Template Groups)

### Tổng Quan

**Templates** là tập hợp các cấu hình giám sát được định nghĩa sẵn, bao gồm Items, Triggers, Graphs, và Dashboards cho một loại đối tượng cụ thể (Linux server, Apache, MySQL, v.v.). Thay vì cấu hình từng thứ thủ công cho mỗi host, người quản trị chỉ cần **gắn (link) template** vào host — toàn bộ cấu hình từ template sẽ được kế thừa tự động. Đây là tính năng cốt lõi giúp Zabbix có khả năng mở rộng lớn với chi phí quản lý thấp.

---

### 3.1 Xem Template Groups

1. Vào **Data collection → Template groups**.
2. Quan sát các nhóm có sẵn: `Templates/Applications`, `Templates/Operating Systems`, v.v.

![Danh sách Template Groups trong Zabbix](https://github.com/user-attachments/assets/9fc0116d-3bd3-4511-bfe8-b0b4e5f06c15)

---

### 3.2 Gắn Template vào Host

1. Vào **Data collection → Hosts** → Click vào host đã tạo.
2. Chuyển sang tab **Templates**.
3. Trong ô **Link new templates**, gõ `Apache` → chọn **Apache by Zabbix agent** từ gợi ý.
4. Click **Update** để lưu.
5. Kiểm tra: tab **Templates** hiển thị template vừa gắn; các Items và Triggers được kế thừa tự động từ template.

![Gắn template Apache vào host](https://github.com/user-attachments/assets/f158e907-1032-48a1-b5ac-18cf048af4c8)

---

## 4. Thu Thập Dữ Liệu và Phân Loại (Items & Tags)

### Tổng Quan

**Item** là đơn vị thu thập dữ liệu nhỏ nhất trong Zabbix — mỗi Item định nghĩa **một chỉ số cần đo** (ví dụ: trạng thái port 80, mức sử dụng CPU, dung lượng RAM) cùng với phương thức lấy dữ liệu (Zabbix Agent, SNMP, HTTP, v.v.) và tần suất thu thập. **Tags** là nhãn metadata gắn vào Item, giúp phân loại và lọc dữ liệu nhanh chóng trong phần Monitoring.

---

### 4.1 Tạo Item Thủ Công

1. Vào **Data collection → Hosts** → Cột **Items** của host `Ubuntu-Apache-Node1` → Click **Create item**.
2. Điền thông tin tại tab **Item**:

| Trường | Giá trị |
|---|---|
| **Name** | `Apache Service Port 80` |
| **Type** | `Zabbix agent` |
| **Key** | `net.tcp.service[http,,80]` |
| **Type of information** | `Numeric (unsigned)` |
| **Update interval** | `30s` |

> **Giải thích Key:** `net.tcp.service[http,,80]` là khóa kiểm tra Agent có kết nối được đến port 80 (HTTP) hay không. Giá trị trả về là `1` nếu port đang mở, `0` nếu đóng.

![Tạo Item giám sát Apache Port 80](https://github.com/user-attachments/assets/0c2f237c-b7ad-40f1-b04f-c458a838f379)

3. Chuyển sang tab **Tags** → Click **Add**:
   - **Name:** `Service` | **Value:** `Apache`
4. Click **Add** để lưu.

**Kiểm tra kết quả:** Vào **Monitoring → Latest data**, lọc theo host `Zabbix Client`, tìm item `Apache Service Port 80`:

![Latest data - Item Apache Port 80](https://github.com/user-attachments/assets/cadad734-1d05-420f-9829-508f1a9b7855)

Giá trị trả về là `1` xác nhận cổng 80 đang mở và dịch vụ Apache đang hoạt động:

![Giá trị Item = 1, cổng 80 đang mở](https://github.com/user-attachments/assets/2f3a4bb0-7680-47b3-830b-1af98d100d05)

---

## 5. Đánh Giá Trạng Thái và Ghi Nhận Sự Cố (Triggers & Events)

### Tổng Quan

**Trigger** là điều kiện logic được định nghĩa dựa trên dữ liệu của một hoặc nhiều Items. Khi điều kiện được thỏa mãn (ví dụ: giá trị Item = 0, tức dịch vụ ngừng hoạt động), Trigger chuyển sang trạng thái **PROBLEM** và tạo ra một **Event**. Event có thể kích hoạt thông báo (email, Telegram), ghi log, hoặc chạy script tự động xử lý. Đây là cơ chế cảnh báo chủ động cốt lõi của Zabbix.

---

### 5.1 Tạo Trigger

1. Vào **Data collection → Hosts** → Cột **Triggers** của host `Zabbix Client` → Click **Create trigger**.
2. Điền thông tin:
   - **Name:** `Apache is DOWN on client`
   - **Severity:** `High`
3. Mục **Expression** → Click **Add**:
   - Chọn item `Apache Service Port 80`
   - **Function:** `last()`
   - **Operator:** `=`
   - **Value:** `0`
   - Expression được tạo ra: `last(/Zabbix_Client/net.tcp.port[,80])=0`

> **Giải thích expression:** `last()` lấy giá trị mới nhất của Item. Khi giá trị đó bằng `0` (port đóng), Trigger kích hoạt và chuyển sang trạng thái **PROBLEM**.

![Tạo Trigger cảnh báo Apache DOWN](https://github.com/user-attachments/assets/2329d444-8932-49f0-bf2b-8a8609f12f06)

4. Click **Add** (trong dialog expression) → Click **Add** để lưu trigger.

**Kiểm tra kết quả:** Dừng dịch vụ Apache trên máy Ubuntu:

```bash
sudo systemctl stop apache2
```

Sau 30–60 giây, vào **Monitoring → Problems** → Trigger `Apache is DOWN on Node 1` xuất hiện với trạng thái **PROBLEM**, kèm timestamp chính xác:

![Trigger kích hoạt, sự cố Apache DOWN](https://github.com/user-attachments/assets/fdad8d33-078f-4c55-af7a-ac8b9becfb00)

---

## 6. Trực Quan Hóa Dữ Liệu (Dashboards)

### Tổng Quan

**Dashboard** trong Zabbix là bảng điều khiển tổng hợp, cho phép hiển thị dữ liệu giám sát dưới nhiều dạng widget khác nhau (biểu đồ, giá trị tức thời, bản đồ mạng, danh sách sự cố, v.v.) trên một màn hình duy nhất. Dashboard hỗ trợ cập nhật theo thời gian thực, giúp người quản trị nắm bắt nhanh toàn cảnh hệ thống mà không cần điều hướng qua nhiều menu.

---

### 6.1 Tạo Dashboard Mới

1. Vào **Monitoring → Dashboards** → Click **Create dashboard**.
2. Đặt tên: `Web Server Monitoring`.
3. Click **Apply**.

---

### 6.2 Thêm Widget Giá Trị Thời Gian Thực (Item Value)

1. Click **Add widget**.
2. Chọn type: **Item value**.
3. Điền:
   - **Name:** `Active Requests`
   - **Item:** chọn item request count của Apache.
4. Click **Apply**.
5. Click **Save changes** để lưu dashboard.

Sau khi lưu, biểu đồ và giá trị cập nhật theo thời gian thực:

![Dashboard Web Server Monitoring](https://github.com/user-attachments/assets/497d3985-5c16-43fd-886b-80612de2a8fb)

![Widget Item Value trên Dashboard](https://github.com/user-attachments/assets/b1f5c601-ad08-4f6f-a338-2c31546e8db7)

---

## 7. Phân Quyền Hệ Thống (Users & User Groups)

### Tổng Quan

Zabbix hỗ trợ hệ thống phân quyền chi tiết theo mô hình **User Group → Host Group**. Người quản trị tạo các User Group, gán quyền truy cập (Read / Read-write / Deny) vào từng Host Group cụ thể, sau đó đưa người dùng vào User Group tương ứng. Nhờ đó, mỗi nhân viên chỉ thấy và can thiệp được vào phạm vi host mà họ được phép, đảm bảo bảo mật và phân tách trách nhiệm rõ ràng.

---

### 7.1 Tạo User Group

1. Vào **Users → User groups** → Click **Create user group**.
2. Tab **User group**:
   - **Group name:** `IT Support Team`
3. Tab **Host group permissions** → Click **Add**:
   - **Host groups:** `Web Servers HA`
   - **Permissions:** `Read-write`
4. Click **Add** để lưu.

![Tạo User Group IT Support Team](https://github.com/user-attachments/assets/d29dd78d-e277-4744-b957-b76ab31c18fd)

---

### 7.2 Tạo User Mới

1. Vào **Users → Users** → Click **Create user**.
2. Tab **User**:
   - **Username:** `it_support_01`
   - **Name:** (tên nhân viên thực tế)
   - **Groups:** chọn `IT Support Team`
   - Đặt **Password** cho tài khoản.
3. Tab **Media** → Click **Add**:
   - **Type:** `Email` hoặc `Telegram`
   - **Send to:** địa chỉ email hoặc Chat ID Telegram của nhân viên.
   - Cấu hình thời gian nhận thông báo nếu cần (theo ca trực).
4. Tab **Permissions** → kiểm tra role: `User`.
5. Click **Add** để lưu.

**Kiểm tra kết quả:** Đăng nhập bằng tài khoản `it_support_01` → chỉ thấy các host thuộc nhóm `Web Servers HA`, không thấy host của nhóm khác.

![Cấu hình User mới - Tab User](https://github.com/user-attachments/assets/753f6106-2da2-46c3-83c9-29b83884dfee)

![Cấu hình User mới - Tab Media](https://github.com/user-attachments/assets/a8b45aee-2504-4bcd-bcad-9ff73c41f8fe)

---

## 8. Báo Cáo Thực Tập Ngày 25/6

### Tổng Quan

**Scheduled Reports** cho phép Zabbix tự động xuất Dashboard thành file PDF và gửi qua email cho danh sách người đăng ký theo lịch định sẵn (hàng ngày, hàng tuần, hàng tháng). Tính năng này phù hợp cho việc báo cáo hiệu năng định kỳ lên quản lý hoặc lưu trữ hồ sơ vận hành mà không cần người quản trị can thiệp thủ công.

> **Yêu cầu tiên quyết:** Chức năng Scheduled Reports cần dịch vụ **Zabbix Web Service** (`zabbix-web-service`) được cài đặt và cấu hình trên Zabbix Server, đồng thời phải cấu hình **Media Type Email** với thông tin SMTP hợp lệ.

---

### 8.1 Tạo Scheduled Report

1. Vào **Reports → Scheduled reports** → Click **Create report**.
2. Điền thông tin:

| Trường | Giá trị |
|---|---|
| **Name** | `Weekly Web Server Performance` |
| **Dashboard** | `Web Server Monitoring` (đã tạo ở Mục 6) |
| **Period** | `Week` |
| **Start time** | `08:00` |
| **Repeat on** | `Monday` |

3. Mục **Subscriptions** → Click **Add**:
   - **User:** chọn các user cần nhận báo cáo (admin, manager, v.v.).
   - **Create like:** `Me` (xuất đúng định dạng PDF theo tài khoản hiện tại).
4. Click **Add** để lưu.

**Kiểm tra kết quả:** Đợi đến lịch đã cấu hình, hoặc sử dụng nút **Test** (nếu có) để gửi thử → kiểm tra email nhận được file PDF chứa ảnh chụp Dashboard.

![Cấu hình Scheduled Report hàng tuần](https://github.com/user-attachments/assets/ed3bcd64-856f-454d-ae7e-4aab3b07764a)

---

