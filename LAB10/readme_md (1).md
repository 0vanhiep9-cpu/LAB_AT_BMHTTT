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

---

## 7. Phần K: Trả lời câu hỏi phân tích

### Câu 1: Vì sao LAB 10 buộc Kali rời DNSLAB và đặt Promiscuous Mode = Deny?
- **Trả lời:** Việc tách Kali sang subnet hoàn toàn độc lập (`ATTACKNET`) và chặn chế độ promiscuous mode nhằm mô phỏng chính xác mô hình tấn công từ xa (remote). Trong thực tế, attacker không nằm cùng phân mạng cục bộ với DNS Resolver của nạn nhân nên không thể sử dụng kỹ thuật sniffing (bắt gói tin lớp 2/3) để đọc trộm các gói tin vấn đáp nội bộ. Điều này buộc attacker phải dựa vào phương pháp đoán mù (blind guessing) thay vì thụ động thu thập thông tin thời gian thực.

### Câu 2: Trong remote poisoning, attacker thiếu thông tin nào so với LAB trước?
- **Trả lời:** So với Local DNS Spoofing (LAB trước), attacker bị thiếu hoàn toàn hai thông số quan trọng trong gói tin DNS query do DNS Resolver gửi lên Upstream/Auth-DNS:
  1. **Transaction ID (TXID):** Mã định danh giao dịch 16-bit.
  2. **UDP Source Port:** Cổng nguồn (source port) mà resolver đang sử dụng để gửi yêu cầu. Do thiếu khả năng sniff, attacker không thể biết chính xác hai giá trị này tại thời điểm gói tin được truyền đi, dẫn đến việc phải thực hiện quét hoặc đoán trên không gian xác suất.

### Câu 3: Nếu chỉ có transaction ID 16-bit, xác suất một batch K giá trị chứa TXID đúng là bao nhiêu?
- **Trả lời:** Không gian giá trị của Transaction ID 16-bit có tổng cộng $2^{16} = 65.536$ khả năng khác nhau. Nếu attacker gửi một batch chứa $K$ giá trị TXID giả mạo trong một cửa sổ thời gian (race window), xác suất chứa TXID đúng trong batch đó được tính bằng công thức:
  $$\text{Xác suất} = \frac{K}{65.536}$$
  *(Ví dụ: Nếu một batch chứa 4.096 giá trị, xác suất thành công trong một lần thử là $\frac{4096}{65536} = \frac{1}{16}$ hay 6.25%).*

### Câu 4: Source-port randomization làm tăng không gian đoán như thế nào?
- **Trả lời:** Khi DNS Resolver áp dụng cơ chế ngẫu nhiên hóa cổng nguồn (*Source-Port Randomization*), cổng nguồn không còn cố định ở một giá trị mà được chọn ngẫu nhiên từ một dải cổng lớn (thường từ dải ephemeral ports, khoảng $2^{14} = 16.384$ cổng). Điều này làm tăng độ phức tạp không gian tìm kiếm, biến tổng không gian đoán thành tích của TXID và Port:
  $$\text{Không gian đoán} = 2^{16} \text{ (TXID)} \times 2^{14} \text{ (Port)} = 2^{30} \text{ (hơn 1 tỷ khả năng)}$$
  Điều này khiến việc tấn công mù bằng phương pháp vét cạn trở nên bất khả thi trong thực tế nếu không có lượng traffic khổng lồ.

### Câu 5: Vì sao dùng tên ngẫu nhiên ở mỗi vòng giúp vượt qua cache effect?
- **Trả lời:** DNS Resolver có cơ chế caching; nếu một tên miền đã được phân giải thành công và lưu trong bộ nhớ đệm, resolver sẽ trả về kết quả trực tiếp từ cache mà không gửi query ra ngoài mạng Internet. Bằng cách sử dụng tên miền hoàn toàn ngẫu nhiên ở mỗi vòng lặp tấn công (ví dụ: `random123.example.lab10.test`), attacker buộc DNS Resolver phải coi đây là một yêu cầu mới hoàn toàn (cache miss), từ đó bắt buộc resolver phải phát sinh gói tin query mới ra máy chủ danh tính (Auth-DNS), tạo ra một cửa sổ thời gian (race window) mới để thực hiện tấn công giả mạo.

### Câu 6: Bản ghi NS và glue A có vai trò gì trong việc đổi hướng resolver?
- **Trả lời:**
  - **Bản ghi NS (Name Server):** Cho phép attacker đánh lừa resolver rằng toàn bộ không gian tên miền của nạn nhân được ủy quyền (delegated) sang một máy chủ danh tính giả mạo do attacker kiểm soát (ví dụ: `ns.evil.example.lab10.test`).
  - **Bản ghi Glue A:** Cung cấp sẵn địa chỉ IP tương ứng của máy chủ NS giả mạo ngay trong phần bổ sung (Additional section) của phản hồi. Nhờ có bản ghi Glue A, resolver không cần mất thêm một bước phân giải địa chỉ IP của máy chủ NS độc hại đó nữa mà lập tức chấp nhận và lưu trữ luôn IP của attacker (`10.10.0.200`) vào cache, hoàn tất quá trình đổi hướng toàn bộ các truy vấn tiếp theo về phía attacker.

### Câu 7: Vì sao Additional record ngoài bailiwick có thể bị resolver từ chối?
- **Trả lời:** Cơ chế kiểm tra **Bailiwick** (phạm vi thẩm quyền) được tích hợp trong các trình phân giải DNS hiện đại nhằm ngăn chặn các cuộc tấn công đầu độc bộ nhớ đệm dạng chéo (cross-domain cache poisoning). Nếu một máy chủ danh tính cố gắng trả về các bản ghi bổ sung (Additional/Glue records) thuộc về một tên miền hoàn toàn khác không nằm trong thẩm quyền của nó, resolver sẽ nhận diện đó là dữ liệu không hợp lệ ngoài vùng quản lý và từ chối lưu vào cache, qua đó bảo vệ hệ thống khỏi việc bị tiêm nhiễm mã độc từ các nguồn không liên quan.

### Câu 8: Tại sao LAB 10 dùng .test thay cho website thật?
- **Trả lời:** Theo chuẩn quốc tế **RFC 2606**, không gian tên miền `.test` (cùng với `.example`, `.invalid`, `.localhost`) được đặt riêng và dành hoàn toàn cho mục đích thử nghiệm, kiểm thử kỹ thuật và nghiên cứu học tập. Việc sử dụng domain `.test` đảm bảo tính an toàn tuyệt đối, cô lập hoàn toàn môi trường thực hành nội bộ, tránh gây ảnh hưởng, xung đột hoặc vi phạm pháp luật khi tấn công/thao túng các hệ thống mạng và website thực tế trên Internet công cộng.

### Câu 9: Phân biệt tác dụng của DNSSEC validation, recursion ACL và firewall segmentation.
- **Trả lời:** Ba lớp bảo vệ này hoạt động ở các tầng khác nhau trong mô hình phòng thủ nhiều lớp (Defense-in-Depth):
  - **Firewall Segmentation (pfSense):** Hoạt động ở tầng mạng (Network Layer), dùng để cô lập các vùng mạng (chia tách rõ ràng mạng DNSLAB, ATTACKNET, DMZ), chặn kết nối trái phép giữa các subnet và giới hạn phạm vi lưu lượng lan truyền.
  - **Recursion ACL:** Hoạt động ở tầng ứng dụng dịch vụ DNS, cấu hình trên BIND9 nhằm chỉ định rõ những dải mạng nào mới được phép sử dụng máy chủ này để phân giải đệ quy, ngăn chặn việc máy chủ trở thành *Open Resolver*.
  - **DNSSEC Validation:** Hoạt động ở tầng giao thức DNS, sử dụng mã hóa khóa công khai để xác thực tính toàn vẹn và nguồn gốc xác thực của dữ liệu trả về từ các máy chủ danh tính, giúp chống lại tận gốc các hành vi giả mạo, đầu độc bản ghi trên đường truyền.

### Câu 10: Vì sao race relay không làm cho LAB 10 trở thành mô hình tấn công Internet thực tế?
- **Trả lời:** Trong môi trường Internet thực tế, attacker không thể cài đặt một đoạn mã trung gian hay chương trình tiếp vận cục bộ (`race harness` chạy trên cổng `5300`) bên trong hệ thống của máy chủ hợp pháp (Auth-DNS/APP-TEST). Do đó, cơ chế phối hợp gửi gói tin thông qua race relay chỉ là một giải pháp mô phỏng mang tính giáo dục giúp thu hẹp khoảng cách thời gian và tái hiện mô hình đoán mù TXID trong điều kiện phòng thí nghiệm an toàn, không thể áp dụng trực tiếp lên không gian mạng Internet rộng lớn bên ngoài.

### Câu 11: Khi khôi phục BIND, vì sao Kali không nên còn quyền recursion?
- **Trả lời:** Nếu để máy tính của attacker (Kali) tiếp tục có quyền truy cập đệ quy (recursion) lên máy chủ DNS nội bộ sau khi kết thúc tấn công, máy chủ DNS đó sẽ đóng vai trò như một **Open Resolver**. Attacker có thể lợi dụng điểm yếu này để thực hiện các cuộc tấn công khuếch đại DNS (DNS Amplification DDoS) nhắm vào các hệ thống khác, hoặc sử dụng máy chủ nội bộ làm bàn đạp để tiếp tục do thám và khai thác các dịch vụ nội mạng.

### Câu 12: Nêu tối thiểu bốn biện pháp hardening resolver production.
- **Trả lời:** Các biện pháp cứng hóa (hardening) quan trọng đối với một máy chủ DNS Resolver trong môi trường thực tế bao gồm:
  1. **Bật và bắt buộc xác thực DNSSEC (`dnssec-validation auto`):** Đảm bảo toàn bộ chuỗi tin cậy được kiểm tra chữ ký số hợp lệ trước khi chấp nhận bản ghi.
  2. **Giới hạn nghiêm ngặt quyền đệ quy (Recursion Control):** Chỉ cho phép các dải IP nội bộ tin cậy được phép query đệ quy thông qua thiết lập ACL chặt chẽ.
  3. **Sử dụng Source-Port Randomization:** Kích hoạt triệt để các cơ chế ngẫu nhiên hóa cổng nguồn truyền thống trên các phiên bản phần mềm DNS hiện đại nhằm nâng cao độ phức tạp khi đoán gói tin.
  4. **Giới hạn tốc độ phản hồi (Rate Limiting - Response Rate Limiting / RRL):** Cấu hình chặn hoặc giảm tốc độ xử lý đối với các yêu cầu dồn dập từ cùng một nguồn nhằm giảm thiểu nguy cơ bị lợi dụng tấn công từ chối dịch vụ (DDoS) hoặc brute-force cache.