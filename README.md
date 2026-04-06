# Báo Cáo Thực Tập - Ngày 06/04/2026

**Thực tập sinh:** Nguyễn Tiến Đông
[cite_start]**Đơn vị thực tập:** Công ty TNHH Phần mềm Nhân Hòa [cite: 3]

## 📖 Tổng quan
[cite_start]Báo cáo này tổng hợp lại quá trình tìm hiểu về cơ cấu tổ chức, sản phẩm của doanh nghiệp Nhân Hòa và các bài Lab thực hành làm quen với các công cụ quản trị máy chủ, công cụ vẽ sơ đồ hạ tầng mạng[cite: 1, 32, 141].

## 🏢 1. Tìm hiểu doanh nghiệp
* [cite_start]**Giới thiệu chung:** Công ty Nhân Hòa được thành lập từ năm 2002, hiện đang cung cấp dịch vụ cho hơn 50.000 khách hàng trên toàn quốc[cite: 3, 5].
* **Các sản phẩm dịch vụ cốt lõi:**
  * [cite_start]Dịch vụ Tên miền (Domain) quốc gia (.vn) và quốc tế (.com, .net,...)[cite: 9, 11, 12].
  * [cite_start]Dịch vụ Lưu trữ & Máy chủ: Web Hosting, Cloud VPS, Dedicated Server, và chỗ đặt máy chủ (Colocation)[cite: 13, 14, 15, 16, 17].
  * [cite_start]Giải pháp thiết kế Website trọn gói (Web4s)[cite: 18, 19].
  * [cite_start]Giải pháp giao tiếp: Email Doanh nghiệp (Umail), Tổng đài ảo (VFone), Hội nghị trực tuyến (Meetnow.vn)[cite: 22, 23, 24].
  * [cite_start]Giải pháp chuyển đổi số và bảo mật: Hóa đơn điện tử, Hợp đồng điện tử, Chứng chỉ SSL, CDN, Backup dữ liệu[cite: 26, 27, 29, 30, 31].

## 🛠️ 2. Thực hành công cụ quản trị Server
[cite_start]Phần thực hành trọng tâm là việc kết nối và cấu hình quản trị trên máy ảo **Windows Server 2022 (IP: 172.16.2.200)** và máy ảo **Kali Linux** thông qua Bridge network[cite: 35, 37, 123].

* [cite_start]**PuTTY:** Đã thực hành cấu hình cài đặt OpenSSH Server và mở Port 22 trên Windows Firewall bằng dòng lệnh PowerShell để thực hiện kết nối SSH thành công[cite: 34, 41, 45, 49].
* [cite_start]**MobaXterm:** Sử dụng công cụ All-in-one để quản lý kết nối đa phiên (Multi-tab) cho cả giao thức SSH và RDP[cite: 53, 59, 64]. [cite_start]Công cụ này có ưu điểm tích hợp sẵn thanh truyền tải file SFTP và khắc phục được lỗi gõ sai mật khẩu nhờ khung popup hiển thị dấu chấm[cite: 54, 55].
* [cite_start]**SecureCRT:** Làm quen với phần mềm chuyên dụng cho kỹ sư mạng (hỗ trợ Session Manager và Scripting) để kết nối dòng lệnh (SSH2) thông qua tính năng Quick Connect[cite: 79, 81, 83, 84, 85].
* [cite_start]**Remote Desktop Connection (mstsc):** Sử dụng công cụ mặc định tích hợp sẵn trên Windows để remote nhanh trực tiếp vào giao diện đồ họa của Windows Server[cite: 92, 93, 94].
* [cite_start]**Remote Desktop Manager (RDM):** Thiết lập và sử dụng phần mềm quản lý hạ tầng tập trung chuyên nghiệp[cite: 102, 103]. [cite_start]Đã thực hành tạo thư mục quản lý, gán Credentials lưu sẵn mật khẩu (Vault) và cấu hình kết nối RDP thành công vào cả máy ảo Windows Server và máy ảo Kali Linux (đã cài đặt thêm gói xrdp)[cite: 111, 114, 117, 123]. [cite_start]Tính năng này giúp quản lý và monitor hàng loạt Server cực kỳ khoa học[cite: 104, 107, 109].

## 🧰 3. Các công cụ hỗ trợ khác
* [cite_start]**Lightshot:** Áp dụng làm công cụ chụp ảnh màn hình để lưu vết cấu hình và log lỗi[cite: 126, 128]. [cite_start]Cung cấp tính năng chọn vùng linh hoạt, chú thích trực tiếp lên ảnh (vẽ khung, mũi tên, chèn text) và sao chép siêu tốc để làm tài liệu báo cáo[cite: 130, 132, 136, 137].
* [cite_start]**Visio và Draw.io:** Tìm hiểu công cụ thiết kế trực quan hóa hạ tầng mạng (Network Topology)[cite: 140, 141]. [cite_start]Đã áp dụng nền tảng Draw.io (diagrams.net) để thực hành vẽ sơ đồ hệ thống/lưu đồ cho "Hệ thống quản lý cửa hàng thời trang"[cite: 149, 154, 157]. [cite_start]Điểm mạnh của Draw.io là miễn phí, thao tác nhanh gọn trên nền tảng Web và tối ưu cho việc lưu trữ đám mây[cite: 150, 152, 155].
