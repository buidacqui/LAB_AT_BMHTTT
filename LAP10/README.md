# LAB 1 – MÃ HÓA ĐỐI XỨNG CỔ ĐIỂN

> Link youtube quá trình thực hiện bài lab:
> https://youtu.be/jRmEuxuLssk

## Thông tin sinh viên

| Mục | Nội dung |
| --- | --- |
| Họ và tên | Bùi Đắc Quí |
| Mã số sinh viên | 1150080033 |
| Lớp | CNPM1 |
| Học phần | Thực hành An toàn Hệ thống thông tin |
| Tên bài Lab | LAB 1 – MÃ HÓA ĐỐI XỨNG CỔ ĐIỂN |

### Phần 1 – Chuẩn bị project

 Tạo project Maven `LAB1_MaHoaCoDien` với package `week_01`.
 Thiết kế JFrame bằng NetBeans GUI Builder.
 Tạo lớp `FileTextIO` để đọc và lưu nội dung tệp `.txt` bằng UTF-8.

### Phần 2 – Sáu thuật toán mật mã cổ điển

| Thuật toán | Plaintext | Key | Ciphertext mong đợi |
|---|---|---|---|
| Caesar | `HELLO` | `3` | `KHOOR` |
| Vigenère | `HELLO` | `KEY` | `RIJVS` |
| Rail Fence | `HELLO WORLD` | `3` | `HOREL OLLWD` |
| Playfair | `HELLOWORLD` | `KEYWORD` | `GYIZSCOKCFBU` |
| Transposition | `BAOMAT` | `2,3,1` | `AAOTBM` |
| Autokey | `HELLO` | `KEY` | `RIJSS` |

Mỗi thuật toán có form riêng hỗ trợ **Encrypt**, **Decrypt**, **Open Ciphertext File** và **Save Ciphertext File**. Riêng Playfair có chức năng tạo ma trận khóa 5×5.

### Phần 3 – Tổng hợp và lưu lịch sử SQL Server

 Form `frm_TongHop` cho phép chọn một trong sáu thuật toán qua Combo Box.
 Kết nối SQL Server qua JDBC (`DbConfig`).
 Ghi lịch sử **ENCRYPT/DECRYPT** vào bảng `dbo.history` (`HistoryDao`).
 Hiển thị lịch sử trong `frm_LichSu` bằng JTable; kiểm tra dữ liệu bằng SSMS.

Thông số kết nối theo LAB:

| Thông số | Giá trị |
|---|---|
| Server | `127.0.0.1` |
| Port | `14333` |
| Database | `LAB1_MaHoaCoDien` |
| User | `lab1_user` |

**Không lưu mật khẩu SQL Server vào mã nguồn hoặc GitHub.** Nhập mật khẩu qua form `frm_KetNoiSQL` khi chạy ứng dụng.

### Phần 4 – Phân tích mật mã cổ điển

 **Brute Force Caesar:** thử 26 khóa. Ví dụ `KHOOR` → khóa `3` → `HELLO`.
 **Frequency Analysis:** đếm tần suất xuất hiện các chữ cái trong bản mã.
 **Known Plaintext (Vigenère):** từ `HELLO` và `RIJVS` suy ra dãy khóa `KEYKE`, phù hợp với khóa `KEY` được lặp.

### Phần 5 – Menu chính

Form `frm_Menu` mở sáu form thuật toán, form tổng hợp, form phân tích mật mã và form kết nối SQL Server.

## 4. Cấu trúc dự án

```text
LAB1_MaHoaCoDien/
├── pom.xml
├── README.md
└── src/main/java/week_01/
    ├── CaesarCipher.java
    ├── VigenereCipher.java
    ├── RailFenceCipher.java
    ├── PlayfairCipher.java
    ├── TranspositionCipher.java
    ├── AutokeyCipher.java
    ├── CipherTongHop.java
    ├── AttackDemos.java
    ├── FileTextIO.java
    ├── DbConfig.java
    ├── HistoryDao.java
    ├── frm_Caesar.java
    ├── frm_Vigenere.java
    ├── frm_RailFence.java
    ├── frm_Playfair.java
    ├── frm_Transposition.java
    ├── frm_Autokey.java
    ├── frm_TongHop.java
    ├── frm_KetNoiSQL.java
    ├── frm_LichSu.java
    ├── frm_TanCong.java
    └── frm_Menu.java
```

Các JFrame tạo bởi NetBeans GUI Builder còn có tệp `.form` tương ứng. Cấu trúc trên biểu diễn các tệp chính, không liệt kê hết tệp sinh tự động.
