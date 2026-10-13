# LAB 6 – Xác thực Active Directory và chính sách truy cập trên pfSense

> Link youtube quá trình thực hiện bài lab:
> https://www.youtube.com/watch?v=ZVVWA2FV0lw 

## Thông tin sinh viên

| Mục | Nội dung |
| --- | --- |
| Họ và tên | Bùi Đắc Quí |
| Mã số sinh viên | 1150080033 |
| Lớp | CNPM1 |
| Học phần | Thực hành An toàn Hệ thống thông tin |
| Tên bài Lab | Lab 6 – Xác thực Active Directory và chính sách truy cập trên pfSense |

## Nội dung đã thực hiện

Dựng một pfSense riêng cho LAB 6 trên VirtualBox, tích hợp Active Directory qua LDAP, bật Captive Portal và thiết kế các policy truy cập theo IP/port/schedule.

**Môi trường và cấu hình nền**

1. Tạo các mạng ảo: NAT Network `LAB6-WAN` (192.168.250.0/24), Host-Only (192.168.56.0/24), Internal Network `lab6-lan` và `lab6-dmz`.
2. Dựng **pfSense-LAB6** với 4 card mạng: WAN (Adapter 1), MGMT (Adapter 2), LAN (Adapter 3), DMZ (Adapter 4).
3. Dựng **DC01** (Windows Server, AD DS + DNS, domain `lab6.test`), user `sv_a`, `sv_b` trong OU `LAB6-Users` và service account `svc_pfsense_ldap` (không phải Domain Admin).
4. Dựng **Client-A**, **Client-B** (IP tĩnh) và **DMZ-WEB** (Ubuntu + Nginx, thêm IP .20/.30/.40 mô phỏng nhóm website).

**Tích hợp AD và Captive Portal**

5. Tạo LDAP Authentication Server `LAB6-AD` (Base DN `DC=lab6,DC=test`, Entire Subtree) và kiểm tra `sv_a`, `sv_b` bằng Diagnostics → Authentication.
6. Tạo Captive Portal zone `LAB6_USERS` trên LAN, xác thực bằng backend LAB6-AD; đăng nhập thử bằng `sv_a` (Client-A) và `sv_b` (Client-B).
7. Tạo Alias (`CLIENT_A`, `CLIENT_B`, `POLICY_CLIENTS`, `GENERAL_SITE`, `JOB_SITE`, `SOCIAL_SITE`, `RESTRICTED_SITE`, `WEB_PORTS`) và Schedule (`LUNCH_WINDOW`, `WORK_HOURS`, `TEST_NOW`).

**Kịch bản chính sách đã thực hiện (Kịch bản 1–5)**

| KB | Nội dung | Rule chính |
| --- | --- | --- |
| 1 | Client-A bị chặn JOB_SITE | Block `CLIENT_A` → `JOB_SITE` TCP/80 (Log) đặt trên Pass `CLIENT_A` → `GENERAL_SITE` |
| 2 | Cấm RESTRICTED_SITE cho cả A và B | Block `POLICY_CLIENTS` → `RESTRICTED_SITE` TCP/80 (Log); Pass → `GENERAL_SITE` |
| 3 | Client-B chỉ được SOCIAL_SITE trong giờ nghỉ | Pass `CLIENT_B` → `SOCIAL_SITE` (schedule `LUNCH_WINDOW`) trên Block cùng đối tượng |
| 4 | Chỉ cho HTTP/HTTPS trong giờ hành chính | Pass `POLICY_CLIENTS` → Any `WEB_PORTS` (schedule `WORK_HOURS`) trên Block `POLICY_CLIENTS` → Any |
| 5 | Cô lập DMZ khỏi LAN | Tab DMZ: Block `DMZ net` → `LAN net`; tab LAN: Pass `POLICY_CLIENTS` → `GENERAL_SITE` |

Ngoài ra bài còn gom ruleset tổng hợp 7 rule trên tab LAN (giải thích thứ tự first-match), điền bảng kết quả và trả lời 12 câu hỏi phân tích. Kịch bản 6 (Block/Reject) và Kịch bản 7 (Firewall Log/State Table) **không** nằm trong phạm vi báo cáo này.

## Mô hình mạng

| Thiết bị / interface | Địa chỉ | Vai trò |
| --- | --- | --- |
| pfSense WAN | DHCP (192.168.250.0/24) | Ra Internet qua NAT Network |
| pfSense MGMT | 192.168.56.254/24 | WebGUI cho máy thật (192.168.56.1) |
| pfSense LAN | 10.6.0.1/24 | Gateway LAN-DOMAIN, Captive Portal |
| pfSense DMZ | 172.16.6.1/24 | Gateway DMZ |
| DC01 | 10.6.0.10/24 | AD DS + DNS, domain `lab6.test` |
| Client-A | 10.6.0.21/24 | Đại diện tài khoản `sv_a` |
| Client-B | 10.6.0.22/24 | Đại diện tài khoản `sv_b` |
| DMZ-WEB | 172.16.6.10/24 (+ .20/.30/.40) | Nginx, website mô phỏng |

## Kết quả thực hiện

> Điền theo kết quả thực tế sau khi thực hành (PASS/FAIL hoặc mô tả ngắn).

| Nội dung | Kết quả mong đợi | Kết quả thực tế |
| --- | --- | --- |
| LDAP `sv_a`, `sv_b` | Xác thực thành công cả hai | ........ |
| Captive Portal | Hiện portal, đăng nhập `sv_a`/`sv_b` thành công, có session | ........ |
| KB1 | Client-A vào `.10`; `.20` bị chặn; log block 10.6.0.21 → 172.16.6.20 | ........ |
| KB2 | A và B đều bị chặn `.40`; vẫn vào `.10` | ........ |
| KB3 | B vào `.30` khi schedule active; bị chặn khi ngoài schedule | ........ |
| KB4 | Web OK trong `WORK_HOURS`; ping/dịch vụ khác và ngoài giờ bị chặn | ........ |
| KB5 | DMZ-WEB không kết nối được DC01; Client vào được `.10` | ........ |

**Kết luận chính**

- LDAP/Captive Portal chỉ cung cấp **xác thực**; firewall rule chuẩn của pfSense khớp theo IP/port/interface/schedule, nên người dùng được ánh xạ bằng IP tĩnh (CLIENT_A = 10.6.0.21, CLIENT_B = 10.6.0.22).
- Rule xử lý từ trên xuống, **first-match**; Block đặc hiệu phải đặt trên Pass rộng.
- Rule gắn schedule biến mất khi không active (không tự đảo thành Block), nên cần rule Block phía dưới để chặn ngoài giờ.
- pfSense là stateful: phải tạo kết nối mới hoặc Reset States khi kiểm thử, và cần baseline trước khi Block.
- Mô hình test-site theo IP chỉ mô phỏng nhóm nội dung ở L3/L4, khác Web Category Filtering thật.
