# LAB 3 – Thiết lập mô hình tường lửa pfSense

## 1. Thông tin sinh viên

| Mục | Nội dung |
| --- | --- |
| Họ và tên | `Trần Văn Hiệp` |
| MSSV | `1150080051` |
| Lớp / Học phần | `11_ĐH_THMT – An toàn hệ thống thông tin` |
| Tên lab | Thực hành an toàn hệ thống thông tin – Thiết lập mô hình tường lửa pfSense |
| Ngày thực hiện | 06/10/2026 |

## 2. Phiên bản môi trường

| Thành phần | Phiên bản / Cấu hình |
| --- | --- |
| Phần mềm ảo hóa | VMware Workstation (đề gốc viết cho VirtualBox, đã ánh xạ tương đương) |
| Firewall | pfSense CE 2.7.2-RELEASE (amd64), 2 GB RAM, ổ đĩa 10 GB (ZFS) |
| Domain Controller | Windows Server 2022 (hostname `DC01`), 2 GB RAM, AD DS + DNS |
| DMZ-Web | Windows Server 2019/2022 + IIS, 2 GB RAM (dùng cho Tình huống 3, 4) |
| LAN-Test | Ubuntu Server 22.04/24.04 LTS, 1 GB RAM (dùng cho Tình huống 2) |

### Bảng địa chỉ IP

| Thiết bị | Interface | IP | Gateway | DNS |
| --- | --- | --- | --- | --- |
| pfSense | WAN (em0) | DHCP – hiện nhận `192.168.80.153/24` | 192.168.80.2 | 192.168.80.2 |
| pfSense | LAN (em1) | 10.0.0.1/8 | – | – |
| pfSense | DMZ / OPT1 (em2) | 172.16.0.1/16 | – | – |
| Domain Controller | LAN | 10.0.0.2/8 | 10.0.0.1 | 10.0.0.2 |
| LAN-Test | LAN | 10.0.0.3/8 | 10.0.0.1 | – |
| DMZ-Web | DMZ | 172.16.0.2/16 | 172.16.0.1 | 8.8.8.8 (sau khi có rule Pass DMZ → Internet) |

## 3. Cách dựng môi trường

### 3.1. Mạng ảo trong VMware

| Mạng | Loại | Subnet | Ghi chú |
| --- | --- | --- | --- |
| VMnet1 | Host-only | 10.0.0.0/8 | Tắt DHCP; bỏ tick "Connect a host virtual adapter" để máy thật không tham gia mạng lab |
| VMnet8 | NAT | 192.168.80.0/24 | Dùng làm WAN, không chồng lấn LAN/DMZ |
| `dmz-net` | LAN Segment | 172.16.0.0/16 | Dùng làm vùng DMZ |

### 3.2. Card mạng máy pfSense

| Adapter | Chế độ | Vai trò | Tên trong pfSense |
| --- | --- | --- | --- |
| Network Adapter (1) | NAT (VMnet8) | WAN | em0 |
| Network Adapter 2 | Host-only (VMnet1) | LAN | em1 |
| Network Adapter 3 | LAN Segment `dmz-net` | DMZ | em2 |

### 3.3. Các bước thực hiện

1. Tạo VM pfSense (FreeBSD 64-bit, 2 GB RAM, 3 card mạng như bảng trên).
2. Kiểm tra SHA-256, giải nén ISO `pfSense-CE-2.7.2-RELEASE-amd64.iso.gz`, gắn ISO và cài đặt (Auto ZFS). Cài xong tháo ISO.
3. Trên console pfSense: chọn `2) Set interface(s) IP address`, đặt LAN `10.0.0.1/8`, không bật DHCP.
4. Trên Windows Server (card nối VMnet1): đặt IP tĩnh `10.0.0.2/255.0.0.0`, gateway `10.0.0.1`, DNS `10.0.0.2`.
5. Từ Windows Server mở `https://10.0.0.1`, đăng nhập pfSense và chạy Setup Wizard (DNS `8.8.8.8`, bỏ tick *Block RFC1918* và *Block bogon* ở WAN, đổi mật khẩu admin).
6. Trên Windows Server: đổi tên máy `DC01`, cài vai trò AD DS, promote thành Domain Controller (forest `vietnam.local`), cấu hình DNS Forwarder `8.8.8.8`.
7. Trên pfSense: gán `em2` thành OPT1, đặt tên `DMZ`, IPv4 `172.16.0.1/16`.
8. Kiểm tra Outbound NAT (Automatic/Hybrid) có rule cho LAN và DMZ.
9. Disable 2 rule *Default allow LAN to any* (IPv4 và IPv6), giữ *Anti-Lockout Rule*, Reset States, tạo rule `Pass | Any | LAN net | Any`.
10. Kiểm tra từ Domain Controller: `ping 8.8.8.8`, `Resolve-DnsName example.com`, `curl.exe -4 https://example.com`.

## 4. Các phần đã thực hiện

| STT | Nội dung | Trạng thái | Kết quả |
| --- | --- | --- | --- |
| B.1 | Chuẩn bị VM pfSense (3 card mạng) | Đã làm | PASS |
| B.2 | Cài đặt pfSense từ ISO | Đã làm | PASS |
| B.3 | Đặt IP LAN 10.0.0.1/8 trên console | Đã làm | PASS |
| B.4 | Cấu hình Domain Controller (IP, AD DS, DNS Forwarder) | `<Cập nhật>` | `<PASS/FAIL>` |
| B.5 | Truy cập WebGUI và chạy Setup Wizard | Truy cập WebGUI đã làm; Wizard `<cập nhật>` | PASS (WebGUI) / `<PASS/FAIL>` |
| B.6 | Cấu hình vùng DMZ | `<Cập nhật>` | `<PASS/FAIL>` |
| B.7 | Kiểm tra Outbound NAT | `<Cập nhật>` | `<PASS/FAIL>` |
| B.8 | Chuẩn hóa ruleset LAN, tạo rule nền tảng | `<Cập nhật>` | `<PASS/FAIL>` |
| B.9 | Kiểm tra rule nền tảng từ DC | `<Cập nhật>` | `<PASS/FAIL>` |

## 5. Các tình huống firewall

| Tình huống | Nội dung | Rule chính | Kết quả mong đợi | Kết quả thực tế |
| --- | --- | --- | --- | --- |
| 1 | Chặn ICMP, vẫn cho Web/DNS | Block ICMP → Pass DNS 53 → Pass HTTP/HTTPS | `ping 8.8.8.8` thất bại; DNS và HTTPS thành công | `<PASS/FAIL>` |
| 2 | Chỉ cho một host ra Internet | Pass 10.0.0.2 → Any; Block LAN net → Any | DC ping được, LAN-Test không ping được | `<PASS/FAIL>` |
| 3 | Cô lập DMZ khỏi LAN | Block DMZ net → LAN net đặt trên Pass DMZ net → Any | Baseline DMZ → DC thành công; sau khi Block thì thất bại; DMZ vẫn ra Internet | `<PASS/FAIL>` |
| 4 | Port Forward WAN → DMZ | WAN:8080 → 172.16.0.2:80 | Thấy trang IIS của DMZ-Web | `<PASS/FAIL>` |
| 5 | Bật logging, đọc Firewall Log | Bật *Log packets* cho rule Block | Firewall Log có bản ghi, xác định đúng rule chặn | `<PASS/FAIL>` |

> Quy trình trước mỗi tình huống: xác nhận 2 rule *Default allow LAN to any* vẫn Disabled, Disable rule thừa, Apply Changes, **Reset States**.

## 6. Lỗi gặp phải và cách khắc phục

| STT | Lỗi | Nguyên nhân | Cách khắc phục |
| --- | --- | --- | --- |
| 1 | Adapter 3 của pfSense đang là NAT thay vì vùng DMZ | Đề viết cho VirtualBox (Internal Network), VMware dùng tên khác | Đổi sang **LAN Segment**, tạo segment tên `dmz-net` trong *LAN Segments...* |
| 2 | Khi đặt IP LAN trên console, câu hỏi IPv6 hiện lặp lại | Gõ `n` vào ô nhập địa chỉ IPv6 nên bị coi là địa chỉ không hợp lệ | Nhấn Enter để trống ô IPv6 rồi trả lời `n` cho câu hỏi DHCP |
| 3 | Windows Server có 2 card mạng, không biết card nào nối VMnet1 | VM được tạo sẵn với nhiều adapter | Giữ 1 card gắn VMnet1 (Custom/Host-only), gỡ hoặc ngắt card còn lại rồi đặt IP `10.0.0.2` |
| 4 | WAN chỉ có IPv6, `DHCP: down`, không có IPv4 | Bridged (Automatic) không xin được DHCP IPv4 từ mạng ngoài | Đổi Adapter 1 sang **NAT (VMnet8)**; Renew WAN, nhận `192.168.80.153/24` |
| 5 | VMnet1 có card ảo của máy thật, có thể trùng IP `10.0.0.1` với pfSense | VMware tự gắn host virtual adapter | Bỏ tick "Connect a host virtual adapter to this network" để mọi thao tác chạy hoàn toàn trong máy ảo |
| 6 | Trình duyệt trong Windows Server khó mở WebGUI | IE Enhanced Security Configuration | Tắt IE ESC trong Server Manager → Local Server (nếu bị chặn) |

## 7. Ghi chú

- WAN dùng NAT (VMnet8) thay cho Bridged do Bridged không cấp IPv4; dải `192.168.80.0/24` không chồng lấn LAN `10.0.0.0/8` và DMZ `172.16.0.0/16`.
- Tình huống 4 với WAN là NAT: dùng một VM khác cùng VMnet8 (ví dụ Kali Linux) để truy cập `http://<IP-WAN-pfSense>:8080`, không dùng máy thật.
- pfSense CE 2.7.2 là bản cũ, chỉ dùng trong mạng lab ảo hóa, không dùng cho production.

## 8. Minh chứng

Ảnh chụp màn hình lưu trong thư mục `images/` (đặt tên theo Hình trong đề: `hinh13.png`, `hinh14.png`, ...).
