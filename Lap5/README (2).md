# LAB 5 – Thiết lập mô hình tường lửa pfSense

> Link youtube quá trình thực hiện bài lab:
> https://www.youtube.com/watch?v=ZVVWA2FV0lw 

## Thông tin sinh viên

| Mục | Nội dung |
| --- | --- |
| Họ và tên | Bùi Đắc Quí |
| Mã số sinh viên | 1150080033 |
| Lớp | CNPM1 |
| Học phần | Thực hành An toàn Hệ thống thông tin |
| Tên bài Lab | Lab 5 – Thiết lập mô hình tường lửa pfSense (pfSense Firewall Configuration) |

## Nội dung đã thực hiện

Dựng mô hình mạng có tường lửa **pfSense CE 2.7.2** bảo vệ vùng LAN và DMZ trên Oracle VirtualBox, sau đó kiểm thử các firewall rule theo tình huống.

**Phần cấu hình nền tảng**

1. Tạo máy ảo pfSense với 3 card mạng: WAN (Bridged), LAN (Host-only), DMZ (Internal Network `dmz-net`); kiểm tra SHA-256 file cài đặt và cài đặt từ ISO.
2. Cấu hình LAN `10.0.0.1/8` trên console; chạy Setup Wizard trên WebGUI `https://10.0.0.1` và đổi mật khẩu quản trị mặc định.
3. Dựng Windows Server làm **Domain Controller** (`10.0.0.2`, AD DS, DNS Forwarder `8.8.8.8`).
4. Gán card thứ ba thành interface **DMZ** (`172.16.0.1/16`) và dựng máy **DMZ-Web** (`172.16.0.2`, IIS).
5. Kiểm tra Outbound NAT tự động (Automatic/Hybrid) cho LAN và DMZ.
6. Disable hai rule *Default allow LAN to any* (giữ *Anti-Lockout Rule*), tạo rule nền tảng `LAN net → Any`, kiểm chứng bằng Reset States và lệnh kiểm thử mới.

**Bài tập theo tình huống**

| # | Tình huống | Cấu hình rule chính |
| --- | --- | --- |
| 1 | Chặn ICMP nhưng vẫn cho Web/DNS | Block ICMP → Pass DNS (53) → Pass HTTP/HTTPS (alias `WEB_PORTS` = 80, 443) |
| 2 | Chỉ cho một host ra Internet | Pass `10.0.0.2` → Any đặt trên Block `LAN net` → Any |
| 3 | Cô lập DMZ khỏi LAN | Block `DMZ net` → `LAN net` đặt trên Pass `DMZ net` → Any |

Ngoài ra bài còn trả lời 6 câu hỏi lý thuyết (NAT và firewall rule, DMZ, thứ tự rule, chặn ping nhưng cho web, logging, hardening).

## Mô hình mạng

| Thiết bị | Interface | IP | Gateway | DNS |
| --- | --- | --- | --- | --- |
| pfSense | WAN | DHCP | — | upstream |
| pfSense | LAN | 10.0.0.1/8 | — | — |
| pfSense | DMZ (OPT1) | 172.16.0.1/16 | — | — |
| Domain Controller | LAN | 10.0.0.2/8 | 10.0.0.1 | 10.0.0.2 |
| LAN-Test (Ubuntu) | LAN | 10.0.0.3/8 | 10.0.0.1 | 8.8.8.8 |
| Máy thật quản trị | Host-only | 10.0.0.100/8 | — | — |
| DMZ-Web | DMZ | 172.16.0.2/16 | 172.16.0.1 | 8.8.8.8 |

## Kết quả thực hiện

| Tình huống | Kiểm thử | Kết quả |
| --- | --- | --- |
| Nền tảng | Rule `LAN net → Any` Enabled; DC ping, DNS, HTTPS | Thành công |
| Nền tảng | Disable rule, Reset States, ping mới từ DC | Bị chặn (default deny) |
| 1 | `ping 8.8.8.8` từ DC | Thất bại (Request timed out) |
| 1 | `Resolve-DnsName example.com -Server 8.8.8.8`, `curl.exe -4 https://example.com` | Thành công |
| 2 | Ping `8.8.8.8` từ DC (`10.0.0.2`) | Thành công |
| 2 | Ping `8.8.8.8` từ LAN-Test (`10.0.0.3`) | Thất bại (100% packet loss) |
| 3 | Baseline: DMZ-Web ping DC `10.0.0.2` (trước khi Block) | Thành công |
| 3 | Sau khi thêm Block `DMZ net → LAN net`: ping DC | Thất bại |
| 3 | DMZ-Web ping `8.8.8.8`, `Resolve-DnsName example.com` | Thành công |

**Kết luận chính:**

- pfSense xử lý rule từ trên xuống và dừng ở rule đầu tiên khớp, nên thứ tự rule quyết định kết quả.
- pfSense là stateful firewall: phải *Reset States* và dùng lệnh kiểm thử mới thì kết luận mới đúng.
- Cần có baseline trước khi chặn để chứng minh chính pfSense (không phải Windows Firewall) là thành phần chặn.
- DMZ cần rule Pass mới ra được Internet dù đã có NAT; rule Block DMZ → LAN ngăn lateral movement vào mạng nội bộ.
