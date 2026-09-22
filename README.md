# README – LAB3: Nhận diện và ứng phó các mối đe dọa đến an toàn thông tin

## 1. Thông tin sinh viên

- **Họ và tên:** [HỌ VÀ TÊN]
- **MSSV:** [MSSV]
- **Tên Lab:** LAB3 – Nhận diện và ứng phó các mối đe dọa đến an toàn thông tin
- **Môn học:** Thực hành An toàn Hệ thống Thông tin
- **Năm học:** 2026–2027

## 2. Phiên bản môi trường

| Thành phần | Phiên bản / cấu hình |
|---|---|
| VMware Workstation | 26H1 |
| Máy ảo | Windows 11 25H2 x64 |
| OS Build | 26200.9445 (KB5124008) |
| Microsoft Defender Antivirus | Tích hợp Windows 11 |
| PowerShell | 5.1 |
| Sysmon | 15.22 |
| Sysmon Schema | 4.90 |
| Autoruns | 14.3 |
| Process Explorer | 17.14 |
| Wireshark | 4.6.8 Stable |
| Npcap | Cài kèm Wireshark |
| Python | 3.14.7 |
| Network | VMware Host-only (mặc định) |

## 3. Cách dựng môi trường

### Bước 1 – Tạo máy ảo

- Tạo VM Windows 11 25H2 x64 trên VMware Workstation Pro 26H1.
- Khuyến nghị tối thiểu: **2 vCPU, 6 GB RAM, 64 GB ổ đĩa**.
- Cấu hình Network Adapter = **Host-only**.
- Cập nhật Windows đến **KB5124008 / OS Build 26200.9445**.
- Tạo snapshot sạch với tên: `LAB3_CLEAN_20260914`.

### Bước 2 – Tạo thư mục LAB3

Mở **PowerShell với quyền Administrator**:

```powershell
$Lab = 'C:\LAB3'
New-Item -ItemType Directory -Force `
  "$Lab\Evidence", "$Lab\Tools", "$Lab\Downloads", "$Lab\Assets" | Out-Null
Get-Date -Format 'yyyy-MM-dd HH:mm:ss zzz' | Out-File "$Lab\Evidence\start_time.txt"
```

### Bước 3 – Chuẩn bị gói dữ liệu

Sao chép `LAB3_Threats_Assets.zip` vào:

```text
C:\LAB3\Downloads
```

Kiểm tra SHA-256:

```powershell
Get-FileHash C:\LAB3\Downloads\LAB3_Threats_Assets.zip -Algorithm SHA256
```

SHA-256 theo tài liệu lab:

```text
96236f95ce59d0cc37f9b7ab8fbb04522f21870cd0b53aad6e52da5a23655439
```

Giải nén:

```powershell
Expand-Archive C:\LAB3\Downloads\LAB3_Threats_Assets.zip -DestinationPath C:\LAB3 -Force
```

### Bước 4 – Cài Python và Wireshark

```powershell
winget install --id Python.Python.3.14 --exact --version 3.14.7 `
  --accept-package-agreements --accept-source-agreements

winget install --id WiresharkFoundation.Wireshark --exact --version 4.6.8 `
  --accept-package-agreements --accept-source-agreements
```

Kiểm tra:

```powershell
python --version
& 'C:\Program Files\Wireshark\tshark.exe' --version | Select-Object -First 1
```

### Bước 5 – Chuẩn bị Sysinternals

Tải từ máy chủ Microsoft, gồm:

- Sysmon 15.22
- Autoruns 14.3
- Process Explorer 17.14

Sau khi tải và giải nén, đặt trong:

```text
C:\LAB3\Tools\
```

Kiểm tra đúng phiên bản trước khi thực hành.

## 4. Các tình huống đã thực hiện

| Tình huống | Nội dung | Kết quả |
|---|---|---|
| TH1 | Baseline, Asset → Vulnerability → Threat → Risk → Control và phân loại 5 nguồn đe dọa | **[PASS/FAIL]** |
| TH2 | EICAR và kiểm tra Microsoft Defender detection/quarantine | **[PASS/FAIL]** |
| TH3 | Đăng nhập thành công/thất bại, Event ID 4624/4625/4648 và đổi mật khẩu | **[PASS/FAIL]** |
| TH4 | Persistence lành tính, Autoruns, Sysmon và listener 127.0.0.1:8080 | **[PASS/FAIL]** |
| TH5 | Wireshark: so sánh HTTP plaintext và HTTPS/TLS | **[PASS/FAIL]** |
| TH6 | Local load giới hạn, phân tích DDoS dataset và mail bombing log offline | **[PASS/FAIL]** |
| TH7 | Phân tích phishing và Social Engineering offline | **[PASS/FAIL]** |
| Cleanup | Xóa artefact, kiểm tra lại hệ thống và khôi phục snapshot | **[PASS/FAIL]** |

### TH1 – Baseline và Risk Register

- Thu thập thông tin hệ điều hành, Defender, Firewall, Network và Process.
- Lập Risk Register với tối thiểu 5 tài sản/nguy cơ.
- Phân loại 5 tình huống thành:
  - Hành động vô ý
  - Hành động cố ý
  - Thảm họa tự nhiên
  - Lỗi kỹ thuật
  - Lỗi quản lý

**Kết quả:** [PASS/FAIL]

### TH2 – Malware / EICAR

- Kiểm tra Real-time Protection và Tamper Protection.
- Tạo file EICAR trong phạm vi LAB.
- Kiểm tra `Get-MpThreatDetection` và Protection History.
- Không tắt Defender và không tạo exclusion.

**Kết quả:** [PASS/FAIL]

### TH3 – Password và Authentication Logging

- Tạo tài khoản `lab3user` dành riêng cho LAB.
- Sinh đăng nhập thành công và thất bại có kiểm soát.
- Kiểm tra Event ID 4624, 4625, 4648.
- Đổi mật khẩu và kiểm chứng credential cũ/mới.

**Kết quả:** [PASS/FAIL]

### TH4 – Backdoor / Persistence

- Cài Sysmon và thu baseline Autoruns.
- Tạo persistence lành tính có tên `LAB3_*`.
- Kiểm tra bằng Autoruns và Sysmon.
- Tạo HTTP listener chỉ trên `127.0.0.1:8080`.
- Đối chiếu PID bằng Process Explorer.

**Kết quả:** [PASS/FAIL]

### TH5 – Sniffing / MITM / Spoofing

- Capture HTTP loopback bằng Wireshark.
- Quan sát chuỗi `TRAINING_ONLY` trong HTTP Request URI.
- Capture HTTPS/TLS trên cổng 443 để so sánh.
- Không thực hiện ARP poisoning, DNS spoofing, session hijacking hoặc MITM chủ động.

**Kết quả:** [PASS/FAIL]

### TH6 – DoS / DDoS / Mail Bombing

- Chạy `local_load_test.py` với tải giới hạn trên `127.0.0.1:8080`.
- Phân tích `ddos_sample.csv` offline.
- Phân tích `mailbomb_sample.csv` offline.
- Không tạo DDoS và không gửi email hàng loạt.

**Kết quả:** [PASS/FAIL]

### TH7 – Social Engineering / Phishing

- Phân tích `phishing_email.txt` offline.
- Nhận diện tối thiểu 5 dấu hiệu phishing.
- Phân loại 6 case trong `social_engineering_cases.csv`.
- Không truy cập domain trong mẫu và không sử dụng tài khoản thật.

**Kết quả:** [PASS/FAIL]

## 5. Bằng chứng và kết quả

Các bằng chứng được lưu trong thư mục:

```text
C:\LAB3\Evidence
```

Các file bằng chứng chính có thể gồm:

- `baseline_os.txt`
- `baseline_defender.txt`
- `baseline_firewall.txt`
- `baseline_network.txt`
- `baseline_processes.txt`
- `defender_eicar.txt`
- `auth_events_before_rotation.txt`
- `autoruns_before.csv`
- `sysmon_persistence.txt`
- `local_load_test.txt`
- `ddos_sources.txt`
- `mail_sender_counts.txt`
- `mail_volume.txt`
- `evidence_sha256.csv`

Ảnh chụp minh chứng cần được lấy trực tiếp từ VM/PC thực hiện LAB.

## 6. Lỗi gặp phải và cách khắc phục

| Lỗi / vấn đề | Nguyên nhân | Cách khắc phục | Kết quả |
|---|---|---|---|
| [Điền lỗi 1] | [Nguyên nhân] | [Cách khắc phục] | [Đã khắc phục/Chưa] |
| [Điền lỗi 2] | [Nguyên nhân] | [Cách khắc phục] | [Đã khắc phục/Chưa] |
| [Điền lỗi 3] | [Nguyên nhân] | [Cách khắc phục] | [Đã khắc phục/Chưa] |

> Nếu không gặp lỗi, ghi: **Không phát sinh lỗi đáng kể trong quá trình thực hiện LAB.**

## 7. Cleanup và phục hồi

Sau khi thu thập đầy đủ bằng chứng:

- Xóa `LAB3_Run_Demo`.
- Xóa `LAB3_Persistence_Demo`.
- Dừng HTTP server trên cổng 8080.
- Xóa tài khoản `lab3user`.
- Kiểm tra lại Microsoft Defender.
- Tạo `autoruns_after.csv` và `autoruns_diff.txt`.
- Tính SHA-256 cho các file Evidence.
- Revert VM về snapshot sạch `LAB3_CLEAN_20260914` hoặc snapshot do giảng viên quy định.

**Kết quả cleanup:** [PASS/FAIL]

## 8. Lưu ý khi đưa lên GitHub

Repository cần có thư mục:

```text
LAB3/
├── README.md
├── BaoCao/
├── Evidence/
└── evidence_sha256.csv
```

Không upload:

- Installer.
- Executable Sysinternals/Wireshark/Python.
- File bị Defender quarantine.
- Mật khẩu, token/API key.
- Cookie/session.
- Email hoặc dữ liệu cá nhân.
- Log chưa làm sạch.
- Thông tin định danh hệ thống thật.

## 9. Tóm tắt kết quả

| Nội dung | Kết quả |
|---|---|
| TH1 | [PASS/FAIL] |
| TH2 | [PASS/FAIL] |
| TH3 | [PASS/FAIL] |
| TH4 | [PASS/FAIL] |
| TH5 | [PASS/FAIL] |
| TH6 | [PASS/FAIL] |
| TH7 | [PASS/FAIL] |
| Cleanup & Recovery | [PASS/FAIL] |
| **Kết quả toàn LAB3** | **[PASS/FAIL]** |

