# LAB 4 – Khảo sát và đánh giá bề mặt mạng bằng Nmap

## 1. Giới thiệu

LAB 4 thuộc học phần **An toàn Hệ thống Thông tin**, tập trung vào việc sử dụng **Nmap** để khảo sát và đánh giá bề mặt mạng trong một môi trường máy ảo được cô lập.

Mục tiêu của bài lab là giúp người học:

- Hiểu mô hình Host/Guest trong môi trường máy ảo.
- Cài đặt và kiểm tra Nmap trên Windows 10/11 và Kali Linux.
- Thiết lập mạng **VirtualBox Host-Only**.
- Xác định địa chỉ IP của Kali Linux, Metasploitable 2 và máy Windows.
- Thực hiện host discovery.
- Khảo sát cổng TCP/UDP.
- Nhận diện dịch vụ và phiên bản.
- Thực hiện OS fingerprinting.
- Sử dụng một số NSE script phục vụ kiểm tra thông tin và rủi ro.
- Xuất kết quả quét thành các tệp bằng chứng.
- So sánh hệ thống trước và sau khi thực hiện hardening.
- Phân tích kết quả và viết báo cáo kỹ thuật có khả năng lặp lại.

> **Lưu ý:** Bài lab chỉ phục vụ học tập và nghiên cứu. Chỉ quét các máy ảo do chính mình quản lý trong mạng Host-Only hoặc các hệ thống được cho phép. Không quét hệ thống công cộng, mạng cơ quan, Wi-Fi của người khác hoặc dịch vụ Internet khi chưa có ủy quyền.

Nguồn: Tài liệu LAB 4 – Khảo sát và đánh giá bề mặt mạng bằng Nmap. fileciteturn0file0L69-L84

---

## 2. Môi trường thực hành

### 2.1. Thành phần

| Thành phần | Vai trò |
|---|---|
| Windows 10/11 | Máy thật/Host |
| VirtualBox | Nền tảng chạy máy ảo |
| Kali Linux | Máy quét |
| Metasploitable 2 | Máy đích cố ý có lỗ hổng |
| Host-Only Network | Mạng cô lập phục vụ thực hành |

Cấu hình khuyến nghị:

- CPU hỗ trợ Intel VT-x/AMD-V.
- RAM tối thiểu 8 GB, khuyến nghị 16 GB.
- Ổ đĩa trống khoảng 25–40 GB.
- VirtualBox tương thích với hệ điều hành.
- Kali Linux VM.
- Metasploitable 2 VM.

Metasploitable 2 chỉ nên kết nối vào **Host-Only Network**, không Bridge trực tiếp vào mạng thật. fileciteturn0file0L85-L108

### 2.2. Topology

Mô hình thực hành:

```text
                 Windows Host
                      |
          VirtualBox Host-Only Network
                 192.168.56.0/24
                 /              \
                /                \
        Kali Linux          Metasploitable 2
        Máy quét                 Máy đích
```

Tài liệu minh họa sử dụng:

```text
Kali Linux        : 192.168.56.10
Metasploitable 2  : 192.168.56.101
Network            : 192.168.56.0/24
```

Địa chỉ IP thực tế phải được kiểm tra trên chính môi trường máy ảo, không được mặc định theo sơ đồ. fileciteturn0file0L178-L195

---

## 3. Quy tắc an toàn

### Chỉ được quét

- Máy ảo do chính mình dựng.
- Máy mục tiêu trong mạng Host-Only.
- Hệ thống được giảng viên cho phép.

### Không được tự ý quét

- Website hoặc máy chủ trên Internet.
- Hệ thống công cộng.
- Mạng cơ quan.
- Wi-Fi của người khác.
- Hệ thống không thuộc quyền quản lý khi chưa có ủy quyền.

Mục tiêu của bài là **khảo sát và phân tích phòng thủ**, không khai thác hoặc phá hoại hệ thống. fileciteturn0file0L79-L84

---

## 4. Cài đặt Nmap

### 4.1. Windows

Theo tài liệu, bản cài Windows của Nmap có thể bao gồm:

- Nmap Core Files
- Npcap
- Zenmap
- Register Nmap Path

Quy trình tổng quát:

1. Truy cập trang tải Nmap chính thức.
2. Chọn bản **Latest stable release self-installer**.
3. Chạy trình cài đặt với quyền Administrator.
4. Giữ các thành phần Nmap, Npcap và Zenmap nếu được cung cấp.
5. Hoàn thành cài đặt Npcap.
6. Đóng và mở lại Terminal/PowerShell/Command Prompt.
7. Kiểm tra phiên bản Nmap.
8. Chụp màn hình kết quả để làm minh chứng.

Nguồn tải chính thức được tài liệu chỉ định là:

`nmap.org/download.html`

fileciteturn0file0L117-L139

### 4.2. Kali Linux

Trên Kali Linux:

1. Khởi động Kali VM.
2. Có thể tạm thời sử dụng NAT để cập nhật gói.
3. Kiểm tra/cài Nmap bằng `apt`.
4. Kiểm tra phiên bản Nmap.
5. Chụp màn hình kết quả.
6. Sau khi cài đặt xong, ngắt NAT trước khi thực hiện các bài quét.
7. Chỉ giữ Host-Only Adapter khi bắt đầu lab.

Zenmap là thành phần tùy chọn; phần chính của bài lab vẫn thực hiện bằng dòng lệnh Nmap. fileciteturn0file0L150-L174

---

## 5. Thiết lập mạng Host-Only

### Kali Linux

- Adapter 1: Host-Only Adapter.
- NAT chỉ sử dụng tạm thời khi cần cập nhật.

### Metasploitable 2

- Chỉ sử dụng Host-Only Adapter.
- Không Bridge vào mạng thật.

### Trước khi quét

Kiểm tra:

- IP của Kali.
- IP của Metasploitable 2.
- Subnet mask.
- Hai máy cùng Host-Only Network.
- Không bị trùng IP.
- Máy đích đang bật.
- Kết nối giữa máy quét và máy đích hoạt động.

Nếu ping thất bại, phải kiểm tra cấu hình mạng trước khi chuyển sang bước quét. fileciteturn0file0L185-L214

---

# 6. Nội dung thực hành

## 6.1. Host Discovery

Mục tiêu:

- Phát hiện các host đang hoạt động trong dải mạng.
- Ghi nhận IP.
- Ghi MAC/Vendor nếu Nmap cung cấp.
- Đối chiếu kết quả với số lượng VM đang bật.

Phương pháp trong bài sử dụng:

```text
Host discovery (-sn)
```

Kết quả cần ghi:

| STT | IP | MAC/Vendor | Vai trò suy đoán | Bằng chứng |
|---|---|---|---|---|
| 1 | ... | ... | ... | ... |
| 2 | ... | ... | ... | ... |
| 3 | ... | ... | ... | ... |

fileciteturn0file0L215-L233

---

## 6.2. TCP Connect Scan

Kỹ thuật:

```text
-sT
```

Yêu cầu:

- Ghi tổng số cổng `open`.
- Ghi tổng số cổng `closed`.
- Ghi tổng số cổng `filtered`.
- Chọn ít nhất 3 cổng mở.
- Ghi dịch vụ Nmap suy đoán.
- Giải thích vì sao `-sT` không cần raw packet privileges như `-sS`.

fileciteturn0file0L234-L239

---

## 6.3. SYN Scan

Kỹ thuật:

```text
-sS
```

Kỹ thuật này thường yêu cầu quyền cao hơn trên hệ thống.

So sánh với `-sT`:

| Tiêu chí | -sT | -sS | Nhận xét |
|---|---|---|---|
| Open | ... | ... | ... |
| Closed | ... | ... | ... |
| Filtered | ... | ... | ... |
| Thời gian | ... | ... | ... |
| Quyền cần thiết | ... | ... | ... |

fileciteturn0file0L240-L251

---

## 6.4. FIN / Xmas / NULL Scan

Ba kỹ thuật:

```text
FIN
Xmas
NULL
```

Mục đích là quan sát phản ứng TCP và sự khác biệt giữa hệ điều hành hoặc thiết bị lọc.

Chỉ thực hiện trong môi trường lab được phép.

### Lưu ý khi phân tích

Không được kết luận:

```text
open|filtered = open
```

`open|filtered` có nghĩa Nmap không thể phân biệt chắc chắn giữa cổng mở và cổng bị bộ lọc làm im lặng.

Kết quả phụ thuộc vào TCP/IP stack và firewall của máy đích. fileciteturn0file0L252-L262

---

## 6.5. ACK Scan

Kỹ thuật:

```text
ACK scan
```

ACK scan chủ yếu dùng để quan sát trạng thái:

- `filtered`
- `unfiltered`

Không sử dụng ACK scan để khẳng định một cổng đang `open`.

Nên so sánh kết quả ACK scan với SYN scan trên cùng một máy đích. fileciteturn0file0L263-L268

---

# 7. UDP Scan

UDP scan thường:

- Chậm hơn TCP scan.
- Có thể xuất hiện nhiều trạng thái `open|filtered`.
- Không có cơ chế bắt tay như TCP.

Trong lab, chỉ quét nhóm cổng UDP phổ biến để tiết kiệm thời gian.

| Cổng UDP | Trạng thái | Dịch vụ | Giải thích |
|---|---|---|---|
| ... | ... | ... | ... |
| ... | ... | ... | ... |
| ... | ... | ... | ... |

fileciteturn0file0L269-L279

---

# 8. Nhận diện dịch vụ và hệ điều hành

## 8.1. Version Detection

Kỹ thuật:

```text
-sV
```

Mục tiêu:

- Xác định dịch vụ.
- Xác định phiên bản dịch vụ.
- Ghi nhận rủi ro liên quan đến phiên bản.

Các cổng được tài liệu yêu cầu quan sát:

| Port | Protocol | Service | Version Nmap phát hiện | Ghi chú rủi ro |
|---|---|---|---|---|
| 21 | TCP | FTP | ... | ... |
| 22 | TCP | SSH | ... | ... |
| 80 | TCP | HTTP | ... | ... |
| 445 | TCP | SMB | ... | ... |
| 3306 | TCP | MySQL | ... | ... |

fileciteturn0file0L280-L290

## 8.2. OS Detection

Kỹ thuật:

```text
-O
```

Mục đích:

- Fingerprinting hệ điều hành.
- Đánh giá loại hệ điều hành mà Nmap suy đoán.

Kết quả `-O` không nên được coi là tuyệt đối; cần phân tích cùng các thông tin khác.

## 8.3. Aggressive Scan

Kỹ thuật:

```text
-A
```

Tài liệu yêu cầu ghi nhận các thành phần mà `-A` cung cấp:

- Version detection.
- OS detection.
- Traceroute.
- Default scripts.

Cần so sánh lượng thông tin và thời gian với `-sV` hoặc `-O` riêng lẻ.

Một scan cung cấp nhiều thông tin thường tạo nhiều lưu lượng hơn và có khả năng dễ bị phát hiện hơn. fileciteturn0file0L294-L300

---

# 9. NSE – SMB và MS17-010

## 9.1. Giới hạn

NSE chỉ được chạy trên:

- Metasploitable 2 của chính sinh viên.
- Windows VM do chính sinh viên quản lý.

Mục tiêu là **nhận biết dấu hiệu rủi ro và đề xuất phòng thủ**, không khai thác lỗ hổng. fileciteturn0file0L301-L304

## 9.2. SMB OS Discovery

Mục tiêu:

- Thu thập thông tin SMB.
- Ghi hệ điều hành.
- Ghi tên máy/domain nếu script trả về.

Nếu cổng 445 đóng hoặc bị lọc, cần giải thích vì sao script không thể thu thập thông tin. fileciteturn0file0L305-L308

## 9.3. Kiểm tra MS17-010

Chỉ kết luận:

> Có dấu hiệu dễ bị ảnh hưởng

khi script thực sự báo `VULNERABLE`.

Nếu script báo:

- timeout,
- không kết nối được,
- không xác định,

thì **không được tự suy diễn rằng hệ thống đã được vá**. fileciteturn0file0L309-L313

---

# 10. Xuất kết quả

Bài lab yêu cầu xây dựng hồ sơ bằng chứng từ kết quả Nmap.

Các dạng kết quả:

### Normal text

Lưu kết quả dưới dạng văn bản.

### XML

Lưu kết quả dưới dạng XML để phục vụ xử lý và chuyển đổi.

### Grepable

Có thể lưu kết quả dạng grepable và lọc nhanh thông tin cần thiết.

Ví dụ trong tài liệu sử dụng quy trình:

```text
Nmap → tạo smb.txt → grep thông tin cổng 445
```

Phải tạo tệp trước khi thực hiện bước lọc.

### HTML

Có thể chuyển kết quả XML thành HTML bằng `xsltproc`.

fileciteturn0file0L325-L335

---

# 11. Before / After Hardening

Mục tiêu là chứng minh hiệu quả của một thay đổi phòng thủ bằng kết quả quét trước và sau.

Quy trình:

1. Chọn Windows VM hoặc dịch vụ test trong Host-Only.
2. Chạy `-sV` và lưu kết quả **Before**.
3. Thực hiện một thay đổi phòng thủ hợp pháp.
4. Chạy lại đúng câu lệnh.
5. Lưu kết quả **After**.
6. So sánh kết quả.
7. Giải thích nguyên nhân thay đổi.
8. Khôi phục snapshot nếu cần.

Ví dụ thay đổi phòng thủ:

- Tắt dịch vụ test.
- Đóng firewall rule.
- Cập nhật cấu hình mạng.

Bảng phân tích:

| Chỉ tiêu | Before | After | Giải thích |
|---|---|---|---|
| Số cổng open | ... | ... | ... |
| Số cổng filtered | ... | ... | ... |
| Dịch vụ bị thay đổi | ... | ... | ... |
| Tác động an toàn | ... | ... | ... |

fileciteturn0file0L339-L355

---

# 12. Ảnh minh chứng bắt buộc

Bài lab yêu cầu tối thiểu các ảnh:

1. `ip -br addr` trên Kali, thể hiện interface Host-Only và IP.
2. Thông tin IP trên Metasploitable 2.
3. Kết quả host discovery `-sn`.
4. Kết quả `-sS` hoặc `-sT`.
5. Kết quả `-sV`.
6. Kết quả `-O` hoặc `-A`.
7. Một NSE script và kết luận dựa trên output.
8. Tệp kết quả `.txt`, `.xml` và/hoặc HTML nếu có.

fileciteturn0file0L356-L365

---

# 13. Câu hỏi phân tích

Bài lab yêu cầu trả lời các câu hỏi:

1. Phân biệt `open`, `closed` và `filtered`.
2. Vì sao `-sS` thường cần quyền cao hơn `-sT`?
3. Vì sao FIN/Xmas/NULL có thể khó diễn giải trên một số hệ điều hành hoặc firewall?
4. ACK scan trả lời câu hỏi gì khác SYN scan?
5. Vì sao UDP scan thường chậm và dễ xuất hiện `open|filtered`?
6. `-sV` có vai trò gì trong quản lý lỗ hổng?
7. OS fingerprinting có những giới hạn nào?
8. NSE timeout có đồng nghĩa với việc không có lỗ hổng không?
9. Thay đổi nào trong Before/After chứng minh hardening có hiệu lực?
10. Nêu ba cấu hình phòng thủ giúp giảm bề mặt tấn công mà không dựa vào việc “ẩn mình” trước Nmap.

fileciteturn0file0L366-L394

---

# 14. Bài tập bổ sung

## 14.1. Bài tập theo yêu cầu

- So sánh `-sT`, `-sS` và `-sA` trên cùng Metasploitable 2.
- Quét toàn bộ 65535 cổng và so sánh với lần quét mặc định.
- Sử dụng `-sV`, xác định một dịch vụ có phiên bản cũ và tra cứu CVE tương ứng nếu có.
- Kiểm tra `smb-vuln-ms17-010` trên Metasploitable 2 và Windows đã cập nhật nếu có.
- So sánh quét có `-D` với quét thông thường và quan sát ý nghĩa phòng thủ.
- Xuất kết quả bằng `-oA` và so sánh các tệp `.nmap`, `.xml`, `.gnmap`.

fileciteturn0file0L395-L411

## 14.2. Lập bản đồ dịch vụ

Yêu cầu:

- Khảo sát toàn bộ dải `192.168.56.0/24`.
- Xác định host đang hoạt động.
- Quét dịch vụ bằng `-sV`.
- Tổng hợp host, port và service.
- Xác định 3 dịch vụ có rủi ro cần chú ý.
- Đề xuất biện pháp khắc phục.

fileciteturn0file0L412-L416

---

# 15. Cấu trúc thư mục đề xuất

Có thể tổ chức kết quả lab như sau:

```text
LAB4-Nmap/
│
├── README.md
│
├── screenshots/
│   ├── 01-kali-ip.png
│   ├── 02-metasploitable-ip.png
│   ├── 03-host-discovery.png
│   ├── 04-tcp-scan.png
│   ├── 05-service-version.png
│   ├── 06-os-detection.png
│   └── 07-nse.png
│
├── results/
│   ├── scan.nmap
│   ├── scan.xml
│   ├── scan.gnmap
│   └── scan.html
│
└── report/
    └── LAB4-Report.pdf
```

> Tên file có thể thay đổi tùy cách tổ chức bài nộp.

---

# 16. Checklist trước khi nộp

- [ ] Kali Linux đã kết nối Host-Only.
- [ ] Metasploitable 2 chỉ kết nối Host-Only.
- [ ] Đã kiểm tra IP thực tế của các máy.
- [ ] Đã kiểm tra kết nối trước khi quét.
- [ ] Đã thực hiện host discovery.
- [ ] Đã thực hiện TCP scan.
- [ ] Đã so sánh `-sT` và `-sS`.
- [ ] Đã thực hiện UDP scan có kiểm soát.
- [ ] Đã thực hiện `-sV`.
- [ ] Đã thực hiện `-O` hoặc `-A`.
- [ ] Đã thực hiện NSE trong phạm vi lab.
- [ ] Đã lưu kết quả quét.
- [ ] Đã chụp đủ ảnh minh chứng.
- [ ] Đã thực hiện Before/After hardening nếu được yêu cầu.
- [ ] Đã trả lời các câu hỏi phân tích.
- [ ] Đã kiểm tra lại tính chính xác của IP và output.
- [ ] Không quét hệ thống ngoài phạm vi được phép.

---

# 17. Tài liệu tham khảo

1. Nmap Project – Download the Free Nmap Security Scanner for Linux/Mac/Windows.
2. Nmap Project – Windows: Nmap Network Scanning.
3. Nmap Project – Nmap Reference Guide.
4. Kali Linux – nmap | Kali Linux Tools.
5. Kali Linux Documentation – Kali inside VirtualBox (Guest VM).
6. Rapid7 – Metasploitable 2.
7. Microsoft Security Bulletin MS17-010.

Các nguồn trên được liệt kê trong tài liệu LAB 4. fileciteturn0file0L431-L444

---

## 18. Kết luận

LAB 4 tập trung vào quy trình khảo sát bề mặt mạng trong một môi trường được cô lập: từ xây dựng mạng Host-Only, xác định host, khảo sát cổng, nhận diện dịch vụ/hệ điều hành, kiểm tra thông tin bằng NSE cho đến lưu trữ bằng chứng và đánh giá thay đổi sau hardening.

Kết quả quan trọng không chỉ là danh sách các cổng mở mà còn là khả năng **đọc trạng thái Nmap, giải thích giới hạn của từng kỹ thuật, liên hệ kết quả với rủi ro và đề xuất biện pháp phòng thủ**.
