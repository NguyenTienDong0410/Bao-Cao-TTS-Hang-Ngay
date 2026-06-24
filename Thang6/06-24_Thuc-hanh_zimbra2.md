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

4. Quản trị log
Khi theo dõi /var/log/zimbra.log, bạn không chỉ nhìn dòng chữ trôi qua mà phải lọc bằng grep. Postfix (MTA của Zimbra) định nghĩa 4 trạng thái cực kỳ quan trọng:

status=sent: Mail đã đi trót lọt.

status=bounced: Bị dội ngược lại (do sai địa chỉ, hòm thư đầy).

status=deferred: Bị hoãn (mạng chập chờn, server đích tạm sập), Postfix sẽ đưa vào Queue để thử lại sau.

status=rejected: Bị từ chối thẳng thừng (thường do dính Spam, sai SPF/DKIM).

log zimbra real time
tail -f /var/log/zimbra.log
<img width="1137" height="422" alt="image" src="https://github.com/user-attachments/assets/f3846646-cc14-4faf-8655-963d633545c2" />

Log đăng nhập và thao tác Mailbox: Dùng để xem ai đang login webmail, sai mật khẩu, hoặc tải file đính kèm.

Bash
tail -f /opt/zimbra/log/mailbox.log

<img width="1394" height="590" alt="image" src="https://github.com/user-attachments/assets/752d1256-10f5-4ae7-929c-0e79c3fce545" />

Thay vì tail -f toàn bộ, khi một user báo lỗi, bạn sẽ tìm đích danh mail của họ:

Bash
# Tìm tất cả log liên quan đến một địa chỉ email cụ thể
grep "a@mail.domain.com" /var/log/zimbra.log

<img width="1399" height="784" alt="image" src="https://github.com/user-attachments/assets/94d11fa6-490e-433d-b7f3-e040ab6f686b" />

# Tìm các mail bị gửi xịt (bounced) trong ngày hôm nay
grep "status=bounced" /var/log/zimbra.log

<img width="1391" height="173" alt="image" src="https://github.com/user-attachments/assets/c80aaa41-de62-4f9c-82b5-410670533a7a" />

Phần 3: Quản trị Backup & Restore (Sao lưu và Phục hồi dữ liệu)


Trên phiên bản Zimbra Open Source Edition (OSE), tính năng sao lưu tự động qua giao diện Web Admin đã bị cắt bỏ. Nếu máy chủ gặp sự cố ổ cứng hoặc người dùng lỡ tay xóa sạch hộp thư, bạn sẽ mất trắng dữ liệu. Do đó, việc nắm vững công cụ dòng lệnh zmmailbox để xuất/nhập dữ liệu dưới định dạng .tgz (bao gồm toàn bộ Email, Danh bạ, Lịch) là kỹ năng sống còn.

2. Hướng dẫn thao tác chi tiết 

Bước 2.1: Thao tác thủ công cho một người dùng cụ thể
Khi có một nhân sự nghỉ việc hoặc cần chuyển dữ liệu sang máy chủ khác, bạn có thể backup riêng tài khoản của họ.

Chuyển sang người dùng zimbra:

Bash
su - zimbra
Thực thi lệnh backup và xuất ra file .tgz:

Bash
zmmailbox -z -m a@domain.com getRestURL "//?fmt=tgz" > /tmp/backup_a.tgz
Bước 2.2: Tự động hóa Backup cho toàn bộ hệ thống bằng Script
Thực tế, không ai gõ lệnh tay mỗi ngày cho hàng trăm tài khoản. Chúng ta sẽ viết một đoạn script tự động lấy danh sách người dùng và backup toàn bộ.

Tạo thư mục chứa file backup và cấp quyền:

Bash
exit # Thoát khỏi user zimbra, quay về root
mkdir -p /backup/zimbra_mailboxes
chown -R zimbra:zimbra /backup/zimbra_mailboxes
Tạo file script /usr/local/bin/zimbra_backup.sh với nội dung:

<img width="1234" height="355" alt="image" src="https://github.com/user-attachments/assets/86b94e17-52c3-4f6b-9e7a-44ebad46c980" />

Bash
#!/bin/bash
# Định nghĩa thư mục lưu trữ và ngày tháng
BACKUP_DIR="/backup/zimbra_mailboxes"
DATE=$(date +"%Y%m%d")

echo "Bắt đầu tiến trình Backup lúc $(date)"

# Lấy danh sách tất cả account
ACCOUNTS=$(su - zimbra -c "zmprov -l gaa")

# Vòng lặp backup từng account
for ACCOUNT in $ACCOUNTS; do
    echo "Đang backup tài khoản: $ACCOUNT"
    su - zimbra -c "zmmailbox -z -m $ACCOUNT getRestURL \"//?fmt=tgz\" > $BACKUP_DIR/${ACCOUNT}_${DATE}.tgz"
done

echo "Tiến trình Backup hoàn tất!"

<img width="1322" height="500" alt="image" src="https://github.com/user-attachments/assets/45bb30d6-950b-4c28-90aa-1069278af1dd" />

Cấp quyền thực thi và đưa vào Crontab (chạy lúc 2h sáng mỗi ngày):


Bash
chmod +x /usr/local/bin/zimbra_backup.sh
crontab -e
# Thêm dòng sau vào cuối file:
# 0 2 * * * /usr/local/bin/zimbra_backup.sh >> /var/log/zimbra_backup.log 2>&1

<img width="1053" height="713" alt="image" src="https://github.com/user-attachments/assets/519e6a62-d9a9-4c27-a1e8-5283edd10d17" />

Bước 2.3: Phục hồi (Restore) dữ liệu
Khi cần khôi phục lại dữ liệu từ file backup đã tạo ở trên:

Bash
su - zimbra
zmmailbox -z -m a@mail.domain.com postRestURL "//?fmt=tgz&resolve=replace" /backup/zimbra_mailboxes/a@mail.domain.com_20260624.tgz
Lưu ý tham số resolve:

resolve=replace: Xóa đè các mail trùng lặp hiện có (Thường dùng khi hộp thư bị lỗi nặng).

resolve=skip: Bỏ qua các mail đã có, chỉ khôi phục các mail bị thiếu.

<img width="1395" height="412" alt="image" src="https://github.com/user-attachments/assets/fd58eb20-7a06-4861-8cd6-4a7a980343c6" />


Phần 4: Quản lý Bảo mật Anti-Spam & Anti-Virus (AS/AV)

Hệ thống Zimbra của bạn khi mở port ra ngoài Internet sẽ liên tục bị rà quét và ném thư rác (Spam). Luồng hoạt động bảo mật của Zimbra diễn ra như sau: Mail đến -> Postfix nhận -> Đẩy qua dịch vụ điều phối Amavisd -> Amavisd gọi ClamAV quét mã độc -> Gọi SpamAssassin chấm điểm ngữ nghĩa, đánh giá IP -> Nếu an toàn, Amavisd trả lại để Postfix phân phối vào hộp thư. Bạn phải kiểm soát tốt bộ máy này.

2. Hướng dẫn thao tác chi tiết

Bước 2.1: Cập nhật cơ sở dữ liệu nhận diện Virus
ClamAV cần được cập nhật mẫu virus mới liên tục (giống như các phần mềm diệt virus trên Windows).

Bash
su - zimbra
# Chạy lệnh cập nhật database (chạy thủ công khi có đợt tấn công mới)
/opt/zimbra/common/bin/freshclam

<img width="1475" height="333" alt="image" src="https://github.com/user-attachments/assets/c93e726e-cbaa-4209-8426-782aadae4f6a" />


Bước 2.2: Quản trị Whitelist và Blacklist (Danh sách trắng/đen)
Đây là thao tác bạn sẽ dùng nhiều nhất khi user phàn nàn "Mail đối tác bị vào Spam" hoặc "Bị gửi mail lừa đảo liên tục".

Thêm vào Whitelist (Bảo vệ mail đối tác quan trọng khỏi bộ lọc Spam):

Bash
# Cấp quyền cho toàn bộ domain của Vingroup gửi vào không bao giờ bị chặn
zmprov md mail.domain.com +amavisWhitelistSender vingroup.net

Thêm vào Blacklist (Chặn đứng nguồn phát tán Spam):

Bash
# Chặn một email lừa đảo
zmprov md mail.domain.com +amavisBlacklistSender hacker@gmail.com



Bước 2.3: Tinh chỉnh cơ chế chấm điểm Spam qua giao diện Web Admin
Ngoài việc chặn bằng lệnh, bạn nên điều chỉnh độ nhạy của bộ lọc để hệ thống tự phân loại.

Truy cập Admin Console (Cổng 7071/9071).

Điều hướng đến Cấu hình (Configure) -> Cài đặt chung (Global Settings) -> AS/AV.

Cấu hình 2 ngưỡng điểm (SpamAssassin Score) quan trọng:

Ngưỡng Gắn thẻ (Tag): Mặc định là 33%. Nếu mail bị đánh giá ngữ nghĩa đạt ngưỡng này, tiêu đề mail sẽ bị gắn thêm tiền tố [SPAM] (hoặc chui vào mục Thư rác). Bạn có thể hạ xuống 25% nếu muốn lọc chặt hơn.

Ngưỡng Xóa bỏ (Kill/Discard): Mặc định là 75%. Nếu mail đạt điểm này, Zimbra sẽ xóa lập tức, người dùng không hề biết có mail đến. Nếu công ty bạn thường xuyên làm việc với các hệ thống có cấu hình mail lỏng lẻo, bạn nên tăng mức này lên 85-90% để tránh mất thư oan.

<img width="1586" height="558" alt="image" src="https://github.com/user-attachments/assets/3e9fd022-5982-4441-b7b5-048190dc4904" />

