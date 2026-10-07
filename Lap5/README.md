# Lab 5 – Thiết lập mô hình tường lửa pfSense

Báo cáo thực hành môn **An toàn Hệ thống thông tin**: dựng mô hình mạng có tường lửa **pfSense CE 2.7.2** bảo vệ vùng LAN và DMZ, sau đó kiểm thử các tình huống firewall trên môi trường ảo hóa **Oracle VirtualBox**.

> Link youtube quá trình thực hiện bài lab:
> https://www.youtube.com/watch?v=ZVVWA2FV0lw 

## Nội dung bài lab

- Cài đặt pfSense từ ISO offline, kiểm tra toàn vẹn file bằng SHA-256.
- Cấu hình 3 interface: **WAN** (Bridged), **LAN** (Host-only), **DMZ** (Internal Network `dmz-net`).
- Dựng Windows Server làm **Domain Controller** (AD DS + DNS Forwarder) và **DMZ-Web** (IIS).
- Kiểm tra Outbound NAT tự động/Hybrid cho LAN và DMZ.
- Vô hiệu hóa rule *Default allow LAN to any*, giữ *Anti-Lockout Rule*, tạo rule nền tảng và kiểm chứng bằng Reset States.
- Thực hiện 3 tình huống firewall (xem bên dưới) và trả lời 6 câu hỏi lý thuyết.

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

Lưu ý: WAN không được chồng lấn LAN `10.0.0.0/8` hoặc DMZ `172.16.0.0/16`; không dùng NAT mặc định `10.0.2.0/24` của VirtualBox cho WAN.

## Các tình huống đã thực hiện

| # | Tình huống | Ý tưởng chính | Kết quả mong đợi |
| --- | --- | --- | --- |
| 1 | Chặn ICMP nhưng vẫn cho Web/DNS | Block ICMP đặt trên Pass DNS (53) và Pass HTTP/HTTPS (80/443) | `ping` thất bại; DNS và HTTPS thành công |
| 2 | Chỉ cho một host ra Internet | Pass `10.0.0.2` → Any đặt trên Block `LAN net` → Any | DC ping được; LAN-Test không ping được |
| 3 | Cô lập DMZ khỏi LAN | Block `DMZ net` → `LAN net` đặt trên Pass `DMZ net` → Any | DMZ → DC bị chặn; DMZ vẫn ra Internet |

Tình huống 3 có kiểm thử **baseline** (ping thành công) trước khi thêm rule Block để chứng minh chính pfSense là thành phần chặn, không phải Windows Firewall.

## Bài học chính

1. pfSense xử lý rule **từ trên xuống, khớp rule đầu tiên thì dừng** – thứ tự rule quyết định kết quả.
2. pfSense là **stateful firewall** – phải *Reset States* và dùng lệnh kiểm thử mới thì kết luận mới đúng.
3. **NAT** quyết định đi bằng địa chỉ nào, **firewall rule** quyết định có được đi hay không; DMZ cần rule Pass mới ra Internet dù đã có NAT.
4. Cần có baseline trước khi chặn để xác định đúng nguyên nhân lỗi.
5. Tách dịch vụ công khai vào DMZ giúp hạn chế lây lan ngang vào mạng nội bộ.

## Cấu trúc báo cáo

| Phần | Nội dung |
| --- | --- |
| Bìa, Mục lục | Thông tin sinh viên, mục lục tự động |
| A. Tổng quan | Mục tiêu, mô hình mạng, bảng IP, tài nguyên máy ảo |
| B. Thực hành | 9 mục từ tạo VM pfSense đến kiểm tra rule nền tảng |
| C. Tình huống | Tình huống 1, 2, 3 (cấu hình, kiểm thử, giải thích) |
| D. Trả lời câu hỏi | 6 câu hỏi lý thuyết |
| E. Kết luận | Tổng kết kết quả đạt được |
| Tài liệu tham khảo | Netgate, Microsoft Learn |
