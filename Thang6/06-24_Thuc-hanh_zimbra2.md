## Báo cáo thực tập 24/6 


Quản trị User, Webmail Gửi nhận nội bộ

### Bước 1: Truy cập Admin Console

Giao diện quản trị của Zimbra (Admin Console) được tách biệt hoàn toàn với giao diện Webmail của người dùng.

1. Mở trình duyệt web và truy cập theo đường dẫn:
* `https://<IP_Server_Zimbra>:7071`


<img width="1912" height="777" alt="image" src="https://github.com/user-attachments/assets/597a9082-6be6-4780-8bb1-d7cd15565563" />


2. **Lưu ý:** Trình duyệt sẽ cảnh báo bảo mật (Your connection is not private) do Zimbra mặc định sử dụng chứng chỉ SSL tự ký (Self-signed certificate). Bạn cứ chọn **Advanced -> Proceed** để tiếp tục.
3. Đăng nhập bằng tài khoản quản trị ( `admin@mail.domain.com`) và mật khẩu đã thiết lập lúc cài đặt.

---

### Bước 2: Tạo Class of Service (CoS) và Account

**Class of Service (CoS)** là một tính năng rất mạnh trong Zimbra giúp bạn gom nhóm cấu hình (Quota, Feature, Theme...) và áp dụng cho hàng loạt user thay vì cấu hình tay cho từng người.

**1. Tạo CoS phân bổ Quota:**

* Trên thanh menu bên trái, điều hướng tới **Configure** -> **Class of Service**.
* Nhấn vào biểu tượng hình bánh răng (góc trên bên phải bảng) -> Chọn **New**.
* 
* **Tạo CoS cho Staff:**
* *General Information:* Đặt tên là `Staff_CoS`.
  
<img width="1415" height="578" alt="image" src="https://github.com/user-attachments/assets/4ed73c74-7325-4a86-bae1-74eaf9881b73" />

* *Advanced:* Tìm đến mục **Account quota**, nhập `5120` MB (tương đương 5GB).

<img width="1426" height="653" alt="image" src="https://github.com/user-attachments/assets/7efbda96-3b7d-48ae-8655-9571ba7462cf" />

* Nhấn **Save**.


* **Tạo CoS cho Manager:**
* Làm tương tự bước trên, đặt tên là `Manager_CoS`.
* *Advanced:* Cấu hình Account quota là `10240` MB (tương đương 10GB).
* Nhấn **Save**.
<img width="1392" height="594" alt="image" src="https://github.com/user-attachments/assets/09398cb4-d0a6-45c0-b5a4-9795b283bc33" />

<img width="1171" height="358" alt="image" src="https://github.com/user-attachments/assets/22e3f4bc-bd48-400f-adb7-4aba89d46f47" />


**2. Tạo Account (User):**

* Điều hướng tới **Manage** -> **Accounts**.
* Nhấn nút bánh răng -> **New**.
* Tạo tài khoản Staff (VD: `nhanvien@domain.com`):
* Điền các thông tin cơ bản: Account Name, Last Name, Password.
  
  <img width="1916" height="827" alt="image" src="https://github.com/user-attachments/assets/de68d39a-4e5c-433f-aab8-7d0717ee23b5" />

* Kéo xuống phần **Class of Service**, bỏ check "default" và chọn `Staff_CoS`.
<img width="1614" height="171" alt="image" src="https://github.com/user-attachments/assets/faa672d7-d73c-43da-93c3-782da6a2753f" />


Thêm sửa xóa tài khoản: 

<img width="1916" height="378" alt="image" src="https://github.com/user-attachments/assets/bf8e4b2f-9687-4178-a57e-5c94a17c72e0" />


* Tạo tài khoản Manager (VD: `quanly@domain.com`):
* Điền thông tin và chọn Class of Service là `Manager_CoS`.
  
<img width="1532" height="495" alt="image" src="https://github.com/user-attachments/assets/1f558bb3-7b49-41d5-9df7-2d0b48862bf9" />


* Nhấn **Save** cho mỗi tài khoản.

---

### Bước 3: Thực hành Webmail và Gửi/Nhận nội bộ

1. Mở một tab ẩn danh (Incognito) để truy cập Webmail của người dùng tại đường dẫn: `https://192.168.197.148` (sử dụng port 443 mặc định).

<img width="1920" height="972" alt="image" src="https://github.com/user-attachments/assets/52d95409-5739-4d80-9978-cceb6512085e" />

3. Đăng nhập bằng tài khoản `a@domain.com` là tài khoản nhân viên.
   <img width="1920" height="804" alt="image" src="https://github.com/user-attachments/assets/a2c899be-c3ec-47ee-9721-bca2b7947fdd" />

5. Soạn một email mới gửi đến `b@domain.com` là tài khoản b để test luông mail nội bộ

    <img width="1918" height="436" alt="image" src="https://github.com/user-attachments/assets/5f92a33f-ce4d-4294-835f-e56bd15d12c7" />

8. Thực hiện Reply lại để xác nhận chiều ngược lại hoạt động ổn định.

<img width="1920" height="608" alt="image" src="https://github.com/user-attachments/assets/6e8bd668-249b-4d7d-b899-0027c04e63fa" />

<img width="1920" height="563" alt="image" src="https://github.com/user-attachments/assets/1cf51149-d817-463f-b6ea-e44be494d748" />


Đã nhận được nội dung 

 có thể di chuột vào tên tài khoản ở góc trên bên phải màn hình Webmail để xem thanh hiển thị Quota, xác nhận xem Staff có đúng nhận 5GB và Manager nhận 10GB như đã cấu hình trong CoS hay không).*
 
 <img width="664" height="199" alt="image" src="https://github.com/user-attachments/assets/0eb501e9-9237-448d-9f62-fc3363aae658" />

<img width="632" height="228" alt="image" src="https://github.com/user-attachments/assets/268e676e-4b0e-46c0-963b-c030ad32847f" />


Thiết lập chữ ký email trên Zimbra
2.1. Tạo chữ ký
Tại giao diện webmail, thực hiện theo trình tự:

Click Tùy chọn (Preferences)
Click Chữ ký (Signatures)
Nhập tên cho chữ ký
Chọn định dạng: Văn bản thường (As Plaintext) hoặc HTML (As Html)

Nhập nội dung của chữ ký
Chọn vị trí hiển thị chữ ký
Click Lưu (Save)

<img width="1282" height="506" alt="image" src="https://github.com/user-attachments/assets/80991d50-3185-4f66-8f7e-ae5534778624" />

<img width="1052" height="329" alt="image" src="https://github.com/user-attachments/assets/4bdd1c6f-429b-4e9e-8bc6-aa30c0f7d526" />

Giao diện tạo chữ ký Zimbra

Lưu ý: Khi soạn email mới, để chữ ký hiển thị đúng định dạng, nên chọn định dạng văn bản tương ứng với định dạng đã thiết lập cho chữ ký.

<img width="1920" height="590" alt="image" src="https://github.com/user-attachments/assets/9b09f21a-6d39-4418-9749-f17817ab5711" />

Tại giao diện quản lý chữ ký: click chọn chữ ký muốn xóa → click nút Delete.


3. Thiết lập forward (chuyển tiếp) email trên Zimbra
Khi tạo một tài khoản email Zimbra mới, mặc định hệ thống đã giới hạn quyền người dùng có thể tự cấu hình chuyển tiếp email. Để thiết lập forward email, thực hiện như sau:

Bước 1: Đăng nhập vào webmail
Đăng nhập webmail Zimbra

Bước 2: Thiết lập forward
Vào Tùy chọn → Thư → Nhận thư, tìm mục cài đặt chuyển tiếp.

Nhập địa chỉ email muốn chuyển tiếp tới và nhấn Lưu.

<img width="1298" height="606" alt="image" src="https://github.com/user-attachments/assets/154203bb-7114-4ef7-8f53-6ea6f4b6f4f2" />

Bước 3: Kiểm tra
Gửi thử một email đến tài khoản vừa cấu hình để xác nhận chuyển tiếp hoạt động đúng.

<img width="1905" height="523" alt="image" src="https://github.com/user-attachments/assets/2c44a519-54f0-43aa-9204-347bb5611a8d" />

 mail từ a dc fowward tới mail admin 

<img width="1919" height="830" alt="image" src="https://github.com/user-attachments/assets/5136306f-c41b-4873-a83a-96e57756dd2e" />


2. Quản lý Mail Queue (Hàng đợi Mail)
Khi mạng chậm, server đích lỗi, hoặc mail bị kẹt, chúng sẽ nằm trong Queue của Postfix.

Chuyển sang user zimbra: su - zimbra

Xem danh sách mail đang kẹt trong hàng đợi:

Bash
mailq
(Hệ thống sẽ in ra danh sách các ID của mail, lý do kẹt và địa chỉ người nhận).

Ép hệ thống gửi lại toàn bộ mail trong Queue ngay lập tức (Flush):

Bash
/opt/zimbra/common/sbin/postqueue -f
Xóa một mail cụ thể khỏi Queue (Nếu đó là mail spam):

Bash
/opt/zimbra/common/sbin/postsuper -d <Queue_ID>
Xóa sạch toàn bộ Queue (Cẩn thận khi dùng):

Bash
/opt/zimbra/common/sbin/postsuper -d ALL

Không có mail nào được nằm trong queue
<img width="654" height="118" alt="image" src="https://github.com/user-attachments/assets/41e83613-f478-4219-a468-2a1e9dee6283" />


