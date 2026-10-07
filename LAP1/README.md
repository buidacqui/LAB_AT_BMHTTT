# LAB 1 – Bắt gói tin Telnet – SSH

> Link youtube quá trình thực hiện bài lab:
> https://www.youtube.com/watch?v=D97avRzYX2M 

## Thông tin sinh viên

| Mục | Nội dung |
| --- | --- |
| Họ và tên | Bùi Đắc Quí |
| Mã số sinh viên | 1150080033 |
| Lớp | CNPM1 |
| Học phần | Thực hành An toàn Hệ thống thông tin |
| Tên bài Lab | Lab 1 – Bắt gói tin Telnet – SSH (Examining SSH & Telnet in Wireshark) |

## Mục tiêu

- Giả lập mạng Telnet – SSH theo mô hình Client/Server.
- Tìm hiểu cơ chế bắt và phân tích gói tin bằng Wireshark.
- So sánh thực nghiệm mức độ an toàn giữa Telnet (không mã hóa) và SSH (có mã hóa).

## Mô hình và môi trường

| Máy | Vai trò | Hệ điều hành | IP | Phần mềm |
| --- | --- | --- | --- | --- |
| Server | Máy chủ Telnet/SSH | Ubuntu Server 26.04 LTS | 10.0.0.1/24 | inetutils-telnetd, openssh-server |
| Client | Máy người dùng | Windows 11 | 10.0.0.2/24 | PuTTY 0.85 |
| Attacker | Máy bắt gói tin | Windows 11 | 10.0.0.3/24 | Wireshark 4.6.8 (Npcap) |

- Ảo hóa: VMware Workstation / VirtualBox (ghi rõ phần mềm đã dùng: ........).
- Điểm bắt gói tin: ........ (Attacker với Promiscuous Mode / Client / Server).
- Mạng lab cô lập, Telnet không được mở ra Internet.

## Nội dung đã thực hiện

1. Dựng mô hình mạng 3 máy, cấu hình IP tĩnh và kiểm tra kết nối bằng `ping`.
2. Cài Wireshark (kèm Npcap) và PuTTY; tạo tài khoản thử nghiệm trên Server.
3. **Telnet:** cài `inetutils-telnetd`, kiểm tra cổng TCP/23, đăng nhập từ Client, bắt gói với filter `tcp.port == 23`, dùng Follow TCP Stream để khôi phục username, password và lệnh; đổi sang mật khẩu phức tạp rồi lặp lại.
4. **SSH:** cài `openssh-server`, kiểm tra cổng TCP/22, đối chiếu host-key fingerprint, đăng nhập bằng PuTTY, bắt gói với filter `tcp.port == 22` và phân tích.
5. Demo đăng nhập SSH bằng khóa công khai Ed25519 (Câu 3).
6. Trả lời 11 câu hỏi dựa trên kết quả thực nghiệm.

## Kết quả thực hiện

| Tiêu chí quan sát | Telnet (TCP/23) | SSH (TCP/22) |
| --- | --- | --- |
| Địa chỉ IP/MAC, cổng | Thấy được | Thấy được |
| Username / Password | Đọc được (plaintext) | Không đọc được |
| Lệnh và kết quả lệnh | Đọc được | Không đọc được |
| Follow TCP Stream | Nội dung rõ ràng | Dữ liệu mã hóa vô nghĩa |

- Với mật khẩu dài và phức tạp, Telnet vẫn bị lộ mật khẩu nguyên văn vì điểm yếu nằm ở kênh truyền không mã hóa.
- Với SSH, Wireshark chỉ quan sát được metadata (IP, cổng, kích thước, thời điểm gói, banner và thuật toán thương lượng).