
# LAB3 – Nhận diện và ứng phó các mối đe dọa đến an toàn thông tin

## 1. Thông tin sinh viên

* **Họ và tên:** [HỌ TÊN]
* **MSSV:** [MSSV]
* **Tên học phần:** Thực hành An toàn Hệ thống Thông tin
* **Tên bài:** Lab 3 – Nhận diện và ứng phó các mối đe dọa đến an toàn thông tin
* **Năm học:** 2026–2027

## 2. Môi trường thực hành

| Thành phần          | Phiên bản / cấu hình         |
| ------------------- | ---------------------------- |
| Ảo hóa              | VMware Workstation Pro       |
| Máy ảo              | Windows 11 25H2 x64          |
| CPU VM              | 2 vCPU                       |
| RAM VM              | 6 GB                         |
| Disk VM             | 64 GB                        |
| Network             | Host-only                    |
| Endpoint protection | Microsoft Defender Antivirus |
| Shell               | Windows PowerShell 5.1       |
| Sysmon              | 15.22                        |
| Autoruns            | 14.3                         |
| Process Explorer    | 17.14                        |
| Wireshark           | 4.6.8                        |
| Python              | 3.14.7                       |

## 3. Mục tiêu

Bài thực hành nhằm:

* Phân biệt Vulnerability, Threat, Risk và Attack.
* Nhận diện các nhóm nguồn đe dọa đối với hệ thống thông tin.
* Thu thập bằng chứng từ endpoint và mạng.
* Sử dụng Microsoft Defender, Windows Event Log, Sysmon, Autoruns, Process Explorer và Wireshark.
* Thực hiện quy trình Baseline → Observe → Detect → Contain → Recover → Verify.
* Lưu trữ bằng chứng, log và giá trị SHA-256 để kiểm tra tính toàn vẹn.

## 4. Chuẩn bị môi trường

### 4.1. Thư mục bài lab

Thư mục làm việc:

```text
C:\LAB3
```

Các thư mục chính:

```text
C:\LAB3
├── Assets
├── Downloads
├── Evidence
└── Tools
```

### 4.2. Gói dữ liệu bài lab

Gói dữ liệu sử dụng:

```text
LAB3_Threats_Assets.zip
```

SHA-256 được kiểm tra trước khi giải nén và đối chiếu với giá trị do giảng viên cung cấp:

```text
96236f95ce59d0cc37f9b7ab8fbb04522f21870cd0b53aad6e52da5a23655439
```

### 4.3. Các công cụ

Các công cụ được sử dụng trong bài:

* Python 3.14.7
* Wireshark 4.6.8 + Npcap
* Sysmon 15.22
* Autoruns 14.3
* Process Explorer 17.14

Các executable và installer không được đưa lên repository.

## 5. Các tình huống thực hành

### TH1 – Asset, Vulnerability, Threat, Risk và Attack

Nội dung:

* Xác định tài sản của máy ảo.
* Lập Risk Register.
* Phân loại 5 nhóm nguồn đe dọa.
* Giải thích mối quan hệ Asset → Vulnerability → Threat → Risk → Control.

**Trạng thái:** Đang thực hiện.

### TH2 – Malware

Nội dung:

* Kiểm tra Microsoft Defender.
* Sử dụng EICAR để kiểm chứng detection/quarantine.
* Thu thập bằng chứng từ Protection History.

**Trạng thái:** Chưa thực hiện.

### TH3 – Password và Authentication

Nội dung:

* Tạo tài khoản lab.
* Sinh các sự kiện đăng nhập thành công/thất bại.
* Phân tích Event ID 4624, 4625 và 4648.
* Đổi mật khẩu và kiểm chứng credential cũ/mới.

**Trạng thái:** Chưa thực hiện.

### TH4 – Persistence và Listener

Nội dung đã thực hiện:

* Cài và kiểm tra Sysmon.
* Thu baseline bằng Autoruns.
* Tạo `LAB3_Run_Demo`.
* Tạo `LAB3_Persistence_Demo`.
* Kiểm tra persistence sau logon.
* Tạo HTTP server chỉ bind `127.0.0.1:8080`.
* Kiểm tra cổng Listen và PID.
* Ánh xạ PID tới `python.exe`.
* Kiểm tra process bằng Process Explorer.

**Trạng thái:** PASS.

Các bằng chứng chính:

```text
H6_Sysmon_Event1.png
H7_Autoruns_LAB3_Run_Demo.png
H8_ProcessExplorer_Python.png
```

### TH5 – Sniffing, MITM và Spoofing

Nội dung:

* Capture HTTP loopback.
* Quan sát dữ liệu HTTP đọc được.
* So sánh HTTP với TLS/HTTPS.
* Không thực hiện ARP poisoning, DNS spoofing, session hijacking hoặc chèn chứng chỉ.

**Trạng thái:** Chưa thực hiện.

### TH6 – DoS, DDoS và Mail Bombing

Nội dung:

* Thực hiện local load test giới hạn trên `127.0.0.1:8080`.
* Phân tích dataset DDoS offline.
* Phân tích log mail bombing offline.
* Không tạo DDoS hoặc gửi email hàng loạt ra bên ngoài.

**Trạng thái:** Chưa thực hiện.

### TH7 – Social Engineering và Phishing

Nội dung:

* Phân tích mẫu phishing offline.
* Xác định các dấu hiệu phishing.
* Phân loại 6 trường hợp Social Engineering.
* Phân biệt Phishing, Spear Phishing, Watering Hole, Pretexting, Baiting và Quid Pro Quo.

**Trạng thái:** Chưa thực hiện.

## 6. Bằng chứng

Bằng chứng được lưu trong thư mục:

```text
LAB3/Evidence/
```

Ảnh chụp trực tiếp từ máy ảo được lưu trong:

```text
LAB3/Images/
```

Các tệp evidence không chứa mật khẩu, token, cookie/session hoặc dữ liệu cá nhân.

## 7. Kết quả

| Tình huống                   | Trạng thái     |
| ---------------------------- | -------------- |
| Chuẩn bị môi trường          | PASS           |
| Kiểm tra SHA-256 gói dữ liệu | PASS           |
| Baseline                     | PASS           |
| TH1                          | PASS           |
| TH2                          | PASS           |
| TH3                          | Đang thực hiện |
| TH4                          | PASS           |
| TH5                          | Chưa thực hiện |
| TH6                          | Chưa thực hiện |
| TH7                          | Chưa thực hiện |
| Cleanup                      | Chưa thực hiện |
| Verify                       | Chưa thực hiện |
| SHA-256 Evidence cuối        | Chưa thực hiện |

## 8. Lỗi gặp phải và cách khắc phục

### Lỗi 1 – Sysmon không xuất hiện trong Event Viewer

**Nguyên nhân:** Sysmon chưa được cài với quyền Administrator và đường dẫn file cấu hình ban đầu chưa đúng.

**Khắc phục:**

* Mở Windows PowerShell bằng `Run as administrator`.
* Xác định chính xác vị trí `sysmon-lab.xml`.
* Cài Sysmon bằng file cấu hình của bài lab.
* Kiểm tra service `Sysmon64`.
* Kiểm tra log `Microsoft-Windows-Sysmon/Operational`.

### Lỗi 2 – Đường dẫn `sysmon-lab.xml` không đúng

**Khắc phục:** sử dụng PowerShell để tìm chính xác file:

```powershell
Get-ChildItem C:\LAB3\Downloads -Filter sysmon-lab.xml -Recurse -File
```

Sau đó sử dụng đường dẫn thực tế khi cài Sysmon.

## 9. Quy tắc an toàn

Trong bài thực hành:

* Không sử dụng tài khoản hoặc mật khẩu thật.
* Không tắt Microsoft Defender hoặc Tamper Protection.
* Không tạo exclusion để vượt qua cơ chế bảo vệ.
* Không thực hiện DDoS, mail bombing hoặc MITM chủ động trên mạng bên ngoài VM.
* Traffic tạo tải chỉ giới hạn trên `127.0.0.1:8080`.
* Không đưa executable, installer hoặc file quarantine vào GitHub.

## 10. Cấu trúc repository

```text
LAB_AT_BMHTTT/
└── LAB3/
    ├── README.md
    ├── BaoCao_LAB3.docx
    ├── Evidence/
    └── Images/
```

## 11. Kế hoạch hoàn thiện

Sau khi hoàn thành toàn bộ tình huống:

1. Thu thập đầy đủ evidence và ảnh chụp.
2. Cleanup các artefact của bài lab.
3. Verify trạng thái hệ thống sau cleanup.
4. Tính SHA-256 cho toàn bộ Evidence.
5. Hoàn thiện báo cáo Word.
6. Cập nhật README với kết quả PASS/FAIL cuối cùng.
7. Kiểm tra repository trước khi nộp.
