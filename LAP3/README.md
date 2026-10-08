# LAB 3 – Nhận diện và ứng phó các mối đe dọa đến an toàn thông tin

> Link youtube quá trình thực hiện bài lab:
> https://youtu.be/Wj-AW22pnto  

## Thông tin sinh viên

| Mục | Nội dung |
| --- | --- |
| Họ và tên | Bùi Đắc Quí |
| Mã số sinh viên | 1150080033 |
| Lớp | CNPM1 |
| Học phần | Thực hành An toàn Hệ thống thông tin |
| Tên bài Lab | Lab 3 – Nhận diện và ứng phó các mối đe dọa đến an toàn thông tin (Identifying and Responding to Information Security Threats) |

## Phiên bản môi trường

| Thành phần | Phiên bản chuẩn của bài lab | Phiên bản thực tế của em |
| --- | --- | --- |
| Ảo hóa | VMware Workstation Pro 26H1 (mạng Host-only) | ........ |
| Máy ảo | Windows 11 25H2 x64, OS build 26200.9445 (KB5124008) | ........ |
| Endpoint protection | Microsoft Defender Antivirus (Real-time + Tamper Protection bật) | ........ |
| Shell | Windows PowerShell 5.1 | ........ |
| Sysmon | 15.22 (schema 4.90) | ........ |
| Autoruns | 14.3 | ........ |
| Process Explorer | 17.14 | ........ |
| Wireshark | 4.6.8 Stable + Npcap | ........ |
| Python | 3.14.7 | ........ |
| Gói dữ liệu | `LAB3_Threats_Assets.zip` – SHA-256 `96236f95ce59d0cc37f9b7ab8fbb04522f21870cd0b53aad6e52da5a23655439` | ........ |
| Snapshot sạch | `LAB3_CLEAN_20260914` | ........ |

## Cách dựng môi trường

1. Tạo VM Windows 11 25H2 x64 (khuyến nghị 2 vCPU, 6 GB RAM, 64 GB đĩa); chọn VM → Settings → Network Adapter → **Host-only**.
2. Cập nhật KB5124008 rồi tạo snapshot `LAB3_CLEAN_20260914`.
3. Tạo cấu trúc thư mục bằng PowerShell (Administrator):
   ```powershell
   $Lab = 'C:\LAB3'
   New-Item -ItemType Directory -Force "$Lab\Evidence", "$Lab\Tools", "$Lab\Downloads", "$Lab\Assets" | Out-Null
   ```
4. Chép `LAB3_Threats_Assets.zip` vào `C:\LAB3\Downloads`, kiểm tra SHA-256 (phải trùng giá trị ở bảng trên) rồi giải nén:
   ```powershell
   Get-FileHash C:\LAB3\Downloads\LAB3_Threats_Assets.zip -Algorithm SHA256
   Expand-Archive C:\LAB3\Downloads\LAB3_Threats_Assets.zip -DestinationPath C:\LAB3 -Force
   ```
5. Cài Python 3.14.7 và Wireshark 4.6.8 (giữ Npcap) bằng `winget`.
6. Tải Sysmon, Autoruns, Process Explorer **chỉ** từ `download.sysinternals.com`, giải nén vào `C:\LAB3\Tools` và kiểm tra phiên bản.
7. Chạy khối lệnh **Baseline** (OS, Defender, firewall, mạng, tiến trình) trước khi tạo bất kỳ tình huống nào.

> Quy ước: Defender/Tamper Protection luôn bật, không tạo exclusion; traffic gây tải chỉ tới `127.0.0.1:8080`; không dùng tài khoản, mật khẩu hay dữ liệu thật.

## Các tình huống đã thực hiện

| TH | Nội dung | Công cụ / bằng chứng chính |
| --- | --- | --- |
| Baseline | Thu trạng thái tham chiếu của máy | PowerShell, Get-MpComputerStatus, Get-NetFirewallProfile |
| TH1 | Risk register (Asset → Vulnerability → Threat → Risk → Control) và phân loại 5 nguồn đe dọa | Bảng risk register, bảng phân loại |
| TH2 | Mã độc: kiểm chứng phát hiện/quarantine bằng EICAR | Windows Security (Protection history), Get-MpThreatDetection |
| TH3 | Tấn công mật khẩu và keylogging: tài khoản `lab3user`, đăng nhập đúng/sai, đổi mật khẩu | Event Viewer 4624/4625/4648, auditpol, runas |
| TH4 | Backdoor: persistence lành tính `LAB3_Run_Demo`, `LAB3_Persistence_Demo` và listener `127.0.0.1:8080` | Sysmon, Autoruns, Process Explorer, Get-NetTCPConnection |
| TH5 | Sniffing/MITM/Spoofing: so sánh HTTP loopback và HTTPS/TLS | Wireshark (filter `http.request`, `tls`) |
| TH6 | DoS cục bộ giới hạn, phân tích dataset DDoS (TEST-NET), phân tích log mail bombing offline | `local_load_test.py`, `ddos_sample.csv`, `mailbomb_sample.csv` |
| TH7 | Social Engineering/Phishing/Spear phishing: phân tích mẫu offline | `phishing_email.txt`, `social_engineering_cases.csv` |
| Cleanup | Xóa artefact LAB3, so sánh Autoruns, hash bằng chứng, revert snapshot | Autoruns diff, `evidence_sha256.csv` |

Ngoài ra báo cáo trả lời 20 câu hỏi (khái niệm, EICAR, brute force/dictionary/keylogger, Event ID, backdoor/persistence, DoS/DDoS, mail bombing, sniffing/MITM/spoofing, social engineering, incident response, SHA-256, defense-in-depth).

## Kết quả PASS/FAIL

> Điền theo kết quả thực tế của em sau khi thực hành.

| TH | Tiêu chí PASS (rút gọn) | Kết quả | Ghi chú |
| --- | --- | --- | --- |
| TH1 | Risk register ≥ 5 tài sản/nguy cơ; phân loại đủ 5 tình huống kèm giải thích | PASS / FAIL | ........ |
| TH2 | RealTimeProtectionEnabled = True; có detection EICAR; không tắt bảo vệ, không tạo exclusion | PASS / FAIL | ........ |
| TH3 | Có 4624/4648 (hợp lệ), 4625 (sai); mật khẩu cũ không còn dùng được sau khi đổi | PASS / FAIL | ........ |
| TH4 | Phát hiện `LAB3_Run_Demo` và `LAB3_Persistence_Demo`; listener 8080 chỉ ở 127.0.0.1 và ánh xạ đúng PID | PASS / FAIL | ........ |
| TH5 | Có capture HTTP loopback đọc được chuỗi huấn luyện và capture TLS/443; không can thiệp chủ động | PASS / FAIL | ........ |
| TH6 | Không có traffic ra ngoài; chỉ nhắm 127.0.0.1; phân biệt DoS/DDoS; xác định sender/volume bất thường | PASS / FAIL | ........ |
| TH7 | Nhận diện ≥ 5 chỉ dấu phishing; phân loại đúng 6 case Social Engineering | PASS / FAIL | ........ |
| Cleanup | Artefact đã loại bỏ; Defender vẫn bật; bằng chứng đã hash | PASS / FAIL | ........ |

## Lỗi gặp phải và cách khắc phục

> Xóa các dòng không gặp và bổ sung lỗi thực tế của em.

| Lỗi / hiện tượng | Nguyên nhân | Cách khắc phục |
| --- | --- | --- |
| `Set-Content` báo lỗi khi tạo file EICAR | Defender chặn ngay khi ghi tệp | Kết quả mong đợi; dùng `eicar_write_error.txt` và Protection history làm bằng chứng |
| Wireshark không có interface loopback | Npcap chưa cài hoặc thiếu hỗ trợ loopback | Cài lại Wireshark/Npcap; chọn *Adapter for loopback traffic capture* |
| Cổng 8080 đã bị chiếm | Còn tiến trình `python` cũ | Tìm PID bằng `Get-NetTCPConnection -LocalPort 8080` rồi dừng tiến trình |
| `Get-WinEvent` không thấy log Sysmon | Sai tên log hoặc Sysmon chưa cài | Dùng tên `Microsoft-Windows-Sysmon/Operational`; kiểm tra `Sysmon64.exe -c` |
| Không thấy event 4624/4625 của `lab3user` | Chưa bật audit Logon hoặc lọc sai thời gian | Chạy lại `auditpol`, tăng `StartTime`, lọc `Message` chứa `lab3user` |
| ........ | ........ | ........ |

## Cấu trúc thư mục

```
LAB3/
├── README.md
├── MaLop-LAB3_MSSV-HoTen.docx     # Báo cáo (Word)
├── evidence/                      # Output/log đã làm sạch (txt, csv)
│   └── evidence_sha256.csv        # SHA-256 của toàn bộ bằng chứng
└── images/                        # Ảnh chụp từ VM (H1 … H11)
```

> Điều chỉnh cho khớp với các file em thực sự tải lên.

## Lưu ý để giảng viên kiểm tra / chạy lại

- Mọi ảnh chụp lấy trực tiếp từ VM của em; output và log khớp timestamp của bài. Hash của từng file bằng chứng nằm trong `evidence/evidence_sha256.csv`.
- **Không** đưa vào repo: installer, file thực thi Sysinternals/Wireshark/Python, tệp bị Defender quarantine.
- **Không** đưa vào repo: mật khẩu, token/API key, cookie/session, dữ liệu cá nhân, email thật, log chưa làm sạch hoặc thông tin định danh hệ thống thật. Mật khẩu của `lab3user` chỉ dùng trong lab và không được ghi lại.
- Không chỉnh `local_load_test.py` (hard-code `127.0.0.1:8080`); không thực hiện mail bomb, DDoS, spoofing hay MITM chủ động ngoài VM lab.
- Để chạy lại: revert VM về snapshot `LAB3_CLEAN_20260914`, làm lại từ bước dựng môi trường và baseline, sau đó thực hiện lần lượt TH1 → TH7 và cleanup theo báo cáo.
- Repository `LAB_AT_BMHTTT` ở chế độ Public; đã commit/push đầy đủ và kiểm tra bằng cửa sổ trình duyệt không đăng nhập trước khi nộp.

## Tài liệu tham khảo

- Microsoft Sysinternals – Sysmon, Autoruns, Process Explorer – https://learn.microsoft.com/sysinternals/
- Wireshark User's Guide – https://www.wireshark.org/docs/wsug_html/
- Microsoft Learn – Validate Microsoft Defender Antivirus with the EICAR test file – https://learn.microsoft.com/defender-endpoint/validate-antimalware
- Microsoft Learn – Audit Logon events 4624, 4625, 4648
- IETF RFC 5737 (TEST-NET) và RFC 2606 (tên miền dành riêng)
