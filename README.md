# Kiểm Thử Hiệu Năng Website YouTube Với Apache JMeter

## 1. Giới thiệu về Apache JMeter

Apache JMeter là một phần mềm mã nguồn mở hoàn toàn bằng Java, được thiết kế chuyên dụng để kiểm thử tải (Load Testing), kiểm thử sức chịu đựng (Stress Testing) và đo lường hiệu năng (Performance Testing) của các ứng dụng web cũng như nhiều dịch vụ khác.

### Các tính năng chính của JMeter:
* **Hỗ trợ đa giao thức:** Kiểm thử hiệu năng trên nhiều loại ứng dụng và giao thức khác nhau như Web (HTTP, HTTPS), REST/SOAP Webservices, FTP, Cơ sở dữ liệu qua JDBC, LDAP, Mail (SMTP/POP3).
* **Mô phỏng tải lớn (Thread Group):** Cho phép giả lập hàng trăm, hàng nghìn người dùng ảo (Virtual Users) truy cập đồng thời vào hệ thống để đánh giá khả năng chịu tải.
* **Thiết kế kịch bản trực quan:** Giao diện đồ họa (GUI) giúp dễ dàng thêm các yêu cầu HTTP, truyền tham số, quản lý phiên đăng nhập (Cookie/Header) và thiết lập bộ đếm thời gian (Timers).
* **Xác minh kết quả tự động (Assertions):** Hỗ trợ kiểm tra tính đúng đắn của dữ liệu phản hồi từ máy chủ thông qua mã trạng thái (Response Code), nội dung văn bản, thời gian phản hồi.
* **Báo cáo và thống kê chi tiết (Listeners):** Cung cấp nhiều dạng biểu đồ và bảng số liệu trực quan như View Results Tree, Summary Report, Aggregate Report giúp phân tích thời gian phản hồi trung bình, độ lệch chuẩn, tỉ lệ lỗi (Error %) và thông lượng (Throughput).
* **Khả năng mở rộng và tích hợp CI/CD:** Hỗ trợ chạy qua dòng lệnh (CLI mode) và tích hợp dễ dàng vào quy trình kiểm thử tự động với Jenkins, GitHub Actions.

### Giao diện overview
![Giao diện tổng quan của JMeter](images/00_overview.png)

---

## 2. Thiết lập kịch bản kiểm thử với trang web YouTube

### 2.1. Cấu hình Thread Group (Người dùng ảo)
* **Mục đích:** Giả lập số lượng người dùng truy cập đồng thời vào trang web YouTube.
* **Thông số thiết lập:**
  * **Number of Threads (users):** 20 người dùng ảo.
  * **Ramp-up period:** 5 giây.
  * **Loop Count:** 2 lần lặp (tổng cộng 40 requests cho mỗi tác vụ).
![Cấu hình Thread Group](images/01_thread_group.png)

### 2.2. Yêu cầu HTTP GET - Truy cập trang chủ YouTube
* **Chức năng:** Kiểm tra tốc độ tải trang chủ YouTube.
* **Cấu hình:**
  * **Protocol:** `https`
  * **Server Name or IP:** `www.youtube.com`
  * **Method:** `GET`
  * **Path:** `/`
![HTTP Request Trang chủ YouTube](images/02_http_homepage.png)

### 2.3. Yêu cầu HTTP GET - Tìm kiếm video trên YouTube
* **Chức năng:** Kiểm thử chức năng tìm kiếm video với tham số truy vấn (`search_query`).
* **Cấu hình:**
  * **Protocol:** `https`
  * **Server Name or IP:** `www.youtube.com`
  * **Method:** `GET`
  * **Path:** `/results`
  * **Parameters:** `search_query = kiem thu phan mem jmeter`
![HTTP Request Tìm kiếm YouTube](images/03_http_search.png)

### 2.4. Thiết lập kiểm tra phản hồi (Response Assertion)
* **Chức năng:** Tự động xác minh mọi phản hồi trả về từ máy chủ YouTube đều đạt trạng thái thành công (`HTTP Status Code = 200`).
![Cấu hình Response Assertion](images/04_response_assertion.png)

---

## 3. Kết quả thực hiện kiểm thử

### 3.1. Kết quả chi tiết từng Request (View Results Tree)
* **Đánh giá:** Các yêu cầu gửi tới trang chủ và trang tìm kiếm YouTube đều hiển thị trạng thái xanh lá (PASS), mã phản hồi `200 OK` và trả về đầy đủ dữ liệu HTML.
![Kết quả View Results Tree](images/05_view_results_tree.png)

### 3.2. Báo cáo thống kê hiệu năng tổng hợp (Summary Report)
* **Các chỉ số chính được ghi nhận:**
  * **Samples:** Tổng số lượng request đã gửi đi.
  * **Average / Min / Max (ms):** Thời gian phản hồi trung bình, nhanh nhất và chậm nhất (tính bằng mili-giây).
  * **Error %:** Tỉ lệ phần trăm request bị lỗi (`0.00%`).
  * **Throughput:** Số lượng yêu cầu được máy chủ xử lý trên mỗi giây.
![Kết quả Summary Report](images/06_summary_report.png)

## 4. File JMeter và cách chạy

Kịch bản JMeter dạng XML được lưu tại [youtube-load-test.jmx](youtube-load-test.jmx). Kịch bản gồm 20 người dùng ảo, ramp-up 5 giây, 2 vòng lặp; mỗi vòng gọi trang chủ và tìm kiếm, do đó dự kiến có 80 mẫu (40 mẫu cho mỗi tác vụ). Có timer 1 giây giữa các request để hạn chế gửi tải quá dồn dập và assertion kiểm tra mã phản hồi 200 cho từng request.

Mở file `.jmx` bằng giao diện JMeter để kiểm tra cấu hình và chạy thử. Khi chạy tải, ưu tiên chế độ dòng lệnh; tạo thư mục báo cáo mới hoặc xóa báo cáo cũ trước khi chạy:

```powershell
jmeter -n -t .\youtube-load-test.jmx -l .\results.jtl -e -o .\jmeter-report
```

Kết quả thực tế phụ thuộc vào mạng, vùng truy cập và phản hồi/chính sách giới hạn tải của YouTube. Không giả định kết quả là 200 hay Error 0% trước khi chạy; xem `results.jtl` và báo cáo HTML được sinh ra để đánh giá. Chỉ chạy với mức tải và mục đích được phép; kịch bản này kiểm thử hai endpoint HTTP, không tải các tài nguyên nhúng hoặc phát video.