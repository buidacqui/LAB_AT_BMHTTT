# LAB3 — Nhận diện và ứng phó các mối đe dọa đến an toàn thông tin

## 2. Phiên bản môi trường thực hành

| Thành phần | Phiên bản |
|---|---|
| Ảo hóa | VMware Workstation Pro 26H1 |
| Máy ảo | Windows 11 25H2 x64, OS Build 26200.9445 (KB5124008) |
| Endpoint | Microsoft Defender Antivirus (Real-time + Tamper Protection bật) |
| Shell | Windows PowerShell 5.1 |
| Sysmon | 15.22, schema 4.90 |
| Autoruns | 14.3 |
| Process Explorer | 17.14 |
| Wireshark | 4.6.8 Stable + Npcap |
| Python | 3.14.7 |
| SHA-256 gói dữ liệu lab | `96236f95ce59d0cc37f9b7ab8fbb04522f21870cd0b53aad6e52da5a23655439` |

> Nếu máy bạn dùng phiên bản khác với bảng chuẩn ở trên, ghi rõ phiên bản thực tế đã dùng và lý do (ví dụ: đã báo giảng viên và được chấp thuận).

## 3. Cách dựng môi trường (tóm tắt các bước đã thực hiện)

1. Tạo VM Windows 11 25H2 x64 trên VMware Workstation Pro 26H1, Network Adapter = Host-only; cập nhật KB5124008; tạo snapshot `LAB3_CLEAN_20260914`.
2. Tạo cấu trúc thư mục `C:\LAB3\{Evidence,Tools,Downloads,Assets}` bằng PowerShell (Administrator).
3. Sao chép `LAB3_Threats_Assets.zip` vào `C:\LAB3\Downloads`, kiểm tra `Get-FileHash -Algorithm SHA256` khớp với hash công bố, sau đó `Expand-Archive` vào `C:\LAB3`.
4. Cài Python 3.14.7 và Wireshark 4.6.8 (kèm Npcap) qua `winget`, xác nhận phiên bản.
5. Tải Sysmon, Autoruns, Process Explorer từ `download.sysinternals.com`, giải nén vào `C:\LAB3\Tools`, xác nhận version từng công cụ.
6. Chạy baseline hệ thống (OS, Defender, Firewall, network, process list) trước khi tạo bất kỳ tình huống thử nghiệm nào.

[ĐIỀN: mô tả chi tiết hơn nếu có khác biệt so với hướng dẫn, ví dụ tên máy, cấu hình vCPU/RAM/đĩa thực tế]

## 4. Các tình huống đã thực hiện

| Tình huống | Nội dung | Trạng thái |
|---|---|---|
| TH1 | Risk register + phân loại 5 nguồn đe dọa | [PASS/FAIL] |
| TH2 | EICAR — kiểm chứng Defender detection/quarantine | [PASS/FAIL] |
| TH3 | Tấn công mật khẩu — Event 4624/4625/4648, đổi mật khẩu | [PASS/FAIL] |
| TH4 | Backdoor — persistence lành tính + listener 127.0.0.1:8080 | [PASS/FAIL] |
| TH5 | Sniffing — capture HTTP loopback vs TLS/443 | [PASS/FAIL] |
| TH6 | DoS (local_load_test) + phân tích DDoS/mail bombing offline | [PASS/FAIL] |
| TH7 | Phishing/Social Engineering — phân tích mẫu offline | [PASS/FAIL] |
| Cleanup | Gỡ artefact, so sánh Autoruns before/after, hash Evidence, revert snapshot | [PASS/FAIL] |

## 5. Lỗi gặp phải và cách khắc phục

| # | Lỗi gặp phải | Nguyên nhân | Cách khắc phục |
|---|---|---|---|
| 1 | [ĐIỀN] | [ĐIỀN] | [ĐIỀN] |
| 2 | [ĐIỀN] | [ĐIỀN] | [ĐIỀN] |

> Nếu không gặp lỗi nào, ghi: "Không phát sinh lỗi trong quá trình thực hiện."

## 6. Cấu trúc thư mục LAB3/ trong repo này

```
LAB3/
├── README.md                          (tệp này)
├── [MãLớp]-LAB3_MSSV-HoTen.docx       (báo cáo chính, có ảnh H1–H11 đã chèn)
├── evidence_sha256.csv                 (hash SHA-256 của toàn bộ Evidence, xuất từ VM)
└── Evidence/
    ├── baseline_os.txt
    ├── baseline_defender.txt
    ├── baseline_firewall.txt
    ├── baseline_network.txt
    ├── baseline_processes.txt
    ├── defender_eicar.txt
    ├── auth_events_before_rotation.txt
    ├── sysmon_persistence.txt
    ├── autoruns_before.csv
    ├── autoruns_after.csv
    ├── autoruns_diff.txt
    ├── local_load_test.txt
    ├── ddos_sources.txt
    └── mail_sender_counts.txt / mail_volume.txt
```

**Lưu ý khi đẩy log lên repo (bắt buộc theo đề bài):**
- Đã rà soát và xóa/thay thế mọi mật khẩu, token, cookie/session, email thật, tên máy hoặc thông tin có thể định danh hệ thống thật trước khi commit.
- Không đưa file cài đặt/executable (Sysmon.zip, Autoruns64.exe, procexp64.exe, Wireshark installer, Python installer) vào repo.
- Không đưa file bị Defender quarantine vào repo.
- `local_load_test.py` được chạy nguyên bản, không chỉnh sửa mục tiêu khỏi `127.0.0.1:8080`.
- Không có traffic DDoS/mail bombing/spoofing/MITM chủ động nào được tạo ra ngoài phạm vi VM lab; các phần DDoS/mail bombing chỉ là phân tích dataset offline có sẵn trong gói lab.

## 7. Kiểm tra trước khi nộp

- [ ] Đã commit/push toàn bộ file lên nhánh `main`.
- [ ] Đã mở URL repository ở cửa sổ trình duyệt **không đăng nhập** để xác nhận quyền truy cập Public.
- [ ] Đã dán URL repository vào Google Classroom.
