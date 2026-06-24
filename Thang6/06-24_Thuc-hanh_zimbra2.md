
# BÁO CÁO THỰC TẬP — NGÀY 24/06/2026
## Quản trị Zimbra Mail Server: User, Webmail và Bảo mật

---

## MỤC LỤC

1. [Quản trị User và Webmail — Gửi/Nhận nội bộ](#phan-1)
   - 1.1. [Truy cập Admin Console](#buoc-1)
   - 1.2. [Tạo Class of Service (CoS) và Account](#buoc-2)
   - 1.3. [Thực hành Webmail và gửi/nhận nội bộ](#buoc-3)
   - 1.4. [Thiết lập chữ ký email](#buoc-4)
   - 1.5. [Thiết lập chuyển tiếp email (Forward)](#buoc-5)
2. [Quản lý Mail Queue (Hàng đợi Mail)](#phan-2)
3. [Quản trị Log](#phan-3)
4. [Quản trị Backup và Restore](#phan-4)
   - 4.1. [Backup thủ công cho một tài khoản](#buoc-2-1)
   - 4.2. [Tự động hóa Backup bằng Script](#buoc-2-2)
   - 4.3. [Phục hồi (Restore) dữ liệu](#buoc-2-3)
5. [Quản lý Bảo mật Anti-Spam và Anti-Virus (AS/AV)](#phan-5)
   - 5.1. [Cập nhật cơ sở dữ liệu nhận diện Virus](#buoc-as-1)
   - 5.2. [Quản trị Whitelist và Blacklist](#buoc-as-2)
   - 5.3. [Tinh chỉnh ngưỡng điểm Spam qua Admin Console](#buoc-as-3)

---

## Phần 1: Quản trị User và Webmail — Gửi/Nhận nội bộ {#phan-1}

### 1.1. Truy cập Admin Console {#buoc-1}

Giao diện quản trị của Zimbra (Admin Console) được tách biệt hoàn toàn với giao diện Webmail của người dùng thông thường. Admin Console chạy trên cổng **7071** với giao thức HTTPS.

**Các bước thực hiện:**

**Bước 1:** Mở trình duyệt web và truy cập theo đường dẫn:


https://<IP_Server_Zimbra>:7071


![Giao diện đăng nhập Admin Console Zimbra](https://github.com/user-attachments/assets/597a9082-6be6-4780-8bb1-d7cd15565563)

**Bước 2:** Trình duyệt sẽ hiển thị cảnh báo bảo mật **"Your connection is not private"** do Zimbra mặc định sử dụng chứng chỉ SSL tự ký (Self-signed Certificate). Đây là hành vi bình thường trong môi trường lab. Chọn **Advanced → Proceed** để tiếp tục.

**Bước 3:** Đăng nhập bằng tài khoản quản trị (ví dụ: `admin@mail.domain.com`) và mật khẩu đã thiết lập trong quá trình cài đặt.

---

### 1.2. Tạo Class of Service (CoS) và Account {#buoc-2}

**Class of Service (CoS)** là một tính năng quan trọng trong Zimbra, cho phép gom nhóm các cấu hình (Quota, Feature, Theme...) và áp dụng đồng loạt cho nhiều tài khoản thay vì phải cấu hình thủ công từng người. Đây là cách tiếp cận hiệu quả khi quản lý hệ thống có nhiều người dùng với các nhóm vai trò khác nhau (nhân viên, quản lý, kỹ thuật...).

#### Tạo CoS phân bổ Quota

Trên thanh menu bên trái, điều hướng tới **Configure → Class of Service**. Nhấn vào biểu tượng bánh răng (góc trên bên phải bảng) → chọn **New**.

**Tạo CoS cho nhóm Staff:**

- **General Information:** Đặt tên là `Staff_CoS`.

![Tạo CoS Staff — General Information](https://github.com/user-attachments/assets/4ed73c74-7325-4a86-bae1-74eaf9881b73)

- **Advanced:** Tìm đến mục **Account Quota**, nhập `5120` MB (tương đương 5 GB).

![Tạo CoS Staff — Cấu hình Quota 5GB](https://github.com/user-attachments/assets/7efbda96-3b7d-48ae-8655-9571ba7462cf)

- Nhấn **Save**.

**Tạo CoS cho nhóm Manager:**

Thực hiện tương tự, đặt tên là `Manager_CoS`. Tại mục **Advanced → Account Quota**, nhập `10240` MB (tương đương 10 GB). Nhấn **Save**.

![Tạo CoS Manager — Cấu hình Quota 10GB](https://github.com/user-attachments/assets/09398cb4-d0a6-45c0-b5a4-9795b283bc33)

![Danh sách CoS sau khi tạo](https://github.com/user-attachments/assets/22e3f4bc-bd48-400f-adb7-4aba89d46f47)

#### Tạo Account (User)

Điều hướng tới **Manage → Accounts**. Nhấn nút bánh răng → **New**.

**Tạo tài khoản Staff** (ví dụ: `nhanvien@domain.com`):

- Điền các thông tin cơ bản: Account Name, Last Name, Password.

![Tạo tài khoản Staff — Điền thông tin cơ bản](https://github.com/user-attachments/assets/de68d39a-4e5c-433f-aab8-7d0717ee23b5)

- Kéo xuống phần **Class of Service**, bỏ chọn "default" và chọn `Staff_CoS`.

![Gán CoS Staff cho tài khoản nhân viên](https://github.com/user-attachments/assets/faa672d7-d73c-43da-93c3-782da6a2753f)

Giao diện quản lý tài khoản (thêm, sửa, xóa):

![Danh sách tài khoản — Thêm/Sửa/Xóa](https://github.com/user-attachments/assets/bf8e4b2f-9687-4178-a57e-5c94a17c72e0)

**Tạo tài khoản Manager** (ví dụ: `quanly@domain.com`):

- Điền thông tin tương tự và chọn **Class of Service** là `Manager_CoS`.

![Tạo tài khoản Manager — Gán Manager_CoS](https://github.com/user-attachments/assets/1f558bb3-7b49-41d5-9df7-2d0b48862bf9)

- Nhấn **Save** cho từng tài khoản.

---

### 1.3. Thực hành Webmail và gửi/nhận nội bộ {#buoc-3}

**Bước 1:** Mở tab ẩn danh (Incognito) để tránh xung đột session với phiên Admin đang mở. Truy cập Webmail theo đường dẫn:


https://192.168.197.148


(Sử dụng cổng 443 mặc định của HTTPS)

![Giao diện đăng nhập Webmail Zimbra](https://github.com/user-attachments/assets/52d95409-5739-4d80-9978-cceb6512085e)

**Bước 2:** Đăng nhập bằng tài khoản nhân viên `a@domain.com`.

![Giao diện Webmail sau khi đăng nhập tài khoản a](https://github.com/user-attachments/assets/a2c899be-c3ec-47ee-9721-bca2b7947fdd)

**Bước 3:** Soạn một email mới gửi đến `b@domain.com` để kiểm tra luồng mail nội bộ.

![Soạn email gửi từ a đến b — Kiểm tra luồng nội bộ](https://github.com/user-attachments/assets/5f92a33f-ce4d-4294-835f-e56bd15d12c7)

**Bước 4:** Đăng nhập tài khoản `b@domain.com` và thực hiện **Reply** để xác nhận chiều ngược lại hoạt động ổn định.

![Tài khoản b nhận được email từ a](https://github.com/user-attachments/assets/6e8bd668-249b-4d7d-b899-0027c04e63fa)

![Tài khoản b thực hiện Reply lại](https://github.com/user-attachments/assets/1cf51149-d817-463f-b6ea-e44be494d748)

**Kết quả:** Email gửi và nhận nội bộ hoạt động bình thường theo cả hai chiều.

**Kiểm tra Quota:** Di chuột vào tên tài khoản ở góc trên bên phải màn hình Webmail để xem thanh hiển thị Quota, xác nhận Staff nhận 5 GB và Manager nhận 10 GB đúng như cấu hình trong CoS.

![Kiểm tra Quota tài khoản Staff — 5GB](https://github.com/user-attachments/assets/0eb501e9-9237-448d-9f62-fc3363aae658)

![Kiểm tra Quota tài khoản Manager — 10GB](https://github.com/user-attachments/assets/268e676e-4b0e-46c0-963b-c030ad32847f)

---

### 1.4. Thiết lập chữ ký email {#buoc-4}

Chữ ký email (Email Signature) giúp thông tin liên hệ được hiển thị tự động ở cuối mỗi email soạn thảo, tạo tính chuyên nghiệp cho người dùng.

**Các bước thực hiện tại giao diện Webmail:**

1. Vào **Tùy chọn (Preferences)** → **Chữ ký (Signatures)**.
2. Nhập tên định danh cho chữ ký.
3. Chọn định dạng: **Văn bản thường (As Plaintext)** hoặc **HTML (As Html)**.
4. Nhập nội dung chữ ký.
5. Chọn vị trí hiển thị (cuối email mới, cuối email trả lời...).
6. Nhấn **Lưu (Save)**.

![Giao diện tạo chữ ký — Preferences > Signatures](https://github.com/user-attachments/assets/80991d50-3185-4f66-8f7e-ae5534778624)

![Nội dung chữ ký sau khi tạo](https://github.com/user-attachments/assets/4bdd1c6f-429b-4e9e-8bc6-aa30c0f7d526)

**Lưu ý:** Khi soạn email mới, cần chọn định dạng soạn thảo tương ứng với định dạng đã thiết lập cho chữ ký (Plaintext với Plaintext, HTML với HTML) để chữ ký hiển thị đúng.

![Chữ ký hiển thị đúng khi soạn email mới](https://github.com/user-attachments/assets/9b09f21a-6d39-4418-9749-f17817ab5711)

**Xóa chữ ký:** Tại giao diện quản lý chữ ký, chọn chữ ký muốn xóa → nhấn nút **Delete**.

---

### 1.5. Thiết lập chuyển tiếp email (Forward) {#buoc-5}

Tính năng Forward cho phép tự động chuyển tiếp toàn bộ email đến một địa chỉ khác, hữu ích khi người dùng muốn tổng hợp thư từ nhiều hộp thư về một nơi.

> **Lưu ý quan trọng:** Trên Zimbra, mặc định quyền tự cấu hình Forward của người dùng có thể bị giới hạn bởi CoS. Quản trị viên cần bật tính năng này trong CoS trước nếu người dùng không thấy tùy chọn.

**Bước 1:** Đăng nhập vào Webmail bằng tài khoản cần cấu hình.

**Bước 2:** Vào **Tùy chọn (Preferences) → Thư (Mail) → Nhận thư (Receiving Messages)**. Tìm mục cài đặt chuyển tiếp, nhập địa chỉ email đích và nhấn **Lưu**.

![Cấu hình Forward email — Preferences > Mail > Receiving Messages](https://github.com/user-attachments/assets/154203bb-7114-4ef7-8f53-6ea6f4b6f4f2)

**Bước 3:** Gửi thử một email đến tài khoản vừa cấu hình để xác nhận Forward hoạt động đúng.

![Kết quả kiểm tra — Email từ a được forward tới admin](https://github.com/user-attachments/assets/2c44a519-54f0-43aa-9204-347bb5611a8d)

**Kết quả:** Email gửi đến `a@domain.com` đã được chuyển tiếp thành công đến tài khoản admin.

![Hộp thư admin nhận được email được forward từ a](https://github.com/user-attachments/assets/5136306f-c41b-4873-a83a-96e57756dd2e)

---

## Phần 2: Quản lý Mail Queue (Hàng đợi Mail) {#phan-2}

Khi mạng chậm, server đích bị lỗi, hoặc mail gặp sự cố trong quá trình gửi, các email đó sẽ được giữ lại trong **Queue của Postfix** (MTA — Mail Transfer Agent của Zimbra) để chờ xử lý tiếp theo. Quản trị viên cần theo dõi và xử lý Queue thường xuyên để tránh mail bị tồn đọng.

**Chuyển sang user zimbra trước khi thực hiện các lệnh:**

```bash
su - zimbra
```

**Xem danh sách mail đang kẹt trong hàng đợi:**

```bash
mailq
```

Hệ thống sẽ in ra danh sách các Queue ID, lý do bị kẹt và địa chỉ người nhận.

**Ép hệ thống gửi lại toàn bộ mail trong Queue ngay lập tức (Flush):**

```bash
/opt/zimbra/common/sbin/postqueue -f
```

Dùng khi server đích đã phục hồi và muốn thử gửi lại ngay thay vì chờ Postfix tự retry.

**Xóa một mail cụ thể khỏi Queue** (ví dụ: mail spam đã xác định):

```bash
/opt/zimbra/common/sbin/postsuper -d <Queue_ID>
```

**Xóa toàn bộ Queue** (thao tác nguy hiểm, cần cân nhắc kỹ):

```bash
/opt/zimbra/common/sbin/postsuper -d ALL
```

> **Cảnh báo:** Lệnh `postsuper -d ALL` sẽ xóa vĩnh viễn tất cả mail đang chờ trong hàng đợi, bao gồm cả mail hợp lệ chưa gửi được. Chỉ dùng khi Queue bị tràn do spam và đã xác nhận không có mail quan trọng.

**Kết quả kiểm tra:** Hàng đợi hiện tại trống, không có mail nào bị kẹt.

![Kết quả mailq — Hàng đợi trống](https://github.com/user-attachments/assets/41e83613-f478-4219-a468-2a1e9dee6283)

---

## Phần 3: Quản trị Log {#phan-3}

Log là công cụ quan trọng nhất để chẩn đoán sự cố mail. Thay vì đọc toàn bộ log, quản trị viên cần biết cách lọc đúng thông tin cần thiết bằng `grep` và `tail`.

**Postfix định nghĩa 4 trạng thái xử lý mail quan trọng:**

| Trạng thái | Ý nghĩa |
|---|---|
| `status=sent` | Mail đã được gửi thành công |
| `status=bounced` | Mail bị trả lại (địa chỉ sai, hòm thư đầy) |
| `status=deferred` | Mail bị hoãn, đưa vào Queue để thử lại sau (mạng chập chờn, server đích tạm sập) |
| `status=rejected` | Mail bị từ chối thẳng (thường do dính Spam, sai cấu hình SPF/DKIM) |

**Theo dõi log Zimbra theo thời gian thực:**

```bash
tail -f /var/log/zimbra.log
```

![Log Zimbra real-time — tail -f /var/log/zimbra.log](https://github.com/user-attachments/assets/f3846646-cc14-4faf-8655-963d633545c2)

**Xem log đăng nhập và thao tác Mailbox** (theo dõi ai đang login Webmail, nhập sai mật khẩu, hoặc tải file đính kèm):

```bash
tail -f /opt/zimbra/log/mailbox.log
```

![Log mailbox.log — Theo dõi hoạt động đăng nhập Webmail](https://github.com/user-attachments/assets/752d1256-10f5-4ae7-929c-0e79c3fce545)

**Tìm tất cả log liên quan đến một địa chỉ email cụ thể** (khi user báo lỗi, tìm đích danh mail của họ):

```bash
grep "a@mail.domain.com" /var/log/zimbra.log
```

![Kết quả grep log theo địa chỉ email cụ thể](https://github.com/user-attachments/assets/94d11fa6-490e-433d-b7f3-e040ab6f686b)

**Tìm các mail bị trả lại (bounced):**

```bash
grep "status=bounced" /var/log/zimbra.log
```

![Kết quả grep status=bounced](https://github.com/user-attachments/assets/c80aaa41-de62-4f9c-82b5-410670533a7a)

---

## Phần 4: Quản trị Backup và Restore (Sao lưu và Phục hồi dữ liệu) {#phan-4}

Trên phiên bản **Zimbra Open Source Edition (OSE)**, tính năng sao lưu tự động qua giao diện Web Admin đã bị loại bỏ (chỉ có trên bản Network Edition thương mại). Do đó, nếu máy chủ gặp sự cố phần cứng hoặc người dùng xóa nhầm dữ liệu, toàn bộ email sẽ bị mất nếu không có chiến lược backup thủ công.

Công cụ `zmmailbox` cho phép xuất/nhập dữ liệu hộp thư dưới định dạng `.tgz`, bao gồm toàn bộ Email, Danh bạ (Contacts) và Lịch (Calendar).

---

### 4.1. Backup thủ công cho một tài khoản {#buoc-2-1}

Dùng khi cần backup riêng lẻ (ví dụ: nhân sự sắp nghỉ việc, chuyển dữ liệu sang server khác).

**Bước 1:** Chuyển sang user zimbra:

```bash
su - zimbra
```

**Bước 2:** Thực thi lệnh xuất dữ liệu hộp thư ra file `.tgz`:

```bash
zmmailbox -z -m a@domain.com getRestURL "//?fmt=tgz" > /tmp/backup_a.tgz
```

---

### 4.2. Tự động hóa Backup bằng Script {#buoc-2-2}

Trong môi trường thực tế với hàng trăm tài khoản, không thể gõ lệnh thủ công mỗi ngày. Script sau sẽ tự động lấy danh sách toàn bộ người dùng và backup tất cả.

**Bước 1:** Thoát khỏi user zimbra (nếu đang ở trong), tạo thư mục lưu trữ và phân quyền:

```bash
exit
mkdir -p /backup/zimbra_mailboxes
chown -R zimbra:zimbra /backup/zimbra_mailboxes
```

**Bước 2:** Tạo file script `/usr/local/bin/zimbra_backup.sh`:

![Nội dung script zimbra_backup.sh](https://github.com/user-attachments/assets/86b94e17-52c3-4f6b-9e7a-44ebad46c980)

```bash
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
```

![Script backup chạy thử — Kết quả từng tài khoản](https://github.com/user-attachments/assets/45bb30d6-950b-4c28-90aa-1069278af1dd)

**Bước 3:** Cấp quyền thực thi và đưa vào Crontab để chạy tự động lúc 2:00 sáng mỗi ngày:

```bash
chmod +x /usr/local/bin/zimbra_backup.sh
crontab -e
```

Thêm dòng sau vào cuối file crontab:

```
0 2 * * * /usr/local/bin/zimbra_backup.sh >> /var/log/zimbra_backup.log 2>&1
```

![Cấu hình Crontab — Lên lịch backup lúc 2h sáng mỗi ngày](https://github.com/user-attachments/assets/519e6a62-d9a9-4c27-a1e8-5283edd10d17)

---

### 4.3. Phục hồi (Restore) dữ liệu {#buoc-2-3}

Khi cần khôi phục dữ liệu từ file backup:

```bash
su - zimbra
zmmailbox -z -m a@mail.domain.com postRestURL "//?fmt=tgz&resolve=replace" /backup/zimbra_mailboxes/a@mail.domain.com_20260624.tgz
```

**Giải thích tham số `resolve`:**

| Tham số | Hành vi |
|---|---|
| `resolve=replace` | Xóa và ghi đè các mail trùng lặp đã có trong hộp thư. Dùng khi hộp thư bị lỗi nặng, cần khôi phục hoàn toàn từ backup. |
| `resolve=skip` | Bỏ qua các mail đã tồn tại, chỉ nhập lại các mail bị thiếu. Dùng khi muốn bổ sung dữ liệu mà không ghi đè. |

![Kết quả lệnh Restore — Phục hồi dữ liệu hộp thư](https://github.com/user-attachments/assets/fd58eb20-7a06-4861-8cd6-4a7a980343c6)

---

## Phần 5: Quản lý Bảo mật Anti-Spam và Anti-Virus (AS/AV) {#phan-5}

Khi mở cổng mail ra Internet, hệ thống Zimbra liên tục bị rà quét và nhận thư rác (Spam). Hiểu và kiểm soát tốt hệ thống lọc là nhiệm vụ quan trọng của quản trị viên.

**Luồng xử lý bảo mật của Zimbra:**

```
Mail đến → Postfix nhận → Amavisd (điều phối) → ClamAV (quét virus) → SpamAssassin (chấm điểm Spam) → Postfix phân phối vào hộp thư
```

---

### 5.1. Cập nhật cơ sở dữ liệu nhận diện Virus {#buoc-as-1}

**ClamAV** là engine diệt virus tích hợp trong Zimbra. Giống như phần mềm diệt virus trên máy tính, ClamAV cần được cập nhật định nghĩa virus mới thường xuyên để nhận diện các mối đe dọa mới nhất.

```bash
su - zimbra
/opt/zimbra/common/bin/freshclam
```

![Kết quả cập nhật ClamAV database bằng freshclam](https://github.com/user-attachments/assets/c93e726e-cbaa-4209-8426-782aadae4f6a)

Lệnh này nên được chạy định kỳ (hoặc đưa vào Crontab) để đảm bảo ClamAV luôn có định nghĩa virus mới nhất.

---

### 5.2. Quản trị Whitelist và Blacklist {#buoc-as-2}

Đây là thao tác thường dùng nhất khi: người dùng phàn nàn "mail đối tác quan trọng bị vào thư mục Spam" hoặc "liên tục nhận mail lừa đảo từ một nguồn cụ thể".

**Thêm vào Whitelist** — Bảo vệ mail từ domain đối tác quan trọng không bao giờ bị lọc Spam:

```bash
zmprov md mail.domain.com +amavisWhitelistSender vingroup.net
```

Lệnh trên sẽ thêm toàn bộ domain `vingroup.net` vào danh sách trắng, mọi email từ domain này sẽ bỏ qua bộ lọc SpamAssassin.

**Thêm vào Blacklist** — Chặn hoàn toàn một địa chỉ email lừa đảo:

```bash
zmprov md mail.domain.com +amavisBlacklistSender hacker@gmail.com
```

> **Lưu ý:** Thay `mail.domain.com` bằng tên domain thực tế của hệ thống Zimbra đang quản lý.

---

### 5.3. Tinh chỉnh ngưỡng điểm Spam qua Admin Console {#buoc-as-3}

**SpamAssassin** chấm điểm từng email dựa trên nhiều tiêu chí: nội dung ngữ nghĩa, danh tiếng IP gửi, cấu hình SPF/DKIM... Zimbra cho phép điều chỉnh hai ngưỡng điểm quan trọng qua giao diện đồ họa.

**Truy cập:** Admin Console → **Configure (Cấu hình)** → **Global Settings (Cài đặt chung)** → tab **AS/AV**.

**Hai ngưỡng quan trọng cần cấu hình:**

| Ngưỡng | Mặc định | Hành vi | Khuyến nghị |
|---|---|---|---|
| **Tag (Gắn thẻ)** | 33% | Mail bị gắn tiền tố `[SPAM]` vào tiêu đề và chuyển vào thư mục Thư rác | Hạ xuống 25% nếu muốn lọc chặt hơn |
| **Kill (Xóa bỏ)** | 75% | Mail bị xóa hoàn toàn, người dùng không nhận được bất kỳ thông báo nào | Tăng lên 85-90% nếu thường xuyên làm việc với các hệ thống có cấu hình mail lỏng lẻo, tránh mất thư hợp lệ |

> **Lưu ý quan trọng về ngưỡng Kill:** Nếu đặt quá thấp, nguy cơ xóa nhầm email hợp lệ rất cao và người dùng sẽ không biết. Cần cân nhắc kỹ dựa trên đặc thù môi trường trước khi điều chỉnh.

![Cấu hình ngưỡng AS/AV trong Admin Console — Global Settings](https://github.com/user-attachments/assets/3e9fd022-5982-4441-b7b5-048190dc4904)
```
