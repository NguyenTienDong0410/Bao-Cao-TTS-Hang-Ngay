


---

# BÁO CÁO THỰC TẬP: TRIỂN KHAI CLOUDFLARE VÀ HỆ THỐNG QUẢN TRỊ MÁY CHỦ (CPANEL/WHM)

## 1. Tổng quan về hệ thống Cloudflare

### 1.1. Khái niệm cơ bản

Cloudflare là một dịch vụ cung cấp mạng lưới phân phối nội dung (CDN - Content Delivery Network), quản lý phân giải tên miền (DNS) và các giải pháp bảo mật internet toàn cầu. Về mặt kiến trúc hệ thống, Cloudflare hoạt động như một Reverse Proxy (máy chủ proxy ngược) đứng trung gian giữa người dùng cuối (Client) và máy chủ gốc (Origin Server - VPS/Hosting).

Thay vì người dùng kết nối trực tiếp đến địa chỉ IP của máy chủ gốc, mọi truy vấn dữ liệu sẽ được định tuyến đi qua hệ thống máy chủ của Cloudflare.

### 1.2. Các chức năng và vai trò cốt lõi

Trong mô hình triển khai ứng dụng web thực tế, việc tích hợp Cloudflare giải quyết các bài toán quan trọng về hiệu năng và bảo mật hệ thống:

* **Mạng phân phối nội dung (CDN) và Bộ nhớ đệm (Caching):** Cloudflare sở hữu mạng lưới trung tâm dữ liệu toàn cầu. Hệ thống tự động lưu trữ các tệp tin tĩnh (hình ảnh, CSS, JavaScript) tại các điểm máy chủ rìa (Edge Server). Khi có luồng truy cập, nội dung được trả về từ máy chủ gần với người dùng nhất, giúp giảm đáng kể độ trễ (latency) và giảm tải băng thông cho máy chủ gốc.
* **Bảo mật và chống tấn công DDoS:** Bằng cách ẩn địa chỉ IP thực của máy chủ gốc, hệ thống ngăn chặn các nỗ lực rà quét lỗ hổng trực tiếp. Đồng thời, Cloudflare tích hợp Tường lửa ứng dụng web (WAF) giúp lọc lưu lượng truy cập độc hại, ngăn chặn botnet và giảm thiểu tác động của các cuộc tấn công từ chối dịch vụ phân tán (DDoS).
* **Quản lý phân giải tên miền (DNS):** Hệ thống DNS của Cloudflare được đánh giá là một trong những hệ thống có tốc độ phân giải nhanh nhất thế giới. Việc thay đổi bản ghi DNS được cập nhật gần như ngay lập tức, khắc phục độ trễ của các hệ thống DNS truyền thống.
* **Mã hóa dữ liệu (SSL/TLS):** Hệ thống cung cấp chứng chỉ SSL linh hoạt, giúp toàn bộ dữ liệu trao đổi giữa người dùng và máy chủ được mã hóa (HTTPS) tự động mà không đòi hỏi các bước cấu hình phức tạp trên máy chủ vật lý.

### 1.3. Lợi ích khi triển khai hệ thống

Việc triển khai website kết hợp với các nền tảng quản trị hosting (như DirectAdmin hoặc cPanel) và Cloudflare mang lại mô hình quản lý tối ưu. Hệ thống quản trị đảm nhiệm vai trò quản lý tài nguyên và môi trường thực thi cục bộ (Local), trong khi Cloudflare đảm nhiệm vai trò tối ưu hóa đường truyền, định tuyến lưu lượng và bảo vệ vành đai an ninh mạng (Perimeter Security) cho toàn bộ ứng dụng.

### 1.4. Hướng dẫn cấu hình tích hợp Cloudflare (Minh họa với DirectAdmin)

**Khởi tạo cấu hình trên Control Panel (DirectAdmin):**
Tiến hành tạo tên miền trên hệ thống:

<img width="1206" height="298" alt="image" src="https://github.com/user-attachments/assets/22761d51-e7fd-4f9b-93b4-c44fb77b59e6" />

Thêm mã nguồn (Source) vào thư mục web:
<img width="1877" height="316" alt="image" src="https://github.com/user-attachments/assets/55736c90-1f60-4d51-8cf7-fe73a72517dc" />

Thiết lập các bản ghi DNS cơ bản (A, CNAME) trên máy chủ gốc:

<img width="1540" height="690" alt="image" src="https://github.com/user-attachments/assets/bc4eb001-2ff7-4fce-a43c-15f0a062ec27" />


**Quy trình trỏ tên miền về Cloudflare:**

* **Bước 1: Thêm Website vào Cloudflare**
* Đăng nhập vào hệ thống Cloudflare, chọn **Add a Site**.
* Nhập tên miền cần quản lý (Ví dụ: `tdong41.id.vn`).
* Chọn gói dịch vụ **Free**.
* Cloudflare sẽ tiến hành quét các bản ghi hiện có. Quản trị viên tiếp tục chuyển sang bước kế tiếp.


* **Bước 2: Cập nhật Nameservers (Bước quyết định)**
* Hệ thống Cloudflare sẽ cung cấp 2 địa chỉ Nameserver (Ví dụ: `abby.ns.cloudflare.com` và `ben.ns.cloudflare.com`).
* Đăng nhập vào trang quản lý dịch vụ tên miền (tại nhà cung cấp tên miền).
* Truy cập mục **Thay đổi Nameservers** (hoặc Máy chủ tên miền).
* Thay thế các Nameserver cũ bằng 2 Nameserver do Cloudflare cung cấp và lưu cấu hình. (Lưu ý: Quá trình phân giải DNS toàn cầu có thể mất từ 5 đến 30 phút).


* **Bước 3: Cấu hình bản ghi DNS trên Cloudflare**
* Trong giao diện Cloudflare, điều hướng tới **DNS > Records**. Cần đảm bảo các bản ghi sau được trỏ chính xác về IP của VPS:



| Type | Name (Host) | Content (IP Address) | Proxy status |
| --- | --- | --- | --- |
| A | `@` | `Địa-chỉ-IP-VPS-của-bạn` | Proxied (Đám mây cam) |
| CNAME | `www` | `tdong41.id.vn` | Proxied (Đám mây cam) |

*Ghi chú: Địa chỉ IP VPS có thể được lấy từ email bàn giao dịch vụ hoặc từ trang quản trị của nhà cung cấp VPS.*

* **Bước 4: Cấu hình SSL/TLS**
* Để tránh cảnh báo "Không bảo mật" trên trình duyệt, truy cập mục **SSL/TLS > Overview** trên Cloudflare:
* Trường hợp VPS chưa cài đặt SSL: Chọn chế độ **Flexible**.
* Trường hợp VPS đã cài đặt chứng chỉ SSL (ví dụ: Let's Encrypt): Chọn chế độ **Full** hoặc **Full (Strict)**.


Giao diện cấu hình tự động TLS/SSL:

<img width="1148" height="858" alt="image" src="https://github.com/user-attachments/assets/07120340-7987-4358-8112-a136ba6a431d" />

Trang web đã được triển khai và hoạt động hoàn thiện:

<img width="1920" height="980" alt="image" src="https://github.com/user-attachments/assets/1c3eb398-a2dd-4647-80cd-6c269d1603dd" />


**Kiểm tra tính năng bảo mật:**

Thông qua công cụ kiểm tra bản ghi web:

<img width="1335" height="948" alt="image" src="https://github.com/user-attachments/assets/1e1ef281-dfe8-4c1c-b803-0949953b268f" />

Kết quả cho thấy địa chỉ IP gốc của VPS đã được ẩn hoàn toàn bởi dải IP của Cloudflare, đảm bảo an toàn cho máy chủ vật lý:

<img width="1153" height="862" alt="image" src="https://github.com/user-attachments/assets/ba795b7e-3df4-4ed0-a076-83da0e269ca6" />

---

## 2. Triển khai Hệ thống cPanel/WHM trên Ubuntu

### 2.1. Yêu cầu hệ thống tối thiểu

Để đảm bảo quá trình cài đặt và vận hành cPanel diễn ra ổn định, máy chủ Ubuntu cần đáp ứng các thông số cấu hình cơ bản sau:

* **Hệ điều hành:** Ubuntu 20.04 LTS (Phiên bản được cPanel hỗ trợ chính thức).
* **Địa chỉ IP:** Yêu cầu một địa chỉ IPv4 tĩnh, công cộng.
* **Tên miền đầy đủ (FQDN):** Máy chủ cần được định danh bằng một FQDN hợp lệ (Ví dụ: `server.yourdomain.com`). FQDN là yếu tố bắt buộc để các dịch vụ như Email, FTP và chứng chỉ SSL/TLS hoạt động chính xác.
* **CPU:** Tối thiểu 2GHz.
* **RAM:** Tối thiểu 4GB.
* **Dung lượng đĩa (Storage):** Tối thiểu 20GB khả dụng.
* **Kiến trúc hệ thống (Architecture):** 64-bit.

### 2.2. Chuẩn bị máy chủ trước khi cài đặt

Trước khi thực thi kịch bản (script) cài đặt, quản trị viên cần thực hiện chuẩn bị môi trường:

**Đăng nhập vào máy chủ bằng quyền Root:**

```bash
ssh root@your_server_ip

```

**Cập nhật hệ thống:**
Đây là bước bắt buộc để vá các lỗ hổng bảo mật và cập nhật các thư viện phần mềm mới nhất, giảm thiểu xung đột trong quá trình cài đặt.

```bash
sudo apt update && sudo apt upgrade -y

```

**Thiết lập Hostname (FQDN):**
Kiểm tra hostname hiện tại bằng lệnh `hostname -f`. Nếu chưa đúng định dạng FQDN, tiến hành thiết lập lại (thay `whm.test.com` bằng FQDN thực tế):

```bash
hostnamectl set-hostname whm.test.com

```

**Quản lý Firewall:**
Firewall mặc định của Ubuntu là UFW (Uncomplicated Firewall) có thể chặn các cổng giao tiếp mà cPanel cần để tải gói tin. Quản trị viên cần tạm thời vô hiệu hóa UFW. (cPanel sẽ tự động triển khai hệ thống tường lửa riêng sau khi hoàn tất).

```bash
sudo ufw disable

```

**Sử dụng công cụ Screen (Khuyến nghị):**
Quá trình biên dịch và cài đặt cPanel tiêu tốn khá nhiều thời gian. Lệnh `screen` tạo ra một phiên làm việc độc lập trên terminal, giúp tiến trình không bị ngắt quãng nếu kết nối mạng từ máy quản trị bị gián đoạn.

```bash
sudo apt install screen -y
screen

```

### 2.3. Quy trình cài đặt cPanel/WHM

Sau khi chuẩn bị xong, tiến hành cài đặt theo các bước sau:

**Chuyển đến thư mục gốc:** Khuyến nghị lưu trữ và chạy script từ thư mục `/home`.

```bash
cd /home

```

**Tải kịch bản cài đặt:** Sử dụng `curl` để tải script mới nhất từ máy chủ cPanel.

```bash
curl -o latest -L https://securedownloads.cpanel.net/latest

```

<img width="1105" height="222" alt="image" src="https://github.com/user-attachments/assets/89fb889d-7a5e-4a4b-b453-01e74a4c9054" />
**Thực thi kịch bản:**

```bash
sh latest

```
<img width="1523" height="699" alt="image" src="https://github.com/user-attachments/assets/ec48e3a5-7877-459a-be2d-eebb8226fa60" />

**Theo dõi tiến trình:** Hệ thống sẽ tự động kiểm tra tương thích, tải xuống và biên dịch hàng trăm gói phần mềm. Quá trình này có thể kéo dài từ 30 đến 60 phút tùy thuộc vào hiệu năng phần cứng và băng thông mạng. *Tuyệt đối không can thiệp hoặc ngắt tiến trình trong giai đoạn này.*

---

## 3. Cấu hình và Vận hành cPanel/WHM

### 3.1. Truy cập và thiết lập ban đầu

Sau khi terminal báo cài đặt hoàn tất, quản trị viên có thể truy cập giao diện web (WHM - Web Host Manager) để cấu hình bước đầu.

* **Truy cập WHM:** Sử dụng trình duyệt truy cập địa chỉ `https://your_server_ip:2087`. Bỏ qua cảnh báo bảo mật ban đầu (do SSL tự ký).
* **Đăng nhập:** * Username: `root`
* Password: Mật khẩu root của máy chủ.

<img width="1483" height="791" alt="image" src="https://github.com/user-attachments/assets/81a4d251-88c7-4b96-9d2f-6848a84c9e39" />

<img width="943" height="615" alt="image" src="https://github.com/user-attachments/assets/884d6159-1672-44d8-9c0e-40de4ac23a39" />
* **Hoàn thiện thông tin:**
* Đồng ý với các điều khoản dịch vụ (TOS).

<img width="1920" height="903" alt="image" src="https://github.com/user-attachments/assets/694c14a8-2a32-4262-a540-55914922fb0b" />
* Khai báo email quản trị viên và thiết lập Nameservers mặc định (ví dụ: `ns1.yourdomain.com` và `ns2.yourdomain.com`).
* Truy cập vào trang chủ tổng quan của WHM.

<img width="1920" height="973" alt="image" src="https://github.com/user-attachments/assets/613d7f5a-9bc6-414e-a4fd-103eac0cd6c0" />

*Ghi chú: Đối với người dùng cuối, sau khi được cấp phát tài khoản, họ có thể quản lý dịch vụ thông qua cPanel tại cổng 2083 (`https://your_domain.com:2083`).*

### 3.2. Cấu trúc chức năng cốt lõi trên WHM

Hệ thống WHM phân cấp quản trị thành nhiều phân hệ chuyên biệt:

* **Cấu hình hệ thống & Mạng (Hạ tầng cốt lõi):**
* **Server Configuration:** Quản lý các thiết lập nền tảng, tinh chỉnh giới hạn tài nguyên cấp phát cho từng tiến trình, quản lý bộ nhớ và đồng bộ thời gian.
<img width="1779" height="846" alt="image" src="https://github.com/user-attachments/assets/7096faec-b9da-4af8-8e5f-5a8f4e6ef307" />


* **Networking Setup:** Thay đổi Hostname, quản lý danh sách IP khả dụng và định cấu hình Nameservers mặc định.

<img width="865" height="615" alt="image" src="https://github.com/user-attachments/assets/2175d65a-f59b-437a-929d-ce907d6a8133" />
* **Clusters:** Thiết lập liên kết đa máy chủ, phục vụ cho mục đích High Availability (HA) hoặc tách biệt máy chủ DNS chuyên dụng.


* **Quản lý Dịch vụ & Bảo mật:**
* **Service Configuration:** Quản lý sâu các daemon như Web Server (Apache/Nginx), FTP Server, Mail Server (Exim), và cấu hình tham số PHP lõi.
* **Security Center:** Trung tâm kiểm soát an ninh, cung cấp tính năng khóa IP tấn công, quản lý giao thức SSH và vòng đời của chứng chỉ SSL/TLS.
* **Backup:** Lên lịch sao lưu toàn diện. Cho phép xuất bản sao lưu ra nền tảng lưu trữ ngoại vi như Amazon S3, Google Drive, hoặc FTP Server theo nguyên tắc 3-2-1.


* **Giám sát & Vận hành:**
* **Server Status:** Cung cấp thông số thời gian thực (Real-time) về mức độ tiêu thụ RAM, CPU, I/O ổ cứng và trạng thái hoạt động của từng dịch vụ đơn lẻ.
* **System Reboot:** Cung cấp phương thức khởi động lại an toàn (Graceful reboot) hoặc ép buộc (Forceful reboot).
* **Server Contacts:** Thiết lập cơ chế cảnh báo tự động (Alert) qua email khi tài nguyên chạm ngưỡng tới hạn.


* **Cấp phát tài khoản (Dành cho mô hình Đại lý):**
* **Resellers:** Cấp quyền cho người dùng đóng vai trò Đại lý, cho phép họ chia nhỏ không gian lưu trữ và tự quản lý danh sách khách hàng riêng.

<img width="748" height="795" alt="image" src="https://github.com/user-attachments/assets/a38c3094-1599-4796-8d42-6b127e396df7" />

### 3.3. Quy trình khởi tạo tài khoản cPanel cho người dùng

**Giai đoạn 1: Khởi tạo Gói tài nguyên (Packages)**
Quản trị viên bắt buộc phải thiết lập Gói tài nguyên trước khi cấp phát tài khoản nhằm ngăn chặn rủi ro một website tiêu thụ cạn kiệt tài nguyên của VPS.

* Tại thanh tìm kiếm WHM, chọn **Add a Package**.
* Thiết lập các giới hạn:
* **Package Name:** Tên định danh gói (Ví dụ: `Goi_Co_Ban`).
* **Disk Quota (MB):** Giới hạn lưu trữ (Ví dụ: 2048 cho 2GB).
* **Monthly Bandwidth (MB):** Băng thông cho phép mỗi tháng.
* **Giới hạn số lượng:** FTP Accounts, Email Accounts, và Databases.
* 
<img width="776" height="782" alt="image" src="https://github.com/user-attachments/assets/daf21197-3d68-456b-b424-3d17e181b0f3" />

* Nhấn **Add** để hoàn tất.

**Giai đoạn 2: Tạo tài khoản cPanel mới (Create a New Account)**

* Tại mục *Account Functions*, chọn **Create a New Account**.
* Khai báo **Domain Information**:
* **Domain:** Tên miền chính (Ví dụ: `tenmien.com`).
* **Username:** Tên đăng nhập (Tối đa 16 ký tự, không dấu).
* **Password & Email:** Mật khẩu và địa chỉ nhận cảnh báo tài nguyên.



* Tại mục **Package**: Chọn gói tài nguyên (ví dụ `Goi_Co_Ban`) đã tạo ở Giai đoạn 1.
* Tại mục **DNS/Mail Routing**: Giữ mặc định (Local Mail Exchanger) nếu dùng mail cục bộ.

* Nhấn **Create** để tiến hành cấp phát.
<img width="1620" height="196" alt="image" src="https://github.com/user-attachments/assets/0ed287c3-34db-402f-a357-4b32675b2f5e" />

Người dùng sau khi nhận thông tin có thể đăng nhập vào cPanel (Port 2083) để quản trị các lớp dịch vụ như File Manager, Database, và Email riêng biệt:


trang quản trị người dùng Cpanel tại port 2083, gồm các chức năng như 
<img width="1920" height="857" alt="image" src="https://github.com/user-attachments/assets/7e6bb4d4-b1c0-4c27-aba5-cb1f1cf920aa" />

Một số tool
<img width="953" height="730" alt="image" src="https://github.com/user-attachments/assets/22087602-e5ae-4400-b1ae-18af91ca46df" />

<img width="1259" height="837" alt="image" src="https://github.com/user-attachments/assets/2d730a98-9a7a-482f-a694-1451a267459a" />

<img width="1159" height="822" alt="image" src="https://github.com/user-attachments/assets/58c04429-5887-48be-a8ef-a8ef4a69fa26" />

### 3.4. Các công cụ quản trị nâng cao

Một số công cụ hữu ích tích hợp trong hệ thống:

* **EasyApache 4 (Tinh chỉnh Web Server & PHP):**
* Công cụ này cho phép biên dịch và cấu hình môi trường web thông qua giao diện trực quan thay vì gõ lệnh trong CLI. Quản trị viên có thể triển khai hệ thống Nginx làm Reverse Proxy cho Apache, quản lý song song nhiều phiên bản PHP (MultiPHP), và bật/tắt các module (extensions) như `mbstring`, `zip` một cách nhanh chóng bằng thao tác *Provision*.

<img width="1115" height="777" alt="image" src="https://github.com/user-attachments/assets/d971a781-f40a-4192-86e5-ecc7055eff3c" />

* **DNS Zone Manager (Quản lý bản ghi máy chủ cục bộ):**
* Đối với các tên miền không sử dụng Proxy của Cloudflare mà trỏ thẳng IP về máy chủ, quản trị viên sử dụng công cụ này (thuộc mục *DNS Functions*) để toàn quyền can thiệp vào các bản ghi A, CNAME, MX, TXT của hệ thống cục bộ.

<img width="1408" height="353" alt="image" src="https://github.com/user-attachments/assets/c453cc61-68e3-40e7-9640-73df6c5792d3" />


* **Hệ thống phòng chống xâm nhập (Security):**
* **cPHulk Brute Force Protection:** Giám sát các lượt xác thực thất bại tại cPanel, SSH, FTP, hoặc Webmail. Hệ thống tự động thiết lập lệnh cấm (Block) đối với địa chỉ IP có hành vi dò quét mật khẩu (vượt quá số lần cho phép), ngăn ngừa các cuộc tấn công Brute Force.

* **ConfigServer Security & Firewall (CSF):** Là một Plugin chuyên dụng thường được cài đặt bổ sung trên WHM. CSF cung cấp giao diện quản trị tường lửa (Iptables) mạnh mẽ, giúp dễ dàng kiểm soát Port (TCP/UDP), phân quyền truy cập theo địa lý và giám sát các tiến trình bất thường.
<img width="820" height="790" alt="image" src="https://github.com/user-attachments/assets/5186014f-dcaa-4f44-bbfc-6cc91c77d8cb" />

* **Backup Configuration (Cấu hình sao lưu định kỳ):**
* Quản trị viên cấu hình tác vụ sao lưu hoàn toàn tự động, phân tách các thiết lập: Lên lịch (*Scheduling* - chọn giờ thấp điểm), Thời gian lưu trữ (*Retention* - số lượng bản sao giữ lại để tối ưu hóa không gian), và Vị trí lưu trữ (*Destination* - đẩy dữ liệu sang máy chủ độc lập). Việc duy trì bản sao lưu từ xa là tiêu chuẩn bắt buộc nhằm bảo vệ toàn vẹn dữ liệu trong trường hợp máy chủ gặp thảm họa phần cứng.



<img width="1075" height="791" alt="image" src="https://github.com/user-attachments/assets/85331d93-4c6c-4e79-af2a-ae6376916317" />

