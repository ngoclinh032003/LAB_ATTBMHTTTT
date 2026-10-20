# LAB 7 -- pfSense Web Protection

## 1. Giới thiệu

Tài liệu này mô tả nội dung bài thực hành **LAB 7 -- pfSense Web
Protection**: mục tiêu, mô hình mạng, các chính sách bảo vệ web cần kiểm
tra, cách ghi nhận kết quả và cấu trúc báo cáo lời giải chi tiết.

> **Lưu ý về hình ảnh:** Các hình trong báo cáo Word đi kèm là hình minh
> họa kết quả dự kiến, không phải ảnh chụp thật từ máy ảo. Khi nộp bài,
> hãy thay hoặc bổ sung bằng ảnh chụp màn hình thực tế từ môi trường lab
> và log tương ứng.

## 2. Mục tiêu

-   Hiểu vai trò của pfSense trong kiểm soát truy cập web.
-   Kiểm tra các chính sách cho phép/chặn truy cập theo domain hoặc danh
    sách.
-   Đánh giá tác động của DNS, firewall, proxy và các chính sách bảo vệ
    web được cấu hình trong bài lab.
-   Ghi nhận kết quả kiểm thử và giải thích vì sao từng trường hợp được
    cho phép, bị chặn hoặc hiển thị cảnh báo.

## 3. Thành phần môi trường

Môi trường triển khai phụ thuộc vào bộ lab và cấu hình máy ảo đang sử
dụng. Thông thường cần có:

-   **pfSense:** thiết bị tường lửa/cổng mạng để áp dụng chính sách.
-   **Máy khách:** máy dùng để thực hiện các truy cập web kiểm thử.
-   **Máy chủ/DNS hoặc dịch vụ phụ trợ:** chỉ sử dụng nếu có trong sơ đồ
    lab.
-   **Trình duyệt và công cụ kiểm tra mạng:** dùng để thử truy cập, xác
    minh kết quả và thu thập bằng chứng.

Hãy điền thông tin thật của môi trường trước khi nộp:

  Thông tin                 Giá trị thực tế
  ------------------------- ------------------------------
  Phiên bản pfSense         Điền phiên bản
  Nền tảng máy ảo           VirtualBox / VMware / khác
  IP LAN của pfSense        Điền địa chỉ
  IP máy khách              Điền địa chỉ
  DNS máy khách sử dụng     Điền địa chỉ
  Proxy/Package liên quan   Điền tên và phiên bản nếu có

## 4. Các nhóm kiểm thử

Dùng đúng các kịch bản trong tài liệu hướng dẫn/bộ đề của lớp. Với mỗi
kịch bản, ghi lại cấu hình, thao tác kiểm thử, kết quả quan sát và kết
luận.

1.  **Kiểm tra kết nối cơ bản:** xác nhận máy khách có kết nối mạng theo
    đúng sơ đồ.
2.  **Kiểm tra DNS:** xác minh phân giải tên miền và xác định liệu kết
    quả bị ảnh hưởng bởi chính sách DNS hay không.
3.  **Kiểm tra chính sách firewall/proxy:** xác định lưu lượng có đi qua
    thành phần kiểm soát hay không.
4.  **Kiểm tra chặn truy cập:** thử domain hoặc URL thuộc danh sách bị
    chặn; ghi lại trang thông báo, lỗi trình duyệt hoặc log.
5.  **Kiểm tra danh sách cho phép (whitelist):** xác nhận địa chỉ được
    cho phép có truy cập thành công hay không.
6.  **Kiểm tra chính sách theo thời gian:** nếu bài lab có cấu hình
    lịch, kiểm tra trong và ngoài khoảng thời gian áp dụng.
7.  **Kiểm tra antivirus/lọc nội dung:** chỉ thực hiện nếu thành phần
    này được yêu cầu và đã cài đặt/cấu hình trong môi trường.
8.  **Kiểm tra log:** đối chiếu thời gian, địa chỉ nguồn, đích và hành
    động cho phép/chặn.

Không tự kết luận một tính năng đã hoạt động chỉ dựa vào cấu hình đã
lưu. Cần có phép thử và bằng chứng quan sát được.

## 5. Mẫu ghi nhận kết quả

Sao chép bảng dưới đây cho từng kịch bản:

  Trường               Nội dung cần ghi
  -------------------- -------------------------------------------------
  Mã kịch bản          Ví dụ: TC-01
  Mục tiêu             Chính sách cần xác minh
  Cấu hình liên quan   Rule, danh sách, DNS hoặc proxy đã thiết lập
  Dữ liệu kiểm thử     Domain/URL hoặc thao tác đã thử
  Kết quả mong đợi     Được phép / bị chặn / cảnh báo
  Kết quả thực tế      Ghi đúng thông báo hoặc hành vi quan sát được
  Bằng chứng           Tên ảnh chụp màn hình và/hoặc log
  Kết luận             Đạt / chưa đạt / chưa thể xác minh
  Giải thích           Nêu nguyên nhân dựa trên cấu hình và bằng chứng

## 6. Quy tắc chụp và đặt tên bằng chứng

-   Chụp ảnh rõ cửa sổ, địa chỉ truy cập hoặc trang kết quả và phần cấu
    hình/log có liên quan.
-   Không chỉnh sửa nội dung ảnh theo cách làm thay đổi kết quả kiểm
    thử.
-   Tránh để lộ mật khẩu, token hoặc thông tin cá nhân.
-   Đặt tên nhất quán, ví dụ:
    -   `TC01_network_connectivity.png`
    -   `TC02_dns_resolution.png`
    -   `TC03_blocked_site.png`
    -   `TC04_whitelist_allowed.png`
    -   `TC05_firewall_or_proxy_log.png`
-   Mỗi ảnh nên có chú thích ngắn: ảnh chứng minh điều gì, được chụp ở
    máy nào và thuộc kịch bản nào.

## 7. Cấu trúc thư mục đề xuất

``` text
LAB7-pfSense-Web-Protection/
├── README.md
├── report/
│   └── LAB7_LoiGiai_ChiTiet_Kem_HinhMinhHoa.docx
├── evidence/
│   ├── TC01_network_connectivity.png
│   ├── TC02_dns_resolution.png
│   ├── TC03_blocked_site.png
│   ├── TC04_whitelist_allowed.png
│   └── logs/
└── notes/
    └── environment.txt
```

Tên file và thư mục có thể thay đổi theo cách tổ chức thực tế. Chỉ đưa
vào thư mục `evidence/` các ảnh/log được thu thập thật trong quá trình
thực hành.

## 8. Đánh giá kết quả

Một kịch bản chỉ nên đánh dấu **Đạt** khi kết quả thực tế phù hợp với
kết quả mong đợi và có bằng chứng hỗ trợ.

-   **Đạt:** chính sách hoạt động như yêu cầu và có ảnh/log xác nhận.
-   **Chưa đạt:** kết quả thực tế khác kết quả mong đợi.
-   **Chưa thể xác minh:** chưa chạy thử, thiếu log/ảnh hoặc môi trường
    chưa có thành phần cần thiết.

Nếu truy cập bị chặn, cần phân biệt nguyên nhân có thể đến từ DNS,
firewall, proxy, danh sách lọc hoặc lỗi kết nối. Không nên quy kết cho
một thành phần khi chưa có bằng chứng.

## 9. Khắc phục sự cố cơ bản

-   **Không truy cập được bất kỳ website nào:** kiểm tra IP, gateway,
    DNS và rule firewall trước.
-   **Domain vẫn truy cập được dù đã thêm vào danh sách chặn:** xác minh
    danh sách đã được áp dụng đúng, kiểm tra proxy/DNS đang sử dụng và
    xem log.
-   **Whitelist không có tác dụng:** kiểm tra thứ tự/độ ưu tiên chính
    sách và điều kiện khớp của rule.
-   **Không thấy log:** xác minh tính năng ghi log đã bật, chọn đúng
    giao diện/thành phần và khoảng thời gian.
-   **Kết quả khác giữa HTTP và HTTPS:** ghi rõ giao thức, cơ chế lọc
    đang dùng và giới hạn của cấu hình; không giả định mọi nội dung
    HTTPS đều được kiểm tra như HTTP.

Chỉ thay đổi cấu hình trong môi trường lab được phép. Sau khi chỉnh sửa,
chạy lại phép thử và lưu bằng chứng mới.

## 10. Tài liệu đi kèm

-   `LAB7_LoiGiai_ChiTiet_Kem_HinhMinhHoa.docx` -- báo cáo lời giải chi
    tiết và hình minh họa kết quả dự kiến.
-   Tài liệu đề bài/hướng dẫn LAB 7 do giảng viên cung cấp -- dùng để
    đối chiếu đúng yêu cầu và thứ tự kịch bản.

## 11. Checklist trước khi nộp

-   [ ] Đã điền phiên bản và thông tin mạng thực tế.
-   [ ] Đã chạy đủ các kịch bản được yêu cầu trong đề.
-   [ ] Đã thay/bổ sung hình minh họa bằng ảnh chụp thực tế.
-   [ ] Đã ghi kết quả thực tế, không chỉ ghi kết quả mong đợi.
-   [ ] Đã đối chiếu log với từng kết luận.
-   [ ] Đã trả lời đầy đủ các câu hỏi cuối bài theo đề.
-   [ ] Đã kiểm tra tên file, bố cục và thông tin nhạy cảm trước khi
    nộp.
