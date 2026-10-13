
# LAB 6 – Xác thực Active Directory và chính sách truy cập trên pfSense

## 1. Thông tin sinh viên

| Mục | Nội dung |
| --- | --- |
| Họ và tên | `Trần Văn Hiệp` |
| MSSV | `1150080051` |
| Lớp / Học phần | `11_ĐH_THMT – An toàn hệ thống thông tin` |
| Tên lab | Thực hành an toàn hệ thống thông tin – Xác thực Active Directory và chính sách truy cập trên pfSense |
| Ngày thực hiện | 13/10/2026 |

## 2. Phiên bản môi trường

| Thành phần | Phiên bản / Cấu hình |
| --- | --- |
| Phần mềm ảo hóa | VMware Workstation 17.x (đề gốc viết cho VirtualBox, đã ánh xạ tương đương) |
| Firewall | pfSense CE 2.7.2-RELEASE (amd64), 2 GB RAM, ổ đĩa 10 GB; VM tên `FreeBSD 64-bit` |
| Domain Controller | Windows Server 2022 (`DC01`), AD DS + DNS, domain `lab6.test` |
| Client-A / Client-B / Máy quản trị | Windows 11 x64, 4 GB RAM, gộp 1 VM (đổi IP và tài khoản theo từng lần test) |
| DMZ-WEB | Ubuntu 64 + Nginx, 1 GB RAM, thêm IP phụ `.20` `.30` `.40` |

### Bảng địa chỉ IP

| Thiết bị | Interface | IP | Gateway | DNS |
| --- | --- | --- | --- | --- |
| pfSense | WAN (em0) | DHCP – hiện nhận `192.168.250.129/24` | Gateway NAT VMware (thường `192.168.250.2`) | – |
| pfSense | MGMT / OPT1 (em1) | 192.168.56.254/24 | – | – |
| pfSense | LAN (em2) | 10.6.0.1/24 | – | – |
| pfSense | DMZ / OPT2 (em3) | 172.16.6.1/24 | – | – |
| Win 11 | Ethernet0 (VMnet1, MGMT) | 192.168.56.20/24 | – | – |
| Win 11 | Ethernet1 (lab6-lan) | 10.6.0.21/24 (Client-A) hoặc 10.6.0.22/24 (Client-B) | 10.6.0.1 | 10.6.0.10 |
| DC01 | LAN | 10.6.0.10/24 | 10.6.0.1 | 10.6.0.10 |
| DMZ-WEB | DMZ | 172.16.6.10/24 (+ .20 / .30 / .40) | 172.16.6.1 | – |

## 3. Cách dựng môi trường

### 3.1. Mạng ảo trong VMware

| Mạng | Loại | Subnet | Ghi chú |
| --- | --- | --- | --- |
| VMnet8 | NAT | 192.168.250.0/24 | Dùng làm WAN, bật DHCP |
| VMnet1 | Host-only | 192.168.56.0/24 | Dùng làm MGMT, tắt DHCP |
| `lab6-lan` | LAN Segment | 10.6.0.0/24 | LAN-DOMAIN: DC01, Win 11 |
| `lab6-dmz` | LAN Segment | 172.16.6.0/24 | DMZ: DMZ-WEB |

### 3.2. Card mạng máy pfSense

| Adapter | Chế độ | Vai trò | Tên trong pfSense |
| --- | --- | --- | --- |
| Network Adapter (1) | NAT (VMnet8) | WAN | em0 |
| Network Adapter 2 | Custom: VMnet1 | MGMT | em1 |
| Network Adapter 3 | LAN Segment `lab6-lan` | LAN | em2 |
| Network Adapter 4 | LAN Segment `lab6-dmz` | DMZ | em3 |

### 3.3. Card mạng các máy còn lại

| Máy | Card | Chế độ |
| --- | --- | --- |
| Win 11 | Network Adapter | Custom: VMnet1 (MGMT, vào WebGUI) |
| Win 11 | Network Adapter 2 | LAN Segment `lab6-lan` |
| DC01 (Windows Server 2022) | Network Adapter | LAN Segment `lab6-lan` |
| DMZ-WEB (Ubuntu 64) | Network Adapter | LAN Segment `lab6-dmz` |

### 3.4. Các bước thực hiện

1. Trong Virtual Network Editor: VMnet8 = NAT `192.168.250.0/24` (bật DHCP); VMnet1 = Host-only `192.168.56.0/24` (tắt DHCP).
2. Gắn đủ 4 card cho pfSense như bảng 3.2; tạo 2 LAN Segment `lab6-lan`, `lab6-dmz` tại *VM Settings → LAN Segments...*.
3. Trên console pfSense: `1) Assign Interfaces` (WAN = em0, LAN = em2, OPT1 = em1, OPT2 = em3).
4. `2) Set interface(s) IP address`: LAN `10.6.0.1/24`, OPT1 `192.168.56.254/24`, OPT2 `172.16.6.1/24`, không bật DHCP server.
5. Trên Win 11, đặt IP card VMnet1 `192.168.56.20/24` (không gateway, không DNS) rồi mở `https://192.168.56.254`.
6. Trong WebGUI: đặt tên OPT1 = `MGMT`, OPT2 = `DMZ`, tạo rule `Pass | MGMT subnets → Any` trên tab MGMT; đổi mật khẩu admin; tạo rule `Pass | LAN net → Any` tạm thời để DC01 ra ngoài.
7. DC01: đặt IP `10.6.0.10/24`, cài AD DS + DNS, forest `lab6.test`, DNS Forwarder, tạo OU `LAB6-Users`, user `sv_a`, `sv_b`, service account `svc_pfsense_ldap` (không cấp Domain Admin).
8. DMZ-WEB: đặt IP `172.16.6.10/24`, cài Nginx, thêm IP `.20` `.30` `.40`.
9. pfSense: tạo LDAP server `LAB6-AD`, kiểm tra ở *Diagnostics → Authentication*; tạo Captive Portal zone `LAB6_USERS` trên LAN.
10. Tạo Alias, Schedule, ruleset theo mục 5 và kiểm thử.

## 4. Các phần đã thực hiện

| STT | Nội dung | Trạng thái | Kết quả |
| --- | --- | --- | --- |
| B.1 | Tạo mạng ảo VMware (VMnet8, VMnet1, LAN Segment) | Đã làm | PASS |
| B.2 | Gắn 4 card mạng cho pfSense | Đã làm | PASS |
| B.3 | Cài đặt pfSense, Assign Interfaces | Đã làm | PASS |
| B.4 | Đặt IP 4 interface trên console | Đã làm | PASS |
| B.5 | Cấu hình card mạng Win 11 (VMnet1 + lab6-lan) | Đang làm: ping `192.168.56.254` còn timeout | `<PASS/FAIL>` |
| B.6 | Truy cập WebGUI, đặt tên MGMT/DMZ, rule MGMT | `<Cập nhật>` | `<PASS/FAIL>` |
| B.7 | Chuẩn bị Domain Controller (IP, AD DS, OU, user, service account) | `<Cập nhật>` | `<PASS/FAIL>` |
| B.8 | Chuẩn bị DMZ-WEB (Nginx, IP .20/.30/.40) | `<Cập nhật>` | `<PASS/FAIL>` |
| C | Tích hợp LDAP, test `sv_a` và `sv_b` | `<Cập nhật>` | `<PASS/FAIL>` |
| D | Captive Portal xác thực AD | `<Cập nhật>` | `<PASS/FAIL>` |
| E | Alias | `<Cập nhật>` | `<PASS/FAIL>` |
| F | Schedule | `<Cập nhật>` | `<PASS/FAIL>` |

## 5. Các tình huống firewall

| Tình huống | Nội dung | Rule chính | Kết quả mong đợi | Kết quả thực tế |
| --- | --- | --- | --- | --- |
| 1 | Client-A bị chặn JOB_SITE | Block CLIENT_A → JOB_SITE TCP/80; Pass CLIENT_A → GENERAL_SITE TCP/80 | `.10` truy cập được, `.20` bị chặn, Firewall Log có `10.6.0.21 → 172.16.6.20` | `<PASS/FAIL>` |
| 2 | Cấm RESTRICTED_SITE cho A và B | Block POLICY_CLIENTS → RESTRICTED_SITE TCP/80 (Log) | Cả hai bị chặn `.40`, vẫn vào được `.10` | `<PASS/FAIL>` |
| 3 | Client-B chỉ vào SOCIAL_SITE giờ nghỉ | Pass CLIENT_B → SOCIAL_SITE (LUNCH_WINDOW); Block bên dưới | Trong giờ vào được `.30`, ngoài giờ bị chặn | `<PASS/FAIL>` |
| 4 | Chỉ cho HTTP/HTTPS trong giờ hành chính | Pass POLICY_CLIENTS → Any WEB_PORTS (WORK_HOURS); Block Any | Trong giờ web chạy, ping bị chặn; ngoài giờ bị chặn hết | `<PASS/FAIL>` |
| 5 | Cô lập DMZ khỏi LAN | Tab DMZ: Block DMZ net → LAN net | DMZ-WEB → `10.6.0.10` bị chặn; LAN → `.10` vẫn Pass | `<PASS/FAIL>` |
| 6 | So sánh Block và Reject | Cùng source/destination, đổi action | Reject báo lỗi nhanh, Block chờ timeout | `<PASS/FAIL>` |
| 7 | Đọc Firewall Log và State Table | Bật Log cho rule Block | Có bản ghi đúng interface/action/source/destination | `<PASS/FAIL>` |

### Ruleset tổng hợp trên tab LAN (thứ tự là một phần của đáp án)

| Ưu tiên | Action | Source | Destination | Port | Schedule | Log |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | Block | CLIENT_A | JOB_SITE | TCP/80 | – | Bật |
| 2 | Block | POLICY_CLIENTS | RESTRICTED_SITE | TCP/80 | – | Bật |
| 3 | Pass | CLIENT_B | SOCIAL_SITE | TCP/80 | LUNCH_WINDOW | |
| 4 | Block | CLIENT_B | SOCIAL_SITE | TCP/80 | – | |
| 5 | Pass | POLICY_CLIENTS | GENERAL_SITE | TCP/80 | – | |
| 6 | Pass | POLICY_CLIENTS | Any | WEB_PORTS | WORK_HOURS | |
| 7 | Block | POLICY_CLIENTS | Any | – | – | Bật |

### Alias và Schedule

| Alias | Type | Giá trị |
| --- | --- | --- |
| CLIENT_A | Host | 10.6.0.21 |
| CLIENT_B | Host | 10.6.0.22 |
| POLICY_CLIENTS | Host(s) | 10.6.0.21, 10.6.0.22 |
| GENERAL_SITE | Host | 172.16.6.10 |
| JOB_SITE | Host | 172.16.6.20 |
| SOCIAL_SITE | Host | 172.16.6.30 |
| RESTRICTED_SITE | Host | 172.16.6.40 |
| WEB_PORTS | Port(s) | 80, 443 |

| Schedule | Khung giờ |
| --- | --- |
| LUNCH_WINDOW | 11:30–13:00, Thứ 2–Thứ 6 |
| WORK_HOURS | 08:00–17:00, Thứ 2–Thứ 6 |
| TEST_NOW | Khoảng 10–15 phút quanh thời điểm thực hành (xóa sau khi test) |

> Quy trình trước mỗi tình huống: Apply Changes, **Reset States**, tạo kết nối mới.

### Bảng kiểm thử bắt buộc

| Nguồn | Đích | Trong schedule | Ngoài schedule | Kết quả thực tế |
| --- | --- | --- | --- | --- |
| Client-A | GENERAL_SITE .10 | Allow | Allow | `<Cập nhật>` |
| Client-A | JOB_SITE .20 | Block | Block | `<Cập nhật>` |
| Client-A | RESTRICTED_SITE .40 | Block | Block | `<Cập nhật>` |
| Client-B | SOCIAL_SITE .30 | Allow (LUNCH) | Block | `<Cập nhật>` |
| Client-B | RESTRICTED_SITE .40 | Block | Block | `<Cập nhật>` |
| DMZ-WEB | DC01 10.6.0.10 | Block | Block | `<Cập nhật>` |

## 6. Lỗi gặp phải và cách khắc phục

| STT | Lỗi | Nguyên nhân | Cách khắc phục |
| --- | --- | --- | --- |
| 1 | Không thấy nơi tạo LAN Segment trong Virtual Network Editor | LAN Segment nằm trong cài đặt từng VM, không phải ở Virtual Network Editor | Vào *VM Settings → Network Adapter → chọn radio LAN segment → nút LAN Segments... → Add* |
| 2 | Console pfSense chỉ hiện WAN và LAN với LAN là `10.0.0.1/8` | pfSense mới cài chỉ nhận 2 card và dùng IP LAN mặc định | Thêm đủ 4 card, chạy Assign Interfaces, đặt lại IP bằng option 2 |
| 3 | Sau khi đặt IP, console báo "access https://172.16.6.1" | Đó chỉ là thông báo của interface cuối cùng vừa đặt (OPT2) | Bỏ qua; địa chỉ quản trị của lab là `https://192.168.56.254` |
| 4 | Đề dùng máy thật làm máy quản trị `192.168.56.1` | Thầy yêu cầu chỉ làm trong VMware, không dùng máy thật | Dùng Win 11 gắn thêm card VMnet1 làm máy quản trị |
| 5 | Win 11 có sẵn card cũ (Host-only/NAT) và card đang để `Custom: VMnet0` | VM dùng lại từ lab trước | Đổi card 1 sang `Custom: VMnet1`, card 2 sang LAN Segment `lab6-lan` |
| 6 | Mở `https://192.168.56.254` báo `ERR_CONNECTION_TIMED_OUT`, ping timeout | Win 11 và pfSense chưa thấy nhau ở tầng mạng (nghi card VMnet sai) | Kiểm tra card của cả hai VM là `Custom: VMnet1`; ping từ pfSense (option 7) tới `192.168.56.20`; đối chiếu MAC em1; nếu cần `pfctl -d` rồi tạo rule MGMT |
| 7 | Tắt/mở VM để đổi card quá lâu | – | VMware cho đổi card khi VM đang chạy (VM → Settings → OK); dùng Suspend và Snapshot |

## 7. Ghi chú

- Toàn bộ lab chạy trong VMware, không dùng máy thật làm máy quản trị hay client.
- Gateway NAT của VMware là `.2` (khác `.1` trong đề VirtualBox), không ảnh hưởng đến lab.
- Client-A và Client-B dùng chung 1 VM Win 11: đổi IP card `lab6-lan` giữa `10.6.0.21` và `10.6.0.22`, đăng nhập Captive Portal bằng `sv_a` hoặc `sv_b`. Sau mỗi lần đổi IP: ngắt session Captive Portal và Reset States.
- Khi test kịch bản, Disable card VMnet1 trên Win 11 để traffic chỉ đi qua card `lab6-lan`.
- pfSense lọc theo IP/port/schedule, không dùng username LDAP làm điều kiện rule; vì vậy A và B được ánh xạ bằng IP tĩnh.
- LDAP TCP/389 chỉ dùng trong lab cô lập; triển khai thật phải dùng LDAP qua TLS.
- Tên card của Ubuntu trong VMware thường là `ens33` (đề ghi `enp0s3`); kiểm tra bằng `ip a` trước khi gán IP phụ.
- pfSense CE 2.7.2 chỉ dùng trong mạng lab ảo hóa, không dùng cho production.

## 8. Minh chứng

Ảnh chụp màn hình lưu trong thư mục `images/` (đặt tên theo Hình trong đề). Danh sách bắt buộc:

- [ ] VMware Settings 4 card mạng của pfSense
- [ ] LDAP test `sv_a` và `sv_b`
- [ ] Captive Portal login
- [ ] Alias, Schedule
- [ ] Mỗi kịch bản: 1 ảnh rule + 1 ảnh kiểm thử (thấy rõ IP bằng `ipconfig`)
- [ ] Firewall Log, State Table
- [ ] File `config.xml` (Diagnostics → Backup & Restore → Download configuration)
