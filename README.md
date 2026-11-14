# Hệ thống quản lý tài sản và hoạt động nghiên cứu tại phòng thí nghiệm Vật lý — NST

Đây là ứng dụng web phục vụ học tập và trình diễn đồ án Cơ sở dữ liệu, được xây dựng bằng **Python + Flask** và **Microsoft SQL Server**.

## 1. Chức năng chính

- **Bảng điều khiển (Dashboard):** hiển thị số lượng phòng thí nghiệm, khu vực/bàn làm việc (Workstation), thiết bị, lượt mượn chưa hoàn tất và số lần truy cập bị từ chối.
- **Danh sách thiết bị (Equipment):** xem thiết bị, lọc theo phòng thí nghiệm/trạng thái và xem lịch sử bảo trì.
- **Tạo lượt mượn (Borrowing):** ghi nhận việc mượn thiết bị.
- **Đặt lịch sử dụng (Reservation):** tạo lịch đặt thiết bị hoặc Workstation. Các Trigger trong SQL Server kiểm tra khả năng sử dụng và phát hiện khoảng thời gian đặt bị trùng.
- **Ghi nhận yêu cầu truy cập (Access Attempt):** Flask chỉ tiếp nhận Person đang `ACTIVE`, xác định `GRANTED` (được phép) hoặc `DENIED` (bị từ chối) dựa trên trạng thái thiết bị và chứng nhận an toàn; lý do từ chối được ánh xạ theo danh mục mã. Trigger ở cơ sở dữ liệu kiểm tra độc lập và từ chối bản ghi `GRANTED` không hợp lệ.
- **Nhật ký truy cập (Access Logs):** xem các lần truy cập gần đây.

## 2. Cấu trúc thư mục dự án

```text
nst/
  app/                         Ứng dụng Flask, giao diện HTML và CSS
  sql/01_schema.sql            Tập lệnh DDL: tạo bảng, ràng buộc, chỉ mục,
                               khung nhìn (View), Trigger và Stored Procedure cho SQL Server
  sql/02_mock_data_bulk.sql    Tập lệnh tạo dữ liệu giả lập quy mô lớn
  sql/03_queries.sql           12 câu truy vấn minh họa
  sql/04_performance.sql       Bốn bài đo hiệu năng, mỗi bài warm-up + 5 lần đo
  sql/05_integrity_tests.sql   17 test case toàn vẹn/nghiệp vụ
  .env.example                 Tệp mẫu cấu hình kết nối và ứng dụng
  requirements.txt             Danh sách thư viện Python cần cài đặt
```

## 3. Yêu cầu phần mềm

- Windows 10/11 (khuyến nghị, phù hợp với môi trường SSMS của nhóm).
- Python phiên bản 3.10 trở lên.
- Microsoft SQL Server (Developer, Express hoặc LocalDB tùy phiên bản nhóm đã cài đặt).
- SQL Server Management Studio (SSMS).
- Microsoft ODBC Driver 18 for SQL Server.

## 4. Quy mô dữ liệu giả lập

Tập lệnh DML được thiết kế để thêm **167.000 dòng dữ liệu giả lập (synthetic data)** vào 15 bảng/quan hệ trên một cơ sở dữ liệu trống.

| Bảng/Quan hệ | Số dòng dự kiến |
|---|---:|
| PERSON | 15.000 |
| STUDENT | 9.000 |
| FACULTY | 3.000 |
| LAB_STAFF | 3.000 |
| LABORATORY | 5.000 |
| WORKSTATION | 12.000 |
| EQUIPMENT | 12.000 |
| RESEARCH_PROJECT | 12.000 |
| CERTIFICATION | 12.000 |
| MAINTENANCE_RECORD | 12.000 |
| PARTICIPATES_IN | 24.000 |
| BORROWS | 12.000 |
| RESERVES_EQUIPMENT | 12.000 |
| RESERVES_WORKSTATION | 12.000 |
| ACCESS_LOG | 12.000 |

Đây là dữ liệu kiểm thử được sinh tự động, **không phải dữ liệu khách hàng hoặc dữ liệu thu thập từ khảo sát thực tế**. Tập lệnh kiểm tra các bảng đích có trống hay không và in ra số dòng thực tế bằng `COUNT_BIG(*)` sau khi chèn dữ liệu.

## 5. Hướng dẫn cài đặt và chạy dự án

### Bước 1. Tạo cấu trúc cơ sở dữ liệu

1. Mở SSMS và kết nối đến phiên bản SQL Server mà nhóm sử dụng.
2. Mở tệp `sql/01_schema.sql` trong SSMS.
3. Thực thi tập lệnh. Tập lệnh tạo cơ sở dữ liệu `physics_lab_tracker`, các bảng, ràng buộc, chỉ mục, View và Trigger.

### Bước 2. Nạp dữ liệu giả lập

1. Mở tệp `sql/02_mock_data_bulk.sql` trong SSMS.
2. Chọn đúng cơ sở dữ liệu `physics_lab_tracker`.
3. Thực thi tập lệnh **một lần trên cơ sở dữ liệu mới tạo và đang trống**.
4. Kiểm tra kết quả thống kê số dòng ở cuối tập lệnh.

Tập lệnh nạp dữ liệu sử dụng transaction. Không chạy lại trên các bảng đã có dữ liệu nếu chưa làm sạch hoặc tạo lại cơ sở dữ liệu kiểm thử. Việc chạy lặp có thể gây lỗi do bảng không còn trống hoặc vi phạm khóa/ràng buộc.

### Bước 3. Chạy truy vấn và kiểm tra hiệu năng (không bắt buộc ở bước cài đặt ban đầu)

- Mở và thực thi `sql/03_queries.sql` để chạy 12 câu truy vấn minh họa.
- Mở `sql/05_integrity_tests.sql` để chạy các test case toàn vẹn/nghiệp vụ (TC-01 đến TC-17). Script dùng dữ liệu hiện có và rollback các dòng thử nghiệm.
- Mở `sql/04_performance.sql` để đo bốn truy vấn hiệu năng (PT-01 đến PT-04). Mỗi bài có một lần warm-up và năm lần đo; bật Actual Execution Plan bằng Ctrl+M trước khi chạy.

Kết quả thời gian thực thi phụ thuộc cấu hình máy, phiên bản SQL Server, bộ nhớ đệm và chỉ mục. Nhóm cần chạy thử trên máy thực tế và ghi nhận kết quả; không sử dụng số liệu giả định như kết quả đo đã xác nhận.

### Bước 4. Tạo môi trường Python

Cài Python 3.10 trở lên. Mở PowerShell tại thư mục gốc `nst`, sau đó chạy:

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
Copy-Item .env.example .env
```

Nếu PowerShell không cho phép kích hoạt môi trường ảo, có thể mở PowerShell với quyền phù hợp hoặc điều chỉnh Execution Policy chỉ trong phiên hiện tại theo hướng dẫn của quản trị viên máy tính.

### Bước 5. Cấu hình kết nối trong tệp `.env`

Mở tệp `.env` vừa tạo và kiểm tra các thông số:

- `SQL_SERVER=localhost` nếu dùng SQL Server instance mặc định; nếu dùng named instance, nhập dạng `localhost\SQLEXPRESS` hoặc đúng tên instance trên máy.
- `SQL_DATABASE=physics_lab_tracker`.
- Nếu dùng Windows Authentication, giữ `SQL_TRUSTED_CONNECTION=yes`. Tài khoản Windows chạy Flask phải có quyền truy cập cơ sở dữ liệu.
- Thay `FLASK_SECRET_KEY` bằng một chuỗi bí mật ngẫu nhiên riêng.
- `SQL_TRUST_SERVER_CERTIFICATE=yes` chỉ là lựa chọn thuận tiện cho môi trường phát triển cục bộ dùng chứng chỉ tự ký. Khi triển khai thực tế, cần cấu hình chứng chỉ đáng tin cậy và đặt giá trị này thành `no`.
- ZIP bàn giao không kèm `.env` thật. Hãy sao chép `.env.example` thành `.env` và tự điền cấu hình trên máy của bạn.

Không chia sẻ tệp `.env` và không đưa tệp này lên GitHub vì có thể chứa thông tin nhạy cảm.

### Bước 6. Khởi chạy ứng dụng Flask

Từ thư mục gốc dự án, khi môi trường ảo đã được kích hoạt, chạy:

```powershell
cd app
python app.py
```

Mở trình duyệt và truy cập:

```text
http://127.0.0.1:5000
```

## 6. Giới hạn và lưu ý quan trọng

- Đây là **bản mẫu phục vụ đồ án trên máy cục bộ**, chưa phải ứng dụng sẵn sàng đưa vào vận hành thực tế.
- Ứng dụng chưa triển khai chức năng đăng nhập và phân quyền người dùng ở tầng ứng dụng. Không công khai ứng dụng lên Internet.
- Chỉ chạy tập lệnh dữ liệu giả lập trên cơ sở dữ liệu kiểm thử sạch. Dữ liệu được sinh tự động, không đại diện cho kết quả khảo sát thực tế.
- Mô hình chuyên biệt hóa `PERSON` theo quy tắc **Total + Disjoint** (toàn phần + rời nhau). Các Trigger SQL kiểm tra tính rời nhau; việc bảo đảm mỗi Person thuộc đúng một subtype dựa vào quy trình thêm dữ liệu có kiểm soát/tập lệnh nạp dữ liệu. Không thêm riêng một dòng vào `PERSON` mà không tạo đúng một dòng subtype tương ứng trong cùng quy trình nghiệp vụ. Khi triển khai thực tế, cần hạn chế quyền ghi trực tiếp vào các bảng và buộc thao tác tạo Person đi qua Stored Procedure hoặc quy trình được kiểm soát.
- Ứng dụng được chuẩn bị để phù hợp với lược đồ quan hệ hiện tại của báo cáo.
