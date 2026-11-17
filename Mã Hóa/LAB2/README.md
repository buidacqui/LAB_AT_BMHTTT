# LAB 2 – MÃ HÓA ĐỐI XỨNG HIỆN ĐẠI

> Link youtube quá trình thực hiện bài lab:
> https://youtu.be/jRmEuxuLssk

## Thông tin sinh viên

| Mục | Nội dung |
| --- | --- |
| Họ và tên | Bùi Đắc Quí |
| Mã số sinh viên | 1150080033 |
| Lớp | CNPM1 |
| Học phần | Thực hành An toàn Hệ thống thông tin |
| Tên bài Lab | LAB 2 – MÃ HÓA ĐỐI XỨNG HIỆN ĐẠI |

## 1. Giới thiệu

Dự án minh họa các thuật toán mã hóa đối xứng hiện đại thông qua ứng dụng desktop **Java Swing** xây dựng với **Apache NetBeans GUI Builder**. Ứng dụng cung cấp các chức năng mã hóa/giải mã, đọc/ghi tệp, kết nối **SQL Server** để lưu thông tin lịch sử thao tác và đăng ký/đăng nhập cho bài AES. Ngoài ứng dụng Java, LAB còn có bài thực hành **AES-CBC bằng Python/Tkinter**.

**Mục đích:** tìm hiểu cách sử dụng khóa, IV/nonce, chế độ hoạt động, định dạng dữ liệu mã hóa và các vấn đề an toàn khi triển khai mật mã.

## 2. Công nghệ sử dụng

| Công nghệ | Vai trò |
|---|---|
| JDK 17 (hoặc 21) | Biên dịch và thực thi Java |
| Apache NetBeans 20+ | IDE, thiết kế giao diện JFrame |
| Java Swing | Giao diện desktop |
| Apache Maven | Quản lý build và dependencies |
| Microsoft JDBC Driver `12.8.1.jre11` | Kết nối SQL Server |
| Bouncy Castle `bcprov-jdk18on:1.78.1` | Hỗ trợ Twofish |
| Microsoft SQL Server Express + SSMS | Lưu tài khoản và lịch sử thao tác |
| Python 3.10+ / `cryptography` / Tkinter | Thực hành AES-CBC với GUI |

## 3. Chức năng

### 3.1. Chuẩn bị hệ thống

- Tạo Maven project `LAB2_MaHoaHienDai`, package `week_02`.
- Dùng `CipherUtil` để chuyển UTF-8/Base64/HEX, tạo byte ngẫu nhiên và dẫn xuất khóa bằng PBKDF2-HMAC-SHA256.
- Dùng `FileTextIO` để đọc/ghi tệp UTF-8.
- Dùng `DbConfig`, `HistoryDao`, `UserDao` để kết nối SQL Server, lưu lịch sử và quản lý tài khoản.

### 3.2. Các bài thực hành mã hóa

| Bài | Thuật toán | Nội dung chính |
|---|---|---|
| 2.1 | DES-CBC | Khóa 8 byte, IV ngẫu nhiên 8 byte, PKCS5Padding, kết quả Base64 |
| 2.2 | TripleDES-CBC | Khóa 24 byte, IV ngẫu nhiên, mã hóa/giải mã |
| 2.3 | AES-GCM | Đăng ký, đăng nhập, Ticket; khóa dẫn xuất từ mật khẩu, phát hiện bản mã bị sửa |
| 2.4 | RC4 | Thuật toán mã dòng, kết quả hiển thị HEX |
| 2.5 | Twofish-CBC | Tạo IV, mã hóa/giải mã, lưu/đọc IV cùng bản mã |
| 2.6 | AES-CTR (Java) | Tạo Key và nonce, mã hóa không padding, lưu/đọc nonce cùng bản mã |
| Nâng cao | AES-CBC (Python) | Giao diện Tkinter, kiểm thử self-test và xử lý tệp |

### 3.3. Cơ sở dữ liệu

- **`dbo.app_users`**: username, salt, `password_hash`, thời gian tạo tài khoản. Không lưu mật khẩu gốc.
- **`dbo.crypto_history`**: thuật toán, thao tác, username, độ dài đầu vào/đầu ra, thời gian. **Không lưu plaintext hay khóa trong lịch sử.**
- Hiển thị các bản ghi mới nhất trên `frm_LichSu`.

## 4. Cấu trúc dự án

```text
C:\BaoMat\
├── LAB2_MaHoaHienDai\
│   ├── pom.xml
│   └── src\main\java\week_02\
│       ├── CipherUtil.java
│       ├── FileTextIO.java
│       ├── DbConfig.java
│       ├── HistoryDao.java
│       ├── UserDao.java
│       ├── DESCipher.java
│       ├── TripleDESCipher.java
│       ├── AESCipher.java
│       ├── RC4Cipher.java
│       ├── TwofishCipher.java
│       ├── AESCTRCipher.java
│       ├── frm_DES.java
│       ├── frm_TripleDES.java
│       ├── frm_AES.java
│       ├── frm_RC4.java
│       ├── frm_Twofish.java
│       ├── frm_AES_CTR.java
│       ├── frm_KetNoiSQL.java
│       ├── frm_LichSu.java
│       └── frm_Menu.java
└── LAB2_Python\
    └── aes_cbc_gui.py
```

> Các JFrame thiết kế bằng NetBeans có tệp `.form` tương ứng. Cấu trúc trên mô tả các tệp chính theo tài liệu thực hành; hãy giữ nguyên cấu trúc thực tế của project khi đưa lên GitHub.

## 5. Hướng dẫn cài đặt và chạy

### 5.1. Java – NetBeans

1. Cài **JDK 17/21**, **Apache NetBeans 20+** và kết nối Internet để Maven tải thư viện lần đầu.
2. Mở project `LAB2_MaHoaHienDai` trong NetBeans.
3. Kiểm tra file `pom.xml` có các dependency `mssql-jdbc` và `bcprov-jdk18on`.
4. Kiểm tra tên class chính khớp chính xác chữ hoa/thường:

   ```xml
   <exec.mainClass>week_02.frm_Menu</exec.mainClass>
   ```

5. Nhấp phải project → **Clean and Build** (`Shift + F11`).
6. Nhấn **F6** để chạy `frm_Menu`. Trước khi tạo menu, có thể nhấp phải từng form → **Run File** (`Shift + F6`).

### 5.2. SQL Server

1. Bật TCP/IP trên instance `SQLEXPRESS`, cấu hình **TCP Port = 14333**, để trống Dynamic Ports.
2. Bật **SQL Server and Windows Authentication mode** (Mixed Mode) và restart SQL Server nếu đã thay đổi.
3. Trong SSMS, chạy script tạo database **`LAB2_MaHoaHienDai`** và các bảng **`dbo.app_users`**, **`dbo.crypto_history`** theo tài liệu LAB.
4. Tạo SQL login **`lab2_user`**, map với database và cấp quyền đọc/ghi cần thiết.
5. Mở **Kết nối SQL Server** trong ứng dụng và nhập:

   | Trường | Giá trị |
   |---|---|
   | Server | `127.0.0.1` |
   | Port | `14333` |
   | Database | `LAB2_MaHoaHienDai` |
   | User | `lab2_user` |
   | Password | Nhập mật khẩu đã cấu hình cho tài khoản LAB |

6. Nhấn **Test Connection**, sau đó mở **Xem lịch sử** để kiểm tra.

> Tài liệu dùng mật khẩu mẫu `LabUser@2026` dành cho máy thực hành cá nhân. **Không commit mật khẩu, tệp cấu hình chứa thông tin bí mật hoặc dữ liệu tài khoản thật lên GitHub.** Tùy chọn `trustServerCertificate=true` trong bài chỉ phù hợp SQL Server nội bộ với chứng chỉ thử nghiệm.

### 5.3. Python – AES-CBC

Cài thư viện:

```powershell
py -m pip install cryptography
```

Chạy self-test:

```powershell
cd C:\BaoMat\LAB2_Python
py aes_cbc_gui.py --self-test
```

Chạy giao diện:

```powershell
py aes_cbc_gui.py
```

## 6. Dữ liệu kiểm thử bắt buộc

| Bài | Dữ liệu/Thao tác | Kết quả mong đợi |
|---|---|---|
| DES | `Xin chào Lab 2 - Mã hóa hiện đại!`, Key `12345678` | Giải mã về văn bản gốc; hai lần Encrypt thường khác nhau do IV |
| TripleDES | Key `123456789012345678901234` | Giải mã về văn bản gốc; Key 23 ký tự bị từ chối |
| AES-GCM | Register → Login → Encrypt/Decrypt | Login thành công, có Ticket; `app_users` và `crypto_history` có bản ghi |
| RC4 | Plaintext `Plaintext`, Key `Key` | HEX `BBF316E8D940AF0AD3` |
| Twofish | Generate IV → Encrypt → Save/Open → Decrypt | Phục hồi IV và bản rõ |
| AES-CTR | Key/nonce mới → Encrypt/Decrypt | Giải mã đúng, số byte bản mã bằng số byte bản rõ |
| AES-CBC Python | `py aes_cbc_gui.py --self-test` | In thông báo `SELF-TEST OK` |

**Lưu ý:** Đây là các **kết quả mong đợi theo tài liệu**, không khẳng định mọi phép thử đã PASS trên máy thực tế. Hãy chạy và ghi nhận kết quả trước khi báo cáo.

## 7. Kiểm tra dữ liệu SQL

Mở SSMS và chạy:

```sql
USE LAB2_MaHoaHienDai;
GO
SELECT user_id, username, created_at
FROM dbo.app_users ORDER BY user_id DESC;

SELECT TOP (50) history_id, created_at, algorithm, operation,
       app_username, input_length, output_length
FROM dbo.crypto_history ORDER BY history_id DESC;
GO
```