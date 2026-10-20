# LAB 7 – Xây dựng chính sách Web Protection trên mô hình pfSense + Web Proxy

> Link youtube quá trình thực hiện bài lab:
> https://www.youtube.com/watch?v=

## Thông tin sinh viên

| Mục | Nội dung |
| --- | --- |
| Họ và tên | Bùi Đắc Quí |
| Mã số sinh viên | 1150080033 |
| Lớp | CNPM1 |
| Học phần | Thực hành An toàn Hệ thống thông tin |
| Tên bài Lab | Lab 7 – Xây dựng chính sách Web Protection trên mô hình pfSense + Web Proxy |

## Nội dung đã thực hiện

Bài lab chuyển các yêu cầu Web Protection của bài Sophos gốc (chặn/cảnh báo theo nhóm nội dung, giới hạn truy cập, quét virus khi tải file, giới hạn dung lượng, mở khóa tạm thời, HTTPS inspection) sang mô hình **pfSense (firewall/gateway) + WEB-PROXY riêng** chạy **E2Guardian + Squid + ClamAV** trên Ubuntu Server 24.04 LTS, kế thừa môi trường từ Lab 6.

> **Thiết kế bắt buộc:** Netgate đã đánh dấu các package Squid, SquidGuard, Lightsquid trên pfSense là *deprecated* (nhiều lỗ hổng upstream chưa vá). Vì vậy Lab 7 **không cài proxy lên pfSense**: pfSense chỉ kiểm soát luồng ở L3/L4, còn WEB-PROXY lọc nội dung ở L7.

**Luồng Web của bài:**

```
Browser (Client-A/B) → E2Guardian 10.6.0.30:8080 → Squid 127.0.0.1:3128 → pfSense → DMZ-WEB 172.16.6.10
```

**Phần cấu hình nền tảng**

1. Kiểm tra lại các VM kế thừa từ Lab 6 (IP, DNS, đăng nhập pfSense qua MGMT), tạo snapshot **LAB7_START**; giải nén bộ `LAB7_Resources` và kiểm tra toàn vẹn bằng `sha256sum -c SHA256SUMS.txt`.
2. Dựng **WEB-PROXY** (Ubuntu Server 24.04, 2 GB RAM, 2 vCPU, 20 GB disk, Internal Network `lab6-lan`, IP tĩnh `10.6.0.30/24` bằng netplan); cài Squid, E2Guardian, ClamAV bằng script `install_lab7_proxy.sh` và kiểm tra phiên bản/trạng thái dịch vụ. Toàn bộ bước cài đặt được làm **trước** khi áp ruleset pfSense vì rule explicit deny sẽ chặn WEB-PROXY tải gói.
3. Cấu hình **Squid chỉ nghe localhost** (`http_port 127.0.0.1:3128`) để client không thể gọi thẳng Squid bỏ qua E2Guardian; cấu hình **E2Guardian** (`filterip = 10.6.0.30`, `filterports = 8080`, `proxyip = 127.0.0.1`, `proxyport = 3128`, bật `clamdscan`, `enablessl = off` cho phần lõi; `reportinglevel = 3`, `bypass = 300`, `contentscanexceptions = off`).
4. Dựng **website mô phỏng trên DMZ-WEB** bằng script `setup_lab7_dmz_web.sh`: 13 virtual host Nginx `*.lab7.test` và các file test `10MB.bin`, `100MB.bin`, `110MB.bin`, `eicar.com.txt`.
5. Tạo zone DNS **`lab7.test`** trên DC01 bằng `Add-LAB7-DNS.ps1` (13 bản ghi A trỏ `172.16.6.10`), kiểm tra bằng `nslookup`/`Resolve-DnsName` từ Client.
6. Cấu hình **Manual Proxy `10.6.0.30:8080`** trên Client-A và Client-B (chưa bật HTTPS inspection ở giai đoạn này).
7. Trên pfSense: tạo Alias, dựng **ruleset LAN 7 rule** ép Client đi qua proxy (Disable các rule policy của Lab 6 và rule *Default allow LAN to any*), kiểm thử chống bypass bằng Reset States và Firewall Log.

**Ruleset LAN của Lab 7**

| # | Action | Source | Destination | Port | Ý nghĩa |
| --- | --- | --- | --- | --- | --- |
| 1 | Pass | LAN net | 10.6.0.10 | TCP/UDP 53 | DNS về DC01 |
| 2 | Pass | LAB7_CLIENTS | LAB7_PROXY | TCP 8080 | Client dùng E2Guardian |
| 3 | Pass | LAN net | Any | MAIL_PORTS | Cho email theo yêu cầu gốc |
| 4 | Block (Log) | LAB7_CLIENTS | Any | WEB_PORTS | Chặn web trực tiếp, chống bypass |
| 5 | Pass | LAB7_PROXY | Any | WEB_PORTS | Cho proxy đi HTTP/HTTPS |
| 6 | Pass | LAB7_PROXY | 172.16.6.10 | TCP 80 | Cho proxy vào site test DMZ |
| 7 | Block (Log) | LAN net | Any | Any | Explicit deny cho bài thực hành |

Alias sử dụng: `LAB7_CLIENTS` (10.6.0.21, 10.6.0.22), `LAB7_PROXY` (10.6.0.30), `WEB_PORTS` (80, 443), `MAIL_PORTS` (25, 465, 587, 110, 995, 143, 993).

> Rule 1 và 2 chỉ mang tính tài liệu hóa vì Client, WEB-PROXY và DC01 cùng subnet `10.6.0.0/24` nên lưu lượng được chuyển mạch lớp 2, không đi qua pfSense. Rule thực sự có tác dụng với lưu lượng ra khỏi subnet là **4, 5, 6, 7**; việc bắt buộc dùng proxy dựa trên rule 4.

**Chính sách Web (E2Guardian)**

| Nhóm | Domain mô phỏng | Cơ chế |
| --- | --- | --- |
| Hard block (không bypass) | `china`, `mideast`, `jobs`, `blockedgame` | `bannedsitelist` + non-bypass list |
| Warning / temporary bypass 300 giây | `weapons`, `politics`, `religion`, `streaming` | `bannedsitelist`, không nằm trong non-bypass list, `bypass = 300` |
| Exception (luôn cho phép) | `normal`, `education`, `download` | `exceptionsitelist` (`download` vẫn bị quét AV/size) |
| Time-window 60 phút | `entertainment`, `games` | `exceptionsitelist` + Schedule `LAB7_ENT_60M` trên pfSense |
| Quét virus | `download.lab7.test/files/eicar.com.txt` | E2Guardian + ClamAV (`clamdscan`) |
| Giới hạn > 100 MB | `100MB.bin`, `110MB.bin` | Squid `reply_body_max_size 104857600` (tùy build) |

**Điều chỉnh khi thực hiện**

- *Entertainment/Game theo thời gian:* 13 FQDN cùng trỏ `172.16.6.10` nên firewall không phân biệt được site. Em gán IP phụ **`172.16.6.50`** trên DMZ-WEB cho `entertainment` và `games`, cập nhật bản ghi A trên DC01, tạo alias `LAB7_ENT_IP` + schedule `LAB7_ENT_60M`, rồi thêm rule **4a** (Pass proxy → `LAB7_ENT_IP` TCP 80, có schedule) và **4b** (Block + Log, không schedule) đặt dưới rule 4 và trên rule 5.
- *Trang block tùy biến:* `custom_block_template.html` không chứa các biến E2Guardian (URL, lý do chặn, liên kết bypass) nên em chỉ chỉnh tiêu đề/màu sắc trên `template.html` mặc định, giữ nguyên biến để trang warning vẫn có liên kết bypass.

**Phần mở rộng – HTTPS inspection (nếu lớp yêu cầu)**

Tạo CA lab 4096 bit bằng `generate_lab7_ca.sh` (CN `LAB7 E2Guardian Root CA`, hiệu lực 3650 ngày), bật `enablessl` và `sslmitm = on`, kiểm thử *trước khi import CA* (lỗi `NET::ERR_CERT_AUTHORITY_INVALID`) và *sau khi import `ca.der` vào Trusted Root* (chứng chỉ do CA lab ký, EICAR qua HTTPS bị chặn). Chỉ chuyển file công khai `ca.der`, không chuyển `ca.key`; gỡ CA khỏi Client và gỡ rule DNS tạm của DC01 sau khi xong.

Ngoài ra bài còn trả lời **10 câu hỏi phân tích** (vì sao không cài Squid trên pfSense, L3/L4 và L7, block direct 80/443, lọc theo IP, hard block và bypass, EICAR qua HTTPS, rủi ro lộ private key CA, `maxcontentfilecachescansize`, schedule và quota, giả thuyết khi E2Guardian không có log).

## Mô hình mạng

| Thiết bị | Interface | IP | Gateway | DNS |
| --- | --- | --- | --- | --- |
| pfSense-LAB6 | MGMT | 192.168.56.254 | — | — |
| pfSense-LAB6 | LAN | 10.6.0.1/24 | — | — |
| pfSense-LAB6 | DMZ | 172.16.6.1/24 | — | — |
| Máy thật quản trị | Host-only (MGMT) | 192.168.56.1/24 | — | — |
| DC01 (AD/DNS) | LAN | 10.6.0.10/24 | 10.6.0.1 | 10.6.0.10 |
| Client-A | LAN | 10.6.0.21/24 | 10.6.0.1 | 10.6.0.10 |
| Client-B | LAN | 10.6.0.22/24 | 10.6.0.1 | 10.6.0.10 |
| **WEB-PROXY (mới)** | LAN | 10.6.0.30/24 | 10.6.0.1 | 10.6.0.10 |
| DMZ-WEB | DMZ | 172.16.6.10/24 (+ 172.16.6.50 cho entertainment/games) | 172.16.6.1 | — |

| Dịch vụ trên WEB-PROXY | Địa chỉ nghe | Vai trò |
| --- | --- | --- |
| E2Guardian | 10.6.0.30:8080 | Điểm proxy của client; URL/content policy, warning, AV |
| Squid | 127.0.0.1:3128 | Upstream forward proxy, chỉ localhost |
| ClamAV daemon | Local socket | Quét đối tượng do E2Guardian chuyển tới |

## Kết quả thực hiện

| # | Kịch bản | Kiểm thử | Kết quả |
| --- | --- | --- | --- |
| 1 | Baseline | `curl.exe -x http://10.6.0.30:8080 -I http://normal.lab7.test` (proxy bật) | Thành công |
| 1 | Chống bypass | `curl.exe --noproxy "*" -m 5 -I ...`, `Test-NetConnection 172.16.6.10 -Port 80` và `8.8.8.8 -Port 443` (proxy tắt) | Thất bại / timeout (rule 4 chặn, có Firewall Log) |
| 1 | WEB-PROXY ra DMZ | `curl` từ WEB-PROXY tới `normal.lab7.test` | HTTP 200 (rule 5/6) |
| 2 | Geo-Sim | `china.lab7.test`, `mideast.lab7.test` | Hard block, không có liên kết bypass |
| 3 | Job Search | `jobs.lab7.test` | Hard block |
| 4 | Warning content | `weapons`, `politics`, `religion` | Trang cảnh báo; bypass 5 phút rồi bị chặn lại |
| 5 | Education | `education.lab7.test` | Truy cập bình thường, không cảnh báo |
| 6 | Entertainment/Game | `entertainment`, `games` trong khung schedule | Truy cập được |
| 6 | Entertainment/Game | Cùng lệnh, ngoài khung schedule (Reset States) | Bị chặn bởi rule 4b (timeout, không có trang block) |
| 7 | Streaming | `streaming.lab7.test` | Cảnh báo, bypass tạm được |
| 8 | Specific game | `blockedgame.lab7.test` | Hard block |
| 9 | Antivirus | Tải `eicar.com.txt` qua proxy | Bị chặn bởi lớp AV (ClamAV nhận diện EICAR) |
| 10 | Size limit | Tải `10MB.bin`, `100MB.bin`, `110MB.bin` | Chỉ đạt khi 110MB bị từ chối và 100MB xử lý đúng cấu hình; nếu cả hai tải được thì **không kết luận** |
| Mở rộng | HTTPS inspection | Trước / sau khi import CA lab | Lỗi chứng chỉ → truy cập được, chứng chỉ do `LAB7 E2Guardian Root CA` ký |

Đối chiếu log sau mỗi kịch bản: **Firewall log pfSense** (luồng có qua firewall không), **E2Guardian `access.log`** (category/lý do block), **Squid `access.log`** (site hard block thường không tới Squid).

**Kết luận chính:**

- Firewall L3/L4 và proxy L7 bổ sung cho nhau: pfSense bắt buộc đường đi (chặn direct TCP/80,443, chỉ WEB-PROXY được ra web), E2Guardian hiểu hostname/nội dung để áp chính sách.
- Lọc theo IP không đủ cho web thật: cả 13 FQDN `.lab7.test` cùng IP `172.16.6.10` nên chỉ proxy dựa vào hostname mới phân biệt được.
- Lưu lượng cùng subnet không đi qua pfSense, nên phải xác định rule nào thực sự có tác dụng (rule 4, 5, 6, 7), không phải rule 1, 2.
- Hard block (non-bypass list) khác warning (bypass = 300 giây, có log); hai nhóm khác nhau ở việc domain có nằm trong non-bypass list hay không.
- Schedule 60 phút chỉ là **time-window policy**, không phải daily category quota như Sophos.
- `maxcontentfilecachescansize` không phải giới hạn dung lượng tải; yêu cầu chặn > 100 MB phải dùng cơ chế riêng (Squid `reply_body_max_size`) và chỉ kết luận khi 110MB bị từ chối còn 100MB xử lý đúng cấu hình.
- Không có SSL MITM thì proxy chỉ thấy `CONNECT host:port`, không quét được EICAR qua HTTPS; MITM đòi hỏi client tin cậy CA lab, và nếu lộ private key CA thì kẻ tấn công có thể giả mạo bất kỳ website nào, nên chỉ dùng trong lab cô lập.
- pfSense là stateful firewall: sau khi đổi rule phải Apply Changes và Reset States trước khi kiểm thử lại.
