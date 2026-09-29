# LAB 4 — KHẢO SÁT VÀ ĐÁNH GIÁ BỀ MẶT MẠNG BẰNG NMAP

## 1. Thông tin sinh viên

* **Họ và tên:** Trần Văn Hiệp
* **MSSV:** 1150050051
* **Tên Lab:** Lab 4 — Khảo sát và đánh giá bề mặt mạng bằng Nmap


## 2. Môi trường thực hành

### Phần cứng và phần mềm

* **Hệ điều hành máy chủ:** Windows 11
* **Phần mềm ảo hóa:** VMware Workstation
* **Máy quét:** Kali Linux
* **Máy mục tiêu:** Metasploitable 2
* **Máy Windows kiểm tra:** Windows 11 VM
* **Công cụ:** Nmap 7.99

### Mạng thực hành

Sử dụng mạng **Host-only** của VMware để cô lập môi trường lab.

| Thiết bị             | Địa chỉ IP       |
| -------------------- | ---------------- |
| VMware Host / VMnet1 | `192.168.56.1`   |
| Kali Linux           | `192.168.56.10`  |
| Windows 11 VM        | `192.168.56.20`  |
| Metasploitable 2     | `192.168.56.101` |
| Subnet Mask          | `255.255.255.0`  |

Môi trường được thiết lập trong mạng nội bộ Host-only, không thực hiện quét các hệ thống bên ngoài phòng lab.


## 3. Cách dựng môi trường

### Bước 1 — Cấu hình VMware

Tạo và sử dụng mạng **Host-only VMnet1** với:

```text
Network: 192.168.56.0/24
Host:    192.168.56.1
```

Các máy ảo được kết nối vào VMnet1.

### Bước 2 — Cấu hình Kali Linux

Đặt địa chỉ IP:

```text
192.168.56.10/24
```

Kali được sử dụng làm máy thực hiện khảo sát bằng Nmap.

### Bước 3 — Cấu hình Windows 11 VM

Đặt địa chỉ IP:

```text
192.168.56.20/24
```

Windows 11 được sử dụng làm một máy trong mạng lab để kiểm tra khả năng kết nối và làm đối tượng khảo sát trong môi trường thực hành.

### Bước 4 — Cấu hình Metasploitable 2

Đặt địa chỉ IP cho card mạng:

```text
192.168.56.101/24
```

Metasploitable 2 được sử dụng làm máy mục tiêu thực hành vì đây là hệ thống được thiết kế cho mục đích học tập và kiểm thử bảo mật.

### Bước 5 — Kiểm tra kết nối

Từ Kali kiểm tra:

```bash
ping -c 4 192.168.56.101
```

và:

```bash
ping -c 4 192.168.56.20
```

Các máy trong mạng lab có thể liên lạc với nhau.

---

## 4. Các tình huống đã thực hiện

### 4.1. Host Discovery

Sử dụng:

```bash
sudo nmap -sn 192.168.56.0/24
```

Kết quả phát hiện 4 host đang hoạt động:

```text
192.168.56.1
192.168.56.10
192.168.56.20
192.168.56.101
```

**Kết quả: PASS**

---

### 4.2. TCP Connect Scan

Sử dụng:

```bash
sudo nmap -sT 192.168.56.101
```

Phát hiện nhiều TCP port đang mở trên Metasploitable 2, trong đó có:

```text
21/tcp    ftp
22/tcp    ssh
23/tcp    telnet
25/tcp    smtp
53/tcp    domain
80/tcp    http
139/tcp   netbios-ssn
445/tcp   microsoft-ds
3306/tcp  mysql
5432/tcp  postgresql
5900/tcp  vnc
6667/tcp  irc
8009/tcp  ajp13
8180/tcp  unknown
```

**Kết quả: PASS**

---

### 4.3. TCP SYN Scan

Sử dụng:

```bash
sudo nmap -sS 192.168.56.101
```

Kết quả phát hiện 23 TCP port mở, tương ứng với kết quả TCP Connect Scan.

**Kết quả: PASS**

---

### 4.4. FIN Scan

Sử dụng:

```bash
sudo nmap -sF 192.168.56.101
```

Nhiều port được xác định ở trạng thái:

```text
open|filtered
```

**Kết quả: PASS**

---

### 4.5. Xmas Scan

Sử dụng:

```bash
sudo nmap -sX 192.168.56.101
```

Nhiều port được xác định ở trạng thái:

```text
open|filtered
```

**Kết quả: PASS**

---

### 4.6. NULL Scan

Sử dụng:

```bash
sudo nmap -sN 192.168.56.101
```

Nhiều port được xác định ở trạng thái:

```text
open|filtered
```

**Kết quả: PASS**

---

### 4.7. ACK Scan

Sử dụng:

```bash
sudo nmap -sA 192.168.56.101
```

Kết quả:

```text
1000 unfiltered tcp ports
```

Kết quả cho thấy các port được kiểm tra không bị xác định là filtered bởi ACK scan.

**Kết quả: PASS**

---

### 4.8. UDP Scan

Đã bắt đầu thực hiện UDP Scan bằng:

```bash
sudo nmap -sU 192.168.56.101
```

Do UDP scan cần nhiều thời gian hơn TCP scan và thời gian thực hành có hạn nên chưa hoàn thành toàn bộ quá trình quét.

**Kết quả: CHƯA HOÀN THÀNH**

---

## 5. Kết quả tổng hợp

| Nội dung                        | Kết quả         |
| ------------------------------- | --------------- |
| Thiết lập mạng Host-only        | PASS            |
| Kết nối Kali → Metasploitable 2 | PASS            |
| Host Discovery                  | PASS            |
| TCP Connect Scan (`-sT`)        | PASS            |
| SYN Scan (`-sS`)                | PASS            |
| FIN Scan (`-sF`)                | PASS            |
| Xmas Scan (`-sX`)               | PASS            |
| NULL Scan (`-sN`)               | PASS            |
| ACK Scan (`-sA`)                | PASS            |
| UDP Scan (`-sU`)                | CHƯA HOÀN THÀNH |

---

## 6. Lỗi gặp phải và cách khắc phục

### Lỗi 1 — Windows 11 nhận IP `169.254.x.x`

Windows 11 ban đầu nhận địa chỉ:

```text
169.254.190.167
```

Đây là địa chỉ APIPA do máy chưa nhận được địa chỉ IP phù hợp từ mạng lab.

**Cách khắc phục:**

Cấu hình IP tĩnh cho Windows 11:

```text
IP:      192.168.56.20
Subnet:  255.255.255.0
```

Sau khi cấu hình, Windows 11 kết nối được vào mạng Host-only.

**Kết quả: Đã khắc phục**

---

### Lỗi 2 — Kali và Metasploitable 2 khác mạng

Kali ban đầu sử dụng mạng khác với mạng Host-only của Lab 4 nên không thể kết nối tới Metasploitable 2.

**Cách khắc phục:**

Đưa Kali về mạng:

```text
192.168.56.0/24
```

và đặt:

```text
Kali:           192.168.56.10
Metasploitable: 192.168.56.101
```

Sau đó kiểm tra bằng `ping`.

**Kết quả: Đã khắc phục**

---

### Lỗi 3 — UDP Scan thực hiện lâu

UDP Scan có thời gian thực hiện dài hơn TCP Scan do đặc điểm giao tiếp của UDP và cơ chế timeout khi không nhận được phản hồi.

**Cách xử lý:**

Đã dừng việc quét toàn bộ UDP port khi thời gian thực hành không còn đủ và ghi nhận UDP Scan là nội dung chưa hoàn thành.

**Kết quả: Chưa hoàn thành**

---

### Lỗi 4 — Cảnh báo DNS của Nmap

Nmap hiển thị:

```text
mass_dns: warning: Unable to determine any DNS servers.
Reverse DNS is disabled.
```

Đây không phải lỗi làm scan thất bại. Nmap vẫn thực hiện khảo sát IP bình thường; chỉ không thực hiện được reverse DNS.

**Cách xử lý:**

Tiếp tục thực hiện scan trong mạng lab nội bộ vì việc phân giải DNS không cần thiết cho các mục tiêu khảo sát IP hiện tại.

**Kết quả: Không ảnh hưởng đến quá trình scan**

---

## 7. Thư mục minh chứng

Các kết quả Nmap được lưu trong thư mục:

```text
LAB4_Evidence/
```

Các file kết quả gồm:

```text
01_host_discovery.txt
02_tcp_connect.txt
03_tcp_syn.txt
04_fin_scan.txt
05_xmas_scan.txt
06_null_scan.txt
07_ack_scan.txt
08_udp_scan.txt
```

Các ảnh chụp màn hình quá trình thực hành được sử dụng làm minh chứng trong báo cáo Lab 4.

---

## 8. Kết luận

Lab 4 đã xây dựng được môi trường khảo sát mạng nội bộ bằng VMware Host-only, sử dụng Kali Linux làm máy quét và Metasploitable 2 làm máy mục tiêu.

Các phương pháp khảo sát Host Discovery và TCP Scan gồm TCP Connect, SYN, FIN, Xmas, NULL và ACK đã được thực hiện và ghi nhận kết quả. Qua quá trình thực hành, nhiều dịch vụ TCP đang hoạt động trên Metasploitable 2 đã được phát hiện.

UDP Scan đã được bắt đầu nhưng chưa hoàn thành do giới hạn thời gian thực hành.

