# LAB 4 – Khảo sát và đánh giá bề mặt mạng bằng Nmap

Môn: **An toàn hệ thống thông tin**
Nội dung: thực hành Nmap trong mạng ảo VirtualBox Host-Only (Kali Linux → Metasploitable 2) và viết báo cáo kỹ thuật có bằng chứng.


## 1. Nội dung repo

| Tệp | Mô tả |
|---|---|
| `LAB4_Nmap_HuongDan_2026.pdf` | Tài liệu hướng dẫn gốc của giảng viên (18 trang) |
| `LAB4_Nmap_BaoCao.docx` | Báo cáo Word: có sẵn khung, bảng, phần phân tích và trả lời câu hỏi |
| `README.md` | Tệp này |
| `evidence/` *(tự tạo)* | Ảnh minh chứng và các tệp kết quả quét (`.txt`, `.xml`, `.gnmap`, `.html`) |

Gợi ý cấu trúc thư mục `evidence/`:

```
evidence/
├── screenshots/     # anh1_ip_kali.png, anh2_ip_msf2.png, ...
├── scans/           # ket_qua.txt, ket_qua.xml, smb.txt, before.txt, after.txt, lab4_full.*
└── bao_cao.html     # xuất từ xsltproc
```

---

## 2. Mô hình mạng

```
                 VirtualBox Host-Only Network 192.168.56.0/24
                                   │
        ┌──────────────────────────┼──────────────────────────┐
        │                          │                          │
  Máy thật (Windows)         VM1 – Kali Linux          VM2 – Metasploitable 2
  192.168.56.1               Máy quét                  Máy đích (cố ý có lỗ hổng)
  VirtualBox + Nmap          192.168.56.10             192.168.56.101
```

Các IP ở trên là giá trị dự kiến theo tài liệu. **IP thực tế phải được xác nhận bằng lệnh** (`ip -br addr`, `ifconfig`) và ghi vào báo cáo.

---

## 3. Yêu cầu môi trường

| Thành phần | Khuyến nghị |
|---|---|
| Máy thật | Windows 10/11 64-bit, quyền Administrator |
| RAM | Tối thiểu 8 GB (khuyến nghị 16 GB); Kali 2–4 GB, Metasploitable 2 1 GB |
| Ổ đĩa trống | 25–40 GB (gồm VM và snapshot) |
| Ảo hóa | Bật Intel VT-x / AMD-V trong BIOS/UEFI |
| VirtualBox | Bản hiện hành (không cần Extension Pack) |
| Máy quét | Kali Linux VM |
| Máy đích | Metasploitable 2 (chỉ nối Host-Only, **không Bridge**) |

---

## 4. Chuẩn bị

1. Cài **Nmap + Npcap + Zenmap** trên Windows từ trang chính thức `nmap.org/download.html` (chạy *Run as administrator*, giữ tùy chọn *Register Nmap Path*). Mở lại Terminal rồi kiểm tra `nmap --version`.
2. Kali: cập nhật và cài `nmap` (bật NAT tạm thời), kiểm tra phiên bản, sau đó **tắt VM và ngắt NAT**, chỉ giữ Host-Only.
3. VirtualBox: tạo Host-Only Network `192.168.56.0/24` (có thể bật DHCP).
4. Tạo snapshot **`Before-LAB4`** cho từng VM.
5. Từ Kali, `ping -c 4 <IP Metasploitable 2>` phải thành công **trước khi** quét. Nếu ICMP bị chặn nhưng IP đúng thì dùng `-Pn`.

---

## 5. Các bước thực hành (tham chiếu nhanh)

| # | Nhiệm vụ | Lệnh chính (trên Kali) |
|---|---|---|
| 1 | Host discovery | `sudo nmap -sn 192.168.56.0/24` |
| 2 | TCP Connect scan | `nmap -sT 192.168.56.101` |
| 3 | SYN scan | `sudo nmap -sS 192.168.56.101` |
| 4 | FIN / Xmas / NULL | `sudo nmap -sF` / `-sX` / `-sN 192.168.56.101` |
| 5 | ACK scan | `sudo nmap -sA 192.168.56.101` |
| 6 | UDP (20 cổng phổ biến) | `sudo nmap -sU --top-ports 20 192.168.56.101` |
| 7 | Nhận diện dịch vụ | `sudo nmap -sV 192.168.56.101` |
| 8 | Nhận diện OS | `sudo nmap -O 192.168.56.101` |
| 9 | Aggressive | `sudo nmap -A 192.168.56.101` |
| 10 | NSE – SMB | `--script smb-os-discovery`, `--script smb-vuln-ms17-010` với `-p 445` |
| 11 | Xuất kết quả | `-oN`, `-oX`, `-oG`, `-oA`; `xsltproc ket_qua.xml -o bao_cao.html` |
| 12 | Before/After hardening | Chạy cùng một lệnh `-sV` trước và sau khi thay đổi phòng thủ |

Lưu ý:
- Với `-oG`, phải chạy lệnh nmap tạo tệp **trước**, sau đó mới `grep "445/open" smb.txt`.
- FIN/Xmas/NULL và NSE chỉ chạy vào Metasploitable 2 hoặc Windows VM của chính mình.
- Trạng thái `open|filtered` **không** đồng nghĩa với `open`.
- NSE báo timeout/không xác định **không** có nghĩa là đã vá. Chỉ kết luận "có dấu hiệu dễ bị ảnh hưởng" khi script báo `VULNERABLE`.

---

## 6. Quy tắc làm bài

- **Tự gõ lệnh** và chụp màn hình sau khi lệnh chạy (không copy/paste từ chatbot hoặc web). Tự sửa lỗi nếu gõ sai và chụp lại.
- Không giả định IP máy đích; phải phát hiện và chứng minh bằng kết quả quét.
- Không dùng IP Wi-Fi/Ethernet thật để quét.
- Khôi phục snapshot nếu thay đổi hardening làm hỏng môi trường.

---

## 7. Cách hoàn thiện báo cáo `LAB4_Nmap_BaoCao.docx`

1. Điền thông tin sinh viên ở trang bìa.
2. Thay các ô màu đỏ dạng `[điền]` bằng kết quả thực tế trên máy của bạn (IP, số cổng, version, MAC/Vendor, thời gian quét…).
3. Dán ảnh chụp màn hình vào các khung *"Dán ảnh chụp màn hình tại đây"* (theo thứ tự Hình 2.1 → 13.3).
4. Đối chiếu phần phân tích/trả lời câu hỏi với output của bạn; chỉnh lại nếu kết quả thực tế khác.
5. Với phần CVE (Bài 3, bản đồ dịch vụ): dùng phiên bản dịch vụ do Nmap phát hiện và tra cứu tại `nvd.nist.gov` trước khi ghi.
6. Xóa các dòng ghi chú không cần thiết, kiểm tra lại trước khi nộp.

### Danh mục ảnh minh chứng bắt buộc

- [ ] Ảnh 1: `ip -br addr` trên Kali (thấy interface Host-Only và IP)
- [ ] Ảnh 2: `ifconfig`/`ip` trên Metasploitable 2
- [ ] Ảnh 3: kết quả host discovery `-sn`
- [ ] Ảnh 4: kết quả `-sS` hoặc `-sT`
- [ ] Ảnh 5: kết quả `-sV`
- [ ] Ảnh 6: kết quả `-O` hoặc `-A`
- [ ] Ảnh 7: một NSE script và kết luận dựa trên output
- [ ] Ảnh 8: tệp kết quả đã lưu (`.txt`/`.xml`) và/hoặc HTML

---

## 8. Xử lý sự cố thường gặp

| Hiện tượng | Cách xử lý |
|---|---|
| `nmap is not recognized` (Windows) | Đóng/mở lại Terminal hoặc cài lại với tùy chọn *Register Nmap Path* |
| Ping tới Metasploitable 2 thất bại | Kiểm tra cùng Host-Only Network, subnet mask, trùng IP, máy đích đã bật, firewall chặn ICMP (dùng `-Pn`) |
| `sudo nmap ...` báo cần quyền | Dùng `sudo` hoặc quyền root cho các kỹ thuật raw (`-sS`, `-sU`, `-O`, …) |
| `grep: smb.txt: No such file` | Chạy lệnh `nmap ... -oG smb.txt` trước khi `grep` |
| Zenmap không mở | Không phải lỗi Nmap; kiểm tra DISPLAY/GTK hoặc làm bằng Terminal |
| Thấy dư host khi `-sn` | Thường là adapter Host-Only của máy thật (`.1`) hoặc DHCP server của VirtualBox |

---

## 9. Tài liệu tham khảo

1. Nmap Project – Download: <https://nmap.org/download.html>
2. Nmap – Windows: <https://nmap.org/book/inst-windows.html>
3. Nmap Reference Guide: <https://nmap.org/book/man.html>
4. Kali Linux Tools – nmap: <https://www.kali.org/tools/nmap/>
5. Kali inside VirtualBox: <https://www.kali.org/docs/virtualization/install-virtualbox-guest-vm/>
6. Rapid7 – Metasploitable 2
7. Microsoft Security Bulletin MS17-010
