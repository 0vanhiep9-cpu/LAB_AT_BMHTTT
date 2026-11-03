# BÁO CÁO THỰC HÀNH - LAB 10: REMOTE DNS CACHE POISONING

## 1. Thông tin sinh viên
- **Họ và tên:** Trần Văn Hiệp
- **MSSV:** 1150080051
- **Tên học phần:** Thực hành An toàn Hệ thống Thông tin
- **Tên bài lab:** Lab 10 – Remote DNS Cache Poisoning trong môi trường pfSense cô lập
- **Năm học:** 2026–2027

---

## 2. Phiên bản môi trường
Môi trường thực hành triển khai trên nền tảng ảo hóa **VMware Workstation Pro** với mô hình mạng phân đoạn bao gồm các máy ảo:
- **pfSense (Gateway / Firewall):** Phiên bản 2.7.2-RELEASE (FreeBSD 14.0-CURRENT), quản lý các phân mạng WAN (`em0`), LAN (`em2` - 10.6.0.0/24), DNSLAB (`em1` - 10.9.0.0/24), MGMT (`em3` - 192.168.56.0/24) và ATTACKNET (`em4` - 10.10.0.0/24).
- **DNS-SERVER (Ubuntu):** IP `10.9.0.10`, chạy dịch vụ BIND9 / resolver mô phỏng (`vuln_resolver_lab10.py`).
- **VICTIM (Windows 11):** IP `10.9.0.100`, đóng vai trò là DNS Client.
- **ATTACKER (Kali Linux):** IP `10.10.0.200`, đặt ở phân mạng độc lập `ATTACKNET`, chạy script tấn công Kaminsky (`kaminsky_remote_lab10.py`), malicious authoritative DNS (`evil_dns_lab10.py`) và web server giả mạo (`fake_index.html`).
- **AUTH-DNS / RELAY (Debian/App-Test):** IP `172.16.6.20`, đóng vai trò máy chủ danh tính hợp lệ và chạy race harness (`auth_dns_relay_lab10.py`).

---

## 3. Cách dựng môi trường
1. **Khởi tạo và cấu hình pfSense:** Thêm Adapter mạng thứ 5 kết nối vào `Internal Network` (gắn nhãn `attack-net`), cấu hình giao diện `OPT3` với địa chỉ IP tĩnh `10.10.0.1/24` và tuyệt đối không gán Upstream Gateway.
2. **Cấu hình Kali Linux (Attacker):** Tách Kali khỏi mạng DNSLAB cũ, gắn vào mạng `ATTACKNET` với địa chỉ IP tĩnh `10.10.0.200/24`, gateway `10.10.0.1`, và cấu hình `Promiscuous Mode = Deny` để đảm bảo tính chất tấn công từ xa.
3. **Cấu hình Windows 11 (Victim):** Thiết lập địa chỉ IP tĩnh `10.9.0.100/24`, gateway `10.9.0.1` và DNS Server trỏ về `10.9.0.10`.
4. **Thiết lập Firewall Rules trên pfSense:** Tạo các quy tắc lọc gói tin trên cổng `OPT3` để cô lập phân mạng ATTACKNET, chỉ cho phép các luồng truyền thông phục vụ cho mục đích bài lab (kích hoạt resolver qua cổng 53 và gói tin race qua cổng 5300) đồng thời chặn hoàn toàn truy cập sang LAN, MGMT và Internet.

---

## 4. Các tình huống đã thực hiện
- **Chuyển đổi mô hình từ LAB cũ sang LAB 10:** Tách biệt hoàn toàn Attacker sang mạng độc lập nhằm vô hiệu hóa khả năng sniffing gói tin trực tiếp.
- **Xác lập tính "Remote" của bài lab:** Kiểm chứng việc Kali không thể quan sát dòng traffic truyền thông giữa DNS-SERVER và AUTH-DNS.
- **Pha tấn công mô phỏng (Attack Phase):** Tạm dừng BIND9 để chuyển sang resolver mô phỏng có chủ đích (`vuln_resolver_lab10.py`), kích hoạt script tấn công Kaminsky kết hợp race relay để giả mạo bản ghi NS và Glue A record.
- **Kiểm chứng hậu quả:** Xác nhận máy nạn nhân (VICTIM) bị chuyển hướng phân giải tên miền `www.example.lab10.test` từ IP thật `172.16.6.10` sang IP độc hại `10.10.0.200` và hiển thị trang web giả mạo.
- **Khôi phục dịch vụ và Hardening:** Dừng resolver mô phỏng, khởi động lại BIND9 hiện đại, thu hẹp quyền truy cập đệ quy (Recursion ACL) và kiểm chứng tính năng ngẫu nhiên hóa cổng nguồn (Source-Port Randomization).

---

## 5. Kết quả PASS / FAIL
| Nội dung thực hiện | Trạng thái |
| :--- | :--- |
| Chuyển đổi mô hình mạng & Cấu hình pfSense | **PASS** |
| Định hình phân mạng ATTACKNET & Cô lập tường lửa | **PASS** |
| Cấu hình DNS-SERVER và Baseline ban đầu | **PASS** |
| Kiểm chứng tính chất Remote (Không sniff được TXID) | **PASS** |
| Thực thi Remote Cache Poisoning / Kaminsky Simulation | **PASS** |
| Kiểm chứng phân giải độc hại từ VICTIM | **PASS** |
| Phục hồi BIND9 và Hardening hệ thống | **PASS** |

---

## 6. Lỗi gặp phải và cách khắc phục
- **Lỗi 1: Không truy cập được giao diện quản trị WebGUI của pfSense từ máy ảo Windows 11.**
  - *Nguyên nhân:* Mặc định pfSense chặn các kết nối quản trị từ các phân mạng nội bộ không phải mạng LAN để đảm bảo an toàn.
  - *Cách khắc phục:* Sử dụng tính năng Shell trên màn hình console của pfSense để tạm thời vô hiệu hóa tường lửa bằng lệnh `pfctl -d`, truy cập cấu hình giao diện qua IP của cổng mạng tương ứng, sau đó bật lại tường lửa bằng lệnh `pfctl -e`.
- **Lỗi 2: Cổng mạng `OPT3` trên pfSense không nhận diện hoặc không liên lạc được với máy Kali.**
  - *Nguyên nhân:* Chưa gán đúng card mạng ảo tương ứng trên VMware Workstation với cấu hình Internal Network (`attack-net`) hoặc chưa thực hiện thao tác `Apply Changes` sau khi gán IP.
  - *Cách khắc phục:* Kiểm tra lại phần cứng ảo (Network Adapter settings) trong cài đặt máy ảo pfSense, thiết lập đúng tên card mạng và thực hiện gán lại cấu hình IP tĩnh `10.10.0.1/24` cho interface OPT3.